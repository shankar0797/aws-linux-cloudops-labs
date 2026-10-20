# Task 3: Create and Encrypt an EBS Volume

## Objective
Create a separate encrypted Amazon EBS data volume, attach it to an EC2 instance running Amazon Linux 2023, create an XFS filesystem, mount it at `/data`, and configure persistent mounting with a filesystem UUID.

## Environment
- AWS Region: `ap-south-1` (Mumbai)
- OS: Amazon Linux 2023
- Volume type/size: `gp3`, 4 GiB
- Encryption: Enabled using KMS alias `aws/ebs`
- Filesystem: XFS
- Mount point: `/data`

## Resource identifiers
| Item | Value |
|---|---|
| New encrypted data volume | `vol-0cf441f44249b5b2e` |
| Linux device | `/dev/nvme1n1` |
| Filesystem UUID | `6d11d909-0e53-41d0-8432-1e11dd4faaca` |
| Mount point | `/data` |
| Existing root disk | `/dev/nvme0n1` |
| Existing root volume | `vol-06d480336d6238952` |

**Important:** The existing root volume was reported as unencrypted. This lab encrypts the separate data volume only; it does not encrypt the root volume.

## Architecture
```text
EC2 instance (Amazon Linux 2023)
├── Existing root disk: /dev/nvme0n1 (12 GiB; do not format)
└── New encrypted EBS data disk: /dev/nvme1n1 (4 GiB; KMS aws/ebs)
    └── XFS filesystem
        └── Mounted at /data
            └── UUID entry in /etc/fstab for automatic mounting
```

## Procedure

### 1. Inspect existing storage
Run on the EC2 Linux terminal:
```bash
lsblk
df -h
ls -l /data
```
Initially, only the 12 GiB root disk and its system partitions were present. `/data` did not exist.

### 2. Create the encrypted EBS volume
In AWS Console, navigate to **EC2 → Elastic Block Store → Volumes → Create volume**. Configure:
- Type: `gp3`
- Size: `4 GiB`
- Availability Zone: `ap-south-1b`, matching the instance
- Encryption: enabled
- KMS key alias: `aws/ebs`

The volume ID was `vol-0cf441f44249b5b2e`. The console showed it as encrypted and initially `Available`.

### 3. Attach and identify the volume
Attach the volume to the intended EC2 instance, then run:
```bash
lsblk
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
```
The new disk appeared as `/dev/nvme1n1`, size 4 GiB, without a filesystem or mount point. The existing root disk remained `/dev/nvme0n1`.

**Safety:** Device names can differ from the attachment name shown in the AWS Console. Identify the new disk by size and mount/filesystem status before formatting. Never format the root disk.

### 4. Create an XFS filesystem
```bash
sudo mkfs.xfs /dev/nvme1n1
lsblk -f
```
The filesystem UUID was:
`6d11d909-0e53-41d0-8432-1e11dd4faaca`

**Warning:** Formatting can make existing data inaccessible. Run `mkfs` only after confirming the device is the intended empty data volume.

### 5. Create the mount point and mount by UUID
```bash
sudo mkdir -p /data
sudo mount UUID=6d11d909-0e53-41d0-8432-1e11dd4faaca /data
df -hT /data
lsblk -f
```
The output showed `/dev/nvme1n1` mounted at `/data` as XFS with approximately 4 GiB capacity.

Test write access:
```bash
sudo touch /data/ebs-test.txt
ls -l /data
df -h /data
```

### 6. Back up and configure `/etc/fstab`
```bash
sudo cp /etc/fstab /etc/fstab.bak
sudo blkid /dev/nvme1n1
sudo nano /etc/fstab
```
Add this single line:
```text
UUID=6d11d909-0e53-41d0-8432-1e11dd4faaca  /data  xfs  defaults,nofail  0  2
```
Save Nano using `Ctrl+O`, press `Enter`, then `Ctrl+X`.

