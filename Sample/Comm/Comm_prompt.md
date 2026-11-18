# COMポート シリアル対話通信制御用 マクロスクリプト自動生成 AI指示書 (Comm.csv 生成用)

> **生成AI**: Antigravity (Gemini 4 Carbon (Medium))

あなたは、通信制御システム（Term / TermAddIn）向けマクロスクリプト（CSV形式）を設計・出力する専門のエンジニアリングAIです。  
以下の「参照ドキュメント」および「システム絶対制約」「COMシリアル機能別パラメータ設計仕様」を厳格に遵守し、RS-232C / RS-422 / RS-485 / USB仮想COMポート接続機器との **対話型シリアル通信・測定値収集・一括設定送信** を自動実行するための Term マクロスクリプト（`Comm.csv`）を作成してください。

---

## 📚 1. 参照ドキュメントおよび準拠規格

1. **システム文法・構文規約（最優先遵守）**
   * `Manual/TermAddIn/マクロスクリプト取扱説明書.Md`（Term マクロスクリプト取扱説明書 兼 AI自動生成指示書）
2. **通信・プロトコル個別仕様（Comm 専用コマンドセットおよび共通基盤）**
   * `Manual/TermAddIn/Comm/Com_Mode.Md`（UART 通信条件・フロー制御・送信デリミタ・受信データモード設定）
   * `Manual/TermAddIn/Comm/Com_Open.Md` / `Manual/TermAddIn/Comm/Com_Close.Md`（COM ポート接続・切断）
   * `Manual/TermAddIn/Comm/Com_Send.Md`（文字列電文送信・改行指定・エラー確認付き送信）
   * `Manual/TermAddIn/Comm/Com_SendFile.Md`（外部テキストファイル一括送信）
   * `Manual/TermAddIn/Comm/Com_WaitPrompt.Md`（応答プロンプト非同期待機・タイムアウト分岐）
   * `Manual/Core/Work/Time_Stamp.Md` / `Manual/Core/Work/Skip_Line.Md` / `Manual/Core/Work/Get_Line.Md` / `Manual/Core/Work/Set_Data.Md`（タイムスタンプ制御・エコー行スキップ・行抽出・受信バッファ吸出）
   * `Manual/Core/Sequence/Judge.Md` / `Manual/Core/Sequence/Jump.Md` / `Manual/Core/Param/Exit.Md`（比較判定・ジャンプ・終了制御）
3. **対象通信仕様（実機エンジン `TermAnalysis.Comm` ランタイム引数規約）**
   * 制御対象: シリアル対話型機器（電子計測器、組込みマイコン評価基板、バーコードリーダー、電子天秤、検査治具コントローラ、または `c2c` 仮想ヌルモデムポートペア）
   * 通信条件設定:
     * `Com_Mode, "[Baud,Parity,DataBits,StopBits]", "[Handshake]", "[TxDelimiter]", "[DataMode]"`（既定: `"9600,N,8,1"`, `"NONE"`, `"TXCRLF"`, `"ASCII"`）
     * `Com_Open, "[PortName]"`（`PortName` が空文字 `""` の場合はプロパティ画面の COM ポート設定を使用、`"COM10"` 等の指定時はシート値で動的オーバーライド）
   * 送受信・プロンプト待機規約:
     * `Com_Send, "[Message]", "[LineEnding]"`（`LineEnding` に `"CRLF"`, `"CR"`, `"LF"`, `"NONE"` 指定時は送信改行を個別指定、省略時は `Com_Mode` の送信デリミタを自動付与）
     * `Com_WaitPrompt, "[Prompt]", "[TimeoutMs]", [JumpIfTimeout]`（指定プロンプト到着まで非同期監視し、タイムアウト時は `#ラベル#` へ分岐）
     * `Com_SendFile, "[FilePath]", "[WaitPromptMode]", "[ClearError]"`（`"WaitPrompt"` または `"NoWait"` モードで外部設定ファイルを逐次送信）

---

## 🚨 2. システム絶対制約（構文エラー・実行時クラッシュ防止の鉄則）

