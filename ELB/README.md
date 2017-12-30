# ELBについて

- 負荷分散
  - バックエンドインスタンス
    - リクエスト数/コネクション数が均等になるように負荷分散
  - ELB自体
    - ELBも負荷に増減して自動スケール
    - DNS名で登録すること

- ヘルスチェック
  - Pingプロトコル
  - Pingポート
  - Pingパス (/index.html)
  - タイムアウト時間
  - ヘルスチェック間隔
  - 異常判定までの回数
  - 正常判定までの回数
  
- 料金 
  - 重量課金 
  - リザーブドなし

- 独自ドメイン名で利用
  - Route53以外のDNSを利用する場合は `CNAME` で登録
  - Route53の場合は `Aレコードのエイリアス` で登録
  - Zone Apex (example.com) の場合は CNAMEは設定不可/エイリアスレコードを利用することで実現可能

- クライアントのIPアドレス
  - バックエンドサーバへの接続はソースIPアドレスがELBのIPアドレスとなる
    - アクセスログにはELBのIPアドレスが記録される
  - HTTP/HTTPSなら `X-Forwarded-For` を参照できる
 
- AZとバックエンドキャパシティの関係
  - リージョン内の複数AZ負荷分散可能
    - 複数リージョンへの分散はRoute53を利用
     
- ELB自体のスケーリング
  - 自動でスケールするがスケーリングが間に合わない場合は HTTP 503を返す
  - 事前に申請しておく。 `Pre-Warming (Bussiness/Enterpriseサポートが必要)`

- VPCでの利用
  - ELBを配置するサブネットをAZ毎に１つ指定
  - サブネットは最小 `/27` 8個以上の空きIPが必要

- ELBのSG
  - ICMPのSGを開ければPing応答する
  
- スティッキーセッション
  - 同じユーザからリクエストを全て同じEC2インスタンスに送信
  - デフォルトでは無効
  - HTTP/HTTPSでのみ利用可能
  - ELBは独自のセッションCookieを挿入
  
- スティッキーセッションの有効期間
  - アプリケーション制御
    - AppがCookie名を指定
    - Cookieの有効パスを合わせる
  - 期間ベース
    - 秒単位で指定
    - 無期限にすることも可能(ブラウザを閉じれば終了)

- Connection Draining
  - バックエンドのEC2インスタンスをELBから登録解除したり、ヘルスチェックが失敗したときに、新規割り降りを中止して、処理中のリクエストが終わるまで一定期間待つ
    - デフォルトで有効 タイムアウト 300s / 最大 3600s
  - Connection Draining
    - Management Consoleではインスタンスの表示が消える
    - ターゲットグループの状態が drainingに変化
    - API/SDK/CLIでは状況がわかる

- アクセスログのs3保管
  - ELBのアクセスログをs3に自動保管
  - 項目
    - timestamp
    - elb_status_code
    - backend_status_code
    - target_status_code
    - received_bytes
    - sent_bytes
    - request
    - user_agent
    - ssl_cipher
    - ssl_protocol
    
- ALBのリソース
  - ロードバランサ
  - リスナー LB側のポート
  - ターゲットグループ
  - ターゲット
  - ルール(/path)

- ALBの課金
  - 利用時間
    - LCU(Load Balancer Capacity Units) 以下の３つディメンションで使用率の高いもののみ請求
      - 新規接続数: 1秒あたりの新しく確率された接続数
      - アクティブ接続数: 1分あたりのアクティブ接続数
      - 帯域幅: ロードバランサで処理させたトラフィック量(Mbps)
