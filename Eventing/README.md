# Knative Eventingを試してみる

Eventingのサンプルいっぺある。
https://knative.dev/docs/eventing/samples/


[ここ](https://knative.dev/docs/eventing/)を参照しながらまずは理解を。

## Design overview
- Knative Eventing services are loosely coupled. These services can be developed and deployed independently on, and across a variety of platforms (for example Kubernetes, VMs, SaaS or FaaS).
- Event producers and event consumers are independent. Any producer (or source), can generate events before there are active event consumers that are listening. Any event consumer can express interest in an event or class of events, before there are producers that are creating those events.
- Other services can be connected to the Eventing system. These services can perform the following functions: Create new applications without modifying the event producer or event consumer.
- Select and target specific subsets of the events from their producers.
- Ensure cross-service interoperability. Knative Eventing is consistent with the CloudEvents specification that is developed by the CNCF Serverless WG.


### Event consumers
２種類のインターフェース
+ Addressable：status.address.hostnameで定義されたアドレスにHTTPでイベントを伝達し、ACKを返す
+ Callable：HTTPでイベントを伝達し、そのイベントに対して０か１つの新しいイベントを返す。返されたイベントは同じように処理され、

## Event brokers and triggers
Brokerが、属性で選別可能なイベントを溜め込んでおいて、Triggerとして定義された条件に合うものをAddressableに渡す。
必要に応じてTriggerは複数作成できる。

## Event registry
コンシューマーが、異なるBrokerから利用するイベントのタイプを識別することができるようにするために、EventTypeっていうオブジェクトを提供している。
Registryは、このEventTypeの集合で構成される。
Registryに格納されているEventTypeには、コンシューマーがTriggerを姿勢するの必要とする情報が全て含まれている。

EventRegistryの詳細：https://knative.dev/docs/eventing/event-registry/


## Event channels and subscriptions
Eventngでは、Channelというイベントの転送と保管を担うレイヤーがある。メッセージングの実装において、チャネルは、ClusterChannelProvisionerというオブジェクトとして提供される。イベントは、サービスもしくは他のチャネルにSubscriptionsを使用して届けられる。これによって、クラスター内のメッセージの伝達は多様性を保つことができる。あるイベントではインメモリで処理されたり、他ではKafkaやNATSStramingなどをs利用して永続化されたりする。

## Future design goals
将来のEventingのリリースにおいては、より簡単なイベントソースの実装にフォーカスされる。ソースは、外部システムからのイベントの伝達や登録をKubernetes CRDを使用して管理される。

## Architecture
ソースから1つのサービへ直接転送する（Knative Serviceや、コアなKubernetesのServiceなどのAddressableエンドポイント）。このケースでは、ソースは宛先のサービスが利用できない場合にリトライやイベントのキューイングなどに責任を持つ。
Channelや、Subxcriptionsを利用した、ソースやサービス応答からの複数のエンドポイントへのFan-outの伝達。このケースでは、チャネルの実装が宛先への伝送に責任を持つ。

実際のメッセージの転送は複数のデータプレーンコンポーネントによって実装される。データプレーンは、Observability、パーシスタンス、異なるメッセージングプロトコルの変換などを行う。

## Sources
各ソースは、k8sのカスタムリソース。ソースのインスタンスを作成する場合に、パラメータの指定が可能になっている。
Knative イベンティングにおいては、sources.eventing.knative.dev APIグループで以下のようなソースを提供している。

以下のタイプはGoで書かれているが、YAMLのリストなどでも記述可能。全てのソースは、「ソース」カテゴリに含まれていなければならず、よって存在しているソースはkubectl get sources コマンドで取得可能。
以下がそのリスト。
- KubernetesEventSource
- GitHubSource
- GcpPubSubSource
- AwsSqsSource
- ContainerSource
- CronJobSource
- KafkaSource
- CamelSource

In addition to the core sources (explained below), there are other sources that you can install.
If you need a Source not covered by the available Source implementations, there is a tutorial on writing your own Source.
If your code needs to send events as part of its business logic and doesn’t fit the model of a Source, consider feeding events directly to a Broker.
