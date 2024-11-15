# 1章
- webサービスを高速化するには必要十分なキャパシティを揃える必要がある
- パフォーマンスチューニングのきほんのき
  - いきなり手を動かさない
  - 推測せず計測する
  - 公平に計測する
    - 前提条件を揃える
  - 一つずつ計測する
    - いわゆる対象実験をしてボトルネックを確実に絞り込む
- パフォーマンスチューニングのきほんのほ
  - ボトルネックにだけアプローチする
    - それ以外にアプローチしてもあまり効果はないしなんなら悪化する可能性もある
  - ボトルネックの特定は外側から順番に
    - どんどん範囲を狭めて原因を特定していく
      - ex)mySQLに原因がある->悪いクエリを探す->改善する
- ボトルネック対処の基本3パターン
  - 解決：課題になっている事象を根本から解決する 
  - 回避：課題になっている事象がボトルネックにならないよう迂回・省略する 
  - 緩和：課題になっている事象の影響を和らげる
- パフォーマンスチューニングのきほんのん
  - 負荷試験を何度も行って試行錯誤する
  - 試験計画をきちんと立てる
  - 負荷をかけながら手動でも利用してみて使用感を確かめると良い 
  - 実施時間・実施結果・メトリクス・ログをセットで自動的に記録しておくと良い
  - 実施結果の内容を都度解釈する 
    - パフォーマンス：X並列でYユーザーがN分間で操作完了 
    - 異常の有無：エラーレスポンス、システムエラー、不審な挙動、不安定なレスポンスタイム ・ボトルネックは移動したか 
    - それぞれの値、リソースメトリクスの値が想定通りに変化したか 
  - 与負荷環境側のメトリクスも同時に確認する（与負荷側がボトルネックになり、十分な負荷が生成できないケースもままある）


# 2章 モニタリング
- モニタリングは2種類
  - 外形監視
    - アプリの外側からモニタリングすること
    - Synthetic Montoringともいわれる
    - できるだけユーザーの近くから行うことネットワーク的な接続トラブルが発見できるという利点がある
      - 多くの場合ではユーザーが不特定多数であることから完全な再現は現実的ではないため、ある程度コストと天秤にかけて決定
  - 内部監視
- モニタリングツールのアーキテクチャ
  - プル型
    - モニタリングアプリケーションが対象アプリ内のエージェントへメトリクスを取得するアーキテクチャ
    - メトリクスの取得間隔をモニタリングアプリケーション側から管理できたり、エージェント側の実装をシンプルにできる
  - プッシュ型
    - アプリ内部のエージェントがモニタリングアプリケーションへメトリクスを送信するアーキテクチャ
    - エージェントが動作しているサーバーにおけるポートに対する接続を許可する必要がない
    - モニタリングアプリケーション側の設定を変更せずともモニタリング対象の増減が可能になる
- モニタリングの注意点
  - 正しい計測結果の見極め
  - 2つのグラフを比較するときは他の条件を合わせる
    - よくあるミスとして、異なる原点を定めたグラフを比較するということがある
  - 高負荷状態のモニタリング
    - 負荷試験などで計測する際には負荷試験のために発生する負荷についても考慮が必要
    - 負荷試験を行う環境は実際に本番環境で運用する環境にできるだけ近づけるべき
  - メトリクスの取得感覚は適切なものを設定すること
    - 解像度が低い、足りていない状態では細かな負荷の変化を見つけられない
    - できるだけリアルタイムに解像度が高くなるようにしておくことで、隠れていた異常を見つけることが可能

# 3章 基礎的な負荷試験
- パフォーマンスチューニングにおいては、「負荷試験の実行と負荷の観察」→「観察結果に基づいたチューニング」→「再度の負荷試験で実施したチューニングが有効かどうか確認する」というサイクルを回していく必要がある
- 並列度を上げていくことによってうまく使えていないリソースを発見できる
- 一般的に、1プロセスで1リクエストを処理するアーキテクチャの場合、workerプロセスはCPU数より大きく（典型的にはCPU数の数倍に）するのが一般的
- 動作するプロセスが増えればそれだけメモリも消費し、CPUの割り込みやコンテキストスイッチと呼ばれる処理も増加する

