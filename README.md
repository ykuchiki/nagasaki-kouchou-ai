## nagasaki-kouchou-ai 静的レポートデプロイ手順

このリポジトリは、Next.js プロジェクトから **静的エクスポートした成果物だけ** を配置するためのリポジトリです。  
ルートの `index.html` から `./nagasaki-kensei/` にリダイレクトし、その配下にレポートのビルド成果物を格納します。

最小構成にするため、`nagasaki-kensei` には次のファイル / ディレクトリ **だけ** を置きます。

- `_next/`
- `images/`
- `js/`
- `meta/`
- `index.html`
- `index.txt`

それ以外（`404`, `faq` などの追加ルートや一時ディレクトリ）は、このリポジトリには含めません。

---

## 更新フロー（毎回のレポート差し替え手順）

ここでは、開発用の Next.js プロジェクトから新しいレポートをエクスポートし、このリポジトリに反映する一連の流れをまとめます。

### 1. 開発用プロジェクトで静的エクスポートを実行

1. 開発用リポジトリでレポートの内容を更新する。
2. Next.js の静的エクスポートを実行し、**日付付きのフォルダ名** で出力する。
   - 例: `kouchou-ai-2026-03-17-18-33`
3. 出力先フォルダ（例: `kouchou-ai-2026-03-17-18-33`）を、このリポジトリのルートに配置する。

> メモ: このフォルダには、`404`, `faq`, 一時ディレクトリなど、今回のレポート公開には不要なルートが含まれていても構いません。後続の手順で必要なものだけを取り出します。

### 2. 旧レポートのバックアップを作成

リポジトリのルートで次を実行します。

```bash
cd /Users/kuchiki/Desktop/nagasaki-kouchou-ai

# 旧レポートをバックアップとして退避
mv nagasaki-kensei nagasaki-kensei-backup-YYYY-MM-DD
mkdir nagasaki-kensei
```

- `YYYY-MM-DD` 部分は実際の日付に置き換えてください。
- 問題が起きた場合は、このバックアップからすぐに戻せます。

### 3. 新しいエクスポートから必要なものだけコピー

先ほど追加した日付付きフォルダ名（ここでは `kouchou-ai-2026-03-17-18-33` を例にします）から、`nagasaki-kensei` に **必要なものだけ** をコピーします。

```bash
cd /Users/kuchiki/Desktop/nagasaki-kouchou-ai

EXPORT_DIR=kouchou-ai-2026-03-17-18-33

cp -R "$EXPORT_DIR/_next" nagasaki-kensei/
cp -R "$EXPORT_DIR/images" nagasaki-kensei/
cp -R "$EXPORT_DIR/js" nagasaki-kensei/
cp -R "$EXPORT_DIR/meta" nagasaki-kensei/

cp "$EXPORT_DIR/index.html" nagasaki-kensei/
cp "$EXPORT_DIR/index.txt" nagasaki-kensei/
```

ポイント:

- `404`, `faq`, `d2b0...` など、`_next`, `images`, `js`, `meta`, `index.html`, `index.txt` 以外は **コピーしません**。
- これにより、`nagasaki-kensei` は常に最小限の構成に保たれます。

### 4. ルートのリダイレクトはそのまま利用

ルートの `index.html` は次のような単純なリダイレクトになっています。

```html
<!doctype html>
<meta charset="utf-8">
<meta http-equiv="refresh" content="0; url=./nagasaki-kensei/">
<link rel="canonical" href="./nagasaki-kensei/">
<title>Redirecting...</title>
```

- `./nagasaki-kensei/` のままであれば、特に変更不要です。
- ブラウザでルートの `index.html`（またはホスティング先の URL）にアクセスし、新しいレポート内容が表示されることだけ確認します。

### 5. 動作確認とコミット / デプロイ

1. ブラウザでレポート画面を開き、内容・リンク・OGP などを確認する。
2. 問題なければ、このリポジトリで変更をコミットし、デプロイ先に反映する。

> もし新しいエクスポートで重大な問題が見つかった場合は、`nagasaki-kensei-backup-YYYY-MM-DD` を `nagasaki-kensei` に戻すことで、即座に元のレポートにロールバックできます。

