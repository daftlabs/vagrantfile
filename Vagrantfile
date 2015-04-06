require 'json'

# globals are bad mkay
$project = File.basename(Dir.getwd)
$ipfile  = "#{ENV['HOME']}/.vagrant_ips"

def getexisting()
  if File.exists?($ipfile)
    existing = JSON.parse( IO.read($ipfile) )
  else
    existing = {};
  end

  return existing;
end

def findprojectid()
  existing = getexisting()

  number = existing[$project]

  if !number
    number = 2;

    while existing.values.include?(number) 
      number += 1;
    end

    existing[$project] = number;
    IO.write($ipfile, existing.to_json);
  end

  return number;
end

def findip()
  number  = findprojectid()
  address = "33.33.33.#{number}"

  puts "[daftlabs] using IP address #{address}" 
  return address 
end

Vagrant.configure("2") do |config|
  config.vm.box = "opscode-ubuntu-14.04"
  config.vm.box_url = "http://opscode-vm-bento.s3.amazonaws.com/vagrant/virtualbox/opscode_ubuntu-14.04_chef-provisionerless.box"
  config.omnibus.chef_version = '11.16.0'

  config.vm.provider "virtualbox" do |box|
    box.memory = 1024
    box.cpus = 1
  end

  config.vm.provision "chef_zero" do |chef|
    chef.log_level         = :debug
    chef.environment       = "development"
    chef.cookbooks_path    = "cookbooks/"
    chef.environments_path = "environments/"
    chef.json              = JSON.parse( IO.read("attributes/default.json") )
    chef.data_bags_path    = "data_bags/"
  end

  config.vm.network :private_network, ip: findip()
  config.vm.synced_folder "./", "/var/www/#{$project}", :mount_options => ['dmode=777,fmode=777']
end
