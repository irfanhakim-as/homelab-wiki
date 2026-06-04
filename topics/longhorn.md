# Longhorn

## Description

Longhorn is a distributed block storage system for Kubernetes that provides persistent, replicated storage for stateful workloads. It manages volume snapshots, replication, and backup natively within the cluster, operating at the block level through the Longhorn engine.

## Directory

- [Longhorn](#longhorn)
  - [Description](#description)
  - [Directory](#directory)
  - [References](#references)
  - [Backup Setup](#backup-setup)
    - [Description](#description-1)
    - [References](#references-1)
    - [Configure Backup Target](#configure-backup-target)
      - [Configure Backup Target on Helm](#configure-backup-target-on-helm)
    - [Configure Backup Schedule](#configure-backup-schedule)
      - [Configure Backup Schedule on Helm](#configure-backup-schedule-on-helm)
  - [Usage](#usage)
    - [Description](#description-2)
    - [Trigger an On-Demand Backup](#trigger-an-on-demand-backup)
    - [Inspect Backups](#inspect-backups)
    - [Restore from Backup](#restore-from-backup)
    - [Disaster Recovery](#disaster-recovery)

## References

- [Longhorn](https://longhorn.io)

---

## Backup Setup

### Description

This details how to configure Longhorn to back up persistent volumes to an S3-compatible object storage bucket on a schedule.

### References

- [Setting a Backup Target](https://longhorn.io/docs/latest/snapshots-and-backups/backup-and-restore/set-backup-target)
- [Recurring Snapshots and Backups](https://longhorn.io/docs/latest/snapshots-and-backups/scheduling-backups-and-snapshots)

### Configure Backup Target

#### Configure Backup Target on Helm

> [!NOTE]
> This guide assumes that Longhorn was deployed using Helm on your [Kubernetes cluster setup](../courses/kubernetes.md#cluster-setup).

This details how to configure the Longhorn backup target to point at an S3-compatible bucket:

1. Ensure an [S3 bucket has been set up](s3.md#s3-bucket-setup) on the S3-compatible storage server for backups, with the following actions scoped to the backup bucket on the identity:

   - `Read:<bucket-name>/*`
   - `List:<bucket-name>`
   - `Write:<bucket-name>/*`

2. Apply a Kubernetes Secret in `longhorn-system` containing the S3 credentials. This Secret is referenced by the Helm values to authenticate Longhorn with the backup store:

   - Generate a base64-encoded value for each credential field:

     ```sh
     echo -n "<value>" | base64
     ```

   - Create the Secret manifest:

     ```yaml
     apiVersion: v1
     kind: Secret
     metadata:
       name: <secret-name>
       namespace: longhorn-system
     type: Opaque
     data:
       AWS_ACCESS_KEY_ID: <base64-access-key>
       AWS_SECRET_ACCESS_KEY: <base64-secret-key>
       AWS_ENDPOINTS: <base64-endpoint-url>
     ```

     For example:

     ```yaml
     apiVersion: v1
     kind: Secret
     metadata:
       name: longhorn-backup-s3
       namespace: longhorn-system
     type: Opaque
     data:
       AWS_ACCESS_KEY_ID: YzlhMWY0ZDIzZTg3YjYwNWE3YzJlOTRmMTgzZDZhNTIwZDFlN2YzYzliNWEyZDhlNGY2YjNjN2E5ZDJlMWYwYjQ=
       AWS_SECRET_ACCESS_KEY: N2IzZTlmMWM0YThkMmI2ZTVmMGEzYzdkOWUyZjRiOGExYzVkN2UzZjliMmE0YzZkOGUwZjFiM2E1YzdkOWUyZjQ=
       AWS_ENDPOINTS: aHR0cHM6Ly9zMy5leGFtcGxlLmNvbQ==
     ```

     Replace all of the value(s) with your own accordingly:

     - `<secret-name>`: Set this to the name of the Kubernetes Secret, which will be referenced in the Helm values (i.e. `longhorn-backup-s3`)
     - `<base64-access-key>`: Set this to the base64-encoded access key from the S3 identity
     - `<base64-secret-key>`: Set this to the base64-encoded secret key from the S3 identity
     - `<base64-endpoint-url>`: Set this to the base64-encoded S3 API endpoint URL (i.e. `https://s3.example.com`)

   - Apply the manifest to the cluster:

     ```sh
     kubectl apply -f <manifest>.yaml
     ```

     For example:

     ```sh
     kubectl apply -f longhorn-backup.yaml
     ```

3. Update the existing Longhorn Helm values with the following, then deploy the updated release:

   - [Update the Longhorn Helm values](helm.md#update-helm-values) by adding the following:

     ```yaml
     defaultBackupStore:
       backupTarget: "s3://<bucket-name>@<region>/"
       backupTargetCredentialSecret: "<secret-name>"
       pollInterval: "<poll-interval>"

     defaultSettings:
       backupCompressionMethod: "lz4"
       backupConcurrentLimit: "<max-concurrent>"
       failedBackupTTL: "<failed-backup-ttl>"
       recurringJobMaxRetention: "<max-snapshots>"
       autoCleanupRecurringJobBackupSnapshot: "true"
       autoCleanupSnapshotWhenDeleteBackup: "true"
     ```

     For example:

     ```yaml
     defaultBackupStore:
       backupTarget: "s3://longhorn-backups@ap-southeast-5/"
       backupTargetCredentialSecret: "longhorn-backup-s3"
       pollInterval: "300"

     defaultSettings:
       backupCompressionMethod: "lz4"
       backupConcurrentLimit: "3"
       failedBackupTTL: "1440"
       recurringJobMaxRetention: "14"
       autoCleanupRecurringJobBackupSnapshot: "true"
       autoCleanupSnapshotWhenDeleteBackup: "true"
     ```

     Replace all of the value(s) with your own accordingly:

     - `<bucket-name>`: The name of the backup S3 bucket (i.e. `longhorn-backups`)
     - `<region>`: Region of the backup S3 bucket, may be ignored by self-hosted S3-compatible storage (i.e. `ap-southeast-5`)
     - `<secret-name>`: The name of the Kubernetes Secret containing the S3 credentials (i.e. `longhorn-backup-s3`)
     - `<poll-interval>`: How often Longhorn syncs its backup list against the backup store, in seconds (i.e. `300`)
     - `<max-concurrent>`: Maximum number of simultaneous backup workers (i.e. `3`)
     - `<failed-backup-ttl>`: How long failed backup metadata is retained, in minutes (i.e. `1440`, or 24 hours)
     - `<max-snapshots>`: Maximum number of backups retained per volume (i.e. `14`)

   - [Deploy the updated release](helm.md#install-or-upgrade-a-helm-chart) with the following options:

     - Namespace: `longhorn-system`
     - Release: `longhorn`
     - Repository: `https://charts.longhorn.io`
     - Chart: `longhorn`

   Once done, Longhorn Manager picks up the backup target configuration immediately.

4. Verify that the backup target is available:

   ```sh
   kubectl -n longhorn-system get backuptargets
   ```

   Sample output:

   ```
   NAME      URL                                       CREDENTIAL           LASTBACKUPAT   AVAILABLE   LASTSYNCEDAT
   default   s3://longhorn-backups@ap-southeast-5/     longhorn-backup-s3   5m0s           true        2026-01-01T12:00:00Z
   ```

   The `default` BackupTarget should show `AVAILABLE: true`. Otherwise, check the Longhorn Manager logs for credential or backup errors:

   ```sh
   kubectl -n longhorn-system logs daemonset/longhorn-manager --tail=30 | grep -i 'error\|credential\|backup'
   ```

5. [Configure the backup schedule](#configure-backup-schedule-on-helm) to complete the Longhorn backup setup.

### Configure Backup Schedule

#### Configure Backup Schedule on Helm

> [!NOTE]
> This guide assumes that you have [configured the Longhorn backup target](#configure-backup-target-on-helm).

This details how to configure a recurring backup schedule for Longhorn volumes using an opt-out approach, where all volumes are backed up by default:

1. Update the existing Longhorn Helm values with the following, then deploy the updated release:

   - [Update the Longhorn Helm values](helm.md#update-helm-values) by adding the following to configure the default StorageClass so that all new volumes automatically inherit the backup label:

     ```yaml
     persistence:
       recurringJobSelector:
         enable: true
         jobList: '[{"name":"<label-key>","isGroup":true}]'
     ```

     For example:

     ```yaml
     persistence:
       recurringJobSelector:
         enable: true
         jobList: '[{"name":"backup","isGroup":true}]'
     ```

     Replace all of the value(s) with your own accordingly:

     - `<label-key>`: Shared key linking the StorageClass, RecurringJob, and volume labels (i.e. `backup`)

   - [Deploy the updated release](helm.md#install-or-upgrade-a-helm-chart) with the following options:

     - Namespace: `longhorn-system`
     - Release: `longhorn`
     - Repository: `https://charts.longhorn.io`
     - Chart: `longhorn`

2. Create a `RecurringJob` manifest and apply it to the `longhorn-system` namespace. This defines the backup schedule and links the job to volumes via the shared label key:

   - Create the manifest:

     ```yaml
     apiVersion: longhorn.io/v1beta2
     kind: RecurringJob
     metadata:
       name: <job-name>
       namespace: longhorn-system
     spec:
       name: <job-name>
       cron: "<cron-schedule>"
       task: "backup"
       groups:
         - <label-key>
       labels:
         recurring-job-group.longhorn.io/<label-key>: enabled
       retain: <retain-count>
       concurrency: <concurrency>
     ```

     For example:

     ```yaml
     apiVersion: longhorn.io/v1beta2
     kind: RecurringJob
     metadata:
       name: daily-backup
       namespace: longhorn-system
     spec:
       name: daily-backup
       cron: "0 15 * * *"
       task: "backup"
       groups:
         - backup
       labels:
         recurring-job-group.longhorn.io/backup: enabled
       retain: 14
       concurrency: 3
     ```

     Replace all of the value(s) with your own accordingly:

     - `<job-name>`: The name of the RecurringJob resource (i.e. `daily-backup`)
     - `<cron-schedule>`: Cron expression in UTC (i.e. `0 15 * * *`)
     - `<label-key>`: The same label key used to tag Longhorn volumes for backup (i.e. `backup`)
     - `<retain-count>`: Number of backups to keep per volume (i.e. `14`)
     - `<concurrency>`: Maximum number of volumes backed up simultaneously (i.e. `3`)

   - Apply the manifest to the cluster:

     ```sh
     kubectl apply -f <manifest>.yaml
     ```

     For example:

     ```sh
     kubectl apply -f longhorn-backup.yaml
     ```

3. While new volumes automatically inherit the backup label from the StorageClass, existing volumes must be labelled manually. Label all existing Longhorn volumes to include them in the backup schedule:

   ```sh
   kubectl -n longhorn-system get volumes.longhorn.io -o name | xargs -I{} kubectl -n longhorn-system label {} recurring-job-group.longhorn.io/<label-key>=enabled
   ```

   For example:

   ```sh
   kubectl -n longhorn-system get volumes.longhorn.io -o name | xargs -I{} kubectl -n longhorn-system label {} recurring-job-group.longhorn.io/backup=enabled
   ```

4. Verify the backup schedule is correctly set up:

   - Confirm the RecurringJob exists with the expected settings:

     ```sh
     kubectl -n longhorn-system get recurringjob <job-name>
     ```

     For example:

     ```sh
     kubectl -n longhorn-system get recurringjob daily-backup
     ```

     Sample output:

     ```
       NAME           GROUPS       TASK     CRON         RETAIN   CONCURRENCY   AGE   LABELS
       daily-backup   ["backup"]   backup   0 15 * * *   14       3             1d    {"recurring-job-group.longhorn.io/backup":"enabled"}
     ```

   - Confirm that volume labels are applied:

     ```sh
     kubectl -n longhorn-system get volumes.longhorn.io \
       -o custom-columns=VOLUME:.metadata.name,PVC:.status.kubernetesStatus.pvcName,NS:.status.kubernetesStatus.namespace,LABEL:.metadata.labels.'recurring-job-group\.longhorn\.io/<label-key>'
     ```

     For example:

     ```sh
     kubectl -n longhorn-system get volumes.longhorn.io \
       -o custom-columns=VOLUME:.metadata.name,PVC:.status.kubernetesStatus.pvcName,NS:.status.kubernetesStatus.namespace,LABEL:.metadata.labels.'recurring-job-group\.longhorn\.io/backup'
     ```

     Volumes showing `enabled` in the `LABEL` column are covered by the schedule, otherwise they are excluded.

   - **(Optional)** Verify that new volumes inherit the label automatically by creating a test PVC and checking the resulting Longhorn volume:

     ```sh
     echo 'apiVersion: v1
     kind: PersistentVolumeClaim
     metadata:
       name: test-label
       namespace: default
     spec:
       storageClassName: longhorn
       accessModes: ["ReadWriteOnce"]
       resources:
         requests:
           storage: 1Gi' | kubectl apply -f -
     sleep 30
     kubectl -n longhorn-system get volumes.longhorn.io -o jsonpath="{.items[?(@.status.kubernetesStatus.pvcName=='test-label')].metadata.labels}" | python3 -m json.tool
     ```

     Sample output:

     ```json
     {
         "backup-target": "default",
         "longhornvolume": "pvc-a1b2c3d4-e5f6-7890-abcd-ef1234567890",
         "recurring-job-group.longhorn.io/backup": "enabled",
         "setting.longhorn.io/remove-snapshots-during-filesystem-trim": "ignored"
     }
     ```

     Confirm `recurring-job-group.longhorn.io/<label-key>: enabled` appears in the output. Delete the test PVC afterward:

     ```sh
     kubectl -n default delete pvc test-label
     ```

   - Confirm the StorageClass selector is stored as a JSON string:

     ```sh
     kubectl get sc longhorn -o jsonpath='{.parameters.recurringJobSelector}'
     ```

     Sample output:

     ```
     [{"name":"backup","isGroup":true}]
     ```

5. **(Optional)** To exclude a specific PVC from the backup schedule, look up its Longhorn volume name and remove the backup label:

   ```sh
   PVC_NAME="<pvc-name>"
   VOLUME=$(kubectl -n longhorn-system get volumes.longhorn.io -o jsonpath="{.items[?(@.status.kubernetesStatus.pvcName==\"${PVC_NAME}\")].metadata.name}" | grep -m1 "")
   kubectl -n longhorn-system label volume "${VOLUME}" recurring-job-group.longhorn.io/<label-key>-
   kubectl -n longhorn-system get volume "${VOLUME}" -o jsonpath='{.metadata.labels.recurring-job-group\.longhorn\.io/<label-key>}'
   ```

   For example, to exclude the `my-app-data-pvc` PVC:

   ```sh
   PVC_NAME="my-app-data-pvc"
   VOLUME=$(kubectl -n longhorn-system get volumes.longhorn.io -o jsonpath="{.items[?(@.status.kubernetesStatus.pvcName==\"${PVC_NAME}\")].metadata.name}" | grep -m1 "")
   kubectl -n longhorn-system label volume "${VOLUME}" recurring-job-group.longhorn.io/backup-
   kubectl -n longhorn-system get volume "${VOLUME}" -o jsonpath='{.metadata.labels.recurring-job-group\.longhorn\.io/backup}'
   ```

   The trailing `-` on the label key removes that label from the volume. The final command should print nothing if the label no longer exists.

---

## Usage

### Description

This details common operations for managing and monitoring Longhorn volumes and backups.

### Trigger an On-Demand Backup

This details how to trigger an immediate backup outside of the regular schedule.

Longhorn `RecurringJob` CRDs have no manual trigger field. To run a backup immediately, temporarily patch the cron schedule to fire within the next two minutes, then revert it:

1. Save the current cron and patch it to fire in the next two minutes:

   ```sh
   ORIGINAL=$(kubectl -n longhorn-system get recurringjob <job-name> -o jsonpath='{.spec.cron}')
   TARGET=$(date -u -d '+2 minutes' '+%M %H')
   kubectl -n longhorn-system patch recurringjob <job-name> --type=merge -p "{\"spec\":{\"cron\":\"$TARGET * * *\"}}"
   ```

   For example:

   ```sh
   ORIGINAL=$(kubectl -n longhorn-system get recurringjob daily-backup -o jsonpath='{.spec.cron}')
   TARGET=$(date -u -d '+2 minutes' '+%M %H')
   kubectl -n longhorn-system patch recurringjob daily-backup --type=merge -p "{\"spec\":{\"cron\":\"$TARGET * * *\"}}"
   ```

2. Watch for new backups to appear, then interrupt with `Ctrl-C`:

   ```sh
   kubectl -n longhorn-system get backups -w
   ```

3. Revert the cron to the original schedule:

   ```sh
   kubectl -n longhorn-system patch recurringjob <job-name> --type=merge -p "{\"spec\":{\"cron\":\"$ORIGINAL\"}}"
   ```

   For example:

   ```sh
   kubectl -n longhorn-system patch recurringjob daily-backup --type=merge -p "{\"spec\":{\"cron\":\"$ORIGINAL\"}}"
   ```

**Alternatively**, a single volume backup could also be done graphically through the Longhorn UI.

### Inspect Backups

This details how to inspect and verify the state of Longhorn backups.

1. To list all backups and their current phases:

   ```sh
   kubectl -n longhorn-system get backups
   ```

   The backup phases are:

   - `New`: Queued, not started
   - `InProgress`: Actively running
   - `Completed`: Finished successfully
   - `Error`: Did not complete
   - `Unknown`: State cannot be determined

2. To see recent backups sorted by creation time, with volume names and phases:

   ```sh
   kubectl -n longhorn-system get backups -o custom-columns=NAME:.metadata.name,VOLUME:.status.volumeName,PHASE:.status.state,CREATED:.metadata.creationTimestamp --sort-by=.metadata.creationTimestamp | tail -20
   ```

3. To check whether any backups are still in progress:

   ```sh
   kubectl -n longhorn-system get backups -o jsonpath='{range .items[?(@.status.state!="Completed")]}{.metadata.name}{" "}{.status.state}{"\n"}{end}'
   ```

   Empty output means all backups have finished.

4. To check when the recurring job last ran and whether it succeeded:

   ```sh
   kubectl -n longhorn-system describe recurringjob <job-name>
   ```

   For example:

   ```sh
   kubectl -n longhorn-system describe recurringjob daily-backup
   ```

5. To view the raw S3 contents of the backup bucket (i.e. `longhorn-backups`), [list the objects](s3.md#list-objects) using the recursive listing.

### Restore from Backup

This details how to restore a Longhorn volume from a backup.

To restore from backup manually using `kubectl`:

1. [Inspect the available backups](#inspect-backups) and take note of the name of the backup to restore from.

2. Create a PVC manifest that references the backup as a data source:

   ```yaml
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: <restored-pvc-name>
     namespace: <namespace>
   spec:
     storageClassName: longhorn
     dataSource:
       name: <backup-name>
       kind: Backup
       apiGroup: longhorn.io
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: <size>
   ```

   For example:

   ```yaml
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: my-app-data-restored
     namespace: default
   spec:
     storageClassName: longhorn
     dataSource:
       name: pvc-a1b2c3d4-e5f6--abc12345
       kind: Backup
       apiGroup: longhorn.io
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: 10Gi
   ```

   Replace all of the value(s) with your own accordingly:

   - `<restored-pvc-name>`: The name of the new PVC to create (i.e. `my-app-data-restored`)
   - `<namespace>`: The namespace to restore the PVC into (i.e. `default`)
   - `<backup-name>`: The name of the Longhorn backup to restore from (i.e. `pvc-a1b2c3d4-e5f6--abc12345`)
   - `<size>`: The storage size, which should match or exceed the original volume size (i.e. `10Gi`)

3. Apply the manifest to the cluster:

   ```sh
   kubectl apply -f <manifest>.yaml
   ```

   The volume is populated from the backup when the PVC is bound. Attach it to a workload as normal.

4. Monitor the restore progress:

   - Look up the corresponding Longhorn volume name from the restored PVC name:

     ```sh
     kubectl -n longhorn-system get volumes.longhorn.io \
       -o custom-columns=VOLUME:.metadata.name,PVC:.status.kubernetesStatus.pvcName,STATE:.status.state \
       | grep <restored-pvc-name>
     ```

     For example:

     ```sh
     kubectl -n longhorn-system get volumes.longhorn.io \
       -o custom-columns=VOLUME:.metadata.name,PVC:.status.kubernetesStatus.pvcName,STATE:.status.state \
       | grep my-app-data-restored
     ```

     Sample output:

     ```
       pvc-a1b2c3d4-e5f6-7890-abcd-ef1234567890   my-app-data-restored   attached
     ```

     The first column value is the Longhorn volume name (i.e. `pvc-a1b2c3d4-e5f6-7890-abcd-ef1234567890`).

   - Describe the volume for restore progress details:

     ```sh
     kubectl -n longhorn-system describe volume <volume-name>
     ```

     For example:

     ```sh
     kubectl -n longhorn-system describe volume pvc-a1b2c3d4-e5f6-7890-abcd-ef1234567890
     ```

**Alternatively**, the backup restore could also be done graphically through the Longhorn UI.

### Disaster Recovery

This details how to recover Longhorn volumes after a complete cluster loss.

If the Kubernetes cluster is destroyed and needs to be rebuilt:

1. [Provision a new cluster](../courses/kubernetes.md#cluster-setup) with Longhorn installed.

2. [Configure the backup target](#configure-backup-target) as done previously, pointing at the same S3 bucket. Longhorn discovers existing backups automatically.

3. [Confirm backups are visible](#inspect-backups).

4. [Restore each volume from backup](#restore-from-backup) individually.

5. Deploy workloads and verify application state.
