---
layout: default
title: Lab 3 - Backup and Restore a Workload
nav_order: 3
---
📖 Part 1. Peform a backup
======================================

*Before we can begin protecting our apps, we need to define a location for Kasten to export backup off of the cluster and primary storage.*

1. Under Policies, click _run once_ on the policy you created in lab 2, entitled _pacman-backup_

    ![Run Once](./assets/images/policy_run_once.png)

2. When prompted, leave the "Snapshot Expiration (Optional)" field blank and select, _Yes, Continue_

    ![Run Once Continue](./assets/images/run_once_continue.png)

3. Click **< Dashboard** in the upper-left corner to monitor the action on the main dashboard

    ![Running Action](./assets/images/running_action.png)

4. Click on the running action to monitor its detailed status

    ![Action Details](./assets/images/action_details.png)

5. After a few minutes, all steps of the action should run successfully

    ![Action Completed](./assets/images/action_completed.png)


Part 2. Simulated Attack
=====================

1. Return to the Pacman tab in your browser and click "View Highscore List" or if the game is playing, hit the space bar and click **High Score**

    ![Leaderboard](./assets/images/pacman_leaderboard.png)

2. From the bastion host, drop the `pacman` database from MongoDB that holds your highscore:

    ```bash
    export MONGODB_ROOT_PASSWORD=$(oc get secret -n pacman pacman-mongodb -o jsonpath="{.data.mongodb-root-password}" | base64 --decode)
    oc exec -it deploy/pacman-mongodb -n pacman -- mongosh pacman --authenticationDatabase admin -u root -p $MONGODB_ROOT_PASSWORD --eval 'db.dropDatabase();'
    ```

    {: .important }
    > This command is simulating a data compromise event, which could be something as innocent as an administrator
    > accidentally dropping a table or database, or as nefarious as a ransomware attack

2. Return to the Pacman tab in your browser and refresh the page. Click __High Score__.
   __!!OH NO YOUR HIGH SCORE IS GONE!!__

    ![no high scores](./assets/images/no_highscores.png)

Part 3. Recover Our Score
==========================

1. No fear, let's restore our backup.  Click on the Kasten K10 tab in your browser and return to the main dashboard by clicking on **< Dashboard**

2. Click __Compliant__ in the Applications modal.

    ![Compliant](./assets/images/compliant.png)

3. Click Restore on the Pacman application to restore from backup

    ![Restore](./assets/images/restore.png)

4. Click the most recent backup.

    {: .note }
    > There are two options from which to restore. The blue box is the local cluster backup, whereas the green box with the title "Exported" is the exported backup which is stored on our S3
    > object storage.
    >
    > In the event of an accidental deletion, restoring from local cluster backup is sufficient, but if we were facing the result of a ransomware attack
    > we would likely want to restore from the S3 bucket.  For the purposes of this lab, we'll just the local cluster backup since restore time will be slightly faster


    Click Today, #:## in the _blue box_ to restore from the local cluster snapshot

    ![Restore Today](./assets/images/restore_today.png)

5. Scroll down and click **Deselect All Artifacts** then click the tick box next to the _pacman-mongodb_ item under the _Snapshot (1)_ section

    ![Restore Volume](./assets/images/volume_only_restore.png)

6. Click Restore.

7. Return to the Dashboard by clicking on the **< Dashboard** link in the upper left corner

8. Click on the running Restore Action to monitor the action
   
9. After a minute or two all phases should complete successfully

    ![Restore Completed](./assets/images/restore_completed.png)

10. Return to the pacman tab and refresh the tab.  Click **High Score**. Our high score is back!
   **REJOICE!**

    ![Leaderboard](./assets/images/pacman_leaderboard.png)

Part 4. Takeaways
====================

- Kasten K10 automatically interrogates and detects namespaces on the cluster
- Backup jobs are configured on a per-namespace basis and can be configured quickly
- We can easily monitor actions via the Kasten UI
- We have granular control on how we restore from backup, including whether from on-cluster or exported storage
- We have granular control over which components we wish to recover and/or overwrite


<div>
<a style="z-index:999999;padding:7px 15px;border-width:2px;border-style:solid;border-radius:8px;font-weight:600;font-size:18px;filter:drop-shadow(0px 0px 15px rgba(26, 19, 72, 0.25));font-family:Guardian Sans, Arial, sans-serif;white-space:nowrap;overflow:hidden;text-overflow:ellipsis;cursor:pointer;background:#4C5BDC;border-color:#FFFFFF;color:#FFFFFF" href="../lab4_k10_blueprints">Continue to LAB 4</a>
</div>