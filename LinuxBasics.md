### Add User to Sudoers File
\> su
input superuser password
\> nano /etc/sudoers
```
# This file MUST be edited with the 'visudo' command as root.
#
# Please consider adding local content in /etc/sudoers.d/ instead of
# directly modifying this file.
#
# See the man page for details on how to write a sudoers file.
#
Defaults	env_reset
Defaults	mail_badpass
Defaults	secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

# Host alias specification

# User alias specification

# Cmnd alias specification

# User privilege specification
root	ALL=(ALL:ALL) ALL

# ADD USER HERE
user    ALL=(ALL:ALL) ALL

# Allow members of group sudo to execute any command
%sudo	ALL=(ALL:ALL) ALL

# ADD USER HERE
user    ALL=(ALL:ALL) ALL

# See sudoers(5) for more information on "@include" directives:

@includedir /etc/sudoers.d
```


### Adding Buster Backports to update and installing podman

```bash
# Debian 10
# Use buster-backports on Debian 10 for a newer libseccomp2
echo 'deb http://archive.debian.org/debian buster-backports main' >> /etc/apt/sources.list

echo 'deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/Debian_10/ /' > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list

curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/Debian_10/Release.key | sudo apt-key add -

sudo apt update
sudo apt -t buster-backports install libseccomp2 crun
sudo apt install podman
```

> `sudo nano /etc/apt/sources.list`
> use archive version of buster-backports to install libseccomp2 and crun dependencies

`/etc/apt/sources.list` should look like this:
```
deb http://deb.debian.org/debian/ buster main contrib non-free
deb-src http://deb.debian.org/debian/ buster main contrib non-free
 
deb http://security.debian.org/debian-security/ buster/updates main contrib non-free
deb-src http://security.debian.org/debian-security/ buster/updates main contrib non-free

deb http://archive.debian.org/debian/ buster-backports main contrib non-free
deb-src http://archive.debian.org/debian/ buster-backports main contrib non-free
```

### Change Passwords:
```bash
sudo passwd username
```
\> input old password
\> input new password
\> retypenew password