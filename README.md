
# 新しい投稿の作り方

**_posts** ディレクトリの中に、新しいファイルを作成します。

### ファイル名の規則

    yyyy-MM-dd-タイトル.md


### ファイルの中身

    ---
    layout: blog    <- 固定
    title: テストタイトル <- タイトルを入力
    tags: [rails,mysql] <- タグを指定
    category: blog <- ルートが変更できます
    summary: ここに概要が入ります <- リスト表示で概要として表示されます
    image: /images/blog/image.png <- サムネイルが必要な場合は入力
    author: aharada <- 投稿者を入力(後述の投稿者設定と同じものを入れる)
    ---

### 投稿者の設定

/_config.yml に設定があります。
プロフィールを追加、修正する場合はこちらに設定してください。
