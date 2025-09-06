# A Guide to Remote SSH Setup on Ubuntu 22.04

This post guides you to setup **Remote SSH** so that you can connect to **another Linux machine or server**, e.g., GPU server, cloud VM, cluster node.

There are two parts:

1. Configure SSH in the **local machine** (client).
2. Ensure the **remote machine** (server) has SSH access enabled.

## I. Local Ubuntu Setup (SSH Client)

Install openssh-client first.

```sh
sudo apt update -y
sudo apt install -y openssh-client
```

Then, generate **SSH key** (if you don't have one yet).

```sh
ssh-keygen -t rsa -b 4096 -C "you_email@example.com"
```

Just enter `ENTER` to accept the default path `~/.ssh/id_rsa`. (You can set as passphrase for security or leave it blank).

## II. Remote Machine Setup (SSH Server)

On the remote server, make sure `openssh-server` is installed.

```sh
sudo apt update -y
sudo apt install -y openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh
```

Find the server's IP.

```sh
ip a
```

Or find its public IP via:

```sh
curl https://ipinfo.io/ip
```

By default, the ssh port is `22`, but you need to check the SSH port in the server's network.

## III. SSH Remote Connnection

Back on the **local machine**, run the follow command:

```sh
ssh-copy-id -p port username@remote_ip
```

This allows passwordless login via SSH. To connect the server, run the command:

```sh
ssh -p port username@remote_ip
```

You should now be inside the remote machine — secured with SSH keys!

## IV. (Optional) `~/.ssh/config` Setup for Aliases

This is great for managing multiple servers with different ports or usernames. Edit SSH config file:

```sh
nano ~/.ssh/config
```

Add an entry:

```ini
Host alias_name
    HostName remote_ip
    User remote_username
    Port remote_port
```

Now, save the file (`CTRL + S` and `CTRL + X` to save and close) and you can simply run:

```sh
ssh alias_name
```

## V. Tips

### 1. Using `scp` Command

The `scp` command is used to transfer files between the local machine and a remote server over SSH.

- Copy files from local machine → remote server.

```sh
scp -P port local_file user@remote_ip:/remote/path/
```

- Copy files from remote server → local machine.

```sh
scp -P port user@remote_ip:/remote/path/ /local/path/
```

- Copy a full folder (recursive) from local machine → remote server.

```sh
scp -P port -r /local/folder/ user@remote_ip:/remote/path/
```

- Copy a full folder (recursive) from remote server → local machine.

```sh
scp -P remote_port -r user@remote_ip:/remote/folder/ /local/folder/ 
```

- You could also add `-C` to enable compression for faster transfer of large file.

```sh
scp -C largefile.zip username@remote_ip:/remote/path/
```

### 2. Using `rsync` Command

The `rsync` command is for copying and syncing files or folders, especially when working with remote servers over SSH. It is faster and more flexible than scp, especially for large directories or incremental backups.

- Sync a folder from local machine → remote server.

```sh
rsync -avz -e "ssh -p port" /local/folder/ user@remote_ip:/remote/folder/
```

- Copy files from remote server → local machine.

```sh
rsync -avz -e "ssh -p remote_port" user@remote_ip:/remote/folder/ /local/folder/
```

### 3. Using SSH in VS Code with Remote

Setting up **Remote - SSH** in VS Code is one of the best ways to work on remote servers.

To install **Remote - SSH** extention, open **VS Code**, go to the **Extension** tab (`CTRL + SHIFT + X`), search for **Remote - SSH** by Microsoft and click **Install**.

To set up SSH access, you must configure to connect the remote server via `Remote - SSH` in VS Code.
1. Press `F1` or `CTRL +  SHIFT + P` → Open the **Command Palette**.
2. Type `Remote-SSH: Connect to Host...`.
3. Select remote alias (*do manually*) or enter full `remote_user@remote_ip`.
4. After connecting, go to **File → Open Folder**.
5. Browse and open the project directory on the remote server.

## Conclusion

**Remote SSH** is now up and running, and you can securely access your machine's command line from anywhere. For more information, you can refer to the man page by using command `man ssh`.