Each instance of your application runs in a container, sandboxed off from everything else. The only thing running in the container will be the application processes.

In order to debug what is happening inside the container with your application, you can gain access to the container and run an interactive shell. To do this, run ``oc rsh`` against the name of the pod:

```
$ oc rsh nbviewer-1-wgzdb
(app-root)sh-4.2$
```

This will work for any container image that includes ``/bin/sh`` within the image. What Unix tools you have access to when debugging your application will depend on what was included in the image.

If, rather than an interactive shell, you want to run just a single command that requires no input, use ``oc exec``:

```
$ oc exec nbviewer-1-wgzdb env | grep HOSTNAME
HOSTNAME=nbviewer-1-wgzdb
```
