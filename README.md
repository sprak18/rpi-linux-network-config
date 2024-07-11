# Setting Up Raspberry Pi Lite with Custom Kernel, Boot Partition, Network Configuration and SSH

## Step 1: Download and Write Raspberry Pi OS Lite to SD Card

1. **Download Raspberry Pi OS Lite**
   - Download the latest Raspberry Pi OS Lite image from the [official Raspberry Pi website](https://www.raspberrypi.com/software/operating-systems/).

2. **Write the Image to SD Card**
   - Use `dd` command or any other image writing tool to write the downloaded image to the SD card.

    ```bash
    sudo dd if=path/to/raspios-lite.img of=/dev/sdX bs=4M status=progress
    sync
    ```

Replace `/dev/sdX` with the appropriate device identifier for your SD card. Be careful to choose the correct device to avoid data loss.

## Step 2: Prepare the Boot Partition

1. **Mount the SD Card Partitions**
- After writing the image, remove and reinsert the SD card, then mount the boot and root partitions.

```bash
sudo mount /dev/sdX1 /mnt/boot
sudo mount /dev/sdX2 /mnt/root
```


2. **Prepare U-Boot Boot Script**
- Create a boot command script to configure U-Boot.

```bash
cat << EOF > boot_cmd.txt
fatload mmc 0:1 ${kernel_addr_r} kernel8.img
setenv bootargs "console=serial0,115200 console=tty1 root=/dev/mmcblk0p2 rw rootwait"
booti ${kernel_addr_r} - ${fdt_addr}
EOF
```


- Convert the boot command script to a U-Boot script.

```bash
~/u-boot/tools/mkimage -A arm64 -O linux -T script -C none -d boot_cmd.txt boot.scr
sudo cp boot.scr /mnt/boot/
```


3. **Edit `config.txt`**
- Update the `config.txt` file to configure U-Boot.

```bash
sudo cp config.txt /mnt/boot/
```

4. **Copy U-Boot Binary**
- Copy the U-Boot binary to the boot partition.

```bash
sudo cp u-boot.bin /mnt/boot/
```

## Step 3: Replace the Kernel

1. **Copy Custom Kernel and Device Tree**
- Copy your custom kernel and device tree files to the boot partition.

```bash
sudo cp arch/arm64/boot/Image /mnt/boot/kernel8.img
sudo cp arch/arm64/boot/dts/broadcom/bcm2711-rpi-4-b.dtb /mnt/boot/
```

2. **Install Kernel Modules**
- Install kernel modules to the root filesystem.

```bash
sudo make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- INSTALL_MOD_PATH=/mnt/root modules_install
```


## Step 4: Prepare the Root Filesystem

1. **Create Necessary Directories**
- Ensure the necessary directories exist in the root filesystem.

```bash
sudo mkdir -p /mnt/root/lib/firmware /mnt/root/lib/modules
```

2. **Copy Firmware Files**
- Copy necessary firmware files to the root filesystem.

```bash
sudo cp -r /path/to/firmware/* /mnt/root/lib/firmware/
```

3. **Unmount Partitions**
- Unmount the partitions.

```bash
sudo umount /mnt/boot
sudo umount /mnt/root
```

## Step 5: Configure Wi-Fi to Connect on Boot

1. **Mount Root Partition**
- Remount the root partition to make changes.

```bash
sudo mount /dev/sdX2 /mnt/root
```


2. **Create wpa_supplicant.conf**
- Create a `wpa_supplicant.conf` file with your network details.

```bash
sudo tee /mnt/root/etc/wpa_supplicant/wpa_supplicant.conf <<EOF
country=US
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
ssid="your_SSID"
psk="your_password"
}
EOF
```

3. **Enable Wi-Fi Interface**
- Create a file to enable the Wi-Fi interface on boot.

```bash
sudo tee /mnt/root/etc/network/interfaces.d/wlan0 <<EOF
auto wlan0
iface wlan0 inet dhcp
wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
EOF
```

4. **Unmount Root Partition**
- Unmount the root partition.

```bash
sudo umount /mnt/root
```


## Step 6: Configure SSH

1. **Enable SSH**
- Create an empty `ssh` file in the boot partition to enable SSH.

```bash
sudo mount /dev/sdX1 /mnt/boot
sudo touch /mnt/boot/ssh
sudo umount /mnt/boot
```

## Step 7: Boot and Connect

1. **Insert the SD Card and Power On**
- Insert the SD card into the Raspberry Pi and power it on.

2. **Set Keyboard Layout to US**
- Once booted, set the keyboard layout to US.

```bash
sudo raspi-config
```

Navigate to `Localisation Options` > `Change Keyboard Layout` and select `Generic 105-key (Intl) PC` and then `English (US)`.

3. **Check Network Connection**
- Verify that the Raspberry Pi is connected to the network.

```bash
ping -c 4 google.com
```

4. **SSH into Raspberry Pi**
- SSH into the Raspberry Pi from your host machine.

```bash
ssh pi@<Raspberry_Pi_IP_Address>
```

If you encounter a warning about the remote host identification, remove the offending key and try again.

```bash
ssh-keygen -f "/home/sprak/.ssh/known_hosts" -R "<Raspberry_Pi_IP_Address>"
```

Then, SSH again.


Enter the password you set up for the `pi` user during the initial configuration.

You're done.
