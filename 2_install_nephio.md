# 2. Install Nephio

> **IMPORTANT:** Nephio's `test-infra` repository contains an Ansible playbook used for deploying Nephio. Although the playbook includes an option to disable KinD installation, this feature is not functioning correctly as of May 2024. Therefore, we have removed the KinD installation option from the Ansible playbook and maintained a new repository in 5GSEC.

> **IMPORTANT:** Change workdir to home directory and start initializing `init.sh`.

### Download `init.sh` script and Update Environment Variables

```bash
##### -----=[ In mgmt cluster ]=----- ####

# download test-infra repo
git clone https://github.com/5gsec/nephio-test-infra.git test-infra

# set env for nephio
export NEPHIO_USER=$USER
export ANSIBLE_CMD_EXTRA_VAR_LIST="k8s.context=kubernetes-admin@mgmt kind.enabled=false host_min_vcpu=4 host_min_cpu_ram=8"
```

> **IMPORTANT:** Before executing the `init.sh` script, the IP addresses of the installation components, specifically `gitea` and `nephio-webui`, must be updated to reflect those of the test environment.

Before executing the `init.sh` script, the following tasks must be completed.

(1) Fork https://github.com/nephio-project/catalog.git repository to your GitHub.

(2) Clone the forked repository.

```bash
# clone codes from forked repository
git clone https://github.com/[YOUR_GIRHUB_ID]/catalog.git
```

(3) Change the 172.18.0.200 IP and 172.18.0.0/24 IP range strings.

```bash
# set env for edit
GITEA_IP_ADDR=$(hostname -I | awk '{print $1}')
SUBNET_IP_RANGE=[subnet_IP_address_range]
NEPHIO_WEBUI_ADDR=[nephio_webui_address]

# using sed command, change ip address, ip ranges
sed 's/172.18.0.200/$GITEA_IP_ADDR/g' catalog/distros/gcp/nephio-mgmt/nephio-controllers/app/deployment-token-controller.yaml
sed 's/172.18.0.200/$GITEA_IP_ADDR/g' catalog/distros/sandbox/gitea/service-gitea.yaml
sed 's/172.18.0.200\/20/$SUBNET_IP_RANGE/g' catalog/distros/sandbox/metallb-sandbox-config/ipaddresspool.yaml
sed 's/172.18.0.200/$GITEA_IP_ADDR/g' catalog/distros/sandbox/repo-porch.yaml
sed 's/172.18.0.200/$GITEA_IP_ADDR/g' catalog/distros/sandbox/repository/set-values.yaml
sed 's/172.18.0.200/$GITEA_IP_ADDR/g' catalog/nephio/core/nephio-operator/app/controller/deployment-controller.yaml
sed 's/172.18.0.200/$GITEA_IP_ADDR/g' catalog/nephio/core/nephio-operator/app/controller/deployment-token-controller.yaml
sed 's/172.18.0.200/$GITEA_IP_ADDR/g' catalog/nephio/optional/rootsync/rootsync.yaml
sed 's/172.18.0.200/$GITEA_IP_ADDR/g' catalog/nephio/optional/rootsync/set-values.yaml
sed 's/172.18.0.200/$NEPHIO_WEBUI_ADDR/g' catalog/nephio/optional/webui/service.yaml
```

(4) Commit the changes and push them to your forked repository.

```bash
# move to `catalog`
cd catalog

# commit the changes
git add .
git commit -m "changed ip address"

# push the commit to your repository
git push
```

(5) Change the string `https://github.com/nephio-project/catalog.git` to your repository `https://github.com/[YOUR_GIRHUB_ID]/catalog.git`.

```bash
# move to workdir
cd ~

# using sed command, change repository name
sed 's/nephio-project/[Github_user_ID]/g' ./test-infra/e2e/provision/playbooks/roles/bootstrap/defaults/main.yaml
sed 's/nephio-project/[Github_user_ID]/g' ./test-infra/e2e/provision/playbooks/roles/install/defaults/main.yaml
```

### Run `init.sh` script

Then, run `init.sh` to set up Nephio R2 with optional packages.

```bash
sudo -E ./test-infra/e2e/provision/init.sh
```

> **IMPORTANT:** If the `test-infra` directory is renamed, the `init.sh` script will attempt to download the original test-infra from the nephio-project, which will result in the installation of KinD. Therefore, it is important to retain the name `test-infra` for the cloned directory from the `nephio-test-infra` git repository.

> **IMPORTANT:** While running the `init.sh` script, `kpt` will install the `gitea` package, which utilizes the PersistentVolumes created above. Please monitor the `gitea` namespace to ensure that `gitea/gitea-0` and `gitea/gitea-postgress-0` are running properly with the PersistentVolumes. If they are in a `CrashLoopBackOff` status, check that the directories designated for the PersistentVolumes have the correct `file permissions`.

### Monitoring Gitea Pods

To monitor the `gitea` namespace, please open another terminal and use the following command to verify that the Pods are functioning properly.

```bash
watch -n 1 kubectl get pods -n gitea
```

### Setting Proper Permissions

Once the `gitea/gitea-0` and `gitea/gitea-postgresql-0` pods have started in the `gitea` namespace, set the permissions of the local path using:

```bash
# change file permission of the hostpath for the PVs
sudo chmod 777 ~/nephio -R 
```

<br></br>
---
|Before|Next|
|--|--|
|[ Go to Before Page](1_prerequsites.md) | [ Go to Next Page ](3_add_k8s_clusters_to_nephio.md)|
