# **RHELにおけるPodman/DockerおよびPodman-Composeの商用環境利用における課題と対応策**

## **はじめに**

Red Hat Enterprise Linux (RHEL) は、エンタープライズレベルのアプリケーションを構築、デプロイ、管理するための堅牢なプラットフォームを提供しています。近年、コンテナ化技術は、その柔軟性と効率性から、RHEL環境においても急速に普及しています。かつてはDockerがコンテナ化のデファクトスタンダードとして広く利用されていましたが、RHEL 8以降ではDockerのサポートが終了し、Red HatはPodmanを推奨しています。1 PodmanはDockerと高い互換性を持ちながら、デーモンレスアーキテクチャやルートレスコンテナといったセキュリティ面で優れた特徴を備えています。2 さらに、podman-docker パッケージをインストールすることで、DockerコマンドをPodmanで実行することができ、Dockerからの移行をスムーズに行うことができます。3

本稿では、RHELにおけるPodman/DockerおよびPodman-Composeを商用環境でコンテナ単体で利用する際の課題と対応策を包括的に解説します。コンテナオーケストレーションツール(Kubernetes, OpenShiftなど)は使用しないことを前提とし、2020年から2025年に確認できたRed Hat社の公式ドキュメント、インターネット検索、コミュニティフォーラム、技術ブログ、事例研究などから得られた知見を網羅的にまとめました。

## **Podmanの機能的な課題**

Podmanは、多くの点でDockerと互換性がありますが、いくつかの機能的な違いや課題が存在します。

### **ルートレスコンテナにおける権限問題**

Podmanのルートレスモードは、コンテナをroot権限なしで実行できるため、セキュリティの向上に大きく貢献します。しかし、コンテナの実行ユーザーに適切な権限が設定されていない場合、ファイルアクセスやネットワーク接続などで問題が発生する可能性があります。4 例えば、ホストのディレクトリをコンテナにマウントする際、コンテナ内のユーザーにそのディレクトリへのアクセス権限がない場合、ファイルの読み書きができません。また、ユーザー名前空間の設定が正しくない場合、コンテナ内でプロセスが正常に動作しないことがあります。

**対応策**

* SELinuxやAppArmorなどのセキュリティモジュールが適切に設定されていることを確認する。これらのモジュールは、コンテナのアクセス制御を強化し、セキュリティリスクを軽減します。  
* /etc/subuid や /etc/subgid を編集し、ユーザー名前空間を適切に設定する。これにより、コンテナ内のユーザーとホスト側のユーザーをマッピングし、権限の問題を回避できます。  
* ボリュームマウント時に、ホスト側のディレクトリに対する適切なアクセス権限をコンテナの実行ユーザーに設定する。

### **Dockerとの非互換機能**

PodmanはDockerとの互換性を重視して開発されていますが、完全に互換性があるわけではありません。1 いくつかのDockerコマンドはPodmanではサポートされていません。また、コマンドのオプションや動作が異なる場合もあります。

**対応策**

* DockerfileをPodmanで使用する場合は、非互換な部分がないか事前に確認し、必要があれば修正する。例えば、レジストリの短縮名を使用している場合は、Podmanの設定で適切に設定する必要があります。  
* Docker Composeを使用している場合は、Podman 3.0以降でDocker Composeがサポートされていることを確認し、Podman ComposeまたはDocker Composeのいずれかを使用する。5  
* Docker Swarmを使用している場合は、Podmanではサポートされていないため、Kubernetesなどの代替ツールを検討する。

### **Kubernetes Podのサポート**

Podmanは、KubernetesのPodと同様の機能を提供します。6 Podは、複数のコンテナをまとめて管理するための単位であり、コンテナ間でリソースを共有したり、ネットワークを接続したりすることができます。

**対応策**

* 将来的にKubernetesの利用を検討している場合は、PodmanのPod機能を活用することで、Kubernetesへの移行をスムーズに行うことができます。

## **コンテナ運用における課題**

コンテナを商用環境で運用する際には、イメージ管理、セキュリティ、パフォーマンス、監視、ログ管理など、さまざまな課題に対処する必要があります。

### **コンテナイメージの管理**

コンテナイメージのサイズは、ビルド時間、配布時間、ストレージ容量、セキュリティリスクなどに影響を与えるため、適切に管理することが重要です。7

**対応策**