# 4章 シナリオをもった負荷試験
- 実際にユーザーが行うシナリオをモニタリングすることでもボトルネックは発見できる

# 5章 データベースのチューニング
- DBのプロセスを確認する
  - 何度も表示されるクエリはWebアプリケーションから数多く実行されているか、1回の実行時間が長く表示されやすいか、もしくはその両方で負荷が高いクエリである可能性がある
- 実行されたクエリを集めて解析するには、スロークエリログがよく利用される
  - 設定した閾値より実行にかかった時間が長くなったクエリを、かかった秒数や処理した行数とともに出力したログ
- プライマリインデックス
  - いわゆる主キーで作られるインデックス
  - ユニークな値で各エンティティに一つだけ
  - プライマリインデックスの先にデータを用意するクラスターインデックスがある
- セカンダリインデックス
  - 任意で作るインデックス
  - 主キーが葉にあり、その値でプライマリインデックスを検索する
- Covering Index
  - セカンダリインデックスでもcountなどidがわかればいいという理論の最適化
- インデックスはソート済みのもう1つのデータベースになる
  - インデックスの作成、データの追加・更新があった場合のインデックス更新には、オーバーヘッドが伴う
  - インデックスの数が増えれば、その負荷は増えていき、データ更新時に負荷が高まったり、速度が落ちたりする原因となる。
- like等の検索には全文検索インデックスを用いるといい
- 空間インデックスもある(詳しく載ってないのであとから探す)
- N+1問題の対策
  - joinを用いる
  - キャッシュを用いた回避も可能
    - NoSQL(memcachedやRedisなど)
    - メモ化再帰とかと感覚は近い
  - 別クエリによるプリロードを用いたN+1の解決
    - in句を用いて回数を抑える
    - IN句に渡す値の数が多過ぎると、クエリのサイズが大きくなり過ぎてエラーになったり、狙ったインデックスが使われなかったりする
  - 正規化をわざと崩す
    - JOINして取得したい情報をあらかじめテーブルにも格納しておくことでシンプルなクエリのままN+1問題を解消する
    - 長期に利用するWebサービスではデータが冗長になり、更新時のコストが高くなるリスクもありますが、高速化の目的がはっきりしている場面では利用可能な手法
  - N+1問題はデータベースだけに限った話ではない
    - 外部サービスにアクセスして情報を取得する場合にも発生する
    - Webサービスのパフォーマンスに影響するようであれば、必要な情報をバルクで取得するAPIを用意し、活用するなどの対策が必要
- MySQLのバージョンやテーブルに作成されているインデックスによっては、開発者の想定と異なる実行計画が取られることがある
  - FORCE INDEXとSTRAIGHT_JOINを用いてヒントを与えることによって解決できる
  - FORCE INDEX
    - インデックスを強制指定するキーワード
  - STRAIGHT_JOIN
    - クエリを上から順にできるキーワード
- SELECTで必要なカラムを指定することでもパフォーマンスが改善する
  - 画像とかは特に
- プリペアドステートメント
  - DBのクエリにおけるキャッシュのようなもの
  - あくまでも取得結果ではなくクエリに対するキャッシュであることに注意
- DBには最大同時接続数が設定されている
  - ここもチューニングの対象
  - 永続的に接続した方がいい?

# 6章 リバースプロキシの利用
- リバースプロシキの主な役割は以下の三つ
  - 負荷分散（ロードバランス）
  - コンテンツのキャッシュ
  - HTTPS通信の終端
- パフォーマンスに影響しやすいものとしては以下がある
  - 転送時のデータ圧縮
  - リクエストとレスポンスのバッファリング
  - リバースプロキシとアップストリームサーバーのコネクション管理
