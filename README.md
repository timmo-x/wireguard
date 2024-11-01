# WireGuard VPN Setup on Fedora Server (Lab Environment)

This guide covers the steps to set up a WireGuard VPN server on a Fedora Server VM, configured with custom settings like disabling SELinux, using `iptables`, and adding essential network and monitoring tools.

## Table of Contents
- [System Preparation](#system-preparation)
- [WireGuard Installation](#wireguard-installation)
- [Generate WireGuard Keys](#generate-wireguard-keys)
- [Configure the WireGuard Server](#configure-the-wireguard-server)
- [Configure the WireGuard Client](#configure-the-wireguard-client)
- [Network Tuning for High Speed](#network-tuning-for-high-speed)
- [Enable and Start WireGuard](#enable-and-start-wireguard)
- [Troubleshooting](#troubleshooting)

---

### System Preparation

1. **Disable SELinux**
   ```bash
   sed -i 's/enforcing/disabled/g' /etc/selinux/config

2. **Remove firewalld and Install iptables**
    ```bash
   dnf -y remove firewalld
   dnf install -y iptables iptables-services

3. **Install Wireguard and Additional Tools**
    ```bash
    dnf install -y rsyslog htop make gcc libtool libyaml-devel openssl-devel wget mlocate tcpdump ethtool psmisc vim net-tools bind-utils nmap tar telnet wireguard-tools

4. **Generate WireGuard Keys**
   ```bash
   cd /etc/wireguard
   umask 077
   wg genkey | tee server_privatekey | wg pubkey > server_publickey
   wg genkey | tee client_privatekey | wg pubkey > client_publickey

5. **Configure the WireGuard Server**
   ```bash
   [Interface]
   Address = 10.0.0.1/24                     # VPN subnet for the server
   ListenPort = 51820                        # Port WireGuard listens on
   PrivateKey = <contents of server_privatekey>  # Server's private key

   # Client configuration on the server side
   [Peer]
   PublicKey = <contents of client_publickey>    # Client's public key
   AllowedIPs = 10.0.0.2/32                      # IP assigned to the client
   
6. **Ensure restricted permissions on the config file:***
   ```bash
   chmod 600 /etc/wireguard/wg0.conf
