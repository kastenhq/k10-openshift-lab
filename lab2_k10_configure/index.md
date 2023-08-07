---
layout: default
title: Lab 2 - Configure Kasten K10
nav_order: 2
---
📖 Part 1. Configure an S3-compatible Object Store for Backup
======================================

*Before we can begin protecting our apps, we need to define a location for Kasten to export backup off of the cluster and primary storage.*

*As part of the lab environment staging, a MinIO Object Storage server has already been deployed to your cluster. This server provides a single bucket, `kasten-bucket`, that you will use as a backup target.*

  > 🚩 ***WARNING***
  >
  > This is a lab environment. In production, exporting your backups to the same storage as what you're trying to protect would really defeat the purpose - don't you think?

1.  Retrive the URL of the Kasten instance by navigating to Networing > Routes
    ![Kasten Route](./assets/images/kasten_route.png)
    *Ensure you have the kasten-io project selected*

2. From the terminal on the bastion host, resolve the URL of your MinIO server and copy the result to your clipboard:

    ```bash
    echo http://$(hostname)
    ```

3. From the ***K10 Dashboard*** tab, select ***Settings***.

    ![settings](./assets/images/settings.png)

4. Under ***Locations***, click the ***+ New Profile*** button.

    ![new profile](./assets/images/new-profile.png)

5. Fill out the following fields:

    | **Field** | **Value** |
    |---|---|
    | ***Profile Name*** | `minio` |
    | ***Storage Provider*** | Select *S3 Compatible*<br>(NOTE: This is a separate option from Amazon S3) |
    | ***S3 Access Key*** | `minioaccess` |
    | ***S3 Secret*** | `miniosecret` |
    | ***Endpoint*** | Paste the URL from Step 1<br>(ex. `http://<ipaddress>:31000`) |
    | ***Region*** | Leave blank |
    | ***Enable Immutable Backups*** | Leave unselected |

    ![location profile](./assets/images/location-profile.png)

6. Click ***Save Profile*** and verify the ***STATUS*** of your Location Profile is ***Valid***.

    ![valid profile](./assets/images/valid-profile.png)

    *This indicates K10 was able to successfully authenticate and access the specified object storage bucket.*

📖 Part 1. Create A Policy
==========================

*Before performing a Kasten install, the "Primer Script" can quickly spot any potential problems with our cluster configuration.*

1. From the 👍 ***K10 Dashboard*** tab, click ***Applications*** to view all applications discovered by K10 on the cluster.

    ![click apps](./assets/images/click-apps.png)

2. Under your `pacman` application, click the ***Create a Policy*** button.

    ![create policy button](./assets/images/create-policy-button.png)

3. By default, the policy will generate hourly snapshots of the application with a standard retention policy. Leave these defaults.

4. Toggle ***Enable Backups via Snapshot Exports*** to the ON position and select `minio` as the ***Export Location Profile***.

    ![enable exports](./assets/images/enable-exports.png)

5. Under ***Select Applications*** observe that we are explicitly protecting all resources in the `pacman` namespace.

    *A single Kasten policy can also protect multiple namespaces, and can even do so dynamically via Kubernetes labels*.

6. Leave all other defaults and click ***Create Policy***.

    ![create policy](./assets/images/create-policy.png)

    ```bash
    helm repo add kasten https://charts.kasten.io/
    helm repo update
    ```

7. You will be returned to the ***Policies*** page. On your new `pacman-backup` Policy, click the ***YAML*** button to view the Kubernetes manifest.

    ![policy yaml](./assets/images/policy-yaml.png)

    *As native Kubernetes resources, K10 policies, profiles, and even running backups or restores can be easily implemented via `kubectl` or API. Exposing the YAML through the UI makes it easy for administrators to copy and modify existing examples of K10 resources or actions.*

8. Click ***Cancel*** to close the YAML window.

🏁 Part 3. Takeaways
====================

- Local snapshots are not backups
- Configuring off-cluster storage for backup is a good idea
- Configuring an S3-compatible bucket in Kasten K10 can be done in a few clicks
- Because Kasten is K8s-native, all actions, policies, resources can be implemented using kubectl commands or API

Click ***Next*** to proceed to the next exercise.