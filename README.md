# WireGuard VPN Setup on Fedora Server (Lab Environment)

This guide covers the steps to set up a WireGuard VPN server on a Fedora Server VM, configured with custom settings like disabling `SELinux`, using `iptables`, and adding essential network and monitoring tools. iptables and the disabling of selinux is for my lab testing, this is a fully working setup but recommended to use SELinux with firewalld instead. Be sure to change DNS server for client config

---

### System Preparation - Server Side!

1. **Disable SELinux**
   ```bash
   sed -i 's/enforcing/disabled/g' /etc/selinux/config

2. **Update, Remove firewalld and Install iptables**
    ```bash
   dnf -y update
   dnf -y remove firewalld
   dnf install -y iptables iptables-services

3. **Install Wireguard and Additional Tools**
    ```bash
    dnf install -y rsyslog htop make gcc libtool libyaml-devel openssl-devel wget mlocate tcpdump ethtool psmisc vim net-tools bind-utils nmap tar telnet wireguard-tools
    reboot

4. **Generate WireGuard Keys**
   ```bash
   cd /etc/wireguard
   umask 077
   wg genkey | tee server_privatekey | wg pubkey > server_publickey
   wg genkey | tee client_privatekey | wg pubkey > client_publickey

5. **Configure the WireGuard Server: wg0.conf**
   ```bash
   [Interface]
   Address = 10.0.0.1/24
   ListenPort = 51820
   PrivateKey = <contents of server_privatekey>

   [Peer]
   PublicKey = <contents of client_publickey>
   AllowedIPs = 10.0.0.2/32
   
6. **Client config file**
   
   On the client device or server or whatever that has cli, create the following WireGuard configuration (e.g., wg0-client.conf):
   ```bash
   [Interface]
   PrivateKey = <contents of client_privatekey>
   Address = 10.0.0.2/24
   DNS = 10.24.32.1

   [Peer]
   PublicKey = <contents of server_publickey>
   Endpoint = yourdomain.com:51820
   AllowedIPs = 0.0.0.0/0, ::/0
   PersistentKeepalive = 25

7. **Edit sysctl.conf file Server Side**
   
   Edit sysctl.conf and add the Following for Forwarding and Additional Network Tuning for Higher Performance if Supported
   ```bash
   # Prime directive
   net.ipv4.ip_forward = 1
   # Increase the maximum amount of memory for socket buffers
   net.core.rmem_max = 16777216
   net.core.wmem_max = 16777216
   net.core.rmem_default = 262144
   net.core.wmem_default = 262144

   # Increase the maximum number of packets in the backlog
   net.core.netdev_max_backlog = 50000

   # Enable TCP window scaling (needed for high-speed links)
   net.ipv4.tcp_window_scaling = 1

   # Increase maximum buffer space for TCP
   net.ipv4.tcp_rmem = 4096 87380 16777216
   net.ipv4.tcp_wmem = 4096 65536 16777216

   # Reduce TCP SYN retries (faster failover in case of connection issues)
   net.ipv4.tcp_syn_retries = 2
   net.ipv4.tcp_synack_retries = 2

   # Enable BBR congestion control for better high-speed performance
   net.core.default_qdisc = fq
   net.ipv4.tcp_congestion_control = bbr

   # Increase allowed local port range
   net.ipv4.ip_local_port_range = 1024 65535

   # Reduce TIME_WAIT for faster socket recycling
   net.ipv4.tcp_fin_timeout = 15
   
   # Increase the maximum number of file descriptors to support high network load
   # and allow more simultaneous connections and open files.
   fs.file-max = 100000

8. **Apply the changes:**
   ```bash
   sudo sysctl -p
   echo '* soft nofile 51200
   * hard nofile 51200'>> /etc/security/limits.conf

9. **Enable NAT with iptables:**
   
    Flush existing rules and set up NAT on the public interface:
    ```bash
    iptables -F
    iptables -t nat -A POSTROUTING -o enp1s0 -j MASQUERADE
    service iptables save

10. **Enable and Start WireGuard**
    ```bash
    systemctl enable wg-quick@wg0
    systemctl start wg-quick@wg0
    systemctl status wg-quick@wg0
