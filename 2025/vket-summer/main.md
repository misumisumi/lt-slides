---
theme: dracula
title: オレオレVPNでどこでもフルトラVR
author: Sumi-Sumi
width: 1600
height: 900
margin: 0.100
slideNumber: true
title-slide-attributes:
  data-background-size: contain
revealjs-url: https://unpkg.com/reveal.js
---

# お前は誰だ？

## プロフィール

:::::::::::::: {.columns}
::: {.column width="60%" }

| 名前     | <font color="cyan">Sumi-Sumi</font> |
| :------- | :---------------------------------- |
| ぶいちゃ | メイド<br>インフラ集会運営 etc...   |
| リアル   | 院生（音声合成系）                  |
| インフラ | HomeLab                             |

![](./assets/imgs/thumbnail.png){width=60%}

:::
::: {.column width="40%"}
![](./assets/profile/avator.jpg){width=75%}
:::
::::::::::::::

## My HomeLab

- 当然のようにブレード鯖！
- InfiniBand、業務用AP (WiFi 6E)完備！
- まだ点火されていない！

![](./assets/imgs/PXL_20250719_112711859.jpg){width=55%}

# これからのイベントと言えば！

## VKET REAL！

![](./assets/imgs/vketreal.webp){width=80%}

---

もあるけれど...

<h1>お盆→帰省</h1>

::::::::::::::{.columns align=center}
::: {.column width="50%"}
![](./assets/imgs/summer_kisei.png){width=80%}
:::
::: {.column width="50%"}
![](./assets/imgs/obon_nasu_kyuri.png){width=30%}  
![](./assets/imgs/hanabi_family_bg.png){width=40%}
:::
::::::::::::::

## 帰省してもすること...無くない??

:::::::::::::: {.columns align=center}
::: {.column width="45%"}
<br>
<br>
![](./assets/imgs/smartphone_gorogoro_man_neet.png)
:::
::: {.column width="10%"}
<br>
<br>
<br>
<br>
![](./assets/imgs/mark_arrow_right.png)
:::
::: {.column width="45%"}
![](./assets/imgs/vr_game_mother.png)

<p style="font-size:72px">→ VRChatしよう！</p>
:::
::::::::::::::

# フルトラをするという強い意思

---

<p style="font-size:72px; color:violet;">フルトラは身体認識に影響を与える</p>

![](./assets/imgs/2025-07-17_19-38.png)
![](./assets/imgs/2025-07-17_19-38_1.png)

## 持ち物リスト

- ゲーミングノートPC
- HMD
- ベースステーション ×2
- トラッカ ×3～5

## ゲーミングノートPC

- お高い
- 割にデスクトップと比較すると性能が...
- とっても重い
- クソデカ充電器
  - 最早鈍器

<p style="font-size:72px; color:violet;">→ できれば買いたくなーい</p>
<p style="font-size:32px;">(HMD, フルトラ機材の重量には目を瞑る)</p>

# オレオレVPNでどこでもフルトラVR

## ![](./assets/imgs/virtual-desktop.png){height=64pt style="margin:0pt"}でリモート接続は可能

![](./assets/imgs/virtual-desktop-settings.png){height=70%}

フルトラは×

---

ドングル=USB...

<p style="font-size:72px;"><u>USBでやりとりされるデータ</u>を<br><font color="violet">ネットワークで送受信</font>すればいいじゃん！</p>

![](./assets/imgs/nobita.jpg){width=30%}

## USB/IP <font size=24px>[Hirofuchi+, 2005]</font>

- Server: USBの共有<font color="violet">元</font>マシン
- Client: USBの共有<font color="cyan">先</font>マシン
- <u>TCP/IP</u>で通信

![](./assets/imgs/usbip-design.png){width=35%}

---

## USB/IP

- Linux kernelに統合済み(3.17～)
- macやwindows向けの実装もある
  - server/client: [jiegec/usbip](https://github.com/jiegec/usbip)
  - client: [vadimgrn/usbip-win2](https://github.com/vadimgrn/usbip-win2)

<!-- Sumi-SumiのフルトラもUSB over Network！ -->

SuperDongleはできない...っぽい

---

ここまでで、フルトラVRはできる

1. USB/IPとVirtualDesktopの設定
2. ポート開放
3. ルーティング設定
   - FW：特定ポート宛ての通信を接続先IPに転送
   - SSHのポート転送
4. 接続確認

**USB/IPは暗号化されていない**

---

おそらく...

<p style="font-size:72px">父母兄妹・隣人はあなたのVR生活に興味津々！<br>通信を<font color="violet">傍受</font>するだろう</p>

:::::::::::::: {.columns align=center}
::: {.column width="50%"}
![](./assets/imgs/rhbms.jpg){width=70%}
:::
::: {.column width="50%"}
![](./assets/imgs/nukitashi.jpg){width=90%}
:::
::::::::::::::

<font size=32px><u>VirtualDesktopはend-to-endで暗号化</u>されています</font>

---

というよりは...

- アプリ毎にルーターでポート開放するのが面倒
- クライアント(接続先)側で操作が必要
  - 場合によってはSSH接続が必要

## <font color="violet">VPN</font>: <font color="violet">V</font>irtual <font color="violet">P</font>rivate <font color="violet">N</font>etwork

- 拠点間を仮想的な専用線で繋げるやつ
  - 「社内ネットワークに繋ぐにはー」で使うやつ
- 遠隔地でも同一ネットワークにいるように見せられる
- ポート開放は<u>VPNで使うやつのみでOK</u>

![](./assets/imgs/Virtual_Private_Network_overview.svg.png){width=40% style="background-color:white;"}

---

![](./assets/imgs/wireguard.svg){width=70% style="background-color:white;"}

- Linux Kernelに統合済(5.6～)
- マルチプラットフォーム
- 仕様が公開されている
  - <u>NDSS (セキュリティ分野のトップ会議)で採択済</u>
- 商用VPNでの採用実績もあり
  - ProtonVPN
  - TailscaleVPN

## 軽量かつ高速

![](./assets/imgs/wg-benchmark.png){width=50%}

<p style="font-size:24px">出典：https://www.wireguard.com/performance/</p>

## シンプルな構築手順

1. 秘密鍵と公開鍵の作成
   - 事前共有鍵も設定可
2. WireGuard用の仮想ネットワークデバイス作成
3. FWの設定
   - ローカルネットワークに出るため必須
4. 接続

---

- <font color="violet">両拠点共に100mbps</font>あれば、快適なVRChatができる
  - フルトラは<u>やや追従にラグ有</u>
- <font color="violet">HaritoraXやmocopiもできる</font>...はず...
- USB/IP
  - mac/windowsを接続元にするのは未検証<br>(事例はネットにあり)
  - Linux on mac (m1)で転送は検証済
- WireGuard
  - <font color="violet">中古デスクトップで十分</font>作成可

# この夏<br>VPN鯖からインフラ沼に飛び込もう！{.unlisted}
