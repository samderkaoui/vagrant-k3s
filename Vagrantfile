# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|

    # Change to add more workers
    NodeCount = 1
  
    # Kubernetes Master
    config.vm.define "master" do |master|
      master.vm.box = "ubuntu/jammy64"
      master.vm.synced_folder '.', '/vagrant' 
      master.vm.hostname = "k3s-master"
      master.vm.network "private_network", ip: "192.168.10.100"
      master.vm.provider "virtualbox" do |v|
        v.name = "k3s-master"
        v.memory = 1024
        v.cpus = 2
      end
      master.vm.provision "shell", inline: <<-SHELL
      # Modify SSH configuration
      sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config 
      sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config 
      sed 's@session\\s*required\\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd
      sed -i 's/KbdInteractiveAuthentication no/KbdInteractiveAuthentication yes/' /etc/ssh/sshd_config
      sudo systemctl reload sshd
      # Install and configure UFW
      sudo apt-get install ufw -y
      sudo systemctl start ufw && sudo systemctl enable ufw
      sudo ufw allow 80/tcp
      sudo ufw allow 443/tcp
      sudo ufw allow 9090/tcp
      sudo ufw allow 22/tcp
      sudo ufw allow 6443/tcp
      sudo ufw allow 10250/tcp
      sudo ufw allow 8472/udp
      sudo ufw allow 51820/udp
      sudo ufw allow 30000:32767/tcp
      sudo systemctl restart ufw
      # Install K3s
      curl -sfL https://get.k3s.io | sh -s - server --node-ip 192.168.10.100 --bind-address 192.168.10.100 --advertise-address 192.168.10.100 --flannel-iface enp0s8

      # Get node token for workers to join
      sudo chmod 644 /etc/rancher/k3s/k3s.yaml
      sudo cat /var/lib/rancher/k3s/server/node-token > /vagrant/node-token
      
      sleep 15
      # Taint so only worker ont des pods
      kubectl taint nodes k3s-master node-role.kubernetes.io/master:NoSchedule

      # Validation
      kubectl describe node k3s-master | grep -i taint
  SHELL

    end
  
    (1..NodeCount).each do |i|
      config.vm.define "worker#{i}" do |worker|
        worker.vm.box = "ubuntu/jammy64"
        worker.vm.synced_folder '.', '/vagrant' 
        worker.vm.hostname = "k3s-worker#{i}"
        worker_ip = "192.168.10.#{i+1}"
        worker.vm.network "private_network", ip: worker_ip
        worker.vm.provider "virtualbox" do |v|
          v.name = "k3s-worker#{i}"
          v.memory = 1024
          v.cpus = 2
        end
        worker.vm.provision "shell", env: {"WORKER_IP" => worker_ip}, inline: <<-SHELL
        # Modify SSH configuration
        sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config 
        sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config 
        sed 's@session\\s*required\\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd
        sed -i 's/KbdInteractiveAuthentication no/KbdInteractiveAuthentication yes/' /etc/ssh/sshd_config
        sudo systemctl reload sshd
        # Install and configure UFW
        sudo apt-get install ufw -y
        sudo systemctl start ufw && sudo systemctl enable ufw
        sudo ufw allow 80/tcp
        sudo ufw allow 443/tcp
        sudo ufw allow 9090/tcp
        sudo ufw allow 22/tcp
        sudo ufw allow 6443/tcp
        sudo ufw allow 10250/tcp
        sudo ufw allow 8472/udp
        sudo ufw allow 51820/udp
        sudo ufw allow 30000:32767/tcp
        sudo systemctl restart ufw
        # Join K3s cluster
        NODE_TOKEN=$(cat /vagrant/node-token)
        curl -sfL https://get.k3s.io | K3S_TOKEN=$NODE_TOKEN sh -s - agent --server https://192.168.10.100:6443 --node-ip $WORKER_IP --flannel-iface enp0s8
      SHELL
      end
    end
  end
