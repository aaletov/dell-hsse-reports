# Поздняков Артемий.
# Отчёт по первому заданию лаборатории Dell Technologies - ВШПИ.
## Установка baremetal-csi на kind.

Установил [`go v.16.16`](https://go.dev/dl/) для linux-amd64.
```
curl -LO https://go.dev/dl/go1.16.15.linux-amd64.tar.gz
tar -xf go1.16.15.linux-amd64.tar.gz
# Добавить в ~/.bashrc
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin
```
Установил `kubectl v1.19.16` пользуясь [руководством](https://kubernetes.io/ru/docs/tasks/tools/install-kubectl/).
Установил `helm` пользуясь [руководством](https://helm.sh/docs/intro/install/).

Загрузил `csi-baremetal` и `csi-baremetal-operator`.

```
git clone https://github.com/dell/csi-baremetal
git clone https://github.com/dell/csi-baremetal-operator
```

Изменил тестовый StatefulSet `csi-baremetal/test/app/nginx.yaml`, убрав оттуда все PVC.

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 3
  podManagementPolicy: Parallel
  selector:
    matchLabels:
      app: nginx
      app.kubernetes.io/name: nginx
  template:
    metadata:
      labels:
        app: nginx
        app.kubernetes.io/name: nginx
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - nginx
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
```

Проделал все действия, описанные в [руководстве](https://github.com/dell/csi-baremetal/blob/master/docs/CONTRIBUTING.md).

Прошёл валидацию.

```
aaletov@LAPTOP-CP48S456:~/uni/dell-hsse$ kubectl get pods
NAME                                             READY   STATUS    RESTARTS   AGE
csi-baremetal-controller-6b79fcbbdb-ls45d        4/4     Running   0          170m
csi-baremetal-node-controller-7d9b84c47f-8ljgw   1/1     Running   0          170m
csi-baremetal-node-kernel-5.4-7kbpx              4/4     Running   0          170m
csi-baremetal-node-kernel-5.4-gl9sl              4/4     Running   0          170m
csi-baremetal-node-kernel-5.4-mvjw5              4/4     Running   0          170m
csi-baremetal-operator-6c548457b7-nw49b          1/1     Running   0          170m
csi-baremetal-se-patcher-rm875                   1/1     Running   0          170m
csi-baremetal-se-r8d2q                           1/1     Running   0          170m
web-0                                            1/1     Running   0          169m
web-1                                            1/1     Running   0          169m
web-2                                            1/1     Running   0          169m
aaletov@LAPTOP-CP48S456:~/uni/dell-hsse$ kubectl get pvc
No resources found in default namespace.
```
