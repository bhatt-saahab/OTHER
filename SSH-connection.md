# Easy Setup Guide: Remote CLI/SSH Connection to Windows

This guide provides step-by-step instructions for setting up an SSH server on a Windows machine to enable remote command-line interface (CLI) access from another device, such as an Android phone using Termux. Both the host (Windows) and client devices must be connected to the **same local network** (e.g., the same Wi-Fi router). SSH provides secure remote access, but ensure you are operating in a trusted environment.

## Prerequisites
- **Windows Version**: Windows 10 (build 1809 or later) or Windows 11.
- **Administrative Privileges**: All steps on Windows require running as an Administrator.
- **Network**: Devices must share the same local network. Verify using `ipconfig` on Windows or equivalent on the client.
- **Security Note**: This setup uses default password authentication. For enhanced security, consider SSH key-based authentication (not covered in this basic guide).
- **Default Port**: SSH uses port 22. Ensure no firewalls or antivirus software block this port beyond the configured rule.

## Step 1: Configure SSH Server on Windows
1. **Open PowerShell as Administrator**:
   - Search for "PowerShell" in the Windows Start menu.
   - Right-click and select **Run as administrator**.

2. **Execute the Installation Script**:
   - Copy and paste the following PowerShell script into the terminal.
   - Press **Enter** to run it.
   - The script automatically installs OpenSSH Server (if not present), starts the service, configures the firewall, and displays the connection details.

```powershell
# PowerShell Script: Install and Configure OpenSSH Server
# This script handles installation via multiple methods, starts the service, sets auto-start, configures firewall, and outputs connection string.

# Verify Administrative Privileges
$isAdmin = ([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)
if (-not $isAdmin) {
    Write-Host "This script requires administrative privileges. Please run as Administrator."
    Read-Host "Press Enter to exit..."
    exit 1
}

# 1. Install OpenSSH Server (Multiple Attempts)
$installSuccess = $false

# Attempt 1: Add-WindowsCapability
try {
    Write-Host "Attempting installation via Add-WindowsCapability..."
    Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0 -ErrorAction Stop
    $installSuccess = $true
    Write-Host "Installation successful via Add-WindowsCapability."
} catch {
    Write-Host "Add-WindowsCapability failed: $_"
}

# Attempt 2: Chocolatey
if (-not $installSuccess) {
    try {
        Write-Host "Attempting installation via Chocolatey..."
        if (-not (Get-Command choco -ErrorAction SilentlyContinue)) {
            Write-Host "Installing Chocolatey..."
            Set-ExecutionPolicy Bypass -Scope Process -Force
            [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
            Invoke-Expression ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
        }
        choco install openssh -y -ErrorAction Stop
        $installSuccess = $true
        Write-Host "Installation successful via Chocolatey."
    } catch {
        Write-Host "Chocolatey installation failed: $_"
    }
}

# Attempt 3: Manual Download
if (-not $installSuccess) {
    try {
        Write-Host "Attempting manual download and installation..."
        $path = "C:\Program Files\OpenSSH"
        if (-not (Test-Path $path)) { New-Item -ItemType Directory -Path $path -Force | Out-Null }
        $zipUrl = "https://github.com/PowerShell/Win32-OpenSSH/releases/latest/download/OpenSSH-Win64.zip"
        $zipPath = "$path\OpenSSH.zip"
        Invoke-WebRequest -Uri $zipUrl -OutFile $zipPath -ErrorAction Stop
        Add-Type -AssemblyName System.IO.Compression.FileSystem
        [System.IO.Compression.ZipFile]::ExtractToDirectory($zipPath, $path)
        Set-Location $path
        & .\install-sshd.ps1
        $installSuccess = $true
        Write-Host "Installation successful via manual download."
    } catch {
        Write-Host "Manual installation failed: $_"
    }
}

if (-not $installSuccess) {
    Write-Host "All installation attempts failed. Exiting."
    Read-Host "Press Enter to exit..."
    exit 1
}

# Verify and Register sshd Service (Auto-Fix if Missing)
try {
    Write-Host "Verifying sshd service..."
    $service = Get-Service -Name sshd -ErrorAction Stop
    Write-Host "sshd service found: $($service.DisplayName)"
} catch {
    Write-Host "sshd service not found. Registering..."
    try {
        $path = "C:\Program Files\OpenSSH"
        if (Test-Path "$path\sshd.exe") {
            Set-Location $path
            & .\install-sshd.ps1
            Write-Host "sshd service registered."
        } else {
            throw "OpenSSH binaries not found at $path."
        }
    } catch {
        Write-Host "Registration failed: $_"
        Read-Host "Press Enter to exit..."
        exit 1
    }
    try {
        $service = Get-Service -Name sshd -ErrorAction Stop
        Write-Host "sshd service found after registration: $($service.DisplayName)"
    } catch {
        Write-Host "sshd service still not found: $_"
        Read-Host "Press Enter to exit..."
        exit 1
    }
}

# 2. Start SSH Service
try {
    Write-Host "Starting SSH service..."
    Start-Service sshd -ErrorAction Stop
    Write-Host "Service started."
} catch {
    Write-Host "Start failed: $_"
    Read-Host "Press Enter to exit..."
    exit 1
}

# 3. Set Auto-Start
try {
    Write-Host "Setting auto-start..."
    Set-Service -Name sshd -StartupType 'Automatic' -ErrorAction Stop
    Write-Host "Auto-start configured."
} catch {
    Write-Host "Auto-start failed: $_"
    Read-Host "Press Enter to exit..."
    exit 1
}

# 4. Verify Service Status
try {
    Write-Host "Checking status..."
    $serviceStatus = Get-Service sshd -ErrorAction Stop
    Write-Host "Status: $($serviceStatus.Status)"
} catch {
    Write-Host "Status check failed: $_"
    Read-Host "Press Enter to exit..."
    exit 1
}

# 5. Configure Firewall Rule
try {
    Write-Host "Configuring firewall..."
    Remove-NetFirewallRule -Name sshd -ErrorAction SilentlyContinue
    New-NetFirewallRule -Name sshd -DisplayName "OpenSSH Server (sshd)" -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22 -ErrorAction Stop
    Write-Host "Firewall rule configured."
} catch {
    Write-Host "Firewall configuration failed: $_"
    Read-Host "Press Enter to exit..."
    exit 1
}

# 6. Retrieve IP Address
$ipAddress = $null
try {
    Write-Host "Retrieving IP..."
    $ipAddress = (Get-NetIPAddress -AddressFamily IPv4 -InterfaceAlias "Wi-Fi" -ErrorAction SilentlyContinue).IPAddress
    if (-not $ipAddress) {
        $ipAddress = (Get-NetIPAddress -AddressFamily IPv4 | Where-Object { $_.InterfaceAlias -notlike "Loopback*" } | Select-Object -First 1).IPAddress
    }
    if (-not $ipAddress) { throw "No valid IPv4 address found." }
    Write-Host "IP: $ipAddress"
} catch {
    Write-Host "IP retrieval failed: $_"
    Read-Host "Press Enter to exit..."
    exit 1
}

# 7. Retrieve Username
try {
    $username = $env:USERNAME
    Write-Host "Username: $username"
} catch {
    Write-Host "Username retrieval failed: $_"
    Read-Host "Press Enter to exit..."
    exit 1
}

# 8. Display Connection String
Write-Host "Connection string: $username@$ipAddress"

# Pause for Review
Read-Host "Press Enter to exit..."
```

