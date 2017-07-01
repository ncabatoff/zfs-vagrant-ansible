# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'pathname'

jessie_host = "jessie"
xenial_host = "xenial"
fedora25_host = "fedora25"

gopath = ENV['GOPATH']
current_dir = File.dirname(File.expand_path(__FILE__))
parent_dir = File.dirname(current_dir)
parent_path = Pathname.new(parent_dir)
pkgname = parent_path.relative_path_from(Pathname.new(gopath)).to_path()

Vagrant.configure("2") do |config|
  config.vm.define :fedora25 do |fedora25|
    fedora25.vm.box = "fedora/25-cloud-base"
    fedora25.vm.hostname = fedora25_host
    # fedora25.vm.network "private_network", ip: "10.0.2.20"
    fedora25.vm.network "private_network", type: "dhcp"
    fedora25.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      # If we're running vagrant up after having halted a successfully initialized fedora25,
      # adding the controller will fail.  Removing the controller prevents that.  This does
      # mean that the first time 
      for i in 0 ... 4
        diskfile = fedora25_host + "_dsk" + i.to_s
        diskfilevdi = diskfile + ".vdi"
        unless File.exist?(diskfilevdi)
          if i == 0
            vb.customize ["storagectl", :id, "--name", "SATA Controller", "--add", "sata"]
          end
          vb.customize ["createhd",  "--filename", diskfile, "--size", "100"]
                                        vb.customize ["storageattach", :id, "--storagectl", "SATA Controller", "--port", (i+1).to_s, "--type", "hdd", "--medium", diskfilevdi]
        end
      end
    end

  end

  config.vm.define :jessie do |jessie|
    jessie.vm.box = "debian/jessie64"
    jessie.vm.hostname = jessie_host
    jessie.vm.network "private_network", type: "dhcp"
    jessie.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      for i in 0 ... 4
        diskfile = jessie_host + "_dsk" + i.to_s
        diskfilevdi = diskfile + ".vdi"
        unless File.exist?(diskfilevdi)
          vb.customize ["createhd",  "--filename", diskfile, "--size", "100"]
                                        vb.customize ["storageattach", :id, "--storagectl", "SATA Controller", "--port", (i+1).to_s, "--type", "hdd", "--medium", diskfilevdi]
        end
      end
    end

  end

# Still having some issues with Ubuntu box
# config.vm.define :xenial do |xenial|
#   xenial.vm.box = "ubuntu/xenial64"
#   xenial.vm.hostname = xenial_host
#   xenial.vm.provider "virtualbox" do |vb|
#     vb.memory = "1024"
#     for i in 0 ... 4
#       diskfile = xenial_host + "_dsk" + i.to_s
#       diskfilevdi = diskfile + ".vdi"
#       unless File.exist?(diskfilevdi)
#         vb.customize ["createhd",  "--filename", diskfile, "--size", "100"]
#                               vb.customize ["storageattach", :id, "--storagectl", "SCSI", "--port", (i+1).to_s, "--type", "hdd", "--medium", diskfilevdi]
#       end
#     end
#   end
# end

  config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vm.synced_folder "..", "/root/" + pkgname,
    create: true, type: "rsync", rsync__args: [
      "--verbose", "--rsync-path='sudo rsync'", "--archive", "--delete", "-z",
      "--exclude", "**.vdi"] #,rsync__verbose: true

  # Bootstrap ansible
  config.vm.provision "init", type: "ansible" do |ansible|
    ansible.playbook = "playbook-init.yml"
    ansible.sudo = true
    #ansible.verbose = '-v'
  end

  # Install ZFS itself and load it.  
  config.vm.provision "zfs", type: "ansible" do |ansible|
    ansible.playbook = "playbook-zfs.yml"
    ansible.sudo = true
    #ansible.verbose = '-v'
    ansible.host_vars = {
      # Since the Fedora25 and Xenial boxes have their disks on IDE/SCSI controllers,
      # the devices we add to a SATA controller come up as sda..sdd, whereas on the
      # Jessie box the root disk comes on a SATA controller and uses sda,
      # leaving us sdb..sde.
      jessie_host => { "zdisk1" => "sdb", "zdisk2" => "sdc", "zdisk3" => "sdd", "zdisk4" => "sde" },
      xenial_host => { "zdisk1" => "sdc", "zdisk2" => "sdd", "zdisk3" => "sde", "zdisk4" => "sdf" },
      fedora25_host => { "zdisk1" => "sda", "zdisk2" => "sdb", "zdisk3" => "sdc", "zdisk4" => "sdd" },
    }
  end
end
