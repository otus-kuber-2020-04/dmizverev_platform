# dmizverev_platform
Dmitriy Zverev Platform repository
dmizverev@yandex.ru

# Урок №2. Знакомство с Kubernetes, основные понятия и архитектура

> Разберитесь почему все pod в namespace kube-system
восстановились после удаления. Укажите причину в описании PR

Предположу, что: 
- kube-apiserver - это static Pod. За его восстановлением следит непосредственно kubelet.
См. https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/
- За восстановление cor-edns, kube-dns отвечает ReplicaSetController.

> Для выполнения домашней работы необходимо создать
> Dockerfile, в котором будет описан образ, запускающий web-сервер на порту 8000

Создан docker image с python web-server `http.server`. 
Docker image размещен в registry `docker.io/dmizverev/otus-frontend:1.0.0`.

> Напишем манифест web-pod.yaml для создания pod web c
> меткой app со значением web, содержащего один контейнер с
> названием web.

Выполнено.

> Добавим в наш pod init-container, генерирующий страницу
> index.html.

Выполнено.

> Для того, чтобы файлы, созданные в init контейнере, были
> доступны основному контейнеру в pod нам понадобится
> использовать volume типа emptyDir

Выполнено.

> Склонируйте и соберите собственный образ для
frontend (используйте готовый Dockerfile)
Поместите собранный образ на Docker Hub

Выполнено

> генерация манифестов средствами kubectl

Выполнено. Файл: frontend-pod.yaml

> Выясните причину, по которой pod frontend находится в статусе
Error

Не заданы переменные окружения

> Создайте новый манифест frontend-pod-healthy.yaml. При его
применении ошибка должна исчезнуть

Выполнено.

# Урок №3. Механика запуска и взаимодействия контейнеров в Kubernetes

Создан ReplicaSet frontend с replicas=3.

> почему обновление ReplicaSet не повлекло обновление
запущенных pod?

ReplicaSet не проверяет соответствие запущенных Pod'ов шаблону. Ему важен только факт наличия Pod.

Создан ReplicaSet pymentservice с replicaset=3.

Создан Deployment paymentservice.

Созданы Deployment с реализацией стратегии обновления Blue-Green и Reverse Rolling Update.

Создан Deployment frontend с readinessProbe.

Создан DaemonSet node-exporter, в котором выставлен toleration к запуску Pod на Kubernetes master.

# Урок №4. Безопасность и управление доступом

Создан ServiceAccount bob. Создан ClusterRoleBinding adminbob, где для bob выдана роль admin.

Создан ServiceAccount dave.

Создан Namespace prometheus. В нем создан ServiceAccount carol.
Создана clusterrole с возможностью для Pod выполнять get list watch.
Эта роль выдана на namespace prometheus.

Создан Namespace dev. В нем создан ServiceAccount jave с ролью admin и ServiceAccount ken
с ролью view.

# Урок №5. Сетевая подсистема Kubernetes

- В Pod web-pod.yaml добавлены Liveness и Readiness probes
- Создан Deployment и Service с типом ClusterIP
- Создан Service с типом ClusterIP с типом LoadBalancer
- Создан headless Service web-svc
- Создан Ingress перенаправляющий запрос с `/web` на headless Service web-svc по порту 8000
  
  Использовать https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.30.0/deploy/static/mandatory.yaml


# Урок №6. Volumes, Storages, StatefulSet

Создан StatefulSet с Pod minio. Environment variables передаются в container из
secret minio-secret. Для minio создан Headless Service. 
Statefulset хранит данные в каталоге `data`.
Проверка того, что данные там есть:
```bash
# in container
> ls -la /data
total 24
drwxr-xr-x    4 root     root          4096 May 22 08:35 .
drwxr-xr-x    1 root     root            64 May 22 08:35 ..
drwxr-xr-x    5 root     root          4096 May 22 08:34 .minio.sys
drwx------    2 root     root         16384 May 22 08:34 lost+found
```
Проверка создаваемых ресурсов Kubernetes
```bash
> kubectl get statefulset
NAME                                                   READY   AGE
alertmanager-prometheus-operator-158141-alertmanager   1/1     100d
minio                                                  1/1     6m9s
prometheus-prometheus-operator-158141-prometheus       1/1     100d

> kubectl get pods minio-0
NAME      READY   STATUS    RESTARTS   AGE
minio-0   1/1     Running   0          7m37s

> kubectl get pvc
NAME             STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
csi-pvc-cinder   Bound    pvc-eb8f17da-6b0b-4a3e-b846-e869c24a564a   1Gi        RWO            csi-cinder     98d
data-minio-0     Bound    pvc-a9769bff-64c9-41f4-ac2d-ce783bf9c9a7   1Gi        RWO            csi-cinder     10m

> kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS   REASON   AGE
pvc-a9769bff-64c9-41f4-ac2d-ce783bf9c9a7   1Gi        RWO            Delete           Bound    default/data-minio-0     csi-cinder              10m
pvc-eb8f17da-6b0b-4a3e-b846-e869c24a564a   1Gi        RWO            Delete           Bound    default/csi-pvc-cinder   csi-cinder              98d

> kubectl describe pvc data-minio-0
Name:          data-minio-0
Namespace:     default
StorageClass:  csi-cinder
Status:        Bound
Volume:        pvc-a9769bff-64c9-41f4-ac2d-ce783bf9c9a7
Labels:        app=minio
Annotations:   pv.kubernetes.io/bind-completed: yes
               pv.kubernetes.io/bound-by-controller: yes
               volume.beta.kubernetes.io/storage-provisioner: cinder.csi.openstack.org
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      1Gi
Access Modes:  RWO
VolumeMode:    Filesystem
Mounted By:    minio-0
Events:
  Type    Reason                 Age                From                                                                                         Message
  ----    ------                 ----               ----                                                                                         -------
  Normal  ExternalProvisioning   15m (x2 over 15m)  persistentvolume-controller                                                                  waiting for a volume to be created, either
 by external provisioner "cinder.csi.openstack.org" or manually created by system administrator
  Normal  Provisioning           8m43s              cinder.csi.openstack.org_csi-cinder-controllerplugin-0_b9cf93ad-4928-4011-9173-39c6bcc6ecbc  External provisioner is provisioning volum
e for claim "default/data-minio-0"
  Normal  ProvisioningSucceeded  8m42s              cinder.csi.openstack.org_csi-cinder-controllerplugin-0_b9cf93ad-4928-4011-9173-39c6bcc6ecbc  Successfully provisioned volume pvc-a9769b
ff-64c9-41f4-ac2d-ce783bf9c9a7

```

