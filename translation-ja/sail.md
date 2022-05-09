# Laravel Sail

- [イントロダクション](#introduction)
- [インストールと準備](#installation)
    - [既存アプリケーションへのSailインストール](#installing-sail-into-existing-applications)
    - [Bashエイリアスの設定](#configuring-a-bash-alias)
- [Sailの開始と停止](#starting-and-stopping-sail)
- [コマンドの実行](#executing-sail-commands)
    - [PHPコマンドの実行](#executing-php-commands)
    - [Composerコマンドの実行](#executing-composer-commands)
    - [Arttisanコマンドの実行](#executing-artisan-commands)
    - [Node/NPMコマンドの実行](#executing-node-npm-commands)
- [データベース操作](#interacting-with-sail-databases)
    - [MySQL](#mysql)
    - [Redis](#redis)
    - [MeiliSearch](#meilisearch)
- [ファイルストレージ](#file-storage)
- [テスト実行](#running-tests)
    - [Laravel Dusk](#laravel-dusk)
- [メールのプレビュー](#previewing-emails)
- [コンテナCLI](#sail-container-cli)
- [PHPバージョン](#sail-php-versions)
- [Nodeバージョン](#sail-node-versions)
- [サイトの共有](#sharing-your-site)
- [Xdebugによるデバッグ](#debugging-with-xdebug)
  - [Xdebug CLI使用法](#xdebug-cli-usage)
  - [Xdebug ブラウザ使用法](#xdebug-browser-usage)
- [カスタマイズ](#sail-customization)

<a name="introduction"></a>
## イントロダクション

[Laravel Sail](https://github.com/laravel/sail)（セイル、帆、帆船）は、LaravelのデフォルトのDocker開発環境を操作するための軽量コマンドラインインターフェイスです。 Sailは、Dockerの経験がなくても、PHP、MySQL、Redisを使用してLaravelアプリケーションを構築するための優れた出発点を提供します。

Sailの本質は、`docker-compose.yml`ファイルとプロジェクトのルートに保存されている`sail`スクリプトです。`sail`スクリプトは、`docker-compose.yml`ファイルで定義されたDockerコンテナを操作するための便利なメソッドをCLIで提供します。

Laravel Sailは、macOS、Linux、Windows（[WSL2](https://docs.microsoft.com/en-us/windows/wsl/about)を使用）に対応しています。

<a name="installation"></a>
## インストールと準備

Laravel Sailはいつも、新しいLaravelアプリケーションとともに自動的にインストールされるため、すぐ使用を開始できます。新しいLaravelアプリケーションを作成する方法については、使用しているオペレーティングシステム用のLaravel[インストールドキュメント](/docs/{{version}}/installation)を参照してください。インストール時には、アプリケーションで操作するSailサポートサービスを選択するよう求められます。

<a name="installing-sail-into-existing-applications"></a>
### 既存アプリケーションへのSailインストール

既存のLaravelアプリケーションでSailを使うことに興味を持っている場合は、Composerパッケージマネージャを使用してSailをインストールできます。もちろん、以降の手順は、既存のローカル開発環境で、Composerの依存関係をインストールできると仮定しています。

```shell
composer require laravel/sail --dev
```

Sailをインストールした後は、`sail:install` Artisanコマンドを実行してください。このコマンドは、Sailの`docker-compose.yml`ファイルをアプリケーションのルートへリソース公開します。

```shell
php artisan sail:install
```

最後に、Sailを立ち上げましょう。Sailの使い方を学び続けるには、このドキュメントの残りを続けてお読みください。

```shell
./vendor/bin/sail up
```

<a name="using-devcontainers"></a>
#### Devcontainerの使用

もし、[Devcontainer](https://code.visualstudio.com/docs/remote/containers)内で開発したい場合は、`--devcontainer`オプションを`sail:install`コマンドで指定してください。`--devcontainer`オプションは、`sail:install`コマンドに、デフォルトの`.devcontainer/devcontainer.json`ファイルをアプリケーションのルートにリソース公開するように指示します。

```shell
php artisan sail:install --devcontainer
```

<a name="configuring-a-bash-alias"></a>
### Bashエイリアスの設定

デフォルトでSailコマンドは、すべての新しいLaravelアプリケーションに含まれている`vendor/bin/sail`スクリプトを使用して起動します。

```shell
./vendor/bin/sail up
```

`vendor/bin/sail`を実行するため繰り返し入力する代わりに、Sailコマンドをより簡単に実行できるようにするBashエイリアスを設定することをおすすめします。

```shell
alias sail='[ -f sail ] && bash sail || bash vendor/bin/sail'
```

Bashエイリアスを設定したら、`sail`と入力するだけでSailコマンドを実行できます。このドキュメントの残りの部分の例は、このエイリアスが設定済みであることを前提としています。

```shell
sail up
```

<a name="starting-and-stopping-sail"></a>
## Sailの開始と停止

Laravel Sailの`docker-compose.yml`ファイルは、Laravelアプリケーションの構築を支援するために連携するさまざまなDockerコンテナを定義します。これらの各コンテナは、`docker-compose.yml`ファイルの`services`設定内のエントリです。`laravel.test`コンテナは、アプリケーションを提供するメインのアプリケーションコンテナです。

Sailを開始する前に、ローカルコンピューターで他のWebサーバまたはデータベースが実行されていないことを確認する必要があります。アプリケーションの`docker-compose.yml`ファイルで定義されているすべてのDockerコンテナを起動するには、`up`コマンドを実行する必要があります。

```shell
sail up
```

すべてのDockerコンテナをバックグラウンドで起動するには、Sailを「デタッチdetached)」モードで起動します。

```shell
sail up -d
```

アプリケーションのコンテナが開始されると、Webブラウザ（http:// localhost）でプロジェクトにアクセスできます。

すべてのコンテナを停止するには、Control＋Cキーを押してコンテナの実行を停止してください。もしくは、コンテナがバックグラウンドで実行されている場合は、`stop`コマンドを使用します。

```shell
sail stop
```

<a name="executing-sail-commands"></a>
## コマンドの実行

Laravel Sailを使用する場合、アプリケーションはDockerコンテナ内で実行され、ローカルコンピューターから分離されます。さらにSailは、任意のPHPコマンド、Artisanコマンド、Composerコマンド、Node / NPMコマンドなど、アプリケーションに対してさまざまなコマンドを実行するための便利な方法も提供します。

**Laravelのドキュメントを読むと、Sailを参照しないComposer、Artisan、Node／NPMコマンドの参照をよく目にするでしょう。**こうした実行例は、これらのツールがローカルコンピューターにインストールされていることを前提としています。ローカルのLaravel開発環境にSailを使用している場合は、Sailを使用してこれらのコマンドを実行する必要があります。

```shell
# Artisanコマンドをローカル環境で実行
php artisan queue:work

# Laravel Sailの中でArtisanコマンドを実行
sail artisan queue:work
```

<a name="executing-php-commands"></a>
### PHPコマンドの実行

PHPコマンドは、`php`コマンドを使用して実行します。もちろん、これらのコマンドは、アプリケーション用に構成されたPHPバージョンを使用して実行されます。Laravel Sailで利用可能なPHPバージョンの詳細については、[PHPバージョンのドキュメント](#sail-php-versions)を参照してください。

```shell
sail php --version

sail php script.php
```

<a name="executing-composer-commands"></a>
### Composerコマンドの実行

Composerコマンドは、`composer`コマンドを使用して実行します。 Laravel Sailのアプリケーションコンテナには、Composer2.xがインストール済みです。

```nothing
sail composer require laravel/sanctum
```

<a name="installing-composer-dependencies-for-existing-projects"></a>
#### 既存アプリケーションでComposer依存関係のインストール

チームでアプリケーションを開発している場合、最初にLaravelアプリケーションを作成するのは自分ではないかもしれません。そのため、アプリケーションのリポジトリをローカルコンピュータにクローンした後、Sailを含むアプリケーションのComposer依存関係は一切インストールされていません。

アプリケーションの依存関係をインストールするには、アプリケーションのディレクトリに移動し、次のコマンドを実行します。このコマンドは、PHPとComposerを含む小さなDockerコンテナを使用して、アプリケーションの依存関係をインストールします。

```shell
docker run --rm \
    -u "$(id -u):$(id -g)" \
    -v $(pwd):/var/www/html \
    -w /var/www/html \
    laravelsail/php81-composer:latest \
    composer install --ignore-platform-reqs
```

`laravelsail/phpXX-composer`イメージを使用する場合、アプリケーションで使用する予定のPHPと同じバージョン（`74`、`80`、`81`）を使用する必要があります。

<a name="executing-artisan-commands"></a>
### Artisanコマンドの実行

Laravel Artisanコマンドは、`artisan`コマンドを使用して実行します。

```shell
sail artisan queue:work
```

<a name="executing-node-npm-commands"></a>
### Node／NPMコマンドの実行

Nodeコマンドは`npm`コマンドを使用して実行し、NPMコマンドは`npm`コマンドを使用して実行します。

```shell
sail node --version

sail npm run prod
```

ご希望であれば、NPMの代わりにYarnを使用できます。

```shell
sail yarn
```

<a name="interacting-with-sail-databases"></a>
## データベース操作

<a name="mysql"></a>
### MySQL

お気づきかもしれませんが、アプリケーションの`docker-compose.yml`ファイルには、MySQLコンテナのエントリが含まれています。このコンテナは[Dockerボリューム](https://docs.docker.com/storage/volumes/)を使用しているため、コンテナを停止して再起動しても、データベースに保存されているデータは保持されます。

さらに、MySQLコンテナの初回起動時に、２つのデータベースが作成されます。最初のデータベースは、環境変数`DB_DATABASE`の値で命名され、ローカル開発用に使用するものです。もうひとつは、`testing`という名前のテスト専用データベースで、テストが開発データに干渉しないようにするためのものです。

コンテナを起動したら、アプリケーションの`.env`ファイル内の`DB_HOST`環境変数を`mysql`に設定することで、アプリケーション内のMySQLインスタンスに接続できます。

ローカルマシンからアプリケーションのMySQLデータベースに接続するには、[TablePlus](https://tableplus.com)などのグラフィカルデータベース管理アプリケーションを使用できます。デフォルトでMySQLデータベースは、`localhost`のポート3306からアクセスできます。

<a name="redis"></a>
### Redis

アプリケーションの`docker-compose.yml`ファイルには、[Redis](https://redis.io)コンテナのエントリも含まれています。このコンテナは[Dockerボリューム](https://docs.docker.com/storage/volumes/)を使用しているため、コンテナを停止して再起動しても、Redisデータに保存されているデータは保持されます。コンテナを起動したら、アプリケーションの`.env`ファイル内の`REDIS_HOST`環境変数を`redis`に設定することで、アプリケーション内のRedisインスタンスに接続できます。

ローカルマシンからアプリケーションのRedisデータベースに接続するには、[TablePlus](https://tableplus.com)などのグラフィカルデータベース管理アプリケーションを使用できます。デフォルトでは、Redisデータベースは`localhost`のポート6379でアクセスできます。

<a name="meilisearch"></a>
### MeiliSearch

Sailのインストール時に[MeiliSearch](https://www.meilisearch.com)サービスのインストールを選択した場合、アプリケーションの`docker-compose.yml`ファイルには、[Laravel Scout](/docs/{{version}}/scout)と[コンパチブル](https://github.com/meilisearch/meilisearch-laravel-scout)である、この強力な検索エンジンのエントリが含まれます。コンテナを起動したら、環境変数`MEILISEARCH_HOST`に`http://meilisearch:7700`を設定すると、アプリケーション内のMeiliSearchインスタンスに接続できます。

ローカルマシンから、Webブラウザの`http://localhost:7700`に移動して、MeiliSearchのWebベース管理パネルへアクセスできます。

<a name="file-storage"></a>
## ファイルストレージ

本番環境でアプリケーションを実行する際に、Amazon S3を使用してファイルを保存する予定であれば、Sailをインストールする際に[MinIO](https://min.io)サービスをインストールするとよいでしょう。MinIOはS3互換のAPIを提供しており、本番のS3環境で「テスト」ストレージバケットを作成せずに、Laravelの`s3`ファイルストレージドライバを使ってローカルに開発するために使用できます。Sailのインストール時にMinIOのインストールを選択すると、アプリケーションの`docker-compose.yml`ファイルにMinIOの設定セクションが追加されます。

アプリケーションのデフォルト`filesystems`設定ファイルには、`s3`ディスクのディスク設定がすでに含まれています。このディスクを使ってAmazon S3と連携するだけでなく、その構成を制御する関連環境変数を変更するだけで、MinIOなどのS3互換のファイルストレージサービスと連携することができます。例えば、MinIOを使用する場合、ファイルシステムの環境変数の設定は次のように定義します。

```ini
FILESYSTEM_DISK=s3
AWS_ACCESS_KEY_ID=sail
AWS_SECRET_ACCESS_KEY=password
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=local
AWS_ENDPOINT=http://minio:9000
AWS_USE_PATH_STYLE_ENDPOINT=true
```

<a name="running-tests"></a>
## テスト実行

Laravelは、はじめからすばらしいテストサポートを提供しています。Sailの`test`コマンドを使用してアプリケーションの[機能と単体テスト](/docs/{{version}}/testing)を実行できます。PHPUnitで指定できるCLIオプションはすべて、`test`コマンドに渡せます。

```shell
sail test

sail test --group orders
```

Sailの`test`コマンドは、`test` Artisanコマンドを実行するのと同じです。

```shell
sail artisan test
```

デフォルトでSailは、テストがデータベースの現在の状態に影響を与えないよう、専用の`testing`データベースを作成します。Laravelのデフォルトインストールは、Sailがテストを実行するときにこのデータベースを使用するように、`phpunit.xml`ファイルを構成します。

```xml
<env name="DB_DATABASE" value="testing"/>
```

<a name="laravel-dusk"></a>
### Laravel Dusk

[Laravel Dusk](/docs/{{version}}/dusk)は、表現力豊かで使いやすいブラウザ自動化およびテストのAPIを提供します。Sailのおかげで、Seleniumやその他のツールをローカルコンピューターにインストールしなくても、これらのテストを実行できます。使い始めるには、アプリケーションの`docker-compose.yml`ファイルでSeleniumサービスのコメントを解除します。

```yaml
selenium:
    image: 'selenium/standalone-chrome'
    volumes:
        - '/dev/shm:/dev/shm'
    networks:
        - sail
```

次に、アプリケーションの`docker-compose.yml`ファイルの`laravel.test`サービスに`selenium`の`depends_on`エントリがあることを確認します。

```yaml
depends_on:
    - mysql
    - redis
    - selenium
```

最後に、Sailを起動して`dusk`コマンドを実行することにより、Duskテストスイートを実行できます。

```shell
sail dusk
```

<a name="selenium-on-apple-silicon"></a>
#### Apple Silicon上のSelenium

ローカルマシンに Apple Silicon チップが搭載されている場合、`selenium`サービスには`seleniarm/standalone-chromium`イメージを使用する必要があります。

```yaml
selenium:
    image: 'seleniarm/standalone-chromium'
    volumes:
        - '/dev/shm:/dev/shm'
    networks:
        - sail
```

<a name="previewing-emails"></a>
## メールのプレビュー

LaravelSailのデフォルトの `docker-compose.yml`ファイルには、[MailHog](https://github.com/mailhog/MailHog)のサービスエントリが含まれています。MailHogは、ローカル開発中にアプリケーションから送信された電子メールをインターセプトし、ブラウザで電子メールメッセージをプレビューできる便利なWebインターフェイスを提供します。Sailを使う場合、MailHogのデフォルトホストは`mailhog`で、ポートは1025です。

```ini
MAIL_HOST=mailhog
MAIL_PORT=1025
MAIL_ENCRYPTION=null
```

Sailの実行中に、`http://localhost:8025`でMailHog Webインターフェイスにアクセスできます。

<a name="sail-container-cli"></a>
## コンテナCLI

アプリケーションのコンテナ内でBashセッションを開始したい場合があるでしょう。`shell`コマンドを使用してアプリケーションのコンテナに接続し、ファイルとインストールされているサービスを検査したり、コンテナ内で任意のシェルコマンドを実行したりできます。

```shell
sail shell

sail root-shell
```

新しい[LaravelTinker](https://github.com/laravel/tinker)セッションを開始するには、`tinker`コマンドを実行します。

```shell
sail tinker
```

<a name="sail-php-versions"></a>
## PHPバージョン

Sailは現在、PHP8.1、PHP8.0、PHP7.4を利用したアプリケーションの実行をサポートしています。SailのデフォルトPHPバージョンは8.1です。アプリケーションの実行に使用するPHPバージョンを変更するには、アプリケーションの`docker-compose.yml`ファイル内の`laravel.test`コンテナの`build`定義を更新してください。

```yaml
# PHP 8.1
context: ./vendor/laravel/sail/runtimes/8.1

# PHP8.0
context: ./vendor/laravel/sail/runtimes/8.0

# PHP7.4
context: ./vendor/laravel/sail/runtimes/7.4
```

さらに、アプリケーションで使用するPHPのバージョンを反映するために、`image`名を更新することもできます。このオプションも、アプリケーションの`docker-compose.yml`ファイルで定義されています。

```yaml
image: sail-8.1/app
```

アプリケーションの`docker-compose.yml`ファイルを更新した後、コンテナイメージを再構築する必要があります。

```shell
sail build --no-cache

sail up
```

<a name="sail-node-versions"></a>
## Nodeバージョン

SailはデフォルトでNode16をインストールします。イメージをビルドする際にインストールするNodeバージョンを変更するには、アプリケーションの`docker-compose.yml`ファイル中の、`laravel.test`サービスの`build.args`定義を変更してください。

```yaml
build:
    args:
        WWWGROUP: '${WWWGROUP}'
        NODE_VERSION: '14'
```

アプリケーションの`docker-compose.yml`ファイルを更新した後、コンテナイメージを再構築する必要があります。

```shell
sail build --no-cache

sail up
```

<a name="sharing-your-site"></a>
## サイトの共有

同僚のためにサイトをプレビューしたり、アプリケーションとのWebhook統合をテストしたりするために、サイトを公開して共有する必要がある場合があります。サイトを共有するには、 `share`コマンドを使用します。このコマンドを実行すると、アプリケーションへのアクセスに使用するランダムな`laravel-sail.site` URLが発行されます。

```shell
sail share
```

`share`コマンドでサイトを共有するときは、`TrustProxies`ミドルウェア内でアプリケーションの信頼できるプロキシを設定する必要があります。これを行わないと、`url`や`route`などのURL生成ヘルパは、URL生成時に使用するべき正しいHTTPホストを決定できません。

    /**
     * アプリケーションで信頼するプロキシ
     *
     * @var array|string|null
     */
    protected $proxies = '*';

共有サイトでサブドメインを選択する場合は、`share`コマンドを実行するときに`subdomain`オプションを指定します。

```shell
sail share --subdomain=my-sail-site
```

> {tip} `share`コマンドは、[BeyondCode](https://beyondco.de)によるオープンソースのトンネリングサービスである[Expose](https://github.com/beyondcode/expose)により提供しています。

<a name="debugging-with-xdebug"></a>
## Xdebugによるデバッグ

LaravelSailのDockerの設定には、PHP用の人気で強力なデバッガである[Xdebug](https://xdebug.org/)をサポートしています。XDebugを有効にするには、[Xdebugを設定](https://xdebug.org/docs/step_debug#mode)するために、アプリケーションの`.env`ファイルに変数を追加する必要があります。XDebugを有効にするには、Sailを開始する前に適切なモードを設定する必要があります。

```ini
SAIL_XDEBUG_MODE=develop,debug
```

#### LinuxホストIP設定

環境変数`XDEBUG_CONFIG`は、内部で`client_host=host.docker.internal`と定義されてため、XdebugはMacとWindows（WSL2）で適切に設定されます。ローカルマシンがLinuxの場合、Docker Engine17.06.0以上とCompose1.16.0以上が動作していることを確認してください。動作していない場合は、以下のようこの環境変数を手作業で定義する必要があります。

まず、以下のコマンドを実行して、環境変数に追加する正しいホストIPアドレスを決定します。通常、`<container-name>`は、アプリケーションを提供するコンテナの名前であるべきで、多くの場合、`_laravel.test_1`で終わります。

```shell
docker inspect -f {{range.NetworkSettings.Networks}}{{.Gateway}}{{end}} <container-name>
```

正しいホストIPアドレスを取得したら、アプリケーションの`.env`ファイル内で`SAIL_XDEBUG_CONFIG`変数を定義する必要があります。

```ini
SAIL_XDEBUG_CONFIG="client_host=<host-ip-address>"
```

<a name="xdebug-cli-usage"></a>
### Xdebug CLI使用法

Artisanのコマンドを実行する際に、`sail debug`コマンドを使ってデバッグセッションを開始することができます。

```shell
# Xdebugなしでartisanコマンドを実行
sail artisan migrate

# Xdebugでartisanコマンドを実行
sail debug migrate
```

<a name="xdebug-browser-usage"></a>
### Xdebug ブラウザ使用法

Webブラウザでアプリケーションを操作しながらデバッグするには、WebブラウザからXdebugセッションを開始するための[Xdebugが提供する手順](https://xdebug.org/docs/step_debug#web-application)に従ってください。

PhpStormを使用している場合は、[設定なしのデバッグ](https://www.jetbrains.com/help/phpstorm/zero-configuration-debugging.html)に関するJetBrainのドキュメントを確認してください。

> {note} Laravel Sailはアプリケーション提供を`artisan serve`に依存しています。`artisan serve`コマンドは、Laravelバージョン8.53.0以降では、`XDEBUG_CONFIG`と`XDEBUG_MODE`変数のみを受け付けます。古いバージョンのLaravel（8.52.0以下）では、これらの変数をサポートしていないため、デバッグ接続を受け付けません。

<a name="sail-customization"></a>
## Sailカスタマイズ

Sailは単なるDockerであるため、Sailに関するほぼすべてを自由にカスタマイズできます。Sail独自のDockerfileをリソース公開するには、`sail:publish`コマンドを実行します。

```shell
sail artisan sail:publish
```

このコマンドを実行すると、Laravel Sailが使用するDockerfileとその他の設定ファイルが、アプリケーションのルートディレクトリ内の`docker`ディレクトリに配置されます。Sailのインストールをカスタマイズした後、アプリケーションの `docker-compose.yml` ファイル内のアプリケーションコンテナのイメージ名を変更したいと思うことでしょう。それを行ったら、`build`コマンドを使用してアプリケーションのコンテナを再構築してください。アプリケーション・イメージに一意な名前を割り当てることは、Sailを使用して1台のマシンで複数のLaravelアプリケーションを開発する場合に特に重要です。

```shell
sail build --no-cache
```
