When you create a project, the name you use for it must satisfy a couple of requirements.

The first requirement is that the name you choose is unique across the whole OpenShift cluster. This means you cannot use a project name that is already in use by another user.

```
$ oc new-project cookbook
Error from server (AlreadyExists): project.project.openshift.io "cookbook" already exists
```

If you delete a project and try to create a project with the same name immediately after, you can see still this error. This is because when you delete a project, it only schedules the project for deletion. It could take a period of time to shutdown applications running in the project and cleanup resources, before it is all gone, and the project name can be reused. If using temporary projects as part of a test system, you should aim to generate a new unique name for each project created rather than try and reuse the same name.

The second requirement is that the name can only use lower case letters, numbers and the dash character. This is necessary as the project name is used as a component in the hostname assigned to an application when it is made visible outside of the OpenShift cluster.

```
$ oc new-project openshift_cookbook
The ProjectRequest "openshift_cookbook" is invalid: metadata.name: Invalid value: "openshift_cookbook": a DNS-1123 label must consist of lower case alphanumeric characters or '-', and must start and end with an alphanumeric character (e.g. 'my-name',  or '123-abc', regex used for validation is '[a-z0-9]([-a-z0-9]*[a-z0-9])?')
```

If the project name is valid and not in use, yet the command to create a new project fails, it can be because the user has not been assigned the necessary role to be able to create projects, or any quota on the number of projects you can create has been exceeded.

```
$ oc new-project cookbook
Error from server (Forbidden): projectrequests.project.openshift.io "cookbook" is forbidden: user cookbook@redhat.com cannot create more than 1 project(s).
```
