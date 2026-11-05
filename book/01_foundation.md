# Module 1: The foundation

## Cloud setup & access

### Introduction

Before you deploy any code, you need a secure environment. This section uses an OVHcloud VPS, but the Linux commands and security concepts also apply to other providers and self-hosted servers.

By the end of this section, you will have a server running Ubuntu 24.04 LTS, accessed via SSH keys, with SSH password and keyboard-interactive authentication disabled.

> [!IMPORTANT]
> Throughout this guide, you will see text inside angle brackets like `<your_username>` or `<your_key_name>`. These are **placeholders**. You must replace them with your actual values and **remove the brackets** when running the commands.

### Order an OVHcloud VPS

Follow these steps to order the server used in the rest of this book. OVHcloud labels or available choices can vary with your account’s country, the selected region, and the VPS offer. Do not assume a plan name, price, payment option, or location shown in another country’s instructions will match your account.

1. Open the [OVHcloud website for your country or region](https://www.ovhcloud.com/) and click **Log in** to open the Control Panel. If you do not have an account, click **Create an account**. Enter your country of tax residence, currency, and Control Panel language, then click **Confirm**. Enter an email address you can access and create a unique password. Enter the one-time code sent to that email, choose the account type that matches your situation, and fill in the requested details. OVHcloud then logs you in. The account may require additional verification depending on your location and order. See [Creating an OVHcloud account](https://docs.ovhcloud.com/en/guides/account-and-service-management/account-information/ovhcloud-account-creation). If you already have an account, enter your NIC handle or primary email and password; complete any verification or multi-factor authentication prompt. See [OVHcloud account login](https://docs.ovhcloud.com/en/guides/account-and-service-management/account-information/ovhcloud-account-login).
2. Make sure the account has a default payment method. If you have ordered from OVHcloud before, one may already be saved. Otherwise, click your account name in the upper-right corner, click **My payment methods**, then click **Add a payment method** and follow the displayed form. Select the option to make it the default if shown. Available payment methods depend on your account and location; follow the options displayed for your account. See [Managing payment methods](https://docs.ovhcloud.com/en/guides/account-and-service-management/managing-billing-payments-and-services/manage-payment-methods).
3. In the Control Panel’s left navigation, click **Bare Metal Cloud**. Click **Order**, then click **VPS**. This opens the VPS order form. These menu labels and the order funnel are documented by OVHcloud; the linked support article is the US regional version, so its catalog and payment examples may differ from your local account. The order path does not include a DigitalOcean-style project creation step. See OVHcloud’s [How to order a VPS](https://support.us.ovhcloud.com/hc/en-us/articles/39499067423379-How-to-order-a-VPS) guide.
4. In the order form, choose a VPS configuration that meets the application’s needs. Compare the offered CPU, RAM, and storage values shown for each option. Plan names and resource bundles differ from DigitalOcean Droplet sizes, and the available range, locations, prices, and included storage depend on your account and offer. For this handbook’s learning environment, choose a modest option that meets the requirements of the later application sections; you can review [VPS resource upgrade options](https://docs.ovhcloud.com/en/guides/bare-metal-cloud/virtual-private-servers/upgrade-resources) before ordering if you are unsure.
5. Choose a data center/location offered for that configuration. Prefer one geographically near your users. I live in Morocco, so I would choose a nearby European data center if one is available for the selected offer. For this handbook, choose a standard data center rather than a **Local Zone** when both are offered; some VPS features described here, including automated backups and Edge Network Firewall availability, differ or are unavailable in Local Zones. Set the quantity to **1** unless you intentionally need more than one server.
6. In the image or operating-system choices, select **Ubuntu 24.04**. OVHcloud’s order guide lists this exact image. Select the plain Ubuntu image, not a preconfigured application image, so the later Linux commands match this handbook. If Ubuntu 24.04 is not offered for your selected configuration or location, stop and choose a compatible offer/location that lists it before continuing; do not silently select a different Ubuntu release.
7. Review optional features shown in the form, such as additional storage or a snapshot. Do not assume that an option is included: check its displayed cost and coverage before selecting it. OVHcloud documents one daily **Automated backup** as included with standard VPS orders, but that option is unavailable in Local Zones; a Premium Automated Backup option is paid. Check the current [VPS backup details](https://docs.ovhcloud.com/en/guides/bare-metal-cloud/virtual-private-servers/using-automated-backups-on-a-vps) and the order summary. Resource upgrades or additional disks may be offered later, depending on the VPS range and model.
8. Select the renewal frequency offered in the form. Review the renewal price and any one-time charges in the order summary. Confirm your billing address and payment method, then review the Terms of Service. When the selections and account information are correct, click **Continue →** to proceed with the order. Read the final order summary and follow the on-screen checkout instructions to submit it. Do not submit until the configuration, location, image, renewal frequency, and total are correct.
9. After OVHcloud processes the order, return to the Control Panel. In the left navigation, open **Bare Metal Cloud → Virtual private servers**, then select the new VPS. On the **Home** tab, review its status, operating system, location, and configuration. In the **IP** section, note the main public IPv4 address. Find the delivery email and open its secure link to retrieve the temporary Ubuntu password. OVHcloud uses an OS account named `ubuntu` for Ubuntu VPS installations. See [Getting started with a VPS](https://docs.ovhcloud.com/en/guides/bare-metal-cloud/virtual-private-servers/starting-with-a-vps) and [VPS management from the OVHcloud Control Panel](https://docs.ovhcloud.com/en/guides/bare-metal-cloud/virtual-private-servers/understand-vps-control-panel).

Before moving on, confirm that the VPS is shown in the Control Panel, its status indicates it is ready, the installed OS is Ubuntu 24.04, and you have the public IPv4 address and temporary password. These are account-side checks; the SSH connection and first-login steps are in **Finalize and connect** below.

### Configure authentication

This section walks through the OVHcloud-specific access steps in order:

1. Order an Ubuntu VPS and note the public IPv4 address, OS username, and temporary password from OVHcloud.
2. Create an SSH key on your computer.
3. Sign in once with the temporary password, change it, and reconnect.
4. Install your public key on the `ubuntu` account and confirm key-based login.
5. Create your named administrator account, then disable root and password SSH login.

The steps below explain each action. The first Linux login uses a **password**. After setup, use an **SSH key**.

OVHcloud creates a login account based on the selected operating system. For Ubuntu, the delivery email identifies the account as `ubuntu`. OVHcloud sends its temporary password in a secure link in the delivery email. This account can run administrative commands with `sudo`; OVHcloud documents that the root user is disabled by default. This differs from DigitalOcean's setup flow, where you can attach a key while creating the Droplet and connect as `root`.

After the initial login, use an SSH key for routine access. Passwords are vulnerable to brute-force attacks, whereas SSH keys are significantly more secure. Think of it like a real lock and key:

- The **public key** is the lock. You put this on the server. Anyone can see the lock, but that does not help them open the door.
- The **private key** is the physical key. You keep this on your computer. You never share it.

When you try to connect, the server checks if your "key" fits its "lock." If it does, you get in without a password.

![Diagram showing SSH key authentication flow between client and server](./images/1_1_10_how_ssh_keys_work.png)
_How SSH keys verify your identity._

The diagram above shows the conversation that happens between your computer and the server:

1. **SSH request**: Your computer contacts the server and says, "I want to connect using this specific key ID."
2. **Challenge**: The server is cautious. It sends back a random string of characters (called a nonce) and says, "Prove you own the private key by signing this random message."
3. **Send signature**: Your computer takes that random message and "signs" it using your private key. This creates a unique digital signature that gets sent back to the server.
4. **The result**: The server uses the public key (which you uploaded to it) to verify the signature. If the signature matches, the server knows you have the private key and grants access.

Generate the key on your local computer, not on the VPS. Use the [Ed25519](https://en.wikipedia.org/wiki/EdDSA#Ed25519) algorithm.

Create the local SSH directory if it does not exist:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
```

Now create the key pair. Replace `<your_key_name>` with a short name such as `ovh_vps`. Replace `<key_comment>` with a label that helps you recognize the key.

```bash
ssh-keygen -t ed25519 -f ~/.ssh/<your_key_name> -C "<key_comment>"
```

Here is what the arguments do:

- `-t ed25519`: Specifies the modern Ed25519 algorithm.
- `-f ~/.ssh/<your_key_name>`: Saves the key with a specific name so you don’t overwrite your default keys.
- `-C "<key_comment>"`: Adds a comment to help you identify the key.

The command asks for a passphrase. This protects the private key file. When SSH asks for the passphrase later, enter it on your computer. The passphrase is not sent to OVHcloud.

```text
Generating public/private ed25519 key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```

I strongly recommend adding a passphrase for your personal key.

> [!NOTE]
> If you need a key for a CI/CD pipeline later (to deploy code automatically), generate a separate key pair without a passphrase. Never remove the passphrase from your personal key.

You have generated the key pair. Now, copy the public key.

```bash
cat ~/.ssh/<your_key_name>.pub
```

The command displays your public key. You will install this key on the VPS after the first login. The public key can be shared; never copy or upload the private key.

### Finalize and connect

Before you connect, collect the public IPv4 address from the **IP** section or the delivery email. Get the OS username and temporary password from the delivery email. OVHcloud sends the password through a secure link. The username is `ubuntu` for Ubuntu.

Connect from your computer's terminal with the Ubuntu account:

```bash
ssh ubuntu@<your_vps_ip>
```

On the first connection, OpenSSH displays the server's host-key fingerprint and asks whether to trust it. Compare it with a fingerprint from a trusted source if OVHcloud provides one for your service. Otherwise, accept it only if you initiated this connection to your new VPS. OpenSSH stores the accepted fingerprint on your computer.

Enter the temporary password from the secure link. Change the password when prompted. OVHcloud then closes the session. Connect again with the same command and enter the new password.

From your local computer, install the public key on the `ubuntu` account:

```bash
ssh-copy-id -i ~/.ssh/<your_key_name>.pub ubuntu@<your_vps_ip>
```

Enter the current `ubuntu` account password if the command prompts for it.

If your computer does not have `ssh-copy-id`, append the public key from your local terminal with this command:

```bash
ssh ubuntu@<your_vps_ip> 'set -e; umask 077; mkdir -p ~/.ssh; cat >> ~/.ssh/authorized_keys; chmod 700 ~/.ssh; chmod 600 ~/.ssh/authorized_keys' < ~/.ssh/<your_key_name>.pub
```

Enter the current `ubuntu` account password if SSH prompts for it.

The public key goes in `/home/ubuntu/.ssh/authorized_keys`. Never copy the private key to the server. See OVHcloud's [SSH key guide](https://docs.ovhcloud.com/en/guides/bare-metal-cloud/dedicated-servers/creating-ssh-keys) for more options.

> [!NOTE]
> **OVHcloud's Control Panel key setup differs from DigitalOcean's.** The original DigitalOcean flow lets you attach an SSH key while creating a Droplet. OVHcloud's [VPS order guide](https://support.us.ovhcloud.com/hc/en-us/articles/39499067423379-How-to-order-a-VPS) does not document that step in the standard order flow; its [first-login guide](https://docs.ovhcloud.com/en/guides/bare-metal-cloud/virtual-private-servers/starting-with-a-vps) instead describes signing in with the temporary password and changing it. You then install the key on the running VPS as shown above. OVHcloud does let you store a public key in your account for use when you reinstall a VPS. Storing it there does **not** add the key to this already-installed VPS.

If you want OVHcloud to retain this public key for a future reinstallation, add it to your Control Panel:

1. Click your account name in the upper-right corner, then click **My offers and services**.
2. In **My services**, click **SSH keys**.
3. Click **Add an SSH key**, then select **Dedicated** from the menu. OVHcloud uses this key category for VPS reinstallations too.
4. Enter a label, paste the complete public key string shown by the `cat ~/.ssh/<your_key_name>.pub` command above, and click **Confirm**.

The stored key will then be available as a pre-install option when reinstalling a VPS. Reinstallation erases the VPS disks, so this is for a planned rebuild only; it is not required for the current server. See OVHcloud's [Control Panel SSH key instructions](https://docs.ovhcloud.com/en/guides/bare-metal-cloud/dedicated-servers/import-keys-control-panel) and [VPS management guide](https://docs.ovhcloud.com/en/guides/bare-metal-cloud/virtual-private-servers/understand-vps-control-panel).

Confirm that key-based login works before continuing:

```bash
ssh -i ~/.ssh/<your_key_name> ubuntu@<your_vps_ip>
```

If key authentication works, SSH opens a shell on the VPS without asking for the server account's password. If SSH still asks for that password, check that you installed the `.pub` file on the `ubuntu` account and that the private key path is correct.

Once connected, you will see a welcome message similar to this. The exact banner and any reboot notice vary by image and update state:

```text
The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

ubuntu@<your_hostname>:~$
```

Update your VM packages by running the following commands:

```bash
sudo apt update
sudo apt upgrade
```

> [!IMPORTANT]
> If you see a configuration screen during the update, select **keep the local version currently installed** unless you have reviewed the proposed replacement. This avoids unexpectedly replacing the SSH configuration you are using to connect.

![Screenshot of dpkg configuration prompt asking to keep or replace sshd_config](./images/1_1_18_configure_open_ssh_server_warning.png)
_Handle the configuration conflict._

Some package upgrades require a reboot, while others do not. Check whether Ubuntu marked one as required:

```bash
if [ -f /var/run/reboot-required ]; then
  echo "A reboot is required."
else
  echo "No reboot is currently required."
fi
```

If Ubuntu reports that a reboot is required, reboot now. Keep the current session open until you have confirmed that you can reconnect after the reboot.

```bash
sudo reboot
```

The connection will close, but you can connect via SSH to the server again after it reboots. If you want to exit the VM, type exit and hit `Enter` or hit `Ctrl+D`.

### Create a non-root user and configure SSH access

Running everything as **root** is dangerous. If you make a mistake, it might be unrecoverable. Also, if an attacker finds a vulnerability in your app while it is running as root, they gain full control of your server.

Create a new user to act as a security barrier.

```bash
sudo adduser <your_username>
```

It will ask you to set a password and fill in some details. You can skip the details by pressing `Enter`.

```text
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for <your_username>
Enter the new value, or press ENTER for the default
Full Name []: <your_fullname>
Room Number []:
Work Phone []:
Home Phone []:
Other []:
Is the information correct? [Y/n] y
info: Adding new user `<your_username>' to supplemental / extra groups `users' ...
info: Adding user `<your_username>' to group `users' ...
```

Give the new user administrative privileges ([sudo](https://www.sudo.ws/news/)).

```bash
sudo usermod -aG sudo <your_username>
```

Now, log out of the server (`exit` or `Ctrl+D`) and attempt to log in as your new user.

```bash
ssh -i ~/.ssh/<your_key_name> <your_username>@<your_vps_ip>
```

You will get an error because the SSH key is authorized for the existing `ubuntu` account, not the new user. Copy the authorized key to the new account.

```text
<your_username>@<your_vps_ip>: Permission denied (publickey).
```

Log back in as the existing `ubuntu` account, which has administrative privileges through `sudo`.

```bash
ssh -i ~/.ssh/<your_key_name> ubuntu@<your_vps_ip>
```

Run these commands to copy the authorized keys to the new user. Set the correct ownership and permissions for the copied key file.

```bash
sudo install -d -m 700 -o <your_username> -g <your_username> /home/<your_username>/.ssh
sudo cp /home/ubuntu/.ssh/authorized_keys /home/<your_username>/.ssh/authorized_keys
sudo chown <your_username>:<your_username> /home/<your_username>/.ssh/authorized_keys
sudo chmod 600 /home/<your_username>/.ssh/authorized_keys
```

Now, exit and try to log in as your new user again.

```bash
ssh -i ~/.ssh/<your_key_name> <your_username>@<your_vps_ip>
```

It should work now. You will see a prompt like this:

```text
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

<your_username>@<your_hostname>:~$
```

Before you move on to the next step, you must verify that your new user can actually perform administrative tasks. If you disable root login and find out your user cannot use sudo, you will be locked out of your own server.

```bash
sudo whoami
```

Enter your new user's password if `sudo` prompts for it. The command should print `root`. This confirms the account can run a command with administrator privileges. If you see an error saying your user is "not in the sudoers file", **STOP**. Do not continue until you fix the user permissions.

From now on, use your named account for administration. Keep the original `ubuntu` account available until you have confirmed the new account works.

### Disable root SSH access

Now that your new user is set up, you should disable direct login for the root account. This reduces your attack surface.

Create a dedicated SSH configuration file so the settings are explicit and easy to review. Ubuntu may load files in `/etc/ssh/sshd_config.d/` before the rest of the main file, so inspect the existing configuration rather than assuming the last line wins.

```bash
sudo nano /etc/ssh/sshd_config.d/00-local-hardening.conf
```

This is a new file, so it will normally be blank. Type or paste the following four lines into the editor; you do not need to search for `PermitRootLogin`. If you already pressed `Ctrl+W` and nano reports that the text was not found, press `Ctrl+C` to cancel the search and return to the blank editor. The message only means the setting was not already in this file.

These settings reject root SSH login and disable SSH password and keyboard-interactive authentication. Public-key authentication remains enabled by default in OpenSSH; it is stated explicitly here so the intended access method is clear:

```text
PermitRootLogin no
PasswordAuthentication no
KbdInteractiveAuthentication no
PubkeyAuthentication yes
```

Save the file by pressing `Ctrl+O` (to write out/save), then exit nano with `Ctrl+X`. Check the SSH configuration syntax before applying it. Keep your current SSH session open while testing a second connection.

```bash
sudo sshd -t
sudo sshd -T | grep -E '^(passwordauthentication|kbdinteractiveauthentication|pubkeyauthentication|permitrootlogin)'
sudo systemctl restart ssh
```

Before closing your current session, open a second terminal and confirm that your named user can still connect with the SSH key:

```bash
ssh -i ~/.ssh/<your_key_name> <your_username>@<your_vps_ip>
```

Now, exit the server.

```bash
exit
```

Try to log in as `root`.

```bash
ssh -i ~/.ssh/<your_key_name> root@<your_vps_ip>
```

You should see a message indicating that authentication was rejected. Depending on your local SSH configuration, you will see one of these two errors:

```text
# Scenario A: Standard rejection
Permission denied (publickey).

# Scenario B: If you have many SSH keys on your computer
Received disconnect from <your_vps_ip> port 22:2: Too many authentication failures
```

A `Permission denied` response confirms the root login was rejected. The “Too many authentication failures” response can also mean your SSH client offered too many keys; if you see it, retry with `IdentitiesOnly yes` to send only the specified key.

> [!NOTE]
> OVHcloud's initial VPS access uses an OS-specific account and a temporary password. After installing and confirming your SSH key, explicitly disable password authentication.
>
> Ubuntu's SSH configuration commonly includes `/etc/ssh/sshd_config.d/*.conf` near the top of `sshd_config`. SSH uses the first value it reads for a setting, so an earlier included file can take precedence over a later value in the main file. The `00-local-hardening.conf` file created above is read before later numbered files on systems that use this include pattern.
>
> Image defaults vary. Check the effective configuration with:
>
> ```bash
> sudo sshd -T | grep -E '^(passwordauthentication|kbdinteractiveauthentication|pubkeyauthentication|permitrootlogin)'
> ```
>
> Confirm the output says `passwordauthentication no`, `kbdinteractiveauthentication no`, `pubkeyauthentication yes`, and `permitrootlogin no` before you close your current session.
### Simplify SSH access

Typing the full SSH command with the key path and IP address every time is tedious. You can configure your local SSH client to remember these details for you.

On your **local computer** (not the server), open or create the SSH config file:

```bash
nano ~/.ssh/config
```

Add the following configuration block at the bottom of the file:

```text
Host my-website
    HostName <your_vps_ip>
    User <your_username>
    IdentityFile ~/.ssh/<your_key_name>
    IdentitiesOnly yes
```

Here is what these lines do:

- **Host:** This is the shortcut name you want to use.
- **HostName:** The actual IP address of your server.
- **User:** The username you created on the server.
- **IdentityFile:** The path to the specific private key for this server.
- **IdentitiesOnly:** This tells SSH to **only** use the specific key file listed here. This prevents the "Too many authentication failures" error if you have many keys on your computer.

Save the file and exit. Now, instead of typing this long command:

```bash
ssh -i ~/.ssh/<your_key_name> <your_username>@<your_vps_ip>
```

You can simply type:

```bash
ssh my-website
```

Your computer will automatically look up the IP, user, and key, and log you straight in.

### What is next?

You have configured a clean Ubuntu server and secured its administrative login with SSH keys. Public network exposure depends on which services are listening and on firewall rules; the next section applies an explicit host firewall policy.

In the next subchapter, **The firewall strategy**, we will lock down the network. You will learn how to configure [UFW](https://documentation.ubuntu.com/server/how-to/security/firewalls/) to block unwanted traffic, set up [Fail2Ban](https://github.com/fail2ban/fail2ban) to stop brute-force attacks, and use OVHcloud KVM or rescue mode if you ever lose access.

## The firewall strategy

### Introduction

Right now, SSH is reachable from the internet so that you can administer the server. Other ports are reachable only if a service is listening and network rules permit the connection. The firewall gives you an explicit policy for restricting incoming traffic.

In this section, you will configure a firewall to block incoming traffic by default, allowing only the connections you explicitly need, using **UFW** (Uncomplicated Firewall). You will also configure **Fail2Ban** to temporarily ban IP addresses that generate repeated failed SSH authentication attempts. Fail2Ban is an additional control; it does not replace SSH keys or UFW.

This section configures UFW inside Ubuntu. OVHcloud also documents an optional [Edge Network Firewall](https://docs.ovhcloud.com/en/guides/bare-metal-cloud/dedicated-servers/firewall-network) for supported public IPs. OVHcloud describes it as a stateless, IPv4-only network layer that is automatically applied when a DDoS attack is detected; it does not open ports on the VPS or replace its operating-system firewall. Availability can be limited for some offers, including Eco products. This handbook does not configure it. For this learning setup, use UFW as described below. If you later configure the Edge Network Firewall, follow its separate rule-order and deny-rule requirements; do not copy UFW rules into it as though it were the same kind of firewall.

### Configure the firewall

Ubuntu comes with a tool called **UFW** (Uncomplicated Firewall). It is designed to be easy to use while providing robust security.

Check the current status of your firewall.

```bash
sudo ufw status
```

On a fresh Ubuntu image it will commonly say `Status: inactive`, but check the result rather than assuming. If it is already active, inspect the existing rules and make sure they allow your SSH port before changing or enabling anything. For an inactive firewall, add the SSH allow rule before enabling UFW.

The commands below assume SSH is using its default port, TCP `22`. If you changed the SSH port, add a UFW rule for that actual port before enabling the firewall; the `OpenSSH` profile may still refer to port `22`.

> [!IMPORTANT]
> If you enable the firewall without a rule allowing your SSH port, your SSH connection may be blocked. Keep your existing session open and verify a second connection after enabling UFW.

```bash
sudo ufw allow OpenSSH
```

Now that SSH is allowed, you can safely enable the firewall.

```bash
sudo ufw enable
```

The system will warn you that this command might disrupt existing SSH connections. Type `y` and hit `Enter`.

```plaintext
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
```

Verify the status again to see the active rules.

```bash
sudo ufw status verbose
```

The output gives you a detailed list. Pay close attention to the Default policy and the ALLOW IN rules.

```plaintext
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                             Action      From
--                             ------      ----
22/tcp (OpenSSH)               ALLOW IN    Anywhere
22/tcp (OpenSSH (v6))          ALLOW IN    Anywhere (v6)
```

The exact rows depend on the OpenSSH profile and whether IPv6 is enabled. Confirm that the active output includes an allow rule for the port your SSH server actually uses.

Here is how to interpret this output:

- **Default: deny (incoming)**: New incoming connections are denied unless an allow rule permits them. This does not stop outgoing connections or change which services are listening.
- **ALLOW IN**: These are your exceptions. Right now, you are only allowing traffic for SSH (port 22).

> [!NOTE]
> Later in the guide, when you install the web server ([Nginx](https://nginx.org/)), you will come back here to open ports [80 (HTTP)](https://en.wikipedia.org/wiki/Port_%28computer_networking%29#Common_port_numbers) and [443 (HTTPS)](https://en.wikipedia.org/wiki/Port_%28computer_networking%29#Common_port_numbers). For now, keeping them closed is safer.

### Protect SSH with Fail2Ban

SSH services on public IP addresses can receive automated login attempts. With password authentication disabled, those attempts do not let an attacker guess a password to log in, but repeated failed authentication attempts can still appear in the system journal. On Ubuntu, check recent SSH service messages with:

```bash
sudo journalctl -u ssh --since today
```

The exact messages and whether `/var/log/auth.log` exists depend on the image's logging configuration. No output for this time range does not by itself indicate a problem.

You can install [Fail2Ban](https://github.com/fail2ban/fail2ban) to monitor failed SSH authentication and temporarily ban an IP address after repeated failures. A ban applies to that source IP and jail on this VPS; it does not stop attempts from other IP addresses, and it can also affect a legitimate user who repeatedly fails to authenticate.

![How Fail2Ban turns log entries into firewall rules](./images/1_2_1_fail2ban_firewall_rules_illustration.png)
_How Fail2Ban turns log entries into firewall rules._

Fail2Ban is available in Ubuntu's default repositories. Install it with this command:

```bash
sudo apt install fail2ban -y
```

The upstream Fail2Ban default ban time is 10 minutes ([default configuration](https://github.com/fail2ban/fail2ban/blob/master/config/jail.conf)). This chapter uses a one-day ban as an example policy; choose a duration appropriate for your server and users. To set it for the SSH jail, create a local configuration file.

> [!Important]
> Never edit the `.conf` file directly, as system updates will overwrite it. Always use a `.local` file for your customizations.

Create the file using nano.

```bash
sudo nano /etc/fail2ban/jail.local
```

Paste the following configuration:

```ini
[sshd]
enabled = true
bantime = 1d
maxretry = 5
findtime = 10m
```

Here is what these settings do:

- `enabled`: Turns on protection for SSH.
- `bantime`: Bans the IP for 1 day.
- `maxretry`: Sets the number of failures allowed within the configured window before the jail bans the source IP.
- `findtime`: The window of time (10 minutes) in which those 5 failures must occur to trigger a ban.

Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`). Enable Fail2Ban now and at startup so the service reads the configuration:

```bash
sudo systemctl enable --now fail2ban
```

You can verify your new ban time (86400 seconds = 24 hours) with this command:

```bash
sudo fail2ban-client get sshd bantime
```

To see if the tool is working, check the status of the SSH jail.

```bash
sudo fail2ban-client status sshd
```

The exact output depends on the installed Fail2Ban package and its log backend. Check that the `sshd` jail is listed and active. A ban count of zero is normal if no source has reached the configured retry threshold; do not interpret it as a failed installation. Do not test the ban from your only administrator IP, since you could temporarily block your own access.

### Recover access to the VPS

What happens if you make a mistake? For example, if you accidentally run `ufw enable` before allowing OpenSSH, or if you configure SSH keys incorrectly and lose access to your private key.

OVHcloud provides a browser-based KVM console and a separate rescue mode. The KVM console opens a console session to the VPS, but OVHcloud states that it is not a workaround when the operating system itself is inaccessible. Use [rescue mode](https://docs.ovhcloud.com/en/guides/bare-metal-cloud/virtual-private-servers/rescue) for operating-system recovery.

To open KVM, go to **Bare Metal Cloud → Virtual private servers**, select your VPS, then on the **General information** tab open the **...** menu next to the VPS name and select **KVM**. See OVHcloud's [KVM console guide](https://docs.ovhcloud.com/en/guides/bare-metal-cloud/virtual-private-servers/using-kvm-for-vps) for the current control-panel steps. Log in with an active Linux account if prompted; the console keyboard layout may differ from your computer's.

If the OS is still running and you can log in through KVM, you can repair a firewall rule. For example, if you forgot to allow SSH:

```bash
sudo ufw allow OpenSSH
```

After fixing the issue, close the browser window and try SSH from your terminal again.

```bash
ssh my-website
```

If you cannot log in to the installed OS, use OVHcloud's [VPS rescue mode](https://docs.ovhcloud.com/en/guides/bare-metal-cloud/virtual-private-servers/rescue). On the **Home** tab, open the **...** menu next to **Boot**, choose **Reboot in rescue mode**, and confirm. This reboots the VPS into a temporary system and interrupts its running services.

Wait for OVHcloud's email with the rescue SSH credentials. The email also appears in the control panel. Click the name in the top-right corner, then open **Service emails**. If the email does not arrive, check that you are listed as the technical contact for the VPS. In rescue mode, log in as `root` with the temporary password from that email:

```bash
ssh root@<your_vps_ip>
```

The rescue system uses a different SSH host key, so your SSH client may warn that the fingerprint changed. Follow the rescue guide to remove or temporarily disable the old entry in your local `~/.ssh/known_hosts` file. If you comment out the old entry, restore it before rebooting the VPS. The guide also explains how to use `lsblk` to identify and mount the installed system partition. Do not assume the partition name from an example. After repairing the system, reboot the VPS in normal mode from the control panel.

### What is next?

Your foundation is solid. You have a secure server that blocks unauthorized traffic and automatically bans attackers.

In the next module, **Module 2: The Application**, we will shift focus from security to software. You will learn how to prepare the environment by installing **Python 3.13** and **Node.js**, setting up [Swap memory](https://en.wikipedia.org/wiki/Memory_paging) to handle builds on a low-RAM server, and configuring [Gunicorn](https://gunicorn.org/) and [Supervisor](https://supervisord.org/) to keep your application running reliably in the background.
