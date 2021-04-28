# Install Docker using repository

## Set up repository

### Update `apt` package

```
sudo apt-get update
sudo apt-get install \ 
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

### Add Docker's official GPG key

```
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

### Set up stable repository

```
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### Install Docker Engine

```
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```


### Test if installation is successed

```
sudo docker run hello-world
```

### Set Docker command to execute without sudo

```
# Add username to docker group
sudo usermod -aG docker ${USER}
# Log out of the server to apply the new group membership
su - ${USER}
```