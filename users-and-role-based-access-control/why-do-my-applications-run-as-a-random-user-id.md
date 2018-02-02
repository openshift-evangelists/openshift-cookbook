---
related:
    - users-and-role-based-access-control/how-can-i-enable-an-image-to-run-as-a-set-user-id.md
---

When you deploy an application it will appear that it is running as a random user ID, overriding what user ID the image itself may specify that it should run as.

The user ID isn't actually entirely random, but is an assigned user ID which is unique to your project. In fact, your project is assigned a range of user IDs that applications can be run as. The set of user IDs will not overlap with other projects. You can see what range is assigned to a project by running ``oc describe`` on the project.

Within the output of ``oc describe`` for the project, the set of annotations recorded against the project show the assigned range of user IDs.

```
openshift.io/sa.scc.uid-range=1008050000/10000
```

By default, any image you deploy to a project will be run as the first user ID in the range assigned to the project. If necessary, you can define an alternate user ID within the range to be used in the deployment configuration for an application. Any attempt to specify that an application should run as a user ID outside of the range will fail.

The purpose of assigning each project a distinct range of user IDs is so that in a multitenant environment, applications from different projects never run as the same user ID. When using persistent storage, any files created by applications will also have different ownership in the file system.

Running processes for applications as different user IDs means that if a security vulnerability were ever discovered in the underlying container runtime, and an application were able to break out of the container to the host, they would not be able to interact with processes owned by other users, or from other applications, in other projects.

Use of assigned user IDs is a part of the multi layer security strategy employed by OpenShift to reduce risks were an application or the container runtime compromised.

Being forced to run as an arbitrary user ID does mean that some container images may not run out of the box in OpenShift. This will be the case where images do not adopt security best practices and need to be run as the ``root`` user ID even though they have no actual requirement to run as ``root``. Even an image which has been setup to run as a fixed user ID which isn't ``root`` may not work.

To ensure container images are portable and will work with container deployment platforms which enforce additional separation between applications by overriding the user ID, images and the applications they contain, should be designed to be able to be run as an arbitrary user ID.
