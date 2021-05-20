playbook = ENV["PLAYBOOK"] || nil

Vagrant.configure("2") do |config|
  # The control-plane node
  config.vm.define "main" do |main|
    main.vm.box = "debian/stretch64"
    main.vm.hostname = "main"
    main.vm.network "private_network", ip: "10.0.0.10"

    main.vm.provider "virtualbox" do |vb|
      vb.memory = 2048
      vb.cpus = 2
    end
  end

  # A worker
  config.vm.define "worker" do |worker|
    worker.vm.box = "debian/buster64"
    worker.vm.hostname = "worker"
    worker.vm.network "private_network", ip: "10.0.0.11"

    worker.vm.provider "virtualbox" do |vb|
      vb.memory = 1024
      vb.cpus = 1
    end
  end

  # Used for NFS and Persistent Volumes
  config.vm.define "data" do |data|
    data.vm.box = "debian/buster64"
    data.vm.hostname = "data"
    data.vm.network "private_network", ip: "10.0.0.12"

    data.vm.provider "virtualbox" do |vb|
      vb.memory = 512
      vb.cpus = 1
    end

    unless playbook.nil?
      # https://www.vagrantup.com/docs/provisioning/ansible.html#ansible-parallel-execution
      data.vm.provision :ansible do |ansible|
        ansible.inventory_path = "hosts/development"
        ansible.limit = "all"
        ansible.playbook = playbook
      end
    end
  end
end
