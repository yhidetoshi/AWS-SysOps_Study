# Quizめも

- RDSを削除すると
  - 自動スナップショットが削除される
  - 手動スナップショットは保持される
  
- 上限
  - VPCサブネットのデフォルト最大数: 200
  - アカウントあたりのグループ数: 100
  - インスタンスプロフィアル: 100
  - IAMロール:250
  - IAMグループ所属が: 10
  - サーバ証明書: 20
  - ユーザ5000
 
- ボリュームステータスチェックは5分ごとに成功か失敗のステータスを返す

- VM Import/Exportに未対応なディストイリビューション
  - 32bit
  - Redhat6.0

- NATインスタンスを利用する場合は SrcDestCheck(送信元/送信先チェックを無効)にする

- s3について
  - オブジェクトの最大サイズは5TB
  - イレブン9は堅牢性、可用性は99.99%
  
- ブロックデバイスで 
  - `/dev/sda` : ルート用
  - `/dev/sd[b-e]` : インスタンスストア
  - `/dev/sd[f-p]` : EBSボリューム
  
- RDS
  - RDSのストレージ最大サイズは6TB
  - SQLサーバは4TB
  - RDSの自動バックアップは5分ごとにトランザクションログを取得している
  - Multi-AZの場合、スタンバイ側はリード処理はできない。必要な場合はリードレプリカを使う
  - シングルAZの場合はスナップショットの取得中のI/Oは数分以内で中断する
  - Multi-AZだとI/O中断の影響は受けない
  - SSLをもちいてアプリケーションとDBインスタンスの間を暗号化接続できる
  - 静的パラメータを変更する場合はDBインスタンスの再起動が必要
  - Backup Retension Periodを0にすると、自動バックアップが無効になる. リードレプリカの作成と時刻指定のリストアのボタン自体も無効になる
  - MySQLのソースDBを消すとリードレプリカは削除されず、そのままリードだけ可能になる
  - RDS for PostgresSQLのDBインスタンスを削除するとリードレプリカが昇格して読み書きが可能になる
  - RDSのバックアップ保持期間は最大35日まで（デフォルト7日間）
  - SQL Serverはmasterデータベースにログインとパスワードを格納する
  - Multi-AZでフェイルおーばが発生すると、正規名レコード(CNAME)をプライマリからセカンダリに変更される
  - フェイルオーバの時間は 60s -120s
  - DBセキュリティグループのイベント通知を利用できる
  - DNSを使用してシームレスな移行のためにスタンバイレプリカに切り替える
  - Multi-AZでメンテナンスを実行されると
    - 1. スタンバイに対してメンテナンスを実行する
    - 2. スタンバイをプライマリに昇格させる
    - 3. 旧プライマリでメンテナンスを実行し、その旧プライマリが新しいスタンバイになる
  - Multi-AZの同期について
    - MySQL/MariaDB/Oracle/PostgresSQLは物理同期しスタンバイ側とプライマリと同期する
    - SQL ServerエンジンのマルチAZは同期論理レプリケーションを使用して SQL Serverネイティブのミラーリングテクノロジーを採用
  - セキュリティパッチをあてるときはDBインスタンスを強制的にオフラインになる
  
