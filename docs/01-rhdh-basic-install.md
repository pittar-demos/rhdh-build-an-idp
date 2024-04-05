# Install Red Hat Developer Hub

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
oc apply -k https://github.com/pittar-demos/rhdh-build-an-idp/gitops/demo/argocd/rhdh-instance -n openshift-gitops
```