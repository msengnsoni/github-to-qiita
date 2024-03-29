---
title: "Interop Tokyo 2023参加レポ"
topics: ["ポエム"]
published: false
---

InteropTokyo 2023に参加してきたので、そこでのセッション体験や次に活かしたいことをまとめていきます。

# Sky株式会社ブース

![IMG_3394.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/959e79df-2374-8d6c-c39e-c85cd2d761c5.jpeg)

- 製品、サービス紹介が主でセッションで紹介されていた製品は以下の通り
  - クライアント運用管理ソフトウェア「SKYSEA Client View」
  - シンクライアント システム 「SKYDIV Desktop Client」
  - 営業支援 名刺管理サービス 「SKYPCE」
- 所感
  - 名刺管理サービスについて、名刺を一度スキャンしたデータをSky側のクラウドに送信した後に情報が返ってくる仕様でだったが責任分界点が気になる
  - 提供事例クラウドプロバイダにAWS、GCP、Azure、OCI(Oracle)の内訳について話をしたが、9割AWSの模様(ほんとうか？)
    - 個人的には大きい企業ほどAWS以外を使ったり、マルチクラウド構成をとったりする印象だったので以外...
    - 単にSkyとしての受注する案件の特色か
- 戦利品
  - ステンレスボトル

# Zabbixブース (セッション登壇者：NTT communications)

![IMG_3396.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/fb29dc56-719f-1be6-f310-3a87095e5a47.jpeg)

- ZABICOM(Zabbix)の製品機能紹介が主
  - PortMap/RackMap/NetMapの導入事例が軽く紹介されていたのが製品としての実感を持たせてくれて印象的
- 所感
  - セッション資料がいらすとやが使われておりかなり簡素
  - 紹介されていた導入事例の顧客が市役所、教育機関、メーカーと、お堅いお客様が多そうなのはやはり製品の知名度か
- 戦利品
  - マグネットクリップ

# 日立製作所 & Microsoftブース (セッション登壇者：日立製作所)

![IMG_3403.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/0894a1e0-3014-cc56-e8ba-8d84f8e80411.jpeg)

- 日立HCIソリューション for Azure Stack HCIの促進販売が主
  - HCIとは
    - Hyper-Converged Infrastructureの略語で、SANやNASなどの外部ストレージを使用せずに一般的なx86サーバーを統合してサーバーの仮想化を実現するアプライアンス製品のこと
  - Azure Stack HCIとは
    - 検証済みハードウェアにAzure Stack HCI OSをインストールし、オンプレミス上にパブリッククラウドであるAzureと連携可能な仮想環境を実現するHCIソリューション
  - 日立製作所としての強みを活かすという点で日立サポート360を推していたことが印象的
- 所感
  - ハイブリッドクラウドは既にオンプレミス環境を持つ企業にとって最も現実的な選択肢であると感じているし採用している企業も多い、日立が推しているということもありSIer業界で今後採用する現場が増えていきそう
  - クラウドのみならずシステム全体をスコープに障害や相談事をまとめて受け付けるというのがいかにも日立らしい(なんでも屋みたいな)
- 戦利品
  - マルチポーチ
  - ポータブル防災7点セット
  - サーモボトル

# テクマトリックス

![IMG_3407.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/6bf80806-9880-5a74-4096-0a8393c4be0e.jpeg)

- NEOの紹介が主
  - 価格も込で紹介していたことが印象的、より導入までのイメージを持ちやすいので良いと思う
- 所感
  - クラウドネイティブ1day入門道場の値段も記載しており、interop来場者限定でタダでこの講義を受けられると紹介していたので、今後の講義の際大変になりそう...
  - テクマトリックスの提供サービスとしては、Appgate SDPが人気？NEOの前にセッションをしていたが聞いている人数が多かった印象

# Spirent Communications Japan

![IMG_3421.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/f7ddd9c6-afdd-32de-61d5-38cef7a4e746.jpeg)

- 事前登録制の「クラウド試験ソリューションとデジタルツイン」講演に参加
  - Spirent Communications Japanの顧客内では数年前よりクラウドネイティブという言葉は日本の業界内で認知はされてはいたが、今年から本格的に動き出している企業が多いそう
    - これまであまり普及しなかったのはネットワークベンダーの責任もあると少しぼやいていた
  - メリット
    - コスト
      - k8sのマスターノードを自前で持ちたいという企業が多かったが、共通化することによってコスト上安くなる
    - 調達
      - 主要な機器は自前で持たなくなるため調達コストが軽減
  - デメリット
    - 組み合わせの複雑化
      - 様々なツールが入り乱れるため学習コストや管理コストがかかる
    - 責任分界点の不明瞭化
      - 自前で管理しない以上どうしても問題になる部分、プロバイダが提示する責任モデルを許容できるかが問題
    - 測定種類の増加と対応
      - 監視対象の増加とそれに伴う対応コストの増加
    - 個々のコンポーネントのアップデートサイクルが早い
      - アップデートをする毎にシステム全体で総合テストすることは現実的に不可能
  - Spirent Communications Japan及び東陽テクニカではこれらの課題を解決するために以下4つの考え方を重要視しクラウドネイティブ関連ソリューションは提供、開発している
    - テスト及びレポート作成自動化
      - テストだけではなく、それを評価するアプトプットを出すところまで自動化しなければ結局作業コストはかかってしまう
    - カオスエンジニアリング
      - 様々なコンポーネントが存在するシステム内で、何かが発生したら何が起きるのかを把握しておくことが重要
    - 多様な測定項目への対応
      - 監視対象が増加することに対応しなければ運用コストが結局増加してしまう
    - デジタルツインラボ
      - デジタルツインの導入により、システム運用のコストを削減
  - 所感
    - クラウドネイティブ×デジタルツインというキーワードに期待したが、デジタルツインに関しては説明のみに留まったのでそこだけは少し残念
    - 実際にソリューションや製品を開発する他社のクラウドネイティブに対するアプローチは勉強になることが多い

# その他 (O'Reilly)

![IMG_3395.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/554835/d662067b-1628-0087-7289-a0bd4ddf646f.jpeg)

- オライリー本が会場限定で20%オフだったので2冊購入、購入特典にTシャツがあったみたいですが品切れてしまったようで残念...

# 総括

- 業界(主にネットワーク・セキュリティ分野)の最新動向を肌で感じることができてとても貴重な経験になった
- 勉強及び企業分析的な視点でしか各ブースを見て回ることができなかったので、今後参加する際はビジネス的な視点も持ち合わせたい
- 本イベントの目玉として、ShowNetが構築した会場内ネットワークがあったのだが、次回はShowNetのツアーに参加してネットワーク技術にもアンテナを伸ばしたい
- もっともっと貪欲に各ブースで製品デモを触れば良かった、促販的な雰囲気から遠慮がちになってしまったのがもったいない
