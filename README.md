# What

This will deploy one or more clusters in the cloud, with optional post-install tasks defined by template.

# Supported platforms

## Container
 * Kubernetes
 * K3s
 * Docker EE
 * Openshift 3.11
 * Openshift 3.11 with CRI-O
 * Openshift 4 (only on AWS at this time)
 * EKS (only makes sense on AWS)

## Cloud
 * AWS
 * GCP
 * Azure
 * Vsphere

## Getting started

1. Install the CLI for your choice of cloud provider and ensure it is configured:
 * AWS: https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html
 * GCP: https://cloud.google.com/sdk/docs/quickstarts
 * Azure: https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest

2. Install and enable Docker.

3. Install bash-completion (optional), eg:
```
$ brew install bash-completion
```
You will need to restart your shell.

4. Run the install script:
```
curl https://raw.githubusercontent.com/andrewh1978/px-deploy/master/install.sh | bash
```
This will:
 * build the Docker image
 * create and populate `$HOME/.px-deploy`
 * provide instructions to configure `.bash_profile`/`.zshrc`

5. Deploy something:
```
px-deploy create --name=myDeployment --template=clusterpair
```
This will provision a VPC and some other objects, and deploy into it from the template.

6. Connect via SSH:
```
px-deploy connect --name myDeployment
```

7. Execute a command:
```
px-deploy connect --name myDeployment "storkctl get clusterpair"
```

8. Tear down the deployment:
```
px-deploy destroy --name myDeployment
```

# NOTES

The deployments can be listed:
```
$ px-deploy list
DEPLOYMENT  CLOUD  REGION         PLATFORM  TEMPLATE  CLUSTERS  NODES  CREATED
foo         aws    eu-west-1      k8s       px               1      3  2020-02-11T16:14:06Z
bar         gcp    europe-north1  gcp       <none>           1      3  2020-02-04T09:50:11Z
```

The templates can be listed:
```
$ px-deploy templates
NAME                         DESCRIPTION
async-dr                     Deploys 2 clusters with Portworx, sets up and configures a cluster pairing,
                             configures an async DR schedule with a loadbalancer in front of the setup.
autopilot                    Deploys a Kubernetes cluster and Portworx with Promethues and Autopilot
                             pre-configured.
backup-restore               Deploys a Kubernetes cluster, Minio S3 storage, Petclinic Application and
                             Backup/Restore config
cockroach                    Deploys a single K8s cluster with Portworx and CockroachDB running
elk                          Deploys an ELK stack of 3 master nodes, 3 data nodes and 2 coordinator nodes, as per
                             https://docs.portworx.com/portworx-install-with-kubernetes/application-install-with-kubernetes/elastic-search-and-kibana/
harbor                       Deploys a single K8s cluster with Portworx and Harbor (https://goharbor.io/)
metro                        Deploys 2 K8s clusters in AWS with a stretched Portworx cluster. It configures
                             Metro, a GUI and Petclinic, ready for a manual failover demo
migration                    Deploys 2 clusters with Portworx, sets up and configures a cluster pairing, and
                             deploys a set of apps and a migration template.
px-backup                    A single Kubernetes cluster with Portworx and PX-Backup via Helm installed
px-fio-example               An example fio benchmark on a gp2 disk and a Portworx volume on a gp2 disk
px-ibm-oem                   A single Openshift 3.11 cluster with Portworx installed with IBM OEM license
px-vs-storageos-fio-example  An example fio benchmark on a gp2 disk, a Portworx volume on a gp2 disk, and a
                             StorageOS volume on a gp2 disk
px-with-gui                  A single K8s cluster with Portworx, Lighthouse and VNC access to access the
                             Lighthouse UI for demo purposes
px                           A single Kubernetes cluster with Portworx installed
storageos                    A single Kubernetes cluster with StorageOS installed
training                     Deploys training clusters
```

Generate a list of IP address, suitable for training:
```
$ px-deploy status --name trainingDeployment
master-1 34.247.219.101 ec2-34-247-219-101.eu-west-1.compute.amazonaws.com
master-2 34.254.155.6 ec2-34-254-155-6.eu-west-1.compute.amazonaws.com
```

The `defaults.yml` file sets a number of deployment variables:
 * `aws_ebs` - a list of EBS volumes to be attached to each worker node. This is a space-separate list of type:size pairs, for example: `"gp2:30 standard:20"` will provision a gp2 volume of 30 GB and a standard volume of 20GB
 * `aws_region` - AWS region
 * `aws_type` - the AWS machine type for each node
 * `cloud` - the cloud on which to deploy (aws or gcp)
 * `clusters` - the number of clusters to deploy
 * `k8s_version` - the version of Kubernetes to deploy
 * `stop_after` - stop the intances after this many hours
 * `post_script` - script to run on each master after deployment, output will go to stdout
 * `quiet` - if "true", hide provisioning output
 * `auto_destroy` - if set to `true`, destroy deployment immediately after deploying (usually used with a `post_script` to output the results of a test or benchmark)
 * `nodes` - the number of worker nodes on each cluster
 * `platform` - can be set to either k8s, k3s, none, dockeree, ocp3, ocp3c (OCPv3 with CRI-O), ocp4 or eks
 * `px_version` - the version of Portworx to install
 * `gcp_disks` - similar to aws_ebs, for example: `"pd-standard:20 pd-ssd:30"`
 * `gcp_region` - GCP region
 * `gcp_type` - the GCP machine type for each node
 * `gcp_zone` - GCP zone
 * `azure_disks` - similar to aws_ebs, for example: `"20 30"`
 * `azure_type` - the Azure machine type for each node
 * `vsphere_host` - vCenter server IP or FQDN
 * `vsphere_datacenter` - vCenter Datacenter
 * `vsphere_compute_resource` - VMware Cluster Name
 * `vsphere_resource_pool` - Cluster Resource Pool to place VMs
 * `vsphere_user` - user with which to provision VMs
 * `vsphere_password` - password
 * `vsphere_template` - full path to CentOS 7 template
 * `vsphere_datastore` - datastore prefix
 * `vsphere_folder` - folder for vSphere VMs
 * `vsphere_disks` - similar to aws_ebs, for example: `"20 30"` (NOTE: these are not provisioned as static block devices, but they are used as clouddrives)
 * `vsphere_network` - vSwitch or dvPortGroup for cluster ex: Team-SE-120
 * `vsphere_memory` - RAM in GB
 * `vsphere_cpu` - number of vCPUs
 * `ocp4_domain` - domain that has been delegated to route53
 * `ocp4_version` - eg `4.3.0`
 * `ocp4_pull_secret` - the pull secret `'{"auths" ... }'`