3. **Script Output**:
   - Monitor the console for progress and any errors.
   - At completion, note the **SSH connection string** in the format `username@ip_address` (e.g., `admin@192.168.1.100`). This is required for client connection.

4. **Verification**:
   - Run `Get-Service sshd` in PowerShell to confirm the service is "Running".
   - If issues arise, check Windows Features or run `Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'`.

## Step 2: Connect from Client Device (Example: Android with Termux)
This section uses Termux on Android as an example. Adapt for other clients like PuTTY (Windows), Terminal (macOS/Linux), or iOS apps.

1. **Install Termux**:
   - Download from Google Play Store or F-Droid.

2. **Update and Install SSH Client**:
   - Open Termux.
   - Run: `pkg update`
   - Run: `pkg install openssh`

3. **Establish SSH Connection**:
   - Run: `ssh username@ip_address` (replace with your connection string from Step 1).
   - Accept the host key fingerprint (type `yes`).
   - Enter your Windows user password when prompted.

4. **Usage**:
   - You are now in a remote Windows shell.
   - Execute commands as needed.
   - Disconnect with `exit`.

## Troubleshooting
| Issue | Possible Cause | Solution |
|-------|----------------|----------|
| Connection Refused | Service not running or firewall block | Restart service (`Start-Service sshd`); verify firewall rule. |
| Authentication Failed | Incorrect credentials | Use Windows login password; ensure username matches. |
| No IP Found | Network interface issue | Run `ipconfig` manually to get IPv4 address. |
| Installation Failed | Compatibility or network error | Ensure Windows version; check internet for downloads. |


❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌
# Quick Guide: Reset Windows Password to "bhatt"

Reset your Windows user password to `bhatt` using PowerShell as Administrator.

1. **Open PowerShell as Admin**:
   - Search `PowerShell`, right-click, select **Run as administrator**.