- プロセス・スレッドに関するアーキテクチャ
  - マルチプロセス・シングルスレッド
    - クライアントからの1リクエストを1プロセスが処理を行っている
    - そのプロセスは処理を行っている間に他のリクエストを処理できまない
    - そのため、このアーキテクチャではプロセス数と同時に処理できるリクエスト数が一致す
    - プロセス数が同時に扱えるリクエスト数の上限であっても、大量のリクエストが来ない限りは十分なパフォーマンスを出せる
    - 同時に大量のリクエストが来る場合はC10K問題が起きる
      - アクセスするクライアント数が1万を超えると、サーバーのスレッド（並列処理の単位）数が増え、サーバーのメモリーなどのリソースが不足してしまう問題
  - シングルプロセス・マルチスレッド
    - 同じプロセス上のメモリ空間をスレッド同士が共有できるため、使用するメモリが少なく済む
    - しかし、スレッドを切り替える時も先程と同様にコンテキストスイッチが発生するの
    - 1スレッドがレスポンスを返すまで占有されるアーキテクチャにすると先程と同様にC10K問題が発生
- アプリケーションサーバーの前にnginxなど他のアーキテクチャで作られたリバースプロキシを前段に置く構成のメリット
  - 遅いクライアントとの通信でアプリケーションサーバーのプロセスが専有されなくなります。
    - 回線が細いなどの理由で遅いクライアントにレスポンスを返す際もリバースプロキシがレスポンスを返してくれるため、
  - 画像・CSS・JavaScriptをリバースプロキシで直接静的ファイルを返した方がパフォーマンスは上がります
    - アプリケーション側で操作が不要なため
- nginx
  - リバースプロキシとして利用される代表的なソフトウェア
    - nginxでは設定ファイルを書くことでリバースプロキシとしてアップストリームに指定したアプリケーションサーバーにリクエストを送ったり、直接静的ファイルを配信できる
    - Mainline versionとStable versionの2種類のバージョンが用意されている
    - nginxは1つのサーバーで複数の設定が異なるHTTPサーバーをそれぞれ動作させることができる
      - ドメインを運用する技術であるバーチャルホストに対応しているため、
  - アーキテクチャ
    - マルチプロセス・シングルスレッド
    - イベント駆動のアーキテクチャを採用している
      - 各プロセスが複数のクライアントからのリクエスト・レスポンスを並行して扱うことができる
    - 多重I/OやノンブロッキングI/Oを用いている
      - ノンブロッキングI/O:通信を待っている際にリソースをブロックしない
  - gzipに対応したHTTPクライアントからのリクエストに対して、gzipを使用して圧縮したレスポンスを返すことができる
    - 圧縮レベルを上げると圧縮度を上げられる
      - 圧縮処理に時間がかかるのでトレードオフである
    - gzip圧縮する場所も重要
      - プロシキサーバだけでなくアプリケーションサーバでも圧縮を検討する
  - アップストリームサーバーのコネクション管理
    - キープアライブを利用することで、アップストリームサーバーへの接続処理を減らすことができる
      - コネクションを保持して使い回すこと
      - 大量のリクエストを受け付けるサーバーの場合、コネクションを頻繁に作り直すとパフォーマンスが落ちたり、負荷が上がって適切に動かなくなることがある

# 7章 キャッシュの活用
- キャッシュを利用する場合はキャッシュしたデータをどこに保存するか決める必要があり
- キャッシュを利用するミドルウェアには以下があればいい
  - keyからvalueが取得できるKVSとしての機能
  - TTLを定められ、TTLがすぎたらexpireしてデータを削除する機能
- memcached
  - メリット
    - KVSとして必要な機能を持っている
    - パフォーマンスが非常に高い
    - 特にPHP接続を永続化できるなど、非常に工夫されている
  - デメリット
    - ストレージを永続化できない
    - レプリケーション機能がないため再起動や障害などで簡単にデータが失われてしまう
  - 消えても困らないキャッシュとして以外の用途は想定されていない
- Redis
  - メリット
      - KVSとしての機能以外にもさまざまな機能を持っている
      - コマンドや扱えるデータ構造も多い
      - データの永続化やレプリケーションの機能もある
      - Redis ClusterやRedis Sentinelなどを使用することでクラスター構成に組むことも可能
    - デメリット
      - 基本的にシングルスレッドで動作するため、単純なGET/SET以外のコマンドを実行する場合に、1クライアントの処理で長時間全体の処理がブロックしてしまう可能性がある
      - 単純なKVSではない使い方をする場合は、発行するコマンドによってパフォーマンスを出しにくいことがある