1. **表記言語・文字コード・AI生成元の明記**
   * **表記言語**: 多言語化を意識した構成とし、スクリプト内のコメント、パラメータ表示名、メッセージ等は、特に指定がない限り **日本語をデフォルトとして記述・設定** してください。
   * **文字コード**: 生成する CSV ファイル（`Comm.csv`）は、日本語 Excel での文字化け防止のため **Shift-JIS (CP932)** で保存してください。
   * **AI生成元の明記（他者再現性の保証）**: CSV ファイルの最先頭ヘッダコメント行（`;`）に、対象システム情報とともに **他者が同一条件で再現できるよう、実行環境（エージェント／IDE名）および自身のAIエンジン（モデル名・エディション／ティア）を正確に判別できる名称（例: `; 生成AI: Antigravity (Gemini 4 Carbon (Medium))`）** を必ず記述してください（エージェント名のみの記載や、推測・例示の丸写しによる不正確なモデル名の記載は厳禁）。
   * **パス表記**: ファイル内に開発環境固有のローカル絶対パスや個人名を含めることは厳禁です。必ず相対パスまたは抽象表記のみで記述してください。

2. **シリアル通信条件および動作モードの完全機能別パラメータ分離と `List` 選択化（最重要仕様）**
   * 通信プロパティ（`"9600,N,8,1"`）を一括りの固定リテラルとせず、かつ `List` 選択肢内への生カンマ混入（フィールド分裂バグ）を防ぐため、各通信要素を完全に分離して `PARAM` 定義してください。また、 **ボーレート・パリティ・データ長・ストップビット・フロー制御・改行コード・データモード・待機モードなど、取り得る文字列が決まっている項目は必ず `List` 型で定義** し、Excel シートのドロップダウンから選択できるようにしてください。
     * **COMポート指定 (`PORT_NAME`)**: ポート番号または空欄で UI 設定継承（`Text`, `""`）
     * **ボーレート (`BAUD`)**: 通信速度（`List`, 初期値 `"9600"`, 追加選択肢 `"19200"`, `"38400"`, `"57600"`, `"115200"`）
     * **パリティ (`PARITY`)**: パリティ検査方式（`List`, 初期値 `"N"`, 追加選択肢 `"E"`, `"O"`）
     * **データ長 (`DATA_BITS`)**: データビット数（`List`, 初期値 `"8"`, 追加選択肢 `"7"`）
     * **ストップビット (`STOP_BITS`)**: ストップビット数（`List`, 初期値 `"1"`, 追加選択肢 `"2"`）
     * **フロー制御 (`FLOW_CTRL`)**: ハンドシェイク方式（`List`, 初期値 `"NONE"`, 追加選択肢 `"RTSCTS"`, `"XONXOFF"`）
     * **送信改行コード (`TX_DELIM`)**: 送信終端デリミタ（`List`, 初期値 `"TXCRLF"`, 追加選択肢 `"TXCR"`, `"NON"`）
     * **受信データモード (`DATA_MODE`)**: 受信バッファ解釈モード（`List`, 初期値 `"ASCII"`, 追加選択肢 `"BIN"`）
   * `Com_Mode` の第 1 引数は `"$BAUD$,$PARITY$,$DATA_BITS$,$STOP_BITS$"` のように二重引用符で囲んで動的置換連結してください。

3. **行頭判定と「第2フィールド」の鉄則**
   * **親ブロック（`TITLE`, `COMMAND`）**: 必ず行頭の1文字目（第1フィールド）から記述してください。
   * **子要素（`PARAM`, `SEQUENCE`）**: 必ず **行頭をカンマ `,`（第1フィールドを空）** から開始し、第2フィールドにキーワードを記述してください。スペースやタブによるインデントは一切禁止です。

4. **PARAM 定義における命名規則および「表示名」「初期値」の省略完全禁止**
   * 書式: `,PARAM, [パラメータキー], [データ型], [表示名], [初期値], [追加選択肢...]`
   * データ型は `Text`, `List`, `Range`, `Value`（または `Number`）, `Label`, `Work` の6種類のみ使用可能です。
   * **いかなるデータ型であっても、第5フィールド（表示名）および第6フィールド（初期値）の省略は絶対禁止** です（4フィールド以下で記述すると `IndexOutOfRangeException` が発生します）。初期値が空の場合は必ず `""` を明記してください。

5. **COMMAND ブロック配下での行末コメント禁止**
   * `COMMAND` ブロック配下の `PARAM` 行および `SEQUENCE` 行では **行末コメント（`; ...`）は完全禁止** です。コメントは必ず独立した行（行頭 `;`）として記述してください。

