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

3. Click _< Dashboard_ in the upper-left corner to monitor the action on the main dashboard

    ![Running Action](./assets/images/running_action.png)

4. Click on the running action to monitor its detailed status

    ![Action Details](./asset/images/action_details.png)

5. All steps of the action should run successfully

    ![Action Completed](./asset/images/action_completed.png)


Part 2. Scores Gone
=====================

1. Return to the Pacman tab in your browser and click "View Highscore List" or if the game is playing, hit <space bar> and click __High Score__

    ![Leaderboard](./assets/images/pacman_leaderboard.png)

2. From the bastion host, drop the `pacman` database from MongoDB that holds your highscore:

    ```bash
    export MONGODB_ROOT_PASSWORD=$(oc get secret -n pacman pacman-mongodb -o jsonpath="{.data.mongodb-root-password}" | base64 --decode)
    oc exec -it deploy/pacman-mongodb -n pacman -- mongosh pacman --authenticationDatabase admin -u root -p $MONGODB_ROOT_PASSWORD --eval 'db.dropDatabase();'
    ```

2. Return to the Pacman tab in your browser and refresh the page. Click __High Score__ __!!OH NO YOUR HIGH SCORE IS GONE!!__

    ![no high scores](./assets/images/no_highscores.png)

Part 3. Recover Our Score
==========================

1. No fear, let's restore our backup.  Click on the Kasten K10 tab in your browser and on the main dashboard, click __Compliant__ in the Applications modal.

    ![Compliant](./assets/images/compliant.png)

2. Click Restore on the Pacman application to restore from backup

    ![Restore](./assets/images/restore.png)

3. Click the most recent backup.

    { .note }
    There are two options from which to restore. The blue box is the local cluster backup, whereas the green box with the title "Exported" is the exported backup which is stored on our S3 object storage

    Click Today, #:## in the _blue box_ to restore from the local cluster snapshot

    ![Restore Today](./assets/images/restore_today.png)

4. Scroll down and click __Deselect All Artifacts__ then click the tick box next to the _pacman-mongodb_ item under the _Snapshot (1)_ section

    ![Restore Volume](./assets/images/volume_only_restore.png)

   Click Restore.
   
5. Click on the Restore action. All phases should complete successfully

    ![Restore Completed](./assets/images/restore_completed.png)

6. Return to the pacman tab and refresh the tab.  Click High Score. Note that your high score is back.

    ![Leaderboard](./assets/images/pacman_leaderboard.png)

🏁 Part 4. Takeaways
====================

- Kasten K10 automatically interrogates and detects namespaces on the cluster
- Backup jobs are configured on a per-namespace basis and can be configured quickly
- We can easily monitor actions via the Kasten UI
- We have granular control on how we restore from backup, including whether from on-cluster or exported storage
- We have granular control over which components we wish to recover and/or overwrite