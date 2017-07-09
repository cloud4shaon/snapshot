Place both the scripts (AzureSnapFunctions.ps1 and AzureStorageFunctions.ps1) at the same location. 
Run the script "AzureSnapFunctions.ps1" , this will do nothing but will load in the memory and also will load the helper script
Run LogMeIn, Mandatory parameters are Subscription Name, Account Name, Password
Description of other functions:
-------------------------------

New-AzureRmVmSnap : Mandatory parameters are $VMName and $SnapshotName. There is a Force Switch that can be used if VM is running.
Eg: New-AzureRmVmSnap -VMname <vm1> -SnapshotName <mysnapshot>
Result: 
a) Shut down the VM
b) Create a new GUID for the Snapshot Set
c) Create an array $DiskUriList wherein URI of OS and Data Disks of VM will be added.
d) Now, for each $DiskURI it will create the following objects
$DiskInfo = Get-DiskINfo $DiskURI
$StorageContext = Get-StorageContextForUri -DiskUri $DiskURI
$DiskBlob = Get-AzureStorageBlob -Container $DiskInfo.ContainerName -Context $StorageContext -Blob $DiskInfo.VHDName
e) Finally it will create the snapshot set and store it in a object
$snapshot = $DiskBlob.ICloudBlob.CreateSnapshot(). The ICloudBlob.CreateSnapshot() function is an inbuilt function which basically creates a fresh "Snapshaot Set" > adds the VHD
to the set and creates a copy of the VHD in the same location. The VM object is pointed to the copied disk. All future writes will occur in this copied disk.
f) The Snapshot metadata will be written in a Table in the same Storage Account. This metadata will help us later to revert/delete the snapshot set.
g) The VM will then be powered on using Start-AzureRmVm


Get-AzureRmVmSnap: Mandatory parameter is $VMName. There is a Force Switch for $GetBlobs. 
This will call the other helper functions and display the Snapshot set details.

Delete-AzureRmVmSnap: This will simply offload the writes from the cloned disks to the base VHDs. Then it will delete the cloned disks and the Snapshot metadata.

Revert-AzureRmVmSnap: This will display the Snapshot Sets one by one to the user and wait for an input. When the correct Snapshot Set is selected it will stop the VM.
Point the VM to use the disks in the selected Snapshot Set and start the VM. 

Scenario 1: Take a snapshot before undertaking some tasks on a VM
- Run New-AzureRmVmSnap
Scenario 2: The tasks were successful. Merge the changes to the base disks and delete the snapshot blobs
- Delete-AzureRmVmSnap
Scenario 3: Tasks failed , OS is not responding or getting registry / other error. Can you roll back the changed ?
- Revert-AzureRmVmSnap
Check if VM is working fine as expected.
-Delete-AzureRmVmSnap
This will delete the snapshot blobs, however changes will not get merged in this case. 


