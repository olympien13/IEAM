# IEAM 4.1 Agent Set-up for Windows Subsystem for Linux and Ubuntu 18.04

In order to abe to install the non-supported IEAM Agent I built compatible with a IEAM4.1 Hub and running on your Windows 10 OS, you'll need to go through this simple process that should take too long to set-up:

1. Enable Windows Subsystem for Linux V2
2. Enable Virtual Machine Platform
3. Set WSL V2 as default
4. Install Linux Distribution Ubuntu 18.04
5. Set-up your Ubuntu DNS Configuration and TIPS for VPN Access
6. Install Docker on Ubuntu 18.04
7. Install your IEAM Agent x86_64-Linux version
8. Replace the way the default agent is started with my own script
9. Then Enjoy your IEAM Services on your Windows 10 Platform

Don't be afraid of the length of this installation guide, most of the text is only the result of what you should see as a result for output commands.

## 1. Enable Windows Subsystem for Linux V2

Windows Subsystem for Linux has two different versions to choose between during the installation process. WSL 2 has better overall performance and we recommend using it 1. Enable WSL
To do this open the **PowerShell tool as an Administrator** and run the command below. Be careful not to mistype or leave out any character in the command:

```
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

## 2. Enable Virtual Machine Platform

WSL 2 requires Windows 10’s “Virtual Machine Platform” feature to be enabled. This is separate from Hyper-V and hands some of the more interesting platform integrations available in the new version of the Windows Subsystem for Linux.
To enable Virtual Machine Platform on Windows 10 (2004) open **PowerShell as Administrator** and run:

```
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

> **To ensure all of the relevant bits and pieces fall neatly in to place you HAVE to restart your system at this point.**

## 3. Set WSL V2 as default

Download the Linux kernel update package provided in this GIT named "wsl_update_x64.msi" . Run the update package downloaded. Double-click to run - you will be prompted for elevated permissions, select ‘yes’ to approve this installation.

Once the installation is complete, move on to the next step - setting WSL 2 as your default version when installing new Linux distributions.
Open **PowerShell as Administrator** and run this command to set WSL 2 as the default version of WSL:

```
wsl --set-default-version 2
```

## 4. Install Linux Distribution Ubuntu 18.04

With WSL and the necessary virtualisation tech all in place all that is left for you to do is pick and install a Linux distro from the Microsoft Store.

Several different distros are available, including OpenSUSE, Pengwin, Fedora Remix, and Alpine Linux. But thhe one supported by IEAM.41 is Ubuntu 18.04 LTS.Select that one then.

https://www.microsoft.com/fr-fr/p/ubuntu-1804-lts/9n9tngvndl3q?rtc=1&activetab=pivot:overviewtab

## 5. Set-up your Ubuntu DNS Configuration and TIPS for VPN Access

- Open an Ubuntu 18.05 instance and check you can access a DNS server. for instance try to perform the command below:

```
curl https://www.google.com
```

- If you see this error:

```
curl: (6) Could not resolve host: www.google.com
```

Please type successively the following commands in order one by one. In this case we are forcing the DNS to be set the Google DNS 8.8.8.8? If you prefer one of your choice. Feel free to change the value after "**nameserver**"

```
sudo chattr -i /etc/resolv.conf
sudo rm /etc/resolv.conf
sudo bash -c 'echo "nameserver 8.8.8.8" > /etc/resolv.conf'
sudo bash -c 'echo "[network]" > /etc/wsl.conf'
sudo bash -c 'echo "generateResolvConf = false" >> /etc/wsl.conf'
sudo chattr +i /etc/resolv.conf
```

- If you see this error:

```
curl: (7) Failed to connect to www.google.com port 443: Connection timed out
```

> You may be obliged like I was to diable your firewall to allow outgoing HTTPS TCP Connection (IT WOULD BE REQUIRED to access your IEAM Hub)

- If like in my case, to access you IEAM Hub, you need a VPN connection to access it. Perform the following steps:
  In your Ubuntu windows instance,

```
$ vi /etc/hosts
```

And add the following lines, assuming you know your IEAM Hub Private IP Address (192.168.XX.YY) and your FQDN (icp-console.apps.domain.com)

