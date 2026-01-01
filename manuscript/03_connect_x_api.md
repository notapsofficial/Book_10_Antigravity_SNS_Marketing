# Part 3｜Xを実行環境としてつなぐ（最初の技術）

## 第1章：2026年の荒野にて：X API の現在地

### 1-1. なぜ今さら X (Twitter) なのか

2026年、SNSの選択肢は無数にある。
TikTok, Instagram Reels, YouTube Shorts が全盛の時代に、なぜ「枯れた技術」である X (旧Twitter) なのか。
答えはシンプルだ。
**「APIが最も扱いやすく、テキストベースでシステムを組みやすいから」**だ。

動画プラットフォーム（TikTok等）は、コンテンツ生成のコストが高すぎる。
AIで動画を作ることは可能だが、レンダリングコスト、品質チェックの手間、APIのアップロード制限など、個人開発者が最初に手を出すにはハードルが高すぎる。
「自動化」を目指して挫折する一番の原因は、この「メディアの重さ」にある。

対して X は、テキスト（と思考）が主体のプラットフォームだ。
データ量が軽く、APIのレスポンスも速く、Pythonスクリプト数行で制御できる。
X はインターネットにおける **CLI (Command Line Interface)** なのだ。
システムとしてのSNS運用を学ぶための「最初の実験場」として、これ以上の環境はない。

### 1-2. 無料枠の死と「必要経費」

夢を見るのはやめよう。現実を直視する時間だ。
かつて存在した「誰でも無料で使い放題のTwitter API」は、2023年に死んだ。
2026年現在、Free Tier（無料枠）は「書き込みのみ月1,500件」や「読み込み不可」といった、極めて厳しい制限が課されている（あるいは完全に廃止されている可能性もある）。

真剣に Antigravity を運用し、成果（トラフィック）を出したいなら、
**「Basic Tier（月額100ドル程度）」** への課金が、事実上のスタートラインだと思ってほしい。

「高い」と思うだろうか？ 月1万円強だ。
しかし、考えてみてほしい。
月1万数千円で、文句も言わず、有給も取らず、24時間365日働き続け、あなたのブランドを世界中に広め続ける広報担当者を雇えるとしたら？
これは法外な安さだ。
サーバー代だと思って払え。AWSに払う金は惜しまないのに、自分のマーケティングエンジンへの投資を惜しむのは、エンジニアの悪癖だ。
これはコストではない。**投資**だ。

## 第2章：Developer Portal 攻略ガイド

Twitter Developer Portal のUIは頻繁に変更されるが、基本構造（概念）は変わらない。
迷子にならないための地図を渡そう。

### 2-1. Project と App の階層構造

1.  **Project (プロジェクト)**:
    一番大きな枠組み。通常は1つあればいい。APIの利用量（Usage Cap）はこのプロジェクト単位で管理される。
2.  **App (アプリ)**:
    具体的な Bot の単位。ここに API Key が紐づく。一つのプロジェクトの中に複数の App を作れるが、まずは「Antigravity_Main」などの名前で1つ作ればいい。

### 2-2. 権限設定の罠 (User Authentication Settings)

ここが最大の落とし穴だ。
Appを作成した直後、デフォルトの権限設定は **`Read Only`** になっていることが多い。
このままでは、どんなに正しいコードを書いても「403 Forbidden」で弾かれる。

必ず以下の手順で権限を変更せよ。

1.  Developer Portal で対象の App を選択し、`User authentication settings` の `Edit` を押す。
2.  **App permissions**:
    必ず **`Read and Write`** (あるいはDMも使うなら `Read and Write and Direct message`) をラジオボタンで選択する。
3.  **Type of App**:
    `Web App, Automated App or Bot` を選択。
4.  **App info**:
    *   **Callback URI / Redirect URL**: テスト段階なら `https://example.com` や `http://127.0.0.1:5000` で一旦埋めておけば通る（OAuth 2.0認証を厳密に行う場合は正確なURLが必要だが、個人のBotならこれで十分だ）。
    *   **Website URL**: 自分のブログやGitHubのプロフィールURLを入れておけばいい。

設定を保存すると、Client ID / Secret が発行される。
だが、今回我々が主に使うのはそれではない。**OAuth 1.0a** のキーだ。

### 2-3. 4つの鍵 (Keys & Tokens)

`Keys and Tokens` タブに移動し、以下の4つを生成（Regenerate）して手元に控える。

1.  **API Key** (Consumer Key): アプリ自体のID。
2.  **API Key Secret** (Consumer Secret): アプリ自体のパスワード。
3.  **Access Token**: あなたのアカウントへのアクセス権。
4.  **Access Token Secret**: その署名鍵。

**重要**: これらは生成直後の一度しか画面に表示されない。
ブラウザを閉じたら二度と見られない。
必ず安全な場所（パスワードマネージャーなど）にコピーすること。
間違っても「ふせん」や「Slackの自分宛てDM」に残してはいけない。

## 第3章：セキュリティ（生存戦略）

### 3-1. 環境変数の鉄則

