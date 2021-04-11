# ansible-playbooks

This is an attempt to get some order into the way I manage my small
but growing army of embedded computers - mostly Raspberry Pi variants.

The goal is to be able to grab the latest Raspbian image, burn it onto
a card (using Etcher), and then easliy run a few playbooks to get the
Pi configured to suit my needs. It's way easier than writing down a
number of step by step instructions

# First Steps

Install Ansible on your host machine - later we will make this a 
Raspberry Pi as well :-)

Be aware of the following things that can trip you up:

- Ansible uses `ssh` under the hood to connect to the device you are
  trying to configure. By default, each new Raspberry Pi will announce
  itself as raspberrypi.local or it may be assigned a DHCP address
  that has been used before.

- If the device name or IP address has been used by `ssh` before, then
  you will get an error from the ssh system like this:

```
WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that the RSA host key has just been changed.
The fingerprint for the RSA key sent by the remote host is ...
```

To get around this, remove the old `ssh` fingerprint like this:

```
ssh-keygen -R [hostname|address] 
```

And then `ssh` in manually one time, like this:

```
ssh pi@[hostname|address] 
```

Remember the default password is `raspberry` - we will change that soon
enough.

# Initial setup of the Raspberry Pi

Now run the inital playbook that permanently enables ssh, configures
the local wifi, changes the hostname, and then powers down or restarts
the Raspberry Pi.

```
ansible-playbook -v -i [hostname|address], -k fresh-rpi.yml --extra-vars '{"hostname":"some-new-hostname"}' 
```

Note the comma following the `hostname` or `address` - this ensures that
the address is not interpreted as a filename.

Once this is completed, we can move on to adding our private ssh keys
and making the system more secure by changing the default password for
the `pi` user.

# Using Encrypted Variables

We want to share our scripts, but not our secrets! Ansible makes this
relatively easy with a feature called the vault. You use the vault to securely
store anything that you need to run your scripts. That might include passwords,
server names, file paths, etc.

To create an ansible-vault encrypted variable, such as a password:

```
mkpasswd --method=sha-512

ansible-vault encrypt_string '$6$t5eSXI8UxtBoU7JV$9CkhiIIT5UOGfRK7vlLRtPBxfZM7IMxkSlkBwSPduldwj/VsKAKBzbHUlUPDEWkwBJfbZHSfFvXZNNMaoN7ym1' --name pi_password
```

This creates an encrypted variable in the Ansible vault called `pi_password`
and is available in any playbook tasks that need it.  If this is the first
variable that you have placed in the vault, you will be asked to provide a
password (and a confirmation).

# Setting yourself up as an admin user on the Raspberry Pi

Next we run another playbook that changes the default pi user password to
something from this Ansible vault, and then adds me as a new user with sudo,
adds me to all the standard pi groups and adds my public ssh key from GitHub so
that I can log into this pi without ever having to enter the password again!

Before running this playbook, make sure to check the file `group_vars/all.yml`
as it will contain the variables for setting up the default user - obviously
you should set this up to your own values.

```
ansible-playbook -v -i [some-hostname|address}, -k my-rpi.yml --ask-vault-pass \
                 --extra-vars '{"hostname":"some-hostname"}'
```

# Manage your .ssh keys on the Raspberry Pi

I use a `config` file in my `~/.ssh` directory to manage access to remote
machines. See this article for details: https://stackoverflow.com/a/38454037

We assume here that you have created a file on your local machine called
`~/.ssh/ssh.tar.bz2` and that it's kept up to date. I keep the instructions
for updating the file in `~/.ssh/config`.

Now run the following playbook to copy and unpack the file in your `~/.ssh~`
directory on the target machine:

```
ansible-playbook -v -i [some-hostname|address}, admin-ssh.yml
```

# Next Steps

From this point forward, we can use Ansible playbooks without a lot of extra
variables because we have set our target device up with a known hostname
and ouselves up as an admin user on that machine.

Let's get started with a few basic roles - roles are the way we can easily
build up playbooks using known-good methods.

From this point forward, we will also be using a local `hosts` file that
will make our inventory of devices easier to manage. We can do this because
we have given our device a unique hostname, so there is no need to supply
an explicit hostname on the `ansible-playbook` command line. 

## Set up the Raspberry Pi for Python vituralenv

```
ansible-playbook -v -i ./hosts python-venv.yml
```

## Set up the Raspberry Pi for LEGO Functional Test Framework

Investigate how to keep the vault passwords locally and avoid passing
the `--ask-vault-pass` option on the command line

```
ansible-playbook -v --ask-vault-pass -i ./hosts lego-testframework.yml
```






- Update to latest packages


This playbook can take a loooong time to run as it updates the apt cache, installs a few
bigger packages, and builds jekyll and the bundler. opens up prot 4000 etc...

# Starting jekyll

The jekyll server can start like this:

bundle exec jekyll serve --host=0.0.0.0 --incremental

# Getting Started With Jekyll

Use Gem based themes to make life easier, start with a new folder and
create a git repo inside it:

mkdir newsite
cd newsite
git init

Now follow the instructions in the MinimalMistakes QuickStart Guide:
https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/

In particular, we are following the "Fresh Start" section

Create a blank Gemfile and put this inside:

gem "minimal-mistakes-jekyll"

And create a blank "\_config.yml" and copy the default contents into it from
the MinimalMistakes repo

Now get things installed by running `bundle` once to get everything installed



# Acknowledgment

Much of the information used to assemble these playbooks comes from:

- https://docs.ansible.com/ansible/latest/user_guide
- https://github.com/chesterbr/chester-ansible-configs
- https://github.com/glennklockwood/rpi-ansible
- https://github.com/giuaig/ansible-raspi-config
- https://github.com/jonashackt/raspberry-ansible
- https://github.com/motdotla/ansible-pi
- https://github.com/davidderus/ansible-rpi
