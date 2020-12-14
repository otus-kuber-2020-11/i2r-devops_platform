# i2r-devops_platform
i2r-devops Platform repository
Описание действий ДЗ:
1. Установлен kubectl
2. Установлен Minikube
2.1 minikube запущен.
3. установлен Dashboard. Еще не запускал.
4. k9s  - не установлен.
5. Выполнена проверка на прочность 
kubectl delete pod --all -n kube-system
Разбераться,  почему все pod в namespace kube-system восстановились после удаления - не удалось, потому что у меня pod в namespace kube-system
НЕ восстановились после удаления.
----------------------------------
kubectl get cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS      MESSAGE                                                                                       ERROR
scheduler            Unhealthy   Get "http://127.0.0.1:10251/healthz": dial tcp 127.0.0.1:10251: connect: connection refused
controller-manager   Unhealthy   Get "http://127.0.0.1:10252/healthz": dial tcp 127.0.0.1:10252: connect: connection refused
etcd-0               Healthy     {"health":"true"}
----------------------------------

6. создан Dockerfile
------
FROM nginx:latest
RUN usermod -u 1001 nginx
EXPOSE 8000
WORKDIR /app
COPY ./default.conf /etc/nginx/conf.d/
-----

6.1 собран образ и помещен в Docker Hub:
i2rdevops/hw2-webserver:latest

6.2 Написан манифест web-pod.yaml
--------------------------------------------
apiVersion: v1 # Версия API
kind: Pod # Объект, который создаем
metadata:
  name: web #Название Pod
  labels: # Метки в формате key: value
    key: value
spec: # Описание Pod
  containers: # Описание контейнеров внутри Pod
  - name: hw2-webserver # Название контейнера
    image: i2rdevops/hw2-webserver:latest # Образ из которого создается контейнер
--------------------------------------------

в namespace default запустился pod web.

6.3 В pod добавлен init контейнер, генерирующий страницу index.html,
и добавлены volumes:
--------------------------------------------
apiVersion: v1 # Версия API
kind: Pod # Объект, который создаем
metadata:
  name: web #Название Pod
  labels: # Метки в формате key: value
    key: value
spec: # Описание Pod
  containers: # Описание контейнеров внутри Pod
  - name: hw2-webserver # Название контейнера
    image: i2rdevops/hw2-webserver:latest # Образ из которого создается контейнер
    imagePullPolicy: Always
    volumeMounts:
      - name: web
        mountPath: /app
  - name: init
    image: busybox:latest
    imagePullPolicy: Always
    command: ['sh', '-c', 'wget -O- https://tinyurl.com/otus-k8s-intro | sh']
    volumeMounts:
      - name: web
        mountPath: /app

  volumes:
    - name: web
      emptyDir: {}
--------------------------------------------

Проверка работы приложения прошла успешно!

6.4 "В качестве альтернативы kubectl port-forward можно использовать удобную обертку - kube-forwarder"  - не получилось запустить.

7.  Hipster Shop
Склонирован и собран собственный образ для frontend и помещен на Docker Hub.
i2rdevops/hw-frontend:latest

7.2  создан frontend-pod-healthy.yaml:
--------------------------------------------
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: frontend
  name: frontend
spec:
  containers:
  - image: i2rdevops/hw-frontend
    name: frontend
    resources: {}
    env:
    - name: PRODUCT_CATALOG_SERVICE_ADDR
      value: "productcatalogservice:3550"
    - name: CURRENCY_SERVICE_ADDR
      value: "currencyservice:7000"
    - name: CART_SERVICE_ADDR
      value: "cartservice:7070"
    - name: RECOMMENDATION_SERVICE_ADDR
      value: "recommendationservice:8080"
    - name: SHIPPING_SERVICE_ADDR
      value: "shippingservice:50051"
    - name: CHECKOUT_SERVICE_ADDR
      value: "checkoutservice:5050"
    - name: AD_SERVICE_ADDR
      value: "adservice:9555"
      #    - name: ENV_PLATFORM
      #value: "gcp"
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
--------------------------------------------

запущен frontend pod:
kubectl apply -f frontend-pod-healthy.yaml
--------------------------------------------

pod запущен и работает успешно:
--------------------------------------------
kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
frontend   1/1     Running   0          68s
--------------------------------------------
