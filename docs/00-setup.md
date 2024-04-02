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
oc apply -k https://github.com/redhat-cop/gitops-catalog/openshift-gitops-operator/operator/overlays/latest
```

And once that is up and running, let's configure it... with GitOps!

```
oc apply -k https://github.com/pittar-demos/demo-catalog/openshift-gitops-instances/argocd/openshift-gitops
```

That will create an Argo CD `Application` that will update the existing config for the Argo CD instance that lives in the `openshift-gitops` namespace.

Great!  Now we have OpenShift GitOps installed and configured.