# Arch Linux install 2023 08

```bash
# актуальное время и шрифт, чтобы глаза не ломать.
setfont ter-132n
timedatectl set-ntp true
timedatectl set-timezone Europe/Moscow
```

1. Очистка диска
```bash
# смотрим диск, на который хотим поставить, в нашем случае
fdisk -l # или
lsblk 
gdisk /dev/sda
x # экспертный режим
z # удаляем GPT
# альтернативный вариант
wipefs -a /dev/sda
```
2. Размечаем диск
```bash
# без swap
cfdisk /dev/sda
# выбираем `gpt`, 300М на 1 раздел /dev/sda1 - boot, все остальное под второй и основную систему. /dev/sda2
# /dev/sda1 - type `EFI System`
# /dev/sda2 - type `Linux root (x86-64)`
# форматируем
mkfs.fat -F32 /dev/sda1
mkfs.ext4 /dev/sda2
```

3. Установка
```bash
mount /dev/sda2 /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot

pacstrap -i /mnt base base-devel linux linux-firmware
genfstab -U -p /mnt >> /mnt/etc/fstab
```

4. Настройка
```bash
arch-chroot /mnt
pacman -Sy vim
EDITOR=vim
# часовой пояс
ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime
hwclock --systohc --utc
# локаль
vim /etc/locale.gen
# разкомментировать `en_US.UTF-8` и `ru_RU.UTF-8`
locale-gen
echo LANG=en_US.UTF-8 > /etc/locale.conf
export LANG=en_US.UTF-8
# имя компа
echo arch > /etc/hostname
    vim /etc/hosts
    127.0.0.1         localhost
    ::1               localhost
    127.0.1.1         arch.localdomain       arch
# сеть через dhcpcd
pacman -S dhcpcd
systemctl enable dhcpcd.service
# Пользователи
passwd # ставим пароль
useradd -m -g users -G wheel,audio,video,optical,storage -s /bin/bash overlord
pacman -S sudo
visudo
# разкомментировать
%wheel ALL=(ALL:ALL) ALL
passwd overlord
```
5. Установка загрузчика
```bash
bootctl install
vim /boot/loader/loader.conf
    timeout 5
    console-mode keep
    default arch
vim /boot/loader/entries/arch.conf
    title   ARCH
    linux   /vmlinuz-linux
    initrd  /initramfs-linux.img
    options root=/dev/sda2 rw
```

```bash
exit
umont -R /mnt/boot
umont -R /mnt
reboot
```


### yay install
```bash
sudo pacman -Sy git  
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
yay -Syu neovim
```
### bash-completion и fzf
```bash
yay -S fzf
sudo pacman -S bash-completion
source /usr/share/bash-completion/bash_completion
```
## dwm, st, slstatus, dmenu
```bash
sudo pacman -S xorg xorg-xinit nerd-fonts
# sudo pacman -S xorg-server xorg-xinit libx11 libxinerame libxft webkit2gtk ttf-font-awesome
mkdir suckless
cd suckless
git clone https://git.suckless.org/st
git clone https://git.suckless.org/dmenu 
git clone https://git.suckless.org/dwm
git clone https://git.suckless.org/slstatus
```
## dwm
```bash
cd st # аналогично для dwm, dmenu, slstatus
sudo make clean install
```

## exec startx, dwm, slstatus
```bash
nvim .bash_profile
    if [ -z "${DISPLAY}" ] && [ "${XDG_VTNR}" -eq 1 ]; then
        exec startx
    fi

nvim .xinitrc
    # раскладка
    setxkbmp -layout "us,ru" -option "grp:alt_shift_toggle"
    # отключаем режим энергосбережения и блокировку экрана
    xset s off
    xset -dpms
    # переменная окружения для Java
    export _JAVA_AWT_WM_NONREPARENTING=1
    # slstatus, dwm
    exec slstaus &
    exec dwm
```
