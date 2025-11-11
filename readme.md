# メカトロ講習

## 環境設定

### Ubuntu 22.04 LTS のインストール

※この章の作業が完了した状態のSDカードを配布する．

#### OSのダウンロード
[公式サイト](https://ubuntu.com/tutorials/how-to-install-ubuntu-desktop-on-raspberry-pi-4#1-overview)を参照すれば用意は簡単にできる．

#### 初回設定

初回起動時にするべきもろもろの設定

- "ubuntu system configuration" は "English" を選択
- "keyboard layout" は "English(US) > English(US)" を選択
- "Wi-Fi" はその時のWi-Fi環境に合わせて設定．
- "Where are you?" は "Tokyo"
- "Who are you" は以下のようにする
   - "Your name" は空白
   - "Your computer;s name" は "raspi-defalut" ("ホスト名"と呼ばれるもの．後で個々人に合わせて変える)
   - "Pick a username" は "robot" (なんでもいい)
   - "Choose a password" と "Confirm your password" は `いつもの`
   - "Log in automatically" を選択

※ 起動後，少し放置しないと，firefoxなどがインストールされてないかも．

#### 最低限のソフトのインストール
```bash
sudo apt update
sudo apt -y install git
sudo apt -y install curl
sudo apt -y install ssh
```

#### 日本語入力の有効化
```bash 
sudo apt install -y ibus-mozc
sudo apt install -y mozc-utils-gui
````


### ラズパイをカスタマイズする

#### ホスト名を変更する
`raspi-{好きな文字列}`にするといいかな．
```bash
sudo hostnamectl set-hostname [コンピュータ名]
```



<!-- todo : write later -->

### PiSugar3 Plus の設定

[このサイト](https://docs.pisugar.com/docs/product-wiki/battery/pisugar3/pisugar-3-series)のいう通りにすればOK

#### ハードウェアの取り付け

1. Raspberry Pi を Power off する
2. PiSugar3 Plus の電源が off であることを確認する
3. PiSugar3 Plus のネジナットの保護フィルムを剥がす
4. PiSugar3 Plus の4つのネジナットを Raspberry Pi ボードに合わせる
    - PiSugar ボードが RPI の下に来るように、PiSugar3 Plus のポゴピンと RPI の GPIO が同じ側に来るようにして、RPI ボードを優しく押し下げる
5. 付属のネジで PiSugar3 Plus ボードを Raspberry Pi ボードに固定する
6. PiSugar3 Plus の電源を on にする(実質的に Raspberry Pi の電源を入れることになる)
    - USB-C側のボタンを 短押し -> 長押し の順に押す
	 - 4つある緑のLEDが1つずつ点灯していき、その後青のLEDが点灯したら話してよい

#### ソフトウェアのインストール

以下のコマンドをターミナルで実行する．
```bash
wget https://cdn.pisugar.com/release/pisugar-power-manager.sh
bash pisugar-power-manager.sh -c release
```
途中の質問は以下のようにしてください．
 - ユーザ名: Admin (デフォルトのままでいい)
 - パスワード：1111 (なんでもいい)

ブラウザで `http://<your_ip_address>:8421/` にアクセスし，バッテリ残量等が確認できれば成功．  
`your_ip_address` は `$ hostname -I` コマンドで確認できるIPアドレスに置き換えてください．
また，認証を求められて場合，上記のユーザ名とパスワードを入力してください．

`http://<your_ip_address>:8421/` の Setting で，
 - Battery Input Protection を Enable
 - Soft Shutdown を Enable
に設定することを推奨します．

### SSH 接続の設定
#### Raspberry Pi 側の設定

<!-- todo : write later -->

#### Windows 側の設定

<!-- todo : write later -->

### pigpio のインストール

[このサイト](https://note.com/caelum_zachanus/n/nb2abaaaa349e)を参考にした．

まず，python のツールを入れる．(すでに入っていても実行して問題ない)
```bash
sudo apt update
sudo apt install python-setuptools python3-setuptools
```

次に，pigpio のソースコードをダウンロードしてインストール/ビルドする．
```bash
wget https://github.com/joan2937/pigpio/archive/master.zip
unzip master.zip
cd pigpio-master
make
sudo make install
```

最後に，pigpio デーモンを systemd サービスとして登録する．  
= pigpioがいつでも使えるようにする

ひとまず，以下のコマンドでCLIエディタを開く．
```bash
sudo nano /lib/systemd/system/pigpiod.service
```
以下の内容をコピペして保存する．
```ini
[Unit]
Description=Daemon required to control GPIO pins via pigpio
[Service]
ExecStart=/usr/local/bin/pigpiod
ExecStop=/bin/systemctl kill -s SIGKILL pigpiod
Type=forking
[Install]
WantedBy=multi-user.target
```
サービス登録のため以下のコマンドを実行する．
```
sudo systemctl enable pigpiod
sudo systemctl start pigpiod
```

### ROS2 Humble のインストール

<!-- todo : write later -->

## モータを制御してみる．

※基板が適切に作成できたという仮定のもとです．

デーモンが立ち上がっているか確認
```bash
pgrep -a pigpiod # プロセスIDが表示されたらOK
pigs pigpv       # pigpio自体のバージョン番号が表示されたらOK.
```

terminalから簡単に動作確認．
```
pigs SERVO 12 15500 # GPIO12のピンから15500μs = 15.5msのパルス幅のPWM信号を出す
pigs SERVO 18 900 # GPIO12のピンから9000μs = 9.0msのパルス幅のPWM信号を出す
```
