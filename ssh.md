# Setting up SSH on a remote server

Goal: Able to log into your remote server with a single command

## Create a new user on remote server

Root login is not adviced, create a sudo user logging in via that user is. 

We will create a new user called `manu` and will be using that for all the steps
here.

Steps:  
1. Creating new user
`adduser manu`
2. Give the user root privileges
`usermod -aG sudo manu`


## Generating ssh key for server on your LOCAL machine

We will create a new ssh key for the server on your local machine, even if you
have an existing key. Will then use that for authentication.

CAUTION: Don't generate ssh key without specifying a file if you already have
an existing key. This will overwrite it.

Steps:
1. Generate ssh key for the server
`ssh-keygen -t rsa -f ~/.ssh/vps`
Here I am naming the keyfile as `vps`, use whatever name you like.

2. Add the ssh key to ssh agent, this will prevent keyphrase prompt.
`ssh-add ~/.ssh/vps`

3. Restart your local ssh 
`sudo launchctl stop com.openssh.sshd`

## Add the key to VPS

Will now add the key to the user we just created on VPS

`ssh-copy-id -i ~/.ssh/vps manu@<VPS.IP.HERE>`

## Change allowed ssh keys in the vps

1. Log into server
2. Switch to root
3. `sudo vi /etc/ssh_config` 
  - Change
  ```
    AuthorizedKeysFile /home/manu/.ssh/authorized_keys
    PasswordAuthentication no 
    PermitRootLogin no
  ```
4. `sudo service ssh restart`

## Add host to ssh config
```
Host <desired_name>
  Hostname <VPS.IP>
  User manu
  Port 22
  IdentityFile ~/.ssh/vps
``` 

## Test

`ssh <desired_name>`
