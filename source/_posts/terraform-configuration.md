---
title: Terraform Configuration
date: 2018-07-19 10:05:02
categories:
- DevOps
tags:
- Terraform
---
Terraform is an declarative system for defining infrastructure, you can find out more from [terraform.io](terraform.io).

Terraform uses text files to describe infrastructure and to set variables. These text files are called Terraform configurations and end in .tf.

# Structure we use

```
<myapp>-infrastructure/
 - infrastructure/
   - envs/
    - <your environment file>.tfvar - overrides of variables.tf
   - common.tf - set up your provider (typically kubernetes) and also set up the requirements for the infrastructure-core components
   - data.tf - backend data store
   - variables.tf - deployment variables like aws_region, environment, etc
   - kubernetes.tf - k8s definitions, mainly for secrets
```

# Configuration

## Load Order and Semantics

When invoking any command that loads the Terraform configuration, Terraform loads all configuration files within the directory specified in alphabetical order.

Override files are the exception, as they're loaded after all non-override files, in alphabetical order.

The configuration within the loaded files are appended to each other. This is in contrast to being merged. This means that two resources with the same name are not merged, and will instead cause a validation error. This is in contrast to overrides, which do merge.

The order of variables, resources, etc. defined within the configuration doesn't matter. Terraform configurations are declarative, so references to other resources and variables do not depend on the order they're defined.

## Syntax

