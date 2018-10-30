---
related:
    - application-lifecycle-management/how-can-i-change-the-deployment-strategy-being-used.md
---

When your application is deployed using a ``DeploymentConfig``, ``Deployment``, ``StatefulSet``, ``DaemonSet``, or with lower level ``ReplicaSet`` and ``ReplicationContoller`` resource types, if an instance of the application stops and the container exits, OpenShift will automatically replace the pod for that instance. OpenShift therefore ensures your application stays running.

If you have a need to manually restart an instance of your application using the latest configuration and image it specifies, and you are using a ``DeploymentConfig``, you can use the command:

```
$ oc rollout latest dc/cookbook
```

For a ``DeploymentConfig``, the steps taken to shutdown any existing instances of your application and replace them with new instances, will be carried out per the deployment strategy defined by the configuration.

If you are using ``Deployment``, ``StatefulSet``, ``DaemonSet``, ``ReplicaSet`` or ``ReplicationController``, to trigger a new deployment you will need to make a manual change to the pod template component of the configuration defined by the resource.

This can be done by updating the value of an annotation within the pod template. The annotation does not need to exist the first time and you can use whatever name for the annotation you wish as long as it doesn't conflict with an existing annotation that has some other purpose. Use a date/time value or other value which can be changed with each deployment.

```
$ oc patch deployment/cookbook --patch \
   "{\"spec\":{\"template\":{\"metadata\":{\"annotations\":{\"last-restart\":\"`date +'%s'`\"}}}}}"
```

This method of updating an annotation on the pod template can also be used with a ``DeploymentConfig``. You can use ``kubectl`` instead of ``oc`` to apply the patch if necessary.

As with ``DeploymentConfig``, the application instances will be restarted according to any deployment strategy or other policy enforced by the resource type used to manage the deployment of the application.
