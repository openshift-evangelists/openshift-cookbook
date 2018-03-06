---
related:
    - logging-monitoring-and-debugging/how-do-i-access-an-interactive-shell-inside-of-a-container.md
---

If your application is failing on startup, with the container also being terminated, OpenShift will keep attempting to restart it. If this keeps happening, the deployment will fail. OpenShift will indicate this by setting the status of the pod to ``CrashLoopBackOff``.

To debug a container that will not start you can use the ``oc debug`` command, running it against the deployment configuration for your application:

```
$ oc debug dc/nbviewer
Debugging with pod/nbviewer-debug, original command: container-entrypoint /tmp/scripts/run
Waiting for pod to start ...

Pod IP: 10.129.3.188
If you don't see a command prompt, try pressing enter.

(app-root)sh-4.2$
```

Rather than starting your application, an interactive shell session will be started. The startup messages when ``oc debug`` is run will show what the original command was that would have been run for the container.

From the shell, you can verify any environment variables or configuration files, change them if necessary, and then run the original command to start your application. Any output from your command will be displayed in the terminal so you can see what error may be occurring on startup.

When your application is started with ``oc debug``, it will not be possible to connect to it from any other pods using its service name, nor will it be exposed by a route for your application if one was created. If you need to send it requests, use ``oc get pods`` to get the name of the pod created for the debug session, then use ``oc rsh`` from a separate terminal to get a second interactive shell in the container. You can then run a command, such as ``curl``, against the application from inside the container:

```
$ oc get pods
NAME               READY     STATUS      RESTARTS   AGE
nbviewer-1-wgzdb   1/1       Running     0          3h
nbviewer-debug     1/1       Running     0          1m

$ oc rsh nbviewer-debug
(app-root)sh-4.2$ curl $HOSTNAME:8080
...
```
