Command to install zeek

1. Install ethtool
2. Update your system
	```sh
	sudo apt update && sudo apt upgrade
	```
4. Install zeek pre-req
	```bash
	sudo apt-get install cmake make gcc g++ flex bison libpcap-dev libssl-dev python2-dev swig zlib1g-dev
	```
5. Add zeek repositories to local repo
	```bash
	wget -nv https://download.opensuse.org/repositories/security:/zeek/xUbuntu_22.04/Release.key -O Release.key
	```

6. Addkey 
	```bash 
	sudo apt-key add - < Release.key
	```
7. Update package to see if there's new repo
    ```
	sudo apt update
	```
8. Add repofile (needs root perm)
	```bash
	sudo sh -c "echo 'deb http://download.opensuse.org/repositories/security:/zeek/xUbuntu_22.04/ /' > /etc/apt/sources.list.d/security:zeek.list"
	```

9. Update to see if we have any new repo
	```
	sudo apt update
	``````
10. Install zeek 
	```bash
	sudo apt install zeek-lts
	```
11. Zeek is now installed!

To verify if we have installed,
```bash
ls /opt/zeek 
```
ls the binary package install location should be in /opt/zeek
	
there should also be commandos in the bin folder

```bash
cd /opt/zeek/bin/
```
```bash
./zeek -h
```
```bash
./zeekctl -h
```
	
13. Installation is verified!

14. command linking
```bash 
echo 'export PATH=$PATH:/opt/zeek/bin' > ~/.bashrc 
``` 