6. **Comm 専用コマンドセットの厳守（コマンドセット横断の完全禁止）**
   * 本スクリプトは `TermAnalysis.Comm` 用であるため、通信制御には必ず **Comm 用コマンドセット（`Com_Mode`, `Com_Open`, `Com_Send`, `Com_SendFile`, `Com_WaitPrompt`, `Com_Close`）** および共通基盤コマンド（`Time_Stamp`, `Skip_Line`, `Get_Line`, `Set_Data`, `Judge`, `Jump`, `Exit`）のみを使用してください。
   * 他通信ドメインのコマンドセット（`Telnet_*`, `Modbus_*`, `Melsec_*`, `Mc_*` 等）を混在させることは排他制御違反となるため **絶対禁止** です。

---

## 📡 3. 対象システム仕様および TITLE ブロック定義

シリアル対話通信制御システム向けとして、`TITLE` ブロックを以下の通り定義してください。

* **タイトル名**: `"COMポート シリアル対話通信制御"`
* **書式**:
  ```csv
  TITLE, "COMポート シリアル対話通信制御"
  ```

---

## 📐 4. 実装すべき COMMAND ブロックと完全機能別パラメータ設計仕様

以下の 6 つの独立した `COMMAND` ブロックを実装してください（`TermAnalysis.Comm` の全 6 コマンドを網羅）。

### ① 通信条件設定・COMポート接続：`COMMAND, "PORT_OPEN"`
* **目的**: UART 通信条件（ボーレート・パリティ・データ長・ストップビット・フロー制御・送信改行・データモード）を `List` パラメータから一括設定し、COM ポートを排他オープンする。
* **パラメータ定義**:
  * `PORT_NAME` (`Text`, `"COMポート(空欄時はUI設定)"`, `""`)
  * `BAUD` (`List`, `"ボーレート"`, `"9600"`, `"19200"`, `"38400"`, `"57600"`, `"115200"`)
  * `PARITY` (`List`, `"パリティ"`, `"N"`, `"E"`, `"O"`)
  * `DATA_BITS` (`List`, `"データ長"`, `"8"`, `"7"`)
  * `STOP_BITS` (`List`, `"ストップビット"`, `"1"`, `"2"`)
  * `FLOW_CTRL` (`List`, `"フロー制御"`, `"NONE"`, `"RTSCTS"`, `"XONXOFF"`)
  * `TX_DELIM` (`List`, `"送信改行コード"`, `"TXCRLF"`, `"TXCR"`, `"NON"`)
  * `DATA_MODE` (`List`, `"受信データモード"`, `"ASCII"`, `"BIN"`)
* **シーケンス手順**:
  1. ステップ 10: `Com_Mode, "$BAUD$,$PARITY$,$DATA_BITS$,$STOP_BITS$", "$FLOW_CTRL$", "$TX_DELIM$", "$DATA_MODE$"`
  2. ステップ 20: `Com_Open, "$PORT_NAME$"`
  3. ステップ 30: `Exit`

### ② 機器初期確認・プロンプト同期：`COMMAND, "DEV_INIT", ON`
* **目的**: 初期確認コマンド（既定 `"*IDN?"`）を送信して応答プロンプト（既定 `">"`）を待機し、通信エラー状態（`COM_ERR_FLG`）が正常（`"CLR"`）であることを検証する。初回タイムアウト時は再同期送出コマンドを送信して 1 回リトライを行う。
* **パラメータ定義**:
  * `INIT_CMD` (`Text`, `"初期確認コマンド"`, `"*IDN?"`)
  * `PROMPT_STR` (`Text`, `"応答プロンプト"`, `">"`)
  * `TIMEOUT_MS` (`Text`, `"待機タイムアウト(ms)"`, `"5000"`)
  * `WAKE_CMD` (`Text`, `"再同期送出コマンド"`, `""`)
  * `ERR_STATUS` (`Work`, `"通信エラー状態"`, `""`)
* **シーケンス手順**:
  1. ステップ 10: `Com_Send, "$INIT_CMD$"`
  2. ステップ 20: `Com_WaitPrompt, "$PROMPT_STR$", "$TIMEOUT_MS$", #RETRY_INIT#`
  3. ステップ 30: `Set_Data, "ERR_STATUS", "COM_ERR_FLG"`
  4. ステップ 40: `Judge, EQU, "$ERR_STATUS$", "CLR", #ERR_EXIT#`
  5. ステップ 50: `Exit`
  6. ステップ `#RETRY_INIT#`: `Com_Send, "$WAKE_CMD$"`
  7. ステップ 70: `Com_WaitPrompt, "$PROMPT_STR$", "$TIMEOUT_MS$", #ERR_EXIT#`
  8. ステップ 80: `Exit`
  9. ステップ `#ERR_EXIT#`: `Exit`

