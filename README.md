# vps-security-checklist

1. Initial Server Access
   - [ ] Create a new VPS and add SSH key
   - [ ] Connect via SSH[^1]
   - [ ] Create non-root user with sudo privileges
   - [ ] Hardening SSH
     - [ ] Add SSH key to non-root user
     - [ ] Disable password authentication
     - [ ] Remove root SSH access
     - [ ] Disable PAM authentication
     - [ ] Other SSH configuration
     - [ ] Change default SSH port (optional[^2])

2. Basic Security
   - [ ] Update system packages
   - [ ] Configure firewall (UFW)
   - [ ] Install and configure fail2ban
   - [ ] Set up automatic security updates

3. Server Configuration
   - [ ] Set timezone
   - [ ] Install other utilities (htop, tmux, etc.)

4. Install Docker
   - [ ] Install Docker & Docker Compose
   - [ ] Add user to docker group
  
[^1]: If first time using shh key after generating it, add the key to an ssh-agent.
[^2]: Change default SSH port is not recommended.

## 1. Initial Server Access

### 1.1. Create a new VPS and add SSH key

### 1.2. Connect via SSH

```bash
# local
ssh root@<server_IPv4>
```

### 1.3. Create non-root user with sudo privileges

```bash
# server
adduser <username>
usermod -aG sudo <username>
```	

### 1.4 SSH hardening

#### 1.4.1 Add SSH key to non-root user
```bash
# local
ssh-copy-id -i ~/.ssh/<key_name> <username>@<server_IPv4>
```

#### 1.4.2 Disable password authentication, PAM authentication and remove root SSH access 

```bash
# server
sudo nano /etc/ssh/sshd_config

# edit ->
# PasswordAuthentication no
# UsePAM no
# PermitRootLogin no

sudo systemctl reload ssh
```

**Nota:** Para acceder de nuevo al servidor es posible que sea necesario agregar la clave SSH al agente ssh. Otra forma es acceder utilizando el nombre de usuario y la clave privada.
```bash
ssh -i ~/.ssh/<key_name> <username>@<server_IPv4>
```

#### 1.4.3 Other SSH configuration

```bash
# server
sudo nano /etc/ssh/sshd_config

# edit ->
# -- Forced disconnection after a certain inactivity (300 seconds)
# ClientAliveInterval 300
# ClientAliveCountMax 1
# -- Automatic disconnection in case of incorrect login
# MaxAuthTries 3
# -- Deactivate unused functions
# AllowTcpForwarding no                   # Disables port forwarding.
# X11Forwarding no                        # Disables remote GUI view.
# AllowAgentForwarding no                 # Disables the forwarding of the SSH login.

sudo systemctl restart ssh
```


## 2. Basic Security

### 2.1. Update system packages

```bash
# server
sudo apt update && sudo apt upgrade -y
```

### 2.2. Configure firewall (UFW)
UFW (Uncomplicated FireWall) already comes installed with Ubuntu

#### 2.2.1. Default policies

```bash
# server
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

#### 2.2.2. **VERY IMPORTANT**. Allow SSH connections

> [!CAUTION]
> If SSH is not allowed, the server will not be accessible.

```bash
# server
sudo ufw allow OpenSSH
```

#### 2.2.3. Enable UFW

```bash
# server
sudo ufw enable
sudo ufw status # check status
```

**NOTE:** Docker overrides some UFW rules and containers can still be accessible although UFW rules block that port. To avoid that use Reverse Proxy (step 5).

### 2.3. Install and configure fail2ban

```bash
# server
sudo apt install fail2ban -y
sudo systemctl enable fail2ban
```
	
#### 2.3.1. Configure fail2ban

```bash
# server
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
```

### 2.4. Set up automatic security updates

**NOTE:** Usually already installed in Ubuntu Server.

```bash
# server
sudo apt install unattended-upgrades -y
systemctl status unattended-upgrades # Check running status
```

## 3. Server Configuration

### 3.1. Set timezone

```bash
# server
sudo timedatectl set-timezone <timezone>
```

### 3.2. Install other utilities (htop, tmux, etc.)
Follow installation guides


## 4. Install Docker

### 4.1. Install Docker & Docker Compose

Follow docs: https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository

### 4.2. Add user to docker group

```bash
# server
sudo usermod -aG docker <username>
```
