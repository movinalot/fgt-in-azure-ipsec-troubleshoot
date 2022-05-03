# IPSEC VPN with FGT in Azure - FAQ

## Main objectives of this document
* Explain how to configure Azure components to establish a ipsec vpn tunnel between the FortiGates in Azure and an on-prem device
* List some troubleshooting options


## FortiGate Active/Passive with Azure ELB and ILB
_https://github.com/fortinet/azure-templates/tree/main/FortiGate/Active-Passive-ELB-ILB_

![ipsec](images/ap-elb-ilb.png)

* To enable IPSEC, you need to create Load Balancing Rules for UDP 500 and UDP 4500 as explained in this [link](https://github.com/fortinet/azure-templates/blob/main/FortiGate/Active-Passive-ELB-ILB/doc/config-inbound-connections.md#configuration---ipsec)

* When **Floating IP** is enabled, Azure LB Azure changes the IP address mapping to the Frontend IP address of the Load Balancer  instead of backend instance's IP, as explained [here](https://docs.microsoft.com/en-us/azure/load-balancer/load-balancer-floating-ip)

    ![floating](images/floating.png)

Do NOT enabled **Floating IP** on the IPSEC VPN LB rules (UDP 500 and UDP 4500), since the FortiGates are not aware and are not configured to listen on the public ip for incoming VPN connections.

![floating](images/floating-disabled-udp500.png)
![floating](images/floating-disabled-udp4500.png)

* Make sure that NAT Traversal is set to **Enable** or  **Forced** on both the FortiGate in Azure and on the remote peer

    ![natt](images/natt.png)

* If the tunnel is failing with the error message **received notify type AUTHENTICATION_FAILED** it is likely that the local-id set by the FortiGate does not match the local id expected by the peer.  To resolve this issue you can configure the FortiGate to set a specific localid-type and value to match the value exptected by the peer.
    The example below shows local-id type set to address and the value set to the public ip.
    The localid-type and value can be set to fqdn, a string or auto, for more information please consult the CLI guide [here](https://docs.fortinet.com/document/fortigate/7.2.0/cli-reference/370620/config-vpn-ipsec-phase1-interface)

    ![localid](images/localid.png)

## FortiGate Active/Active with Azure ELB and ILB
_https://github.com/fortinet/azure-templates/tree/main/FortiGate/Active-Active-ELB-ILB_

![ipsec](images/aa-elb-ilb.png)


* The setup of Active/Active FortiGates in Azure (link above) requires that each FortiGate is accessible separately to your remote peer to establish an IPSEC VPN tunnel. To expose each FortiGate separately through Azure Public Load Balancer, please create **2 Inbound NAT rules per FGT**  as shown in the screenshot below. **Each FortiGate must have its own frontend ip address so you should have at least 2 public ip attached to the Load Balancer**

    ![a-a](images/ipsec-a-a.png)



## FortiGate Active/Passive LB Sandwich with Azure Internal LB only


## Troubleshooting