# **RHELにおけるPodman/Dockerの商用利用に関する調査報告**

## **概要**

本報告書は、RHEL環境におけるPodman/Docker、Podman-compose/Docker-composeの商用利用に関する調査結果をまとめたものです。特に、コンテナのみの環境における失敗事例、障害事例、サービス影響が出た事例に焦点を当て、その原因、影響範囲、対応策を具体的に考察しています。

Podmanは、Dockerと比較して、デーモンレスアーキテクチャ、rootlessコンテナ、ユーザーネームスペース、SELinuxのサポートといったセキュリティ面での利点を備えています。1 これらの機能により、Podmanは商用環境でより安全なコンテナ実行環境を提供することができます。

## **調査内容**

本調査では、以下の手順で情報を収集しました。

1. RHELのPodman/Docker、Podman-compose/Docker-composeの公式ドキュメントを参照し、既知の問題点や制限事項を把握しました。  
2. コンテナのみの環境でRHELのPodman/Docker、Podman-compose/Docker-composeを商用環境で利用している企業の事例を調査しました。特に、失敗事例、障害事例、サービス影響が出た事例に焦点を当てました。  
3. 上記の事例について、原因、影響範囲、対応策を具体的に調査しました。  
4. コンテナセキュリティ、リソース管理、モニタリング、ログ管理など、商用環境で考慮すべき点について調査し、RHELのPodman/Docker、Podman-compose/Docker-composeでどのように対応できるかを検討しました。  
5. 障害発生時の切り戻し手順、バックアップ/リストア方法など、可用性確保のための対策を調査しました。  
6. パフォーマンスチューニング、スケーラビリティ、運用効率化など、商用環境で求められる要件を満たすための方法を調査しました。  
7. コンテナ技術の最新動向、RHELのPodman/Docker、Podman-compose/Docker-composeのロードマップなどを調査し、将来的なリスクを検討しました。

## **RHELにおけるPodman/Dockerの既知の問題点と制限事項**

RHEL 9で利用可能なPodmanの最新バージョンでは、インターネットに接続されていないホストでコンテナを実行しようとすると、rootless Podmanのネットワークスタックを構成する pasta(1) が原因でエラーが発生するケースが報告されています。2 これは、pasta(1) がコンテナにネットワーク接続を提供するために、デフォルトでインターフェースからアドレスとルートを取得する必要があるためです。インターフェースが存在しない、またはインターフェースにアドレスがない場合、pasta(1) は起動を拒否します。

この問題を回避するには、--network host オプションを指定してコンテナを実行する必要があります。ただし、--network host オプションを使用すると、コンテナはホストのネットワークスタックを共有するため、セキュリティリスクが高まります。そのため、インターネットに接続されていないホストでPodmanを使用する場合は、pasta(1) の設定を調整するか、別のネットワークソリューションを検討する必要があります。

## **Podman/Dockerの商用利用における失敗事例・障害事例**

### **事例1：ネットワーク接続の喪失によるコンテナ起動エラー 2**

RHEL 9.5環境において、Podmanの最新バージョンにアップデートした後、インターネットに接続されていないホストでコンテナを実行しようとすると、pasta(1) のエラーが発生し、コンテナが起動できないという問題が発生しました。

**原因:**

前述の通り、pasta(1) はコンテナにネットワーク接続を提供するために、デフォルトでインターフェースからアドレスとルートを取得する必要があるため、インターフェースにアドレスがない場合、起動を拒否します。

**影響範囲:**

インターネットに接続されていないホストで実行されている全てのコンテナが起動できなくなりました。

**対応策:**

* \--network host オプションを指定してコンテナを実行する。  
* pasta(1) の設定を調整し、アドレスとルートを手動で設定する。2  
* 別のネットワークソリューションを検討する。

**教訓:**

Podmanのネットワーク設定は、環境によっては注意深く調整する必要があります。特に、インターネットに接続されていないホストで使用する場合は、pasta(1) の設定や代替のネットワークソリューションについて事前に検討しておくことが重要です。

