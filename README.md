# ASL/DVSwitch Bootable Mirror

Create a verified, bootable backup of a Raspberry Pi system disk on another physical drive.

**Current version:** v1.0

## 📖 What this does

`asl-bootable-mirror` copies a running Raspberry Pi system to another disk, such as:

- NVMe to USB SSD
- USB SSD back to NVMe
- SD card to USB SSD
- SD card to NVMe on a Raspberry Pi 5

It creates a real bootable filesystem mirror, not a compressed image file. If the main system drive fails, the destination can be connected or selected in the Raspberry Pi boot order and used as a replacement system disk.

The program was built for AllStarLink and DVSwitch nodes, but it can also mirror another compatible Raspberry Pi Debian-based installation that uses the supported disk layout.

> [!WARNING]
> Initializing or rebuilding a destination erases every file and partition on that destination disk. Read the displayed model, serial number, device path, and confirmation text carefully.

## ✅ Main features

- Automatically detects and protects the active system disk.
- Shows only other physical disks as destination choices.
- Uses the destination model, serial number, WWN, size, and transport for identification.
- Requires an exact typed confirmation before erasing a disk.
- Creates fresh FAT32 boot and ext4 root filesystems.
- Gives the destination unique PARTUUID values.
- Updates only the destination copy of `/etc/fstab` and `cmdline.txt`.
- Supports a destination that is larger or smaller than the source when it has enough usable capacity.
- Uses the full remaining destination space for the root partition.
- Performs an initial copy while the node remains online.
- Can briefly pause selected services for a consistent final synchronization.
- Keeps SSH and networking running during the service pause.
- Automatically recommends AllStarLink, DVSwitch, Mosquitto, Pi Monitor, PCP, and common database services when present.
- Restarts every paused service before completing.
- Compares the source and destination contents.
- Creates and verifies SHA-256 manifests by reading the destination files back.
- Runs final FAT32 and ext4 filesystem checks.
- Detects new USB, UAS, timeout, reset, and storage errors during critical operations.
- Recognizes an existing mirror for later updates or verification.
- Can resume after formatting when copying was intentionally postponed.
- Recreates `/var/swap` correctly on the clone's first boot instead of making an unusable sparse copy.
- Does not change Raspberry Pi EEPROM settings or boot order.

## 🔄 How it works

The program uses seven stages:

1. **Preflight** — identifies the active source disk, available destination disks, disk capacity, and existing destination state.
2. **Destination initialization** — erases and partitions a new or rebuilt destination after exact confirmation.
3. **Initialization checkpoint** — checks the new partition table and filesystems before copying begins.
4. **Initial filesystem copy** — copies the root and boot filesystems with `rsync`.
5. **Consistent final synchronization** — optionally pauses services that can change stored data, then copies the final changes.
6. **Content and boot verification** — checks the Raspberry Pi boot structure, compares source and destination, and performs SHA-256 readback verification.
7. **Final filesystem checks** — unmounts the destination, checks both filesystems, and prints the final report.

The result is a normal FAT32 boot partition and ext4 root partition. It is not a sector-by-sector clone, so the destination does not need to be exactly the same size or model as the source.

## 🧭 Destination choices

The available menu depends on what the program finds:

| State | Available actions |
| --- | --- |
| New or unknown disk | Initialize and create a new mirror |
| Recognized mirror | Update, verify without copying, or rebuild from scratch |
| Interrupted after initialization | Resume copying or rebuild |

An **update** keeps the existing destination layout and copies current changes.

A **verification** reads and checks the recognized mirror without copying new source files.

A **rebuild** erases, repartitions, formats, and copies the destination again. It always requires the complete destructive confirmation process.

## 🖥️ Compatibility

This release is designed specifically for Raspberry Pi bootable disks.

| Platform or layout | Support |
| --- | --- |
| Raspberry Pi 5, ARM64, compatible two-partition layout | ✅ Tested |
| Raspberry Pi 4, ARM32 or ARM64, compatible two-partition layout | ✅ Designed for |
| Raspberry Pi Debian-based system with FAT boot and ext4 root | ✅ Compatible when all requirements below are met |
| Generic ARM single-board computers | ❌ Not supported |
| AMD64/x86-64 PCs | ❌ Not supported |
| i386/i686 PCs | ❌ Not supported |
| UEFI, BIOS/GRUB, LVM, Btrfs, ZFS, or encrypted root | ❌ Not supported |

