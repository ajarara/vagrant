# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "bento/fedora-34"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
    vb.memory = "8096"
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # pull in the foreign guix script
  config.vm.provision "shell", inline: <<-SHELL
    cd /tmp
    wget https://git.savannah.gnu.org/cgit/guix.git/plain/etc/guix-install.sh
    chmod +x guix-install.sh
  SHELL

  # the installer for guix on top of foreign distributions doesn't have non-interactive support
  # so we pull in expect and have it press the buttons :)
  config.vm.provision "shell", inline: <<-SHELL
    dnf install -y expect
  SHELL

  # start off the puppet to  install guix 
  config.vm.provision "shell", path: "guix-puppet"  


  config.vm.provision "shell", inline: <<-SHELL
    # kind of surprised we have to do this, we went through the guix-install script.
    guix archive --authorize < /var/guix/profiles/per-user/root/current-guix/share/guix/berlin.guix.gnu.org.pub
    guix archive --authorize < /var/guix/profiles/per-user/root/current-guix/share/guix/ci.guix.gnu.org.pub
    guix archive --authorize < /var/guix/profiles/per-user/root/current-guix/share/guix/ci.guix.info.pub

    guix pull

    echo <<EOF >> /home/vagrant/.profile
GUIX_PROFILE="/home/vagrant/.config/guix/current"
. "$GUIX_PROFILE/etc/profile"
EOF

  SHELL

  config.vm.provider :virtualbox do |vb|

    # adopted from https://github.com/Yubico/yubico-piv-tool/blob/master/vagrant/development/Vagrantfile#L25-L40
    FILTER_NAME="YubiKey 5"
    MANUFACTURER="Yubico"
    VENDOR_ID="0x1050"
    PRODUCT_ID="0x0406"
  
    vb.customize ['modifyvm', :id, '--usb', 'on']
    vb.customize ['usbfilter', 'add', '0',
                  '--target', :id,
                  '--name', FILTER_NAME,
                  '--manufacturer', MANUFACTURER,
                  '--vendorid', VENDOR_ID,
                  '--productid', PRODUCT_ID]
  end 

end
