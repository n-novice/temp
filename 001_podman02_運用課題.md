# **RHELにおけるPodman/DockerおよびPodman-Composeの商用環境利用に関する調査レポート**

## **概要**

本レポートは、Red Hat Enterprise Linux (RHEL) におけるPodman/DockerおよびPodman-Composeを商用環境で利用する際の不具合や問題点、課題を整理することを目的としています。

調査期間は2020年から2025年とし、Red Hat Bugzilla、Red Hat Knowledgebase、関連するフォーラムやコミュニティサイト、技術ブログや記事などを情報源としています。

## **不具合・問題点**

### **Podman**

* **Podman 4.9.4 にアップデート後、docker.io 以外のレジストリからのイメージでコンテナを実行できない** 1  
  * 概要：quadlet ファイルで docker.io 以外のレジストリからのイメージを指定すると、systemctl \--user daemon-reload 実行時にエラーが発生し、コンテナを実行できない。  
  * 発生条件：Podman 4.9.4、docker.io 以外のレジストリからのイメージを使用  
  * 原因：不明  
  * 対策：不明  
  * 影響範囲：docker.io 以外のレジストリからのイメージを使用するユーザー  
  * ステータス：未解決  
  * 課題の分類：バグ  
* **RHEL 9 で最新版の Podman を使用し、インターネットに接続されていないホストでコンテナを実行しようとすると失敗する** 2  
  * 概要：インターネットに接続されていない環境でコンテナを作成・起動しようとすると、"pasta failed with exit code 1: External interface not usable." というエラーメッセージが表示され、失敗する。  
  * 発生条件：RHEL 9、最新版の Podman、インターネットに接続されていない環境  
  * 原因：不明  
  * 対策：不明  
  * 影響範囲：インターネットに接続されていない環境で Podman を使用するユーザー  
  * ステータス：未解決  
  * 課題の分類：バグ、ネットワーク  
* **RHEL 7 で Podman のアップグレードがコンテナに問題を引き起こす** 3  
  * 概要：Podman のアップグレード後、コンテナの起動に失敗する。  
  * 発生条件：RHEL 7、Podman のアップグレード  
  * 原因：不明  
  * 対策：不明  
  * 影響範囲：RHEL 7 で Podman を使用するユーザー  
  * ステータス：未解決  
  * 課題の分類：バグ  
* **CentOS 8 で Docker をインストールしようとすると、Podman パッケージでエラーが発生する** 4  
  * 概要：CentOS 8 に Docker をインストールしようとすると、Podman パッケージに関するエラーが発生する。  
  * 発生条件：CentOS 8、Docker のインストール  
  * 原因：podman、containerd.io、docker-ce パッケージ間の依存関係の競合  
  * 対策：yum コマンドで containerd.io と docker-ce をインストールする前に、podman と関連パッケージをアンインストールする。 4  
  * 影響範囲：CentOS 8 に Docker をインストールするユーザー  
  * ステータス：回避策あり  
  * 課題の分類：互換性  
* **Oracle Linux 8 で特権のないユーザーとして Quadlet を実行すると失敗する** 5  
  * 概要：Quadlet がルート権限なしで Oracle Linux 8 上で実行できない。  
  * 発生条件：Oracle Linux 8、ルート権限なし  
  * 原因：特権のないユーザーが cgroup ファイルシステムに書き込むためのアクセス権を持っていない。  
  * 対策：ルートユーザーで Quadlet を実行するか、Oracle Linux 9 にアップグレードする。  
  * 影響範囲：Oracle Linux 8 でルートレス Podman を使用するユーザー  
  * ステータス：回避策あり  
  * 課題の分類：セキュリティ  
* **Oracle Linux 8 の特定の環境で、特権のないユーザーがコンテナストレージにアクセスできない** 5  
  * 概要：一部の環境では、特権のないユーザーに対してコンテナストレージのマウントに失敗する可能性がある。  
  * 発生条件：Oracle Linux 8、特権のないユーザー、特定の環境  
  * 原因：不明  
  * 対策：/etc/containers/storage.conf の mountopt に index=off パラメータを追加する。  
  * 影響範囲：Oracle Linux 8 でルートレス Podman を使用するユーザー  
  * ステータス：回避策あり  
  * 課題の分類：セキュリティ  
