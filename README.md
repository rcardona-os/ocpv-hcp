## Prerequisites

- [x] OCP v4.17

- [x] ODF Installed and configure (default block storage class)

- [x] OCP Virtualization installed and configured


## Steps:

#### 0 - Installing Muticluster operator

##### 0.1 - Make sure the OCP cluster has wildcard DNS routes enabled
```bash
$ oc patch ingresscontroller -n openshift-ingress-operator default \
  --type=json -p '[{ "op": "add", "path": "/spec/routeAdmission", "value": {wildcardPolicy: "WildcardsAllowed"}}]'
```

##### 0.2 - The OpenShift Container Platform management cluster has a default storage class
```bash
$ oc patch storageclass ocs-storagecluster-ceph-rbd -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

##### 0.3 - Install *"Multicluster Engine for Kebernetes"* Operator

##### 0.4 - Test if the multicluster engine Operator has at least one managed OCP cluster
```bash
$ oc get managedclusters local-cluster
```


####  1 - installing hcp cli
##### 1.0 - Get the URL to download the hcp binary by running the following command:
```bash
$ oc get ConsoleCLIDownload hcp-cli-download -o json | jq -r ".spec"
```

##### 1.2 - Download the hcp binary by running the following command:
```bash
$ wget <hcp_cli_download_url> 
Replace hcp_cli_download_url with the URL that you obtained from the previous step.
```
e.i.

```bash
$ wget https://hcp-cli-download-multicluster-engine.apps.ov.sandbox2941.opentlc.com/darwin/amd64/hcp.tar.gz \
  --no-check-certificate
```

##### 1.3 - Unpack the downloaded archive by running the following command:
```bash
$ tar xvzf hcp.tar.gz
```

##### 1.4 - Make the hcp binary file executable by running the following command:
```bash
$ chmod +x hcp
```

##### 1.5 - Move the hcp binary file to a directory in your path by running the following command:
```bash
$ sudo mv hcp /usr/local/bin/.
```

Note: If you download the CLI on a Mac computer, you might see a warning about the hcp binary file. You need to adjust your security settings to allow the binary file to be run.

##### 1.6 - Verification (platform: aws, kubevirt, etc)
```bash
$ hcp create cluster <platform> --help 
```

#### 2 - provisioning a hosted cluster
##### 2.0 - provisioning hosted cluster
```bash
$ oc new-project hcp
```

```bash
$ hcp create cluster kubevirt \
  --name guest-cluster \
  --namespace hpc \
  --node-pool-replicas 3 \
  --pull-secret pull-secret.json \
  --memory 10 \
  --cores 8 \
  --release-image quay.io/openshift-release-dev/ocp-release:4.17.0-multi \
  --etcd-storage-class=ocs-storagecluster-ceph-rbd
```

##### 2.1 - To check the status of the hosted cluster
```bash
$ oc get hostedclusters -n hostedproject
```

#### 3 - Accessing the hosted cluster

##### 3.0 - Generating the kubeconfig file
```bash
$ hcp create kubeconfig \
  --namespace <HOSTED_CLUSTER_NAMESPACE> \
  --name <HOSTED_CLUSTER_NAME> > <HOSTED_CLUSTER_NAME>.kubeconfig
``` 

```bash
$ hcp create kubeconfig \
  --namespace hcp \
  --name guest-cluster > guest-cluster.kubeconfig
```

```bash
$ oc --kubeconfig guest-cluster.kubeconfig get nodes
```