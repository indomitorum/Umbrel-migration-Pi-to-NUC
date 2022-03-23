# Umbrel-migration-Pi-to-NUC
Umbrel migration from Raspberry Pi to NUC / Laptop running Linux
Umbrel migration from Raspberry Pi to NUC / Laptop running Linux

I chose Debian OS as it’s meant to be better for running a node keeping the file system in better order. In case of drive failure, Debian is superior at recovering bad blocks and repairing them but you can use Ubuntu if you prefer or even Mint.

Installation of Linux and Umbrel in new machine

1.	Install the Linux OS flavour of choice in your machine 
2.	Update it  
a.	sudo apt update & sudo apt upgrade & sudo reboot
3.	Install Umbrel on Linux  
a.	https://github.com/getumbrel/umbrel#-installation
b.	Scroll down to Installation Requirements and install docker / Python / docker compose, fswatch, jq, rsync, curl, git
c.	Create an Umbrel directory - mkdir umbrel & cd umbrel
d.	Download Umbrel in the above directory
e.	Run Umbrel on new machine : sudo ./scripts/start
f.	Let it sync a few blocks then sudo ./scripts/stop

Migration Process

Raspberry Pi

1.	ssh in to the R Pi
2.	I recommend uninstalling all apps as when I tested the migration with apps, I run into problems. If you want to keep for example LNDg database so you don’t lose anything, before deleting it, 
a.	stop LNDg (GUI or ~/umbrel/scripts/app stop lndg)
b.	Copy the database from ~/umbrel/app-data/lndg/db.sqlite3 to ~/umbrel
3.	Uninstall all apps
4.	Sudo scripts/stop 
5.	Ensuring the channel.db file integrity is crucial so you don’t end up penalised starting the new node with an old / corrupted channel.db which would be a disaster.
a.	cd umbrel/lnd/data/graph/mainnet
b.	sha256sum channel.db > checksum
c.	now let’s check its ok first with sha256sum –c checksum. Output should be: OK . it can take a while if your channel.db is big.
6.	sudo systemctl stop umbrel-startup
7.	Check everything is down: docker ps or docker-compose ps from ~/umbrel
8.	Check bitcoin (debug.log) and lnd (lnd.log) logs for shutdown complete
9.	Disconnect SSD from Pi. Turn off Pi

Linux machine – NUC / Gigabyte / Laptop

1.	~/umbrel/scripts/stop
2.	Connect Raspberry Pi SSD and mount it in a directory of your choice in the new machine. 
a.	Example: Run the lsblk command to see the SSD (unmounted for now)
b.	Create a directory to act as a mount point. sudo mkdir /media/mymountpoint
c.	Mount it : sudo mount /dev/(whatever the drive is called from lsblk - for example /dev/sdb1) /media/mymountpoint
d.	lsblk will show it as mounted now

3.	Copy the whole Raspberry Pi ~/umbrel directory to the new machine
a.	Rsync –azhP /media/path-to-your-~/umbrel/  ~/umbrel 
b.	DO NOT forget the trailing slash on the source so that rsync does not create the source folder on the destination and only copies the directory’s files
c.	This will take a while as it copies everything including the blockchain (2-3 hrs is not unheard of)
4.	Once its finished copying the ~/umbrel directory drom pi to new machine, check the channel.db integrity. From the channel.db directory (where channel.db is) in the new machine run sha256sum –c checksum. Output should be : OK. If it isn’t, DO NOT start the new machine as you can lose funds.
5.	Backup your lnd.conf for example lnd.conf_bak as we will reconfigure and delete it. 
6.	From ~/umbrel, run: sudo rm –rf lnd/tls* && rm –f lnd/lnd.conf && sudo scripts/configure
7.	sudo reboot 
8.	Add the old lnd.conf lines to the new one carefully one by one.
9.	~/umbrel/scripts/start

Optional

LNDg database

1.	Reinstall it with GUI or CLI
2.	Once installed, the database that we had saved ~/umbrel/db.sqlite3 can be copied to where the app resides. Example: rsync –azhP ~/umbrel/dbsqlite3 ~/umbrel/app-data/lndg/db.sqlite3

Umbrel autorestart 

A service unit needs to be created so umbrel starts otherwise if you restart from GUI, Umbrel will not restart automatically in your new Linux machine

1.	Set a new service unit as follows 
2.	Sudo nano /etc/system/system/umbrel.service
[Unit]
Description=umbrel.service
After=network.target
Wants=network.target
StartLimitIntervalSec=0

[Service]
Type=forking
Restart=always
RemainAfterExit=yes
RestartSec=1
User=root
ExecStart=path-to-umbrel-scripts/start
ExecStop=path-to-umbrel-scripts/stop

[Install]
WantedBy=multi-user.target


3.	Start service : systemctl start umbrel.service
4.	Add it to start (init): systemctl enable.service

If you run into problems, issues, DM me on Telegram : indomitorum (indomitus).

IndomitusBTC ⚡






