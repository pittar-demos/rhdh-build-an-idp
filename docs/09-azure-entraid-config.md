# Install Red Hat Developer Hub (Backstage)

## Add Authentication with Azure Entra ID

One of the first things any IDP team will want to do is configure authentication.  For this demo we will use Azure Entra ID (formerly Azure AD).  Keycloak in another great option, but since this demo is running on OpenShift Local, offloading this task to Entra ID will save some local resources.

### Step 1:  Create an App Registration

This isn't meant to be a tutorial on EntraID, but hopefully the following instructions will be good enough to get you going for this demo!

If you have an existing App Registration to use, you can skip this next part.

Log into your Azure portal and go to Entra ID, then:
* Navigate to the "App Registration" option in the left navigation.
* Click on `+ New registration` near the top of the screen.
* Give your App Registration a name.
* Keep the "Supporte account types" at the default setting.
* For "Redirect URI" select **Web** from the drop down and for the redirect value use `https://backstage-developer-hub-rhdh.apps-crc.testing/api/auth/microsoft/handler/frame`.  This is the default backstage redirect URL you should have for your OpenShift Local instance.
* Click "Register"

You now have an App Registration!

If you're re-using an existing App registration, be sure to add the same callback url to your existing App Registration.

Take note of your `Application (Client) ID` and your `Directory (tenant) ID`.  You'll need those later.

For "Client Credentials", click on the link to create a new secret.
* Create a new client secret.
* Give it a description.
* Choose a duration for the secret.
* Click "Add"

On the next screen, copy the "Value" (not the Secret ID). You won't be able to get it back once you navigate away from this page, so keep it safe!

We're done with Azure for now!

### Step 2: Red Hat Developer Hub Secrets

Some of your configuration for Red Hat Developer Hub will be secret (like your App Registration Secret).  To keep these secrets "secret", you will use a specific Kubernetes `Secret`.

Create a yaml file called `rhdh-secrets.yaml` that has the following contents:

```
kind: Secret
apiVersion: v1
metadata:
  name: rhdh-secrets
  namespace: rhdh
type: Opaque
stringData:
  BACKEND_AUTH_SECRET: abc123
  AZURE_CLIENT_ID: yourAppRegClientID
  AZURE_TENANT_ID: yourAppRegTenantID
  AZURE_CLIENT_SECRET: yourClientSecretValue
```

Substitute your actual values in and create this secret in the `rhdh` namespace.

```
oc apply -f rhdh-secrets.yaml -n rhdh
```

For the actual configuation of Red Hat Developer Hub, you need a `ConfigMap` named `app-config-rhdh.yaml`.  One already exists in this repo (at `/gitops/demo/overlays/auth`).