The CPU architecture is not the main limitation. The program is Raspberry Pi-specific because it expects Raspberry Pi boot files and boot configuration, including `config.txt`, `cmdline.txt`, `kernel*.img`, device-tree files, and PARTUUID-based booting.

### Required source layout

- Raspberry Pi 4 or Raspberry Pi 5
- Debian-based Raspberry Pi or AllStarLink installation
- DOS/MBR partition table
- Exactly two partitions on the active source disk
- First partition: FAT boot filesystem mounted at `/boot` or `/boot/firmware`
- Second partition: ext4 root filesystem mounted at `/`
- Boot and root partitions located on the same physical disk
- Source `/etc/fstab` entries that resolve to the active boot and root partitions
- `cmdline.txt` root entry that identifies the active root partition
- A destination physical disk with enough capacity
- `systemd` and the standard Linux disk utilities used by the script

> [!NOTE]
> v1.0 was fully tested on a Raspberry Pi 5 running Debian 13 Trixie with AllStarLink 3 and DVSwitch. Testing covered NVMe to USB, USB boot, USB back to NVMe, NVMe boot, normal updates, full rebuilds, service recovery, swap-file regeneration, checksum verification, and EEPROM boot-order switching.

## 📦 Files

| File | Purpose |
| --- | --- |
| `asl-bootable-mirror` | Main mirror and verification program |
| `install-asl-bootable-mirror` | Installs the program in `/usr/local/sbin` |
| `README.md` | Project documentation |

## 🚀 Installation

Use only one installation method. Every terminal example below is one copy-and-paste command.

### Method 1 — Git

```bash
cd /home/asl && git clone https://github.com/ke2hni/asl-bootable-mirror.git && cd asl-bootable-mirror && chmod 755 asl-bootable-mirror install-asl-bootable-mirror && sudo ./install-asl-bootable-mirror
```

### Method 2 — Wget

```bash
cd /home/asl && wget -O asl-bootable-mirror https://raw.githubusercontent.com/ke2hni/asl-bootable-mirror/main/asl-bootable-mirror && wget -O install-asl-bootable-mirror https://raw.githubusercontent.com/ke2hni/asl-bootable-mirror/main/install-asl-bootable-mirror && chmod 755 asl-bootable-mirror install-asl-bootable-mirror && sudo ./install-asl-bootable-mirror
```

### Method 3 — Curl

```bash
cd /home/asl && curl -fL -o asl-bootable-mirror https://raw.githubusercontent.com/ke2hni/asl-bootable-mirror/main/asl-bootable-mirror && curl -fL -o install-asl-bootable-mirror https://raw.githubusercontent.com/ke2hni/asl-bootable-mirror/main/install-asl-bootable-mirror && chmod 755 asl-bootable-mirror install-asl-bootable-mirror && sudo ./install-asl-bootable-mirror
```

The installer:

- Checks the main script for Bash syntax errors.
- Backs up an existing installed version with a timestamp.
- Installs the program as `/usr/local/sbin/asl-bootable-mirror`.
- Sets root ownership and executable permissions.

## ▶️ How to run it

### Check the installed version

```bash
sudo asl-bootable-mirror --version
```

Expected output:

```text
asl-bootable-mirror 1.0
```

### Run the read-only preflight

Always run preflight first:

```bash
sudo asl-bootable-mirror --preflight
```

Preflight asks you to select the destination so it can check the exact disk, capacity, and current mirror state. It does not write anything.

### Create or update the mirror

```bash
sudo asl-bootable-mirror
```

Follow the displayed prompts. For the strongest verification, accept the recommended service list when asked.

> [!TIP]
> The first rebuild takes longer because every file is copied. Later updates normally transfer only changed files, although full checksum and readback verification still take time.

## 🔐 Safety design

- The active source disk cannot be selected as the destination.
- The destination must be a whole physical disk.
- A stable serial number or WWN is required.
- Device identity is checked again during destructive and critical operations.
- Initializing or rebuilding requires typing `ERASE` followed by the exact displayed destination identity.
- Source and destination PARTUUID values must be different.
- The program stops if the source disk, boot mount, destination identity, partition layout, filesystem type, or expected boot files change unexpectedly.
- SSH, networking, systemd, D-Bus, logging, and other protected services cannot be selected for the consistency pause.
- Cleanup attempts to unmount the destination and restart paused services if an error or interruption occurs.
- The program does not modify the active source filesystem, source PARTUUID values, or Raspberry Pi EEPROM.

> [!WARNING]
> This is a disk-maintenance tool. A mistaken destination selection can erase the wrong disk. Keep separate current backups of anything important.

