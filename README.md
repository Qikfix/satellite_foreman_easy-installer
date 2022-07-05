# Satellite and Foreman Easy Installer

Hi all

This project will help you to install your Foreman or Satellite, easily!

The goal of this project is to present how to use collections, in a way that you can install `Foreman` or `Red Hat Satellite` easily. That said, let's start.



# Requirements
- One server that will be with storing the play + collections
- The machine that will be installed the `Foreman` or `Red Hat Satellite`

---

## The server
At this moment, just installing the packages on RHEL7 vanilla was not enough, so, let's proceed with the below step that will be valid for any environment.

- Installing the RHEL7

    Here you can proceed with a basic/minimal installation of RHEL7. You don't need too much memory or storage, a simple configuration will be enough.

- Creating the pub ssh key

    You will need to ssh your clients in order to use the plays. That said, at this moment, let's execute the step in a sequence to create the pub-key
    ```
    # ssh-keygen
    ```
    here you can see the output
    ```
    # ssh-keygen 
    Generating public/private rsa key pair.
    Enter file in which to save the key (/root/.ssh/id_rsa): 
    Enter passphrase (empty for no passphrase): 
    Enter same passphrase again: 
    Your identification has been saved in /root/.ssh/id_rsa.
    Your public key has been saved in /root/.ssh/id_rsa.pub.
    The key fingerprint is:
    SHA256:WzpSZPj8g4qtZEZD7U05Gsma27ncml5XHmsvQ6Kee/U root@waldirio-prov-test.lab.eng.rdu2.redhat.com
    The key's randomart image is:
    +---[RSA 2048]----+
    |                 |
    |     o o .       |
    |    . * *        |
    |   . + X .       |
    |    = o S . o    |
    |   . + o *.ooo   |
    |    = + *.+++.   |
    |   + + B.+.oo.E  |
    |    ooOo=o   o.  |
    +----[SHA256]-----+
    ```

- Installing python3 and git

    Assuming that you are consuming content from the basic repo, just run the command below to install python3, git and optionally vim
    ```
    # yum install python3 -y
    # yum install git -y
    # yum install vim -y
    ```

- Creating the virtual environment

    Awesome. Once you have python3 installed, we can proceed creating the virtual environment.
    ```
    # python3 -m venv ~/.virtualenv/installer
    ```

- Installing the modules

    At this moment, let's load the virtual environment and install some modules that will be necessary for this project
    ```
    # source ~/.virtualenv/installer/bin/activate
    (installer) # pip install --upgrade pip
    (installer) # pip install ansible
    ```

- Cloning the repo

    Now, we can move on and sync the repo, this will clone the initial templates that will help you in this initial process.

    ```
    # cd ~
    # mkdir code
    # cd code
    # git clone https://github.com/waldirio/satellite_foreman_easy-installer.git
    # cd satellite_foreman_easy-installer
    ```

- Installing the dependencies using the answer file

    Inside the directory, you will be able to see the file named `requirements.yml`. This file contains some dependencies that will be required for this project, in fact, it will be installing the collections that will help us to install `Foreman` and/or `Red Hat Satellite`. That said, let's install them.
    ```
    (installer) # ansible-galaxy install -r requirements.yml
    ```

Ok, at this moment, the server that will be pushing all the packages and commands it's ready to go. The next step will be how to use the play.

---

## How to use it?

### Foreman

- Install your destination server

    For example, CentOS 7.9, with the IP address `10.10.10.10`


- Create the entry in the inventory.yml file

    Here you can see an example, add the ip address according to your target server
    ```
    # cat inventory.yml 
    [foreman01]
    10.10.10.10
    ```

- Copy your pub-key to the external server

    You can copy the pubkey using the ssh-copy-id command
    ```
    # ssh-copy-id root@10.10.10.10
    ```

    After this step, you should be able to run a command on the remote server with no necessity of password

    ```
    # ssh root@10.10.10.10 hostname
    ```

    The response of the above command should be the remote hostname.


- Set the target in your playbook

    Edit the foreman.yml file and update the line `hosts` accordingly
    ```
    hosts: foreman01
    ```

- Run the playbook

    You can call the playbook passing some parameters via `-e` flag, in this case, we are not passing it.
    ```
    # ansible-playbook -i inventory.yml foreman.yml
    ```

    This will install the foreman in the latest version in your target server.


    Here you can see an output example
    ```
    (installer) satellite_foreman_easy-installer]# ansible-playbook -i inventory.yml foreman.yml
    ...
    PLAY RECAP **********************************************************************************************************
    10.10.10.10                : ok=12   changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
    ```

    Once you get this output, you should be able to access your server via `https://10.10.10.10` and be able to authenticate in your fresh Foreman server.


---

### Satellite

- Install your destination server

    For example, RHEL 7.9, with the IP address `10.10.10.20`


- Create the entry in the inventory.yml file

    Here you can see an example, add the ip address according to your target server
    ```
    # cat inventory.yml 
    [satellite01]
    10.10.10.20
    ```

- Copy your pub-key to the external server

    You can copy the pubkey using the ssh-copy-id command
    ```
    # ssh-copy-id root@10.10.10.20
    ```

    After this step, you should be able to run a command on the remote server with no necessity of password

    ```
    # ssh root@10.10.10.20 hostname
    ```

    The response of the above command should be the remote hostname.


- Set the target in your playbook

    Edit the satellite.yml file and update the line `hosts` accordingly
    ```
    hosts: satellite01
    ```

- The manifest file

    Before you run the Satellite playbook, please, keep in mind that a manifest will be required, that said, you can access your customer portal, create a manifest and overwrite the file `manifest.zip` under the folder `files`. The current one it's just an empty file to be used as reference.


- Run the playbook

    One attention point here, in this case, it will be necessary to authenticate the server in order to register through RHSM. In this case, you can set the credentials in two different ways, via `hard code` or `parameter`. Let's check both.

    Via `hard code`, you can open the file `satellite.yml` and update the follow lines
    ```
    rhsm_username: "your_portal_username_here"
    rhsm_password: "your_portal_password_here"
    ```

    Doing that, you can safely execute the command below and your Satellite will be installed with no issues.
    ```
    # ansible-playbook -i inventory.yml satellite.yml
    ```

    Or

    Via `parameter`, you don't need to edit the file `satellite.yml`, instead, you can run the command as below
    ```
    # ansible-playbook -i inventory.yml -e "rhsm_username=your_portal_user_here" -e "rhsm_password=your_portal_password_here" satellite.yml 
    ```

    Ps.: Soon I'll be adding the info presenting how to use vault.


    Here you can see an output example
    ```
    (installer) satellite_foreman_easy-installer]# ansible-playbook -i inventory.yml satellite.yml
    ...
    PLAY RECAP **********************************************************************************************************
    10.10.10.20                : ok=12   changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
    ```

    Once you get this output, you should be able to access your server via `https://10.10.10.20` and be able to authenticate in your fresh Satellite server.



