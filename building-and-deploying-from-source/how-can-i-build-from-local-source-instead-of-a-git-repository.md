When building from source code using the docker or source build strategies, the typical scenario would be that the source code would be contained in a hosted Git repository. If you use a source code version control system other than Git, or want to use source code in a directory on your local system, you can use what is called a binary input build.

A binary input build is also useful when you are developing an application and iterating on changes but don't want to push the changes to your hosted Git repository without first testing them.

In this later case, where your build configuration already exists and is linked to your Git repository, you can trigger a single build using input from your local system by running ``oc start-build``.

```
$ oc start-build cookbook --from-dir=.
Uploading directory "." as binary input for the build ...
build "cookbook-2" started
```

For this single build, everything in the directory passed to the ``--from-dir`` option will be uploaded to OpenShift and used in place of what is held by the hosted Git repository already linked to the build.

This will work for a standalone build configuration created using ``oc new-build``, or where the build configuration was created as part of deploying an application from source code using ``oc new-app``.

Once you are happy that your code changes work, you can commit and push the changes to your hosted Git repository, and trigger a new build, using what is in the hosted Git repository, by running ``oc start-build`` again, but without the ``--from-dir`` option.

```
$ oc start-build cookbook
build "cookbook-3" started
```

If you don't have a hosted Git repository, or always want to use source files from your local system, you can create a build configuration where it is required that any input must be provided with each build. This is done using the ``oc new-build --binary`` command.

If your source code is for building an image from a ``Dockerfile``, the build is created by running:

```
$ oc new-build --name cookbook --binary --strategy docker
```

If you want to setup an S2I build, you also need to supply the details of the S2I builder image using the ``--image-stream`` option.

```
$ oc new-build --name cookbook --binary --strategy source --image-stream python:3.5
```

Because the ``--binary`` option is provided in each of these cases, this will only create the build configuration. To run an actual build, you still need to use ``oc start-build``.

```
$ oc start-build cookbook --from-dir=.
```

If you trigger a new build without supplying source code using ``--from-dir``, the build will fail, as the build configuration doesn't know how to obtain the source code and is relying on you providing it each time ``oc start-build`` is run.