## 🚫 Files intentionally not copied

Live virtual filesystems, temporary data, other mounted filesystems, and program-generated metadata are excluded. Important examples include:

- Runtime contents under `/dev`, `/proc`, `/sys`, and `/run`
- Temporary contents under `/tmp` and `/var/tmp`
- Mounted filesystem contents under `/mnt` and `/media`
- `/lost+found`
- `/var/lib/pcp/tmp`
- `/var/swap`
- The program log and destination metadata

The required empty mount-point directories are preserved. `/var/swap` is intentionally absent from the completed mirror so `dphys-swapfile` can create a correctly allocated swap file when the clone first boots.

## 🔎 Verification and reports

When the recommended services are paused, v1.0 performs:

1. Source-to-destination rsync checksum and metadata comparison.
2. Destination root SHA-256 manifest creation and complete readback.
3. Destination boot SHA-256 manifest creation and complete readback.
4. Raspberry Pi boot-structure and PARTUUID checks.
5. Read-only FAT32 and ext4 filesystem checks after unmounting.

Successful completion ends with:

```text
RESULT: VERIFIED BOOTABLE FILESYSTEM MIRROR
```

The destination stores:

```text
/var/lib/asl-bootable-mirror/clone.conf
/var/lib/asl-bootable-mirror/last-report.txt
/var/lib/asl-bootable-mirror/SHA256SUMS-root.txt
/var/lib/asl-bootable-mirror/SHA256SUMS-boot.txt
```

The running system log is:

```text
/var/log/asl-bootable-mirror.log
```

## 💾 Testing the bootable mirror

The Raspberry Pi EEPROM is stored on the Pi itself, not on the NVMe, USB drive, or SD card.

The mirror program does not change boot order. Before testing, either disconnect higher-priority bootable drives or temporarily change the Pi EEPROM `BOOT_ORDER` so the mirror is tried first.

After booting the mirror, confirm the active root and boot devices:

```bash
echo "ROOT=$(findmnt -nro SOURCE /)"; echo "BOOT=$(findmnt -nro SOURCE /boot/firmware 2>/dev/null || findmnt -nro SOURCE /boot)"
```

To confirm swap regeneration and service health:

```bash
systemctl is-active dphys-swapfile.service; swapon --show; systemctl --failed --no-pager
```

## 🛠️ Troubleshooting

### The destination is not listed

Make sure it is connected as a physical disk and visible with:

```bash
lsblk -o NAME,SIZE,MODEL,SERIAL,WWN,TRAN,FSTYPE,MOUNTPOINTS
```

### The destination belongs to a different source drive

The mirror identity marker protects against updating a destination from an unexpected source disk. Confirm which system is currently running. If you intentionally changed the source, select **Rebuild it from scratch** and complete the full destructive confirmation.

### Full verification reports a changing file

Choose the service-review option and add the service that owns the changing file. The final synchronization and source comparison must occur while persistent files are stable.

### A paused service fails to restart

The program reports the exact unit name and stops. Inspect it with:

```bash
sudo systemctl status SERVICE-NAME --no-pager --full
```

### USB or storage errors appear

Check the power supply, USB enclosure, cable, SSD health, and kernel log. The program intentionally stops when it detects new storage transport errors during critical operations.

## 🧪 Tested configuration

- Raspberry Pi 5 Model B
- Debian 13 Trixie, ARM64
- AllStarLink 3
- DVSwitch multimode services
- NVMe source and destination testing
- USB SSD source and destination testing
- 256 GB NVMe and 256 GB USB SSD
- Successful NVMe → USB → NVMe recovery cycle
- Successful full RC10 rebuild promoted to v1.0
- Verified USB boot with the NVMe still installed
- Verified return to NVMe boot after restoring EEPROM order
- No failed services or boot-log errors during final testing

## 🧱 Design principles

- Protect the active source first.
- Identify the physical destination repeatedly.
- Require clear confirmation before destructive work.
- Prefer normal filesystems over raw sector cloning.
- Keep the node reachable during normal copying and verification.
- Stop on unexplained differences instead of claiming success.
- Verify destination data by reading it back.
- Never report a mirror as bootable without checking its Raspberry Pi boot structure.

## 👤 Author

Jeff Milne — KE2HNI

## 📄 License

BSD 3-Clause License.

Partition and PARTUUID safety concepts were adapted from `rpi-clone`, originally by Bill Wilson and maintained through later community forks, including work by Jeff Geerling and Tony Galmiche.

Use at your own risk and keep current backups.
