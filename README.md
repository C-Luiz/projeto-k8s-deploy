Projeto de Deploy de Aplicação Fullstack em Kubernetes
Este projeto demonstra o deploy de uma aplicação fullstack (React, Flask, PostgreSQL) em um cluster Kubernetes, garantindo alta disponibilidade, gerenciamento de configuração e persistência de dados.

👥 Integrantes da Equipe
[Seu Nome 1] - [Link para seu GitHub/LinkedIn (opcional)]

[Seu Nome 2] - [Link para seu GitHub/LinkedIn (opcional)]

[Seu Nome 3] - [Link para seu GitHub/LinkedIn (opcional)]
(Adicione todos os membros da sua equipe aqui)

🎯 Objetivo do Projeto
O objetivo principal desta atividade é praticar o uso dos principais componentes do Kubernetes para realizar o deploy completo de uma aplicação que consiste em:

Backend (Flask): API REST para gerenciamento de mensagens.

Frontend (React): Interface web que consome a API do backend.

Banco de Dados (PostgreSQL): Armazenamento persistente para as mensagens.

O deploy visa garantir:

Alta Disponibilidade: Múltiplas réplicas para frontend e backend.

Configuração Dinâmica: Uso de ConfigMap e Secret para variáveis de ambiente e credenciais.

Persistência de Dados: Garantia de que os dados do banco não serão perdidos.

Acesso Externo: Comunicação via IngressController NGINX.

🚀 Arquitetura da Aplicação
A aplicação é dividida em três componentes principais:

Frontend (React): Servido por Nginx em um contêiner, consome a API do backend através de um caminho /api/ roteado pelo Ingress. Deployado como um Deployment com 2 réplicas.

Backend (Flask): API Python que interage com o banco de dados. Deployado como um Deployment com 2 réplicas. Configurações de ambiente são injetadas via ConfigMap e credenciais via Secret.

Banco de Dados (PostgreSQL): Armazena as mensagens. Deployado como um StatefulSet com 1 réplica, utilizando um PersistentVolumeClaim (PVC) para garantir a persistência dos dados. Credenciais são injetadas via Secret.

Comunicação:

Interna: Frontend e Backend se comunicam com o PostgreSQL usando Service (ClusterIP).

Externa: O acesso à aplicação (frontend) e à API do backend é feito através de um Ingress NGINX, que roteia o tráfego baseado em caminhos:

http://<IP_DO_CLUSTER>/ ➡️ Frontend

http://<IP_DO_CLUSTER>/api/ ➡️ Backend

Os recursos são organizados em dois Namespaces para melhor isolamento: app (para frontend e backend) e database (para PostgreSQL).

📁 Estrutura do Projeto
projeto-k8s-deploy/
├── README.md                          # Este arquivo
├── frontend/
│   ├── deployment.yaml                # Deployment + Service do React
│   ├── Dockerfile                     # Dockerfile para a imagem do frontend
│   └── src/App.jsx                    # Código fonte do frontend (alterado para caminho relativo)
├── backend/
│   ├── deployment.yaml                # Deployment + Service do Flask
│   ├── configmap.yaml                 # ConfigMap com variáveis de ambiente do backend
│   ├── Dockerfile                     # Dockerfile para a imagem do backend
│   └── app.py                         # Código fonte do backend
├── database/
│   ├── statefulset.yaml               # StatefulSet + Service do PostgreSQL
│   ├── pvc.yaml                       # PVC para volume persistente do PostgreSQL
│   └── secret.yaml                    # Secret com credenciais do banco
├── ingress/
│   └── ingress.yaml                   # Regras de Ingress para acesso externo
└── namespace.yaml                     # Definição dos Namespaces
🛠️ Pré-requisitos
Docker: Para construir as imagens da aplicação.

kubectl: Ferramenta de linha de comando para interagir com clusters Kubernetes.

Um cluster Kubernetes:

Minikube (recomendado para testes locais)

Kind

Ou um cluster Kubernetes real (GKE, EKS, AKS, etc.)

Nginx Ingress Controller instalado no seu cluster: Se estiver usando Minikube, você pode habilitá-lo com minikube addons enable ingress. Para outros clusters, siga a documentação oficial do Nginx Ingress Controller.

