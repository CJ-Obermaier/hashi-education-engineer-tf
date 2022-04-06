# Getting Started with Terraform
In this guide, you will learn the fundamental Terraform commands to deploy and destroy infrastructure by deploying an NGINX web server running in a Docker container. 

## What is Terraform?
Terraform is an infrastructure as code tool that lets you define both cloud and on-prem resources in human-readable configuration files that you can version, reuse, and share.

## Prerequisites
- **Terraform**
  -  [Terraform setup guide](https://learn.hashicorp.com/tutorials/terraform/install-cli?in=terraform/aws-get-started#install-terraform) 
- **Docker** 
  - [Docker setup guide](https://docs.docker.com/get-started/)

## Verify that Terraform and Docker are installed and running
#### Terraform
```shell
$ terraform -help                                                                

Usage: terraform [global options] <subcommand> [args]

The available commands for execution are listed below.
The primary workflow commands are given first, followed by
less common or more advanced commands.
```
#### Docker
```shell
$ docker                                                               

Usage:  docker [OPTIONS] COMMAND

A self-sufficient runtime for containers

```
## Create your workspace
Create a workspace on your system. This is where you will create and run your Terraform.

```shell
$ mkdir terraform-demo
$ cd terraform-demo
```

## Create Terraform code

In the terraform-demo folder, create a main.tf
```shell
$ touch main.tf
```
In Powershell
```powershell
$null > main.tf
```

Add the following code to the  `main.tf` file.

```hcl
terraform {
  required_providers {
    docker = {
      source = "kreuzwerker/docker"
    }
  }
}
provider "docker" {
    host = "unix:///var/run/docker.sock"
}
resource "docker_container" "nginx" {
  image = docker_image.nginx.latest
  name  = "training"
  ports {
    internal = 80
    external = 80
  }
}
resource "docker_image" "nginx" {
  name = "nginx:latest"
}
```
The code above will create two resources. The first resource `docker_container` will start a container by setting the `image` value to `docker_image.nginx.latest`. The second resource `docker_image` will pull an NGINX container from Docker Hub and then make it available to the `docker_container` resource under the reference name `docker_image.nginx.latest`. 

## Terraform validate
Run `terraform validate` in the `terraform-demo` folder to check if the `main.tf` code is valid.
```shell
$ terraform validate

Success! The configuration is valid.
```

## Initialize Terraform
Initialize Terraform with the `init` command. The AWS provider will be installed in the hidden `.terraform` folder.

```shell
$ terraform init
```
The output should be similar to the following:
```shell
$ ~/terraform-demo » terraform init

Initializing the backend...

Initializing provider plugins...
- Finding latest version of kreuzwerker/docker...
- Installing kreuzwerker/docker v2.16.0...
- Installed kreuzwerker/docker v2.16.0 (self-signed, key ID BD080C4571C6104C)

Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/cli/plugins/signing.html

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

## Terraform plan
`terraform plan` compares the desired infrastructure state to the actual infrastructure state. You are starting from a fresh environment so `terraform plan` should display two resources that will be created. 

```shell
terraform plan
```
Below is the result of running `terraform plan`. Notice the bottom of the output. Terraform informs you that this code will add two resources if applied.

```shell
$ ~/terraform-demo » terraform plan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # docker_container.nginx will be created
  + resource "docker_container" "nginx" {
      + attach           = false
      + bridge           = (known after apply)
      + command          = (known after apply)
      + container_logs   = (known after apply)
      + entrypoint       = (known after apply)
      + env              = (known after apply)
      + exit_code        = (known after apply)
      + gateway          = (known after apply)
      + hostname         = (known after apply)
      + id               = (known after apply)
      + image            = (known after apply)
      + init             = (known after apply)
      + ip_address       = (known after apply)
      + ip_prefix_length = (known after apply)
      + ipc_mode         = (known after apply)
      + log_driver       = (known after apply)
      + logs             = false
      + must_run         = true
      + name             = "training"
      + network_data     = (known after apply)
      + read_only        = false
      + remove_volumes   = true
      + restart          = "no"
      + rm               = false
      + security_opts    = (known after apply)
      + shm_size         = (known after apply)
      + start            = true
      + stdin_open       = false
      + tty              = false

      + healthcheck {
          + interval     = (known after apply)
          + retries      = (known after apply)
          + start_period = (known after apply)
          + test         = (known after apply)
          + timeout      = (known after apply)
        }

      + labels {
          + label = (known after apply)
          + value = (known after apply)
        }

      + ports {
          + external = 80
          + internal = 80
          + ip       = "0.0.0.0"
          + protocol = "tcp"
        }
    }

  # docker_image.nginx will be created
  + resource "docker_image" "nginx" {
      + id          = (known after apply)
      + latest      = (known after apply)
      + name        = "nginx:latest"
      + output      = (known after apply)
      + repo_digest = (known after apply)
    }

Plan: 2 to add, 0 to change, 0 to destroy.
```

## Terraform apply
If the `terraform plan` looks good then you are ready to deploy your Terraform managed infrastructure. 

```shell
$ terraform apply
```
Terraform will ask if you want to apply the configuration generated above. Answer `yes` to deploy the infrastructure. This deployment might take a few minutes, but when finished, will display the resources created at the bottom.


```shell
$ ~/terraform-demo
Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

docker_image.nginx: Creating...
docker_image.nginx: Creation complete after 6s [id=sha256:12766a6745eea133de9fdcd03ff720fa971fdaf21113d4bc72b417c123b15619nginx:latest]
docker_container.nginx: Creating...
docker_container.nginx: Creation complete after 1s [id=fb6c099f6287e07622c793078a580fb43cb9b44d20f3b623987a9b01f6bd91a1]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```
## Verify the deployment
Congratulations! You have deployed an NGINX server running in a Docker container. To verify the deployment, open a web browser and go to http://localhost:80. You should see the NGINX welcome screen.


## Clean up
Once you are ready to destroy your Terraform managed infrastructure, run `terraform destroy` in the `terraform-demo` folder where the `main.tf` lives. Terraform will generate an output with what will be destroyed. The bottom of the output will list 2 resources that will be destroyed. These resources are the `docker_container` and `docker_image` that you deployed with `terraform apply`

```shell
$ terraform destroy
```
You will be prompted if you are sure you want to destroy all resources. Reply `yes` if you are ready.

```shell
Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

docker_container.nginx: Destroying... [id=fb6c099f6287e07622c793078a580fb43cb9b44d20f3b623987a9b01f6bd91a1]
docker_container.nginx: Destruction complete after 0s
docker_image.nginx: Destroying... [id=sha256:12766a6745eea133de9fdcd03ff720fa971fdaf21113d4bc72b417c123b15619nginx:latest]
docker_image.nginx: Destruction complete after 0s

Destroy complete! Resources: 2 destroyed.
```

## Next steps
In this guide, you learned how to validate, plan, apply and destroy infrastructure using Terraform. If you enjoyed this tutorial, check out the tutorials at [Hashicorp Learn](https://learn.hashicorp.com/terraform), specifically the additional [Docker tutorials](https://learn.hashicorp.com/collections/terraform/docker-get-started).