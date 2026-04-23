i.MX Linux Board Support Package (BSP) 技術ブリーフィング：LF6.12.49_2.2.0

エグゼクティブ・サマリー

本資料は、NXP Semiconductorsが提供するi.MXプラットフォーム向けLinux OS Board Support Package（BSP）の構築、インストール、および高度な機能利用に関する技術指針をまとめたものである。リビジョンLF6.12.49_2.2.0に基づき、i.MX 6から最新のi.MX 9シリーズまでを網羅している。

主要な要点は以下の通りである：

* 広範なハードウェアサポート: i.MX 6, 7, 8, 8M, 8X, および9ファミリーの各SoCと開発プラットフォーム（SABRE-SD, EVK, MEK, FRDM等）に対応。
* 統合ツールチェーン: Yocto Projectを標準フレームワークとして採用し、Universal Update Utility (UUU) による効率的なイメージ書き込みをサポート。
* 複雑なブート構成: 特にi.MX 8および9シリーズにおいて、U-Boot、Arm Trusted Firmware、セキュリティファームウェア（Sentinel/ELE等）を統合した「imx-boot」コンポーネントが必須となっている。
* カーネル基盤: バージョン6.12.49のLinuxカーネルを採用し、デバイスツリー（DTB）による動的なハードウェア構成管理を行う。

1. システム概要と対象ハードウェア

本BSPは、i.MX開発システム上でU-Bootブートローダー、Linuxカーネル、およびルートファイルシステムを構築するためのバイナリ、ソースコード、支援ファイルの集合体である。

1.1 サポート対象SoCファミリー

ファミリー	代表的なSoC / プラットフォーム
i.MX 6	6QuadPlus, 6Quad, 6DualLite, 6SoloX, 6SLL, 6UltraLite, 6ULL, 6ULZ
i.MX 7	7Dual, 7ULP (Ultra Low Power)
i.MX 8	8QuadMax, 8QuadPlus, 8ULP
i.MX 8M	8M Plus, 8M Quad, 8M Mini, 8M Nano
i.MX 8X	8QuadXPlus, 8DXL, 8DXL OrangeBox, 8DualX
i.MX 9	91, 93, 95, 943 (EVK, QSB, Verdin, OrangeBox等)

1.2 技術ドキュメント体系

本ガイド（UG10163）に加え、以下の関連ドキュメントが補完関係にある：

* RN00210 (Release Notes): リリース情報、特定ボードの機能制限。
* UG10164 (Yocto Project User's Guide): ホスト設定、ツールチェーン構築、イメージ生成。
* RM00293 (Linux Reference Manual): i.MX用Linuxドライバの詳細。
* RM00284 (EdgeLock Enclave API): i.MX 8ULP, 93, 95のハードウェアセキュリティモジュール。

2. ソフトウェア構成要素の詳細

Linux OSを起動し実行するためには、SoCの世代に応じた複数のコンポーネントが必要となる。

2.1 ブートローダー (U-Boot / imx-boot)

* i.MX 6 / 7: U-Bootが単体で推奨される。SDカードの1KBオフセット位置に配置される。
* i.MX 8 / 9: imx-mkimageツールで構築されたimx-bootを使用。これにはU-Boot、Arm Trusted Firmware、DCD、システムコントローラファームウェア、SECO/Sentinelファームウェア等が含まれる。
* SPL (Second Program Loader): i.MX 8M等で有効。DDRの初期化を行い、U-BootやTEE OSをロードする役割を担う。

2.2 カーネルとデバイスツリー (DTB)

* カーネルイメージ: i.MX 6/7は32ビット（zImage）、i.MX 8/9は64ビット（Image）を使用。
* デバイスツリー: ハードウェア構成を記述。ファイル名はImage-[platform]-[board]-[configuration].dtbの形式。
  * LDOバイパス非対応の1.2 GHz CPU構成では、*ldo.dtbを使用する必要がある。

2.3 ルートファイルシステム (rootfs)

* BusyBox、共通ライブラリ、グラフィカルバックエンド（XWayland等）を含む。
* 拡張子は.ext4（標準）または.wic（SDカード用統合イメージ）。

3. 基本セットアップとユーティリティ

3.1 端末接続設定

ボードとホストPC間のシリアル通信には、以下の設定が必要である：

* 通信速度: 115200 bps
* データビット: 8
* パリティ: なし
* ストップビット: 1
* フロー制御: なし
* 接続端子: i.MX 8等ではMicro USB（FT4232等）を介して、Armコア用（第1ポート）とSCU用（第2ポート）のコンソールが提供される。

3.2 Universal Update Utility (UUU)

UUUは、WindowsまたはLinuxホストからi.MXデバイスへイメージをダウンロードする製造ツールである。

* 要件: バージョン1.5.220以上。
* 主な操作コマンド例:
  * eMMCへの全イメージ書き込み: uuu -b emmc_all <bootloader> <rootfs.wic>
  * FlexSPI NORへの書き込み: uuu -b qspi <bootloader>
  * SPLおよびU-BootのUSB経由実行: uuu -b spl <bootloader>

4. ブートメディアの準備とブートプロセス

4.1 SD/MMCカードのレイアウト

デフォルトのイメージレイアウトは以下の通り定義されている：

開始アドレス (セクタ)	サイズ (セクタ)	形式	説明
0x2 / 32 / 33 *	最大 20478	RAW	ブートローダー (U-Boot / imx-boot)
20480 (10MB)	1024000	FAT	カーネルイメージおよびDTB
1228800	残り	Ext3/Ext4	ルートファイルシステム

*SoCモデルによりオフセットが異なる（i.MX 6/7は2、i.MX 8M系は66、その他i.MX 8/9は64セクタ）。

4.2 高度なインストール手順

1. パーティション作成: fdiskを使用して、ブートイメージ用（FAT）とルートファイルシステム用（Ext3/4）の2つのパーティションを作成する。
2. ブートローダーの書き込み: ddコマンドを使用し、SoCに応じた適切なseekオフセットを指定する。
3. カーネルとDTBの配置: FATパーティションに直接コピーするか、特定のRAWアドレス（1MB以降等）に書き込む。
4. M4コアイメージの利用: i.MX 6SoloX, 7Dual, 7ULP等では、QuadSPIメモリにArm Cortex-M4イメージを書き込むことで全機能を利用可能。U-Bootのrun update_m4_from_sdコマンド等が利用される。

5. U-Bootによるシステム構成と運用

U-Bootコンソールでは、ネットワーク通信（TFTP/DHCP）を介した動的なイメージのダウンロードや、環境変数の管理が可能である。

* 環境変数の初期化: env default -f -a
* ネットワーク設定: serverip、bootfile、fdtfileを個別に設定。
* 表示出力の設定: videoパラメータを用いて、HDMIやLVDS（HannStar等）の解像度、色深度、エミュレーション設定をカーネルに渡す。
  * 例：video=HDMI-A-1:1920x1080@60
* eMMCの利用: ボード上のeMMC（i.MX 6 SABREではUSDHC4, i.MX 8ではUSDHC1等）をブートデバイスとして構成し、SDカードから移行することが可能。
