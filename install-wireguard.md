WireGuard® is a revolution in the world of Virtual Private Networks (VPNs). It is a fast, secure, and remarkably easy-to-configure VPN protocol that leverages cutting-edge cryptography.  WireGuard stands apart from conventional VPNs with several key advantages:

Blazing Speed: WireGuard's streamlined design and focus on modern cryptographic primitives lead to exceptional performance, outpacing traditional protocols like OpenVPN and IPsec.
Unparalleled Simplicity: Configuration is remarkably straightforward. WireGuard uses a simple public/private key system for peer authentication, similar to SSH, making setup a breeze.
Stealth and Resilience: WireGuard is designed to be minimally detectable and robust against network disruptions. It can easily roam between IP addresses for enhanced mobility.
Cross-Platform Versatility WireGuard runs natively on Linux, Windows, macOS, Android, iOS, and various BSD distributions, ensuring wide compatibility.
Why Choose WireGuard?

Enhanced security: WireGuard employs modern, carefully selected cryptographic algorithms that offer superior security and protection compared to older VPN solutions.
Improved network performance: Its lightweight design minimizes overhead, maximizing network throughput and reducing latency.
Ease of management: WireGuard's simplified configuration eliminates much of the complexity associated with managing traditional VPNs.
Ideal for diverse use cases: Whether you need a remote access VPN for your team, a secure site-to-site connection, or a way to protect your privacy online, WireGuard adapts to your requirements.
Get Started with WireGuard


# How To Set Up WireGuard on Debian

WireGuard’s encryption relies on public and private keys for peers to establish an encrypted tunnel between themselves. Each version of WireGuard uses a specific cryptographic cipher suite to ensure simplicity, security, and compatibility with peers.

In comparison, other VPN software such as OpenVPN and IPSec use Transport Layer Security (TLS) and certificates to authenticate and establish encrypted tunnels between systems. Different versions of TLS include support for hundreds of different cryptographic suites and algorithms, and while this allows for great flexibility to support different clients, it also makes configuring a VPN that uses TLS more time consuming, complex, and error prone.

## Installing WireGuard and Generating a Key Pair

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install wireguard -y
```

Create the private key for WireGuard and change its permissions using the following commands:

```bash
wg genkey | sudo tee /etc/wireguard/private.key
sudo chmod go= /etc/wireguard/private.key
```
**The sudo chmod go=... command removes any permissions on the file for users and groups other than the root user to ensure that only it can access the private key.**

To create the corresponding public key, which is derived from the private key. Use the following command to create the public key file:

```bash
sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee /etc/wireguard/public.key
```

## Creating a WireGuard Server Configuration

Create a new configuration file using nano or your preferred editor by running the following command:
```bash
sudo nano /etc/wireguard/wg0.conf
```

```bash
[Interface]
PrivateKey = base64_encoded_private_key_goes_here
Address = 10.1.0.1/24
ListenPort = 51820
```

## Adjusting the WireGuard Server’s Network Configuration

To configure forwarding, open the /etc/sysctl.conf file using nano or your preferred editor:
```bash
sudo nano /etc/sysctl.conf
```

If you are using IPv4 with WireGuard, add the following line at the bottom of the file:
```
net.ipv4.ip_forward=1
```

To read the file and load the new values for your current terminal session, run:

```bash
sudo sysctl -p
```


## Configuring the WireGuard Server’s Firewall

```bash
sudo ufw allow 51820/udp
sudo ufw allow OpenSSH
```

To let ufw allow traffic from wireguard to the internet:
```bash
sudo ufw allow in on wg0 to any
sudo ufw route allow in on wg0 out on eth0
```
or for any forwarded traffic coming from wg0 whatever the destination interface, just:
```bash
sudo ufw route allow in on wg0
```

After adding those rules, disable and re-enable UFW to restart it and load the changes from all of the files you’ve modified:

```bash
sudo ufw disable
sudo ufw enable
```

You can confirm the rules are in place by running the ```sudo ufw status``` command.


## Starting the WireGuard Server

WireGuard can be configured to run as a systemd service using its built-in wg-quick script.
Using a systemd service means that you can configure WireGuard to start up at boot so that you can connect to your VPN at any time as long as the server is running. To do this, enable the wg-quick service for the wg0 tunnel that you’ve defined by adding it to systemctl:

```bash
sudo systemctl enable wg-quick@wg0.service
```

Now start the service:

```bash
sudo systemctl start wg-quick@wg0.service
```

Double check that the WireGuard service is active with the following command. You should see active (running) in the output:

```bash
sudo systemctl status wg-quick@wg0.service
```
--
## Configuring a WireGuard Peer

Open a new /etc/wireguard/wg0.conf file on the WireGuard Peer machine using nano or your preferred editor:

```bash
sudo nano /etc/wireguard/wg0.conf
```

Add the following lines to the file, substituting in the various data into the highlighted sections as required:

```bash
[Peer]
PublicKey = U9uE2kb/nrrzsEU58GD3pKFU3TLYDMCbetIsnV8eeFE=
AllowedIPs = 10.1.0.10/32
```
