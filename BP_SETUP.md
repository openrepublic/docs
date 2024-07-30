# Debian Server Configuration

## Table of Contents
1. [Check System Version](#check-system-version)
2. [Change Machine Name](#change-machine-name)
3. [Add User and Set to Reset Password on First Login](#add-user-and-set-to-reset-password-on-first-login)
4. [Add New User to Sudoers](#add-new-user-to-sudoers)
5. [Setup SSH Keys for New User](#setup-ssh-keys-for-new-user)
6. [Disable SSH Password Login, Only Allow Key Authentication](#disable-ssh-password-login-only-allow-key-authentication)
7. [Configure CPU Governor for Best Performance](#configure-cpu-governor-for-best-performance)
8. [Configure TCP/IP Stack](#configure-tcpip-stack)
9. [Set Permanent ulimit Values](#set-permanent-ulimit-values)
10. [Set Up Unattended Upgrades](#set-up-unattended-upgrades)
11. [Install and Configure Fail2ban](#install-and-configure-fail2ban)
12. [Install and Configure Chrony](#install-and-configure-chrony)

## Check System Version

To determine your Debian system version, you can use the following command in the terminal:

```bash
cat /etc/debian_version
```

This command will display the version of Debian you are running. Alternatively, you can use the `lsb_release` command for more detailed information:

```bash
lsb_release -a
```

This will provide you with detailed information about your Debian distribution, including the release version and codename.

For more details, refer to the [Debian Documentation](https://www.debian.org/doc/manuals/debian-reference/ch02.en.html#_obtaining_version_information).

## Change Machine Name

To change the machine name (hostname) in Debian, you need to update two files and then reboot your system.

1. **Edit the `/etc/hostname` file:**

   Open the file in a text editor with superuser privileges:

   ```bash
   sudo nano /etc/hostname
   ```

   Change the content of this file to your new hostname, for example, `new_hostname`. Save the file and exit the editor.

2. **Edit the `/etc/hosts` file:**

   Open the file in a text editor with superuser privileges:

   ```bash
   sudo nano /etc/hosts
   ```

   Find the line that looks like this:

   ```
   127.0.0.1    hostname
   ```

   Change `hostname` to `new_hostname`, so it looks like this:

   ```
   127.0.0.1    new_hostname
   ```

   Save the file and exit the editor.

3. **Reboot your system:**

   ```bash
   sudo reboot
   ```

   After rebooting, your machine name should be updated to `new_hostname`.

For more details, refer to the [Debian Hostname Configuration](https://wiki.debian.org/Hostname).

## Add User and Set to Reset Password on First Login

To add a new user and set the account to require a password reset on the first login, follow these steps:

1. **Add a new user:**

   Use the `adduser` command to create a new user. Replace `username` with the desired username.

   ```bash
   sudo adduser username
   ```

   Follow the prompts to set the user's initial password and other details.

2. **Set the user to reset the password on the first login:**

   Use the `chage` command to set the user account to require a password change on first login. Replace `username` with the username you created.

   ```bash
   sudo chage -d 0 username
   ```

   This command sets the password expiration date to the epoch (i.e., January 1, 1970), which forces the user to change their password at the next login.

For more details, refer to the [Debian User Management Documentation](https://www.debian.org/doc/manuals/debian-reference/ch07.en.html#_user_management).

## Add New User to Sudoers

To grant a new user sudo privileges, follow these steps:

1. **Open the sudoers file:**

   Use the `visudo` command to safely edit the sudoers file:

   ```bash
   sudo visudo
   ```

2. **Add the user to the sudoers file:**

   Find the line that specifies user privileges, and add the following line, replacing `username` with the new user's username:

   ```bash
   username ALL=(ALL) ALL
   ```

   This line grants the user sudo privileges.

3. **Save and exit the editor:**

   Save the changes and exit the editor. If you're using `nano`, you can do this by pressing `CTRL+X`, then `Y`, and then `Enter`.

For more details, refer to the [Debian Sudo Configuration](https://wiki.debian.org/sudo).

## Setup SSH Keys for New User

To set up SSH keys for the new user, follow these steps:

1. **Generate SSH key pair on your local machine:**

   On your local machine, open a terminal and use the following command to generate an SSH key pair with a specific filename. Replace `your_email@example.com` with your email address.

   ```bash
   ssh-keygen -t ed25519 -C "your_email@example.com" -f ~/.ssh/ed25519
   ```

   Follow the prompts to save the key pair and optionally set a passphrase.

2. **Copy the public key to the server:**

   Use the `ssh-copy-id` command to copy your public key to the new user's `~/.ssh/authorized_keys` file on the server. Replace `username` with the new user's username and `server_ip` with the server's IP address.

   ```bash
   ssh-copy-id -i ~/.ssh/ed25519.pub username@server_ip
   ```

   Alternatively, you can manually copy the public key:

   ```bash
   cat ~/.ssh/ed25519.pub | ssh username@server_ip "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
   ```

3. **Set the correct permissions on the server:**

   On the server, ensure the `.ssh` directory and `authorized_keys` file have the correct permissions:

   ```bash
   ssh username@server_ip
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/authorized_keys
   ```

For more details, refer to the [Debian SSH Configuration](https://www.debian.org/doc/manuals/securing-debian-howto/ch-sec-services.en.html#s-ssh).

## Disable SSH Password Login, Only Allow Key Authentication

To disable SSH password login and only allow key authentication, follow these steps:

1. **Open the SSH configuration file:**

   Open the SSH daemon configuration file in a text editor with superuser privileges:

   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

2. **Disable password authentication:**

   Find the following lines and modify or add them to disable password authentication:

   ```bash
   PasswordAuthentication no
   ```

   Ensure the following lines are set to the correct values to allow key-based authentication:

   ```bash
   PubkeyAuthentication yes
   ```

3. **Ensure Challenge-Response Authentication is disabled:**

   ```bash
   ChallengeResponseAuthentication no
   ```

4. **Use PAM (Pluggable Authentication Modules):**

   Ensure this line is present and uncommented:

   ```bash
   UsePAM yes
   ```

5. **Disable root login with password:**

   Find and set the following line to ensure root login is disabled with a password:

   ```bash
   PermitRootLogin prohibit-password
   ```

6. **Restart the SSH service:**

   Restart the SSH service to apply the changes:

   ```bash
   sudo systemctl restart ssh
   ```

7. **Verify the changes:**

   Open a new terminal session or use another device to attempt to SSH into your server. The server should now only allow key-based authentication and deny password-based logins.

8. **Check for any additional configurations:**

   Ensure there are no other configuration files overriding the main `sshd_config`. Sometimes, additional configurations can be included in `/etc/ssh/sshd_config.d/` directory.

   ```bash
   sudo nano /etc/ssh/sshd_config.d/custom.conf
   ```

   Ensure that the `custom.conf` file or any other files in this directory do not have conflicting settings.

For more details, refer to the [Debian SSH Configuration](https://www.debian.org/doc/manuals/securing-debian-howto/ch-sec-services.en.html#s-ssh).

## Configure CPU Governor for Best Performance

To configure the CPU governor for the best performance, follow these steps:

1. **Install the `cpufrequtils` package:**

   Install the necessary package to manage CPU frequency:

   ```bash
   sudo apt-get update
   sudo apt-get install cpufrequtils
   ```

2. **Set the CPU governor to performance:**

   Edit the `cpufrequtils` configuration file:

   ```bash
   sudo nano /etc/default/cpufrequtils
   ```

   Add or modify the following line to set the governor to `performance`:

   ```bash
   GOVERNOR="performance"
   ```

   Save the file and exit the editor.

3. **Restart the `cpufrequtils` service:**

   Restart the service to apply the changes:

   ```bash
   sudo systemctl restart cpufrequtils
   ```

4. **Verify the CPU governor:**

   Verify that the CPU governor is set to `performance`:

   ```bash
   cpufreq-info | grep "current policy"
   ```

   The output should indicate that the current policy is set to `performance`.

## Configure TCP/IP Stack

To optimize the TCP/IP stack for better performance, follow these steps:

1. **Edit the `/etc/sysctl.conf` file:**

   Open the sysctl configuration file in a text editor with superuser privileges:

   ```bash
   sudo nano /etc/sysctl.conf
   ```

2. **Add the following lines to the file:**

   ```bash
   net.core.somaxconn=1024
   net.ipv4.tcp_tw_reuse=1
   net.ipv4.ip_local_port_range=1024 65535
   net.ipv4.tcp_max_syn_backlog=4096
   net.ipv4.tcp_fin_timeout=15
   ```

3. **Save the file and exit the editor:**

   Save the changes and exit the editor.

4. **Apply the changes:**

   Apply the changes by running the following command:

   ```bash
   sudo sysctl -p
   ```

   This command will reload the sysctl configuration and apply the new settings.

5. **Make the changes permanent:**

   Create a custom sysctl configuration file to ensure the changes persist across reboots:

   ```bash
   sudo nano /etc/sysctl.d/99-custom.conf
   ```

   Add the same configuration lines:

   ```bash
   net.core.somaxconn=1024
   net.ipv4.tcp_tw_reuse=1
   net.ipv4.ip_local_port_range=1024 65535
   net.ipv4.tcp_max_syn_backlog=4096
   net.ipv4.tcp_fin_timeout=15
   ```

   Save the file and exit the editor. The settings will now be applied at every system startup.

## Set Permanent ulimit Values

To set the `ulimit` values permanently, follow these steps:

1. **Edit the `/etc/security/limits.conf` file:**

   Open the limits configuration file in a text editor with superuser privileges:

   ```bash
   sudo nano /etc/security/limits.conf
   ```

2. **Add the following lines to set the desired `ulimit` values:**

   ```bash
   * hard core unlimited
   * soft core unlimited
   * hard nofile 65535
   * soft nofile 65535
   * hard stack 64000
   * soft stack 64000
   ```

3. **Save the file and exit the editor:**

   Save the changes and exit the editor.

4. **Edit the `/etc/pam.d/common-session` file:**

   Open the PAM session configuration file in a text editor with superuser privileges:

   ```bash
   sudo nano /etc/pam.d/common-session
   ```

5. **Add the following line at the end of the file to ensure the limits are applied:**

   ```bash
   session required pam_limits.so
   ```

6. **Edit `/etc/profile` file:**

   Open the profile configuration file in a text editor with superuser privileges:

   ```bash
   sudo nano /etc/profile
   ```

7. **Add the following lines at the end of the file:**

   ```bash
   ulimit -c unlimited
   ulimit -n 65535
   ulimit -s 64000
   ```

8. **Reboot the system:**

   Reboot the system to apply the changes:

   ```bash
   sudo reboot
   ```

After the reboot, the `ulimit` settings will be applied permanently.

For more details, refer to the [Debian ulimit Configuration](https://wiki.debian.org/CommonTasks).

## Set Up Unattended Upgrades

To set up unattended upgrades on your Debian server, follow these steps:

1. **Install the unattended-upgrades package:**

   Install the necessary package to manage unattended upgrades:

   ```bash
   sudo apt-get update
   sudo apt-get install unattended-upgrades
   ```

2. **Enable unattended upgrades:**

   Enable the unattended upgrades service:

   ```bash
   sudo dpkg-reconfigure --priority=low unattended-upgrades
   ```

3. **Configure unattended upgrades:**

   Edit the configuration file to specify the types of upgrades you want to be applied automatically:

   ```bash
   sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
   ```

   Ensure the following lines are uncommented and set to your preferences:

   ```bash
   // Automatically upgrade packages from these origins
   Unattended-Upgrade::Allowed-Origins {
       "${distro_id}:${distro_codename}-security";
       "${distro_id}:${distro_codename}-updates";
       "${distro_id}:${distro_codename}-proposed-updates";
       "${distro_id}:${distro_codename}-backports";
   };

   // Do automatic removal of unused packages after upgrade
   Unattended-Upgrade::Remove-Unused-Dependencies "true";
   ```

4. **Enable automatic updates:**

   Create or edit the following file to enable automatic updates:

   ```bash
   sudo nano /etc/apt/apt.conf.d/20auto-upgrades
   ```

   Add the following lines:

   ```bash
   APT::Periodic::Update-Package-Lists "1";
   APT::Periodic::Unattended-Upgrade "1";
   ```

5. **Reboot the system:**

   Reboot the system to ensure all changes are applied:

   ```bash
   sudo reboot
   ```

   After these steps, your system will automatically handle updates and security patches.

   For more details, refer to the [Debian Unattended Upgrades Documentation](https://wiki.debian.org/UnattendedUpgrades).


## Install and Configure Fail2ban

   Fail2ban is a security tool that protects your server from brute-force attacks by monitoring log files and banning IPs that show malicious signs.

1. **Install Fail2ban**

   To install Fail2ban, use the following command:

   ```bash
   sudo apt-get update
   sudo apt-get install fail2ban
   ```

2. **Configure Fail2ban**

   The default configuration file is located at /etc/fail2ban/jail.conf. It is recommended to create a local copy for custom configurations:

   ```bash
   sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
   ```

   Edit the jail.local file to customize Fail2ban settings:

   ```bash
   sudo nano /etc/fail2ban/jail.local
   ```

3. **Basic Configuration**

   Modify the [DEFAULT] section to set the ban time, find time, and max retry settings:

   ```ini
   [DEFAULT]
   bantime  = 10m
   findtime  = 10m
   maxretry = 5
   ```

4. **Enable SSH Protection**

   Ensure the [sshd] section is enabled to protect SSH:

   ```ini
   [sshd]
   enabled = true
   port    = ssh
   logpath = %(sshd_log)s
   backend = %(sshd_backend)s
   ```
5. **Install the python3-systemd package:**

   Use the following command to install the python3-systemd package:

   ```bash
   sudo apt-get update
   sudo apt-get install python3-systemd
   ```
 
6. **Start and Enable Fail2ban**

   Start the Fail2ban service and enable it to run at boot:

   ```bash
   sudo systemctl start fail2ban
   sudo systemctl enable fail2ban
   ```

7. **Check Fail2ban Status**

   Check the status of Fail2ban to ensure it's running properly:

   ```bash
   sudo systemctl status fail2ban
   ```

8. **Verify Banned IPs**

   To verify which IPs are currently banned, use:

   ```bash
   sudo fail2ban-client status sshd
   ```
   This will show the status of the SSH jail and list any banned IPs.

For more details, refer to the [Fail2ban Documentation](https://www.fail2ban.org/wiki/index.php/Main_Page).

## Install and Configure Chrony


1. **Install Chrony:**

   ```bash
   sudo apt update
   sudo apt install chrony
   ```
2. **Configure Chrony:**
   
   Edit the Chrony configuration file:

   ```bash
   sudo nano /etc/chrony/chrony.conf
   ```

   Add or modify the server lines to include your preferred NTP servers. For example:

   ```conf
   server 0.debian.pool.ntp.org iburst
   server 1.debian.pool.ntp.org iburst
   server 2.debian.pool.ntp.org iburst
   server 3.debian.pool.ntp.org iburst
   ```

3. **Start and Enable Chrony:**

   ```bash
   sudo systemctl start chrony
   sudo systemctl enable chrony
   ```

4. **Verify Chrony is Working:**

   Check the status of the Chrony service:

   ```sh
   sudo systemctl status chrony
   ```

   Ensure it's active and running.

5. **Check Synchronization:**

   Verify that Chrony is synchronizing the time correctly:

   ```bash
   chronyc tracking
   chronyc sources -v
   ```

For further details, refer to the [official Debian Chrony Documentation](https://chrony-project.org/documentation.html).


