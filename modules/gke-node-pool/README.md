# Terraform Google GKE Node Pool
A terraform module to setup Node Pool

## Usage

```hcl
#------------------------------
# NODE POOL
#------------------------------
module "gke_node_pool" {
  source     = "."
  project_id = var.project_id
  name       = "${local.project}-gke-nodepool-${var.environment}"
  cluster    = module.gke_cluster.name
  location   = var.region

  initial_node_count = "1"
  min_node_count     = "0"
  max_node_count     = "2"

  image_type   = "COS"
  machine_type = var.machine_type
  disk_type    = var.gke_node_pool_disk_type

  oauth_scopes = var.gke_node_pool_oauth_scopes
  tags         = var.gke_node_pool_tags
  labels = {
    all-pools     = "true"
  }

  depends_on = [module.gke_cluster]
}

```

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

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Requirements

No requirements.

## Providers

| Name | Version |
|------|---------|
| <a name="provider_google"></a> [google](#provider\_google) | n/a |

## Modules

No modules.

## Resources

| Name | Type |
|------|------|
| [google_container_node_pool.node_pool](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/container_node_pool) | resource |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| <a name="input_auto_repair"></a> [auto\_repair](#input\_auto\_repair) | (Optional) Whether the nodes will be automatically repaired. | `bool` | `true` | no |
| <a name="input_auto_upgrade"></a> [auto\_upgrade](#input\_auto\_upgrade) | (Optional) Whether the nodes will be automatically upgraded. | `bool` | `true` | no |
| <a name="input_autoscaling"></a> [autoscaling](#input\_autoscaling) | Whether to enable autoscaling. Values of min\_node\_count and max\_node\_count will be used | `bool` | `true` | no |
| <a name="input_cluster"></a> [cluster](#input\_cluster) | Name of GKE cluster | `string` | n/a | yes |
| <a name="input_create_before_destroy"></a> [create\_before\_destroy](#input\_create\_before\_destroy) | Whether to create a new node pool before destroying the old one | `bool` | `true` | no |
| <a name="input_create_timeout"></a> [create\_timeout](#input\_create\_timeout) | n/a | `string` | `"30m"` | no |
| <a name="input_delete_timeout"></a> [delete\_timeout](#input\_delete\_timeout) | n/a | `string` | `"30m"` | no |
| <a name="input_disable_legacy_metadata_endpoints"></a> [disable\_legacy\_metadata\_endpoints](#input\_disable\_legacy\_metadata\_endpoints) | disable-legacy-endpoints metadata set | `bool` | `true` | no |
| <a name="input_disk_size_gb"></a> [disk\_size\_gb](#input\_disk\_size\_gb) | (Optional) Size of the disk attached to each node, specified in GB. The smallest allowed disk size is 10GB | `number` | `30` | no |
| <a name="input_disk_type"></a> [disk\_type](#input\_disk\_type) | (Optional) Type of the disk attached to each node (e.g. 'pd-standard', 'pd-balanced' or 'pd-ssd'). | `string` | `"pd-standard"` | no |
| <a name="input_enable_integrity_monitoring"></a> [enable\_integrity\_monitoring](#input\_enable\_integrity\_monitoring) | (Optional) Defines if the instance has integrity monitoring enabled. | `bool` | `true` | no |
| <a name="input_enable_secure_boot"></a> [enable\_secure\_boot](#input\_enable\_secure\_boot) | Defines if the instance has Secure Boot enabled. | `bool` | `false` | no |
| <a name="input_image_type"></a> [image\_type](#input\_image\_type) | (optional) The type of image to be used | `string` | `"COS"` | no |
| <a name="input_initial_node_count"></a> [initial\_node\_count](#input\_initial\_node\_count) | (optional) Number of nodes to be deployed when Autoscaling is enabled | `string` | `"3"` | no |
| <a name="input_is_preemptible"></a> [is\_preemptible](#input\_is\_preemptible) | A boolean that represents whether or not the underlying node VMs are preemptible | `bool` | `false` | no |
| <a name="input_kubernetes_version"></a> [kubernetes\_version](#input\_kubernetes\_version) | The Kubernetes version for the nodes in this pool | `string` | `null` | no |
| <a name="input_labels"></a> [labels](#input\_labels) | maps containing node label | `map(string)` | `{}` | no |
| <a name="input_location"></a> [location](#input\_location) | (Optional) The region/zone the cluster is in | `string` | `null` | no |
| <a name="input_machine_type"></a> [machine\_type](#input\_machine\_type) | (optional) The type of machine to deployed | `string` | `null` | no |
| <a name="input_max_node_count"></a> [max\_node\_count](#input\_max\_node\_count) | (optional) Maximum amount Node when autoscaling enabled | `string` | `"5"` | no |
| <a name="input_max_pods_per_node"></a> [max\_pods\_per\_node](#input\_max\_pods\_per\_node) | (Optional) The maximum number of pods per node in this node pool. | `any` | `null` | no |
| <a name="input_metadata"></a> [metadata](#input\_metadata) | Optional) The metadata key/value pairs assigned to nodes in the node pool | `map` | `{}` | no |
| <a name="input_min_node_count"></a> [min\_node\_count](#input\_min\_node\_count) | (optional) Minimum amount of Nodes when autoscaling is enabled | `string` | `"1"` | no |
| <a name="input_name"></a> [name](#input\_name) | node pool name. name and name\_prefix cannot be specified on the same node pool | `string` | `null` | no |
| <a name="input_name_prefix"></a> [name\_prefix](#input\_name\_prefix) | node pool name prefix. name and name\_prefix cannot be specified on the same node pool | `string` | `null` | no |
| <a name="input_node_count"></a> [node\_count](#input\_node\_count) | (optional) Number of nodes to be deployed. Should not be used alongside autoscaling | `string` | `"1"` | no |
| <a name="input_oauth_scopes"></a> [oauth\_scopes](#input\_oauth\_scopes) | (Optional) Scopes that are used by NAP when creating node pools. | `list(string)` | `[]` | no |
| <a name="input_project_id"></a> [project\_id](#input\_project\_id) | name of the google project | `string` | n/a | yes |
| <a name="input_service_account"></a> [service\_account](#input\_service\_account) | (optional) Service account created GKE | `string` | `null` | no |
| <a name="input_tags"></a> [tags](#input\_tags) | (Optional) The list of instance tags applied to all nodes. Tags are used to identify valid sources or targets for network firewalls. | `list` | `[]` | no |
| <a name="input_taints"></a> [taints](#input\_taints) | (Optional) A list of Kubernetes taints to apply to nodes | <pre>list(object({<br>    key    = string<br>    value  = string<br>    effect = string<br>  }))</pre> | `[]` | no |
| <a name="input_update_timeout"></a> [update\_timeout](#input\_update\_timeout) | n/a | `string` | `"30m"` | no |
| <a name="input_workload_metadata_config"></a> [workload\_metadata\_config](#input\_workload\_metadata\_config) | whether to Run the GKE Metadata Server on this node. | `bool` | `true` | no |
| <a name="input_zones"></a> [zones](#input\_zones) | The zones to host the cluster in (optional if regional cluster / required if zonal) | `list(string)` | `null` | no |

## Outputs

| Name | Description |
|------|-------------|
| <a name="output_name"></a> [name](#output\_name) | n/a |
<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