* マルチステージビルドを活用する。8 マルチステージビルドでは、複数のステージでイメージをビルドすることで、最終的なイメージサイズを小さくすることができます。  
* 不要なパッケージをインストールしない。8 イメージに含まれるパッケージは最小限にすることで、イメージサイズを削減し、セキュリティリスクを低減することができます。  
* イメージのレイヤー数を最小限に抑え、レイヤーの順番を最適化する。8 レイヤー数が少ないほど、イメージのビルドと配布が高速になります。また、レイヤーの順番を最適化することで、イメージのサイズを削減することができます。  
* 信頼できるベースイメージを使用する。8 信頼できるベースイメージを使用することで、セキュリティリスクを低減することができます。  
* docker scan コマンド9 やサードパーティのセキュリティスキャンツールを使用して、イメージの脆弱性をスキャンする。これにより、イメージに含まれる脆弱性を早期に発見し、修正することができます。  
* Red Hatは、AMD64、Intel 64、PowerPC、IBM Z、ARM 64-bitなど、さまざまなアーキテクチャ向けにコンテナイメージとコンテナ関連ソフトウェアを提供しています。1 アプリケーションのプラットフォーム互換性を確保するために、適切なアーキテクチャ向けのイメージを使用することが重要です。

| Architecture | Base Image | Layered Images |
| :---- | :---- | :---- |
| AMD64 and Intel 64 | ○ | ○ |
| PowerPC 8 and 9 64-bit | ○ | ほとんど |
| 64-bit IBM Z | ○ | ほとんど |
| ARM 64-bit | ○ | \- |

### **セキュリティ**

コンテナはホストOSのカーネルを共有するため、コンテナのセキュリティ対策は、ホストOS全体のセキュリティに影響を与えます。10

**対応策**

* コンテナの実行ユーザーに最小限の権限を与える。7 これにより、コンテナが侵害された場合でも、ホストOSへの影響を最小限に抑えることができます。  
* SELinuxやAppArmorなどのセキュリティモジュールを有効にする。これらのモジュールは、コンテナのアクセス制御を強化し、セキュリティリスクを軽減します。  
* seccomp profilesを使用して、コンテナが実行できるシステムコールを制限する。これにより、コンテナの攻撃対象領域を縮小することができます。  
* 定期的にセキュリティアップデートを適用する。これにより、既知の脆弱性を修正し、セキュリティリスクを低減することができます。

### **パフォーマンス**

コンテナのパフォーマンスは、ホストOSのリソースやコンテナの設定に影響を受けます。11 特に、I/Oパフォーマンスはコンテナのパフォーマンスに大きく影響するため、注意が必要です。

**対応策**

* ホストOSのリソースを適切に割り当てる。CPU、メモリ、ストレージなどのリソースを、コンテナの要件に合わせて適切に割り当てることで、パフォーマンスを向上させることができます。  
* ストレージとして、パフォーマンスの高いSSDなどを使用する。SSDは、HDDに比べて高速な読み書きが可能であるため、コンテナのパフォーマンスを向上させることができます。  
* コンテナのCPUやメモリ使用量を制限する。これにより、コンテナが過剰なリソースを使用することを防ぎ、他のコンテナやホストOSのパフォーマンスへの影響を最小限に抑えることができます。  
* アプリケーションを最適化し、リソース使用量を削減する。アプリケーションのコードや設定を最適化することで、コンテナのリソース使用量を削減し、パフォーマンスを向上させることができます。

### **Container Management with Systemd**

Podmanは、systemdと統合してコンテナをサービスとして管理することができます。12 これにより、コンテナの起動、停止、再起動などをsystemdの機能を使用して行うことができます。

**対応策**

* systemdのユニットファイルを作成し、コンテナをサービスとして登録することで、コンテナの管理を自動化することができます。

### **監視とログ管理**

コンテナの監視とログ管理は、問題発生時の原因究明やパフォーマンス分析に不可欠です。13

**対応策**

* podman stats コマンドやサードパーティツールを使用して、コンテナのリソース使用状況を監視する。これにより、CPU使用率、メモリ使用量、ネットワークトラフィックなどをリアルタイムに監視することができます。  
* podman logs コマンドやFluentdなどのログ収集ツールを使用して、コンテナのログを収集する。コンテナのログを収集することで、問題発生時の原因究明やパフォーマンス分析に役立てることができます。  
* ログを分析し、問題発生の兆候を早期に検知する。ログ分析ツールを使用して、ログに含まれるエラーや警告などを分析することで、問題発生の兆候を早期に検知することができます。

## **Dockerの課題**

