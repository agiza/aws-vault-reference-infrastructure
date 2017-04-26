# A reference architecture for [Hashicorp Vault](https://github.com/hashicorp/vault) on AWS
This project aims to show how to build an secure and resilient impementation of Vault on the AWS platform, consumed by a simple web application that includes some good security coding practises.
Hopefully this will help you get the best out of your Vault out of your projects. The project is licensed under the Apache v2.0 license, so you are free to use or modify it however you like, but we don't offer any kind of warranty!

## Overview ##

Vault is a good fit for a few use cases. The main one we are looking at is for a running application. This solves a few problems:

* It protects agains the insider threat of rogue admins/operators abusing DB credentials. This is a hard issue to solve in traditional ways and in the end the mitigation comes down to auditing DB logs for unusual activity. That is obviously a bit late because the damage has already been done by the time you look at the logs..
* It automates credential rotation, making it easy to comply with your crypto key management best practises
* It removes the need to store credentials in potentially insecure config files or similar

Here is the overall architecture:

![Architecure Overview](/images/Vault Reference Architecture.png)

Some key features:

* If the Vault fails, any dependent application fails too, so it is multi AZ and multi-server in each AZ
* The storage backend must also support HA (we have chosen [DynamoDB](https://www.vaultproject.io/docs/config/#dynamodb))
* The [MySQL secret backend](https://www.vaultproject.io/docs/secrets/mssql/index.html) generates credentials on the fly so it needs access to the DB. We are using VPG Gateways to lock down access to the DB
* We are using the [AWS EC2 Auth backend](https://www.vaultproject.io/docs/auth/aws-ec2.html) so that we can make use of [IAM roles](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) to control access to the Vault

## Infrastructure ##

To do:

* Network
* Components
* Deployment/CloudFormation

## Vault config ##

To do: 

* Initialisation
* Storage backend
* Audit backend
* MySQL secret backend

## IAM policies ##

To do:

* Normal privilege
* High privilege

## The client application ##

To do:

* Making the app "vault aware"
* Dynamic privilege elevation

## About the authors ##
We are not affiliated to Hashicorp or the Vault project in any way and any recommendations made are our own and not endorsed by Hashicorp. We just like Vault, AWS, security and messing about with cool tech :-)

---
## Installation Prerequisites

There are a couple of things you need to satisfy before you can deploy the Vault Reference Infrastructure in your own AWS environment;

* **An AWS Account** - You can set up an account by following the instructions listed [here](https://aws.amazon.com/account/). At the moment one account will suffice, but the final deployment will utilise two separate AWS accounts with one VPC in each.  The account creation is the only thing that cannot be automated.
* **AWS CLI** - This deployment uses the CLI to automate the build process, so you will need to install and configure this before attempting the deploy.  Full instructions to install the CLI tools are located [here](http://docs.aws.amazon.com/cli/latest/userguide/installing.html), with configuration instructions located [here](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html).

## Installation Files

The files required for the installation (and are located in this GitHub repository) are listed below;

| File Name | Description |
|-----------|-------------|
| setup.sh  |  A Bash script which makes the set up of variables much easier.  It's been tested on Linux but should also work on all macOS and OSX versions.  A Powershell script will also be added for ease of Installation on Windows machines.|
|vault-core-networking.json | This is the first Cloudformation template which generates all of the basic networking required (VPCs, subnets and gateways.)  |
|vault-core-networking-parameters.json | This file contains all of the parameters which you will need to change to deploy the system within your AWS infrastructure.  These include the VPC CIDR address, Choice of AWS Region Availability Zones, Subnet CIDR addresses and your IP address for connecting into the Vault servers if required.   |
|vault-core-security.json | The Cloudformation template that handles Security Groups and Network ACLs. |
|vault-core-instances.json | The Cloudformation template which generates the EC2 instances, AutoScaling Groups and ELBs.|
|config.hcl | Basic config file for DynamoDB creation.  You will only need to change the AWS Region to where you require DynamoDB to be created (The default is EU-West-1.) |
|supervisord.conf | This is the Supervisord config file which allows Vault to run automatically upn deploy.|
|supervisord | The init.d script for Supervisord to start. |

## Installation Process

Once you have created the AWS account, you will need to create an IAM user (with ability to run the Cloudformation templates and create AWS services) and also download the associated SSH key (which is used to SSH onto the Bastion and Vault Servers.)

You will have two files located in the <code>~/.aws/</code> directory called <code>config</code> and <code>credentials</code>.  The contents of the <code>credentials</code> file will look similar to;

<pre><code>[default]
aws_access_key_id = AKIGHSJJKKKLKLLAKHJK
aws_secret_access_key = k/wKqksafjejfhkhadflkjlixjklqnekdm;lw8Xd
</code></pre>

You don't need to know anything in this file, other than what is between the square brackets, in this case <code>[default]</code>.  This is the profile name and will be used later when the setup script is run.

The next step is to clone the GitHub repository to your local machine, using the following command;

<code>
https://github.com/mike-goodwin/aws-vault-reference-infrastructure.git
</code>

Locate the file called <code>setup.sh</code> (you may need to set the file executable bit using <code>chmod +x setup.sh</code>) and then run it.  You will see the following;

<pre>---------------------------------------------------------------------------------------
AWS VAULT REFERENCE INSTALLATION
---------------------------------------------------------------------------------------
Select from the following options...

<pre>1. VALIDATE CLOUDFORMATION TEMPLATES
2. BUILD CLOUDFORMATION STACK
0. Quit

Enter Selection [0-2]</pre></pre>

Press 1, if you want to check that all of the templates downloaded are syntactically correct before deploying.

To deploy the whole Cloudformation Stack, press 2 and you will see;

<pre>---------------------------------------------------------------------------------------
BUILD CLOUDFORMATION STACK
---------------------------------------------------------------------------------------
Enter the name of the AWS profile you wish to use (Leave blank for 'default' profile)...

default

Enter the name of the Cloudformation Stack...

Vault Reference Stack</pre>

Type the name of the profile in the first field (the one in the square brackets of the credentials file) and in the second field, type in any name or phrase you wish to call the Cloudformation Stack.  The Stack will then build (this will take some time) and providing everything goes well, the Vault side of the infrastructure will be built and will be running.








