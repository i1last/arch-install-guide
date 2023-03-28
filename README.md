# Установка archlinux

## Подключение к интернету

Использовать либо раздачу с мобильного через модем, либо через iwd:
```
1$ iwctl
2[iwd]$ device list
3[iwd]$ station *устройство* scan
4[iwd]$ station *устройство* get-netwokrs
5$ *Ctrl + D*
6$ iwctl --passphrase *пароль* station *устройство* connect *SSID*
```
