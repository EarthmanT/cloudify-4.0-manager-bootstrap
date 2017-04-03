# cloudify-4.0-manager-bootstrap

## Prerequisites

This README requires that you have installed the Cloudify 4.0 CLI on your workstation. See (instructions)[http://docs.getcloudify.org/4.0.0/installation/installation-overview/].

* I want to use a pre-baked Cloudify 4.0 manager image. [See pre-baked image](#Pre-baked Image).
* I need to create the environment (networks, VMs) that I will install Cloudify on [See Management Environment Installation](#Management Environment Installation).
* I have a VM that I want to install Cloudify Manager on. [See Bootstrap](#Bootstrap).


#### Pre-baked Image

Cloudify manager is distributed as a pre-baked image in the following image formats and marketplaces:

* [QCow2](http://getcloudify.org/downloads/get_cloudify.html)
* [VMDK](http://getcloudify.org/downloads/get_cloudify.html)
* [AWS Marketplace](http://getcloudify.org/downloads/get_cloudify.html)
* [Azure Marketplace](http://getcloudify.org/downloads/get_cloudify.html)

Pre-baked images are useful for users who do not have advanced security requirements or environment restrictions. They are also the fastest installation method.

**Note:** If you are going to create your management environment using our example blueprints, you can use the pre-baked images. There are special instructions in the installation steps for each Cloud.

**Note:** _If you want to use pre-baked images, but have restrictions that prevent you from using on of our's, you can build your own images using the (cloudify-image-bakery)[https://github.com/cloudify-cosmo/cloudify-image-bakery]._


## Management Environment Installation

We assume that you have existing Cloud infrastructure that you want to manage with Cloudify, including a specific network/VPC/etc. If we guessed, right, skip to Bootstrap.

If that assumption is false, we have several example infrastructure blueprints that you can use to deploy your Cloudify Manager. To use these examples, first download and extract the example repository:

```shell
$ curl -L https://github.com/cloudify-examples/aws-azure-openstack-blueprint/archive/4.0.tar.gz | tar xv
```

### AWS Infrastructure Installation

Prepare your AWS inputs file.

```shell
$ cp aws-azure-openstack-blueprint-4.0/inputs/aws.yaml.example inputs.yaml
```

Uncomment and provide the following fields:

- aws_access_key_id: ''
- aws_secret_access_key: ''

_If you do not have these credentials, follow these (instructions)[http://stackoverflow.com/questions/21440709/how-do-i-get-aws-access-key-id-for-amazon] or talk to your administrator._

All other fields are optional. Most frequently you will want to change the region. In that case, you will also need to update the *Region Overrides* section in the example.

Now install the AWS infrastructure:

```shell
$ cfy install aws-azure-openstack-blueprint-4.0/aws/blueprint.yaml -i inputs.yaml --task-retries=15 --task-retry-interval=15```
Initializing local profile ...
Initialization completed successfully
Initializing blueprint...
Initialized blueprint.yaml
If you make changes to the blueprint, run `cfy init blueprint.yaml` again to apply them
2019-12-31 00:00:00.000  CFY <local> Starting 'install' workflow execution
```

To get information about a resource, you can run:

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

**The value of the _example_aws_elastic_ip_ is the IP that you will use to install your Cloudify Manager in the Bootstrap phase.**


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

Uncomment and provide the following fields:

- subscription_id
- tenant_id
- client_id
- client_secret
- example_azure_virtual_machine_public_key_data

_If you do not have these credentials, follow these (instructions)[https://docs.microsoft.com/en-us/rest/api/#client-registration] or talk to your administrator._

All other fields are optional.

Now install the Azure infrastructure:

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


## Bootstrap

If you have a clean Centos or RHELL VM that you want to install Cloudify 4.0 manager on, then you need to bootstrap.

First, log into the VM, install the Cloudify RPM, and copy your Cloudify 4.0 manager inputs file to your local directory.

```shell
[cloudify@cloudify ~]$ sudo rpm -i http://repository.cloudifysource.org/cloudify/4.0.0/rc1-release/cloudify-4.0.0~rc1.el6.x86_64.rpm
You're about to install Cloudify!
Thank you for installing Cloudify!
[cloudify@cloudify ~]$ cp /opt/cfy/cloudify-manager-blueprints/simple-manager-blueprint-inputs.yaml inputs.yaml
```

Uncomment and provide the following fields:

- public_ip: '' # I recommend the IP that you SSHed using, or even localhost works.
- private_ip: '' # This should be the IP of interface, such as eth0 or eth1, that you want the manager to listen on.
- ssh_key_filename: '' # If you need to generate a new one, I recommend these (instructions)[https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent].
- ssh_user: '' # This is the SSH user that you want Cloudify to use during bootstrap. Usually this will be centos, although it depends on the VM you are using. For example, in this example my user is "cloudify".

All other fields are optional and depend on your system needs.

Now you will bootstrap:

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

```shell
$ cfy profiles use [public_ip] --profile-name [alias] -s [ssh_user] -k [ssh_key_filename] -u admin -p [manager_password] -t default_tenant
Attempting to connect...
Initializing profile azure...
Initialization completed successfully
Using manager [public_ip] with port 80
```


















