# MT7927搭載のProArt X870EでUbuntu 24.04のWiFi/Bluetoothを動かすまで

去年、自作PCに挑戦したのですが、その際にProArt X870E-CREATOR WIFIというモデルのマザーボードを用いて組みました。
WindowsとUbuntuをそれぞれインストールしたのですが、Windowsでは有線・無線のインターネットとBluetoothが正常に動作するにも関わらず、UbuntuではネットワークもBluetoothも動作しない状況となりました。
というのも、ProArt X870E-CREATOR WIFIで採用されているネットワークやBluetoothを処理しているMediaTek MT7927というチップが比較的最新のものであったために、Ubuntuでは対応するドライバーがなく、認識できない状況にありました。
ネットワークが全く動かないとなると非常に不便だったため、最近重い腰を上げて解決しようと試みました。
試行錯誤の結果としてネットワーク機能が回復し、Bluetoothも完全放電後であれば認識できるようになり、一定は成果があったことから、備忘録として残したいと思います。

## 前提

前提を詳細に書いておきます。

### 自作PCのスペックやOSの情報

OSはWindows11 ProをSSD1に、Ubuntu 24.04.4 LTSをSSD2にそれぞれインストールしました。
マザーボードのBiosは比較的最近の2202を、Ubuntu 24.04.4 LTSのカーネルはLinux 6.17.0-23-genericを用いています。
（メモリの高騰が激しい昨今ですが、同様のスペックを今購入したら何円くらいになるのだろうか...）

| カテゴリ     | 型番                                        |
| ------------ | ------------------------------------------- |
| CPU          | AMD Ryzen 9 9950X3D                         |
| マザーボード | ASUS ProArt X870E-CREATOR WIFI              |
| メモリー     | Crucial Pro 128GB Kit (64GBx2)              |
| SSD1         | Samsung SSD 9100 PRO 2TB                    |
| SSD2         | Samsung SSD 990 PRO 2TB                     |
| GPU          | ASUS TUF Gaming GeForce RTX 4080 SUPER 16GB |
| 電源         | Cooler Master X Silent MAX Platinum 1300    |

### 症状

２つのOSをインストールして動作確認したことで、比較をすることができました。
Windows11 Proではネットワーク機能もBluetooth機能も正常に動作します。
それに対して、Ubuntuではネットワークが正常に動作せず、Bluetoothも設定に項目すら表示されない状況でした。
そのため、Ubuntuはネットワーク環境なしでしばらく使用していました。

| 機能                           | Windows11 Pro | Ubuntu 24.04.4 LTS |
| ------------------------------ | ------------- | ------------------ |
| Wifiや有線のインターネット機能 | 正常動作      | 動作不良           |
| Bluetooth機能                  | 正常動作      | 動作不良           |

:::note info
Ubuntu 24.04.4 LTSでの症状をもう少し詳しくかいておきます。
設定画面ではそもそもWifiの項目がなく、Bluetoothは項目があるものの、オンにしようとするも反応なし。
インターネットの有線接続の項目では、２つあるLANポートをそれぞれ認識するものの、ping 8.8.8.8を叩いてもインターネットには結局接続できない状態でした。
:::

:::note warn
SNSを見ると同じチップを使用したマザーボードを使用している方で似た症状で悩んでいる方が見受けられます。
しかし、完全放電後にのみ使用可能など、少しばかり症状に差異が見られました。
読者の環境でも適用可能かどうかは、ご自身で見極めていただけますと幸いです。
:::

## 原因

前項で示した通り初期不良ではなく、自作PCのマザーボードであるProArt X870E-CREATOR WIFIに搭載されているMT7927というチップが比較的新しいものであり、Ubuntuが対応するドライバーを持っていなかったことから、WifiやBluetoothが機能しない状態にありました。

## やったこと

### USB-LAN変換器を用いたネットワーク接続

