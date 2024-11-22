# Ubuntu Server
---

### Step 1: Create a Bootable Pendrive

1. **Download** the following:
   - Rufus tool
   - Ubuntu server ISO image

2. **Open Rufus**:
   - In the **Device** dropdown, select your pen drive.
   - Click **Select** and choose the downloaded Ubuntu ISO image.

3. **Configure Rufus**:
   - In **Partition scheme**, select **MBR**.
   - Set **Target system** to **BIOS or UEFI**.
   - Leave all other settings unchanged.

4. Click **Start**, and when prompted, select **Write in DD image mode**.

5. Wait for the process to complete, then click **OK** and wait for the **Status** to indicate completion.

---

### Step 2: BIOS Configuration

1. **Enter BIOS settings**:
   - Set the boot order to prioritize booting from the pen drive first.
   - Enable necessary option. It differs for everyone.

2. Press **F10** to save changes and reboot. You may need to press additional keys for the system to boot from the pen drive.

---

### Step 3: Installation

1. Proceed with the standard Ubuntu Server installation.

2. Select **Use entire disk** for the disk partitioning option, and **disable** the partition option to avoid any further partitioning steps.

---

### Step 4: Configure Static IP 

- One way is to Choose an Unassigned IP in Your Network Subnet
- Reserve an IP in the Router Panel

- I am using the 1st option.

**Ping the IP Address**:  
   Use the following command to check if an IP is already in use:  
   ```bash
   ping -c 3 <IP>
   ```
   - If you get a response, the IP is already in use.  
   - If there is no response, the IP is free to use.

Note: The IP need to be from your Network Subnet.

1. After installation, run the following command to identify your network interface:
   ```bash
   ip a
   ```
   Example interfaces:
   - Wired connection: `eth0`
   - Wi-Fi connection: `wlan0`

2. Navigate to the network configuration directory:
   ```bash
   cd /etc/netplan/
   ls
   ```

3. Use an editor to open a file (likely named something like `00-installer-config.yaml`):
   ```bash
   sudo nano 00-installer-config.yaml # It can differ
   ```

4. **Backup the file** before making any changes.

5. Add the following configuration for a static IP:
```yaml
   network:
     version: 2
     renderer: networkd
     ethernets:
       eth0:  # Replace 'eth0' with your actual interface name
         addresses:
           - (ip/subnet)  # Replace with your static IP and subnet mask
         gateway4: (gateway)  # Replace with your network gateway
         nameservers:
           addresses:
             - 8.8.8.8 # Add which you want
             - 8.8.4.4
```

6. Apply the changes:
   ```bash
   sudo netplan apply
   ```

7. Reboot the server:
   ```bash
   sudo reboot
   ```

8. After rebooting, check the IP address to ensure it has been set to static and remains unchanged.

---

## Step 5: Configure Wake-On-LAN WOL  (Optional)

1. **Ensure WOL is enabled in BIOS**:
   - If WOL is not available in BIOS, your system does not support it.

2. Install the required tools:
   ```bash
   sudo apt-get install ethtool
   ```

3. Verify if WOL is supported by running:
   ```bash
   sudo ethtool eth0
   ```
   - Look for the line `Supports Wake-on`. If it includes `g`, your network adapter supports WOL.

4. Create a systemd service to enable WOL on boot:
   ```bash
   sudo nano /etc/systemd/system/wol.service
   ```

5. Add the following content to the service file:
```init
   [Unit]
   Description=Config wake on lan

   [Service]
   Type=oneshot
   ExecStart=/usr/sbin/ethtool -s eth0 wol g

   [Install]
   WantedBy=basic.target
```
6. Reload the systemd daemon to recognize the new service:
   ```bash
   sudo systemctl daemon-reload
   ```

7. Enable the WOL service to start on boot:
   ```bash
   sudo systemctl enable wol.service
   ```

8. Start the WOL service:
   ```bash
   sudo systemctl start wol.service
   ```

9. Reboot the system to ensure the settings persist:
   ```bash
   sudo reboot now
   ```

---