初心者はやりがちだ。コードの中に直接キーを書いてしまう。

```python
# ❌ 絶対にやってはいけない (Death Flag)
api_key = "abc123secret..." 
secret = "xyz987very_secret..."
```

これを GitHub のパブリックリポジトリに push した瞬間、何が起きるか。
世界中を巡回している Bot が数秒であなたのキーを検知し、スクレイピングし、悪用する。
あなたのアカウントは一瞬で「暗号資産の詐欺ツイート」を大量に撒き散らすスパムマシンに変貌し、X社によって永久凍結される。
最悪の場合、賠償責任すら問われる。

これは脅しではない。日常茶飯事だ。

### 3-2. .env ファイルによる隔離

必ず「環境変数」を使う。これがシステムオーナーの嗜みだ。
Python の場合、`python-dotenv` というライブラリが標準的だ。

プロジェクトのルートに `.env` というファイルを作る。
```text
# .env
X_API_KEY=your_real_api_key_here
X_API_SECRET=your_real_secret_here
X_ACCESS_TOKEN=your_real_token_here
X_ACCESS_TOKEN_SECRET=your_real_token_secret_here
```

### 3-3. .gitignore の設定

そして、Git管理から外すために `.gitignore` ファイルを作り、最初の行にこう書く。

```text
# .gitignore
.env
__pycache__/
*.pyc
.DS_Store
```

これで、あなたが `git add .` をしても、キー情報はリポジトリに含まれない。
これはマナーではない。**生存戦略**だ。

## 第4章：Hello World（最初の接続）

### 4-1. Python環境の準備

言語は何でもいいが、本書ではエコシステムの広さから Python を推奨する。
ライブラリは、歴史と信頼のある `tweepy` を使う。

```bash
# 作業ディレクトリを作成
mkdir antigravity_bot
cd antigravity_bot

# 仮想環境の作成（推奨）
python -m venv venv
source venv/bin/activate  # Windowsなら venv\Scripts\activate

# ライブラリのインストール
pip install tweepy python-dotenv
```

### 4-2. 接続・投稿スクリプト (v2 API)

以下が、世界への第一声を上げるための最小構成コードだ。
`post_test.py` として保存せよ。

```python
import tweepy
import os
from dotenv import load_dotenv

# .env ファイルから環境変数を読み込む
load_dotenv()

# V2 Client の初期化
# Consumer Key/Secret と Access Token/Secret の4つを使う
client = tweepy.Client(
    consumer_key=os.getenv("X_API_KEY"),
    consumer_secret=os.getenv("X_API_SECRET"),
    access_token=os.getenv("X_ACCESS_TOKEN"),
    access_token_secret=os.getenv("X_ACCESS_TOKEN_SECRET")
)

def main():
    try:
        # Hello World の送信
        # 同じ内容は連続投稿できないため、テスト時は毎回内容を変えること
        text = "Hello, Antigravity Control Plane! #SystemTest v1.0"
        
        response = client.create_tweet(text=text)
        
        print("======== SUCCESS ========")
        print(f"Tweet ID: {response.data['id']}")
        print(f"Text: {response.data['text']}")
        print("=========================")
        
    except tweepy.TweepyException as e:
        print("======== ERROR ========")
        print(f"Failed to verify credentials or post tweet.")
        print(f"Detail: {e}")
        print("=======================")

if __name__ == "__main__":
    main()
```

### 4-3. 実行と確認（運命の瞬間）

ターミナルで実行する。

```bash
python post_test.py
```

`======== SUCCESS ========` と表示されたら、ブラウザで自分の X アカウントを見に行こう。
そこに機械的な文字で「Hello, Antigravity Control Plane!」と表示されていたら、勝利だ。

もしエラーが出たら？
慌てずにエラーメッセージを読むのだ。
*   `401 Unauthorized`: キーが間違っているか、環境変数が読み込めていない。
*   `403 Forbidden`: `Read and Write` 権限が付与されていない（設定変更後、Keysの再生成が必要な場合もある）。

### 意義
たった数行のスクリプトだが、この意味は重い。
あなたは今、スマートフォンのUIを経由せず、APIという「裏口」から世界に干渉した。
この瞬間から、X は「暇つぶしのSNS」から**「あなたがプログラム可能な出力デバイス」**へと変わったのだ。

## 第5章：この章の到達点（マイルストーン）

### この章のチェックリスト
*   [x] X Developer Portal で App を作成し、`Read and Write` 権限を付与した。
*   [x] 4つのキー（Consumer系、Access Token系）を取得した。
*   [x] `.env` と `.gitignore` を設定し、キー漏洩対策を行った。
*   [x] Pythonスクリプト経由で「Hello World」を投稿し、成功を確認した。
*   [x] テスト投稿を実際にブラウザで確認し、すぐに削除した（ゴミを残さない）。

インフラは整った。
次章、このパイプラインを使って、いよいよ運用を回し始める。
ただし、AIに全て任せるのはまだ早い。
「部長（あなた）と新入社員（AI）」の二人三脚モデルを構築し、事故らないフローを作る。
