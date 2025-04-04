# Raspberry Pi Print Server Setup with Ansible

This project provides an Ansible-based setup for turning a Raspberry Pi into a network print server for an HP LaserJet P1005/P1006 (or similar USB-only printer). It installs and configures CUPS, Avahi, and the appropriate drivers (assuming the HPLIP proprietary plugin has already been installed).

---

## System Requirements

- **Raspberry Pi Model:** Raspberry Pi 1 Model B (or newer)
- **Printer:** HP LaserJet P1005 (or P1006) — USB-only, requires HPLIP plugin
- **Operating System:** Raspberry Pi OS (Bookworm or newer)
- **Connectivity:** SSH access to the Raspberry Pi with certificate-based login
- **Local Machine:** Ansible installed (v2.10+ recommended)
- **Optional:** Basic familiarity with command-line and SSH

---

## Architecture Overview

```mermaid
flowchart LR
    subgraph LocalMachine[Your Computer (Ansible Controller)]
        A[Ansible + SSH]
    end

    subgraph RaspberryPi[Raspberry Pi Print Server]
        B[CUPS]
        C[Avahi (Bonjour)]
        D[HPLIP + Plugin]
        B --> USBConnection
    end

    A -->|SSH + Ansible| B
    USBConnection[USB Connection] --> E[HP LaserJet P1005]

    subgraph NetworkClients[Client Devices]
        F[macOS / Windows / Linux]
    end

    RaspberryPi -->|Bonjour/AirPrint| F
```

---

## Features

- Installs and configures CUPS with network access
- Adds USB-connected HP printer using the correct driver
- Broadcasts printer over the network using Avahi (Bonjour/AirPrint)
- Easily re-runnable with Ansible
- Designed to be generic and reusable via external variables
- Skips manual plugin reminder with a toggle flag

---

## Prerequisites

### 1. Setting Up SSH Key Access

To avoid using passwords and to enable Ansible access:

1. On your **local machine** (if not already created):

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/<your_private_key>
```

2. Copy the public key to your Raspberry Pi:

```bash
ssh-copy-id -i ~/.ssh/<your_private_key>.pub <your_user>@<your_host>
```

> You can also manually copy `~/.ssh/<your_private_key>.pub` contents into the Pi’s `~/.ssh/authorized_keys`.

Test it:

```bash
ssh -i ~/.ssh/<your_private_key> <your_user>@<your_host>
```

---

### 2. Manually Installing HPLIP Plugin (Required for HP P1005/P1006)

The HP P1005 and P1006 require a proprietary plugin not available via APT. Do this **once manually**, unless you're using the `skip_plugin_check` flag:

1. SSH into your Raspberry Pi:

```bash
ssh -i ~/.ssh/<your_private_key> <your_user>@<your_host>
```

2. Install required build dependencies:

```bash
sudo apt install build-essential libtool m4 python3-dev libusb-1.0-0-dev libjpeg-dev libcups2-dev libsnmp-dev libssl-dev libavahi-core-dev
```

3. Download and install latest HPLIP:

```bash
wget https://downloads.sourceforge.net/project/hplip/hplip/3.23.12/hplip-3.23.12.run
chmod +x hplip-3.23.12.run
./hplip-3.23.12.run
```

4. Follow the prompts, accept the license, and install the plugin.

5. Reboot the Raspberry Pi and run the Ansible playbook again.

---

## Folder Structure

```
rpi-print/
├── setup_print_server.yml        # Main Ansible playbook
├── inventory_template.yml        # Generic inventory with placeholders
├── .gitignore                    # Ignore sensitive files like vars.yml (if used)
└── README.md                     # This file
```

---

## Usage

### Option 1: Inject Variables via Environment

```bash
ANSIBLE_HOST_KEY_CHECKING=False \
ansible-playbook -i inventory_template.yml setup_print_server.yml \
  -e inventory_host=<your_host> \
  -e inventory_user=<your_user> \
  -e inventory_key=~/.ssh/<your_private_key> \
  -e skip_plugin_check=false
```

### Option 2: Inject Variables Manually via CLI

```bash
ansible-playbook -i inventory_template.yml setup_print_server.yml \
  -e "inventory_host=<your_host> inventory_user=<your_user> inventory_key=~/.ssh/<your_private_key> skip_plugin_check=false"
```

### Option 3: Use a Custom `vars.yml` File (Optional)

Create a `vars.yml` file in the root of the project with the following structure:

```yaml
inventory_host: <your_host>
inventory_user: <your_user>
inventory_key: ~/.ssh/<your_private_key>
skip_plugin_check: false
```

Then run:

```bash
ansible-playbook -i inventory_template.yml setup_print_server.yml -e @vars.yml
```

> Note: If you create `vars.yml`, consider adding it to `.gitignore` to avoid committing secrets.

---

## Troubleshooting

- If your printer is not detected, make sure it’s plugged in and powered on before running the playbook
- Ensure the proprietary plugin is installed via `hp-setup`
- Reboot the Pi after installing the plugin if the printer is still not detected
- If plugin reminder appears, you can set `skip_plugin_check: true`

---

## Future Improvements

- Make plugin install automatable with license acceptance flag (if supported)
- Add role structure for modular reuse
- Support other USB printers via dynamic detection

---

## License

This project is licensed under the MIT License.
