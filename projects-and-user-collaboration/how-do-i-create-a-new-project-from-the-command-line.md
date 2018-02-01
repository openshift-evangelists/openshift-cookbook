---
related:
    projects-and-user-collaboration/why-when-i-try-to-create-a-new-project-does-it-keep-failing.md
---

Projects can be created from the command line by using the ``oc new-project`` command.

```
$ oc new-project myproject --display-name 'My Project'
Already on project "myproject" on server "https://localhost:8443".

You can add applications to this project with the 'new-app' command.
For example, try:

    oc new-app centos/ruby-22-centos7~https://github.com/openshift/ruby-ex.git

to build a new example application in Ruby.
```

You can list all projects you have access to using the ``oc projects`` command.

```
$ oc projects
You have one project on this server: "My Project (myproject)".

Using project "myproject" on server "https://localhost:8443".
```

The name of the current project, against which your commands will be applied, can be determined by running the command ``oc project``.

```
$ oc project
Using project "myproject" on server "https://localhost:8443".
```

Where you have access to multiple projects, you can set the current project by running ``oc project`` and specifying the name of the project.

```
$ oc project myproject
Now using project "myproject" on server "https://localhost:8443".
```

When you create a new project using oc new-project, the new project will automatically be set as the current project.
