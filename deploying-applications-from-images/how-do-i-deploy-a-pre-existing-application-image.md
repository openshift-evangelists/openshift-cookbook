---
related:
    - image-registry-and-image-streams/how-do-i-import-an-image-from-an-external-image.md
    - image-registry-and-image-streams/how-do-i-push-an-image-to-the-internal-image-registry.md
    -  users-and-role-based-access-control/why-do-my-applications-run-as-a-random-user-id.md
    - users-and-role-based-access-control/how-can-i-enable-an-image-to-run-as-a-set-user-id.md
---

To deploy a pre-existing application image, the image must be hosted on an external image registry, or exist in the internal image registry of OpenShift. From the command line you can use the ``oc new-app`` command to deploy the image.

Where the application image is being hosted on DockerHub, the argument passed to ``oc new-app`` should be the full name of the image as listed on DockerHub. For example, to deploy the ``guestbook`` image owned by the ``kubernetes`` organization on DockerHub, use:

```
oc new-app kubernetes/guestbook
```

By default the deployment will be given a name corresponding to the last portion of the image name. In this case ``guestbook`` would be used as the name. If you need to override the name of the deployment, use the ``--name`` argument to ``oc new-app``.

If the application image is hosted on an image registry other than DockerHub, include the details of the image registry in the full name of the image. Using the details of the image registry for DockerHub, the above command could also be run as:

```
oc new-app docker.io/kubernetes/guestbook
```

Where the application already exists in the internal image registry of OpenShift and an image stream definition exists for the image in the current project, or the system ``openshift`` project, you can pass just the name of the image stream to ``oc new-app``. For example, if the name of the image stream was ``guestbook``, use:

```
oc new-app guestbook
```

Note that not all images on DockerHub may run out of the box on OpenShift. You can encounter issues where images require they be run as the UNIX ``root`` user, or other set UNIX user ID. Running such images requires the OpenShift cluster administrator to bless the service account an image is being run from, to run the image as a set UNIX user ID.
