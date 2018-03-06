When pods are created from a deployment for an application, they are each given a name which incorporates a random component. This means you cannot know what the name of a pod, and thus the hostname, is in advance.

In order to be able to communicate with the pods for an application, you would create a _Service_. This has a fixed hostname and IP address. That a set of pods are associated with a service is through the matching of a label selector on the service description, with the pods.

When you connect using the hostname for the service, the connection will be randomly routed through to one of the pods.

This mechanism for providing access to pods is suitable where a client doesn't care which pod they communicate with. It is not suitable where a client needs to be able to separately address each pod. It is also not suitable where a client needs to be able to dynamically discover the pods associated with an application, including being able to recognise when additional pods are added due to the application being scaled up.

For the case where the number of pods is static, an alternative is that rather than use a _DeploymentConfig_, use a _StatefulSet_. This method allows the number of replicas to be specified, but where each pod will be named not with a random value, but with the instance count, from 1 up to the number of replicas. A static configuration could then be used in a client to identify each pod.

This method doesn't help where the replica count is dynamic, such as when an auto scaler is being used, and the client needs to know about each pod. A solution for this case is to use what is called a headless service. When this is used, the DNS server will be configured with a [SRV](https://en.wikipedia.org/wiki/SRV_record) record which can be queried to get a list of all the pods.

To create a headless service definition, you can use the ``oc expose`` command against your deployment configuration.

```
oc expose dc/your-app-name \
 --name your-app-name-headless --cluster-ip=None
```

If you already have a service for your application with the same name as the deployment configuration, you must use the ``--name`` option to give the headless service definition a different name.

From within the cluster, a client would then be able to query the DNS SRV record entries using the name of the headless service definition.

```
>>> import dns
>>> for item in dns.resolver.query('your-app-name-headless', 'SRV'): item
...
<DNS IN SRV rdata: 10 25 0 2555bb89.your-app-name-headless.your-project.svc.cluster.local.>
<DNS IN SRV rdata: 10 25 0 830bdb18.your-app-name-headless.your-project.svc.cluster.local.>
<DNS IN SRV rdata: 10 25 0 aeb532de.your-app-name-headless.your-project.svc.cluster.local.>
<DNS IN SRV rdata: 10 25 0 a432c21f.your-app-name-headless.your-project.svc.cluster.local.>
```

If a client is relying on the DNS SRV record entries to also determine the port a service is using, you can query based on service/protocol and name.

```
_service._proto.name
```

The service names assigned to the ports will be ``port-1``, ``port-2``, up to ``port-n``, for however many ports are defined by the deployment.

```
>> for item in dns.resolver.query('_port-1._tcp.blog-django-py-metrics', 'SRV'): item
...
<DNS IN SRV rdata: 10 25 8080 2555bb89.your-app-name-headless.your-project.svc.cluster.local.>
<DNS IN SRV rdata: 10 25 8080 830bdb18.your-app-name-headless.your-project.svc.cluster.local.>
<DNS IN SRV rdata: 10 25 8080 aeb532de.your-app-name-headless.your-project.svc.cluster.local.>
<DNS IN SRV rdata: 10 25 8080 a432c21f.your-app-name-headless.your-project.svc.cluster.local.>

>>> for item in dns.resolver.query('_port-2._tcp.blog-django-py-metrics', 'SRV'): item
...
<DNS IN SRV rdata: 10 25 8081 2555bb89.your-app-name-headless.your-project.svc.cluster.local.>
<DNS IN SRV rdata: 10 25 8081 830bdb18.your-app-name-headless.your-project.svc.cluster.local.>
<DNS IN SRV rdata: 10 25 8081 aeb532de.your-app-name-headless.your-project.svc.cluster.local.>
<DNS IN SRV rdata: 10 25 8081 a432c21f.your-app-name-headless.your-project.svc.cluster.local.>
```

This latter form is used by applications such as Prometheus. In the case of Prometheus, it allows it to automatically discover pods for an application without the Prometheus configuration needing to be changed when the name and number of pods changes.

```
scrape_configs:
  - job_name: 'your-app-name'
    dns_sd_configs:
      - names: ['_port-2._tcp.your-app-name-headless.your-project.svc.cluster.local']
```

Note, don't use the ``--port`` option to ``oc expose`` when creating a headless service. This is because in that case it doesn't add a name to the port and the service/protocol DNS SRV record entries aren't created as a result. If you need to restrict the headless service definition to a specific port, generate a raw service definition by running:

```
oc expose dc/your-app-name \
 --name your-app-name-headless --cluster-ip=None \
 --port 8081 -o json --dry-run
```

Edit the result to add a name to the port and use ``oc create -f`` to create the service definition from the raw service definition.
