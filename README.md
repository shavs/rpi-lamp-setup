# Setting up a Raspberry Pi Zero with LAMP, OTG and Node.js

These are some basic instructions to configure a Raspberry Pi Zero as a simple, portable server.

I just found it annoying that a LAMP server is difficult to set up.

## 1. Writing the Raspberry Pi image to the SD Card
There are many ways of writing Raspbian Lite to the SD card.

On Linux, using `dd` is the easiest way.

*Note*, use `lsblk` or another tool to find out which device is the 
correct one. In this example, `/dev/sdc` is used. Normally, this would 
be the third drive on your machine. Alternatively, use a tool like 
`gparted` to visually determine the device, and click "View", then 
"Device Information". This will then display information about the 
device, such as the manufacturer.

In this example, `yyyy` is the year, `mm` is the month, and `dd` is the 
day. `jessie` is the Debian distribution version - this may change in 
time.

```bash
sudo dd if=yyyy-mm-dd-raspbian-jessie-lite.img of=/dev/sdc bs=4M status=progress

```

## 2. Setting up the SD Card

Once the image has been written to, it should appear as two separate devices in your file manager.

These two devices should be called:
- `boot`
- Unnamed Volume (this should be unnamed, and should be roughly 1.3 GB Volume)

We are going to use the `boot` partition to change the initial boot.

*Note:* when editing files on the Raspberry Pi Zero, make sure that the format of the text file is Linux, 
UTF-8. Preferably, use a text editor like Notepad++ or Mousepad, both of which read and write files in UTF-8. 
Otherwise, the file may not be able to be read and may damage the Raspberry Pi.

### 2.1 - `config.txt`

First, we need to edit `config.txt`. This contains all of the boot information for the Raspberry Pi. 

Add this line to the bottom of the `config.txt` file.

```
# Enable the OTG capability of the Raspberry Pi Zero
dtoverlay=dwc2
```

### 2.2 - `cmdline.txt`

In this file, we will enable the module overlay.

Simply place this line after `modules`.

```
modules-load=dwc2,g_ether
```

For example:
```
dwc_otg.lpm_enable=0 console=serial0,115200 console=tty1 root=/dev/mmcblk0p2 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait modules-load=dwc2,g_ether quiet init=/usr/lib/raspi-config/init_resize.sh
```

### 2.3 - `SSH`
From December 2016 onwards, the new Raspberry Pi system images need a file called `SSH` placed in the boot 
directory. This is to prevent people from accidentally enabling it, and also for people who will probably 
never need to use SSH on the Raspberry Pi.

For this step, simply create a file called `SSH` on the `boot` partition.
Nothing needs to be in the file, and once the Raspberry Pi has booted, it will be deleted, and SSH will be 
enabled.

