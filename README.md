# elasticsearch-assignment

copy shell.nix to your working directory
nix-shell
kind create cluster --name=elastic-linkerd --config=kind-config.yaml

# check if kind cluster created
kubectl get no

# install argocd
kubectl create ns argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# get argocd password
argocd admin initial-password -n argocd
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d


# port forward and login through cli
kubectl port-forward -n argocd service/argocd-server 8443:443 &
argocd login localhost:8443

# add repo git
token=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
argocd repo add https://github.com/AdamMilen/elasticsearch-assignment.git --username AdamMilen --password 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

# deploying system application and user applications using app of apps pattern
1. kubectl apply -f ./argocd/root-system-apps.yaml
2. kubectl apply -f ./argocd/root-user-apps.yaml


# changing elastic built-in user password
kubectl exec -c elasticsearch -it elastic-linkerd-es-default-0 -n elastic-system -- sh
elasticsearch-users passwd elastic -p adminadmin

# Linkerd installation
linkerd check --pre <br />
linkerd install --crds | kubectl apply -f - <br />
linkerd install | kubectl apply -f - /n <br />

# edit /etc/hosts
127.0.0.1 nginx.elasticsearch.local

## Manual way ##


# install nginx ingress controller
kubectl apply -f https://kind.sigs.k8s.io/examples/ingress/deploy-ingress-nginx.yaml


# Install ECK operator
kubectl create -f https://download.elastic.co/downloads/eck/2.16.1/crds.yaml
linkerd inject https://download.elastic.co/downloads/eck/2.16.1/operator.yaml | kubectl apply -f -

kubectl apply firerealm-secret.yaml
kubectl apply elasticsearch.yaml

# adding user devops
kubectl exec -c elasticsearch -it elastic-linkerd-es-default-0 -n elastic-system -- sh
elasticsearch-users passwd elastic -p adminadmin

kubectl port-forward service/quickstart-es-http 9200

# Install Prometheus Operator (grafana & Alertmanager included)
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install kind-prometheus --namespace monitoring --create-namespace prometheus-community/kube-prometheus-stack -f values-prom-operator.yaml

# Installing prometheus exporter and setting the correct ClusterIP and without cert check
helm install elasticsearch-exporter prometheus-community/prometheus-elasticsearch-exporter -f values-elasticsearch-exporter.yaml


# Connecting exporter to Prometheus: configuring servicemonitor
kubectl apply -f servicemonitor.yaml


# port-forward grafana
export POD_NAME=$(kubectl --namespace monitoring get pod -l "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=kind-prometheus" -oname)

kubectl --namespace monitoring port-forward $POD_NAME 3000 &

# port-forward prometheus
kubectl port-forward svc/prometheus-operator-kube-p-prometheus -n monitoring 9090:9090 &

# Alerts for elasticsearch
In order to prevent disks from filling up in the future and act proactively, you should create alerts based on **disk usage that will notify you when the disk starts filling up**.

Think about adding recording rules, recording rules are essential for optimizing the performance of your Prometheus setup. As your environment grows, the number of metrics and complexity of queries increases. Running complex queries in real-time can become slow and resource-intensive.

# setting up alerts to slack
https://medium.com/@joudwawad/comprehensive-beginners-guide-to-kube-prometheus-in-kubernetes-monitoring-alerts-integration-4ade4fa8fa8c

https://hooks.slack.com/services/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