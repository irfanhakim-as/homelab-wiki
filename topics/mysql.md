# MySQL

## Description

MySQL is an open-source relational database management system (RDBMS). Its name is a combination of "My", the name of co-founder Michael Widenius's daughter My, and "SQL", the acronym for Structured Query Language.

## Directory

- [MySQL](#mysql)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [MySQL Server](#mysql-server)
    - [Description](#description-1)
    - [References](#references-1)
    - [Deploying MySQL Server](#deploying-mysql-server)
  - [MySQL Database](#mysql-database)
    - [Description](#description-2)
    - [References](#references-2)
    - [Deploying MySQL Database](#deploying-mysql-database)

## References

- [MySQL](https://www.mysql.com)
- [Wikipedia](https://en.wikipedia.org/wiki/MySQL)

---

## MySQL Server

### Description

This details how to deploy a MySQL server to a Kubernetes cluster.

### References

- [Bitnami package for MySQL](https://github.com/bitnami/charts/tree/main/bitnami/mysql)

### Deploying MySQL Server

1. Ensure Helm is [installed](helm.md#installation) on your system.

2. Add the Bitnami [Helm chart repository](helm.md#adding-chart-repository) to your system:

   - Repository name: `bitnami`
   - Repository source: `https://charts.bitnami.com/bitnami`

3. Get the [Helm values file](helm.md#get-helm-chart-values) as a `values.yaml` file for the following chart:

   - Repository: `bitnami`
   - Chart: `mysql`
   - Version: `12.2.0` (include the flag that sets the chart version)

4. [Prepare the values file](helm.md#update-helm-values) with the following configuration considerations:

   - `global.defaultStorageClass`: Set the value to `longhorn`

   - `architecture`: Set the value to `replication` (or leave as default i.e. `standalone`)

   - `auth.rootPassword`: Set the value to a secure password

   - `auth.replicationPassword`: Set the value to a secure password (you may use the same password as `auth.rootPassword`)

   - Either `primary.resourcesPreset` or `primary.resources`: Update the resource limits according to your cluster resources

   - Either `secondary.resourcesPreset` or `secondary.resources`: Update the resource limits according to your cluster resources

   - **(Optional)** `primary.persistence.size` and `secondary.persistence.size`: Update the values according to your storage requirement

   - **(Optional)** `primary.service.type`: Set the value to `NodePort` if you need access to the service from outside the cluster

   - **(Optional)** `primary.service.nodePorts.mysql`: Set the value to a specific node port number (i.e. `30000`)

   - **(Optional)** `secondary.replicaCount`: Set the value to `0` to disable the secondary replica

5. [Deploy the Helm release](helm.md#install-or-upgrade-a-helm-chart) using the values file you had prepared with the following recommended options:

   - Namespace: `databases` (include the flag that creates the namespace if it does not exist)
   - Release: `mysql-server-1`
   - Repository: `bitnami`
   - Chart: `mysql`
   - Version: `12.2.0` (include the flag that sets the chart version)

6. Verify that the MySQL server service is up and running:

    ```sh
    kubectl --context <cluster> --namespace <namespace> get svc
    ```

    Replace `<cluster>` and `<namespace>` with the name of the cluster (i.e. `my-cluster`) and namespace (i.e. `databases`) respectively. For example:

    ```sh
    kubectl --context my-cluster --namespace databases get svc
    ```

    Sample output:

    ```
      NAME                                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
      mysql-server-1-primary               ClusterIP   10.43.987.576   <none>        3306/TCP         15s
      mysql-server-1-primary-headless      ClusterIP   None            <none>        3306/TCP         15s
      mysql-server-1-secondary             ClusterIP   10.43.997.521   <none>        3306/TCP         15s
      mysql-server-1-secondary-headless    ClusterIP   None            <none>        3306/TCP         15s
    ```

---

## MySQL Database

> [!NOTE]  
> This part of the guide assumes that you have [deployed a MySQL server](#deploying-mysql-server) to your Kubernetes cluster.

### Description

This details how to deploy a user-database pair on a MySQL server.

### References

- [MariaDB-Agent](https://github.com/irfanhakim-as/charts/tree/master/mika/mariadb-agent)

### Deploying MySQL Database

1. Ensure Helm is [installed](helm.md#installation) on your system.

2. Add the Mika [Helm chart repository](helm.md#adding-chart-repository) to your system:

   - Repository name: `mika`
   - Repository source: `https://irfanhakim-as.github.io/charts`

3. Get the [Helm values file](helm.md#get-helm-chart-values) as a `values.yaml` file for the following chart:

   - Repository: `mika`
   - Chart: `mariadb-agent`

4. [Prepare the values file](helm.md#update-helm-values) with the following configuration considerations:

   - `image.mariadb.repository`: Set the value to `bitnami/mysql`

      ```yaml
      image:
        mariadb:
          repository: "bitnami/mysql"
      ```

   - `image.mariadb.tag`: Set the value to `8.4.3-debian-12-r4`

      ```yaml
      image:
        mariadb:
          tag: "8.4.3-debian-12-r4"
      ```

   - `mariadb.client`: Set the value to `mysql`

      ```yaml
      mariadb:
        client: "mysql"
      ```

   - `mariadb.host`: Set the value to the fully qualified domain name (FQDN) of the primary MariaDB server

      ```yaml
      mariadb:
        host: "<mysql-server-service>.<namespace>.svc.cluster.local"
      ```

      For example, if the name of the primary MySQL server service is `mysql-server-1-primary` and the namespace where the server was deployed to is `databases`:

      ```yaml
      mariadb:
        host: "mysql-server-1-primary.databases.svc.cluster.local"
      ```

   - `mariadb.root.password`: Set the value to the root password of the MySQL server (i.e. `rootpassword`)

      ```yaml
      mariadb:
        root:
          password: "rootpassword"
      ```

   - `mariadb.databases`: Populate the list with values of the database-user pair(s) you wish to deploy to the MySQL server

      ```yaml
      mariadb:
        databases:
          - name: "mydatabase"
            user: "myuser"
            password: "myuserpassword"
            create: true
            drop: false
            custom: false
            custom_command: ""
      ```

5. [Deploy the Helm release](helm.md#install-or-upgrade-a-helm-chart) using the values file you had prepared with the following recommended options:

   - Namespace: `agents` (include the flag that creates the namespace if it does not exist)
   - Release: The name of the database (i.e. `mydatabase`)
   - Repository: `mika`
   - Chart: `mariadb-agent`

6. After the database-user pair has been created, you can supply the database values to any service that require them. For example:

   - Type: `mysql`
   - Host: `mysql-server-1-primary.databases.svc.cluster.local`
   - Port: MySQL's default port number (i.e. `3306`)
   - Name: `mydatabase`
   - User: `myuser`
   - Password: `myuserpassword`