- キャッシュデータをWebアプリケーションのインメモリに保存する方法もある
  - ミドルウェアとの通信コストが不要になるため、Webアプリケーション上で高速に動作する
  - シングルプロセス・マルチスレッドのアーキテクチャの場合、並行に読み込み・書き込みができるように適切なロックを取る必要がある
  - マルチプロセス・シングルスレッドのアーキテクチャの場合、簡単にはプロセス間でメモリを共有できない
  - 実装によってはTTLの実装を自分でする必要がある
  - デプロイした時やサーバー追加時にキャッシュがないため、デプロイした直後のパフォーマンス劣化や、デプロイ直後にThundering herd problem（詳しくは後述）が発生し、データベースなどに負荷が集中する可能性がある
  - アクセスが集中したことを理由にサーバーを追加する場合、キャッシュが存在しないサーバーを追加することでデータベースなどの負荷が更に増すなど、状況を悪化させる可能性がある ●問題のあるデータをキャッシュしたときに簡単に消せないことが多い
- キャッシュをサーバー上のファイルに保存する方法もある
  - こちらもサーバー追加時にキャッシュがない状態から始まるので、インメモリキャッシュ同様のデメリットある
- インメモリやファイルによるキャッシュはミドルウェアへのリクエストを減らしたいときに補助的に利用するのが吉
  - インメモリやファイルによるキャッシュは扱いにくいことが多い
  - インメモリキャッシュのメリットである高速というメリットを享受しつつ、デメリットも減らせる
- キャッシュを用いるメリット
  - CPUへの負荷が大きな処理や時間がかかる処理の実行回数を抑えられるので、パフォーマンスが上がる 
  - インフラコストも下げられる 
  - 外部APIを使用している場合は外部APIの呼び出し回数に制限がある（レートリミット）ことが多い
    - その場合はその制限に達しないようにキャッシュを利用する必要がある
  - 大量のリクエストに耐えられる仕組みが比較的容易に作れる
- デメリット
  - 古いデータが表示されることがある
    - データ上、不整合が発生することもある
    - データ更新時にキャッシュの削除・更新を適切に行うことで、ある程度軽減できる
  - キャッシュを保存するミドルウェアが新しいWebサービス上の障害点になる
    - ミドルウェアの空き容量の不足がないかなど正しく動作しているか監視をする必要がある
    - ミドルウェアの再起動などによってミドルウェアに保存したデータが一気に失われることもある
  - 想定外のデータを表示してしまい、情報流出に繋がる可能性がある
     -  重大なセキュリティリスクになることもある
  -  プログラムの実装が複雑になるため、問題が起こったときの原因究明の難易度が上がる
  -  キャッシュに乗っていないタイミングで大量にリクエストが来ると、実装によってはキャッシュ生成の重い処理が大量に同時実行されることがある
     -  Thundering herd problemと呼ばれる問題
- 以下のことを考えて導入するといい
- データの不整合がどこまで許されるか
  - 決済情報など重要なデータは不整合が致命的になるので、キャッシュを使うべきではない
  - 更新したはずのデータが更新されないとユーザーからバグを疑われる
- データの特性上、本当にキャッシュを使う必要があるか
  - ユーザー情報などはユーザー毎にキャッシュが分散するため、有効にキャッシュを使えない可能性がある
  - 有効に使えないキャッシュが増えると、キャッシュを保存するミドルウェアの容量が足りなくなる可能性がある
  - ユーザー情報を取り違えると、重大なセキュリティリスクに繋がる可能性がある
- データの更新頻度はどの程度か
  - データが頻繁に更新される場合、キャッシュをしても有効に活用できない可能性がある
  - データの鮮度が重要な機能の場合、更新頻度が低いとユーザー体験が悪化する
- データの生成コストを考えているか
  - 生成コストが低いならキャッシュを使う必要はない
  - 生成コストが高すぎる場合、キャッシュデータが失われると長時間復旧できないため、生成結果をRDBMSなどデータが失われにくいデータベースに保存する必要がある
