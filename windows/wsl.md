# WSL2 で USBデバイスを使用する

## Install
### Update WSL2 kernel
Need over ver 5.10.60.1.
```
wsl --update  
wsl --status
```
### Install USB IP software on Windows
```
winget install --interactive --exact dorssel.usbipd-win
```
### Install USB IP driver on Linux (Ubuntu)
```
sudo apt update  
sudo apt install -y linux-tools-5.4.0-77-generic hwdata  
sudo update-alternatives --install /usr/local/bin/usbip >usbip /usr/lib/linux-tools/5.4.0-77-generic/usbip 20  
```
Annotaion:  Using Ubuntu 22.04, replace version to 5.15.0-25.

Teach path of usbip to root user.
```
sudo visudo
```

Add "/usr/lib/linux-tools/5.4.0-77-generic" to "Defaults secure_path".

---
## Attach USB device on Windows
Show USB device list.
```
usbipd wsl list
```

Attach to distribution during running wsl on win10.
```
usbipd wsl attach --busid <dev no> -d <dist name>
```

Show USB device on Linux.
```
lsusb
```

---
## Detach USB device on Windows
Show USB device list.
```
usbipd wsl list
```

Detach from distribution.
```
usbipd wsl detach --busid <dev no>
```

---
## references
### links
* https://learn.microsoft.com/ja-jp/windows/wsl/connect-usb  
* https://kledgeb.blogspot.com/2021/11/wsl-227-wsl-2usblinuxusb.html  
* https://scratchpad.jp/wsl2-usb-camera-1/  
* https://serip39.hatenablog.com/entry/2022/06/27/070000  

### Command of "update-alternatives"
* https://graziegrazie.hatenablog.com/entry/2015/11/14/101050

`--install` option
```
sudo update-alternatives --install <path of synblic link> <group name> <path of entity> <priority>
```

```
ex) for GCC
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.8 10
```
