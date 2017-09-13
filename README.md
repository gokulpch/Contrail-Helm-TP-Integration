# Contrail-Helm-TP-Integration
Contrail Helm in Partners Kubernetes Infrastructure


* Download Helm  
	* Use this [link](https://storage.googleapis.com/kubernetes-helm/helm-v2.4.2-linux-amd64.tar.gz) to downlod the helm client
	* Unpack it using the below command
	```bash
    tar -zxvf helm-v2.4.2-linux-amd64.tar.gz
           mv linux-amd64/helm /usr/local/bin/helm
    ```
* Download contrail related charts and manifests
	```bash
    git clone https://github.com/Juniper/contrail-docker.git
    ```
* Installing helm's tiller pods
	```bash
    cd contrail-docker/kubernetes/manifests
    kubectl create -f tiller.yaml
    ```
* Initialize the helm client using the below command
	```bash
    helm init --client-only
    ```
* Create the clusterRoleBinding
	```bash
    kubectl create clusterrolebinding contrail-manager --clusterrole=cluster-admin --serviceaccount=kube-system:default
    ```
* Edit the values.yaml file, please refer the [input to the values file section](#input-to-the-values-file) for more details on various variables defined
   ```bash
    cd ../helm
    vi contrail/values.yaml
   ```
* Install contrail charts using the below command
    ```bash
    helm install --name <deployment name> <path to chart>
    ```

## Input to the values file

| Input variable | Optional/Mandatory | default | Sample
