# Day 1

## Instalação
- [Docker](https://docs.docker.com/engine/install)
- [Kubernetes](https://kubernetes.io/docs/tasks/tools/)
- [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)

## Experimentos

### 1. Cluster Simples (Hello World)
Criar um cluster com Kind: `kind create cluster`

```shell
# Saída esperada
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.34.0) 🖼
 ✓ Preparing nodes 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Have a question, bug, or feature request? Let us know! https://kind.sigs.k8s.io/#community 🙂
```

Verificar o nodes, pods, deployments, services e replicaset

```shell
kubectl get nodes

kubectl get pods -A

kubectl get deployments -A

kubectl get services -A

kubectl get replicaset -A
```
Obs: flag `-A` significa que vai fazer o get em todos os namespaces. Se quiser especificar o namespace utilize a flag `-n <namespace>`, exemplo: `kubectl get pods -n kube-system`

Obs2: A flag com parâmetro `-o wide` pode ser usada nos comandos acima para trazer mais informações. Exemplo: `kubectl get pods -n kube-system -o wide`

Deletar o cluster: `kind delete cluster`


### 2. Criar um cluster a partir de um arquivo de configuração.

Criar o arquivo [kind-cluster.yaml](kind/kind-cluster.yaml).

Neste arquivo dá para configura a quantidade de **control-planes** e **workers** que devem existir no cluster.

Depois de criar o arquivo de configuração executar o comando: `kind create cluster --config kind-cluster.yaml --name giropops`

Agora verifique os nodes: `kubectl get nodes`

```shell
# Saída esperada
NAME                     STATUS   ROLES           AGE   VERSION
giropops-control-plane   Ready    control-plane   29s   v1.34.0
giropops-worker          Ready    <none>          15s   v1.34.0
giropops-worker2         Ready    <none>          15s   v1.34.0
```

### 3. Criando pods pelo terminal

Exemplo de criação de pod com nginx: `kubectl run --image nginx --port 80 giropops`

```shell
# Verifique o pod com kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
giropops   1/1     Running   0          15s
```

Expose do pod girpops com tipo NodePort: `kubectl expose pods giropops --type NodePort`

```shell
# Verifique o pod com kubectl get services -A
NAMESPACE     NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
default       giropops     NodePort    10.96.228.107   <none>        80:30124/TCP             3s
default       kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP                  10m
kube-system   kube-dns     ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP,9153/TCP   10m
```

Criar um cluster com kind e dando um nome ao cluster

Deletar pods: `kubectl delete pods giropops`

### 4. Criar pods a partir de um arquivo de configuração

Criar arquivo yaml a partir do terminal: `kubectl run --image nginx --port 80 giropops --dry-run=client -o yaml`

```yaml
# Saída esperada
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: giropops
  name: giropops
spec:
  containers:
  - image: nginx
    name: giropops
    ports:
    - containerPort: 80
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

Salve esse texto yaml em um arquivo de mesmo tipo e execute o comando: `kubectl apply -f pod.yaml`

Verifique o pod, deployment, service, replicaset com o `kubectl get`

Finalize deletando o cluster: `kind delete cluster --name giropops`

## Dicas

Crie um alias no terminal para o kubectl: `alias k=kubectl`. Agora em todos os comandos com o kubectl digite apenas k + comando. Exem: `k get nodes` 