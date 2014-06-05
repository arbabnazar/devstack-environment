devstack-environment
====================

A single- or multi-node DevStack deployment driven by Vagrant and Ansible

This allows you to bring up a complete single- or multi-node DevStack
deployment with just a single command.


Prerequisites
=============
You need to have installed:

    - Virtualbox
    - Vagrant
    - Ansible

On Ubuntu based systems, you should be able to do this:

    $ apt-get install virtualbox vagrant ansible

If you want to make sure you have the latest version of Ansible (highly
recommended), and you are still running an older version of Ubuntu (such
as 12.04, for example), you could instead do:

    $ apt-get install virtualbox vagrant python-pip
    $ pip install ansible


Instructions
============

    1. Clone this repository:

        $ git clone git@github.com:CiscoSystems/devstack-environment.git

    2. Change into the directory:

        $ cd devstack-environment

    3. Start the virtual machines:

        $ vagrant up

       Depending on your network connection, this step can take more than
       an hour, so please be patient.

    4. If everything has completed without problem, you can log into the
       controller and compute host, like so:

        $ vargant ssh controller
        $ vagrant ssh compute-1

You will then be logged in as user 'vagrant'. In your home directory you
have a 'workspace' directory, inside of which you will find 'log' and
'devstack'. On the controller, after changeing into the devstack directory,
you can run 'source openrc admin demo', after which you can issue various
OpenStack commands.


What does it do?
================
Have a look at the file 'Vagrantfile'. It describes to Vagrant which virtual
machines it should create. You can see three machines: One just serves as
an apt and pip cache (very useful), the other two are a controller and a
compute host (please note that the controller also acts as a compute host,
so OpenStack guests can also run on the controller machine).

After the VMs are created, they are going to be provisioned via Ansible
playbooks.

If you have your machines created already and just wish to re-run the
Ansible provisioning, you can just do:

    $ vagrant provision

If you wish to destroy the compute and controller host and restart just
those two again, you can do these steps:

    $ vagrant destroy controller compute-1
    $ vagrant up
    $ vagrant provision

Note that we are avoiding destruction of the cache host. No need to do so.

FAQ
===
What if I want different IP addresses for my hosts?
---------------------------------------------------
In the Vargantfile you find this section:

    servers = {
                :"controller"      => '192.168.99.11',
                :"compute-1"       => '192.168.99.22',
                :"apt-pip-cache"   => '192.168.99.99',
              }
    last_host_name = "apt-pip-cache"
    last_host_ip   = "192.168.99.99"

This is where the IP addresses of the machines are defined. You can change
those to some other private addresses that you may prefer.

Please note that you will also have to edit the "vagrant_hosts_multi" file,
specfically this section:

    [all-hosts]
    controller        ansible_ssh_host=192.168.99.11
    compute-1         ansible_ssh_host=192.168.99.22
    apt-pip-cache     ansible_ssh_host=192.168.99.99

Adjust the IP addresses to whatever values you have chosen.


What if I just want a single-node DevStack? Or a 5-node DevStack?
-----------------------------------------------------------------
In that case, look at that same section that we have described above in the
Vagrantfile. Comment our (or remove) the 'compute-1' host. Likewise, do the
same in the "vagrant_hosts_multi" file. In that case, find the start of the
'[compute-hosts'] section in that file and remove the compute-1 entry.

If you want more compte hosts, add them in the Vargantfile as well as the
"vagrant_hosts_multi" file.


Where can I determine what version of DevStack I'm deploying?
-------------------------------------------------------------
Find the "vars/vagrant_extra_vars.yml" file and look for the DEVSTACK section.


What if I want to use different modules/tunnels/settings for DevStack?
----------------------------------------------------------------------
The DevStack settings are controller by their 'localrc' files. Please have a
look at the 'playbooks/templates' directory. In that directory, you can see
sub-directories, such as "gre". Now look inside there and you can find template
files, one for the controller and one for the compute hosts. You can adjust the
settings there as you wish.

Now look back in the "vars/vagrant_extra_vars.yml" file and find the
DEVSTACK.tunnel_type setting. That setting may be "gre", for example. The
value of this setting determines the sub-directory out of which the localrc
files are taken. So, if you want to create your own configs, create a new
sub-directory in the templates directory, put your localrc templates for
compute and controller hosts in there and adjust. Then change the 'tunnel_type'
setting to the name of your new sub-directory.


