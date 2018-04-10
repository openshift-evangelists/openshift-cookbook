If a user manages an OpenShift cluster and is able to login to the nodes of the cluster, they can run ``oc`` commands as a cluster admin. When they login remotely to the OpenShift cluster using the ``oc`` command, they would only be able to run commands with the access rights granted to their user.

It is possible to add roles to a normal user account which gives them the rights to do anything a cluster admin can do when logged in remotely, but this should be avoided if possible.

If a user needs to be able to run ``oc`` commands as a cluster admin when logged in remotely, it is better to add that user to the ``sudoer`` role. When in this role, and they run an ``oc`` command, it would still run with the access rights of their user. To run an ``oc`` command as a cluster admin, they use the ability granted by the ``sudoer`` role, to impersonate the special user called ``system:admin``. This is done by supplying the ``--as system:admin`` option to the ``oc`` command.

```
$ oc get nodes --as system:admin
```

By requiring the extra step of needing to explicitly say you want to impersonate the ``system:admin``, it makes it harder to make mistakes where you accidentally modify or delete resource objects you wouldn't normally have the access rights to modify.

Adding the ``sudoer`` role to a user can only be done by a user with existing cluster admin access. This is done using the ``oc adm policy add-cluster-role-to-user`` command.

```
$ oc adm policy add-cluster-role-to-user sudoer <username> --as system:admin
```

If you are using the REST API directly and need to perform an action as ``system:admin``, you need to supply an additional header with the HTTP request against the REST API endpoint. The name of the header is ``Impersonate-User``.

```
#!/bin/sh

SERVER=`oc whoami --show-server`
TOKEN=`oc whoami --show-token`
USER="system:admin"

URL="$SERVER/api/v1/nodes"

curl -k -H "Authorization: Bearer $TOKEN" -H "Impersonate-User: $USER" $URL
```
