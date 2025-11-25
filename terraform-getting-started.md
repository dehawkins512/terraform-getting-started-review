# Run Your First NGINX Container with Terraform

In this guide, you run an NGINX web server inside a Docker container using Terraform.  
You learn the complete Terraform workflow: write configuration, initialize providers, preview changes, apply infrastructure, and destroy resources when you no longer need them.

**After completing this guide, you will be able to**:

- Write a basic Terraform configuration that uses the Docker provider
- Initialize a Terraform project and download required providers
- Preview planned changes with `terraform plan`
- Apply infrastructure changes safely
- Clean up resources with `terraform destroy`

## Prerequisites

- Docker Engine or Docker Desktop running locally
- Terraform 1.6 or later (download from https://developer.hashicorp.com/terraform/install)

## 1. Create a project directory

Create a new directory for the project and navigate into it.

```shell
$ mkdir terraform-demo && cd terraform-demo
```

## 2. Write the Terraform configuration

Create a file named `main.tf` and add the following content:

```hcl
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 3.0"        # ← Added version constraint for reproducibility
    }
  }
}

provider "docker" {}

data "docker_image" "nginx" {
  name = "nginx:latest"
}
<!-- Comment on the data source block -->
<!-- Suggested comment: Replaced deprecated `docker_image.nginx.latest` reference with a proper data source. Using `.latest` on a resource has been deprecated for years and can cause non-deterministic behavior. -->

resource "docker_container" "nginx" {
  name  = "training-nginx"
  image = data.docker_image.nginx.repo_digest

  ports {
    internal = 80
    external = 8000           # ← Changed from 80 → 8000 to avoid conflicts with services already running on host port 80
  }
}
```

The Docker provider automatically connects to the local Docker daemon on macOS, Windows (Docker Desktop), and Linux.

## 3. Initialize the Terraform working directory

```shell
$ terraform init
```

**Example output**
```
Initializing the backend...

Initializing provider plugins...
- Finding kreuzwerker/docker versions matching "~> 3.0"...
- Installing kreuzwerker/docker v3.0.2...
- Installed kreuzwerker/docker v3.0.2 (self-signed, key ID 1234567890ABCDEF)

Partner and community providers are signed by their developers.

Terraform has been successfully initialized!
```

## 4. Review the execution plan

```shell
$ terraform plan
```

**Example output (truncated)**
```
data.docker_image.nginx: Reading...
data.docker_image.nginx: Read complete after 0s [id=sha256:5a0f3a9...]

Terraform will perform the following actions:

  # docker_container.nginx will be created
  + resource "docker_container" "nginx" {
      + name  = "training-nginx"
      + image = "nginx@sha256:5a0f3a9..."
      + ports {
          + external = 8000
          + internal = 80
        }
      ...
    }

Plan: 1 to add, 0 to change, 0 to destroy.
```

## 5. Apply the configuration

```shell
$ terraform apply
```

Type `yes` when prompted.

**Example output (truncated)**
```
data.docker_image.nginx: Reading...
data.docker_image.nginx: Read complete after 0s
docker_container.nginx: Creating...
docker_container.nginx: Creation complete after 2s [id=abcd1234...]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

Open http://localhost:8000 in your browser. You see the NGINX welcome page.

## 6. Destroy the infrastructure

When you finish, remove the container.

```shell
$ terraform destroy
```

Type `yes` when prompted.

**Example output (truncated)**
```
docker_container.nginx: Destroying... [id=abcd1234...]
docker_container.nginx: Destruction complete after 1s

Destroy complete! Resources: 1 destroyed.
```