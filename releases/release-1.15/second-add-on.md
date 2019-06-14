## API Changes

- CRDs get support for x-kuberntes-int-or-string to allow faithful representation of IntOrString types in CustomResources. ([#78815](https://github.com/kubernetes/kubernetes/pull/78815), [@sttts](https://github.com/sttts))


## Notes From Multiple SIGs

### SIG Cloud Provider, and SIG VMWare

- vSphere: allow SAML token delegation (required for Zones support) ([#78876](https://github.com/kubernetes/kubernetes/pull/78876), [@dougm](https://github.com/dougm))

### SIG Autoscaling, SIG Cluster Lifecycle, and SIG GCP

- Update Cluster Autoscaler to 1.15.0; changelog: https://github.com/kubernetes/autoscaler/releases/tag/cluster-autoscaler-1.15.0 ([#78866](https://github.com/kubernetes/kubernetes/pull/78866), [@losipiuk](https://github.com/losipiuk))

### SIG Cluster Lifecycle, and SIG Testing

- Revert the CoreDNS version to 1.3.1 ([#78691](https://github.com/kubernetes/kubernetes/pull/78691), [@rajansandeep](https://github.com/rajansandeep))

### SIG Apps, and SIG Testing

- Resolves spurious rollouts of workload controllers when upgrading the API server, due to incorrect defaulting of an alpha procMount field in pods ([#78885](https://github.com/kubernetes/kubernetes/pull/78885), [@liggitt](https://github.com/liggitt))


## Notes from Individual SIGs

### SIG API Machinery

- CRDs get support for x-kuberntes-int-or-string to allow faithful representation of IntOrString types in CustomResources. ([#78815](https://github.com/kubernetes/kubernetes/pull/78815), [@sttts](https://github.com/sttts))

### SIG Cluster Lifecycle

- kubeadm: implement retry logic for certain ConfigMap failures when joining nodes ([#78915](https://github.com/kubernetes/kubernetes/pull/78915), [@ereslibre](https://github.com/ereslibre))

### SIG Storage

- Minor fix ConstructVolumeSpec in iSCSI plugin ([#68108](https://github.com/kubernetes/kubernetes/pull/68108), [@wenjun93](https://github.com/wenjun93))



