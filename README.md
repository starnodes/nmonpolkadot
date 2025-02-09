# nmonpolkadot

A complete log file based Polkadot validator uptime monitoring solution for Zabbix. It consists of the shell script nmon.sh for generating log files on the host and the template zbx_5_template_nmonpolkadot.xml for a Zabbix 5.x server. Also useful for other monitoring platforms (with log exporter) and as a tool.

### quick start

```sh
sudo apt install -y screen
sudo apt -y install jq bc
sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates && curl -sL https://deb.nodesource.com/setup_current.x | sudo -E bash -
sudo apt update && sudo apt -y install nodejs
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add - && echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt update && sudo apt install yarn
sudo yarn global add @polkadot/api-cli
wget -qO /home/nmon.sh https://raw.githubusercontent.com/toxi42/nmonpolkadot/main/nmon.sh
chmod +x /home/nmon.sh
!change your validator adress in nmon.sh
rm -f /home/nmon-root*
screen -d -m -S nmon /home/nmon.sh
screen -S nmon -X quit #if you need stop
crontab -e
@reboot sleep 15; rm -f /home/nmon-root*
@reboot sleep 60; screen -d -m -S nmon /home/nmon.sh
install template to ur zabbix server
use only active zabbix-agent!
change HOST macros to ur location log /home/nmon-root.log 
```

### Concept

nmon.sh generates human-readable logs that look like:

`
[2021-01-18 07:58:54+00:00] status=synced height=5822773 elapsed=18 behind=1 finalization=3 peers=58 session=9998 isValidator=yes authoredBlocks=1 heartbeat=waiting pctSessionElapsed=51.77 era=1784 pctEraElapsed=41.96`
 
`
[2021-01-18 07:59:39+00:00] status=synced height=5822777 elapsed=9 behind=0 finalization=3 peers=59 session=9998 isValidator=yes authoredBlocks=1 heartbeat=waiting pctSessionElapsed=53.02 era=1784 pctEraElapsed=42.17`
 
`
[2021-01-18 08:00:24+00:00] status=synced height=5822784 elapsed=6 behind=0 finalization=3 peers=55 session=9998 isValidator=yes authoredBlocks=1 heartbeat=ok pctSessionElapsed=54.25 era=1784 pctEraElapsed=42.37`

The log line entries that are imported by the Zabbix server are:

* **status** can be {scriptstarted | synced | catchingup | error} 'error' can have various causes, typically the `polkadot` process is down.
* **height** current height
* **elapsed** time in seconds since current height (useful for latency or chain halt detection)
* **behind** difference between highest and current height
* **finalization** difference between highest and last finalized height
* **peers** number of peers
* **session** the current session
* **isValidator** (only if validator address is configured) can be {yes | no}
* **authoredBlocks** (only if validator address is configured) authored blocks from the configured validator address for the current session
* **heartbeat** (only if validator address is configured) can be {waiting | ok | missing | ipmissing} 'waiting' if heartbeat-after-height is not yet reached for current session, 'ok' if either a block was authored or a heartbeat sent upon heartbeat-after-height, 'missing' heartbeat was not sent, 'ipmissing' heartbeat message does not contain the configured ip (useful for confirming local node)

### Installation

The script for the host has a configuration section on top where parameters can be set. There is also information for the required dependencies and how to install the JS-API.

A Zabbix server is required that connects to the host running the Polkadot validator. On the host side the Zabbix agent needs to be installed and configured for active mode. There is various information on the Zabbix site and from other sources that explains how to connect a host to the server and utilize the standard Linux OS templates for general monitoring. Once these steps are completed the Polkadot template file can be imported. Under `All templates/Template App Polkadot` there is a `Macros` section with several parameters that can be configured (e.g. threshold for min. number of peers), in particular the path to the log file must be set. Do not change those values there, instead go to `Hosts` and select the particular host, then go to `Macros`, then to `Inherited and host macros`. There the macros from the generic template are mirrored for the specific host and can be set without affecting other hosts using the same template.

Testing heartbeat alarm: In the config section of the script set `heartbeatoffset` to a high negative value like -1000, then a false alarm should get triggered before heartbeat-after-height. 

### New

More options for logfile management (option 1 or 2 recommended for Zabbix)

### Issues

The js-api requests are slow, some differentials might be slightly inaccurate.

If suppression of error messages is preferred, start like `./nmon.sh 2> /dev/null`.
