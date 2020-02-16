Terminología
#######

Vamos a trabajar en la siguiente ruta: **/opt/kubernetes**, para crear la carpeta ejecutamos:

> mkir -p /opt/kubernetes

## POD
Unidad mínima de k8s

Creamos la carpeta **pod** en */opt/kubernetes*

> pod.yml

``` 
apiVersion: v1
kind: Pod
metadata:
  name: dbmongo
spec:
  containers:
  - image: mongo
    name: dbmongo
```

ahora, para crear el pod via yaml, ejecutamos:

> kubectl apply -f pods.yml  

Para listar los pods creados: 

> kubectl get pods

Para ver información del pod:

> kubectl describe pod dbmongo   

tambien:  

> kubectl get all 
> kubectl describe pod/dbmongo

Podemos crear un POD de la siguiente manera:

> kubectl run webnginx --image=nginx --restart=Never

y listamos los pods:

> kubectl get po y/o  pods

Si nos olvidamos del manifiesto que creo un pod, podemos recurrir a:

> kubectl get pod webnginx -o yaml  

y/o  
> kubectl get pod/webnginx -o yaml

para acceder el POD:
> kubectl exec -it pod/webnginx bash  

para poder ver el estado de nginx: 
> curl $IP 

Si queremos exponer el puerto del pod para que pueda ser accesible desde el HOST, esto se conoce como **NodePort**, para lo cual podemos realizar:

> kubectl port-forward webnginx 80

Para validarlo, desde el HOST:  

> curl 127.0.0.1

Ahora vamos a mapear un puerto especifico, 8080: 

> kubectl port-forward webnginx 80:8080

Para validarlo, desde el HOST:

> curl 127.0.0.1:8080


## LABELS
Sirve para cualquier objeto de k8s, añade una clave y valor a un objeto, con el objetivo de identificar pods ()

Creamos la carpeta **label** en */opt/kubernetes*

> label.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: webnginx-label 
  labels:
      name: webnginx-label 
      pod-template: "mypodweb"
      run: nginx

spec:
  containers:
  - image: nginx
    name:  webnginx-label 
```

Para ejecutarlo via el archivo yaml: 

> kubectl apply -f label.yml


Podemos crear tambien labels de forma directa, para ello bastaría con ejecuta: 

> kubectl label pods nginx foo=bar 

**Desventaja:**  si se borra, se pierde el label ojo !!! 

Para listar las etiquetas/labels:

> kubectl get pods --show-labels 

Si queremos buscar LABELs y mostrarlos en nuevas columnas:

> kubectl get pods -l foo=bar
> kubectl get pods -Lrun

## REPLICA-CONTROLLER (replicaset)
Nos permite tener tantas *"replicas"* de un POD que necesitemos ejecutar.

> rc.yaml
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: wnginx
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: wnginx
  template:
    metadata:
      labels:
        app: wnginx
    spec:
      containers:
        - image:  nginx
          name:  nginx
```

para aplicarlo:
> kubectl apply -f rc.yml 

ojo también se puede usar:
> kubectl create -f rs.yml

de esta forma, no se puede editar directamente el manifest, y a parte que no es muy usado.

Si deseamos escalar los pods, por un tema de carga 

> kubectl scale rc rs-nginx --replicas=5

y para listar los pods en base a los replicaset:

> kubectl get pods --watch 

## DEPLOYMENT
Lo que nos permite un deployment:
- replica management
- pod scaling
- rolling updates - 0 downtime
- rollback 
- clean-up policies

> deploy.yml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dp-nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: dp-nginx
  template:
    metadata:
      labels:
        app: dp-nginx
    spec:
      containers:
      - name:  dpnginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

lo aplicamos:

> kubectl apply -f deploy.yml

si queremos escalar el deployment:

> kubectl scale deploy dp-nginx --replicas=4

Si queremos listar los deployments:

> kubectl get deployments

Ahora, como se mencionó podemos hacer updates en “caliente” de la sgt forma:

> kubectl set image deployment.apps/deploy-nginx webnginxprd=nginx:1.16.1 --all

## OBS:
```
deployment.apps/dp-nginx : nombre del deployment
dpnginx :                  nombre del contenedor
```

## Actividad 

Vamos a usar un ejemplo:
> kubectl run dokuwiki --image=dokuwiki --record

vemos el manifiesto creado:
> kubectl get deployments dokuwiki -o yaml

y listamos el estado del pod:
> kubectl get pods

si hacemos el update:
> kubectl set image deployment/dokuwiki dokuwiki=bitnami/dokuwiki:09 --all

Listamos el estado de los pod “actualizados”
``` 
kubectl get pods
dokuwiki-fcbbf5844-rrmck        0/1     ErrImagePull   0          5s
```

vemos que hay un error en la imagen, entonces recurrimos a un rollback

para ellos vamos a listar los históricos:
> kubectl rollout history deployment/dokuwiki

y procedemos a realizar el rollback
> kubectl rollout undo deployment/dokuwiki

listamos ahora los pods
> kubectl get pods

Vamos a repasar los términos, para ello vamos a trabajar con la imagen de bitnami/nginx

> kubectl run webnginx --image=bitnami/nginx:1.12 --record

listamos los pods:
> kubectl get pods

y listamos los deployments:
> kubectl get deploy

para listar/editar el deployment:
> kubectl edit deploy/webnginx

si deseo listar los pods con labels
> kubectl get pod --show-labels

si deseo solo listar el pod en ejecución:
> kubectl get pods -w -l run=webnginx

escalemos en 8 replicas:
> kubectl scale deploy/webnginx --replicas=8

*mirar la otra consola*

y miramos los deployments:
> kubectl get deploy

si borramos un pod, vemos que se vuelve a crear, esto debido al replicaset que ha sido creado vía el deployment

listo el historico :
> kubectl rollout history deploy/webnginx

actualizamos la imagen:
> kubectl set image deploy/webnginx webnginx=bitnami/nginx:1.14--all

se presenta un error como se presenta, entonces hacemos el rollback

> kubectl get rs -l run=webnginx

antes de hacer el rollback vamos a dejar corriendo el estado del replicaset:

> kubectl get rs -w -l run=webnginx
```
kubectl rollout undo deploy/webnginx
deployment.extensions/webnginx rolled back
```

si borramos el replicaset:
>kubectl delete replicaset.apps/webnginx-5fdb9bd66b

este se vuelve a regenerar debido a que el deployment se mantiene activo.

OBS.

para que siempre guarde las actividades, hay que indicarlo:
> kubectl run nginx --image=bitnami/nginx:1.12 --record

para ver los logs de un pod en base a su label:
> kubectl logs -f -l run=webnginx
