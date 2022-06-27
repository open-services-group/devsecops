# How to operate Superset using ODH

This section expands on existing documentation offered by ODH on how to deploy and manage Superset on OpenShift.

### Getting Started
Things you will need:

- OpenShift Cluster with Cluster Admin access
- Git
- Kustomize

### Install ODH Operator (Optional)
As per [ADR-0001][].

### Deploy Superset: Quick Start
This method works well if you do not plan to run a long term deployment of Superset, and intend to mostly use this for Demo purposes. You want to bring up Superset with as few steps as possible.

The best way to do this is to use the ODH operator. The operator allows you to install Superset manifests offered by ODH using the operator by deploying a `kfdef` custom resource. The `kfdef` acts like a pointer for the operator, you provide a source for where these manifests are located, and the operator will then deploy them in the namespace where that `kfdef` is deployed. Detailed instructions on how to use the `kfdef` to deploy Superset are offered by ODH, and can be found [here][odhsuperset].

The alternative is to simply deploy the manifests located in [odh-manifests][odh-manifests] repo directly. If using this method, then you can manually set the `kfdef` `parameters` by updating the file [here][parameters] and building these manifests using kustomize like so:

```
git clone https://github.com/opendatahub-io/odh-manifests.git
cd odh-manifests/superset
kustomize build .

# Alternatively you can log into your OCP cluster and deploy these manifests via command line
kustomize build . | oc apply -f -
```

### Deploy Superset: Long term instance
If you would like to maintain a deployment of Superset on OCP (with users) then the above solution is not sufficient. The main reason is that the manifests offered in the [odh-manifests][] repo are mostly a *recipe* for installing Superset in OCP, and often times you will need to update some of these to fit your individual use case. ODH makes an attempt at exposing some customization options by introducing [parameters][] but these are often not sufficient, and the human operator needs to manually expose other manifests as well. At the same time, we may want to stay up to date with the upstream _recipe_ so that we may benefit from their changes/support as time goes on.

To accomplish all this, we recommend maintaining a GitHub fork of the [odh-manifests][] repo in your own GitHub account or organization. You can then use this fork to configure all the manifests as needed for your deployment.

If you are using the ODH operator, you will need to update the `repo` field to point to your fork and branch/tag/commit that house the manifests you will like the operator to deploy. The operator expects a tar file, so use the GitHub tar link. For example if we wanted to deploy Superset from [this Operate First fork of ODH-Manifests][odh-manifests-fork] we would structure the kfdef like so:

```yaml
apiVersion: kfdef.apps.kubeflow.org/v1
kind: KfDef
metadata:
  name: superset
spec:
  applications:
    - kustomizeConfig:
        repoRef:
          name: manifests
          path: odh-common
      name: odh-common
    - kustomizeConfig:
        repoRef:
          name: manifests
          # The path in the fork where superset manifests reside
          path: superset
      name: superset
  repos:
      # Here we are deploying from a fork from `operate-first/odh-manifestgs` and branch `osc-cl2-v1.1.2`
    - name: manifests
      uri: https://github.com/operate-first/odh-manifests/tarball/osc-cl2-v1.1.2
  version: v1.1.0
```

As mentioned earlier, if not using the ODH operator, you can manually build and deploy the manifests using `kustomize`. To automate this process you can also use a tool like [ArgoCD][] to deploy these manifests.

### Key configuration files

#### Superset Config
As per [Superset documentation][] you need to manage a file called `superset_config.py`. ODH also includes this file in their Superset recipe, you can find it [here][odh-superset-config]. If you are applying new changes to the Superset configs, as offered from the docs, you will likely be updating this file.

#### Managing Secrets
You will find that the Superset recipe offered by ODH contains some k8s `secrets`. The ODH docs offer [a way to provided your own secret][odhsuperset]. We highly recommend you do this, and do not expose them in Git unless they are encrypted in some fashion. To introduce new secrets, you will need to update the `spec` portions in the `deployment` files within the `odh-manifests` repo. You may find this useful if you are looking to access confidential information from the `superset_config.py` via environment variables. Instead of exposing them in the config file, you can ingest them as enviroment variables which are in turn [pulled from k8s secrets][k8s-env].

#### Using external Database for Superset
Superset itself has a database that houses it's own metadata. The ODH recipe deploys its own Postgres database, but you can substitute your own. You will need to update the `SQLALCHEMY_DATABASE_URI` in the `superset_config.py` mentioned above. We recommend using a HA database for this purpose.

#### Use OCP rbac to map roles
The ODH Superset config comes OCP oauth integration. Docs for this can be found [here][oauth-docs].

### Considerations when using the Operator
Currently the Operator does not automatically pick up changes from `odh-manifests` , if you update your fork of `odh-manifests` you will need to reboot the ODH operator pod (found in `openshift-operators` if using OLM) to fetch new updates.

### Keeping up to date with upstream
It is a good idea to keep your manifests up to date with upstream. Use the fork model and rebase as new changes are adopted. This will ensure that you are up to date with the latest manifests, should any become outdated or incompatible with newer OCP versions.

### Do not rely on ODH manifests release notes when upgrading Superset
The ODH community cannot be reasonably expected to account for all factors when they upgrade any single component. Superset offers their own [release notes][]. If you notice the Superset image being updated in a given update to the `odh-manifests` repo, then ensure you also read the accompanying Superset release notes, and confirm no breaking changes to your environment are being introduced.

[release notes]: https://github.com/apache/superset/releases
[k8s-env]: https://kubernetes.io/docs/concepts/configuration/secret/#using-a-secret
[oauth-docs]: https://github.com/operate-first/odh-manifests/tree/osc-cl2-v1.1.2/superset#superset-config-file-customization
[odh-superset-config]: https://github.com/opendatahub-io/odh-manifests/blob/master/superset/base/superset_config.py
[Superset documentation]: https://superset.apache.org/docs/installation/configuring-superset
[ArgoCD]: https://github.com/argoproj/argo-cd
[odh-manifests-fork]: https://github.com/operate-first/odh-manifests/tree/osc-cl2-v1.1.2
[parameters]: https://github.com/opendatahub-io/odh-manifests/blob/master/superset/base/params.env
[odh-manifests]: https://github.com/opendatahub-io/odh-manifests/tree/master/superset
[odhsuperset]: https://github.com/opendatahub-io/odh-manifests/blob/master/superset/README.md
[ADR-0001]: https://raw.githubusercontent.com/open-services-group/devsecops/master/ADR/0001-dont-use-olm.md
