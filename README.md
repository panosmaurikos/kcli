### KCLI ###
KCLI is a powerful command-line tool designed to enable management of virtual environments, including virtual machines and containers, for various purposes such as development, testing, and production. It allows users to specify the type and quantity of virtual machines and their configurations through a simple configuration file. KCLI automates the setup and management of these virtual machines, saving time and effort. In addition, it offers the ability to start, stop, connect to virtual machines and create snapshots. It supports various virtualization providers and Kubernetes cluster types. Finally, it allows the use of custom templates for virtual machines and integration into workflows and scripts.

![kcli](https://github.com/panosmaurikos/kcli/assets/72733593/20ad67f6-2438-4b0a-847b-9ca04f53e311)


### Kcli installation ###

1. Install Kcli:
~~~
 curl -s https://raw.githubusercontent.com/karmab/kcli/main/install.sh | bash
~~~
This command installs Kcli using a script from the GitHub repository.

2. Install Required Packages:

~~~
sudo apt install -y qemu-kvm virt-manager libvirt-daemon-system virtinst libvirt-clients bridge-utils
~~~
Installs necessary packages for Kcli, including QEMU-KVM, Virt-Manager, and libvirt components.

3. Enable and Start libvirtd Service:
~~~
sudo systemctl enable --now libvirtd
sudo systemctl start libvirtd
~~~
These commands enable and start the libvirtd service, which is required for managing virtual machines.

4. Check libvirtd Status:

~~~
sudo systemctl status libvirtd
~~~
This command checks the status of the libvirtd service to ensure it's running correctly.

5. Add User to 'kvm' and 'libvirt' Groups:

~~~
sudo usermod -aG kvm $USER
sudo usermod -aG libvirt $USER
~~~
These commands add the current user to the 'kvm' and 'libvirt' groups, allowing the user to manage virtualization.




6. Create a Storage Pool:
~~~
kcli create pool -p /var/lib/libvirt/images default
~~~
This command creates a storage pool named 'default' with the path '/var/lib/libvirt/images' to store virtual machine images or others templates.

Now let's see some examples for how to use kcli


## Example 1  ##
### Create a Ubuntu VM  ###

1. Download an Ubuntu 22.04 Image:
~~~
kcli download image ubuntu2204
~~~
This command downloads an Ubuntu 22.04 image that can be used for creating virtual machines.

2. Create a Virtual Machine:
~~~
kcli create vm -i ubuntu2204 myvm
~~~
This command creates a virtual machine named 'myvm' using the Ubuntu 22.04 image.

3. List Virtual Machines:

~~~
kcli list vm
~~~
Lists the virtual machines, including the newly created 'myvm'.

4. SSH into the Virtual Machine:
~~~
kcli ssh myvm
~~~
This command allows you to SSH into the 'myvm' virtual machine.

**_NOTE:_**  If you want to create a vm with custom resources:
~~~
kcli create vm -i ubuntu2204 myvm -p numcpus=5 -p memory=8192 -p disk=20
~~~


## Example 2  ##
### Create VM with custom template ###

We create a vm based on the following profile file and with any cloud image we want.
~~~
myubuntu:
  client:
    - dell03
    - dell04
  image: ubuntu2204
  numcpus: 1
  disks:
    - size: 10
  nets:
    - name: br0
  cmds:
    - echo "root:unix1234" | chpasswd

~~~

1. Create VM and view the list.
~~~
kcli -c local create vm -p myubuntu ubuntu2

kcli -c local list vm 
~~~

2. Login with ssh in VM with default user.(Should you have a pair of keys in id_rsa, id_rsa.pub).

~~~
ssh -i ~/.kcli/id_rsa ubuntu@ubuntu2
~~~

3. For testing reason we can create a test file inside in VM.(optional).

~~~
echo "DEMO" > test2.txt
~~~

4. Restart cloud init and delete machine-id.(In VM)

~~~
sudo -i   

truncate -s0 /etc/hostname
hostnamectl set-hostname localhost
rm /etc/netplan/50-cloud-init.yaml
truncate -s0 /etc/machine-id
rm /var/lib/dbus/machine-id
ln -s /etc/machine-id /var/lib/dbus/machine-id
cloud-init clean
truncate -s0 ~/.bash_history
history -c
shutdown -h now
~~~

5.Export the template.

~~~
kcli -c local export ubuntu2 -i ubuntu2
~~~

6. In the profile file add the full path of the new image.
~~~
    myubuntu:
  client:
    - dell03
    - dell04
  image: ubuntu2204
  numcpus: 1
  disks:
    - size: 10
  nets:
    - name: br0
  cmds:
    - echo "root:unix1234" | chpasswd

myotherubuntu:
 client: local
 image: /var/lib/libvirt/images/ubuntu2_0.qcow2

~~~

7. Crete a new VM with the new template.

~~~
kcli -c local create vm -p myotherubuntu ubuntu3 
~~~

8. Login with ssh in VM with default user.

~~~
kcli ubuntu3
~~~

9. Check if the file  test.txt exist in the new VM.

~~~
nano test.txt
~~~

**_NOTE:_**   To be able to properly play the network interface part with cloud-init must have word ubuntu. 

## Example 3  ##
### Create VM from a Plan ###

Plans allow us to build multiple objects like Vms, Containers

1. Create a plan.

~~~
nano image.yaml

mycentos:
 type: image
 url: https://cloud.centos.org/centos/8-stream/x86_64/images/CentOS-Stream-GenericCloud-8-20210603.0.x86_64.qcow2

vm1:
  memory: 512
  numcpus: 2
  nets: 
   - default
  pool: default
  image: mycentos


  kcli create plan -f image.yaml myplan
~~~

2. Check the vms with:
~~~
kcli list vms
~~~


3. Login with ssh in VM.

~~~
kcli ssh vm1
~~~
