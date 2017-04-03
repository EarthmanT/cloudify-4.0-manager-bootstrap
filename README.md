# cloudify-4.0-manager-bootstrap

## Management Environment.

We assume that you have existing Cloud infrastructure that you want to manage with Cloudify, including a specific network/VPC/etc.

If that assumption is false, we have several example infrastructure blueprints that you can use to deploy your Cloudify Manager. To use these examples, first download and extract the example repository:

```shell
$ curl -L https://github.com/cloudify-examples/aws-azure-openstack-blueprint/archive/4.0.tar.gz | tar xv
```

#### AWS Infrastructure Installation

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

When the installation is completed, you can see the resources that were created.

```shell
$ cfy deployments outputs
...
{
...
  "example_aws_private_subnet":
    "example_aws_private_subnet": "subnet-20e90f69"
  },
...
...
  "example_aws_public_subnet":
    "example_aws_public_subnet": "subnet-b7ed0bfe"
  },
...
  "example_aws_vpc": {
    "example_aws_vpc": "vpc-644d5400"
  }
...
}
```

To get information about only one resource, you can run:

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

*The value of the _example_aws_elastic_ip_ is the IP that you will use to install your Cloudify Manager in the Bootstrap phase.*


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


















Screening Questions:

Questions to answer before installing your Cloudify 4.0 manager:
Does the VM that I want to install Cloudify Manager on already exist?
Do I already have an environment that I want to install my Cloudify Manager on?
Can I use a pre-baked image?

Questions to answer after installing your Cloudify 4.0 manager:
Do I have an existing Cloudify Manager that I want to upgrade?
Do I have blueprints from an earlier Cloudify version that I want to use?