### **事例2：イメージレジストリへの認証エラー 3**

RHEL 9環境において、Podmanをバージョン4.9.4にアップデートした後、docker.io以外のレジストリからイメージを取得してコンテナを実行しようとすると、認証エラーが発生し、コンテナが起動できなくなりました。

**原因:**

Podmanのアップデートにより、docker.io以外のレジストリに対する認証処理に変更があった可能性があります。

**影響範囲:**

docker.io以外のレジストリからイメージを取得して実行している全てのコンテナが起動できなくなりました。

**対応策:**

* Podmanのバージョンを4.6.1にダウングレードする。  
* 認証情報を再設定する。  
* レジストリ側の設定を確認する。

**教訓:**

Podmanのアップデートは、既存のコンテナ環境に影響を与える可能性があります。アップデートを行う前に、変更内容を十分に確認し、必要があればテスト環境で動作検証を行うことが重要です。

### **事例3：ビルド時のレイヤーキャッシュの不具合とCPU使用率の増加 4**

本番環境でPodmanを使用していた際に、ビルド時のレイヤーキャッシュが正常に動作せず、ビルド時間が大幅に増加するという問題が発生しました。また、fuse-overlayfs が原因でCPU使用率が常に100%に達し、システムパフォーマンスに悪影響を及ぼしました。

**原因:**

Podmanのレイヤーキャッシュの不具合、およびfuse-overlayfs のパフォーマンス問題が原因と考えられます。

**影響範囲:**

* アプリケーションのビルド時間が大幅に増加し、開発効率が低下しました。  
* CPU使用率の増加により、システム全体のパフォーマンスが低下し、他のサービスにも影響が出ました。

**対応策:**

* Docker CEに切り替えることで、ビルド時間とCPU使用率の問題を解決しました。  
* 将来的には、Podmanのバージョンアップや設定変更によってこれらの問題が解決される可能性があります。

**教訓:**

Podmanはまだ発展途上の技術であり、本番環境での利用には注意が必要です。パフォーマンスや安定性に問題が発生する可能性があるため、Docker CEなどの代替手段も検討する必要があります。

## **Podman-compose/Docker-composeの商用利用における失敗事例・障害事例**

### **事例4：Podman ComposeでのComposeファイル実行エラー 5**

ArchiveBoxのdocker-compose.ymlファイルをPodman Composeで実行しようとすると、エラーが発生し、コンテナが起動できませんでした。

**原因:**

Podman Composeがdocker-compose.ymlファイルの dockerfile\_inline 命令に対応していないことが原因と考えられます。

**影響範囲:**

dockerfile\_inline 命令を使用しているComposeファイルが実行できなくなりました。

**対応策:**

* dockerfile\_inline 命令を使用しないようにComposeファイルを修正する。  
* Docker Composeを使用する。

**教訓:**

Podman ComposeはDocker Composeの全ての機能をサポートしているわけではありません。そのため、Docker Composeで作成されたComposeファイルを使用する場合は、互換性について事前に確認しておく必要があります。

### **事例5：Podman Composeでのsecrets定義エラー 6**

Podman Composeで environment: にsecretsを定義すると、エラーが発生し、Composeファイルが正しく解析されませんでした。

**原因:**

Podman Composeが environment: で定義されたsecretsに対応していないことが原因と考えられます。

**影響範囲:**

environment: でsecretsを定義しているComposeファイルが実行できなくなりました。

**対応策:**

* secretsを別の方法で定義する。  
* Docker Composeを使用する。

**教訓:**

Podman ComposeはDocker Composeの全ての機能をサポートしているわけではありません。7 そのため、Docker Composeで作成されたComposeファイルを使用する場合は、互換性について事前に確認しておく必要があります。

## **コンテナセキュリティ**

Podmanは、Dockerと比較してセキュリティ面でいくつかの利点があります。

