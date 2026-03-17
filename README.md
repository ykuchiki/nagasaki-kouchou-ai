# nagasaki-kouchou-ai 静的レポートデプロイ手順

このリポジトリは、Next.js プロジェクトから静的エクスポートしたレポートを GitHub Pages で公開するためのリポジトリです。

- ルートの `index.html` → `./nagasaki-kensei/` にリダイレクト
- `nagasaki-kensei/` 配下がレポート本体（シングルレポート構成を常に維持）
- 公開 URL: `https://ykuchiki.github.io/nagasaki-kouchou-ai/nagasaki-kensei/`

---

## 前提知識

開発プロジェクト（kouchou-ai）の静的エクスポートは **マルチレポート構成**（`/[slug]/` 配下に各レポート）でビルドされます。  
そのまま配置すると GitHub Pages のパスと一致しないため、以下の3点を手動で修正してからデプロイします。

1. **webpack publicPath の修正**  
   動的チャンクの取得先が `/_next/` (絶対パス) のままだと GitHub Pages では 404 になるため `./_next/` (相対パス) に書き換える
2. **index.html の絶対パスを相対パスに変換**  
   `/_next/`, `/images/`, `/meta/` → `./_next/`, `./images/`, `./meta/`
3. **index.txt の slug 書き換え**  
   新ビルドのレポート ID（例: `d2b0836e-...`）を `nagasaki-kensei` に置換し、シングルレポート構成を維持する

---

## 更新手順

### 1. 開発プロジェクトで静的エクスポートを実行

開発プロジェクト（kouchou-ai）でレポートを更新後、静的エクスポートして **日付付きフォルダ名** でこのリポジトリのルートに配置します。

```
nagasaki-kouchou-ai/
├── kouchou-ai-YYYY-MM-DD-HH-mm/   ← 今回のエクスポート
├── nagasaki-kensei/                ← 現在公開中
└── ...
```

エクスポートフォルダの中には以下のようなディレクトリ構成があります。

```
kouchou-ai-YYYY-MM-DD-HH-mm/
├── _next/
├── d2b0836e-4ad6-476c-aa42-c1a405ac5f63/  ← 公開したいレポート（IDはその都度変わる）
│   ├── index.html
│   └── index.txt
├── images/
├── js/
├── meta/
└── ...（faq, 404 など今回は不要）
```

### 2. 変数を設定する

ターミナルでこのリポジトリのルートに移動し、以下の変数を設定します。

```bash
cd /Users/kuchiki/Desktop/nagasaki-kouchou-ai

EXPORT_DIR=kouchou-ai-YYYY-MM-DD-HH-mm   # 今回のエクスポートフォルダ名
REPORT_DIR=d2b0836e-4ad6-476c-aa42-c1a405ac5f63  # 公開したいレポートのディレクトリ名
```

> `REPORT_DIR` はエクスポートフォルダ内に UUID 形式で作られます。`ls $EXPORT_DIR/` で確認してください。

### 3. 変更されたチャンクを差し替える

新旧ビルドで変更されたチャンクファイル（JS）だけを差し替えます。

```bash
CHUNKS="nagasaki-kensei/_next/static/chunks"
NEW_CHUNKS="$EXPORT_DIR/_next/static/chunks"
SLUG_DIR="$CHUNKS/app/[slug]"
NEW_SLUG_DIR="$NEW_CHUNKS/app/[slug]"

# 変更チャンクを差し替え（diff で確認してから実行）
diff <(ls "$CHUNKS/") <(ls "$NEW_CHUNKS/")
diff <(ls "$CHUNKS/app/") <(ls "$NEW_CHUNKS/app/")
diff <(ls "$SLUG_DIR/") <(ls "$NEW_SLUG_DIR/")
```

diff の結果をもとに、削除と追加を行います。例として前回の差分：

