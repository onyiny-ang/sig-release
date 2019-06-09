## New Features

- Move online volume expansion to beta ([#77755](https://github.com/kubernetes/kubernetes/pull/77755), [@gnufied](https://github.com/gnufied)) Courtesy of SIG Node, SIG Storage, and SIG Testing


## API Changes

- Fixed a spurious error where update requests to the status subresource of multi-version custom resources would complain about an incorrect API version. ([#78713](https://github.com/kubernetes/kubernetes/pull/78713), [@liggitt](https://github.com/liggitt))


## Notes From Multiple SIGs

### SIG Node, and SIG Scheduling

- Revert Promotion of resource quota scope selector to GA ([#78696](https://github.com/kubernetes/kubernetes/pull/78696), [@ravisantoshgudimetla](https://github.com/ravisantoshgudimetla))

### SIG Node, and SIG Windows

- Fixes a memory leak in Kubelet on Windows caused by not not closing containers when fetching container metrics ([#78594](https://github.com/kubernetes/kubernetes/pull/78594), [@benmoss](https://github.com/benmoss))

### SIG Azure, and SIG Cloud Provider

- fix: retry detach azure disk issue ([#78700](https://github.com/kubernetes/kubernetes/pull/78700), [@andyzhangx](https://github.com/andyzhangx))

### SIG Cluster Lifecycle, SIG Node, and SIG Windows

- kube-up.sh scripts now disable the KubeletPodResources feature for Windows nodes, due to issue #78628. ([#78668](https://github.com/kubernetes/kubernetes/pull/78668), [@mtaufen](https://github.com/mtaufen))

### SIG Apps, and SIG Autoscaling

- Horizontal Pod Autoscaling can now scale targets up even when one or more metrics are invalid/unavailable as long as one metric indicates a scale up should occur. ([#78503](https://github.com/kubernetes/kubernetes/pull/78503), [@gjtempleton](https://github.com/gjtempleton))

### SIG Network, and SIG Windows

- Windows kube-proxy will wait for HNS network creation on start ([#78612](https://github.com/kubernetes/kubernetes/pull/78612), [@ksubrmnn](https://github.com/ksubrmnn))


## Notes from Individual SIGs

### SIG API Machinery

- Fix admission metrics histogram bucket sizes to cover 25ms to ~2.5 seconds. ([#78608](https://github.com/kubernetes/kubernetes/pull/78608), [@jpbetz](https://github.com/jpbetz))
- Fixed a spurious error where update requests to the status subresource of multi-version custom resources would complain about an incorrect API version. ([#78713](https://github.com/kubernetes/kubernetes/pull/78713), [@liggitt](https://github.com/liggitt))

### SIG Cluster Lifecycle

- kubeadm: revert the CoreDNS version to 1.3.1 ([#78545](https://github.com/kubernetes/kubernetes/pull/78545), [@neolit123](https://github.com/neolit123))

### SIG Storage

- Fix issue with kubelet waiting on invalid devicepath on AWS ([#78595](https://github.com/kubernetes/kubernetes/pull/78595), [@gnufied](https://github.com/gnufied))
- StorageOS volumes now show correct mount information (node and mount time) in the StorageOS administration CLI and UI. ([#78522](https://github.com/kubernetes/kubernetes/pull/78522), [@croomes](https://github.com/croomes))



