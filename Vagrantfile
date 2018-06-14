# -*- mode: ruby -*-
# vi: set ft=ruby :
# Copyright (c) 2018 Armin Ranjbar Daemi @rmin
# Licensed under the MIT License

nodes_num = ENV.fetch('SWARM_NODES', 3).to_i
node_memory = ENV.fetch('SWARM_MEMORY', 512).to_i
node_cpus = ENV.fetch('SWARM_CPU', 1).to_i
ip_prefix = ENV.fetch('SWARM_IPPREFIX', "192.168.80.")
node_worker = ENV.fetch('SWARM_WORKER', false)

nodes = []
(1..nodes_num).each do |n|
  nodes.push({:id => n, :name => "swarm#{n}", :ip => "#{ip_prefix}#{n+1}"})
end

File.open("./hosts", 'w') { |file| 
  nodes.each do |i|
    file.write("#{i[:ip]} #{i[:name]} #{i[:name]}\n")
  end
}

Vagrant.configure(2) do |config|
  if (/cygwin|mswin|mingw|bccwin|wince|emx/ =~ RUBY_PLATFORM) != nil
    config.vm.synced_folder ".", "/vagrant", mount_options: ["dmode=700,fmode=600"]
  else
    config.vm.synced_folder ".", "/vagrant"
  end

  config.vm.provider "virtualbox" do |v|
    v.memory = node_memory
    v.cpus = node_cpus
  end

  nodes.each do |node|
    config.vm.define node[:name] do |i|
      i.vm.box = "ubuntu/bionic64"
      i.vm.hostname = node[:name]
      i.vm.network "private_network", ip: "#{node[:ip]}"
      i.vm.provision "shell", inline: "curl -fsSL get.docker.com -o get-docker.sh && sh get-docker.sh"
      i.vm.provision "shell", inline: "service docker start", privileged: true
      if File.file?("./hosts")
        i.vm.provision "file", source: "hosts", destination: "/tmp/hosts"
        i.vm.provision "shell", inline: "cat /tmp/hosts >> /etc/hosts", privileged: true
      end
      if node[:id] == 1
        i.vm.provision "shell", inline: "docker swarm init --advertise-addr #{node[:ip]}", privileged: true
        i.vm.provision "shell", inline: "docker swarm join-token -q manager > /vagrant/token-manager", privileged: true
        i.vm.provision "shell", inline: "docker swarm join-token -q worker > /vagrant/token-worker", privileged: true
      else
        if node_worker
          i.vm.provision "shell", inline: "docker swarm join --advertise-addr #{node[:ip]} --listen-addr #{node[:ip]}:2377 --token `cat /vagrant/token-worker` #{nodes[0][:ip]}:2377", privileged: true
        else
          i.vm.provision "shell", inline: "docker swarm join --advertise-addr #{node[:ip]} --listen-addr #{node[:ip]}:2377 --token `cat /vagrant/token-manager` #{nodes[0][:ip]}:2377", privileged: true
        end
      end
    end
  end

  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :box
  end
  
  if Vagrant.has_plugin?("vagrant-vbguest")
    config.vbguest.auto_update = false
    config.vbguest.no_install = true
    config.vbguest.no_remote = true
  end

end
