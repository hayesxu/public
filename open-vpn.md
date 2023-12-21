# Using Windows Server
### Software
OpenVPN Community release [OpenVPN-2.6.7-I001-amd64.msi](https://openvpn.net/community-downloads/)
### Installation
- Full install, customize with easyrsa option
### Current issue: 
- Need manually share network connection with openVPN TAP-windows6 driver
- openVPN TAP-windows6 driver does not always work correctly.

# Using AWS Ubuntu
## AWS
    - Need a public IP to your EC2(Elastic Ip)
    - Security group to allow incoming UDP request on your preferred port (e.g. 1194)
# Ubuntu
### [Install](https://ubuntu.com/server/docs/service-openvpn)
    apt update
    apt upgrade
    apt install tzdata
    dpkg-reconfigure tzdata
    sudo apt install openvpn easy-rsa

# Server config file:
    server.conf:
        proto udp
        port 1194
        dev tun
        server 10.8.0.0 255.255.255.0
        ca ca.crt
        cert server.crt
        key server.key  # Keep this file secret
        dh dh2048.pem
        tls-auth ta.key 0  # Shared key to enhance security
        cipher AES-256-CBC
        verb 3
        keepalive 10 120
        persist-key
        persist-tun
        duplicate-cn
        push "redirect-gateway def1"
        client-to-client
        topology subnet

    Note: 
        Below are a list of certifiate/key files required for the VPN server, they can be generated using 
        easy-rsa that come with OpenVPN Community release (Make sure you install easy-rsa via custom install):
            ca.crt
            server.crt
            server.key
            ta.key
            dh2048.pem


# Client config file:
    vpn-client.ovpn:
        client
        dev tun
        proto udp
        remote 18.205.74.241 1194
        ;18.205.74.241=AWS EC2 Public IP
        resolv-retry infinite
        nobind
        persist-key
        persist-tun
        
        <ca>
        -----BEGIN CERTIFICATE-----
        Ms......
        ......
        -----END CERTIFICATE-----
        </ca>
        
        <cert>
        -----BEGIN CERTIFICATE-----
        MIID
        .....
        -----END CERTIFICATE-----
        </cert>
        
        <key>
        -----BEGIN ENCRYPTED PRIVATE KEY-----
        .......
        -----END ENCRYPTED PRIVATE KEY-----
        </key>
        
        <tls-auth>
        -----BEGIN OpenVPN Static key V1-----
        ......
        -----END OpenVPN Static key V1-----
        </tls-auth>
        
        key-direction 1
        remote-cert-tls server
        cipher AES-256-CBC
        verb 3


# Client software
Windows: [openVPN connect client](https://openvpn.net/downloads/openvpn-connect-v3-windows.msi) 

# Enable client/server
## Server:
    Add this to cron job:
        @reboot sudo /usr/sbin/iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE; sudo /usr/sbin/openvpn --config /home/ubuntu/server.conf > /home/ubuntu/vpn.log 2>/home/ubuntu/vpn.err &
## Client:
    Send vpn-client.ovpn to whoever needs VPN and using openvpn connect to open it.