RHEL 8以降ではDockerはサポートされていませんが、RHEL 7以前のバージョンでは引き続き使用することができます。しかし、Dockerを使用する場合には、以下の課題を考慮する必要があります。

### **サポート終了**

RHEL 8以降ではDockerのサポートが終了しており、セキュリティアップデートなどが提供されなくなっています。14 これにより、セキュリティリスクが高まる可能性があります。

**対応策**

* 将来的にRHEL 8以降にアップグレードする予定がある場合は、Podmanに移行することを検討する。  
* RHEL 7以前のバージョンでDockerを使用する必要がある場合は、Mirantis Container Runtime (MCR) などの商用サポート付きDockerを使用する。14 MCRは、Docker Enterprise Editionをベースとしたコンテナランタイムであり、Red Hat Enterprise Linuxで商用サポートを提供しています。

### **ライセンス問題**

Docker Desktopは、商用利用に制限があり、従業員数や年間収益によっては有料のサブスクリプションが必要となります。15

**対応策**

* Docker Desktopのライセンス要件を確認し、必要に応じて有料サブスクリプションを購入する。  
* Podman Desktopなどの代替ツールを使用する。Podman Desktopは、Docker Desktopと同様の機能を提供するGUIツールであり、無料で商用利用することができます。

## **Podman-Composeの課題**

Podman-Composeは、Docker Composeの代替ツールとして開発されていますが、いくつかの課題が存在します。

### **機能制限**

Podman-ComposeはDocker Composeのすべての機能をサポートしているわけではありません。16 特に、初期のバージョンでは、Docker ComposeのYAMLファイルとの互換性に課題がありました。

**対応策**

* Podman-Composeの最新バージョンを使用する。Podman-Composeは活発に開発が進められており、最新バージョンではDocker Composeとの互換性が向上しています。  
* Podman-Composeでサポートされていない機能がある場合は、代替手段を検討する。  
* 必要に応じて、Docker Composeを使用する。

### **運用上の課題**

Podman-Composeは、Docker Composeと比較して、まだ新しいツールであるため、運用に関する情報やノウハウが少ないという課題があります。

**対応策**

* Podman-Composeの公式ドキュメントやコミュニティフォーラムなどを参照する。  
* Podman-Composeの開発に貢献し、ツールの改善に協力する。

## **コミュニティフォーラムにおける知見**

Red HatのコミュニティフォーラムやPodman/Dockerに関するフォーラムでは、商用環境でのPodman/Dockerの利用に関する議論や問題報告が活発に行われています。15 これらのフォーラムでは、以下のような知見が得られます。

* Podmanは、Docker Desktopのライセンス問題を回避できるため、商用環境で利用する際の選択肢として注目されている。  
* Podman Desktopは、Docker Desktopの代替として、Windows環境でのコンテナ開発に役立つツールである。  
* Podmanは、Dockerと高い互換性を持つため、DockerからPodmanへの移行は比較的容易である。  
* Podmanは、デーモンレスアーキテクチャを採用しているため、セキュリティ面で優れている。  
* Podmanは、Kubernetes Podをサポートしているため、Kubernetesへの移行を容易にする。

## **事例研究からの知見**

RHELのPodman/Dockerを商用環境で利用している企業や組織の事例を調査することで、以下の知見が得られました。

* Docker Desktopのライセンス変更により、Podmanを採用する企業が増えている。18  
* 商用LinuxでDockerを運用する場合、ベンダーのサポートを受けることができる。19  
* Dockerfileのベストプラクティスに従うことで、コンテナのセキュリティとパフォーマンスを向上させることができる。20

## **結論**

RHELにおいてPodman/DockerおよびPodman-Composeを商用環境でコンテナ単体で利用する際には、いくつかの課題が存在します。しかし、これらの課題は適切な対応策を講じることで克服することができます。

Podmanは、Dockerと高い互換性を持ちながら、デーモンレスアーキテクチャやルートレスコンテナといったセキュリティ面で優れた特徴を備えています。また、Podman-ComposeはDocker Composeの代替ツールとして、複数コンテナの管理を容易にします。

コンテナイメージの管理、セキュリティ、パフォーマンス、監視、ログ管理など、運用上の課題に対しても、適切なツールやベストプラクティスを活用することで、安定した運用を実現することができます。

RHELにおけるコンテナ技術は、Podmanを中心として進化を続けています。Red Hatは、Podmanの開発に積極的に投資しており、今後も機能強化やセキュリティ改善などが期待されます。商用環境でコンテナ技術を利用する際には、Podmanを第一の選択肢として検討することを推奨します。

