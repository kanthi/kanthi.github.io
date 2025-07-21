+++
date = "2020-06-02T14:00:23+05:30"
draft = false
title = "My Linux Laptop Setup II (2020): Manjaro Edition"
tags = ["general", "setup", "manjaro_os", "linux"]
categories = ["General", "Setup", "Manjaro_OS"]
+++

After using Elementary OS for almost half a decade since the Luna release, I wanted to explore other distributions. As I mentioned in my previous posts, Slackware was my first Linux experience, so I wanted to go for something similar this time and ended up with Arch-based Manjaro. It turned out to be a great choice! I'm still using the same laptop from my previous setup post (HP EliteBook 2570P).

This post documents my complete Manjaro setup process and the applications I use for development, productivity, and daily computing.

![Manjaro Desktop](/img/my-linux-laptop-setup-2/4.png)
*My Manjaro desktop with custom theme*

![System Information](/img/my-linux-laptop-setup-2/5.png)
*System specs and neofetch output*

![Application Menu](/img/my-linux-laptop-setup-2/0.png)
*Application launcher and installed software*

![Development Environment](/img/my-linux-laptop-setup-2/1.png)
*Development tools and IDE setup*

![Terminal Setup](/img/my-linux-laptop-setup-2/2.png)
*Terminal configuration with custom prompt*

![System Monitor](/img/my-linux-laptop-setup-2/3.png)
*System monitoring and resource usage*

# Initial System Setup

## System Update and Upgrade

The first thing to do after a fresh Manjaro installation is to update the system:

```bash
# Update package database and upgrade system
sudo pacman -Syu

# Update AUR packages (if using yay)
yay -Syu
```

## Privacy and Security Settings

### Disable GNOME Tracker (if using GNOME)

GNOME Tracker can be resource-intensive and privacy-concerning:

```bash
# Disable tracker services
systemctl --user mask tracker-store.service tracker-miner-fs.service tracker-miner-rss.service tracker-miner-apps.service tracker-writeback.service

# Stop running tracker processes
tracker reset --hard
```

### Configure Firewall

```bash
# Install and enable UFW firewall
sudo pacman -S ufw
sudo ufw enable
sudo systemctl enable ufw
```

# Essential Applications

## Web Browsers

I install multiple browsers for different use cases:

```bash
# Install browsers via pacman
sudo pacman -S firefox

# Install from AUR
yay -S google-chrome
yay -S opera
yay -S brave-bin
yay -S vivaldi
```

**Browser Usage:**
- **Firefox**: Primary browser for general use
- **Chrome**: Web development and testing
- **Opera**: VPN and workspace features
- **Brave**: Privacy-focused browsing
- **Vivaldi**: Power user features and customization

## Terminal and Shell

```bash
# Install terminal emulators
sudo pacman -S terminator tilix

# Install shell enhancements
sudo pacman -S zsh fish
yay -S oh-my-zsh-git

# Install terminal utilities
sudo pacman -S htop neofetch tree bat exa fd ripgrep
```

## Media Players

```bash
# Install media players
sudo pacman -S mpv vlc audacious

# Install codecs
sudo pacman -S gst-plugins-good gst-plugins-bad gst-plugins-ugly gst-libav
```

**Media Setup:**
- **MPV**: Lightweight video player for quick viewing
- **VLC**: Full-featured media player for all formats
- **Audacious**: Music player with Winamp-like interface

## Productivity Applications

```bash
# Email client
sudo pacman -S geary thunderbird

# Note-taking and documentation
yay -S joplin-appimage
yay -S cherrytree
yay -S notion-app

# E-book management
sudo pacman -S calibre

# Time management
yay -S solanum

# Screen utilities
sudo pacman -S redshift
yay -S caffeine-ng
```

## Cloud Storage and Sync

```bash
# Install cloud storage clients
yay -S dropbox
yay -S github-desktop-bin

# Alternative sync tools
sudo pacman -S rsync rclone
```

# Development Environment

## Programming Languages and Runtimes

### Python Development

