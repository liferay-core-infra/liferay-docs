---
header-id: finding-extension-points
---

# 拡張ポイントを見つける

[TOC levels=1-4]

@product@では、ユーザーが作業を実行するのに役立つ機能を多数提供しています。
ただし、[組み込み機能をカスタマイズ](/docs/7-1/tutorials/-/knowledge_base/t/customizing)する必要がある場合があります。
カスタマイズしたいエリアを**見つける**のは簡単ですが、それをカスタマイズする**方法**を見つけるのは困難な作業のように思えるかもしれません。しかし、@product@は簡単にカスタマイズできるように開発されており、ご希望に合わせて使用できる拡張ポイントが多数あります。

拡張ポイントを見つけるプロセスは簡単です。

1. まず、変更する機能を提供しているバンドル（モジュール）を探して、
3. モジュールで利用可能なコンポーネントを見つけます。
4. そして、カスタマイズするコンポーネントの拡張ポイントを探します。

このチュートリアルでは、拡張ポイントを見つける方法を、LDAPユーザーをインポートするための拡張ポイントを探すという簡単な例を用いて、段階的に説明します。この例では、@product@の[アプリケーションマネージャ](/docs/7-1/user/-/knowledge_base/u/managing-and-configuring-apps#using-the-app-manager)と[Felix Gogoシェル](/docs/7-1/reference/-/knowledge_base/r/using-the-felix-gogo-shell)を使用します。

## 関連モジュールとコンポーネントを見つる

まずは、変更するアプリケーションの動作を説明する言葉について考えます。
適切なキーワードを使用することで、目的のモジュールとそのコンポーネントを簡単に探し出すことができるからです。ここでは、LDAPユーザーをインポートする例で考えてみます。コンポーネントを見つけるためのキーワード候補には、*import*、*user*、および*LDAP*が挙げられます。

特定のLiferayの機能を担当するモジュールを探すには、アプリケーションマネージャの使用が最も簡単です。アプリケーションマネージャは、アプリ・スイートとそれに含まれるモジュール/コンポーネントを使いやすいインターフェイスに一覧表示します。それだけではなく、サードパーティのアプリも一覧表示します。キーワードを使用して、該当するコンポーネントをターゲットにします。

1. *[パネルコントロール]* → *[アプリ]* → *[アプリケーションマネージャ]*に移動して、アプリケーションマネージャを開きます。最上位には、アプリ・スイート、独立したアプリ、独立したモジュールが一覧表示されています。

2. アプリ・スイート、アプリ、およびモジュールをナビゲートするか、検索機能を使用して、目的の拡張ポイントを提供する可能性のあるコンポーネントを見つけます。要素名と説明のキーワードも必ず確認してください。キーワードの*LDAP*がLiferay Foundationのアプリ・スイートのアプリと機能のリストにあります。アプリ・スイートを選択します。

   ![図1：Liferay Foundationのアプリスイートには、LDAP認証アプリケーションが含まれています。](../../images/ldap-keyword-app-manager.png)

3. アプリケーションのリストから、*LDAP*アプリケーションを選択します。

4. LDAPアプリケーションにはモジュールが1つしかありませんが、通常のアプリケーションには調べるべきモジュールが複数あります。*[Liferay Portal Security LDAP]*モジュールを選択します。

   ![図2：アプリケーションマネージャには、モジュール、パッケージ名、バージョン、およびステータスが一覧表示されます。](../../images/app-manager-breakdown.png)

5. キーワードをガイドとして適用し、コンポーネントを検索します。カスタマイズする機能に最適と思われるコンポーネント名をコピーしておきます。Gogoシェルを使用して、そのコンポーネントについて調べるためです。

   ![図3：コンポーネント名は、アプリケーションマネージャを使用して見つけることができます。](../../images/usermodellistener-component.png)

   | **注：**Gogoシェルを使用する場合に、探しているコンポーネントを見つけるまでに| 数回試行する可能性がある点に留意してください。Liferayの命名| 規則により、管理可能な時間枠で拡張ポイントを見つけることが容易になります。

次は、Gogoシェルを使用して、コンポーネントの拡張ポイントを調べます。

## コンポーネント内の拡張ポイントを探し出す

拡張したい機能に関連するコンポーネントを入手したら、GogoシェルのService Component Runtime（SCR）コマンドを使用して検査できます。SCRコマンドは、[Liferay Blade CLI](/docs/7-1/tutorials/-/knowledge_base/t/blade-cli)または[Gogoシェル](/docs/7-1/reference/-/knowledge_base/r/using-the-felix-gogo-shell)で実行できます。
このチュートリアルでは、Gogoシェルを使用していることを前提としています。

以下のコマンドを実行します。

    scr:info [COMPONENT_NAME]

前にコピーしたLDAPのサンプルコンポーネントの場合、コマンドは以下のようになります。

    scr:info com.liferay.portal.security.ldap.internal.messaging.UserImportMessageListener

出力には、多くの情報が含まれています。今回の場合、コンポーネントが参照するサービスを対象としています。これらは拡張ポイントです。たとえば、以下はLDAPユーザーをインポートするサービスのリファレンスです。

    ...
    Reference: LdapUserImporter
    Interface Name: com.liferay.portal.security.ldap.exportimport.LDAPUserImporter
    Cardinality: 1..1
    Policy: static
    Policy option: reluctant
    Reference Scope: bundle
    ...

`LDAPUserImporter`は、LDAPユーザーのインポートプロセスをカスタマイズするための拡張ポイントです。どのリファレンスも探しているものに当てはまらない場合は、アプリケーションマネージャから他のコンポーネントを検索します。

参照されるサービスをオーバーライドする予定がある場合は、リファレンスのポリシーとポリシーオプションを理解する必要があります。ポリシーが`static`で、ポリシーオプションが`reluctant`の場合、バインドされたサービスの代わりに新しい上位のサービスをバインドするには、コンポーネントを再アクティブ化するか、ターゲットを変更する必要があります。その他のポリシーおよびポリシーオプションの詳細については、[OSGi specification](https://osgi.org/download/r6/osgi.enterprise-6.0.0.pdf)（特にセクション112.3.5および112.3.6）を参照してください。コンポーネントのサービス参照をオーバーライドする方法については、[こちら](/docs/7-1/tutorials/-/knowledge_base/t/overriding-service-references)のチュートリアルを参照してください。

**重要** すべてのLiferay拡張ポイントが参照されるサービスとして利用できるわけではありません。サービス参照は、宣言型サービス（DS）コンポーネントで一般的ですが、拡張ポイントは他の方法でも公開できます。以下は、@product@のその他の潜在的な拡張ポイントの簡単なリストです。

- `org.osgi.util.tracker.ServiceTracker<S, T>`のインスタンス
- Liferayの`Registry.getServiceTracker`の使用
- Liferayの`ServiceTrackerMap`または`ServiceTrackerCollection`の使用
- ブループリント、Apache Dependency Managerなどの追跡サービスをサポートする他のコンポーネントフレームワークまたはホワイトボードの実装（例: HTTP、JAX-RS）も拡張ポイントを導入する可能性があります。

説明は以上です。これで、アプリケーションマネージャでキーワードを使用して、動作を変更するためのモジュールコンポーネントを探し出し、Gogoシェルを使用して、カスタマイズを実装するためのコンポーネント拡張ポイントが見つけられるようになります。