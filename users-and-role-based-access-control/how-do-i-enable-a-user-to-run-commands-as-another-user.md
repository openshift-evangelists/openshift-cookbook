---
related:
    - projects-and-user-collaboration/how-do-i-allow-another-user-to-work-on-my-project.md
---

If multiple users need to collaborate together on the development or management of an application, you would grant each separate user needing access to the application, a role in the project the application is deployed in. A separate login is used for each user. You should not create a single user and share login credentials as then you cannot control separately what each user can do, and revoking access to one user means changing the login credentials of the single account, affecting all users.

If it is necessary to be able to run an action as another user, then this can be done from a cluster admin account using user impersonation.

```
$ oc new-project username-project --as username
```

If you need to grant a non admin user the ability to run commands as another user, the ability to impersonate other users can be granted to that user via a cluster role.

From an account that can execute commands as a cluster admin, you can create the cluster role using ``oc create clusterrole``. Replace ``username`` with that of the user as appropriate.

```
$ oc create clusterrole impersonate-username --verb impersonate --resource users --resource-name username --as system:admin
```

To assign this role to a user, use the ``oc adm policy add-cluster-role-to-user`` command. Replace ``developer`` with the name of the user who is being granted this ability.

```
$ oc adm policy add-cluster-role-to-user impersonate-username developer
```

The user ``developer`` could then run the command:

```
$ oc new-project username-project --as username
```

Instead of a role for impersonating a single user, the cluster role could be configured to allow impersonation of any user in a specific group.

```
$ oc create clusterrole impersonate-usergroup --verb impersonate --resource groups --resource-name groupname --as system:admin
```

This could be used as a means of delegating some admin responsibilities for managing a group of users to a specific user.

Instead of granting the role to a user, the role could also be granted to a service account. This might be used where an application needs to perform actions against the OpenShift cluster as if it were a specific user. This might be used by an application which helps manage the creation of projects and deployment of applications on behalf of a user.

```
$ oc create sa serviceaccountname
$ oc adm policy add-cluster-role-to-user impersonate-usergroup -z serviceaccountname -n projectname
```

Having created the service account and added the role to it, you would then run the application as that service account. Any users which the application needed to create projects and application deployments for, would then be added to the group ``groupname``.