### ③ 測定値読み出し・エコー除去抽出：`COMMAND, "MEAS_READ", "RESULT_DATA"`
* **目的**: タイムスタンプ自動付与を一時 OFF にした上で測定要求コマンド（既定 `"MEAS?"`）を送信し、応答プロンプト受信後にエコーバック行のスキップ（`ECHO_SKIP` が 1 以上の場合のみ `Skip_Line` 実行）および指定行抽出（`Get_Line`）を行って結果セル（`RESULT_DATA`）へ出力する。初回タイムアウト時は 1 回再送し、再タイムアウト時は `"ERROR_TIMEOUT"` を格納してタイムスタンプ設定を復帰する。
* **パラメータ定義**:
  * `QUERY_CMD` (`Text`, `"測定要求コマンド"`, `"MEAS?"`)
  * `PROMPT_STR` (`Text`, `"応答プロンプト"`, `">"`)
  * `TIMEOUT_MS` (`Text`, `"待機タイムアウト(ms)"`, `"5000"`)
  * `ECHO_SKIP` (`List`, `"エコー行スキップ数"`, `"1"`, `"0"`, `"2"`)
  * `TARGET_LINE` (`Text`, `"抽出対象行番号"`, `"1"`)
  * `RESULT_DATA` (`Range`, `"測定データ結果"`, `""`)
* **シーケンス手順**:
  1. ステップ 10: `Time_Stamp, "OFF"`
  2. ステップ 20: `Com_Send, "$QUERY_CMD$"`
  3. ステップ 30: `Com_WaitPrompt, "$PROMPT_STR$", "$TIMEOUT_MS$", #RETRY_MEAS#`
  4. ステップ `#PARSE_DATA#`: `Judge, GT, "$ECHO_SKIP$", "0", #EXTRACT_LINE#`
  5. ステップ 50: `Skip_Line, "$ECHO_SKIP$"`
  6. ステップ `#EXTRACT_LINE#`: `Get_Line, "RESULT_DATA", "$TARGET_LINE$"`
  7. ステップ 70: `Time_Stamp, "ON"`
  8. ステップ 80: `Exit`
  9. ステップ `#RETRY_MEAS#`: `Com_Send, "$QUERY_CMD$"`
  10. ステップ 100: `Com_WaitPrompt, "$PROMPT_STR$", "$TIMEOUT_MS$", #TIMEOUT_ERR#`
  11. ステップ 110: `Jump, #PARSE_DATA#`
  12. ステップ `#TIMEOUT_ERR#`: `Set_Data, "RESULT_DATA", "ERROR_TIMEOUT"`
  13. ステップ 130: `Time_Stamp, "ON"`
  14. ステップ 140: `Exit`

### ④ 任意コマンド送信・全応答データ取得：`COMMAND, "CMD_SEND", "RESULT_DATA"`
* **目的**: 任意のシリアルコマンド文字列を指定の行末改行コード（`CRLF` / `CR` / `LF` / `NONE`）で送信し、プロンプト待機後に受信リングバッファの応答テキスト全文を `RESULT_DATA` へ格納する。
* **パラメータ定義**:
  * `SEND_TEXT` (`Text`, `"送信コマンド文字列"`, `"STATUS"`)
  * `LINE_END` (`List`, `"行末改行指定"`, `"CRLF"`, `"CR"`, `"LF"`, `"NONE"`)
  * `PROMPT_STR` (`Text`, `"応答プロンプト"`, `">"`)
  * `TIMEOUT_MS` (`Text`, `"待機タイムアウト(ms)"`, `"5000"`)
  * `RESULT_DATA` (`Range`, `"応答テキスト結果"`, `""`)
* **シーケンス手順**:
  1. ステップ 10: `Com_Send, "$SEND_TEXT$", "$LINE_END$"`
  2. ステップ 20: `Com_WaitPrompt, "$PROMPT_STR$", "$TIMEOUT_MS$", #CMD_TIMEOUT#`
  3. ステップ 30: `Set_Data, "RESULT_DATA"`
  4. ステップ 40: `Exit`
  5. ステップ `#CMD_TIMEOUT#`: `Set_Data, "RESULT_DATA", "ERROR_TIMEOUT"`
  6. ステップ 60: `Exit`

