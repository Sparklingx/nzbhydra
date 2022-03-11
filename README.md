NZBHydra is a meta search for NZB indexers. It provides easy access to a number of raw and newznab based indexers. You can search all your indexers from one place and use it as indexer source for tools like Sonarr or CouchPotato.

### Note
* I don't have anything to do with www.nzbhydra.com
* While this version still runs fine I recommend using its successor [NZBHydra2](https://github.com/theotherp/nzbhydra2) which runs a lot faster and has more features. Therefore I have removed all instructions regarding usage or development.


### Contact ###
Send me an email at TheOtherP@gmx.de or a PM at https://www.reddit.com/user/TheOtherP

### Donate ###
If you like to help me with any running or upcoming costs you're welcome to send money to my bitcoin: 1PnnwWfdyniojCL2kD5ZDBWBuKcFJvrq4t

### Thanks ###
<img src="https://github.com/theotherp/nzbhydra/raw/gh-pages/images/logo.png" width="60px"/> To Jetbrains for kindly providing me a license for PyCharm - I can't imagine developing without it

To all testers, bug reporters, donators, all around awesome people

### License ###
   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

Install NZBHydra on Ubuntu

Update your Ubuntu repositories and install python 2.7 and git

sudo apt-get update
sudo apt-get install python2.7 git-core -y
If you are on Ubuntu 14.04 then use this ppa to make sure you get Python 2.7.9

sudo add-apt-repository ppa:fkrull/deadsnakes-python2.7
sudo apt-get update
sudo apt-get upgrade
Clone the latest NZBHydra from the official repository

sudo git clone https://github.com/theotherp/nzbhydra /opt/nzbhydra
Change ownership to your preferred user for running programs (replace both instances of htpcguides)

sudo chown -R htpcguides:htpcguides /opt/nzbhydra
Test that NZBHydra runs, you can kill the process afterwards with Ctrl+C in the SSH terminal

cd /opt/nzbhydra
python nzbhydra.py --nobrowser
You should be able to see the NZBHydra web interface on its default listening port 5075

Autostart NZBHydra on Ubuntu
You can use the NZBHydra init.d script on any Ubuntu system. Ubuntu 15.x and newer users that prefer systemd can use the systemd script.

If you are in doubt whether you are using systemd or init.d (sysvinit)

sudo stat /proc/1/exe | grep -i file | awk '{print $4}'
If you see this line, then you have systemd

‘/lib/systemd/systemd’
NZBHydra init.d Script
Create the NZBHydra init.d script

sudo nano /etc/init.d/nzbhydra
Paste this NZBHydra init.d script, adjust the DAEMON_USER to match the one you used above for sudo chown command

#!/bin/sh
### BEGIN INIT INFO
# Provides:          NZBHydra
# Required-Start:    $local_fs $remote_fs $network
# Required-Stop:     $local_fs $remote_fs $network
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: NZBHydra
# Description:       Usenet search aggregator
### END INIT INFO
 
 
# Documentation available at
# http://refspecs.linuxfoundation.org/LSB_3.1.0/LSB-Core-generic/LSB-Core-generic/iniscrptfunc.html
# Debian provides some extra functions though
. /lib/lsb/init-functions
 
DAEMON_NAME="NZBHydra"
DAEMON_USER=htpcguides
DAEMON_PATH="/usr/bin/python"
DAEMON_OPTS="nzbhydra.py --nobrowser"
DAEMON_PWD="/opt/nzbhydra"
DAEMON_DESC=$(get_lsb_header_val $0 "Short-Description")
DAEMON_PID="/var/run/${DAEMON_NAME}.pid"
DAEMON_NICE=0
DAEMON_LOG='/var/log/nzbhydra'
 
[ -r "/etc/default/${DAEMON_NAME}" ] && . "/etc/default/${DAEMON_NAME}"
 
do_start() {
  local result
 
	pidofproc -p "${DAEMON_PID}" "${DAEMON_PATH}" > /dev/null
	if [ $? -eq 0 ]; then
		log_warning_msg "${DAEMON_NAME} is already started"
		result=0
	else
		log_daemon_msg "Starting ${DAEMON_DESC}" "${DAEMON_NAME}"
		touch "${DAEMON_LOG}"
		chown $DAEMON_USER "${DAEMON_LOG}"
		chmod u+rw "${DAEMON_LOG}"
		if [ -z "${DAEMON_USER}" ]; then
			start-stop-daemon --start --quiet --oknodo --background \
				--nicelevel $DAEMON_NICE \
				--chdir "${DAEMON_PWD}" \
				--pidfile "${DAEMON_PID}" --make-pidfile \
				--exec "${DAEMON_PATH}" -- $DAEMON_OPTS
			result=$?
		else
			start-stop-daemon --start --quiet --oknodo --background \
				--nicelevel $DAEMON_NICE \
				--chdir "${DAEMON_PWD}" \
				--pidfile "${DAEMON_PID}" --make-pidfile \
				--chuid "${DAEMON_USER}" \
				--exec "${DAEMON_PATH}" -- $DAEMON_OPTS
			result=$?
		fi
		log_end_msg $result
	fi
	return $result
}
 
do_stop() {
	local result
 
	pidofproc -p "${DAEMON_PID}" "${DAEMON_PATH}" > /dev/null
	if [ $? -ne 0 ]; then
		log_warning_msg "${DAEMON_NAME} is not started"
		result=0
	else
		log_daemon_msg "Stopping ${DAEMON_DESC}" "${DAEMON_NAME}"
		killproc -p "${DAEMON_PID}" "${DAEMON_PATH}"
		result=$?
		log_end_msg $result
		rm "${DAEMON_PID}"
	fi
	return $result
}
 
do_restart() {
	local result
	do_stop
	result=$?
	if [ $result = 0 ]; then
		do_start
		result=$?
	fi
	return $result
}
 
do_status() {
	local result
	status_of_proc -p "${DAEMON_PID}" "${DAEMON_PATH}" "${DAEMON_NAME}"
	result=$?
	return $result
}
 
do_usage() {
	echo $"Usage: $0 {start | stop | restart | status}"
	exit 1
}
 
case "$1" in
start)   do_start;   exit $? ;;
stop)    do_stop;    exit $? ;;
restart) do_restart; exit $? ;;
status)  do_status;  exit $? ;;
*)       do_usage;   exit  1 ;;
esac
Ctrl+X, Y and Enter to Save

Make the NZBHydra init.d script on Ubuntu executable

sudo chmod +x /etc/init.d/nzbhydra
Tell your Ubuntu server to use the NZBHydra init.d script by default

sudo update-rc.d nzbhydra defaults
Start NZBHydra init.d service

sudo service nzbhydra start
NZBHydra Systemd Script
Create the NZBHydra systemd script

sudo nano /etc/systemd/system/nzbhydra.service
Paste the NZBHydra systemd script, adjust your User and Group to the one you used in the chown command during installation

[Unit]
Description=NZBHydra Daemon
Documentation=https://github.com/theotherp/nzbhydra
After=network.target

[Service]
User=htpcguides
Group=htpcguides
Type=simple
ExecStart=/usr/bin/python /opt/nzbhydra/nzbhydra.py --daemon --nobrowser

KillMode=process
Restart=on-failure

[Install]
WantedBy=multi-user.target
Enable NZBHydra systemd script on Ubuntu

sudo systemctl enable nzbhydra
Start the NZBHydra systemd service

sudo service nzbhydra start
Now you can configure NZBHydra which is done similarly to NZBMegaSearch.

Remember the default NZBHydra listening port is 5075.
