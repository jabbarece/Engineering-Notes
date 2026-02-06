
# HPE Slingshot Notes:

> [!NOTE] 
> Wiki
> 
> **FM work flow**
> -> [https://rndwiki-pro.its.hpecorp.net/display/SASSHOT/Fabric+Manager+Services+Startup+Sequence](https://rndwiki-pro.its.hpecorp.net/display/SASSHOT/Fabric+Manager+Services+Startup+Sequence)
> 
> **Slinghshot common services framework:**
> [https://rndwiki-pro.its.hpecorp.net/display/SASSHOT/Slingshot+Common+Services+Framework+Architecture](https://rndwiki-pro.its.hpecorp.net/display/SASSHOT/Slingshot+Common+Services+Framework+Architecture)


## Build

First -> 
```bash
./runBuildPrep.sh
```

Then -> 
```bash
mvn clean -T 1C package -DskipTests #(First time only)
```

Next time onwards -> 
```bash
mvn -o -T 1C package -DskipTests
```


## Commands:

### Build the mac agent branch (Typically all its dependent as well)

1. Pull the parent branch [https://github.hpe.com/hpe/hpc-hms_ec-hms-scimage](https://github.hpe.com/hpe/hpc-hms_ec-hms-scimage)
2.  `export WORKAREA=/home/ubuntu/work/debian_build`
3.  `cd hpc-hms_ec-hs-scimage`
4. `./build_all` (First time)
5. For subsequent build of mac_agent branch alone (`./build_all -s 76 -e 76`)

## Curl command to post the REST payload

```bash
    curl -H "Content-Type: application/json" -X POST -d '{
    "testName": "Jabbar",
    "vniTestId": [
        "100"
    ]
}' http: //localhost:8000/fabric/vni/test | jq
```

## Fabric manager show status

```bash
fmn-show-status
```

## Print logs in FM and FA

```bash
journalctl -u fabric-agent-host -f
journalctl -u fabric-manager -f
```

## Topology json file:

`/opt/cray/topology_template.json`

## Recover RBT from bad state:

[https://rndwiki-pro.its.hpecorp.net/display/SASSHOT/How+to+load+FMN+and+switch+firmware+manually+on+SLES+and+RHEL](https://rndwiki-pro.its.hpecorp.net/display/SASSHOT/How+to+load+FMN+and+switch+firmware+manually+on+SLES+and+RHEL)


## To get the mgmt IP of switches and compute nodes

```bash
cat /etc/hosts 
```

## On the switch manually settiing Cray TLV

```bash
root@x0c0r0b0:~# /usr/sbin/lldptool set-tlv -r -V CrayNet enableTx=yes network='{
    "ip_addr": "192.168.1.10”,”ttl": "forever"
}' -i ros0p18
```

## TCPDUMP command:

```bash
tcpdump -i hsn0 -nn -s 1500 -c 10 -X ether proto 0x88cc
```


```bash
#export HPE_GITHUB_TOKEN=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
```

## SITF Repo:

[https://github.hpe.com/hpe/hpc-lonesshot-sitf-tests/tree/master/tests/40_net_protocols](https://github.hpe.com/hpe/hpc-lonesshot-sitf-tests/tree/master/tests/40_net_protocols)

## SITF Setup on VM

[https://github.hpe.com/hpe/hpc-lonesshot-sitf-py/blob/master/README.md](https://github.hpe.com/hpe/hpc-lonesshot-sitf-py/blob/master/README.md)

## SITF pre-merge test run on slingshot-devtest-system-request channel for PR:
```
!sitf pre-merge https://github.hpe.com/hpe/hpc-lonesshot-fmn-minimal/pull/774 --branch 2.9 --email abdul-jabbar.s@hpe.com
```

## STT configuration ->

[https://github.hpe.com/hpe/hpc-sshot-slingshot_docs/blob/e7214cddc99f7e7cc1611ed5b2c5a9b34e750d54/portal/developer-portal/snippets/fm/configure_fmn_p2p_shasta.md](https://github.hpe.com/hpe/hpc-sshot-slingshot_docs/blob/e7214cddc99f7e7cc1611ed5b2c5a9b34e750d54/portal/developer-portal/snippets/fm/configure_fmn_p2p_shasta.md)

## SITF pre-merge test:

1. clone sitf-jenkins and sitf-tests repo in parallel
2. `make sitf`
3. in `tools/sitf-newton7.cfg` (modify required fields)
4. `make clean` 
5. `make local_test SYSTEM=newton7 > log 2>&1`


## Switch reset:
```bash
fmn-reset-switch -a -r
```


## PR squah technique using GIT command: (This will retain the same PR number, but will erase the commit history)
```bash
git checkout new-clean-branch
git push origin new-clean-branch:SSHOTCP-8175 --force
```

(or)
```bash
git reset --soft <main’s head commit hash> && git commit -m "SSHOTCP-8175: Disable MAC learning per VLAN, Port and VLAN+Port"
git push --force origin <branch-name>
```


## Cray TLV (nicConfigMap) python script 

-> [https://github.hpe.com/hpe/hpc-lonesshot-fmn-minimal/blob/master/scripts/fmn-update-hsn-nic-config](https://github.hpe.com/hpe/hpc-lonesshot-fmn-minimal/blob/master/scripts/fmn-update-hsn-nic-config)
-> For this to work, the template.json file should have hsnIP (For that follow STT link above)
        To change the Cray TLV (nicConfigMap) for a given port, do `fmctl update fabric/agent/x0c0r0b0 -f cray_tlv.json | jq`)

`root@sms1 sabdulja# cat cray_tlv.json` 
```json
{
    "desiredStatus": {
        "nicConfigMap": {
            "x0c0r0j3p0": {
                "ipAddress": "192.168.0.12/16",
                "mtu": 9000,
                "ttl": "forever"
            }
        }
    },
    "actualStatus": {
        "nicConfigMap": {
            "x0c0r0j1p0": {
                "ipAddress": "1.1.1.1/16",
                "mtu": 0,
                "ttl": ""
            }
        }
    }
}
```

## POC:

James Swaro -> Slingshot Host
RBT login issue POC -> Clayton Graves


## Code:

1. LLDP true/false fabricProperyMap is handled in -> `TopologyPolicyService`
2. `FabricAgentsService.java` ->  Where desired and actual status is processed (Eg: syncLldp)


## FM service restart:

```bash
systemctl stop fabric-manager
rm -rf /opt/slingshot/data/slingshot/fabric-manager/8000/
systemctl start fabric-manager
sleep 15
fmctl update /host-settings migrationMode=false -r | jq . 
fmn-create-switch-inventory
fmn-update-password --admin-credential root:initial0 root:initial0 --commit --all
fmn-create-certificate --cert-only --all
fmn-update-fabric-policy
fmn-update-fabric-links
fmctl update topology-policies/template-policy active=true
fmn-show-status
```

> [!CAUTION] 
> Note: (If the scripts throws error, use below)
> ——————————————————————
> ```bash
> export PYTHONPATH=/opt/slingshot/lib/python3/venv/lib/python3.9/site-packages
> 
> systemctl stop fabric-manager
> rm -rf /opt/slingshot/data/slingshot/fabric-manager/8000/
> systemctl start fabric-manager
> sleep 15
> fmctl update /host-settings migrationMode=false -r | jq . 
> /opt/python39/bin/python3.9 /usr/bin/fmn-create-switch-inventory
> fmn-update-password --admin-credential root:initial0 root:initial0 --commit --all
> fmn-create-certificate --cert-only --all
> fmn-update-fabric-policy
> /opt/python39/bin/python3.9 /usr/bin/fmn-update-fabric-links
> fmctl update topology-policies/template-policy active=true
> /opt/python39/bin/python3.9 /usr/bin/fmn-show-status
> ```

## FA Service restart:
——————————
```bash
systemctl stop fabric-agent-host
rm -rf /opt/slingshot/data/slingshot/8000/lucene/
systemctl start fabric-agent-host
```



## Get port status on the switch using swtest:

`# jack_to_port -x <Slingshot_Port_xname>`

The following output is displayed:
```
[root@gl-slingshot-fmn ~]# jack_to_port -x x3000c0r21j4p1
j4p1: p3
```

1. Check the port status from the switch.

`# swtest -c "port $((1 << <Slingshot_Port_number> )) info"`

The following output is displayed:
```
root@x3000c0r21b0:~# swtest -c "port $((1 << 3)) info"
** 3 **
configured: tpml
type: ethernet
subtype: ieee
state: running
uptime: 98148 (1d 3h 15m 48s)
dfa: ff800218
name: "x3000c0r21j4p1"

….
….
```

## Fetch Redfish DB cable info from the switch:

Execute below on switch or from the FMN with switch name:[x0c0r0b0]
```bash
    curl -k -u root:initial0 https://127.0.0.1/redfish/v1/Chassis/Enclosure/NetworkAdapters/Rosetta/Oem/NetworkCables | jq
    # -> This DB contatains the cable info that "fmn-check-cables" script from FMN uses to find the cable connectivity details.
```

MAC Address Vendor
* 00:40:a6 -> Cray Inc
* ec:0d:9a -> Mellanox Technologies, Inc.


## How to setup opengrok

1. First setup the tomcat10 using below link (Follow all the steps)
    -> [https://www.digitalocean.com/community/tutorials/how-to-install-apache-tomcat-10-on-ubuntu-20-04](https://www.digitalocean.com/community/tutorials/how-to-install-apache-tomcat-10-on-ubuntu-20-04)
2. Setup opengrok   
    -> [https://opstree.com/blog/2021/07/13/opengrok-setup-and-features/](https://opstree.com/blog/2021/07/13/opengrok-setup-and-features/)


## SITF and container on Ubuntu Arm64

```bash
sudo apt install -y python3-venv

source ~/venv/bin/activate

sudo apt update
sudo apt install -y qemu-user-static
export CONTAINER_DEFAULT_PLATFORM=linux/amd64
sudo update-binfmts --enable qemu-x86_64

sudo systemctl enable binfmt-support  #(-----> Enable on boot)
```




## Slingshot Architecture:
——————————————
￼
User set port to be Online on port-policy
	->  Policy validator, (a periodic service that runs on FM) sends patch request to FabricAgentsService -  (using UPDATE_SWITCH_AGENTS( AgentState - Desired status &Actual Status)
	-> Fabric Agent service handle this patch and send patch request to FA.
		-> (On FA - User space) -> `AgentActual` -> `AgentDesired` -> `JNI` -> `SDK` -> `librossw`				
                -> (On FA - Kernel space) -> `PCIe` (PCI Express is a kernel module/driver) -> `ASIC`

DeploymentService detects mismatch in desired vs actual state periodically
										
> [!NOTE] 
> ->If the deployment service fails to update for some reason use workaround, `fmctl update agents/x0c0r0b0 switchConfiguratinSyncTask.stage=CREATED`														

**what is FabricAgentsService ?**
			--> This runs on FM and FA
                       —> On FM, this handles the patch from policy validator and sends the patch request to FA (desired status will be sent)
			--> On FA, Deployment service will send a patch to FabricAgentService when there is a mismatch in actual vs desired. 

**FM doesn't talk to FA directly**
	FM -> Send HTTPS packet -> Received by NGINX on Rosetta (Agent), as Ngnix is the one validate the https certificate. Then send to FA as HTTP packet
	Hence, we should not capture the packet on mgmt interface as it is encrypted. we have to do tcpdump on loopback on Rosetta for debugging the http payload
	Nginx logs can be found in --> `/var/log/nginx/access.log` (Grep for PATCH)	

Via the above path,FM will patch the Desired status to FA periodically.

**How this config applied to the hardware?**
	-> Once FA receives the patch of desired status, Deployment Service on FA will start an FSM --> Send patch to FabricAgentService -> Calls syncMethod (eg: syncLLDP) ->  Calls Rosetta API (V1 or V2 instance), --> JNI  (h/w specific libarry) -> SDK --> (on Return SUCCSS) update the Actual status on FA (now actual and desired will be identical)

**Kernel Space:**
	-> There are 3 kernel modules in Rosetta kernel (rossw, roscore and rosnic)
	-> rosnic kernel module is responsible for exposing the ros0px interfaces to the Userspace. This is useful to send/receive LLDP packet from userspace on the edge ports. 

`chfs` -> This is a filesystem in rosetta, where some hardware info related to switch (Eg. fan) will be there (/chfs is the path)

`Redfish` -> This is standard REST APIs support in Rosetta that will give interface and cable information

`swtest` -> This is a kind of daemon (not sure), that will skip the Rosetta s/w stack and directly call the kernel (rossw) via librossw. Like in some  cases, we can simulate cable unplug without physically unplug the cable.
			[https://rndwiki-pro.its.hpecorp.net/pages/viewpage.action?pageId=171121796](https://rndwiki-pro.its.hpecorp.net/pages/viewpage.action?pageId=171121796)
			-> Eg: `swtest` (This is the command to enter the swtest cli mode in Rosetta switch)
			-> `p2m 58` (This is to add the port number 58 to the bitmap)
                       -> `link map down` (This will bring the link down. This is similar to unplug the cable from port. when the port goes down due to physical cable remove, rossw module will send notification to SDK.)
			-> So like above we can simulate some h/w related tests


## FM/FA code flow
————————————

User (set LLDP = true)
	-> `TopologyPolicyService.handlePatch` (Receives the payload with referer unknown client)
			-> Just do `post.complete()` and return to the client
	-> `TopologyPolicyService.handlePatch` (Receives patch from periodicMaintance from the self service)
			-> Calls “processPropertyMaps”
			-> Calls “mapFabricProportiesToAgentsAsync”
 			-> Calls “mapTopologyProportyToAgentsAsync”
			-> Calls “reapplyPolicyToSwitchAgentIfNeededAsync”
			-> Sends patch to Agent service “agentState.documentSelfLink”
			-> Agent service will just update the “desiredStatus.lldp” and send patch.complete (this will not send to the switch/agent)
 
> [!NOTE] 
> 	-> The FabricPropertyMapHelper will send “n” number of patch if “n” switches exists. It gets the switch info from switchGroup (getSwitchGroupAsync)
> 	-> There will be one FabricAgentsService per agent/switch.


**Deployment Service:**
		->	After the above, the DeploymentService that runs periodically will find the diff between desired vs actual and send the patch to FabricAgentService (switchConfigurationSyncTask 
 Will be set to create in this patch message). 
		-> 	Once the `FabricAgentsService.handlePatch` is invoked, it will send the patch to the remote agent URI and wait for completion.
		->	When the `FabricAgentsService.handlePatch` in FA is invoked, it will call the syncMethod
		-> 	Then it will call JNI and SDK API
		-> 	Once the SDK API return SUCCESS, the actual status in the agent will be updated.		->	FA will send the response to the FM’s FabricAgentsService (where it waits for completion handle)
		->	On FM’s FAbricAgentsService, when it receives the completion handler, it will update the actual status for the given agent in the FM’s DB.


## MAC Learning Notes:
-> . Non Algorithmic or Globally Assigned MAC address learning
The Ingress Transform requires that the Ethernet Lookup (ELU) Hash table has been populated with all the required globally assigned MAC address hosts that are in the node’s VLAN. This has to be done by a software learning process.


## MAC Agent build

```bash
ubuntu@sabdulja hpc-hms_ec-hms-scimage# ./build-all -s 52.1 -e 52.9
Running order:
52.1_build_sshot_swss_common
52.2_build_msg_broker
52.3_build_sshot_db_library
52.9_build_common_infra_examples
```


If some packages not found download it from working switch and. place the .deb file in the `“$WORKAREA/artifacts/packages/arm64/“`

Command to spawn the build container without it getting deleted after build:
```bash
	docker run -dit   -w /work/mac_agent   --user ubuntu   --name mac-agent-250326001015   --volume /work/hpc-hms_ec-hms-scimage/tools:/tools:Z   --volume /work/workarea2:/work:Z   --volume /work/workarea2/artifacts:/artifacts:Z   --volume /work/workarea2/apt_proxy.conf:/etc/apt/apt.conf.d/proxy.conf:Z   --network=host   x86-base:250326001015   bash
```

Command to run build on the above container:
```bash
	docker exec -it mac-agent-250326001015 /tools/helper-dpkg-build-sshotsw-repo mac-agent
```


## MAC Agent on Switch container

	Set below, as Mac agent depends on a library in this path
		`export LD_LIBRARY_PATH=/opt/hpe/r1/lib/x86_64-linux-gnu:$LD_LIBRARY_PATH`
       
       If the doTask is not called when the redis-db is updated, we need to enabled notify-keyspace-events as below:
```bash
       		root@x3000c0r40b0:/sonic-swss-common/tests# redis-cli CONFIG GET notify-keyspace-events
		1) "notify-keyspace-events"
		2) ""
		root@x3000c0r40b0:/sonic-swss-common/tests# redis-cli CONFIG SET notify-keyspace-events "KEA"
		OK
		root@x3000c0r40b0:/sonic-swss-common/tests# redis-cli CONFIG GET notify-keyspace-events
		1) "notify-keyspace-events"
		2) "AKE"
```

	If the docker-compose build fails, run below
	——————————————————————
	`z`


## REDIS COMMAND:
—————————

### Docker build container:

**sshot_db_connector:**
```bash
docker run -w /work/sshot_db_connector -i -t   --name sshot-db-connector-250409133656   --user root   --volume /work/hms_amd/hpc-hms_ec-hms-scimage/tools:/tools:Z   --volume /work/hms_amd/workarea:/work:Z   --volume /work/hms_amd/workarea/artifacts:/artifacts:Z   --volume /work/hms_amd/workarea/apt_proxy.conf:/etc/apt/apt.conf.d/proxy.conf:Z   x86-base:250409133656 /bin/bash

# /tools/helper-dpkg-build-sshotsw-repo sshot-db-connector "downgrade_doxygen"
```

**mac-agent:**
```bash
docker run -w /work/mac_agent -i -t --name mac-agent-build-amd --user root   --volume /work/hms_amd/hpc-hms_ec-hms-scimage/tools:/tools:Z   --volume /work/hms_amd/workarea:/work:Z   --volume /work/hms_amd/workarea/artifacts:/artifacts:Z   --volume /work/hms_amd/workarea/apt_proxy.conf:/etc/apt/apt.conf.d/proxy.conf:Z   x86-base:250425034852 /bin/bash
```

**mac-agent aarch64:**
```bash
docker run -w /work/mac_agent -i -t --name mac-agent-build-arm --user root   --volume /work/mac-learn-ut/hpc-hms_ec-hms-scimage/tools:/tools:Z   --volume /work/mac-learn-ut/workarea:/work:Z   --volume /work/mac-learn-ut/workarea/artifacts:/artifacts:Z   --volume /work/mac-learn-ut/workarea/apt_proxy.conf:/etc/apt/apt.conf.d/proxy.conf:Z   x86-base:250425034856 /bin/bash
```


## Compiler optimization

```bash
cmake -DCMAKE_C_FLAGS="-g -O0" -DCMAKE_CXX_FLAGS="-g -O0" ..
```


## sudo fix for container build:

```bash
chown root:root /usr/lib/sudo/sudoers.so
chown root:root /etc/sudoers
chmod 0440 /etc/sudoers
chown root:root /etc/sudo.conf
chmod 0644 /etc/sudo.conf
chown root:root /usr/bin/sudo
chmod 4755 /usr/bin/sudo
chown -R root:Debian-exim /var/lib/exim4
chown -R Debian-exim:adm /var/log/exim4
chmod 640 /var/log/exim4/mainlog
chown root:root /etc/sudoers.d/README
```


## GDB python debugging on mac-agent:

```bash
apt install g++-10 libstdc++6-10-dbg
```



## wget command to download all files in a directory of http server: (place the files in /var/www/html/mac-learning)

```bash
wget -r -np -nH --cut-dirs=1 http://10.103.17.132/mac-learning/ && rm -rf *index*
```



## Docker Swarm:

```bash
docker network create --driver overlay --attachable ss-net
docker network create --driver overlay --subnet 20.10.0.0/16 ss-net
docker swarm init --advertise-addr 10.22.173.142
docker swarm join --token <WORKER_TOKEN> <MANAGER-IP>:2377
docker swarm leave        # For worker node
docker swarm leave --force  # Forcefully leave (for manager)
docker node ls
docker node rm <NODE-ID>
docker stack deploy -c docker-compose.yml <STACK_NAME>
docker stack ls
docker info | grep -A10 'Swarm:'
 
docker service ls -q | xargs -r docker service rm #Stop all service
```

**To distribute the switches across the compute nodes: Run this command in the yml file in vim**
```vim
:%s/^\(\s\+\)hostname:.*$/\0\r\1deploy:\r\1  mode: replicated\r\1  replicas: 1\r\1  placement:\r\1    preferences:\r\1      - spread: node.hostname/
```



## Socket check:

```bash
root@x2001c0r1b0:/common-services-infra# sudo ss -ap --packet
Netid            State             Recv-Q            Send-Q                       Local Address:Port                         Peer Address:Port            Process            
p_raw            UNCONN            0                 0                                       ip:eth0                                     *                 users:(("maclppd",pid=9475,fd=7))
root@x2001c0r1b0:/common-services-infra# 
```


## Create Vritual interface on Switch:

```bash
# Create veth pair: eth10 <--> ros0
ip link add eth10 type veth peer name ros0
ip link set eth10 up
ip link set ros0 up 
#Create Bridge interface 
ip link add br0 type bridge
ip link set br0 up
ip link set eth10 master br0  

#Assign switch MAC address to ros0
ip link set ros0 down
ip link set ros0 address  02:ff:00:00:00:00
ip link set ros0 up  

#Now the packet sent on eth10 should be received on ros0 and maclppd will pickup the packet for processing
```


## GIT rebase: (branch2 is pulled from branch1 and branch1 has some commits)

```bash
# First, make sure you're on your child branch
git checkout <branch2>

# Fetch latest changes
git fetch origin

# Rebase your current branch onto updated parent
git rebase origin/<branch1>

#If push issue comes after rebase
git push --force-with-lease origin <branch2>

#If the base is pointing to main (use interactive rebase)
git rebase -i origin/<base-branch> 
	-> Then pick only the newer commit. Save and exit 
       -> git push --force-with-lease origin <target-branch>
```

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## During  (docker build) and get permission denied for git clone:

```bash
echo "ghp_xxxxxxxxxxxxxxxxx" > ~/git_pat.txt          ———> Place the Github token here

DOCKER_BUILDKIT=1 docker build \
  -f dockerfile_dev \
  -t ss-mgmt-image \
  --secret id=git_pat,src=$HOME/git_pat.txt .
```

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------



## VS Code Tunnel Setup Notes

1. Start the tunnel in the background:
   `code tunnel &`

2. Login to your account:
   `code tunnel user login`

3. Authenticate:
   - Copy the login link shown in the terminal
   - Open the link in a browser
   - Enter the code provided
   - Sign in with your GitHub or Microsoft account

4. Connect:
   - Use the tunnel link shown in the terminal
   - OR in VS Code desktop app: Command Palette → Remote: Connect to Tunnel → select your tunnel

**Optional:**
- To logout:  `code tunnel user logout`
- To check current account:  `code tunnel user`
- To run tunnel as a service (auto-start after reboot):
```bash
    code tunnel service install --accept-server-license-terms
    systemctl --user enable code-tunnel
    systemctl --user start code-tunnel
```

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


## If the mac-agent on the switch is not able to connect to the msg-broker on the FMN (as conainer). Then it could be a firewall/iptable issue

```bash
napier2-fmn1:~/jabbar # sudo iptables -L -n | grep 1883
ACCEPT     tcp  --  0.0.0.0/0            192.168.48.4         tcp dpt:1883     #>>>>>>> This means, the tcp connection is restricted to 192.x n/w (which is container network)

napier2-fmn1:~/jabbar # sudo iptables -I DOCKER-USER -p tcp --dport 1883 -j ACCEPT #>>>>>> Run this command to accept connection on all the interfaces of FMN
napier2-fmn1:~/jabbar # 
napier2-fmn1:~/jabbar # sudo iptables -L -n | grep 1883
ACCEPT     tcp  --  0.0.0.0/0            192.168.48.4         tcp dpt:1883
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:1883
napier2-fmn1:~/jabbar # 
```

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Artifactory links:
1. FM/FA RPMs -> [https://arti.hpc.amslabs.hpecorp.net/ui/native/slingshot-rpm-unstable-local/predev/main/rocky_8_9/x86_64/](https://arti.hpc.amslabs.hpecorp.net/ui/native/slingshot-rpm-unstable-local/predev/main/rocky_8_9/x86_64/)

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## ros0 stats command

```bash
cat /sys/class/net/ros0/statistics/rx_packets
cat /sys/class/net/ros0/statistics/rx_dropped

grep ros0 /proc/net/dev

ip -s link show ros0
```

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Rosetta h/w commands:

1. Check the MAC entries in hash
```bash
		root@x3000c0r6b0:/tmp# hashdump -p 0                                                                                                                                  
		L2 config: enabled=1 port_l2=0 loopback_l2=0 vlan_l2=0                                                                                                                
		DBG: c000000000041000 0000000020200000 00041000                                                                                                                       
		593 : 00:40:a6:95:15:38       DFA: u-g0s1p1        : 1536 => 307                                                                                                      
		DBG: c000000000001000 0000000020200000 00001000                                                                                                                       
		1330: 00:40:a6:89:e6:7d       DFA: u-g0s0p1        : 1536 => 307 
```

 2. To check TCAM (Can be checked if the packet is forwarded by unknown unicase flood or not)
```bash
		watch -n 1 dgrtcamdebug -p 0
```

3. To check if the mac-learning enabled
```bash
		root@x3000c0r6b0:/tmp# dgrcsr -p 19 -n r_pf_elu_cfg_out_learn[0]                                                                                                      
			Port 19:              r_pf_elu_cfg_out_learn (0x44F806C0) = 0x04c4000000000000           <<<<<<<<< 0xc*** = Enabled, 0x0*** = Disabled
```
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Increase the “terminal width on switch”
```bash
		stty cols 120
```

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

> [!NOTE] 
> 	When the FA start, it will connect to Redis, if the connection is not made successful FA will not start. So to test SSDB connect retry, we cannot stop redis at the beginning. so instead use below rule after 5 to 10 seconds of FA start. This will help simulating SSDB connect failure.

**iptable to Block Redis connection:**
```bash
			iptables -A OUTPUT -p tcp --dport 6379 -j REJECT
```

**To remove: (Get the number and use that number to remove rule)**
```
	Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
	num   pkts bytes target     prot opt in     out     source               destination         
	1    28353  261M            all  --  *      *       0.0.0.0/0            0.0.0.0/0           
	2    25402  261M            all  --  *      *       0.0.0.0/0            0.0.0.0/0           
	3        3   184 REJECT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:6379 reject-with icmp-port-unreachable
```

**To remove ->** `iptables -D OUTPUT 3`

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Native Memory Tracking

**Add these args to the FA java:**
`-XX:+UnlockDiagnosticVMOptions -XX:NativeMemoryTracking=summary`

**Shell command to monitor memory:**
```bash
root@x0c0r0b0:/opt/slingshot# PID=$(pgrep -f slingshot-fabric-manager)
root@x0c0r0b0:/opt/slingshot# grep -E 'VmRSS|RssAnon|VmData|Threads' /proc/$PID/status > FA-T0 && sleep 900 && grep -E 'VmRSS|RssAnon|VmData|Threads' /proc/$PID/status > FA-T15
```

**Memory Output Example:**
```bash
root@x0c0r0b0:/opt/slingshot# cat FA-T0
VmRSS:    361152 kB
RssAnon:  317888 kB
VmData:   568892 kB
Threads:  69
```

> [!NOTE] 
> `jcmd 10258 VM.native_memory detail > NTM-T60`
> 
> **Here the RssAnon memory -> is anonymous memory which is outside of JVM's Garbage Collector (Mostly from Native library)**


## How to run ASAN instrumented FA
-> [https://jira-pro.it.hpe.com:8443/browse/SSHOTCP-8820](https://jira-pro.it.hpe.com:8443/browse/SSHOTCP-8820)

**Steps to run ASAN-instrumented Fabric Agent (FA):**

1. **Clone hms_scimage** (release/3.0 branch) -> [https://github.hpe.com/hpe/hpc-hms_ec-hms-scimage/tree/release/slingshot-3.0](https://github.hpe.com/hpe/hpc-hms_ec-hms-scimage/tree/release/slingshot-3.0)
2. **Instrument the required native libraries** (e.g., librossdk2, routingJNI, etc.) using **ASAN flags**.
   Refer: [https://github.hpe.com/hpe/hpc-sshot-rosetta2-sdk/pull/867](https://github.hpe.com/hpe/hpc-sshot-rosetta2-sdk/pull/867)
3. **Build all the libraries and generate the ITB{}**
4. Upgrade/Downgrade the **Rosetta-2 fabric** to **3.0.0{}**
5. Load the ITB on the switch *(Strictly Rosetta-2 hardware – not supported on simulator)*
6. **Perform the fabric bring-up on the FMN** (regular bring-up)
7. Stop the FA process on the switch:
   ```bash
   systemctl stop fabric-agent-host
   ```
8. Add the following lines to the FA service file (`/lib/systemd/system/fabric-agent-host.service`):
   ```ini
   [Service]
   Environment=LD_PRELOAD=/lib/aarch64-linux-gnu/libasan.so.6
   Environment=ASAN_OPTIONS=detect_leaks=1:abort_on_error=1
   Environment=LSAN_OPTIONS=verbosity=1
   ```
9. **Reload systemd and restart FA:**
   ```bash
   systemctl daemon-reload
   systemctl restart fabric-agent-host
   ```
   * The fabric-agent-host journalctl logs will now print **ASAN verbose output**.

### Observation from testing:
In my testing, ASAN reports native memory leaks shortly after FA startup (approximately in **43 seconds**). There may be additional leaks in other libraries or SDK during later stages of fabric bring-up; however, ASAN aborts the FA process early because it detects **significant native leaks during startup itself**.


## Debugging / Verification Steps (To confirm the loaded ITB is properly instrumented)

**1. Verify ASAN is linked into the SDK library (Or whichever is/are instrumented)**
```bash
ldd /usr/lib/aarch64-linux-gnu/librossdk2.so | grep asan
# Expected output should contain:
# libasan.so.6 => /lib/aarch64-linux-gnu/libasan.so.6
```

**2. Verify Fabric Agent (Java) process is running (Check after starting the FA with new service file)**
```bash
pgrep -a java
```

**3. Verify SDK library is loaded by FA using /proc**
```bash
PID=$(pgrep -f slingshot-fabric-manager)
cat /proc/$PID/maps | grep librossdk2

# This confirms the ASAN-instrumented librossdk2.so is loaded into the FA JVM process.
# (Optional – list all loaded shared libraries)
cat /proc/$PID/maps | grep '\.so'
```

**4. Verify ASAN runtime initialization**
```bash
journalctl -u fabric-agent-host | grep AddressSanitizer
# Expected messages include:
# AddressSanitizer Init done
# libc interceptors initialized
```

**5. Observe ASAN leak summary**
**Example:**
```
SUMMARY: AddressSanitizer: <bytes> leaked in <allocations>
```

**Expected behavior**
* FA process aborts after ASAN detects leaks (abort_on_error=1)

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

