# 最強って...？

## ~~最強...~~ 要件定義をしよう ...

1. *継続的*なサービスの提供
2. *ディスク冗長性*の確保
3. *いつでもどのマシンでも稼動*する状態の維持・管理
4. できるだけ低コストであること

内外のネットワークの冗長性、構成パーツの堅牢性については<br>ここでは考慮しない

## 要件を満すための技術選定

## 1. *継続的*なサービスの提供

:::::::::::::: {.columns}
::: {.column width="60%"}

- `pacemaker + corosync`
  - 2 台以上のマシンで構築可能
  - 古くからある
- `kubernetes`
  - 注目度がかなり高い
  - おうち k8s ってなんかかっこいい！

:::
::: {.column width="40%"}

<div class="stack">
  <img src="https://clusterlabs.org/assets/pacemaker-notext-36740eee11c7c4ccf3f653d855bbe5b28673ba7950234fa84b619c86a7d4d608.svg" height="130">
  <img src="https://raw.githubusercontent.com/kubernetes/kubernetes/master/logo/logo.svg" height="150">
</div>
:::
::::::::::::::

<p style="font-size: 200%; color: magenta;">-> kubernetes を選定</p>

## 2. *ディスク冗長性*の確保

:::::::::::::: {.columns}
::: {.column width="70%"}

- `drbd + pacemaker + corosync`
  - 2 台のマシン間でディスクをミラーリング
  - 小リソースでも冗長性を確保可能
- `Ceph (Rook/Ceph)`
  - 分散ストレージ
  - 構築・チューニングが難しい
  - 拡張性と信頼性に優れる
- `NAS + Raid`

:::
::: {.column height="90%" width="30%"}

<div class="stack">
  <img src="https://upload.wikimedia.org/wikipedia/commons/c/c2/DRBD_logo.svg" height="80">
  <img src="https://ceph.io/assets/bitmaps/Ceph_Logo_Stacked_RGB_Reversed_120411_fa.png" height="200">
</div>
:::
::::::::::::::
<p style="font-size: 150%; ">大容量ディスク -> <text style="color: magenta">drbd</text></p>
<p style="font-size: 150%; "> k8s 用永続ボリューム -> <text style="color: magenta">Rook/Ceph</text></p>

## 3. *いつでもどのマシンでも稼動*する状態の維持・管理

- k8s ノードは VM で必要数を確保
- -> 仮想化基盤が必要

![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiMtKKlItQH9ynKFnOgfa8O6Oen8m9wCOOn_XbFskZP9FGo3o9mcvQKhiwTK35uQ8Y3AAUiUm7jqMewAJ7vK2By-PSsA56e8G7ok7K_-3B9xU6EU-L5qxK5OA8adk1zXBo3iByrSJcYK-Y/s800/computer_server.png){width=450}

## 仮想化基盤をどうするか

:::::::::::::: {.columns}
::: {.column width="70%"}

- `Proxmox VE`
  - web コンソールで手軽に使える
  - よく使われていて情報が多い
- `VMWare vSphere`
  - 高くて買えない
- `Linux Distribution`
  - 使いなれた好みの環境で構築可能
  - 一からの構築になるため~~愛~~ 覚悟が必要

:::
::: {.column height="90%" width="30%"}

<div class="stack">
  <img src="https://www.proxmox.com/images/proxmox/logos/mediakit-proxmox-server-solutions-logos-dark.svg" height="120">
  <img src="https://assets.ubuntu.com/v1/594d0a0c-Canonical%20Ubuntu%20Dark.svg" height="100">
  <img src="https://archlinux.org/static/logos/archlinux-logo-light-scalable.1ae4cc2e2469.svg" height="120">
</div>
:::
::::::::::::::
-> <text style="font-size: 200%; color: Cyan;">NixOS</text> を採用

# 仮想化基盤に NixOS を<br>採用するということ

## NixOS

:::::::::::::: {.columns}
::: {.column width="70%"}

- ディシュトリビューションの一種
- システム・ユーザー**環境の全てを宣言的に設定可能**
  - 純粋関数型パッケージマネージャー `nix`
- 依存関係の強力な解決や環境のロールバックなどの魅力
- ZFS も比較的使いやすい！

:::
::: {.column width="35%"}

![](https://raw.githubusercontent.com/NixOS/nixos-artwork/master/logo/nixos-white.svg)

<text style="font-size: 80%"> 関数 $\lambda$ でラテン語<br>「Nix = 雪の結晶」を<br>表したロゴ</text>

:::
::::::::::::::

## 依存関係の強力な解決：Flakes
