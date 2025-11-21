# PoC: Розгортання Kubernetes та доступ до Argo CD UI (AsciiArtify)

Цей документ описує практичні кроки PoC для стартапу **AsciiArtify**:
1. Розгортання локального Kubernetes кластера за допомогою **k3d** (затверджено в Concept).
2. Встановлення **Argo CD**.
3. Отримання доступу команди до графічного інтерфейсу Argo CD у **GitHub Codespaces**.

> Формат демо орієнтований на офіційну документацію Argo CD.

---

## 0. Передумови

- Локальний кластер Kubernetes піднятий через **k3d**.
- Налаштований `kubectl` і є доступ до кластеру.
- У Codespaces встановлений Docker (або Docker-in-Docker у devcontainer).

Перевірка доступу:

```bash
kubectl cluster-info
kubectl get nodes
```

---

## 1. Створення кластера k3d (якщо ще не створено)

> Якщо кластер уже створений — пропусти цей розділ.

```bash
k3d cluster create asciiartify \
  --servers 1 \
  --agents 2 \
  -p "30080:80@loadbalancer"
```

Перевірка:

```bash
kubectl get nodes
kubectl get pods -A
```

---

## 2. Встановлення Argo CD

### 2.1 Створення namespace

```bash
kubectl create namespace argocd
```

### 2.2 Інсталяція з офіційного маніфесту

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 2.3 Перевірка готовності компонентів

```bash
kubectl get pods -n argocd
```

Очікувано: основні поди `argocd-*` у статусі `Running` (jobs можуть бути `Completed`).

---

## 3. Доступ до Argo CD UI у GitHub Codespaces

За замовчуванням сервіс `argocd-server` має тип `ClusterIP`, тож UI недоступний зовні.
У Codespaces **найпростіший та надійний спосіб — port-forward**.

### 3.1 Port-forward Argo CD server

> Важливо: Argo CD UI працює по **HTTPS**, тому прокидаємо порт 443 на хостовий 8443.

```bash
kubectl port-forward --address 0.0.0.0 svc/argocd-server -n argocd 8443:443
```

Термінал має “висіти” з повідомленням:

```
Forwarding from 0.0.0.0:8443 -> 443
```

### 3.2 Відкриття порту в Codespaces

1. Внизу Codespaces відкрий вкладку **Ports**.
2. Знайди порт **8443**.
3. Натисни **Open in Browser** / іконку 🌐.
4. Якщо Codespaces питає видимість — вистав **Private** або **Public** (для демо можна Public).

Браузер відкриє URL виду:

```
https://<codespace-name>-8443.app.github.dev
```

> Може зʼявитися попередження про self‑signed сертифікат — натисни **Advanced → Continue**.

---

## 4. Отримання admin‑пароля

Логін **admin** створюється автоматично. Початковий пароль лежить у secret.

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d && echo
```

### Дані для входу:

- **Username:** `admin`  
- **Password:** *(вивід команди вище)*

---

## 5. (Опційно) Логін через Argo CD CLI

Якщо потрібно керувати Argo CD з CLI:

```bash
argocd login localhost:8443 \
  --username admin \
  --password <PASSWORD> \
  --insecure
```

Перевірка:

```bash
argocd account get-user-info
```

---

## 6. Troubleshooting (типові проблеми)

### 6.1 Порт зайнятий

Якщо бачиш `address already in use`, візьми інший порт, наприклад 9443:

```bash
kubectl port-forward --address 0.0.0.0 svc/argocd-server -n argocd 9443:443
```

Відкривай у браузері відповідний порт у вкладці **Ports**.

### 6.2 404 у браузері

Причина майже завжди в тому, що відкрили **HTTP** порт замість **HTTPS**.  
Переконайся, що port-forward саме `8443:443`, і URL у браузері починається з `https://`.

### 6.3 Пода `argocd-server` ще не Ready

Почекай і перевір знову:

```bash
kubectl get pods -n argocd
kubectl describe pod -n argocd -l app.kubernetes.io/name=argocd-server
```

---

## 7. Результат PoC

- Kubernetes‑кластер **k3d** успішно розгорнутий локально.
- Argo CD встановлено у namespace `argocd`.
- Команда отримала доступ до web‑UI Argo CD через Codespaces port‑forward.

Файл підготовлено для репозиторію:  
`doc/POC.md`

