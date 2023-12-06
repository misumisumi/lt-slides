# NIXで管理するpython環境

エンジニア作業飲み集会アドカレ3日目はpython環境についてです。

機械学習を用いた研究をしていると「先輩のコードが動かない」、
「公開されている古いプロジェクトのメンテが終了しており新規環境で動かない」ということに直面し、
環境構築に多くの時間が割かれるという悲劇に見舞わることが多々あります。

しかし、NIXを用いることでそのような悲劇を極限まで回避できることができます。

なお、既に`flake`が使用できる`nix`がシステムに導入されているものとします。

## Project Templates

[nix-templates](https://github.com/misumisumi/nix-templates/tree/main)に今回作成した環境のテンプレートを置いておきます。  
`nix flake new <project-name> -t github:misumisumi/nix-templates#<templatename>`でテンプレートを用いて環境を作成できます。  
`How to`には`flake.nix`のみ掲載しています。

## NIXで管理するpython環境 (nix+poetry)

任意バージョンのpythonインタプリタやバイナリパッケージ、ライブラリなどプロジェクト全体を`nix flake`によって管理し、pythonパッケージを`poetry`によって管理します。  
この方法ではubuntuなどで使うように`poetry`でpythonプロジェクトを管理できます。  
PyPiからインストールされるビルド済みパッケージは依存関係が同梱されている場合は動作することが多いですが、動作しない場合は環境変数`LD_LIBRARY_PATH`にパスを追加することで動作させます。  
[devenv](https://devenv.sh)により`nix`環境をcontainerへ出力することが可能なため、`nix`は使用できないが`docker`が動作する環境への可搬性はあります。  
この方法では`nix`の知識はそれほど必要ありませんが、環境変数を駆使するために開発シェル外のappの動作に支障を及ぼす場合があります。

## How to

```sh
# 開発環境の起動 (プロジェクト内にpoetryによって.venvが作成される)
nix develop ".#default"

# パッケージの追加
poetry add <package-name>
```

<details><summary>flake.nix</summary><div>

```nix
{
  description = "template of python project managed by poetry";
  inputs =
    {
      nixpkgs.url = "github:NixOS/nixpkgs/nixpkgs-unstable";
      nixpkgs-stable.url = "github:NixOS/nixpkgs/nixos-23.11";
      flake-parts.url = "github:hercules-ci/flake-parts";
      nixpkgs-python.url = "github:cachix/nixpkgs-python";
      devenv = {
        url = "github:cachix/devenv/python-rewrite";
        inputs.nixpkgs.follows = "nixpkgs";
        inputs.poetry2nix.follows = "poetry2nix";
      };
      mk-shell-bin.url = "github:rrbutani/nix-mk-shell-bin";
      nix2container = {
        url = "github:nlewo/nix2container";
        inputs.nixpkgs.follows = "nixpkgs-stable";
      };
      poetry2nix = {
        url = "github:nix-community/poetry2nix";
        inputs.nixpkgs.follows = "nixpkgs";
      };
    };

  outputs = inputs @ { flake-parts, ... }:
    flake-parts.lib.mkFlake
      {
        inherit inputs;
      }
      {
        imports = [
          inputs.devenv.flakeModule
        ];
        flake = {
          # nixpkgsからのインストールにビルド済みパッケージを使用する
          nixConfig = {
            extra-substituters = [
              "https://nixpkgs-python.cachix.org"
              "https://cuda-maintainers.cachix.org"
              "https://devenv.cachix.org"
            ];
            extra-trusted-public-keys = [
              "nixpkgs-python.cachix.org-1:hxjI7pFxTyuTHn2NkvWCrAUcNZLNS3ZAvfYNuYifcEU="
              "cuda-maintainers.cachix.org-1:0dq3bujKpuEPMCX6U4WylrUDZ9JyUG0VpVZa7CNfq5E="
              "devenv.cachix.org-1:w1cLUi8dv3hnoSPGAuibQv+f9TZLr6cv/Hm9XgU50cw="
            ];
          };
        };
        systems = [ "x86_64-linux" ];
        perSystem =
          { config
          , self'
          , inputs'
          , pkgs
          , lib
          , system
          , ...
          }:
          {
            _module.args.pkgs = import inputs.nixpkgs {
              inherit system;
              # nixpkgsに問題がある場合はここでパッチを当てる
              overlays = [
                inputs.poetry2nix.overlays.default
                (final: prev: {
                  inherit (inputs.nixpkgs-stable) skopeo;
                })
              ];
              config = {
                allowUnfree = true;
              };
            };
            devenv.shells.default =
              let
                inherit (inputs.poetry2nix.lib.mkPoetry2Nix { inherit pkgs; }) mkPoetryEnv;
                # LD_LIBRARY_PATHに追加するパッケージ
                buildInputs = with pkgs;[
                  cudaPackages_11_8.cudatoolkit
                  cudaPackages_11_8.cudnn_8_9
                  pythonManylinuxPackages.manylinux2014Package
                  stdenv.cc.cc
                  zlib
                ];
              in
              {
                containers.default = {
                  name = "python-poetry";
                  startupCommand = "bash";
                  copyToRoot = null;
                };
                env = {
                  # 一部、LD_LIBRARY_PATHに記載しないと動作しない場合がある
                  # /run/opengl-driver/libはNixOSでCUDAを使用するときに必要
                  LD_LIBRARY_PATH = "${with pkgs; lib.makeLibraryPath buildInputs}:/run/opengl-driver/lib";
                  XLA_FLAGS = "--xla_gpu_cuda_data_dir=${pkgs.cudaPackages_11_8.cudatoolkit}"; # For tensorflow with GPU support
                };
                # バイナリパッケージはここ記載
                packages = with pkgs; [
                  bashInteractive
                ];
                languages.python = {
                  enable = true;
                  manylinux.enable = false;
                  package = pkgs.python310;
                  poetry = {
                    enable = true;
                  };
                };
              };
          };
      };
}


```

</dev></details>

## NIXで管理するpython環境 (poetry2nix)

より再現性を重視するのであれば[poetry2nix]()を使用することを検討してください。  
`poetry2nix`は`poetry`によって生成された`poetry.lock`を解析し、nixpkgsの形式である`derivation`の形式に変換することで`nix`によってpythonパッケージも管理します。  
`nix`で解決し`flake`によるピンニングにより環境の再現性は前述の`nix+poetry`よりも飛躍的に高いです。
反面、pythonパッケージをsourceからビルドするまたはビルド済みパッケージにpatchを当てるために環境作成が非常に遅く、また`poetry2nix`で追跡されていないパッケージについてビルドが失敗する場合が多くこの場合は`override`によって依存関係を手動で解決する必要があります。  
特に`override`については`nix`を使い始めた人にとっては敷居が高いでしょう。

### How to

```sh

# poetryの実行
nix run ".#poetry" -- <command>

# パッケージの追加 (--lock必須!)
nix run ".#poetry" -- add <name> --lock

# 開発環境へ入る
nix develop ".#default"

# direnvを使っているならば`poetry.lock`を作成した後に
direnv allow
```

<details><summary>flake.nix</summary><div>

```nix

{
  description = "template of python project managed by poetry2nix";
  inputs =
    {
      nixpkgs.url = "github:NixOS/nixpkgs/nixpkgs-unstable";
      nixpkgs-stable.url = "github:NixOS/nixpkgs/nixos-23.11";
      flake-parts.url = "github:hercules-ci/flake-parts";
      nixpkgs-python.url = "github:cachix/nixpkgs-python";
      devenv = {
        url = "github:cachix/devenv/python-rewrite";
        inputs.nixpkgs.follows = "nixpkgs";
        inputs.poetry2nix.follows = "poetry2nix";
      };
      mk-shell-bin.url = "github:rrbutani/nix-mk-shell-bin";
      nix2container = {
        url = "github:nlewo/nix2container";
        inputs.nixpkgs.follows = "nixpkgs-stable";
      };
      poetry2nix = {
        url = "github:nix-community/poetry2nix";
        inputs.nixpkgs.follows = "nixpkgs";
      };
    };

  outputs = inputs @ { flake-parts, ... }:
    flake-parts.lib.mkFlake
      {
        inherit inputs;
      }
      {
        imports = [
          inputs.devenv.flakeModule
        ];
        flake = {
          nixConfig = {
            extra-substituters = [
              "https://nixpkgs-python.cachix.org"
              "https://cuda-maintainers.cachix.org"
              "https://devenv.cachix.org"
            ];
            extra-trusted-public-keys = [
              "nixpkgs-python.cachix.org-1:hxjI7pFxTyuTHn2NkvWCrAUcNZLNS3ZAvfYNuYifcEU="
              "cuda-maintainers.cachix.org-1:0dq3bujKpuEPMCX6U4WylrUDZ9JyUG0VpVZa7CNfq5E="
              "devenv.cachix.org-1:w1cLUi8dv3hnoSPGAuibQv+f9TZLr6cv/Hm9XgU50cw="
            ];
          };
        };
        systems = [ "x86_64-linux" ];
        perSystem =
          { config
          , self'
          , inputs'
          , pkgs
          , lib
          , system
          , ...
          }:
          {
            _module.args.pkgs = import inputs.nixpkgs {
              inherit system;
              # nixpkgsに問題がある場合はここでパッチを当てる
              overlays = [
                inputs.poetry2nix.overlays.default
                (final: prev: {
                  inherit (inputs.nixpkgs-stable) skopeo;
                })
              ];
              config = {
                allowUnfree = true;
                cudaSupport = true;
              };
            };
            apps = {
              poetry.program = "${pkgs.poetry}/bin/poetry";
            };
            devenv.shells.default =
              let
                inherit (inputs.poetry2nix.lib.mkPoetry2Nix { inherit pkgs; }) mkPoetryEnv;
              in
              {
                containers.default = {
                  name = "project-name";
                  startupCommand = "bash";
                  copyToRoot = null;
                };
                env = {
                  LD_LIBRARY_PATH = "${with pkgs; lib.makeLibraryPath [stdenv.cc.cc]}:/run/opengl-driver/lib";
                  POETRY_VIRTUALENVS_CREATE = true;
                  POETRY_VIRTUALENVS_IN_PROJECT = true;
                };
                packages =
                  let
                    myPythonEnv = mkPoetryEnv {
                      projectDir = ./.;
                      editablePackageSources = {
                        my-app = ./src;
                      };
                      python = pkgs.python310;
                      preferWheels = true;
                      extraPackages = ps: with ps; [ ];
                      overrides = pkgs.callPackage ./override.nix { };
                    };
                  in
                  with pkgs;
                  [
                    bashInteractive
                    myPythonEnv
                    poetry
                  ];
              };
          };
      };
}

```

</dev></details>

<details><summary>override.nix (参考)</summary><div>

```nix
{ lib, pkgs, ... }:
pkgs.poetry2nix.overrides.withDefaults (final: prev:
let
  notUseWheelPackages = [ "llvmlite" "numba" "soundfile" "torch" "torchvision" ];
in
lib.listToAttrs (map (name: lib.nameValuePair name (prev.${name}.override { preferWheel = false; })) notUseWheelPackages)
//
{
  typing = null;
  pybind11 = pkgs.python310Packages.pybind11.overridePythonAttrs (old: {
    inherit (prev.pybind11) src;
  });
  pytextgrid = prev.pytextgrid.overridePythonAttrs (old: {
    postInstall = ''
      rm -f $out/LICENSE
    '';
  });
  inaspeechsegmenter = prev.inaspeechsegmenter.overridePythonAttrs (old: {
    postInstall = ''
      rm -f $out/LICENSE
    '';
  });
  onnxruntime-gpu = prev.onnxruntime-gpu.overridePythonAttrs (old: {
    buildInputs = with pkgs.cudaPackages_11_8; old.buildInputs ++ [
      cudnn
      cudatoolkit
    ];
    autoPatchelfIgnoreMissingDeps = lib.optionals pkgs.stdenv.isLinux [
      "libcuda.so.1"
      "libnvinfer.so.8"
      "libnvinfer_plugin.so.8"
      "libnvonnxparser.so.8"
    ];
  });
}
  //
(with pkgs; with prev;
let
  fixDerivation = { name, setupRequires, installRequires, override }:
    (prev.${name}.override override).overridePythonAttrs (old: {
      nativeBuildInputs = (old.nativeBuildInputs or [ ]) ++ setupRequires;
      propagatedBuildInputs = (old.propagatedBuildInputs or [ ]) ++ installRequires;
    });
  mkOverrides = lib.mapAttrs
    (name: value: fixDerivation {
      inherit name;
      setupRequires = value.setupRequires or [ ];
      installRequires = value.installRequires or [ ];
      override = value.override or  { };
    });
in
mkOverrides {
  mecab = { setupRequires = [ setuptools pkgs.mecab ]; };
  mecab-python3 = { setupRequires = [ setuptools pkgs.mecab ]; };
  tensorflow-io-gcs-filesystem = { installRequires = [ libtensorflow ]; };
  pyreaper = { setupRequires = [ setuptools cython ]; };
  pyopenjtalk = { setupRequires = [ setuptools cmake ]; };
  nnmnkwii = { setupRequires = [ setuptools ]; };
  openai-whisper = { setupRequires = [ setuptools ]; };
  torchvision = { setupRequires = [ autoPatchelfHook ]; };
}
))
```

</dev></details>

## まとめ

ここまででNIX上でのpython環境の構築を紹介しました。
NIX上でのpython環境の構築は他のdistribution上の構築するよりも煩雑です。  
これはpythonパッケージが命令型の管理であるのに対し`nix`が宣言型の管理であることに由来し、現在もコミュニティによって様々な手法が検討されています。  
また、開発環境全体の管理であればdev containerで十分と感じる人もいるでしょう。
しかし、`nix`による再現性の安心感を知ってしまうと中々他のツールには戻れないものです...。

## NIXを知らない人向け

### 現在のpython環境の問題

[poetry](https://python-poetry.org)や[rye](https://rye-up.com)の登場によって、pythonの依存パッケージについてLockファイルの生成による再現可能な環境の構築ができるようになりました。  
しかし、CUDAやBLASのようなpython外の依存関係を用いる場合にはシステムにインストールされているバージョンに左右されるため再現できない問題が生じます。  
dockerなどのdev containerを用いることで一応の解決はできますが、依存関係の記載漏れ、パッケージバージョンの不明記などのリスクにより再現性には難があります。

### NIXとは

[公式HP](https://nixos.org)で以下のように紹介されています。

```
Nix is a tool that takes a unique approach to package management and system configuration.
Learn how to make reproducible, declarative and reliable systems.
```

`nix`は非常に優秀なパッケージマネージャーです。  
他のパッケージマネージャーと異なりグローバルにパッケージがインストールされることはありません。(`/usr/bin`や`/usr/lib`の中に何もない)  
また、パッケージビルドはサンドボックス内で実行されます。  
そのため、依存関係が不十分であればビルドが失敗するため依存関係の記載漏れのリスクが少なくなります。  
加えて各パッケージはハッシュで管理されており同一バージョンのパッケージでも依存関係のバージョンが異なれば別々で扱われます。  
例え依存関係の破壊的変更によりパッケージが動作しなくてもロールバックによって以前の状態に戻すことが容易に可能です。

パッケージビルドで使用されるサンドボックス環境を開発環境として使用することもできます。(`nix-shell`)  
以前の`nix`では`nixpkgs`(パッケージリポジトリ)は導入した時点のものが使用されるために今日の環境が明日動作することは保証されませんでした。
しかし、[Flakes](https://nixos.wiki/wiki/Flakes)の登場によってこの問題の解決が図られました。  
Flakesは宣言的な入力(依存するリポジトリなど)と出力、Lockファイルによるピンニングにより環境の再現性を実現します。
