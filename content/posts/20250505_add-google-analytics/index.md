---
date: '2025-05-05T17:10:45+09:00'
draft: false
title: 'HugoにGoogle Analyticsを導入した'
showToc: true
categories:
  - tech
tags:
  - Hugo
  - Google
---

# はしがき

1つ前の記事でGoogle Analyticsを仕込みたいと言いましたが、早速導入しました。  

自分のGoogle Analyticsを導入する目的は、単純に閲覧数が知りたいってだけです。
おそらく他にも色んな情報が取れると思いますが、それらをどう利用するかは全く考えていません。  
（今後何かしらの参考にする可能性はあります）

# 導入方法

この記事で説明している内容のGitHub上のコミットは[こちら](https://github.com/tn0ha/tn0ha.github.io/commit/a3f775ee0ce95397128cb1fcf593dea5f9006194)を参照ください。  

Google Analytics 4を利用する前提で記載します。サイトに埋め込む測定IDは`G-XXXXXXXXXX`の形式のものになります。  
Google Analyticsから測定IDを取得するまでの手順は省略します。  

導入に当たって、セキュリティや誤使用防止の観点から測定IDをコードに直接書くのを避けています。
測定IDは、ビルド&デプロイのワークフローの手順の中で埋め込むようにしました。

## 追加した設定

今回、追加または更新したファイルは以下です。  
- `hugo.yaml`
- `layouts/_internal/google_analytics.html`
- `.github/workflows/deploy-hugo.yaml`

---
**hugo.yaml**

[Hugoの設定例](https://gohugo.io/configuration/services/)を参考に、Analyticsの設定を追加します。  

```yaml:hugo.yaml
services:
  googleAnalytics:
    id: __GA_MEASUREMENT_ID__
```

ここで、キー`id`の値に測定IDをベタ書き(ハードコード)せず、変数みたいな値を入れ込んでいます。使い方は後述します。

---
**layouts/_internal/google_analytics.html**

このファイルには、Google Analyticsで測定IDを発行した後に「このコードをあなたのサイトに埋め込んでください」と示されるコードを貼り付けます。  
そして、測定IDを埋め込む部分を`{{ site.Config.Services.GoogleAnalytics.ID }}`に置き換えます。
```html:google_analytics.html
<script async src="https://www.googletagmanager.com/gtag/js?id={{ site.Config.Services.GoogleAnalytics.ID }}"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag() {
    dataLayer.push(arguments);
  }
  gtag("js", new Date());

  gtag("config", "{{ site.Config.Services.GoogleAnalytics.ID }}");
</script>
```

`{{ site.Config.Services.GoogleAnalytics.ID }}`はHugoのサイト変数と呼ばれるものです。この変数はhugo.yamlに対応しています。
変数を使うことで、測定IDを色んな場所にハードコードしなくてもいいようにしています。

---
**.github/workflows/deploy-hugo.yaml**

GitHub Pagesへデプロイするためのワークフローに、測定IDを埋め込むための処理を加えています。  

追記した部分を抜粋:
```yaml:deploy-hugo.yaml
jobs:
  build:
    steps:
      ...
      - name: Replace Google Analytics Measurement ID
        run: |
          if [ -f hugo.yaml ]; then
            sed -i "s/__GA_MEASUREMENT_ID__/${{ vars.GA_MEASUREMENT_ID }}/g" hugo.yaml
          fi
      ...
```

ここで、先ほどhugo.yamlに入れ込んだ値`__GA_MEASUREMENT_ID__`を、GitHubリポジトリ側で設定してある変数に置き換えるようにしています。


## 必要ない設定

このサイトはHugo + PaperModで作られていますが、PaperMod側の設定にはHugo側の設定と一部オーバーラップするような項目があります。  
下記はPaperMod側の、Google Analyticsを導入する場合のhugo.yamlの設定例として[Wiki](https://github.com/adityatelange/hugo-PaperMod/wiki/Installation#sample-hugoyml)に載っているものですが、この設定は必要ありません。  
```yaml:hugo.yaml
params:
  analytics:
    google:
      SiteVerificationTag: "XYZabc"
    bing:
      SiteVerificationTag: "XYZabc"
    yandex:
      SiteVerificationTag: "XYZabc"
```

# 参考URL
導入にあたっては以下の記事を参考にしました。

- [How to Integrate Google Analytics Into Hugo PaperMod Theme](https://dzintars.dev/posts/how-to-integrate-google-analytics-into-hugo-papermod-theme/)

# おわりに

はしがきに書いた通り、Google Analyticsの導入目的は閲覧数の可視化です。それ以上のこと(ユーザの追跡など)をするつもりはありません。  
広告ブロッカーに真っ先にブロックされると思うので、どこまで可視化できるかは分からないところではあります。かくいう自分もChromeにuBlock入れてますしね。  
今回の導入によって、分析する側の気持ちも少しは分かればいいなーと思っています。