2. **Run Script**:
   - Paste and press Enter:
     ```powershell:disable-run
     Write-Host "Finding user..."
     $currentUser = $env:USERNAME
     Write-Host "Username: $currentUser"
     try {
         Write-Host "Resetting password to 'bhatt'..."
         net user $currentUser bhatt
         Write-Host "Success."
     } catch {
         Write-Host "Failed: $_"
     }
     Read-Host "Press Enter to exit..."
     ```

3. **Check Output**:
   - Look for: `Username: your_username`, `Success.`.
   - Password is now `bhatt`.

4. **Test**:
   - **Login**: Use `bhatt` to log in.
   - **SSH**: Run `ssh your_username@your_ip` with `bhatt`.

## Troubleshooting
- **Access Denied**: Run as Administrator.
- **Script Blocked**: Run `Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned`.
- **SSH Issues**: Check `Get-Service sshd` and `Get-NetFirewallRule -Name sshd`.
❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌






  🐧🐧🐧🐧🐧🐧🐧🐧🐧🐧🐧🐧🐧🐧🐧🐧🐧🐧🐧🐧🐧🐧🐧🐧🐧🐧🐧🐧🐧🐧🐧
# Easy Setup Guide: Remote CLI/SSH Connection to Linux

This guide provides step-by-step instructions for setting up an SSH server on a Linux machine to enable remote command-line interface (CLI) access from another device, such as an Android phone using Termux. Both the host (Linux) and client devices must be connected to the **same local network** (e.g., the same Wi-Fi router). SSH provides secure remote access, but ensure you are operating in a trusted environment.

## Prerequisites
- **Linux Distribution**: Ubuntu, Debian, Fedora, Arch, or similar (script supports common package managers).
- **Root/Sudo Privileges**: All steps require running with `sudo` or as root.
- **Network**: Devices must share the same local network. Verify using `ip addr` on Linux or equivalent on the client.
- **Security Note**: This setup uses default password authentication. For enhanced security, consider SSH key-based authentication (not covered in this basic guide). Root SSH login may be disabled by default—edit `/etc/ssh/sshd_config` if needed.
- **Default Port**: SSH uses port 22. Ensure no firewalls block this port beyond the configured rule.

## Step 1: Configure SSH Server on Linux
1. **Open Terminal**:
   - Open a terminal on your Linux machine (e.g., `Ctrl + Alt + T`).

2. **Execute the Installation Script**:
   - Copy and paste the following Bash script into the terminal.
   - Run it with `sudo bash -c 'paste_the_script_here'` or save as a file, make executable (`chmod +x file.sh`), and run `sudo ./file.sh`.
   - The script automatically installs OpenSSH Server, starts the service, configures the firewall (if `ufw` is available), and displays the connection details.

```bash
#!/bin/bash
# Automated Bash Script: Install and Configure OpenSSH Server on Linux
# Run with sudo or as root. Installs OpenSSH, configures firewall, and displays SSH connection string.

# Exit on any error
set -e

# 1. Check for root/sudo privileges
if [ "$(id -u)" -ne 0 ]; then
    echo "This script requires root privileges. Please run with sudo or as root."
    read -p "Press Enter to exit..."
    exit 1
fi

# 2. Install OpenSSH Server
echo "Installing OpenSSH Server..."
if command -v apt-get >/dev/null; then
    apt-get update
    apt-get install -y openssh-server
elif command -v yum >/dev/null; then
    yum install -y openssh-server
elif command -v dnf >/dev/null; then
    dnf install -y openssh-server
elif command -v pacman >/dev/null; then
    pacman -Syu --noconfirm openssh
else
    echo "Unsupported package manager. Please install openssh-server manually."
    read -p "Press Enter to exit..."
    exit 1
fi
echo "OpenSSH Server installed."

# 3. Start and Enable SSH Service
echo "Starting SSH service..."
systemctl start sshd
systemctl enable sshd
echo "SSH service started and set to auto-start."

# 4. Verify Service Status
echo "Checking SSH service status..."
if systemctl is-active --quiet sshd; then
    echo "SSH service status: Running"
else
    echo "SSH service failed to start."
    read -p "Press Enter to exit..."
    exit 1
fi

# 5. Configure Firewall (if ufw is available)
if command -v ufw >/dev/null; then
    echo "Configuring firewall..."
    ufw allow 22/tcp
    ufw --force enable
    echo "Firewall rule for SSH (port 22) configured."
else
    echo "ufw not found. Ensure port 22 is open in your firewall."
fi

# 6. Get IP Address
echo "Retrieving IP address..."
ip_address=$(ip -4 addr show | grep -v "127.0.0.1" | grep inet | awk '{print $2}' | cut -d'/' -f1 | head -n 1)
if [ -z "$ip_address" ]; then
    echo "Failed to retrieve IP address."
    read -p "Press Enter to exit..."
    exit 1
fi
echo "IP Address: $ip_address"

# 7. Get Username
username=$(whoami)
if [ -z "$username" ]; then
    echo "Failed to retrieve username."
    read -p "Press Enter to exit..."
    exit 1
fi
echo "Username: $username"

# 8. Display Connection String
echo "SSH connection string: $username@$ip_address"

# Pause for review
read -p "Press Enter to exit..."
```

