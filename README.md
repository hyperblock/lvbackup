# lvbackup

A tool to backup and restore LVM2 thinly-provisioned volumes.

This tool can do full backup and incremental for thin volume. In full backup mode, lvbackup only saves the blocks used by the thin volume to minimize the backup size. In incremental backup mode, lvbackup detects the created/changed/deleted blocks from source volume to target volume. 

All the data, which are written to standard output, can be compressed and saved as local file, or transported to anothing host using network tools, like nc and ssh. 

The volumes can be restored from single full backup file, or multiple continus incremental backup files. As the volume UUID is recorded in the backup, broken incremental backup chain will be detected and reported.

The incremental backup method is inspired by [lvmsync](https://github.com/mpalmer/lvmsync). 

The sub commands is inspired by zfs send/recv.

## Usage

#_NEED RUN AS ROOT_

Create thin snapshot for thin volumes as usual (it's better to freeze the file system before creating snapshot

    lvcreate -s -n {SNAP_NAME} {VG_NAME}/{LV_NAME}
    
<!Or send it to another host by network:
 
    lvbackup send -v {VG_NAME} -l {SNAP_NAME} | nc {OTHER_HOST}>
    
You can create incremental backup:
  
    lvbackup send -v {VG_NAME} -l {SNAP_NAME} -i {OLD_SNAP_NAME} --head {HEADER_FILE} -o {OUTPUT_FILE}
	
	eg. lvbackup send -v vg001 -l sp001 -i vol0 --head header -o backup_sp001_0

<!To check the info of backup file:
    
    lvbackup info {BACKUP_FILE}>

To restore the volume from backup, you need have the old volume. Then, run recv subcommand: 

    lvbackup recv -v {VG_NAME} -p {POOL_NAME} -l {LV_NAME} -i {BACKUP_FILE}

<! restore the volume from backup chain:i

    cat {FULL_BACKUP} {DELTA_0} {DELTA_1} ... | \
    lvbackup recv -v {VG_NAME} -p {POOL_NAME} -l {LV_NAME}>

Please note that the chunk size of thin pool for restoring must be equal to that in the backup files.

##Exported Format of Layers (backup file)

>HYPERLAYER/1.0
><key>: <value>
><key>: <value>
><key>: <value>
><key>: <value>
>
><offset> <length>
><data of $length*512 bytes>
><offset> <length>
><data of $length*512 bytes>
><offset> <length>
><data of $length*512 bytes>
>...


<!
## TODO

* Feature: merge continus incremental backups into single one
* Feature: display backup/restore progress
* Enhance: add unit tests
* Enhance: improve the error message displaying
* Enhance: add document about the format of stream data
>
