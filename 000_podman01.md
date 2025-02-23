# **RHEL環境におけるPodmanとPodman Composeの商用利用**

## **はじめに**

コンテナ技術は、アプリケーションの開発、デプロイ、運用を効率化する上で重要な役割を果たしています。Dockerは広く普及しているコンテナエンジンですが、近年ではPodmanという新たなコンテナエンジンが注目を集めています。Podmanは、セキュリティとパフォーマンスの面でDockerよりも優れている点があり、特にエンタープライズ環境での利用に適しています。

Podmanは、デーモンレスアーキテクチャを採用しているため、Dockerよりも軽量で高速に動作します 1。デーモンとは、バックグラウンドで常時実行されるプロセスであり、Dockerではコンテナの管理にデーモンを使用しています。一方、Podmanはデーモンを使用しないため、システムリソースの消費を抑え、セキュリティリスクを低減することができます 2。また、Podmanはroot権限を必要とせずにコンテナを実行できるルートレスモードをサポートしており 2、セキュリティをさらに強化することができます。

さらに、PodmanはKubernetesのPodの概念をサポートしています 3。Podとは、複数のコンテナをグループ化し、リソースを共有するための機能です。Podmanでは、Podを使用してコンテナを管理することで、Kubernetesと同様の運用が可能になります。

本稿では、Red Hat Enterprise Linux (RHEL) 環境においてPodmanとPodman Composeを商用環境で利用する場合の考慮事項と懸念点について解説します。具体的には、冗長構成、可用性、スケーラビリティ、パフォーマンス、セキュリティ、ログ管理、モニタリング、リソース制限、RHEL特有の考慮事項、Kubernetesとの比較、商用サポートなどについて詳しく説明します。

## **メリットとデメリット**

RHEL環境でPodmanとPodman Composeを商用環境で利用する際のメリットとデメリットをまとめると、以下のようになります。

| メリット | デメリット |
| :---- | :---- |
| デーモンレスアーキテクチャによるセキュリティとパフォーマンスの向上 | Kubernetesのような高度なオーケストレーション機能は提供されない |
| ルートレスコンテナのサポート | Docker Composeとの完全な互換性はない |
| systemdとの連携 | RHEL HAアドオンの使用制限 5 |
| Red Hatによる商用サポート 2 |  |
| コスト削減 (Docker Desktopのライセンス費用が不要) 6 |  |

## **冗長構成の担保**

商用環境では、システムの可用性を高めるために冗長構成が必須となります。PodmanとPodman Composeの冗長構成を実現する方法としては、以下のようなものがあります。

* 複数ノードへのPodmanコンテナのデプロイ: 複数の物理サーバーまたは仮想マシンにPodmanコンテナをデプロイすることで、1つのノードに障害が発生した場合でも、他のノードでコンテナが動作し続けることができます 7。  
* ロードバランシング: ロードバランサーを使用して、複数のノードにトラフィックを分散させることで、負荷を分散し、可用性を高めることができます 8。  
* 高可用性ソフトウェアの利用: Evidian SafeKitのような高可用性ソフトウェアを使用することで、リアルタイムのファイルレプリケーションとフェイルオーバーを実現し、Podmanサービスの高可用性を確保することができます 9。

これらの方法を組み合わせることで、商用環境で求められる可用性レベルを満たすことができます。

### **障害発生時のフェイルオーバー**

高可用性を実現する上で、障害発生時のフェイルオーバーの仕組みは重要です。上記の高可用性ソフトウェアを利用する場合、フェイルオーバー時間は、障害検出時間（デフォルトで30秒）とアプリケーションの起動時間の合計となります 9。また、CyberArk Conjurのようなソフトウェアでは、自動フェイルオーバー機能により、スタンバイノードが新しいリーダーに昇格し、古いリーダーはクラスターから削除されます 10。

### **リカバリ**

