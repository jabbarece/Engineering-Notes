# 🚀 Slingshot / Cray / Rosetta / Fabric Manager Cheat Sheet

**Abdul's personal field notes** – HPE Slingshot development, debugging & operations  
**Last major update:** February 2026  
**Location vibe:** Bengaluru nights & coffee-fueled debugging sessions ☕

## 🔗 Must-Know Internal Links

- FM Startup Sequence  
  https://rndwiki-pro.its.hpecorp.net/display/SASSHOT/Fabric+Manager+Services+Startup+Sequence

- Slingshot Common Services Framework Architecture  
  https://rndwiki-pro.its.hpecorp.net/display/SASSHOT/Slingshot+Common+Services+Framework+Architecture

- Manual FMN + Switch Firmware Load (recovery from bad state)  
  https://rndwiki-pro.its.hpecorp.net/display/SASSHOT/How+to+load+FMN+and+switch+firmware+manually+on+SLES+and+RHEL

- FMN P2P / STT Config Snippet  
  https://github.hpe.com/hpe/hpc-sshot-slingshot_docs/blob/e7214cddc99f7e7cc1611ed5b2c5a9b34e750d54/portal/developer-portal/snippets/fm/configure_fmn_p2p_shasta.md

- SITF Tests (net protocols section)  
  https://github.hpe.com/hpe/hpc-lonesshot-sitf-tests/tree/master/tests/40_net_protocols

- SITF Python VM Setup  
  https://github.hpe.com/hpe/hpc-lonesshot-sitf-py/blob/master/README.md

- Cray TLV / nicConfigMap Update Script  
  https://github.hpe.com/hpe/hpc-lonesshot-fmn-minimal/blob/master/scripts/fmn-update-hsn-nic-config

- swtest CLI deep-dive wiki  
  https://rndwiki-pro.its.hpecorp.net/pages/viewpage.action?pageId=171121796

## 🛠️ Build – mac-agent & friends

```bash
# One-time prep
./runBuildPrep.sh

# First full build
mvn clean -T 1C package -DskipTests

# Fast rebuilds after that
mvn -o -T 1C package -DskipTests
mac-agent branch build
Bashgit clone https://github.hpe.com/hpe/hpc-hms_ec-hms-scimage
cd hpc-hms_ec-hms-scimage

export WORKAREA=/home/ubuntu/work/debian_build

# Full build (first time)
./build_all

# Later – just mac-agent stages (adjust range)
./build_all -s 76 -e 76
📌 Everyday Commands
Logs (always useful)
Bashjournalctl -u fabric-manager       -f
journalctl -u fabric-agent-host    -f
Quick status
Bashfmn-show-status
Full switch reset
Bashfmn-reset-switch -a -r
REST API smoke test (VNI)
Bashcurl -s -H "Content-Type: application/json" -X POST -d '{
  "testName": "Abdul-Test",
  "vniTestId": ["100"]
}' http://localhost:8000/fabric/vni/test | jq .
Topology template location
text/opt/cray/topology_template.json
Quick mgmt IP lookup
Bashcat /etc/hosts
LLDP packet capture
Bashtcpdump -i hsn0 -nn -s 1500 -c 10 -X ether proto 0x88cc
Manual Cray TLV set (example)
Bashlldptool set-tlv -r -V CrayNet enableTx=yes network='{
    "ip_addr": "192.168.1.10",
    "ttl": "forever"
}' -i ros0p18
🔄 Service Restart & Recovery Flows
Fabric Manager full reset + re-init
Bashsystemctl stop fabric-manager
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
Fix python path if scripts complain:
Bashexport PYTHONPATH=/opt/slingshot/lib/python3/venv/lib/python3.9/site-packages
# then prefix commands with: /opt/python39/bin/python3.9 /usr/bin/fmn-...
Fabric Agent restart
Bashsystemctl stop fabric-agent-host
rm -rf /opt/slingshot/data/slingshot/8000/lucene/
systemctl start fabric-agent-host
🛠️ Rosetta / Switch Deep Debugging
Port → internal mapping
Bashjack_to_port -x x3000c0r21j4p1     # example → j4p1: p3
swtest -c "port $((1 << 3)) info"
Cable info via Redfish
Bashcurl -k -u root:initial0 \
  https://127.0.0.1/redfish/v1/Chassis/Enclosure/NetworkAdapters/Rosetta/Oem/NetworkCables | jq
MAC hash table dump
Bashhashdump -p 0
Check if MAC learning enabled (port 19 example)
Bashdgrcsr -p 19 -n r_pf_elu_cfg_out_learn[0]    # 0xc... = enabled, 0x0... = disabled
ros0 stats quick view
Baship -s link show ros0
cat /sys/class/net/ros0/statistics/rx_{packets,dropped}
grep ros0 /proc/net/dev
🧪 SITF Local Test Quick Path
Bash# Clone side-by-side
git clone .../sitf-jenkins
git clone .../sitf-tests

make sitf

# Edit config as needed
vim tools/sitf-newton7.cfg

make clean
make local_test SYSTEM=newton7 > sitf-newton7.log 2>&1
Pre-merge comment example
text!sitf pre-merge https://github.hpe.com/hpe/hpc-lonesshot-fmn-minimal/pull/774 --branch 2.9 --email abdul-jabbar.s@hpe.com
🔧 Git Tricks I Use Often
Squash to single commit, keep same PR number
Bashgit reset --soft <main-head-commit-hash>
git commit -m "SSHOTCP-8175: Disable MAC learning per VLAN/Port/VLAN+Port"
git push --force origin HEAD
Rebase child branch onto updated parent
Bashgit fetch origin
git rebase origin/parent-branch-name
git push --force-with-lease origin your-branch-name
🐳 Docker Build Container Snippets
Persistent container for mac-agent build
Bashdocker run -dit --name mac-agent-build-2026 \
  -w /work/mac_agent --user root \
  --volume /work/...:/work:Z \
  --volume /work/.../artifacts:/artifacts:Z \
  x86-base:250425034852 bash
Run build inside
Bashdocker exec -it mac-agent-build-2026 /tools/helper-dpkg-build-sshotsw-repo mac-agent