なにをしようにもネットワーク環境がないと、他のパソコンからデータをUSBで移行しないといけなかったりで不便なため、一時的なものでも良いのでネットワーク環境を用意することにしました。
Ubuntuに対応したチップを搭載したUSB-LAN変換器もしくはネットワークカードを用意しました。
これまでしっかりとしたUSB-LAN変換器を持っていなかったので、[UGREEN 有線LANアダプター 2.5Gbps](https://amzn.asia/d/0dhuPXJ8)を購入。

接続すると認識されるものの、設定画面上ではインターネット接続中となってしまいました。
nmcliコマンドを叩くと当該変換器のIPが割り当てられておらず、GENERAL.STATEも70となっていました。
省電力モードになっていたことが原因のようで、下記のコマンドで回復しました。

```bash
sudo ethtool --set-eee enx[hogehoge] eee off
```

### MT7927のドライバー用意とインストール

参考にした記事はこちらになります。

[Fix: ASUS ProArt X870E (MT7927 / 14c3:6639) WiFi Not Working on Ubuntu 24.04 – Firmware Solution](https://www.reddit.com/r/Ubuntu/comments/1rjepzd/fix_asus_proart_x870e_mt7927_14c36639_wifi_not/)

1\. ドライバーの準備
こちらのパッケージ[mediatek-mt7927-dkms](https://github.com/jetm/mediatek-mt7927-dkms)をインストールします。

```bash
sudo apt install dpkg-dev
make deb
sudo dpkg -i mediatek-mt7927-dkms_*.deb
```

2\. マザーボード対応のドライバをダウンロード

使用しているマザーボードのWindows11用の無線ドライバをダウンロードします。
著者の環境では[ProArt X870E-CREATOR WIFIの公式サイト](https://www.asus.com/jp/motherboards-components/motherboards/proart/proart-x870e-creator-wifi/helpdesk_download?model2Name=ProArt-X870E-CREATOR-WIFI)からWindows11用のWifiドライバーをダウンロードしました。

ドライバーをunzipして、mtkwlan.datという名前のファイルがあることを確認する

```bash
unzip hogehoge-driver.zip
cd hogehoge-driver
ls | grep mtkwlan.dat
```

:::note info
hogehoge-driverにはダウンロードしたファイルの名前を指定してください
:::

3\. 対応するドライバーを抽出する

1で利用したパッケージのスクリプトを用いてドライバーを抽出します。
[mediatek-mt7927-dkms](https://github.com/jetm/mediatek-mt7927-dkms)をローカルに用意します。

```bash
git clone https://github.com/jetm/mediatek-mt7927-dkms.git
```

mediatek-mt7927-dkmsフォルダの直下にmtkwlan.datを移動させ、抽出したドライバーを格納するための適当なフォルダを用意した後に、extract_firmware.pyで抽出します。

```bash
mv mtkwlan.dat mediatek-mt7927-dkms
cd mediatek-mt7927-dkms
mkdir results
python extract_firmware.py mtkwlan.dat results
```

resultsフォルダに３つのファイルがあれば成功です。

```txt
BT_RAM_CODE_MT6639_2_1_hdr.bin
WIFI_MT6639_PATCH_MCU_2_1_hdr.bin
WIFI_RAM_CODE_MT6639_2_1.bin
```

4\. ドライバを配置

3で抽出したドライバーをMT7927用のフォルダに移動させます。

```bash
cd results
sudo mv BT_RAM_CODE_MT6639_2_1_hdr.bin /lib/firmware/mediatek/mt7927/
sudo mv WIFI_MT6639_PATCH_MCU_2_1_hdr.bin /lib/firmware/mediatek/mt7927/
sudo mv WIFI_RAM_CODE_MT6639_2_1.bin /lib/firmware/mediatek/mt7927/
```

Initramfsをアップデートした後、再起動します

```bash
sudo update-initramfs -u
sudo rfkill unblock all
sudo reboot
```

## 結果

ネットワークが回復し、有線、無線ともに安定して使用できる状態となりました。
未だにBluetoothがWindowsにおいて完全放電後でも認識されないこともありますが、今回の目標としていたネットワークの問題の解決は達成することができました。

## 学び

- Ubuntuをインストールする際はドライバーが公式にサポートされているかを事前に確認すると楽になる。
- ハイエンドのマザーボードだからといって（お金をかけたからといって）OS側で使用できる訳ではない。
- 起動時の動作検証は```sudo dmesg```である程度の原因の切り分けが可能
- こうした低レイヤーの話でもChatGPTを利用すれば解決策が様々出てくるものの、実行する際は本当に必要なコマンドか疑う必要がある

## 参考
