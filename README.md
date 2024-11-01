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
   sudo sed -i 's/enforcing/disabled/g' /etc/selinux/config
   
sudo dnf -y remove firewalld
sudo dnf install -y iptables iptables-services