- キャッシュにおいては十分短いTTLを設定する必要がある
- データが更新されたときにキャッシュも同時に更新するようにする
- Thundering herd problem
  - キャッシュがない段階でサーバーに大量にリクエストを投げることで高負荷になってしまう問題
    - この場合、キャッシュが作成されていない状態で大量にリクエストを投げるのでキャッシュが活用できずに高負荷になってしまう
  - 対策
    - キャッシュの残り時間も取得し、そのキャッシュの残り時間が指定した時間を下回っている場合は一定の確率でexpireしているとみなし、キャッシュの再構築をする実装をする
    - キャッシュがなければデフォルト値や古いキャッシュを返し、非同期にキャッシュ更新処理を実行する
      - メリット
        - レスポンスはほとんどのケースで高速に返せる
        - アクセスがあったものだけキャッシュを生成するため効率的
      - デメリット
        - ロジックが複雑
        - キャッシュ更新を非同期に行うため、ジョブキューなどの非同期に処理を実行できる仕組みが必要
        - キャッシュがなければデフォルト値を返す実装の場合、キャッシュがないタイミングでリクエストした人には適切なレスポンスを返せない
          - 古いキャッシュを返す場合も、本来なら使うべきでない古いキャッシュを使用している
        - Thundering herd problemは解決していない
          - キャッシュ更新時の処理が複数実行されないようにする仕組みは別に必要
    - バッチ処理などで定期的にキャッシュを更新する
      - メリット
        - 実装は比較的簡単
        - Thundering herd problemが発生しない
      - バッチで生成できるデータにしか使えない
      - アクセスがほぼ来ないデータも事前に生成する必要があるので、ほぼ使われないキャッシュも管理する必要がある
      - 障害でキャッシュが揮発したときに復旧に時間がかかる可能性がある
        - キャッシュが存在しなければ、データを取得できないためエラーになってしまう
        - バッチを実行すれば復旧できるはずだが、実行する必要があるバッチが大量にある場合は復旧の難易度が高い
- キャッシュの監視
  - 以下の二つを監視する必要がある
    - expireしていないのにキャッシュから追い出されたアイテム数(evicted items)
    - キャッシュヒット率(cache-hit ratio)

# 8章 抑えておきたい高速化手法
- 外部コマンド実行ではなく、ライブラリを利用する
  - 外部コマンドを実行すると、アプリケーションとは別のプロセスを起動する必要がある
  - 別のプロセスを起動するコストがかかり、起動したプロセスがメモリを消費するため、メモリの消費量も増えてしまう
- 開発用の設定で冗長なログを出力しない
  - 冗長なログも動作を遅くする原因
- HTTPクライアントについて
  - 以下のことを確認して導入するといい
    - 同一ホストへのコネクションを使い回す
      - コネクションの確立にもコストがかかる
        - TCP、TLSハンドシェイクによる通信回数増加
        - ローカルポートの大量消費
    - 適切なタイムアウトを設定する
      - 通信は成功する保証はなく、その間リソースを消費し続けるので適切なタイムアウトを設定すること
    - 同一ホストに大量のリクエストを送る場合、対象ホストへのコネクション数の制限を確認する
      - ライブラリやネットワークのシステムによっては外部Webサービスへ負荷をかけにくいようにしたいなどの理由で、対象ホストへのコネクション数が制限されていることがある
      - 同一ホストに大量のリクエストを送る場合、ドキュメントを確認して、制限がどうなっているか確認する必要があります。
- 静的ファイル配信をリバースプロキシから直接配信する
- HTTPヘッダーを活用してクライアント側にキャッシュさせる
  - 画像ファイルやCSS・JavaScriptのファイルなどは更新頻度が低く、かつ同じコンテンツを何回も参照することがある
  - Cache-Controlヘッダーを活用すれば、無駄な通信が発生しないWebサービスを構築できる
- CDN上にHTTPレスポンスをキャッシュする

