# Raspberry Pi用冷却ファンとシリアルコンソール
Raspberry Pi (1|2)用の冷却ファンの回路とソフトおよび，シリアルコンソールを外出しするための回路．

# 1. 作成した経緯など
Raspberry Piを自宅で24時間運転することを考慮すると，エアコン無しでも
熱で壊れないことが重要．

ヒートシンクも存在するが，やはり心配なのでファンを入れることにした．
ただし，24時間ファンが回るとうるさいので，システムの温度が上昇したときだけ
まわるようにしたかった．

あと，ディスプレイやキーボード無しで運用するため，シリアルコンソールを簡単に接続できるように
するための回路も追加．

「回路」ディレクトリには，Raspberry Pi用のユニバーサル基板を利用してシールドを作成する
場合のfritzingの回路図を追加．「コンソールなど用Piシールド」では，コンソール用USBシリアル，
ファン制御用の端子の他に，I2C接続のRTC端子を追加．このシールドで利用したRTCモジュールは
Spark FunのBOB-12708です．あと，秋月電子通商でも入手が容易な8564NBの場合の回路図も
付けてあります．

# 2. コンソールやRTCの設定
## 2.1 Raspberry Pi3 のシリアルコンソールの問題
UARTの速度がCPUの動作周波数と連動してしまうため，シリアルコンソールでは
文字化け，入力不可能な状態が続く．

