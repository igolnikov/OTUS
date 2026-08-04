# Домашнее задание:

## Введение в Kubernetes: Работа с хранилищами данных и конфигурациями

### Цель:

- понять ключевые компоненты Kubernetes, такие как ConfigMap, Secrets и Persistent Volumes;научиться работать с ЯО;
- понять ключевые компоненты Kubernetes, такие как ConfigMap, Secrets и Persistent Volumes;
- освоить использование ConfigMap и Secrets для управления конфигурациями и секретами;
- научиться работать с Helm для упрощения управления приложениями в Kubernetes;

### Краткое содержание:

Архитектура и базовые сущности Кubernetes: StatefulSet, ConfigMap, Persistent Volume, Persistent Volume Claim.


#### Подготовка машины для установки дистрибутива:



| Параметр | Значение                                            |
| ---------------- | ----------------------------------------------------------- |
| OS               | Ubuntu 24.04 LTS                                            |
| CPU              | 4 cores                                                     |
| RAM              | 8 GB (minikube съедает много)                   |
| Disk             | 40 GB                                                       |
| Network          | DHCP/статика с доступом в интернет |

```
sudo apt update && sudo apt install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo usermod -aG docker $USER
newgrp docker
```

Установка kubectl:

```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

#### Установка и запуск minikube:

```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

```
VERSION="v1.32.0"   # or check https://github.com/kubernetes-sigs/cri-tools/releases for newer
curl -L https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-$VERSION-linux-amd64.tar.gz | tar -xz -C /usr/local/bin

chmod +x /usr/local/bin/crictl
crictl --version
```

```
VERSION="v1.32.0"
curl -L https://github.com/kubernetes-sigs/cri-tools/releases/download/${VERSION}/crictl-${VERSION}-linux-amd64.tar.gz | tar -C /usr/local/bin -xz
chmod +x /usr/local/bin/crictl

# Проверяем
crictl --version
```

```
VER=$(curl -s https://api.github.com/repos/Mirantis/cri-dockerd/releases/latest | grep tag_name | cut -d '"' -f 4 | sed 's/v//')
echo "Version: $VER"

wget https://github.com/Mirantis/cri-dockerd/releases/download/v${VER}/cri-dockerd-${VER}.amd64.tgz
tar xvf cri-dockerd-${VER}.amd64.tgz
sudo mv cri-dockerd/cri-dockerd /usr/local/bin/
cri-dockerd --version

wget https://raw.githubusercontent.com/Mirantis/cri-dockerd/master/packaging/systemd/cri-docker.service
wget https://raw.githubusercontent.com/Mirantis/cri-dockerd/master/packaging/systemd/cri-docker.socket

sudo mv cri-docker.service cri-docker.socket /etc/systemd/system/
sudo sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service

sudo systemctl daemon-reload
sudo systemctl enable cri-docker.service
sudo systemctl enable --now cri-docker.socket
sudo systemctl start cri-docker

systemctl status cri-docker
```

```
CNI_VERSION="v1.6.2"   # актуальная стабильная версия
curl -L "https://github.com/containernetworking/plugins/releases/download/${CNI_VERSION}/cni-plugins-linux-amd64-${CNI_VERSION}.tgz" | sudo tar -C /opt/cni/bin -xz
```

#### Установка Helm:

```
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
# Проверяем версию 
helm version
```

#### Включи storage-provisioner:

```
minikube addons enable storage-provisioner
minikube addons enable default-storageclass
kubectl get storageclass
```

#### Создаем pod с Postgres:

```
# Создай файл postgres.yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres
  labels:
    app: postgres
spec:
  ports:
    - port: 5432
      name: postgres
  clusterIP: None          # Headless Service — важно для StatefulSet
  selector:
    app: postgres
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: "postgres"
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:15
        ports:
        - containerPort: 5432
          name: postgres
        env:
        - name: POSTGRES_DB
          value: myapp
        - name: POSTGRES_USER
          value: myuser
        - name: POSTGRES_PASSWORD
          value: mypassword          # потом вынесем в Secret
        volumeMounts:
        - name: postgres-data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: postgres-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

#### Применяем наш конфиг postgres:

```
kubectl apply -f postgres.yaml

# Проверяем вывод

kubectl get pods
kubectl get pvc
kubectl get svc
```

#### Создаем  pod Secret с паролем к БД:

```
# Создаем фаил secret.yaml

apiVersion: v1
kind: Secret
metadata:
  name: postgres-secret
type: Opaque
stringData:
  POSTGRES_PASSWORD: mypassword
  DATABASE_URL: "postgresql://myuser:mypassword@postgres:5432/myapp"
```

#### Применяем наш конфиг secret:

```
kubectl apply -f secret.yaml
```

#### Создаем  pod ConfigMap:

```
# Создаем фаил configmap.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_NAME: "Hello Kubernetes"
  APP_ENV: "dev"
  LOG_LEVEL: "info"
```

#### Применяем наш конфиг ConfigMap:

```
kubectl apply -f configmap.yaml
```

#### Веб-приложение (Deployment + Service):

```
# Создаем фаил webapp.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: hashicorp/http-echo
        args:
          - "-text=Hello from Kubernetes + Postgres!"
        ports:
        - containerPort: 5678
        envFrom:
        - configMapRef:
            name: app-config
        - secretRef:
            name: postgres-secret
---
apiVersion: v1
kind: Service
metadata:
  name: webapp
spec:
  type: NodePort
  selector:
    app: webapp
  ports:
    - port: 80
      targetPort: 5678
      nodePort: 30080
```

#### Применяем конфиг webapp.yaml

```
kubectl apply -f webapp.yaml
```

#### Проверяем все вместе:

```
kubectl get pods,svc,secret,configmap,pvc
```

![Альтернативный текст](img/image.png)

Проверяем что pods в статусе Running:

![Альтернативный текст](img/image1.png)

#### Проверяем что наше приложение запустилось и работает:

![Альтернативный текст](img/image2.png)
