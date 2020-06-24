# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    config.vm.box = "ubuntu/xenial64"
    config.vm.provider "virtualbox"
    config.ssh.insert_key = false
    config.vm.provider :virtualbox do |v|
        v.memory = 2048
        v.cpus = 2
        v.linked_clone = true
        v.customize ['modifyvm', :id, '--audio', 'none']
    end

    # Define the control node
    config.vm.define "ansible" do |ansible|
        ansible.vm.hostname = "ansible"
        ansible.vm.network :private_network, ip: "192.168.7.2"
        config.vm.synced_folder './data', '/data'
        # Copy private SSH key to be used by Ansible
        ansible.vm.provision "file", source: "./ssh-keys/ansible", destination: "/home/vagrant/.ssh/ansible"
        ansible.vm.provision "file", source: "./ansible.cfg", destination: "/home/vagrant/.ansible.cfg"
        # Install Ansible
        ansible.vm.provision :shell, inline: <<-SHELL
            chmod 600 /home/vagrant/{.ssh/ansible,.ansible.cfg}
            apt-add-repository -y ppa:ansible/ansible
            apt-get update -y
            apt-get install -y ansible
        SHELL
    end

    # Define managed nodes
    managed_nodes = [
        { :name => "node1", :ip => "192.168.7.3" },
        { :name => "node2", :ip => "192.168.7.4" }
    ]
    managed_nodes.each_with_index do |opts, index|
        config.vm.define opts[:name] do |managed_node|
            managed_node.vm.hostname = opts[:name] + ".managed"
            managed_node.vm.network :private_network, ip: opts[:ip]
            managed_node.vm.provision :shell, privileged: false do |shell_action|
                # Copy public SSH keys to be used by Ansible
                ssh_public_key = File.readlines("./ssh-keys/ansible.pub").first.strip
                shell_action.inline = <<-SHELL
                    echo #{ssh_public_key} >> /home/vagrant/.ssh/authorized_keys
                    sudo bash -c "echo #{ssh_public_key} >> /root/.ssh/authorized_keys"
                SHELL
            end
        end
    end
end