# コマンドリファレンス：Com_Close

## 概要

**Com_Close** は、現在オープンされている COM ポート（UART 回線）の非同期受信監視スレッドを安全に停止し、シリアルポートハンドルを解放・切断するコマンドです。

> 💡 **補足** : シーケンス終了時や通信テスト完了時に確実にポートを解放するために使用します。

---

## 💡 書式・パラメータ仕様

### コマンド構文

```csv
,SEQUENCE, [ステップキー], Com_Close
```

### パラメータ（引数）一覧

| 引数位置 | パラメータ名 | 必須/任意 | 型・制約 | 説明 |
| :--- | :--- | :---: | :--- | :--- |
| （引数なし） | - | - | - | 現在接続中の COM ポートをクローズします。 |

---

## 🔄 特殊置換規約・分岐規約の適用

本コマンドの引数における特殊記法および解釈ルールです。
* **動的置換（`$`）** : 本コマンドは引数を取らないため、引数に対する `$PARAMキー$` 動的置換は適用されません。（適用可能な引数: 非適用）
* **ジャンプ先指定（`#`）** : 本コマンドはジャンプ制御を行わないため適用されません。（適用可能な引数: 非適用）
* **共通コマンド指定（`@`）** : 本コマンドは外部マクロを呼び出さないため、共通コマンド呼び出し記法は適用されません。（適用可能な引数: 非適用）
* **カンマのエスケープ** : 通信条件文字列（例: `"9600,E,7,1"`）や複数データのカンマ区切り値を単一引数として渡す場合は、必ず `"..."` と二重引用符で囲んで記述します。
* **エイリアス（別名）** : `Close`, `Term_Close`, `Analysis_Term_Close`

---

## 💡 動作例（正しい構文での記述例）

### スクリプト記述例

※COMMAND ブロック配下の子要素行（PARAM, SEQUENCE）は必ず行頭カンマから開始し、スペースインデントや行末コメント（`;`）は記述しないでください。

```csv
COMMAND, "SAMPLE_COM_CLOSE", ""
,PARAM, "RECV_LINE", Work, "受信応答", ""
; 1. COMポートを開いてステータスコマンドを送信
,SEQUENCE, 010, Com_Open
,SEQUENCE, 020, Com_Send, "STATUS?"
,SEQUENCE, 030, Com_WaitPrompt, "OK", "3000"
; 2. 受信バッファの応答をWork変数へ格納した後にCOMポートをクローズ
,SEQUENCE, 040, Set_Data, "RECV_LINE"
,SEQUENCE, 050, Com_Close
,SEQUENCE, 060, Exit
```

### 実行時の挙動・変数変化

* **実行前の状態** : Term 通信エンジンが初期化され、必要な通信パラメータおよび変数プール（`PARAM`）が準備されている状態。
* **実行後の状態** : `Set_Data`（第 2 引数省略）により受信バッファ（`TermRingBufferManager`）内の応答文字列が変数 `RECV_LINE` に取り込まれます。 `Com_Close` の実行により `TermRingBufferManager.CloseSession` を通じて UART 通信回線（`UartTransportCore.Close()`）がクローズされ、アクティブドメインが `None` に解放されます。

---

## 🔄 動作フロー（シーケンス図）

```mermaid
sequenceDiagram
    autonumber
    participant Script as "スクリプト実行エンジン (TermSequenceDriver / AnalysisEngine)"
    participant Cmd as "Com_CloseCommand (TermAnalysis.Comm)"
    participant Buf as "TermRingBufferManager (排他セッション管理)"
    participant Port as "UartTransportCore / TermForm"

    Script->>Cmd: Func(parent, param)
    Cmd->>Cmd: ValidateParamRange(0, 0) 引数個数検証
    alt 引数個数が正常 (0個) の場合
        Cmd->>Buf: CloseSession(TermTransportDomain.Com, parent)
        Buf->>Port: UartCore.Close() / ExecuteClosePort()
        Port-->>Buf: シリアルポート切断完了
        Buf-->>Cmd: ActiveDomain = None 更新完了
        Cmd-->>Script: 正常終了 (IsCommandGo = true)
    else 余剰引数指定 (-10) または切断例外発生 (-1) 時
        Cmd->>Cmd: エラーログ出力 (_log.LogError / _log.LogCritical)
        Cmd-->>Script: 安全停止 (SetErrorAndHalt(-10 / -1, ...))
    end
```

---

## 🚫 エラーハンドリング・安全設計（仕様制限）

1. **余剰引数指定時の境界検証と安全停止** :
   - 本コマンドは引数を取らない仕様（`ValidateParamRange(0, 0)`）であり、誤って 1 個以上の引数が指定された場合はエラーログを出力し、エラーコード `-10`（`SetErrorAndHalt(-10, errMsg)`、`IsCommandGo = false`）として安全に実行を停止します。
2. **切断例外発生時のフェイルセーフと未接続時 No-Op 保証** :
   - ポートが未接続の状態で実行した場合はエラーとせず安全に次ステップへ進みます（受信リングバッファ内の既存データも消去されず保持されます）。万一 OS ドライバ側で予期せぬ切断例外が発生した場合は `_log.LogCritical` を記録し、`SetErrorAndHalt(-1, ex.Message)` によりクラッシュを防いで安全に停止します。

---

## 🧪 ユニットテスト仕様

本コマンドクラス（`Com_CloseCommand.cs`）に実装されている `RunUnitTest()` メソッドが検証するユニットテストの仕様です。

### テスト実行前提条件

* **実行トリガー** : `RunUnitTest()` メソッドの呼び出し
* **有効化条件** : システム全体の最小ログレベル（`ErrorLogger.MinimumLevel`）が `LogLevel.Test` であること（それ以外の場合は無効化ガードにより早期リターン）
* **検証結果出力** : `_log.LogTest` によるログ出力。検証失敗時は `_log.LogError`、予期せぬ例外発生時は `_log.LogCritical` で記録し、`finally` ブロックで `UartTransportCore.Shared.IsSimulationMode` をテスト開始前の状態へ復元します。

### テストケース一覧

| No | テストカテゴリ | テスト条件（入力値） | 期待される結果（出力値） | 検証の目的 / 観点 |
| :---: | :--- | :--- | :--- | :--- |
| **1** | 正常系 | `IsSimulationMode = true` 環境下で `OpenExclusiveSession(TermTransportDomain.Com, null, useSocket: false)` 実行後、引数なしで `Func(null, param)` を実行 | `!param.HasError` かつ `!UartTransportCore.Shared.IsOpen`、`ActiveDomain == TermTransportDomain.None` となる | COM 排他セッションの正常切断およびポート解放の検証 |
| **2** | 異常系 | 余剰引数 1 個（`"EXTRA_INVALID"`）を指定して `Func(null, invalidParam)` を実行 | `invalidParam.HasError == true` かつ `!invalidParam.IsCommandGo` となり安全に拒否・停止する | 引数なし制約（`ValidateParamRange(0, 0)`）違反時の境界防御検証 |
