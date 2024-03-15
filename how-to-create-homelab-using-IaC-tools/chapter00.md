---
theme: dracula
title: IaCを駆使して<br>最強の HomeLab を構築したよ！
author: Sumi-Sumi
width: 1280
height: 720
margin: 0.05
slideNumber: true
dependencies: [{ src: "plugin/plantuml.js", async: true }]
---

## 自己紹介

:::::::::::::: {.columns}
::: {.column width="60%" }

|      |                                                        |
| ---- | ------------------------------------------------------ |
| 名前 | **Sumi-Sumi** <br> (~~🐦~~ $\mathbb{X}$: @SumiSumiVRC) |
| 役割 | しがない (やつれた？) 大学院生 / ときどきメイド        |
| 専門 | 音声合成 (特に声質変換)                                |
| 最近 | おうち k8s環境が完成しました！ <br> 就活ダメです！     |

:::
::: {.column width="40%"}
![](./figs/VRChat_2024-02-05_00-40-26.027_1080x1920_trim.jpg){height=480px}
:::
::::::::::::::

## 目次

- イントロダクション
- 自宅環境の紹介
- 要件定義と技術選定
- 開発環境、クラスタ管理・設定、全てを IaC で構築
- 仮想化基盤としての NixOS
- NixOS による k8s クラスタの構築
- Terraform による仮想インスタンスの管理
- Ansible による HomeLab・k8s の初期設定
