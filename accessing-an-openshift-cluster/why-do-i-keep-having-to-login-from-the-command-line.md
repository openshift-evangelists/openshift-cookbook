---
related:
    - accessing-an-openshift-cluster/how-can-i-create-a-service-account-for-scripted-access.md
---

When you login from the command line as a user a new session is created. A token corresponding to the session will be cached within your local account. When you run the ``oc`` command line tool, that token will be supplied with each request made to OpenShift.

In a typical OpenShift environment these session tokens will expire after one day. The expiry time will differ if the administrator of the OpenShift environment has overridden the default value used.

Once the session token expires, to use the ``oc`` command line tool, you will need to login again.

Because the login session will expire without notice, you should avoid using the session token for a normal user account with scripts using the OpenShift REST API. For scripted access it is better to create a service account. The service account has an associated access token which you can use and which will not expire.
