# GKE Cluster Module

The GKE Cluster module is used to administer the [cluster master](https://cloud.google.com/kubernetes-engine/docs/concepts/cluster-architecture)
for a [Google Kubernetes Engine (GKE) Cluster](https://cloud.google.com/kubernetes-engine/docs/how-to/cluster-admin-overview).

The Module is adapted from [Gruntwork's GKE Module](https://github.com/gruntwork-io/terraform-google-gke)

## Doc generation

Code formatting and documentation for variables and outputs is generated using [pre-commit-terraform hooks](https://github.com/antonbabenko/pre-commit-terraform) which uses [terraform-docs](https://github.com/segmentio/terraform-docs).


And install `terraform-docs` with
```bash
go get github.com/segmentio/terraform-docs
```
or
```bash
brew install terraform-docs.
```

## Contributing

Report issues/questions/feature requests on in the issues section.

Full contributing guidelines are covered [here](CONTRIBUTING.md).

## What is a GKE Cluster?

The GKE Cluster, or "cluster master", runs the Kubernetes control plane
processes including the Kubernetes API server, scheduler, and core resource
controllers.

The master is the unified endpoint for your cluster; it's the "hub" through
which all other components such as nodes interact. Users can interact with the
cluster via Kubernetes API calls, such as by using `kubectl`. The GKE cluster
is responsible for running workloads on nodes, as well as scaling/upgrading
nodes.

## How do I attach worker machines using a GKE node pool?

A "[node](https://kubernetes.io/docs/concepts/architecture/nodes/)" is
a worker machine in Kubernetes; in GKE, nodes are provisioned as
[Google Compute Engine VM instances](https://cloud.google.com/compute/docs/instances/).

[GKE Node Pools](https://cloud.google.com/kubernetes-engine/docs/concepts/node-pools)
are a group of nodes who share the same configuration, defined as a [NodeConfig](https://cloud.google.com/kubernetes-engine/docs/reference/rest/v1/NodeConfig).
Node pools also control the autoscaling of their nodes, and autoscaling
configuration is done inline, alongside the node config definition. A GKE
Cluster can have multiple node pools defined.

Node pools are configured directly with the
[`google_container_node_pool`](https://www.terraform.io/docs/providers/google/r/container_node_pool.html)
Terraform resource by providing a reference to the cluster you configured with
this module as the `cluster` field.

## What VPC network will this cluster use?

You must explicitly specify the network and subnetwork of your GKE cluster using
the `network` and `subnetwork` fields; this module will not implicitly use the
`default` network with an automatically generated subnetwork.

The modules in the [`terraform-google-network`](https://github.com/gruntwork-io/terraform-google-network)
Gruntwork module are a useful tool for configuring your VPC network and
subnetworks in GCP.

## What is a VPC-native cluster?

A VPC-native cluster is a GKE Cluster that uses [alias IP ranges](https://cloud.google.com/vpc/docs/alias-ip), in that
it allocates IP addresses from a block known to GCP. When using an alias range, pod addresses are natively routable
within GCP, and VPC networks can ensure that the IP range the cluster uses is reserved.

While using a secondary IP range is recommended [in order to to separate cluster master and pod IPs](https://github.com/gruntwork-io/terraform-google-network/tree/master/modules/vpc-network#how-is-a-secondary-range-connected-to-an-alias-ip-range),
when using a network in the same project as your GKE cluster you can specify a blank range name to draw alias IPs from your subnetwork's primary IP range. If
using a shared VPC network (a network from another GCP project) using an explicit secondary range is required.

See [considerations for cluster sizing](https://cloud.google.com/kubernetes-engine/docs/how-to/alias-ips#cluster_sizing)
for more information on sizing secondary ranges for your VPC-native cluster.

## What is a private cluster?

In a private cluster, the nodes have internal IP addresses only, which ensures that their workloads are isolated from the public Internet.
Private nodes do not have outbound Internet access, but Private Google Access provides private nodes and their workloads with
limited outbound access to Google Cloud Platform APIs and services over Google's private network.

If you want your cluster nodes to be able to access the Internet, for example pull images from external container registries,
you will have to set up [Cloud NAT](https://cloud.google.com/nat/docs/overview).
See [Example GKE Setup](https://cloud.google.com/nat/docs/gke-example) for further information.

You can create a private cluster by setting `enable_private_nodes` to `true`. Note that with a private cluster, setting
the master CIDR range with `master_ipv4_cidr_block` is also required.

### How do I control access to the cluster master?

In a private cluster, the master has two endpoints:

* **Private endpoint:** This is the internal IP address of the master, behind an internal load balancer in the master's
VPC network. Nodes communicate with the master using the private endpoint. Any VM in your VPC network, and in the same
region as your private cluster, can use the private endpoint.

* **Public endpoint:** This is the external IP address of the master. You can disable access to the public endpoint by setting
`enable_private_endpoint` to `true`.

You can relax the restrictions by authorizing certain address ranges to access the endpoints with the input variable
`master_authorized_networks_config`.

### How do I configure logging and monitoring with Stackdriver for my cluster?

Stackdriver Kubernetes Engine Monitoring is enabled by default using this module. It provides improved support for both
Stackdriver Monitoring and Stackdriver Logging in your cluster, including a GKE-customized Stackdriver Console with
fine-grained breakdown of resources including namespaces and pods. Learn more with the [official documentation](https://cloud.google.com/monitoring/kubernetes-engine/#about-skm)

Although Stackdriver Kubernetes Engine Monitoring is enabled by default, you can use the legacy Stackdriver options by
modifying your configuration. See the [differences between GKE Stackdriver versions](https://cloud.google.com/monitoring/kubernetes-engine/#version)
for the differences between legacy Stackdriver and Stackdriver Kubernetes Engine Monitoring.

#### How do I use Prometheus for monitoring?

Prometheus monitoring for your cluster is ready to go through GCP's Stackdriver Kubernetes Engine Monitoring service. If
you've configured your GKE cluster with Stackdriver Kubernetes Engine Monitoring, you can follow Google's guide to
[using Prometheus](https://cloud.google.com/monitoring/kubernetes-engine/prometheus) to configure your cluster with
Prometheus.

### Private cluster restrictions and limitations

Private clusters have the following restrictions and limitations:

* The size of the RFC 1918 block for the cluster master must be /28.
* The nodes in a private cluster must run Kubernetes version 1.8.14-gke.0 or later.
* You cannot convert an existing, non-private cluster to a private cluster.
* Each private cluster you create uses a unique VPC Network Peering.
* Deleting the VPC peering between the cluster master and the cluster nodes, deleting the firewall rules that allow
ingress traffic from the cluster master to nodes on port 10250, or deleting the default route to the default
Internet gateway, causes a private cluster to stop functioning.

## How do I configure the cluster to use Google Groups for GKE?

If you want to enable Google Groups for use with RBAC, you have to provide a G Suite domain name using input variable `var.gsuite_domain_name`. If a
value is provided, the cluster will be initialised with a security group `gke-security-groups@[yourdomain.com]`.

In G Suite, you will have to:

1. Create a G Suite Google Group in your domain, named gke-security-groups@[yourdomain.com]. The group must be named exactly gke-security-groups.
1. Create groups, if they do not already exist, that represent groups of users or groups who should have different permissions on your clusters.
1. Add these groups (not users) to the membership of gke-security-groups@[yourdomain.com].

After the cluster has been created, you are ready to create Roles, ClusterRoles, RoleBindings, and ClusterRoleBindings
that reference your G Suite Google Groups. Note that you cannot enable this feature on existing clusters.

For more information, see https://cloud.google.com/kubernetes-engine/docs/how-to/role-based-access-control#google-groups-for-gke.


<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Requirements

| Name | Version |
|------|---------|
| terraform | >= 0.12.26 |

## Providers

| Name | Version |
|------|---------|
| google | n/a |
| google-beta | n/a |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| alternative\_default\_service\_account | Alternative Service Account to be used by the Node VMs. If not specified, the default compute Service Account will be used. Provide if the default Service Account is no longer available. | `string` | `null` | no |
| basic\_auth\_password | The password used for basic auth; set both this and `basic_auth_username` to "" to disable basic auth. | `string` | `""` | no |
| basic\_auth\_username | The username used for basic auth; set both this and `basic_auth_password` to "" to disable basic auth. | `string` | `""` | no |
| cluster\_secondary\_range\_name | The name of the secondary range within the subnetwork for the cluster to use | `string` | n/a | yes |
| description | The description of the cluster | `string` | `""` | no |
| disable\_public\_endpoint | Control whether the master's internal IP address is used as the cluster endpoint. If set to 'true', the master can only be accessed from internal IP addresses. | `bool` | `false` | no |
| enable\_client\_certificate\_authentication | Whether to enable authentication by x509 certificates. With ABAC disabled, these certificates are effectively useless. | `bool` | `false` | no |
| enable\_legacy\_abac | Whether to enable legacy Attribute-Based Access Control (ABAC). RBAC has significant security advantages over ABAC. | `bool` | `false` | no |
| enable\_network\_policy | Whether to enable Kubernetes NetworkPolicy on the master, which is required to be enabled to be used on Nodes. | `bool` | `true` | no |
| enable\_private\_nodes | Control whether nodes have internal IP addresses only. If enabled, all nodes are given only RFC 1918 private addresses and communicate with the master via private networking. | `bool` | `false` | no |
| enable\_vertical\_pod\_autoscaling | Whether to enable Vertical Pod Autoscaling | `string` | `false` | no |
| enable\_workload\_identity | Enable Workload Identity on the cluster | `bool` | `false` | no |
| gsuite\_domain\_name | The domain name for use with Google security groups in Kubernetes RBAC. If a value is provided, the cluster will be initialized with security group `gke-security-groups@[yourdomain.com]`. | `string` | `null` | no |
| horizontal\_pod\_autoscaling | Whether to enable the horizontal pod autoscaling addon | `bool` | `true` | no |
| http\_load\_balancing | Whether to enable the http (L7) load balancing addon | `bool` | `true` | no |
| identity\_namespace | Workload Identity Namespace. Default sets project based namespace [project\_id].svc.id.goog | `string` | `null` | no |
| ip\_masq\_link\_local | Whether to masquerade traffic to the link-local prefix (169.254.0.0/16). | `bool` | `false` | no |
| ip\_masq\_resync\_interval | The interval at which the agent attempts to sync its ConfigMap file from the disk. | `string` | `"60s"` | no |
| kubernetes\_version | The Kubernetes version of the masters. If set to 'latest' it will pull latest available version in the selected region. | `string` | `"latest"` | no |
| location | The location (region or zone) to host the cluster in | `string` | n/a | yes |
| logging\_service | The logging service that the cluster should write logs to. Available options include logging.googleapis.com/kubernetes, logging.googleapis.com (legacy), and none | `string` | `"logging.googleapis.com/kubernetes"` | no |
| maintenance\_start\_time | Time window specified for daily maintenance operations in RFC3339 format | `string` | `"05:00"` | no |
| master\_authorized\_networks\_config | The desired configuration options for master authorized networks. Omit the nested cidr\_blocks attribute to disallow external access (except the cluster node IPs, which GKE automatically whitelists)<br>  ### example format ###<br>  master\_authorized\_networks\_config = [{<br>    cidr\_blocks = [{<br>      cidr\_block   = "10.0.0.0/8"<br>      display\_name = "example\_network"<br>    }],<br>  }] | `list(any)` | `[]` | no |
| master\_ipv4\_cidr\_block | The IP range in CIDR notation to use for the hosted master network. This range will be used for assigning internal IP addresses to the master or set of masters, as well as the ILB VIP. This range must not overlap with any other ranges in use within the cluster's network. | `string` | `""` | no |
| monitoring\_service | The monitoring service that the cluster should write metrics to. Automatically send metrics from pods in the cluster to the Stackdriver Monitoring API. VM metrics will be collected by Google Compute Engine regardless of this setting. Available options include monitoring.googleapis.com/kubernetes, monitoring.googleapis.com (legacy), and none | `string` | `"monitoring.googleapis.com/kubernetes"` | no |
| name | The name of the cluster | `string` | n/a | yes |
| network | A reference (self link) to the VPC network to host the cluster in | `string` | n/a | yes |
| network\_project | The project ID of the shared VPC's host (for shared vpc support) | `string` | `""` | no |
| non\_masquerade\_cidrs | List of strings in CIDR notation that specify the IP address ranges that do not use IP masquerading. | `list(string)` | <pre>[<br>  "10.0.0.0/8",<br>  "172.16.0.0/12",<br>  "192.168.0.0/16"<br>]</pre> | no |
| project | The project ID to host the cluster in | `string` | n/a | yes |
| release\_channel | (Optional) The release channel to get upgrades of your GKE clusters from | `string` | `null` | no |
| resource\_labels | The GCE resource labels (a map of key/value pairs) to be applied to the cluster. | `map(any)` | `{}` | no |
| secrets\_encryption\_kms\_key | The Cloud KMS key to use for the encryption of secrets in etcd, e.g: projects/my-project/locations/global/keyRings/my-ring/cryptoKeys/my-key | `string` | `null` | no |
| services\_secondary\_range\_name | The name of the secondary range within the subnetwork for the services to use | `string` | `null` | no |
| stub\_domains | Map of stub domains and their resolvers to forward DNS queries for a certain domain to an external DNS server | `map(string)` | `{}` | no |
| subnetwork | A reference (self link) to the subnetwork to host the cluster in | `string` | n/a | yes |

## Outputs

| Name | Description |
|------|-------------|
| client\_certificate | Public certificate used by clients to authenticate to the cluster endpoint. |
| client\_key | Private key used by clients to authenticate to the cluster endpoint. |
| cluster\_ca\_certificate | The public certificate that is the root of trust for the cluster. |
| endpoint | The IP address of the cluster master. This is private is disable\_public\_access it true |
| master\_version | The Kubernetes master version. |
| name | The name of the cluster master. This output is used for interpolation with node pools, other modules. |
| private\_endpoint | The Private IP address of the cluster master. |
| public\_endpoint | The Public IP address of the cluster master. |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
