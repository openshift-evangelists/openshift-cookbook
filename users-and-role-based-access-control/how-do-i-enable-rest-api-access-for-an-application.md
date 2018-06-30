When you deploy an application in OpenShift, an access token for accessing the REST API for the OpenShift cluster is mounted into the container at the path:

```
/var/run/secrets/kubernetes.io/serviceaccount/token
```

The hostname and port for the REST API endpoint, is passed through the environment variables:

```
KUBERNETES_SERVICE_PORT
KUBERNETES_SERVICE_HOST
```

or you can use:

```
https://openshift.default.svc.cluster.local
```

These can be used by the application, or in a script, running in the container to make requests against the REST API endpoint.

```
#!/bin/sh

SERVER="https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT"
TOKEN=`cat /var/run/secrets/kubernetes.io/serviceaccount/token`

URL="$SERVER/oapi/v1/users/~"

curl -k -H "Authorization: Bearer $TOKEN" $URL
```

If the ``oc`` binary is available in the image, you can also use it, and it will automatically pick up the token and use it without you needing to login, or pass it explicitly to each ``oc`` command.

Although the token and endpoint details are provided in the container, any access will by default fail. In order to be able to make requests, you also need to enable access to the REST API by the service account that the container runs as.

Normally when you deploy an application, the container uses the service account with name ``default``. You could grant REST API access to this ``default`` service account, but because it is used as the default for all applications in a project, you are better off creating a new separate service account. Using a separate service account is better because you can ensure that only the applications which need REST API access will have it, and you can also control what level of access they have.

To create a new service account, use the command:

```
oc create sa rest-api-edit
```

In this example the new service account is called ``rest-api-edit``, but you can use whatever name you want to use.

Next you need to give the service account the appropriate level of access your application needs. This is done by adding an appropriate role to the service account.

The roles you can use here are the same roles you might add to users in a project which dictate what they can do.

* ``admin`` -  A project manager. The user will have rights to view any resource in the project and modify any resource in the project except for quotas. A user with this role for a project will be able to delete the project.
* ``edit`` - A user that can modify most objects in a project, but does not have the power to view or modify roles or bindings. A user with this role can create and delete applications in the project.
* ``view`` - A user who cannot make any modifications, but can see most objects in a project.

In this example we want the application to be able to create resource objects as well as view them. We don't want them to be able to delete the project or perform any other actions related to the administration of the project. We therefore use the ``edit`` role. This can be added using the command:

```
oc policy add-role-to-user edit -z rest-api-edit
```

The final step required is to change the deployment configuration for the application to use this service account. This is done by adding the setting ``dc.spec.template.spec.serviceAccountName``.

```
$ oc explain dc.spec.template.spec.serviceAccountName
FIELD: serviceAccountName <string>

DESCRIPTION:
     ServiceAccountName is the name of the ServiceAccount to use to run this
     pod. More info: https://kubernetes.io/docs/tasks/configure-pod-container/
     configure-service-account//
```

Where the deployment configuration already exists, you can add the setting using ``oc patch``.

```
oc patch dc my-app-name --patch \
  '{"spec":{"template":{"spec":{"serviceAccountName":"rest-api-edit"}}}}'
```

Once the deployment configuration has been updated, re-deploy the application if necessary, and you should now be able to access the REST API endpoint from within the container.
