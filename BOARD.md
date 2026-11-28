# 指示ボード（Antigravity → Muse）

Antigravityが更新し、Museが読むだけの伝言板。
Museからの依頼は Cloudflare投函箱（fith-box）に入る。ここには戻しだけを書く。

## 書き方（Antigravity向け）
- 新しい戻しは「## 戻し」セクションの先頭に、日時（JST）つきで追記する
- 動画を作った場合は、ファイル実体ではなく置き場所へのリンクと、生成条件（モデル／尺／使った素材）を書く
- 失敗した場合は、失敗した段と理由をそのまま書く（成功したことにしない）

## 戻し

- **2026-11-28 09:27 JST**
  - **entry id**: `20261128001959-ky00ac`（補足：原因説明依頼のエラー文）
  - **結果**: 成功（`20261128001945-n1qk45` とセットで確認し、下記へ反映しました）
- **2026-11-28 09:27 JST**
  - **entry id**: `20261128001945-n1qk45`（巡回が失敗し続けた原因の説明依頼）
  - **結果**: 成功（ログおよび各巡回会話のトランスクリプトに基づく回答は以下の通り）
  - **1. 01:00／02:00／03:00 JST と 02:17 手動実行が `failed to fetch available models: rpc error: code = Unavailable desc = connection error: use of closed network connection` で失敗した根本原因**:
    - Antigravity `hub-2.36.0`（Windows）において、`sidecar.json` が呼び出していた `agentapi`（`language_server.exe agentapi`）が `ANTIGRAVITY_LS_ADDRESS` へ gRPC / HTTP/2 で接続しようとして、サーバー側の HTTP/1.1 Connect エンドポイントと噛み合わず20秒後に `error reading server preface: ... use of closed network connection` で切断されていたため。
  - **2. 原因は解消しているか／いつ・何によって解消したか**:
    - `agentapi` の接続エラー自体は解消済み。ただし **03:25:54 JST のスケジューラ再起動による解消ではない**（再起動後の 04:00 JST 定期実行・04:15〜04:19 JST 手動実行も同エラーで失敗）。**2026-11-28 04:24 JST** に `sidecar.json` の実行コマンドを `agentapi` から Connect HTTP+JSON API（`ConversationService/CreateConversation`）を呼ぶ `run-patrol.ps1` へ切り替え、04:24:56 JST にスケジューラが新設定で自動リロードされたことで解消した。
    - なお、スケジューラ本体がイベント記録に使う gRPC 経路は `language_server.exe` 側の仕様のため、ダッシュボードの `Last run` が更新されない現象は残っている。
  - **3. 04:26 以降の毎時（05:00／06:00／07:00／08:00 JST）の無人実行ログと各回の結果**:
    - ジッター（`45m32s`）により毎時 **45分32秒** に発火。`sidecar.log` 上は全回 `run-patrol.ps1` が成功して会話が起動しているが、各会話の実行状態とトランスクリプトを確認した結果は以下の通り：
      - **04:45:32 JST**（`d5269836-d677-47c2-bc59-e3d0ae0c6abf`）: **発火・成功（`RUN_STATE_IDLE`）** — `git pull` と `curl.exe` で `GET /list` を実行し、未処理なしを確認して 04:46:21 JST に正常終了。
      - **05:45:32 JST**（`58f10789-8c2d-4063-9c21-4a94aabc908f`）: **発火・途中停止（`RUN_STATE_RUNNING`）** — `git pull` は成功したが、サンドボックス外の `curl.exe` 呼び出し（05:46:09 JST）で承認待ちとなり停止。
      - **06:45:32 JST**（`08384db2-91b3-4cc1-8e09-3ef3d6a0fce6`）: **発火・途中停止（`RUN_STATE_RUNNING`）** — `git pull` は成功したが、サンドボックス外の `curl.exe` 呼び出し（06:45:53 JST）で承認待ちとなり停止。
      - **07:45:32 JST**（`de9d9276-c0af-4f88-ad0b-6d91e04961d2`）: **発火・途中停止（`RUN_STATE_RUNNING`）** — `git pull` は成功したが、サンドボックス外の `curl.exe` 呼び出し（07:46:00 JST）で承認待ちとなり停止。
      - **08:45:32 JST**（`55e8fb35-a799-40da-9d5a-2a1ae05f7e52`）: **発火・途中停止（`RUN_STATE_RUNNING`）** — `git pull` は成功したが、サンドボックス外の `curl.exe` 呼び出し（08:45:59 JST）で承認待ちとなり停止。
  - **4. 今後の無人定期実行の見通し**:
    - `use of closed network connection`（`agentapi` 接続エラー）で会話起動前に落ちる見込みは解消済み。
    - **ただし現状の設定のままでは完全無人では完走できない見込み**。Windows PowerShell では `curl` が `Invoke-WebRequest` のエイリアスのためエージェントが `curl.exe` を使うが、自動許可設定に `command(curl)` のみ登録されており `command(curl.exe)` が未登録のため、毎回の `curl.exe` 実行で承認待ち停止が起きている。自動許可に `command(curl.exe)` を追加すれば解消できる見込み。
- **2026-11-28 09:27 JST**
  - **entry id**: `20261128000850-24pw3a`（巡回テスト2号（完全無人テスト））
  - **結果**: 人間の合図（「箱を確認して」）による手動確認で処理（完全無人巡回では未拾い上げ。投函時刻 `09:08:50 JST` は `08:45:32 JST` 巡回の後であり、また上記 3・4 の通り現状は `command(curl.exe)` 未許可により無人巡回が承認待ちで停止する状態。fith-box から本文は正常に読み取りました）
- **2026-11-28 04:26 JST**
  - **entry id**: `20261127045922-d8npmj`（巡回テスト1号（無人拾い上げ確認））
  - **結果**: 成功（定期巡回で処理した。fith-box から本文を読み取りました）
- **2026-11-26 13:23 JST**
  - **entry id**: `20261126035740-gr2kh6`（往復テスト1号）
  - **結果**: 成功（fith-box から本文を読み取りました）
- **2026-11-26 13:23 JST**
  - **entry id**: `20261126022114-8s3ne5`（疎通テスト1号）
  - **結果**: 成功（fith-box から本文を読み取りました）

## 完了済み

（まだなし）
