# How to deploy ODH

- Status: acepted
- Deciders: Anand, Harshad, Humair, Gage, Gregory
- Date: 2022-03-21

## Context and Problem Statement

Open Data Hub (ODH) community currently recommends that their operator be [installed using Operator Lifecycle Manager][odhinstall] (OLM). This deployment strategy introduces a known security vulnerability for the cluster. When ODH is installed on a cluster via OLM it is given *Cluster Admin* to that cluster. ODH utilizes a custom resource called a `kfdef` which effectively works as a pointer to a collection of manifests that a user can instruct ODH to deploy. Therefore, if a user has the ability to deploy a `kfdef` they can use ODH to deploy any resources to this cluster. When installing via OLM, every namespace/project admin (roles that come with any OpenShift installation) is given aggregated roles to be able to deploy `kfdefs`. This means such project admins can utilize ODH as a back door into taking over the cluster (e.g. by deploying `clusterrolebindings` with `clusteradmin` level permissions).

This is a serious vulnerability, however it is only an issue if the cluster is expected to have direct OCP users that will be directly interfacing withe cluster. For example, if a team plans to deploy ODH in a cluster that contains existing OCP users that own projects and namespaces.

If instead, the cluster will only be directly interacted with by the cluster admins, and all end users will be interfacing with only the *components* that ODH will deploy. This vulnerability is not a concern. For example, there is no need to hand out project admin roles to users that will be interacting with JupyterHub, Superset, and the like.

## Considered Options

1. Only install ODH manually
2. Only use OLM but do not hand out project admins, instead manage custom created admin roles
3. Hybrid solution depending on use case. Use OLM when users will not interact with clusters directly. Us manual deployment if ODH will be deployed on a cluster with existing project admin OCP users.

OLM should not be used to deploy ODH unless users plan to only interact with services installed on the cluster, and not the cluster directly. Therefore, users cannot be given `admin` and `edit` access to their namespaces when using OLM to install ODH.

## Decision

We will opt for the 3rd option.

## Consequence

In clusters shared by project admins, the only reasonable choice is to run a manual ODH operator deployment. Manually managing project admin roles is a big pain point, as this is an OCP default, various services / operators add aggregations during installation to these roles. These roles would have to also be maintained and updated during OCP upgrades. There may be other integrations that we may also not be aware of and it is not trivial to account for all cases.

On clusters where users will only interface with services deployed/managed on the cluster OLM will make ODH easier to install and manage.

The following list outlines various pros of considering an OLM deployment path:

- OLM allows maintainers granular authoring of the update path.
- OLM is easier to install.
- OLM managers do not need to manage resources manually.
- OLM makes it easier to manage updates.
- OLM is the currently encouraged/advertised way to install ODH



[odhinstall]: https://opendatahub.io/docs/getting-started/quick-installation.html    "0"
