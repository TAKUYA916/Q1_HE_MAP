# QMK Keymap Editor

QMKファームウェアのキーマップを視覚的に編集できるWebアプリケーションです。Google Cloud Run上で動作し、Google Cloud Storageにデータを保存できます。

## 主な機能

### 1. ビジュアルキーマップエディター
- キーボードレイアウトの視覚的な表示
- ドラッグ&ドロップでキーコードを編集
- 複数レイヤーの管理
- エンコーダー対応

### 2. クラウド保存機能
- Google Cloud Storageへのデータ保存・読み込み
- 10個のスロットで複数のキーボード設定を管理
- 自動エラーハンドリング

### 3. GitHub連携
- GitHubリポジトリから`keyboard.json`と`keymap.c`を直接読み込み
- レイアウト情報とキーマップの自動解析
- 元のコード構造を完全に保持

### 4. keymap.c出力
- 編集したキーマップを`keymap.c`形式でダウンロード
- 元のコード(ライセンス、include、関数など)を保持
- キーマップ配列部分のみを置換

## ローカルでの実行

### 前提条件
- Node.js 18以上
- Google Cloud SDKがインストールされていること
- GCPプロジェクトが作成されていること

### 手順

1. **依存関係のインストール**
```bash
cd "O:\google cloud run ツール\QMK-Keymap-Editor"
npm install
```

2. **GCSバケットの作成**
```bash
gcloud storage buckets create gs://qmk-map-storage --location=asia-northeast1
```

3. **ローカルで実行**
```bash
npm start
```

4. **ブラウザでアクセス**
```
http://localhost:8080
```

## Cloud Runへのデプロイ

### 1. Dockerイメージのビルドとプッシュ

```bash
cd "O:\google cloud run ツール\QMK-Keymap-Editor"

# プロジェクトIDを設定
PROJECT_ID="your-project-id"

# Dockerイメージをビルド
gcloud builds submit --tag gcr.io/$PROJECT_ID/qmk-keymap-editor
```

### 2. Cloud Runにデプロイ

```bash
gcloud run deploy qmk-keymap-editor \
  --image gcr.io/$PROJECT_ID/qmk-keymap-editor \
  --platform managed \
  --region asia-northeast1 \
  --allow-unauthenticated \
  --set-env-vars BUCKET_NAME=qmk-map-storage
```

### 3. サービスアカウントに権限を付与

Cloud Runのサービスアカウントに、GCSバケットへのアクセス権限を付与します。

```bash
# サービスアカウントのメールアドレスを取得
SERVICE_ACCOUNT=$(gcloud run services describe qmk-keymap-editor --region=asia-northeast1 --format='value(spec.template.spec.serviceAccountName)')

# Storage Object Admin権限を付与
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:$SERVICE_ACCOUNT" \
  --role="roles/storage.objectAdmin"
```

## 使い方

### 1. GitHubからキーマップを読み込む

1. 「保存/読込」タブを開く
2. 「GitHub URL から読み込み」セクションに移動
3. `keyboard.json`のGitHub raw URLを入力
4. `keymap.c`のGitHub raw URLを入力
5. 「GitHubから読み込み」ボタンをクリック

### 2. キーマップを編集する

1. キーボードレイアウト上のキーをクリック
2. 右側のパネルでキーコードを選択または入力
3. 複数のレイヤーを切り替えて編集

### 3. クラウドに保存する

1. 「保存/読込」タブを開く
2. 「クラウド保存」セクションでスロットを選択
3. 「クラウドに保存」ボタンをクリック

### 4. keymap.cをダウンロードする

1. 「保存/読込」タブを開く
2. 「キーマップ生成」セクションで「keymap.c ダウンロード」ボタンをクリック
3. 元のコード構造を保持したまま、編集したキーマップが出力されます

## トラブルシューティング

### 保存・読み込みエラー

**エラー**: `バケット "qmk-map-storage" が存在しません`
- **解決策**: GCSバケットを作成してください
```bash
gcloud storage buckets create gs://qmk-map-storage --location=asia-northeast1
```

**エラー**: `Cloud Runサービスアカウントに Storage Object Admin 権限が必要です`
- **解決策**: サービスアカウントに権限を付与してください(上記の「サービスアカウントに権限を付与」を参照)

### GitHub読み込みエラー

**エラー**: `GitHub returned 404`
- **解決策**: URLが正しいか確認してください。GitHub raw URLは`https://raw.githubusercontent.com/...`で始まる必要があります

## ライセンス

このプロジェクトはMITライセンスの下で公開されています。
