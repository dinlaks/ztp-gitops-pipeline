# Zero Touch Provisioning with OpenShift

This repo contains YAML manifests and instructions for various ZTP related demos.

![Lab Overview](docs/lab-overview.png?raw=true "Lab Overview")


### Pre-req/steps for ZTP Deployment

1. Deploy OCP with 3 node BM cluster.
2. Deploy Local Storage Operator and dont do disk discovery
3. Deploy ODF Operator
   - Configure StorageSystem
   - Select 3 nodes for all disks
   - set default storage class "ocs-storagecluster-ceph-rbd"
4. Deploy ACM using Operator
5. Deploy ZTP hub components
6. Deploy OpenShift GitOps (ArgoCD) using Operator
7. Deploy TALM Operator

8. Deploy argocd patch before kicking off GitOps ZTP Pipeline Deployments for Siteconfig and Policygentemplates
```
oc patch argocd openshift-gitops -n openshift-gitops --type=merge --patch-file argocd-openshift-gitops-patch.json
oc apply -k deployment/
```



### ODF Pre-Install Checks

1. To ensure ODF disks are clean if previous install was used by ceph
Login to each node;

```
$ lsblk
$ sudo sgdisk --zap-all /dev/sdx
```

This will clean/wipe the disks to ensure the ceph is cleared from previous installation

### ODF Post-Install Checks

2. Ceph tools install during Storage(persistent volume) creation in ODF
Ceph tools are useful to check the state of the ceph nodes while installing those components

To enable Ceph Tools
```

oc patch OCSInitialization ocsinit -n openshift-storage --type json --patch '[{ "op": "replace", "path": "/spec/enableCephTools", "value": true }]'
```

To check ceph health/status

```
oc -n openshift-storage exec $(oc -n openshift-storage get pod  -l "app=rook-ceph-tools" -o jsonpath='{.items[0].metadata.name}') -- ceph -s
```

To check ceph osd tree list

```
oc -n openshift-storage exec $(oc -n openshift-storage get pod  -l "app=rook-ceph-tools" -o jsonpath='{.items[0].metadata.name}') -- ceph osd tree
```

### Set Default Storage Class

3. Post to ODF Storage(persistent volume) is created, set the default storage class to use for ZTP

```
oc patch storageclass ocs-storagecluster-ceph-rbd -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

4. check the pv

```
oc get pv
```



### Troubleshooting

#### 1. If bmh stuck in De-provisioning while cleaning cluster

delete the finalizer entry in secrets and save it.

```
oc edit secrets -n <cluster_namespace>
```

delete the finalizer entry in bmh and save it.

```
oc edit bmh -n <cluster_namespace>
```

#### 2. To watch for logs during spoke cluster install

ssh to the target cluster node

```
journalctl -f
```

#### 3. Post to ztp hub configuration deployed

```
oc get AgentServiceConfig agent
oc get po -n openshift-machine-api
oc get AgentServiceConfig agent -o yaml
```

#### 4. Some useful commands to check if cluster install stuck 

```
oc describe infraenvs.agent-install.openshift.io ztpc156 -n ztpc156
oc describe agentclusterinstalls.extensions.hive.openshift.io ztpc156 -n ztpc156
oc get nmstateconfigs.agent-install.openshift.io ztpc156 -n ztpc156 -o yaml
```
