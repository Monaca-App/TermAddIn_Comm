# コマンドリファレンス：Com_Open

## 概要

**Com_Open** は、COM ポート（UART / シリアル通信ポート）をオープンし、非同期受信監視スレッドおよび受信リングバッファ（`TermRingBufferManager`）の稼働を開始する通信制御系コマンドです。引数を省略した場合はプロパティ設定画面の環境設定値を自動採用し、引数を指定した場合は個別通信条件（`PortName`〜`StopBits`）でオーバーライド可能です。
実行時に Term 通信制御の排他選択モードを **COM（UART）モード** （`TermAnalysis.Comm.dll`）へ確定させます。

> 💡 **補足** : 引数をすべて省略した場合はプロパティ設定画面（環境設定）の COM ポート番号・通信条件を自動採用し、第 1 引数〜第 5 引数を指定した場合はスクリプト側から個別通信条件（`PortName`〜`StopBits`）でオーバーライドしてオープンできます。物理 COM ポート（RS-232C / USB-シリアル変換器）に加え、仮想シリアル中継ツール **[c2c](https://github.com/Monaca-App/c2c)** が公開中の仮想 COM ポート（例: `COM10` ⇔ `COM11`）も透過的にオープン可能です。

---

## 💡 書式・パラメータ仕様

### コマンド構文

```csv
,SEQUENCE, [ステップキー], Com_Open, "[PortName]", "[BaudRate: 9600]", "[Parity: None]", "[DataBits: 8]", "[StopBits: 1]"
```

※ 全引数を省略した場合は、UI プロパティ画面または `Com_Mode` で設定されている現在の通信条件を使用して COM ポートをオープンします。

### パラメータ（引数）一覧

| 引数位置 | 引数名 | 必須/任意 | 説明・制約 |
| :--- | :--- | :---: | :--- |
| 第 1 引数 | `PortName` | 任意 | オープンする COM ポート名（例: `"COM3"`、省略または空文字 `""` 指定時は UI `CommSettings.CommPort` または `UartTransportCore.Shared.PortName` の現在値を使用）。 |
| 第 2 引数 | `BaudRate` | 任意 | 通信ボーレート［bps］（`300` 以上の整数。引数省略時は `9600` へリセットせず `UartTransportCore.Shared.BaudRate`／UI 設定の現在値を維持。第 2 引数を空文字 `""` で明示渡した場合のみ `9600`）。 |
| 第 3 引数 | `Parity` | 任意 | パリティ種別（`"None"` / `"Even"` / `"Odd"` / `"Mark"` / `"Space"` および 1 文字略記 `"N"` / `"E"` / `"O"` / `"M"` / `"S"`。引数省略または空文字 `""` 時は `UartTransportCore.Shared.Parity` の現在値を維持。`ParameterSpec` 宣言既定値: `"None"`）。 |
| 第 4 引数 | `DataBits` | 任意 | データビット長（`5`〜`8`。引数省略時は `8` へリセットせず `UartTransportCore.Shared.DataBits`／UI 設定の現在値を維持。第 4 引数を空文字 `""` で明示渡した場合のみ `8`）。 |
| 第 5 引数 | `StopBits` | 任意 | ストップビット長（`"1"` / `"1.5"` / `"2"`。引数省略または空文字 `""` 時は `UartTransportCore.Shared.StopBits` の現在値を維持。`ParameterSpec` 宣言既定値: `"1"`）。 |

---

## 🔄 特殊置換規約・分岐規約の適用

本コマンドの引数における特殊記法および解釈ルールです。
* **動的置換（`$`）** : 引数内で `$PARAMキー$` のように記述すると、実行時に変数プール（PARAM）の値へ動的置換されます。（適用可能な引数: 全引数対応）
* **ジャンプ先指定（`#`）** : 本コマンドはジャンプ制御を行わないため適用されません。（適用可能な引数: 非適用）
* **共通コマンド指定（`@`）** : 本コマンドは外部マクロを呼び出さないため、共通コマンド呼び出し記法は適用されません。（適用可能な引数: 非適用）
* **カンマのエスケープ** : 通信条件文字列（例: `"9600,E,7,1"`）や複数データのカンマ区切り値を単一引数として渡す場合は、必ず `"..."` と二重引用符で囲んで記述します。
* **エイリアス（別名）** : `Open`, `Term_Open`, `Analysis_Term_Open`

---

## 💡 動作例（正しい構文での記述例）

### スクリプト記述例

※COMMAND ブロック配下の子要素行（PARAM, SEQUENCE）は必ず行頭カンマから開始し、スペースインデントや行末コメント（`;`）は記述しないでください。

```csv
COMMAND, "SAMPLE_COM_OPEN", ""
; 1. 通信条件・デリミタ・データモードを事前設定してからCOMポートをオープン
,SEQUENCE, 010, Com_Mode, "9600,N,8,1", "NONE", "TXCRLF", "ASCII"
,SEQUENCE, 020, Com_Open
; 2. 機器の初期化コマンド送信とプロンプト待機
,SEQUENCE, 030, Com_Send, "INIT"
,SEQUENCE, 040, Com_WaitPrompt, "READY>", "5000", #ERROR_TIMEOUT#
,SEQUENCE, 050, Com_Close
,SEQUENCE, 060, Exit
; タイムアウト時の異常系クローズ処理
,SEQUENCE, #ERROR_TIMEOUT#, Msg_Box, "初期化応答タイムアウトが発生しました"
,SEQUENCE, 070, Com_Close
,SEQUENCE, 080, Exit
```

### 実行時の挙動・変数変化

* **実行前の状態** : Term 通信エンジンが初期化され、必要な通信パラメータおよび変数プール（`PARAM`）が準備されている状態。
* **実行後の状態** : `Com_Mode` によりボーレート `9600`、パリティ `None`、データビット `8`、ストップビット `1`、フロー制御なし、送信改行 `CR+LF`、ASCII モードが設定されます。 `Com_Open` により既存の通信セッションがある場合は自動クローズされた上で COM ポートが開かれ、受信リングバッファ（`TermRingBufferManager`）のクリア・通信エラーフラグ（`"CLR"`）の初期化・バックグラウンド非同期受信購読が開始されます。

---

## 🔄 動作フロー（シーケンス図）

```mermaid
sequenceDiagram
    autonumber
    participant Script as "スクリプト実行エンジン (TermSequenceDriver / AnalysisEngine)"
    participant Cmd as "Com_OpenCommand (TermAnalysis.Comm)"
    participant Buf as "TermRingBufferManager (排他セッション管理)"
    participant Port as "UartTransportCore / TermForm"

    Script->>Cmd: Func(parent, param)
    Cmd->>Cmd: ライセンス検証・ValidateParamRange(0, 5)・TryGetInt 引数検証
    alt ライセンス未認証 (-100) または引数異常 (個数超過 -10 / 非整数 -11 / 範囲外 -12)
        Cmd->>Cmd: エラーログ出力 (_log.LogError)
        Cmd-->>Script: OpenExclusiveSession 前に早期停止 (SetErrorAndHalt(-100 / -10 / -11 / -12, errMsg))
    else 引数・ライセンス正常
        Cmd->>Port: 指定された引数のみ UartTransportCore.Shared / CommSettings へ上書き反映 (省略引数は現在値を維持)
        Cmd->>Buf: OpenExclusiveSession(TermTransportDomain.Com, parent, useSocket: false)
        opt 既存セッション (COM または Telnet/Modbus 等) が稼働中の場合
            Buf->>Buf: CloseSessionInternal() により既存接続を安全に自動クローズ
        end
        Buf->>Buf: ClearBuffer() / ClearCommunicationError() ("CLR" 初期化)
        Buf->>Port: UartCore.Open() / TryOpenPort()
        alt ポートオープン成功
            Port-->>Buf: DataReceived 非同期イベント購読開始 (true)
            Buf-->>Cmd: ActiveDomain = Com 確立 (true)
            Cmd-->>Script: 正常終了 (IsCommandGo = true)
        else ポート不在・他プロセス占有等によるオープン失敗
            Port-->>Buf: オープン失敗 (false)
            Buf-->>Cmd: false 返却
            Cmd->>Cmd: エラーログ出力 (_log.LogError)
            Cmd-->>Script: 安全停止 (SetErrorAndHalt(-1, errMsg))
        end
    end
```

---

## 🚫 エラーハンドリング・安全設計（仕様制限）

1. **既存セッションおよび他通信ドメインからの安全な自動切替（`OpenExclusiveSession`）と省略引数の現在値維持** :
   - 第 2〜第 5 引数を省略した場合は既定値へ強制リセットせず、`UartTransportCore.Shared` や UI プロパティ設定（または直前の `Com_Mode`）の現在設定値をそのまま維持してオープンします。また、すでに同一 COM ポートまたは他ドメイン（`Telnet_Open` / `Modbus_Open` 等）のセッションが稼働中の場合は、既存セッションを内部で安全に自動クローズしてから新しい COM セッションへ排他切り替えし、受信リングバッファと通信エラー状態を初期化します。
2. **引数異常時の早期検証停止とポート不在・アクセス拒否時の安全停止** :
   - 引数個数が上限を超過した場合（6 個以上：エラーコード `-10`）、`BaudRate` や `DataBits` に非整数文字列を指定した場合（`-11`）、または許容範囲外（`BaudRate < 300`、`DataBits < 5` または `> 8`：`-12`）を指定した場合は、`OpenExclusiveSession` を呼び出す前に即座にエラー停止して既存セッションの誤切断を防ぎます。また、指定された COM ポートが存在しない場合や他アプリケーションが使用中の場合は、エラーログを出力し `SetErrorAndHalt(-1, errMsg)` により安全に停止します。
3. **エクステンションライセンス検証** :
   - `Comm` エクステンションライセンスが未認証の環境で実行された場合は、最優先でエラーコード `-100` を設定してシーケンスを安全に停止します。

---

## 🧪 ユニットテスト仕様

本コマンドクラス（`Com_OpenCommand.cs`）に実装されている `RunUnitTest()` メソッドが検証するユニットテストの仕様です。

### テスト実行前提条件

* **実行トリガー** : `RunUnitTest()` メソッドの呼び出し
* **有効化条件** : システム全体の最小ログレベル（`ErrorLogger.MinimumLevel`）が `LogLevel.Test` であること（それ以外の場合は無効化ガードにより早期リターン）
* **検証結果出力** : `_log.LogTest` によるログ出力。検証失敗時は `_log.LogError`、予期せぬ例外発生時は `_log.LogCritical` で記録し、`finally` ブロックで `TermRingBufferManager.Instance.CloseSession(TermTransportDomain.Com, null)` および `UartTransportCore.Shared.IsSimulationMode` の復元（UI タブ検証時は `TabControl?.Dispose()` と一時設定 `"TermUnitTest_TempComOpen"` の削除）を実行します。

### テストケース一覧

| No | テストカテゴリ | テスト条件（入力値） | 期待される結果（出力値） | 検証の目的 / 観点 |
| :---: | :--- | :--- | :--- | :--- |
| **1** | 正常系 | `IsSimulationMode = true` 環境下で引数 `"COM3"`, `"115200"`, `"Even"`, `"8"`, `"1"` を指定して実行 | `!param.HasError` かつ `IsOpen == true`、`ActiveDomain == TermTransportDomain.Com`、`PortName == "COM3"`、`BaudRate == 115200`、`Parity == Parity.Even` となる | 個別通信条件オーバーライドおよび COM 排他セッション確立の検証 |
| **2** | 異常系 | セッション切断後、`IsSimulationMode = false` 環境下で存在しない物理ポート `"COM254"`, `"9600"` を指定して実行 | `failParam.HasError == true`、`!failParam.IsCommandGo`、`!IsOpen`、`ActiveDomain == TermTransportDomain.None` となり、`LastErrorMessage` にエラー内容が格納される | 存在しない物理 COM ポート指定時のオープン失敗検出と安全停止の検証 |
| **3** | 正常系 | `TermPropertiesCom`（一時設定 `"TermUnitTest_TempComOpen"`）で `"COM1"` 表示後、未接続ポート `"COM254"` と `"115200,N,8,1"` を設定 | `comTab.CommPort` が `"COM1"` へ巻き戻らず `"COM254"` のまま保持される | UI プロパティタブにおける未接続ポート名保持動作の検証 |
