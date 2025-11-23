# Kubernetes + GitOps Demo

Acesta este un proiect demo care ilustrează cum să rulezi o aplicație containerizată într-un cluster Kubernetes și să folosești ArgoCD pentru GitOps. Proiectul include un microserviciu Flask simplu (`demo-app`) și configurări Kubernetes (Deployment, Service).

---

## Prerequisites

- [Docker](https://www.docker.com/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Minikube](https://minikube.sigs.k8s.io/docs/)
- [Helm](https://helm.sh/)
- [ArgoCD CLI](https://argo-cd.readthedocs.io/en/stable/cli_installation/)
- Python 3 (pentru aplicația Flask)
- Opțional: un registry Docker pentru push-ul imaginii (Docker Hub, GHCR)

---

## 1. Clonare repo

```bash
git clone https://github.com/ToaderMadalina/k8s-gitops-cicd-demo.git
cd k8s-gitops-cicd-demo
2. Construirea și rularea aplicației local (opțional)
bash
Copy code
cd app
# Construiește imaginea Docker
docker build -t toaderms/demo-app:1.0 .

# Rulează containerul local
docker run -p 5000:5000 toaderms/demo-app:1.0
Testare:

bash
Copy code
curl http://localhost:5000
# Răspuns: "Kubernetes + GitOps demo: app running!"
3. Push imagine în registry Docker
bash
Copy code
docker login
docker push toaderms/demo-app:1.0
Kubernetes are nevoie de imagine accesibilă dintr-un registry. Imaginea locală nu poate fi folosită direct în Pod-uri.

4. Porniți Minikube
bash
Copy code
minikube start
Verifică că Minikube este activ:

bash
Copy code
minikube status
5. Aplică configurațiile Kubernetes
bash
Copy code
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
Verifică Pod-urile și serviciile:

bash
Copy code
kubectl get pods
kubectl get svc demo-app-service
Dacă folosești NodePort, poți accesa aplicația:

bash
Copy code
minikube service demo-app-service
6. Instalare ArgoCD (dacă nu e deja instalat)
bash
Copy code
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
Verifică starea Pod-urilor ArgoCD:

bash
Copy code
kubectl get pods -n argocd
Expune serverul ArgoCD:

bash
Copy code
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
kubectl get svc -n argocd argocd-server
7. Instalare ArgoCD CLI
bash
Copy code
# Linux
curl -sSL -o argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod +x argocd
sudo mv argocd /usr/local/bin/
Verifică:

bash
Copy code
argocd version
8. Login în ArgoCD
bash
Copy code
argocd login <IP_NODE>:<NODEPORT> --username admin --password <initial_password> --insecure
initial_password se obține din Pod-ul ArgoCD:

bash
Copy code
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
9. Crearea aplicației în ArgoCD
bash
Copy code
argocd app create demo-app \
  --repo https://github.com/ToaderMadalina/k8s-gitops-cicd-demo.git \
  --path k8s \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --sync-policy automated
Sincronizare:

bash
Copy code
argocd app sync demo-app
Verifică aplicația:

bash
Copy code
argocd app get demo-app
10. Testare aplicație
După ce aplicația este sincronizată:

bash
Copy code
curl http://<NODE_IP>:<NODEPORT>
# Răspuns: "Kubernetes + GitOps demo: app running!"
Notes
Dacă apare ImagePullBackOff, asigură-te că imaginea Docker este push-uită și accesibilă în registry.

Porturile NodePort trebuie să fie disponibile (nu ocupate de alte servicii).

Pentru demo local, minikube service <service> este cel mai simplu mod de a accesa aplicația.

Structura proiectului
arduino
Copy code
k8s-gitops-cicd-demo/
├── app/                  # codul aplicației Flask + Dockerfile
├── k8s/                  # fișiere YAML: deployment.yaml, service.yaml
├── README.md
└── .gitignore
Comenzi utile
bash
Copy code
# Verifică Pod-urile
kubectl get pods -w

# Verifică serviciile
kubectl get svc -o wide

# Deschide serviciul în browser
minikube service <service-name>

# Șterge resursele
kubectl delete -f k8s/deployment.yaml
kubectl delete -f k8s/service.yaml
