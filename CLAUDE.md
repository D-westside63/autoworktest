# CLAUDE.md

このファイルは、本リポジトリで作業する Claude Code へのガイドです。

## プロジェクト概要

- **プロジェクト名**: autoworktest-app
- **目的**: Google Apps Script (GAS) を使って業務を自動化するアプリ
- **実行環境**: Google Apps Script（V8 ランタイム）

## コミュニケーション

- 返答・説明・コードコメントは**必ず日本語**で書くこと。
- 技術用語や識別子（関数名・API 名など）は原文のまま使ってよい。

## 技術スタック

- 言語: JavaScript（GAS / V8 ランタイム）
- 主に利用する Google サービス（必要に応じて追加）:
  - `SpreadsheetApp`（スプレッドシート操作）
  - `GmailApp` / `MailApp`（メール送信・取得）
  - `DriveApp`（ファイル操作）
  - `CalendarApp`（カレンダー操作）
  - `UrlFetchApp`（外部 API 呼び出し）
- ローカル開発には `clasp` の利用を想定する。

## ディレクトリ構成（想定）

```
autoworktest-app/
├── CLAUDE.md
├── .clasp.json        # clasp 設定（scriptId を含む）
├── appsscript.json    # GAS マニフェスト（タイムゾーン・スコープ等）
└── src/
    ├── main.js        # エントリポイント・トリガー用関数
    ├── config.js      # 定数・設定値
    └── utils.js       # 共通ユーティリティ
```

構成を変更した場合はこのセクションも更新すること。

## 開発コマンド（clasp）

```bash
npm install -g @google/clasp   # clasp のインストール
clasp login                     # Google アカウントでログイン
clasp clone <scriptId>          # 既存プロジェクトの取得
clasp push                      # ローカルの変更を GAS へ反映
clasp pull                      # GAS 側の変更をローカルへ取得
clasp open                      # ブラウザでエディタを開く
clasp logs                      # 実行ログの確認
```

## コーディング規約

- `const` / `let` を使い、`var` は使わない。
- 関数名・変数名は camelCase、定数は UPPER_SNAKE_CASE。
- GAS はファイルをまたいでグローバルスコープを共有するため、関数名の重複に注意する。
- 内部用のヘルパー関数は末尾に `_` を付ける（例: `formatDate_`）。末尾 `_` の関数はエディタの実行メニューや `google.script.run` から呼べなくなる。
- 公開する関数（トリガー・メニューから呼ぶもの）には JSDoc で用途を書く。

## GAS 特有の注意点

- **実行時間制限**: 1 回の実行は最大 6 分。大量データは分割処理し、続きは時間主導型トリガーで再開する。
- **API 呼び出しの効率化**: `getValue()` / `setValue()` をループ内で繰り返さず、`getValues()` / `setValues()` で一括処理する。
- **クォータ**: メール送信数・`UrlFetchApp` の呼び出し回数などの上限に注意する。
- **同時実行**: 重複実行の恐れがある処理には `LockService` を使う。
- **エラー処理**: トリガー実行は失敗に気づきにくいため、`try...catch` でログ出力や通知を行う。
- **スコープ**: `appsscript.json` の `oauthScopes` は必要最小限にする。

## セキュリティ

- API キー・パスワード・トークンなどの秘密情報はコードに直接書かず、`PropertiesService`（スクリプトプロパティ）に保存する。
- `.clasp.json` や認証情報ファイルを公開リポジトリにコミットしないこと。
- 外部から受け取った値（Web アプリのパラメータ等）は必ず検証する。

## テスト・動作確認

- 本番データに影響する処理は、テスト用のスプレッドシートやフォルダで先に確認する。
- 動作確認用に `test` で始まる関数（例: `testSendReport`）を用意し、エディタから手動実行できるようにする。
- ログは `console.log` を使い、`clasp logs` または実行ログで確認する。

## Claude への作業ルール

- `clasp push` やトリガーの作成など、本番の GAS プロジェクトに影響する操作は実行前にユーザーへ確認すること。
- メール送信・ファイル削除など取り消せない処理を含むコードを書く場合は、その旨を明示すること。
