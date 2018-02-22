# Storage on DigitalOcean

Before we talk about backup, restore, and disaster recovery planning, it is important to explain the storage options available on DigitalOcean. Not all storage is created equal and it is important to know the implications of each storage option.

There are three main considerations for each storage option. No one option has all these points perfectly solved. Each option has a compromise in one or more of the following areas:
* **Performance:** speed of input/output
* **Security:** encryption of data when not in use
* **Redundancy:** multiple copies of data exists and is resilient to corruption

### Local Droplet Storage
This is the most common form of storage on DigitalOcean. Currently, each Droplet is assigned a virtual disk that is located physically on the hypervisor. The hypervisors have redundant arrays of independent SSD drives (RAID). There are a few version of RAID in use within our hypervisor fleet, but they all offer protection over from a failing disk. It is statistically rare that a hypervisor would experience a total RAID failure, but it can happen. Because of this reason, having your data in more than one place is crucial.

<!-- TODO: Can we publish our failure rates?  -->

The local Droplet storage is the highest performing storage option that is available on DigitalOcean. Our entire fleet of hypervisors uses enterprise grade SSD drives and we continue to evaluate newer, faster options as they are more widely available.

_It is worth noting that the virtual disk located on the hypervisor RAID is not encrypted at rest._

### Block Storage Volumes
Local Droplet storage sizes increase in a linear fashion with other resources. A larger Droplet will have more local storage. Often you may find that you need more storage on a smaller Droplet. Block Storage Volumes allow you to attach additional drives to Droplets. 

There are a few main benefits to storing your data on a Volume:
* The storage cluster has a built in redundancy ensuring multiple copies of your data within the cluster
* Volumes and Volume Snapshots are encrypted at rest with AES-256 bit LUKS encryption within the storage cluster
    * The file system on the Volume can also be placed in a LUKES encrypted drive
* Volumes can be increased independently as needed up to 16TB
* A Volume can be detached from one Droplet and attached to a different Droplet in the same region easily

You can see that the redundancy and security of the data is increased with Volumes. Also, it's easy to move data to a new Droplet should an existing Droplet begin to exhibit problems. 

Block Storage Volumes are limited in performance when compared to the local Droplet storage. The storage cluster that hosts Volumes are equipped with 100% solid state drives (SSD), but there is an inherent performance as the Volumes are attached to Droplets over network connections. Volumes may not be an ideal storage solution for use cases requiring an intense amount of input/output operations per second (IOPS). Placing the files for a database on a Volume is one example of a heavy I/O use case.

NOTE: Because a Volume is attached over a network connection to a Droplet, the Volume and Droplet need to be in the same region. 
<!-- TODO: Region List with Volume availability   -->

Block Storage is literally a block of storage. You run a file system on top of the device and it the Droplet interprets it just as it would an additional hard drive on a physical server. This also means that you not only have to be mindful of the file system, but the size of the Volume as well.  **What if the file system has some level of corruption?** _The data is copied in multiple places of the storage cluster, but it is corrupted at the file system level and you have multiple copies of bad data._ If you resize the Volume, you also have to expand the file system as well. What if you used storage for thousands of images or for organizing logs? Object Storage using Spaces on DigitalOcean may be a better storage option.

_It is worth noting that redundancy of storage is not a substitute for backing up data. We will cover more aspects of data backup and recovery in the next chapter._


### Object Storage with Spaces
Up to this point, we have discussed storage options that are available to a single Droplet at a time. The Droplet is the main mechanism for accessing your data. Object Storage does away with this. Amazon pioneered mainstream Object Storage with their S3 product. <!-- TODO: Trademark/copywrite needed?  --> S3 stands for _Simple Storage Service_. While the concept is simple in nature, using S3 hasn't always been use friendly. Luckily a large amount of third party software writers have built software and libraries for interacting with S3 and the subsequent S3-compatible services that have sprung up. 

Spaces is DigitalOcean's version of Object Storage. Spaces is built on top of an open source project called Ceph. Ceph has multiple components and it is important to look at the components to understand the characteristics of this storage option. 

* The Object Storage Device (OSD) is the physical / logical drive storing data. Spaces is the only storage option on DigitalOcean that uses hard disk drives (HDD) instead of solid state drives (SSD). The data is stored across multiple OSD's
* The RADOS Gateway (RGW) is what provides the interface with the storage objects and acts as an S3-compatible API gateway
* There are other monitor, map, and pool aspects that keep the cluster functioning, but are less important for our discussion

The Spaces backend is made up of many OSD's. The data is stored is encrypted and is managed as individual files. The multiple copies of each object are compared daily to provide data integrity. There should no data corruption because if one copy doesn't match up, the cluster will resolve the issue. <!-- TODO: accurate? -->

Because any computer on the internet can send requests to the RGW's, there is no need for a Space to exist in the same region as a Droplet. For example Droplets in NYC1 or NYC2 will have fast access to the Spaces in NYC3 through our NYC regional fiber ring. 

_It is worth noting that bandwidth for outbound traffic within the regional fiber rings is free of charge. The current regional fiber rings are NYC: [NYC1, NYC2, and NYC3] and Europe: [LON1, FRA1, AMS2, and AMS3]_

So far this storage option seems to be the best. We do not have to worry about a single Droplet being a point of failure. We don't have to worry about data corruption and the data stored is encrypted for security. While we have redundancy and security simplified with Spaces, performance is going to be the compromise. All requests to store or pull files from the Spaces backend goes through the RGW's. While we strive for a 99.99%<!-- TODO: ?? --> availability with the Spaces API, it is not the best use for serving files at a high rate of request per minute. The ideal situation is to place your Spaces behind a Content Delivery Network (CDN) to speed and availability. <!-- Hoping for DigitalOcean CDN option by publising -->

Spaces is going to be great for hosting images and static HTML files behind a CDN and for storing logs and backups. 

If you want to learn more about Ceph, DigitalOcean's very own Anthony D'Artri and Viaav Behembre are co-authors on [Learning Ceph - Second Edition: Unifed, scalable, and reliable open source storage solution](https://www.amazon.com/Learning-Ceph-scalable-reliable-solution-ebook/dp/B01NBP2D9I). We might be able to get them to sign your Kindle for you as well. 

_It is worth noting that Volumes' storage clusters are also running on Ceph._


### The Best Storage Option
So after all that, there is one clear best choice right? Unfortunately, it's not that easy. The goal with this chapter is to outline ways to best use all of the storage options for their prominent features. Knowing the downfalls of each storage option allows you to plan accordingly. The next chapter talks about this. Planning your data backup and recovery needs properly hedges against corruption or data loss. 


<!-- TODO: distributed file systems like GlusterFS ? -->