- S3
  - 3箇所以上のデータセンターで同期している
  - 静的コンテンツに耐久性のあるストレージ
  - 新しいオブジェクトのPUTは「書き込み後の読み込み」整合性
  - 米国スタンダードリージョンのS3バケットのみ結果整合性
  - PUTおよびDELETEの上書きについては結果整合性
  - マルチアップロードではあれば、アップロードのサスペンド・レジュームは可能(通常の方法では不可)
  - オブジェクトの最大は5TB
  - 100MB以上のファイルはマルチアップロードが推奨
  - 重要なファイルの削除ミスを防ぐには、バージョニング管理とMFAを有効にする
  - マルチアップロードでスループットが改善される
  - ACLで、バケットとオブジェクトへのアクセスを管理ができる。
  - 標準/標準-IAはリージョンの複数の施設にまたがる複数デバイスで保管される
  - 同じオブジェクトの異なるバージョンに対して異なる暗号化キーを使用できる
  - ユーザ側で用意した暗号化キーでのサーバー側の暗号化(SSE-C)を使用すると独自の暗号化キーを設定できる。
  - ユーザがAWSが提供しているAES-256暗号化キーを使用しない場合、各API呼び出しでキーと暗号化アルゴリズムを送信する必要がある
  - ユーザが用意した暗号化キーで暗号化するとデータの復号化の鍵を管理する必要はない
  - S3のPUTのパフォーマンスを出すためには、キー名にランダムはプレフィックスを追加する
    - タイムスタンプやアルファベット順などの連続するプレフィックスを使用するとS3が大量のキーに対して特定のパーティションを対象にするため、そのパーティションのI/O能力が逼迫する可能性いが増大する。
    - キー名のプレフィックスをランダムにすると、キー名、したがってI/Oロードは複数のパーティションに分散される
  - 他のAWSアカウントに許可を与えるためにACLにはAWSアカウントの正規ユーザIDを通じて所有者を識別するOwner要素が含まれている
  - オブジェクトとバケットの両方にACLを設定できる。フォルダにはACLを設定できない
  

- StorageGateway  
  - ボリュームゲートウェイは、オンプレミスのアプリケーションサーバからiSCSIデバイスとしてマウントできる、クラウドベースのストレージボリュームを提供
  - キャッシュ型ボリューム
    - データをS3に保存し、頻繁にアクセスするデータサブセットのコピーをローカルに保持する。
    - プライマリーストレージのコストを大幅に削減しストレージをオンプレミスで拡張する必要を最小現に押させる
    - 頻繁にアクセスするデータへのアクセスを低レイテンシーに保つことができる
  - 保管型ボリューム
    - データセット全体への低レンテンシーアクセスが必要な場合に可能なオンプレミスゲートウェイ設定
    - すべてのデータをローカルに保存し、そのデータのポイントインタイムスナップショットをS3に非同期でバックアップする
    - 耐久性が高く低コストのオフサイトバックアップを提供する
    - 障害復旧のための代替容量が必要な場合はEC2にバックアップを復元できる
  

- AWSの規約
  - AWS側の瑕疵でもデータ漏洩したら、ユーザの責任
  - AWSがサービス撤退は30日以内にユーザ通知すればよい
  - AWSはいつでも利用規約を変更できる
  
- EBS
  - ボリュームのために利用可能だが推奨しないのは, RAID5. RAID6
  - プレイスメントグループを削除する場合、インスタンスを終了する必要がある
  - EBSボリューム数の制限は5,000 スナップショット数は10,000
  - AES-256 インスタンスホスト上で暗号化処理される
  - 作成済みのボリュームは暗号化できない。作成時に暗号化設定をする
  - Windowsベールのinstance-store-backed AMIはEBS-Backed AMIに変換できない
  - プロビジョンドIOPSボリュームの予想IOPSまたはスループットがアプリケーションで実現できていない場合、10Gbネットワーク接続されていないEC2帯域幅が制限要因であるか、インスタンスがEBSに最適化されていないか、存在しないか。ボリュームのサイズを大きくしてもIOPSには影響しない。
  - ボリュームに対するプロビジョンドIOPSは1GiBあたり最大50IOPSでプロビジョニングできる
  - 400GiB以上のボリュームで最大20,000IOPSまでプロビジョニングできる
  - スナップショットを効率的に実施するには、 ディスクI/Oの停止→ EBSスナップショットを開始→ ディスクI/Oの再開


- SNS
  - 通知のsubscriber: Lambda/SQS/HTTP(s)/Email/SMS

- CloudTrail
  - S3に保存されるログファイルは S3 Server Side Encryption(SSE)で暗号化される
  - CloudTrailはリージョン毎に有効
  - CloudWatch Logsに連携することも可能