### ⑤ 外部設定ファイル一括送信：`COMMAND, "BATCH_SEND"`
* **目的**: 外部テキストファイルに記述された初期化・設定コマンド群を `Com_SendFile` で一括送信し、完了プロンプトおよび通信エラーフラグ（`COM_ERR_FLG`）を確認する。
* **パラメータ定義**:
  * `FILE_PATH` (`Text`, `"送信ファイルパス"`, `"./config/serial_init.txt"`)
  * `WAIT_MODE` (`List`, `"送信待機モード"`, `"WaitPrompt"`, `"NoWait"`)
  * `CLEAR_ERR` (`List`, `"送信前エラークリア"`, `"true"`, `"false"`)
  * `END_PROMPT` (`Text`, `"完了確認プロンプト"`, `">"`)
  * `TIMEOUT_MS` (`Text`, `"待機タイムアウト(ms)"`, `"10000"`)
  * `ERR_STATUS` (`Work`, `"通信エラー状態"`, `""`)
* **シーケンス手順**:
  1. ステップ 10: `Com_SendFile, "$FILE_PATH$", "$WAIT_MODE$", "$CLEAR_ERR$"`
  2. ステップ 20: `Com_WaitPrompt, "$END_PROMPT$", "$TIMEOUT_MS$", #BATCH_END#`
  3. ステップ 30: `Set_Data, "ERR_STATUS", "COM_ERR_FLG"`
  4. ステップ 40: `Judge, EQU, "$ERR_STATUS$", "CLR", #BATCH_END#`
  5. ステップ `#BATCH_END#`: `Exit`

### ⑥ COMポート切断・リソース解放：`COMMAND, "PORT_CLOSE"`
* **目的**: オープン中の COM ポート回線を安全に切断し、OS シリアルハンドルを解放する。
* **パラメータ定義**: なし
* **シーケンス手順**:
  1. ステップ 10: `Com_Close`
  2. ステップ 20: `Exit`

---

## ✅ 5. 出力前セルフチェックリスト

- [ ] 他通信ドメインのコマンド（`Telnet_*`, `Modbus_*`, `Melsec_*`, `Mc_*` 等）を一切使用せず、Comm 用コマンドセット全 6 種（`Com_Mode`, `Com_Open`, `Com_Send`, `Com_SendFile`, `Com_WaitPrompt`, `Com_Close`）と共通基盤コマンドのみで完結しているか？
- [ ] 通信条件（ボーレート・パリティ・データ長・ストップビット・フロー制御・送信改行・受信データモード）および各種モード選択項目がすべて機能別に分離され、`List` 型でドロップダウン選択可能になっているか？
- [ ] `List` 型の選択肢要素内に生のカンマ（`,`）が含まれていないか（`"$BAUD$,$PARITY$,$DATA_BITS$,$STOP_BITS$"` のように `SEQUENCE` 側で結合しているか）？
- [ ] `ECHO_SKIP` が `"0"` の場合に `Skip_Line, "0"`（1以上必須制約エラー）を呼び出さないよう、事前に `Judge, GT, "$ECHO_SKIP$", "0", #EXTRACT_LINE#` で安全に分岐しているか？
- [ ] すべての `TITLE`, `COMMAND` 行が行頭（第1フィールド）から始まっているか？
- [ ] すべての `PARAM`, `SEQUENCE` 行が行頭カンマ（`,`）から始まっているか？
- [ ] すべての `PARAM` 行において、第5フィールド（表示名）および第6フィールド（初期値）が省略・欠落せずに明記されているか？
- [ ] `COMMAND` ブロック配下の `PARAM` 行・`SEQUENCE` 行に行末コメント（`;`）が一切含まれていないか？
- [ ] ファイル先頭ヘッダに正確な AI エンジン識別名（`; 生成AI: Antigravity (Gemini 4 Carbon (Medium))`）が記載され、ローカル絶対パスが含まれていないか？
- [ ] 生成される CSV ファイルは Shift-JIS (CP932) + CRLF で保存される前提となっているか？

---

## 📋 6. 出力フォーマット

余計な自然言語による前置きや解説を省き、すぐに CSV ファイル（`Comm.csv`）として保存できるように、 **マークダウンのコードブロック（`csv` 形式）に作成した CSV コードのみ** を出力してください。
