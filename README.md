# kali-vagrant
The reason why we use kali in dedicated environments is so we can't contaminate the environments. We can use kali both for offensive as defensive purposes.

# Setting up your environment
1. Make sure you have a working [virtualbox](https://www.virtualbox.org/wiki/Downloads).
2. Make sure you have a working [vagrant](https://www.vagrantup.com/downloads).
3. Make sure you have [ansible](https://docs.ansible.com/ansible/latest/installation_guide/index.html) installed on the host system.

# Pulling the vagrant box for kali
To pull the kali you do ```vagrant box add kalilinux/rolling```. That will pull [kali](https://app.vagrantup.com/kalilinux/boxes/rolling) from the Hashicorp Vagrant Cloud.

# Running your kali box
First thing to do is create your project folder. Typically this will be the customer name with an identification number.

```
$ mkdir <project>
$ cd <project>
```

## Initializing the Vagrantfile
Next thing to do is to initialize the Vagrantfile
```$ vagrant init kalilinux/rolling```

That will create a file called Vagrantfile we need to adapt for our purposes.

## Configuration of the Vagrantfile
Since your box will need some resource you can either go with the default or configure a couple of parameters. We recommend you to have a look at the Vagrant [announcement](https://www.kali.org/news/announcing-kali-for-vagrant/) of Kali to get an idea of what you can configure.

## Starting the virtual machine
To start the virtual machine you do ```vagrant up```.

## Stopping the virtual machine
To stop the virtual machine you do ```vagrant halt```.

## Destroying the virtual machine.
To destroy the virtual machine you do ```vagrant destroy```.

In order not to loose your data you need to alter the Vagrantfile and edit to your needs so that you are able to "export" the data before you destroy the machine.
```
  # Share an additional folder to the guest VM. 
  # The first argument is the path on the host to the actual folder.
  # The second argument is the path on the guest to mount the folder.
  # And the optional third argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"
```

There is also a ```/vagrant``` added to the box, this is a mount point shared between the host and the guest. It corresponds with the project folder.
The owner and group for ```/vagrant``` are root.

Another ways to handle the data "export" is to use git repositories, file sharing platforms, etc. 

# Connecting to the box
The vagrant box provided by [Offensive Security](https://www.offensive-security.com/) is a standard kali virtual machine for vmware. The only thing to take into account is that the default kali/kali credentials are replaced by vagrant/vagrant.

## Connection over SSH
To connect over ssh you do ```vagrant ssh``` in the project and this will connect you to the virtual machine

## Connection using the GUI
You can also use the virtual machine using the GUI.

# Provisioning the box for our purpose
## Updating the box during initialization
We can perfectly tell vagrant to upgrade the kali linux to the latest and greatest linux kernel and update all packages. To do this we specify the command line commands to do this.

```
config.vm.provision "shell", inline: <<-SHELL
     # make the installation non-interactive
     echo 'debconf debconf/frontend select Noninteractive' | sudo debconf-set-selections
     # run the upgrade process (this takes a while)
     apt-get update -y
     apt-get install -y python-is-python3
     apt-get dist-upgrade -y
     apt-get autoremove -y
     apt-get autoclean -y
     SHELL
```

The reason why we do this part of the process through the command shell is because this is a specific part to the kali linux environment where the installation of our software needs is rather generic and thus we can leverage Ansible like we would try to leverage it on any other environment.

The first line will simply update your local repositories. The next line is to make certain python3 is the default environment. If you would not do this the python environment would still be python2. Next we update our distribution and do a clean up. We use one line per command to increase readability.

## Installing our software needs using Ansible
This is the last part of code we add to our Vagrantfile.

```
config.vm.provision "ansible" do |ansible|
      ansible.limit = "all, localhost"
      ansible.verbose = "v"
      ansible.playbook = "playbook.yaml"
  end
```

It will use our playbook.yaml in the project directory and our host's installation of Ansible.

# Kali Training
If you need training, the Offensive Security people have a [good start place](https://kali.training/lessons/introduction/).

# Security Training
If you need security training and want to work with us, contact us at https://theshell.company/
