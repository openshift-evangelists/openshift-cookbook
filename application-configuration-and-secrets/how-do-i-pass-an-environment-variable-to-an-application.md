Environment variables can be used to pass configuration to an application when it is run. This is done by adding the definition of the environment variable to the deployment configuration for the application.

To add a new environment variable use the ``oc set env`` command.

```
$ oc set env dc/your-app-name BANNER_COLOR=blue
```

Adding an environment variable to a deployment configuration would typically result in a new deployment being triggered.

If you need to set more than one environment variable at the same time, you can list them all with the one command:

````
$ oc set env dc/your-app-name BANNER_COLOR=blue SITE_NAME="My Blog"
```

If you have the environment variables to be set in a file or want to set them from your local environment, you can pipe them into the ``oc set env`` command, passing a ``-`` to indicate it should read them from the pipe:

```
$ env | grep '^AWS_' | oc set env dc/your-app-name -
```

Any time you are setting the value of an environment variable, if you need to compose the value from other environment variables that are already being set, you can use ``$(<VARNAME>)`` in the value. Ensure you surround the argument with single quotes when setting it from the command line, to avoid the local shell trying to interpret the value:

```
$ oc set env dc/your-app-name \
  DATABASE_USERNAME=user145c30ca \
  DATABASE_PASSWORD=EbAYDR1sJsvW \
  DATABASE_URL='postgres://$(DATABASE_USERNAME):$(DATABASE_PASSWORD)@blog-db/blog'
```

If you need the value to include the literal string of form ``$(<VARNAME>)``, use ``$$(<VARNAME>)`` to prevent it from being interpreted. The result will be passed through as ``$(<VARNAME>)``.

To list the names and values of the environment variables run the ``oc set env`` command and pass the ``--list`` option:

```
set env dc/your-app-name --list
# deploymentconfigs your-app-name, container blog
DATABASE_USERNAME=user145c30ca
DATABASE_PASSWORD=EbAYDR1sJsvW
DATABASE_URL=postgres://user145c30ca:EbAYDR1sJsvW@blog-db:5432/blog
```

To delete an environment variable, instead of using the name of the variable followed by = and the value, use the name of the variable followed by ``-``:

```
$ oc set env dc/your-app-name BANNER_COLOR-
```