```
# IEAM4.1 Private IP Address
192.168.XX.YY   icp-console.apps.domain.com
```

- Try to ping your IEAM Hub in your Ubuntu Window instance. If you are connected through a VPN and you can not.
  > Perform the following procedure successively:

1. Disconnect from your actual VPN Connection

2. Open a Windows Power Shell as Administrator and type the following command:

```
netsh interface set interface "vEthernet (WSL)" disable
```

3. Reconnect your VPN Connection

4. On the same Windows Power Shell as Administrator and type the following command:

```
netsh interface set interface "vEthernet (WSL)" enable
```

## 6. Install Docker on Ubuntu 18.04

- Open an Ubuntu 18.05 instance and then type successively one by one the following commands

```
$ sudo apt-get update
$ sudo apt-get install -y apt-transport-https ca-certificates curl gnupg-agent  software-properties-common
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release) -cs) stable"
$ sudo apt-get update
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
$ sudo /etc/init.d/docker start
$ sudo /etc/init.d/docker status
$ sudo docker run -it hello-world
$ sudo docker ps
```

## 7. Install your IEAM Agent x86_64-Linux version

- Get the agentInstallFiles-x86_64-Linux.tar.gz file and the appropriate API key from your administrator.
- Open an Ubuntu 18.05 instance and Copy the archive file under the Ubuntu 18.04 folders (your windows virtual disls are accessible through **"/mnt"** mount point, for instance cp /mnt/c/download/agentInstallFiles-x86_64-Linux.tar.gz).
- Place it in a directory of your choice under Ubuntu and then type successively one by one the following commands:

```
$ sudo apt update
$ sudo apt install jq
$ cp /mnt/c/download/agentInstallFiles-x86_64-Linux.tar.gz .
$ export AGENT_TAR_FILE=agentInstallFiles-x86_64-Linux.tar.gz
$ tar -zxvf $AGENT_TAR_FILE agent-install.sh
$ export HZN_EXCHANGE_USER_AUTH=iamapikey:<api-key>
$ sudo -s ./agent-install.sh -i . -u $HZN_EXCHANGE_USER_AUTH -p IBM/pattern-ibm.helloworld -w ibm.helloworld -o IBM -z $AGENT_TAR_FILE
```

- This last script should display the following content and stop at the same step shown above:

