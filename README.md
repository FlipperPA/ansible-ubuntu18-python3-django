# Ubuntu 18.04, Python 3 & Django Ansible Playbooks and Roles

This repository holds Ansible playbooks and roles for configuring Python 3 & Django servers. You should be able to clone the repository and follow these steps to get a server up and running; this has been tested with Digital Ocean droplets. These instructions assume you have a working knowledge of how SSH keypairs work; [click here for a good introductory tutorial](https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys).

You'll want to come up with a service account name that you'll use in place of `your_project_ansible_user`. After cloning this repository, edit `ansible.cfg` and change `remote_user = your_project_ansible_user` with the username you will use.

## Preparing a Destination Server

The destination server will need to be kickstarted with Ubuntu 18.04. Log in as root to your freshly created Digital Ocean droplet (you used SSH keys, right?), and create a user called `your_project_ansible_user` on the destination server:

```bash
adduser your_project_ansible_user
usermod -aG sudo your_project_ansible_user
chmod 640 /etc/sudoers
```

Then edit `/etc/sudoers`. Find this line:
`%sudo   ALL=(ALL:ALL) ALL`
...and change it to:
`%sudo   ALL=(ALL:ALL) NOPASSWD: ALL`

Then let's become the `your_project_ansible_user` user, and generate keys:

```bash
su - your_project_ansible_user
ssh-keygen -b 4096
```

After issuing the `ssh-keygen` command, hit enter three times to use the defaults. You will then need to add your public key from your host control machine to `your_project_ansible_user`'s authorized keys in `~/.ssh/authorized_keys`:

```bash
echo "ssh-ed25519 AAAAC3NzDummyDI1Z72sk0VuRo48DummydF2dtADummyTHxNTE5AoDummyMyckiqF2 you@yourdomain.com" >> .ssh/authorized_keys
chmod 600 .ssh/authorized_keys
exit
```

We'll want to create a separate service user account with less privileges to deploy your Django project. I'll call this account `web_deploy`, but you can choose whatever name you like.

```bash
adduser web_deploy
usermod -aG www-data web_deploy
su - web_deploy
ssh-keygen -b 4096
echo "ssh-ed25519 AAAAC3NzDummyDI1Z72sk0VuRo48DummydF2dtADummyTHxNTE5AoDummyMyckiqF2 you@yourdomain.com" >> .ssh/authorized_keys
chmod 600 .ssh/authorized_keys
exit
```

## Getting started

Clone this repository.

    git clone git@github.com:YourUsername/ansible-ubuntu18-python3-django.git
    cd ansible-ubuntu18-python3-django.git

Edit the `inventory` file and add the destination servers.

    [web_py]
    dev.yourserver.com
    123.34.56.789

Edit the `ansible.cfg` file and change the `remote_user` to the one you created above on the destination server.

    remote_user = your_project_ansible_user

Deploy the server:

    ansible-playbook playbooks/web-py.yml

NOTE: if you're using Vagrant, you may get an error about the config file being world-writable due to Vagrant's shared folders. You can issue commands with an environment variable as a work-around:

    ANSIBLE_CONFIG=./ansible.cfg ansible-playbook playbooks/web-py.yml

Show some servers:

    ansible ubuntu_18 --list-hosts
    ansible web_py --list-hosts

Run some commands; `shell` is required for arguments and shell functions (`command` runs without the target user's shell):

    ansible all -m command -a uptime -i inventory
    ansible all -m shell -oa 'ps -eaf'
    ansible ubuntu_18 -m copy -a 'content="Welcome to your server, Ansiblized.\n" dest=/etc/motd' -u apache --become --become-user root

Documentation: list help section, show documentation for specific command (with examples!):

    ansible-doc -l
    ansible-doc yum
