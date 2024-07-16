# Intial Setup

An IDP is build on a solid foundation of tools and processes.  This means, we need a few tools in place before we start building out our IDP.  For this demo I've chosen the following tool stack.  Your setup might be different.  This is perfectly fine!  Red Hat Developer Hub (and by extension, Backstage) includes plugins for almsot all of the most popular CI tools.

* Kubernetes:  OpenShift Local - a free single-node OpenShift cluster you can run on your laptop.
* Git:  I'm using Gitea for simplicity.  
* Argo CD: I'm using OpenShift GitOps as it is the Red Hat supported build of Argo CD!
* Pipelines: OpenShift Pipelines (Tekton).
* Quay.io: Image registry.  Powerful and free to use for individuals.
* Authentication: Azure EntraID (formerly Azure AD).

## Install OpenShift GitOps

Let's install GitOps with the CLI... who wants to point-and-click anyway?

```
oc apply -k https://github.com/pittar-demos/rhdh-build-an-idp/gitops/setup/openshift-gitops/operator
```

And once that is up and running, let's complete the setup with GitOps!

```
oc create -f https://raw.githubusercontent.com/pittar-demos/rhdh-build-an-idp/main/gitops/setup/argocd/bootstrap/bootstrap-application.yaml -n openshift-gitops
```

This will:
* Perform additional configuration of Argo CD.
* Install the Gitea operator and a Gitea instance.
* Deploy the Red Hat Developer Hub (Backstage) operator.

This will take a few minutes to complete.  When you see Gitea (in the `scm` namespace) is up and running, you can continue on to the next step.

**NOTE:** If things seem stuck, log in to your new OpenShift GitOps instance and see if any of the apps need a manual "Sync".

Your base configuration is now in place and you're ready to start deploying and configuring your Internal Developer Platform!

[1: Basic Red Hat Developer Hub Installation](01-rhdh-basic-install.md)