⚙️ Passos para o Deploy Completo
Siga estes passos para fazer o deploy da aplicação no seu cluster Kubernetes.

1. Construir e Publicar Imagens Docker
Primeiro, construa e faça o push das imagens Docker para um registro público (como o Docker Hub).

Para o Backend:
Navegue até o diretório backend:

Bash

cd backend
docker build -t Cluiz/backend-mensagens:latest .
docker push Cluiz/backend-mensagens:latest
cd .. # Voltar para o diretório raiz do projeto
Para o Frontend:
Navegue até o diretório frontend:

Bash

cd frontend
docker build -t Cluiz/frontend-mensagens:latest .
docker push Cluiz/frontend-mensagens:latest
cd .. # Voltar para o diretório raiz do projeto
Atenção: Certifique-se de substituir Cluiz pelo seu próprio nome de usuário do Docker Hub.

2. Aplicar os Arquivos de Configuração no Kubernetes
Execute os comandos kubectl apply na ordem correta para garantir que as dependências sejam satisfeitas. Certifique-se de estar no diretório raiz do projeto (projeto-k8s-deploy).

2.1. Criar Namespaces
Bash

kubectl apply -f namespace.yaml
Verifique se foram criados:

Bash

kubectl get namespaces
2.2. Deploy do Banco de Dados (PostgreSQL)
Navegue até o diretório database e aplique os arquivos:

Bash

cd database
kubectl apply -f secret.yaml
kubectl apply -f pvc.yaml
kubectl apply -f statefulset.yaml
cd .. # Voltar para o diretório raiz do projeto
Verifique os recursos do banco de dados:

Bash

kubectl get all -n database
kubectl logs -n database -f <nome-do-pod-postgres> # Para verificar logs do PG
Aguarde até que o pod do PostgreSQL esteja Running.

2.3. Deploy do Backend (Flask API)
Navegue até o diretório backend e aplique os arquivos:

Bash

cd backend
kubectl apply -f configmap.yaml
kubectl apply -f deployment.yaml
cd .. # Voltar para o diretório raiz do projeto
Verifique os recursos do backend:

Bash

kubectl get all -n app -l app=backend
kubectl logs -n app -f <nome-do-pod-backend> # Para verificar logs do backend
Aguarde até que os pods do backend estejam Running.

2.4. Deploy do Frontend (React App)
Navegue até o diretório frontend e aplique o arquivo:

Bash

cd frontend
kubectl apply -f deployment.yaml
cd .. # Voltar para o diretório raiz do projeto
Verifique os recursos do frontend:

Bash

kubectl get all -n app -l app=frontend
kubectl logs -n app -f <nome-do-pod-frontend> # Para verificar logs do frontend
Aguarde até que os pods do frontend estejam Running.

2.5. Configurar Ingress para Acesso Externo
Navegue até o diretório ingress e aplique o arquivo:

Bash

cd ingress
kubectl apply -f ingress.yaml
cd .. # Voltar para o diretório raiz do projeto
Verifique o status do Ingress:

Bash

kubectl get ingress -n app
3. Acesso à Aplicação
Para acessar a aplicação, você precisará do IP do seu Ingress Controller.

Se estiver usando Minikube:
Obtenha o IP do Ingress Controller:

Bash

minikube ip
O endereço de acesso esperado será: http://<IP_DO_MINIKUBE>/
E para a API (uso interno pelo frontend, mas também acessível para testes): http://<IP_DO_MINIKUBE>/api/

Se estiver usando Kind ou outro cluster:
O IP pode ser o EXTERNAL-IP do seu serviço Nginx Ingress Controller (se houver, pode ser um LoadBalancer ou NodePort). Caso contrário, você pode precisar configurar um /etc/hosts para mapear o host: 192.168.49.2 (do seu Ingress) para o IP do Node onde o Ingress Controller está rodando, ou para o IP do LoadBalancer se ele for provisionado.

Exemplo de acesso:
Abra seu navegador e acesse: http://<IP_DO_SEU_CLUSTER> (substitua pelo IP real obtido).

Você deverá ver a interface do frontend, e ao interagir com ela, as mensagens serão persistidas no banco de dados via backend.
