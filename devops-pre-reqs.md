# Course: DevOps Pre-Requisites Course

## Creating a systemd service

Create a systemd unit file to manage the service.

The unit file is typically located in `/etc/systemd/system/`.

The filename is the name of the service and should have a `.service` extension, for example, `my_app.service`.

```bash
sudo touch /etc/systemd/system/my_app.service
```

```bash
# my_app.service
[Unit]
# Description of the service
Description=My Application Service

[Service]
# Execution command
ExecStart=/usr/bin/python3 /opt/code/my_app.py
# Execute BEFORE the service starts
ExecStartPre=/usr/bin/echo "Starting My Application"
# Execute AFTER the service stops
ExecStopPost=/usr/bin/echo "My Application has stopped"
# Restart the service if it crashes
Restart=always

[Install]
# Define when the service should start, in this case, when the system boots
WantedBy=multi-user.target
```

After creating a new unit file, reload the systemd manager configuration:

```bash
sudo systemctl daemon-reload
```

Then, it's possible to see the status, start, stop, enable, or disable the service:

```bash
sudo systemctl status my_app
sudo systemctl start my_app
sudo systemctl stop my_app
sudo systemctl enable my_app
sudo systemctl disable my_app
```

Example of `httpd.service` unit file:

```bash
# /etc/systemd/system/httpd.service

[Unit]
Description=The Apache HTTP Server
Wants=httpd-init.service
After=network.target remote-fs.target nss-lookup.target httpd-init.service
Documentation=man:httpd.service(8)

[Service]
Type=notify
Environment=LANG=C

ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND
ExecReload=/usr/sbin/httpd $OPTIONS -k graceful
# Send SIGWINCH for graceful stop
KillSignal=SIGWINCH
KillMode=mixed
PrivateTmp=true
OOMPolicy=continue

[Install]
WantedBy=multi-user.target
```

## Networking Basics

### Network Configuration

```bash
# Check network interfaces
ip link

# Check IP addresses
ip addr

# Assign an IP address to an interface
sudo ip addr add <IP_ADDRESS>/<SUBNET_MASK> dev <INTERFACE_NAME>
# Example: Assigning IP address 192.168.1.100/24 to eth0
sudo ip addr add 192.168.1.100/24 dev eth0

# Check routing table
ip route

# Add a static route
sudo ip route add <DESTINATION_NETWORK>/<SUBNET_MASK> via <GATEWAY_IP>
# Example: Adding a route to 10.0.0.0/24 via 192.168.1.1
sudo ip route add 10.0.0.0/24 via 192.168.1.1

# Add a specific route to a host
sudo ip route add <HOST_IP> via <GATEWAY_IP>
# Example: Adding a route to 10.0.0.5 via 192.168.1.1
sudo ip route add 10.0.0.5 via 192.168.1.1

# Add a default gateway
sudo ip route add default via <GATEWAY_IP>

# Check network connectivity
ping google.com

# Check open ports
ss -tulnp
```

### DNS

```bash
# Hosts file (by default, it has precedence over DNS)
cat /etc/hosts

# Add a new entry to the hosts file
echo "<IP_ADDRESS> <HOSTNAME>" | sudo tee -a /etc/hosts
# Example: Adding an entry for a local server
echo "192.168.1.100 myserver" | sudo tee -a /etc/hosts

# The precedence order can be changed in the `nsswitch.conf` file
cat /etc/nsswitch.conf | grep hosts
# Example output: `hosts:   files dns`

# Check DNS resolution
cat /etc/resolv.conf

# Add a DNS server
echo "nameserver <DNS_SERVER_IP>" | sudo tee -a /etc/resolv.conf
# Example: Adding Google's public DNS server
echo "nameserver 8.8.8.8" | sudo tee -a /etc/resolv.conf

# Search for a domain
echo "search <DOMAIN_NAME>" | sudo tee -a /etc/resolv.conf
# Example: Adding a search domain
echo "search griaule.com" | sudo tee -a /etc/resolv.conf
# Then, when pinging a host, it will append the search domain
# For example, `ping support` will resolve to `support.griaule.com`

# Record types
# A: Address record
# AAAA: IPv6 address record
# CNAME: Canonical name record (alias), e.g., `eat.example.com` to `food.example.com`
# MX: Mail exchange record
# TXT: Text record (often used for SPF, DKIM, etc.)

# Query a hostname from a DNS server
nslookup <HOSTNAME> <DNS_SERVER_IP>
dig <HOSTNAME> @<DNS_SERVER_IP>
# These tools don't consider the local hosts file, they only query the DNS server
# Example: Querying `google.com` from Google's public DNS server
nslookup google.com 8.8.8.8
dig google.com @8.8.8.8
```
