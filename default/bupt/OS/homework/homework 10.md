# 1
We have three level of index table, and each level is going to have 128 sub block, so in total we can store $128^3\times 512=1\text{GB}$ at most.
Saving first 436 bytes of file in the inode provides better locality and we can access small files faster.

# 2
In FAT, a directory is actually a file containing an array of directory entries, so we can use a bit in the attribute byte to check whether it is a file or directory.
In FFS, the file name is connected to a inode, we can check the type of inode to tell whether it is a directory or not.
In NTFS, the file name is connected to a MFT record, we can read the record flags to check its type.

# 3
Direct only is 72 KB.
Largest we can have $1000^{4}+2\times 1000^{3} +1000^{2} + 1000 + 12$ blocks in total. Times 6 KB will be the final size.

# 4
100, 116, 185, 87, 75, 22, 11, 3

# 5
| RAID Level        | Min Disks | Data Layout            | Redundancy Method         | Fault Tolerance   | Read Performance | Write Performance | Storage Efficiency | Typical Use                         |
| ----------------- | --------- | ---------------------- | ------------------------- | ----------------- | ---------------- | ----------------- | ------------------ | ----------------------------------- |
| **RAID 0**        | 2         | Striping               | None                      | 0 disks           | ⭐⭐⭐⭐⭐            | ⭐⭐⭐⭐⭐             | 100%               | High-performance, non-critical data |
| **RAID 1**        | 2         | Mirroring              | Full copy                 | 1 disk per mirror | ⭐⭐⭐⭐             | ⭐⭐⭐               | 50%                | OS, critical data                   |
| **RAID 2**        | ≥3        | Bit-level striping     | Hamming code ECC          | 1 disk            | ⭐⭐               | ⭐                 | Low                | Mostly theoretical                  |
| **RAID 3**        | ≥3        | Byte-level striping    | Dedicated parity disk     | 1 disk            | ⭐⭐⭐⭐             | ⭐⭐                | (N−1)/N            | Large sequential I/O                |
| **RAID 4**        | ≥3        | Block-level striping   | Dedicated parity disk     | 1 disk            | ⭐⭐⭐⭐             | ⭐⭐                | (N−1)/N            | Rarely used                         |
| **RAID 5**        | ≥3        | Block-level striping   | Distributed parity        | 1 disk            | ⭐⭐⭐⭐             | ⭐⭐⭐               | (N−1)/N            | General-purpose servers             |
| **RAID 6**        | ≥4        | Block-level striping   | Double distributed parity | 2 disks           | ⭐⭐⭐              | ⭐⭐                | (N−2)/N            | Large storage arrays                |
| **RAID 10 (1+0)** | 4         | Striping over mirrors  | Mirroring + striping      | 1 disk per mirror | ⭐⭐⭐⭐⭐            | ⭐⭐⭐⭐              | 50%                | Databases, high IOPS                |
| **RAID 01 (0+1)** | 4         | Mirroring over stripes | Striping + mirroring      | Limited           | ⭐⭐⭐⭐             | ⭐⭐⭐               | 50%                | Obsolete                            |
