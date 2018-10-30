The ``oc get`` command allows you to display details about resource objects. When run, it will display a set of default fields for that resource type. You can use a label selector to restrict the query to a set of resource objects.

```
$ oc get pods --selector app=training
NAME                               READY     STATUS    RESTARTS   AGE
training-22-g67cv                  1/1       Running   0          1h
training-dc712ef4-d9b5-11e8-93ff   1/1       Running   0          1h
```

By default a minimal set of fields will be displayed for any resource type. You can display a wider set of fields using the ``wide`` output format.

```
$ oc get pods --selector app=training -o wide
NAME                               READY     STATUS    RESTARTS   AGE       IP           NODE
training-22-g67cv                  1/1       Running   0          1h        172.17.0.4   localhost
training-dc712ef4-d9b5-11e8-93ff   1/1       Running   0          1h        172.17.0.5   localhost
```

In either case, extracting out specific fields is not convenient due to the use of variable amounts of white space between fields, with the possibility that field values can be missing when there is no value.

A few options exist to make it easier to extract and display attributes from resource objects.

The first is that you can retrieve a raw resource object definition as ``yaml`` or ``json``.

To retrieve details on a specific resource object by name you can use:

```
$ oc get pod/training-22-g67cv -o json
{
    "apiVersion": "v1",
    "kind": "Pod",
    ...
}
```

You could capture the YAML or JSON and process it using a scripting language such as Python.

```
$ oc get pod/training-22-g67cv -o json | \
  python -c "import json, sys; data=json.loads(sys.stdin.read()); print(data['status']['podIP'])"
172.17.0.4
```

Note that in this case, because the query of the resource object was by name, only a single object is returned. If you do not specify a resource object name, or query using a label selector, you will get back a list of objects and you would need to iterate over the list and extract details from each as necessary.

```
$ oc get pods --selector app=training -o json
{
    "apiVersion": "v1",
    "kind": "List"
    "items": [
        {
            "apiVersion": "v1",
            "kind": "Pod",
            ...
        },
        {
            "apiVersion": "v1",
            "kind": "Pod",
            ...
        }
    ],
    ...
}
```

Another option is to use an output template. Multiple fields can be selected for display using this.

```
$ oc get pod/training-22-g67cv -o template --template '{{.metadata.name}} {{.status.podIP}}{{"\n"}}'
training-22-g67cv 172.17.0.4
```

If the output from ``oc get`` returns a list of objects, it is not enough to provide a template pertaining to one instance of the object.

```
$ oc get pod --selector app=training -o template --template '{{.metadata.name}}"\n"{{.status.podIP}}{{"\n"}}'
<no value>"\n"<no value>
```

In this case, you need to use a loop within the template to apply it to each returned object.

```
$ oc get pod --selector app=training -o template --template '{{range .items}}{{.metadata.name}} {{.status.podIP}}{{"\n"}}{{end}}'
training-22-g67cv 172.17.0.4
training-dc712ef4-d9b5-11e8-93ff 172.17.0.5
```

Much more complicated templates can be constructed, including use of conditionals and definition of sub templates.

For more details on the template language, see:

* https://golang.org/pkg/text/template/

For an explanation of what fields in a raw resource definition represent, you can use the ``oc explain`` command.

```
$ oc explain pod.status.podIP
FIELD: podIP <string>

DESCRIPTION:
     IP address allocated to the pod. Routable at least within the cluster.
     Empty if not yet allocated.
```
