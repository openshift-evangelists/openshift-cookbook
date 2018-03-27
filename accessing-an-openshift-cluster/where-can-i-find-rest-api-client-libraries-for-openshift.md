---
related:
    - accessing-an-openshift-cluster/how-can-i-create-a-service-account-for-scripted-access.md
---

As a user, you would normally interact with OpenShift via the web console or ``oc`` command line client. When using either of these methods, under the covers they are talking to OpenShift via a REST API endpoint.

You can access this REST API endpoint directly using a HTTP client such as ``curl``.

```
#!/bin/sh

SERVER=`oc whoami --show-server`
TOKEN=`oc whoami --token`

URL="$SERVER/oapi/v1/users/~"

curl -H "Authorization: Bearer $TOKEN" $URL
```

As well as being able to use a HTTP client such as ``curl``, REST API client libraries are also available for a number of different programming libraries.

* [Go](https://github.com/openshift/client-go)
* [Java](https://github.com/openshift/openshift-restclient-java)
* [Python](https://github.com/openshift/openshift-restclient-python)

Because OpenShift uses Kubernetes, you can also use any Kubernetes API client library, but will be restricted to only being able to use those to interact with Kubernetes resource objects and API endpoints. The Kubernetes clients will not know about additional resource object types and API endpoints added by OpenShift.
