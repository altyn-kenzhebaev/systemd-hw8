# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
    :systemd => {
        :box_name => "centos/7",
        :cpus => 1,
        :memory => 512,
    },
}

Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
      config.vm.define boxname do |box|
          box.vm.box = boxconfig[:box_name]
          box.vm.host_name = boxname.to_s 
          box.vm.provider :virtualbox do |vb|
            vb.memory = boxconfig[:memory]
            vb.cpus = boxconfig[:cpus] 	        
          end
      end
  end
end