* **X509 証明書がレガシーな共通名フィールドに依存している** 5  
  * 概要：Podman バージョン 3.0 以降では、適切なサブジェクト代替名 (SAN) が含まれていない証明書に対して、TLS 証明書の検証を必要とする Podman コマンドがエラーを返す。  
  * 発生条件：Podman バージョン 3.0 以降、SAN が設定されていない自己署名証明書を使用  
  * 原因：Podman のビルドに使用される Go コンパイラの新しいバージョン  
  * 対策：証明書を更新して適切な SAN を使用するか、GODEBUG 環境変数を x509ignoreCN=0 に設定する。  
  * 影響範囲：自己署名証明書を使用するローカルまたはプライベートなイメージレジストリを操作するユーザー  
  * ステータス：回避策あり  
  * 課題の分類：セキュリティ  
* **コンテナが利用できない場合、podman attach \--latest の実行でパニックが発生する** 5  
  * 概要：podman attach \--latest コマンドを実行したときに、環境にコンテナが存在しないと、ランタイムエラーが発生する。  
  * 発生条件：コンテナが存在しない環境で podman attach \--latest コマンドを実行  
  * 原因：不明  
  * 対策：コンテナを作成してからコマンドを実行する。  
  * 影響範囲：podman attach \--latest コマンドを使用するユーザー  
  * ステータス：未解決  
  * 課題の分類：バグ  
* **コンテナをデタッチするためのデフォルトのキーシーケンスを使用するには、-t オプションが必要** 5  
  * 概要：コンテナをデタッチするためのデフォルトのキーシーケンス (CTRL+P, CTRL+Q) を使用するには、コンテナの作成時に \-t オプションを使用する必要がある。  
  * 発生条件：-t オプションを指定せずにコンテナを作成  
  * 原因：不明  
  * 対策：コンテナ作成時に \-t オプションを指定する。  
  * 影響範囲：デフォルトのキーシーケンスでコンテナをデタッチするユーザー  
  * ステータス：回避策あり  
  * 課題の分類：操作性  
* **podman pull コマンドでイメージをプルしようとすると、正しい名前または完全な名前を指定しないと、認証エラーが発生する** 5  
  * 概要：podman pull image-name コマンドを実行してイメージをプルしようとすると、正しい名前または完全な名前を指定しないと、認証エラーが発生する。  
  * 発生条件：イメージ名にレジストリ名を含めない  
  * 原因：不明  
  * 対策：イメージ名を完全な形式で指定する (例：docker.io/library/fedora)。  
  * 影響範囲：podman pull コマンドを使用するユーザー  
  * ステータス：回避策あり  
  * 課題の分類：操作性  
* **Docker Hub から oraclelinux イメージを検索またはプルしようとすると、「manifest unknown」エラーで失敗する** 5  
  * 概要：oraclelinux イメージを Docker Hub から検索またはプルしようとすると、「manifest unknown」エラーで失敗する。  
  * 発生条件：Docker Hub から oraclelinux イメージを検索またはプルしようとすると  
  * 原因：skopeo ユーティリティがデフォルトで latest タグが存在することを想定している。  
  * 対策：oraclelinux:latest またはタグなしのダウンストリームイメージを oraclelinux:7 に更新する。  
  * 影響範囲：Docker Hub から oraclelinux イメージを検索またはプルするユーザー  
  * ステータス：回避策あり  
  * 課題の分類：互換性

### **Podman-Compose**

* **podman-compose up コマンドが iptables テーブルフィルターの非互換性エラーを出力する** 6  
  * 概要：sudo podman-compose up コマンドを実行すると、iptables テーブルフィルターの非互換性エラーが出力される。  
  * 発生条件：RHEL 8.7、podman-compose up コマンドの実行  
  * 原因：iptables テーブルフィルターの非互換性  
  * 対策：nft ツールを使用する。  
  * 影響範囲：RHEL 8.7 で podman-compose を使用するユーザー  
  * ステータス：回避策あり  
  * 課題の分類：互換性  
* **podman-compose がボリュームパスを見つけられない** 7  
  * 概要：podman-compose が docker-compose.yml で指定されたボリュームパスを見つけられない。  
  * 発生条件：ボリュームパスが存在しない  
  * 原因：podman-compose が Docker Compose のように、存在しないボリュームパスを自動的に作成しない。  
  * 対策：ボリュームパスを手動で作成する。  
  * 影響範囲：podman-compose を使用するユーザー  
  * ステータス：回避策あり  
  * 課題の分類：互換性  
