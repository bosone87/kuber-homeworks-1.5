# Домашнее задание к занятию «Сетевое взаимодействие в K8S. Часть 2»

### Цель задания

В тестовой среде Kubernetes необходимо обеспечить доступ к двум приложениям снаружи кластера по разным путям.

------

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым Git-репозиторием.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Инструкция](https://microk8s.io/docs/getting-started) по установке MicroK8S.
2. [Описание](https://kubernetes.io/docs/concepts/services-networking/service/) Service.
3. [Описание](https://kubernetes.io/docs/concepts/services-networking/ingress/) Ingress.
4. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

------

### Задание 1. Создать Deployment приложений backend и frontend

1. Создать Deployment приложения _frontend_ из образа nginx с количеством реплик 3 шт.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-frontend
  labels:
    app: deploy-frontend
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: deploy-frontend
  template:
    metadata:
      labels:
        app: deploy-frontend
    spec:
      containers:
      - name: nginx
        image: nginx:1.25.3-alpine
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
          name: frontend
```

2. Создать Deployment приложения _backend_ из образа multitool. 

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-backend
  labels:
    app: deploy-backend
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: deploy-backend
  template:
    metadata:
      labels:
        app: deploy-backend
    spec:
      containers:
      - name: multitool
        image: wbitt/network-multitool
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
          name: backend
```

3. Добавить Service, которые обеспечат доступ к обоим приложениям внутри кластера. 

| frontend service |
| :---: |
```yaml 
apiVersion: v1
kind: Service
metadata:
  name: svc-frontend
  namespace: default
spec:
  ports:
    - name: nginx
      port: 9001
      targetPort: 80
  selector:
    app: deploy-frontend
  type: ClusterIP
```

| backend service |
| :---: |
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-backend
  namespace: default
spec:
  ports:
    - name: multitool
      port: 9002
      targetPort: 80
  selector:
    app: deploy-backend
  type: ClusterIP
```


<p align="center">
    <img width="1200 height="600" src="/img/deployments_services.png">
</p>

1. Продемонстрировать, что приложения видят друг друга с помощью Service.
2. Предоставить манифесты Deployment и Service в решении, а также скриншоты или вывод команды п.4.

<p align="center">
    <img width="1200 height="600" src="/img/backend_to_frontend_pod_name.png">
</p>

<p align="center">
    <img width="1200 height="600" src="/img/backend_to_frontend_svc_name.png">
</p>

<p align="center">
    <img width="1200 height="600" src="/img/frontend_to_backend_pod_svc_name.png">
</p>

------

### Задание 2. Создать Ingress и обеспечить доступ к приложениям снаружи кластера

1. Включить Ingress-controller в MicroK8S.

<p align="center">
    <img width="1200 height="600" src="/img/ingress_enable.png">
</p>

<p align="center">
    <img width="1200 height="600" src="/img/ingress_create.png">
</p>

2. Создать Ingress, обеспечивающий доступ снаружи по IP-адресу кластера MicroK8S так, чтобы при запросе только по адресу открывался _frontend_ а при добавлении /api - _backend_.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-apps
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: "nginx"
  rules:
  - host: localhost
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: svc-frontend
            port:
              name: nginx
      - path: /api
        pathType: Exact
        backend:
          service:
            name: svc-backend
            port:
              name: multitool
```

3. Продемонстрировать доступ с помощью браузера или `curl` с локального компьютера.
4. Предоставить манифесты и скриншоты или вывод команды п.2.

<p align="center">
    <img width="1200 height="600" src="/img/curl_host_api.png">
</p>

------

### Правила приема работы

1. Домашняя работа оформляется в своем Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl` и скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

------
