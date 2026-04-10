
# Création d’un kubeconfig qui permettra la communication entre le cluster k3s et gitAction

## 1. Création du ServiceAccount

Créer un ServiceAccount nommé **etudiant-srvaccount** dans les trois namespaces :

```bash
kubectl create serviceaccount etudiant-srvaccount -n dev
kubectl create serviceaccount etudiant-srvaccount -n qa
kubectl create serviceaccount etudiant-srvaccount -n prod
```

---

## 2. Création d’un Role avec permissions minimales

Créer un fichier `role-etudiant.yaml` :

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: etudiant-role
rules:
- apiGroups: ["", "apps"]
  resources: ["pods", "services", "deployments"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```

Appliquer ce rôle dans les trois namespaces :

```bash
kubectl apply -f role-etudiant.yaml -n dev
kubectl apply -f role-etudiant.yaml -n qa
kubectl apply -f role-etudiant.yaml -n prod
```

---

# 3️. Création des RoleBindings

Associer le ServiceAccount au rôle dans chaque namespace :

```bash
kubectl create rolebinding etudiant-rb --role=etudiant-role --serviceaccount=dev:etudiant-srvaccount -n dev
kubectl create rolebinding etudiant-rb --role=etudiant-role --serviceaccount=qa:etudiant-srvaccount -n qa
kubectl create rolebinding etudiant-rb --role=etudiant-role --serviceaccount=prod:etudiant-srvaccount -n prod
```

---

# 4️. Génération du token du ServiceAccount

Générer un token pour le ServiceAccount (un seul token suffit pour toute la cohorte) :

```bash
kubectl create token etudiant-srvaccount -n dev
```

Conserver ce token : il sera utilisé dans le kubeconfig.

---

# 5️. Récupération des informations du cluster

Afficher le kubeconfig maître de k3s :

```bash
sudo cat /etc/rancher/k3s/k3s.yaml
```

Récupérer :

- `certificate-authority-data: <BASE64_CA>`

---

# 6️. Construction du kubeconfig final

Créer un fichier `kubeconfig.yaml` :

```yaml

apiVersion: v1
kind: Config

clusters:
- name: k3s-cluster
  cluster:
    server: https://192.168.21.100:6443 # est l'adresse du control-plane
    certificate-authority-data: <CA_BASE64>

users:
- name: etudiant
  user:
    token: <TOKEN_GENERÉ>

contexts:
- name: etudiant-context
  context:
    cluster: k3s-cluster
    user: etudiant
    namespace: dev

current-context: etudiant-context

```

