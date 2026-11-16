# test-app Helm umbrella chart

This repository provides a starter Helm umbrella chart for deploying an MSA
application, Kubernetes policies, and a VirtualMachineInstance (VMI) on SUSE
Virtualization (KubeVirt).

## Chart layout

```text
test-app/
├── Chart.yaml
├── values.yaml
└── charts/
    ├── msa-application/          # Deployment, Service, ConfigMap, Ingress, HPA
    ├── policy/                   # MSA NetworkPolicy, virtualization NetworkPolicy, PSA labels
    └── vm/
        └── templates/            # SUSE Virtualization VMI
```

## Prerequisites

- Kubernetes and Helm 3
- SUSE Virtualization/KubeVirt installed before enabling the VMI
- A CNI that enforces Kubernetes NetworkPolicy, if NetworkPolicies are enabled
- A namespace prepared for Pod Security Admission (PSA), if PSA labels are enabled

## Install

```sh
helm dependency update ./test-app
helm upgrade --install test-app ./test-app --namespace test-app --create-namespace
```

The application and policy chart are enabled by default. NetworkPolicies,
Namespace creation/PSA labels, and the VMI are opt-in. To let the chart create
a namespace with PSA labels, install the release into an existing management
namespace and set a separate target namespace:

```sh
helm upgrade --install test-app ./test-app \
  --namespace helm-system \
  --set global.targetNamespace=test-app \
  --set policy.namespace.create=true
```

The target namespace must not already exist when chart ownership is enabled.
When `global.targetNamespace` is empty, workloads use the Helm release
namespace; in that mode, create/manage the namespace outside the chart.

To render a VMI for review (requires KubeVirt at apply time):

```sh
helm template test-app ./test-app \
  --namespace test-app \
  --set vm.virtualizationVmi.enabled=true
```

Configure workload images, resources, policies, and VM shape in
`test-app/values.yaml`. Replace example image/configuration values before
production use. NetworkPolicy selectors and ingress/egress rules are cluster
and application specific; the chart intentionally does not assume traffic
should be allowed or denied.

An enabled NetworkPolicy with empty `ingress` or `egress` rules denies traffic
in that policy direction for its selected pods. Define and validate the rules
before enabling either policy.