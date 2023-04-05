# Примечание
- `*` внутри команд - это особое значение. (Н-р: вместо `*username*` пишем свое имя пользователя)

# 1. Установка archlinux
## 1.1 Подключение к интернету
Использовать либо раздачу с мобильного через модем, либо через iwd:
1.  ```bash
    iwctl
    ```
2.  ```bash
    device list
    ```
3.  ```bash
    station *устройство* scan
    ```
4.  ```bash
    station *устройство* get-netwokrs
    ```
5. Выходим --> `*Ctrl + D*`
6.  ```bash
    iwctl --passphrase *пароль* station *устройство* connect *SSID*
    ```

> `ping google.com` - для проверки сети
---
## 1.2 Время

1.  ```bash
    timedatectl set-ntp true
    ```
2.  ```bash
    timedatectl set-timezone Europe/Moscow
    ```
> `timedatectl status` - проверить статус
---
## 1.3 Разметка диска

1.  ```bash
    fdisk -l
    ```
2.  ```bash
    cfdisk
    ```
3.  Создаем разметку (ниже). **NB!** Используем GPT disklabel
4.  ```bash
    mkfs.fat -F32 /dev/*efi system*
    ```
5.  ```bash
    mkfs.ext4 /dev/*linux filesystem*
    ```

> Моя разметка:
> - 300M - EFI System - FAT32
> - Оставшееся - Linux Filesystem - ext4/btrfs
---
## 1.4 Монтирование
1.  ```bash
    mount --mkdir /dev/*efi system* /mnt/efi
    ```
2.  ```bash
    mount /dev/*linux filesystem* /mnt
    ```
---
## 1.5 Pacstrap и genfstab
1.  ```bash
    pacstrap /mnt base linux-zen linux-firmware
    ```
2.  ```bash
    genfstab -U /mnt >> /mnt/etc/fstab
    ```
---
## 1.6 Смена корневого каталога
1.  ```bash
    arch-chroot /mnt
    ```
2.  ```bash
    pacman -S neovim dhcpcd
    ```
---
## 1.7 Время, локали и хосты
1.  ```bash
    ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime
    ```
2.  ```bash
    hwclock --systohc
    ```
3.  ```bash
    nvim /etc/locale.gen
    ```
    Раскомментировать: `en_US.UTF-8 UTF-8`; `ru_RU.UTF-8 UTF-8`
4.  ```bash
    locale-gen
    ```
5.  ```bash
    echo "LANG=en_US.UTF-8" > /etc/locale.conf
    ```
    Тут мы ставим язык системы. `LANG=ru_RU.UTF-8` для русского языка
6.  ```bash
    nvim /etc/hostname
    ```
    Пишем свое имя компьютера
7.  ```bash
    nvim /etc/hosts
    ```
    Пишем следующее:
    ```
    127.0.0.1    localhost
    ::1          localhost
    127.0.0.1    *hostname*.localdomain    *hostname*
    ```
---
## 1.8 Пароль root и создание пользователя
1.  ```bash
    passwd
    ```
2.  ```bash
    useradd -m *username*
    ```
3.  ```bash
    passwd *username*
    ```
4.  ```bash
    usermod -aG wheel,audio,video,optical,storage *username*
    ```
    > Проверить группы: `userdbctl groups-of-user *username*`
---
## 1.9 Редактор текста по умолчанию, visudo
1.  ```bash
    EDITOR=nvim
    ```
2.  ```bash
    pacman -S sudo
    ```
3.  ```bash
    visudo
    ```
    Раскомментировать строку `%wheel ALL=(ALL) All`
---
## 1.10 Монтирование EFI System, скачивание и настройка GRUB
1.  ```bash
    pacman -S grub efibootmgr dosfstools os-prober mtools
    ```
2.  ```bash
    mount --mkdir /dev/*efi system* /boot/EFI
    ```
    > Вспомнить диск: `fdisk -l`
3.  ```bash
    nvim /etc/default/grub
    ```
    Выставляем `GRUB_TIMEOUT=1`
4.  ```bash
    grub-install --target=x86_64-efi --bootloader-id="*name*" --recheck
    ```
    name - тут можно назвать метку, которую BIOS будет отображать для этой системы (**NB!** использовать только латиницу)
5.  ```bash
    grub-mkconfig -o /boot/grub/grub.cfg
    ```
---
## 1.11 Перезагрузка
1.  ```bash
    exit
    ```
2.  ```bash
    reboot
    ```
    Когда компьютер будет выключен - вынимаем загрузочную флешку
3.  ```bash
    sudo systemctl enable dhcpcd
    ```
---
## 1.12 Настройки pacman и установка yay
1. Подключаемся к интернету удобным способом (может понадобиться перезагрузка (`sudo reboot`))
2. ```bash
    sudo nvim /etc/pacman.conf
    ```
    Раскомментируем `multilib` (**NB!** Не перепутать `multilib-testing` и `multilib`)
3.  ```bash
    sudo pacman -Sy --needed pacman-contrib curl sed git base-devel
    ```
4.  ```bash
    sudo curl -s "https://archlinux.org/mirrorlist/?country=RU&country=NL&protocol=https&use_mirror_status=on" | sed -e 's/^#Server/Server/' -e '/^#/d' | rankmirrors -n 10 - > /etc/pacman.d/mirrorlist
    ```
    Ожидание может занять до нескольких минут.
    > Для проверки: `nvim /etc/pacman.d/mirrorlist`