```bash
# Install Python and package manager
sudo pacman -S python python-pip python-virtualenv

# Install development tools
pip install --user pylint black autopep8 flake8
pip install --user jupyter notebook ipython
```

### Node.js Development

```bash
# Install Node.js and npm
sudo pacman -S nodejs npm

# Configure npm for global packages without sudo
echo 'prefix = ~/.npm' > ~/.npmrc
echo 'export PATH=$PATH:~/.npm/bin' >> ~/.bashrc

# Install global packages
npm install -g typescript @angular/cli create-react-app
```

**NPM Configuration Details:**

If you don't want to execute npm with root privileges for global packages, here's the complete setup:

1. **Configure npm**: Create `.npmrc` in your home directory:
```bash
prefix = ~/.npm
```

2. **Configure PATH**: Add to your `~/.bash_profile`:
```bash
export PATH=$PATH:~/.npm/bin
```

This tells npm to install global packages in `~/.npm/lib/node_modules` and makes executables available in `~/.npm/bin`.

### Go Development

```bash
# Install Go
sudo pacman -S go

# Set up Go workspace
mkdir -p ~/go/{bin,src,pkg}
echo 'export GOPATH=$HOME/go' >> ~/.bashrc
echo 'export PATH=$PATH:$GOPATH/bin' >> ~/.bashrc
```

### Rust Development

```bash
# Install Rust via rustup
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source ~/.cargo/env

# Install common tools
cargo install cargo-edit cargo-watch
```

### Other Languages

```bash
# Dart and Flutter
yay -S flutter

# Ruby
sudo pacman -S ruby rubygems

# Lua
sudo pacman -S lua

# D language
sudo pacman -S dmd

# PowerShell
yay -S powershell-bin
```

## Text Editors and IDEs

```bash
# Install editors
sudo pacman -S gedit vim neovim
yay -S visual-studio-code-bin
yay -S sublime-text-4

# Install Android Studio
yay -S android-studio
```

**Editor Configuration:**
- **VSCode**: Primary IDE with extensions for all languages
- **Sublime Text**: Quick editing and large file handling
- **Neovim**: Terminal-based editing with custom config
- **Android Studio**: Mobile app development

## Version Control

```bash
# Install Git and related tools
sudo pacman -S git git-lfs
yay -S github-cli

# Configure Git
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
git config --global init.defaultBranch main
```

# Virtualization and Containerization

## Docker Setup

```bash
# Install Docker
sudo pacman -S docker docker-compose

# Start and enable Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Add user to docker group
sudo usermod -aG docker $USER

# Verify installation
sudo docker version
```

## VMware Workstation

```bash
# Install VMware Workstation from AUR
yay -S vmware-workstation

# Load VMware modules
sudo modprobe -a vmw_vmci vmmon

# Enable VMware services
sudo systemctl enable --now vmware-networks.service
sudo systemctl enable --now vmware-usbarbitrator.service
sudo systemctl enable --now vmware-hostd.service
```

**VMware Services Explained:**
- `vmware-networks.service`: Provides network access inside VMs (most people need this)
- `vmware-usbarbitrator.service`: Allows USB devices to be connected inside VMs
- `vmware-hostd.service`: Enables sharing of VMs on the network

## KVM/QEMU Setup

```bash
# Install KVM and related packages
sudo pacman -S qemu virt-manager virt-viewer dnsmasq vde2 bridge-utils openbsd-netcat ebtables iptables

# Install text editor if needed
sudo pacman -S vim

# Configure libvirt
sudo vim /etc/libvirt/libvirtd.conf
```

**Configure libvirt** by editing `/etc/libvirt/libvirtd.conf`:

```bash
# Set UNIX domain socket group ownership (around line 85)
unix_sock_group = "libvirt"

# Set UNIX socket permissions (around line 102)
unix_sock_rw_perms = "0770"
```

```bash
# Add user to libvirt group
sudo usermod -a -G libvirt $(whoami)
newgrp libvirt

# Restart and enable libvirt daemon
sudo systemctl restart libvirtd.service
sudo systemctl enable libvirtd.service
```

