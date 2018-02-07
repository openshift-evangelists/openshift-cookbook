---
related:
    - users-and-role-based-access-control/why-do-my-applications-run-as-a-random-user-id.md
---

When an application is deployed it will run as a user ID unique to the project it is running in. This overrides the user ID which the application image defines it wants to be run as. That the application is run as a different user ID can result in it failing to start.

The best solution is to build the application image so it can be run as an arbitrary user ID. This avoids the risks associated with having to run an application as the ``root`` user ID, or other fixed user ID which may be shared with applications in other projects.

If an image can't be modified, you can elect to override the default security configuration of OpenShift and have it run as the user the image specifies, but this can only be done by an administrator of the OpenShift cluster. This cannot be done by normal developers, nor a project administrator. This cannot be done on hosting services such as OpenShift Online.

The change required to override the default security configuration, is to grant rights to the service account the application is run under, to run images as a set user ID.

The service account within a project which applications would usually be run as is the ``default`` service account. Because you may run other applications in the same project, and don't necessarily want to override the user ID used for all applications, create a new service account which can be granted the special rights. In the project where the application is to run, run the ``oc create serviceaccount`` command, passing the name to be given to the service account.

```
$ oc create serviceaccount runasanyuid
serviceaccount "runasanyuid" created
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

To allow an application to be run as any user ID, including the ``root`` user ID, you want to use the ``anyuid`` SCC.

To associate the new service account with the SCC, run the ``oc adm policy add-scc-to-user`` command. The ``-z`` option indicates to apply the command to the service account in the current project, so ensure you run this within the correct project.

```
$ oc adm policy add-scc-to-user anyuid -z runasanyuid --as system:admin
```

With applications run under this service account now being able to run as any user ID, you need to update the deployment configuration for the application to use the service account. This can be done by patching the deployment configuration in place using the ``oc patch`` command.

```
$ oc patch dc/minimal-notebook --patch '{"spec":{"template":{"spec":{"serviceAccountName": "runasanyuid"}}}}'
deploymentconfig "minimal-notebook" patched
```

Once the deployment configuration has been updated, trigger a new deployment if necessary.

```
$ oc rollout latest minimal-notebook
deploymentconfig "minimal-notebook" rolled out
```

Allowing a user to run applications as any user ID will allow them to also run application images as ``root`` inside of the container. Because of the risks associated with allowing applications to run as ``root``, if ``root`` isn't required, use the ``nonroot`` SCC instead of ``anyuid``. This will allow an application to be run as any user ID except ``root``.

Do note though that for ``nonroot`` to work, the ``USER`` specified for the image, must be defined with an integer user ID rather than a user name. If a user name is used, it is not possible to verify that it does in fact map to a user ID other than ``0``.

If an image doesn't use an integer user ID for ``USER``, the alternative is to create a new SCC which enforces running as a single specific user ID.

To do this for the user ID ``1000``, create a file ``uid1000.json`` containing:

```
{
    "apiVersion": "v1",
    "kind": "SecurityContextConstraints",
    "metadata": {
        "name": "uid1000"
    },
    "requiredDropCapabilities": [
        "KILL",
        "MKNOD",
        "SYS_CHROOT",
        "SETUID",
        "SETGID"
    ],
    "runAsUser": {
        "type": "MustRunAs",
        "uid": "1000"
    },
    "seLinuxContext": {
        "type": "MustRunAs"
    },
    "supplementalGroups": {
        "type": "RunAsAny"
    },
    "fsGroup": {
        "type": "MustRunAs"
    },
    "volumes": [
        "configMap",
        "downwardAPI",
        "emptyDir",
        "persistentVolumeClaim",
        "projected",
        "secret"
    ]
}
```

To create the new SCC, you need to be an administrator.

```
$ oc create -f uid1000.json --as system:admin
securitycontextconstraints "uid1000" created
```

Create the service account in the project:

```
$ oc create serviceaccount runasuid1000
serviceaccount "runasuid1000" created
```

Set the SCC to be used by the service account to that created above.

```
$ oc adm policy add-scc-to-user uid1000 -z runasuid1000 --as system:admin
```

Finally patch the deployment configuration.

```
$ oc patch dc/minimal-notebook --patch '{"spec":{"template":{"spec":{"serviceAccountName": "runasuid1000"}}}}'
deploymentconfig "minimal-notebook" patched
```

For a SCC which sets ``runAsUser`` to be ``MustRunAs`` and provides a set user ID, when the application is run, it will be forced to run as that user ID, overriding whatever user ID the image may have specified. Whether the image sets ``USER`` to a user name rather than an integer user ID therefore doesn't matter. The SCC just needs to use the same integer user ID that the user name maps to for the image.
