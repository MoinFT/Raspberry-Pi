# Linux (Raspbian OS) Commands

## Structure

1. [Change Password](#1-change-password)
2. [Config](#2-config)
3. [OS](#3-os)
4. [Root](#4-root)
5. [Packages](#5-packages)
6. [Directories](#6-directories)
7. [Files](#7-files)
8. [File permissions](#8-file-permissions)
9. [File System](#9-file-system)
10. [Services](#10-services)
11. [Installation](#11-installation)
12. [Firmware Update](#12-firmware-update)

### 1. Change password

```bash
passwd
```

### 2. Config

```bash
sudo raspi-config
```

### 3. OS

Shutdown (now):

```bash
sudo shutdown -h now
```

Reboot:

```bash
sudo reboot
```

Regular execution of a process:

```bash
crontab -e
```

### 4. Root

Root is the super user from the system. The root user all rights on a system.

Login as root:

```bash
sudo -i
```

To use root rights for one command write `sudo` at the start of the line.

```bash
sudo [command]
```

### 5. Packages

Package installation:

```bash
sudo apt-get install [package-name]
```

Package update:

```bash
sudo apt-get update & sudo apt-get upgrade
```

Package uninstall:

```bash
sudo apt-get purge [package-name]
```

### 6. Directories

Create directory:

```bash
mkdir [directory-name]
```

Rename directory:

```bash
mv [old-directory-name] [new-directory-name]
```

Copy directory:

```bash
cp -a [source-directory] [destination]
```

Delete empty directory:

```bash
rm -d [directory-name]
```

Delete non empty directory:

```bash
rm -r [directory-name]
```

### 7. Files

Create file:

```bash
touch [file-name]
```

Rename file:

```bash
mv [old-file-name] [new-file-name]
```

Edit file:

```bash
[editor] [file-name]
```

Editors:

- vi (default)
- vim
- nano

Copy file:

```bash
cp [source-file] [destination]
```

Delete file:

```bash
rm -i [file-name]
```

### 8. File Permissions

All permission commands can used for files and directories.

Show permissions:

```bash
ls -l [name]
```

Add permissions for the signed in user:

```bash
chmod +x [name]
```

Change permissions:

```bash
chmod UGO [name]
```

|Short|Name |
|-----|-----|
|U    |User |
|G    |Group|
|O    |Other|

|Short|Name    |Value|
|-----|--------|-----|
|R    |Read    |4    |
|W    |Write   |2    |
|X    |Execute |1    |

### 9. File System

Mount an external file system into a directory of the own file system.

1. [Create the directory](#6-directories)
2. Get the name of your external media

    ```bash
    lsblk -p
    ```

3. Use the path of your external media and your directory to mount the external media.

    ```bash
    mount /dev/[path-external-media] /[path-directory]
    ```

### 10. Services

Services can be used to start any process on boot and get a log file for the process.

Create a service:

 ```bash
sudo touch /etc/systemd/system/[file-name]
sudo [editor] /etc/systemd/system/[file-name] 
```

Insert the following code into this service file:

You will need to:

- Set the username after User=, which user should start the script
- Set the proper path to your script after ExecStart=

```shell
[Unit]
Description=Description for your service
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=always
RestartSec=1
User=username
ExecStart=/your/path/script.sh

[Install]
WantedBy=multi-user.target
```

Start a service:

```bash
systemctl start [service-name]
```

Show the status of a service:

```bash
systemctl status [service-name]
```

```bash
journalctl -u [service-name]
```

Restart a service:

```bash
systemctl restart [service-name]
```

Stop a service:

```bash
systemctl stop [service-name]
```

### 11. Installation

- [Node.js](#nodejs)
- [Apache Webserver with PHP](#apache-webserver-with-php)
- [MariaDB](#mariadb)

### 12. Firmware update

Get the actual Raspbain OS Version:

```bash
cat /etc/os-release
```

To update the firmware of the Raspberry Pi you have to execute the following command:

```bash
sudo apt update
sudo apt full-upgrade
```

## Installations

Before install any package update all packages with `sudo apt-get update & sudo apt-get upgarde`

### Node.js

Get the latest version from <https://nodejs.org/en/download/>. Then insert the version instead of version into the link.

From the Raspberry Pi get the version of ARM with `uname -m` and insert the version instead of version into the link.

```bash
wget https://nodejs.org/dist/[version]/node-[version]-linux-[uname].tar.gz
tar -xzf node-[version]-linux-[uname].tar.gz
cd node-[version]-linux-[uname]/
sudo cp -R * /usr/local/
```

### Apache Webserver with PHP

```bash
sudo apt-get install apache2
sudo apt-get install php
cd /var/www/html/
sudo nano phpinfo.php
```

Insert the following code into phpinfo.php:

```php
<?php
    phpinfo();
?>
```

### MariaDB

1. sudo apt-get install mariadb-server-10.0 php7.3-mysql mariadb-client-10.0
2. sudo reboot

3. sudo apt-get install phpmyadmin
4. sudo mysql -u root -p

    ```sql
    GRANT ALL PRIVILEGES ON *.* TO 'USERNAME'@'%' IDENTIFIED BY 'PASSWORD' WITH GRANT OPTION;
    ```

5. Change bind-address from mariaDB

    ```bash
    sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf 
    ```

    Change the line `bind-address` from

    ```bash
    bind-address = 127.0.0.1
    ```

    To

    ```bash
    bind-address = 0.0.0.0
    ```

    Restart the mariaDB service

    ```bash
    sudo systemctl restart mysql.service
    sudo systemctl restart mariadb.service
    ```
