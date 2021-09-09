# Laradock and WSL2

---

Dev Enviroment - Laradock running at WSL2 + XDebug


### Requirements

- Download & Install <a href="https://github.com/shayne/go-wsl2-host/releases/latest/download/wsl2host.exe">Go WSL2 Host (last release)</a>

```bash
.\wsl2host.exe install
```
**Important** If you perform any windows update you must reinstall the service and reconfigure it.

Font: https://github.com/shayne/go-wsl2-host/releases
## Ubuntu
Install and Configuration of Ubuntu
```bash
- Install Ubuntu from Windows Store
- After install is complete, open that and fill user and password for your account
# Command in powershell
    ### Turn off Ubuntu ###
    wsl --shutdown
    ### Convert wsl into wsl2 ###
    wsl --set-version Ubuntu 2
    
# After this process end you must open again and run next steps
```
Inside of Ubuntu
```bash
# Create file for service WSL2 Hosts ( This sync your domains.local into windows hosts file )
$ nano ~/.wsl2hosts
    # Content of file
        host.docker.internal laradock_mysql_1 mysql ubuntu.wsl
# Remove old instalations
sudo apt-get remove docker docker-engine docker.io containerd runc
# Update the source listing
sudo apt-get update
# Ensure that you have the binaries needed to fetch repo listing
sudo apt-get install apt-transport-https ca-certificates curl gnupg2 software-properties-common
# Fetch the repository listing from docker's site & add it
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
# Update source listing now that we've added Docker's repo
sudo apt-get update
# Install docker-ce
sudo apt-get install docker-ce
# Add your user to docker group
sudo usermod -aG docker $USER
# Create scripts for docker start
sudo nano /usr/local/sbin/start_docker.sh
    with the following contents:
        #!/usr/bin/env bash
        sudo cgroupfs-mount
        sudo service docker start
# Configure script permissions
sudo chmod +x /usr/local/sbin/start_docker.sh
# Lock down edit privileges
sudo chmod 755 /usr/local/sbin/start_docker.sh
/bin/sh /usr/local/sbin/start_docker.sh
sudo nano /etc/sudoers
# And add the following to the bottom of the file — making sure to put in your own username (use echo $USER if you’re unsure what it is):
# Enable docker services to start without sudo
<your username here> ALL=(ALL:ALL) NOPASSWD: /bin/sh /usr/local/sbin/start_docker.sh
# Now you must Reboot your bash or run this command
source ~/.bashrc
# After this commands
sudo service docker stop
sudo nohup docker daemon -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock &
sudo service docker start
docker run --rm hello-world
#If you see hello world thats ok. You can verify if your docker is running using this
    docker ps
```
Now we will configure folders where you go work
```bash
cd ~
mkdir Workspace
cd Workspace; git clone <url of repo with our laradock>
# Configure your git parameters
git config --global user.name "Your Name"
git config --global user.email "put@youremail.there"
# Install composer
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
# go ~/Workspace/laradock and start laradock
cd ~/Workspace/laradock; docker-compose up -d nginx mysql phpmyadmin portainer
This is start containers that we need. Will take some time.... 
After, I advise you to run the following (force permissions)
cd ~
sudo chown -R username:www-data Workspace/
```
## Our Commands
Edit your .bashrc using 
```bash 
$ sudo nano ~/.bashrc
```
Add this content at end of file
```bash
#laradock comands custom
alias server-stop="cd ~/Workspace/laradock/; ./php-fpm/xdebug stop; docker-compose down"
alias apache-start="sudo service docker start; cd ~/Workspace/laradock/; docker-compose up -d apache2 mysql phpmyadmin portainer; ./php-fpm/xdebug start"
alias apache-restart="cd ~/Workspace/laradock/; ./php-fpm/xdebug stop; docker-compose down; docker-compose up -d apache2 mysql phpmyadmin portainer; ./php-fpm/xdebug star>
alias nginx-start="sudo service docker start; cd ~/Workspace/laradock/; docker-compose up -d nginx mysql phpmyadmin portainer; ./php-fpm/xdebug start"
alias nginx-restart="cd ~/Workspace/laradock/; ./php-fpm/xdebug stop; docker-compose down; docker-compose up -d nginx mysql phpmyadmin portainer; ./php-fpm/xdebug start"
alias nginx-reload="cd ~/Workspace/laradock/; docker-compose exec nginx nginx -s reload"
alias server-bash="docker container exec -it laradock_workspace_1 bash"
alias cron-start="cd ~/Workspace/laradock/; docker-compose up -d php-worker"
```
With this commands you can start, restart and turn off containers and docker.
```bash
apache-start # start your containers apache environment
nginx-start # start your containers nginx environment
apache-restart # restart your containers apache environment
nginx-restart # restart your containers nginx environment
server-stop # stop all containers
server-bash # bash of container php-fpm
nginx-reload # when we add a new configuration file to nginx (or run nginx-restart)
cron-start # run php-worker container
```
If you need can add new commands at your file .bashrc
# Setup new Projects
To create a project inside that structure you need create a folder inside ~/Workspace

Example:

```bash
~/Workspace/
    laradock/
    myproject1/
    myproject2/
```
Next, you need create a conf file at nginx / apache2
~/Workspace/laradock/nginx/sites OR ~Workspace/laradock/apache2/sites

Add your domain into ~/.wsl2hosts ( Add new domains always at first line of file)

- Reload system using 
```bash
server-reload
```

Now you can navigate with your browser http://yourdomain.local 

If you have problems 

#### Rebuild container images
```bash
cd ~/Workspace/laradock; docker-compose up --build -d mysql phpmyadmin nginx portainer
```
### **Notes:**

- You can change your configs at .env
- This is configurated to work with xdebug
- At vscode you can add this debug config (example)

```bash
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Listen for XDebug",
            "type": "php",
            "request": "launch",
            "port": 9000,
            "pathMappings": {
                "/var/www/yourprojectfolder": "${workspaceRoot}",
                },
            /* "xdebugSettings": {
                "max_data": -1
            } */
        }
    ]
}
```

Tools:
- <a href="http://ubuntu.wsl:8080/index.php">phpMyAdmin</a>
- <a href="http://ubuntu.wsl:9010/">Portainer</a>

## Testing
```bash
docker ps # if you see containers running after (server-start) your process is complete.
```
## Security
If you discover any security related issues, please email the [author](stephanesoares11@gmail.com) instead of using the issue tracker.
