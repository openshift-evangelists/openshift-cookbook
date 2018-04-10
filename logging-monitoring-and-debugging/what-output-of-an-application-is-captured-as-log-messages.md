---
related:
    - logging-monitoring-and-debugging/how-do-i-view-the-log-messages-output-by-an-application.md
---

When running applications in containers, any messages logged by the application to the standard output and error streams will be captured. If your application logs to separate files in the filesystem, these will not be captured and will not be displayed when running the ``oc logs`` command. Further, if those separate log files keep growing, and are not truncated or rolled over, with older entries deleted, they can eventually fill up the available space of the container filesystem.

If your application doesn't provide a way of logging directly to the standard output and error streams of a process, and requires that a filesystem path be provided, you should use the path:

```
/proc/1/fd/1
```

Using this special file under the ``/proc`` filesystem path will result in log messages being merged with the standard output of the application process with process ID 1, allowing it to be captured.

Note that if the logging framework used by the application, or any supervisor system, uses log file rotation, it should be disabled. The logging framework should not attempt to rename this special file, or replace it.

It is also recommended that you avoid using the special files ``/proc/self/fd/1`` or ``/dev/stdout``. These equate to standard output of the process it is used with, which if it were a sub process, may not result in it being captured if output from the sub process was being redirected.
