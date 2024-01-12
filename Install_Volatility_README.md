# Install Volatility on a remote host using Ansible

Volatility is open source memory forensics platform which is used of incident response and malware analysis.

Ansible is a open source automation tool, that enables software provisioning, configuration management, application deployement functionality and etc.

## Prerequistes

1. For this setup we need to have a controller and a host machine.
2. Make sure anisble is installed on the controller.
3. Make sure passwordless authentication is enabled on remote hosts.

## Installation

To install ansible on controller, use the below command (my instance is running on ubuntu).

```bash
sudo apt install ansible
```

To check which version of ansible is installed/present.
```bash
ansible --version
```

## Enabling passwordless authentication

To enable passwordless authentication use below steps:

```bash
ssh-keygen ## which generates public/private key pair.
ubuntu@controller:~$ cd /home/ubuntu/.ssh/
ubuntu@controller:~/.ssh$ ls
authorized_keys  id_rsa  id_rsa.pub
ubuntu@controller:~/.ssh$ pwd
/home/ubuntu/.ssh
ubuntu@controller:~/.ssh$ cat id_rsa.pub
```

copy the key and paste the key contents in authorized_keys on target server
on target server generate key using ssh-keygen
```bash
ssh-keygen
ubuntu@remotehost:~$ cd /home/ubuntu/.ssh
ubuntu@remotehost:~/.ssh$ ls
authorized_keys  id_rsa  id_rsa.pub
ubuntu@remotehost:~/.ssh$ vi authorized_keys
```
now on controller server ssh to target server and it should authenticate without password.

## Ansible playbook to download and install Volatility2 on remote host
Ansible playbooks are in written in YAML

```bash
---
 - name: This playbook is to install volatility2
   hosts: all
   become: true
   tasks:
     - name: Install unzip
       apt:
         name: unzip
         state: present

     - name: download volatility2
       ansible.builtin.unarchive:
         src: http://downloads.volatilityfoundation.org/releases/2.6/volatility_2.6_lin64_standalone.zip
         dest: /home/ubuntu/volatility2
         remote_src: yes

     - name: change ownership
       ansible.builtin.shell: |
         cd /home/ubuntu/volatility2
         chown -R ubuntu volatility_2.6_lin64_standalone
         cd volatility_2.6_lin64_standalone
         ls -ltr
         mv volatility_2.6_lin64_standalone vol2
```

## Execution of Ansible playbook

```bash
ansible-playbook install_volatility.yml -i inventory
```
install_volatilty.yml is the anisble playbook.
inventory is the inventory file which contains the list of remote hosts on which anisble playbook is executed.

once the playbook is sucessfully executed, login to remote host and check volatility2 is installed as below.

```bash
ubuntu@remotehost:~/volatility2/volatility_2.6_lin64_standalone$ ./vol2 -h
Volatility Foundation Volatility Framework 2.6
Usage: Volatility - A memory forensics analysis platform.

Options:
  -h, --help            list all available options and their default values.
                        Default values may be set in the configuration file
                        (/etc/volatilityrc)
  --conf-file=/home/ubuntu/.volatilityrc
                        User based configuration file
  -d, --debug           Debug volatility
  --plugins=PLUGINS     Additional plugin directories to use (colon separated)
  --info                Print information about all registered objects
  --cache-directory=/home/ubuntu/.cache/volatility
                        Directory where cache files are stored
  --cache               Use caching
  --tz=TZ               Sets the (Olson) timezone for displaying timestamps
                        using pytz (if installed) or tzset
  -f FILENAME, --filename=FILENAME
                        Filename to use when opening an image
  --profile=WinXPSP2x86
                        Name of the profile to load (use --info to see a list
                        of supported profiles)
  -l LOCATION, --location=LOCATION
                        A URN location from which to load an address space
  -w, --write           Enable write support
  --dtb=DTB             DTB Address
  --shift=SHIFT         Mac KASLR shift address
  --output=text         Output in this format (support is module specific, see
                        the Module Output Options below)
  --output-file=OUTPUT_FILE
                        Write output in this file
  -v, --verbose         Verbose information
  -g KDBG, --kdbg=KDBG  Specify a KDBG virtual address (Note: for 64-bit
                        Windows 8 and above this is the address of
                        KdCopyDataBlock)
  --force               Force utilization of suspect profile
  --cookie=COOKIE       Specify the address of nt!ObHeaderCookie (valid for
                        Windows 10 only)
  -k KPCR, --kpcr=KPCR  Specify a specific KPCR address
```