```bash
# ルート直下
rm -f "$CHUNKS/609-e6afbba7cf646b18.js"            # ← diff で < になったファイル
rm -f "$CHUNKS/main-app-d1e5c503935f9a8e.js"
cp "$NEW_CHUNKS/347-544cffd195e8f4e1.js" "$CHUNKS/" # ← diff で > になったファイル
cp "$NEW_CHUNKS/main-app-bd6aa490016514ed.js" "$CHUNKS/"

# app/layout, app/page
rm -f "$CHUNKS/app/layout-be7f58cf6f9b9f21.js"
rm -f "$CHUNKS/app/page-fe99431e78a91718.js"
cp "$NEW_CHUNKS/app/layout-0a969c4cbb7e098e.js" "$CHUNKS/app/"
cp "$NEW_CHUNKS/app/page-d0cdddc5a1a00233.js" "$CHUNKS/app/"

# app/[slug]
rm -f "$SLUG_DIR/error-b2784470a4722ef4.js"
rm -f "$SLUG_DIR/not-found-5a5b49287a481c18.js"
rm -f "$SLUG_DIR/page-2d806954bbec0f0a.js"
rm -f "$SLUG_DIR/opengraph-image.png/route-998f4c15612006bc.js"
cp "$NEW_SLUG_DIR/error-0d4b6919983e5735.js" "$SLUG_DIR/"
cp "$NEW_SLUG_DIR/not-found-c94d826b811e1902.js" "$SLUG_DIR/"
cp "$NEW_SLUG_DIR/page-5dc97d82915a1506.js" "$SLUG_DIR/"
cp "$NEW_SLUG_DIR/opengraph-image.png/route-b32c0c0ef44fb0ba.js" "$SLUG_DIR/opengraph-image.png/"

# app/faq, app/_not-found
rm -f "$CHUNKS/app/faq/page-ca803cab7d48a890.js"
rm -f "$CHUNKS/app/_not-found/page-de776987810f4020.js"
cp "$NEW_CHUNKS/app/faq/page-71abe237355b167f.js" "$CHUNKS/app/faq/"
cp "$NEW_CHUNKS/app/_not-found/page-c8e814617426d792.js" "$CHUNKS/app/_not-found/"
```

### 4. buildManifest ディレクトリを差し替える

ビルドごとにハッシュ名のディレクトリが変わります。

```bash
# 現在のハッシュを確認
ls nagasaki-kensei/_next/static/

# 新しいハッシュを確認
ls "$EXPORT_DIR/_next/static/"

# 旧ハッシュディレクトリを削除し、新しいものをコピー
rm -rf nagasaki-kensei/_next/static/<旧ハッシュ>
cp -R "$EXPORT_DIR/_next/static/<新ハッシュ>" nagasaki-kensei/_next/static/
```

### 5. webpack の publicPath を修正する

新ビルドの webpack ファイルは `r.p="/_next/"` のまま。これを `./_next/"` に書き換えてコピーします。

```bash
sed 's|r\.p="/_next/"|r.p="./_next/"|g' \
  "$EXPORT_DIR/_next/static/chunks/webpack-0c583ebe95f27565.js" \
  > nagasaki-kensei/_next/static/chunks/webpack-0c583ebe95f27565.js

# 確認（./_next/ と表示されれば OK）
grep -o 'r\.p="[^"]*"' nagasaki-kensei/_next/static/chunks/webpack-0c583ebe95f27565.js
```

### 6. index.html の絶対パスを相対パスに変換する

新ビルドのレポート詳細ページ (`$REPORT_DIR/index.html`) をコピーしつつ、パスを書き換えます。

```bash
sed 's|href="/_next/|href="./_next/|g
     s|src="/_next/|src="./_next/|g
     s|href="/images/|href="./images/|g
     s|src="/images/|src="./images/|g
     s|href="/meta/|href="./meta/|g
     s|src="/meta/|src="./meta/|g' \
  "$EXPORT_DIR/$REPORT_DIR/index.html" \
  > nagasaki-kensei/index.html

# 確認（0件なら OK）
grep -cE '(href|src)="/((_next|images|meta)/)' nagasaki-kensei/index.html
```

### 7. index.txt の slug を書き換える

```bash
sed "s|$REPORT_DIR|nagasaki-kensei|g" \
  "$EXPORT_DIR/$REPORT_DIR/index.txt" \
  > nagasaki-kensei/index.txt

# 確認（0件なら OK）
grep -c "$REPORT_DIR" nagasaki-kensei/index.txt
```

### 8. 動作確認してコミット・push

```bash
# 変更内容を確認
git diff --stat

# 問題なければコミット・push
git add nagasaki-kensei
git commit -m "chore: update report data YYYY-MM-DD"
git push
```

GitHub Pages が更新されるまで 1〜2 分かかります。  
https://ykuchiki.github.io/nagasaki-kouchou-ai/nagasaki-kensei/ で確認してください。

---

## ロールバック手順

万が一表示が壊れた場合は、直前のコミットを `git revert` して push することで即座に戻せます。

```bash
git log -3 --oneline   # 戻したいコミットのハッシュを確認
git revert <hash>
git push
```
