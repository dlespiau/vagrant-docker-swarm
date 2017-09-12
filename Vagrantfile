# This Vagrantfile creates a docker swarm of n_workers+1 nodes: 1 master and
# n_workers workers (containers can be scheduled on masters as well)

$n_workers = 2

$install_docker = <<EOF
echo Installing Docker...
curl -sSL https://get.docker.com/ | sh
usermod -aG docker ubuntu
EOF

$setup_manager = <<EOF
echo Setting up manager...
docker swarm init --listen-addr 10.64.0.1:2377 --advertise-addr 10.64.0.1:2377
docker swarm join-token --quiet worker > /vagrant/swarm_token
EOF

$setup_worker = <<EOF
echo Worker joining swarm...
docker swarm join --token $(cat /vagrant/swarm_token) 10.64.0.1:2377
EOF

Vagrant.configure("2") do |config|
    box = "ubuntu/xenial64"

    # Start the manager

    config.vm.define :manager, primary: true do |manager|
        manager.vm.box = box
        manager.vm.hostname = "manager"
        manager.vm.network :private_network, ip: "10.64.0.1"
        manager.vm.synced_folder ".", "/vagrant"
        manager.vm.provision "shell", inline: $install_docker
        manager.vm.provision "shell", inline: $setup_manager
        manager.vm.provider "virtualbox" do |vb|
            vb.name = "swarm manager"
            vb.memory = "512"
        end
    end


    # Start n_workers workers that will join the swarm

    (1..$n_workers).each do |i|
        config.vm.define "worker-#{i}" do |worker|
            worker.vm.box = box
            worker.vm.hostname = "worker-#{i}"
            worker.vm.network :private_network, ip: "10.64.0.#{10+i}"
            worker.vm.synced_folder ".", "/vagrant"
            worker.vm.provision "shell", inline: $install_docker
            worker.vm.provision "shell", inline: $setup_worker
            worker.vm.provider "virtualbox" do |vb|
                vb.name = "swarm worker"
                vb.memory = "1024"
            end
        end
    end
end