* **podman-compose で userns\_mode: "keep-id" を使用しても、コンテナ内のすべてのユーザーがホストユーザーにマッピングされない** 8  
  * 概要：userns\_mode: "keep-id" を使用しても、コンテナ内のすべてのユーザーがホストユーザーにマッピングされない。  
  * 発生条件：userns\_mode: "keep-id" を使用、複数のユーザーがボリュームにアクセスするコンテナ  
  * 原因：userns\_mode: "keep-id" は1対1のマッピングのみをサポートし、複数のIDを単一のIDに統合しない。  
  * 対策：バインドマウントディレクトリを使用する。  
  * 影響範囲：userns\_mode: "keep-id" を使用し、複数のユーザーがボリュームにアクセスするコンテナを使用するユーザー  
  * ステータス：回避策あり  
  * 課題の分類：セキュリティ  
* **podman compose pull が Podman マシンをクラッシュさせる** 9  
  * 概要：podman compose pull を実行すると、Podman マシンがクラッシュする。  
  * 発生条件：podman compose pull の実行  
  * 原因：gvproxy のクラッシュ  
  * 対策：podman \--log-level debug machine start で Podman マシンを起動する。  
  * 影響範囲：podman compose pull を使用する macOS arm アーキテクチャのユーザー  
  * ステータス：回避策あり  
  * 課題の分類：バグ  
* **Fedora で podman、podman-docker、docker-compose を同時にインストールできない** 10  
  * 概要：Fedora で podman、podman-docker、docker-compose を同時にインストールしようとすると競合が発生する。  
  * 発生条件：Fedora で podman、podman-docker、docker-compose を同時にインストールしようとすると  
  * 原因：docker-compose パッケージが docker-cli のインストールを明示的に要求するため。  
  * 対策：podman compose または docker compose ( podman-docker がインストールされている場合) コマンドを使用する。  
  * 影響範囲：Fedora で podman、podman-docker、docker-compose を同時に使用したいユーザー  
  * ステータス：回避策あり  
  * 課題の分類：互換性

## **Podman Desktop**

Podman Desktopは、Podman、Buildah、Skopeo、Kindなどのコンテナツールをデスクトップ環境で管理するためのオープンソースのアプリケーションです。 11 12

### **メリット**

* GUIでコンテナやイメージを管理できるため、コマンドラインに慣れていないユーザーでも容易に操作できる。  
* 複数のコンテナエンジンをサポートしており、PodmanだけでなくDockerも管理できる。  
* Kubernetesとの連携機能があり、Kindを使用してローカルにKubernetesクラスターを構築し、Podman Desktopから管理することができる。  
* 拡張機能をサポートしており、Red Hatアカウントとの連携や、追加のツールを統合することができる。 12

### **デメリット**

* デスクトップ環境で動作するため、サーバー環境での利用には適さない。  
* まだ開発途上のツールであるため、安定性に欠ける部分がある。

### **商用環境での利用における注意点**

* 商用環境で利用する場合は、サポート体制や安定性などを考慮する必要がある。  
* 重要なシステムの運用には、十分な検証を行った上で利用する必要がある。

## **課題**

RHEL の Podman/Docker および Podman-Compose を商用環境で利用する際の課題は以下の点が挙げられます。

* **セキュリティ**  
  * ルートレスコンテナは、従来のルートで実行されるコンテナに比べてセキュリティリスクが低いため、商用環境での利用に適しています。しかし、ルートレスコンテナであっても、適切なセキュリティ対策を講じる必要があります。 13 14  
  * イメージの信頼性を確保するために、信頼できるレジストリからイメージを取得し、デジタル署名などを用いて検証する必要があります。 13  
  * SELinuxやAppArmorなどのセキュリティモジュールを活用することで、コンテナのセキュリティをさらに強化することができます。 13 15  
  * 特に、ルートレスコンテナでは、SELinuxラベルの設定が重要になります。コンテナがアクセスできるファイルやディレクトリを制限するために、適切なラベルを設定する必要があります。 16  
  * また、ユーザーネームスペースのマッピングを設定することで、コンテナ内のユーザーとホストのユーザーを分離し、セキュリティを向上させることができます。 16  