* **デーモンレスアーキテクチャ:** Podmanはデーモンを必要としないため、攻撃対象領域が少なく、セキュリティリスクを低減できます。1 Dockerでは、デーモンがroot権限で動作するため、デーモンに脆弱性があると、ホストシステム全体が侵害されるリスクがあります。一方、Podmanはデーモンレスであるため、そのようなリスクを回避できます。  
* **rootlessコンテナ:** Podmanはrootlessコンテナをネイティブにサポートしており、root権限なしでコンテナを実行できます。1 これにより、コンテナが侵害された場合でも、ホストシステムへのrootアクセスを防ぐことができます。rootlessコンテナでは、コンテナ内のプロセスは、コンテナを実行するユーザーの権限で実行されます。そのため、コンテナが侵害された場合でも、攻撃者はそのユーザーの権限しか取得できません。  
* **ユーザーネームスペース:** Podmanはユーザーネームスペースをサポートしており、コンテナをさらに分離することができます。9 ユーザーネームスペースを使用すると、コンテナ内のプロセスは、ホストシステムとは異なるユーザーIDとグループIDで実行されます。これにより、コンテナ間の分離が強化され、セキュリティリスクを低減できます。  
* **SELinuxのサポート:** PodmanはSELinuxをサポートしており、コンテナのアクセス制御を強化することができます。1 SELinuxは、Linuxカーネルのセキュリティ機能であり、プロセスがアクセスできるリソースを制限することができます。PodmanはSELinuxと連携することで、コンテナのアクセス制御をより厳密に行うことができます。

これらのセキュリティ機能により、Podmanは商用環境でより安全なコンテナ実行環境を提供することができます。

## **リソース管理**

Podmanは、コンテナのリソース使用量を制限するための様々なオプションを提供しています。10 これらのオプションを使用することで、コンテナが過剰なリソースを使用することを防ぎ、ホストシステムの安定性を確保することができます。

* **メモリ制限:** \--memory オプションを使用して、コンテナが使用できるメモリの最大量を指定できます。11 また、--memory-swap オプションを使用して、スワップを含むメモリ使用量を制限することもできます。12  
* **CPU制限:** \--cpus オプションを使用して、コンテナが使用できるCPUリソースの量を制限できます。13 また、--cpu-shares オプションを使用して、CPUリソースの相対的な重みを指定することもできます。  
* **ブロックIOの重み付け:** \--blkio-weight オプションを使用して、コンテナのブロックIOの相対的な重みを指定できます。11 これにより、IO負荷の高いコンテナが他のコンテナのパフォーマンスに影響を与えることを防ぐことができます。

## **モニタリング**

Podmanは、コンテナの稼働状況を監視するためのツールを提供しています。これらのツールを使用することで、コンテナの健全性を監視し、問題が発生した場合に迅速に対応することができます。

* **podman stats:** コンテナのリソース使用状況に関するリアルタイムの統計情報を表示します。12 CPU使用率、メモリ使用量、ネットワークI/Oなどの情報を確認できます。  
* **podman events:** Podmanイベントを監視します。10 コンテナの起動、停止、作成、削除などのイベントをリアルタイムに追跡できます。  
* **podman top:** コンテナで実行されているプロセスを表示します。10 コンテナ内でどのプロセスが実行されているか、CPUやメモリをどの程度使用しているかを確認できます。

## **ログ管理**

Podmanは、コンテナのログを管理するためのオプションを提供しています。これらのオプションを使用することで、ログの保存場所、ログのローテーション、ログのフォーマットなどをカスタマイズすることができます。

* **\--log-driver:** ログドライバを指定します。11 journald、json-file、syslog などのログドライバを選択できます。  
* **\--log-opt:** ログドライバのオプションを指定します。11 例えば、max-size オプションでログファイルの最大サイズを指定したり、max-file オプションでログファイルの最大数を指定したりできます。

## **可用性確保のための対策**

### **バックアップ/リストア方法**

