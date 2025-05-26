# wp-booster-lite

## 3.1 概要

**役割:**

*   WordPressサイトに構造化データ（JSON-LD）を自動的に挿入し、AI検索結果での可視性を向上させます。
*   アクションエンジン（GPT）から送信された改善提案をワンクリックで適用できます。
*   OSS（GPL）プラグインとしてリリースされ、GitHubおよびWordPress.org経由で配布されます。

## 3.2 ディレクトリ構造の例

```
wp-booster-lite/
 ├─ booster-lite.php      # メインプラグインファイル（ヘッダー：プラグイン名、GPLなど）
 ├─ inc/
 │   ├─ jsonld.php        # JSON-LD生成ロジック
 │   ├─ webhook.php       # REST APIルート（radineer/v1/action）
 │   └─ ...
 ├─ build/
 │   └─ admin.js          # ReactまたはWP-scriptsによって作成されたバンドル
 ├─ src/ (オプション)
 │   └─ admin-ui/         # Reactソースコード
 ├─ languages/
 │   └─ booster-lite-ja.po
 ├─ readme.txt            # WordPress.org固有（最低限必要なバージョン/テスト済みバージョンなど）
 ├─ package.json
 ├─ webpack.config.js or wp-scripts.config.js
 ├─ composer.json (オプション)
 ├─ tests/                # PHPUnit + WP_Mock
 └─ README.md
```

## 3.3 主要ファイル

*   **`booster-lite.php`**:
    *   プラグインヘッダー（プラグイン名、バージョン、作成者、ライセンスなど）。
    *   `register_activation_hook`、`register_deactivation_hook`など。
*   **`inc/jsonld.php`**:
    *   記事保存時に`<script type="application/ld+json">`を挿入（`add_action('save_post', 'radineer_add_jsonld')`を使用）。
    *   FAQ / HowTo / Article / Productなどのスキーマを生成。
*   **`inc/webhook.php`**:
    *   `register_rest_route('radineer/v1', '/action', [...])`経由の受信エンドポイント。
    *   JWTとnonceの検証 → 投稿メタ`_radineer_action_log`に保存 → 管理UIに表示。
*   **`build/admin.js`**:
    *   wp-scripts（React）で構築された管理GUI。
    *   APIキー入力フィールド/ダッシュボードリンク/提案リスト+採用ボタンなどを含む。

## 3.4 インストールと開発フロー

*   **環境**: WordPress 5.8以上
*   **推奨PHP**: 7.4以上
*   **ローカルテスト**:
    1.  `npm install` → `npm run build`で`admin.js`をビルド。
    2.  `wp-booster-lite`フォルダをWordPressに配置 → 有効化。
*   **単体テスト**:
    *   `phpunit --bootstrap=tests/bootstrap.php tests/test-webhook.php`
    *   WP_MockまたはWP-CLIテスト。
*   **デバッグ**:
    *   `WP_DEBUG=true`を設定して通知/警告を確認。
    *   ログ: `wp-content/debug.log`。
*   **配布（ZIP作成）**:
    *   GitHub Actions: タグプッシュ → ZIP作成。
    *   VirusTotalスキャンまたはWP.org SVNへのアップロード。

## 3.5 Webhook統合に関する注意点

*   **セキュリティ**:
    *   JWT: `Authorization: Bearer <token>`
    *   WordPress側: `wp_verify_nonce()`または同等の権限チェック（例: `current_user_can('manage_options')`）。
*   **ログ管理**:
    *   カスタムフィールド`_radineer_action_log`またはカスタムテーブル。
    *   アクションの提案は、管理パネルの「Booster Lite → 提案リスト」で確認可能。

## 3.6 JSON-LD挿入ロジック

*   **例: FAQスキーマ**
    ```php
    function radineer_add_jsonld($post_id) {
        // 投稿ステータスを確認し、「公開」でなければ何もしない
        // 配列 $faq_data = [ ... ] を生成
        $jsonld = json_encode($faq_data, JSON_UNESCAPED_SLASHES|JSON_UNESCAPED_UNICODE);
        echo "<script type='application/ld+json'>$jsonld</script>";
    }
    ```
*   **キャッシュ処理**:
    *   `DOING_AUTOSAVE`中またはリビジョンでは処理しない。
    *   オブジェクトキャッシュ/トランジェントを検討。
