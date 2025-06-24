# 下載 Rocky Linux 10
cd /var/lib/vz/template/iso/
wget https://dl.rockylinux.org/pub/rocky/10/images/x86_64/Rocky-10-GenericCloud-Base.latest.x86_64.qcow2

# 建立 Rocky Linux 10 模板
qm create 901 --name rocky10-template --memory 8192 --cores 20 --net0 virtio,bridge=vmbr0 --machine q35 --bios ovmf

# 匯入磁碟
qm importdisk 901 Rocky-10-GenericCloud-Base.latest.x86_64.qcow2 ssd1-zfs-pool

# 設定磁碟
qm set 901 --scsihw virtio-scsi-pci --scsi0 ssd1-zfs-pool:vm-901-disk-0

# 調整磁碟大小到 20G
qm resize 901 scsi0 20G

# UEFI 磁碟
qm set 901 --efidisk0 ssd1-zfs-pool:1,efitype=4m,pre-enrolled-keys=1

# 開機設定
qm set 901 --boot order=scsi0

# Cloud-Init
qm set 901 --ide2 ssd1-zfs-pool:cloudinit

# 使用者和網路設定 (IP 用 .91)
qm set 901 --ciuser rocky --cipassword 123456
qm set 901 --ipconfig0 ip=192.168.200.91/21,gw=192.168.200.1
qm set 901 --nameserver 8.8.8.8

# 顯示設定
qm set 901 --vga virtio

# 轉成模板
qm template 901