Podmanは、更新されたコンテナの起動に失敗した場合、自動的に最後の正常なバージョンにロールバックする機能を備えています 3。これにより、アプリケーションの信頼性を向上させることができます。また、podman system resetコマンドを使用することで、Podman環境をリセットすることができます 11。 さらに、Podman Desktopでは、コンテナイメージをtarでアーカイブし、バックアップとリストアを行うことができます 12。ただし、コンテナが異常な状態で停止した場合、podman container stopコマンドが機能しない場合があるため注意が必要です 13。

## **可溶性**

商用環境では、トラフィックの増加や負荷の変動に対応するために、システムの可溶性が必要となります。PodmanとPodman Composeの可溶性を実現する方法としては、以下のようなものがあります。

* 複数ノードへのPodmanコンテナのデプロイ: 複数のノードにコンテナをデプロイし、ロードバランサーでトラフィックを分散することで、スケーラビリティを向上させることができます 7。  
* Podman Podの利用: Podman Podは、複数のコンテナをグループ化し、リソースを共有するための機能です。Podman Composeと組み合わせることで、スケーラビリティと可用性を向上させることができます 14。  
* Kubernetesとの連携: PodmanはKubernetesと連携することができます。Podman DesktopからKubernetes YAMLファイルを生成し、Kubernetes環境にデプロイすることができます 15。

### **複数ノードへのPodmanコンテナのデプロイと管理**

Podman Desktopでは、VPNやプロキシの設定、複数のイメージレジストリとの対話、リモートクラスタへの接続とデプロイを行うことができます 16。これらの機能を利用することで、複数ノードへのコンテナのデプロイと管理を効率化することができます。

### **ロードバランシング**

ロードバランシングは、複数のノードにトラフィックを分散させることで、負荷を分散し、可用性を高めるための技術です。HAProxyは、HTTP (Layer 7\) と TCP (Layer 4\) のロードバランシングとプロキシサービスを提供するオープンソースのソフトウェアです 17。PodmanでHAProxyをコンテナとして実行し、複数のバックエンドサーバーへのトラフィックを分散させることができます。

HAProxyをPodmanで利用する例として、以下のような手順が考えられます。

1. 4台のOracle Linuxシステムを用意します (HAProxyノード1台、Webアプリケーションノード3台) 17。  
2. 各システムにOracle Linuxをインストールし、必要なパッケージをインストールします。  
3. HAProxyノードで、Podmanを使用してHAProxyコンテナを起動します。  
4. HAProxyの設定ファイルで、WebアプリケーションノードのIPアドレスとポートを指定します。  
5. ロードバランサーのIPアドレスをクライアントに公開します。

### **スケーリング**

スケーリングは、システムの負荷に応じてリソースを増減させることで、パフォーマンスを維持するための技術です。Podman Composeでは、docker-compose.ymlファイルに環境変数を追加することで、コンテナのレプリカ数を増やし、スケーリングすることができます 18。ただし、Podman単体では、podman pod scaleのようなコマンドは提供されていません 19。大規模なスケーリングが必要な場合は、Kubernetesのようなコンテナオーケストレーションツールとの連携を検討する必要があります 20。

## **非機能面**

商用環境では、パフォーマンス、セキュリティ、ログ管理、モニタリングなど、様々な非機能要件を満たす必要があります。PodmanとPodman Composeは、これらの要件を満たすための機能を備えています。

### **パフォーマンス**

Podmanは、デーモンレスアーキテクチャを採用しているため、Dockerよりも軽量で高速に動作します 1。また、runC実行コンテナを直接使用することで、パフォーマンスを向上させています 21。Podmanのパフォーマンスは、コマンドラインオプション、OCIランタイム、ホストファイルシステム、コンテナイメージなど、様々な要因に影響されます 22。 さらに、Podmanでは、イメージのプルやプッシュにzstd圧縮を使用することができます 23。zstd圧縮は、gzipよりも効率的で高速であるため、ネットワークトラフィックとストレージ容量を削減することができます。

### **セキュリティ**

