# Contrail-Helm-TP-Integration
Contrail Helm in Partners Kubernetes Infrastructure. Expects that the environment already has Kubernetes installed and should not have any other CNI active on the cluster (FLannel, Calico etc. or any other CNI).

## Install Helm

* Download and Installing Helm  
	```bash
    export HELM_VERSION=v2.5.1
    export TMP_DIR=$(mktemp -d)
    curl -sSL https://storage.googleapis.com/kubernetes-helm/helm-${HELM_VERSION}-linux-amd64.tar.gz | tar -zxv --strip-components=1 -C ${TMP_DIR}
    sudo mv ${TMP_DIR}/helm /usr/local/bin/helm
    rm -rf ${TMP_DIR}
    ```
* Download contrail related charts and manifests
	```bash
    git clone https://github.com/Juniper/contrail-docker.git -b R4.0
    ```
* Installing helm's tiller pods with the right tiller version
	```bash
    cd contrail-docker/kubernetes/manifests
    kubectl create -f tiller.yaml
    kubectl patch ds/tiller-ds --type json   -p='[{"op": "replace", "path": "/spec/updateStrategy/type", "value": "RollingUpdate"}]' -n kube-system && kubectl set image ds/tiller-ds tiller=gcr.io/kubernetes-helm/tiller:${HELM_VERSION} -n kube-system
    ```
* Initialize the helm client using the below command
	```bash
    helm init --client-only
    ```

* Edit the values.yaml file, please refer the input options can be found below for more details on various variables which could be defined
   ```bash
    cd ../helm
    vi contrail/values.yaml
   ```
* Install contrail charts using the below command
    ```bash
    helm install --name <deployment name> <path to chart>
    ```

### Clone the required files for installtion:

git clone https://github.com/gokulpch/Contrail-Helm-TP-Integration.git

### Get the required Contrail_Docker_Images:

Load the docker images locally on the host using "docker load < image.tgz>"

REPOSITORY                                             TAG                 IMAGE ID            CREATED             SIZE
contrail-controller-ubuntu16.04                        4.0.1.0-41          d94521c02aff        8 days ago          1.623 GB
contrail-analyticsdb-ubuntu16.04                       4.0.1.0-41          19cea60507f8        8 days ago          1.027 GB
contrail-analytics-ubuntu16.04                         4.0.1.0-41          bb7f84108939        8 days ago          682.3 MB
contrail-agent-ubuntu16.04                             4.0.1.0-41          8a8fec285b50        8 days ago          763.2 MB
contrail-kube-manager-ubuntu16.04                      4.0.1.0-41          5e9960d101ec        8 days ago          615.3 MB
contrail-kubernetes-agent-ubuntu16.04                  4.0.1.0-41          80c452358237        8 days ago          302.9 MB

## Input to the values file (/contrail-docker/kubernetes/helm/contrail/values.yaml)

* Provide the image details in the images section (be sure to have the proper tags of the images)

```
  images:
  controller: "contrail-controller-ubuntu16.04:4.0.1.0-41"
  analyticsdb: "contrail-analyticsdb-ubuntu16.04:4.0.1.0-41"
  analytics: "contrail-analytics-ubuntu16.04:4.0.1.0-41"
  kubemanager: "contrail-kube-manager-ubuntu16.04:4.0.1.0-41"
  vrouterAgent: "contrail-agent-ubuntu16.04:4.0.1.0-41"
  kubernetesAgent: "contrail-kubernetes-agent-ubuntu16.04:4.0.1.0-41"
  imagePullPolicy: "IfNotPresent"
```

* Change the IP details in the conf section for controller, analytics and analyticsdb (these all can be the same if user is installin a all-in-one cluster)

