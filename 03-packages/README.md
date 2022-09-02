# Install Tanzu Packages 1.5.4

## Setup kapp-controller and secretgen-controller

* Create the psp for the kapp-controller and secretgen-controller

```bash
kubectl apply -f tanzu-system-kapp-ctrl-restricted.yaml
kubectl apply -f tanzu-system-secretgen-ctrl-restricted.yaml
```

* Role out the controllers

```bash
kubectl apply -f kapp-controller.yaml
kubectl apply -f secretgen-controller.yaml
```

* Check the status

```bash
kubectl get pods -n tkg-system
```

* Add the Tanzu Package Repository 1.5.4 to the global repo list

```bash
kubectl apply -f tanzu-standard-repo.yaml
```

* List tanzu repos and make sure its *Reconcile succeeded*

```bash
kubectl get packagerepositories -n tanzu-package-repo-global
```

* List available packages in Tanzu Standard

```bash
kubectl get packages -n tanzu-package-repo-global
```

* Create package service account and namespace

```bash
kubectl apply -f packages-ns-sa.yaml
```

* Install Cert-manager 1.5.3

```bash
kubectl apply -f cert-manager/cert-manager-package-install.yaml
```

