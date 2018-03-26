---
related:
    - storing-data-in-persistent-volumes/how-can-i-scale-applications-using-rwo-persistent-volumes.md
---

A deployment strategy defines the process by which a new version of your application is started and the existing instances shut down.

A rolling deployment slowly replaces instances of the previous version of an application with instances of the new version of the application. In a rolling deployment a check will be run against a new instance (using the readiness probe) to determine whether it is ready to accept requests, before adding its IP address to the list of service endpoints and scaling down the old instance. If a significant issue occurs, the rolling deployment can be aborted.

This is the default deployment strategy in OpenShift. Because it runs old and new instances of your application in parallel and balances traffic across them as new instances are deployed and old instances shut down, it enables an update with no downtime.

A rolling deployment should not be used if your application code is affected by the following scenarios:

* When the new version of the application code will not work with the existing schema of a separate database, and a database migration is required first.
* When the new and old application code cannot be run at the same time due to reasons other than dependence on a separate database schema version.
* When it wouldn’t be safe to run more than one instance of the application at the same time, even of the same version.

The use of persistent storage can also limit the ability to use a rolling deployment.

* When you are using persistent storage of type ``ReadWriteOnce`` and you are not able to define an affinity rule for pods so they will all be scheduled to the same node.

In these cases the recreate deployment strategy should be used.

Instead of starting up new instances of your application while still running old instances, when the recreate strategy is used, the existing running instances of the application will be shut down first. Only after all the old instances of the application have been shut down will the new instances be started.

With this strategy, as all old instances of the application will be stopped before deploying instances with the new code, there may be a period of time when your application is not available. Unless you have taken steps to first route traffic to a temporary application that shows a maintenance page, users will see an ``HTTP 503 Service Unavailable`` error response.

To view what strategy a deployment is using, run the ``oc describe`` command on the deployment configuration.

```
oc describe dc/cookbook
```

The current deployment strategy is listed against the _Strategy_ field.

```
Strategy: Rolling
```

To change the deployment strategy you need to edit the deployment configuration using ``oc edit``, or run ``oc patch`` to patch the deployment configuration in place:

```
$ oc patch dc/blog --patch '{"spec":{"strategy":{"type":"Recreate"}}}'
```
