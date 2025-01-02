<p align="center">
<img src="https://upload.wikimedia.org/wikipedia/commons/d/d1/Raspberry_Pi_OS_Logo.png" alt="raspberrypi-logo"/>
</p>
# Fully Automate System Configuration on Raspberry Pi OS

This tutorial guides you through setting up an automated system configuration on Raspberry Pi OS using systemd services and timers. This approach offers a cost-effective alternative to relying on virtual machines or cloud resources for continuous automation tasks.<br />

## Video Demonstration

- ### [YouTube: Automate System Configuration on Raspberry Pi](https://www.youtube.com)

## Environments and Technologies Used

- Raspberry Pi Device
- Linux Terminal
- SSH
- VIM
- SystemD
- Bash Scripting

## Operating Systems Used

- Raspberry Pi OS
- Windows 11

## High-Level Deployment and Configuration Steps

- **Step 1:** Set Up Raspberry Pi & Install Operating System
- **Step 2:** Develop the Automation Script
- **Step 3:** Configure SystemD Service and Timer
- **Step 4:** Deploy and Monitor the Automated Service

# Deployment and Configuration Steps

## SSH into RaspberryPi
<p align="center">
  <img src="https://github.com/user-attachments/assets/297c9d8a-3c5a-4260-8c9f-cdea03dd0f78" height="80%" width="80%" alt="SSH into device over LAN"/>
</p>
<p>
  Begin by connecting to your Raspberry Pi via SSH over your local network. Ensure that SSH is enabled and that you can successfully log in to your device. This remote access is crucial for managing and deploying scripts without needing a direct monitor or keyboard connection.
</p>
<br />

## Create and Add Script to Automate the Updates
<p align="center">
  <img src="https://github.com/user-attachments/assets/e89687ee-23ba-4984-ae37-e8fdfbdf4042" height="80%" width="80%" alt="Script Development in VIM"/>
</p>
<p>
  Develop your automation script using VIM or your preferred text editor. This script will handle system updates, maintenance tasks, and any other configurations you wish to automate. Ensure the script has executable permissions and is tested manually before integrating it with systemd.
</p>
<br />

## Configure System-D Service File
<p align="center">
  <img src="https://github.com/user-attachments/assets/766c34ff-1030-44ae-b6ee-389d7bfab866" height="80%" width="80%" alt="SystemD Configuration"/>
</p>
<p>
  Configure systemd by creating a service and timer unit file. The service file defines the script to be executed, while the timer file schedules the execution at desired intervals (e.g., every 24 hours). Use `sudo` privileges to edit these files and ensure they have the correct permissions.
</p>
<br />

## Configure System-D Timer File
<p align="center">
  <img src="https://github.com/user-attachments/assets/7adaed39-1462-47de-99dc-72b10615b634" height="80%" width="80%" alt="SystemD Configuration"/>
</p>
<p>
  Configure systemd by creating a service and timer unit file. The service file defines the script to be executed, while the timer file schedules the execution at desired intervals (e.g., every 15 hours). Use `sudo` privileges to edit these files and ensure they have the correct permissions.
</p>
<br />

## Deploy the .service and .timer files, restart daemon, and check logs
<p align="center">
  <img src="https://github.com/user-attachments/assets/a79e1e0d-523c-48b0-b1f7-7a277c7a89d6" height="80%" width="80%" alt="Monitoring Service Logs"/>
<p>
  After deploying the service and timer, monitor the logs to verify that the automation runs as expected. Utilize `journalctl` to check for any errors and ensure that the script executes successfully at each scheduled interval. Adjust configurations as needed based on the log outputs.
</p>
<br />

# Detailed Steps

### Step 1: Set Up Raspberry Pi & Install Operating System
<p>
  - Download the latest Raspberry Pi OS image.<br />
  - Use tools like Balena Etcher to flash the OS onto your SD card.<br />
  - Insert the SD card into your Raspberry Pi and boot up.<br />
  - Connect to your network via Ethernet or Wi-Fi.<br />
  - Enable SSH by creating an empty `ssh` file in the boot partition.<br />