Podmanは、コンテナのチェックポイントを作成およびリストアするための機能を提供しています。14 チェックポイントは、実行中のコンテナの状態をフリーズし、メモリの内容と状態をディスクに保存します。これにより、コンテナを別のホストに移動したり、スナップショットを作成したり、リモートデバッグを行ったりすることができます。チェックポイントの作成には podman container checkpoint コマンドを、リストアには podman container restore コマンドを使用します。

また、Podmanはボリュームのエクスポートおよびインポート機能を提供しています。15 ボリュームは、コンテナのデータ永続化に使用されます。ボリュームをエクスポートすることで、ボリュームのバックアップを作成することができます。ボリュームをインポートすることで、バックアップからボリュームをリストアすることができます。ボリュームのエクスポートには podman volume export コマンドを、インポートには podman volume import コマンドを使用します。

### **障害発生時の切り戻し手順**

Podmanは、自動アップデート機能とロールバック機能を提供しています。16 自動アップデート機能は、コンテナイメージの新しいバージョンが利用可能になると、自動的にイメージを更新し、コンテナを再起動します。ロールバック機能は、更新されたイメージでコンテナが正常に起動しなかった場合、以前のバージョンに自動的に戻します。

自動アップデート機能を使用するには、podman create \--label io.containers.autoupdate={registry,local} のように、コンテナ作成時に io.containers.autoupdate ラベルを設定します。registry ポリシーはレジストリから最新のイメージを取得し、local ポリシーはローカルのイメージのみを比較します。

ロールバック機能は、Podman v3.4以降で利用可能です。コンテナがsystemdサービスとして実行されている場合、Podmanはサービスの再起動が成功したかどうかを検知し、失敗した場合は以前のバージョンにロールバックします。

## **パフォーマンスチューニング**

Podmanのパフォーマンスを向上させるためには、以下の点に注意する必要があります。

* **高速なOCIランタイムを使用する:** Podmanは、コンテナを実行するためにOCIランタイムを使用します。crun は、runc よりも高速なOCIランタイムです。17 crun を使用することで、コンテナの起動時間や実行速度を向上させることができます。  
* **適切なストレージドライバを選択する:** Podmanは、コンテナイメージやコンテナのファイルシステムを管理するためにストレージドライバを使用します。overlayfs は、fuse-overlayfs や vfs よりも高速なストレージドライバです。17 特に、rootless Podmanを使用する場合は、overlayfs を使用することでパフォーマンスを significantly 向上させることができます。  
* **rootless Podmanのネットワークパフォーマンスを向上させる:** rootless Podmanを使用する場合、ネットワークトラフィックは通常 pasta ネットワークドライバを介して処理されます。pasta は、ユーザーネームスペースでネットワーク接続を提供するためのツールですが、パフォーマンスのオーバーヘッドがあります。ソケットアクティベーションを使用することで、pasta を介さずにネットワークトラフィックを処理することができます。17 これにより、rootless Podmanのネットワークパフォーマンスを向上させることができます。  
* **コンテナイメージの遅延プルを使用する:** コンテナイメージの遅延プルを使用すると、コンテナの起動時にイメージ全体をダウンロードする必要がなくなり、起動時間を短縮することができます。Podmanは、zstd:chunked 形式のコンテナイメージに対して遅延プルをサポートしています。17  
* **高速なホストファイルシステムを選択する:** ホストシステムのファイルシステムは、コンテナのパフォーマンスに影響を与えます。XFS や ext4 は、btrfs よりも高速なファイルシステムです。  
* **\--log-driver オプションを使用する:** ログドライバを指定することで、ログの書き込みパフォーマンスを向上させることができます。17 journald ログドライバは、json-file ログドライバよりも高速です。  
* **コンテナイメージをビルドする際にパッケージリポジトリキャッシュを再利用する:** コンテナイメージをビルドする際に、パッケージリポジトリキャッシュを再利用することで、ビルド時間を短縮することができます。17 Podmanは、--build-arg オプションを使用して、ビルド時にホストシステムのキャッシュディレクトリをコンテナにマウントすることができます。

