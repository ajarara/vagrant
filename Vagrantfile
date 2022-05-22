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

  config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
    vb.memory = "16192"
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

    cat <<EOF >> /home/vagrant/.profile
GUIX_PROFILE="/home/vagrant/.config/guix/current"
. "$GUIX_PROFILE/etc/profile"
EOF

  SHELL

  # for gpg
  config.vm.provision "shell", run: 'always', inline: <<-SHELL
    # https://github.com/LudovicRousseau/PCSC/blob/615160ff2f1e6f0f0ee324f7442b0068552d0068/doc/README.polkit
    mkdir -p /usr/share/polkit-1/rules.d/
    cat <<EOF > /usr/share/polkit-1/rules.d/01-yubikey-polkit.rules
polkit.addRule(function(action, subject) {
    if (action.id == "org.debian.pcsc-lite.access_pcsc") {
        return polkit.Result.YES;
    }
    if (action.id == "org.debian.pcsc-lite.access_card") {
        return polkit.Result.YES;
    }
});
EOF
  SHELL

  # for ssh
  config.vm.provision "shell", run: 'always', inline: <<-SHELL
    mkdir -p /etc/udev/rules.d
    cat <<EOF > /etc/udev/rules.d/01-yubikey-udev.rules
# this udev file should be used with udev 188 and newer
ACTION!="add|change", GOTO="u2f_end"

# Yubico YubiKey
KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{idVendor}=="1050", ATTRS{idProduct}=="0113|0114|0115|0116|0120|0121|0200|0402|0403|0406|0407|0410", TAG+="uaccess", GROUP="vagrant", MODE="0660"

LABEL="u2f_end"
EOF
  udevadm trigger
  SHELL

  config.vm.provision "shell", inline: <<-SHELL
    dnf install -y ccid
    systemctl enable pcscd
  SHELL

  config.vm.provision "shell", run: 'always', inline: <<-SHELL
    systemctl restart pcscd
  SHELL

  config.vm.provision "shell", run: 'always', inline: <<-SHELL
    mkdir -p /home/vagrant/.gnupg/
    echo "pinentry-program /usr/bin/pinentry-tty" > /home/vagrant/.gnupg/gpg-agent.conf
    chown -R vagrant:vagrant /home/vagrant/.gnupg
  SHELL

  config.vm.provision "shell", run: 'always', inline: <<-SHELL
    cat <<EOF > /home/vagrant/.profile
gpg-connect-agent reloadagent /bye
sudo pkill ssh-agent
eval \`ssh-agent\`
EOF
    chown vagrant:vagrant /home/vagrant/.profile
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
