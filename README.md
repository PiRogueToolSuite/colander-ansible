<div align="center">
<img width="60px" src="https://pts-project.org/android-chrome-512x512.png">
<h1>Ansible playbooks for Colander</h1>
<p>
A set of Ansible playbooks to deploy Colander.
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

# Deployment procedure
The first step is to clone this repository on your computer (Ansible control node), not on the server. 
```
git clone https://github.com/PiRogueToolSuite/colander-ansible.git
```

When needed, run the following command to update the playbooks:
```
git pull origin main
```

## Set up your Ansible control node
To make things easier, we recommend to copy the SSH key of the `colander` user on your server. 

On your [Ansible control node](https://docs.ansible.com/ansible/latest/getting_started/basic_concepts.html#control-node), run the command:
```
ssh-copy-id -i ~/.ssh/my-key colander@[IP address of your server]
```

Then, create a Python 3 virtual environment and install the dependencies with the commands: 
```
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
ansible-galaxy install -r requirements.yml
```


## Create your Ansible inventory file
Rename the file `production-example.yml` to `production.yml` and replace:
* `[IP address of the server]` with the IP address of your server
* `[path of your SSH private key]` with the path of your SSH private key

In the folder `host_vars`, rename the file `production-example.yml` to `production.yml` in which you can override the default configuration. You can specify that you don't want to use:
* Threatr, by setting `use_threatr` to False
* Cyberchef, by setting `use_cyberchef` to False
* Mandolin, by setting `use_mandolin` to False
* Playwright, by setting `use_playwright` to False

Example of host configuration:
```yaml
stack_overrides:
  flavor:
    use_threatr: True
    use_cyberchef: False
    use_mandolin: False
    use_playwright: True
  services:
    elasticsearch:
      xms: 512m
      xmx: 512m
```

## Initial setup of your server 
Your server must have the latest version Debian already installed before starting the deployment. 

### Create a user for Colander
Before running Ansible playbooks to install Colander, you must create a user `colander` on your server. Run the following commands on your server:
```
useradd -m colander --shell /bin/bash
usermod -aG sudo colander
passwd colander
```

### Install Docker
Then, it's necessary to install Docker on your server. Run the playbook `install_docker` with the command:
```
ansible-playbook -K -i production.yml playbooks/install_docker.yml
```
Ansible will ask you to enter the *BECOME password* which corresponds to the password of the user *colander* you have created. By default, the playbook installs Docker in the [rootless mode](https://docs.docker.com/engine/security/rootless/).


## Generate the configuration of Colander
Before deploying Colander on your server, you must generate its configuration. The generated file contains all the secrets of your Colander server.

To generate it, run the command on your Ansible control node:
```
ansible-playbook -i production.yml playbooks/generate_configuration.yml
```

The playbook will ask you to enter the root domain corresponding to your Colander server. Alternatively, you can pass it directly in the command line with: 
```
ansible-playbook -i production.yml playbooks/generate_configuration.yml --extra-vars "root_domain=my.domain"
```
This creates a file `group_vars/colander/vault`. 

In this file, you must specify:
* `acme_email`: the email address passed to Let's Encrypt
* `admin_name`: the server administrator’s full name
* `admin_email`: the email address the crash reports will be sent to
* `django_default_from_email`: the email address to be used when sending emails
* `email_host`: the host of the SMTP server
* `email_host_user`: the username for the SMTP server
* `email_host_password`: the password for the SMTP server
* `email_port`: the port for the SMTP server
* `email_use_tls`: true if the SMTP server uses TLS, false otherwise
* `email_use_ssl`: true if the SMTP server uses SSL, false otherwise

Finally, back up and encrypt this file as it contains sensitive informations and secrets:
```
ansible-vault encrypt group_vars/colander/vault
```
Don't forget to save the password of your vault in your favorite password manager.

⚠️ Before going further, make sure you have configured subdomains pointing to your server:
* `colander.[your domain]`
* `threatr.[your domain]` only if you are planning to deploy Threatr, check your inventory file
* `cyberchef.[your domain]` only if you are planning to deploy Cyberchef, check your inventory file
* `traefik.[your domain]`

As an example, if your domain is `my.domain` and the public IP address of your server is `253.127.98.9`, your A record would look like `colander.my.domain A 253.127.98.9`.

You are now ready to deploy Colander on your server 🎉


## Deploy Colander
You can now deploy Colander on your server. The playbook `colander` configures and deploys it on your server:
```
ansible-playbook -J -i production.yml playbooks/colander.yml --tags configure,deploy
```
Depending on your server, the first deployment takes several minutes to complete. The playbook automatically creates the `admin` users for Colander and Threatr, and deploy the whole stack on your server.

Colander is now up and accessible on `https://colander.[root domain]/`.

Note that Colander is designed to update by itself, but it doesn't update nor upgrade the server operating system.


## Tear down Colander
If you want to completely uninstall Colander, run the playbook `colander.yml` with the tag `teardown`. All containers, volumes, images, data, backups, configuration files will be permanently deleted. This action cannot be reverted.

```
ansible-playbook -J -K -i production.yml playbooks/colander.yml --tags teardown
```

The playbook will ask you the password of the user `colander` and the password of your vault.