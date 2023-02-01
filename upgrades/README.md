### TALM Upgrade and Rollback with Backup

1. Push the du-upgrade.yaml under policygentemplates folder to git and trigger the ZTP pipeline

2. check the created policies by running the following command:

```
oc get policies -A | grep platform-upgrade
```

3. Apply the upgrade-platform-prep ClusterGroupUpgrade CR with the du-upgrade-platform-upgrade-prep policy and the target spoke clusters to the cgu-platform-upgrade-prep.yaml

```
oc apply -f cgu-platform-upgrade-prep.yaml
```

4. Monitor the update process. Upon completion, ensure that the policy is compliant by running the following command

```
oc get policies --all-namespaces
```

5. To check the backup directly on the node, ssh to the target cluster node 

```
oc get po -n openshift-talo-backup
oc logs -f <podname> -n openshift-talo-backup
```


6. Apply the upgrade-platform ClusterGroupUpgrade CR with the du-upgrade-platform-upgrade policy and the target spoke clusters to the cgu-platform-upgrade.yaml. Also, ensure to set backup: true if backup is needed otherwise set to false.

```
oc apply -f cgu-platform-upgrade.yaml
```

7. Start the platform update 

```
oc --namespace=ztp-group patch clustergroupupgrade.ran.openshift.io/cgu-platform-upgrade --patch '{"spec":{"enable":true, "preCaching": false}}' --type=merge
```

8. Check the status of the upgrade in the hub cluster by running the following command

```
oc get cgu -n ztp-group cgu-platform-upgrade -o jsonpath='{.status}' | jq .
```

### Recovery using TALM Backup

1. Delete the previously created ClusterGroupUpgrade custom resource (CR) by running the following command

```
oc delete cgu cgu-platform-upgrade -n ztp-group
```

2. Log in (ssh to SNO node) to the cluster that you want to recover.
3. Check the status of the platform OS deployment by running the following command

```
ostree admin status
```

4. To trigger a rollback of the platform OS deployment, run the following command

```
rpm-ostree rollback -r
```

5. The first phase of the recovery shuts down containers and restores files from the backup partition to the targeted directories. To begin the recovery, run the following command

```
/var/recovery/upgrade-recovery.sh
```

6. When prompted, reboot the cluster by running the following command

```
systemctl reboot
```

7. After the reboot, restart the recovery by running the following command

```
/var/recovery/upgrade-recovery.sh  --resume
```

8. To check the status of the recovery run the following command

```
oc get clusterversion,nodes,clusteroperator
```



