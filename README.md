# Static-VTI-Point-to-Point-Tunnels

The use of IPsec VTIs simplifies the configuration process when you must provide protection for site-to-site VPN tunnels and offers a simpler alternative to the use of Generic Routing Encapsulation (GRE) or Layer 2 Tunneling Protocol (L2TP) tunnels for encapsulation and crypto maps with IPsec. A major benefit of IPsec VTIs is that the configuration does not require a static mapping of IPsec sessions to a physical interface. The IPsec tunnel endpoint is associated with a virtual interface. Because there is a routable interface at the tunnel endpoint, you can apply many common interface capabilities to the IPsec tunnel.

The following are characteristics of the simplest form of Cisco IOS Software tunnel-based site-to-site IPsec VPN configuration:

1. It replaces cryptographic map-based configuration.
2. It is intuitive to configure and integrates better with other Cisco IOS Software features.

note : It is not recommended to use the same IP for the tunnel interface as used on the physical interface as it can lead to recursive routing or a flapping tunnel when dynamic routing protocols are enabled.

## VTI features include the following:

1. They behave as regular tunnels, one for each remote site of the VPN.
2. Their encapsulation must be either IPsec Encapsulating Security Payload (ESP) or Authentication Header (AH).
3. Their line protocol depends on the state of the VPN tunnel (IPsec Security Associations [SAs]).

## An IPsec VTI has several benefits:

1. Simplifies configuration: Customers can use the virtual tunnel constructs to configure an IPsec peering, thus simplifying the complexity of the VPN configuration as compared to crypto maps or GRE IPsec tunnels.
2. Flexible interface feature support: An IPsec VTI is an encapsulation that uses its own Cisco IOS Software interface. This characteristic offers the flexibility of defining features to run on either the physical interface (that operates on ciphertext traffic) or on the IPsec VTI (that operates on cleartext traffic).
3. Multicast support: Customers can use the IPsec VTIs to securely transfer multicast traffic such as voice and video applications from one site to another.
4. Improved scalability: IPsec VTIs need fewer established SAs to cover different types of traffic, both unicast and multicast, thus enabling improved scaling.
5. Provides a routable interface: Like GRE IPsec, IPsec VTIs can natively support all types of IP routing protocols, which provide scalability and redundancy.

## The IPsec VTI has the following limitations:

1. The IPsec VTI is limited to only IP unicast and multicast traffic, as opposed to GRE tunnels, which have a wider, multiprotocol application.
2. Cisco IOS Software IPsec stateful failover is not supported with IPsec VTIs. However, you can use alternative failover methods such as dynamic routing protocols to achieve similar functionality.

## Deployment choices

Deployment Choice

Use static or dynamic VTI tunnels? 
Use static or dynamic routing protocol over VTI tunnels? 

Criteria

Use dynamic VTI tunnels for the hub in large hub-and-spoke networks. Otherwise, use static VTI tunnels.
Use a dynamic routing protocol in large networks and to provide path or peer redundancy with multiple VTI tunnels. Otherwise, use static routing over VTI tunnels.

Consider the following guidelines when deploying VTI-based site-to-site IPsec VPNs:

1. Use VTI-based site-to-site VPNs as the default IPsec technology for individual point-to-point VPN links and for hub-and-spoke VPNs.
2. Consider deploying DMVPN or Group Encrypted Transport (GET VPN) for larger environments with partial or fully meshed VPN requirements.

## Understanding Basic IKE Peering

Configuring basic IKE peering using PSKs includes the following tasks:

1. Set up an IKE SA between two peers.
    - Use PSKs for mutual authentication.
    - Use an encryption and hashing algorithm to guarantee confidentiality and integrity of the key management session.
    - Use a DH exchange of an appropriate strength (group) to provide keying material to IKE and IPsec.
    - Use appropriate session lifetimes.
2. Create a PSK and bind it to the name or IP address of the VPN peer.

Configuring IKE peering between VPN members is the first step when configuring VTI-based IPsec VPNs. You should determine the IKE (ISAKMP) phase 1 policy requirements that you want to use, and then configure those policies on all peers. Having a detailed IKE policy plan lessens the chances of improper configuration.

Here are some planning steps:
    - Determine the peer authentication method: Choose the peer authentication method based on the credentials that are provisioned to all peers. Cisco IOS Software supports either PSKs, Rivest–Shamir–Adleman (RSA)-encrypted nonces, or RSA signatures to authenticate IPsec peers. This topic focuses on using PSKs.
    - Determine the session protection policy for the IKE session: Specify an encryption and hashing algorithm that will be used to protect IKE packets.
    - Determine the strength of the session key exchange method: IPsec uses the Diffie-Hellman (DH) algorithm to exchange session keys. The length of DH keys is determined by the DH group, which is part of an IKE policy.
    - Use an appropriate IKE session lifetime: Typically, the default Cisco IOS Software IKE session lifetime is appropriate for most use scenarios.
