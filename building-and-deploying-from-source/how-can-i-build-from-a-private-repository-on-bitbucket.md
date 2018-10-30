A private Git repository on Bitbucket can be accessed using either SSH or HTTPS. The preferred method is to always use SSH and a SSH key pair. Only use HTTPS if you have no choice.

The first step to using a private Git repository on Bitbucket using a repository SSH key is to generate the SSH key pair to be used with that repository. Remember that it is recommended to use a distinct SSH key pair. Do not use your primary identity SSH key as you will need to upload the private key file of the SSH key pair to OpenShift.

```
$ ssh-keygen -C "openshift-source-builder/repo@bitbucket" -f repo-at-bitbucket -N ''
```

To register the repository SSH key with your private repository on Bitbucket, go to the *Settings* for the repository.

On Bitbucket the repository SSH key is referred to by the term *Access key*. Search down the settings page and find the *Access keys* section and select it.

Click on the **Add key** button. In the popup window, give the key a name and paste in the contents of the public key file from the SSH key pair. This is the file with the ``.pub`` extension, which in this example is called ``repo-at-bitbucket.pub``.

Bitbucket repository SSH keys provide read-only access and it is not possible to enable them as having write access.

Upon clicking on **Add key** the key will be registered for the repository.

The next step is to create a secret in OpenShift to hold the private key of the SSH key pair. When using the command line, to create the secret run:

```
$ oc create secret generic repo-at-bitbucket \
     --from-file=ssh-privatekey=repo-at-bitbucket \
     --type=kubernetes.io/ssh-auth
```

Enable access to the secret from the builder service account:

```
$ oc secrets link builder repo-at-bitbucket
```

To create a new build and deployment using ``oc new-app``, which uses this source secret, supply the ``--source-secret`` option to ``oc new-app``.

```
$ oc new-app httpd~git@bitbucket.org:osevg/private-repo.git \
    --source-secret repo-at-bitbucket --name mysite
```

If only creating a new build, supply ``--source-secret`` to the ``oc new-build`` command instead.

```
$ oc new-build httpd~git@bitbucket.org:osevg/private-repo.git \
    --source-secret repo-at-bitbucket --name mysite
```

If you have an existing build configuration that you need to update to use the source secret, run ``oc set build-secret``.

```
$ oc set build-secret mysite repo-at-bitbucket --source
```

If the OpenShift cluster you are using is located behind a corporate firewall and SSH connections are blocked, you need to use a personal access token and HTTPS connection rather than SSH.

From the web interface of Bitbucket browse to your Bitbucket settings.

On Bitbucket a personal access token is referred to by the term *App password*. Search down the settings page and find the *App passwords* section, click on it and then *Create app password*.

Enter in a name for the token and enable the *Read* checkbox against *Repositories*. This ensures that a user of the personal access token has read-only access to any repositories.

They will still be able to read any repositories the account has write access to. This is one of the reasons why read-only repository SSH keys bound to a specific repository are preferred.

When you are done with setting the permissions for the personal access token, click on **Create** and you will be shown the value of the token. Make sure you make a copy of this as you cannot view it later on in the Bitbucket settings.

Create the secret from the command line using the ``oc create secret`` command.

```
$ oc create secret generic user-at-bitbucket \
    --from-literal=username=machineuser \
    --from-literal=password=accesstoken \
    --type=kubernetes.io/basic-auth
```

You will need to supply the name of the user account which the personal access token was created under as the value to ``username``. It is better to create a machine user account for an organization, which has access to the repository, rather than use a personal user account. Also supply the access token as the value to ``password``.

Run ``oc secrets link`` to allow the builder service account to use it.

```
$ oc secrets link builder user-at-bitbucket
```

To create a new build and deployment using ``oc new-app``, which uses this source secret, supply the ``--source-secret`` option to ``oc new-app``, passing the name of the secret. Similarly, supply ``--source-secret`` to ``oc new-build`` if creating just a build.