[参考URL](https://github.com/RPi-Distro/repo/issues/22)

もし，ディスプレイやUSBキーボードを
繋がずに利用するため，
シリアルコンソール使う場合は，ブート用コンフィグファイルに以下の行を追加．

```/boot/config.txt
enable_uart=1

core_freq=250
force_turbo=1
```

## 2.2 RTC
シリアルコンソール用のシールドには，RTCも搭載しているため，これを利用する場合，以下のURLを参照し，パッケージのインスールや設定を
行う必要がある．
### Spark Fun BOB-12708
- [DS1307関係参考URL1](<https://mynotebook.h2np.net/post/1085> "Raspberry Pi 2 にRTCモジュールをつける") Raspberry Pi 2 にRTCモジュールをつける
- [DS1307関係参考URL2](<https://learn.adafruit.com/adding-a-real-time-clock-to-raspberry-pi?view=all> "Adding a Real Time Clock to Raspberry Pi") Adding a Real Time Clock to Raspberry Pi

### (秋月)8564NB
- [8564NB関係参考URL1](<http://news.mynavi.jp/articles/2014/08/21/raspberry-pi4/002.html> "超小型PC「Raspberry Pi」で夏休み自由課題・第4回 - Raspberry Piの屋外モバイル、高層エレベーターの気圧変化を調べる") 超小型PC「Raspberry Pi」で夏休み自由課題・第4回 - Raspberry Piの屋外モバイル、高層エレベーターの気圧変化を調べる
- [8564NB関係参考URL2](<http://tomosoft.jp/design/?p=5812> "Raspberry PIへリアルタイムクロックモジュールのI2C接続") Raspberry PIへリアルタイムクロックモジュールのI2C接続

# 3. 用意した部品
## 3.1 共通
### 3.1.1 本体など
* 本体 : [Raspberry Pi 2 Model B][pi2]
* ヒートシンク : [Seeed Studio 800035001 Raspberry Pi用アルミニウム・ヒートシンク][sink]
* USBシリアル変換 : [スイッチサイエンスSSCI-010320 FTDI USBシリアル変換アダプター(5V/3.3V切り替え機能付き)][ftdi]

### 3.1.2 ファンの制御回路用
* 抵抗 : [カーボン抵抗(炭素皮膜抵抗) 1/4W 3kΩ (100本)][3kohm]
* ダイオード : [ショットキーダイオード][diode]
* トランジスタ : [トランジスタ (2SC1815GR 60V 150mA) 20個][2SC1815]

ただし，今回旧型Pi用のヒートシンクを利用したのは，旧型用は小型のヒートシンクが余分についている代わりに，熱伝導両面テープが付いているので，特別な材料とかなしに，チップに貼り付けることができるためです．

## 3.2 千石電商の箱をそのまま利用
* [Seeed Studio 114990129/141107004【ファン付き】Raspberry Pi B+専用ケース(クリア/ロゴなし/組立式)][case1]
* [Seeed Studio 114990130 Raspberry Pi B+専用ケース(クリア/ロゴなし/組立式)][case2]

GPIOの横の部品にスリットがあり，そこが折れたため，スリットがないファンなしの箱を追加で買って，部品交換しましまた．

![箱の壊れた部分][brokenCase]

![千石のファン付き箱に収めた写真][system2]

## 3.3 改造版
シリアルを簡単に繋げられるようにしたかったので，GPIO端子周りに余裕がある箱を改造してファンを付けた．
* [MultiComp MC-RP002-CLR Raspberry Pi B+専用ケース(クリア/ロゴあり)][case3]
* [SHICOH F4006AP-05QCW (0406-5) DCファン(5V)][dcFan]
* [カモンSRS-40 ファン防振シリコンシート][si-sheet]
* [40mm角ファンガード][FanGuard]

![自作箱に収めた写真][system1]

![自作箱を開けた写真][openCase]

# 4. 回路図など
![ブレッドボード利用時の配線イメージ][breadboard]

![回路図][circuit]

# 5. ファンの制御プログラム

## getTemperature.sh
    #!/bin/sh
    cat /sys/class/thermal/thermal_zone0/temp

##fanOff.sh
    #!/bin/sh
    
    echo 18 >/sys/class/gpio/export
    echo out >/sys/class/gpio/gpio18/direction
    echo 0 > /sys/class/gpio/gpio18/value


## fanOn.sh

    #!/bin/sh
    
    echo 18 >/sys/class/gpio/export
    echo out >/sys/class/gpio/gpio18/direction
    echo 1 > /sys/class/gpio/gpio18/value

##fanctrl

    #!/usr/bin/perl
    
    use Switch;
    
    open(IN, "/sys/class/thermal/thermal_zone0/temp");
    $cputemp = <IN>/1000.0; #numbers
    close(IN);
    $cputempavg = $cputemp;
    
    system("echo 18 > /sys/class/gpio/export");
    system("echo out > /sys/class/gpio/gpio18/direction");
    system("echo 1 > /sys/class/gpio/gpio18/value");
    $fan_run = 1;
    
    while(1)
    {
    	open(IN, "/sys/class/thermal/thermal_zone0/temp");
    	$cputemp = <IN>/1000.0; #numbers
    	close(IN);
    	
    	$cputempavg = $cputempavg*0.9 + $cputemp*0.1;
    
    	if($cputempavg > 45)
    	{
    		$fan_run = 1;
    	}
    	if($cputempavg < 35)
    	{
    		$fan_run = 0;
    	}
    	if($cputempavg > 65)
    	{
    		$fan_run = 2;
    	}
    	switch ($fan_run) {
    		case 1 { system("echo 1 > /sys/class/gpio/gpio18/value");}
    		case 0 { system("echo 0 > /sys/class/gpio/gpio18/value");}
    		else   { system("shutdown -h now");}
    	}
    	sleep(10);
    }


## チューニングのポイント
to be done.

# 6. 製作記事
ハードの工作の詳細は製作記事を参照してね．

## 千石の箱に収めた場合
[千石の箱版][sengoku]

## 自分で箱に穴を開けて，ファンを付けた場合
[改造箱版][original]

<!--以下はリンクの定義-->
<!--参考文献-->
[sengoku]: <http://hautbrion.blogspot.jp/2015/05/raspberry-pi-2usb-1.html> "千石の箱版"
[original]: <http://hautbrion.blogspot.jp/2014/10/raspberry-pi-2.html> "改造箱版"

<!--ハード関連-->

[pi2]: <http://akizukidenshi.com/catalog/g/gM-09024/> "Raspberry Pi 2 Model B"
[case1]: <http://www.sengoku.co.jp/mod/sgk_cart/detail.php?code=EEHD-4N4C> "Seeed Studio 114990129/141107004【ファン付き】Raspberry Pi B+専用ケース(クリア/ロゴなし/組立式)"
[case2]: <http://www.sengoku.co.jp/mod/sgk_cart/detail.php?code=EEHD-4R3F> "Seeed Studio 114990130 Raspberry Pi B+専用ケース(クリア/ロゴなし/組立式)"
[sink]: <https://www.sengoku.co.jp/mod/sgk_cart/detail.php?code=EEHD-4FBE> "Seeed Studio 800035001 Raspberry Pi用アルミニウム・ヒートシンク"
[ftdi]: <http://www.sengoku.co.jp/mod/sgk_cart/detail.php?code=EEHD-0SK8> "スイッチサイエンスSSCI-010320 FTDI USBシリアル変換アダプター(5V/3.3V切り替え機能付き)"
[3kohm]: <http://akizukidenshi.com/catalog/g/gR-25302/> "カーボン抵抗(炭素皮膜抵抗) 1/4W 3kΩ (100本)"
[diode]: <http://akizukidenshi.com/catalog/g/gI-00881/> "ショットキーダイオード"
[2SC1815]: <http://akizukidenshi.com/catalog/g/gI-00881/> "トランジスタ (2SC1815GR 60V 150mA) 20個"
[dcFan]: <https://www.sengoku.co.jp/mod/sgk_cart/detail.php?code=557S-43JV> "SHICOH F4006AP-05QCW (0406-5) DCファン(5V)"
[si-sheet]: <https://www.sengoku.co.jp/mod/sgk_cart/detail.php?code=4AZ6-GFHU> "カモンSRS-40 ファン防振シリコンシート"
[FanGuard]: <https://www.sengoku.co.jp/mod/sgk_cart/detail.php?code=328B-2CE8>"40mm角ファンガード"
[case3]: <http://www.sengoku.co.jp/mod/sgk_cart/detail.php?code=EEHD-4KRC> "MultiComp MC-RP002-CLR Raspberry Pi B+専用ケース(クリア/ロゴあり)"


<!--イメージファイル-->
[system1]: system1.jpg "自作箱に収めた写真"
[system2]: system2.jpg "千石のファン付き箱に収めた写真"
[breadboard]: breadboard.jpg "ブレッドボード利用時の配線イメージ"
[circuit]: circuit.jpg "回路図"
[openCase]: openCase.jpg "自作箱を開けた写真"
[brokenCase]: brokenCase.jpg "箱の壊れた部分"




