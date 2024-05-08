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


- Run the playbook

    You can call the playbook passing some parameters via `-e` flag, in this case, we will pass the server group name.
    ```
    # ansible-playbook -i inventory.yml -e "server_group=foreman01" foreman.yml
    ```

    This will install the foreman in the latest version in your target server.


    Here you can see an output example
    ```
    (installer) satellite_foreman_easy-installer]# ansible-playbook -i inventory.yml -e "server_group=foreman01" foreman.yml
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

    [sat65]
    wallsat65.local.net

    [sat66]
    wallsat66.local.net

    [sat67]
    wallsat67.local.net

    [sat68]
    wallsat68.local.net

    [sat69]
    wallsat69.local.net

    [sat610]
    wallsat610.local.net

    [sat611-rhel7]
    wallsat611-rhel7.local.net

    [sat611-rhel8]
    wallsat611-rhel8.local.net
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

    Here you can find some available parameters
    
    - `-e "rhsm_username=your_portal_user_here"`
    - `-e "rhsm_password=your_portal_password_here"`
    - `-e "server_group=sat65"` up to `-e "server_group=sat610"`, this will be according to the name in the `inventory.yml` file.
    - `-e "sat_version=6.5"` up to `-e "sat_version=6.11"`
    - `-e "base_os=rhel8"`, only for rhel8 servers
    - `-e "manifest_path=/path/to/the/manifest.zip"`, to pass the manifest file path


    Let's check some examples


    - Satellite 6.5
    ```
    $ ansible-playbook -i inventory.yml -e "rhsm_username=your_portal_user_here" -e "rhsm_password=your_portal_password_here" -e "server_group=sat65" -e "sat_version=6.5" -e "manifest_path=/path/to/the/manifest.zip" satellite.yml
    ```

    - Satellite 6.6
    ```
    $ ansible-playbook -i inventory.yml -e "rhsm_username=your_portal_user_here" -e "rhsm_password=your_portal_password_here" -e "server_group=sat66" -e "sat_version=6.6" -e "manifest_path=/path/to/the/manifest.zip" satellite.yml
    ```

    - Satellite 6.7
    ```
    $ ansible-playbook -i inventory.yml -e "rhsm_username=your_portal_user_here" -e "rhsm_password=your_portal_password_here" -e "server_group=sat67" -e "sat_version=6.7" -e "manifest_path=/path/to/the/manifest.zip" satellite.yml
    ```

    - Satellite 6.10
    ```
    $ ansible-playbook -i inventory.yml -e "rhsm_username=your_portal_user_here" -e "rhsm_password=your_portal_password_here" -e "server_group=sat610" -e "sat_version=6.10" -e "manifest_path=/path/to/the/manifest.zip" satellite.yml
    ```

    - Satellite 6.11 over RHEL7
    ```
    $ ansible-playbook -i inventory.yml -e "rhsm_username=your_portal_user_here" -e "rhsm_password=your_portal_password_here" -e "server_group=sat611-rhel7" -e "sat_version=6.11" -e "manifest_path=/path/to/the/manifest.zip" satellite.yml
    ```

    - Satellite 6.11 over RHEL8
    ```
    $ ansible-playbook -i inventory.yml -e "rhsm_username=your_portal_user_here" -e "rhsm_password=your_portal_password_here" -e "server_group=sat611-rhel8" -e "sat_version=6.11" -e "manifest_path=/path/to/the/manifest.zip" -e "base_os=rhel8" satellite.yml
    ```

    - Satellite 6.12 over RHEL8
    ```
    $ ansible-playbook -i inventory.yml -e "rhsm_username=your_portal_user_here" -e "rhsm_password=your_portal_password_here" -e "server_group=sat612" -e "sat_version=6.12" -e "manifest_path=/path/to/the/manifest.zip" -e "base_os=rhel8" satellite.yml
    ```

    - Satellite 6.13 over RHEL8
    ```
    $ ansible-playbook -i inventory.yml -e "rhsm_username=your_portal_user_here" -e "rhsm_password=your_portal_password_here" -e "server_group=sat613" -e "sat_version=6.13" -e "manifest_path=/path/to/the/manifest.zip" -e "base_os=rhel8" satellite.yml
    ```

    - Satellite 6.14 over RHEL8
    ```
    $ ansible-playbook -i inventory.yml -e "rhsm_username=your_portal_user_here" -e "rhsm_password=your_portal_password_here" -e "server_group=sat614" -e "sat_version=6.14" -e "manifest_path=/path/to/the/manifest.zip" -e "base_os=rhel8" satellite.yml
    ```

    - Satellite 6.15 over RHEL8
    ```
    $ ansible-playbook -i inventory.yml -e "rhsm_username=your_portal_user_here" -e "rhsm_password=your_portal_password_here" -e "server_group=sat615" -e "sat_version=6.15" -e "manifest_path=/path/to/the/manifest.zip" -e "base_os=rhel8" satellite.yml    
    ```

    Note. `base_os="rhel7"` will be the standard value. It will be necessary to pass it only when installing Satellite over `RHEL8`.


    Ps.: Soon I'll be adding the information presenting how to use vault feature.


    Here you can see an output example
    ```
    (installer) satellite_foreman_easy-installer]# ansible-playbook -i inventory.yml -e "rhsm_username=your_portal_user_here" -e "rhsm_password=your_portal_password_here" -e "server_group=satellite01" -e "sat_version=6.10" satellite.yml
    ...
    PLAY RECAP **********************************************************************************************************
    10.10.10.20                : ok=12   changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
    ```

    Once you get this output, you should be able to access your server via `https://10.10.10.20` and be able to authenticate in your fresh Satellite server.


    On both cases, if for any reason you see error during the process, please, check the error, if you see no reason for that, feel free to rerun the command, probably it will work as expected in the second run.
