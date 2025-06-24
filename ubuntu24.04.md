# 下載映像
cd /var/lib/vz/template/iso/
wget https://cloud-images.ubuntu.com/daily/current/noble-server-cloudimg-amd64.img

# 先刪除原本的 VM (如果需要重建)
qm destroy 900

# 重新建立 VM 用你的規格
qm create 900 --name ubuntu-template --memory 8192 --cores 20 --net0 virtio,bridge=vmbr0 --machine q35 --bios ovmf

# 匯入磁碟
qm importdisk 900 noble-server-cloudimg-amd64.img ssd1-zfs-pool

# 設定 VirtIO SCSI 和磁碟
qm set 900 --scsihw virtio-scsi-pci --scsi0 ssd1-zfs-pool:vm-900-disk-0

# 調整磁碟大小到 20G
qm resize 900 scsi0 20G

# 加入 UEFI 磁碟 (q35 + OVMF 需要)
qm set 900 --efidisk0 ssd1-zfs-pool:1,efitype=4m,pre-enrolled-keys=1

# 設定開機
qm set 900 --boot order=scsi0

# 加入 Cloud-Init
qm set 900 --ide2 ssd1-zfs-pool:cloudinit

# 設定使用者和網路
qm set 900 --ciuser ubuntu --cipassword 123456
qm set 900 --ipconfig0 ip=192.168.200.90/21,gw=192.168.200.1
qm set 900 --nameserver 8.8.8.8

# 設定顯示 (UEFI 用)
qm set 900 --vga virtio

# 轉成模板
qm template 900