### Enable Nested Virtualization (Optional)

Nested Virtualization allows you to run VMs inside a VM:

```bash
# Enable nested virtualization for Intel
sudo modprobe -r kvm_intel
sudo modprobe kvm_intel nested=1

# Make configuration persistent
echo "options kvm-intel nested=1" | sudo tee /etc/modprobe.d/kvm-intel.conf

# Verify nested virtualization is enabled
systool -m kvm_intel -v | grep nested
cat /sys/module/kvm_intel/parameters/nested
```

Expected output should show:
```
nested              = "Y"
nested_early_check  = "N"
```

## VirtualBox

```bash
# Install VirtualBox
sudo pacman -S virtualbox virtualbox-host-modules-arch

# Add user to vboxusers group
sudo usermod -a -G vboxusers $USER

# Load VirtualBox modules
sudo modprobe vboxdrv
```

## Network Simulation

```bash
# Install GNS3 for network simulation
yay -S gns3-gui gns3-server

# Install additional network tools
sudo pacman -S wireshark-qt nmap netcat
```

# Graphics and Design

```bash
# Install graphics applications
sudo pacman -S inkscape gimp blender

# Install image viewers and editors
sudo pacman -S gwenview feh imagemagick

# Install screenshot tools
sudo pacman -S spectacle flameshot
```

# System Utilities

## File Management

```bash
# Install file managers
sudo pacman -S dolphin thunar ranger

# Install archive tools
sudo pacman -S ark unrar p7zip

# Install disk utilities
sudo pacman -S gparted baobab ncdu
```

## System Monitoring

```bash
# Install monitoring tools
sudo pacman -S htop iotop nethogs iftop
yay -S btop

# Install system information tools
sudo pacman -S neofetch screenfetch inxi
```

## USB and ISO Tools

```bash
# Install USB creation tools
yay -S popsicle-gtk
sudo pacman -S dd_rescue

# Install ISO tools
sudo pacman -S fuseiso acetoneiso
```

**Popsicle** is a great tool for creating bootable USB drives with a simple GUI interface.

# System Tweaks and Optimizations

## Performance Tweaks

```bash
# Install performance tools
sudo pacman -S preload profile-sync-daemon

# Enable preload for faster application startup
sudo systemctl enable preload

# Configure swappiness for better performance
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
```

## Power Management

```bash
# Install power management tools
sudo pacman -S tlp tlp-rdw powertop

# Enable TLP for laptop battery optimization
sudo systemctl enable tlp
sudo systemctl start tlp

# Configure laptop mode
sudo pacman -S laptop-mode-tools
```

## Audio Configuration

```bash
# Install audio tools
sudo pacman -S pulseaudio pulseaudio-alsa pavucontrol

# Install additional audio codecs and Bluetooth support
sudo pacman -S pulseaudio-bluetooth a2dp-alsa
```

# Backup and Maintenance

## Backup Solutions

```bash
# Install backup tools
sudo pacman -S rsync borgbackup timeshift

# Configure automatic snapshots with Timeshift
sudo timeshift --create --comments "Initial setup complete"
```

## System Maintenance

```bash
# Install maintenance tools
sudo pacman -S bleachbit stacer

# Set up automatic package cache cleaning
sudo pacman -S pacman-contrib
sudo systemctl enable paccache.timer
```

# Custom Configurations

## Shell Configuration

Create a custom `.bashrc` with useful aliases:

```bash
# Add to ~/.bashrc
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'
alias grep='grep --color=auto'
alias fgrep='fgrep --color=auto'
alias egrep='egrep --color=auto'
alias ..='cd ..'
alias ...='cd ../..'
alias h='history'
alias c='clear'
alias df='df -h'
alias du='du -h'
alias free='free -h'
alias ps='ps auxf'
alias psgrep='ps aux | grep -v grep | grep -i -e VSZ -e'

# Git aliases
alias gs='git status'
alias ga='git add'
alias gc='git commit'
alias gp='git push'
alias gl='git log --oneline'

# Docker aliases
alias dps='docker ps'
alias dpa='docker ps -a'
alias di='docker images'
alias drm='docker rm'
alias drmi='docker rmi'

# Pacman aliases
alias pacupg='sudo pacman -Syu'
alias pacin='sudo pacman -S'
alias pacins='sudo pacman -U'
alias pacre='sudo pacman -R'
alias pacrem='sudo pacman -Rns'
alias pacrep='pacman -Si'
alias pacfind='pacman -Ss'
alias pacown='pacman -Qo'
alias pacorphan='sudo pacman -Rs $(pacman -Qtdq)'
```

