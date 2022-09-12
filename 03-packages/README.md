# Install Tanzu Packages 1.5.4

Tanzu Packages 1.4 are officially supported by TKGs, unfortunatelly so the 1.4 packages won't work with Kubernetes 1.22+. So this is my approach at installting the 1.5 packages to a TKGs cluster.

If you have TMC (Tanzu Mission Controll) available please use the [Catalog feature](https://docs.vmware.com/en/VMware-Tanzu-Mission-Control/services/tanzumc-using/GUID-EF35646D-8762-41F1-95E5-D2F35ED71BA1.html) it will greatly improve the expirience.

- [Install Tanzu Packages 1.5.4](#install-tanzu-packages-154)
  - [Background](#background)
  - [Preparation](#preparation)
  - [TMC Installation](#tmc-installation)
  - [Setup kapp-controller and secretgen-controller](#setup-kapp-controller-and-secretgen-controller)
  - [Cert-manager](#cert-manager)
  - [Contour](#contour)
  - [External DNS](#external-dns)

## Background

An additional challenge with the Tanzu Packages is that they natively don't come with [PSPs (Pod Security Policies)](https://kubernetes.io/docs/concepts/security/pod-security-policy/). They assume there are no PSPs enabled in the cluster. On the other hand PSPs are enabled by default with TKGs. There are two ways to you can solve this. Either assign the **vmware-system-privileged** PSP to all service accounts, or create dedicated service accounts and PSPs for every service.

For security purposes I prefere to keep the PSPs enabled and have dedicated service accounts for the service without root priviliges.

## Preparation

This is all tested on a TKGs 1.22 [v1.22.9---vmware.1-tkg.1.cc71bc8] cluster. I created an example cluster [shared-tools-cluster.yaml](shared-tools-cluster.yaml) to verify the installation.

I highly recommand to install the [carvel tools](https://carvel.dev/):

- [kapp](https://carvel.dev/kapp/docs/v0.52.0/install/)
  
- [ytt](https://carvel.dev/ytt/docs/v0.42.0/install/)
  
- [imgpkg](https://carvel.dev/imgpkg/docs/v0.31.0/install/)
  
- [kbld](https://carvel.dev/kbld/docs/v0.34.0/install/)

- [tanzu-cli](https://github.com/vmware-tanzu/community-edition/releases/tag/v0.12.1)

## TMC Installation

As an alternative to the proposed method here you can install the 1.5.4 packages via TMC (Tanzu Mission Controll). The [Catalog feature](https://docs.vmware.com/en/VMware-Tanzu-Mission-Control/services/tanzumc-using/GUID-EF35646D-8762-41F1-95E5-D2F35ED71BA1.html) even handel the installation of *kapp-controller* as well as *secretgen-controller*.

Just be aware as the Tanzu Packages don't support [PSPs](https://kubernetes.io/docs/concepts/security/pod-security-policy/) out of the box you need to disable them via a [security policies](https://docs.vmware.com/en/VMware-Tanzu-Mission-Control/services/tanzumc-using/GUID-939955FC-17EF-4A84-B686-CAF0BBE018D4.html) assignement.

If you have TMC *Advanced* you can create a custome policy to meet your needs. If you have TMC *Standard* you need to select the **Baseline** policy and **Disable native pod security policies**.

You can skip the rest of the instructions and use the TMC documentation linked above as well as the [Tanzu Packages 1.5.4 documentation](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/1.5/vmware-tanzu-kubernetes-grid-15/GUID-packages-index.html) to set the paremters for the available packages.

## Setup kapp-controller and secretgen-controller

- Create the psp for the kapp-controller and secretgen-controller

```bash
kubectl apply -f tanzu-system-kapp-ctrl-restricted.yaml
kubectl apply -f tanzu-system-secretgen-ctrl-restricted.yaml
```

- Role out the controllers

```bash
kubectl apply -f kapp-controller.yaml
kubectl apply -f secretgen-controller.yaml
```

- Check the status

```bash
kubectl get pods -n tkg-system
```

- Add the Tanzu Package Repository 1.5.4 to the global repo list

```bash
kubectl apply -f tanzu-standard-repo.yaml
```

- List tanzu repos and make sure its *Reconcile succeeded*

```bash
kubectl get packagerepositories -n tanzu-package-repo-global
```

- List available packages in Tanzu Standard

```bash
kubectl get packages -n tanzu-package-repo-global
```

- Create package service account and namespace

```bash
kubectl apply -f packages-ns-sa.yaml
```

## Cert-manager

```bash
kubectl apply -f cert-manager/cert-manager-package-install.yaml
```

## Contour

The envoy installation will fail with the *vmware-system-restricted* psp so we need to create a dedicated one for envoy. Please have a look at the **[config.yaml](contour/config.yaml)** and update it to your needs. Once done base64 encode it an update the secret in the [contour-package-install.yaml](contour/contour-package-install.yaml)

```bash
kubectl apply -f contour/contour-psp.yaml
kubectl apply -f contour/contour-package-install.yaml
```

## External DNS

Please prepare the [config file](external-dns/config.yaml) for your DNS provider e.g. AWS Route 53, BIND, Microsoft Azure. The example files can be found in the [Tanzu Packages documentation](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid/1.5/vmware-tanzu-kubernetes-grid-15/GUID-packages-external-dns.html).

Before you base 64 encode your