- Billing
  - 請求アラームを作成するには請求アラートを有効にする
  - 連結アカウントに属するためには、一括請求管理している管理者に要求する
  - 一括請求のメリット
    - ひとつの請求ですむ
    - 簡単なトラッキング: CSVでコストデータをダウンロード可能
    - 使用量の統合: 複数のアカウントの使用量が合計される.ボリューム割り引きで料金を削減できる可能性がある
  - バージニア北部でメトリクスが取得できる（US-EAST-1） 含まれるのは全リージョンの金額
  - Dev/Prdのように環境毎にコストを把握するには、コスト配分タグとコスト配分レポートを使用する
  - 支払いアカウント(一括請求する側)はリンクされたアカウントのAWS請求の詳細を表示できる

- ElasticCache/Memchached
  - Evictions: 新しく書き込むための領域を確保するためにキャッシュが排除した期限切れでない項目の数
  - GetMisses: キャッシュが受信したが、リクエストされたキーが見つからなかった数

- EC2Config Service
  - EBSボリューム、インスタンスストアボリュームをフォーマットおよび、マウントしボリューム名をドライブ文字にマップすることができる

- OpsWorks
  - Chefレシピを使用して、アプリケーションをホストするアプリケーションサーバを構成することができる。
  - OpsWorks for Chef Automateには完全マネージド型のChefサーバと継続的なデプロイのためのワークフロー自動化やコンプライアンスとセキュリティのための自動化を可能したもの

