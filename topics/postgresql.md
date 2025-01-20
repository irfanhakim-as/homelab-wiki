# PostgreSQL

## Description

PostgreSQL also known as Postgres, is a free and open-source relational database management system (RDBMS) emphasising extensibility and SQL compliance. PostgreSQL features transactions with atomicity, consistency, isolation, durability (ACID) properties, automatically updatable views, materialised views, triggers, foreign keys, and stored procedures.

## Directory

- [PostgreSQL](#postgresql)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [PostgreSQL Server](#postgresql-server)
    - [Description](#description-1)
    - [References](#references-1)
    - [Deploying PostgreSQL Server](#deploying-postgresql-server)
  - [PostgreSQL Database](#postgresql-database)
    - [Description](#description-2)
    - [References](#references-2)
    - [Deploying PostgreSQL Database](#deploying-postgresql-database)
    - [Deleting PostgreSQL Database](#deleting-postgresql-database)

## References

- [PostgreSQL](https://www.postgresql.org)
- [Wikipedia](https://en.wikipedia.org/wiki/PostgreSQL)

---

## PostgreSQL Server

### Description

This details how to deploy a PostgreSQL server to a Kubernetes cluster.

### References

- [Bitnami package for PostgreSQL](https://github.com/bitnami/charts/tree/main/bitnami/postgresql)

### Deploying PostgreSQL Server

1. Ensure Helm is [installed](helm.md#installation) on your system.

2. Add the Bitnami [Helm chart repository](helm.md#adding-chart-repository) to your system:

   - Repository name: `bitnami`
   - Repository source: `https://charts.bitnami.com/bitnami`

3. Get the [Helm values file](helm.md#get-helm-chart-values) as a `values.yaml` file for the following chart:

   - Repository: `bitnami`
   - Chart: `postgresql`
   - Version: `15.5.1` (include the flag that sets the chart version)

4. [Prepare the values file](helm.md#update-helm-values) with the following configuration considerations:

   - `global.storageClass`: Set the value to `longhorn`

   - `architecture`: Set the value to `replication` (or leave as default i.e. `standalone`)

   - `auth.postgresPassword`: Set the value to a secure password

   - `auth.replicationPassword`: Set the value to a secure password (you may use the same password as `auth.rootPassword`)

   - Either `primary.resourcesPreset` or `primary.resources`: Update the resource limits according to your cluster resources

   - Either `readReplicas.resourcesPreset` or `readReplicas.resources`: Update the resource limits according to your cluster resources

   - **(Optional)** `primary.persistence.size` and `readReplicas.persistence.size`: Update the values according to your storage requirement

   - **(Optional)** `primary.service.type`: Set the value to `NodePort` if you need access to the service from outside the cluster

   - **(Optional)** `primary.service.nodePorts.postgresql`: Set the value to a specific node port number (i.e. `30000`) or leave it as random

   - **(Optional)** `readReplicas.replicaCount`: Set the value to `0` to disable the secondary replica

5. [Deploy the Helm release](helm.md#install-or-upgrade-a-helm-chart) using the values file you had prepared with the following recommended options:

   - Namespace: `databases` (include the flag that creates the namespace if it does not exist)
   - Release: `postgresql-server-1`
   - Repository: `bitnami`
   - Chart: `postgresql`
   - Version: `15.5.1` (include the flag that sets the chart version)

6. Verify that the PostgreSQL server service is up and running:

    ```sh
    kubectl --context <cluster> --namespace <namespace> get svc
    ```

    Replace `<cluster>` and `<namespace>` with the name of the cluster (i.e. `my-cluster`) and namespace (i.e. `databases`) respectively. For example:

    ```sh
    kubectl --context my-cluster --namespace databases get svc
    ```

    Sample output:

    ```
      NAME                                         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
      postgresql-server-1-postgresql-primary       ClusterIP   10.43.987.576   <none>        5432/TCP         15s
      postgresql-server-1-postgresql-primary-hl    ClusterIP   None            <none>        5432/TCP         15s
      postgresql-server-1-postgresql-read          ClusterIP   10.43.997.521   <none>        5432/TCP         15s
      postgresql-server-1-postgresql-read-hl       ClusterIP   None            <none>        5432/TCP         15s
    ```

7. **(Optional)** Keep the Helm release values file (i.e. `values.yaml`) for future use (i.e. for updates).

---

## PostgreSQL Database

> [!NOTE]  
> This part of the guide assumes that you have [deployed a PostgreSQL server](#deploying-postgresql-server) to your Kubernetes cluster.

### Description

This details how to deploy or delete a user-database pair on a PostgreSQL server.

### References

- [Postgres-Agent](https://github.com/irfanhakim-as/charts/tree/master/mika/postgres-agent)

### Deploying PostgreSQL Database

This details how to deploy a user-database pair on a PostgreSQL server:

1. Ensure Helm is [installed](helm.md#installation) on your system.

2. Add the Mika [Helm chart repository](helm.md#adding-chart-repository) to your system:

   - Repository name: `mika`
   - Repository source: `https://irfanhakim-as.github.io/charts`

3. Get the [Helm values file](helm.md#get-helm-chart-values) as a `values.yaml` file for the following chart:

   - Repository: `mika`
   - Chart: `postgres-agent`

4. [Prepare the values file](helm.md#update-helm-values) with the following configuration considerations:

   - `postgres.host`: Set the value to the fully qualified domain name (FQDN) of the primary PostgreSQL server

      ```yaml
      postgres:
        host: "<postgresql-server-service>.<namespace>.svc.cluster.local"
      ```

      For example, if the name of the primary PostgreSQL server service is `postgresql-server-1-postgresql-primary` and the namespace where the server was deployed to is `databases`:

      ```yaml
      postgres:
        host: "postgresql-server-1-postgresql-primary.databases.svc.cluster.local"
      ```

   - `postgres.root.password`: Set the value to the root password of the PostgreSQL server (i.e. `rootpassword`)

      ```yaml
      postgres:
        root:
          password: "rootpassword"
      ```

   - `postgres.databases`: Populate the list with values of the database-user pair(s) you wish to deploy to the PostgreSQL server

      ```yaml
      postgres:
        databases:
          - name: "mydatabase"
            user: "myuser"
            password: "myuserpassword"
            create: true
            drop: false
            custom: false
            custom_command: ""
      ```

      You can deploy multiple database-user pairs by adding more entries to the list. For example:

      ```yaml
      postgres:
        databases:
          - name: "mydatabase"
            user: "myuser"
            password: "myuserpassword"
            create: true
            drop: false
            custom: false
            custom_command: ""
          - name: "mydatabase2"
            user: "myuser2"
            password: "myuser2password"
            create: true
            drop: false
            custom: false
            custom_command: ""
      ```

5. [Deploy the Helm release](helm.md#install-or-upgrade-a-helm-chart) using the values file you had prepared with the following recommended options:

   - Namespace: `agents` (include the flag that creates the namespace if it does not exist)
   - Release: The name of the database (i.e. `mydatabase`)
   - Repository: `mika`
   - Chart: `postgres-agent`

6. After the database-user pair has been created, you can supply the database values to any service that require them. For example:

   - Type: `postgresql`
   - Host: `postgresql-server-1-postgresql-primary.databases.svc.cluster.local`
   - Port: PostgreSQL's default port number (i.e. `5432`)
   - Name: `mydatabase`
   - User: `myuser`
   - Password: `myuserpassword`

7. **(Optional)** Keep the Helm release values file (i.e. `values.yaml`) for future use (i.e. for updates).

### Deleting PostgreSQL Database

This details how to delete a user-database pair on a PostgreSQL server:

1. Assuming you have [deployed a user-database pair](#deploying-postgresql-database) to your PostgreSQL server, you can delete them by [updating](helm.md#update-helm-values) the same Helm values file (i.e. `values.yaml`) you had used to deploy them:

   - `postgres.databases`: Set the database-user pair(s) you wish to delete to `drop` instead of `create`

      ```yaml
      postgres:
        databases:
          - name: "mydatabase"
            user: "myuser"
            password: "myuserpassword"
            create: false
            drop: true
            custom: false
            custom_command: ""
      ```

2. [Deploy the Helm release](helm.md#install-or-upgrade-a-helm-chart) using the values file you have updated with the same values you had used to deploy them. For example:

   - Namespace: `agents`
   - Release: The name of the database (i.e. `mydatabase`)
   - Repository: `mika`
   - Chart: `postgres-agent`

3. **(Optional)** Keep the Helm release values file (i.e. `values.yaml`) for future use or delete it if you no longer need it.
