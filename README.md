# Setting Up a Firewall Using iptables

## Overview
This is a simple guide I put together to help you set up a firewall using `iptables` on Ubuntu or Debian. The goal is to keep things secure by blocking unwanted traffic while allowing essential connections.

---

## Step 1: Install Necessary Packages

First, update your system and install the required tools:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install iptables iptables-persistent python3 python3-pip -y
```

---

## Step 2: Set Up iptables Rules

### Clear Existing Rules
Before adding new rules, it's good practice to remove any existing ones:

```bash
sudo iptables -F
sudo iptables -X
sudo iptables -t nat -F
sudo iptables -t mangle -F
```

### Default Policies (Block All by Default)

```bash
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT
```
### Checking of the above commands are working by pinging the ip address of virtual machine from my Windows machine.

![image](https://github.com/user-attachments/assets/d9a625a2-98af-4322-ba12-31706b59b05d)

#### Hence these commands are blocking the packets sent from Windows system.


### Allow Important Connections

#### Allow SSH (Port 22)
```bash
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

#### Allow Web Traffic (HTTP & HTTPS)
```bash
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
```

#### Allow Established Connections
```bash
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```

#### Allow Ping (ICMP)
```bash
sudo iptables -A INPUT -p icmp -j ACCEPT
```

#### Drop Invalid Packets
```bash
sudo iptables -A INPUT -m conntrack --ctstate INVALID -j DROP
```

#### Mitigate SYN Flood Attacks
```bash
sudo iptables -A INPUT -p tcp --syn -m limit --limit 1/s --limit-burst 3 -j ACCEPT
```

---

## Step 3: Save & Enable Firewall Rules

### Save the Rules
```bash
sudo iptables-save > /etc/iptables/rules.v4
```

####You may encounter the following error message:

![image](https://github.com/user-attachments/assets/e7717249-e3b7-4045-8476-cd911b1b93cf)

#### IF So, Instead use this command : 
```bash
sudo iptables-save | sudo tee /etc/iptables/rules.v4 > /dev/null
```
####The command saves the current iptables rules and writes them to /etc/iptables/rules.v4 using tee with sudo to avoid permission issues, while suppressing terminal output with > /dev/null.

### Enable Firewall on Boot
```bash
sudo systemctl enable netfilter-persistent
sudo systemctl start netfilter-persistent
```

---

## Step 4: Check Firewall Status

Run this command to see if your rules are applied correctly:

```bash
sudo iptables -L -v -n
```

### What to Expect
- If rules appear under `INPUT`, `FORWARD`, and `OUTPUT`, everything is working.
- If the output is empty, the firewall isn’t active.

**(Insert your screenshot of the command output here)**

---

## Additional Tips
- Feel free to tweak these rules to suit your needs.
- If you need to remove a rule, use `sudo iptables -D`.
- To completely reset the firewall, run `sudo iptables -F` (use with caution)

---

## Useful Resources
- [iptables Documentation](https://www.netfilter.org/projects/iptables/)
- [Ubuntu Firewall Guide](https://ubuntu.com/server/docs/security-firewall)