```
$ sudo -s ./agent-install.sh -i . -u $HZN_EXCHANGE_USER_AUTH -p IBM/pattern-ibm.helloworld -w ibm.helloworld -o IBM -z $AGENT_TAR_FILE
[sudo] password for flivigni:
Sorry, try again.
[sudo] password for flivigni:
2020-10-23 08:38:15 HZN_EXCHANGE_URL: https:///icp-console.apps.domain.com/edge-exchange/v1/ (from agent-install.cfg)
2020-10-23 08:38:15 HZN_FSS_CSSURL: https:///icp-console.apps.domain.com/edge-css/ (from agent-install.cfg)
2020-10-23 08:38:15 HZN_ORG_ID: mycluster (from agent-install.cfg)
2020-10-23 08:38:15 HZN_EXCHANGE_NODE_AUTH: (not found)
2020-10-23 08:38:15 HZN_EXCHANGE_USER_AUTH: ****** (from environment)
2020-10-23 08:38:15 NODE_ID: (not found)
2020-10-23 08:38:15 CERTIFICATE: (not found)
2020-10-23 08:38:15 HZN_MGMT_HUB_CERT_PATH: (not found)
2020-10-23 08:38:15 HZN_EXCHANGE_PATTERN: (not found)
2020-10-23 08:38:15 HZN_NODE_POLICY: (not found)
Installation packages location: .
Ignore package tree: true
Node policy:
NODE_ID=
Image Full Path On Edge Cluster Registry:
Internal URL for Edge Cluster Registry:
2020-10-23 08:38:15 Node type is: device
2020-10-23 08:38:15 Detection results: OS is linux, distribution is ubuntu, release is bionic, architecture is amd64
2020-10-23 08:38:15 Installing agent on ubuntu, version bionic, architecture amd64
2020-10-23 08:38:15 hzn not found, installing it...
+ set +e
+ dpkg -i ./bluehorizon_2.26.12~ppa~ubuntu.bionic_all.deb ./horizon-cli_2.26.12~ppa~ubuntu.bionic_amd64.deb ./horizon-cli_2.26.12~ppa~ubuntu.bionic_arm64.deb ./horizon_2.26.12~ppa~ubuntu.bionic_amd64.deb ./horizon_2.26.12~ppa~ubuntu.bionic_arm64.deb
Selecting previously unselected package bluehorizon.
(Reading database ... 29326 files and directories currently installed.)
Preparing to unpack .../bluehorizon_2.26.12~ppa~ubuntu.bionic_all.deb ...
Unpacking bluehorizon (2.26.12~ppa~ubuntu.bionic) ...
Selecting previously unselected package horizon-cli.
Preparing to unpack .../horizon-cli_2.26.12~ppa~ubuntu.bionic_amd64.deb ...
Unpacking horizon-cli (2.26.12~ppa~ubuntu.bionic) ...
dpkg: error processing archive ./horizon-cli_2.26.12~ppa~ubuntu.bionic_arm64.deb (--install):
 package architecture (arm64) does not match system (amd64)
Selecting previously unselected package horizon.
Preparing to unpack .../horizon_2.26.12~ppa~ubuntu.bionic_amd64.deb ...
Unpacking horizon (2.26.12~ppa~ubuntu.bionic) ...
dpkg: error processing archive ./horizon_2.26.12~ppa~ubuntu.bionic_arm64.deb (--install):
 package architecture (arm64) does not match system (amd64)
Setting up horizon-cli (2.26.12~ppa~ubuntu.bionic) ...
dpkg: dependency problems prevent configuration of horizon:
 horizon depends on docker-engine (>= 17.0) | docker-ce (>= 17.0) | docker (>= 17.0) | docker.io (>= 17.0); however:
  Package docker-engine is not installed.
  Package docker-ce is not installed.
  Package docker is not installed.
  Package docker.io is not installed.

dpkg: error processing package horizon (--install):
 dependency problems - leaving unconfigured
dpkg: dependency problems prevent configuration of bluehorizon:
 bluehorizon depends on horizon (= 2.26.12~ppa~ubuntu.bionic); however:
  Package horizon is not configured yet.

dpkg: error processing package bluehorizon (--install):
 dependency problems - leaving unconfigured
Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
Errors were encountered while processing:
 ./horizon-cli_2.26.12~ppa~ubuntu.bionic_arm64.deb
 ./horizon_2.26.12~ppa~ubuntu.bionic_arm64.deb
 horizon
 bluehorizon
+ set -e
2020-10-23 08:38:17 Resolving any dependency errors...
+ apt update
Hit:1 http://archive.ubuntu.com/ubuntu bionic InRelease
Get:2 http://archive.ubuntu.com/ubuntu bionic-updates InRelease [88.7 kB]
Get:3 http://security.ubuntu.com/ubuntu bionic-security InRelease [88.7 kB]
Get:4 http://archive.ubuntu.com/ubuntu bionic-backports InRelease [74.6 kB]
Get:5 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 Packages [1723 kB]
Get:6 http://archive.ubuntu.com/ubuntu bionic-updates/universe amd64 Packages [1680 kB]
Fetched 3655 kB in 17s (220 kB/s)
Reading package lists... Done
Building dependency tree
Reading state information... Done
63 packages can be upgraded. Run 'apt list --upgradable' to see them.
+ apt-get install -y -f
Reading package lists... Done
Building dependency tree
Reading state information... Done
Correcting dependencies... Done
The following additional packages will be installed:
  bridge-utils cgroupfs-mount containerd docker.io pigz runc ubuntu-fan
Suggested packages:
  ifupdown aufs-tools debootstrap docker-doc rinse zfs-fuse | zfsutils
The following NEW packages will be installed:
  bridge-utils cgroupfs-mount containerd docker.io pigz runc ubuntu-fan
0 upgraded, 7 newly installed, 0 to remove and 63 not upgraded.
2 not fully installed or removed.
Need to get 63.7 MB of archives.
After this operation, 319 MB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu bionic-updates/universe amd64 runc amd64 1.0.0~rc10-0ubuntu1~18.04.2 [2000 kB]
Get:2 http://archive.ubuntu.com/ubuntu bionic-updates/universe amd64 containerd amd64 1.3.3-0ubuntu1~18.04.2 [21.7 MB]
Get:3 http://archive.ubuntu.com/ubuntu bionic-updates/universe amd64 docker.io amd64 19.03.6-0ubuntu1~18.04.2 [39.9 MB]
Get:4 http://archive.ubuntu.com/ubuntu bionic/universe amd64 pigz amd64 2.4-1 [57.4 kB]
Get:5 http://archive.ubuntu.com/ubuntu bionic/main amd64 bridge-utils amd64 1.5-15ubuntu1 [30.1 kB]
Get:6 http://archive.ubuntu.com/ubuntu bionic/universe amd64 cgroupfs-mount all 1.4 [6320 B]
Get:7 http://archive.ubuntu.com/ubuntu bionic/main amd64 ubuntu-fan all 0.12.10 [34.7 kB]
Fetched 63.7 MB in 1min 38s (652 kB/s)
Preconfiguring packages ...
Selecting previously unselected package runc.
(Reading database ... 29386 files and directories currently installed.)
Preparing to unpack .../0-runc_1.0.0~rc10-0ubuntu1~18.04.2_amd64.deb ...
Unpacking runc (1.0.0~rc10-0ubuntu1~18.04.2) ...
Selecting previously unselected package containerd.
Preparing to unpack .../1-containerd_1.3.3-0ubuntu1~18.04.2_amd64.deb ...
Unpacking containerd (1.3.3-0ubuntu1~18.04.2) ...
Selecting previously unselected package docker.io.
Preparing to unpack .../2-docker.io_19.03.6-0ubuntu1~18.04.2_amd64.deb ...
Unpacking docker.io (19.03.6-0ubuntu1~18.04.2) ...
Selecting previously unselected package pigz.
Preparing to unpack .../3-pigz_2.4-1_amd64.deb ...
Unpacking pigz (2.4-1) ...
Selecting previously unselected package bridge-utils.
Preparing to unpack .../4-bridge-utils_1.5-15ubuntu1_amd64.deb ...
Unpacking bridge-utils (1.5-15ubuntu1) ...
Selecting previously unselected package cgroupfs-mount.
Preparing to unpack .../5-cgroupfs-mount_1.4_all.deb ...
Unpacking cgroupfs-mount (1.4) ...
Selecting previously unselected package ubuntu-fan.
Preparing to unpack .../6-ubuntu-fan_0.12.10_all.deb ...
Unpacking ubuntu-fan (0.12.10) ...
Setting up runc (1.0.0~rc10-0ubuntu1~18.04.2) ...
Setting up cgroupfs-mount (1.4) ...
invoke-rc.d: could not determine current runlevel
Setting up containerd (1.3.3-0ubuntu1~18.04.2) ...
Created symlink /etc/systemd/system/multi-user.target.wants/containerd.service → /lib/systemd/system/containerd.service.
Setting up bridge-utils (1.5-15ubuntu1) ...
Setting up ubuntu-fan (0.12.10) ...
Created symlink /etc/systemd/system/multi-user.target.wants/ubuntu-fan.service → /lib/systemd/system/ubuntu-fan.service.
invoke-rc.d: could not determine current runlevel
Setting up pigz (2.4-1) ...
Setting up docker.io (19.03.6-0ubuntu1~18.04.2) ...
Created symlink /etc/systemd/system/sockets.target.wants/docker.socket → /lib/systemd/system/docker.socket.
invoke-rc.d: unknown initscript, /etc/init.d/docker not found.
invoke-rc.d: could not determine current runlevel
Setting up horizon (2.26.12~ppa~ubuntu.bionic) ...
Setting up bluehorizon (2.26.12~ppa~ubuntu.bionic) ...
System has not been booted with systemd as init system (PID 1). Can't operate.
Created symlink /etc/systemd/system/multi-user.target.wants/horizon.service → /lib/systemd/system/horizon.service.
System has not been booted with systemd as init system (PID 1). Can't operate.
Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
Processing triggers for ureadahead (0.100.0-21) ...
Processing triggers for systemd (237-3ubuntu10.42) ...
Enter host password for user '-p':
Error: Can't connect to the Horizon REST API to run GET http://localhost:8510/node. Run 'systemctl status horizon' to check if the Horizon agent is running. Or set HORIZON_URL to connect to another local port that is connected to a remote Horizon agent via a ssh tunnel. Specific error is: Get "http://localhost:8510/node": dial tcp 127.0.0.1:8510: connect: connection refused
Error: Can't connect to the Horizon REST API to run GET http://localhost:8510/agreement. Run 'systemctl status horizon' to check if the Horizon agent is running. Or set HORIZON_URL to connect to another local port that is connected to a remote Horizon agent via a ssh tunnel. Specific error is: Get "http://localhost:8510/agreement": dial tcp 127.0.0.1:8510: connect: connection refused
Error: Can't connect to the Horizon REST API to run GET http://localhost:8510/node. Run 'systemctl status horizon' to check if the Horizon agent is running. Or set HORIZON_URL to connect to another local port that is connected to a remote Horizon agent via a ssh tunnel. Specific error is: Get "http://localhost:8510/node": dial tcp 127.0.0.1:8510: connect: connection refused
2020-10-23 08:40:45 Registering node with existing id
+ '[' -z agent-install.crt ']'
+ [[ a != \/ ]]
+ set -x
+ sudo cp agent-install.crt /etc/horizon/agent-install.crt
+ systemctl restart horizon.service
System has not been booted with systemd as init system (PID 1). Can't operate.
```

