+++
date = '2026-05-24'
draft = false
title = '在 Raspberry Pi 4 開啟完整 boot log'
tags = ['raspberry pi']
description = '開啟 Raspberry Pi 4 完整 boot log 的方法，包含 firmware 階段'
showToc = true
TocOpen = true
showReadingTime = true
showWordCount = true
+++

最近因為工作想要進一步熟悉 embedded Linux，
跟強者朋友借來一台 Raspberry Pi 4，自己也採買了相關配件。

不過 console 接好線並上電後，只看到系統的 login prompt，
沒有熟悉的 kernel log：

```
Debian GNU/Linux 13 pi ttyS0

My IP address is 192.168.1.120 2001:b011:3809:ff1a:b5b9:c2e5:c48d:c8f9

pi login:
```

這篇簡單記錄啟用 kernel log 與 firmware log 的方式，
以及 Pi 的 boot flow 。


## 硬體配置

以下簡單紀錄使用到的硬體

- Hardware: Raspberry Pi 4 Model B
- OS: Debian GNU/Linux 13 (trixie) / Raspberry Pi OS
- Kernel: 6.12.75+rpt-rpi-v8
- USB-TTL: CP2102


## 連接 UART Console

參考 [Raspberry Pi GPIO pinout guide](https://pinout.xyz/)，
USB-TTL 接線的方式：
- TTL 的 `GND` 接到任一標明 `GND` 的接腳
- TTL 的 `RX` 接到 8 號 `TXD` 接腳
- TTL 的 `TX` 接到 10 號 `RXD` 接腳


## 開啟 UART

網路文件指出 Raspberry Pi 預設不會把 UART 打開，
使用者需要進入系統開啟 `config.txt` 的 `enable_uart` 設定。

```text {title="/boot/firmware/config.txt"}
enable_uart=1
```

但我實際利用 Raspberry Pi Imager 燒錄 micro SD 卡，
並把 SD 卡插入沒有接上 HDMI 的 Pi 4 時，
開機的時候 console 有輸出 login prompt，
而且登入系統後確認 `enable_uart` 已經打開，
因為好奇，到 GitHub 的 `pi-gen` Repository 確認 `config.txt` 的 [template](https://github.com/RPi-Distro/pi-gen/blob/master/stage1/00-boot-files/files/config.txt)，
可以確認原本沒有 `enable_uart` 的選項，
應該是 first-boot 的相關機制加上去的。

也因為這樣，不需要另購 Micro-HDMI to HDMI 線就能直接以命令列方式操作 Pi 4。


## 開啟 Kernel Log

要開啟 kernel log 的步驟很簡單，只要把 `cmdline.txt` 裡面的 `quiet` 字串刪掉就可以了。

```text {title="/boot/firmware/cmdline.txt"}
console=serial0,115200 console=tty1 root=PARTUUID=113d805a-02 rootfstype=ext4 fsck.repair=yes rootwait **quiet** splash plymouth.ignore-serial-consoles ds=nocloud;i=rpi-imager-1779788245938 cfg80211.ieee80211_regdom=TW
```

重開機後可以看到類似下面的紀錄：

```
[    0.000000] Linux version 6.12.75+rpt-rpi-v8 (serge@raspberrypi.com) (aarch64-linux-gnu-gcc-14 (Debian 14.2.0-19) 14.2.0, GNU ld (GNU Binutils for Debian) 2.44) #1 SMP PREEMPT Debian 1:6.12.75-1+rpt1 (2026-03-11)
[    0.000000] KASLR enabled
[    0.000000] random: crng init done
[    0.000000] Machine model: Raspberry Pi 4 Model B Rev 1.2

......
```

## 開啟 Firmware Log

如果想要進一步確認 Linux 被載入之前的紀錄，
也就是 `start4.elf` 的 firmware log，
要在 `config.txt` 裡面開啟 `uart_2ndstage`

```text {title="/boot/firmware/config.txt"}
uart_2ndstage=1
```

重開機後可以看到類似下面的紀錄：

```
  4.99 [sdcard] vl805.bin not found
  4.99 [sdcard] pieeprom.upd not found
  4.99 [sdcard] recover4.elf not found
  5.00 [sdcard] recovery.elf not found
  5.35 Read start4.elf bytes  2303680 hnd 0x1946c

......

MESS:00:00:05.303314:0: arasan: arasan_emmc_open
MESS:00:00:05.304971:0: arasan: arasan_emmc_set_clock C0: 0x00800000 C1: 0x000e 0047 emmc: 200000000 actual: 390625 div: 0x00000100 target: 400000 min: 400000  max: 400000 delay: 5

......
```

## 開機流程

把兩個設定都打開以後，
就可以拿實際開機的紀錄對照 Pi 4 的開機流程：

- Stage 1: SoC BootROM (mask ROM, 看不到 log)
- Stage 2: EEPROM bootloader (SPI flash, "[sdcard] xxx" log)
   - 初始化 SD card / USB
   - 尋找 boot partition
   - 載入 start4.elf 到 GPU
- Stage 3: start4.elf (GPU firmware, "MESS:xxx" log)
   - 讀 config.txt
   - 載入 DTB + overlays
   - 套用 dtparam / dtoverlay
   - 載入 kernel8.img + initramfs
   - 設定 ARM CPU、釋放 reset、跳轉
- Stage 4: Linux kernel ("[ 0.000000] Booting Linux")


## 相關連結

- [Raspberry Pi GPIO pinout guide](https://pinout.xyz/)
- [`pi-gen` Repository](https://github.com/RPi-Distro/pi-gen/)
- [Raspberry Pi Documentation - config.txt](https://www.raspberrypi.com/documentation/computers/config_txt.html#enable_uart)