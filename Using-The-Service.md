# Using OpenVPN Manager - User Guide

This guide covers how end users can generate and download OpenVPN profiles using OpenVPN Manager's web interface.

## 🚀 Getting Started

OpenVPN Manager provides a self-service portal for users to generate their own OpenVPN certificates and configuration files. The system integrates with your organization's OIDC identity provider (such as Azure AD, Okta, or Google Workspace) for secure authentication.

### Prerequisites

- Access to your organization's OpenVPN Manager instance
- Valid account with your organization's identity provider (OIDC)
- Modern web browser with JavaScript enabled

## 🔐 Authentication

### Web Browser Authentication

1. **Navigate** to your organization's OpenVPN Manager URL (e.g., `https://vpn.yourcompany.com`)
2. **Click** the "Generate Profile" button
3. **Authenticate** using your organization's single sign-on (SSO) system
4. **Complete** any multi-factor authentication (MFA) requirements
5. **Return** to the OpenVPN Manager portal automatically

The system uses OpenID Connect (OIDC) to integrate with enterprise identity providers, ensuring that only authorized users can generate certificates.

## 📱 Generating Your OpenVPN Profile

### Step 1: Access the Profile Generator

After authentication, you'll see the main profile generation interface:

- **Profile Generator**: The main interface for creating your OpenVPN configuration
- **Options Selection**: Choose connection protocols and server options
- **Download Area**: Where your generated profile will appear

### Step 2: Select Connection Options

OpenVPN Manager supports multiple connection methods:

- **UDP on port 1194** (default): Best performance for most users
- **TCP on port 443**: Better for restrictive networks and firewalls
- **Combined profile**: Includes both UDP and TCP configurations

**Recommendation**: Start with UDP for best performance. Switch to TCP if you experience connection issues through firewalls or corporate networks.

### Step 3: Generate Your Certificate

1. **Select** your preferred connection options
2. **Click** "Generate Profile"
3. **Wait** for the system to:
   - Generate your unique certificate
   - Create the OpenVPN configuration
   - Log the certificate issuance for audit purposes

### Step 4: Download Your Profile

Once generation is complete:

1. **Download** the `.ovpn` file to your device
2. **Save** the file in a secure location
3. **Import** the profile into your OpenVPN client

The download uses a secure, time-limited token to ensure your certificate is delivered safely.

## 🔧 OpenVPN Client Setup

### Desktop Clients

