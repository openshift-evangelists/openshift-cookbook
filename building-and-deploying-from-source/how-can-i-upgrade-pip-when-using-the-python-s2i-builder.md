Source-to-Image (S2I) builders are available in OpenShift for a number of different Python versions. These all use Python from the [Software Collections Library](https://www.softwarecollections.org/) (SCL) and will pre-create a Python virtual environment for you, into which Python packages will be installed. Packaging tools such as ``pip`` will also be supplied.

As the version of ``virtualenv`` used to create the Python virtual environment also comes from SCL, it may not be the latest version. This means that packaging tools provided by the ``pip``, ``wheel`` and ``setuptools`` packages may also not be the latest versions. This can be an issue when wanting to install Python packages from a Python wheel, as some Python wheels can require the latest version of ``pip`` and ``wheel`` tools to be available.

If a Python wheel file is not being used, or fails to install, you will need to upgrade the versions of ``pip``, ``wheel`` and ``setuptools`` to the latest versions. If you are using the Python S2I builder, this can be triggered by setting the build time environment variable ``UPGRADE_PIP_TO_LATEST``.

To set this environment variable on an existing build configuration, you can use the ``oc set env`` command on the build configuration.

```
$ oc set env bc/cookbook UPGRADE_PIP_TO_LATEST=true
```

and then trigger a new build.

```
$ oc start-build cookbook
```

If creating a new build or deployment, you can use the ``--build-env`` option to ``oc new-app`` or ``oc new-build``.

```
$ oc new-app \
    python:3.5~https://github.com/openshift-katacoda/blog-django-py \
    --build-env UPGRADE_PIP_TO_LATEST=true
```

The environment variable can be set to any non empty value to trigger the upgrade. It is suggested to use the value ``true``. If needing to later prevent the upgrade from occuring, you will need to delete the environment variable setting, or set it to an empty value.