## **付録**

### **Red Hat Enterprise Linuxにおけるコンテナ関連技術のサポートポリシー**

Red Hatは、RHELで利用可能なコンテナ関連技術のサポートポリシーを公開しています。1 このポリシーは、コンテナオーケストレーション、コンテナホスト、コンテナエンジン、コンテナランタイム、コンテナイメージなど、さまざまなコンポーネントを網羅しており、商用環境でコンテナ技術を利用する際の重要な情報源となります。

| Technology | Description | Support Status |
| :---- | :---- | :---- |
| Red Hat OpenShift | Kubernetes および Linux の完全なエンタープライズディストリビューション | サポート対象 |
| Red Hat Enterprise Linux (RHEL) | コンテナエンジン、コンテナーランタイム、コンテナーイメージ、そして一緒にビルドおよびテストされた Linux カーネルを含む、完全にカスタマイズ可能なコンテナーホスト | サポート対象 |
| Red Hat Enterprise Linux CoreOS | Kubernetes クラスター内の完全に管理されたコンポーネント | サポート対象 |
| CRI-O | コンテナエンジン | サポート対象 |
| Docker | コンテナエンジン | RHEL 8以降ではサポート対象外 |
| Podman | コンテナエンジン | サポート対象 |
| Buildah | コンテナイメージビルドツール | サポート対象 |
| Skopeo | コンテナイメージ操作ツール | サポート対象 |
| runc | コンテナランタイム | サポート対象 |
| crun | コンテナランタイム | サポート対象 |
| Red Hat Enterprise Linux ベースイメージ |  | サポート対象 |
| Red Hat Universal Base Image (UBI) |  | サポート対象 |
| ISV パートナー製品イメージ |  | Red Hat 認定の場合、サポート対象 |

### **Podman Desktop**

Podman Desktopは、PodmanのGUIツールであり、コンテナの管理や操作を容易にします。22 Windows、macOS、Linuxに対応しており、Docker Desktopの代替として利用できます。

### **Red Hat Universal Base Images (UBI)**

UBIは、Red Hatが提供するコンテナベースイメージであり、Red Hat Enterprise Linuxのサブスクリプションがなくても利用できます。23 商用環境でRed Hatのサポートを受けながらコンテナを利用したい場合に適しています。

#### **引用文献**