* **パフォーマンス**  
  * コンテナイメージのサイズを小さくすることで、起動時間やディスク使用量を削減し、パフォーマンスを向上させることができます。マルチステージビルドなどの手法を用いて、イメージの最適化を行うことが重要です。 14  
  * コンテナのリソース使用量を監視し、必要に応じてCPUやメモリの制限を設定することで、リソースの枯渇を防ぎ、安定したパフォーマンスを維持することができます。 14  
  * 具体的なツールとしては、podman statsコマンドやcgroupsなどを利用することができます。  
* **互換性**  
  * Podman は Docker と高い互換性がありますが、Docker Compose と Podman Compose は完全な互換性があるわけではありません。Docker Compose を使用していた場合は、Podman Compose での動作確認が必要です。 17 18  
  * また、Podman は Docker Swarm をサポートしていません。代わりに、Kubernetes を使用することができます。 19  
  * 他のツールとの連携については、Podman は Kubernetes との親和性が高く、podman generate kube コマンドや podman play kube コマンドを使用して、Kubernetes の YAML ファイルを生成したり、Podman で Kubernetes の Pod を実行したりすることができます。 14 18  
* **運用管理**  
  * Podman は systemd と統合されており、コンテナを systemd サービスとして管理することができます。これにより、コンテナの起動、停止、再起動などを systemd で一元管理することができ、運用管理の効率化に繋がります。 20 15  
  * 一方で、systemd の設定に不慣れな場合は、運用管理が複雑になる可能性もあります。  
  * podman generate systemd コマンドを使用することで、コンテナの systemd サービス定義ファイルを生成することができます。また、.container ファイルを作成することで、コンテナの設定を systemd で管理することができます。  
  * ログ管理については、Podman は systemd のジャーナル機能と連携しており、コンテナのログを systemd のジャーナルに記録することができます。 14  
  * ネットワーク管理については、Podman は CNI (Container Network Interface) を使用してコンテナのネットワークを管理します。 14  
* **サポート**  
  * RHEL で Podman/Docker および Podman-Compose を商用環境で利用する場合、Red Hat の商用サポートを利用することができます。 21  
  * また、Podman は活発なコミュニティがあり、フォーラムやメーリングリストなどでサポートを受けることができます。 22

## **注意点・推奨事項**

| 注意点・推奨事項 | 説明 |
| :---- | :---- |
| 最新バージョンを使用する | 最新バージョンには、バグ修正やセキュリティ対策が施されているため、常に最新バージョンを使用することが推奨されます。 |
| セキュリティ対策を適切に実施する | 本レポートで挙げたセキュリティに関する推奨事項を参考に、適切なセキュリティ対策を実施してください。 |
| パフォーマンスを考慮した設定を行う | 本レポートで挙げたパフォーマンスに関する推奨事項を参考に、パフォーマンスを考慮した設定を行ってください。 |
| Docker Compose との互換性に注意する | Docker Compose と Podman Compose は完全な互換性があるわけではないため、Docker Compose を使用していた場合は、Podman Compose での動作確認が必要です。 |
| 必要に応じて、商用サポートを利用する | 商用環境で Podman/Docker および Podman-Compose を利用する場合は、必要に応じて Red Hat の商用サポートを利用することを検討してください。 |
| コンテナの自動更新を検討する | コンテナの自動更新は、最新の状態を維持するために有効ですが、予期せぬ問題が発生する可能性もあるため、注意が必要です。 23 |
| 運用管理ツールを検討する | Portainer や Cockpit などの運用管理ツールを活用することで、コンテナの管理を効率化することができます。 23 |
| リソース監視ツールを検討する | Cadvisor などのリソース監視ツールを活用することで、コンテナのリソース使用量を把握し、パフォーマンスの最適化に役立てることができます。 23 |

## **結論**

RHEL の Podman/Docker および Podman-Compose は、コンテナ化技術を活用するための強力なツールですが、商用環境で利用する際にはいくつかの不具合や課題が存在します。

本レポートでは、Podman/Docker および Podman-Compose の概要、不具合や問題点、商用環境で利用する際の課題、注意点と推奨事項などをまとめました。

これらの情報を参考に、適切な対策を講じることで、リスクを軽減し、安定した運用を実現することができます。

### **今後の展望**

Podman は、Docker の代替として注目されているコンテナエンジンであり、今後も活発な開発が続けられると予想されます。特に、Kubernetes との連携機能が強化され、商用環境での利用がさらに促進されると考えられます。

Docker Compose v2 は、Go で書き直された新しいバージョンであり、Podman でもサポートされています。Buildkit API のサポートも予定されており、今後の動向に注目です。

