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
  config.vm.box = "opscode-ubuntu-12.04-chef11"
  config.vm.box_url = "https://opscode-vm.s3.amazonaws.com/vagrant/opscode_ubuntu-12.04_chef-11.2.0.box"
  #config.vm.network "forwarded_port", guest: "80", host: "808#{findprojectid()}"

  config.vm.provider "virtualbox" do |box|
    box.memory = 1024
    box.cpus = 1
  end

  config.vm.provision "chef_solo" do |chef|
    chef.node_name = "development"
    chef.add_recipe $project

    if File.file?("config.json") then
      chef.json = JSON.parse( IO.read("config.json") )
    end
  end

  config.vm.network :private_network, ip: findip()
  config.vm.synced_folder "./", "/var/www/#{$project}", :mount_options => ['dmode=777,fmode=777']
end
