# HPE Slingshot Engineering Notes (Verbatim)

⚠️ IMPORTANT  
This file intentionally preserves content **exactly as written**.  
Do NOT reformat, auto-indent, or “prettify”.

---

Testing:
---------
bash-4.4# cat lagProperties.json
{
    "lagPropertyMap": {
        "30": {
            "portLinks": [
                    "/fabric/ports/x0c0r3j11p0"
            ],
            "dmacs" : [
                "b2:00:00:00:00:0c",
                "b2:00:00:00:00:0d",
                "aa:00:00:00:00:00/ff:ff:ff:00:00:00"

            ],
            "lagFeatureMode": "STATIC",
            "lagPortSelection": ["FLOW_LABEL","VLAN", "DMAC","SMAC","DIP","SIP","IP_PROTOCOL","TCP_OR_UDP_SRC_DST_PORTS"]
        }
    }
}

Command to configure:
---------------------
fmctl update topology-policies/template-policy -f lag.json | jq


Absolute FDB will be programmed in Hash -> dgrhashdump -p 0 (To get all MAC entries from Hash)
                                        -> dgrhashdump -p 0 -v <index-of-mac>

Masked FDB will be programmed in TCAM -> dgrtcam -p 0 (To get MAC entries from TCAM)
                                      -> dgrtcam -p 0 -x <index-of-mac> -v


     */
    public enum LAGPortSelection {
        "FLOW_LABEL",
        "VLAN",
        "DMAC",
        "SMAC",
        "DIP",
        "SIP",
        "IP_PROTOCOL",
        "TCP_OR_UDP_SRC_DST_PORTS",
        COUNT
    }

HPE Slingshot Notes:
--------------------

Wiki
-----
    FM work flow
        -> https: //rndwiki-pro.its.hpecorp.net/display/SASSHOT/Fabric+Manager+Services+Startup+Sequence

Slinghshot common services framework:
	https://rndwiki-pro.its.hpecorp.net/display/SASSHOT/Slingshot+Common+Services+Framework+Architecture


Build
——
First -> ./runBuildPrep.sh
Then -> mvn clean -T 1C package -DskipTests #(First time only)
Next time onwards -> mvn -o -T 1C package -DskipTests


Commands:
---------

Build the mac agent branch (Typically all its dependent as well)
————————————————————————————————
1. Pull the parent branch https://github.hpe.com/hpe/hpc-hms_ec-hms-scimage
2.  export WORKAREA=/home/ubuntu/work/debian_build
3.  cd hpc-hms_ec-hs-scimage
4. ./build_all (First time)
5. For subsequent build of mac_agent branch alone (./build_all -s 76 -e 76)

Curl command to post the REST payload
-------------------------------------
    curl -H "Content-Type: application/json" -X POST -d '{
    "testName": "Jabbar",
    "vniTestId": [
        "100"
    ]
}' http: //localhost:8000/fabric/vni/test | jq

Fabric manager show status
--------------------------
fmn-show-status

Print logs in FM and FA
------------------------
journalctl -u fabric-agent-host -f
journalctl -u fabric-manager -f

Topology json file:
-------------------
/opt/cray/topology_template.json

Recover RBT from bad state:
———————————————
https://rndwiki-pro.its.hpecorp.net/display/SASSHOT/How+to+load+FMN+and+switch+firmware+manually+on+SLES+and+RHEL

[... CONTENT CONTINUES EXACTLY AS PROVIDED ...]

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

END OF VERBATIM NOTES

