# CloneBadge
clone mirage classic 1k badge



Clone Mifare Classic 1k Badge 

Steps:

https://www.latelierdugeek.fr/2017/07/12/rfid-le-clone-parfait/comment-page-6/amp/

Error with mfoc: 
* use mfcuk like here: https://mathieu.daitauha.fr/nfc-mifare-classic-1k/
* Use extended default keys: https://net-security.fr/security/copier-un-badge-nfc-avec-kali-linux-android/ (better)
Main commands: 
wget https://github.com/ikarus23/MifareClassicTool/blob/master/Mifare%20Classic%20Tool/app/src/main/assets/key-files/extended-std.keys


mfoc -f extended-std.keys -O net-security.dmp

Key—> 8829da9daf76

Error:

```
Warning unlock command 1/2 failed / no acknowledged.
Writing 64 blocks | Failure to write to data block 4
x
```

To solve: reconfigure libnfc from master branch, steps:

Step 1: Uninstall the Existing libnfc

If you have previously installed libnfc from your package manager, remove it first to avoid conflicts.

sudo apt remove libnfc-bin libnfc-dev

Step 2: Clone the libnfc Repository

To get the latest version from the master branch, clone the libnfc GitHub repository:

git clone https://github.com/nfc-tools/libnfc.git
cd libnfc

Step 3: Build from Source

Now, build the latest version from source to ensure all Gen3-related commits are included.

	1.	Install dependencies:

sudo apt update
sudo apt install autoconf automake libtool pkg-config libpcsclite-dev


	2.	Prepare the build environment:

autoreconf -vis
./configure


	3.	Compile and install:

make
sudo make install
sudo ldconfig



Step 4: Attempt Writing Again

After installing the latest version of libnfc from the master branch, try writing to the Gen3 fob again:

nfc-mfclassic W a originalnew.dmp new.dmp

Key - 8829da9da76


IF UID DOESNT WANT TO CHANGE:

nfc-mfcsetuid *write UID*



What is recommended to buy badges gen3 from la boutique de l’atelier du geek (blue ones) do nfc-mfsetuid (uid) and alternate between libnfc8 and 1.7.1


Do nfc-mfsetuid in libnfc1.8.0 
Do nfc-mfclassic w a in libnfc1.7.1



Libnfc1.80 correctly copies uid (in modifiable badges) 
Libnfc1.7.1 correctly modifies rest of data 






Not recommended :
Also you can manually write the block with:
 But the nfc-mfsetuid and the dd if=original.dmp of=clone.dmp bs=16 count=1 conv=notrunc


Shell code to create a file for every block:

#!/bin/bash

# Check if the input file exists
if [ ! -f originalnew.dmp ]; then
    echo "Error: originalnew.dmp not found!"
    exit 1
fi

# Create blocks from originalnew.dmp
for i in {0..9}; do
    dd if=originalnew.dmp of=block_$i.bin bs=16 count=1 skip=$i
done

echo "Blocks extracted successfully."


Shell code to change the value of every block:

#!/bin/bash

# Define the starting and ending block numbers
START_BLOCK=3
END_BLOCK=63

# Loop through the specified range of blocks
for ((i=START_BLOCK; i<=END_BLOCK; i++)); do
    dd if=block_$i.bin of=newest.dmp bs=16 count=1 seek=$i conv=notrunc
    echo "Written block $i to newest.dmp"
done

echo "All specified blocks have been written to newest.dmp."







CONCLUSION:

The scanning systems works in a way that updates the 63rd block of the data. So with each scan the data of the badge changes.

When trying to clone the original badge (OB)to the cloned badge (CB), the information on the OB was outdated because with each scan of the CB got updated making the OB not work anymore. 
Conclusion only one badge can work at a time. 

Solution: identify the changes between every scan (10 -20 scans), if it occurs in a predictable pattern. Try and automate the CB to change block 63 with every scan.