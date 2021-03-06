## Add Extra Disk and Set up Blobfuse on a DSVM

### Attach extra data disk to DSVM
1. Follow the following sections in these instructions on your DSVM: [attach data disk](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/attach-disk-portal#attach-a-new-disk)
* Find the virtual machine
* Attach a new disk (200GB is the default value I use but it will depend on the size of your data)
* Connect to the Linux VM to mount the new disk
* Find the disk
* Prepare a new empty disk
* Mount the disk
* Verify the disk

### Set up blobfuse on DSVM to connect to storage account
1. The full explanation of blobfuse can be found here: [blobfuse](https://microsoft-my.sharepoint.com/:w:/p/mifore/EZlNemxyiLlMu2XZT9qSGUgB-gJTGcAWl8JD873avRS9vg?e=4%3Aw1pDQ8&at=9&CID=70F8643E-DB62-4AB2-B5E8-CCF4C13A8697&wdLOR=c0A7DAC7F-8A99-412A-873A-7760D313397C)
2. Create a temporary directory that will store the cache of the data
```
sudo mkdir /mnt/resource/blobfusetmp -p
```
3. Give yourself permissions to that folder
```
sudo chown <username> /mnt/resource/blobfusetmp
```
4. Create a cfg file that stores the credentials associated with your storage account. The accountKey is found under Storage Account->Security & Networking->Access Keys->key1->Key.
```
vim ~/fuse_connection.cfg

accountName <name of storage account>
accountKey <paste in accountKey>
containerName <name of container you want to mount>
```
5. Modify permissions of that file
```
chmod 600 fuse_connection.cfg
```
6. Now create a directory where you want to mount the container. This will be the directory where you want to access the files in the container and should be located in the new disk drive that you attached.
```
cd /datadrive
mkdir text_data
```
7. Mount with this command. Check to make sure you can now access the files. Now, when you add data to /datadrive/text_data it will automatically be uploaded to the storage account and vice versa. (Note: When you restart the VM you will lose the mount and therefore have to run this command again).
```
blobfuse /datadrive/text_data --tmp-path=/mnt/resource/blobfusetmp --config-file=~/fuse_connection.cfg -o attr_timeout=240 -o entry_timeout=240 -o negative_timeout=120
```
8. To mount the drive so that it is persistent even when you restart VM. Add this line to the bottom of /etc/fstab.
```
sudo vim /etc/fstab

blobfuse /datadrive/text_data fuse defaults,_netdev,--tmp-path=/mnt/resource/blobfusetmp,--config-file=~/fuse_connection.cfg,--log-level=LOG_DEBUG,allow_other 0 0
```
9. To unmount type this command:
```
sudo umount /datadrive/text_data
```
