# Changes to the Operate First ODH Manifests

## Status

_Proposed_

## Context and Problem Statement

Open Data Hub (ODH) utilizes a custom resource called a `kfdef` which effectively works as a pointer to a collection of manifests that instruct ODH what to deploy. ODH currently [recommends](https://opendatahub.io/docs/administration/installation-customization/customization.html) to fork `odh-manifests` [repository](https://github.com/opendatahub-io/odh-manifests) and extend it with overlays for components to customize the installation.

The Operate First `osc-cl2` cluster keeps custom ODH manifests in the same repository as the the clusters' definition. They use overrides in the cluster's overlay's `kfdefs` to apply the changes to the manifests. In contrast, Red Hat Open Data Hub (RHODH) uses a forked repo of upstream ODH `odh-manifests`, makes changes, but bakes the manifests into the ODH operator image so that the operator does not have to reach out to GitHub at runtime.

## Considered Options
1. Use a forked version of ODH `odh-manifests` repo and maintain the customizations there.
2. Use overrides in the `kfdefs`.
3. Follow option 1 but bake the manifests into the operator.

## Decision

We chose option 1.

## Consequence

### JupyterHub
- increased `singleuser_pvc_size`
- removed authentication for prometheus metrics
- added admin and allowed groups
- added `node_tolerations` and `node_affinity` customizations to the `jupyterhub-singleuser-profiles.yaml` config file.

### Seldon
- removed the `startingCSV` from the subscription

### Superset
- added postgresql env variables
- removed some kustomization resources

### Trino
- removed some kustomization resources
- removed properties under volumes
- removed AWS env keys
