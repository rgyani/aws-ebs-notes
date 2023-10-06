# EBS Volumes


When you create AWS EC2 instances, either attach
* EBS volumes (persistent storage option)
* Instance Store volumes (ephemeral storage option)

Let's discuss the **credit system** and **burst modes** provided by some of the EBS volume types

## EBS Storage Performance

EBS storage performance could be measured in two ways.

* IOPS — The number of I/O operations per second
* Throughput — The speed of data transfer measured in megabytes per second. This is based on the number of I/O operations and the block size of each I/O operation. (Throughput = Number of IOPS * Block size)

Latency means, the time it takes for an I/O operation to complete.

## SSD vs HDD

|  | SSD Volumes | HDD Volumes |
|---|---|---|
| Operations | Small, Random I/O operations | Large, sequential I/O operations|
| Boot Volume? | YES | NO |
| Use Case | Transactional Workloads eg. Databases| Throughput intenstive workloads eg. Big Data, Data Warehousing, Log Progressing, Kafka |
| Performance Attribute | IOPS | Throughput |


## EBS Volume Types

| Volume Type | AKA | Use Case | Size |Max IOPS | Bursting Capability |
|---|---|---|---|---|---|
| General Purpose SSD      | gp2, gp3 | Almost everything                               | 1GB - 16TB    | 16,000 | gp2 only |
| Provisioned IOPS SSD     | io1, io2 | Higher cost, Large Database Workloads           | 4GB - 16TB    | 64,000 | no       |
| Throughput Optimized HDD | St1      | Low cost, Big data warehouse, Log processing    | 500GB - 16 TB | 500    | no       |
| Cold HDD                 | Sc1      | Lowest Cost, File Servers                       | 500GB - 16 TB | 250    | no       |
| EBS Magnetic HDD         | Standard | Previous Generation HDD                         | 1GB - 1 TB    | 40-200 | no       |

## gp2 Volumes

The current default general purpose SSD Storage type with size ranging from 1GB - 16TB

Gp2 volume type IOPS are calculated based on the size of the volume. This is known as the **baseline performance**

### Baseline Performance
We can calculate the baseline performance for a given gp2 volume based on its volume size (see the formula below).

**IOPS for the gp2 volume = (size of the volume in GiB x 3)**

however **the minimum IOPS is always 100** and **the maximum IOPS is always 16000**

This means, up to 33.3 GB volume capacity will have the support up to 100 IOPS (33.3 * 3). After 33.3 GB, the above generic formula will be applied.

eg.

| gp2 Volume Size | Max IOPS |
|---|---|
| 1 GB  | 100 |
| 10 GB | 100 |
| 30 GB | 100 |
| 50 GB | 150 |
| 1 TB  | 3000 |
| 10 TB | 16000 |
| 16TB  | 16000|

### I/O Credits and Burst Mode

I/O credits represent the available bandwidth that the gp2 volume can use to burst when more than the baseline performance is needed.   
The more credits your volume has for I/O, the more time it can burst beyond its baseline performance level and the better it performs when more performance is needed

Each volume receives an initial balance of 5.4 million I/O credits to its gp2 burst bucket.   
**This initial balance of 5.4 million I/O credits is enough to support maximum performance of 3000 IOPS for 30 minutes continuously** (3000 IOPS * 60 * 30 = 5.4 million I/O Credits).

This initial credit balance of 5.4 million I/O credits is primarily designed to provide a fast initial boot cycle for EBS boot volumes and relatively a good bootstrapping experience for other applications whenever required.

3000 IOPS means 1 TB (1000 GB). That means, volumes till 1 TB can be burst up to 3000 IOPS and volumes which are over 1 TB size can move up to 16,000 IOPS as the maximum (at 5.33 TB volume size). From 5.33 TB to 16 TB baseline performance remains at 16,000 IOPS

Once the gp2 volume uses all its I/O credit balance, the maximum IOPS performance of the volume remains at the baseline IOPS performance level and it will replenish the burst bucket during idle times or when performing low levels of I/O.

If you notice that your volume performance is frequently limited to the baseline level (due to an empty I/O credit balance), you should consider two options.
* By increasing the volume size to a higher one. Since gp2 baseline performance is relying on the size of the volume this will work. You can try this by using the “Modify volume” option under EBS volumes.
* By switching to a more robust gp3 volume.

## gp3 Volumes

This is the next generation general purpose SSD volume type.

**Unlike gp2, the baseline performance is independent of the storage capacity and provides the baseline performance of 3000 IOPS irrespective of the storage size.   
Hence, there is no credit system with gp3 as you did find with gp2.**

This further provides 20% lower price than the gp2 volumes.