1\. Building, running, and managing containers | Red Hat Product Documentation, 2月 23, 2025にアクセス、 [https://docs.redhat.com/en/documentation/red\_hat\_enterprise\_linux/10-beta/html-single/building\_running\_and\_managing\_containers/index](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10-beta/html-single/building_running_and_managing_containers/index)  
2\. Podman とは？をわかりやすく解説 \- Red Hat, 2月 23, 2025にアクセス、 [https://www.redhat.com/ja/topics/containers/what-is-podman](https://www.redhat.com/ja/topics/containers/what-is-podman)  
3\. コンテナーの構築、実行、および管理 | Red Hat Product Documentation, 2月 23, 2025にアクセス、 [https://docs.redhat.com/ja/documentation/red\_hat\_enterprise\_linux/8/html-single/building\_running\_and\_managing\_containers/index](https://docs.redhat.com/ja/documentation/red_hat_enterprise_linux/8/html-single/building_running_and_managing_containers/index)  
4\. ルートレスなPodmanでLaravel環境を構築するハンズオン \- home | tamakoma.com, 2月 23, 2025にアクセス、 [https://tamakoma.com/blog/hands-on-laravel-podman/](https://tamakoma.com/blog/hands-on-laravel-podman/)  
5\. Using Podman and Docker Compose \- Red Hat, 2月 23, 2025にアクセス、 [https://www.redhat.com/en/blog/podman-docker-compose](https://www.redhat.com/en/blog/podman-docker-compose)  
6\. コンテナ仮想化「Podman」の使用方法 \#Linux \- Qiita, 2月 23, 2025にアクセス、 [https://qiita.com/Nao\_Ishimatsu/items/f1b20b34b09e3611696e](https://qiita.com/Nao_Ishimatsu/items/f1b20b34b09e3611696e)  
7\. Docker のセキュリティに関するベストプラクティス 10 項目 \- Snyk, 2月 23, 2025にアクセス、 [https://snyk.io/jp/blog/10-docker-image-security-best-practices/](https://snyk.io/jp/blog/10-docker-image-security-best-practices/)  
8\. 社内のDockerfileのベストプラクティスを公開します \- Zenn, 2月 23, 2025にアクセス、 [https://zenn.dev/forcia\_tech/articles/20210716\_docker\_best\_practice](https://zenn.dev/forcia_tech/articles/20210716_docker_best_practice)  
9\. イメージビルドのベストプラクティス | Docker ドキュメント, 2月 23, 2025にアクセス、 [https://matsuand.github.io/docs.docker.jp.onthefly/get-started/09\_image\_best/](https://matsuand.github.io/docs.docker.jp.onthefly/get-started/09_image_best/)  
10\. Dockerとは？使い方やメリット・デメリットを徹底解説！ | GeeklyMedia(ギークリーメディア) | Geekly（ギークリー） IT・Web・ゲーム業界専門の人材紹介会社, 2月 23, 2025にアクセス、 [https://www.geekly.co.jp/column/cat-technology/1902\_047/](https://www.geekly.co.jp/column/cat-technology/1902_047/)  
11\. Dockerについて徹底解説｜メリット、デメリット、将来性まとめ \- 金融情報システム開発なら20年以上の実績があるテンファイブ株式会社, 2月 23, 2025にアクセス、 [https://10-5.jp/blog-tenfive/1940/](https://10-5.jp/blog-tenfive/1940/)  
12\. How to Install and Use Podman on RHEL: A Comprehensive Guide | by Jerome Decinco, 2月 23, 2025にアクセス、 [https://medium.com/@jeromedecinco/how-to-install-and-use-podman-on-rhel-a-comprehensive-guide-191f4bc0d421](https://medium.com/@jeromedecinco/how-to-install-and-use-podman-on-rhel-a-comprehensive-guide-191f4bc0d421)  
13\. Docker のベストプラクティス, 2月 23, 2025にアクセス、 [https://www.docker.com/ja-jp/resources/docker-best-practices-on-demand-training/](https://www.docker.com/ja-jp/resources/docker-best-practices-on-demand-training/)  
14\. Red Hat Enterprise Linux 8以降でDocker Engineを利用するには \- クリエーションライン株式会社, 2月 23, 2025にアクセス、 [https://www.creationline.com/tech-blog/blogchallenge/64379](https://www.creationline.com/tech-blog/blogchallenge/64379)  
15\. Podman環境を作ろう \#container \- Qiita, 2月 23, 2025にアクセス、 [https://qiita.com/shupeluter/items/2ff6f79bcf84c30f7265](https://qiita.com/shupeluter/items/2ff6f79bcf84c30f7265)  
16\. podman-compose, 2月 23, 2025にアクセス、 [https://docs.podman.io/en/v5.1.1/markdown/podman-compose.1.html](https://docs.podman.io/en/v5.1.1/markdown/podman-compose.1.html)  
17\. Podman Desktopを使って、商用無償なコンテナ環境を作ろう \#Docker \- Qiita, 2月 23, 2025にアクセス、 [https://qiita.com/kitfactory/items/57e1c6bb9b0229bba3df](https://qiita.com/kitfactory/items/57e1c6bb9b0229bba3df)  
18\. バージョン1.0が公開されたPodman DesktopをWindows/Macで試してみた｜QESブログ, 2月 23, 2025にアクセス、 [https://www.qes.co.jp/media/Microservices/a294](https://www.qes.co.jp/media/Microservices/a294)  
19\. Dockerの導入前に知っておくべきこと | Think IT（シンクイット）, 2月 23, 2025にアクセス、 [https://thinkit.co.jp/story/2015/08/18/6326](https://thinkit.co.jp/story/2015/08/18/6326)  
20\. 開発環境をDockerに乗せる方法とメリットを3ステップで学ぶチュートリアル \- Qiita, 2月 23, 2025にアクセス、 [https://qiita.com/KeitaMoromizato/items/ae1a57fc62b41b942d71](https://qiita.com/KeitaMoromizato/items/ae1a57fc62b41b942d71)  
21\. Red Hat コンテナーのサポートポリシー, 2月 23, 2025にアクセス、 [https://access.redhat.com/articles/3054631](https://access.redhat.com/articles/3054631)  
22\. Podman, 2月 23, 2025にアクセス、 [https://podman.io/](https://podman.io/)  
23\. Lab 3.0 \- Intro to Podman and base images \- Red Hat | Public Sector, 2月 23, 2025にアクセス、 [http://redhatgov.io/workshops/security\_container\_intro/lab03-podman/](http://redhatgov.io/workshops/security_container_intro/lab03-podman/)
