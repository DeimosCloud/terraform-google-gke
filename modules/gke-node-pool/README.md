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
| google | n/a |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| auto\_repair | (Optional) Whether the nodes will be automatically repaired. | `bool` | `true` | no |
| auto\_upgrade | (Optional) Whether the nodes will be automatically upgraded. | `bool` | `true` | no |
| autoscaling | Whether to enable autoscaling. Values of min\_node\_count and max\_node\_count will be used | `bool` | `true` | no |
| cluster | Name of GKE cluster | `string` | n/a | yes |
| create\_timeout | n/a | `string` | `"30m"` | no |
| delete\_timeout | n/a | `string` | `"30m"` | no |
| disable\_legacy\_metadata\_endpoints | disable-legacy-endpoints metadata set | `bool` | `true` | no |
| disk\_size\_gb | (Optional) Size of the disk attached to each node, specified in GB. The smallest allowed disk size is 10GB | `number` | `30` | no |
| disk\_type | (Optional) Type of the disk attached to each node (e.g. 'pd-standard', 'pd-balanced' or 'pd-ssd'). | `string` | `"pd-standard"` | no |
| enable\_integrity\_monitoring | (Optional) Defines if the instance has integrity monitoring enabled. | `bool` | `true` | no |
| enable\_secure\_boot | Defines if the instance has Secure Boot enabled. | `bool` | `false` | no |
| image\_type | (optional) The type of image to be used | `string` | `"COS"` | no |
| initial\_node\_count | (optional) Number of nodes to be deployed when Autoscaling is enabled | `string` | `"3"` | no |
| is\_preemptible | A boolean that represents whether or not the underlying node VMs are preemptible | `bool` | `false` | no |
| kubernetes\_version | The Kubernetes version for the nodes in this pool | `string` | `null` | no |
| labels | maps containing node label | `map(string)` | `{}` | no |
| location | (Optional) The region/zone the cluster is in | `string` | `null` | no |
| machine\_type | (optional) The type of machine to deployed | `string` | `null` | no |
| max\_node\_count | (optional) Maximum amount Node when autoscaling enabled | `string` | `"5"` | no |
| max\_pods\_per\_node | (Optional) The maximum number of pods per node in this node pool. | `any` | `null` | no |
| metadata | Optional) The metadata key/value pairs assigned to nodes in the node pool | `map` | `{}` | no |
| min\_node\_count | (optional) Minimum amount of Nodes when autoscaling is enabled | `string` | `"1"` | no |
| name | node pool name | `string` | n/a | yes |
| node\_count | (optional) Number of nodes to be deployed. Should not be used alongside autoscaling | `string` | `"1"` | no |
| oauth\_scopes | (Optional) Scopes that are used by NAP when creating node pools. | `list(string)` | `[]` | no |
| project\_id | name of the google project | `string` | n/a | yes |
| service\_account | (optional) Service account created GKE | `string` | `null` | no |
| tags | (Optional) The list of instance tags applied to all nodes. Tags are used to identify valid sources or targets for network firewalls. | `list` | `[]` | no |
| taints | (Optional) A list of Kubernetes taints to apply to nodes | <pre>list(object({<br>    key    = string<br>    value  = string<br>    effect = string<br>  }))</pre> | `[]` | no |
| update\_timeout | n/a | `string` | `"30m"` | no |
| workload\_metadata\_config | Metadata configuration to expose to workloads on the node pool. | `string` | `null` | no |
| zones | The zones to host the cluster in (optional if regional cluster / required if zonal) | `list(string)` | `null` | no |

## Outputs

| Name | Description |
|------|-------------|
| name | n/a |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