Podmanは、セキュリティを重視した設計となっています。デーモンレスアーキテクチャを採用することで、攻撃対象領域を削減し、セキュリティリスクを低減しています 2。また、ルートレスコンテナをサポートしており 2、root権限を持たないユーザーでもコンテナを実行することができます。さらに、SELinuxラベルを使用して各コンテナを起動することで 2、コンテナプロセスに提供されるリソースと機能をより詳細に制御することができます。 また、--cap-add=CAP\_SYS\_ADMINオプションを使用することで、コンテナに特定のケーパビリティを付与することができます 24。例えば、CAP\_SYS\_ADMINケーパビリティを付与することで、コンテナ内でシステムコールを実行することができます。ただし、ケーパビリティを付与する際は、セキュリティリスクを十分に評価する必要があります。

API設計においては、Broken Object Level Authorization (BOLA)を防ぐためのセキュリティ対策を講じる必要があります 25。BOLAとは、APIの認可が不適切なために、ユーザーがアクセス権限を持たないリソースにアクセスできてしまう脆弱性です。BOLAを防ぐためには、アクセス制御リスト (ACL) を使用し、各リソースに対するアクセス許可を明確に定義する必要があります。

### **ログ管理**

Podmanでは、podman logsコマンドを使用してコンテナのログを表示することができます 26。また、podman eventsコマンドを使用することで、Podmanで発生するイベントを監視することができます 27。ログ管理には、FluentdやElasticsearchなどのツールを組み合わせて使用することができます。

### **モニタリング**

Podmanのモニタリングには、Instanaなどのツールを使用することができます 28。Instanaは、Podmanコンテナのメトリックを自動的に収集し、リアルタイムで表示することができます 29。また、Podman Webコンソールを使用することで 2、コンテナが使用しているCPUとメモリの量を監視することができます。

### **データ管理**

大量のデータを扱う場合、データの整合性を保つために、重要なデータのコピー前後にチェックサムを確認することを推奨します 26。これにより、データが正しくコピーされたかを確認することができます。

## **サーバ性能**

PodmanとPodman Composeを利用する際のサーバー性能への影響は、コンテナの数や種類、実行されるアプリケーションなどによって異なります。必要なCPU、メモリ、ストレージ容量は、コンテナの要件に合わせて適切に設定する必要があります。

### **CPU、メモリ、ストレージ容量**

コンテナのCPUやメモリ使用状況を確認するには、podman statsコマンドを使用します 30。ストレージ容量は、コンテナイメージのサイズやコンテナ内で生成されるデータ量によって異なります。

OpenShift Localを動作させるためのハードウェアリソースは、以下のとおりです 31。

* 物理CPUコア: 4個  
* 空きメモリ: 10.5GB  
* ストレージ領域: 35GB

### **パフォーマンスチューニング**

Podmanのパフォーマンスチューニングには、以下のような方法があります。

* リソース制限: コンテナにCPUやメモリの制限を設定することで、リソースの過剰な消費を防ぐことができます 32。  
* ストレージドライバ: 適切なストレージドライバを選択することで、ストレージのI/Oパフォーマンスを向上させることができます 33。  
* ネットワーク: 高速なネットワークを使用することで、コンテナ間の通信速度を向上させることができます。

## **リソース制限**

Podmanでは、--ulimitオプションを使用してコンテナのリソース制限を設定することができます 34。CPU、メモリ、ストレージなど、様々なリソースを制限することができます。 また、-cpus引数を使用してCPUリソースを 35、--vm-bytes引数を使用してメモリリソースを 36、それぞれコンテナに割り当てることができます。 containers.confファイルでpids\_limitを設定することで、コンテナ内の最大プロセス数を変更することもできます 32。 リソース制限を設定することで、コンテナが過剰なリソースを消費することを防ぎ、システム全体の安定性を確保することができます。

### **リソース制限による影響**

