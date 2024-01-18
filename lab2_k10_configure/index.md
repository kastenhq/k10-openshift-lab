---
layout: default
title: Lab 2 - Configure Kasten K10
nav_order: 2
---

📖 Part 1. Configure cluster storage for K10
======================================

{: .important }
> Prior to configuring backups, we need to ensure our cluster storage is capable of performing snapshots of our persistent data workloads.
> Because our cluster is deployed on top of Amazon EC2, we can leverage EBS storage and it's associated CSI driver to perform volume snapshot operations.
> 
> To do so, we need to ensure our VolumeSnapshotClass object created on the cluster for the ebs.csi.aws.com provisioner is correctly annotated.  We need to add the annotation
> _k10.kasten.io/is-snapshot-class: "true"_

1. On the bastion host, run the following command to patch the csi-aws-vsc VolumeSnapshotClass so K10 can use it to perform snapshot operations

    ```bash
    oc annotate volumesnapshotclass $(oc get volumesnapshotclass \
    -o=jsonpath='{.items[?(@.metadata.annotations.snapshot\.storage\.kubernetes\.io\/is-default-class=="true")].metadata.name}') \
    k10.kasten.io/is-snapshot-class=true
    ```

2. Run the following command to validate the VolumeSnapshotClass has been annotated

    ```bash
    oc get volumesnapshotclasses.snapshot.storage.k8s.io csi-aws-vsc -o yaml
    ```

📖 Part 2. Configure an S3-compatible Object Store for Backup
======================================

*Before we can begin protecting our apps, we need to define a location for Kasten to export backup off of the cluster and primary storage.*

*As part of the lab environment staging, a MinIO Object Storage server has already been deployed to your cluster. This server provides a single bucket, `kasten-bucket`, that you will use as a backup target.*

  {: .warning }
  > This is a lab environment. In production, exporting your backups to the same storage as what you're trying to protect would really defeat the purpose - don't you think?

1.  In the projects drop-down in OpenShift, select kasten-io.
2.  Find the URL of the Kasten instance by navigating to Networing > Routes in the OpenShift console. Click the Location URL
    ![Kasten Route](./assets/images/kasten_route.png)
    *Ensure you have the kasten-io project selected*

3.  When prompted for login details, enter the following:

      | **Username** | **Password** |
      |---|---|
      | kasten | kasten123 |

4. When prompted, enter your email and organization and accept the EULA:
   ![EULA](./assets/images/eula.png)

5. Take a quick tour if you'd like, otherwise just click "No, Thanks"

    ![Take a Tour](./assets/images/take_a_tour.png)

6.  From the ***K10 Dashboard*** using the menu on the left, navigate to ***Profiles*** > ***Location***

    ![settings](./assets/images/profiles.png)

7. Under ***Locations***, click the ***+ New Profile*** button.

8. From the environment configuration details for your cluster, copy the hostname of your bastion host to your clipboard (the part after _lab-user@_). It should look similar to:

    ```bash
    bastion.xyza.sandbox1234.opentlc.com
    ```

9. Fill out the following fields:

    | **Field** | **Value** |
    |---|---|
    | ***Profile Name*** | `minio` |
    | ***Storage Provider*** | Select *S3 Compatible*<br>(NOTE: This is a separate option from Amazon S3) |
    | ***S3 Access Key*** | `minioaccess` |
    | ***S3 Secret*** | `miniosecret` |
    | ***Endpoint*** | Type _http://_ Paste the URL from Step 2, followed by _:9000_<br>(eg. `http://<bastion_host_FQDN>:9000`) |
    | ***Skip certificate chain and hostname verification***| Leave unselected |
    | ***Region*** | us-east-2 |
    | ***Bucket*** | kasten-bucket |
    | ***Enable Immutable Backups*** | Leave unselected |

    ![new profile](./assets/images/new-profile.png)

10. Click **Save Profile**

    {: .note }
    > Occassionally, the Minio server fails to start during the lab's provisioning. If this happens, you will see an error at this step stating **"There was a problem validating the profile"**:
    > ![Problem Validating Profile](./assets/images/problem_validating_profile.png)
    > If this occurs, you can manually start the server on the bastion host by connecting to it via SSH and running the following two commands:
    > ```bash
    > sudo su ec2-user
    > /usr/local/bin/minio server /home/ec2-user/minio --console-address :9090 >/dev/null 2>&1 &
    > ```
    > After doing so, return to the Kasten UI and click **Save Profile** again and Kasten should be able to validate the location profile.

11. Verify the ***STATUS*** of your Location Profile is ***Valid***.

    ![valid profile](./assets/images/valid-profile.png)

    *This indicates K10 was able to successfully authenticate and access the specified object storage bucket.*

📖 Part 3. Create A Policy
==========================

*Before performing a Kasten install, the "Primer Script" can quickly spot any potential problems with our cluster configuration.*

1. Click ***< Dashboard*** to return to the Kasten main dashboard, click ***Applications*** to view all applications discovered by K10 on the cluster.

    ![click apps](./assets/images/click-apps.png)

2. Type `pacman` in the filter search bar, then click the ***Create a Policy*** button.

    ![create policy button](./assets/images/create-policy-button.png)

3. Leave the default name and optionally add a comment.  By default, the policy will generate hourly snapshots of the application with a standard retention policy. Leave these defaults.

4. Scroll down slightly and toggle ***Enable Backups via Snapshot Exports*** to the ON position and select `minio` as the ***Export Location Profile***.

    ![enable exports](./assets/images/enable-exports.png)

5. Under ***Select Applications*** observe that we are explicitly protecting all resources in the `pacman` namespace.

    {: .note }
    A single Kasten policy can also protect multiple namespaces, and can even do so dynamically via Kubernetes labels.

6. Leave all other defaults and click ***Create Policy***.

    ![create policy](./assets/images/create-policy.png)

7. You will be returned to the ***Policies*** page. On your new `pacman-backup` Policy, click the ***YAML*** button to view the Kubernetes manifest.

    ![policy yaml](./assets/images/policy-yaml.png)

    As native Kubernetes resources, K10 policies, profiles, and even running backups or restores can be easily implemented via `oc` or API. Exposing the YAML through the UI makes it easy for administrators to copy and modify existing examples of K10 resources or actions.

8. Click ***Cancel*** to close the YAML window.


🏁 Part 4. Takeaways
====================

- Local snapshots are not backups
- Configuring off-cluster storage for backup is a good idea
- Configuring an S3-compatible bucket in Kasten K10 can be done in a few clicks
- Because Kasten is K8s-native, all actions, policies, resources can be implemented using oc commands or API

<div>
<a style="z-index:999999;padding:7px 15px;border-width:2px;border-style:solid;border-radius:8px;font-weight:600;font-size:18px;filter:drop-shadow(0px 0px 15px rgba(26, 19, 72, 0.25));font-family:Guardian Sans, Arial, sans-serif;white-space:nowrap;overflow:hidden;text-overflow:ellipsis;cursor:pointer;background:#4C5BDC;border-color:#FFFFFF;color:#FFFFFF" href="../lab3_k10_backup">Continue to LAB 3</a>
</div>