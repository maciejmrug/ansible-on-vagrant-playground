# Ansible playground on Vagrant
Runs an Ansible [control node](https://docs.ansible.com/ansible/latest/network/getting_started/basic_concepts.html#control-node)
and an arbitrary no. of [managed nodes](https://docs.ansible.com/ansible/latest/network/getting_started/basic_concepts.html#managed-nodes)
within a single Vagrant setup.

## Rationale
- You want to experiment with Ansible on your local dev environment without touching any remote servers.
- You don't want to install Ansible on your local workstation.

NOTE: This solution DOESN'T use Ansible as a [Vagrant provisioner](https://www.vagrantup.com/docs/provisioning/ansible.html#setup-requirements)
precisely because of the above requirements.

## How to run
1. Generate an SSH key pair to be used by Ansible, and save it in the `ssh-keys` directory:
```sh
ssh-keygen -t rsa -b 4096 -C "Ansible" -f ./ssh-keys
```
2. Bring up the environment:
```sh
vagrant up
```
3. The `./data` dir in the repo is synced with `/data` on the Ansible control node, so you can just develop your
playbooks there and SSH into the control node to start interacting with the managed nodes:
```sh
vagrant ssh ansible
# on the vagrant node:
cd /data
ansible-playbook -i inventory playbook-ping.yml
```

## How to develop
### Managed nodes
You can tweak the number and config of managed nodes in the [Vagrantfile](./Vagrantfile#L34). Then, update the
[inventory file](./data/inventory) accordingly.

### Ansible config
Use `./ansible.cfg`, which is [copied](./Vagrantfile#L23) to Vagrant user's home dir during provisioning.

CAVEAT: If you don't want to reload Vagrant on every change to `ansible.cfg`, consider other ways of providing
[config options](https://docs.ansible.com/ansible/devel/reference_appendices/config.html#the-configuration-file).
However, if you copy `ansible.cfg` to the `./data` dir which is synced with the VM, Ansible will ignore it
because of permissions issue:
```
[WARNING] Ansible is being run in a world writable directory (/data), ignoring it as an ansible.cfg source
```
