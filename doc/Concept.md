
# 1. Вступ

Аналізу трьох інструментів для розгортання Kubernetes кластерів в локальному середовищі — minikube, kind та k3d:

- **minikube** — найпоширеніший dev-кластер, який працює через драйвери (VM, Docker, Podman).
- **kind** — Kubernetes IN Docker, орієнтований на швидкі lightweight-кластери та CI.
- **k3d** — обгортка над k3s, надшвидка та легка альтернатива “ванільному” Kubernetes.

Також враховується ризик ліцензування Docker Desktop і можливість роботи через Podman.

---

## 2. Характеристики інструментів

### Minikube
- Платформи: Linux, Windows, macOS.
- Архітектури: amd64, arm64.
- Драйвери запуску: Docker, Podman, VirtualBox, KVM, Hyper-V.
- Має систему **addons** (Ingress, Dashboard, Registry, Metrics).
- Підходить для локальної розробки, навчання та простої симуляції Kubernetes.

### kind (Kubernetes IN Docker)
- Платформи: Linux, Windows, macOS.
- Архітектури: amd64, arm64.
- Всі ноди — Docker-контейнери.
- Легкий, швидкий, ідеальний для CI/CD.
- Немає встроєних addons, все встановлюється вручну.
- Офіційна підтримка rootless Docker/Podman — experimental.

### k3d
- Платформи: Linux, Windows (через Docker Desktop/WSL2), macOS.
- Дистрибутив k3s від Rancher — легкий, із зменшеною кількістю компонентів.
- Вбудовані компоненти: ServiceLB, Traefik.
- Найшвидший інструмент для створення multi-node кластерів.
- Працює поверх Docker або Podman (experimental).

---
- ## 3. Порівняльна таблиця 

| Характеристика | **Minikube** | **kind** | **k3d** |
|-----------------------------|----------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------|
| **Підтримувані платформи** | Linux, macOS, Windows | Linux, macOS, Windows | Linux, macOS, Windows (через Docker Desktop або WSL2) |
| **Архітектури** | amd64, arm64 | amd64, arm64 | amd64, arm64 |
| **Тип Kubernetes** | Vanilla Kubernetes | Vanilla Kubernetes | k3s (lightweight Kubernetes від Rancher) |
| **Швидкість запуску** | Середня (особливо з VM-драйверами) | Висока | Найвища |
| **Використання ресурсів** | Високе (особливо при використанні VM) | Низьке | Найнижче |
| **Multi-node** | Підтримується, але важче та повільніше | Підтримується, легко | Підтримується, найпростіше |
| **Вбудовані функції / Addons** | Dashboard, Ingress, Metrics-server, Registry | Немає (все вручну) | ServiceLB, Traefik (можна вимкнути) |
| **Придатність до CI/CD** | Середня | Висока | Висока |
| **Простота установки** | Проста | Дуже проста | Дуже проста |
| **Простота використання** | Висока | Висока | Найвища |
| **Підтримка Podman** | Experimental, можливі проблеми | Experimental, можливі регресії | Experimental через Docker API у Podman |
| **Мережеві особливості** | Гнучкі, через драйвери | Порожня docker-мережа, треба налаштовувати | Автоматичний ServiceLB, просте порт-маппінг |
| **Сумісність з production** | Найближче до реального Kubernetes | Близько, але без деяких компонентів | k3s не ідентичний production-k8s |
| **Ідеальний use-case** | Навчання, локальна розробка | CI/CD, інтеграційні тести | PoC, microservices, ML-прототипи |
| **Недоліки** | Важкий, повільніший, VM споживають ресурси | Немає addons, іноді важко з мережею | Не "ванільний" Kubernetes |
| **Переваги** | Addons + повний Kubernetes | Легкий, ідеальний для GitHub Actions | Найшвидший, найлегший, multi-node “в один рядок” |

---
## 4. Вбудоване демо 

[![asciicast](https://asciinema.org/a/xtNMIgEUuijROt7YMOPURDqDe.svg)](https://asciinema.org/a/xtNMIgEUuijROt7YMOPURDqDe)

Нижче наведено демонстрацію розгортання простого застосунку “Hello World” у кластері **k3d**, який рекомендовано для стартапу.

Інструкція зі встановлення k3d:  
🔗 https://k3d.io/v5.6.0/#releases

### Створення кластера
k3d cluster create asciiartify \
  --servers 1 \
  --agents 2 \
  -p "30080:80@loadbalancer"

### Перевірка стану
kubectl get nodes
kubectl get pods -A

### Деплой “Hello World”
kubectl create deployment hello --image=nginx:alpine
kubectl expose deployment hello --port=80 --type=ClusterIP

### Зміна типу сервісу на LoadBalancer
kubectl patch svc hello -p '{"spec": {"type": "LoadBalancer"}}'





### Створення Ingress
kubectl apply -f - <<'EOF'
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: hello
            port:
              number: 80

kubectl get ingress


### Перевірка
curl http://localhost:30080

### Очікуваний результат:
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

---

## 5. Висновки

### **Minikube**
- навчання Kubernetes;
- роботи з dashboard;
- емуляції “майже продакшен” кластера.

Minikube зручний, максимально наближено до vanilla Kubernetes, але він споживає більше ресурсів і повільніший при multi-node.

---

### **kind**
- CI/CD pipelines;
- інтеграційних тестів;
- швидкого створення disposable-кластерів.

kind — чудовий інструмент для автоматизації та тестування, але потребує ручної інсталяції компонентів (Ingress, Metrics) і не має addons з коробки.

---

### **k3d — рекомендований інструмент**
- мінімальне споживання ресурсів;
- дуже швидке створення кластерів;
- зручність використання для розробників без глибоких DevOps-знань;
- ідеально підходить для ML-прототипів та microservices;
- простота розгортання multi-node в одну команду;
- автоматичний ServiceLB та просте порт-маппінг.