リソース制限を設定することで、コンテナのパフォーマンスに影響を与える可能性があります。例えば、CPU制限を設定した場合、コンテナの処理速度が低下する可能性があります。リソース制限を設定する際は、コンテナの要件とシステム全体のバランスを考慮する必要があります。

### **リソース制限のベストプラクティス**

リソース制限のベストプラクティスとしては、以下のようなものがあります。

* コンテナの要件に合わせて制限を設定する: コンテナが必要とするリソース量を把握し、それに合わせて制限を設定する必要があります。  
* 制限値を監視する: リソース制限を設定した後、コンテナのリソース使用状況を監視し、必要に応じて制限値を調整する必要があります。  
* セキュリティを考慮する: リソース制限は、セキュリティ対策としても有効です。コンテナが過剰なリソースを消費することを防ぐことで、DoS攻撃などのリスクを低減することができます。

## **RHEL特有の考慮事項や制限事項**

RHEL環境でPodmanを利用する際の考慮事項としては、SELinuxの設定やsystemdとの連携などがあります 34。 podman generate systemdコマンドを使用することで、コンテナをsystemdサービスとして管理するためのユニットファイルを作成することができます 37。 また、RHEL HAアドオンのpodmanクラスターリソーススクリプトには、使用制限があります 5。

ルートレスコンテナを実行する場合は、restorecon \-Rコマンドを使用して、コンテナを実行する非rootユーザーのホームディレクトリにSELinuxのラベルを再設定する必要があります 38。 また、RHEL8でcgroupv2を使用している場合は、/etc/containers/containers.confファイルを編集してcgroupv2のバグを修正する必要があります 38。 さらに、loginctl enable-lingerコマンドを使用して、コンテナユーザーに対してlingerを有効にする必要があります 38。 lingerを有効にすることで、ユーザーがログアウトした後も、コンテナが実行され続けるようになります。

systemdでpodman-composeを使用する場合は、podman-compose systemd \-a create-unitコマンドでsystemdユニットファイルを作成することができます 38。 ユニットファイルを作成する際は、podman-compose upコマンドに--in-podフラグを追加する必要があります 38。 \--in-podフラグを追加することで、コンテナがPod内で実行されるようになります。

macOSでPodmanを使用する場合、Podman Machineの性能と安定性が向上しているため、Dockerよりも高速かつ安定して動作します 39。

## **Kubernetesとの比較**

Podmanは、Kubernetesのようなコンテナオーケストレーションツールと比較して、軽量でシンプルな点が特徴です。小規模なシステムや単一ホストでの利用に適しています 20。 一方で、Kubernetesは、大規模なシステムや複数ノードでの利用に適しており、高度なスケーラビリティ、可用性、管理機能を提供します 7。

PodmanはKubernetesと連携することができます。Podman DesktopからKubernetes YAMLファイルを生成し 15、Kubernetes環境にデプロイすることができます。 また、podman generate kubeコマンドを使用することで、コンテナからKubernetes YAMLファイルを生成することもできます 40。 さらに、Podman Desktopでは、ターミナルからKubernetes Podに接続することができます 12。

podman kube playコマンドを使用することで、Kubernetes YAMLファイルに基づいてコンテナ、Pod、ボリュームを作成することができます。ただし、podman kube playコマンドは、Kubernetesのすべての機能をサポートしているわけではありません 41。 例えば、resources.requests.cpuの設定は適用されません。

## **商用サポートの有無と内容**

Podmanは、Red Hat Enterprise Linuxサブスクリプションに含まれており、商用サポートを受けることができます 2。商用サポートの内容としては、バグ修正、セキュリティアップデート、技術サポートなどが提供されます。

## **推奨構成**

RHEL環境でPodmanとPodman Composeを商用環境で利用する場合の推奨構成は、以下のとおりです。

