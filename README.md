# NVIDIA-GPU-Operator

# 1. IOMMUグループの確認
```
for d in /sys/kernel/iommu_groups/*/devices/*; do n=${d#*/iommu_groups/*}; n=${n%/*}; printf 'IOMMU Group %s: %s\n' "$n" "$(basename $d)"; done | grep '01:00'
```
# 2. 10de:1cb1 (VGA) と 10de:0fb9 (Audio) のIDをvfio-pciに割り当てる。
```
echo "options vfio-pci ids=10de:1cb1,10de:0fb9" | sudo tee /etc/modprobe.d/vfio-pci.conf
```
# 3. nouveauを完全にブロックする。
```
echo "blacklist nouveau" | sudo tee /etc/modprobe.d/blacklist-nouveau.conf
echo "options nouveau modeset=0" | sudo tee -a /etc/modprobe.d/blacklist-nouveau.conf
```
# 4. snd_hda_intelをブロックする。
```
echo "blacklist snd_hda_intel" | sudo tee /etc/modprobe.d/blacklist-snd-hda-intel.conf
```
# 5. 再起動後も vfio-pci を自動ロードする
```
echo "vfio-pci" | sudo tee /etc/modules-load.d/vfio-pci.conf
```
# 6. 初期RAMディスクを更新する。
```
sudo update-initramfs -u
```
# 7. 


