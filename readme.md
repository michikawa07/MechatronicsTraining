# メカトロ講習

## ラズパイの環境設定

### 1. Ubuntu 22.04 LTS のインストール

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
sudo reboot
```

#### 日本語入力の有効化
やったこと(最短手順ではない可能性が高い)
 - Setting > Reaion & language > Manage Installed Lanuages をクリックして，出てきたポップアップで install を選ぶ．
 - Setting > Reaion & language > Language で 言語を Japanese に変更して，ログアウト&ログイン
    - ログイン時にフォルダ名を日本語にしますか？という質問が出るが絶対に **No**
 - Setting > Keyboard > Input Sources の "+" マークを押して Japanese > Japanese を選択

#### SSH 接続の設定

(これはRaspberry Pi 側の設定)

 - [参考1, SSH接続全般](https://qiita.com/010Ri/items/0a09356633655b5613ee)
 - [参考2, IP固定なしでの接続](https://qiita.com/NeK/items/a16cbc33dd3195226940#avahi-daemon2)
 - [参考3, ssh-keygenなしでの接続](https://qiita.com/FGtatsuro/items/4893dfb138f70d972904)

接続される側に必要なソフトのインストール
```bash
sudo apt -y install openssh-server # SSHのホストになるために必要なやつ
sudo apt -y install avahi-daemon   # ホスト名でアクセスできるようにするやつ
reboot # 念のため再起動
```

`openssh-server`が起動しているか確認
```bash
robot@raspi-default:~$ sudo systemctl status ssh
[sudo] password for robot: 
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2025-11-11 14:18:43 JST; 30min ago
```

`avahi-daemon`が起動しているかの確認
```bash
robot@raspi-default:~$ sudo systemctl status avahi-daemon
● avahi-daemon.service - Avahi mDNS/DNS-SD Stack
     Loaded: loaded (/lib/systemd/system/avahi-daemon.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2025-11-11 14:18:42 JST; 31min ago
```

SSH接続にはSSH鍵の生成と共有という手順が本来必要．面倒なので，SSH鍵なしで簡易接続できるようにする.

以下の設定ファイルを書き換える．
```bash
sudo nano /etc/ssh/sshd_config
```
長い設定ファイルの以下の部分を見つけて，同じ記述になるように書き換える．(ctrl+Sで保存，ctrl+Xで通常ターミナルへ戻る．)
```
...
# PermitEmptyPasswords no
PermitEmptyPasswords yes
...
# UsePAM yes
UsePAM no
```
設定を反映させるために以下のコマンドを実行する．
```bash
sudo service ssh restart
```
  
### 2. ラズパイをカスタマイズする

(作業はここから)

#### ホスト名を変更する
`{ホスト名}`の部分に自分のラズパイ固有の名前を付ける．
`raspi-{好きな文字列}`にするといいかな．
```bash
sudo hostnamectl set-hostname {ホスト名}
```

#### SSH接続できるか試す

※ここだけ windows PC を触ります．
windows PC で `Git Bash`を起動し，以下のコマンドを実行してssh接続できるか試す．
```bash
ssh robot@{ホスト名}.local
```

### 3. PiSugar2/3 Plus の設定

[PiSugar2の場合はこのサイト](https://docs.pisugar.com/docs/product-wiki/battery/pisugar2/pisugar-2-plus)，
[PiSugar3の場合はこのサイト](https://docs.pisugar.com/docs/product-wiki/battery/pisugar3/pisugar-3-series)のいう通りにすればOK

#### ハードウェアの取り付け

1. Raspberry Pi を Power off する
2. PiSugar2/3 Plus の電源が off であることを確認する
3. PiSugar2/3 Plus のネジナットの保護フィルムを剥がす
4. PiSugar2/3 Plus の4つのネジナットを Raspberry Pi ボードに合わせる
    - PiSugar ボードが RPI の下に来るように、PiSugar3 Plus のポゴピンと RPI の GPIO が同じ側に来るようにして、RPI ボードを優しく押し下げる
5. 付属のネジで PiSugar2/3 Plus ボードを Raspberry Pi ボードに固定する
6. PiSugar2/3 Plus の電源を on にする(実質的に Raspberry Pi の電源を入れることになる)
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

### 4. pigpio のインストール

[このサイト](https://note.com/caelum_zachanus/n/nb2abaaaa349e)を参考にした．

まず，python のツールを入れる．(すでに入っていても実行して問題ない)
```bash
sudo apt update
sudo apt install python-setuptools python3-setuptools build-essential
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

### 5. ROS2 Humble のインストール

今はやらない

<!-- todo : write later -->

## 開発環境の設定

Windows 側の設定


## モータを制御してみる．

※基板が適切に作成できたという仮定のもとです．

デーモンが立ち上がっているか確認
```bash
pgrep -a pigpiod # プロセスIDが表示されたらOK
pigs pigpv       # pigpio自体のバージョン番号が表示されたらOK.
```

terminalから簡単に動作確認．
```
pigs SERVO 12 1550 # GPIO12のピンから15500μs = 15.5msのパルス幅のPWM信号を出す
pigs SERVO 18 900 # GPIO12のピンから9000μs = 9.0msのパルス幅のPWM信号を出す
```
