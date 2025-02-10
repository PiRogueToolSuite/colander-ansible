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


## Set up your Ansible host
To make things easier, we recommend to copy the SSH key of the `colander` user on your server. 

On your Ansible host, run the command:
```
ssh-copy-id -i ~/.ssh/my-key colander@[IP address of your server]
```

Then, create a Python 3 virtual environment with the commands: 
```
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```


## Create your inventory
Rename the file `production-example.yml` to `production.yml` and replace:
* `[IP address of the server]` with the IP address of your server
* `[path of your SSH private key]` with the path of your SSH private key

In the folder `host_vars`, rename the file `production-example.yml` in which you can override the default configuration. You can specify that you don't want to use:
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
Before running Ansible playbooks to install Colander, you must create a user `colander` on your server. Run the following commands on your server:
```
useradd -m colander --shell /bin/bash
usermod -aG sudo colander
passwd colander
```

Then, it's necessary to install Docker on the server. Run the playbook `install_docker` with:
```
ansible-playbook -K -i production.yml playbooks/install_docker.yml
```
By default, the playbook installs Docker in the [rootless mode](https://docs.docker.com/engine/security/rootless/).


## Generate the configuration of Colander
Before deploying Colander on your server, you must generate its initial configuration. The generated file contains all the secrets of your Colander server, this configuration file must be generated only once.

To generate it, run the command below on your Ansible host:
```
ansible-playbook -i production.yml playbooks/generate_configuration.yml
```

The playbook will ask you to enter the root domain corresponding to your Colander server. Alternatively, you can pass it directly in the command line with: 
```
ansible-playbook -i production.yml playbooks/generate_configuration.yml --extra-vars "root_domain=my.domain"
```
This creates a file named `configuration-vault.yml`.

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

Finally, **we recommend to encrypt and backup this file** as it contains sensitive informations and secrets:
```
ansible-vault encrypt configuration-vault.yml
```

You are now ready to deploy Colander on your server 🎉


## Deploy Colander
You can now deploy Colander on your server. The paybook `colander` configure and deploy Colander:
```
ansible-playbook -J -e configuration-vault.yml -i production.yml playbooks/colander.yml --tags configure,deploy
```
The playbook automatically creates the `admin` users for Colander and Threatr, and deploy the whole stack on your server.

Colander is now up and accessible on `https://colander.[root domain]/`.


## Other operations
* Tear down the whole stack: `ansible-playbook -J -e configuration-vault.yml -i production.yml playbooks/colander.yml --tags teardown`