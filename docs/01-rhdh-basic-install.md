# 1. Install Red Hat Developer Hub (Backstage)

The Red Hat Developer Hub **Operator** was installed as part of the GitOps setup.  This section concentrates on deploying and configuring your *instance* of Red Hat Developer Hub.

## Basic Install

First, let's deploy the most basic configuration to get Developer Hub up and running.  

The most basic configuration to get things up and running looks like this:

```
apiVersion: rhdh.redhat.com/v1alpha1
kind: Backstage
metadata:
  name: developer-hub
  namespace: rhdh
spec:
  application:
    appConfig:
      mountPath: /opt/app-root/src
    extraFiles:
      mountPath: /opt/app-root/src
    replicas: 1
    route:
      enabled: true
  database:
    enableLocalDb: true
```

This will create a Backstage instance as well as a PostgreSQL database.  Of course, we will deploy this with GitOps :)

```
oc apply -k https://github.com/pittar-demos/rhdh-build-an-idp/gitops/demo/argocd/rhdh-instance/basic -n openshift-gitops
```

If you're already logged into the Argo CD user interface, you can watch the Backstage deployment roll out.

You can also go to the `rhdh` namespace and watch the pods spin up:
[https://console-openshift-console.apps-crc.testing/topology/ns/rhdh?view=graph](https://console-openshift-console.apps-crc.testing/topology/ns/rhdh?view=graph)

When the pods have started up (both circles are dark blue), you can click on the route to go to your Red Hat Developer Hub UI!  It should be available at [https://backstage-developer-hub-rhdh.apps-crc.testing/](https://backstage-developer-hub-rhdh.apps-crc.testing/).

Next, we will configure add some configuration to make Developer Hub more useful!

[2: Golden Path Templates](02-gpts.md)
