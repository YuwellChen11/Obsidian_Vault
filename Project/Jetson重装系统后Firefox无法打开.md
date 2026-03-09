**解决方案：卸载 Snap 版，改用 Mozilla 官方 PPA 的 deb 版**
1. 完全卸载 Snap 版
```bash
sudo snap remove firefox
sudo apt remove firefox -y
sudo apt autoremove -y
```
2. 添加 Mozilla 官方 PPA
```bash
sudo add-apt-repository ppa:mozillateam/ppa
```
3. 设置优先使用 deb 版而非 snap
```bash
echo '
Package: *
Pin: release o=LP-PPA-mozillateam
Pin-Priority: 1001
' | sudo tee /etc/apt/preferences.d/mozilla-firefox

echo 'Unattended-Upgrade::Allowed-Origins:: "LP-PPA-mozillateam:${distro_codename}";' | sudo tee /etc/apt/apt.conf.d/51unattended-upgrades-firefox
```
4. 安装 deb 版 Firefox
```bash
sudo apt update
sudo apt install firefox -y
```