# Installing Cloudify Manager 4.0

## Prerequisites

This README requires that you have installed the Cloudify 4.0 CLI on your workstation. See [instructions](http://docs.getcloudify.org/4.0.0/installation/installation-overview/).

The following user stories are fully explained:
* I want to use a pre-baked Cloudify 4.0 manager image. (See pre-baked image)[#About the pre-baked image option].
* I need to create the environment (networks, VMs) that I will install Cloudify on (See management environment installation)[#Management environment installation].
* I have a VM on which I want to install Cloudify Manager. See (Bootstrap)[#bootstrap].
* I have Cloudify Manager 4.0 and I want to configure it. See (manager configuration)[#manager-configuration]
* I have a blueprint for a previous version of Cloudify and I want to use it with Cloudify Manager 4.0. See (Bootstrap)[#using-pre-4.0-blueprint-with-cloudify-manager-4.0].


## About the pre-baked image option

Cloudify Manager is distributed as a pre-baked image in the following image formats and marketplaces:

* [QCow2](http://getcloudify.org/downloads/get_cloudify.html)
* [VMDK](http://getcloudify.org/downloads/get_cloudify.html)
* [AWS Marketplace](http://getcloudify.org/downloads/get_cloudify.html)
* [Azure Marketplace](http://getcloudify.org/downloads/get_cloudify.html)

Pre-baked images are useful for users who do not have advanced security requirements or environment restrictions. They are also the fastest installation method.

**Note:** If you are going to create your management environment using our example blueprints, you can use the pre-baked images. There are special instructions in the installation steps for each Cloud.

**Note:** _If you want to use pre-baked images, but have restrictions that prevent you from using one of ours, you can build your own images using the [cloudify-image-bakery](https://github.com/cloudify-cosmo/cloudify-image-bakery)._


## Management environment installation

We assume that you have existing Cloud infrastructure that you want to manage with Cloudify, including a specific network/VPC/etc. If we guessed correctly, skip to Bootstrap.

If that assumption is false, we have several example infrastructure blueprints that you can use to deploy your Cloudify Manager. To use these examples, first download and extract the example repository:

```shell
$ curl -L https://github.com/cloudify-examples/aws-azure-openstack-blueprint/archive/4.0.tar.gz | tar xv
```

### AWS Infrastructure Installation

Prepare your AWS inputs file.

```shell
$ cp aws-azure-openstack-blueprint-4.0/inputs/aws.yaml.example inputs.yaml
```

Uncomment and provide values for the following fields:

- aws_access_key_id: ''
- aws_secret_access_key: ''

_If you do not have these credentials, follow (these instructions)[http://stackoverflow.com/questions/21440709/how-do-i-get-aws-access-key-id-for-amazon] or talk to your administrator._

All other fields are optional. Most frequently you will want to change the region. In that case, you will also need to update the *Region Overrides* section in the example.

**Note:**
> To use a pre-baked image, set the value of the example_aws_virtual_machine_image_id input to the AMI of the pre-baked Cloudify Manager 4.0 image.
> The current versions for each region are documented on (our website)[http://getcloudify.org/downloads/get_cloudify.html].

Now, install the AWS infrastructure:

```shell
$ cfy install aws-azure-openstack-blueprint-4.0/aws/blueprint.yaml -i inputs.yaml --task-retries=15 --task-retry-interval=15```
Initializing local profile ...
Initialization completed successfully
Initializing blueprint...
Initialized blueprint.yaml
If you make changes to the blueprint, run `cfy init blueprint.yaml` again to apply them
2019-12-31 00:00:00.000  CFY <local> Starting 'install' workflow execution
```

To obtain information about a resource, you can run:

```shell
$ cfy node-instances example_aws_elastic_ip
[
...
   "id": "example_aws_elastic_ip_q0qu0b",
   "name": "example_aws_elastic_ip",
...
   "runtime_properties": {
...
     "aws_resource_id": "XX.XXX.XX.X",
...
]
```

_The value of the _example_aws_elastic_ip_ is the IP that you will use to install your Cloudify Manager in the Bootstrap phase. You can also get this value from_ ```cfy deployments outputs```.


### Azure Infrastructure Installation

To prepare for this install, make sure that you have a key pair. If you do not generate them with these (instructions)[https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent].

Copy the public key material that you generated:

```shell
$ cat ~/.ssh/id_rsa.pub # copy this to clipboard.
```

Prepare your Azure inputs file.

```shell
$ cp aws-azure-openstack-blueprint-4.0/inputs/azure.yaml.example inputs.yaml
```

Uncomment and provide values for the following fields:

- subscription_id
- tenant_id
- client_id
- client_secret
- example_azure_virtual_machine_public_key_data

_If you do not have these credentials, follow these (instructions)[https://docs.microsoft.com/en-us/rest/api/#client-registration] or talk to your administrator._

All other fields are optional.

**Note:**
> If you want to use a pre-baked image, set the value of the example_azure_virtual_machine_resource_config input to the following:
> XYXYXYXYXYXYXYXYXYXYXYXXYXYXYXYXYXYXYXYXYXYXYXXYXYX
> XYXYXYXYXYXYXYXYXYXYXYXXYXYXYXYXYXYXYXYXYXYXYXXYXYX
> XYXYXYXYXYXYXYXYXYXYXYXXYXYXYXYXYXYXYXYXYXYXYXXYXYX
> XYXYXYXYXYXYXYXYXYXYXYXXYXYXYXYXYXYXYXYXYXYXYXXYXYX


Now, install the Azure infrastructure:

```shell
$ cfy install aws-azure-openstack-blueprint-4.0/azure/blueprint.yaml -i inputs.yaml --task-retries=15 --task-retry-interval=15```
Initializing local profile ...
Initialization completed successfully
Initializing blueprint...
Initialized blueprint.yaml
If you make changes to the blueprint, run `cfy init blueprint.yaml` again to apply them
2019-12-31 00:00:00.000  CFY <local> Starting 'install' workflow execution
```

To get information about a resource, you can run:

```shell
$ cfy node-instances example_azure_virtual_machine

[
...
  {
...
    "name": "example_azure_virtual_machine",
...
    "runtime_properties": {
...
      "ip": "10.10.1.4",
...
      "public_ip": "XX.XXX.XX.XX"
    },
    "state": "started",
    "version": 9
  }
...
]
```

_The value of the example_azure_virtual_machine public_ip runtime property is the IP that you will use to install your Cloudify Manager in the bootstrap phase. You can also get this value from_ ```cfy deployments outputs```.


### Openstack Infrastructure Installation

Prepare your Openstack inputs file.

```shell
$ cp aws-azure-openstack-blueprint-4.0/inputs/openstack.yaml.example inputs.yaml
```

Uncomment and provide values for the following fields:

- keystone_username
- keystone_password
- keystone_tenant_name
- keystone_url
- region
- example_openstack_virtual_machine_image_id
- example_openstack_virtual_machine_flavor_id

_If you do not have these credentials, follow these instructions (keystone v2)[https://docs.openstack.org/developer/python-keystoneclient/using-api-v2.html] or (keystone v3)[https://docs.openstack.org/developer/python-keystoneclient/using-api-v3.html] or talk to your administrator._

**Note:**
> To use a pre-baked image, set the value of the example_openstack_virtual_machine_image_id input to the image_id of the pre-baked Cloudify Manager 4.0 image.
> If you do not have one in your Openstack account, download an image from (our website)[http://getcloudify.org/downloads/get_cloudify.html], then upload it to your openstack via the (Horizon UI)[https://docs.openstack.org/user-guide/dashboard-manage-images.html] or via (Glance)[https://docs.openstack.org/cli-reference/glance.html].

Now, install the Openstack infrastructure:

```shell
$ cfy install aws-azure-openstack-blueprint-4.0/openstack/blueprint.yaml -i inputs.yaml --task-retries=15 --task-retry-interval=15```
Initializing local profile ...
Initialization completed successfully
Initializing blueprint...
Initialized blueprint.yaml
If you make changes to the blueprint, run `cfy init blueprint.yaml` again to apply them
2019-12-31 00:00:00.000  CFY <local> Starting 'install' workflow execution
```

To obtain information about a resource, you can run:

```shell
$ cfy node-instances example_openstack_floating_ip
[
...
  {
...
    "name": "example_openstack_floating_ip",
...
    "runtime_properties": {
      "floating_ip_address": "XXX.XX.XXX.XX"
    },
...
  }
...
]

```

_The value of the example_openstack_floating_ip floating_ip_address runtime property is the IP that you will use to install your Cloudify Manager in the bootstrap phase. You can also get this value from_ ```cfy deployments outputs```.


## Bootstrap

If you have a clean Centos or RHEL VM on which you want to install Cloudify Manager 4.0, you need to bootstrap. If you used a pre-baked image to install your manager, skip to (manager configuration)[#manager-configuration].

First, log into the VM, install the Cloudify RPM, and copy your Cloudify Manager 4.0 inputs file to your local directory.

```shell
[cloudify@cloudify ~]$ sudo rpm -i http://repository.cloudifysource.org/cloudify/4.0.0/rc1-release/cloudify-4.0.0~rc1.el6.x86_64.rpm
You're about to install Cloudify!
Thank you for installing Cloudify!
[cloudify@cloudify ~]$ cp /opt/cfy/cloudify-manager-blueprints/simple-manager-blueprint-inputs.yaml inputs.yaml
```

Uncomment and provide values for the following fields:

- public_ip: '' # It is recommended that you use the IP with which you SSHed, or you could use localhost.
- private_ip: '' # This should be the IP of the interface, such as eth0 or eth1, that you want Cloudify Manager to listen on.
- ssh_key_filename: '' # To generate a new one, it's recommended to follow these (instructions)[https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent].
- ssh_user: '' # This is the SSH user that you want Cloudify to use during bootstrap. Usually this will be Centos, although it depends on the VM you are using. For example, in this instance the user is "cloudify".

All other fields are optional and depend on your system requirements.

Now, you will bootstrap:

```shell
[cloudify@cloudify ~]$ cfy bootstrap /opt/cfy/cloudify-manager-blueprints/simple-manager-blueprint.yaml -i inputs.yaml
Initializing profile temp-AAAAAA...
Initialization completed successfully
Executing bootstrap validation...
Initializing blueprint...
....
2017-04-03 11:19:40.715  CFY <manager> 'execute_operation' workflow execution succeeded
Bootstrap complete
Manager is up at [public_ip]
##################################################
Manager password is [manager_password]
##################################################
```

At the end of the bootstrap process the admin default_tenant username is printed. Use it in the next step.

## Manager configuration

There are a few recommended steps for configuring Cloudify Manager.

### Create profiles and tenants.

First, you create a profile using the Cloudify CLI:

```shell
$ cfy profiles use [public_ip] --profile-name [alias] -s [ssh_user] -k [ssh_key_filename] -u admin -p [manager_password] -t default_tenant
Attempting to connect...
Initializing profile azure...
Initialization completed successfully
Using manager [public_ip] with port 80
```

Now, you create any tenants that you require, and switch to the tenant for further configuration:

```shell
$ cfy tenants create demo
Tenant `demo` created
$ cfy profiles use [public_ip] --profile-name [alias-demo] -u admin -p [manager_password] -t demo
Attempting to connect...
Initializing profile azure...
Initialization completed successfully
Using manager [public_ip] with port 80
```

### Add your secrets:

_Secrets are shared with tenants._

#### AWS

```shell
$ cfy secrets create aws_access_key_id -s AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Secret `aws_access_key_id` created
$ cfy secrets create aws_secret_access_key -s AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Secret `aws_secret_access_key` created
```

#### Azure

```shell
$ cfy secrets create subscription_id -s AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Secret `subscription_id` created
$ cfy secrets create tenant_id -s AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Secret `tenant_id` created
$ cfy secrets create client_id -s AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Secret `client_id` created
$ cfy secrets create client_secret -s AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Secret `client_secret` created
```


#### Openstack

```shell
$ cfy secrets create keystone_username -s AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Secret `keystone_username` created
$ cfy secrets create keystone_password -s AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Secret `keystone_password` created
$ cfy secrets create keystone_tenant_name -s AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Secret `keystone_tenant_name` created
$ cfy secrets create keystone_url -s AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Secret `keystone_url` created
```

### Upload plugins to Cloudify Manager:

**Note:** You need to download the plugin packages from (our website)[http://getcloudify.org/downloads/plugin-packages.html]:

**Example:**

```shell
$ curl -O http://repository.cloudifysource.org/cloudify/wagons/cloudify-diamond-plugin/1.3.5/cloudify_diamond_plugin-1.3.5-py27-none-linux_x86_64-centos-Core.wgn
$ curl -O http://repository.cloudifysource.org/cloudify/wagons/cloudify-diamond-plugin/1.3.5/cloudify_diamond_plugin-1.3.5-py27-none-linux_x86_64-Ubuntu-trusty.wgn
$ curl -O http://repository.cloudifysource.org/cloudify/wagons/cloudify-openstack-plugin/2.0.1/cloudify_openstack_plugin-2.0.1-py27-none-linux_x86_64-centos-Core.wgn
$ curl -O http://repository.cloudifysource.org/cloudify/wagons/cloudify-aws-plugin/1.4.4/cloudify_aws_plugin-1.4.4-py27-none-linux_x86_64-centos-Core.wgn
$ cfy plugins upload cloudify_diamond_plugin-1.3.5-py27-none-linux_x86_64-centos-Core.wgn
$ cfy plugins upload cloudify_diamond_plugin-1.3.5-py27-none-linux_x86_64-Ubuntu-trusty.wgn
$ cfy plugins upload cloudify_openstack_plugin-2.0.1-py27-none-linux_x86_64-centos-Core.wgn
$ cfy plugins upload cloudify_aws_plugin-1.4.4-py27-none-linux_x86_64-centos-Core.wgn
```

### Using pre-4.0 blueprint with Cloudify Manager 4.0 

_We will support a method for configuring Cloudify Manager to support blueprints writing for Cloudify <4.0._