## **スケーラビリティ**

Podmanは、コンテナのスケーラビリティを向上させるための機能を提供しています。

* **Pod:** Podは、複数のコンテナをグループ化し、一緒に管理するための機能です。1 Podを使用することで、コンテナ間の依存関係を定義し、コンテナをまとめて管理することができます。また、PodはKubernetesのPodと互換性があるため、Podmanで実行しているコンテナをKubernetesクラスタに簡単に移行することができます。  
* **Kubernetesとの統合:** PodmanはKubernetes YAMLを直接使用することができます。18 podman play kube コマンドを使用することで、Kubernetes YAMLファイルに定義されたコンテナやPodをPodmanで実行することができます。また、podman generate kube コマンドを使用することで、Podmanで実行しているコンテナやPodからKubernetes YAMLファイルを生成することができます。

## **運用効率化**

Podmanは、コンテナの運用を効率化するための機能を提供しています。

* **systemdとの統合:** Podmanはsystemdと統合されており、コンテナをシステムサービスとして実行することができます。1 これにより、コンテナの起動、停止、再起動などをsystemdで一元管理することができます。また、systemdのサービス監視機能を利用することで、コンテナの可用性を向上させることもできます。  
* **自動アップデート:** Podmanは自動アップデート機能を提供しており、コンテナイメージの新しいバージョンが利用可能になると、自動的にイメージを更新します。16 これにより、常に最新のイメージを使用することができ、セキュリティリスクを低減することができます。  
* **ロールバック:** Podmanはロールバック機能を提供しており、更新されたイメージでコンテナが正常に起動しなかった場合、以前のバージョンに自動的に戻します。16 これにより、アップデートによるサービス停止のリスクを最小限に抑えることができます。

## **コンテナ技術の最新動向と将来的なリスク**

コンテナ技術は常に進化しており、新しい技術やツールが次々と登場しています。これらの最新動向を踏まえ、RHELのPodman/Docker、Podman-compose/Docker-composeのロードマップを考慮すると、以下の将来的なリスクが考えられます。

* **新技術への対応:** コンテナ技術の進化に追従し、新技術に対応していく必要があります。例えば、WebAssembly (WASM) や eBPF などの新しい技術がコンテナ技術に影響を与える可能性があります。  
* **セキュリティリスク:** 新しい脆弱性や攻撃手法が出現する可能性があります。19 常に最新の情報に注意し、セキュリティ対策を強化していく必要があります。  
* **互換性の問題:** Podman/Docker、Podman-compose/Docker-composeのバージョンアップにより、互換性の問題が発生する可能性があります。21 バージョンアップを行う前に、互換性について十分に確認する必要があります。

### **コンテナ技術の最新動向**

* **ゼロトラストアーキテクチャ:** ゼロトラストアーキテクチャは、ネットワーク内のいかなる暗黙の信頼も想定しないセキュリティモデルです。19 コンテナセキュリティにおいても、ゼロトラストアーキテクチャの採用が重要になっています。  
* **AI/MLの活用:** AI/MLは、コンテナセキュリティの脅威予測、行動分析、自動応答などに活用されています。20 AI/MLを活用することで、コンテナセキュリティをより高度化することができます。  
* **マルチクラウド/ハイブリッド環境:** コンテナは、マルチクラウド/ハイブリッド環境で実行されることが増えています。19 これに伴い、異なるプラットフォーム間で一貫したセキュリティポリシーを確保することが課題となっています。  
* **Policy as Code:** Policy as Codeは、セキュリティポリシーをコードとして定義し、自動的に適用する手法です。19 コンテナ環境においても、Policy as Codeの採用が進んでいます。Policy as Codeを使用することで、セキュリティポリシーの管理を自動化し、セキュリティリスクを低減することができます。  
* **サービスメッシュ:** サービスメッシュは、マイクロサービス間の通信を管理するための技術です。サービスメッシュを使用することで、コンテナ間の通信をセキュアに、そして効率的に行うことができます。