```
conf:
  # global section configuration needed by all containers
  global:
    controller:
      # nodes - (MANDATORY) list of controller node IP
      nodes:
      - 10.87.1.53
      # virtualIp - (OPTIONAL) Virtual IP handled by load balancer in case of multi-node controllers
      # virtualIp:
      # enableControlService - (OPTIONAL) enable control service or not, by default it is enabled
      enableControlService: "true"

    analyticsdb:
      # nodes - (MANDATORY) list of analyticsdb node IP
      nodes:
        - 10.87.1.53
      # cassandraUser - (OPTIONAL) Cassandra username to access analyticsdb
      cassandraUser: ''
      # cassandraPassword - (OPTIONAL) Cassandra password to access analyticsdb
      cassandraPassword: ''

    analytics:
      # nodes -  (MANDATORY) list of analyticsdb node IP
      nodes:
        - 10.87.1.53
      # virtualIp - (OPTIONAL) Virtual IP handled by load balancer in case of multi-node analytics
      # virtualIp:
```

* All the parameters have explanation, parameters can be customized to suit the environmernt

### Input variables which could be defined while provisioning contrail for k8s (Optional)

| Input Varaible - Helm charts| Input Varaible - Single yaml file | default | Sample Values  | Description |
| -------------- | :----------------: | :-----: | :------------: | :-------------------------: |
| conf.global.cloudOrchestrator  | cloud_orchestrator |  - | kubernetes | Orchestration for which contrail is being provisioned |
| conf.global.ssl.sandesh | sandesh_ssl_enable | False | True/False | Enabling ssl for sandesh connection |
| conf.global.ssl.introspect | introspect_ssl_enable | False | True/False | Enabling ssl for introspect connection |
| conf.global.config.enableConfigService | enable_config_service | True | True/False | Enabling config service |
| conf.global.controller.enableControlService | enable_control_service | True | True/False | Enabling control service |
| conf.global.webui.enableWebuiService | enable_webui_service | True | True/False | Enableing webui service |
| conf.global.config.nodes | config_nodes | - | 10.84.13.7,10.84.13.8 | Comma separated list of config node IP address |
| conf.global.controller.nodes | controller_nodes | - | 10.84.13.7,10.84.13.8 | Comma separated list of controller node IP address |
| conf.global.analytics.nodes | analytics_nodes | - | 10.84.13.7,10.84.13.8 | Comma separated list of analytics node IP address |
| conf.global.analyticsdb.nodes | analyticsdb_nodes | - | 10.84.13.7,10.84.13.8 | Comma separated list of analyticsdb node IP address |
| conf.agent.compileVrouterModule | compile_vrouter_module | - | True/False | Falg to compile vrouter module |
| conf.agent.ctrlDataNetwork | ctrl_data_network | - | 192.168.10.0/24 | Control data network |
| conf.kubernetes.apiServer | api_server | - | 10.84.13.7 | IP address of kube-apiserver |
| conf.kubernetes.podSubnets | pod_subnets | 10.32.0.0/12 | 10.32.0.0/12 | CIDR for pod network |
| conf.kubernetes.svcSubnets | service_subnets | 10.96.0.0/12 | 10.96.0.0/12 | CIDR for service subnet |
| conf.kubernetesVNC.publicFipPool | public_fip_pool | - | {'domain': 'domain-name', 'project': 'project-name', 'network': 'network-name', 'name': 'fip-pool-name' } | FQDN name of FIP pool |

### Label the node where the contrail controller components are installed

helm install --name contrail helm/contrail

### Install Contrail Helm Charts

helm install --name contrail helm/contrail


```
helm ls
NAME            REVISION        UPDATED                         STATUS          CHART                   NAMESPACE
contrail        1               Tue Sep 12 02:37:27 2017        DEPLOYED        Contrail-4.0.1.0        default 
```

```
helm status contrail
LAST DEPLOYED: Tue Sep 12 02:37:27 2017
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME                DATA  AGE
contrailctl-config  7     7d

==> v1beta1/DaemonSet
NAME                    DESIRED  CURRENT  READY  UP-TO-DATE  AVAILABLE  NODE-SELECTOR  AGE
contrail-controller     1        1        1      1           1          <none>         7d
contrail-analyticsdb    1        1        1      1           1          <none>         7d
contrail-vrouter-agent  1        1        1      1           1          <none>         7d
contrail-kube-manager   1        1        1      1           1          <none>         7d
contrail-analytics      1        1        1      1           1          <none>         7d


NOTES:
Thank you for installing Contrail.

Your release is named contrail.

To learn more about this contrail release, try:

  $ helm status contrail
  $ helm get contrail

```

To delete use: helm delete --purge contrail
