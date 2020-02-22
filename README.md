# Amazon Web Services OpenVPN Server: Vagrant + Ansible

My attempt at building up an Amazon Web Services instance to host an OpenVPN
server. The goal is to be able to quickly build up and tear down the server
when needed, and let Ansible handle the provisioning each time. Vagrant will
spin an Amazon Web Services instance of choice, assign it an Elastic IP
address, and pass the provisioning to Ansible. The Ansible playbook will
configure the AWS instance to run an openvpn server, and issue certificates for
the user. 

<!-- GETTING STARTED -->
## Getting Started
These instructions will get you a copy of the project up and running on your
local machine for development and testing purposes. 

<!-- PREREQS -->
### Prerequisites
The following are required for setup. See the accompanying resources to install/configure on your respective environment: 
- [Vagrant][1]
    - [vagrant-aws plugin][2]
- [Ansible][3]
- [Amazon Web Services Account][4]

[1]: https://www.vagrantup.com/docs/installation/
[2]: https://github.com/mitchellh/vagrant-aws
[3]: https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html
[4]: https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/

### Amazon Web Services
A couple of things are required from the AWS console prior to starting. Once an AWS account is setup:
- Grab your [AWS Access Key and AWS Secret Access Key][5], as these are required for Vagrant to authenticate to your AWS account.
- Assign your account an [elastic IP address][7], so we won't have to update the Ansible configurations every time we create a new instance.
- Add a [security group][8] that allows SSH inbound traffic (Ansible), and UDP Port 1194 (OpenVPN). If you want an added bit of security, you can whitelist your
public IP address, but reconfiguring the security group will be necessary if
you ever change physical locations. 
- Lastly, this project uses an Ubuntu 18.04 LTS image. If you would like to tweek the playbook for a different Linux distribution, it is possible to feed Vagrant any [AMI you desire][9]. 

[5]: https://aws.amazon.com/blogs/security/wheres-my-secret-access-key/
[6]: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html
[7]: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html
[8]: https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html
[9]: https://aws.amazon.com/amazon-linux-ami/

### Vagrant Setup
For an added bit of security, export the following environment variables to be
loaded by the Vagrantfile:
```
export AWS_ACCESS_KEY_ID="YOUR ACCESS KEY"
export AWS_ELASTIC_IP="YOUR ELASTIC IP ADDRESS"
export AWS_SECRET_ACCESS_KEY="YOUR SECRET ACCESS KEY"
```

The quickest way to get started is to follow the instructions for the Vagrant
AWS plugin instructions by usying a dummy AWS box:
```
$ vagrant box add dummy https://github.com/mitchellh/vagrant-aws/raw/master/dummy.box
```

Additionally, ensure that your keypair path, region, security group name, and
default ssh username are all correct in the Vagrantfile.

Note the default Vagrantfile uses the free t2.micro instance. 

### Ansible Setup
Insure that your inventory file has your Elastic IP address, and private key
path correctly identified.
```
openvpn ansible_ssh_host="YOUR ELASTIC IP ADDRESS" ansible_ssh_port=22 ansible_ssh_user='ubuntu' ansible_ssh_private_key_file='~/.aws/.ssh/aws-openvpn.pem'
``` 

<!-- Usage Examples -->
## Usage
Once configurations are all complete, simply use the following command to
create the AWS instance:
```
vagrant up --provider=aws
```

## OpenVPN Client
The Ansible playbook will grant certificates in the
provision/roles/easyrsa/files/fetched directory. It's important that the
following files are all in the same directory to connect to the OpenVPN server:
- client.key
- client.crt
- client.csr
- client.opvpn
- ca.crt
- ta.key

*SECURITY NOTE:* It is generally bad practice to issue certificates on the same
OpenVPN server. In this test case, it is not that big of an issue, as it is
easily available to tear down the OpenVPN instance, and new certificates will
be assigned after each rebuild. A possible solution would be to have Vagrant
spin up a virtual machine, and generate the certificate and store them locally.

Lastly, it's required to configure an OpenVPN client to actually connect to the
running OpenVPN server. There are different clients available that are OS
dependent, but there are many guides that will guide you through the [client
installation][10].

[10]: https://www.linode.com/docs/networking/vpn/configuring-openvpn-client-devices/

For a quick check if you are connecting to your OpenVPN server, do a quick
google search of your public IP address to confirm it is your Elastic IP
address.

### References
[Set up a Hardened OpenVPN Server on Debian 9](https://www.linode.com/docs/networking/vpn/set-up-a-hardened-openvpn-server/)\
[Vagrant Documentation](https://www.vagrantup.com/docs/)\
[Ansible Documentation](https://docs.ansible.com/)
