To create a service account, with a session token which does not expire, for use with scripted access, use the ``oc create sa`` command, and pass the name to give the service account.

```
$ oc create sa robot
serviceaccount "robot" created
```

To view details of the service account created, run ``oc describe`` on the service account resource.

```
$ oc describe sa robot
Name:        robot
Namespace:   cookbook
Labels:      <none>
Annotations: <none>

Image pull secrets: robot-dockercfg-vl9qn

Mountable secrets:  robot-token-mhf9x
                    robot-dockercfg-vl9qn

Tokens:             robot-token-4nkdw
                    robot-token-mhf9x
```

Secrets for two access tokens will be created.

One is mounted into any containers which are run as this service account to allow an application running in the container to access the REST API if required.

The second is referenced in the separate secret for the docker configuration used when pulling images from the internal docker registry.

Of the two tokens, the first token, which would normally be used from within containers running with this service account to access the REST API, also be used when accessing the REST API from outside of the cluster.

To view the access token, run ``oc describe`` on the secret.

```
$ oc describe secret robot-token-mhf9x
Name:        robot-token-mhf9x
Namespace:   cookbook
Labels:      <none>
Annotations: kubernetes.io/service-account.name=robot

Type:        kubernetes.io/service-account-token

Data
====
ca.crt:         1070 bytes
namespace:      8 bytes
service-ca.crt: 2186 bytes
token:          eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...
```

The token will not expire. If you need to revoke the access token you can delete the secret for the access token using ``oc delete`` and a new secret will be created.

```
$ oc delete secret robot-token-mhf9x
secret "robot-token-mhf9x" deleted
```

The service account, along with any secrets associated with it, can be deleted by running ``oc delete`` against the service account.

```
$ oc delete sa robot
serviceaccount "robot" deleted
```

Note that the service account will by default have no access to do anything within the project via the REST API. You will need to grant appropriate roles to the service account to enable it to view or make changes to any resource objects.

For further information on creating and using service accounts see:

* https://docs.openshift.org/latest/dev_guide/service_accounts.html