### **Podmanのロードマップ 22**

Podmanのロードマップには、以下の項目が含まれています。

* **セキュリティの強化:** rootlessコンテナの機能強化、セキュリティポリシーの適用など、セキュリティ対策を強化していく予定です。  
* **パフォーマンスの向上:** コンテナの起動時間や実行速度を向上させるための取り組みを継続していきます。  
* **Kubernetesとの統合強化:** Kubernetesとの統合を強化し、Podmanで実行しているコンテナをKubernetesクラスタに簡単に移行できるようにしていきます。  
* **新機能の追加:** コンテナの運用管理を効率化するための新機能を追加していく予定です。

## **結論**

RHEL環境におけるPodman/Docker、Podman-compose/Docker-composeの商用利用は、適切な設定と運用を行うことで、セキュリティとパフォーマンスの両面で利点があります。しかし、いくつかの既知の問題点や制限事項、そして将来的なリスクも存在します。

商用環境でPodman/Docker、Podman-compose/Docker-composeを使用する場合は、これらの問題点やリスクを理解し、適切な対策を講じる必要があります。具体的には、以下の点が重要です。

* 最新の情報に注意し、常に最新バージョンを使用する。  
* セキュリティ対策を強化し、rootlessコンテナやSELinuxなどのセキュリティ機能を活用する。  
* リソース管理を行い、コンテナが過剰なリソースを使用することを防ぐ。  
* 適切なモニタリングとログ管理を行い、コンテナの健全性を監視する。  
* 障害発生時の切り戻し手順、バックアップ/リストア方法を確立しておく。  
* パフォーマンスチューニングを行い、コンテナのパフォーマンスを向上させる。  
* スケーラビリティを考慮し、必要に応じてPodやKubernetesなどの技術を活用する。  
* 運用効率化を図り、systemdとの統合や自動アップデートなどの機能を活用する。

これらの点を考慮することで、RHEL環境におけるPodman/Docker、Podman-compose/Docker-composeの商用利用を成功させることができます。 特に、PodmanはDockerと比較してセキュリティ面で優れているため、セキュリティを重視する商用環境ではPodmanの利用を検討する価値があります。しかし、Podman ComposeはDocker Composeの全ての機能をサポートしているわけではないため、Docker Composeで作成されたComposeファイルを使用する場合は、互換性について事前に確認しておく必要があります。

#### **引用文献**

