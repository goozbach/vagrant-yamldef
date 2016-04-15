# -*- mode: ruby -*-
# vi: set ft=ruby :
#
puts "Arguments: #{ARGV}"

# import yaml defs
require 'yaml'

# pull in yaml configs from groupvars/all
current_dir = File.dirname(File.expand_path(__FILE__))
yamlcfg = YAML.load_file("#{current_dir}/ansible/envs/dev.yml")

# load up the machines array
machines = []
puts "Loading machine definitions from #{current_dir}/ansible/envs/dev.yml"

yamlcfg['vagrant_hosts'].each do |imgval|
  machines.push(imgval)
end

# load up containers array
containers = []
puts "Loading container definitions from #{current_dir}/ansible/envs/dev.yml"
yamlcfg['containers'].each do |cont|
  containers.push(cont)
end

puts "\n" # to seperate our debugging output from vagrant output

Vagrant.configure(2) do |config|
  config.ssh.forward_agent = true

  # iterate over machines
  machines.each do |machine|
    config.vm.define "#{machine['name']}", primary: machine['primary'], autostart: machine['autostart'] do |vgtbox|
      vgtbox.vm.box = machine['box']
      vgtbox.vm.hostname = machine['hostname']
      vgtbox.vm.network "private_network", ip: machine['address']
      vgtbox.ssh.guest_port = machine['ssh_port']
      vgtbox.vm.network "forwarded_port", guest: 22, host: machine['ssh_port']
    end
  end

  containers.each do |container|
    config.vm.define container['name'], autostart: container['autostart'] do |cntnr|
      cntnr.vm.synced_folder ".", "/vagrant", disabled: true
      cntnr.vm.provider "docker" do |docker|
        docker.vagrant_vagrantfile = "host/Vagrantfile"
        docker.image = container['image']
        docker.name = container['name']
        docker.pull = container['pull']
        docker.remains_running = true
        docker.ports = ["#{container['ssh_port']}:#{container['ssh_dest_port']}"]
        docker.has_ssh = "true"
      end
    end
  end

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "ansible/vagrant.yml"
    ansible.extra_vars = {
        envrion: "vagrant",
        ansible_ssh_user: 'vagrant'
    }
    ansible.sudo = true
    ansible.skip_tags = 'novagrant'
    ansible.inventory_path = 'ansible/inv-dev/hosts'
  end
end
