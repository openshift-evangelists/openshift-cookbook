To create a service account, with a session token which does not expire, for use with scripted access, use the ``oc create sa`` command, and pass the name to give the service account.

```
$ oc create sa robot
serviceaccount "robot" created
```

To view details of the service account created, run ``oc describe`` on the service account resource.

```
$ oc describe sa robot
Name:		robot
Namespace:	cookbook
Labels:		<none>
Annotations:	<none>

Image pull secrets:	robot-dockercfg-vl9qn

Mountable secrets: 	robot-token-mhf9x
                   	robot-dockercfg-vl9qn

Tokens:            	robot-token-4nkdw
                   	robot-token-mhf9x
```

Secrets for two access tokens will be created.

One will be mounted into any containers which are run as this service account to allow an application running in the container to access the REST API if required.

The second can be used from outside of the OpenShift cluster as an access token to login from the command line, or when using the OpenShift REST API.

To view the access token, run ``oc describe`` on the secret.

```
$ oc describe secret robot-token-4nkdw
Name:		robot-token-4nkdw
Namespace:	cookbook
Labels:		<none>
Annotations:	kubernetes.io/created-by=openshift.io/create-dockercfg-secrets
		kubernetes.io/service-account.name=robot

Type:	kubernetes.io/service-account-token

Data
====
ca.crt:		1070 bytes
namespace:	8 bytes
service-ca.crt:	2186 bytes
token:		eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...
```

The token will not expire. If you need to revoke the access token you can delete the secret for the access token using ``oc delete`` and a new secret will be created.

```
$ oc delete secret robot-token-4nkdw
secret "robot-token-4nkdw" deleted
```

The service account, along with any secrets associated with it, can be deleted by running ``oc delete`` against the service account.

```
$ oc delete sa robot
serviceaccount "robot" deleted
```

Note that the service account will by default have no access to do anything within the project via the REST API. You will need to grant appropriate roles to the service account to enable it to view or make changes to any resource objects.

For further information on creating and using service accounts see:

* https://docs.openshift.org/latest/dev_guide/service_accounts.html
