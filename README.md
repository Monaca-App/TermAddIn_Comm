<img src="images/Monaca_store.png" align="right" width="130" alt="Monaca Mascot" />

# TermAddIn_Comm - COM ポートシリアル通信エクステンション

👉 [← TermAddIn 統合本体に戻る](https://github.com/Monaca-App/TermAddIn)  
👉 [← Monaca アプリ工房（トップ）に戻る](https://github.com/Monaca-App/store)

---

[![Platform](https://img.shields.io/badge/Platform-Windows%2010%20%2F%2011-blue.svg)](https://www.microsoft.com/)
[![Excel](https://img.shields.io/badge/Excel-2016%20%2F%202019%20%2F%202021%20%2F%20M365-green.svg)](https://www.microsoft.com/excel)
[![Framework](https://img.shields.io/badge/.NET%20Framework-4.8-purple.svg)](https://dotnet.microsoft.com/)
[![Architecture](https://img.shields.io/badge/Architecture-TermAddIn%20Extension-orange.svg)]()
[![License](https://img.shields.io/badge/License-TermAddIn__Comm%20License-blue.svg)]()

## 📖 概要

**TermAddIn_Comm** は、Excel 総合通信アドイン **TermAddIn** 上で動作する **COM ポート・シリアル通信専用エクステンション（機能ドメイン単一 DLL）** です。

RS-232C、RS-422、RS-485、UART、および USB 仮想 COM ポート経由での各種電子計測器、組込みマイコン評価基板、バーコードリーダー、電子天秤、検査治具コントローラとの通信対話・コマンド自動検証・ログ収集を Excel ワークシート上から完全自動化します。

VBA マクロを一切記述することなく、 **標準の `.xlsx`（マクロ無効ブック）形式のままでコマンドの送受信、プロンプト自動待機、外部設定ファイルの一括流し込み、受信データのセル自動展開** が安全に実行できます。

> [!NOTE]
> **利用前提とライセンス体系**  
> 本エクステンションは [TermAddIn 本体](https://github.com/Monaca-App/TermAddIn) 上で動作します。  
> TermAddIn ではプロトコル毎に個別ライセンスを設ける体系を採用しており、本機能のご利用には **`TermAddIn_Comm` 個別ライセンス（一括 ¥32,000 / 月額 ¥3,000 / 週額 ¥1,000・税別）** が適用されます（ライセンス未配置・無効時は、Excel ブック再オープン時の既存ボタン無効化、および `Term.exe` 単体起動時の 3 分間期間限定版として動作します）。

---

## ✨ 主な機能・特徴

1. **多彩なシリアルインターフェースに完全対応**
   * オンボード COM ポート（RS-232C）はもちろん、各社 USB-シリアル変換アダプタ、仮想 COM ポート、RS-422 / RS-485 変換器に幅広く対応。
2. **柔軟な通信条件の動的設定 (`Com_Mode`)**
   * ボーレート（1200〜921600 bps 等）、パリティ（None/Odd/Even）、データ長（7/8 bit）、ストップビット（1/1.5/2 bit）、フロー制御（None/RTS-CTS/Xon-Xoff/Both）、送信終端デリミタ（`TXCR` / `TXCRLF` / `NON`）、およびデータモード（ASCII / バイナリ）をシーケンス実行中に動的切り替え可能。
3. **プロンプト自動待機 ＆ タイムアウト分岐 (`Com_WaitPrompt`)**
   * 相手機器から特定のプロンプト（例: `">"`, `"OK"`, `"login:"` 等）が返るまで非同期待機。指定タイムアウト（ミリ秒）到達時の自動ラベルジャンプ機能により、堅牢な対話シーケンスを実現。
4. **外部テキストファイルの一括送信 (`Com_SendFile`)**
   * 初期設定スクリプトや複数行のコマンドリストファイルを読み込み、行単位で順次一括送信。
5. **Excel 32bit / 64bit 完全動作マトリクス**
   * .NET Framework 標準のマネージド通信基盤（`System.IO.Ports`）を採用しており、外部の専用ネイティブ DLL に依存しないため、Excel 32bit 版および 64bit 版の双方で制限なく完全動作。
6. **実機・ドライバ不要の仮想 COM 折り返し ＆ 仮想機器テスト連携（`c2c` / `TermEmulator` 対応）**
   * 公式無償ツール **[c2c](https://github.com/Monaca-App/c2c)** によるドライバ不要の仮想 COM ポートペア（例: `COM10` ⇔ `COM11`）や、 **[TermEmulator](https://github.com/Monaca-App/TermEmulator)** のドライバレス仮想 COM 計測器プリセット（SCPI・電子天秤・バーコードリーダ等）と直結し、送受信電文をリアルタイム監視・CSV 保存しながら事前開発を行えます。

---

## 🔌 提供コマンド一覧（全 6 コマンド）

各コマンドの詳細な構文・パラメータ仕様・記述例は、以下の個別リファレンスをご参照ください。

| コマンド名 | 構文概要 | 主な役割・用途 | 詳細マニュアル |
| :--- | :--- | :--- | :---: |
| **`Com_Open`** | `Com_Open` | 設定された COM ポートをオープンし非同期受信監視を開始 | [詳細説明](./Manual/Comm/Com_Open.Md) |
| **`Com_Close`** | `Com_Close` | オープン中の COM ポートを安全に切断・リソース解放 | [詳細説明](./Manual/Comm/Com_Close.Md) |
| **`Com_Mode`** | `Com_Mode, "[設定値]", "[フロー]", "[デリミタ]", "[モード]"` | 通信条件（ボーレート・パリティ等）や送受信モードの動的変更 | [詳細説明](./Manual/Comm/Com_Mode.Md) |
| **`Com_Send`** | `Com_Send, "[送信文字列/ファイルパス]", "[モード]", "[改行/エラー格納先]"` | コマンド電文または外部ファイルの送信 | [詳細説明](./Manual/Comm/Com_Send.Md) |
| **`Com_SendFile`** | `Com_SendFile, "<ファイルパス>", "[ダイアログ表示有無]"` | 外部テキストファイルを読み込み行単位で順次送信 | [詳細説明](./Manual/Comm/Com_SendFile.Md) |
| **`Com_WaitPrompt`** | `Com_WaitPrompt, "[プロンプト]", "[タイムアウトms]", "[タイムアウト時ジャンプ先]"` | 応答プロンプト文字列の受信待機および分岐処理 | [詳細説明](./Manual/Comm/Com_WaitPrompt.Md) |

---

## 💻 動作環境・システム要件

| 項目 | システム要件 |
| :--- | :--- |
| **OS** | Windows 10 / Windows 11 (64bit / 32bit) |
| **対応 Excel** | Microsoft Excel 2016 / 2019 / 2021 / Microsoft 365 (デスクトップ版 32bit / 64bit) |
| **必須アドイン** | [TermAddIn](https://github.com/Monaca-App/TermAddIn) (Excel VSTO アドイン) |
| **ランタイム** | .NET Framework 4.8 / Visual Studio 2010 Tools for Office Runtime |
| **インターフェース** | RS-232C, RS-422, RS-485, USB 仮想 COM ポート, [c2c](https://github.com/Monaca-App/c2c) 仮想ヌルモデムポート |

---

## 🚀 インストール手順

<a href="https://github.com/Monaca-App/TermAddIn_Comm/raw/main/Setup/TermAddIn_Comm_Setup.zip">
  <img src="images/Monaca_app_get.jpg" width="140" alt="プログラム（入手）" />
</a>

> 上記の **「プログラム（入手）」アイコン** をクリックすると、セットアッププログラム一式（[`TermAddIn_Comm_Setup.zip`](https://github.com/Monaca-App/TermAddIn_Comm/raw/main/Setup/TermAddIn_Comm_Setup.zip)）が直接ダウンロードされます。展開後、`TermAnalysis.Comm\setup.exe`（または `TermAnalysis.Comm.Setup.msi`）を実行してください。

> [!IMPORTANT]
> **導入前提**  
> 本エクステンションを利用するには、事前に **[TermAddIn 本体](https://github.com/Monaca-App/TermAddIn)** （`Term.Setup` および `TermAddIn.Setup`）をインストールしておく必要があります。

### 配布アーカイブ（本リポジトリ `Setup/` 配下）

* **アーカイブ**: [`Setup/TermAddIn_Comm_Setup.zip`](https://github.com/Monaca-App/TermAddIn_Comm/raw/main/Setup/TermAddIn_Comm_Setup.zip)（収録内容: `TermAnalysis.Comm/setup.exe`, `TermAnalysis.Comm/TermAnalysis.Comm.Setup.msi`）

### 導入ステップ

1. **Microsoft Excel** および **Term** をすべて終了します。
2. ダウンロードした **`TermAddIn_Comm_Setup.zip`** を展開し、 **`TermAnalysis.Comm/`** フォルダ内の **`setup.exe`** （または **`TermAnalysis.Comm.Setup.msi`** ）を実行します。
   * ※ 管理者権限（UAC 承認）が必要です。
   * ※ 拡張 DLL は共通インストール先（`C:\Program Files (x86)\Monacaアプリ工房\Monaca Common\`）へ配置され、サンプル資材が `C:\Program Files (x86)\Monacaアプリ工房\Monaca Sample\Comm\` へ導入されます。
3. インストール完了後、Excel（または `Term.exe`）を起動し、メイン画面の **`[通信モード]`** メニューから **`RS-232C`** が選択できることを確認します。

### アンインストール手順
* Windows の「設定」>「アプリ」>「インストールされているアプリ」（または「コントロールパネル」の「プログラムと機能」）から **`TermAnalysis.Comm.Setup`** を選択してアンインストールしてください。

---

## 🔰 シーケンス記述例・同梱サンプル資材（`Sample/Comm/`）

Excel シート上での典型的なシリアル通信・対話シーケンスの構成例です。

```csv
; 1. COMポートをオープン
,SEQUENCE, 010, Com_Open
; 2. バージョン確認コマンドを送信し、プロンプト ">" を最大 5000ms 待機（タイムアウト時は #ERROR# へ）
,SEQUENCE, 020, Com_Send, "VER?"
,SEQUENCE, 030, Com_WaitPrompt, ">", 5000, #ERROR#
; 3. 受信バッファから応答テキストを取得
,SEQUENCE, 040, Set_Data, "FW_VER"
; 4. ポートをクローズして正常終了
,SEQUENCE, 050, Com_Close
,SEQUENCE, 060, Exit
; エラー処理ブロック
,SEQUENCE, #ERROR#, Msg_Box, "シリアル通信がタイムアウトしました。"
,SEQUENCE, 070, Com_Close
,SEQUENCE, 080, Exit
```

### 同梱サンプルマクロ・AI自動生成プロンプト集
COM ポートシリアル通信の全 6 コマンド（通信条件設定・プロンプト同期・エコー除去測定値抽出・任意コマンド送受信・外部設定ファイル一括送信・切断）に対応した動作確認用マクロスクリプト（CSV）および AI 自動生成プロンプトが **`Sample/Comm/`** に収録されています（インストール時には `C:\Program Files (x86)\Monacaアプリ工房\Monaca Sample\Comm\` へ配置され、デスクトップの `Monaca\Sample` ショートカットから参照できます）。

* **[`Sample/Comm/Comm.csv`](./Sample/Comm/Comm.csv)**: COM ポート シリアル対話通信制御 汎用マクロスクリプトサンプル
* **[`Sample/Comm/Comm_prompt.md`](./Sample/Comm/Comm_prompt.md)**: COM ポート シリアル通信用マクロスクリプト AI 自動生成プロンプト

---

## 💳 ライセンス・購入プラン（TermAddIn_Comm）

> [!NOTE]
> **試用動作およびオンライン決済について**  
> ライセンス未配置・無効時は、Excel ブック再オープン時の既存ボタン無効化（新規追加ボタンのみ試用可能）、および `Term.exe` 単体起動時の 3 分間期間限定版として動作します。  
> 用途や利用期間に合わせて、一括購入（買い切り）またはサブスクリプション（月額・週額）からお選びいただけます。メールアドレスの誤入力によるライセンス不着を防ぐため、 **事前のメールアドレス確認** を行った上で、 **Stripe** による安全なオンライン決済へご案内いたします（表記価格はすべて税別です）。

| プラン | 価格（税別） | お支払い・お申込み |
| :--- | :---: | :---: |
| **一括購入（買い切りライセンス）**<br>無期限で永続利用できるライセンスです。 | **¥32,000** | [👉 一括購入手続きへ](https://stripe-license-generator.y-nakai-5a3.workers.dev/?target=comm) |
| **月額サブスクリプション**<br>月単位で利用できる定期プランです。中長期のプロジェクトにおすすめです。 | **¥3,000** / 月 | [👉 月額プランに申し込む](https://stripe-license-generator.y-nakai-5a3.workers.dev/?target=comm) |
| **週額サブスクリプション**<br>週単位で手軽に利用できるプランです。短期評価や単発テストにおすすめです。 | **¥1,000** / 週 | [👉 週額プランに申し込む](https://stripe-license-generator.y-nakai-5a3.workers.dev/?target=comm) |

* ※ **ライセンスファイル名**: `TermAddIn_Comm.License`
* ※ **配置先**: `%USERPROFILE%\Desktop\Monaca\TermAddIn_Comm.License`（設定ファイル `Setting.xml` と同階層）
* ※ **ライセンス形態**: 1 ユーザー / 1 PC（ノードロック方式：`PC名` および `ログインユーザー名` に紐づけ）

---

## 📄 ライセンス & サポート

* **開発元**: [Monaca アプリ工房 (Monaca App Store)](https://github.com/Monaca-App/store)
* **親プロジェクト**: [TermAddIn](https://github.com/Monaca-App/TermAddIn)
* **関連コンパニオンツール**: [c2c - ドライバ不要の仮想ヌルモデム＆通信モニター](https://github.com/Monaca-App/c2c)
* **Copyright**: Copyright (C) Monaca App Store 2026 All rights reserved.
* **お問い合わせ**: 不具合報告や機能要望は、GitHub の Issue までお気軽にお寄せください。