</p>
<br />

### Step 2: Develop the Automation Script
<p>
  - ** Create a directory for your scripts: **<br />
    <code>mkdir -p ~/scripts</code><br />
  - **Create and edit your automation script:**<br />
    <code>nano ~/scripts/auto_update.sh</code><br />
  - **Add the following content:**<br />
  
    ```bash
    #!/usr/bin/env bash

    LOGFILE="/home/pi/logs/auto_update.log"

    echo "===== Auto-Update Script Started at $(date) =====" >> "$LOGFILE"

    # Update package lists
    sudo apt-get update -y >> "$LOGFILE" 2>&1

    # Upgrade installed packages
    sudo apt-get upgrade -y >> "$LOGFILE" 2>&1

    # Remove unused packages
    sudo apt-get autoremove -y >> "$LOGFILE" 2>&1

    # Disk usage
    echo "Disk Usage:" >> "$LOGFILE"
    df -h >> "$LOGFILE" 2>&1

    # Memory usage
    echo "Memory Usage:" >> "$LOGFILE"
    free -h >> "$LOGFILE" 2>&1

    echo "===== Auto-Update Script Finished at $(date) =====" >> "$LOGFILE"
    echo "" >> "$LOGFILE"
    ```
  - **Make the script executable:**<br />
    <code>chmod +x ~/scripts/auto_update.sh</code><br />
</p>
<br />

### Step 3: Configure SystemD Service and Timer
<p>
  <strong>Create the Service File:</strong><br />
  
  - **Open the service file for editing:**<br />
  
    <code>sudo nano /etc/systemd/system/auto-update.service</code><br />
    
  - **Add the following content:**<br />
    
  ```ini
    [Unit]
    Description=Auto Update Raspberry Pi
    After=network.target

    [Service]
    Type=oneshot
    ExecStart=/home/pi/scripts/auto_update.sh
    User=pi #or your specific username
    Group=pi #or your specific group
    Restart=on-failure
    Environment=PATH=/usr/bin:/bin

    [Install]
    WantedBy=multi-user.target

```
  - **Save and exit.**<br /><br />

  <strong>Create the Timer File:</strong><br />
  - **Open the timer file for editing:**<br />
    <code>sudo nano /etc/systemd/system/auto-update.timer</code><br />
  - **Add the following content:**<br />
    ```ini
    [Unit]
    Description=Run auto-update service every 24 hours

    [Timer]
    OnBootSec=5min
    OnUnitActiveSec=24h
    Unit=auto-update.service

    [Install]
    WantedBy=timers.target
    ```
  - **Save and exit.**<br /><br />

  <strong>Enable and Start the Timer:</strong><br />
  - **Reload systemd to recognize the new unit files:**<br />
    <code>sudo systemctl daemon-reload</code><br />
  - **Enable the timer to start on boot:**<br />
    <code>sudo systemctl enable auto-update.timer</code><br />
  - **Start the timer immediately:**<br />
    <code>sudo systemctl start auto-update.timer</code><br />
</p>
<br />

### Step 4: Deploy and Monitor the Automated Service
<p>
  
  **Check Timer Status:**<br />
    <code>systemctl status auto-update.timer</code><br />
    Ensure it is active and the next run time is correct.<br /><br />

  **Check Service Logs:**<br />
    <code>journalctl -u auto-update.service -f</code><br />
    Monitor real-time logs to verify successful execution.<br /><br />

  **Review Script Logs:**<br />
    <code>cat /home/pi/auto_update.log</code><br />
    Confirm that updates and system checks are being logged as expected.<br /><br />

  **Troubleshoot Issues:**<br />
    If the service fails to execute, check the following:<br />
    - Correct script path and permissions.<br />
    - Proper user and group settings in the service file.<br />
    - Environment variables and PATH settings.<br />
    - Any errors in the script itself.
</p>
<br />

## Conclusion

By following this tutorial, you have successfully automated system updates and checks on your Raspberry Pi using systemd services and timers. This setup ensures your device remains up-to-date and monitored without manual intervention, providing a reliable and efficient automation solution.

