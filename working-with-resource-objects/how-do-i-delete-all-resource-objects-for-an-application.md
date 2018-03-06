Resource objects can be deleted using the ``oc delete`` command from the command line. You can delete a single resource object by name, or delete a set of resource objects by specifying a label selector.

When an application is deployed, resource objects for that application will typically have an ``app`` label applied to them with value corresponding to the name of the application. This can be used with the label selector to delete all resource objects for an application.

To test what resource objects would be deleted when using a label selector, use the ``oc get`` command to query the set of objects which would be matched.

```
$ oc get all --selector app=nbviewer -o name
deploymentconfigs/nbviewer
routes/nbviewer
pods/nbviewer-1-wgzdb
replicationcontrollers/nbviewer-1
services/nbviewer
```

If you are satisfied that what is shown are the resource objects for your application, then run ``oc delete``.

```
$ oc delete all --selector app=nbviewer
```

Note that ``all`` matches on a subset of all resource object types that exist. It targets the core resource objects that would be created for a build and deployment. It will not include resource objects such as persistent volume claims (``pvc``), config maps (``configmap``), secrets (``secret``), and others.

You will either need to delete these resource objects separately, or if they also have been labelled with the ``app`` tag, list the resource object types along with ``all``.

```
$ oc delete all,configmap,pvc,serviceaccount,rolebinding --selector app=jupyterhub
```

If you are not sure what labels have been applied to resource objects for your application, you can run ``oc describe`` on the resource object to see the labels applied to it.

```
$ oc describe route/nbviewer
Name:                   nbviewer
Namespace:              nbviewer-quickstart
Created:                10 minutes ago
Labels:                 app=nbviewer
Annotations:            openshift.io/host.generated=true
Requested Host:         ...
                          exposed on router router (host ...) 10 minutes ago
Path:                   <none>
TLS Termination:        edge
Insecure Policy:        Redirect
Endpoint Port:          8080-tcp

Service:        nbviewer
Weight:         100 (100%)
Endpoints:      10.130.28.93:8080
```

It is important to check what labels have been used with your application if you have created it using a template, as templates may not follow the convention of using the ``app`` label.

Note that there is no easy way from the web console to delete all resource objects related to an application. It is recommended you use the command line, and a label selector to identity the resource objects, when deleting an application.