- Check your Horizon environment  
  You should see:

```
$ hzn env
Horizon Agent HZN Environment Variables are:
HZN_ORG_ID:
HZN_EXCHANGE_USER_AUTH:
HZN_EXCHANGE_URL: https://icp-console.apps.domain.com/edge-exchange/v1
HZN_FSS_CSSURL: https://icp-console.apps.domain.com/edge-css

```

- Add to your **.bashrc** file the following environment variables

```
$ vi $HOME/.bashrc
export HZN_ORG_ID=<your_org_id>
export HZN_EXCHANGE_USER_AUTH=iamapikey:<api-key>
export HORIZON_URL=http://localhost:8081
```

## 8. Replace the way the default agent is started with my own script

- Donwload the file named "horizon-container" provided in this GIT repo and place it under an Ubuntu folder
- Open an Ubuntu 18.05 instance and type in sequence the following commands

```
cd /usr/horizon/bin
sudo mv  horizon-container horizon-container.orig
cp <your_directory>/horizon-container .
sudo chmod 755 horizon-container
```

- You are now almost ready to start the IEAM Agent, but before you need to fetch the IEAM Agent I recompiled in order to make it work under WSL2/Ubuntu by running the command:

```
docker pull tncibmniceteam/amd64_win_anax:2.26.12
```

