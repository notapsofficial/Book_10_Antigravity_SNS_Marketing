# 第一章：門の鍵を開ける（X Developer Portalの歩き方）

「自動化」という言葉には甘美な響きがありますが、その入り口には常に無機質で冷徹な「門番」が立っています。それが**API（Application Programming Interface）**です。

多くの人がここで挫折します。英語のドキュメントや複雑な設定画面に心を折られてしまうのです。

しかし、安心してください。この本は、私が泥臭く躓きながら構築した「SNS投稿パイプライン」を、あなたが再現できるようにするための「手順書」です。まずは、最も開発者フレンドリー（とはいえ癖はありますが）な **X (旧Twitter)** から攻略していきましょう。

## なぜ、Xから始めるのか？

InstagramやTikTokは画像や動画が必須で、審査も厳格です。一方、Xは「テキスト」が主体。APIの構造も比較的素直です。「プログラムから文字を送って、投稿させる」という基本動作を学ぶのに、これほど適した教材はありません。

## 手順1：Developer Portalへの登録

まずは [X Developer Portal](https://developer.twitter.com/en/portal/dashboard) にアクセスし、開発者アカウントを申請します。（情報は2025年12月現在のものです）

> [!NOTE]
> すでにXのアカウントを持っている前提で進めます。電話認証が済んでいない場合は、先に済ませておく必要があります。

Freeプラン（無料）で十分です。月間1,500ツイートまで投稿できます。個人の自動化システムとしては十分すぎる量です。
「どのような目的でAPIを使用するか？」という英語の質問攻めにあうかもしれませんが、正直に「個人的な学習と、自分のアカウントへの投稿自動化のため (For personal learning and automating posts to my own account)」と答えれば問題ありません。ChatGPTに翻訳させて、200文字程度の作文をして貼り付けましょう。

## 手順2：ProjectとAppの作成

登録が完了すると、Dashboardに入れます。ここで混乱するのが「Project」と「App」の概念です。

- **Project**: 大きな箱。
- **App**: その箱に入っている実際の「プログラム」の身分証明書。

1つのProjectの中にAppを作ります。名前は何でも構いません（例: `MyAutoPoster`）。

## 手順3：【重要】User Authentication Settings（最大の詰まりポイント1）

Appを作った直後に、絶対にやっておかなければならない設定があります。これを忘れると、後でコードが完璧でも「権限がありません（403 Forbidden）」というエラーで弾かれ、数時間を無駄にします。私はしました。

1. AppのSettings画面（歯車アイコン）を開く。
2. **User authentication settings** の 「Edit」を押す。
3. **App permissions** の項目を見る。

> [!WARNING]
> **デフォルトでは「Read only（読み取り専用）」になっています！**
> これでは投稿ができません。必ず **「Read and Write」** に変更してください。

4. **Type of App** は 「Web App, Automated App or Bot」 を選択。
5. **Callback URI / Redirect URL** は、ウェブアプリを作らないなら適当で構いませんが、入力必須です。 `http://localhost:3000` と入力しておきましょう。
6. **Website URL** も必須です。自分のブログや、なければXのプロフィールURLを入れておけばOKです。

保存を押すと、Client IDなどが表示されますが、これは一旦無視して構いません（Webアプリを作る場合に使うものです）。

## 手順4：Keys & Tokensの取得（最大の詰まりポイント2）

ここが一番の「わけがわからない」ポイントです。
「Keys and tokens」タブを開くと、似たような文字列が大量に並んでいます。
パイプライン構築に必要なのは、以下の**4つ**です。これ以外は忘れてください。

### 1. Consumer Keys (API Key and Secret)
これは「**あなたの作ったアプリ（プログラム）のIDパスワード**」です。
- API Key (IDのようなもの)
- API Key Secret (パスワードのようなもの)

### 2. Authentication Tokens (Access Token and Secret)
これは「**あなた自身（ユーザー）の身分証明書**」です。アプリが「あなたに代わって」投稿するために必要です。

> [!IMPORTANT]
> **Generate** ボタンを押して生成します。
> この時、一度しか表示されないので、必ずメモ帳やパスワード管理ツールにコピーしてください。
> もし「Read and Write」権限を設定する前にこのTokenを発行してしまっていた場合、そのTokenは「Read only」のままです。**権限設定を変えたら、必ずTokenを再生成（Regenerate）してください。** これも私がハマった罠です。

### 最終確認：手元にあるべき4つの鍵

手元のメモ帳に、以下の4行が揃っていることを確認してください。

1. **API Key** (Consumer Key)
2. **API Key Secret** (Consumer Secret)
3. **Access Token**
4. **Access Token Secret**

この4つがあれば、門は開きます。次は、これらを使って実際にPythonから「Hello World」を叫ばせてみましょう。
