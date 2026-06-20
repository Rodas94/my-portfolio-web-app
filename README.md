# Configuring a PXE Network Boot Server for Ubuntu 20.04 and Windows 10

This guide details the steps to configure a PXE network boot server on Ubuntu 20.04 with the following settings:

- **Server IP:** `10.240.34.55`
- **Server Name:** `ns1.astu.local.net`
- **Boot Operating Systems:** Ubuntu 20.04 and Windows 10

## Table of Contents

- [Prerequisites](#prerequisites)
- [Step 1: Install Required Packages](#step-1-install-required-packages)
- [Step 2: Configure DHCP Server](#step-2-configure-dhcp-server)
- [Step 3: Configure TFTP Server](#step-3-configure-tftp-server)
- [Step 4: Set Up PXE Boot Loader](#step-4-set-up-pxe-boot-loader)
- [Step 5: Configure NFS for Ubuntu Installation](#step-5-configure-nfs-for-ubuntu-installation)
- [Step 6: Set Up Windows 10 PXE Files](#step-6-set-up-windows-10-pxe-files)
- [Firewall and Security Rules](#firewall-and-security-rules)
- [Verify Services](#verify-services)
- [Monitor PXE Boot Logs](#monitor-pxe-boot-logs)

## Prerequisites

- An Ubuntu 20.04 server with root access
- DHCP and TFTP services
- Installation ISO images for Ubuntu 20.04 and Windows 10
- Network interface configured with static IP (`10.240.34.55`)

## Step 1: Install Required Packages

```bash
sudo apt update
sudo apt install isc-dhcp-server tftp-hpa tftpd-hpa syslinux pxelinux nfs-kernel-server
```

## Step 2: Configure DHCP Server

### Network Information

Display routing table and network interface configuration:

```bash
route -n          # Displays the routing table
ifconfig          # Displays the current configuration for a network interface
```

### DHCP Configuration

Edit the DHCP configuration file:

```bash
sudo vi /etc/dhcp/dhcpd.conf
```

Add the following configuration:

```text
ddns-update-style none;
authoritative;

subnet 10.240.34.0 netmask 255.255.255.0 {
    interface enp0s3;
    range 10.240.34.100 10.240.34.200;
    option routers 10.240.34.1;
    option broadcast-address 10.240.34.255;
    option subnet-mask 255.255.255.0;
    option domain-name-servers 10.240.34.55;
    option domain-name "your-domain.local";
    next-server 10.240.34.55;
    filename "pxelinux.0";
    default-lease-time 600;
    max-lease-time 7200;
}
```

### DHCP Service Configuration

Edit the `/etc/default/isc-dhcp-server` file:

```bash
sudo vi /etc/default/isc-dhcp-server
```

Set the interface:

```text
INTERFACESv4="enp0s3"
```

### Restart DHCP Service

```bash
sudo systemctl restart isc-dhcp-server
sudo systemctl status isc-dhcp-server
```

### Troubleshooting DHCP

If any error occurs, check the process ID and logs:

```bash
# Check the process ID (example: 2167)
journalctl _PID=2167
```

## Step 3: Configure TFTP Server

Edit the TFTP configuration file:

```bash
sudo vi /etc/default/tftpd-hpa
```

Set the following options:

```text
TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/var/lib/tftpboot"
TFTP_ADDRESS="0.0.0.0:69"
TFTP_OPTIONS="--secure"
```

Restart TFTP service:

```bash
sudo systemctl enable tftpd-hpa.service
sudo systemctl restart tftpd-hpa
sudo systemctl status tftpd-hpa
```

## Step 4: Set Up PXE Boot Loader

### Create Required PXE Directories and Files

```bash
sudo mkdir -p /var/lib/tftpboot/pxelinux.cfg
sudo mkdir -p /var/lib/tftpboot/
sudo cp /usr/lib/PXELINUX/pxelinux.0 /var/lib/tftpboot/
sudo cp /usr/lib/syslinux/modules/bios/* /var/lib/tftpboot/
```

### Create PXE Boot Menu Configuration

Create the PXE boot menu configuration file:

```bash
sudo vi /var/lib/tftpboot/pxelinux.cfg/default
```

Add the following content:

```text
DEFAULT menu.c32
PROMPT 0
TIMEOUT 100
ONTIMEOUT local
MENU TITLE PXE Boot Menu

LABEL Ubuntu
    MENU LABEL Install Ubuntu 20.04
    KERNEL ubuntu/vmlinuz
    APPEND initrd=ubuntu/initrd root=/dev/nfs nfsroot=10.240.34.55:/srv/nfs/ubuntu ip=dhcp rw

LABEL Windows10
    MENU LABEL Install Windows 10
    KERNEL boot/wimboot
    INITRD boot/bcd,boot/boot.sdi,boot/boot.wim,boot/bootmgr

LABEL LocalBoot
    MENU LABEL Boot from Local Drive
    LOCALBOOT 0
```

## Step 5: Configure NFS for Ubuntu Installation

### Set Up NFS Directory

```bash
sudo mkdir -p /srv/nfs/ubuntu
```

### Mount Ubuntu ISO

```bash
sudo chmod 644 /path/to/ubuntu.iso
sudo mount -o loop /path/to/ubuntu.iso /mnt
```

### Copy Ubuntu Boot Files to TFTP Directory

Verify that the kernel (`vmlinuz`) and initramfs (`initrd`) files are present:

```bash
ls -l /var/lib/tftpboot/ubuntu/
```

If they are missing, copy them:

```bash
sudo mkdir -p /var/lib/tftpboot/ubuntu
sudo cp /mnt/casper/vmlinuz /var/lib/tftpboot/ubuntu/
sudo cp /mnt/casper/initrd /var/lib/tftpboot/ubuntu/
```

### Copy Ubuntu Files to NFS

```bash
sudo rsync -av /mnt/ /srv/nfs/ubuntu/
```

### Unmount the ISO

```bash
sudo umount /mnt
```

### Configure NFS Exports

Edit the NFS exports configuration:

```bash
sudo vi /etc/exports
```

Add the following line:

```text
/srv/nfs/ubuntu *(ro,sync,no_root_squash,no_subtree_check)
```

### Verify NFS Export

```bash
sudo exportfs -v
```

You should see output similar to:

```text
/srv/nfs/ubuntu  <world>(ro,sync,wdelay,no_root_squash,no_subtree_check)
```

### Restart NFS Server

```bash
sudo systemctl restart nfs-kernel-server
sudo systemctl status nfs-kernel-server
```

### Test NFS

Test from a client machine or the server itself:

```bash
showmount -e 10.240.34.55
```

You should see `/srv/nfs/ubuntu` listed.

## Step 6: Set Up Windows 10 PXE Files

### Mount the Windows ISO

Create a mount point:

```bash
sudo mkdir -p /mnt/windowsAIO
```

Mount the Windows ISO:

```bash
sudo mount -o loop /home/rod/ftp/upload/Windows.10.X64.22H2.iso /mnt/windowsAIO
```

### Create TFTPboot Directory

```bash
sudo mkdir -p /var/lib/tftpboot/boot
```

### Copy Required Boot Files

Copy `wimboot` from the ISO:

```bash
# The exact location of wimboot may vary
sudo cp /mnt/windowsAIO/sources/boot/wimboot /var/lib/tftpboot/
```

If `wimboot` is missing from the ISO, download it from the iPXE project:

```bash
sudo wget -P /var/lib/tftpboot/ https://ipxe.org/wimboot
sudo chmod 644 /var/lib/tftpboot/wimboot
```

Copy essential boot files:

```bash
sudo cp /mnt/windowsAIO/sources/boot/{BCD,boot.sdi,boot.wim} /var/lib/tftpboot/boot/
sudo cp /mnt/windowsAIO/boot/* /var/lib/tftpboot/boot/
```

### Unmount the Windows ISO

```bash
sudo umount /mnt/windowsAIO
```

## Firewall and Security Rules

Configure the firewall on the PXE server to allow TFTP, NFS, and DHCP traffic:

```bash
sudo ufw allow 69/udp    # TFTP
sudo ufw allow 2049/tcp  # NFS
sudo ufw allow 67/udp    # DHCP
```

## Verify Services

Ensure all services are running:

```bash
sudo systemctl restart isc-dhcp-server tftpd-hpa nfs-kernel-server
sudo systemctl status isc-dhcp-server tftpd-hpa nfs-kernel-server
```

## Monitor PXE Boot Logs

Check the logs to identify errors:

### DHCP Logs

```bash
sudo journalctl -u isc-dhcp-server
```

### TFTP Logs

```bash
sudo journalctl -u tftpd-hpa
```

### NFS Logs

```bash
sudo journalctl -u nfs-kernel-server
```

## Troubleshooting

### Common Issues

1. **DHCP not starting**: Check the interface name in `/etc/default/isc-dhcp-server` matches your actual network interface.

2. **TFTP permission denied**: Ensure the TFTP directory has correct permissions:

   ```bash
   sudo chown -R tftp:tftp /var/lib/tftpboot
   ```

3. **NFS mount fails**: Verify the NFS export is correct:

   ```bash
   sudo exportfs -ra
   ```

4. **Windows PXE boot fails**: Ensure all required files are copied to the boot directory:
   ```bash
   ls -la /var/lib/tftpboot/boot/
   ```

### Debugging Tips

- Check all service statuses: `sudo systemctl status isc-dhcp-server tftpd-hpa nfs-kernel-server`
- Verify network connectivity: `ping 10.240.34.55`
- Test TFTP from client: `tftp 10.240.34.55 -c get pxelinux.0`

## Additional Resources

- [Ubuntu DHCP Server Documentation](https://ubuntu.com/server/docs/service-dhcp)
- [Syslinux Documentation](https://wiki.syslinux.org/)
- [iPXE Project](https://ipxe.org/)
- [NFS HOWTO](https://nfs.sourceforge.net/)

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Support

For issues or questions, please create an issue in the repository or contact the system administrator.