For more information about this, please see the [Raspberry Pi Foundation's page on SSH](https://www.raspberrypi.org/documentation/remote-access/ssh/).


## 3. Booting the Raspberry Pi

Once the steps above have been completed, simply unmount/eject the SD Card, put the SD card into the Raspberry 
Pi Zero, plug a Micro USB cable that supplys both data and power (an Android phone cable usually works) into 
the leftmost Micro USB port of the Raspberry Pi.

It will take a while to boot. Once the Raspberry Pi has booted:

- On Windows, it will usually come up with a message such as `RNDIS USB Ethernet Gadget`, and telling you that 
the driver has been installed

- On Linux, nothing will show (this is normal). If you are using NetworkManager, right-click the icon and 
select "Edit Connections". From here, click "Add" on the right-hand side, select "Ethernet" and click "OK". 
Once you have clicked "OK", it should allow you to enter a name for the network connection. Next, click "IPv4 
Settings", then in the select box labeled "Method", change it to "Shared to other computers". Once this has 
been done, click "Save". Change the Raspberry Pi Zero's connection to the new one, and you should get a 
notification.


## 4. SSH-ing into the Raspberry Pi

On Windows and Linux (any Linux distribution with Avahi Daemon installed, such as Ubuntu or OpenSuSE), you 
should be able to SSH into the Raspberry Pi using PuTTY or the `ssh` command.

By default, the password of the Raspberry Pi is `raspberry`.

In order to SSH into the machine, use:
```bash
ssh pi@raspberrypi.local
```

If you do not have Avahi installed, or cannot access the Raspberry Pi using the above command, try using:
```bash
ssh pi@IP_ADDRESS_HERE
```
where IP_ADDRESS_HERE is the IP address given to the Pi, from your machine.


## 5. Changing the default password
In order to change the default password of the user `pi`, run:
```bash
passwd
```

You might also want to change the user you are using. To do so, we are going to create a new user, change to 
the new user, then log in as the new user:
```bash
# Create a new user:
sudo adduser bobo
# This should prompt for a password.

# In order for the new user to have sudo access, you need to edit the 
# sudoers file. In order to do this, type:

sudo visudo

# Once in here, find the line that reads "User privilege specification".
# Below the root user, add this line:

bobo  ALL=(ALL:ALL) ALL

# Once this line is added, the user should have sudo access.
# Write the contents to the file (in nano, using Ctrl and o) 

# Now log out using Ctrl and D, or by typing:

exit

# Now log into the user, using the same login command as Step 4 - SSHing into the Raspberry Pi.

# Once logged in, the old user (in this case, pi) can be deleted:
sudo userdel -r pi
# The -r will remove the user's home directory.

``` 

## 6. Installing the LAMP Server
### 6.1 - Installation Preparation
Make sure that the rest of the system is up to date using:
```bash
sudo apt-get update && sudo apt-get upgrade
```

### 6.2 - MySQL
Install MySQL using:
```bash
sudo apt-get install mysql-server
```
This is known as a **meta package**, which is a collection of packages for installing a program labelled under 
one name. This makes installation easier than having to specify each package individually.


During the install, the root password for MySQL will be set. Either remember this password, or write it down, 
as you'll be needing this password later when installing phpMyAdmin.

### 6.3 - Apache
```bash
sudo apt-get install apache2 apache2-doc
```

### 6.4 - PHP
Currently, only PHP 5 is available from the Raspbian repositories. If you need PHP 7, please use [this guide from the Symfony Finland page, which allows you to either build from source or to install precompiled binaries](https://www.symfony.fi/entry/install-php-7-on-raspbian-raspberry-pi).
Alternatively, you may be able to take a package from [Debian Backports](https://backports.debian.org/Instructions/), and use that. However, this is outside of the scope of this guide.

This is because PHP5.6.30 is the last formal release - every release hereafter is a security release. [See here for more information on the release of PHP 5.6.30](https://secure.php.net/archive/2017.php#id2017-01-19-3).

```bash
sudo apt-get install php5 php5-mysql libapache2-mod-php5
```

### 6.5 - phpMyAdmin

This installs an administrative interface for MySQL, using PHP as a backend.
```bash
sudo apt-get install phpmyadmin
```

When installing, select `apache2` as the web server that should be automatically configured.
Use the spacebar to select `apache2`, and the arrow keys on your keyboard.

Make sure to configure phpmyadmin with `dbconfig-common`, which will ask you for the user and password of the user who controls the database.

### 6.6 - Configuring PHP correctly

#### 6.6.1 - Enable User directories
In order to enable user directories, you will need to edit and enable user directories.

```bash
sudo a2enmod userdir && sudo service apache2 restart
```

Then edit the configuration file, located at `/etc/apache2/mods-enabled/userdir.conf`.
It should look like this, after editing:

```xml
<IfModule mod_userdir.c>
        UserDir public_html
        UserDir disabled root
        <Directory /home/*/public_html>
                AllowOverride All
                Options MultiViews Indexes SymLinksIfOwnerMatch
                <Limit GET POST OPTIONS>
                        Require all granted
                </Limit>
                <LimitExcept GET POST OPTIONS>
                        Require all denied
                </LimitExcept>
        </Directory>
</IfModule>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet

```

Next, you need to edit the Apache configuration file for PHP,
 located at `/etc/apache2/mods-available/php5.conf`. There is a section
in this file that needs to be "commented out", so that the machine can 
use user directories. Usually, this is located at the bottom of the document:

```xml
# Running PHP scripts in user directories is disabled by default
#
# To re-enable PHP in user directories comment the following lines
# (from <IfModule ...> to </IfModule>.) Do NOT set it to On as it
# prevents .htaccess files from disabling it.
#<IfModule mod_userdir.c>
#    <Directory /home/*/public_html>
#        php_admin_flag engine Off
#    </Directory>
#</IfModule>
```

Next, restart Apache again by using:
```bash
sudo service apache2 restart
```

### 6.6.2 - Testing PHP and phpMyAdmin

We need to create a directory called `public_html` in the user's "home"
directory. By default, you should be in this directory, but if you need to move
back to this directory, you can enter:
```bash
cd ~/
```
Which should move you back to the home directory. Then, create the `public_html`:
```bash
mkdir public_html
```

Next, we will create a basic PHP test file for the directory. Using your favourite editor,
create a file named `index.php`, and type:
```php
<?php phpinfo(); ?>
```

Which will print out information about the PHP installation, 
and serves as a good test of PHP.

In order to load up the PHP file in your browser, you'll need to enter in either:
- the IP address of the Raspberry Pi
- the hostname of the Raspberry Pi

then, add this after the IP address or hostname:
```
/~USERNAME_HERE/index.php
```

In addition, you should be able to access PHPMyAdmin using:
```
/phpmyadmin
```

### 6.6.3 - Setting up phpMyAdmin
By default, you will only have access to phpMyAdmin via the `root` account.
Therefore, if you want to create a new user, you need to log into phpMyAdmin.

1. Log in to phpMyAdmin
2. Click on "Users" tab at the top
3. Click "Add user"
4. Enter a username
5. Enter a password or generate a password and reconfirm
6. Under "Database for user", make sure to tick "Create database with same name and grant all permissions"
7. Under "Global privileges", click which SQL commands you would like to allow that user, and the resource limits you would like to impose for that user

Next:
1. On the left-hand side, click the newly created user.
2. Create a new table by typing a new name, and selecting the number of columns you would like.
3. Once you have finished creating a table schema, click "Save".
4. Now, create a new record by clicking on the newly created table on the left, underneath the user
5. Click the "SQL" tab, at the top.
6. Click the "insert" button, underneath the text editor.

For the SQL query, *make sure to remove the ID value, as this can mess up the SQL automatic incrementation*.
For example, here is a query that inserts into a table named `test_table`:

```sql
INSERT INTO `test_table`(`firstname`, `lastname`, `year`, `month`) VALUES ("bob","smith","2008","10")
```

### 6.6.4 - Testing the Database connection for PHP
To test if the database can be accessed by a PHP connection,
create a file named `database_test.php` in the user's `public_html` folder.

In this file, we will need the user's database (usually the username of the user), the username and the password as well.

For example:
```php
<?php
// This file serves as a method to test a database connection.
// It should not be used in a production environment.

$conn = new PDO("mysql:host=localhost;dbname=DATABASE NAME HERE;", "USERNAME HERE","PASSWORD HERE");

$results = $conn->query("SELECT * FROM TABLE NAME HERE");

echo "<h1>This is a database test.</h1>";
echo "<h2>This should ONLY be used in a testing environment.</h2>";


// During this part, where there is a square bracket, put in each column name,
// delete and change each echo as needed.

while($row=$results->fetch()){
  echo "<h3>Test Table ID: $row[id]</h3>";
  echo "<p>First Name: $row[firstname] </p>";
  echo "<p>Last Name: $row[lastname] </p>";
  echo "<p>Year: $row[year] </p>";
  echo "<p>Month: $row[month] </p>";
}

?>
```

This script should print out all of the available information.

Success!


In the next set of steps, we are going to stop the Apache server from running,
and we are going to set up Node.js and MongoDB.


## 7. - Installing Node and MongoDB
### 7.1 - Node
The Node JS version supplied in the Raspbian repositories is dreadfully stale - it remains at version 0.10.
Therefore, the easiest way to install Node is to do the following:

1. Get an ARM v6l build of Node JS, via `wget`
2. Extract and move the folder to the expected location of Node.
3. Link up Node to appear in the correct directories.

To that end, [Steven de Salas has written a script that automates this completely](https://github.com/sdesalas/node-pi-zero).

Remember to uninstall Node JS on the Raspberry Pi by running:
```bash
sudo apt-get remove nodejs
``` 
which will remove Node JS and all associated packages downloaded from the
Raspbian repositories.

Test Node is running and is the correct version by typing
```bash
node -v

# My output was:
# v6.10.1
# as I had chosen to install the LTS version of Node.
```

Running this script again for a new / different version of Node should also remove the currently-installed version as well.

*Note:* This is going against the Debian method of installation, as it violates the security of the distribution.


### 7.2 - Installing MongoDB

MongoDB is currently (as of writing) at a very old, unstable version. Therefore,
in order to get the current version of MongoDB, you will either need to:

- Use [Arch Linux ARM](https://archlinuxarm.org/packages), which has the latest version of MongoDB and Node in it's repositories, but only a 64-bit version of MongoDB
- Build it from source (not recommended by the Debian developers, for the same reason as Node.)
- Install Linuxbrew (this would only work on an x86_64 machine, not on ARM v6l)
- Host the database on a separate machine or server which is compatible with current MongoDB versions
In addition, there are severe incompatibilities between MongoDB and ARM v6l, 
and therefore MongoDB is only available for ARM 64-bit processors. There is no support for 32-bit processors.

Even if MongoDB is successfully built for the Raspberry Pi Zero, it would have severe performance issues,
and would not be able to make large database sizes (less that 2GB, as per the limitation of 32-bit).

An alternative, such as a Raspberry Pi 3, would work, as the System-on-a-Chip is 64-bit, and is compatible with the current MongoDB version.



### 7.3 - Allowing applications to access MongoDB

Once you have installed Mongo and any related tools, simply run:
```bash
sudo npm install -g mongodb
```
which will globally install the MongoDB driver.

Once you are working within a project, simply run:
```bash
npm link mongodb
```
which will allow the project to make use of the MongoDB driver.


