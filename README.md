steps to follow:

1. need to have the pyang in the ubuntu
2. neet to enable the NETCONF in the cisco router
show running-config | include netconf

3. need to know what to do:
show ip interface brief
so Idea here to provide an IP address 10.10.2.1 in the interface : GigabitEthernet2


CREATION of YANG file:

So, we will create a yaml model with the requirment which is [A]

then need to check the file by 


pyang cisco-interface-config.yang 

and should not have any error

also need to check the tree:
pyang -f tree cisco-interface-config.yang





4. now Based on the YANG model file as we created(cisco-interface-config.yang),

 we will need to have the NETCONF XML payload to configure the interface:
 
CREATION of NETCONF XML PAYLOAD: [here NETCONF is the protocol, where the UBUNTU and Cisco router will communnicate, and will execute the yang file, /content of the yang file]
 
here is the NETCONF XML payload [B] to configure GigabitEthernet2 with IP 10.10.2.1/24 



5.
Apply Configuration Using Python (NCCLIENT) module: [C]


The ncclient Python module is used to interact with network devices/CISCO using the NETCONF protocol[where used yang data model to define what to do in side the .yang file]. 
NCCLIENT allows automation of network configurations, retrieving operational data, and managing devices remotely in a structured way.


need to install ncclient if required.

pip install ncclient

6. execute now:
file name:
apply_config.py

7.
show ip interface brief



+++++++++++


A.

name of the file: cisco-interface-config.yang 
content of the file:



module cisco-interface-config {
  namespace "urn:cisco:yang:cisco-interface-config";
  prefix "iface";

  import ietf-inet-types {
    prefix inet;
  }

  organization "Cisco Systems";
  contact "support@cisco.com";
  description "YANG model to configure IP addresses on Cisco router interfaces";

  revision "2025-02-21" {
    description "Initial revision";
  }

  container interfaces {
    description "Interface configurations";

    list interface {
      key "name";
      description "List of router interfaces";

      leaf name {
        type string;
        description "Interface name (e.g., GigabitEthernet2)";
      }

      leaf ip-address {
        type inet:ipv4-address;
        description "IPv4 address assigned to the interface";
      }

      leaf subnet-mask {
        type inet:ipv4-address;
        description "Subnet mask of the interface";
        default "255.255.255.0";
      }

      leaf admin-status {
        type enumeration {
          enum up;
          enum down;
        }
        description "Administrative status of the interface";
      }
    }
  }
}



B.

<rpc xmlns="urn:ietf:params:xml:ns:netconf:base:1.0" message-id="1">
  <edit-config>
    <target>
      <running/>
    </target>
    <config>
      <interfaces xmlns="urn:cisco:yang:cisco-interface-config">
        <interface>
          <name>GigabitEthernet2</name>
          <ip-address>10.10.2.1</ip-address>
          <subnet-mask>255.255.255.0</subnet-mask>
          <admin-status>up</admin-status>
        </interface>
      </interfaces>
    </config>
  </edit-config>
</rpc>



[C] file name apply_config.py


from ncclient import manager

# Router connection details
router = {
    "host": "192.168.96.133",  # Router IP
    "port": 830,  # NETCONF Port
    "username": "admin",
    "password": "Hellog@123",
    "hostkey_verify": False
}

# Corrected NETCONF XML payload using standard YANG model
config_payload = """
<config>
  <interfaces xmlns="urn:ietf:params:xml:ns:yang:ietf-interfaces">
    <interface>
      <name>GigabitEthernet2</name>
      <description>Configured via NETCONF</description>
      <enabled>true</enabled>
      <ipv4 xmlns="urn:ietf:params:xml:ns:yang:ietf-ip">
        <address>
          <ip>10.10.2.1</ip>
          <netmask>255.255.255.0</netmask>
        </address>
      </ipv4>
    </interface>
  </interfaces>
</config>
"""

# Connect to the router and apply the configuration
try:
    with manager.connect(**router) as m:
        response = m.edit_config(target="running", config=config_payload)
        print("[+] Configuration applied successfully!")
        print(response)
except Exception as e:
    print(f"[!] Error: {e}")
