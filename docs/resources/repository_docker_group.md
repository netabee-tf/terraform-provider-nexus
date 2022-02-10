---
page_title: "Resource nexus_repository_docker_group"
subcategory: "Repository"
description: |-
  Use this resource to create a group docker repository.
---
# Resource nexus_repository_docker_group
Use this resource to create a group docker repository.
## Example Usage
```terraform
resource "nexus_repository_docker_hosted" "internal" {
  name = "internal"

  docker {
    force_basic_auth = false
    v1_enabled       = false
  }

  storage {
    blob_store_name                = "default"
    strict_content_type_validation = true
    write_policy                   = "ALLOW"
  }
}

resource "nexus_repository_docker_proxy" "dockerhub" {
  name = "dockerhub"

  docker {
    force_basic_auth = false
    v1_enabled       = false
  }

  docker_proxy {
    index_hub = "HUB"
  }

  storage {
    blob_store_name                = "default"
    strict_content_type_validation = true
  }

  proxy {
    remote_url       = "https://registry-1.docker.io"
    content_max_age  = 1440
    metadata_max_age = 1440
  }

  negative_cache {
    enabled      = true
    time_to_live = 1440
  }

  http_client {
    blocked    = false
    auto_block = true
  }
}

resource "nexus_repository_docker_group" "group" {
  name   = "docker-group"
  online = true

  docker {
    force_basic_auth = false
    http_port        = 8080
    https_port       = 8433
    v1_enabled       = false
  }

  group {
    member_names = [
      nexus_repository_docker_hosted.internal.name,
      nexus_repository_docker_proxy.dockerhub.name
    ]
    writable_member = nexus_repository_docker_hosted.internal.name
  }

  storage {
    blob_store_name                = "default"
    strict_content_type_validation = true
  }
}
```
<!-- schema generated by tfplugindocs -->
## Schema

### Required

- **docker** (Block List, Min: 1, Max: 1) docker contains the configuration of the docker repository (see [below for nested schema](#nestedblock--docker))
- **group** (Block List, Min: 1, Max: 1) Configuration for repository group (see [below for nested schema](#nestedblock--group))
- **name** (String) A unique identifier for this repository
- **storage** (Block List, Min: 1, Max: 1) The storage configuration of the repository (see [below for nested schema](#nestedblock--storage))

### Optional

- **online** (Boolean) Whether this repository accepts incoming requests

### Read-Only

- **id** (String) Used to identify resource at nexus

<a id="nestedblock--docker"></a>
### Nested Schema for `docker`

Required:

- **force_basic_auth** (Boolean) Whether to force authentication (Docker Bearer Token Realm required if false)
- **v1_enabled** (Boolean) Whether to allow clients to use the V1 API to interact with this repository

Optional:

- **http_port** (Number) Create an HTTP connector at specified port
- **https_port** (Number) Create an HTTPS connector at specified port


<a id="nestedblock--group"></a>
### Nested Schema for `group`

Required:

- **member_names** (Set of String) Member repositories names

Optional:

- **writable_member** (String) Pro-only: This field is for the Group Deployment feature available in NXRM Pro.


<a id="nestedblock--storage"></a>
### Nested Schema for `storage`

Required:

- **blob_store_name** (String) Blob store used to store repository contents

Optional:

- **strict_content_type_validation** (Boolean) Whether to validate uploaded content's MIME type appropriate for the repository format
## Import
Import is supported using the following syntax:
```shell
# import using the name of repository
terraform import nexus_repository_docker_group.group docker-group
```