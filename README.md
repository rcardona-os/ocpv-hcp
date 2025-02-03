## Prerequisites

- [x] OCP v4.17

- [x] ODF Installed and configure (default block storage class)

- [x] OCP Virtualization installed and configured


## Steps:

#### - installing hcp cli
1- Get the URL to download the hcp binary by running the following command:

```bash
$ oc get ConsoleCLIDownload hcp-cli-download -o json | jq -r ".spec"
```

2- Download the hcp binary by running the following command:

```bash
$ wget <hcp_cli_download_url> 
Replace hcp_cli_download_url with the URL that you obtained from the previous step.
```
e.i.

```bash
$ wget https://hcp-cli-download-multicluster-engine.apps.ov.sandbox2941.opentlc.com/darwin/amd64/hcp.tar.gz \
  --no-check-certificate
```

3- Unpack the downloaded archive by running the following command:

```bash
$ tar xvzf hcp.tar.gz
```

4- Make the hcp binary file executable by running the following command:

```bash
$ chmod +x hcp
```

5- Move the hcp binary file to a directory in your path by running the following command:

```bash
$ sudo mv hcp /usr/local/bin/.
```

Note: If you download the CLI on a Mac computer, you might see a warning about the hcp binary file. You need to adjust your security settings to allow the binary file to be run.

6- Verification (platform: aws, kubevirt, etc)

```bash
$ hcp create cluster <platform> --help 
```

#### installing Muticluster operator

1- Make sure the OCP cluster has wildcard DNS routes enabled
```bash
$ oc patch ingresscontroller -n openshift-ingress-operator default \
  --type=json -p '[{ "op": "add", "path": "/spec/routeAdmission", "value": {wildcardPolicy: "WildcardsAllowed"}}]'
```

2- The OpenShift Container Platform management cluster has a default storage class
```bash
$ oc patch storageclass ocs-storagecluster-ceph-rbd -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

3- Make sure the multicluster engine Operator has at least one managed OCP cluster
```bash
$ oc get managedclusters local-cluster
```

#### provisioning hosted cluster
```bash
$ hcp create cluster kubevirt \
  --name hostedcluster \
  --namespace hostedproject
  --node-pool-replicas 0 \
  --pull-secret pull-secret.json \
  --memory 10 \
  --cores 8 \
  --etcd-storage-class=ocs-storagecluster-ceph-rbd
```

#### to check the status of the hosted cluster
```bash
$ oc get --namespace clusters hostedclusters
```