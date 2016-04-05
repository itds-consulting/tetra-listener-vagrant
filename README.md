# TETRA Listener - Vagrant template

This repository provides setup script to get [tetra-listener](https://github.com/itds-consulting/tetra-listener) up and running using Vagrant virtual environment manager

## Install Dependencies

  1. Vagrant 1.6 and later
  2. VMware Workstation or VMware Fusion (VirtualBox has been tested, but has performance problems)
  3. Vagrant VMware plugins (see https://www.vagrantup.com/docs/vmware/installation.html)
    - `vagrant plugin install vagrant-vmware-fusion`
    - or `vagrant plugin install vagrant-vmware-workstation`

## Usage

  1. `git clone https://github.com/itds-consulting/tetra-listener-vagrant`
  2. `cd tetra-listener-vagrant`
  3. edit `Vagrantfile` and change number of CPUs and allocated Memory if you have plenty
  4. `vagrant up --provider vmware_fusion` (or `vmware_workstation`, depending on what you do have installed)
    - `vagrant up --provider virtualbox` if you want to try if your VirtualBox is better
  5. wait for install to be finished (depending on your machine performance can take up to few hours)
  6. `vagrant ssh` (to login into machine)
  7. `tetra-listener` will be installed and ready to use in folder `/home/vagrant/tetra-listener/`

## Access

  - username: vagrant / password: vagrant
  - enabled with sudo permissions (simply type sudo bash to get root shell)
  - mysql user: root (or tetra) / password: tetra

## Tetra-Listener documentation

  See repository https://github.com/itds-consulting/tetra-listener for documentation