# Урок №7. Шаблонизация манифестов.

- Создан аккаунт в Google Cloud.

- Создан Kubernetes кластер. Его конфигурация подключена к `kubectl`.
    ```bash
    $ kubectl cluster-info
    Kubernetes master is running at https://34.76.167.115
    GLBCDefaultBackend is running at https://34.76.167.115/api/v1/namespaces/kube-system/services/default-http-backend:http/proxy
    KubeDNS is running at https://34.76.167.115/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
    Metrics-server is running at https://34.76.167.115/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy
    ```
- Установлен Helm 3.

- Добавлен Helm-репозиторий `https://kubernetes-charts.storage.googleapis.com`.

- Установлен Helm release stable/nginx-ingress версии 1.39.0.
    
    Версия 1.11.1 устанавливается с ошибкой.
    ```bash
    helm repo add stable https://kubernetes-charts.storage.googleapis.com
    helm repo list
    kubectl apply -f kubernetes-templating/nginx-ingress/namespace.yaml
    helm upgrade --install nginx-ingress stable/nginx-ingress --wait --namespace=nginx-ingress --version=1.39.0  
    ```

- Установлен cert-manager версии 0.15.0
  
  Создан ClusterIssuer
  ```bash
  helm repo add jetstack https://charts.jetstack.io
  kubectl apply -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.9/deploy/manifests/00-crds.yaml
  kubectl create namespace cert-manager
  kubectl label namespace cert-manager certmanager.k8s.io/disable-validation="true"
  helm upgrade --install cert-manager jetstack/cert-manager --wait --namespace=cert-manager --version=0.15.0
  kubectl apply -f kubernetes-templating/cert-manager/cluster-issuer.yaml
  ```

  Cert-manager протестирован при помощи файла `kubernetes-templating\cert-manager\test-resources.yaml`

- Установлен chartmuseum версии 2.13.0
  ```bash
  kubectl create namespace chartmuseum
  helm upgrade --install chartmuseum stable/chartmuseum --namespace=chartmuseum --version=2.13.0 -f kubernetes-templating/chartmuseum/values.yaml
  ``` 
  
  Chartmuseum доступен по адресу: https://chartmuseum.34.78.254.203.nip.io

- Сборка helm chart 
    
    Команды для сборки Helm chart и итправки их в helm repository 
    ```bash
    #!/usr/bin/env bash
   
    set -e
    
    export MYPRODUCT_VERSION=8.1.0
  
    helm repo add chartmuseum https://chartmuseum.34.78.254.203.nip.io
    helm plugin install https://github.com/chartmuseum/helm-push.git
    
    # Lint Helm chart
    helm lint myproduct
    
    # Package Helm chart
    echo "==> Package myproduct"
    helm package myproduct \
    --app-version $MYPRODUCT_VERSION \
    --version $MYPRODUCT_VERSION \
    --dependency-update \
    --destination .  
  
    echo "==> Push myproduct"
    helm push myproduct-$MYPRODUCT_VERSION.tgz chartmuseum
  ```

- Установлен harbor версии 1.3.2
  ```bash 
  helm repo add harbor https://helm.goharbor.io
  kubectl create namespace harbor
  helm upgrade --install harbor harbor/harbor --namespace=harbor --version=1.3.2  -f kubernetes-templating/harbor/values.yaml 
  ```
  Harbor доступен по адресу: `https://harbor.34.78.254.203.nip.io`.  
  
  Реквизиты по умолчанию - admin/Harbor12345
    
- Созданы helm chart frontend и heapster-shop
  
  Frontend является зависимостью к heapster-shop

- Установлен heapster-shop
  ```bash
  kubectl create ns hipster-shop
  helm dep update kubernetes-templating/hipster-shop 
  helm upgrade --install hipster-shop --atomic kubernetes-templating/hipster-shop --namespace hipster-shop
  ```
  
  Heapster-shop доступен по адресу: `http://shop.34.78.254.203.nip.io/` 

- Создан пакет hipster-shop
    ```bash
    helm package kubernetes-templating/hipster-shop
  ```

- Kubecfg
  
  Установлен kubecfg
  
  Создан шаблон `kubernetes-templating\kubecfg\services.jsonnet`, описывающий сервисы paymentservice и shippingservice
  
  Применение шаблона
  ```bash
  kubecfg update kubernetes-templating/kubecfg/services.jsonnet --namespace hipster-shop
  ```
  
- Kustomize
  
  Шаблонизирован компонент cartservice в каталоге `kubernetes-templating\kustomize`
  
  
  Установка на production
  ```bash
  kubectl apply -k kubernetes-templating/kustomize/overrides/prod/
  ```

# Урок №8. Custom Resource Definitions. Operators
[Описание](doc/lesson-8-kubernetes-operators.md)

# Урок №10. Логирование
[Описание](doc/lesson-10-kubernetes-logging.md)