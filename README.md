# NVIDIA-GPU-Operator

### 1-1. IOMMUグループの確認
```
for d in /sys/kernel/iommu_groups/*/devices/*; do n=${d#*/iommu_groups/*}; n=${n%/*}; printf 'IOMMU Group %s: %s\n' "$n" "$(basename $d)"; done | grep '01:00'
```
### 1-2. /etc/default/grubの編集
/etc/default/grubを以下のように編集する。
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash intel_iommu=on vfio_iommu_type1.allow_unsafe_interrupts=1 iommu=pt vfio-pci.ids=10de:1cb1,10de:0fb9"
```
### 1-3. 10de:1cb1 (VGA) と 10de:0fb9 (Audio) のIDをvfio-pciに割り当てる。
```
echo "options vfio-pci ids=10de:1cb1,10de:0fb9" | sudo tee /etc/modprobe.d/vfio-pci.conf
```
### 1-4. nouveauを完全にブロックする。
```
echo "blacklist nouveau" | sudo tee /etc/modprobe.d/blacklist-nouveau.conf
echo "options nouveau modeset=0" | sudo tee -a /etc/modprobe.d/blacklist-nouveau.conf
```
### 1-5. snd_hda_intelをブロックする。
```
echo "blacklist snd_hda_intel" | sudo tee /etc/modprobe.d/blacklist-snd-hda-intel.conf
```
### 1-6. 再起動後も vfio-pci を自動ロードする
```
echo "vfio-pci" | sudo tee /etc/modules-load.d/vfio-pci.conf
```
### 1-7. 初期RAMディスクを更新する。
```
sudo update-initramfs -u
```
### 1-8. Reboot
```
reboot
```
### 1-9. lspciによる確認
vfio-pciがロードできているかを確認する。
```
$ lspci -nnk -s 01:00.0
01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GP107GL [Quadro P1000] [10de:1cb1] (rev a1)
	Subsystem: NVIDIA Corporation GP107GL [Quadro P1000] [10de:11bc]
	Kernel driver in use: vfio-pci
	Kernel modules: nvidiafb, nouveau

$ lspci -nnk -s 01:00.1
01:00.1 Audio device [0403]: NVIDIA Corporation GP107GL High Definition Audio Controller [10de:0fb9] (rev a1)
	Subsystem: NVIDIA Corporation GP107GL High Definition Audio Controller [10de:11bc]
	Kernel driver in use: vfio-pci
	Kernel modules: snd_hda_intel
```
# 2. Vagrantによる仮想マシンの起動
```
git clone https://github.com/developer-onizuka/NVIDIA-GPU-Operator
cd NVIDIA-GPU-Operator
cd kubernetes
vagrant up --provider=libvirt
```

