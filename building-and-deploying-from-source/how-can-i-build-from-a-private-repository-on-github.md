A private Git repository on GitHub can be accessed using either SSH or HTTPS. The preferred method is to always use SSH and a SSH key pair. Only use HTTPS if you have no choice.

The first step to using a private Git repository on GitHub using a repository SSH key is to generate the SSH key pair to be used with that repository. Remember that it is recommended to use a distinct SSH key pair. Do not use your primary identity SSH key as you will need to upload the private key file of the SSH key pair to OpenShift.

```
$ ssh-keygen -C "openshift-source-builder/repo@github" -f repo-at-github -N ''
```

To register the repository SSH key with your private repository on GitHub, go to the *Settings* for the repository.

On GitHub the repository SSH key is referred to by the term *Deploy key*. Search down the settings page and find the *Deploy keys* section and select it.

Click on the **Add deploy key** button. In this section, give the key a name and paste in the contents of the public key file from the SSH key pair. This is the file with the ``.pub`` extension, which in this example is called ``repo-at-github.pub``.

Leave the *Allow write access* option unchecked, as we only want to provide read-only access to the Git repository using this key. This ensures that even if someone has access to the private key, they will not be able to make modifications to any files hosted by the Git repository.

Press **Add key** and the public key file will be registered.

The next step is to create a secret in OpenShift to hold the private key of the SSH key pair. When using the command line, to create the secret run:

```
$ oc create secret generic repo-at-github \
     --from-file=ssh-privatekey=repo-at-github \
     --type=kubernetes.io/ssh-auth
```

Enable access to the secret from the builder service account:

```
$ oc secrets link builder repo-at-github
```

To create a new build and deployment using ``oc new-app``, which uses this source secret, supply the ``--source-secret`` option to ``oc new-app``.

```
$ oc new-app httpd~git@github.com:osevg/private-repo.git \
    --source-secret repo-at-github --name mysite
```

If only creating a new build, supply ``--source-secret`` to the ``oc new-build`` command instead.

```
$ oc new-build httpd~git@github.com:osevg/private-repo.git \
    --source-secret repo-at-github --name mysite
```

If you have an existing build configuration that you need to update to use the source secret, run ``oc set build-secret``.

```
$ oc set build-secret mysite repo-at-github --source
```

If the OpenShift cluster you are using is located behind a corporate firewall and SSH connections are blocked, you need to use a personal access token and HTTPS connection rather than SSH.

From the web interface of GitHub browse to your user *Settings* and under *Developer settings* you will find *Personal access tokens*.

Select **Generate new token**, enter in a name as the *Token description* and enable the *repo* checkbox.

GitHub doesn’t provide a way of setting the scope of a personal access token such that it has read-only access to repositories. Instead, one has to enable the repo scope which gives full control of private repositories.

Giving full control is not ideal as it means that anyone who gets control over the personal access token would also be able to write to any repositories the account has write access to. This is one of the reasons why read-only repository SSH keys bound to a specific repository are preferred.

When you are done with setting the permissions for the personal access token, click on **Generate token** and you will be shown the value of the token. Make sure you make a copy of this as you cannot view it later on in the GitHub settings.

Create the secret from the command line using the ``oc create secret`` command.

```
$ oc create secret generic user-at-github \
    --from-literal=username=machineuser \
    --from-literal=password=accesstoken \
    --type=kubernetes.io/basic-auth
```

You will need to supply the name of the user account which the personal access token was created under as the value to ``username``. It is better to create a machine user account for an organization, which has access to the repository, rather than use a personal user account. Also supply the access token as the value to ``password``.

Run ``oc secrets link`` to allow the builder service account to use it.

```
$ oc secrets link builder user-at-github
```

To create a new build and deployment using ``oc new-app``, which uses this source secret, supply the ``--source-secret`` option to ``oc new-app``, passing the name of the secret. Similarly, supply ``--source-secret`` to ``oc new-build`` if creating just a build.
