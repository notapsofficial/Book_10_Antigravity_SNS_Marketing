# Part 3｜Xを実行環境としてつなぐ（最初の技術）

## 1. 2026年の荒野にて：X API の現在地

### なぜ今さら X なのか
TikTokが全盛の時代に、なぜ X (旧Twitter) なのか。
答えはシンプルだ。**「APIが最も扱いやすく、テキストベースでシステムを組みやすいから」**だ。

動画プラットフォーム（TikTok, Instagram, YouTube）は、コンテンツ生成のコストが高すぎる。
画像生成AIや動画生成AIをパイプラインに組み込むのは、次のステップ（Part 6）の話だ。
まずは「テキスト（思考）」を自動でデプロイする経路を確保する。その実験場として、X 以上の場所はない。X はインターネットの CLI (Command Line Interface) なのだ。

### 無料枠の死と「必要経費」
現実を直視しよう。2023年のAPI有料化以降、X の Free Tier（無料枠）は「書き込みのみ、月1,500件」や「読み込み不可」といった厳しい制限が課されている（2026年時点でもこの傾向は続いているか、さらに厳格化している）。

真剣に Antigravity を運用するなら、**「Basic Tier（月額100ドル程度）」** への課金が必要になる可能性が高い。
「高い」と思うだろうか？
しかし、月1万円ちょっとで、文句も言わず24時間働き、あなたのブランドを広め続ける広報担当者を雇えるとしたら？
これは法外な安さだ。
サーバー代だと思って払え。それが「システムオーナー」の覚悟だ。

## 2. Developer Portal 攻略ガイド

### プロジェクトとアプリの作成
X Developer Portal の UI は頻繁に変わるが、基本構造は変わらない。

1.  **Project (プロジェクト)**: 一番大きな枠組み。
2.  **App (アプリ)**: 具体的な Bot の単位。ここに API Key が紐づく。

まず `Project` を作り、その中に `App` を作成する。
名前は何でもいい。「Antigravity_Test」で十分だ。

### 権限設定 (User Authentication Settings)
ここが最大の落とし穴だ。
デフォルトでは `Read Only` になっていることが多い。これでは投稿できない。

1.  App Settings の `User authentication settings` を開く。
2.  **App permissions**: 必ず **`Read and Write`** (あるいは `Read and Write and Direct message`) を選ぶ。
3.  **Type of App**: `Web App, Automated App or Bot` を選択。
4.  **Callback URI / Website URL**: テスト段階なら `https://example.com` などで一旦埋めておけば通る（OAuth認証を使わない場合）。

変更を保存すると、Client ID / Secret が発行されるが、今回使うのはそれではない。
**「Consumer Keys」と「Authentication Tokens」** だ。

### 4つの鍵 (Keys & Tokens)
`Keys and Tokens` タブに移動し、以下を生成（Regenerate）して手元に控える。

1.  **API Key** (Consumer Key)
2.  **API Key Secret** (Consumer Secret)
3.  **Access Token**
4.  **Access Token Secret**

この4つが揃って初めて、あなたのコードから X のサーバーへアクセスが可能になる。
**注意**: これらは一度しか表示されない。必ず安全なメモ帳（パスワードマネージャーなど）にコピーすること。

## 3. セキュリティ：鍵をGitHubに上げるな

### 環境変数の鉄則
初心者はやりがちだ。コードに直接キーを書いてしまう。

```python
# ❌ 絶対にやってはいけない
api_key = "abc123secret..." 
```

これを GitHub に push した瞬間、世界中の Bot があなたのキーをスクレイピングし、スパム投稿の踏み台にする。
アカウントは凍結され、最悪の場合、賠償責任すら問われる。

### .env ファイルの作成
必ず「環境変数」を使う。
プロジェクトのルートに `.env` というファイルを作る。

```text
# .env
X_API_KEY=your_api_key_here
X_API_SECRET=your_api_secret_here
X_ACCESS_TOKEN=your_access_token_here
X_ACCESS_TOKEN_SECRET=your_access_token_secret_here
```

### .gitignore の設定
そして、Git管理から外すために `.gitignore` に以下を追記する。

```text
# .gitignore
.env
__pycache__/
*.pyc
```

これで、あなたのリポジトリには `.env` が含まれなくなる。
これはマナーではない。**生存戦略**だ。

## 4. Hello World：最初の接続

### Python環境の準備
言語は何でもいいが、本書では Python を推奨する。ライブラリが豊富だからだ。
最も有名な `tweepy` を使おう。

```bash
pip install tweepy python-dotenv
```

### 接続・投稿スクリプト
以下が、世界への第一声を上げるための最小構成コードだ。

```python
import tweepy
import os
from dotenv import load_dotenv

# .env を読み込む
load_dotenv()

# 認証を行う
client = tweepy.Client(
    consumer_key=os.getenv("X_API_KEY"),
    consumer_secret=os.getenv("X_API_SECRET"),
    access_token=os.getenv("X_ACCESS_TOKEN"),
    access_token_secret=os.getenv("X_ACCESS_TOKEN_SECRET")
)

# 投稿する
try:
    response = client.create_tweet(text="Hello, Antigravity Control Plane! #SystemTest")
    print(f"Success! Tweet ID: {response.data['id']}")
except Exception as e:
    print(f"Error: {e}")
```

### 実行と確認
ターミナルで実行する。

```bash
python post_test.py
```

`Success! Tweet ID: 12345...` と表示されたら、ブラウザで自分の X アカウントを見に行こう。
そこに機械的な文字で「Hello, Antigravity Control Plane!」と表示されていたら、勝利だ。

あなたは今、アプリのUIを経由せず、APIという「裏口」から世界に干渉した。
この瞬間から、X は「暇つぶしのSNS」から**「あなたがプログラム可能な出力デバイス」**へと変わったのだ。

## 5. この章のまとめとマイルストーン

### マイルストーン
*   [x] X Developer Portal で App を作成し、`Read and Write` 権限を付与した。
*   [x] 4つのキー（Consumer系、Access Token系）を取得した。
*   [x] `.env` と `.gitignore` を設定し、キー漏洩対策を行った。
*   [x] Pythonスクリプト経由で「Hello World」を投稿し、成功を確認した。

次章、このパイプラインを使って、いよいよ運用を回し始める。
ただし、AIに全て任せるのはまだ早い。
「部長（あなた）と新入社員（AI）」の二人三脚モデルを構築する。
