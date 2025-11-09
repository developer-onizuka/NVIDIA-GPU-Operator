# NVIDIA-GPU-Operator

# 1. GPUのPass Through設定
### 1-1. デバイスIDの確認
```
$ lspci |grep NVIDIA
01:00.0 VGA compatible controller: NVIDIA Corporation GP104GL [Quadro P4000] (rev a1)
01:00.1 Audio device: NVIDIA Corporation GP104 High Definition Audio Controller (rev a1)
```
```
$ lspci -nnk -s 01:00.0 |grep VGA
01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GP104GL [Quadro P4000] [10de:1bb1] (rev a1)
$ lspci -nnk -s 01:00.1 |grep Audio
01:00.1 Audio device [0403]: NVIDIA Corporation GP104 High Definition Audio Controller [10de:10f0] (rev a1)
```
### 1-2. /etc/default/grubの編集
/etc/default/grubを以下のように編集し、Pass Throughを有効にする。編集後、sudo update-grubを実行する。
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash intel_iommu=on vfio_iommu_type1.allow_unsafe_interrupts=1 iommu=pt"
```
### 1-3. VGAとAudioのIDをvfio-pciに割り当てる。
確認したデバイスIDをもとに、vfio-pciに割り当てる。
```
echo "options vfio-pci ids=10de:1bb1,10de:10f0" | sudo tee /etc/modprobe.d/vfio-pci.conf
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
01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GP104GL [Quadro P4000] [10de:1bb1] (rev a1)
	Subsystem: NVIDIA Corporation GP104GL [Quadro P4000] [10de:11a3]
	Kernel driver in use: vfio-pci
	Kernel modules: nvidiafb, nouveau, nvidia_drm, nvidia
$ lspci -nnk -s 01:00.1
01:00.1 Audio device [0403]: NVIDIA Corporation GP104 High Definition Audio Controller [10de:10f0] (rev a1)
	Subsystem: NVIDIA Corporation GP104 High Definition Audio Controller [10de:11a3]
	Kernel driver in use: vfio-pci
	Kernel modules: snd_hda_intel
```
# 2. VagrantによるKubernetesクラスタの起動
```
git clone https://github.com/developer-onizuka/NVIDIA-GPU-Operator
cd NVIDIA-GPU-Operator
cd kubernetes
vagrant up --provider=libvirt
```
状況に応じて、Vagrantfileを以下のように修正してください。
```
Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-24.04"
  config.ssh.forward_x11 = true
  config.vm.synced_folder ".", "/vagrant", type: "rsync"  <---この行を追加する。
```
# 3. Vagrantによるnfsサーバーの起動
```
git clone https://github.com/developer-onizuka/NVIDIA-GPU-Operator
cd NVIDIA-GPU-Operator
cd nfs
vagrant up --provider=libvirt
```
# 4. Install Helm chart at Master node
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 \
&& chmod 700 get_helm.sh && ./get_helm.sh
```
```
helm repo add nvidia https://nvidia.github.io/gpu-operator && helm repo update
```
# 5. Install GPU Operator at Master node
You can find the latest version of GPU Operator below:<br>
- https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/release-notes.html
```
VERSION="580.82.07"
RELEASE_NAME="gpu-operator-v$(echo $VERSION | tr '.' '-')"
helm install --wait $RELEASE_NAME nvidia/gpu-operator --set driver.version=$VERSION
```
### 5-1. Check if gpu-operator properly finished deploying DaemonSet
```
kubectl get pods -o wide
```
### 5-2. Uninstall GPU operator (if needed)
```
helm delete $(helm ls -n default | awk '/gpu-operator/{print $1}') -n default
```
