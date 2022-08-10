# A Frappe Framework installation guide (Ubuntu Linux)

## Pre-requisites

These pre-requisites are as per the official documentation [here](https://frappeframework.com/docs/v14/user/en/installation). Slight changes have been made to include other installations and instructions.

```
+ Ubuntu Desktop 20.04 LTS

+ Python 3.10+ (v14)

+ Node.js 16

+ Redis 6                                       (caching and realtime updates)

+ MariaDB 10.3.x / Postgres 9.5.x               (to run database driven apps)

+ yarn 1.12+                                    (js dependency manager) ** NOT npm **

+ pip 20+                                       (py dependency manager)

+ wkhtmltopdf (version 0.12.5 with patched qt)  (for PDF generation)

+ cron                                          (bench's scheduled jobs: automated certificate renewal, scheduled backups)

+ NGINX                                         (proxying multitenant sites in production)
```

## Update and upgrade APT packages

```
sudo apt update -y

sudo apt upgrade -y
```

## Important note:

Whilst most installation guides recommend doing the installation under a new, non-root, user with root privileges, this installation is done under the root system user. It's until later in the installation was there a need to create a new user and finish the installation. It must be noted that this was only observed for hosted systems. Thus, local machine installation won't necessarily require the creation of a new user. Be that as it may, other alternatives in this regard can still be explored!

## Install `Python 3.10` and set it as default

Highly likely Ubuntu 20.04 LTS will come with Python 3.8 as its default Python Interpreter, so you need to have Python 3.10 installed and set as the default installation. Otherwise, if this is already the case for your system, move to the next section.

### Install required dependency for adding custome PPAs

```
sudo apt install software-properties-common -y
```

### Add deadsnakes PPA to APT sources list

```
sudo add-apt-repository ppa:deadsnakes/ppa
```

### Install Python 3.10 and its virtual environment manager

```
sudo apt install python3.10 python3.10-venv -y
```

### Update python3 symbolic link

```
rm /usr/bin/python3

ln -s /usr/bin/python3.10 /usr/bin/python3
```

## Install `python3.10-dev`, `python3.10-distutils` and `python3-pip`

```
sudo apt install python3.10-dev python3.10-distutils python3-pip -y
```

To get Python 3.10 `pip3` update, run:

```
curl -sS https://bootstrap.pypa.io/get-pip.py | python3.10
```

## Install `git` and `redis`

```
sudo apt install git redis-server -y
```

## Install and configure `MariaDB`

```
sudo apt install mariadb-server mariadb-client -y
```

Edit MariaDB config file: `nano /etc/mysql/my.cnf`

Insert the following config:

```
[mysqld]
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

[mysql]
default-character-set = utf8mb4
```

Restart mysql service: `service mysql restart`

## Install `NodeJS`, `npm` and `yarn`

```
sudo apt install nodejs npm -y

sudo npm i -g n yarn

n stable
```

## Install `wkhtmltopdf`

```
sudo apt install xvfb libfontconfig wkhtmltopdf -y
```

## Install `Bench CLI` and setup `bench` site

```
cd ~

pip3 install frappe-bench

bench init <project-name>
```

On hosted servers, a `You should not run this command as root` warning is encountered. Under this case a new user with root permissions should be created.

```
sudo adduser <username>

sudo usermod -aG sudo <username>

sudo su <username>

cd ~

bench init <project-name>
```

### Create new site in project directory

```
cd <project-name>

bench new-site <site-name>
```

MySQL will require a root password. This installation didn't setup a root password for MySQL. So, you need to reset the root password by running:

```
sudo mysql_secure_installation
```

When asked to enter root password just press `ENTER` and continue. When asked to set root password, `INSERT` you new password. Answer `Y` to the remaining configurations.

Now configure `mysql` password using the password selected in the previous step:

```
sudo mysql -uroot -p<your-password> -Bse "GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' IDENTIFIED BY '<your-password>' WITH GRANT OPTION;"
```

Re-run: `bench new-site <site-name>` and if a directory already exists under the site name, just delete it to create a new site. Enter MySQL root password to continue.

## Run bench

```
bench start
```

Site should open on port `8000`. If the site is `not found` or `doesn't exit`, create a `currentsite.txt` file under `<project-name>/sites` directory and just insert the name of the site you created and save the file. Re-run `bench start`

## System missing packages

Changing the system default Python installation can result in loss of required modules to run essential tasks in the terminal.

Something like:

```
Traceback (most recent call last):
  File "/usr/lib/command-not-found", line 28, in <module>
    from CommandNotFound import CommandNotFound
  File "/usr/lib/python3/dist-packages/CommandNotFound/CommandNotFound.py", line 19, in <module>
    from CommandNotFound.db.db import SqliteDatabase
  File "/usr/lib/python3/dist-packages/CommandNotFound/db/db.py", line 5, in <module>
    import apt_pkg
ModuleNotFoundError: No module named 'apt_pkg'
```

Fix this by removing and reinstalling `python3-apt`

```
sudo apt remove python3-apt -y

# restart terminal

sudo apt install python3-apt -y
```

## Next steps

This is pretty much installation. The next steps are adding applications, creating custom applications, the application setup, etc. More can be found on the [official documentation](https://frappeframework.com/docs/v14/user/en/introduction).