The syntax of Terraform configurations is called [HashiCorp Configuration Language (HCL)](https://github.com/hashicorp/hcl). It is meant to strike a balance between human readable and editable as well as being machine-friendly.

Example:

```hcl
# An AMI
variable "ami" {
  description = "the AMI to use"
}

/* A multi
   line comment. */
resource "aws_instance" "web" {
  ami               = "${var.ami}"
  count             = 2
  source_dest_check = false

  connection {
    user = "root"
  }
}
```

Basic bullet point reference:
* Strings are in double-quotes.
* Strings can interpolate other values using syntax wrapped in ${}, such as ${var.foo}.
* You can perform simple math in interpolations, allowing you to write expressions such as ${count.index + 1}. And you can also use conditionals to determine a value based on some logic.
* You can escape interpolation with double dollar signs: $${foo} will be rendered as a literal ${foo}.
* More about [Interpolation, Available Variables, Conditionals, Built-in Functions, Templates, Math](https://www.terraform.io/docs/configuration/interpolation.html)
* Multiline strings can use shell-style "here doc" syntax, with the string starting with a marker like <<EOF and then the string ending with EOF on a line of its own. The lines of the string and the end marker must not be indented.
* Numbers are assumed to be base 10. If you prefix a number with 0x, it is treated as a hexadecimal number.
* Boolean values: true, false.
* Maps can be made with braces ({}) and colons (:): { "foo": "bar", "bar": "baz" }. Quotes may be omitted on keys, unless the key starts with a number, in which case quotes are required. Commas are required between key/value pairs for single line maps. A newline between key/value pairs is sufficient in multi-line maps.

## Overrides

Overrides have a few use cases:

* Machines (tools) can create overrides to modify Terraform behavior without having to edit the Terraform configuration tailored to human readability.
* Temporary modifications can be made to Terraform configurations without having to modify the configuration itself.

Overrides names must be override or end in _override, excluding the extension. Examples of valid override files are override.tf, override.tf.json, temp_override.tf.

Override files are loaded last in alphabetical order.

## Resource Configuration

Resources are a component of your infrastructure. It might be some low level component such as a physical server, virtual machine, or container. Or it can be a higher level component such as an email provider, DNS record, or database provider.

Example:

```hcl
resource "aws_instance" "web" {
  ami           = "ami-408c7f28"
  instance_type = "t1.micro"
}
```

The resource block creates a resource of the given TYPE (first parameter) and NAME (second parameter). The combination of the type and name must be unique.

Within the block (the { }) is configuration for the resource. The configuration is dependent on the type.

### Meta-parameters

### Timeouts

### Explicit Dependencies

A resource automatically depends on anything it references via interpolations. The automatically determined dependencies are all that is needed most of the time. You can also use the depends_on parameter to explicitly define a list of additional dependencies.

### Connection block

### Provisioner

## Data Source Configuration

Data sources allow data to be fetched or computed for use elsewhere in Terraform configuration. Use of data sources allows a Terraform configuration to build on information defined outside of Terraform, or defined by another separate Terraform configuration.

Every data source in Terraform is mapped to a provider based on longest-prefix matching. For example the aws_ami data source would map to the aws provider (if that exists).

Example:

```hcl
# Find the latest available AMI that is tagged with Component = web
data "aws_ami" "web" {
  filter {
    name   = "state"
    values = ["available"]
  }

  filter {
    name   = "tag:Component"
    values = ["web"]
  }

  most_recent = true
}
```

The data block creates a data instance of the given TYPE (first parameter) and NAME (second parameter). The combination of the type and name must be unique.

Within the block (the { }) is configuration for the data instance. The configuration is dependent on the type.

Each data instance will export one or more attributes, which can be interpolated into other resources using variables of the form data.TYPE.NAME.ATTR. For example:

```hcl
resource "aws_instance" "web" {
  ami           = "${data.aws_ami.web.id}"
  instance_type = "t1.micro"
}
```

## Provider Configuration

Providers are responsible in Terraform for managing the lifecycle of a resource: create, read, update, delete.

Most providers require some sort of configuration to provide authentication information, endpoint URLs, etc. Where explicit configuration is required, a provider block is used within the configuration.

By default, resources are matched with provider configurations by matching the start of the resource name. For example, a resource of type vsphere_virtual_machine is associated with a provider called vsphere.

Example:

```hcl
provider "aws" {
  access_key = "foo"
  secret_key = "bar"
  region     = "us-east-1"
}
```

A provider block represents a configuration for the provider named in its header. For example, provider "aws" above is a configuration for the aws provider.

Within the block body (between { }) is configuration for the provider. The configuration is dependent on the type, and is documented for each provider.

The arguments alias and version, if present, are special arguments handled by Terraform Core for their respective features described above. All other arguments are defined by the provider itself.

A provider block may be omitted if its body would be empty. Using a resource in configuration implicitly creates an empty provider configuration for it unless a provider block is explicitly provided.

Each time a new provider is added to configuration -- either explicitly via a provider block or by adding a resource from that provider -- it's necessary to initialize that provider before use. Initialization downloads and installs the provider's plugin and prepares it to be used.

Providers are released on a separate rhythm from Terraform itself, and thus have their own version numbers. For production use, it is recommended to constrain the acceptable provider versions via configuration, to ensure that new versions with breaking changes will not be automatically installed by terraform init in future.

You can define multiple configurations for the same provider in order to support multiple regions, multiple hosts, etc. 

## Input Variable Configuration

Input variables serve as parameters for a Terraform module.

When used in the root module of a configuration, variables can be set from CLI arguments and environment variables. For child modules, they allow values to pass from parent to child.

Example:

```hcl
variable "key" {
  type = "string"
}

variable "images" {
  type = "map"

  default = {
    us-east-1 = "image-1234"
    us-west-2 = "image-4567"
  }
}

variable "zones" {
  default = ["us-east-1a", "us-east-1b"]
}
```

* [How to use variables](https://www.terraform.io/intro/getting-started/variables.html)
* [Variable Types](https://www.terraform.io/docs/configuration/variables.html)

## Output Configuration

Outputs define values that will be highlighted to the user when Terraform applies, and can be queried easily using the output command.

Terraform knows a lot about the infrastructure it manages. Most resources have attributes associated with them, and outputs are a way to easily extract and query that information.

Example:

```hcl
output "address" {
  value = "${aws_instance.db.public_dns}"
}
```

The output block configures a single output variable. Multiple output variables can be configured with multiple output blocks. The NAME given to the output block is the name used to reference the output variable.

[More details](https://www.terraform.io/docs/configuration/outputs.html)

## Local Value Configuration

Local values assign a name to an expression, that can then be used multiple times within a module.

Comparing modules to functions in a traditional programming language, if variables are analogous to function arguments and outputs are analogous to function return values then local values are comparable to a function's local variables.

Example:

```hcl
# Ids for multiple sets of EC2 instances, merged together
locals {
  instance_ids = "${concat(aws_instance.blue.*.id, aws_instance.green.*.id)}"
}

# A computed default name prefix
locals {
  default_name_prefix = "${var.project_name}-web"
  name_prefix         = "${var.name_prefix != "" ? var.name_prefix : local.default_name_prefix}"
}

# Local values can be interpolated elsewhere using the "local." prefix.
resource "aws_s3_bucket" "files" {
  bucket = "${local.name_prefix}-files"
  # ...
}
```

## Module Configuration

Modules are used in Terraform to modularize and encapsulate groups of resources in your infrastructure.

Example:

```hcl
module "consul" {
  source  = "hashicorp/consul/aws"
  servers = 5
}
```


A module block instructs Terraform to create an instance of a module, and in turn to instantiate any resources defined within it.

The name given in the block header is used to reference the particular module instance from expressions within the calling module, and to refer to the module on the command line. It has no meaning outside of a particular Terraform configuration.

Within the block body is the configuration for the module. All attributes within the block must correspond to variables within the module, with the exception of the following which Terraform treats as special:

* source - (Required) A module source string specifying the location of the child module source code.
* version - (Optional) A version constraint string that specifies which versions of the referenced module are acceptable. The newest version matching the constraint will be used. version is supported only for modules retrieved from module registries.
* providers - (Optional) A map whose keys are provider configuration names that are expected by child module and whose values are corresponding provider names in the calling module. This allows provider configurations to be passed explicitly to child modules. If not specified, the child module inherits all of the default (un-aliased) provider configurations from the calling module.

[More details](https://www.terraform.io/docs/modules/index.html)

# References
* [Segment Terraform Stack Example Repository](https://github.com/segmentio/stack)
* [Blog Post on Structuring Repository](http://www.antonbabenko.com/2016/09/21/how-i-structure-terraform-configurations.html)