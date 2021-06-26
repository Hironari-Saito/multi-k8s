## Overview

```
Trafic -> `Ingress Service` -> ClusterIP Service: Deployment(client)
                |
                |                                                 Deployment(worker)
                |                                                 |
                |                                                 ↓
                -> ClusterIP Service: Deployment(server) -> ClusterIP Service: Deployment (Redis)
                           |
                           |
                           -> ClusterIP Service: Deployment(Postgres)  -> Postgres PVC
```

## Path to Production

1. Create config files for each service and deployment
2. Test locally on minikube
3. Create a Github/Travis flow to build images and deploy
4. Deploy app to a cloud provider

## secret の作成

このクラスタを動作させるために secret を登録する必要がある。

```
kubectl create secret generic pgpassword --from-literal PGPASSWORD=XXXXX
```

## nginx ingress controller のインストール

`https://kubernetes.github.io/ingress-nginx/deploy/#provider-specific-steps`から nginx ingress controller をインストールする。

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.47.0/deploy/static/provider/cloud/deploy.yaml
```

## minikube での ingress を有効化

```
minikube addons enable ingress
```

## GCE-GKE

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.47.0/deploy/static/provider/cloud/deploy.yaml
```

## ingress-nginx-controller について

LoadBalancer が完全に作成されるまで EXTERNAL IP が<pending>となる。 EXTERNAL-IP が作成されるのを待つ必要がある。

ただし、Kubernetes の Service type LoadBalancer に対応していないインフラストラクチャ上で type LoadBalancer な Service を作成するとずっと<pending>のままとなる。

```
kubectl get svc -n ingress-nginx
```

## minikube known issues

https://minikube.sigs.k8s.io/docs/drivers/docker/#known-issues

```
minikube delete
```

```
minikube start --driver=hyperkit

or

minikube start --driver=virtualbox
```

## Production deployment

1. create github repo
2. tie repo to Travis CI
3. create Google Cloud project
4. Enabling billing for the project
5. add deployment scripts to the repo

## travis に GCE のアクセス用の Json ファイルを配置するために暗号化する。

```
brew install travis
travis login --github-token $GITHUB_TOKEN_TRAVIS
travis encrypt-file service-account.json -r Hironari-Saito/multi-k8s
## 生成されたファイルをコミットし、元のファイルは削除する。.travis.ymlに生成されたコマンドを記載する。
```
