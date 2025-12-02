# A Guide to install Docker Engine on Ubuntu 22.04

**Docker** is the industry standard for containerization—a software deployment process that bundles an application's code with all the files and libraries it needs to run on any infrastructure. While Ubuntu has a version of Docker in its default repository (`docker.io`), it is often outdated. his guide will show you how to install the latest Docker Community Edition (CE) from the official Docker repository.

## I. Docker Installation

### 1. Removing Old Version (If Any)

Ensureyour system is clean by removing any older, pre-installed versions of Docker that might conflict with the new installation.

```sh
sudo apt remove -y docker docker-engine docker.io containerd runc
```

It is okay if `apt` reports that none of these packages are installed.

### 2. Updating APT and Installing Dependencies

Update `apt` and install prerequisites:

```sh
sudo apt update -y
sudo apt install -y ca-certificates curl gnupg lsb-release
```

Add Docker's official GPG key and repository URL to your system's package manager.

```sh
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

Set up the Docker `apt` repository

```sh
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```

### 3. Installing Docker Engine

```sh
sudo apt update -y
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

The Docker service starts automatically after installation. To verify that Docker is running, use:

```sh
sudo systemctl status docker
```

If not, you can manually start the Docker service:

```sh
sudo systemctl start docker
```

## II. Post-Instation Steps for Docker Engine

### 1. Manage Docker as a Non-Root User

By default, running Docker commands requires `sudo`. To avoid typing `sudo` for every command, add your current user to the `docker` group.

Create the docker group.

```sh
sudo groupadd docker
```

Add your user to the `docker` group.

```sh
sudo usermod -aG docker $USER
```

Log out and log back in so that your group membership is re-evaluated. You can also run the following command to activate the changes to groups:

```sh
newgrp docker
```

### 2. Configuring Docker to Start on Boot with `systemd` (Optional)

On Ubuntu, the Docker service start on boot by default. To automatically start Docker and containerd on boot for other Linux distributions using systemd, run the following commands:

```sh
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

To stop this behavior, use `disable` instead.

```sh
sudo systemctl disable docker.service
sudo systemctl disable containerd.service
```

## III. Verifying Installation

### 1. Checking the Docker version.

```sh
docker --version
```

You should see something like:

```txt
Docker version 29.1.1, build ...
```

### 2. Running Test Container

Run the "Hello World" image to comfirm that Docker is installed and running correctly.

```sh
docker run hello-world
```
If successful, you will see a message saying: **"Hello from Docker! This message shows that your installation appears to be working correctly."**

If you initially ran Docker CLI commands using `sudo` before adding your user to the `docker` group, you may see the following error:

```txt
WARNING: Error loading config file: /home/user/.docker/config.json -
stat /home/user/.docker/config.json: permission denied
```

This error indicates that the permission settings for the `~/.docker/` directory are incorrect, due to having used the sudo command earlier.

To fix this problem, either remove the `~/.docker/` directory (it's recreated automatically, but any custom settings are lost), or change its ownership and permissions using the following commands:

```sh
sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
sudo chmod g+rwx "$HOME/.docker" -R
```

### 3. Verifying Docker Compose

```sh
docker compose version
```

The output should be something like **"Docker Compose version v2.40.3."**

## IV. Uninstalling Docker (If Needed)

To remove Docker Engine, CLI, and containerd:

```sh
sudo apt purge -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
```

### V. Basic Docker Commands

```sh
# List images
docker images

# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Remove a container
docker rm <container_id>

# Remove an image
docker rmi <image_id>
```
## Conclusion

You now have a fully functional Docker environment on Ubuntu 22.04. You can start pulling images, running containers, and building your own Dockerfiles.
