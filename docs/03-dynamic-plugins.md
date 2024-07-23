# 3. Dynamic Plugins

You have your first Golden Path Template ready to go in Red Hat Developer Hub, but it wont' work quite yet... you still need to integrate Developer Hub with Gitea for scaffolding and with Argo CD to create new Argo CD Applications!

## Enabling Dynamic Plugins

Red Hat Developer Hub comes with a number of supported dynamic plugins.  [You can see the list of pre-installed dynamic plugins here](https://access.redhat.com/documentation/en-us/red_hat_developer_hub/1.1/html/administration_guide_for_red_hat_developer_hub/rhdh-installing-dynamic-plugins#con-preinstalled-dynamic-plugins).

In addition, you can include community dynamic plugins.  For this next step we will need both supported plugins (for Argo CD) and community (for Gitea).

To enable Dynamic Plugins, first create a new `ConfigMap` called `dynamic-plugins-rhdh.yaml`.  It will look like this:

```
kind: ConfigMap
apiVersion: v1
metadata:
  name: dynamic-plugins-rhdh
data:
  dynamic-plugins.yaml: |
    includes:
      - dynamic-plugins.default.yaml
    plugins:
      - package: '@backstage/plugin-scaffolder-backend-module-gitea'
        disabled: false
        integrity: sha512-cysdZLSttt7nv6cScEAHC+oOFe8gnfdMfNQ8tnJkGnlp8SkKO5Z19efyFEvKPSMF3VDQbffE0kz1vIyM+l9HZA==
      - package: ./dynamic-plugins/dist/roadiehq-backstage-plugin-argo-cd
        disabled: false
      - package: ./dynamic-plugins/dist/roadiehq-backstage-plugin-argo-cd-backend-dynamic
        disabled: false
      - package: ./dynamic-plugins/dist/roadiehq-scaffolder-backend-argocd-dynamic
        disabled: false
```

This includes the `Gitea` community dynamic plugin, as well as three Argo CD plugins that are pre-packaged and supported by Red Hat.  You can find the Argo CD plugins on the list of supported plugins that was linked above.  Note how you add `disabled: false` in order to "enable" a plugin.

This will require another update to the `app-config-rhdh.yaml` ConfigMap.  This stanza has been added:

```
    argocd:
      waitCycles: 25
      secure: false
      projectSettings:
        # Sets the allowed resources at the cluster level
        clusterResourceWhitelist:
          - group: '*'
            kind: '*'
        # Sets the allowed resources at the namespace level
        namespaceResourceWhitelist:
          - group: '*'
            kind: '*'
      appLocatorMethods:
        - type: 'config'
          instances:
            - name: openshift-gitops
              url: https://openshift-gitops-server-openshift-gitops.apps-crc.testing
              username: ${ARGOCD_ADMIN_USER}
              password: ${ARGOCD_ADMIN_PASSWORD}
```

This tells Developer Hub where to find your Argo CD instance as well as what credentials to use.  You can have more than one instance of Argo CD configured, but that goes beyond the scope of this demo.

As you can see, we need to add more to our secret.  The easiest way to do that is to delete the old secret and re-create it.

```
oc delete secret rhdh-secrets -n rhdh

oc create secret generic rhdh-secrets  \
    --from-literal=BACKEND_AUTH_SECRET=abc123 \
    --from-literal=GITEA_USERNAME=developer \
    --from-literal=GITEA_PASSWORD=openshift \
    --from-literal=ARGOCD_ADMIN_USER=admin \
    --from-literal=ARGOCD_ADMIN_PASSWORD=$(oc extract secret/openshift-gitops-cluster -n openshift-gitops --to=-) \
    --from-literal=K8S_CLUSTER_NAME=openshiftlocal \
    --from-literal=K8S_CLUSTER_URL=https://api.crc.testing:6443 \
    --from-literal=K8S_CLUSTER_TOKEN=$(oc get secret rhdh-token -n rhdh -o jsonpath="{.data['token']}" | base64 -d) \
    -n rhdh
```

Finally, you will need to update the `Backstage` CRD again so that the operator knows about the new Dynamic Plugins `ConfigMap`.

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
```

## Update Red Hat Developer Hub

Just like last time, log into Argo CD and update your `rhdh-instance` Application... with GitOps!

```
oc apply -k \
    https://github.com/pittar-demos/rhdh-build-an-idp/gitops/demo/argocd/rhdh-instance/plugins \
    -n openshift-gitops
```

This time, update the path to end in `gitops` instead of `gpts`.

Save your Application and wait for Developer Hub to roll out.  When it's done, you should be able to create a GPT with the following result:
* New source and gitops private repos created in Gitea.
* Argo CD applicaiton created for the gitops repo.
* Argo CD sync status reporting in your new Component.

Next... adding authentication with Azure Entra ID.