# 9章 OSの基礎知識とチューニング
- Linuxではコードを書き換えずともカーネルの挙動を変える機能としてカーネルパラメータが存在する
  - これらの設定を変更することで、だいたいのユースケースには対応可能
- Linuxは、OSとしてのコア機能をLinux Kernelと呼ばれるソフトウェアが担っている
  - OS上で動作するアプリケーションはシステムコールと呼ばれる命令を用いてLinux Kernelの機能を利用している
  - アプリケーションとOSのコア部分の間にインターフェースを設けて実装を分割することで、さまざまなハードウェアごとの違いなど、OS上で動作するアプリケーションがOS以下のレイヤーにおける違いを意識することなく利用できるようになっている
- Linux Kernel側をカーネル空間（カーネルランド）
- システムコールを利用するLinux OS上のアプリケーションが動作する部分をユーザ空間（ユーザランド）と呼ぶ
- プロセスの中で並行実行する概念をスレッドという
  - 一個のプロセスにスレッドは１つ以上ある
- OSは通信を受け取るとインタラプトという割り込みをCPUにかける
  - パケット処理やキーボード入力など即時処理が必要なものを「ハードウェア割り込み」
  - 割り込みの中でも遅れて実行するものを「ソフト割り込み」と定義している
- CPUはコンテキストスイッチといって超高速にプロセスを切り替えることによって並行実行しているように見せかける
  - 切り替えにかかる時間をコンテキストスイッチコストという
- Linux NAPI
  - 割り込み命令を用いた手法とポーリング手法を組み合わせて利用
  - できるだけ割り込み命令を減らしながら遅延の少ない形でパケットを処理するため
- LinuxのディスクI/O
  - HDD（Hard Disk Drive）であれば磁気ディスクを実際に回転させて読み書きを行っているため、保存領域の最初から逐次的に読み書きを行うシーケンシャルリード／シーケンシャルライトは比較的高速に行えます。 　
  - しかし、現代の用途では保存領域の上から綺麗に読み書きするのは現実的ではなく、実際には書き込み位置が特定されないランダムリード／ランダムライトが重視される
  - SSD（Solid State Drive）は、HDDと比較してランダムリード／ランダムライトを高速に行うことができる
    - 磁気ディスクを利用せずNANDフラッシュメモリを用意した上で、その中に書き込む
- ストレージ性能は、スループット・レイテンシ・IOPS（Input/Output Per Second）の3つに大きく分けられる
  - 小さなファイルを大量に扱うWebアプリケーションであった場合はスループットよりもIOPSを重視し、大きなファイルを扱う場合はIOPSよりもスループットを重視
- ブロックサイズ
  - 1度に扱えるファイルサイズ
- 物理的もしくはネットワーク経由で接続されたブロックストレージをWebアプリケーションからファイルシステムとして読み書きできるようにするために、ディスクを「マウント」する必要がある
  - 接続されたブロックストレージのことを、ブロックデバイスと呼びます。
- CPU利用率
- topコマンドで閲覧できる
- us - User：ユーザ空間におけるCPU利用率
  - 実装しデプロイされているWebアプリケーションが多くのCPUを利用している場合に上昇する値
- sy- System：カーネル空間におけるCPU利用率
  - カーネル空間におけるCPU利用率を指す
  - プロセスのforkが多く発生している環境やコンテキストスイッチを行っている時間が長くなっている環境においてはカーネル空間の処理が大きくなるため、syの値が上昇する
    - fork:プロセスのコピー
  - Webアプリケーションを運用する場合においては、WebアプリケーションやミドルウェアがCPUの支援処理を利用する際に大きく上昇することも多い
- ni - Nice：nice値（優先度）が変更されたプロセスのCPU利用率
  - nice値は-20~19で定義される
  - 低いほと優先度が高い
  - 特に指定がない場合は、fork(2)の際に親プロセスの定義がそのままコピーされる
  - PID=1のプロセスの優先度は多くの実装で0のため、ユーザ空間で動作する多くのプロセスは0になっている
  - 実際にサービスが運用しているサーバーにおいてログファイルの圧縮や削除のような大きなI/O処理を行う際に、サービスに影響が出ないようionice(1)コマンドで優先度を下げるなどの運用を行うことがあある
