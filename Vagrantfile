$script = <<SCRIPT
  echo I am provisioning...
  echo cd "/project" > ~/.bash_profile
  echo cd "/project" > ~/.bashrc
  sudo chown -R vagrant ~/.rbenv/versions/2.1.0/lib/ruby/gems/2.1.0

  sudo apt-get -y install redis-server imagemagick
  redis-server

  cd /project
  find config -name '*.example' | sed 's/.example//' | xargs -I{} cp -v {}.example {}
   bundle
  rake db:create
  rake db:migrate
  rake db:seed_fu
  rbenv rehash
  rails s
SCRIPT



Vagrant.configure("2") do |config|
  config.vm.box = "parallels/ubuntu-12.04"

  config.vm.network :private_network, ip: "192.168.50.4"
  config.vm.synced_folder Dir.pwd, "/project", :nfs => true
  config.ssh.forward_agent = true


  [3000].each do |port|
    config.vm.network :forwarded_port, guest: port, host: port
  end

  # config.vm.provision :shell, inline: $script_before, privileged: false

  config.vm.provision :chef_solo do |chef|
    chef.cookbooks_path = ["cookbooks"]
    chef.add_recipe "apt"
    chef.add_recipe 'git'
    chef.add_recipe 'postgresql::server'
    chef.add_recipe 'postgresql::contrib'
    chef.add_recipe 'postgresql::server_dev'

    chef.add_recipe 'ruby_build'
    chef.add_recipe 'rbenv::user'
    chef.add_recipe 'oh_my_zsh'
    chef.json = {
      'postgresql' => {
        "version" => "9.2",
        "users" => [
          {
            "username" => "vagrant",
            "password" => '',
            "superuser" => true,
            "createdb" => true,
            "login" => true
          }
        ]
      },
      'rbenv' => {
        'user_installs' => [
          { 'user' => 'vagrant',
            'rubies' => ['2.1.0'],
            'global' => '2.1.0',
            'gems' => {
              '2.1.0' => %w(bundler pg).collect{|gem_name| { 'name' => gem_name } }
            }
          }
        ]
      },
      'oh_my_zsh' => {
        'users' => [{
          :login => 'vagrant',
          :theme => 'mortalscumbag',
          :plugins => ['gem', 'git', 'rails3', 'redis-cli', 'ruby', 'heroku', 'rake', 'rbenv', 'capistrano']
        }]
      }
    }
  end

  config.vm.provision 'shell', inline: $script, privileged: false
end