- You are now ready to start the IEAM Agent by running the command:

```
sudo horizon-container start
```

You should see the following content:

```
$ sudo horizon-container start
2.26.12: Pulling from tncibmniceteam/amd64_win_anax
Digest: sha256:14b110c01bd77a487960f2efd12345e813fd97d6ee7710a3bd8e6f673e434f38
Status: Image is up to date for tncibmniceteam/amd64_win_anax:2.26.12
docker.io/tncibmniceteam/amd64_win_anax:2.26.12
Starting the Horizon agent container tncibmniceteam/amd64_win_anax:2.26.12...
4c99296ab8d19f01e076462b9f1af0269fbebd2503d71083ee55e35989789ead
Horizon agent started successfully. Now export HORIZON_URL=http://localhost:8081, then use 'hzn node list', 'hzn register ...', and 'hzn agreement list'
```

- Do a "docker ps" to check that the AGENT is running. You can also check it at the Windows Docker Desktop Dashboard

```
flivigni@LAPTOP-BE2M10A3:/usr/horizon/bin$ docker ps
CONTAINER ID        IMAGE                                   COMMAND                  CREATED             STATUS              PORTS                      NAMES
4c99296ab8d1        tncibmniceteam/amd64_win_anax:2.26.12   "/root/anax.service …"   4 seconds ago       Up 3 seconds        127.0.0.1:8081->8510/tcp   horizon1
```

