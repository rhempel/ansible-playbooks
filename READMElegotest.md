# legotest-playbooks

The current method for managing the RaspberryPi based Firmware Test
Framework (FTF) at LEGO is a Python script that runs on a PC or OSX
machine. This works but requires too much installation on the local
machine and requires re-learning how to do it every time.

The new procedure uses RedHat Ansible running on a RaspberryPi to
manage configuring and upating the FTF.

Initially we will run Ansible from a Linux host machine, but the final
version will provide instructions for doing a minimal manual setup
to get a RaspberryPi set up to run Ansible, and look after itself and
the FTF afterwards.

The goal is to be able to grab the latest Raspbian image, burn it onto
a card (using Etcher), and then easliy run a few playbooks to get the
RaspberryPi configured and added to the test framework. As a bonus, the
files we use to configure Ansible also provide documentation on the 
FTF is configured.

# First Steps

Install Ansible on your Linux or OSX host machine - later we will make
this a Raspberry Pi as well :-) Unfortunately Windows is NOT supported
as an Ansible host machine.

Flash the latest Raspbian Buster Lite image to the SD card using
Balena Etcher

https://www.raspberrypi.org/downloads/raspbian/
https://www.balena.io/etcher/

By default, the RaspberryPi has ssh disabled, so when you have finished
flashing the microSD card you will need to create a blank file 
called 'ssh' on the card:

```
touch /media/yourusername/boot/ssh
```

For security, we will use wired networking and disable WiFi on our
RaspberryPis in the FTF. So plug in an Ethernet cable and connect to
your host machine.

You may need to enable link-local networking on your local machine.

 - we will add a section here on setting up the Ansible hosting RaspberryPi

Now insert the card in the RaspberryPi and boot - you may need to wait
a few minutes before the device is available.

Check that you can reach the RaspberryPi on your wired network:

```
ping raspberrypi.local
```

Now run the inital playbook that permanently enables ssh, disables
the local wifi, changes the hostname, and then powers down the rpi:

```
ansible-playbook -v -i ./legotest_hosts -u pi -k fresh-rpi.yml --extra-vars '{"hostname":"some-new-hostname"}' 
```

You will be asked for the 'pi' user's SSH password for the
RaspberryPi which is of course 'raspberry' - yes it's insecure
but we're going to change that shortly!




 
To create an ansible-vault encrypted variable, such as a password:

mkpasswd --method=sha-512

ansible-vault encrypt_string '$6$t5eSXI8UxtBoU7JV$9CkhiIIT5UOGfRK7vlLRtPBxfZM7IMxkSlkBwSPduldwj/VsKAKBzbHUlUPDEWkwBJfbZHSfFvXZNNMaoN7ym1' --name pi_password

Next we run another playbook that changes the default pi user password
to something from this Ansible vault, and then adds me as a new user
with sudo, adds me to all the standard pi groups and adds my public
ssh key from GitHub so that I can log into this pi without ever having
to enter the password again!

```
ansible-playbook -v -i ./hosts -u pi -k my-rpi.yml --ask-vault-pass --extra-vars '{"hostname":"zwischenwagen"}' 

```

# Next Steps

From this point forward, we can use Ansible playbooks without a lot of extra parameters
so let's get started with a few basic roles - roles are the way we can easily build up
playbooks using known-good methods.

- Update to latest packages

```
ansible-playbook -v -i ./hosts rpi-jekyll.yml 
```

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
gem "jekyll-include-cache"

And create a blank "_config.yml" and copy the default contents into it from
the MinimalMistakes repo



# Acknowledgment

Much of the information used to assemble these playbooks comes from:

- https://docs.ansible.com/ansible/latest/user_guide
- https://github.com/chesterbr/chester-ansible-configs
- https://github.com/glennklockwood/rpi-ansible
- https://github.com/giuaig/ansible-raspi-config
- https://github.com/jonashackt/raspberry-ansible
- https://github.com/motdotla/ansible-pi
- https://github.com/davidderus/ansible-rpi
