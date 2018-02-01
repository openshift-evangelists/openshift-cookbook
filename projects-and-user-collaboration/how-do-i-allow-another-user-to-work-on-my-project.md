As the owner of the project, initially you are the only one who can access it and work in it. If you need to collaborate on a project with other users, you can add additional members to the project. When adding a user to the project, they can be added in one of three primary roles.

* ``admin`` - A project manager. The user will have rights to view any resource in the project and modify any resource in the project except for quota. A user with this role for a project will be able to delete the project.
* ``edit`` - A user that can modify most objects in a project, but does not have the power to view or modify roles or bindings. A user with this role can create and delete applications in the project.
* ``view`` -  A user who cannot make any modifications, but can see most objects in a project.

To add another user with edit role to the project, so they can create and delete applications, you need to use the ``oc adm policy`` command. You must be in the project when you run this command.

```
oc adm policy add-role-to-user edit <collaborator>
```

Replace ``<collaborator>`` with the name of the user as displayed by the ``oc whoami`` command when run by that user.

To remove a user from a project, run:

```
oc adm policy remove-role-from-user edit <collaborator>
```

To get a list of the users who have access to a project, and in what role, a project manager can run the ``oc get rolebindings`` command.