There are two ways to override these variables. The first is to specify a template with the `--template=...` parameter. For example:
```
$ cat templates/clusterpair.yml
clusters: 2
scripts: ["install-px", "clusterpair"]
```

More on `scripts` below.

The second way to override the defaults is to specify on the command line. See `px-deploy create -h` for a full list. For example, to deploy petclinic into the `foo` deployment:
```
px-deploy create --name=foo --clusters=5 --template=petclinic --nodes=6
```

This example is a mixture of both methods. The template is applied, then the command line parameters are applied, so not only is the template overriding the defaults, but also the parameters are overriding the template.

`scripts` is a list of scripts to be executed on each master node. For example:
```
$ cat ~/.px-deploy/scripts/petclinic
# Install petclinic on each cluster
kubectl apply -f /assets/petclinic.yml
```

These variables are passed to the script:
 * `$nodes`
 * `$clusters`
 * `$px_version`
 * `$k8s_version`

All files in `~/.px-deploy/assets` will be copied to `/assets` on the master nodes. They are then available to be used by the script, as above.

In addition to these, there are some more variables available:
 * `$cluster` - cluster number

There is also an optional cluster object. At the moment, all it can be used for is defining cluster-specific scripts. These will be executed after the scripts above. For example:
```
cluster:
- id: 1
  scripts: ["script-1", "script-2"]
- id: 2
  scripts: ["script-3", "script-4"]
```

`post_script` is a script that will be run on each master node after all of the scripts have completed, and the output will go to stdout. The default is to display the external IP address of master-1, but it could be used to show benchmark outputs, for example.

Last, environment variables can be define in templates or defaults.yml, and these are also available to scripts:
```
$ cat templates/metro.yml
...
env:
  licenses: "XXXX-XXXX-XXXX-XXXX-XXXX-XXXX-XXXX-XXXX XXXX-XXXX-XXXX-XXXX-XXXX-XXXX-XXXX-XXXX"
```
Enviroment variables can also be defined on the command line:
```
px-deploy create -n foo -t clusterpair -e install_apps=true,foo=bar
```

The `install-px` script looks for an environment variable called `cloud_drive`. If it exists, it will deploy Portworx using a clouddrive rather than looking for all attached devices. Note that this is a requirement for Openshift 4. For example:
```
px-deploy create -n foo -t px -e cloud_drive=type%3Dgp2%2Csize%3D150
px-deploy create -n bar -t px --platform ocp4 -e cloud_drive=type%3Dgp2%2Csize%3D150,operator=true
```

# Notes for vSphere

Before you can start deploying in vSphere, a template must be built. The command
```
$ px-deploy vsphere-init
```
will read the vsphere variables from `defaults.yml` and provision a template at the path defined in `vsphere_template`.

# Notes for OCP4 + AWS

A "master" node will be provisioned for each cluster. This is not really a master node - it is just where `openshift-install` is run. The root user will have a kubeconfig, so it can be treated as a master node for the purposes of the scripts used in the templates.

A subdomain must be delegated to Route53 on the same AWS account, so you will need to be able to create records for your own domain:

1. Login to the AWS console and go to Route53.

2. Click on "Hosted Zones". "Click on Created hosted zone".

3. Enter the subdomain, eg openshift.example.com and click "Created hosted zone". It will give you 4 authoritative nameservers for the subdomain. 

4. Login to your DNS provider.

5. Create an NS record for each of the nameservers for the subdomain, eg:
```
$ host -t ns openshift.example.com
openshift.example.com name server ns-1386.awsdns-45.org.
openshift.example.com name server ns-1845.awsdns-38.co.uk.
openshift.example.com name server ns-282.awsdns-35.com.
openshift.example.com name server ns-730.awsdns-27.net.
```

6. Wait a few minutes for the changes to be reflected. Then validate all is well in Route53:
```
$ host -t soa openshift.example.com
openshift.example.com has SOA record ns-730.awsdns-227.net. awsdns-hostmaster.amazon.com. 1 7200 900 1209600 86400
```

# Bugs

 * The Azure Vagrant plugin will [fail when provisioning VMs in parallel](https://github.com/Azure/vagrant-azure/issues/229), so px-deploy disables parallel provisioning. This is really slow, and if a template uses a script that will not terminate until another VM is up, then it will never finish provisioning.
 * Provisioning multiple deployments in an Azure region at the same time gives DNS errors and fails.
