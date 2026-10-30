# NETWORKWALKS-B084-WK1-PM1-CYBERSECURITY-LAB-SETUP
setting up my cyber security lab
# NETWORKWALKS-B084-WK1-PM1-CYBERSECURITY-LAB-SETUP
setting up my cyber security lab
# 🐉 Kali Linux Attacker Lab on VirtualBox (Windows)

![VirtualBox](https://img.shields.io/badge/VirtualBox-7.x%20latest-183A61?logo=virtualbox&logoColor=white)
![Kali](https://img.shields.io/badge/Kali-Pre--built%20VM-557C94?logo=kalilinux&logoColor=white)
![Network](https://img.shields.io/badge/NAT%20Network-10.0.0.0%2F24-orange)
![IP](https://img.shields.io/badge/Kali%20IP-10.0.0.2%2F24-red)

## 🎯 Target Setup

| 🧩 Requirement | 🎨 Value |
|---|---|
| 🟦 Hypervisor | VirtualBox (latest 7.x) |
| 🐉 Attacker machine | Kali Linux (pre-built VM) |
| 🌐 Network type | **NAT Network** |
| 📡 Subnet | `10.0.0.0/24` |
| 📍 Kali IP | `10.0.0.2/24` |
| 📋 Clipboard + Drag & Drop | Bidirectional |
| 📁 Shared folder | Host `downloads` → Kali `/downloads` |
| 🌍 Internet | Full access |

```
   🖥️ Windows Host
        │
   ┌────▼───────────── NAT Network 10.0.0.0/24 ──────────────┐
   │  🚪 Gateway 10.0.0.1                                     │
   │  🐉 Kali  10.0.0.2   (+ future target VMs 10.0.0.x)      │
   └──────────────────────────┬───────────────────────────────┘
                              ▼
                         🌍 Internet
```

---

## 1️⃣ Enable Virtualization (BIOS/UEFI)

1. 🔄 Reboot → enter BIOS (`F2` / `F10` / `Del` / `Esc`)
2. 🔛 Enable **Intel VT-x** or **AMD-V / SVM**
3. 💾 Save & exit

Verify: `Task Manager → Performance → CPU → Virtualization: Enabled` ✔️

---

## 2️⃣ Download

| 🎯 What | 🔗 Link |
|---|---|
| 🟦 VirtualBox (Windows hosts) + Extension Pack | https://www.virtualbox.org/wiki/Downloads |
| 🐉 Kali pre-built VM (**Virtual Machines → VirtualBox**, `.7z`) | https://www.kali.org/get-kali/#kali-virtual-machines |
| 🗜️ 7-Zip | https://www.7-zip.org/ |

🔐 Verify checksum (compare with the SHA256 on the Kali page):

```powershell
Get-FileHash .\kali-linux-*-virtualbox-amd64.7z -Algorithm SHA256
```

---

## 3️⃣ Install VirtualBox

1. ▶️ Run installer → accept defaults
2. ⚠️ Accept the temporary network disconnect warning
3. ✅ Allow driver installs
4. 🧩 Double-click the **Extension Pack** (`.vbox-extpack`) → *Install*

> 💡 If VirtualBox is very slow or VT-x errors appear, see **Troubleshooting** at the bottom (Hyper-V conflict).

---

## 4️⃣ Import Kali

1. 📦 Extract the `.7z` with 7-Zip
2. 🖱️ Double-click the `.vbox` file (or `Machine → Add…`)
3. ❌ **Do not start it yet**, configure first

---

## 5️⃣ Create the NAT Network (10.0.0.0/24)

`File → Tools → Network Manager → NAT Networks` tab → **➕ Create**

| ⚙️ Field | 🎨 Value |
|---|---|
| Name | `LabNAT` |
| IPv4 Prefix | `10.0.0.0/24` |
| Enable DHCP | ☐ **Unchecked** (Kali uses a static IP) |

✅ Click **Apply**.

<details>
<summary>⌨️ CLI alternative (click to expand)</summary>

```powershell
cd "C:\Program Files\Oracle\VirtualBox"
.\VBoxManage.exe natnetwork add --netname LabNAT --network "10.0.0.0/24" --enable --dhcp off
```

</details>

---

## 6️⃣ Configure the Kali VM

Select Kali → **Settings**

### 🌐 Network
- **Adapter 1** → ☑️ Enable Network Adapter
- **Attached to:** `NAT Network`
- **Name:** `LabNAT`
- Advanced → Adapter Type: `Intel PRO/1000 MT Desktop` (or leave default) · Cable Connected ☑️

### 📋 Clipboard & Drag'n'Drop
`General → Advanced`
- **Shared Clipboard:** `Bidirectional`
- **Drag'n'Drop:** `Bidirectional`

### 📁 Shared Folder
`Shared Folders → ➕ Add`

| ⚙️ Field | 🎨 Value |
|---|---|
| Folder Path | your host folder, e.g. `C:\downloads` (create it if missing) |
| Folder Name | `downloads` |
| Mount point | `/downloads` |
| ☑️ Read-only | off (so you can write) |
| ☑️ Auto-mount | on |
| ☑️ Make Permanent | on |

### 🖥️ Display & CPU (recommended)
- `Display → Video Memory:` `128 MB`
- `System → Processor:` `2–4 CPUs`
- `System → Base Memory:` `4096 MB` (min 2048)

Click **OK**.

---

## 7️⃣ First Boot & Login

▶️ Start the VM.

🔑 Default credentials: **`kali` / `kali`**

🔒 Change the password right away:

```bash
passwd
```

---

## 8️⃣ Set Static IP `10.0.0.2/24` + Internet

Find the connection name:

```bash
nmcli con show
```

Apply the settings (replace `"Wired connection 1"` with the name you saw):

```bash
sudo nmcli con mod "Wired connection 1" \
  ipv4.method manual \
  ipv4.addresses 10.0.0.2/24 \
  ipv4.gateway 10.0.0.1 \
  ipv4.dns "1.1.1.1 8.8.8.8"

sudo nmcli con down "Wired connection 1"; sudo nmcli con up "Wired connection 1"
```

✅ Verify:

```bash
ip a show eth0          # should show 10.0.0.2/24
ip route                # default via 10.0.0.1
ping -c 3 10.0.0.1      # gateway
ping -c 3 8.8.8.8       # internet (IP)
ping -c 3 kali.org      # internet + DNS
```

> 🧠 **Why `.1` as gateway?** VirtualBox's NAT Network uses the first address of the subnet as its gateway/router. NAT gives Kali full outbound internet through the Windows host.
>
> ⚠️ Note: in VirtualBox NAT networks, `.2` is also the alias for the host's loopback. Using it for Kali is fine, you just can't reach the host via `10.0.0.2` (it's Kali itself). Use the host's real LAN IP instead.

---

## 9️⃣ Guest Additions (needed for clipboard, drag & drop, shared folders)

Check if already running:

```bash
systemctl status virtualbox-guest-utils --no-pager | head -5
```

If missing/not working:

```bash
sudo apt update
sudo apt install -y virtualbox-guest-x11 virtualbox-guest-utils
sudo reboot
```

### 📁 Shared folder permissions

```bash
sudo usermod -aG vboxsf $USER
sudo reboot
```

Test after reboot:

```bash
ls -la /downloads
touch /downloads/test-from-kali.txt     # should appear on Windows
```

> 🛟 If `/downloads` isn't mounted, mount manually:
> ```bash
> sudo mkdir -p /downloads
> sudo mount -t vboxsf downloads /downloads
> ```

---

## 🔟 Final Checks ✅

- [ ] 🟦 VirtualBox 7.x + Extension Pack installed
- [ ] 🌐 NAT Network `LabNAT` = `10.0.0.0/24`
- [ ] 🐉 Kali attached to `LabNAT`
- [ ] 📍 `ip a` shows `10.0.0.2/24`
- [ ] 🌍 `ping kali.org` works
- [ ] 📋 Copy/paste works both ways
- [ ] 🖱️ Drag & drop a file host ↔ Kali works
- [ ] 📁 `/downloads` shows the host folder contents
- [ ] 📸 Snapshot taken: `Machine → Take Snapshot` → `Clean-Lab`

---

## 🛠️ Troubleshooting

| 😵 Problem | 💡 Fix |
|---|---|
| ❌ No internet in Kali | Check gateway `10.0.0.1`, DNS set, adapter = NAT Network, cable connected |
| 🔌 NAT Network not in dropdown | Create it in Network Manager (Step 5) and ensure it's **enabled** |
| 📋 Clipboard/drag-drop dead | Guest Additions missing (Step 9), set both to Bidirectional, reboot |
| 📁 `/downloads` permission denied | `sudo usermod -aG vboxsf $USER` then reboot |
| 🖤 Black screen | `Display → Graphics Controller: VMSVGA`, VRAM 128 MB |
| 🖱️ Mouse captured | Press **Right Ctrl** |
| 💥 `VERR_VMX_NO_VMX` / very slow | Hyper-V conflict (below) |



---


