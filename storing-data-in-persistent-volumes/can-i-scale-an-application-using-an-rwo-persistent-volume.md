A persistent volume of type _ReadWriteOnce_ (RWO) can be mounted on only one node in an OpenShift cluster at a time. An application using an RWO persistent volume can not use rolling deployments or be scaled up to multiple replicas. This is because there is no guarantee, by default, that all pods will be deployed to the same node in the cluster.

If you need to be able to use rolling deployments or scale an application, and only have RWO persistent volumes available, you can set an affinity rule for pods so they will all be scheduled to the same node.

Do note that by using an affinity rule to have all pods scheduled to the same node, there must be sufficient memory and CPU resources available to host all pods on that node. If there is insufficient resources, the scheduler will not be able to start all pods and the deployment will fail.

In the case of a rolling deployment, there also needs to be enough resources to create additional pods on top of the number defined by the replica count.

If the node becomes unavailable, the application will become unavailable, until the pods can be restarted on a different node, as you do not benefit from the fact that the pods would be distributed across nodes in a cluster.

To add the affinity rule, you need to edit the deployment configuration and add a ``spec.template.spec.affinity`` entry.  The pod affinity type ``requiredDuringSchedulingIgnoredDuringExecution`` should be used.

```
"affinity": {
    "podAffinity": {
        "requiredDuringSchedulingIgnoredDuringExecution": [
            {
                "labelSelector": {
                    "matchExpressions": [
                        {
                            "key": "app",
                            "operator": "In",
                            "values": [
                                "your-app-name"
                            ]
                        }
                    ]
                },
                "topologyKey": "kubernetes.io/hostname"
            }
        ]
    }
}
```

Update ``key`` and ``values`` for the label selector to match the label used for your pods in the deployment configuration.

You can run ``oc edit`` on the deployment configuration to make the change, or use ``oc patch``.

```
oc patch dc/your-app-name --patch '{
    "spec": {
        "template": {
            "spec": {
                "affinity": {
                    "podAffinity": {
                        "requiredDuringSchedulingIgnoredDuringExecution": [
                            {
                                "labelSelector": {
                                    "matchExpressions": [
                                        {
                                            "key": "app",
                                            "operator": "In",
                                            "values": [
                                                "your-app-name"
                                            ]
                                        }
                                    ]
                                },
                                "topologyKey": "kubernetes.io/hostname"
                            }
                        ]
                    }
                }
            }
        }
    }
}'
```