#### Windows
1. **Download** OpenVPN GUI from [openvpn.net](https://openvpn.net/community-downloads/)
2. **Install** the application
3. **Right-click** on your `.ovpn` file and select "Start OpenVPN on this config file"
4. **Connect** using the system tray icon

#### macOS
1. **Download** Tunnelblick from [tunnelblick.net](https://tunnelblick.net/)
2. **Install** the application
3. **Double-click** your `.ovpn` file to import
4. **Connect** from the Tunnelblick menu

#### Linux
1. **Install** OpenVPN client: `sudo apt install openvpn` (Ubuntu/Debian)
2. **Copy** your `.ovpn` file to `/etc/openvpn/client/`
3. **Connect** using: `sudo openvpn --config your-profile.ovpn`

### Mobile Clients

#### iOS
1. **Install** OpenVPN Connect from the App Store
2. **Import** your `.ovpn` file via AirDrop, email, or cloud storage
3. **Add** the profile when prompted
4. **Connect** from the app interface

#### Android
1. **Install** OpenVPN Connect from Google Play Store
2. **Import** your `.ovpn` file from your device storage
3. **Add** the profile when prompted
4. **Connect** from the app interface

## 🔄 Profile Management

### Certificate Validity

- **Validity Period**: Certificates are valid for 30 days by default
- **Renewal**: Generate a new profile before your current certificate expires
- **Multiple Profiles**: You can generate new profiles at any time; previous certificates remain valid until expiry

### Security Best Practices

1. **Secure Storage**: Store your `.ovpn` file securely and don't share it
2. **Regular Renewal**: Generate new certificates regularly for better security
3. **Multiple Devices**: Generate separate profiles for each device you use
4. **Report Issues**: Contact your administrator if you suspect certificate compromise

## 🛠️ Command-Line Tool

For advanced users and automation, OpenVPN Manager provides a command-line tool for profile retrieval.

### Installation

```bash
# Download the tool from your OpenVPN Manager instance
curl -o get_openvpn_config.py https://vpn.yourcompany.com/tools/get_openvpn_config.py

# Make it executable
chmod +x get_openvpn_config.py
```

### Usage

#### Basic Profile Retrieval
```bash
# Interactive authentication and download
python get_openvpn_config.py --server-url https://vpn.yourcompany.com

# Specify output location
python get_openvpn_config.py --server-url https://vpn.yourcompany.com --output ~/vpn-config.ovpn

# Select specific options (e.g., TCP instead of UDP)
python get_openvpn_config.py --server-url https://vpn.yourcompany.com --options tcp
```

#### Configuration File

Create a config file at `~/.config/ovpn-manager/config.yaml`:

```yaml
server_url: https://vpn.yourcompany.com
output: ~/Downloads/config.ovpn
overwrite: false
options: udp  # or tcp, or tcp,udp for combined
```

Then simply run:
```bash
python get_openvpn_config.py
```

### Environment Variables

The tool also supports environment variables:

```bash
export OVPN_MANAGER_URL=https://vpn.yourcompany.com
export OVPN_MANAGER_OUTPUT=~/my-vpn-config.ovpn
export OVPN_MANAGER_OPTIONS=tcp

python get_openvpn_config.py
```

## ❓ Troubleshooting

### Connection Issues

**Problem**: Cannot connect to OpenVPN server
- **Solution**: Try switching between UDP and TCP protocols
- **Check**: Firewall settings and network restrictions
- **Verify**: Certificate hasn't expired

**Problem**: Authentication fails
- **Solution**: Generate a new profile with fresh certificate
- **Check**: System time is correct (affects certificate validation)

**Problem**: Slow connection speeds
- **Solution**: Use UDP protocol instead of TCP
- **Check**: Try different server endpoints if multiple are available

### Profile Generation Issues

**Problem**: "Authentication Required" error
- **Solution**: Clear browser cookies and re-authenticate
- **Check**: Your OIDC account is active and has VPN access

**Problem**: Download fails or times out
- **Solution**: Try again; download tokens are single-use and time-limited
- **Check**: Browser popup blockers aren't interfering

### Common Error Messages

- **"Certificate has expired"**: Generate a new profile
- **"Authentication failed"**: Check your username/password or MFA
- **"Network unreachable"**: Check internet connectivity and firewall settings
- **"TLS handshake failed"**: Verify certificate validity and server configuration

## 🆘 Getting Help

### Self-Service Resources

1. **Try generating a new profile** - Often resolves certificate-related issues
2. **Check different connection options** - UDP vs TCP can resolve network issues
3. **Verify system time** - Incorrect time can cause certificate validation failures

### Contact Support

For persistent issues:

- **IT Help Desk**: Contact your organization's IT support team
- **Network Team**: For firewall and network connectivity issues
- **Security Team**: For certificate or authentication problems

Include in your support request:
- Operating system and OpenVPN client version
- Error messages (screenshots helpful)
- Network environment (office, home, public WiFi)
- Whether the issue is new or ongoing

---

**Stay Connected**: OpenVPN Manager makes it easy to maintain secure access to your organization's resources. Generate profiles as needed and keep your certificates current for the best security posture.

## 🤖 AI Assistance Disclosure

This documentation was developed with assistance from AI tools. While released under a permissive license that allows unrestricted reuse, we acknowledge that portions of the implementation may have been influenced by AI training data. Should any copyright assertions or claims arise regarding uncredited imported code, the affected portions will be rewritten to remove or properly credit any unlicensed or uncredited work.