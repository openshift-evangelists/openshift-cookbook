---
related:
    - image-registry-and-image-streams/how-do-i-push-an-image-to-the-internal-image-registry.md
---

When deploying a pre-existing application image it must be hosted on an external image registry, or exist in the internal image registry of OpenShift. One reason that an image may exist in the internal image registry is if it was built within OpenShift from either a ``Dockerfile``, or from application source code using a Source-to-Image (S2I) builder.

The internal image registry of OpenShift can also be loaded with a pre-existing application image by importing it from an external image registry. To do this, run ``oc import-image`` passing the full name of the image.

```
oc import-image kubernetes/guestbook --confirm
```

Include the image registry details if necessary.

```
oc import-image docker.io/kubernetes/guestbook --confirm
```

This will create an image stream definition with details of the imported image. By default the image stream will be given a name corresponding to the last portion of the image name. In this case ``guestbook`` would be used as the name. If you need to override the name of the deployment, use the ``--name`` argument to ``oc new-app``.

The ``--confirm`` option is required to confirm the image stream should be created when no prior image stream existed.

Note that where the image on the external image registry has multiple tags, an entry for each tag will be created in the image stream definition. If you are only interested in one specific tag, you can append the tag to the image name when running ``oc import-image``.

```
oc import-image docker.io/kubernetes/guestbook:latest --confirm
```

If ``oc import-image`` is run where an image stream already existed, the image stream definition will be updated to use the latest versions of any tagged images in the image stream definition.
