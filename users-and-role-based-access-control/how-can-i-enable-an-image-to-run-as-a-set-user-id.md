---
related:
    users-and-role-based-access-control/why-do-my-applications-run-as-a-random-user-id.md
---

When an application is deployed it will run as a user ID unique to the project it is running in. This overrides the user ID which the application image defines it wants to be run as. That the application is run as a different user ID can result in it failing to start.

The best solution is to build the application image so it can be run as an arbitrary user ID. This avoids the risks associated with having to run an application as the ``root`` user ID, or other fixed user ID which may be shared with applications in other projects.

If an image can't be modified, you can elect to override the default security configuration of OpenShift and have it run as the user the image specifies, but this can only be done by an administrator of the OpenShift cluster. This cannot be done by normal developers, nor a project administrator. This cannot be done on hosting services such as OpenShift Online.

The change required to override the default security configuration, is to grant rights to the service account the application is run under, to run images as a set user ID.

The service account within a project which applications would usually be run as is the ``default`` service account. Because you may run other applications in the same project, and don't necessarily want to override the user ID for all applications, create a new service account in the project which can be granted the special rights by running the ``oc create serviceaccount`` command, passing the name to be given to the service account.

```
$ oc create serviceaccount supremo
serviceaccount "supremo" created
```

The next step is that which must be run as a cluster administrator. It is the granting of the appropriate rights to the service account. This is done by specifying that the service account should run with a specific security context constraint (SCC).

As an administrator, you can see the list of SCCs that are defined in the cluster by running the ``oc get scc`` command.

```
$ oc get scc --as system:admin
NAME               PRIV      CAPS      SELINUX     RUNASUSER          FSGROUP     SUPGROUP    PRIORITY   READONLYROOTFS   VOLUMES
anyuid             false     []        MustRunAs   RunAsAny           RunAsAny    RunAsAny    10         false            [configMap downwardAPI emptyDir persistentVolumeClaim projected secret]
hostaccess         false     []        MustRunAs   MustRunAsRange     MustRunAs   RunAsAny    <none>     false            [configMap downwardAPI emptyDir hostPath persistentVolumeClaim projected secret]
hostmount-anyuid   false     []        MustRunAs   RunAsAny           RunAsAny    RunAsAny    <none>     false            [configMap downwardAPI emptyDir hostPath nfs persistentVolumeClaim projected secret]
hostnetwork        false     []        MustRunAs   MustRunAsRange     MustRunAs   MustRunAs   <none>     false            [configMap downwardAPI emptyDir persistentVolumeClaim projected secret]
nonroot            false     []        MustRunAs   MustRunAsNonRoot   RunAsAny    RunAsAny    <none>     false            [configMap downwardAPI emptyDir persistentVolumeClaim projected secret]
privileged         true      [*]       RunAsAny    RunAsAny           RunAsAny    RunAsAny    <none>     false            [*]
restricted         false     []        MustRunAs   MustRunAsRange     MustRunAs   RunAsAny    <none>     false            [configMap downwardAPI emptyDir persistentVolumeClaim projected secret]
```

By default applications would run under the ``restricted`` SCC. The ``MustRunAsRange`` value for ``RUNASUSER`` is what indicates that the application needs to run within the user ID range associated with the project.

There are two options for which SCC to use when wanting to run the image as the user ID it defines. These are:

* ``anyuid`` - This will enable the image to be run as any user ID, including the ``root`` user ID.
* ``nonroot`` - This will enable the image to be run as any user ID, except the ``root`` user ID.

If the image doesn't need to run as the ``root`` user ID, try and use ``nonroot`` instead of ``anyuid``. Do note though that for ``nonroot`` to work, the ``USER`` specified for the image, must be defined with a numeric user ID rather than a user name. If a user name is used, it is not possible to verify that it does in fact map to a user ID other than ``0``.

To associate the new service account with the SCC, run the ``oc adm policy add-scc-to-user`` command. The ``-z`` option indicates to apply the command to the service account in the current project, so ensure you run this within the correct project.

```
$ oc adm policy add-scc-to-user anyuid -z supremo --as system:admin
```

With applications run under this service account now being able to run as any user ID, you need to update the deployment configuration for the application to use the service account. This can be done by patching the deployment configuration in place using the ``oc patch`` command.

```
$ oc patch dc/minimal-notebook --patch '{"spec":{"template":{"spec":{"serviceAccountName": "supremo"}}}}'
deploymentconfig "minimal-notebook" patched
```

Once the deployment configuration has been updated, trigger a new deployment if necessary.

```
$ oc rollout latest minimal-notebook
deploymentconfig "minimal-notebook" rolled out
```
