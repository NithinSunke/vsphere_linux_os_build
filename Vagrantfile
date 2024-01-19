require 'yaml'
params = YAML.load_file 'config/vagrant.yml'
var_box            = params['box']
var_vm_name        = params['vm_name']
var_esxi_hostname  = params['esxi_hostname']
var_esxi_username  = params['esxi_username']
var_esxi_password  = params['esxi_password']
var_esxi_hostport  = params['esxi_password']
var_esxi_disk_store =  params['esxi_disk_store']
var_public_ip      =  params['public_ip']
var_guest_name     = params['guest_name']
var_guest_memsize  = params['guest_memsize']
var_guest_numvcpus = params['guest_numvcpus']
var_guest_disk_type = params['guest_disk_type']
var_guest_guestos   = params['guest_guestos']
var_esxi_virtual_network = params['esxi_virtual_network']

Vagrant.configure("2") do |config|
  config.vm.box = var_box
  config.vm.hostname = var_vm_name
  config.vm.provider :vmware_esxi do |esxi|
    esxi.esxi_hostname = "192.168.0.13"
    esxi.esxi_username = "nsunke"
    esxi.esxi_password = "sns#0712"
    esxi.esxi_hostport = var_esxi_hostport
    esxi.esxi_virtual_network = var_esxi_virtual_network
    esxi.esxi_disk_store = var_esxi_disk_store
    esxi.guest_name = var_guest_name
    esxi.guest_memsize = var_guest_memsize
    esxi.guest_numvcpus = var_guest_numvcpus
    esxi.guest_disk_type = 'thin'
    esxi.guest_guestos = var_guest_guestos
  end
  config.vm.provision "shell", inline: <<-SHELL
    yum install git -y
    cd /tmp
    git clone https://github.com/NithinSunke/esxi_vagrant_centos8.git
    chmod -R 777 vagrant
    .  /tmp/esxi_vagrant_centos8/config/install_env.sh
    echo "******************************************************************************"
    echo "Set root and oracle password and change ownership of /u01." `date`
    echo "******************************************************************************"
    echo -e "${ROOT_PASSWORD}\n${ROOT_PASSWORD}" | passwd
    groupadd -g 5003 admin
    useradd -u 6001 -g admin -G wheel nsunke
    usermod -aG admin vagrant 
    echo -e "${NSUNKE_PASSWORD}\n${NSUNKE_PASSWORD}" | passwd nsunke
    echo "%admin         ALL=(ALL)       NOPASSWD: ALL" >>   /etc/sudoers

    echo "modifing sshd_config file"
    echo "+++++++++++++++++++++++"
    mv /etc/ssh/sshd_config /etc/ssh/sshd_config_org
    cp /tmp/esxi_vagrant_centos8/config/sshd_config /etc/ssh/sshd_config
    systemctl restart sshd
    cd /tmp/esxi_vagrant_centos8/config
    cp /etc/sysconfig/network-scripts/ifcfg-eth0 /etc/sysconfig/network-scripts/ifcfg-eth0_org
    echo "$(sed "s/#IPADDRESS#/$public_ip/g" ifcfg-eth0)" > /etc/sysconfig/network-scripts/ifcfg-eth0
    cd /tmp
    rm -rf vagrant 
    init 6
  SHELL
end
