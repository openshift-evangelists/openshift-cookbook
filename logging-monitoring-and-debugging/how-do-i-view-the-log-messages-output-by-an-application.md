---
related:
    - logging-monitoring-and-debugging/what-output-of-an-application-is-captured-as-log-messages.md
---

When running applications in containers, any messages logged by the application to the standard output and error streams will be captured. To view the messages produced by a specific instance of the application, you need to run ``oc logs`` on the pod in which it is running.

To get a list of the pods for an application, you can use ``oc get pods``.

```
$ oc get pods
NAME               READY     STATUS      RESTARTS   AGE
nbviewer-1-wgzdb   1/1       Running     0          3h
```

Then run the ``oc logs`` command.

```
$ oc logs pod/nbviewer-1-wgzdb
```

You can use the ``--follow`` option to ``oc logs`` to continually monitor the logs for that pod.

By default no timestamps are shown for individual log messages unless the application itself adds them. To display alongside logged messages the time of the message as captured by OpenShift, use the ``--timestamps`` option.
