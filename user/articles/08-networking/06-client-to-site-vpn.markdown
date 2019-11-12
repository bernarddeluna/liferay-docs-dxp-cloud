# Client-to-Site VPN



## Overview

DXP Cloud basic setup provides a <u>VPN client-to-site connection with port forwarding support and redundant tunnels support.</u> This feature is commonly used to connect an environment from the cloud, such as the customer production environment, to the customer company internal network. For security and reliability, VPN connections are isolated per environment: production, staging, development.

![Topology 1 - DXP Cloud VPN client-to-site topology](../../images/cliente-site-vpn-01.png)

The VPN feature supports the following protocols:

1. 1.IPsec (IKEv1, IKEv2, L2TP)
2. 2.OpenVPN

In order to increase resiliency, it’s possible to enable redundant VPN tunnels by configuring a connection to multiple destination VPN server IPs provided by the customer. The redundancy is placed in different availability zones to provide more resiliency.  At the moment, DXP Cloud does not support Site-to-Site VPN connection. The most common use-case, such as connecting to a service running on the company network, are covered by the client-to-site approach, this model is recommended for the containerized architecture and Kubernetes network layer provided.

## Configuration

The customer can choose one of the protocols (IPSec or OpenVPN) to perform the connection from DXP Cloud Console settings page for the desired environment. Port forwarding rules are, conveniently, mapped from the user interface.

![Image 1 - DXP Cloud VPN settings for the customer environment](../../images/cliente-site-vpn-02.png)

The VPN service will create the routes to each forwarding host and port provided at the Settings page in the Forwarding Port section and provide access to the mapped host from inside the cloud environment. For both, IPSec or OpenVPN, the required configurations are the same.

**Information required to establish a connection**

To successfully establish a connection between DXP Cloud and the customer company network, the customer must have a public IPSec or OpenVPN server and provide the following information (Table 1).

 

| **Server Address**  | `<SERVER IP>`:`<SERVER PORT>`                                |
| ------------------- | ------------------------------------------------------------ |
| **Server Username** | `<LOGIN>`                                                    |
| **Server Password** | `<PASSWORD>`                                                 |
| **Certificate**     | `<CA CERTIFICATE>`                                           |
| **Forwarding Port** | `<APPLICATION IP>`<br />`<APPLICATION PORT>`<br />`<LOCAL PORT>` |

_Table 1 - Customer VPN server information and credentials_

  

**_Glossary_**

_&lt;SERVER ADDRESS&gt; The IP where the VPN server is provided._

_&lt;SERVER PORT&gt; The PORT where the VPN server is provided._

_&lt;LOGIN&gt; The USERNAME for this VPN server._

_&lt;PASSWORD&gt; The PASSWORD for this VPN server._

_&lt;CA CERTIFICATE&gt; The CA CERTIFICATE for this connection VPN connection._

_&lt;APPLICATION IP&gt; The IP for the internal host which you what to access._

_&lt;APPLICATION PORT&gt; The PORT for the internal host which you what to access._

_&lt;LOCAL PORT&gt; The desired PORT to expose the mapped service, port range from 0 to 65535._



<u>The customer must send a configuration as the following</u> (See Table 2):

 

| **Server Address**  | 18.188.145.101:500                                         |
| ------------------- | ---------------------------------------------------------- |
| **Server Username** | user.liferay                                               |
| **Server Password** | password                                                   |
| **Certificate**     | -----BEGIN CERTIFICATE-----<br />-----END CERTIFICATE----- |
| **Forwarding Port** | 192.168.100.30<br />8080<br />33000                        |

_Table 2 - Sample customer VPN server information and credentials_



## Example

**Connecting DXP Cloud to IPSec VPN Server**

A common use-case for this feature is a DXP Portal instance running inside DXP Cloud accessing an HTTP service running inside the customer company network.

![Topology 2 - Portal instance accessing an HTTP service inside the customer’s company network](../../images/cliente-site-vpn-03.png)

In this example, the Hello World service **192.168.100.30:8080** running inside the customer company network is accessible from DXP Portal service by the address **vpn:33000**. This is possible because of the client-to-server connection was established to the customer company VPN server **18.188.145.101:500** and a port forwarding rule was exposed into the local port **33000 **mapping to the application **192.168.100.30:8080**.

After the connection and port forwarding rule is setup, requests to the Hello World service can be made from any DXP Cloud service, such as the DXP Portal.

 

curl vpn:33000_

&lt;body&gt;&lt;h1&gt;Hello world!&lt;/h1&gt;&lt;/body&gt;&lt;/html>



**DXP Cloud IP Ranges for Shared Cluster**

<u>By default outgoing external IP for your DXP Cloud services are ephemeral</u>. The best way to get a stable outgoing external IP for your DXP Cloud services would be to contract the DXP Cloud Private Cluster feature.

**DXP Cloud IP Ranges for Private Cluster**

The DXP Cloud Private Cluster feature isolates the customer into their own dedicated cluster, this isolation brings extra capabilities for the customer environment. With this package, <u>the cluster is configured with a dedicated gateway for all outbound Internet traffic from the customer cluster, and a static external IP is assigned.</u>



## Appendix

**Basic setup for the IPsec server**

This is a basic sample setup configuration used on our IPsec test server.

 

**/etc/ipsec.conf**

config setup

    charondebug="ike 1, knl 1, cfg 0"
    
    uniqueids=no

conn ikev2-vpn

    auto=add
    
    compress=no
    
    type=tunnel
    
    keyexchange=ikev2
    
    fragmentation=yes
    
    forceencaps=yes
    
    dpdaction=clear
    
    dpddelay=300s
    
    rekey=no
    
    left=%any
    
    leftid=18.188.145.101
    
    leftcert=server-cert.pem
    
    leftsendcert=always
    
    leftsubnet=0.0.0.0/0
    
    right=%any
    
    rightid=%any
    
    rightauth=eap-mschapv2
    
    rightsourceip=10.10.10.0/24
    
    rightdns=8.8.8.8,8.8.4.4
    
    rightsendcert=never
    
    eap_identity=%identity
