# Setting up NTP on various machines
Windows Server 2016 and 2019 - w32time
Debian 10 Buster - systemd-timesyncd
Ubuntu 18 - sysd
CentOS - ntpd
Fedora - chrony

### Setting Up Server Side NTP Host Using `chrony`
#### Installation
```bash
sudo apt update
sudo apt install -y chrony
```

#### Configure Host Server

Edit the `chrony` configuration file `/etc/chrony/chrony.conf`
Add or modify the following lines:
```conf
# Upstream NTP servers to synchronize with
server 0.debian.pool.ntp.org iburst
server 1.debian.pool.ntp.org iburst
server 2.debian.pool.ntp.org iburst
server 3.debian.pool.ntp.org iburst

# Allow clients on your local network to sync with this server
allow 172.20.240.0/24
allow 172.20.241.0/24
allow 172.20.242.0/24

# Enable logging
logdir /var/log/chrony
```
#### Check and Apply

```bash
# Restart and Enable chrony Service
sudo systemctl restart chrony
sudo systemctl enable chrony

# Verify Chrony Status and sync details
sudo chronyc tracking

# Check listening client status
sudo chronyc sources -v
```

### Setting up Client-Side NTP
#### 1. **Debian and Ubuntu Clients Using `systemd-timesyncd`**

1. **Edit the Configuration File**:
```bash
sudo nano /etc/systemd/timesyncd.conf
```
Add or modify the `NTP` line to point to your Debian 10 NTP server:
```conf
[Time]
NTP=172.20.240.10
```

2. **Restart `systemd-timesyncd`**
```bash
sudo systemctl restart systemd-timesyncd
```

3. **Check Synchronization**:
```bash
timedatectl status
```

#### 2. **CentOS 7 Client Using `ntpd`**

1. **Install `ntp`**:
```bash
sudo yum install -y ntp
```

2. **Edit the Configuration**:
   
```bash
sudo nano /etc/ntp.conf
```

Add the Debian 10 server:

```conf
server 172.20.240.10 iburst
```

3. **Start and Enable `ntpd`**:

```bash
sudo systemctl start ntpd
sudo systemctl enable ntpd
```

4. **Verify Synchronization**:

```bash
ntpq -p
```

#### 3. **Fedora Client Using `chrony`**

1. **Install `chrony`**:
```bash
sudo dnf install -y chrony
```

2. **Edit the Configuration**:
```bash
sudo nano /etc/chrony/chrony.conf
```
Add the Debian 10 server:
```conf
server 172.20.240.10 iburst
```

3. **Start and Enable `chrony`**:
```bash
sudo systemctl start chronyd
sudo systemctl enable chronyd
```

4. **Verify Synchronization**:
```bash
chronyc tracking
```

#### 4. **Windows Server 2016 / 2019 Client Using `w32time`**

1. **Open Command Prompt as Administrator**.
2. **Set the NTP Server**:
```bash
w32tm /config /manualpeerlist:"172.20.240.10" /syncfromflags:manual /reliable:YES /update
```
3. **Restart the Windows Time Service**:
```bash
net stop w32time
net start w32time
```
4. **Verify Synchronization**:
```bash
w32tm /query /status
```