## Git Configuration

```bash
# Enhanced Git configuration
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.visual '!gitk'
git config --global core.editor vim
git config --global merge.tool vimdiff
```

# Additional Applications

## Note-Taking and Documentation

- **Joplin**: Cross-platform note-taking with sync capabilities
- **CherryTree**: Hierarchical note-taking application
- **Notion**: All-in-one workspace (via web app)

## Communication

```bash
# Install communication apps
yay -S discord
yay -S slack-desktop
yay -S telegram-desktop
sudo pacman -S thunderbird
```

# Hardware Considerations

## HP EliteBook 2570P Specific

This laptop works well with Manjaro, but here are some specific considerations:

- **Graphics**: Intel HD Graphics 4000 works out of the box
- **WiFi**: Intel Centrino Advanced-N 6205 has excellent Linux support
- **Battery**: TLP configuration significantly improves battery life
- **Thermal**: No special configuration needed, runs cool

## Performance Optimizations

```bash
# Install CPU frequency scaling
sudo pacman -S cpupower

# Set CPU governor for better battery life
sudo cpupower frequency-set -g powersave

# Or for performance
sudo cpupower frequency-set -g performance
```

# Future Plans

Looking ahead, I'm considering upgrading to:

- **MacBook Pro**: For better battery life and build quality
- **Dell XPS Developer Edition**: Official Linux support with excellent hardware
- **Lenovo ThinkPad with Fedora**: The upcoming Lenovo+Fedora collaboration
- **Framework Laptop**: Modular and repairable design with Linux support

The main factors driving a potential upgrade:
- Better battery life (8+ hours)
- Higher resolution display
- More RAM (16GB+)
- Faster SSD storage
- Better build quality and portability

# Conclusion

This Manjaro setup has served me exceptionally well for development, productivity, and daily computing tasks. The Arch-based foundation provides access to the latest software while maintaining stability through Manjaro's testing process.

## Key Advantages of This Setup

- **Rolling Release**: Always up-to-date software without major version upgrades
- **AUR Access**: Vast software repository with community packages
- **Performance**: Lightweight and fast, even on older hardware
- **Customization**: Highly configurable to match personal preferences
- **Development-Friendly**: Excellent tooling support for multiple languages
- **Hardware Compatibility**: Great support for older laptops

## Workflow Benefits

The combination of traditional applications and modern development tools creates a versatile environment suitable for:

- Web development (Node.js, Python, Go)
- Mobile development (Android Studio, Flutter)
- System administration (Docker, KVM, networking tools)
- Content creation (graphics, video, documentation)
- Research and learning (virtualization, multiple browsers)

## Maintenance Strategy

Regular maintenance tasks I perform:

```bash
# Weekly system update
sudo pacman -Syu && yay -Syu

# Monthly cleanup
sudo pacman -Sc
yay -Sc
sudo journalctl --vacuum-time=2weeks

# Quarterly full backup
sudo timeshift --create --comments "Quarterly backup"
```

## Lessons Learned

After using this setup for several months:

1. **AUR is powerful** but requires occasional manual intervention
2. **Rolling releases** need regular updates to avoid issues
3. **Virtualization** is essential for testing and development
4. **Multiple browsers** are necessary for web development
5. **Backup strategy** is crucial with rolling releases

The modular approach allows for easy customization and expansion as needs evolve, making this setup both practical and future-proof for development work.

![Neofetch Output](/img/my-linux-laptop-setup-2/neofetch.png)
*Final system configuration with all tools installed*
