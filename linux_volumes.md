# Linux Volumes, EBS & LVM — Full Notes

## 1. The Basics — AWS Storage on an EC2 Instance

* When you launch an EC2 instance, AWS gives you a **root volume** — minimum size is **8GB**, you cannot go lower than that (you can go higher).
* This root volume is itself an **EBS (Elastic Block Store)** volume — it's just automatically attached at launch time.
* Beyond the root volume, you can attach **extra EBS volumes** of any size you want — e.g. one 10GB, one 12GB, one 14GB — as separate disks to the same instance.
* Extra EBS volumes are useful when you need more storage, want to separate data from the OS disk, or want to combine multiple volumes into one bigger usable space (covered in the LVM section below).

## 2. Launching an Instance With Storage, Then SSH-ing In

1. Go to EC2 → Launch Instance.
2. Choose your AMI (OS image), instance type, key pair, security group, etc.
3. Under **"Configure Storage"**, you'll see the root volume defaulted to 8GB — you *can* increase it here, but 8GB is the floor.
4. Launch the instance.
5. SSH into it once it's running:
   ```
   ssh -i your-key.pem ec2-user@<public-ip-or-dns>
   ```

## 3. Creating an Extra EBS Volume (AWS Console)

1. Go to **EC2 → Volumes** (left sidebar, under "Elastic Block Store").
2. Click **Create Volume**.
3. Configure:
   * **Size** — how big you want it (e.g. 10GB, 12GB, 14GB).
   * **Volume type** — affects IOPS/throughput performance (gp3, io2, etc.).
   * **IOPS / Throughput** — performance settings, higher = faster but costs more.
   * **Snapshot** — optionally create the volume from an existing snapshot (backup), or leave blank for a fresh empty volume.
   * **Availability Zone** — ⚠️ **must match the AZ of the EC2 instance** you plan to attach it to. A volume in a different AZ cannot be attached to your instance.
4. Click **Create Volume**.

## 4. Device Naming — sda vs sdf (and friends)

* `/dev/sda` (or `/dev/xvda`, `/dev/nvme0n1` on newer instance types) → typically the **root volume** (the OS disk).
* `/dev/sdf`, `/dev/sdg`, `/dev/sdh`... → the naming convention AWS recommends for **additional/extra volumes** you attach yourself.
* Basically: **"sda" range = system/root disk. "sdf" onward = extra data disks you attach.** This is just a naming convention to avoid clashing with the root device name — it keeps things organized so you know at a glance which disk is which.
* Note: on newer Nitro-based instances, the actual device name inside Linux might show up as `/dev/nvme1n1` etc. regardless of what name you picked in the console — always double check with `lsblk` (see below) rather than assuming.

## 5. Attaching the Volume

1. In the **Volumes** page, select your new volume (currently shows as "available").
2. Click **Actions → Attach Volume**.
3. Select the instance to attach it to, and the device name (e.g. `/dev/sdf`).
4. Click **Attach**.

**Check it worked (from inside the instance):**
```
lsblk
```
This lists all block devices attached to the instance — you should now see your new disk (e.g. `xvdf` or `nvme1n1`) listed, but with no mount point yet.

## 6. `df -h` and `lsblk` — what they actually do

* **`df -h`** → shows disk space **usage** for filesystems that are currently **mounted** — how much space is used/free, and where each filesystem is mounted (`-h` = human-readable, shows GB/MB instead of raw bytes).
  ```
  df -h
  ```
  If a disk isn't mounted yet, it **won't show up** in `df -h` at all.

* **`lsblk`** → lists **all block devices** (disks/partitions) attached to the system, whether mounted or not, and shows their size and mount point (if any).
  ```
  lsblk
  ```
  This is the command you use to *see a newly attached volume even before it's mounted* — `df -h` can't see it yet, but `lsblk` can.

**Rule of thumb:** `lsblk` = "what disks exist," `df -h` = "what's actually mounted and how full is it."

## 7. Attach vs Mount — the key difference (and why mount matters)

