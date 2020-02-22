#Require the AWS provider plugin
require 'vagrant-aws'

# Quick fix for the Vagrant 2.2.7 version. See Vagrant Issue #566 for details
# https://github.com/mitchellh/vagrant-aws/issues/566
class Hash
  def slice(*keep_keys)
    h = {}
    keep_keys.each { |key| h[key] = fetch(key) if has_key?(key) }
    h
  end unless Hash.method_defined?(:slice)
  def except(*less_keys)
    slice(*keys - less_keys)
  end unless Hash.method_defined?(:except)
end

Vagrant.configure('2') do |config|
  config.vm.box = 'dummy'
  
  # Note on MacOS Catalina, user is prompted for SMB credentials. Vagrant
  # attempts to sync folders based on the capabilities of the host and guest.
  # To force rsync:
  config.vm.allowed_synced_folder_types = [:rsync]

  #Specify AWS configuration. For security, importing as environ variables
  config.vm.provider :aws do |aws, override|
    
    # AWS authentication information
    aws.access_key_id = ENV['AWS_ACCESS_KEY_ID']
    aws.secret_access_key = ENV['AWS_SECRET_ACCESS_KEY']
    
    # SSH Configuration 
    aws.keypair_name = 'aws-openvpn'
    #override.ssh.username = 'ec2-user' # Amazon Linux 2
    override.ssh.username = 'ubuntu'
    override.ssh.private_key_path = '~/.aws/.ssh/aws-openvpn.pem'
   
    # Specify region, AMI ID, and security groups 
    aws.region = 'us-east-1'
    #aws.ami = 'ami-062f7200baf2fa504' # Amazon Linux 2
    aws.ami = 'ami-046842448f9e74e7d' # Ubuntu 18.04 LTS amd64 hvm:ebs-ssd
    aws.security_groups = ['vagrant-openvpn']

    # Specific intance 
    aws.instance_type = 't2.micro'
    
    # Networking Configurations
    aws.elastic_ip = ENV['AWS_ELASTIC_IP']
  end

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "provision/playbook.yml"
    ansible.limit = 'all'
    ansible.inventory_path = "provision/staging/inventory"
 
    ansible.verbose = true
  end

end
