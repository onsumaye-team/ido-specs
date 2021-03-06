```text
SPDX-License-Identifier: Apache-2.0
Copyright (c) 2019-2021 Intel Corporation
```
<!-- omit in toc -->
# Harbor Registry Service in Smart Edge Open
- [Deploy Harbor registry](#deploy-harbor-registry)
  - [System Prerequisite](#system-prerequisite)
  - [Ansible Playbooks](#ansible-playbooks)
  - [Projects](#projects)
- [Harbor login](#harbor-login)
- [Harbor registry image push](#harbor-registry-image-push)
- [Harbor registry image pull](#harbor-registry-image-pull)
- [Harbor UI](#harbor-ui)
- [Harbor CLI](#harbor-cli)
  - [CLI - List Project](#cli---list-project)
  - [CLI - List Image Repositories](#cli---list-image-repositories)
  - [CLI - Delete Image](#cli---delete-image)

Harbor registry is an open source cloud native registry which can support images and relevant artifacts with extended functionalities as described in [Harbor](https://goharbor.io/). On the Smart Edge Open environment, Harbor registry service is installed on Control plane Node by Harbor Helm Chart [github](https://github.com/goharbor/harbor-helm/releases/tag/v1.5.1). Harbor registry authentication enabled with self-signed certificates as well as all nodes and control plane will have access to the Harbor registry.

## Deploy Harbor registry

### System Prerequisite
* The available system disk should be reserved at least 20G for Harbor PV/PVC usage. The defaut disk PV/PVC total size is 20G. The values can be configured in the ```roles/harbor_registry/controlplane/defaults/main.yaml```.
* If huge pages enabled, need 1G(hugepage size 1G) or 300M(hugepage size 2M) to be reserved for Harbor usage.
 
### Ansible Playbooks 
Ansible `harbor_registry` roles created in Converged Edge Experience Kits. For deploying a Harbor registry on Kubernetes, control plane roles are enabled in the main `network_edge.yml` playbook file.

```ini
role: harbor_registry/controlplane
role: harbor_registry/node
```

The following steps are processed by converged-edge-experience-kits during the Harbor registry installation on the Smart Edge Open control plane node.

* Download Harbor Helm Charts on the Kubernetes Control plane Node.
* Check whether huge pages is enabled and templates values.yaml file accordingly.
* Create namespace and disk PV for Harbor Services (The default disk PV/PVC total size is 20G. The values can be configured in the `roles/kubernetes/harbor_registry/controlplane/defaults/main.yaml`).
* Install Harbor on the control plane node using the Helm Charts (The CA crt will be generated by Harbor itself). 
* Create the new project - ```intel``` for Smart Edge Open microservices, Kurbernetes enhanced add-on images storage.
* Docker login the Harbor Registry, thus enable pulling, pushing and tag images with the Harbor Registry


On the Smart Edge Open edge nodes, converged-edge-experience-kits will conduct the following steps:
* Get harbor.crt from the Smart Edge Open control plane node and save into the host location
  /etc/docker/certs.d/<Kubernetes_Control_Plane_IP:port>
* Docker login the Harbor Registry, thus enable pulling, pushing and tag images with the Harbor Registry
* After above steps, the Node and Ansible host can access the private Harbor registry.
* The IP address of the Harbor registry will be: "Kubernetes_Control_Plane_IP"
* The port number of the Harbor registry will be: 30003


### Projects 
Two Harbor projects will be created by CEEK as below:
- ```library``` The registry project can be used by edge application developer as default images registries.
- ```intel```   The registry project contains the registries for the Smart Edge Open microservices and relevant kubernetes addon images. Can also be used for Smart Edge Open sample application images.

## Harbor login
For the nodes inside of the Smart Edge Open cluster, converged-edge-experience-kits ansible playbooks automatically login and prepare harbor CA certifications to access Harbor services. 

For the external host outside of the Smart Edge Open cluster, can use following commands to access the Harbor Registry:

```shell
# create directory for harbor's CA crt
mkdir /etc/docker/certs.d/${Kubernetes_Control_Plane_IP}:${port}/

# get EMCO harbor CA.crt
set -o pipefail && echo -n | openssl s_client -showcerts -connect ${Kubernetes_Control_Plane_IP}:${port} 2>/dev/null | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > /etc/docker/certs.d/${Kubernetes_Control_Plane_IP}:${port}/harbor.crt

# docker login harobr registry
docker login ${Kubernetes_Control_Plane_IP}:${port} -uadmin -p${harborAdminPassword}
```
The default access configuration for the Harbor Registry is:
 ```ini
Kubernetes_Control_Plane_IP: 30003(default)
harborAdminPassword: Harbor12345(default)
 ```

## Harbor registry image push
Use the Docker tag to create an alias of the image with the fully qualified path to your Harbor registry after the tag successfully pushes the image to the Harbor registry.

 ```shell
  docker tag nginx:latest {Kubernetes_Control_Plane_IP}:30003/intel/nginx:latest
  docker push {Kubernetes_Control_Plane_IP}:30003/intel/nginx:latest
 ```
Now image the tag with the fully qualified path to your private registry. You can push the image to the registry using the Docker push command.

## Harbor registry image pull
Use the `docker pull` command to pull the image from Harbor registry:

 ```shell
  docker pull {Kubernetes_Control_Plane_IP}:30003/intel/nginx:latest
 ```

## Harbor UI
Open the https://{Kubernetes_Control_Plane_IP}:30003 with login username ```admin``` and password ```Harbor12345```:
![](smartedge-open-cluster-setup-images/harbor_ui.png)

You could see two projects: ```intel``` and ```library``` on the Web UI. For more details about Harbor usage, can refer to [Harbor docs](https://goharbor.io/docs/2.1.0/working-with-projects/).

## Harbor CLI
Apart for Harbor UI, you can also use ```curl``` to check Harbor projects and images. The examples will be shown as below.
```text
In the examples, 10.240.224.172 is IP address of {Kubernetes_Control_Plane_IP}
If there is proxy connection issue with ```curl``` command, can add ```--proxy``` into the command options.
```

### CLI - List Project
Use following example commands to check projects list:
 ```shell
 # curl -X GET "https://10.240.224.172:30003/api/v2.0/projects" -H "accept: application/json" -k --cacert /etc/docker/certs.d/10.240.224.172:30003/harbor.crt -u "admin:Harbor12345 | jq"
 [
  {
    "creation_time": "2020-11-26T08:47:31.626Z",
    "current_user_role_id": 1,
    "current_user_role_ids": [
      1
    ],
    "cve_allowlist": {
      "creation_time": "2020-11-26T08:47:31.628Z",
      "id": 1,
      "items": [],
      "project_id": 2,
      "update_time": "2020-11-26T08:47:31.628Z"
    },
    "metadata": {
      "public": "true"
    },
    "name": "intel",
    "owner_id": 1,
    "owner_name": "admin",
    "project_id": 2,
    "repo_count": 3,
    "update_time": "2020-11-26T08:47:31.626Z"
  },
  {
    "creation_time": "2020-11-26T08:39:13.707Z",
    "current_user_role_id": 1,
    "current_user_role_ids": [
      1
    ],
    "cve_allowlist": {
      "creation_time": "0001-01-01T00:00:00.000Z",
      "items": [],
      "project_id": 1,
      "update_time": "0001-01-01T00:00:00.000Z"
    },
    "metadata": {
      "public": "true"
    },
    "name": "library",
    "owner_id": 1,
    "owner_name": "admin",
    "project_id": 1,
    "update_time": "2020-11-26T08:39:13.707Z"
  }
 ]

 ```

### CLI - List Image Repositories 
Use following example commands to check images repository list of project - ```intel```:
 ```shell
 # curl -X GET "https://10.240.224.172:30003/api/v2.0/projects/intel/repositories" -H "accept: application/json" -k --cacert /etc/docker/certs.d/10.240.224.172:30003/harbor.crt -u "admin:Harbor12345" | jq
 [
  {
    "artifact_count": 1,
    "creation_time": "2020-11-26T08:57:43.690Z",
    "id": 3,
    "name": "intel/sriov-device-plugin",
    "project_id": 2,
    "pull_count": 1,
    "update_time": "2020-11-26T08:57:55.240Z"
  },
  {
    "artifact_count": 1,
    "creation_time": "2020-11-26T08:56:16.565Z",
    "id": 2,
    "name": "intel/sriov-cni",
    "project_id": 2,
    "update_time": "2020-11-26T08:56:16.565Z"
  },
  {
    "artifact_count": 1,
    "creation_time": "2020-11-26T08:49:25.453Z",
    "id": 1,
    "name": "intel/multus",
    "project_id": 2,
    "update_time": "2020-11-26T08:49:25.453Z"
  }
 ]

 ```

### CLI - Delete Image 
Use following example commands to delete the image repository of project - ```intel```, for example:
 ```shell
  # curl -X DELETE "https://10.240.224.172:30003/api/v2.0/projects/intel/repositories/nginx" -H "accept: application/json" -k --cacert /etc/docker/certs.d/10.240.224.172:30003/harbor.crt -u "admin:Harbor12345"
 ```
 
Use following example commands to delete a specific image version:
 ```sh
 # curl -X DELETE "https://10.240.224.172:30003/api/v2.0/projects/intel/repositories/nginx/artifacts/1.14.2" -H "accept: application/json" -k --cacert /etc/docker/certs.d/10.240.224.172:30003/harbor.crt -u "admin:Harbor12345"
 ```
