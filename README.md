# rclone-mount-service  
![GitHub stars](https://img.shields.io/github/stars/nobody55555/rclone-mount-service) ![MIT License](https://img.shields.io/badge/license-MIT-blue)

*A robust solution for mounting cloud storage (Google Drive, OneDrive, Dropbox, etc.) as a service on Linux distributions, with troubleshooting for SELinux-enabled systems.*

---

## Table of Contents

- [Key Features](#key-features)  
- [Quick Start (Fedora / systemd)](#quick-start-fedora--systemd)  
- [Repository Structure](#repository-structure)  
- [Manual Installation](#manual-installation)  
- [Troubleshooting](#troubleshooting)  
- [Init System Support](#init-system-support)  
- [Contributing](#contributing)  
- [License](#license)  

---

## Key Features

- **SELinux compatibility** without disabling enforcement  
- **FUSE configuration** and permission setup  
- **Multi-init support** (systemd, OpenRC, future init systems)  
- **Automatic mounts on boot** with clean unmounts  
- **Automatic restarts** on failure  
- **Comprehensive troubleshooting** based on real-world errors  

---

## Quick Start (Fedora / systemd)

```
git clone https://github.com/nobody55555/rclone-mount-service.git
cd rclone-mount-service

# Deploy the systemd unit
sudo cp init/systemd/rclone-gdrive.service /etc/systemd/system/
sudo systemctl daemon-reload

# Enable and start the Google Drive mount service
sudo systemctl enable --now rclone-gdrive.service

# Verify status and mount
sudo systemctl status rclone-gdrive.service
ls ~/GoogleDrive
```

---

## Repository Structure

```
rclone-mount-service/
├── README.md
├── LICENSE
├── init/
│   ├── systemd/
│   │   └── rclone-gdrive.service       ← Systemd unit files
│   └── openrc/
│       └── rclone-gdrive               ← OpenRC runscript
├── scripts/                            ← Future helper scripts
├── selinux/
│   ├── rclone-policies/                ← Custom SELinux modules
│   └── troubleshoot-selinux.sh         ← SELinux diagnostics
├── docs/
│   ├── TROUBLESHOOTING.md              ← Detailed troubleshooting
│   ├── MULTI-CLOUD.md                  ← Multi-provider instructions
│   └── ADVANCED.md                     ← Advanced configuration
└── templates/
    └── user-service-template.conf      ← Generic service template
```

---

## Manual Installation

1. **Install Dependencies**  
   ```
   # Fedora example; adjust for other distros
   sudo dnf install rclone fuse3 fuse3-devel policycoreutils-python-utils
   ```

2. **Configure FUSE**  
   Edit `/etc/fuse.conf` and ensure it contains:
   ```
   user_allow_other
   ```

3. **Add User to fuse Group**  
   ```
   sudo usermod -aG fuse $USER
   # log out and back in to apply
   ```

4. **Prepare Mount Directory**  
   ```
   mkdir -p ~/GoogleDrive
   chown $USER:$USER ~/GoogleDrive
   ```

5. **Deploy Service**  
   ```
   sudo cp init/systemd/rclone-gdrive.service /etc/systemd/system/
   sudo systemctl daemon-reload
   sudo systemctl enable --now rclone-gdrive.service
   ```

6. **SELinux Context (if needed)**  
   ```
   sudo semanage fcontext -a -t user_home_t "$HOME/GoogleDrive(/.*)?"
   sudo restorecon -Rv ~/GoogleDrive
   ```

---

## Troubleshooting

### FUSE Permission Error  
```
option allow_other only allowed if 'user_allow_other' is set in /etc/fuse.conf
```  
*Fix:*  
```
echo 'user_allow_other' | sudo tee -a /etc/fuse.conf
```

### SELinux Denials  
```
sudo ausearch -m AVC -c rclone
sudo ausearch -c 'rclone' --raw | audit2allow -M my-rclone
sudo semodule -i my-rclone.pp
```

### Directory Already Mounted  
```
failed to mount FUSE fs: directory already mounted
```  
*Fix:*  
```
sudo fusermount -uz ~/GoogleDrive
sudo systemctl restart rclone-gdrive.service
```

### User Credential Failures  
```
Failed to determine user credentials: No such process
```  
*Fix:*  
- Verify your user exists:  
  ```
  id $USER
  ```  
- Ensure `User=` and `Group=` in the service file match your username.

### Systemd Syntax Errors  
```
Failed to parse Type=notify: Invalid argument
```  
*Fix:*  
Remove inline comments; use separate lines beginning with `#`.

#### Debug Commands

```
sudo systemctl status rclone-gdrive.service
sudo journalctl -fu rclone-gdrive.service
pgrep -a rclone
mount | grep GoogleDrive
```

---

## Init System Support

- **systemd** (Fedora, Debian, Ubuntu, Arch, etc.): `init/systemd/`  
- **OpenRC** (Gentoo, Alpine, etc.): `init/openrc/`  

Deploy the appropriate script for your distribution.

---

## Contributing

1. Fork the repository  
2. Create a feature branch (`git checkout -b feature-name`)  
3. Test on a clean system  
4. Commit your changes (`git commit -m "Add feature"`)  
5. Push to your branch and open a Pull Request  

---

## License

This project is licensed under the LGPL 2.1 License. See [LICENSE](LICENSE) for details.

---

**⭐ If this project helped you, please give it a star!**  
```
