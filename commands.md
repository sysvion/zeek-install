## Installing Zeek on a raspberry pi

### Commands

1. Updating the system and installing prerequisites:

```bash
sudo apt update
sudo apt full-upgrade -y
sudo apt install -y --no-install-recommends g++ cmake make libpcap-dev curl
```

2. Adding the opensuse repositories and installing zeek:

```bash
echo 'deb http://download.opensuse.org/repositories/security:/zeek/Debian_12/ /' | sudo tee /etc/apt/sources.list.d/security:zeek.list
curl -fsSL https://download.opensuse.org/repositories/security:zeek/Debian_12/Release.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/security_zeek.gpg > /dev/null
sudo apt update
sudo apt install zeek
```

3. Adding zeek to PATH:

```bash
echo "PATH=$PATH:/opt/zeek/bin" >> ~/.bashrc
source ~/.bashrc
```

4. Set-up zeek node:

```bash
sudo sed 's/interface=eth0/interface=wlan0/' -i /opt/zeek/etc/node.cfg
```

5. Enable JSON output:

```bash
echo '@load policy/tuning/json-logs.zeek' | sudo tee -a /opt/zeek/share/zeek/site/local.zeek
```

6. Install zeek:

```bash
sudo /opt/zeek/bin/zeekctl install
```

7. Start zeek:

```bash
sudo /opt/zeek/bin/zeekctl start
```

8. Check the status:

```bash
sudo /opt/zeek/bin/zeekctl status
```
Expected output:
```
Name         Type       Host          Status    Pid    Started
zeek         standalone localhost     running   4525   15 Dec 14:12:06
```

9. Create the zeek service file:

And put the contents of the following codeblock in `/etc/systemd/system/zeek.service`

> [!note]
> you need root permissions to write the file
>

```service
[Unit]
Description=Zeek Collection Server
After=network.target

[Service]
[Unit]
Description=Zeek Collection Server
After=network.target

[Service]
ExecStartPre=/opt/zeek/bin/zeekctl check
ExecStart=/opt/zeek/bin/zeekctl start
ExecReload=/opt/zeek/bin/zeekctl reload
ExecStop=/opt/zeek/bin/zeekctl stop
Restart=on-failure
RestartPreventExitStatus=255
Type=forking

[Install]
WantedBy=multi-user.target
Alias=zeek.service
```

10. Enable and start the service:

```bash
sudo systemctl start zeek
sudo systemctl enable zeek
```
out:
```
Created symlink /etc/systemd/system/multi-user.target.wants/zeek.service → /etc/systemd/system/zeek.service.
```
11. Add the zeek cleanup config:

```bash
crontab -e
```

And add
```
*/5 * * * * /opt/zeek/bin/zeekctl cron
```

## Installing Splunk Forwarder

### Commands

1. Add a dedicated user for splunk forwarder and escalate to this user:

> [!note]
> your system may deny the following command. to fix it you can add `sudo` before the command.
> 

```bash
adduser splunk -q --disabled-password
```

2. Download and install splunk forwarder:

```bash
wget -O splunkforwarder-9.1.2-b6b9c8185839-Linux-armv8.deb "https://download.splunk.com/products/universalforwarder/releases/9.1.2/linux/splunkforwarder-9.1.2-b6b9c8185839-Linux-armv8.deb"
sudo dpkg -i splunkforwarder-9.1.2-b6b9c8185839-Linux-armv8.deb
```

3. Add forward server:

```bash
sudo /opt/splunkforwarder/bin/splunk add forward-server 172.233.32.125:9997
```

Admin credentials:

```bash
pi:pipipipi
```

4. Edit the splunk forwarder inputs:

```bash
sudo vi /opt/splunkforwarder/etc/system/local/inputs.conf
```

Contents
```conf
[default]
host = zeek-01

[monitor:///opt/zeek/logs/current/conn.log]
_TCP_ROUTING = *
index = zeek
source = bro.conn.log
sourcetype = bro:json

[monitor:///opt/zeek/logs/current/dns.log]
_TCP_ROUTING = *
index = zeek
source = bro.dns.log
sourcetype = bro:json

[monitor:///opt/zeek/logs/current/software.log]
_TCP_ROUTING = *
index = zeek
source = bro.software.log
sourcetype = bro:json

[monitor:///opt/zeek/logs/current/smtp.log]
_TCP_ROUTING = *
index = zeek
source = bro.smtp.log
sourcetype = bro:json

[monitor:///opt/zeek/logs/current/ssl.log]
_TCP_ROUTING = *
index = zeek
source = bro.ssl.log
sourcetype = bro:json

[monitor:///opt/zeek/logs/current/ssh.log]
_TCP_ROUTING = *
index = zeek
source = bro.ssh.log
sourcetype = bro:json

[monitor:///opt/zeek/logs/current/x509.log]
_TCP_ROUTING = *
index = zeek
source = bro.x509.log
sourcetype = bro:json

[monitor:///opt/zeek/logs/current/ftp.log]
_TCP_ROUTING = *
index = zeek
source = bro.ftp.log
sourcetype = bro:json

[monitor:///opt/zeek/logs/current/http.log]
_TCP_ROUTING = *
index = zeek
source = bro.http.log
sourcetype = bro:json

[monitor:///opt/zeek/logs/current/rdp.log]
_TCP_ROUTING = *
index = zeek
source = bro.rdp.log
sourcetype = bro:json

[monitor:///opt/zeek/logs/current/smb_files.log]
_TCP_ROUTING = *
index = zeek
source = bro.smb_files.log
sourcetype = bro:json

[monitor:///opt/zeek/logs/current/smb_mapping.log]
_TCP_ROUTING = *
index = zeek
source = bro.smb_mapping.log
sourcetype = bro:json

[monitor:///opt/zeek/logs/current/snmp.log]
_TCP_ROUTING = *
index = zeek
source = bro.snmp.log
sourcetype = bro:json

[monitor:///opt/zeek/logs/current/sip.log]
_TCP_ROUTING = *
index = zeek
source = bro.sip.log
sourcetype = bro:json

[monitor:///opt/zeek/logs/current/files.log]
_TCP_ROUTING = *
index = zeek
source = bro.files.log
sourcetype = bro:json
```

5. Start and enable splunk forwarder to start at boot:

```bash
sudo /opt/splunkforwarder/bin/splunk enable boot-start
sudo /opt/splunkforwarder/bin/splunk start
```

### Install the TA for Zeek:
https://splunkbase.splunk.com/app/5466