5.  ```bash
    git clone https://aur.archlinux.org/yay.git
    ```
6.  ```bash
    cd yay
    ```
7.  ```bash
    makepkg -si
    ```
---
## 1.13 Установка zram
**NB!** Так то необходимо как-то выключать zswap в настройках ядра, но я хз как, поэтому просто ставим как ставим

1.  ```bash
    sudo pacman -S zram-generator
    ```
2.  ```bash
    sudo nvim /etc/systemd/zram-generator.conf
    ```
    Пишем следующее:
    ```conf
    [zram0]
    zram-size = zram
    compression-algorithm = zstd
    ```
3.  ```bash
    reboot
    ```
> Для проверки работы zram - `zramctl`
---
## 1.14 Прочее
1.  ```bash
    sudo nvim /etc/modprobe.d/nobeep.conf
    ```
    Пишем следующее:
    ```conf
    blacklist pcspkr
    ```
---
# 2. Настройка системы
## 2.1 Установка базовых пакетов и первый запуск
1.  ```bash
    sudo pacman -S xorg xorg-xinit xorg-server bspwm sxhkd  \
    alacritty python python-pip rofi rofi-calc rofi-emoji thunar \
    thunar-volman gvfs gvfs-mtp firefox pulseaudio pulseaudio-jack \
    pulseaudio-alsa pulseaudio-bluetooth pavucontrol alsa-utils \
    polybar feh dunst brightnessctl networkmanager numlockx \
    neofetch tldr exa ncdu htop noto-fonts
    ```
2.  ```bash
    pip install pyperclip
    ```
3.  ```bash
    yay -S picom-jonaburg-git ly betterlockscreen
    ```
4.  ```bash
    sudo systemctl enable ly
    ```
5.  ```bash
    echo "exec sxhkd &\nexec bspwm" > ~/.xinitrc
    ```
6.  ```bash
    nvim ~/.config/bspwm/bspwmrc
    ```
    Пишем следующее:
    ```bash
    #!/bin/bash
    alacritty
    thunar
    firefox
    xset r rate 170 40
    setxkbmap -layout us,ru -option grp:win_space_toggle
    ```
7.  ```bash
    sudo systemctl --user enable pulseaudio
    ```
8.  ```bash
    reboot
    ```
---
## 2.2 Восстановление файлов конфигурации
1. Подключаем флешку к ПК и перемещаем все файлы и директории конфигурация по их местам
2. Делаем все рабочие файлы в `~/.config/bspwm/` исполняемыми (`chmod +x *file*`):
    - `bspwmrc`
    - `sxhkdrc`
    - `scripts/*`
---
## 2.3 Устанавливаем остальные пакеты
1.  Программы:
    ```bash
    sudo pacman -S figma-linux-agent figma-linux-bin \
    ttf-jetbrains-mono telegram-desktop discord code \
    zoom flameshot zsh yarn npm gulp lxapperance aria2 \
    lutris
    ```
    Зависимости к ним:
    ```bash
    sudo pacman -S --needed tk lib32-mesa vulkan-radeon lib32-vulkan-radeon \
    vulkan-icd-loader lib32-vulkan-icd-loader wine-staging giflib lib32-giflib \
    libpng lib32-libpng libldap lib32-libldap gnutls lib32-gnutls mpg123 lib32-mpg123 \
    openal lib32-openal v4l-utils lib32-v4l-utils libpulse lib32-libpulse libgpg-error \
    lib32-libgpg-error alsa-plugins lib32-alsa-plugins alsa-lib lib32-alsa-lib libjpeg-turbo \
    lib32-libjpeg-turbo sqlite lib32-sqlite libxcomposite lib32-libxcomposite libxinerama \
    lib32-libgcrypt libgcrypt lib32-libxinerama ncurses lib32-ncurses ocl-icd lib32-ocl-icd \
    libxslt lib32-libxslt libva lib32-libva gtk3 lib32-gtk3 gst-plugins-base-libs \
    lib32-gst-plugins-base-libs
    ```
2.  ```bash
    yay -S reversal-icon-theme-git papirus-icon-theme \
    catppuccin-cursors-macchiato catppuccin-gtk-theme-macchiato
    ```
---
## 2.4 Настройка рабочего окружения
1. Выставляем свои темы в lxapperance
2.  Ставим обоину в betterlockscreen
    ```bash
    betterlockscreen -u ~/.config/bspwm/themes/default/walls/wall-01.png
    ```
3.  Задаем настройки клавиатуры
    ```bash
    sudo nvim /etc/X11/xorg.conf.d/00-keyboard.conf
    ```
    Пишем следующее:
    ```
    Section "InputClass"
            Identifier "system-keyboard"
            MatchIsKeyboard "on"
            Option "XkbLayout" "us,ru"
            Option "XkbModel" "pc105"
            Option "XkbOptions" "grp:win_space_toggle"
            Option "AutoRepeat" "170 40"
    EndSection
    ```
---
## 2.5 Настраиваем приложения
### 2.5.1 Thunar
1. Edit --> Configure Custom Actions --> Open Terminal Here --> command --> `alacritty --working-directory %f`
2. File --> RMC --> Open With --> Set Default Application --> command --> `alacritty -e nvim %f`
### 2.5.2 LxApperance
- Тема: Catppuccin-Macchiato-Standart-Green-Dark
- Шрифт: Noto Sans 11
- Иконки: Reversal-grey
- Курсор: Catppuccin-Macchiato-Green