* **Attach** = a purely AWS/infrastructure-level action. It just plugs the EBS volume into the instance, like plugging in a USB drive to a laptop — the OS can *see* the disk now (`lsblk` shows it), but it's not usable yet.
* **Mount** = an OS-level action that makes a disk's filesystem accessible at a specific folder (directory) on your system, so you can actually read/write files to it.

**Real scenario to make this click:**
Imagine you plug a brand new USB drive into your laptop.
- Windows/Mac would usually auto-mount it and show it as a drive letter/icon.
- On a Linux server, attaching ≠ mounting. Plugging in the drive (attach) just makes the OS aware the hardware exists. But until you tell Linux *"take this disk, and make its contents show up at folder `/data`"* (mount), you literally cannot `cd` into it, read from it, or write to it. It just sits there, invisible to your file browsing.

So: **attach = "the disk exists on the machine now." mount = "the disk's data is now usable at a specific folder."** You need both, in that order, before you can actually use a new volume.

Also: a brand-new raw EBS volume has **no filesystem** on it yet — you must format it (`mkfs`) before you can even mount it. (More on this below.)

---

## 8. LVM — Logical Volume Manager

### What is LVM???

> LVM is a system that sits **between** your raw physical disks and the filesystem, letting you combine multiple physical disks into one flexible storage pool, and then carve out "logical" volumes from that pool — which you can resize/extend later without the headache of dealing with raw partitions directly.

Think of it in 3 layers:

1. **PV (Physical Volume)** → a raw disk (or partition) that you register with LVM. This is just "telling LVM: this disk is available for use."
2. **VG (Volume Group)** → a pool made by combining one or more PVs together. This is where the *combining* happens — e.g. your 10GB + 12GB + 14GB disks get merged into one 36GB pool.
3. **LV (Logical Volume)** → a slice you carve out of the VG's total space, which is what you actually format and mount. You can have one LV using the whole VG, or multiple LVs sharing it.

**Why use LVM instead of just formatting each disk separately?**
- You can combine multiple small disks into one big usable volume.
- You can **resize (extend)** an LV later — very easily, without repartitioning — as long as the VG has free space (or you add another disk to the VG).
- Much more flexible for real-world servers where storage needs grow over time.

### Your example — combining 10G + 12G + 14G into a 36G group

1. Turn each disk into a PV.
2. Combine all 3 PVs into one VG (10+12+14 = 36GB total pool).
3. Create an LV by slicing out space from that 36GB VG (you can take all 36GB, or less, leaving room to grow later).

---

## 9. LVM Commands — Step by Step

### Step 1 — Create Physical Volumes (PV)

```
sudo pvcreate /dev/xvdf /dev/xvdg /dev/xvdh
```
This registers your 3 raw disks as LVM-usable physical volumes.

**Check them:**
```
pvs           # quick summary table
sudo pvdisplay  # detailed info per PV
```

### Step 2 — Create the Volume Group (VG)

```
sudo vgcreate myvg /dev/xvdf /dev/xvdg /dev/xvdh
```
This combines all 3 PVs into one volume group named `myvg` (≈36GB total).

**Check it:**
```
vgs             # quick summary table
sudo vgdisplay    # detailed info
```

### Step 3 — Create the Logical Volume (LV)

```
sudo lvcreate -L 36G -n mylv myvg
```
* `-L 36G` → size of the LV to create
* `-n mylv` → name of the LV
* `myvg` → which volume group to carve this LV out of

(You could also use `-l 100%FREE` instead of `-L 36G` to use up all remaining free space in the VG automatically.)

**Check it:**
```
lvs             # quick summary table
sudo lvdisplay    # detailed info
```

At this point, you have a logical volume `/dev/myvg/mylv` — but it's still just raw space, no filesystem yet.

### Step 4 — Format the LV

```
sudo mkfs -t ext4 /dev/myvg/mylv
```
This puts an actual filesystem (ext4) on the logical volume — required before you can mount it.

### Step 5 — Create a mount point and mount it

