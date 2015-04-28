require 'json'
require 'rbconfig'

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
  config.vm.synced_folder "./", "/var/www/#{$project}", :mount_options => ['dmode=777,fmode=777']

  config.omnibus.chef_version = '11.16.0'

  config.vm.provider "virtualbox" do |box, override|
    box.memory = 1024
    box.cpus = 1
    
    if RbConfig::CONFIG['host_os'] =~ /mswin|win|mingw/
      # Allows long filenames in Windows
      # See: https://github.com/mitchellh/vagrant/issues/1953
      box.customize ["sharedfolder", "add", :id, "--name", "project", "--hostpath", (("//?/" + File.dirname(__FILE__)).gsub("/","\\"))]
      
      # Allow symlinks (Run `vagrant up` from a CMD Prompt with Admin Privs)
      box.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/project", "1"]
      
      # Dissable the "default" Synced Folder mount (via Vagrant) in favor of VBox
      override.vm.synced_folder "./", "/var/www/#{$project}", disabled: true
      
      # This is the default way the folder is typically created
      override.vm.provision :shell, inline: "mkdir /var/www/#{$project} -p"
      override.vm.provision :shell, inline: "mount -t vboxsf -o uid=`id -u vagrant`,gid=`getent group vagrant | cut -d: -f3` project /var/www/#{$project}", run: "always"
    end
    
    # Ensures this only executes on VirtualBox
    override.vm.provision "chef_solo" do |chef|
      json = JSON.parse( IO.read("attributes/default.json") )
      json.store('project', $project)
      
      chef.custom_config_path = "./vendor/vagrantfile/Vagrantfile.chef"
      chef.environment        = "development"
      chef.cookbooks_path     = "cookbooks/"
      chef.environments_path  = "environments/"
      chef.json               = json
    end
    
  end
  
  config.vm.network :private_network, ip: findip()
  
end
