# 下載 Rocky 10 (如果還沒有)
cd /var/lib/vz/template/iso/
# wget https://dl.rockylinux.org/pub/rocky/10/images/x86_64/Rocky-10-GenericCloud-Base.latest.x86_64.qcow2

# 刪除舊的重建
qm destroy 901

# 建立 VM - **重點：加上 CPU host**
qm create 901 --name rocky10-template --memory 8192 --cores 20 --cpu host --net0 virtio,bridge=vmbr0 --machine q35 --bios ovmf

# 匯入磁碟
qm importdisk 901 Rocky-10-GenericCloud-Base.latest.x86_64.qcow2 ssd1-zfs-pool

# 設定磁碟
qm set 901 --scsihw virtio-scsi-pci --scsi0 ssd1-zfs-pool:vm-901-disk-0

# 調整磁碟大小到 20G
qm resize 901 scsi0 20G

# UEFI 磁碟
qm set 901 --efidisk0 ssd1-zfs-pool:1,efitype=4m,pre-enrolled-keys=1

# 開機順序
qm set 901 --boot order=scsi0

# Cloud-Init
qm set 901 --ide2 ssd1-zfs-pool:cloudinit

# 使用者和網路設定
qm set 901 --ciuser rocky --cipassword 123456
qm set 901 --ipconfig0 ip=192.168.200.91/21,gw=192.168.200.1
qm set 901 --nameserver 8.8.8.8

# 顯示設定
qm set 901 --vga virtio --serial0 socket

# 啟動測試
qm start 901

# 測試成功後轉成模板
qm template 901
----
# 複製新 VM
qm clone 901 102 --name my-rocky10 --full

# 改 IP
qm set 102 --ipconfig0 ip=192.168.200.92/21,gw=192.168.200.1

# 啟動
qm start 102

# SSH 連線
ssh rocky@192.168.200.92