Field meanings:
- `UUID=...`: filesystem identifier
- `/data`: mount point
- `xfs`: filesystem type
- `defaults,nofail`: standard mount options; `nofail` helps boot continue if the data volume is unavailable
- `0`: legacy `dump` field
- `2`: filesystem-check pass field; XFS does not use traditional boot-time `fsck` checks like ext4

An initial validation showed the entry was missing because it had not been saved. The line was then added and verified.

### 7. Validate and test persistent mounting
```bash
sudo findmnt --verify --verbose
sudo umount /data
sudo mount -a
findmnt /data
df -hT /data
```
The final validation reported `Success, no errors or warnings detected`. `findmnt /data` showed `/dev/nvme1n1` mounted at `/data`, filesystem `xfs`.

### 8. Verify persistence after reboot
Create a marker file:
```bash
sudo sh -c 'echo "EBS persistence test" > /data/persistence-test.txt'
cat /data/persistence-test.txt
```
Reboot:
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
Actual post-reboot results confirmed `/dev/nvme1n1` mounted at `/data`, XFS capacity of 4 GiB, and the marker text `EBS persistence test` remained readable.

## Verification summary
| Requirement | Result |
|---|---|
| Separate EBS data volume created | Completed |
| EBS encryption using KMS | Completed; AWS Console showed `Encrypted`, alias `aws/ebs` |
| Volume attached to EC2 | Completed |
| XFS filesystem created | Completed |
| Mounted at `/data` | Completed |
| UUID-based `/etc/fstab` entry | Completed |
| `findmnt` validation and `mount -a` | Completed without errors |
| Mount and data verified after reboot | Completed |

## Troubleshooting
- **`mount: /data: can't find in /etc/fstab`**: The `/data` entry is missing or was not saved. Check `cat /etc/fstab` and `grep -n '<UUID>' /etc/fstab`.
- **Wrong filesystem appears at `/data`**: Check `findmnt /data` and confirm the source is `/dev/nvme1n1`.
- **Device name differs**: Use `lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS` and `sudo blkid` to identify the new volume safely.
- **Encryption verification**: Confirm the `Encryption` and KMS key fields in the AWS Console. Linux `lsblk` does not prove EBS encryption status.

## Interview questions
1. **What is EBS?** Persistent block storage that can be attached to EC2 instances.
2. **What is encryption at rest?** Stored data is encrypted on the underlying storage; EBS handles encryption/decryption transparently for authorized access.
3. **Why use a separate data volume?** It separates application data from the OS disk and helps with capacity management, snapshots, and recovery.
4. **Why use a UUID in `/etc/fstab`?** Device enumeration can change; UUIDs provide a more stable filesystem identifier.
5. **What does `/etc/fstab` do?** Defines filesystems to mount, including automatically during boot.
6. **What does `nofail` do?** Helps the system boot if a nonessential data volume is unavailable.
7. **`mount` vs `/etc/fstab`?** A manual mount affects the current runtime; `/etc/fstab` defines persistent mount configuration.
8. **How was persistence tested?** After reboot, `findmnt /data` showed the volume mounted and the marker file remained readable.

## Evidence screenshots
Save your actual screenshots in `screenshots/`:
1. `01-ebs-volume-encrypted.png`
2. `02-ebs-volume-attached.png`
3. `03-filesystem-created.png`
4. `04-data-volume-mounted.png`
5. `05-fstab-configuration.png`
6. `06-mount-after-reboot.png`

Review screenshots for exposed account identifiers before publishing.

## Cost and cleanup note
EBS volumes incur charges while they exist, including when detached. Snapshots and some KMS key types can have separate charges. To clean up, verify no needed data remains, unmount the filesystem, detach the intended data volume, and delete only that volume in the AWS Console. Never delete the root volume.

## Learning outcome
This lab demonstrates creating and encrypting a separate EBS volume, attaching it to EC2, identifying the Linux device, creating an XFS filesystem, mounting it by UUID, configuring `/etc/fstab`, and verifying persistence after reboot.