The goal of this planning step is to gather the precise data that you will need in later steps to minimize misconfiguration.

Cisco IOS Software Release 12.4(20)T introduced default IKE policies. There are eight default IKE policies that are supported with protection suites of priorities 65507–65514, where 65507 is the highest priority and 65514 is the lowest priority policy. The highest priority (65508) PSK-based default policy in the figure provides the highest protection of the default policy options. It uses AES as the encryption algorithm, SHA as the hash algorithm and DH group 5.

Avoid policies that use MD5 as the hash algorithm, because SHA-1 is stronger. Newer, stronger hash algorithms are now available including SHA-2 (SHA-256, SHA-384 and SHA-512) and are preferred for better security in the future. Avoid policies that use DH group 2, because DH group 5 is stronger and more suitable as the default method strength. Cisco IOS also supports the stronger DH groups 14,15,16…24, which provide even stronger key exchanges.

Default lifetimes are conservative. For systems that will not agree on the lifetimes during negotiation, change the lifetime to an acceptable value. Cisco IOS Software does not require IKE peers to have a matching IKE lifetime setting. The IKE SA will establish it based on the shorter of the two settings.

![Creating a VTI](https://raw.githubusercontent.com/deliawolf/Static-VTI-Point-to-Point-Tunnels/main/Screenshot%202023-10-31%20at%2014.41.06.png)

### Configuration example
```
crypto isakmp policy 10
 authentication pre-share
 hash sha
 encr aes 128
 group 14
 lifetime 3600
!
crypto isakmp key jg40fb90FFrhn98R3Bv9ng9fe4 address 172.17.2.24
```
When configuring IKE peering, first configure the IKE policy. Then configure the authentication credentials.

Before verifying static point-to-point VTI tunnels, troubleshooting IKE peering is recommended.

Cisco IOS Software includes the following IKE peering validation commands:
1. Use the show crypto isakmp policy command to display the parameters for each local IKE policy.
2. Use the show crypto isakmp sa command to verify the status of the IKE peering SAs. This command displays all existing IKE peering SAs.

###  Configure Static VTI Point-to-Point Tunnels 

On the R2 router, create a new IKE policy.

Specify the following parameters:

  1. Authentication method: preshared

  2. Encryption algorithm: AES 128

  3. Hash algorithm: SHA

  4. Key exchange method: 14

  5. Lifetime: 1 hour


```
R2(config)#crypto isakmp policy 10
R2(config-isakmp)#authentication pre-share  
R2(config-isakmp)#hash sha
R2(config-isakmp)#encryption aes 128
R2(config-isakmp)#group 14
R2(config-isakmp)#lifetime 3600
```
On the R2 router, create a PSK and bind it to the IP address of the R3 router (172.18.4.2).
```
R2(config)#crypto isakmp key cisco address 172.18.4.2
```
The transform set determines the encryption and data authentication hash for the IPsec SA. The transform set must match an equal policy on the peer. On the R2 router, create an IPsec transform set for user traffic protection. Use ESP with 128-bit AES as the encryption transform and use ESP with SHA-1 (HMAC variant) as the authentication transform.

```
R2(config)#crypto ipsec transform-set MYSET esp-aes 128 esp-sha-hmac
```
On the R2 router, create an IPsec profile and include transform set in the profile.
```
R2(config)#crypto ipsec profile MYPROFILE
R2(ipsec-profile)#set transform-set MYSET
```
On the R2 router, create a new tunnel interface. Configure the interface to use the IP address and subnet of the Ethernet 0/0 interface. Specify a tunnel source (Ethernet 0/0) and a tunnel destination of R3’s Ethernet 0/0 interface IP address (172.18.4.2).
```
R2(config)#interface tunnel 0
R2(config-if)#ip unnumbered ethernet 0/0
R2(config-if)#tunnel source ethernet 0/0
R2(config-if)#tunnel destination 172.18.4.2
R2(config-if)#
```
On the R2 router, specify IPsec as the tunnel encapsulation. Specify the traffic protection policy by referencing the configured IPsec profile.
```
R2(config-if)#tunnel mode ipsec ipv4
R2(config-if)#tunnel protection ipsec profile MYPROFILE
```
On the R2 router, create a static route to the R3 internal network (10.10.2.0/24) that is reachable over the tunnel.
```
R2(config)#ip route 10.10.2.0 255.255.255.0 tunnel 0
```
CONFIGURATION IS SAME ON BOTH END, THE DIFFERENCE MAYBE ONLY INTERFACE AND DESTINATION IP