* 冗長構成: 複数の物理サーバーまたは仮想マシンにPodmanコンテナをデプロイし、ロードバランサーでトラフィックを分散します。  
* 高可用性ソフトウェア: Evidian SafeKitのような高可用性ソフトウェアを使用して、フェイルオーバーとリカバリの仕組みを構築します。  
* モニタリング: Instanaなどのツールを使用して、コンテナのパフォーマンスとリソース使用状況を監視します。  
* リソース制限: \--ulimitオプションを使用して、コンテナのリソース制限を設定します。  
* セキュリティ: SELinuxを有効化し、セキュリティのベストプラクティスに従ってシステムを構成します。

## **開発環境**

Podmanは、様々な開発ツールと連携することができます。 Visual Studio Codeには、Podmanの拡張機能が用意されており 42、Visual Studio CodeからPodmanコンテナを操作することができます。

## **CI/CD**

Podmanは、Cirrus CLIやGitHub ActionsなどのCI/CDツールとも連携することができます 42。 Cirrus CLIは、Podmanを使用してコンテナ化されたタスクを実行するためのツールです。 GitHub Actionsは、GitHubが提供するCI/CDサービスであり、Podmanのサポートが含まれています。

## **その他**

Podmanの公式Webサイトでは、Podmanの塗り絵が公開されています 42。 塗り絵を通して、Podmanについて楽しく学ぶことができます。

## **結論**

PodmanとPodman Composeは、RHEL環境でコンテナを実行するための強力なツールです。セキュリティとパフォーマンスの面でDockerよりも優れており、商用環境での利用に適しています。 また、デーモンレスアーキテクチャを採用しているため、システムの安定性とセキュリティが向上します。さらに、ルートレスコンテナをサポートしているため、セキュリティリスクを低減することができます。

PodmanはKubernetesと連携することもでき、Kubernetes YAMLファイルを生成したり、Podman DesktopからKubernetes Podに接続したりすることができます。ただし、Kubernetesのような高度なオーケストレーション機能は提供されないため、大規模なシステムや複雑なアプリケーションを運用する場合は、Kubernetesとの連携を検討する必要があります。

PodmanはRed Hat Enterprise Linuxサブスクリプションに含まれており、商用サポートを受けることができます。 Docker Desktopのライセンス変更により、商用利用に制限が設けられたDocker Desktopと比較して、Podmanはコスト面でも優位性があります。

RHEL環境でコンテナ技術を利用する際は、PodmanとPodman Composeを積極的に検討することを推奨します。

#### **引用文献**

