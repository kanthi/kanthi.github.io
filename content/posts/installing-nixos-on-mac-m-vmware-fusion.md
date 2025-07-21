---
title: "Installing NixOS on Apple Silicon Macs with VMware Fusion"
date: 2025-07-21T10:15:00+05:30
draft: false
tags: ["nixos", "vmware-fusion", "virtualization", "apple-silicon", "m3", "arm64", "flakes"]
categories: ["NixOS", "Virtualization", "macOS"]
---

# Installing NixOS on Apple Silicon Macs with VMware Fusion in 2025

In this updated guide for 2025, I'll walk through the process of installing NixOS on VMware Fusion running on an Apple Silicon Mac (M1/M2/M3). NixOS has become increasingly popular for developers seeking reproducible environments, and with the latest improvements in both NixOS and VMware Fusion, the experience on Apple Silicon is now excellent.

## Prerequisites

- VMware Fusion 14 or later (optimized for Apple Silicon)
- NixOS ISO image for AArch64 (ARM64) - minimal or graphical installation image
- At least 40GB of free disk space (recommended for comfortable usage)
- Minimum 8GB RAM to allocate to the VM (4GB will work but performance will suffer)
- macOS Sequoia (14.0) or later

## Step 1: Download NixOS

1. Visit the [NixOS Download Page](https://nixos.org/download.html)
2. Select the "NixOS 25.05 (Latest)" version
3. Download the minimal or graphical ISO for AArch64 (ARM64)
   - Minimal ISO is smaller and boots faster
   - Graphical ISO includes a desktop environment for easier installation

![NixOS Download Page](/posts/installing-nixos-on-mac-m-vmware-fusion/images/nixos-download.png)

## Step 2: Create New VM in VMware Fusion

1. Open VMware Fusion
2. Click "+" or "File > New" to create a new virtual machine
3. Drag and drop the downloaded NixOS ISO or click "Create a custom virtual machine"
4. Choose "Linux" as the operating system
5. Select "Other Linux 6.x kernel 64-bit ARM" as the version (VMware Fusion 14 now has better ARM Linux support)
6. Choose "Create a new virtual disk" when prompted

![VMware New VM](/posts/installing-nixos-on-mac-m-vmware-fusion/images/vmware-new-vm.png)

## Step 3: Configure VM Settings

Configure the following settings for optimal performance in 2025:

1. Memory: 8GB (recommended for modern workloads)
2. Processors: 4 cores (Apple Silicon Macs have plenty of cores to spare)
3. Hard Disk: 40GB (NixOS store can grow large with multiple generations)
4. Network Adapter: Share with my Mac (NAT)
5. Display: Enable 3D graphics (now works well with NixOS on ARM)
6. USB & Bluetooth: Enable to allow device passthrough
7. Advanced options:
   - Enable "Virtualize CPU performance counters" for better performance monitoring
   - Set firmware type to "UEFI" (not legacy BIOS)

![VM Settings](/posts/installing-nixos-on-mac-m-vmware-fusion/images/vm-settings.png)

## Step 4: Install NixOS

### Option 1: Using the Graphical Installer (New in 2025)

If you downloaded the graphical ISO:

1. Start the VM and boot from the ISO
2. When the desktop environment loads, click on the "Install NixOS" icon
3. Follow the graphical installer prompts:
   - Select language and keyboard layout
   - Choose automatic or manual partitioning
   - Create your user account
   - Select desktop environment (GNOME, KDE, etc.)
   - Review and confirm installation

### Option 2: Traditional Command Line Installation

If you prefer the command line or are using the minimal ISO:

1. Start the VM and boot from the ISO
2. When the boot menu appears, select "NixOS Installer"
3. Log in as 'nixos' user (no password required)
4. Run the following commands:

```bash
sudo -i

# Create a GPT partition table
parted /dev/vda -- mklabel gpt

# Create partitions
parted /dev/vda -- mkpart ESP fat32 1MiB 512MiB
parted /dev/vda -- set 1 esp on
parted /dev/vda -- mkpart primary 512MiB 100%

# Format partitions
mkfs.fat -F 32 /dev/vda1
mkfs.ext4 /dev/vda2

# Mount partitions
mount /dev/vda2 /mnt
mkdir -p /mnt/boot
mount /dev/vda1 /mnt/boot

# Generate NixOS configuration
nixos-generate-config --root /mnt

# Edit configuration
nano /mnt/etc/nixos/configuration.nix
```

![NixOS Installation](/posts/installing-nixos-on-mac-m-vmware-fusion/images/nixos-install.png)

## Step 5: Modern NixOS Configuration

### Option 1: Traditional Configuration

Add these essential configurations to `/mnt/etc/nixos/configuration.nix`:

```nix
{ config, pkgs, ... }:

{
  imports =
    [ # Include the results of the hardware scan.
      ./hardware-configuration.nix
    ];

  # Enable systemd-boot
  boot.loader.systemd-boot.enable = true;
  boot.loader.efi.canTouchEfiVariables = true;

  # Enable Nix flakes
  nix.settings.experimental-features = [ "nix-command" "flakes" ];

  # VMware guest optimization
  virtualisation.vmware.guest.enable = true;

  # Network configuration
  networking.hostName = "nixos-vm"; # Define your hostname
  networking.networkmanager.enable = true;

  # Set your time zone
  time.timeZone = "Asia/Kolkata";

  # Enable sound with PipeWire (modern replacement for PulseAudio)
  security.rtkit.enable = true;
  services.pipewire = {
    enable = true;
    alsa.enable = true;
    alsa.support32Bit = true;
    pulse.enable = true;
  };

  # Enable Wayland with X11 compatibility
  services.xserver = {
    enable = true;
    displayManager.gdm.enable = true;
    displayManager.gdm.wayland = true;
    desktopManager.gnome.enable = true;
  };

  # Define a user account
  users.users.yourusername = {
    isNormalUser = true;
    extraGroups = [ "wheel" "networkmanager" ];
    initialPassword = "changeme";
    shell = pkgs.zsh;
  };

  # Enable ZSH
  programs.zsh.enable = true;

  # Allow unfree packages
  nixpkgs.config.allowUnfree = true;

  # List packages installed in system profile
  environment.systemPackages = with pkgs; [
    vim
    wget
    git
    firefox
    vscode
    btop
    curl
    gnome.gnome-tweaks
  ];

  # System state version - do not change
  system.stateVersion = "25.05";
}
```

### Option 2: Modern Flakes-Based Configuration (Recommended for 2025)

For a more modern approach, create a flake-based configuration:

1. First, create a basic flake.nix file:

```bash
mkdir -p /mnt/etc/nixos/flake
cd /mnt/etc/nixos
mv configuration.nix hardware-configuration.nix flake/
```

2. Create `/mnt/etc/nixos/flake.nix`:

```nix
{
  description = "NixOS configuration for VMware on Apple Silicon";

  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-25.05";

    home-manager = {
      url = "github:nix-community/home-manager/release-25.05";
      inputs.nixpkgs.follows = "nixpkgs";
    };
  };

  outputs = { self, nixpkgs, home-manager, ... }:
    let
      system = "aarch64-linux";
      pkgs = nixpkgs.legacyPackages.${system};
    in {
      nixosConfigurations.nixos-vm = nixpkgs.lib.nixosSystem {
        inherit system;
        modules = [
          ./flake/configuration.nix
          home-manager.nixosModules.home-manager
          {
            home-manager.useGlobalPkgs = true;
            home-manager.useUserPackages = true;
            home-manager.users.yourusername = import ./flake/home.nix;
          }
        ];
      };
    };
}
```

3. Create `/mnt/etc/nixos/flake/home.nix`:

```nix
{ config, pkgs, ... }:

{
  home.username = "yourusername";
  home.homeDirectory = "/home/yourusername";
  home.stateVersion = "25.05";

  # Install user packages
  home.packages = with pkgs; [
    ripgrep
    fd
    jq
    fzf
  ];

  # Configure Git
  programs.git = {
    enable = true;
    userName = "Your Name";
    userEmail = "your.email@example.com";
    extraConfig = {
      init.defaultBranch = "main";
    };
  };

  # Configure VS Code
  programs.vscode = {
    enable = true;
    extensions = with pkgs.vscode-extensions; [
      vscodevim.vim
      ms-python.python
      github.copilot
    ];
  };
}
```

![Configuration Edit](/posts/installing-nixos-on-mac-m-vmware-fusion/images/config-edit.png)
```

![Configuration Edit](/posts/installing-nixos-on-mac-m-vmware-fusion/images/config-edit.png)

## Step 6: Install the System

### For Traditional Configuration

Run the installation:

```bash
nixos-install
```

Set root password when prompted. After installation completes, reboot:

```bash
reboot
```

### For Flakes-Based Configuration

Run the installation with flakes enabled:

```bash
nixos-install --flake /mnt/etc/nixos#nixos-vm
```

Set root password when prompted. After installation completes, reboot:

```bash
reboot
```

![Installation Complete](/posts/installing-nixos-on-mac-m-vmware-fusion/images/install-complete.png)

## Post-Installation Steps

### For Traditional Configuration

1. Log in with your user account
2. Change your password using `passwd`
3. Update your system:
```bash
sudo nix-channel --update
sudo nixos-rebuild switch
```

### For Flakes-Based Configuration

1. Log in with your user account
2. Change your password using `passwd`
3. Update your system:
```bash
cd /etc/nixos
sudo nix flake update
sudo nixos-rebuild switch --flake .#nixos-vm
```

### Install VMware Tools (if not already included)

If you didn't enable VMware guest support in your configuration:

```bash
sudo nano /etc/nixos/configuration.nix
# Add: virtualisation.vmware.guest.enable = true;
sudo nixos-rebuild switch
```

### Enable Shared Folders (Optional)

To access folders shared from your Mac:

```bash
# Add to configuration.nix:
fileSystems."/mnt/hgfs" = {
  device = ".host:/";
  fsType = "fuse./run/current-system/sw/bin/vmhgfs-fuse";
  options = [ "defaults" "allow_other" "uid=1000" ];
};
```

![First Boot](/posts/installing-nixos-on-mac-m-vmware-fusion/images/first-boot.png)

## Performance Optimization

To get the best performance from your NixOS VM on Apple Silicon:

1. **CPU Optimization**:
   ```nix
   # Add to configuration.nix
   hardware.cpu.arm.enable = true;
   ```

2. **Memory Management**:
   ```nix
   # Add to configuration.nix
   boot.kernel.sysctl = {
     "vm.swappiness" = 10;
     "vm.vfs_cache_pressure" = 50;
   };
   ```

3. **I/O Scheduler**:
   ```nix
   # Add to configuration.nix
   boot.kernelParams = [ "elevator=none" ];
   ```

4. **GPU Acceleration**:
   VMware Fusion 14+ supports limited 3D acceleration for Linux guests on Apple Silicon. Enable it in VM settings.

## Troubleshooting

Common issues you might encounter:

1. **Black screen after boot**: Try adding `nomodeset` to your boot parameters in the GRUB menu

2. **Network issues**: Ensure VMware Tools is installed:
   ```nix
   virtualisation.vmware.guest.enable = true;
   ```

3. **Resolution problems**: Configure display resolution in configuration.nix:
   ```nix
   services.xserver.resolutions = [
     { x = 1920; y = 1080; }
   ];
   ```

4. **Slow performance**: Check if virtualization extensions are enabled:
   ```bash
   dmesg | grep -i kvm
   ```

5. **Flakes errors**: If you encounter issues with flakes, ensure experimental features are enabled:
   ```nix
   nix.settings.experimental-features = [ "nix-command" "flakes" ];
   ```

## Advanced Usage

### Running Docker in NixOS VM

Docker works well in NixOS on Apple Silicon:

```nix
# Add to configuration.nix
virtualisation.docker = {
  enable = true;
  enableOnBoot = true;
};

# Add your user to the docker group
users.users.yourusername.extraGroups = [ "docker" ];
```

### Development Environments with Devenv

[Devenv](https://devenv.sh/) works great with NixOS on Apple Silicon:

```bash
# Install devenv
nix profile install github:cachix/devenv/latest

# Create a new project
mkdir my-project && cd my-project
devenv init
```

## Conclusion

You now have a modern NixOS installation running on VMware Fusion on your Apple Silicon Mac. This setup provides an excellent environment for development, experimentation, and learning about NixOS's unique approach to system configuration.

With the flakes-based approach, you can easily version control your entire system configuration and reproduce it across different machines or share it with teammates.

## References

- [Official NixOS Manual](https://nixos.org/manual/nixos/stable/)
- [Nix Flakes Documentation](https://nixos.wiki/wiki/Flakes)
- [VMware Fusion Documentation](https://docs.vmware.com/en/VMware-Fusion/index.html)
- [NixOS Wiki](https://nixos.wiki/)
- [NixOS on ARM](https://nixos.wiki/wiki/NixOS_on_ARM)
- [Zero to Nix](https://zero-to-nix.com/)
- [Home Manager Manual](https://nix-community.github.io/home-manager/)
