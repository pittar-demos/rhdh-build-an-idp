# 2. Golden Path Templates

Now that Red Hat Developer Hub is running, it's time to talk about Golden Path Templates (GPTs).

The Gitea instance already has a sample GPT ready to be pulled in by Red Hat Developer Hub, but you will need to update your config to see it.

## Coniguration and Secrets

It's time for some more advanced configuration.  In order to use Gitea we need to configure the [Gitea integration](https://backstage.io/docs/integrations/gitea/locations).  To do this we will need a few things:

1. A `Secret` containing username/password information.
2. A `ConfigMap` with additional Backstage configuration.
3. An update to the `Backstage` CRD to tell the operator where to find this config information.

Let's start with the secret.  Since a secret shouldn't be stored in Git, we'll create it with the command line.  You can also use your secret management tool of choice, of course.

```
oc create secret generic rhdh-secrets  \
    --from-literal=BACKEND_AUTH_SECRET=abc123 \
    --from-literal=GITEA_USERNAME=developer \
    --from-literal=GITEA_PASSWORD=openshift \
    -n rhdh
```

This will create a secret with the Gitea username and password in the `rhdh` namespace.

Next, we will create a `ConfigMap` called `app-config-rhdh.yaml` where we will specify our additional configuration.  Here's what that looks like:

```
kind: ConfigMap
apiVersion: v1
metadata:
  name: app-config-rhdh
data:
  app-config-rhdh.yaml: |
    app:
      title: Super Duper IDP
      baseUrl: https://backstage-developer-hub-rhdh.apps-crc.testing

    integrations:
      gitea:
        - host: gitea-scm.apps-crc.testing
          username: ${GITEA_USERNAME}
          password: ${GITEA_PASSWORD}

    catalog:
        rules:
          - allow: [Component, System, Group, Resource, Location, Template, API]
        locations:
          - type: url
            target: https://gitea-scm.apps-crc.testing/developer/rhdh-build-an-idp/src/branch/main/gpts/demo-templates.yaml
            
    backend:
      auth:
        keys:
          - secret: ${BACKEND_AUTH_SECRET}
      baseUrl: https://backstage-developer-hub-rhdh.apps-crc.testing
      cors:
        origin: https://backstage-developer-hub-rhdh.apps-crc.testing

    enabled:
      gitea: true
```

The important pieces to note are:

* The `app` stanza defines a custom title for our Developer Hub instance.
* The `integrations` stanza now includes Gitea config.  This in turn specifies:
    * The location of our Gitea repostiroy.
    * The username and password to use.  These values will come from the secret we created.
    * The `catalog` stanza points to the location in Gitea where our list of GPTs can be found.  This list is what will populate Developer Hub.

Finally, we will update the `Backstage` CRD to reference the new `ConfigMap` and `Secret`:

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
      configMaps:
        - name: app-config-rhdh
    dynamicPluginsConfigMapName: dynamic-plugins-rhdh
    extraEnvs:
      envs:
        # Only needed because gitea uses a self-signed cert.
        - name: NODE_TLS_REJECT_UNAUTHORIZED
          value: "0"
      secrets:
        - name: rhdh-secrets
    replicas: 1
    route:
      enabled: true
  database:
    enableLocalDb: true
```

## Update Red Hat Developer Hub

To update Developer Hub, log into Argo CD and: 
* Open the `rhdh-instance` Application.
* Click the **Edit** button.
* Update the git path to end with `gpts` instead of `basic`.
* Click **Save**.

Since the Backstage CRD has been update, the update should automatically roll out.  If for some reason you don't see your Backstage pod cycling, then you may need to manually delete it.  It will take a few minutes for the new pod to spin up.

## Viewing your GPT.

Once Developer Hub has started again, you can go back to the UI and click the `Create` link from the left navigation menu and see your first Golden Path Template!

If you want to better understand how the template works, you can browse to the template in the Gitea repository and have a look.

Don't create an instance of this GPT quite yet, as a few integrations are still required.

Next, we will add Dynamic Plugins to enable Gitea repo scaffolding and Argo CD integration.

[3: Dynamic Plugins](03-dynamic-plugins.md)