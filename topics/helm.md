# Helm

## Description

Helm helps you manage Kubernetes applications — Helm Charts help you define, install, and upgrade even the most complex Kubernetes application.

## Directory

- [Helm](#helm)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Installation](#installation)
    - [Description](#description-1)
    - [References](#references-1)
    - [Steps](#steps)
  - [Helm Chart Repository](#helm-chart-repository)
    - [Description](#description-2)
    - [References](#references-2)
    - [Adding Chart Repository](#adding-chart-repository)
    - [Update Chart Repository](#update-chart-repository)
    - [Remove Chart Repository](#remove-chart-repository)
  - [Helm Charts](#helm-charts)
    - [Description](#description-3)
    - [References](#references-3)
    - [Get Helm Charts](#get-helm-charts)
    - [Install or Upgrade a Helm Chart](#install-or-upgrade-a-helm-chart)
    - [Uninstall a Helm Release](#uninstall-a-helm-release)
  - [Helm Values](#helm-values)
    - [Description](#description-4)
    - [References](#references-4)
    - [Get Helm Chart Values](#get-helm-chart-values)
    - [Get Helm Release Values](#get-helm-release-values)
    - [Update Helm Values](#update-helm-values)

## References

- [Helm](https://helm.sh)

---

## Installation

### Description

This details how to install Helm on your system.

### References

- [Installing Helm](https://helm.sh/docs/intro/install)

### Steps

1. Helm should be available in your software repository by default - [install](package-manager.md#install-software) the `helm` package using your package manager (i.e. `apt`).

2. **Alternatively**, you can install Helm through source:

   - Clone the Helm repository to your home directory:

      ```sh
      git clone https://github.com/helm/helm.git ~/.helm
      ```

   - Enter the local Helm repository (i.e. `~/.helm`):

      ```sh
      cd ~/.helm
      ```

   - Build and install the Helm binary:

      ```sh
      make
      ```

---

## Helm Chart Repository

### Description

This details how to add, update, or remove a Helm chart repository on your system and query available Helm charts to install.

### References

- [Initialize a Helm Chart Repository](https://helm.sh/docs/intro/quickstart/#initialize-a-helm-chart-repository)
- [Helm Example Repository](https://github.com/helm/examples)

### Adding Chart Repository

This details how to add a Helm chart repository to your system:

1. Use the following command to add a public Helm chart repository to your system:

    ```sh
    helm repo add <repo> <repo-source>
    ```

    Replace `<repo>` and `<repo-source>` with the (arbitrary) name of the chart repository (i.e. `examples`) and the source of the repository (i.e. `https://helm.github.io/examples`) respectively. For example:

    ```sh
    helm repo add examples https://helm.github.io/examples
    ```

2. **Alternatively**, if the Helm chart repository is private, you can use the following command to add it to your system:

    ```sh
    helm repo add <repo> <repo-source> --username <repo-user> --password <repo-password>
    ```

    Replace `<repo-user>` and `<repo-password>` with the right user credentials which has access to the Helm chart repository.

3. [Update the chart repository](#update-chart-repository) to ensure it is up-to-date.

### Update Chart Repository

This details how to update a Helm chart repository on your system:

1. Use the following command to update a Helm chart repository on your system:

    ```sh
    helm repo update <repo>
    ```

    Replace `<repo>` with the name of the chart repository (i.e. `examples`). For example:

    ```sh
    helm repo update examples
    ```

2. **Alternatively**, remove the chart repository name from the same command to update all Helm chart repositories on your system:

    ```sh
    helm repo update
    ```

### Remove Chart Repository

This details how to remove a Helm chart repository from your system:

1. Use the following command to remove a Helm chart repository from your system:

    ```sh
    helm repo remove <repo>
    ```

    Replace `<repo>` with the name of the chart repository (i.e. `examples`). For example:

    ```sh
    helm repo remove examples
    ```

2. **(Optional)** To remove more than one chart repository, simply add their names each separated by a space:

    ```sh
    helm repo remove <repo1> <repo2>
    ```

3. Verify the Helm chart repository has been removed by listing available Helm chart repositories on your system:

    ```sh
    helm repo list
    ```

---

## Helm Charts

### Description

This details how to search or deploy a Helm chart.

### References

- [Helm Search Repo](https://helm.sh/docs/helm/helm_search_repo)
- [Helm Upgrade](https://helm.sh/docs/helm/helm_upgrade)

### Get Helm Charts

This details how to show available Helm charts from a Helm chart repository:

1. Use the following command to show available Helm charts from a Helm chart repository on your system:

    ```sh
    helm search repo <repo>
    ```

    Replace `<repo>` with the name of the chart repository (i.e. `examples`). For example:

    ```sh
    helm search repo examples
    ```

2. **Alternatively**, remove the chart repository name from the same command to show available Helm charts from all Helm chart repositories on your system:

    ```sh
    helm search repo
    ```

### Install or Upgrade a Helm Chart

> [!NOTE]  
> This section of the guide assumes that you have [added a Helm chart repository](#adding-chart-repository) to your system.

This details how to install a Helm chart or upgrade an existing Helm release:

1. [Prepare the values file](#update-helm-values) (i.e. `values.yaml`) for the Helm chart or release you wish to install or upgrade.

2. Use the following command to install or upgrade a Helm chart:

    ```sh
    helm upgrade --install --wait --kube-context <cluster> --namespace <namespace> <release> <chart-repo>/<chart> --values values.yaml
    ```

    Replace `<cluster>`, `<namespace>`, `<release>`, `<chart-repo>`, and `<chart>` with the name of the cluster (i.e. `my-cluster`), namespace (i.e. `default`), release (i.e. `hello-world-app`), chart repository (i.e. `examples`), and chart itself (i.e. `hello-world`) respectively. For example:

    ```sh
    helm upgrade --install --wait --kube-context my-cluster --namespace default hello-world-app examples/hello-world --values values.yaml
    ```

3. **(Optional)** If you wish to deploy the Helm chart to a specific namespace you have not created, simply include the following flag to the same command:

    ```sh
    --create-namespace
    ```

4. **(Optional)** If you wish to deploy the Helm chart of a specific version, simply include the following flag to the same command:

    ```sh
    --version <chart-version>
    ```

    Replace `<chart-version>` with the version number (i.e. `0.1.0`) of the Helm chart to deploy.

### Uninstall a Helm Release

> [!CAUTION]  
> Uninstalling a Helm release will irreversibly delete all the resources associated with it, including any persistent data.

This details how to remove or uninstall a Helm release you have deployed:

1. Uninstall the Helm release from the cluster by using the following command:

    ```sh
    helm uninstall --wait --kube-context <cluster> --namespace <namespace> <release>
    ```

    Replace the `<cluster>`, `<namespace>`, and `<release>` placeholders with the same values you had used to [deploy](#install-or-upgrade-a-helm-chart) it. For example:

    ```sh
    helm uninstall --wait --kube-context my-cluster --namespace default hello-world-app
    ```

2. Verify that the Helm release has been removed by listing all Helm releases on the namespace:

    ```sh
    helm ls --kube-context <cluster> --namespace <namespace> | grep "<release>"
    ```

    If this command returns nothing, the Helm release has been removed successfully.

---

## Helm Values

### Description

This details how to get or update Helm values from a Helm chart or deployment (release).

### References

- [Helm Show Values](https://helm.sh/docs/helm/helm_show_values)
- [Helm Get Values](https://helm.sh/docs/helm/helm_get_values)

### Get Helm Chart Values

This details how to get the default Helm values from a Helm chart:

1. Use the following command to get the default values of a Helm chart:

    ```sh
    helm show values <chart-repo>/<chart>
    ```

    Replace `<chart-repo>` and `<chart>` with the name of the chart repository (i.e. `examples`) and chart itself (i.e. `hello-world`) respectively. For example:

    ```sh
    helm show values examples/hello-world
    ```

    If you need the values of a specific chart version, simply include the version number (i.e. `0.1.0`) to the command like so:

    ```sh
    helm show values <chart-repo>/<chart> --version <chart-version>
    ```

2. **(Optional)** If you wish to save the values to a file, simply send it to the desired location (i.e. `values.yaml`):

    ```sh
    helm show values <chart-repo>/<chart> > values.yaml
    ```

### Get Helm Release Values

This details how to get the values of a Helm release:

1. Use the following command to get the values of a Helm release:

    ```sh
    helm --kube-context <cluster> --namespace <namespace> get values <release>
    ```

    Replace `<cluster>`, `<namespace>`, and `<release>` with the name of the cluster (i.e. `my-cluster`), namespace (i.e. `default`), and release (i.e. `hello-world-app`) respectively. For example:

    ```sh
    helm --kube-context my-cluster --namespace default get values hello-world-app
    ```

2. **(Optional)** If you wish to save the values to a file, simply send it to the desired location (i.e. `values.yaml`):

    ```sh
    helm --kube-context <cluster> --namespace <namespace> get values <release> > values.yaml
    ```

### Update Helm Values

This details how to update a Helm values file before installing or upgrading a release:

1. If you are installing a Helm chart for the first time, [get the Helm chart values](#get-helm-chart-values) (latest or of a specific version) as a `values.yaml` file.

    **Alternatively**, if you are upgrading an existing Helm release, [get its Helm release values](#get-helm-release-values) as a `values.yaml` file.

2. Update the Helm `values.yaml` file with the intended configurations for your deployment:

    ```sh
    nano values.yaml
    ```

    Refer to the official documentations when possible and pay extra attention to the descriptions and sample values provided.

3. Save changes made to the values file. [Install or upgrade](#install-or-upgrade-a-helm-chart) the intended Helm chart or release using the file to apply it.
