# Примечание
- "*" внутри команд - особое значение. Н-р: вместо *username* пишем свое имя пользователя

# 1. Установка archlinux

## 1.1 Подключение к интернету

Использовать либо раздачу с мобильного через модем, либо через iwd:
```bash
1$ iwctl
2[iwd]$ device list
3[iwd]$ station *устройство* scan
4[iwd]$ station *устройство* get-netwokrs
5$ *Ctrl + D*
6$ iwctl --passphrase *пароль* station *устройство* connect *SSID*
```

> `ping google.com` для проверки сети

## 1.2 Время

```bash
1$ timedatectl set-ntp trur
2$ timedatectl set-timezone Europe/Moscow
```
> `timedatectl status` - статус

## 1.3 Разметка диска

```bash
1$ fdisk -l
2$ cfdisk {GPT disklabel}
3$ ...
4$ mkfs.fat -F32 /dev/*efi system*
5$ mkfs.ext4 /dev/*linux filesystem*
```

Моя разметка: 
```bash
300M - EFI System - FAT32
Оставшееся - Linux Filesystem - ext4/btrfs
```

## 1.4 Монтирование
```bash
1$ mount --mkdir /dev/*efi system* /mnt/efi
2$ mount /dev/*linux filesystem* /mnt
```

## 1.5 Pacstrap
```bash
1$ pacstrap /mnt base linux-zen linux-firmware
```

## 1.6 Генерация Genfstab
```bash
1$ genfstab -U /mnt >> /mnt/etc/fstab
```

## 1.7 Смена корневого каталога
```bash
1$ arch-chroot /mnt
2$ pacman -S vim dhcpcd
```

## 1.8 Время, локали и хосты
```bash
1$ ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime
2$ hwclock --systohc

3$ vim /etc/locale.gen # Рвскомментировать: en_US.UTF-8 UTF-8; ru_RU.UTF-8 UTF-8
4$ locale-gen

5$ echo "LANG=en_US.UTF-8" > /etc/locale.conf
6$ vim /etc/hostname # Пишем имя компьютера
7$ vim /etc/hosts # Пишем следующее:
                    | 127.0.0.1    localhost
                    | ::1          localhost
                    | 127.0.0.1    *hostname*.localdomain    *hostname*
```

## 1.9 Пароль root и создание пользователя
```bash
1$ passwd

2$ useradd -m *username*
3$ passwd *username*
4$ usermod -aG wheel,audio,video,optical,storage *username*
5$ userdbctl groups-of-user *username*
```

## 1.10 Эдитор текста по умолчанию, visudo
```bash
1$ EDITOR=vim
2$ pacman -S sudo
3$ visudo # Раскомментировать строку "%wheel ALL=(ALL) All"
```

## 1.11 Монтирование EFI System, скачивание и настройка GRUB
```bash
1$ pacman -S grub efibootmgr dosfstools os-prober mtools
.$ fdisk -l
2$ mount --mkdir /dev/*efi system* /boot/EFI

3$ vim /etc/default/grub # Выставляем "GRUB_TIMEOUT=1"
4$ grub-install --targen=x86_64-efi --bootloader-id="*name*" --recheck # name - тут можно назвать метку, которую BIOS будет отображать для этой системы (использовать только латиницу)
4$ grub-mkconfig -o /boot/grub/grub.cfg
```

## 1.12 Перезагрузка
```bash
1$ exit
2$ reboot # Когда компьютер будет выключен - вынимаем загрузочную флешку
3$ sudo systemctl enable dhcpcd
```

## 1.13 Настройки pacman и установка yay
```bash
.$ # Подключаемся к интернету удобным способом. Также может понадобиться перезагрузка (`sudo reboot`)
1$ sudo vim /etc/pacman.conf # Раскомментируем multilib но! не перепутать multilib-testing и multilib
2$ sudo pacman -Sy pacman-contrib curl sed
3$ sudo curl -s "https://archlinux.org/mirrorlist/?country=RU&country=NL&protocol=https&use_mirror_status=on" | sed -e 's/^#Server/Server/' -e '/^#/d' | rankmirrors -n 10 - > /etc/pacman.d/mirrorlist # ожидание может занять до нескольких минут
.$ vim /etc/pacman.d/mirrorlist # проверяем

4$ sudo pacman -S --needed git base-devel
5$ git clone https://aur.archlinux.org/yay.git
6$ cd yay
7$ makepkg -si
```

## 1.14 Установка zram
```bash
NB!$ # Так то необходимо как-то выключать zswap в настройках ядра, но я хз как, поэтому просто ставим как ставим

1$ sudo pacman -S zram-generator
2$ sudo vim /etc/systemd/zram-generator.conf # Пишем следующее:
                                                | [zram0]
                                                | zram-size = zram
                                                | compression-algorithm = zstd
3$ reboot
.$ zramctl # Проверка работы
```

## 1.15 Прочее
```bash
1$ sudo vim /etc/modprobe.d/nobeep.cong # Пишем следующее:
                                          | blacklist pcspkr
```


# 2. Настройка системы
## 2.1 Установка базовых пакетов и первый запуск
```bash
1$ sudo pacman -Rsc vim

2$ sudo pacman -S xorg xorg-xinit xorg-server bspwm sxhkd neovim alacritty python python-pip rofi rofi-calc rofi-emoji thunar thunar-volman gvfs firefox pulseaudio pulseaudio-jack pulseaudio-alsa pulseaudio-bluetooth pavucontrol alsa-utils polybar feh dunst brightnessctl networkmanager numlockx neofetch tldr exa ncdu htop
3$ pip install pyperclip
4$ yay -S picom-jonaburg-git ly betterlockscreen
5$ sudo systemctl enable ly
6$ echo "exec sxhkd & \nexec bspwm"

7$ nvim ~/.config/bspwm/bspwmrc # Пишем следующее:
                                    | #!/bin/bash
                                    | alacritty
                                    | thunar
                                    | firefox
                                    | xset r rate 170 40
                                    | setxkbmap -layout us,ru -option grp:win_space_toggle

8$ sudo systemctl --user enable pulseaudio
9$ reboot
```

## 2.2 Восстановление файлов конфигурации
1. Подключаем флешку к ПК и перемещаем все файлы и директории конфигурация по их местам
2. Делаем все рабочие файлы в `~/.config/bspwm/` исполняемыми (`chmod +x *file*`):
    - `bspwmrc`
    - `sxhkdrc`
    - `scripts/*`

## 2.3 Устанавливаем остальные пакеты
```bash
1$ sudo pacman -S figma-linux-agent figma-linux-bin ttf-jetbrains-mono telegram-desktop discord code zoom zsh yarn npm gulp lxapperance
2$ betterlockscreen -u ~/.config/bspwm/themes/default/walls/wall-01.png
3$ yay -Sy reversal-icon-theme-git papirus-icon-theme catppuccin-cursors-macchiato catppuccin-gtk-theme-macchiato
```
Далее в lxapperance выставляем свои темы

## 2.4 Настройка клавиатуры
```bash
1$ sudo nvim /etc/X11/xorg.conf.d/00-keyboard.conf # Пишем следующее:
                                                      | Section "InputClass"
                                                      |         Identifier "system-keyboard"
                                                      |         MatchIsKeyboard "on"
                                                      |         Option "XkbLayout" "us,ru"
                                                      |         Option "XkbModel" "pc105"
                                                      |         Option "XkbOptions" "grp:win_space_toggle"
                                                      |         Option "AutoRepeat" "170 40
                                                      | EndSection
```
