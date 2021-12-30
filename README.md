## Starting up a X-node cluster on Cloudlab
1. When starting a new experiment on Cloudlab, select the **small-lan** profile
2. In the profile parameterization page, 
        - Set **Number of Nodes** as **X**
        - Set OS image as **Ubuntu 18.04**
        - Set physical node type as **c220g2**
        - Please check **Temp Filesystem Max Space**
        - Keep **Temporary Filesystem Mount Point** as default (**/mydata**)
<!-- 3. We use `node-0` as master node. `node-1` to `node-10` are used as worker node. -->

## Extend the disk
On the master node and worker nodes, run
```bash
sudo chown -R $(id -u):$(id -g) <mount point(to be used as extra storage)>
cd <mount point>
git clone https://github.com/ShixiongQi/UNIVERSE.git
cd <mount point>/UNIVERSE
git checkout mu-share
```
Then run `export MYMOUNT=<mount point>` with the added storage mount point name

- if your **Temporary Filesystem Mount Point** is as default (**/mydata**), please run
```
sudo chown -R $(id -u):$(id -g) /mydata
cd /mydata
git clone https://github.com/ShixiongQi/UNIVERSE.git
cd /mydata/UNIVERSE
git checkout mu-share
export MYMOUNT=/mydata
```

## Deploy Kubernetes Cluster
1. Run `./100-docker_install.sh` without *sudo* on both *master* node and *worker* node
2. Run `source ~/.bashrc`
3. On *master* node, run `./200-k8s_insatll.sh master <master node IP address>`
4. On *worker* node, run `./200-k8s_install.sh slave` and then use the `kubeadm join ...` command obtained at the end of the previous step run in the master node to join the k8s cluster. Run the `kubeadm join` command with *sudo*

```
# For single node deployment
kubectl taint nodes --all node-role.kubernetes.io/master-

# install byobu, htop, ab, perf
sudo apt install -y byobu htop apache2-utils
sudo apt-get install -y linux-tools-common linux-tools-generic linux-tools-`uname -r`

echo 'source <(kubectl completion bash)' >>~/.bashrc
```

## Clone the Kubernetes and Knative repository
```
./300-git_clone.sh
```

## Install ko
```
./400-ko-install.sh
```

## Install DPDK dependencies
```
./dpdk-dependencies.sh
```

## Build Knative from source
```
export DOCKER_USER=shixiongqi
echo "export KO_DOCKER_REPO='docker.io/$DOCKER_USER'" >> ~/.bashrc
source ~/.bashrc

sudo docker login

cd /mydata/go/src/knative.dev/serving/

ko apply -Rf config/
```

## Replace the default kubelet (not required at this moment)
0. Check golang version (>=1.15.X)
```
go version
```
1. Run `./300-git_clone.sh` to clone the Kubernetes repos.
2. Compiling the customized kubelet
```
cd kubernetes/
make WHAT=cmd/kubelet KUBE_BUILD_PLATFORMS=linux/amd64
```
3. Backup default kubelet
```
sudo cp /usr/bin/kubelet /usr/bin/backup_kubelet 
```
4. Terminate default kubelet and copy custimized kubelet to `/usr/bin/`
```
sudo kill -9 $(pgrep kubelet) && sudo cp _output/bin/kubelet /usr/bin/kubelet
```
