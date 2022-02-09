# ODH/RHODS Discovery

## about

This file will document differences in `ODH` and `RHODS` found through initial discovery. Its purpose is to answer core the questions which will allow the DevSecOPs working group to deploy a version of ODH very similar to `RHODS`.

## `RHODS` vs. `ODH`

|       | `RHODS`     | `ODH`   |
|-------|-----------|-------|
| Components | RODH Operator, JupyterHub, RHODS Dashboard  | Apache Airflow, Apache Kafka, Apache Spark, Apache Superset, Argo, Grafana, JupyterHub, Prometheus, Seldon [from `ODH` components](https://github.com/opendatahub-io/odh-manifests#Components) |
| Images used in deployment | rhods-operator, [rhods-jupyterhub](https://github.com/red-hat-data-services/jupyterhub-odh), Jupyterhub notebook images (Minimal, data-science, pytorch, tensorflow), [RHODS dashboard](https://github.com/red-hat-data-services/odh-dashboard), Monitoring stack (Grafana, Prometheus), configmap-puller image | [see `ODH` images](https://github.com/opendatahub-io/odh-images) |
| Differences in ODH-manifests | ODH-manifests are only updated from upstream components ([see ODH-deployer](https://github.com/red-hat-data-services/odh-deployer)) | |
| High Availability | Everything in `RHODS` is HA by requirement, including the monitoring stack. Where possible, the upstream components were first made HA prior to pulling them to downstream `RHODS`. | HA for `ODH` is something handled by the communities upstream of `ODH` per component. |
| Grafana Dashboards | Grafana Dashboard (kustomize combines [dashboards](https://github.com/opendatahub-io/odh-manifests/tree/master/grafana/grafana/base) into singular dashboard) which includes `RHODS`'s SLOs. Out of sync with upstream.  | includes [kafka changes](https://github.com/opendatahub-io/odh-manifests/pull/447), as Kafka is not deployed in `RHODS`. |
| SLIs | SLIs are the same for `RHODS` as for `ODH`. However SLOs for `RHODS` are defined [here](https://docs.google.com/document/d/1YsVYg5ZIdLYBKLxU4iEpt4e_KykE8C8nJD2SEFg52eY/edit) |  SLIs are the same for `RHODS` as for `ODH`. |
| Kfdef service | It is expected that users never interact with a Kfdef object in `RHODS`. | |
| Kafka | `RHODS` doesnt include Kafk.a | Kafka deployed through ODH-manifest. |
| ODH-Dashboard | `RHODs` includes extra ISV content in their version of the ODH Dashboard. | `RHODS` uses the the [ODH-Dashboard](https://github.com/opendatahub-io/odh-dashboard) as a base. |
| Public Deployment Document | There is not such document for `RHODS`. | [ODH quick-installation guide](https://opendatahub.io/docs/getting-started/quick-installation.html) |
| Any differences based on time availability or human resources | No, `RHODS` is intended to be more stable, with all images built internally rather than images used by others. Differences are attributed to security and safety requirements of the `RHODS` product.| As open-source software, `ODH` is designed to be customizeable and open . |
| Resource Quotas | `RHODS` does not use Openshift Resource Quotas | Resource quotas in Openshift are set per namespace/project and `ODH` does enforce default quotas. Therefore if someone were trying to enforce quotas it could be done per project each component is deployed into, see [openshift docs](https://docs.openshift.com/container-platform/4.9/applications/quotas/quotas-setting-per-project.html) on how to do so |
| Deployment | While `RHODS` does use the opendatahub operator, it actually “bakes” the manifests into the operator image so that the operator does not have to reach out to github at runtime.  Since `RHODS` is installed via the OCM addon flow,an initContainer is used for the operator when it is installed to create the specially crafted KfDef that installs `RHODS`.  That initContainer also sets-up the monitoring stack and sets-up the buildconfigs for the runtime build chain. | `ODH` relies on operator/kfdef or kfctl to install (see [deployment section of odh-manifests repo](https://github.com/opendatahub-io/odh-manifests#Deploy)) |
| Database | `RHODS` utilizes a postgres database instance in AWS (created and managed by the Cloud Resource Operator--CRO) to handle the storage needed by jupyterhub.  That AWS database satisfies `RHODS`'s HA requirement that could not be satisfied by running a postgres database local to the cluster. | `ODH` is deployed on Openshift and without the HA requirement, it can be run with any type of storage that works in openshift. The [docs](https://opendatahub.io/docs/administration/advanced-installation/object-storage.html) suggest `Ceph` or `OCS` |
| ISV components | All ISV components used in `RHODS` can't be included with `RHODS` but instead are installed through the [Red Hat Marketplace](https://marketplace.redhat.com/en-us/search). | Can be installed directly through `Kfdef`s |