1\. What is Podman? \- Red Hat, 2月 23, 2025にアクセス、 [https://www.redhat.com/en/topics/containers/what-is-podman](https://www.redhat.com/en/topics/containers/what-is-podman)  
2\. Latest version of Podman available for RHEL 9 fails when trying to run container on disconnected host \#24614 \- GitHub, 2月 23, 2025にアクセス、 [https://github.com/containers/podman/issues/24614](https://github.com/containers/podman/issues/24614)  
3\. Podman fails to run containers if image is not from docker.io after ..., 2月 23, 2025にアクセス、 [https://github.com/containers/podman/issues/22578](https://github.com/containers/podman/issues/22578)  
4\. I've been trying to use podman in production and it is not working ..., 2月 23, 2025にアクセス、 [https://news.ycombinator.com/item?id=26105104](https://news.ycombinator.com/item?id=26105104)  
5\. Podman Compose failing on Archivebox docker-compose.yml ..., 2月 23, 2025にアクセス、 [https://github.com/containers/podman-compose/issues/1065](https://github.com/containers/podman-compose/issues/1065)  
6\. podman-compose build does not support environment type secrets ..., 2月 23, 2025にアクセス、 [https://github.com/containers/podman-compose/issues/1066](https://github.com/containers/podman-compose/issues/1066)  
7\. Using Podman and Docker Compose \- Red Hat, 2月 23, 2025にアクセス、 [https://www.redhat.com/en/blog/podman-docker-compose](https://www.redhat.com/en/blog/podman-docker-compose)  
8\. Security: Docker, Podman and Rootless Containers \- JOHNAIC: Personal AI Computer, 2月 23, 2025にアクセス、 [https://von-neumann.ai/blog/security-rootless-containers.html](https://von-neumann.ai/blog/security-rootless-containers.html)  
9\. Exploring Podman: A More Secure Docker Alternative | Better Stack Community, 2月 23, 2025にアクセス、 [https://betterstack.com/community/guides/scaling-docker/podman-vs-docker/](https://betterstack.com/community/guides/scaling-docker/podman-vs-docker/)  
10\. containers.conf \- Podman documentation, 2月 23, 2025にアクセス、 [https://docs.podman.io/en/stable/markdown/podman.1.html](https://docs.podman.io/en/stable/markdown/podman.1.html)  
11\. podman-run, 2月 23, 2025にアクセス、 [https://docs.podman.io/en/v5.1.1/markdown/podman-run.1.html](https://docs.podman.io/en/v5.1.1/markdown/podman-run.1.html)  
12\. Understanding Memory Management in Containers with Podman | by Jason Bell \- Medium, 2月 23, 2025にアクセス、 [https://jasebell.medium.com/understanding-memory-management-in-containers-with-podman-e916ea9039df](https://jasebell.medium.com/understanding-memory-management-in-containers-with-podman-e916ea9039df)  
13\. Resource constraints \- Docker Docs, 2月 23, 2025にアクセス、 [https://docs.docker.com/engine/containers/resource\_constraints/](https://docs.docker.com/engine/containers/resource_constraints/)  
14\. Podman Checkpointing with Volume Restore: A Step-by-Step Guide ..., 2月 23, 2025にアクセス、 [https://medium.com/@shubham.jadhav\_18910/podman-checkpointing-with-volume-restore-a-step-by-step-guide-d5960a182940](https://medium.com/@shubham.jadhav_18910/podman-checkpointing-with-volume-restore-a-step-by-step-guide-d5960a182940)  
15\. Podman Container Backups \- Fedora Discussion, 2月 23, 2025にアクセス、 [https://discussion.fedoraproject.org/t/podman-container-backups/124758](https://discussion.fedoraproject.org/t/podman-container-backups/124758)  
16\. How to use auto-updates and rollbacks in Podman \- Red Hat, 2月 23, 2025にアクセス、 [https://www.redhat.com/en/blog/podman-auto-updates-rollbacks](https://www.redhat.com/en/blog/podman-auto-updates-rollbacks)  
17\. podman/docs/tutorials/performance.md at main \- GitHub, 2月 23, 2025にアクセス、 [https://github.com/containers/podman/blob/main/docs/tutorials/performance.md](https://github.com/containers/podman/blob/main/docs/tutorials/performance.md)  
18\. Podman, 2月 23, 2025にアクセス、 [https://podman.io/](https://podman.io/)  
19\. Container Security in 2024: Trends and Best Practices \- DEV Community, 2月 23, 2025にアクセス、 [https://dev.to/giladmaayan/container-security-in-2024-trends-and-best-practices-of5](https://dev.to/giladmaayan/container-security-in-2024-trends-and-best-practices-of5)  
20\. The Future of Container Security: Trends and Open Source Solutions, 2月 23, 2025にアクセス、 [https://linuxsecurity.com/features/future-of-container-security](https://linuxsecurity.com/features/future-of-container-security)  
21\. Docker Installation Failed on Red Hat and CentOS 8 Due to Podman and Containers-Common \- Medium, 2月 23, 2025にアクセス、 [https://medium.com/@wasiualhasib/docker-installation-failed-on-red-hat-and-centos-8-due-to-podman-and-containers-common-946ea5757229](https://medium.com/@wasiualhasib/docker-installation-failed-on-red-hat-and-centos-8-due-to-podman-and-containers-common-946ea5757229)  
22\. podman/README.md at main \- GitHub, 2月 23, 2025にアクセス、 [https://github.com/containers/podman/blob/main/README.md](https://github.com/containers/podman/blob/main/README.md)
