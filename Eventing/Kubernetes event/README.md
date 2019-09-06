# Kubernetes event

以下で試行することにする。
https://knative.dev/docs/eventing/samples/kubernetes-event-source/


### Broker
```
(⎈ |tomcluster:default)eb82649:~ eb82649@jp.ibm.com$ kubectl label namespace default knative-eventing-injection=enabled
namespace/default labeled
```

### Service Account
ApiServerSourceを動かすサービスアカウントを作成する
ApiServerSourceは、イベントを監視してBrokerに渡す。
```
(⎈ |tomcluster:default)eb82649:~ eb82649@jp.ibm.com$ kubectl apply --filename serviceaccount.yaml 
serviceaccount/events-sa created
clusterrole.rbac.authorization.k8s.io/event-watcher created
clusterrolebinding.rbac.authorization.k8s.io/k8s-ra-event-watcher created
```

### Eventソースの作成

```
(⎈ |tomcluster:default)eb82649:~ eb82649@jp.ibm.com$ kubectl apply --filename /Users/eb82649@jp.ibm.com/Box\ Sync/ISE\ Cloud\ Native\ Apps\ 関連タスク/10\ SD/knative/work/ohta/k8s-events.yaml 
apiserversource.sources.eventing.knative.dev/testevents created
```

### Trigger
ApiServerSourceを動かすために、Knativeサービスを作成する。入ってきたメッセージwログに書き出して、Triggerを作成する
```
(⎈ |tomcluster:default)eb82649:~ eb82649@jp.ibm.com$ kubectl apply --filename trigger.yaml 
trigger.eventing.knative.dev/testevents-trigger created
service.serving.knative.dev/event-display created
```

### イベントを生成する
```
(⎈ |tomcluster:default)eb82649:~ eb82649@jp.ibm.com$ kubectl run busybox --image=busybox --restart=Never -- ls
pod/busybox created
(⎈ |tomcluster:default)eb82649:~ eb82649@jp.ibm.com$ kubectl delete pod busybox
pod "busybox" deleted
```

### 確認
```
(⎈ |tomcluster:default)eb82649:~ eb82649@jp.ibm.com$ kubectl get pods
NAME                                     READY   STATUS    RESTARTS   AGE
default-broker-filter-74bdd77cf-q24cc    1/1     Running   0          18m
default-broker-ingress-55bff44b9-rvq4q   1/1     Running   0          18m
details-v1-86bf466fc-ffqsc               2/2     Running   0          17h
productpage-v1-5c4bbd6dcc-64tq5          2/2     Running   0          17h
ratings-v1-645c7d554f-69xtj              2/2     Running   0          17h
reviews-v1-7db855b4f-qb65g               2/2     Running   0          17h
reviews-v2-7c49994c7d-lf848              2/2     Running   0          17h
reviews-v3-7c5d8f7f99-gbrwm              2/2     Running   0          17h


(⎈ |tomcluster:default)eb82649:~ eb82649@jp.ibm.com$ kubectl logs -l serving.knative.dev/service=event-display -c user-container
：
☁️  CloudEvent: valid ✅
Context Attributes,
  SpecVersion: 0.2
  Type: dev.knative.apiserver.resource.add
  Source: https://172.21.0.1:443
  ID: dbc7bd50-cbbc-4e1d-90ed-ef643764c6b1
  Time: 2019-08-09T04:11:38.177208476Z
  ContentType: application/json
  Extensions: 
    subject: /apis/v1/namespaces/default/events/event-display-jfz2r-deployment-79d8699f66.15b925cabc81048c
    knativehistory: default-kn-trigger-channel-brt5f.default.svc.cluster.local
Transport Context,
  URI: /
  Host: event-display.default.svc.cluster.local
  Method: POST
Data,
  {
    "apiVersion": "v1",
    "count": 1,
    "eventTime": null,
    "firstTimestamp": "2019-08-09T04:11:38Z",
    "involvedObject": {
      "apiVersion": "apps/v1",
      "kind": "ReplicaSet",
      "name": "event-display-jfz2r-deployment-79d8699f66",
      "namespace": "default",
      "resourceVersion": "969012",
      "uid": "61ea4a81-6ca7-48fe-9084-bc717d961a74"
    },
    "kind": "Event",
    "lastTimestamp": "2019-08-09T04:11:38Z",
    "message": "(combined from similar events): Created pod: event-display-jfz2r-deployment-79d8699f66-2tbpx",
    "metadata": {
      "creationTimestamp": "2019-08-09T04:11:38Z",
      "name": "event-display-jfz2r-deployment-79d8699f66.15b925cabc81048c",
      "namespace": "default",
      "resourceVersion": "969020",
      "selfLink": "/api/v1/namespaces/default/events/event-display-jfz2r-deployment-79d8699f66.15b925cabc81048c",
      "uid": "96da1687-349f-4ea9-8529-abc09523cccc"
    },
    "reason": "SuccessfulCreate",
    "reportingComponent": "",
    "reportingInstance": "",
    "source": {
      "component": "replicaset-controller"
    },
    "type": "Normal"
  }
：
```

一応、k8s内で発生したイベントが発行され、それが見えるところまで確認できたが、その過程などこれだけだとなかなか追えない。
