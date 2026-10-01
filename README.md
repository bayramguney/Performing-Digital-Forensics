# Performing-Digital-Forensics

# 🛡️ Applied Lab: Performing Digital Forensics

> Perform forensic investigations using Kali Linux and The Sleuth Kit (TSK) to discover hidden partitions, recover deleted files, and perform file carving from damaged disk images.

---

## 📖 Overview

In this lab, I performed several post-incident digital forensic investigations on compromised disk images. Using Kali Linux and multiple forensic utilities, I analyzed storage media to locate hidden partitions, recover deleted evidence, and extract files from corrupted disk images.

The lab demonstrates common forensic techniques used during incident response and digital investigations.

---

## 🎯 Objectives

This lab aligns with the following **CompTIA Security+ (SY0-701)** objectives:

- **4.8** – Explain appropriate incident response activities
- **4.9** – Use data sources to support an investigation

---

# 🖥️ Lab Environment

| Component | Details |
|-----------|---------|
| Operating System | Kali Linux |
| User | root |
| Password | `Pa$$w0rd` |
| Tools | fdisk, TestDisk, The Sleuth Kit (TSK), fiwalk |
| Evidence Source | Student-Resources-L25.ISO |

---

# Part 1 – Hidden Partition Analysis

## Scenario

An Incident Response Team (IRT) was unable to determine how an attacker hid evidence on a compromised system.

A forensic disk image (`ext-part-test-2.dd`) was provided for investigation.

The objective was to discover hidden partitions and examine their contents.

---

## Tools Used

- fdisk
- testdisk
- fiwalk
- fsstat
- mmls
- fls
- istat
- losetup
- mount

---

## Step 1 – Prepare the Evidence

Mounted the ISO and copied the forensic images.

```bash
ls /media/cdrom0/

cp /media/cdrom0/* /root/Downloads/
```

Extracted the forensic image.

```bash
cd /root/Downloads

unzip 1-extend-part.zip

cd 1-extend-part
```

---

## Step 2 – Examine Partition Table

Displayed partition information.

```bash
fdisk -l ext-part-test-2.dd
```

### Findings

- Extended partition detected
- 6 formattable partitions
- One extended partition

### Knowledge Check

**How many formattable partitions exist?**

✅ **6**

Possible explanations for unaccounted sectors:

- ✅ Hidden logical drive
- ✅ Unallocated space

---

## Step 3 – Discover Hidden Partition

Used TestDisk.

```bash
testdisk -l ext-part-test-2.dd
```

Unlike fdisk, TestDisk discovered an additional partition.

---

## Step 4 – Analyze Image with fiwalk

```bash
fiwalk ext-part-test-2.dd | less
```

Observed:

- FAT16 partitions
- Partition metadata
- Hidden logical partition

---

## Step 5 – Determine Partition Offset

```bash
mmls ext-part-test-2.dd
```

Hidden partition offset:

```
262143
```

---

## Step 6 – View File System Information

```bash
fsstat -f fat16 ext-part-test-2.dd -o 262143
```

Displayed:

- FAT16 metadata
- File system information
- Allocation details

---

## Step 7 – List Files

```bash
fls -f fat16 ext-part-test-2.dd -o 262143
```

Recovered:

```
second-3.txt
```

Also displayed hidden system metadata files.

---

## Step 8 – View Inode Information

```bash
istat -f fat16 ext-part-test-2.dd -o 262143 3
```

Observed:

- File metadata
- Timestamps
- Allocation information

---

## Step 9 – Mount Hidden Partition

Created a loop device.

```bash
losetup --partscan --find --show ext-part-test-2.dd
```

Mounted hidden partition.

```bash
mkdir /mnt/p6

mount /dev/loop0p7 /mnt/p6

ls -l /mnt/p6
```

Recovered file:

```
second-3.txt
```

---

# Part 2 – Recover Deleted Files

## Scenario

A second compromised system contained deleted evidence.

The objective was to recover deleted NTFS files.

---

## Tools Used

- mount
- tsk_recover
- ls

---

## Step 1 – Extract Evidence

```bash
cd /root/Downloads

unzip 7-undel-ntfs.zip

cd 7-undel-ntfs
```

---

## Step 2 – Mount Image

```bash
mkdir /mnt/temp7

mount 7-ntfs-undel.dd /mnt/temp7
```

