## Installing Suricata and Creating a Rule on Ubuntu 22.04: A Step-by-Step Tutorial

This tutorial will guide you through installing Suricata, a powerful open-source intrusion detection/prevention system (IDS/IPS), on Ubuntu 22.04 and creating a basic rule.

**Step 1: Update System Packages**

Before installing any new software, it's crucial to update your system's package list and upgrade existing packages. Open your terminal and run the following commands:

```bash
sudo apt update
sudo apt upgrade -y
```

**Step 2: Install Suricata**

The easiest way to install Suricata is from the official OISF (Open Information Security Foundation) repository. This ensures you get a stable and up-to-date version.

```bash
sudo add-apt-repository ppa:oisf/suricata-stable
sudo apt update
sudo apt install suricata -y
```

**Step 3: Configure Suricata**

After installation, you need to configure Suricata. The main configuration file is located at `/etc/suricata/suricata.yaml`. You can edit it using your preferred text editor (e.g., `nano`, `vim`).

```bash
sudo nano /etc/suricata/suricata.yaml
```

Here are some key settings you might want to adjust:

*   **`HOME_NET`**: This defines your internal network. By default, it's set to `192.168.0.0/16`. Change this to match your network. For example, if your network is `10.0.0.0/24`, change the line to `HOME_NET: "[10.0.0.0/24]"`.
*   **`EXTERNAL_NET`**: This defines the external network (usually the internet). It's typically set to `!$HOME_NET`.

**Step 4: Enable Network Interface**

To enable Suricata to capture network traffic, you need to specify the network interface. You can do this in the `suricata.yaml` file (as mentioned in the previous step) or by starting Suricata with the `-i` option. (To see your interface just do ip a and look for your IP)

**Step 5: Create a Custom Rule**

Now, let's create a simple rule to detect ICMP (ping) traffic. Suricata rules are stored in files within the `/var/lib/suricata/rules/` directory. You can create a new file for your custom rules.

```bash
sudo nano /var/lib/suricata/rules/local.rules
```

Add the following rule to the `local.rules` file:

```
alert icmp any any -> any any (msg:"ICMP Ping Detected"; sid:1000001; rev:1;)
```

This rule does the following:

*   **`alert`**: Specifies the action to take when the rule matches (generate an alert).
*   **`icmp`**: Specifies the protocol to inspect (ICMP).
*   **`any any -> any any`**: Specifies the source and destination IP addresses and ports (any).
*   **`msg:"ICMP Ping Detected"`**: Specifies the message to include in the alert.
*   **`sid:1000001`**: Specifies the rule ID (must be unique).
*   **`rev:1`**: Specifies the rule revision.

**Step 6: Test the Rule**

To test the rule, you need to restart Suricata to load the new rule.

```bash
sudo systemctl restart suricata
```

Now, try pinging your Ubuntu machine from another device on your network. You should see alerts in the Suricata log file, which is typically located at `/var/log/suricata/fast.log`.

```bash
tail -f /var/log/suricata/fast.log
```

**Step 7: Update Suricata Rules (Optional)**

You can update the Emerging Threats (ET) Open ruleset, which provides a comprehensive set of rules.

```bash
sudo apt install oinkmaster
sudo oinkmaster -o /var/lib/suricata/rules/ -C /etc/oinkmaster.conf
sudo suricata-update
```

**Step 8: More advanced Rules (Build at least 2)**

You can build your own rules, for example, this is one to detect a specific hex content in a TCP packet payload.
```
alert tcp any any -> any any (msg:"TCP Packet Containing Hexadecimal Sequence Detected"; content:"|41 42 43|"; sid:1000003; rev:1;)
```
Now, try to write rules that can detect port scanning! Just play around and give me a new couple rules. I want to see them in action in the video!

**Conclusion**

This tutorial has shown you how to install Suricata on Ubuntu 22.04, configure it, create a basic rule, and test it. You can now start exploring more advanced rule writing and use Suricata to enhance your network security.