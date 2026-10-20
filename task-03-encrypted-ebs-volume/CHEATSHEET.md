# Task 3 — Encrypted EBS Volume Cheatsheet

## Lab facts
- Region: `ap-south-1`
- Encrypted data volume: `vol-0cf441f44249b5b2e`
- Size/type: `4 GiB`, `gp3`
- KMS alias: `aws/ebs`
- Linux device: `/dev/nvme1n1`
- Filesystem: XFS
- UUID: `6d11d909-0e53-41d0-8432-1e11dd4faaca`
- Mount point: `/data`
- Root disk: `/dev/nvme0n1` — **do not format**

## Inspect storage
```bash
lsblk
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
df -h
df -hT /data
sudo blkid /dev/nvme1n1
lsblk -f
```

## Create filesystem and mount
Only after confirming the correct new, empty data disk:
```bash
sudo mkfs.xfs /dev/nvme1n1
sudo mkdir -p /data
sudo mount UUID=6d11d909-0e53-41d0-8432-1e11dd4faaca /data
findmnt /data
df -hT /data
```
**Warning:** `mkfs.xfs` formats the target device and can make existing data inaccessible.

## Test write access
```bash
sudo touch /data/ebs-test.txt
ls -l /data
sudo sh -c 'echo "EBS persistence test" > /data/persistence-test.txt'
cat /data/persistence-test.txt
```

## Persistent mounting
```bash
sudo cp /etc/fstab /etc/fstab.bak
sudo blkid /dev/nvme1n1
sudo nano /etc/fstab
```
Add:
```text
UUID=6d11d909-0e53-41d0-8432-1e11dd4faaca  /data  xfs  defaults,nofail  0  2
```
Save Nano: `Ctrl+O`, `Enter`, `Ctrl+X`.

Validate and test:
```bash
cat /etc/fstab
grep -n '6d11d909-0e53-41d0-8432-1e11dd4faaca' /etc/fstab
sudo findmnt --verify --verbose
sudo umount /data
sudo mount -a
findmnt /data
df -hT /data
```
Do not reboot until validation succeeds and `/data` is mounted correctly.

## Reboot persistence test
```bash
sudo reboot
```
After reconnecting:
```bash
findmnt /data
df -hT /data
cat /data/persistence-test.txt
lsblk -f
```
Expected: `/dev/nvme1n1` mounted at `/data`; marker prints `EBS persistence test`.

## Troubleshooting
- `mount: /data: can't find in /etc/fstab` → entry is missing or not saved.
- `/data` shows the root filesystem → check `findmnt /data`; source should be `/dev/nvme1n1`.
- Confirm UUID with `sudo blkid /dev/nvme1n1`.
- Identify disks with `lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS` before formatting.
- Verify encryption and KMS alias in AWS Console; `lsblk` does not prove encryption.

## Key concepts
- **EBS:** Persistent block storage for EC2.
- **KMS:** Key management for encrypted EBS data.
- **XFS:** Linux filesystem on the data volume.
- **UUID:** Stable filesystem identifier used in `/etc/fstab`.
- **`/etc/fstab`:** Persistent filesystem mount configuration.
- **`nofail`:** Helps boot proceed if the data volume is unavailable.
- **`findmnt`:** Shows which filesystem is mounted at a target.
- **`df -hT`:** Shows filesystem type, capacity, usage, and mount point.
