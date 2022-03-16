# When To Use OLM

## Context and Problem Statement

Open Data Hub (ODH) currently recommends to be installed using Operator Lifecycle Manager (OLM). This deployment strategy introduces a known security vulnerability for the cluster. By default all users with `admin` and `edit` access on any namespace/project can deploy a kfdef resource which effects the whole cluster. They can point kfdefs to deploy manifests from any location and then put a cluster role binding with themself as a cluster admin. 

## Decision

OLM should not be used to deploy ODH unless users plan to only interact with services already installed on the cluster, and not the cluster directly. Therefore, users cannot be given `admin` and `edit` access to their namespaces when using OLM.

## Consequences

OLM made ODH easier to install and manage, so the decision to manually install ODH will add complexity to the deployment process.

### OLM Pros

- OLM allows maintainers granular authoring of the update path.
- OLM is easier to install.
- OLM managers do not need to manage resources manually.
- OLM makes it easier to manage updates.
- OLM is the currently encouraged/advertised way to install ODH

### OLM Cons

- Cluster admin access gives users the ability to create their own kfdefs, allowing them to deploy whatever they want (security risk)
     1. ODH has cluster admin, meaning ODH deploys kfdefs.
     2. A user can deploy a kfdef as a project admin or with an edit role.
     3. A user can point kfdef to deploy manifests from any location.
     4. A user can put a cluster rolebinding with me as a cluster admin in a repo and have a kfdef point to this repo
