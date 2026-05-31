+++
date = '2026-05-28'
draft = false
title = 'Device Tree Overlay 的簡易練習 (以 w1-gpio 改 pin 為例)'
tags = ['raspberry pi', 'device tree']
description = '在 Raspberry Pi 4 上新增 Device Tree Overlay 檔案並更改 1-Wire 模組的 pin'
showToc = true
TocOpen = true
showReadingTime = true
showWordCount = true
+++

這篇記錄自行改寫 1-Wire 的 Device Tree Overlay 檔案並讓樹莓派載入的過程，
原本樹莓派的 1-Wire 模組預設接在 GPIO 4，這篇練習將其接在 GPIO 17 。


## 硬體配置

以下紀錄使用到的硬體

- Hardware: Raspberry Pi 4 Model B
- OS: Debian GNU/Linux 13 (trixie) / Raspberry Pi OS
- Kernel: 6.12.75+rpt-rpi-v8
- 1-Wire: DS18B20 溫度感測器模組(已含上拉電阻)


## 連接模組

參考 [Raspberry Pi GPIO pinout guide](https://pinout.xyz/)，
1-Wire 模組的接線方式：
- `GND` (-) 接到任一標明 `GND` 的接腳，例如 Pin 9
- `VDD` (+) 接到 `3.3V` 接腳，例如 Pin 1
- `DQ` (out) 接到 GPIO 17 也就是 Pin 11


## 簡易路線

修改 config 檔案，加入`dtoverlay`參數並指定 GPIO Pin 為 17。

```text {title="/boot/firmware/config.txt"}
dtoverlay=w1-gpio,gpiopin=17
```

重開機後就會生效。

如果只是要換 pin，這樣就夠了。但如果想了解怎麼自己寫一份 overlay，請繼續往下看。

### 使用自訂的 Device Tree Overlay

#### 取得原始 DTO 檔

首先從樹莓派的 Repo 抓 1-Wire 的 Device Tree Overlay 原始碼，
並修改成自訂的檔名以和原本的檔案做出區別。

```bash
wget https://raw.githubusercontent.com/raspberrypi/linux/rpi-6.12.y/arch/arm/boot/dts/overlays/w1-gpio-overlay.dts
cp w1-gpio-overlay.dts my-w1-gpio.dts
```

先來看看原本的 DTO 檔內容：

```c {title="w1-gpio-overlay.dts",linenos=true}
// Definitions for w1-gpio module (without external pullup)
/dts-v1/;
/plugin/;

/ {
        compatible = "brcm,bcm2835";

        fragment@0 {
                target-path = "/";
                __overlay__ {

                        w1: onewire@0 {
                                compatible = "w1-gpio";
                                pinctrl-names = "default";
                                pinctrl-0 = <&w1_pins>;
                                gpios = <&gpio 4 0>;
                                status = "okay";
                        };
                };
        };

        fragment@1 {
                target = <&gpio>;
                __overlay__ {
                        w1_pins: w1_pins@0 {
                                brcm,pins = <4>;
                                brcm,function = <0>; // in (initially)
                                brcm,pull = <0>; // off
                        };
                };
        };

        __overrides__ {
                gpiopin =       <&w1>,"gpios:4",
                                <&w1>,"reg:0",
                                <&w1_pins>,"brcm,pins:0",
                                <&w1_pins>,"reg:0";
                pullup;         // Silently ignore unneeded parameter
        };
};
```

#### 修改 DTO 檔

對原內容做如下修改：

1. 把 `gpios` 和 `brcm,pins` 的 pin number 從 4 改成 17。
2. (Optional) 把 node name 從 onewire 改成 my-onewire，方便下次開機後驗證載入的 DTO。
3. (Optional) 把 node name 的 unit address `@xxx` 去掉，避免編譯時出現下列 Warning：
    ```
    my-w1-gpio.dts:12.21-18.6: Warning (unit_address_vs_reg): /fragment@0/__overlay__/my-onewire@0: node has a unit name, but no reg or ranges property
    my-w1-gpio.dts:25.23-29.6: Warning (unit_address_vs_reg): /fragment@1/__overlay__/w1_pins@0: node has a unit name, but no reg or ranges property
    ```
4. (Optional) 把 gpios 的第三個屬性從 0 改成 6 ， 6 是 `GPIO_OPEN_DRAIN` 的意思，可以查閱 [GPIO bindings header](https://github.com/torvalds/linux/blob/master/include/dt-bindings/gpio/gpio.h)。目的是明確標示Pin的電氣特性，避免開機出現這樣的 Warning：
    ```
    gpio-529 (onewire@0): enforced open drain 
    please flag it properly in DT/ACPI DSDT/board file
    ```
5. (Optional) 刪去 `__overrides__` 區塊。這是讓 `config.txt` 支援參數修改（如 gpiopin=17）的機制，我們的 overlay 已經寫死 GPIO17，不需要這個彈性，所以可以刪去。

修改的diff如下：

```diff
--- w1-gpio-overlay.dts 2026-05-28 23:21:05.915015822 +0800
+++ my-w1-gpio.dts      2026-05-29 20:05:05.657742903 +0800
@@ -9,11 +9,11 @@
                target-path = "/";
                __overlay__ {

-                       w1: onewire@0 {
+                       w1: my-onewire {
                                compatible = "w1-gpio";
                                pinctrl-names = "default";
                                pinctrl-0 = <&w1_pins>;
-                               gpios = <&gpio 4 0>;
+                               gpios = <&gpio 17 6>; // 6 = GPIO_OPEN_DRAIN
                                status = "okay";
                        };
                };
@@ -22,19 +22,11 @@
        fragment@1 {
                target = <&gpio>;
                __overlay__ {
-                       w1_pins: w1_pins@0 {
-                               brcm,pins = <4>;
+                       w1_pins: w1_pins {
+                               brcm,pins = <17>;
                                brcm,function = <0>; // in (initially)
                                brcm,pull = <0>; // off
                        };
                };
        };
-
-       __overrides__ {
-               gpiopin =       <&w1>,"gpios:4",
-                               <&w1>,"reg:0",
-                               <&w1_pins>,"brcm,pins:0",
-                               <&w1_pins>,"reg:0";
-               pullup;         // Silently ignore unneeded parameter
-       };
 };
```

#### 編譯、安裝與指定

修改好之後，就可以編譯檔案並安裝到 overlays 資料夾：

```bash
# 編譯
dtc -@ -I dts -O dtb -o my-w1-gpio.dtbo my-w1-gpio.dts

# 安裝
sudo cp my-w1-gpio.dtbo /boot/firmware/overlays/
```

最後只要在 `config.txt` 裡面指定 `my-w1-gpio` 就完成了。

```text {title="/boot/firmware/config.txt"}
dtoverlay=my-w1-gpio
```

只要重開機就可以載入自訂的 DTO 檔。

#### 結果驗證

重開機後，可以用下列兩種指令確認 `my-onewire` 佔用了 GPIO 17：

```bash
$ cat /sys/kernel/debug/gpio | grep -i "GPIO17"
 gpio-529 (GPIO17              |my-onewire        ) out hi
$ sudo cat /sys/kernel/debug/pinctrl/*/pinmux-pins | grep "pin 17"
pin 17 (gpio17): my-onewire pinctrl-bcm2711:529 function gpio_in group gpio17
```

到這邊我們就確認自訂的 Device Tree Overlay 被系統成功載入了。

如果要從 DS18B20 讀取溫度，可以確認系統路徑 `/sys/bus/w1/devices/` 下的 `28-xxx` 資料夾，
裡面的 `temperature` 和 `w1_slave` 檔案都有溫度資訊。

```bash
$ cat /sys/bus/w1/devices/28-0000008007df/temperature
30437
$ cat /sys/bus/w1/devices/28-0000008007df/w1_slave
e7 01 7f 80 7f ff 09 10 26 : crc=26 YES
e7 01 7f 80 7f ff 09 10 26 t=30437
```
