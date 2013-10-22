Ghostpack!
=======
### A Packer template for Ghost.

![](img/ghostbusters-elevator.jpg)
```
Dr Ray Stantz: You know, it's just occurred to me we really haven't had a completely successful test of this equipment.
Dr. Egon Spengler: I blame myself.
Dr. Peter Venkman: So do I.
Ray: Well, no sense worrying about it now.
Venkman: Why worry? Each one of us is carrying an unlicensed nuclear accelerator on his back.
Ray: ...yep. Let's get ready. Switch me on.
```
<a href="http://youtu.be/WzLobQxY6gg?t=15s" target="_blank">YouTube</a>

## What is this?

<a href="http://ghost.org" target="_blank">Ghost</a> is a new blogging platform, designed to be beautiful and easy to use. To deploy it, though, you need a server. Cloud providers are a good choice. [Bitnami, Rackspace, and DigitalOcean](http://docs.ghost.org/installation/deploy/) all have pre-installed images to run. Or you can roll your own.

Wouldn't it be nice to have your own setup and configuration options in your build? Or to be able to generate and deploy an AWS AMI?

Enter <a href="http://packer.io" target="_blank">Packer</a>, an open source tool for creating identical machine images for multiple platforms from a single source configuration.

Ghostpack is a Packer build template to help you create a fully-functional deploy of Ghost. The first builder is for AWS AMIs. I will add DigitalOcean next. You can easily add your own [OpenStack or VMWare](http://www.packer.io/docs) builders as well.

## Why?

Honestly, if you are savvy enough to install and run Packer, you can probably do this on your own. My goal is just to save you some time.

Also, Packer lets you hack around in the guts of the image and rebuild it in seconds to your preference.

## "Let's get ready. Switch me on."
1. Install Packer: <http://www.packer.io/docs/installation.html>
1. Fork this repo, clone locally.
1. Configure the build (see below)
1. Build it (`$ packer build ...`) (again, see below)
1. Deploy it. (*Make sure you set a security group that allows HTTP access!*) (see below)

# ATTENTION: LET ME REPEAT THAT - MAKE SURE YOU LAUNCH WITH A SECURITY GROUP THAT ALLOWS HTTP ACCESS

## Configuration
1. Copy the `sample_packer_config.json` file as `packer_config.json` and fill out your own keys/passwords, etc. You can write these variables into your builder, but if you do, be sure not to push to a public repo.
1. Edit `playbook/group_vars/all` to your liking. 
1. Add a public key file to `files/authorized keys`. (See note below for warning about sudo passwd.)
1. (optional) Tweak any of the config files in `files` and the playbook role template directories to suit your preferences.

## Build
1. To build an AMI:  
`$ packer build -var-file=config/packer_config.json ghostpack.json`  

## Deploy
1. Launch an EC2 instance with your new AMI.
1. Make sure you select a security group that allows HTTP access.
1. Map your domain to an IP address associated with your instance.

## Issues
1. Right now, Ghost stores images and files locally. This is not good, in that if you lose the local server, you lose your files. This is slated to be [fixed in Ghost 0.4](https://github.com/TryGhost/Ghost/issues/635). When that rolls out, we can update Ghostpack with S3 credentials or whatnot.
1. Add an inline sleep command if a race condition occurs when SSH is started before other services
1. Set `export PACKER_LOG=1` before running build to see logs on stderr
1. If you add an ssh user, Ghostpack is configured to not require a sudo password for that user. This is what AWS does with the `ubuntu` user, but it's generally not a good idea and you shouldn't do it. Pull requests welcome.
1. This relies on AWS Security Groups as your firewall. If you don't have a similar firewall, you should ssh in and install UFW or something similar.
1. I left fail2ban's default settings in place.
1. Can make this pull from the TryGhost Github repo later. Now just grabs the [0.3.3 release from en.ghost.org](https://en.ghost.org/download/).
1. If Packer hangs on a file upload, make sure there's a newline at the end.
1. Included is a Vagrantfile, if you want to use Vagrant to debug locally. You'll need to run the shell commands to install Ansible manually, then `vagrant ssh` and `cd /vagrant_playbook`. The command Packer uses to run Ansible is:  
`ansible-playbook ghost.yml -c local -i "127.0.0.1,"`
