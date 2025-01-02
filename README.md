<div align="center">
<img width="60px" src="https://pts-project.org/android-chrome-512x512.png">
<h1>Ansible playbooks for Colander</h1>
<p>
A set of Ansible playbook to deploy Colander.
</p>
<p>
License: GPLv3
</p>
<p>
<a href="https://pts-project.org">Website</a> | 
<a href="https://pts-project.org/docs/pirogue/overview/">Documentation</a> | 
<a href="https://discord.gg/qGX73GYNdp">Support</a>
</p>
</div>

## Initial setup of the server 
Creating a user `colander` on the target server is a requirement.

To do so, run the following commands:
```
useradd -m colander --shell /bin/bash
usermod -aG sudo colander
passwd colander
```
and copy your SSH key.
```
ssh-copy-id -i ~/.ssh/my-key colander@192.168.122.189
```

## Example of usage of the playbooks
```
ansible-galaxy role install geerlingguy.docker
ansible-galaxy collection install community.docker

# Install Docker on the target
ansible-playbook -K -i inventory/production/hosts.ini playbooks/docker.yml

# Generate the configuration template 
ansible-playbook -i inventory/production/hosts.ini playbooks/configure.yml  

# Encrypt the configuration as a vault
ansible-vault encrypt playbooks/vault.yml

# Configure Colander server
ansible-playbook -K -e playbooks/vault.yml -i inventory/production/hosts.ini playbooks/colander_configuration.yml --tags configure

```
