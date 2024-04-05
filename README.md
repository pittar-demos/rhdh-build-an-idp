# Let's Build an IDP!

## Building an IDP with Red Hat Developer Hub (Backstage)

This demo will walk through the steps of deploying and customizing your own Internal Developer Portal (IDP) using Red Hat Developer Hub - Red Hat's supported and opinionated build of the Backstage project.

To keep things simple (and free!) this demo uses [OpenShift Local](https://developers.redhat.com/products/openshift-local/getting-started) - a free single-node OpenShift instance that you run on your laptop.  Since we are deploying a number of components to our cluster, I've given my cluster lots of resources to work with.  Here is my configuration:

```
$ crc config view
- consent-telemetry                     : no
- cpus                                  : 8
- disk-size                             : 100
- memory                                : 22888
- pull-secret-file                      : /home/apitt/config/pull-secret.txt
```

With that out of the way, it's time to get started!

[0: Base Cluster Configuration](docs/00-setup.md)