- DynammoDBは
  - 保存するデータは NULL/コレクションデータ型/スカラーデータ型(数値・文字列・バイナリ・ブール
  - webセッションの管理。JSONドキュメントの保存/S3オブジェクトのメタデータの保存
  - 一般的な用途: モバイルアプリケーション・ゲーム・デジタル広告サービス・ライブイベント・eコマースのショッピングカート・ウェブセッション管理

- CloudWatch
  - SSDの場合は5分間隔
  - Provisioned IOPSは1分間隔で自動送信する
  - regionをまたがってデータを収集できない
  - Cloudwatchへのメトリクスをputする場合はIAM-Roleを使う
  - CloudWatchアラームにアクションでインスタンスを停止・終了させることができる
  - SNSと連携すると、自動スケーリングを設定すると特定のアカウントでインスタンスが多すぎると通知できる
  - 統計: Min/Max/Sum/Ave/SampleCount
  - ユーザがキャプチャしたデータをCloudwatchのカスタムメトリクスをサポートしている.(CLI or APIで可能)
  - CLIを利用してアラーム状態をアラームに設定し、SNSで通知テストを実施できる
  - サポートされるログは、テキストベースの一般的なログデータ形式・JSON形式であればOK
  - 取り込み、集計、モニタリングが可能で最小は１分。タイムゾーンの指定は必須ではない
  - メトリクスの保持期間は14日間から15ヶ月間に保存できるようになった
  - カスタムメトリクスのデータアップロードには約２分かかる。約15分後にのみ見えることができる
  - グラフをAbsoluteタブで開始点・終了点を指定できる
  - ステータス INSUFFICIENT_DATA: アラーム開始直後・メトリクスが利用できないかデータ不足
  - ASGのNameSpace
    - GroupMinSize
    - GroupMaxSize
    - GroupDesiredCapacity
    - GroupInServeiceInstances
    - GroupPendingInstances
    - GroupStanbyInsances
    - GroupTermminatingInstances
    - GroupTotalInstanaces

- EC2
  - EC2のタグについてはkeyが127文字、値が255文字
  - インスタンスストア領域は再起動では消えない。停止すると消える
  - システム状態がimpairedの場合はホストが壊れている可能性がある。対処としてインスタンスを停止して開始する。
  - 再起動では別のホストに移動せずに起動する。停止→起動だと別のホスト上で起動する
  - EC2へのsshでタイムアウトする場合は: SGで許可されていない/秘密鍵が正しくない/ログインユーザ名が違う/CPU使用率が高い
  - インスタンス使用率が70%以上の場合はRIのほうが得
  
- Route53
  - デフォルトのヘルスチェク間隔は30s
  - 加重ルーティング/レイテンシーベースルーティング
  - zone apexはRoute53のAliasレコードを使用する
  - エイリアスレコードはあるDNS名を別のAmazon Route53 DNS名にマッピングできる
    - CloudFrontディストリビューション/Elasitc Beacnstalk/ELB/静的WebホスティングS3
    - 有効期限(TTL)を設定できない
  - CNAMEレコードはどこにでもホストされている任意のDNSレコードを指すことができる
  - Route53のレコードセットはリージョンに依存しなく、グローバルで利用できる

- 無料枠
  - EC2: 750時間 t2.micro
  - RDS: 750時間
  - S3: 5GB

- AS
  - 終了保護をつけてASGのインスタンスの終了保護を防止できない
  - 終了ポリシーで最初に終了させるインスタンスを指定することができる
  - 最後のインスタンスが起動するとクールダウン期間が有効になる
  - ユーザアクション中はスケールアウトが中断される
  - スポットインスタンスの場合は入札が完了してから有効になる
  - スケジュールのアクションは、一位の時間値が必要で、他のスケジューリングと競合するとエラーメッセージが表示される
  - マネジメントコンソールから起動設定を作成した場合に基本モニタリングが有効
  - CLIまたはAPIを使用して起動設定を作成した場合に詳細モニタリングが有効になる(ASの場合の詳細モニタリングは追加料金なし 7種のメトリクスだけ)
  - AutoScaling変更直後は Desired Capacityの数だけインスタンスが作成される。(Desired Capacityがなければ最小で起動する)
  - ASGを削除する場合は desired capacityとminを0にしてterminateしてから削除する
  - スケジューリングを作成する場合、開始時間・終了時間。グループ名が必須で、[Min/Max/Desired Capacity]のうちひとつが必須
 
- Glacier
  - 1ファイル40TBまで
  
- Elasitc Transcoder
  - トランスコーディング（パイプライン）の数は各ユーザで制限されている
   
- SQS
  - AppがEC2とSQS間で十分な帯域保証するには、EC2インスタンスをスケーラブルにする必要がある
  - 4日間メッセージを保持する。最大で14日間保持できる
  - ユーザは手動でキューを削除することができる
  - FIFOは保証できない
  - 水平スケーリングが容易
  - 非同期処理

- AWS側のセキュリティ
  - DDos
  - MITM
  - IPスプーフィング
  - ポートスキャン
  - パケットスニッフィング

- ELB
  - スティッキーセッション: ユーザのセッションを特定のアプリケーションインスタンスにバインドする。
    - ユーザのセッション中のすべてのリクエストが同じインスタンスに送信される
  - webソケットなどの長いポーリングを使用していると同じバックエンドインスタンスに接続することになる
  - connection Draining: 1秒から3600秒で設定可能
  - Pre-Warmingでスケールする. 間に合わないと503を返す
  - connection Draining: 1秒から
  - ログ間隔は5分 or 60分で指定できる
  - バケット名/Prefix/awsアカウントID/region/タイムスタンプ/ロードバランサ名/end-time/ELB-IP/random-stringが含まれる

- CloudFormation
  - WaitConditionリソースを使用すると設定プロセスのステータスを追跡できる
  - スタックの作成が失敗したら、作成したリソースを削除してロールバックする
  - リソースに重点をおいて自動化する(S3からダウンロードやELBのセットアップなど)
  - テンプレートの一部として必須のコンポーネントはResource
  - アプリケーションモデリング・デプロイメント・構成・管理・および関連するアクティビティを提供するアプリケーション管理ツール
    - テンプレート: アプリケーションのデプロイおよび、実行に必要なすべてをAWSリソースに記述するJSON形式のテキストファイル
    - スタック: AWSCloudFormationがテンプレートをインスタンス化するときに単一のユニットとして作成および管理されるAWSリソースコレクション
    - スタック作成中にテンプレートをアップロードして必要に応じてパラメータのデータを提供する
 
- IAM
  - IAMポリシー/ロールはリージョンに依存しない。グローバルで利用できる
  - GovCloud(米国)中国(北京)の￥はリージョンのみで使用できる


- SQSのBlackBeltを確認する
