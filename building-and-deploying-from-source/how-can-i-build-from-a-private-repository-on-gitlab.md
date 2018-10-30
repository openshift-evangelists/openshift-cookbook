A private Git repository on GitLab can be accessed using either SSH or HTTPS. The preferred method is to always use SSH and a SSH key pair. Only use HTTPS if you have no choice.

The first step to using a private Git repository on GitLab using a repository SSH key is to generate the SSH key pair to be used with that repository. It is recommended to use a distinct SSH key pair for this purpose. Do not use your primary identity SSH key as you will need to upload the private key file of the SSH key pair to OpenShift.

```
$ ssh-keygen -C "openshift-source-builder/repo@gitlab" -f repo-at-gitlab -N ''
```

To register the repository SSH key with your private repository on GitLab, go to the *Settings* for the repository and find the *Repository* settings page.

On GitLab the repository SSH key is referred to by the term *Deploy Key*. Search down the *Repository* settings page and find the *Deploy Keys* section and expand it. In this section, give the key a name and paste in the contents of the public key file from the SSH key pair. This is the file with the ``.pub`` extension, which in this example is called ``repo-at-gitlab.pub``.

Ensure that the *Write access allowed* checkbox is not enabled. That is, you do not want to grant any user of this repository SSH key write access to the repository, read-only access is sufficient.

Upon clicking on **Add key**, the key will be registered for the repository.

The next step is to create a secret in OpenShift to hold the private key of the SSH key pair. When using the command line, to create the secret run:

```
$ oc create secret generic repo-at-gitlab \
     --from-file=ssh-privatekey=repo-at-gitlab \
     --type=kubernetes.io/ssh-auth
```

Enable access to the secret from the builder service account:

```
$ oc secrets link builder repo-at-gitlab
```

To create a new build and deployment using ``oc new-app``, which uses this source secret, supply the ``--source-secret`` option to ``oc new-app``.

```
$ oc new-app httpd~git@gitlab.com:osevg/private-repo.git \
    --source-secret repo-at-gitlab --name mysite
```

If only creating a new build, supply ``--source-secret`` to the ``oc new-build`` command instead.

```
$ oc new-build httpd~git@gitlab.com:osevg/private-repo.git \
    --source-secret repo-at-gitlab --name mysite
```

If you have an existing build configuration that you need to update to use the source secret, run ``oc set build-secret``.

```
$ oc set build-secret mysite repo-at-gitlab --source
```
