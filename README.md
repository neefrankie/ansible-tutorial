# Ansible Tutorial

Based on [Ansible: Up and Running](https://www.amazon.com/Ansible-Automating-Configuration-Management-Deployment/dp/1491979801/ref=sr_1_1?ie=UTF8&qid=1544085913&sr=8-1&keywords=ansible+up+and+running)

## Using Vagrant to Set up a Test Server

First install [Virtual](https://www.virtualbox.org/) and [Vagrant](https://www.vagrantup.com/).

Create a Vagrant configuration file:

```
vagrant init ubuntu/bionic64
vagrant up
```

Then it starts to downloading Ubuntu images. It takes some time.

After download finised, test SSH:
```
vagrant ssh
```

Type `exit` to quit the SSH session.

`vagrant ssh-config` to output the SSH connection details.

## Inventory file

Create a `hosts` file.

A host could be sepcified in one of two ways:

If you already configure you ssh with a `Host` alias, use that name directly:
```
testserver ansible_host=myserver
```

Or you can specify explicitly if you do not have a ssh config file on your machine:

```
testserver ansible_host=127.0.0.1 ansigle_user=user_name ansible_private=~/.ssh/id_rsa
```

Run the command to see if the connection works:

```
ansible testserver -i hosts -m ping
```

Note: target server must have Python2 installed.

## Use ansible.cfg File

Ansible looks for an ansible.cfg file in the following places, in this order:

* File specified by the `ANSIBLE_CONFIG` environment variable
* `ansible.cfg` (in the current directory)
* `~/.ansible.cfg` (in the home directory)
* `/etc/ansible/ansible.cfg`

Use the first file found, all others are ignored.

Create this file with:
```
[defaults]
inventory = hosts
remote_user = node
private_key_file = ~/.ssh/id_rsa
```

With default values set, we no longer need to specify the SSH user or key file in `hosts` file:

```
testserver ansible_host=127.0.0.1 ansible_port=2222
```

Then we can invoke Ansible without passing the `-i hostname` arguments:

```
testserver testserver -m ping
```

You can execute arbitrary commands with the `command module`. You also need to pass an argument to the module with the `-a` flag, which is the command to run:

```
ansible testserver -m command -a uptime
```

The `command` module is the default module, so we can omit it:

```
ansible testserver -a uptime
```

If command contains spaces, quote it:

```
ansible testserver -a "tail /var/log/dmesg"
```

If you need root access, pass in the `-b` flag to become the root user:

```
ansible testserver -b -a "tail /var/log/syslog"
```

To insall Nginx:
```
ansible testserver -b -m apt -a name=nginx
```

To update the package list before installing Nginx:
```
ansible testserver -b -m apt -a name=nginx update_cache=yes
```

Restart Nginx:
```
ansible testserver -b -m service -a "name=nginx state=restarted"
```