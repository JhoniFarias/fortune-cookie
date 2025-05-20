
# Guia de Instalação e Configuração do Minikube, ArgoCD e Kubernetes

## 1. Instalação do Minikube

### Instalar o Minikube
```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

### Instalar o Kubectl
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

### Download do ArgoCD CLI (versão 2.13.3)
```bash
curl -sSL -o /tmp/argocd-v2.13.3 https://github.com/argoproj/argo-cd/releases/download/v2.13.3/argocd-linux-amd64
chmod +x /tmp/argocd-v2.13.3
sudo mv /tmp/argocd-v2.13.3 /usr/local/bin/argocd
```

## 2. Criação do Cluster com o Minikube

### Configuração do arquivo KUBECONFIG
```bash
export KUBECONFIG=$HOME/.kube/config
mkdir $HOME/.kube
touch $KUBECONFIG
```

### Instalação do Podman (ou Docker)
```bash
sudo apt install podman -y
```

### Startando o socket do Podman
```bash
systemctl --user start podman.socket
```

### Acessar o arquivo sudoers
```bash
sudo vi /etc/sudoers
```

### Configuração do usuário para acessar o Podman
```bash
[usuario] ALL=(ALL) NOPASSWD: /usr/bin/podman
```

### Start do Minikube com o driver do Podman
```bash
minikube start -- driver=podman
```

### Comando para verificar o status do Minikube
```bash
minikube status
```

## 3. Configuração do ArgoCD

### Criação do Namespace
```bash
kubectl create namespace argocd
```

### Instalação do ArgoCD
```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Port-forward do serviço ArgoCD para teste
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

### Buscar a secret inicial
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

### Login no ArgoCD via terminal
```bash
argocd login localhost:8080 --username admin --password ***** --insecure
```

### Listar todos os clusters
```bash
argocd cluster list
```

### Gerar chave de acesso SSH
```bash
ssh-keygen -t rsa -b 4096 -C "SEU_EMAIL" -f ~/.ssh/argocd_rsa
```

### Editando o bashrc
```bash
vi .bashrc
eval "$(ssh-agent -s)"
source .bashrc
ssh-add ~/.ssh/argocd_rsa
```

### Adicionando a chave pública no GitHub
```bash
cat .ssh/argocd_rsa.pub
```
- Copiar o conteúdo da chave pública e adicionar na seção "SSH and GPG keys" do GitHub com o nome `argocd-chave-alura`.

### Testando a conexão SSH no GitHub
```bash
ssh -T git@github.com
```

### Adicionar o repositório no ArgoCD
```bash
argocd repo add git@github.com:JhoniFarias/fortune-cookie.git --ssh-private-key-path ~/.ssh/argocd_rsa_no_passphrase
```

## 4. Criar a aplicação no ArgoCD

### Criar a aplicação no ArgoCD via comando
```bash
argocd app create fortune-cookie   --repo git@github.com:[seu-usuario]/fortune-cookie.git   --path k8s-manifests   --dest-server https://kubernetes.default.svc   --dest-namespace default   --revision argocd
```

### Listar todas as revisões da aplicação
```bash
argocd app history fortune-cookie
```

### Rollback de versão
```bash
argocd app rollback fortune-cookie (1,2,3)
```