- Do a "hzn node list" command. You should see:

```
$ hzn node list
{
  "id": "LAPTOP-BE2M10A3",
  "organization": null,
  "pattern": null,
  "name": null,
  "nodeType": null,
  "token_last_valid_time": "",
  "token_valid": null,
  "ha": null,
  "configstate": {
    "state": "unconfigured",
    "last_update_time": ""
  },
  "configuration": {
    "exchange_api": "https://icp-console.apps.domain.com/edge-exchange/v1/",
    "exchange_version": "2.30.0",
    "required_minimum_exchange_version": "2.23.0",
    "preferred_exchange_version": "2.23.0",
    "mms_api": "https://icp-console.apps.domain.com/edge-css",
    "architecture": "amd64",
    "horizon_version": "local build"
  }
}
```

- Do a "hzn register" on the defaut IBM HelloWorld Sample to check that it is working fine

```
  $ hzn register -p IBM/pattern-ibm.helloworld
  Horizon Exchange base URL: https://icp-console.apps.domain.com/edge-exchange/v1
  Using node ID 'LAPTOP-BE2M10A3' from the Horizon agent
  Generated random node token
  Updating node token...
  Will proceeed with the given pattern IBM/pattern-ibm.helloworld.
  Initializing the Horizon node with node type 'device'...
  Warning: no input file was specified. This is only valid if none of the services need variables set (including GPS coordinates).
  However, if there is 'userInput' specified in the node already in the Exchange, the userInput will be used.
  Changing Horizon state to configured to register this node with Horizon...
  Horizon node is registered. Workload agreement negotiation should begin shortly. Run 'hzn agreement list' to view.
```

- Do a "hzn agreement list"

```
  hzn agreement list

  [
    {
        "name": "pattern-ibm.helloworld_ibm.helloworld_IBM_amd64 merged with pattern-ibm.helloworld_ibm.helloworld_IBM_amd64",
        "current_agreement_id": "756d332035c987cc634954d41ba7d2a425db337a5d144039513f10dd5bbe53c4",
        "consumer_id": "IBM/mycluster-agbot",
        "agreement_creation_time": "2020-10-23 09:55:42 +0200 CEST",
        "agreement_accepted_time": "2020-10-23 09:55:51 +0200 CEST",
        "agreement_finalized_time": "2020-10-23 09:56:08 +0200 CEST",
        "agreement_execution_start_time": "2020-10-23 09:55:57 +0200 CEST",
        "agreement_data_received_time": "",
        "agreement_protocol": "Basic",
        "workload_to_run": {
            "url": "ibm.helloworld",
            "org": "IBM",
            "version": "1.0.0",
            "arch": "amd64"
        }
    }
]
```

- Re-Do a "docker ps" to check that the IBM HelloWorld docker container is running. You can also check it at the Windows Docker Desktop Dashboard

```
$ docker ps
CONTAINER ID        IMAGE                                   COMMAND                  CREATED             STATUS              PORTS                      NAMES
92356fa5a75a        openhorizon/ibm.helloworld_amd64        "/bin/sh -c /service…"   33 seconds ago      Up 32 seconds                                  756d332035c987cc634954d41ba7d2a425db337a5d144039513f10dd5bbe53c4-ibm.helloworld
4c99296ab8d1        tncibmniceteam/amd64_win_anax:2.26.12   "/root/anax.service …"   11 minutes ago      Up 11 minutes       127.0.0.1:8081->8510/tcp   horizon1
```

- You can also play with all other HZN commands you know:

```
hzn version
hzn exchange version
hzn exchange status
hzn exchange user list
hzn exchange pattern list
....
```

- Finally to the agent, just run the command:

```
horizon-container stop
```

9. ## ENJOY !!!!!
