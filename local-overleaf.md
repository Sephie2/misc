## Install Linux in VM

- Install Ubuntu or Debian (WSL / Hyper-V)
	- If you are using Debian, you must then install sudo and add the user to the sudoer group. This is already included in Ubuntu

```bash
apt update && apt install sudo
usermod -aG sudo USERNAME
```
- You must then log out and log in again for the change to take effect

## Install Docker

```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

- Check if Docker was installed successfully
```bash
sudo docker run hello-world
```

### System user for Docker (so you don't need sudo)

```bash
sudo addgroup --system docker
sudo adduser USERNAME docker
newgrp docker
```

## SSH auf die VM

- install net-tools to make `ifconfig` work. The install command will be displayed when you run `ifconfig`
- then use `ifconfig` to determine your own IP
- **Hyper-V → external network switch? So that you can also access the OS from outside via the IP. This is not the case with the default switch!**

- Install ssh-server in Debian
```bash
sudo apt update && apt install openssh-server -y
sudo systemctl enable ssh
sudo systemctl start ssh
sudo systemctl status ssh
```

## Install Overleaf Container

- download the Overleaf-toolkit
```bash
cd ~
git clone https://github.com/overleaf/toolkit.git ./overleaf
cd overleaf
bin/init
```

- in the `~/overleaf/config/overleaf.rc` set the variables `OVERLEAF_LISTEN_IP=`,`NGINX_HTTP_LISTEN_IP=`, `NGINX_TLS_LISTEN_IP=` to the externally accessible IP address that we determined with `ifconfig`. For example: 192.168.178.15
- The variable `SIBLING_CONTAINERS_ENABLED=false` (wrong default setting (true), otherwise a warning message is displayed when starting)
```bash
bin/start (start in server mode)
bin/up  (start in console mode)
bin/stop (stops all containers)
```


## Upgrade Container (Update Overleaf)
```
docker-compose stop sharelatex
docker-compose rm sharelatex
docker pull sharelatex/sharelatex:latest
```
alternative
```bash
bin/upgrade
```

## Add full TexLive image (All LaTeX packages)

- Update tlmgr: 
```bash
docker exec sharelatex tlmgr update --self
```

- install texlive-full scheme (this will take a while)
```bash
docker exec sharelatex tlmgr install scheme-full
docker exec sharelatex tlmgr path add
docker commit sharelatex sharelatex/sharelatex:with-texlive-full
```

- Add `-with-texlive-full` to the end of the version number in `config/version` and run "bin/stop" and "bin/start" to restart Overleaf
- Adding the full TexLive must be done again after each container upgrade