3. **Script Output**:
   - Monitor the terminal for progress and any errors.
   - At completion, note the **SSH connection string** in the format `username@ip_address` (e.g., `user@192.168.1.100`). This is required for client connection.

4. **Verification**:
   - Run `systemctl status sshd` to confirm the service is "active (running)".
   - If issues arise, check `sshd_config` or reinstall OpenSSH.

## Step 2: Connect from Client Device (Example: Android with Termux)
This section uses Termux on Android as an example. Adapt for other clients like PuTTY (Windows), Terminal (macOS/Linux), or iOS apps.

1. **Install Termux**:
   - Download from Google Play Store or F-Droid.

2. **Update and Install SSH Client**:
   - Open Termux.
   - Run: `pkg update`
   - Run: `pkg install openssh`

3. **Establish SSH Connection**:
   - Run: `ssh username@ip_address` (replace with your connection string from Step 1).
   - Accept the host key fingerprint (type `yes`).
   - Enter your Linux user password when prompted.

4. **Usage**:
   - You are now in a remote Linux shell.
   - Execute commands as needed.
   - Disconnect with `exit`.

## Troubleshooting
| Issue | Possible Cause | Solution |
|-------|----------------|----------|
| Connection Refused | Service not running or firewall block | Restart service (`sudo systemctl restart sshd`); verify firewall rule (`ufw status`). |
| Authentication Failed | Incorrect credentials | Use Linux login password; ensure username matches. |
| No IP Found | Network interface issue | Run `ip addr` manually to get IPv4 address. |
| Installation Failed | Unsupported distro or network error | Install manually (e.g., `sudo apt install openssh-server`); check internet. |
❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌




# Quick Guide: Reset Linux Passwords to "bhatt"

Reset your Linux current user and root passwords to `bhatt` using Bash as root/sudo.

1. **Open Terminal**:
   - Open a terminal.

2. **Run Script**:
   - Paste and press Enter (with `sudo`):
     ```bash:disable-run
     #!/bin/bash
     # Automated Bash Script: Reset Root and Current User Passwords to 'bhatt'
     # Run with sudo or as root. Changes passwords for root and the current user.

     # Exit on any error
     set -e

     # 1. Check for root/sudo privileges
     if [ "$(id -u)" -ne 0 ]; then
         echo "This script requires root privileges. Please run with sudo or as root."
         read -p "Press Enter to exit..."
         exit 1
     fi

     # 2. Get and display current user
     echo "Finding current user..."
     current_user=$(logname 2>/dev/null || echo $SUDO_USER)
     if [ -z "$current_user" ]; then
         echo "Failed to identify current user."
         read -p "Press Enter to exit..."
         exit 1
     fi
     echo "Current Username: $current_user"

     # 3. Reset current user's password to 'bhatt'
     echo "Resetting password for $current_user to 'bhatt'..."
     if echo "$current_user:bhatt" | chpasswd; then
         echo "Success: $current_user password reset."
     else
         echo "Failed to reset $current_user password."
         read -p "Press Enter to exit..."
         exit 1
     fi

     # 4. Reset root user's password to 'bhatt'
     echo "Resetting password for root to 'bhatt'..."
     if echo "root:bhatt" | chpasswd; then
         echo "Success: root password reset."
     else
         echo "Failed to reset root password."
         read -p "Press Enter to exit..."
         exit 1
     fi

     # Pause for review
     echo "Password reset complete for $current_user and root."
     read -p "Press Enter to exit..."
     ```

3. **Check Output**:
   - Look for: `Current Username: your_username`, `Success` for both resets.
   - Passwords are now `bhatt`.

4. **Test**:
   - **Login**: Use `bhatt` for current user or root (`su -`).
   - **SSH**: Run `ssh your_username@your_ip` or `ssh root@your_ip` with `bhatt`.

## Troubleshooting
- **Permission Denied**: Run with `sudo` or as root.
- **Script Blocked**: Ensure executable or paste directly.
- **SSH Issues**: Check `sudo systemctl status sshd` and firewall (`ufw status`).

❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌❌
```
