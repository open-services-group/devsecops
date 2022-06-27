# How Operate-first Deploys and Manages ODH

This section will expalin how deployments of ODH work within Operate-First.

## How is ODH Deployed?

This has been covered in depth in [ADR-0001][]. We use OLM to deploy ODH but refer to the ADR for more information.

## How are versions of ODH managed?

This is an issue that we are still working out, and once a resolution has been reached an ADR will be created for it. We are considering two paradigms for deployment for each version of ODH:

1. A `master`-like branch is created for each combination of the cluster and ODH version.
    - This is the method that we are currently utilizing to deploy ODH.
    - Example branches could look like branch `osc-cl2-v1.1.2` where `osc-cl2` is the name of the cluster, and `v1.1.2` is the tagged version of ODH used as the base of the branch.
    - Once a new branch has been properly constructed from a new version of ODH, we simply alter the `Kfdef`s in the respective overlay of [`operate-first/apps` kfdefs](https://github.com/operate-first/apps/tree/master/kfdefs/overlays) to point to that branch in [operate-first/odh-manifests][]) for the specified cluster.
    - How are branches constructed?
        - Branches are started by checking out a tag in [opendatahub-io/odh-manifests][] corresponding to the desired version of ODH.
        - Once the base of this branch has been constructed, the `odh-manifests` overlay corresponding to the cluster you wish to deploy to needs to be migrated from [operate-first/apps][] to [operate-first/odh-manifests][]. The reasoning behind this will be discussed [below](./odh-deployment.md#why-did-operate-first-fork-odh-manifests) however it can be summarized; operate first provides additional configurations for its deployments of ODH.

2. A `master`-like branch is created for each cluster, and a tag is created for each version of ODH.
    - Example branches could look like branch `osc-cl2` or `smaug`, refering to clusters in Operate-First.
    - For every version of ODH, a corresponding tag is created in `operate-first/odh-manifests` that is built from the tag of [opendatahub-io/odh-manifests], along with any related `Operate-First` cloud resources.
    - In this method, branches are simply constructed directly from the `operate-first/odh-manifests` tag corresponding to the version of ODH that you wish to deploy. In this way, the branch would get populated with both the upstream version changes and Operate-First related resources.

## Why did `operate-first` fork ODH manifests?

Certain configurations differ between `operate-first/odh-manifests` and `opendatahub-io/odh-manifests`. These resources are specific to the `Operate-First` cloud, such as various ODH component secrets, jupyterhub profiles and profile quotas, custom routes, etc, and stem from the fact that Operate-First maintains an actual deployment of ODH, where as the upstream repository ([opendatahub-io/odh-manifests][]) represents a catalog of components of ODH that can be deployed. This is why we do not recommend directly deploying from [opendatahub-io/odh-manifests][], because it will be missing any of these resources specific to your deployment. In our initial stages of deployment of ODH, we were happy to keep these configurations in `operate-first/apps` and use them to overide the odh-manifests available upstream. However, deploying ODH in a more secure way (similar to Red Hat's RHODS project) has been made a priority in the Operate-First community. As such, the oppertunity was taken to split it into a seperate repo, helping to provide a centralized location for our ODH configurations. As a result, this repository should serve as an example to anyone looking to extend ODH to meet their requirements.

## How should my kfdefs look?

Firstly our Kfdefs live in [operate-first/apps] rather than in this repository. However it is related to the deployment of `odh-manifests` so it will be discussed here. When creating a Kfdef to work with the Operate-First deployment of ODH there is really only one thing to keep in mind. The various `kustomizeConfigs` you may or may not have should reference the `operate-first/odh-manifests` repo at the specifc branch of the ODH version / Operate-First cluster to which you are attempting to deploy. For all other questions partaining to Kfdefs, refer to the [kubeflow operator documentaiton](https://www.kubeflow.org/docs/distributions/operator/introduction/).


[ADR-0001]: https://raw.githubusercontent.com/open-services-group/devsecops/master/ADR/0001-dont-use-olm.md
[operate-first/odh-manifests]: https://github.com/operate-first/odh-manifests
[opendatahub-io/odh-manifests]: https://github.com/opendatahub-io/odh-manifests
[operate-first/apps]: https://github.com/operate-first/apps
