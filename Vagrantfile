# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

# Default password wherever it required.
PASSWORD = 'secret'

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.

  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = "opscode_ubuntu-12.04_provisionerless"

  # The url from where the 'config.vm.box' box will be fetched if it
  # doesn't already exist on the user's system.
  config.vm.box_url = "https://opscode-vm-bento.s3.amazonaws.com/vagrant/opscode_ubuntu-12.04_provisionerless.box"

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  [3000].each do |port|
    config.vm.network :forwarded_port, guest: port, host: port
  end

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  config.vm.network :private_network, ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network :public_network

  # If true, then any SSH connections made will enable agent forwarding.
  # Default value: false
  config.ssh.forward_agent = true

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  { '~/Code' => '/code' }.each do |from, to|
    # config.vm.synced_folder from, to, nfs: !(RUBY_PLATFORM =~ /mingw32/)
  end

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider :virtualbox do |vb|
  #   # Don't boot with headless mode
  #   vb.gui = true
  #
  #   # Use VBoxManage to customize the VM. For example to change memory:
    vb.customize ["modifyvm", :id, "--memory", "1024", "--cpus", "2"]
  end
  #
  # View the documentation for the provider you're using for more
  # information on available options.

  # Enable provisioning with Puppet stand alone.  Puppet manifests
  # are contained in a directory path relative to this Vagrantfile.
  # You will need to create the manifests directory and a manifest in
  # the file base.pp in the manifests_path directory.
  #
  # An example Puppet manifest to provision the message of the day:
  #
  # # group { "puppet":
  # #   ensure => "present",
  # # }
  # #
  # # File { owner => 0, group => 0, mode => 0644 }
  # #
  # # file { '/etc/motd':
  # #   content => "Welcome to your Vagrant-built virtual machine!
  # #               Managed by Puppet.\n"
  # # }
  #
  # config.vm.provision :puppet do |puppet|
  #   puppet.manifests_path = "manifests"
  #   puppet.manifest_file  = "init.pp"
  # end

  # Set ulimit before running chef-solo (source: http://bit.ly/1cV6w1g)
  config.vm.provision :shell, inline: 'ulimit -n 4048'

  # Don't install gem documentation anymore!
  config.vm.provision :shell, inline: %{
    if [ ! -f /home/vagrant/.gemrc ]; then
      echo "gem: --no-document" > /home/vagrant/.gemrc
    fi
  }

  # Enable provisioning with chef solo, specifying a cookbooks path, roles
  # path, and data_bags path (all relative to this Vagrantfile), and adding
  # some recipes and/or roles.
  #
  config.vm.provision :chef_solo do |chef|
    chef.cookbooks_path = %w(cookbooks)
  #   chef.roles_path = "../my-recipes/roles"
  #   chef.data_bags_path = "../my-recipes/data_bags"
    chef.add_recipe "apt"
    chef.add_recipe "git"
    chef.add_recipe "grc"
    chef.add_recipe "imagemagick::rmagick"
    chef.add_recipe "oh_my_zsh"
    chef.add_recipe "postgresql::server"
    chef.add_recipe "postgresql::contrib"
    chef.add_recipe "postgresql::libpq"
    chef.add_recipe "mysql::server"
    chef.add_recipe "redis::install"
    chef.add_recipe "mongodb::10gen_repo"
    chef.add_recipe "mongodb::default"
    chef.add_recipe "sphinx"
    chef.add_recipe "ruby_build"
    chef.add_recipe "rbenv::user"
    chef.add_recipe "nodejs"
    chef.add_recipe "nodejs::npm"
    chef.add_recipe "nfs" # Last because of problems with restarting :(
  #   chef.add_role "web"
  #
  #   # You may also specify custom JSON attributes:
    chef.json = {
      postgresql: {
        users: [
          {
            username: 'vagrant',
            password: PASSWORD,
            superuser: true,
            createdb: true,
            login: true
          }
        ]
      },
      mysql: {
        server_debian_password: PASSWORD,
        server_root_password: PASSWORD,
        server_repl_password: PASSWORD,
        allow_remote_root: true
      },
      rbenv: {
        user_installs: [
          {
            user: 'vagrant',
            rubies: %w(2.1.0),
            global: '2.1.0',
            gems: {
              '2.1.0' => %w(
                bundler pry pg mysql2 spring
              ).map { |g| Hash[:name, g] }
            }
          }
        ]
      },
      oh_my_zsh: {
        users: [
          {
            login: 'vagrant',
            theme: 'ys',
            plugins: %w(git gem bundler rails)
          }
        ]
      },
      sphinx: {
        use_mysql: true,
        use_postgres: true,
        use_stemmer: true
      }
    }
  end

  # Enable provisioning with chef server, specifying the chef server URL,
  # and the path to the validation key (relative to this Vagrantfile).
  #
  # The Opscode Platform uses HTTPS. Substitute your organization for
  # ORGNAME in the URL and validation key.
  #
  # If you have your own Chef Server, use the appropriate URL, which may be
  # HTTP instead of HTTPS depending on your configuration. Also change the
  # validation key to validation.pem.
  #
  # config.vm.provision :chef_client do |chef|
  #   chef.chef_server_url = "https://api.opscode.com/organizations/ORGNAME"
  #   chef.validation_key_path = "ORGNAME-validator.pem"
  # end
  #
  # If you're using the Opscode platform, your validator client is
  # ORGNAME-validator, replacing ORGNAME with your organization name.
  #
  # If you have your own Chef Server, the default validation client name is
  # chef-validator, unless you changed the configuration.
  #
  #   chef.validation_client_name = "ORGNAME-validator"

  config.vm.provision :shell, inline: %{
    if [ $(date +%Z) != 'MSK' ]; then
      echo "Europe/Moscow" | sudo tee /etc/timezone \
      && dpkg-reconfigure --frontend noninteractive tzdata
    fi
  }

  config.vm.provision :shell, inline: %{
    if [ ! -f /etc/ssl/certs/cacert.pem ]; then
      wget -O /etc/ssl/certs/cacert.pem \
      http://curl.haxx.se/ca/cacert.pem
    fi
  }

  config.vm.provision :shell, inline: %{
    if [ -n $SSL_CERT_FILE ]; then
      echo "export SSL_CERT_FILE=/etc/ssl/certs/cacert.pem" >> /home/vagrant/.zshrc
    fi
  }

  config.omnibus.chef_version = :latest
end
