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

3. **Install Additional Tools**
    ```bash
    dnf install -y rsyslog htop make gcc libtool libyaml-devel openssl-devel wget mlocate tcpdump ethtool psmisc vim net-tools iptables iptables-services bind-utils nmap tar telnet