Only one folder was visible:

```
System Volume Information
```

This indicated deleted user files.

---

## Step 3 – Recover Deleted Files

```bash
tsk_recover 7-ntfs-undel.dd output
```

Recovered:

```
Files Recovered: 8
```

---

## Step 4 – Examine Recovered Files

```bash
ls -l output
```

Additional recovered directories:

```bash
ls output/dir1

ls output/dir1/dir2
```

Recovered files included:

- frag1.dat
- frag3.dat
- mult2.dat
- sing1.dat

---

# Part 3 – File Carving

## Scenario

A damaged FAT image prevented normal forensic analysis.

The objective was to recover files by rebuilding the boot sector and carving data.

---

## Tools Used

- fdisk
- fiwalk
- fsstat
- mmls
- TestDisk

---

## Step 1 – Extract Image

```bash
cd /root/Downloads

unzip 11-carve-fat.zip

cd 11-carve-fat
```

---

## Step 2 – Attempt to Mount

```bash
mkdir /mnt/temp11

mount 11-carve-fat.dd /mnt/temp11
```

Mount failed due to corruption.

---

## Step 3 – Analyze Image

```bash
fdisk -l 11-carve-fat.dd
```

Observed:

- Invalid disk identifier
- Missing partition table

---

## Step 4 – Additional Analysis

```bash
fiwalk 11-carve-fat.dd
```

Multiple errors reported:

```
Possible encryption detected
```

---

## Step 5 – Attempt File System Analysis

```bash
fsstat 11-carve-fat.dd
```

Result:

```
Possible encryption detected
```

Even after specifying FAT16:

```bash
fsstat -f fat16 11-carve-fat.dd
```

Received:

```
Invalid magic value
```

---

## Step 6 – Recover Files with TestDisk

Created an output folder.

```bash
mkdir output
```

Started TestDisk.

```bash
testdisk 11-carve-fat.dd
```

Recovery process:

- Proceed
- Partition Type → None
- Unknown partition
- Select FAT16
- Boot
- Rebuild Boot Sector
- List Files
- Select All (`a`)
- Copy (`Shift + C`)
- Save into output folder

Recovery completed successfully.

```
15 files recovered
```

---

## Step 7 – Review Evidence

View recovered files.

```bash
cd output

ls -l
```

Opened recovered images.

```bash
xdg-open haxor2.jpg
```

Opened recovered PDF.

```bash
xdg-open lin_1.2.pdf
```

---

## Knowledge Check

**Which recovered image contained cats?**

✅ **haxor2.jpg**

---

# Key Commands

```bash
fdisk -l image.dd

testdisk image.dd

fiwalk image.dd

mmls image.dd

fsstat -f fat16 image.dd -o OFFSET

fls -f fat16 image.dd -o OFFSET

istat -f fat16 image.dd -o OFFSET INODE

tsk_recover image.dd output

losetup --partscan --find --show image.dd

mount /dev/loop0p7 /mnt/p6

xdg-open filename
```

---

# Skills Demonstrated

- Digital forensic investigation
- Hidden partition discovery
- Partition analysis
- File system analysis
- Metadata examination
- NTFS deleted file recovery
- File carving
- Boot sector reconstruction
- Incident response evidence collection
- Linux forensic command-line tools
- Working with forensic disk images
- The Sleuth Kit (TSK)
- TestDisk recovery
- FAT16 and NTFS analysis

---

# Key Takeaways

- Multiple forensic tools should be used because each may reveal different evidence.
- Hidden partitions can conceal attacker data and require specialized analysis.
- Deleted files often remain recoverable until overwritten.
- File carving allows investigators to recover data even when file systems are damaged.
- Rebuilding a corrupted boot sector in memory can restore access without modifying the original evidence.
- Maintaining forensic integrity is essential throughout an investigation.

---

## Technologies Used

- Kali Linux
- The Sleuth Kit (TSK)
- TestDisk
- fdisk
- fiwalk
- FAT16
- NTFS
- Loop Devices
- Digital Forensics
- Incident Response

---

## Lab Outcome

Successfully completed a forensic investigation by:

- Identifying a hidden partition
- Recovering deleted NTFS files
- Rebuilding a corrupted boot sector
- Performing file carving
- Extracting evidence from damaged forensic images
- Using industry-standard digital forensic tools commonly employed during incident response investigations.
