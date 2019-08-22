# Machine
This installation uses the AWS i3.xlarge instance because it has a solid state hard drive which is large enough
- i3.xlarge ($0.374 per Hour)
- 1 x 950 NVMe SSD Storage
- 4 CPU
- 30.5 GB RAM

# Software (OS)
```
sudo apt-get update
sudo apt-get -y upgrade
sudo apt-get install -y build-essential
```

# Auxillary hardware
View NVMe volume (which is not yet mounted/mapped/formatted)
```bash
lsblk
```
There is a ~300Gb drive with the name of nvme0n1
```bash
nvme0n1     259:2    0 279.4G  0 disk 
```
Create a file system
```bash
sudo mkfs -t ext4 /dev/nvme0n1 
```
Part of the output from the above mkfs command will include the Filesystem UUID. Cut and paste this UUID because it will be used in an upcoming command.
```bash
Filesystem UUID: 6f6177fe-947a-46a2-b593-c36dfaaa8407
```
Create an easily accesible mount point on the main drive (where the operating system runs) and then set the permissions of this mount point to the ubuntu user.
```bash
sudo mkdir /media/nvme
sudo chown -R ubuntu:ubuntu /media/nvme/
```
Ensure that this drive is mounted each time the system is restarted. Add this line to the */etc/fstab* file (remember the UUID from the previous step?).
```bash
UUID=6f6177fe-947a-46a2-b593-c36dfaaa8407 /media/nvme ext4 defaults 0 0
```
Once the above commands have succeeded, reboot the instance.
```bash
sudo shutdown -r now
```
After the reboot, see the mounted ~300Gb NVMe SSD using the df command
```bash
df -h
/dev/nvme0n1    275G   65M  260G   1% /media/nvme
```
```bash
#ensure that the /media/nvme directory is owned by ubuntu by typing ls -la /media/nvme If it is not then type the following command
sudo chown -R ubuntu:ubuntu /media/nvme/
```
Create directory for Ethereum data
```bash
mkdir /media/nvme/ethereum_data
```
# Software (Ethereum)
Install Ethereum
```bash
sudo apt-get install software-properties-common
sudo add-apt-repository -y ppa:ethereum/ethereum
sudo apt-get update
sudo apt-get install ethereum
```

Start Ethereum full node - in current terminal
```bash
geth --datadir "/media/nvme/ethereum_data" --maxpeers=50 --syncmode "full" --rpc --rpcport "8545" --rpcaddr "0.0.0.0" --rpccorsdomain "*" --cache 4096
```

Or to be able to start Ethereum full node and log out of console with Geth still running use the following twist on the above command

```bash
nohup geth --datadir "/media/nvme/ethereum_data" --maxpeers=50 --syncmode "full" --rpc --rpcport "8545" --rpcaddr "0.0.0.0" --rpccorsdomain "*" --cache 4096 >/dev/null 2>&1 &
```

Ensure that port 8545 is only visible to the smart contract search engine IP

Connect to geth  console
```bash
geth attach ipc:/media/nvme/ethereum_data/geth.ipc 
```

Note: See [this Ethereum Blog](https://blog.ethereum.org/2019/07/10/geth-v1-9-0/) for more information about blockchain size and sync times.
