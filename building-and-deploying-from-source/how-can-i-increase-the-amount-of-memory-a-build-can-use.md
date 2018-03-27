When quotas have been configured, an OpenShift cluster will enforce a limit on how much memory a build of an application can use. For hosted environments such as OpenShift Online, the default limit on the amount of memory that can be used for a build is 512Mi.

In some cases a build may require more memory than the limit. This can occur when developing using Node.JS and using ``npm`` to install packages, or when using ``webpack``. It can also occur when using Python, and using ``pip`` to install packages with C extensions from source code rather than binary wheels, such as the ``numpy`` package.

To override the amount of memory that can be used, you can use the ``oc patch`` command to update the build configuration.

```
$ oc patch bc/cookbook --patch '{"spec":{"resources":{"limits":{"memory":"1Gi"}}}}'
buildconfig "cookbook" patched
```

When setting the memory limit for a build, the value needs to be less than the quota for terminating resources.
