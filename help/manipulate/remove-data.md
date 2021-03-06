

## erase full partition / filesystem or device

Deleting files merely makes the data hard to find and liable to be overwritten. 
If you are concerned about wiping your data to avoid casual recovery using only software tools, 
the most important task is to ensure that ALL data is physically, actually overwritten. 

See also:

* why you need to overwrite [GitHub/Lubuild.files/help/manipulate/disk-recovery-and-forensics.md]
* how proper total wiping is hard on flash [GitHub/Lubuild.files/help/manipulate/flash-drives-and-SSDs.md]


### simple overwrites with dd
```
# depending on the device and your system you may need to sudo some commands
# all the commands below need sdXY changing to your real device
# ENSURE you are really overwritting data you no longer want - wrong choices will cause you pain

# with ANY of the commands below, if you are curious to check the dd progress, use another window to...
watch -n30 'sudo pkill -usr1 dd'

# speedy overwrite with zeros
dd if=/dev/zero of=/dev/sdXY

#### random overwrites
# if you plan to use encryption, then it is recommended you fill the space with random data first 
# to reduce the ability for others to understand anything about the size of contents of the encrypted area 

# rather SLOW overwrite with quality psuedo-random data stream
dd if=/dev/urandom of=/dev/sdXY bs=1M

# use openssl to encrypt the zeros (much QUICKER simple pseudo-RANDOM - better than patterns)
head -c 32 /dev/urandom | openssl enc -rc4 -nosalt -in /dev/zero -pass stdin | dd of=/dev/sdXY bs=1M
# credit - http://askubuntu.com/a/359547

# use built-in cryptsetup to access the kernel dm-crypt encryption for quick psuedo-random
sudo cryptsetup open --type plain /dev/sdXY container --key-file /dev/random
dd if=/dev/zero of=/dev/mapper/container
sudo cryptsetup close container 
# credit - https://wiki.archlinux.org/index.php/Dm-crypt/Drive_preparation#dm-crypt_wipe_on_an_empty_disk_or_partition


```
### multiple overwrite

The following utilities perform multiple 'pass' wipes, repeatedly overwriting the same areas. 
They are based on theories from Guttmann in the late 1990s that erased data could be recovered 
using magnetic microscope technology. Later [experimnents by Wright in 2008](https://digital-forensics.sans.org/blog/2009/01/15/overwriting-hard-drive-data/) debunked such theories.

If you are unsure which case to beleive, you might consider either of the following contrasting viewpoints:
* does it cost me much extra time to multi-pass wipe my disk
* what is the chance that someone will want to spend time examining my old disk with expensive equipment

If you want some more of other people's opinions then try [http://security.stackexchange.com/questions/10464]

Remember, as mentioned above, overwriting multiple times will not help if it misses an area containing data. 
Some modern storage technologies like to manage things themselves and make it hard to access all writable areas. 
Perhaps this is why people and organisations who want to take NO risk will physically shred or destroy old devices?

```
#### shred

# * part of GNU Coreutils
# * installed by default under Lubuntu
# * help - [https://www.gnu.org/software/coreutils/manual/html_node/shred-invocation.html]

# basic example
shred -vu MyFiles*

# select the device from which you want to totally wipe the filesystem  (but leave dev node intact)
MY_DRIVE=/dev/sdX
# Make totally SURE this is the right device FIRST
mount -l | grep $MY_DRIVE
df | grep $MY_DRIVE
# say goodbye to all contents
shred -v $MY_DRIVE

# claims to work as is for floppy drive 

#### wipe

# * in ubuntu repos
# * help - http://manpages.ubuntu.com/manpages/hardy/man1/wipe.1.html

# wipe man pages recommends telling the util how much to overwrite as floppies don't always correctly report their dimensions
wipe -l 1440k /dev/sdX
# wipe can follow symlinks (if you have a psuedo-device like /dev/floppy) with option  -D

```