- id - Idle：利用されていないCPU
  - CPUの大まかな利用率を知りたい場合は、100からidの値を引いた値を参照すれば他の値の合計値
- wa - Wait：I/O処理を待っているプロセスのCPU利用率
  - waの値が上がっている場合は、ディスクなどへの読み書きの終了を待っているプロセスが多く存在していることを示している
  - ディスクへの読み書き読み書きを行わないよう設計を変更したり、極力読み書きを減らすような変更を行ったりする必要がある
- hi - Hardware Interrupt：ハードウェア割り込みプロセスの利用率
- si - Soft Interrupt：ソフト割り込みプロセスの利用率
- st - Steal：ハイパーバイザによって利用されているCPU利用率
  - パブリッククラウドなど仮想化された環境のLinux上で利用されるCPU利用率
- Linuxにおける効率的なシステム設定
- ulimit
  - ulimit（User limit）は、プロセスが利用できるリソースの制限を設定する概念
  - 各プロセスはどのリソースをどのぐらい利用できるのかについて、制限がかけられている
  - プロセス単位で設定されている
  - ミドルウェアの多くは効率的に処理を行うためファイルの読み書きを行う場合がある
  - 特にパケットを多くやりとりする環境ではコネクションも多く生成されるため、同時に利用するファイルの数も増える傾向にある
  - アクセスが多くなった際に突然デーモンが落ちてしまい障害が発生することもあるため、事前に制限を上げておく必要がある
- Linuxカーネルパラメータ
  - sysctlコマンドで確認できる
  - net.core.somaxconn
    - パケット通信における「接続要求のキュー」であるbacklogがどのぐらい受け入れられるか
      - listen(2)でソケットへの接続要求を待ち、accept(2)で接続要求のキューから取り出す
      - 漏れるとパケットを破棄する
  - net.ipv4.ip_local_port_range
    - Ephemeral Portsの範囲を設定するパラメータ
      - クライアント側で開けるポート番号のこと
      - 逆にサーバー側で用いるHTTPなら80、HTTPSなら443番といったポート番号はシステムポートと呼ばれている
      -  - 0～1023番ポートが「System Ports」、1024～49151が「User Ports」、49152～65535が「Dynamic and/or Private Ports」と呼ばれている
    - アプリケーションサーバーがMySQLなどのデータベースや他のシステムに接続する場合など、通信のクライアント側として動作する際にこのポートを使い切ることがある
    - 実はipv6にも有効
    - 同じホスト内の別プロセスへの通信に対してはUNIX domain socketを利用することもできる
      - 具体的にはWebサーバーから同じサーバーに存在するWebアプリケーションへの接続やデータベースサーバーへの接続が該当
      - サーバーのポートを使わず通信できる
      - ソケットファイルに接続するため同じホスト内での通信でしか利用できないという制約はある
      - ポートへの接続と比較して「どのポートが利用可能か」などの処理を回避できる
      - パフォーマンス面でも利点が多く、利用できるパターンであれば利用することを推奨し
  - 通常のlisten(2)を用いた通信ではなくソケットファイルを通して接続を待機することもできる
- MTU（Maximum Transmission Unit）
  - そのネットワークインターフェースから送信できる最大サイズ
  - 設定した値よりも大きなパケットを送信する際には、MTUのサイズまで分割して送信
  - TCPにおいてもMSS（Maximum Segment Size）という技術でMTUに近い形での分割送信を行っている
  - Webサービスを提供する上でMTU/MSSの設定はあまり大きな効果を得ることができない
    - ネットワークの様々な経路で設定されているため
  - 多くの場合においては、MTUの設定はOSの定めるデフォルト設定を利用するべき

# 感想
知らない知識があまりに沢山あったがそのどれもが理解できない訳ではないのでいい勉強になったなと思う
業務でどれぐらい生きるかは怪しいが長い目で見て役にたつことを願いたい
実践的にハンズオンした訳ではないため、実際にやるとなったら手間取るんだろうなあと思うがその土台は作れたと思う
