# Day 2

## Comandos importantes para pods

- `kubectl get pods`
- `kubectl get pods -o wide`
- `kubectl get pods -w`
- `kubectl run --image nginx girus`
- `kubectl run --image nginx girus --dry-run=client -o yaml`
- `kubectl apply -f pods.yaml`
- `kubectl create -f pods.yaml`
- `kubectl describe pods girus`
- `kubectl logs girus`
- `kubectl logs girus -c nginx`
- `kubectl exec -ti girus -- bash`
- `kubectl delete pods girus`

## Limitando recursos

No manifesto do pod coloque o **resources** na definição do container, ex:

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: giropops
  name: giropops
spec:
  containers:
  - image: ubuntu
    name: ubuntu
    resources:
      limits: # Limite máximo do container
        memory: "128Mi"
        cpu: "0.5"
      requests: # Garante os limites (minimo?) do container 
        memory: "64Mi"
        cpu: "0.3"
    args:
    - sleep
    - "600"
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

## Persistindo dados no container dos pods

Definir volumes ao Container do Pod e também definir o volume que são utilizados nos containers. Ex:

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: giropops
  name: giropops
spec:
  containers:
  - image: nginx
    name: nginx
    volumeMounts:
    - mountPath: /giropops
      name: primeiro-emptydir
    resources:
      limits:
        cpu: "1"
        memory: "128Mi"
      requests:
        cpu: "0.3"
        memory: "64Mi"
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:
  - name: primeiro-emptydir
    emptyDir:
      sizeLimit: 256Mi
status: {}
```

## Exercício

[pod-exercicio.yaml](./pod-exercicio.yaml)

## Desafio

Criando namespace: `kubectl create namespace treinamento-ch2`

Criando template: `kubectl run pod-faminto --image polinux/stress --namespace treinamento-cht2 --command "stress" --dry-run=client -o yaml`

Depois do ajuste: [pod-faminto.yaml](./pod-faminto.yaml)
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: pod-faminto
  name: pod-faminto
  namespace: treinamento-ch2
spec:
  containers:
  - name: stress
    image: polinux/stress
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "250M", "--vm-hang", "1"]
    resources:
      limits:
        memory: "200Mi"
      requests:
        memory: "100Mi"
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

Verificando com `kubectl get pods -n treinamento-ch2 -w`:
```shell
NAME          READY   STATUS    RESTARTS   AGE
pod-faminto   0/1     Pending   0          0s
pod-faminto   0/1     Pending   0          0s
pod-faminto   0/1     ContainerCreating   0          0s
pod-faminto   0/1     OOMKilled           0          7s
pod-faminto   0/1     OOMKilled           1 (3s ago)   9s
pod-faminto   0/1     CrashLoopBackOff    1 (2s ago)   10s
pod-faminto   0/1     OOMKilled           2 (16s ago)   24s
```

Conclusão: Não tem espaço suficiente no container para o exercício de stress.

Para ajustar isso somente aumentar o limite máximo de 200Mi para 300Mi [pod-comportado.yaml](./pod-comportado.yaml)


```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: pod-faminto
  name: pod-faminto
  namespace: treinamento-ch2
spec:
  containers:
  - name: stress
    image: polinux/stress
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "250M", "--vm-hang", "1"]
    resources:
      limits:
        memory: "300Mi"
      requests:
        memory: "100Mi"
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

Verificando com `kubectl get pods -n treinamento-ch2 -w`:
```shell
pod-faminto   0/1     Pending             0             0s
pod-faminto   0/1     Pending             0             0s
pod-faminto   0/1     ContainerCreating   0             0s
pod-faminto   1/1     Running             0             3s
```

Conclusão: Pod executado com sucesso!