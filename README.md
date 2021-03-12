# ansible-playbooks  
An collection of playbooks for ansible  

# Useage taken from my sys-265 tech journal:  


# Prepare our Hosts and Controller:


## Linux Nodes:
Note: Our nodes will be running Centos, but it should not matter the distro besides the commands for creating a user.  

Add a deployer user that ansible can use and give it root access:  
**MAKE SURE TO MAKE THE PASSWORDS THE SAME ON EVERY NODE**  

`sudo adduser deployer`  
`sudo passwd deployer`  
`sudo usermod -aG wheel deployer`  
 
Now we will create a file that will allow our deployer user to elevate to root without a password:  

`sudo vim /etc/sudoers.d/ansible`  

Add the following line:  
```
deployer          ALL=(ALL)          NOPASSWD: ALL
```
## Controller:
Note: Our controller is running Ubuntu Server 20.04 LTS  

Create a deployer user and give it root access:  
`sudo adduser deployer`  
`sudo usermod -aG sudo deployer`  

Install ansible with:  
`sudo apt install ansible sshpass python3-paramiko`  

Generate an ssh key to use when we log into systems:  
`ssh-keygen -t rsa -C "{key comment goes here}"`  

Copy that key to all of your linux systems:    
`sudo ssh-copy-id -i ~/.ssh/id_rsa.pub deployer@{IP OR HOSTNAME HERE}` 

To give us some time, execute these commands to allow login with the key without prompting for the password for 4 hours:  
`eval$(ssh-agent)`  
`ssh-add -t 14400`  

Be sure to test that you can ssh into each linux host using the key and elevate to root without a password prompt!  




## Windows Nodes:
Install openssh via an admin powershell prompt:  

`Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0`  
`Start-Service sshd`  
`Set-Service -Name sshd -StartupType 'Automatic'`  

Set it so powershell is our default prompt when we ssh in with these **2** commands:  

`Set-ItemProperty "HKLM:\Software\Microsoft\Powershell\1\ShellIds" -Name ConsolePrompting -Value $true`  
  
`New-ItemProperty -Path HKLM:\SOFTWARE\OpenSSH -Name DefaultShell -Value "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" -PropertyType String -Force`  

Now we should be able to ssh into our windows host from our controller!  


# Start Using Ansible

## Create our inventory file

First we need an inventory file. This file holds all of our hosts.  
We can also set groups of hosts to run runbooks with.    
Dont forget the `[windows:vars]` and `ansible_shell_type=powershell` lines if using windows hosts!  
Here is an example inventory.txt:  

On our controller:  

`sudo vim inventory.txt`  

```
[nodeexporter_nodes]
ansible1-thomas
ansible4-thomas
[webmin_nodes]
ansible2-thomas
ansible3-thomas
[windows_nodes]
mgmt01-thomas
wks01-thomas
[windows:vars]
ansible_shell_type=powershell
```
## Testing connection to our nodes:

Now we can test connectivity to our hosts using some of the built in ping modules:

Here we test our nodeexporter_nodes:  
`ansible nodeexporter_nodes -m ping -i inventory.txt` 

Here we test our windows_nodes:
Note: We are using a domain account that can log into all of our nodes.  
`ansible windows -i inventory.txt -m win_ping -u thomas.autiello-adm@thomas.local --ask-pass`  

Note: If you get an error about using a ssh password you can either accept the keys on the hosts or make a file where you are running ansible with the following in it:  

`sudo vim ansible.cfg`  

```
[defaults]
host_key_checking = false
```

## installing ansible images  

Using ansible galaxy we can grab images to use with our playbooks.  
Here is an example of us downloading the node_exporter image to the roles folder:  

`ansible-galaxy install geerlingguy.apache -p roles/`  

## Using Playbooks:

Now that we are all set up we can use runbooks to install some software.  
Some examples [are listed on my repo here](https://github.com/fearherbs1/ansible-playbooks).  

Be sure to check that your host group names are the same as the ones in the playbook!   

For the sample node_exporter linux playbook:  
`ansible-playbook -i inventory.txt roles/nodexporter.yml`  

For the sample windows_software windows playbook:  
`ansible-playbook -i inventory.txt roles/windows_sofware.yml -u thomas.autiello-adm@thomas.local --ask-pass`  


The software has now been installed via our playbooks!