1\. Podman vs Docker: Key Differences and Which is Better | Last9, 2月 23, 2025にアクセス、 [https://last9.io/blog/podman-vs-docker/](https://last9.io/blog/podman-vs-docker/)  
2\. Podman とは？をわかりやすく解説 \- Red Hat, 2月 23, 2025にアクセス、 [https://www.redhat.com/ja/topics/containers/what-is-podman](https://www.redhat.com/ja/topics/containers/what-is-podman)  
3\. What is Podman? \- Red Hat, 2月 23, 2025にアクセス、 [https://www.redhat.com/en/topics/containers/what-is-podman](https://www.redhat.com/en/topics/containers/what-is-podman)  
4\. 系統設計面試中Docker、Kubernetes 和Podman 有何不同？ \- CodeLove 論壇, 2月 23, 2025にアクセス、 [https://codelove.tw/@tony/post/gqBjPq](https://codelove.tw/@tony/post/gqBjPq)  
5\. RHEL HA addon単体でpodmanクラスターリソースはサポートしません | Dell 日本, 2月 23, 2025にアクセス、 [https://www.dell.com/support/kbdoc/ja-jp/000204825/rhel-ha-addon-%E5%8D%98%E4%BD%93%E3%81%A7podman-%E3%82%AF%E3%83%A9%E3%82%B9%E3%82%BF%E3%83%BC%E3%83%AA%E3%82%BD%E3%83%BC%E3%82%B9%E3%81%AF%E3%82%B5%E3%83%9D%E3%83%BC%E3%83%88%E3%81%97%E3%81%BE%E3%81%9B%E3%82%93](https://www.dell.com/support/kbdoc/ja-jp/000204825/rhel-ha-addon-%E5%8D%98%E4%BD%93%E3%81%A7podman-%E3%82%AF%E3%83%A9%E3%82%B9%E3%82%BF%E3%83%BC%E3%83%AA%E3%82%BD%E3%83%BC%E3%82%B9%E3%81%AF%E3%82%B5%E3%83%9D%E3%83%BC%E3%83%88%E3%81%97%E3%81%BE%E3%81%9B%E3%82%93)  
6\. Podman環境を作ろう \#container \- Qiita, 2月 23, 2025にアクセス、 [https://qiita.com/shupeluter/items/2ff6f79bcf84c30f7265](https://qiita.com/shupeluter/items/2ff6f79bcf84c30f7265)  
7\. Kubernetes入門：コンテナオーケストレーションの基本を学ぼう | 1R\_Techブログ \- Wantedly, 2月 23, 2025にアクセス、 [https://www.wantedly.com/companies/company\_3056942/post\_articles/549475](https://www.wantedly.com/companies/company_3056942/post_articles/549475)  
8\. 2.3. ロードバランサーとデータベースの設定 | Red Hat Product Documentation, 2月 23, 2025にアクセス、 [https://docs.redhat.com/ja/documentation/red\_hat\_quay/3.7/html/deploy\_red\_hat\_quay\_-\_high\_availability/set\_up\_load\_balancer\_and\_database](https://docs.redhat.com/ja/documentation/red_hat_quay/3.7/html/deploy_red_hat_quay_-_high_availability/set_up_load_balancer_and_database)  
9\. Podman: the simplest high availability cluster between two redundant servers \- Evidian, 2月 23, 2025にアクセス、 [https://www.evidian.com/products/high-availability-software-for-application-clustering/podman-the-simplest-high-availability-cluster-between-two-redundant-servers/](https://www.evidian.com/products/high-availability-software-for-application-clustering/podman-the-simplest-high-availability-cluster-between-two-redundant-servers/)  
10\. Repair cluster health after auto-failover \- CyberArk Docs, 2月 23, 2025にアクセス、 [https://docs.cyberark.com/conjur-enterprise/13.1/en/content/deployment/highavailability/repair-cluster-after-auto-failover.htm?TocPath=Administration%7CCluster%20management%7CAuto%20and%20manual%20failover%7CAuto-failover%7C\_\_\_\_\_3](https://docs.cyberark.com/conjur-enterprise/13.1/en/content/deployment/highavailability/repair-cluster-after-auto-failover.htm?TocPath=Administration%7CCluster+management%7CAuto+and+manual+failover%7CAuto-failover%7C_____3)  
11\. Podman commands fail with the error "Error: error unmarshalling state for container". \#17634 \- GitHub, 2月 23, 2025にアクセス、 [https://github.com/containers/podman/issues/17634](https://github.com/containers/podman/issues/17634)  
12\. Docker Desktopの代替となる「Podman Desktop 1.9」リリース。Macでの安定性や性能が大幅に向上したコンテナエンジン「Podman 5.0」を搭載 \- Publickey, 2月 23, 2025にアクセス、 [https://www.publickey1.jp/blog/24/docker\_desktoppodman\_desktop\_19macpodman\_50.html](https://www.publickey1.jp/blog/24/docker_desktoppodman_desktop_19macpodman_50.html)  
13\. podman fails ungracefully if log destinations have no free space for writes \#19943 \- GitHub, 2月 23, 2025にアクセス、 [https://github.com/containers/podman/issues/19943](https://github.com/containers/podman/issues/19943)  
14\. How to Scale Containers Efficiently Using Podman Pods and veth Pairs for High-Performance Networking \- Cyber Solin, 2月 23, 2025にアクセス、 [https://www.cybersolin.com/post/how-to-scale-containers-efficiently-using-podman-pods-and-veth-pairs-for-high-performance-networking](https://www.cybersolin.com/post/how-to-scale-containers-efficiently-using-podman-pods-and-veth-pairs-for-high-performance-networking)  
15\. 什么是Podman Desktop？ \- Red Hat, 2月 23, 2025にアクセス、 [https://www.redhat.com/zh/topics/containers/what-is-podman-desktop](https://www.redhat.com/zh/topics/containers/what-is-podman-desktop)  
16\. Podman Desktop とは \- Red Hat, 2月 23, 2025にアクセス、 [https://www.redhat.com/ja/topics/containers/what-is-podman-desktop](https://www.redhat.com/ja/topics/containers/what-is-podman-desktop)  
17\. Deploy HAProxy Using Podman on Oracle Linux, 2月 23, 2025にアクセス、 [https://docs.oracle.com/en/learn/ol-podman-haproxy/index.html](https://docs.oracle.com/en/learn/ol-podman-haproxy/index.html)  
18\. Getting started with Compose on Podman Desktop, 2月 23, 2025にアクセス、 [https://podman-desktop.io/blog/getting-started-with-compose](https://podman-desktop.io/blog/getting-started-with-compose)  
19\. scaling containers/pods · containers podman · Discussion \#20525 \- GitHub, 2月 23, 2025にアクセス、 [https://github.com/containers/podman/discussions/20525](https://github.com/containers/podman/discussions/20525)  
20\. Kubernetes vs Podman \- DeployPRO DOCS, 2月 23, 2025にアクセス、 [https://www.docs.deploypro.dev/glossary/kubernetes-vs-podman](https://www.docs.deploypro.dev/glossary/kubernetes-vs-podman)  
21\. Results of Scientific Testing of Docker and Podman vs Docker \- Reddit, 2月 23, 2025にアクセス、 [https://www.reddit.com/r/podman/comments/1hf79fd/results\_of\_scientific\_testing\_of\_docker\_and/](https://www.reddit.com/r/podman/comments/1hf79fd/results_of_scientific_testing_of_docker_and/)  
22\. podman/docs/tutorials/performance.md at main \- GitHub, 2月 23, 2025にアクセス、 [https://github.com/containers/podman/blob/main/docs/tutorials/performance.md](https://github.com/containers/podman/blob/main/docs/tutorials/performance.md)  
23\. コンテナ \- Oracle Help Center, 2月 23, 2025にアクセス、 [https://docs.oracle.com/cd/F61088\_01/relnotes9.4/ol9-features-Containers.html](https://docs.oracle.com/cd/F61088_01/relnotes9.4/ol9-features-Containers.html)  
24\. Security implications of \--cap-add=CAP\_SYS\_ADMIN for rootless containers? · containers podman · Discussion \#23558 \- GitHub, 2月 23, 2025にアクセス、 [https://github.com/containers/podman/discussions/23558](https://github.com/containers/podman/discussions/23558)  
25\. OWASP API Security Top 10 の概要とその重要性, 2月 23, 2025にアクセス、 [https://www.issoh.co.jp/tech/details/4704/](https://www.issoh.co.jp/tech/details/4704/)  
26\. Podmanでよく利用するコマンド一覧とその使い方 \- 株式会社一創, 2月 23, 2025にアクセス、 [https://www.issoh.co.jp/tech/details/2845/](https://www.issoh.co.jp/tech/details/2845/)  
27\. Chapter 21\. Monitoring containers | Red Hat Product Documentation, 2月 23, 2025にアクセス、 [https://docs.redhat.com/en/documentation/red\_hat\_enterprise\_linux/8/html/building\_running\_and\_managing\_containers/assembly\_monitoring-containers](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html/building_running_and_managing_containers/assembly_monitoring-containers)  
28\. Podmanのモニタリング \- IBM, 2月 23, 2025にアクセス、 [https://www.ibm.com/docs/ja/instana-observability/288?topic=technologies-monitoring-podman](https://www.ibm.com/docs/ja/instana-observability/288?topic=technologies-monitoring-podman)  
29\. Monitoring Podman \- IBM, 2月 23, 2025にアクセス、 [https://www.ibm.com/docs/en/instana-observability/current?topic=technologies-monitoring-podman](https://www.ibm.com/docs/en/instana-observability/current?topic=technologies-monitoring-podman)  
30\. Fedora 39 : Podman : リソース使用状況を確認する \- Server World, 2月 23, 2025にアクセス、 [https://www.server-world.info/query?os=Fedora\_39\&p=podman\&f=14](https://www.server-world.info/query?os=Fedora_39&p=podman&f=14)  
31\. 学習環境としてOpenShift Localをインストールする｜Fusayuki Minamoto \- note, 2月 23, 2025にアクセス、 [https://note.com/fminamot/n/ne4d842d85a82](https://note.com/fminamot/n/ne4d842d85a82)  
32\. podmanでのプロセスカウント方法の注意点 \- Qiita, 2月 23, 2025にアクセス、 [https://qiita.com/skitoy4321/items/0bb5f9d0e027137e8199](https://qiita.com/skitoy4321/items/0bb5f9d0e027137e8199)  
33\. The Complete Podman vs Docker Analysis: Features, Performance & Security \- Uptrace, 2月 23, 2025にアクセス、 [https://uptrace.dev/comparisons/podman-vs-docker](https://uptrace.dev/comparisons/podman-vs-docker)  
34\. 10 セキュリティに関する推奨事項 \- Oracle Help Center, 2月 23, 2025にアクセス、 [https://docs.oracle.com/cd/F61410\_01/podman/podman-SecurityRecommendations.html](https://docs.oracle.com/cd/F61410_01/podman/podman-SecurityRecommendations.html)  
35\. コンテナおよびPodへのCPUリソースの割り当て \- Kubernetes, 2月 23, 2025にアクセス、 [https://kubernetes.io/ja/docs/tasks/configure-pod-container/assign-cpu-resource/](https://kubernetes.io/ja/docs/tasks/configure-pod-container/assign-cpu-resource/)  
36\. コンテナおよびPodへのメモリーリソースの割り当て \- Kubernetes, 2月 23, 2025にアクセス、 [https://kubernetes.io/ja/docs/tasks/configure-pod-container/assign-memory-resource/](https://kubernetes.io/ja/docs/tasks/configure-pod-container/assign-memory-resource/)  
37\. Use Compose Files with Podman on Oracle Linux, 2月 23, 2025にアクセス、 [https://docs.oracle.com/en/learn/ol-podman-compose/index.html](https://docs.oracle.com/en/learn/ol-podman-compose/index.html)  
38\. podman-compose and systemd | IT-Hure, 2月 23, 2025にアクセス、 [https://www.it-hure.de/2024/02/podman-compose-and-systemd/](https://www.it-hure.de/2024/02/podman-compose-and-systemd/)  
39\. Docker互換のコンテナエンジン「Podman 5.0」正式リリース。Macでの安定性や性能が大幅に向上, 2月 23, 2025にアクセス、 [https://www.publickey1.jp/blog/24/dockerpodman\_50mac.html](https://www.publickey1.jp/blog/24/dockerpodman_50mac.html)  
40\. How to Install and Use Podman on RHEL: A Comprehensive Guide | by Jerome Decinco, 2月 23, 2025にアクセス、 [https://medium.com/@jeromedecinco/how-to-install-and-use-podman-on-rhel-a-comprehensive-guide-191f4bc0d421](https://medium.com/@jeromedecinco/how-to-install-and-use-podman-on-rhel-a-comprehensive-guide-191f4bc0d421)  
41\. Podman で Kubernetes native なアプリ開発の Innter loop から Outer loop への移行をスムーズに \- Qiita, 2月 23, 2025にアクセス、 [https://qiita.com/k-srkw/items/952ab09f6071a732e3bd](https://qiita.com/k-srkw/items/952ab09f6071a732e3bd)  
42\. Podman, 2月 23, 2025にアクセス、 [https://podman.io/](https://podman.io/)