```
sudo mkdir /mnt/mydata
sudo mount /dev/myvg/mylv /mnt/mydata
```
Now the combined 36GB space is usable at `/mnt/mydata`. Verify with `df -h` or `lsblk`.

### Step 6 — Make the mount persistent (survive reboot)

By default, a manual `mount` disappears after a reboot. To make it permanent, add an entry to `/etc/fstab`:
```
sudo blkid /dev/myvg/mylv   # get the UUID
sudo nano /etc/fstab
```
Add a line like:
```
UUID=xxxx-xxxx  /mnt/mydata  ext4  defaults  0  2
```

---

## 10. Unmounting and Re-mounting

**Unmount:**
```
sudo umount /mnt/mydata
```
(Note: it's `umount`, no "n" — common typo.) If you get a "device is busy" error, make sure no one has a terminal `cd`'d into that folder or a process using it.

**Mount again (after unmount, or after reboot if not in fstab):**
```
sudo mount /dev/myvg/mylv /mnt/mydata
```

---

## 11. The Other Way — Direct Mount (No LVM at all)

You don't *have* to use LVM. For a single disk with no need to combine/resize later, you can format and mount it directly:

```
sudo mkfs -t ext4 /dev/xvdf         # format the raw disk directly
sudo mkdir /mnt/simpledata
sudo mount /dev/xvdf /mnt/simpledata
```

**Difference between the two approaches:**
* **Direct mount** → simple, one disk = one mount, but if that disk fills up, you're stuck — you'd need to unmount, resize the partition manually, or migrate data. Less flexible.
* **LVM approach** → more setup upfront (PV → VG → LV), but you can combine multiple disks and easily **grow** the volume later without downtime or data migration headaches. Preferred for anything that might need to scale.

---

## 12. Dynamic Storage Management — Extending an LV (lvextend)

**Scenario:** Your `mylv` is running out of space, and you either have free space left in the VG, or you've attached and added a new disk to the VG.

**Step 1 — If needed, add a new PV to the VG first:**
```
sudo pvcreate /dev/xvdi
sudo vgextend myvg /dev/xvdi
```

**Step 2 — Extend the LV:**
```
sudo lvextend -L +10G /dev/myvg/mylv
```
`+10G` means "add 10GB to the current size" (you could also specify an absolute size like `-L 50G`).

Or, to use all available free space in the VG at once:
```
sudo lvextend -l +100%FREE /dev/myvg/mylv
```

**Step 3 — Important: resize the filesystem too!**
Extending the LV only grows the underlying block device — the filesystem on top still thinks it's the old size until you tell it to grow too:
```
sudo resize2fs /dev/myvg/mylv        # for ext4
# OR, if using XFS filesystem:
sudo xfs_growfs /mnt/mydata
```

**Verify the new size:**
```
df -h
lvs
```

---

## Quick Command Cheat Sheet

| Layer | Create | List (quick) | List (detailed) |
|---|---|---|---|
| Physical Volume | `pvcreate /dev/xvdf` | `pvs` | `pvdisplay` |
| Volume Group | `vgcreate myvg /dev/xvdf` | `vgs` | `vgdisplay` |
| Logical Volume | `lvcreate -L 10G -n mylv myvg` | `lvs` | `lvdisplay` |

| Other Key Commands | Purpose |
|---|---|
| `lsblk` | See all disks/block devices, mounted or not |
| `df -h` | See mounted filesystems and space usage |
| `mkfs -t ext4 /dev/xxx` | Format a disk/LV with a filesystem |
| `mount /dev/xxx /path` | Mount a formatted disk/LV to a folder |
| `umount /path` | Unmount it |
| `vgextend myvg /dev/xxx` | Add a new disk to an existing VG |
| `lvextend -L +10G /dev/vg/lv` | Grow an LV's size |
| `resize2fs` / `xfs_growfs` | Grow the filesystem after growing the LV |

**One-line memory trick:** PV → VG → LV → format → mount, is basically:
*"register the disk → pool the disks → slice out a volume → give it a filesystem → plug it into a folder."*
