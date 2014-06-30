# globals are bad mkay
$project = File.basename(Dir.getwd)
$ipfile  = "#{ENV['HOME']}/.vagrant_ips"

def findip()
  number  = nil;

  if File.exists?($ipfile)
    existing = JSON.parse( IO.read($ipfile) )
  else
    existing = {};
  end

  number = existing[$project]

  if !number
    number = 2;

    while existing.values.include?(number) 
      number += 1;
    end

    existing[$project] = number;
  end

  IO.write($ipfile, existing.to_json);

  address = "33.33.33.#{number}"

  puts "[daftlabs] using IP address #{address}" 
  return address 
end

Vagrant.configure("2") do |config|
  config.vm.box = "opscode-ubuntu-12.04-chef11"
  config.vm.box_url = "https://opscode-vm.s3.amazonaws.com/vagrant/opscode_ubuntu-12.04_chef-11.2.0.box"

  config.vm.provision :chef_client do |chef|
    chef.chef_server_url = "https://api.opscode.com/organizations/#{$project}"
    chef.validation_key_path = "~/.ssh/#{$project}-validator.pem"
    chef.validation_client_name = "#{$project}-validator"

    chef.environment = "development"
    chef.add_recipe $project
    chef.node_name = "#{ENV['USER']}.development.#{$project}"
  end

  config.vm.network :private_network, ip: findip()
  config.vm.synced_folder "./", "/var/www/#{$project}", :mount_options => ['dmode=777,fmode=777']
  config.vm.synced_folder "./logs", "/var/log/apache2", :mount_options => ['dmode=777,fmode=777']
end