### **最終的な推奨事項**

* 商用環境で Podman/Docker および Podman-Compose を利用する場合は、最新バージョンを使用し、セキュリティ対策を適切に実施してください。  
* パフォーマンスを考慮した設定を行い、Docker Compose との互換性に注意してください。  
* 必要に応じて、Red Hat の商用サポートやコミュニティサポートを利用してください。

## **付録**

### **不具合・問題点・課題一覧**

| ID | 概要 | 発生条件 | 原因 | 対策 | 影響範囲 | ステータス | 課題の分類 | 関連情報へのリンク |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| 1 | Podman 4.9.4 にアップデート後、docker.io 以外のレジストリからのイメージでコンテナを実行できない | Podman 4.9.4、docker.io 以外のレジストリからのイメージを使用 | 不明 | 不明 | docker.io 以外のレジストリからのイメージを使用するユーザー | 未解決 | バグ | [https://github.com/containers/podman/issues/22578](https://github.com/containers/podman/issues/22578) |
| 2 | RHEL 9 で最新版の Podman を使用し、インターネットに接続されていないホストでコンテナを実行しようとすると失敗する | RHEL 9、最新版の Podman、インターネットに接続されていない環境 | 不明 | 不明 | インターネットに接続されていない環境で Podman を使用するユーザー | 未解決 | バグ、ネットワーク | [https://github.com/containers/podman/issues/24614](https://github.com/containers/podman/issues/24614) |
| 3 | RHEL 7 で Podman のアップグレードがコンテナに問題を引き起こす | RHEL 7、Podman のアップグレード | 不明 | 不明 | RHEL 7 で Podman を使用するユーザー | 未解決 | バグ | [https://github.com/containers/podman/issues/14936](https://github.com/containers/podman/issues/14936) |
| 4 | CentOS 8 で Docker をインストールしようとすると、Podman パッケージでエラーが発生する | CentOS 8、Docker のインストール | podman、containerd.io、docker-ce パッケージ間の依存関係の競合 | yum コマンドで containerd.io と docker-ce をインストールする前に、podman と関連パッケージをアンインストールする。 | CentOS 8 に Docker をインストールするユーザー | 回避策あり | 互換性 | [https://forums.docker.com/t/problem-with-installed-package-podman/116529](https://forums.docker.com/t/problem-with-installed-package-podman/116529) |
| 6 | podman-compose up コマンドが iptables テーブルフィルターの非互換性エラーを出力する | RHEL 8.7、podman-compose up コマンドの実行 | iptables テーブルフィルターの非互換性 | nft ツールを使用する。 | RHEL 8.7 で podman-compose を使用するユーザー | 回避策あり | 互換性 | [https://github.com/containers/podman-compose/issues/637](https://github.com/containers/podman-compose/issues/637) |
| 7 | podman-compose がボリュームパスを見つけられない | ボリュームパスが存在しない | podman-compose が Docker Compose のように、存在しないボリュームパスを自動的に作成しない。 | ボリュームパスを手動で作成する。 | podman-compose を使用するユーザー | 回避策あり | 互換性 | [https://stackoverflow.com/questions/61966654/podman-compose-failing-to-compose](https://stackoverflow.com/questions/61966654/podman-compose-failing-to-compose) |
| 8 | podman-compose で userns\_mode: "keep-id" を使用しても、コンテナ内のすべてのユーザーがホストユーザーにマッピングされない | userns\_mode: "keep-id" を使用、複数のユーザーがボリュームにアクセスするコンテナ | userns\_mode: "keep-id" は1対1のマッピングのみをサポートし、複数のIDを単一のIDに統合しない。 | バインドマウントディレクトリを使用する。 | userns\_mode: "keep-id" を使用し、複数のユーザーがボリュームにアクセスするコンテナを使用するユーザー | 回避策あり | セキュリティ | [https://github.com/containers/podman/issues/24925](https://github.com/containers/podman/issues/24925) |
| 9 | podman compose pull が Podman マシンをクラッシュさせる | podman compose pull の実行 | gvproxy のクラッシュ | podman \--log-level debug machine start で Podman マシンを起動する。 | podman compose pull を使用する macOS arm アーキテクチャのユーザー | 回避策あり | バグ | [https://github.com/containers/podman/issues/23114](https://github.com/containers/podman/issues/23114) |
| 10 | Fedora で podman、podman-docker、docker-compose を同時にインストールできない | Fedora で podman、podman-docker、docker-compose を同時にインストールしようとすると | docker-compose パッケージが docker-cli のインストールを明示的に要求するため。 | podman compose または docker compose ( podman-docker がインストールされている場合) コマンドを使用する。 | Fedora で podman、podman-docker、docker-compose を同時に使用したいユーザー | 回避策あり | 互換性 | [https://discussion.fedoraproject.org/t/conflicts-when-trying-to-install-docker-compose-having-podman-and-podman-docker-already-installed/132760?page=2](https://discussion.fedoraproject.org/t/conflicts-when-trying-to-install-docker-compose-having-podman-and-podman-docker-already-installed/132760?page=2) |
| 5 | Oracle Linux 8 で特権のないユーザーとして Quadlet を実行すると失敗する | Oracle Linux 8、ルート権限なし | 特権のないユーザーが cgroup ファイルシステムに書き込むためのアクセス権を持っていない。 | ルートユーザーで Quadlet を実行するか、Oracle Linux 9 にアップグレードする。 | Oracle Linux 8 でルートレス Podman を使用するユーザー | 回避策あり | セキュリティ | [https://docs.oracle.com/en/operating-systems/oracle-linux/podman/podman-KnownIssues.html](https://docs.oracle.com/en/operating-systems/oracle-linux/podman/podman-KnownIssues.html) |
| 5 | Oracle Linux 8 の特定の環境で、特権のないユーザーがコンテナストレージにアクセスできない | Oracle Linux 8、特権のないユーザー、特定の環境 | 不明 | /etc/containers/storage.conf の mountopt に index=off パラメータを追加する。 | Oracle Linux 8 でルートレス Podman を使用するユーザー | 回避策あり | セキュリティ | [https://docs.oracle.com/en/operating-systems/oracle-linux/podman/podman-KnownIssues.html](https://docs.oracle.com/en/operating-systems/oracle-linux/podman/podman-KnownIssues.html) |
| 5 | X509 証明書がレガシーな共通名フィールドに依存している | Podman バージョン 3.0 以降、SAN が設定されていない自己署名証明書を使用 | Podman のビルドに使用される Go コンパイラの新しいバージョン | 証明書を更新して適切な SAN を使用するか、GODEBUG 環境変数を x509ignoreCN=0 に設定する。 | 自己署名証明書を使用するローカルまたはプライベートなイメージレジストリを操作するユーザー | 回避策あり | セキュリティ | [https://docs.oracle.com/en/operating-systems/oracle-linux/podman/podman-KnownIssues.html](https://docs.oracle.com/en/operating-systems/oracle-linux/podman/podman-KnownIssues.html) |
| 5 | コンテナが利用できない場合、podman attach \--latest の実行でパニックが発生する | コンテナが存在しない環境で podman attach \--latest コマンドを実行 | 不明 | コンテナを作成してからコマンドを実行する。 | podman attach \--latest コマンドを使用するユーザー | 未解決 | バグ | [https://docs.oracle.com/en/operating-systems/oracle-linux/podman/podman-KnownIssues.html](https://docs.oracle.com/en/operating-systems/oracle-linux/podman/podman-KnownIssues.html) |
| 5 | コンテナをデタッチするためのデフォルトのキーシーケンスを使用するには、-t オプションが必要 | \-t オプションを指定せずにコンテナを作成 | 不明 | コンテナ作成時に \-t オプションを指定する。 | デフォルトのキーシーケンスでコンテナをデタッチするユーザー | 回避策あり | 操作性 | [https://docs.oracle.com/en/operating-systems/oracle-linux/podman/podman-KnownIssues.html](https://docs.oracle.com/en/operating-systems/oracle-linux/podman/podman-KnownIssues.html) |
| 5 | podman pull コマンドでイメージをプルしようとすると、正しい名前または完全な名前を指定しないと、認証エラーが発生する | イメージ名にレジストリ名を含めない | 不明 | イメージ名を完全な形式で指定する (例：docker.io/library/fedora)。 | podman pull コマンドを使用するユーザー | 回避策あり | 操作性 | [https://docs.oracle.com/en/operating-systems/oracle-linux/podman/podman-KnownIssues.html](https://docs.oracle.com/en/operating-systems/oracle-linux/podman/podman-KnownIssues.html) |
| 5 | Docker Hub から oraclelinux イメージを検索またはプルしようとすると、「manifest unknown」エラーで失敗する | Docker Hub から oraclelinux イメージを検索またはプルしようとすると | skopeo ユーティリティがデフォルトで latest タグが存在することを想定している。 | oraclelinux:latest またはタグなしのダウンストリームイメージを oraclelinux:7 に更新する。 | Docker Hub から oraclelinux イメージを検索またはプルするユーザー | 回避策あり | 互換性 | [https://docs.oracle.com/en/operating-systems/oracle-linux/podman/podman-KnownIssues.html](https://docs.oracle.com/en/operating-systems/oracle-linux/podman/podman-KnownIssues.html) |

### **セキュリティに関する推奨事項**

* ホストのカーネルとOSソフトウェアを定期的に更新する。 13  
* 必要最低限のOSを使用し、セキュリティのベストプラクティスに従う。 13  
* OSとカーネルの安全性を定期的に精査する。 13  
* 可能な限り最高のセキュリティ機能セットを提供する成熟したシステムコンポーネントを使用する。 13  
* Linuxセキュリティモジュールを使用する。 13  
* イメージが検証済みおよび信頼できるソースからのものであることを確認する。 13  
* 確実に再現可能なイメージを作成する。 13  
* イメージにインストールされているパッケージを最小限にする。 13  
* コンテナを非rootユーザーとして実行する。 13  
* メモリとCPUの使用量を制限してコンテナを実行することを検討する。 13  
* 適切なSELinuxラベルを設定する。 16  
* ユーザーネームスペースを修正する。 16  
* 必要最低限の権限でコンテナを実行する。 14  
* 定期的に更新を行う。 14  
* 安全な構成を行う。 14  
* 監査とログ記録を有効にする。 14  
* ネットワークポリシーを定義して適用する。 14  
* SELinuxまたはAppArmorを使用する。 15

### **パフォーマンスに関する推奨事項**

* コンテナイメージを最適化する。 14  
* Podmanのルートレスモードを活用する。 14  
* リソース使用量を監視および調整する。 14

#### **引用文献**

1\. Podman fails to run containers if image is not from docker.io after ..., 2月 23, 2025にアクセス、 [https://github.com/containers/podman/issues/22578](https://github.com/containers/podman/issues/22578)  
2\. Latest version of Podman available for RHEL 9 fails when trying to ..., 2月 23, 2025にアクセス、 [https://github.com/containers/podman/issues/24614](https://github.com/containers/podman/issues/24614)  
3\. Podman Upgrade caused issues with containers · Issue \#14936 ..., 2月 23, 2025にアクセス、 [https://github.com/containers/podman/issues/14936](https://github.com/containers/podman/issues/14936)  
4\. Problem with installed package podman \- General \- Docker ..., 2月 23, 2025にアクセス、 [https://forums.docker.com/t/problem-with-installed-package-podman/116529](https://forums.docker.com/t/problem-with-installed-package-podman/116529)  
5\. Known Issues \- Oracle Help Center, 2月 23, 2025にアクセス、 [https://docs.oracle.com/en/operating-systems/oracle-linux/podman/podman-KnownIssues.html](https://docs.oracle.com/en/operating-systems/oracle-linux/podman/podman-KnownIssues.html)  
6\. RHEL 8.7 podman-compose error iptables · Issue \#637 · containers ..., 2月 23, 2025にアクセス、 [https://github.com/containers/podman-compose/issues/637](https://github.com/containers/podman-compose/issues/637)  
7\. podman-compose failing to compose \- docker \- Stack Overflow, 2月 23, 2025にアクセス、 [https://stackoverflow.com/questions/61966654/podman-compose-failing-to-compose](https://stackoverflow.com/questions/61966654/podman-compose-failing-to-compose)  
8\. podman \--user not working in compose file · Issue \#24925 \- GitHub, 2月 23, 2025にアクセス、 [https://github.com/containers/podman/issues/24925](https://github.com/containers/podman/issues/24925)  
9\. podman compose not working correctly for some compose yamls · Issue \#23114 \- GitHub, 2月 23, 2025にアクセス、 [https://github.com/containers/podman/issues/23114](https://github.com/containers/podman/issues/23114)  
10\. Conflicts when trying to install \`docker-compose\` having \`podman\` and \`podman-docker\` already installed \- Page 2 \- Fedora Discussion, 2月 23, 2025にアクセス、 [https://discussion.fedoraproject.org/t/conflicts-when-trying-to-install-docker-compose-having-podman-and-podman-docker-already-installed/132760?page=2](https://discussion.fedoraproject.org/t/conflicts-when-trying-to-install-docker-compose-having-podman-and-podman-docker-already-installed/132760?page=2)  
11\. 2025 年に Podman を選ぶべき 5 つの理由 \- Red Hat, 2月 23, 2025にアクセス、 [https://www.redhat.com/ja/blog/5-reasons-choose-podman-2025](https://www.redhat.com/ja/blog/5-reasons-choose-podman-2025)  
12\. podman-desktop-redhat-account-ext/README.md at main \- GitHub, 2月 23, 2025にアクセス、 [https://github.com/redhat-developer/podman-desktop-redhat-account-ext/blob/main/README.md](https://github.com/redhat-developer/podman-desktop-redhat-account-ext/blob/main/README.md)  
13\. Security Recommendations \- Oracle Help Center, 2月 23, 2025にアクセス、 [https://docs.oracle.com/en/operating-systems/oracle-linux/podman/podman-SecurityRecommendations.html](https://docs.oracle.com/en/operating-systems/oracle-linux/podman/podman-SecurityRecommendations.html)  
14\. Unleashing Podman: The Ultimate Guide to Revolutionizing Your ..., 2月 23, 2025にアクセス、 [https://medium.com/@williamwarley/unleashing-podman-the-ultimate-guide-to-revolutionizing-your-container-management-3a5bdbbd5ef8](https://medium.com/@williamwarley/unleashing-podman-the-ultimate-guide-to-revolutionizing-your-container-management-3a5bdbbd5ef8)  
15\. Podman Deployment Hardening Patterns, 2月 23, 2025にアクセス、 [https://lists.podman.io/archives/list/podman@lists.podman.io/thread/NXEOIKXY3CUDT77I53NMJAI5LCMX4TUY/](https://lists.podman.io/archives/list/podman@lists.podman.io/thread/NXEOIKXY3CUDT77I53NMJAI5LCMX4TUY/)  
16\. podman/troubleshooting.md at main · containers/podman · GitHub, 2月 23, 2025にアクセス、 [https://github.com/containers/podman/blob/main/troubleshooting.md](https://github.com/containers/podman/blob/main/troubleshooting.md)  
17\. Podmanでの複数コンテナの連携方法 \#Podman \- Qiita, 2月 23, 2025にアクセス、 [https://qiita.com/t-kigi/items/a14dfd3c2c1620f54c43](https://qiita.com/t-kigi/items/a14dfd3c2c1620f54c43)  
18\. Podman Compose or Docker Compose: Which should ... \- Red Hat, 2月 23, 2025にアクセス、 [https://www.redhat.com/ja/blog/podman-compose-docker-compose](https://www.redhat.com/ja/blog/podman-compose-docker-compose)  
19\. Using Podman and Docker Compose \- Red Hat, 2月 23, 2025にアクセス、 [https://www.redhat.com/en/blog/podman-docker-compose](https://www.redhat.com/en/blog/podman-docker-compose)  
20\. How to Install and Use Podman on RHEL: A Comprehensive Guide ..., 2月 23, 2025にアクセス、 [https://medium.com/@jeromedecinco/how-to-install-and-use-podman-on-rhel-a-comprehensive-guide-191f4bc0d421](https://medium.com/@jeromedecinco/how-to-install-and-use-podman-on-rhel-a-comprehensive-guide-191f4bc0d421)  
21\. Chapter 5\. Using the docker command and service | Red Hat ..., 2月 23, 2025にアクセス、 [https://docs.redhat.com/en/documentation/red\_hat\_enterprise\_linux\_atomic\_host/7/html/getting\_started\_with\_containers/using\_the\_docker\_command\_and\_service](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux_atomic_host/7/html/getting_started_with_containers/using_the_docker_command_and_service)  
22\. Podman Troubleshooting Guide | Podman, 2月 23, 2025にアクセス、 [https://podman.io/blogs/2020/08/17/work-the-problems](https://podman.io/blogs/2020/08/17/work-the-problems)  
23\. Reasons to use Podman \- Reddit, 2月 23, 2025にアクセス、 [https://www.reddit.com/r/podman/comments/1gyt6yy/reasons\_to\_use\_podman/](https://www.reddit.com/r/podman/comments/1gyt6yy/reasons_to_use_podman/)
