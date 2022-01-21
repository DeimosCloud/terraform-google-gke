# GKE Cluster Module

The GKE Cluster module is used to administer the [cluster master](https://cloud.google.com/kubernetes-engine/docs/concepts/cluster-architecture)
for a [Google Kubernetes Engine (GKE) Cluster](https://cloud.google.com/kubernetes-engine/docs/how-to/cluster-admin-overview).

The Module is adapted from [Gruntwork's GKE Module](https://github.com/gruntwork-io/terraform-google-gke)

## What's in this Repo
- [Modules](./modules)
The modules directory contains the main modules that should be used in your code
  - [gke-node-pool](./modules/gke-node-pool): Module for creating GKE Node Pools

- [Examples](./examples): Example on how to use this module


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
| <a name="requirement_terraform"></a> [terraform](#requirement\_terraform) | >= 0.12.26 |

## Providers

| Name | Version |
|------|---------|
| <a name="provider_google"></a> [google](#provider\_google) | n/a |
| <a name="provider_google-beta"></a> [google-beta](#provider\_google-beta) | n/a |

## Modules

No modules.

## Resources

| Name | Type |
|------|------|
| [google-beta_google_container_cluster.cluster](https://registry.terraform.io/providers/hashicorp/google-beta/latest/docs/resources/google_container_cluster) | resource |
| [google_container_engine_versions.location](https://registry.terraform.io/providers/hashicorp/google/latest/docs/data-sources/container_engine_versions) | data source |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| <a name="input_alternative_default_service_account"></a> [alternative\_default\_service\_account](#input\_alternative\_default\_service\_account) | Alternative Service Account to be used by the Node VMs. If not specified, the default compute Service Account will be used. Provide if the default Service Account is no longer available. | `string` | `null` | no |
| <a name="input_basic_auth_password"></a> [basic\_auth\_password](#input\_basic\_auth\_password) | The password used for basic auth; set both this and `basic_auth_username` to "" to disable basic auth. | `string` | `""` | no |
| <a name="input_basic_auth_username"></a> [basic\_auth\_username](#input\_basic\_auth\_username) | The username used for basic auth; set both this and `basic_auth_password` to "" to disable basic auth. | `string` | `""` | no |
| <a name="input_cluster_secondary_range_name"></a> [cluster\_secondary\_range\_name](#input\_cluster\_secondary\_range\_name) | The name of the secondary range within the subnetwork for the cluster to use | `string` | n/a | yes |
| <a name="input_description"></a> [description](#input\_description) | The description of the cluster | `string` | `""` | no |
| <a name="input_disable_public_endpoint"></a> [disable\_public\_endpoint](#input\_disable\_public\_endpoint) | Control whether the master's internal IP address is used as the cluster endpoint. If set to 'true', the master can only be accessed from internal IP addresses. | `bool` | `false` | no |
| <a name="input_enable_client_certificate_authentication"></a> [enable\_client\_certificate\_authentication](#input\_enable\_client\_certificate\_authentication) | Whether to enable authentication by x509 certificates. With ABAC disabled, these certificates are effectively useless. | `bool` | `false` | no |
| <a name="input_enable_legacy_abac"></a> [enable\_legacy\_abac](#input\_enable\_legacy\_abac) | Whether to enable legacy Attribute-Based Access Control (ABAC). RBAC has significant security advantages over ABAC. | `bool` | `false` | no |
| <a name="input_enable_network_policy"></a> [enable\_network\_policy](#input\_enable\_network\_policy) | Whether to enable Kubernetes NetworkPolicy on the master, which is required to be enabled to be used on Nodes. | `bool` | `true` | no |
| <a name="input_enable_private_nodes"></a> [enable\_private\_nodes](#input\_enable\_private\_nodes) | Control whether nodes have internal IP addresses only. If enabled, all nodes are given only RFC 1918 private addresses and communicate with the master via private networking. | `bool` | `false` | no |
| <a name="input_enable_pubsub_notification"></a> [enable\_pubsub\_notification](#input\_enable\_pubsub\_notification) | Option to enable GKE pub sub notification | `bool` | `null` | no |
| <a name="input_enable_vertical_pod_autoscaling"></a> [enable\_vertical\_pod\_autoscaling](#input\_enable\_vertical\_pod\_autoscaling) | Whether to enable Vertical Pod Autoscaling | `string` | `false` | no |
| <a name="input_enable_workload_identity"></a> [enable\_workload\_identity](#input\_enable\_workload\_identity) | Enable Workload Identity on the cluster | `bool` | `false` | no |
| <a name="input_gsuite_domain_name"></a> [gsuite\_domain\_name](#input\_gsuite\_domain\_name) | The domain name for use with Google security groups in Kubernetes RBAC. If a value is provided, the cluster will be initialized with security group `gke-security-groups@[yourdomain.com]`. | `string` | `null` | no |
| <a name="input_horizontal_pod_autoscaling"></a> [horizontal\_pod\_autoscaling](#input\_horizontal\_pod\_autoscaling) | Whether to enable the horizontal pod autoscaling addon | `bool` | `true` | no |
| <a name="input_http_load_balancing"></a> [http\_load\_balancing](#input\_http\_load\_balancing) | Whether to enable the http (L7) load balancing addon | `bool` | `true` | no |
| <a name="input_identity_namespace"></a> [identity\_namespace](#input\_identity\_namespace) | Workload Identity Namespace. Default sets project based namespace [project\_id].svc.id.goog | `string` | `null` | no |
| <a name="input_ip_masq_link_local"></a> [ip\_masq\_link\_local](#input\_ip\_masq\_link\_local) | Whether to masquerade traffic to the link-local prefix (169.254.0.0/16). | `bool` | `false` | no |
| <a name="input_ip_masq_resync_interval"></a> [ip\_masq\_resync\_interval](#input\_ip\_masq\_resync\_interval) | The interval at which the agent attempts to sync its ConfigMap file from the disk. | `string` | `"60s"` | no |
| <a name="input_kubernetes_version"></a> [kubernetes\_version](#input\_kubernetes\_version) | The Kubernetes version of the masters. If set to 'latest' it will pull latest available version in the selected region. | `string` | `"latest"` | no |
| <a name="input_location"></a> [location](#input\_location) | The location (region or zone) to host the cluster in | `string` | n/a | yes |
| <a name="input_logging_service"></a> [logging\_service](#input\_logging\_service) | The logging service that the cluster should write logs to. Available options include logging.googleapis.com/kubernetes, logging.googleapis.com (legacy), and none | `string` | `"logging.googleapis.com/kubernetes"` | no |
| <a name="input_maintenance_start_time"></a> [maintenance\_start\_time](#input\_maintenance\_start\_time) | Time window specified for daily maintenance operations in RFC3339 format | `string` | `"05:00"` | no |
| <a name="input_master_authorized_networks_config"></a> [master\_authorized\_networks\_config](#input\_master\_authorized\_networks\_config) | The desired configuration options for master authorized networks. Omit the nested cidr\_blocks attribute to disallow external access (except the cluster node IPs, which GKE automatically whitelists)<br>  ### example format ###<br>  master\_authorized\_networks\_config = [{<br>    cidr\_blocks = [{<br>      cidr\_block   = "10.0.0.0/8"<br>      display\_name = "example\_network"<br>    }],<br>  }] | `list(any)` | `[]` | no |
| <a name="input_master_ipv4_cidr_block"></a> [master\_ipv4\_cidr\_block](#input\_master\_ipv4\_cidr\_block) | The IP range in CIDR notation to use for the hosted master network. This range will be used for assigning internal IP addresses to the master or set of masters, as well as the ILB VIP. This range must not overlap with any other ranges in use within the cluster's network. | `string` | `""` | no |
| <a name="input_monitoring_service"></a> [monitoring\_service](#input\_monitoring\_service) | The monitoring service that the cluster should write metrics to. Automatically send metrics from pods in the cluster to the Stackdriver Monitoring API. VM metrics will be collected by Google Compute Engine regardless of this setting. Available options include monitoring.googleapis.com/kubernetes, monitoring.googleapis.com (legacy), and none | `string` | `"monitoring.googleapis.com/kubernetes"` | no |
| <a name="input_name"></a> [name](#input\_name) | The name of the cluster | `string` | n/a | yes |
| <a name="input_network"></a> [network](#input\_network) | A reference (self link) to the VPC network to host the cluster in | `string` | n/a | yes |
| <a name="input_network_project"></a> [network\_project](#input\_network\_project) | The project ID of the shared VPC's host (for shared vpc support) | `string` | `""` | no |
| <a name="input_non_masquerade_cidrs"></a> [non\_masquerade\_cidrs](#input\_non\_masquerade\_cidrs) | List of strings in CIDR notation that specify the IP address ranges that do not use IP masquerading. | `list(string)` | <pre>[<br>  "10.0.0.0/8",<br>  "172.16.0.0/12",<br>  "192.168.0.0/16"<br>]</pre> | no |
| <a name="input_project"></a> [project](#input\_project) | The project ID to host the cluster in | `string` | n/a | yes |
| <a name="input_pubsub_topic"></a> [pubsub\_topic](#input\_pubsub\_topic) | Pub sub topic to publish GKE notifications | `string` | `null` | no |
| <a name="input_release_channel"></a> [release\_channel](#input\_release\_channel) | (Optional) The release channel to get upgrades of your GKE clusters from | `string` | `null` | no |
| <a name="input_resource_labels"></a> [resource\_labels](#input\_resource\_labels) | The GCE resource labels (a map of key/value pairs) to be applied to the cluster. | `map(any)` | `{}` | no |
| <a name="input_secrets_encryption_kms_key"></a> [secrets\_encryption\_kms\_key](#input\_secrets\_encryption\_kms\_key) | The Cloud KMS key to use for the encryption of secrets in etcd, e.g: projects/my-project/locations/global/keyRings/my-ring/cryptoKeys/my-key | `string` | `null` | no |
| <a name="input_services_secondary_range_name"></a> [services\_secondary\_range\_name](#input\_services\_secondary\_range\_name) | The name of the secondary range within the subnetwork for the services to use | `string` | `null` | no |
| <a name="input_stub_domains"></a> [stub\_domains](#input\_stub\_domains) | Map of stub domains and their resolvers to forward DNS queries for a certain domain to an external DNS server | `map(string)` | `{}` | no |
| <a name="input_subnetwork"></a> [subnetwork](#input\_subnetwork) | A reference (self link) to the subnetwork to host the cluster in | `string` | n/a | yes |

## Outputs

| Name | Description |
|------|-------------|
| <a name="output_client_certificate"></a> [client\_certificate](#output\_client\_certificate) | Public certificate used by clients to authenticate to the cluster endpoint. |
| <a name="output_client_key"></a> [client\_key](#output\_client\_key) | Private key used by clients to authenticate to the cluster endpoint. |
| <a name="output_cluster_ca_certificate"></a> [cluster\_ca\_certificate](#output\_cluster\_ca\_certificate) | The public certificate that is the root of trust for the cluster. |
| <a name="output_endpoint"></a> [endpoint](#output\_endpoint) | The IP address of the cluster master. This is private is disable\_public\_access it true |
| <a name="output_master_version"></a> [master\_version](#output\_master\_version) | The Kubernetes master version. |
| <a name="output_name"></a> [name](#output\_name) | The name of the cluster master. This output is used for interpolation with node pools, other modules. |
| <a name="output_private_endpoint"></a> [private\_endpoint](#output\_private\_endpoint) | The Private IP address of the cluster master. |
| <a name="output_public_endpoint"></a> [public\_endpoint](#output\_public\_endpoint) | The Public IP address of the cluster master. |
<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
