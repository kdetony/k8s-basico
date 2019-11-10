# Terminología
####

## POD
unidad mínima de k8s

>pod.yml
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

ahora: 
>kubectl apply -f pods.yml  

Para listar los pods creados: 
>kubectl get pods

para ver información del pod:
>kubectl describe pod dbmongo   

tambien:  
>kubectl get all && kubectl describe pod/dbmongo

Podemos crear un POD de la siguiente manera:
>kubectl run webnginx --image=nginx --restart=Never

y listamos los pods:
>kubectl get po y/o  pods

Si nos olvidamos del manifiesto que creo un pod, podemos recurrir a:
>kubectl get pod webnginx -o yaml  

y/o  
>kubectl get pod/webnginx -o yaml

para acceder el POD:
>kubectl exec -it pod/webnginx bash  

para poder ver el estado de nginx: 
>curl $IP 

si queremos exponer el puerto, esto se conocer como NodePort, para lo cual podemos realizar: 

( acceso desde la ip del NODO/MASTER ojo!  no es para exponer al mundo )
>kubectl port-forward webnginx 80

Para validarlo, desde un nodo:  
>curl 127.0.0.1


>kubectl port-forward webnginx 80:8080

Para validarlo, desde un nodo: 
>curl 127.0.0.1:8080


## LABELS
sirve para cualquier objeto de k8s, añade una clave y valor a un objeto, con el objetivo de identificar pods ()
```
ejm:
apiVersion: v1
kind: Pod
metada:
………..
    labels:
       pod-template: “mypod”
       run: nginx 
```

Para ejecutarlo de forma manual
>kubectl label pods nginx foo=bar 

desventaja:  si se borra, se pierde el label ojo !!! 

Para listar las etiquetas:
>kubectl get pods --show-labels 

Si queremos buscar LABELs y mostrarlos en nuevas columnas:
>kubectl get pods -l foo=bar
>kubectl get pods -Lrun

## REPLICA-CONTROLLER (replicaset)
nos permite tener tantas “replicas” de un POD que necesitemos ejecutar.

ejm.
```
apiVersion: extensions/v1beta1
kind: ReplicaSet
metadata:
   name: rs-nginx
   namespace: default
spec:
   replicas: 3
   selector:
        matchLabels:
            app: appnginx  #de esta manera k8s sabe como aplicar las mismas acciones a los mismos POD
    template:
         metadata:
       labels:
          app: appnginx
      spec:
           containers:
image: nginx
name: webnginx 
```

para aplicarlo:
>kubectl apply -f rs.yml 

ojo también se puede usar:
>kubectl create -f rs.yml
de esta forma, no se puede editar directamente el manifesto…. y a parte que no es muy usado.

Si deseamos escalar los pods, por un tema de carga 
>kubectl scale rs rs-nginx --replicas=5

y para listar los pods en base a los replicaset
>kubectl get pods --watch 

## DEPLOYMENT
lo que nos permite un deployment:
- replica management
- pod scaling
- rolling updates - 0 downtime
- rollback 
- clean-up policies

ejemplo:
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: deploy-nginx
  namespace: default
spec:
  revisionHistoryLimit: 5              #cuantos rollbacks podemos hacer
  strategy:
     type: RollingUpdate
  replicas: 3
  template:
     metadata:
       labels:
         app: web-nginx
     spec:
       containers:
       - image: nginx
         name: webnginxprd
         ports:
         - name: http
           containerPort: 8080
```

lo aplicamos
>kubectl apply -f deployment.yml

si queremos escalar el deployment:
>kubectl scale deployment deploy-nginx --replicas=4

Si queremos listar los deployments
>kubectl get deployments

Ahora, como se mencionó podemos hacer updates en “caliente” de la sgt forma:
>kubectl set image deployment.apps/deploy-nginx webnginxprd=nginx:1.16.1 --all

#### OBS:
```
deployment.apps/deploy-nginx : nombre del deployment
webnginxprd : nombre del contenedor
```

Vamos a usar un ejemplo:
>kubectl run dokuwiki --image=dokuwiki --record

vemos el manifiesto creado:
>kubectl get deployments dokuwiki -o yaml

y listamos el estado del pod:
>kubectl get pods

si hacemos el update:
>kubectl set image deployment/dokuwiki dokuwiki=bitnami/dokuwiki:09 --all

Listamos el estado de los pod “actualizados”
``` 
kubectl get pods
dokuwiki-fcbbf5844-rrmck        0/1     ErrImagePull   0          5s
```

vemos que hay un error en la imagen, entonces recurrimos a un rollback

para ellos vamos a listar los históricos:
>kubectl rollout history deployment/dokuwiki

y procedemos a realizar el rollback
>kubectl rollout undo deployment/dokuwiki

listamos ahora los pods
>kubectl get pods

Vamos a repasar los términos, para ello vamos a trabajar con la imagen de bitnami/nginx

>kubectl run webnginx --image=bitnami/nginx:1.12 --record

listamos los pods:
>kubectl get pods

y listamos los deployments:
>kubectl get deploy

para listar/editar el deployment:
>kubectl edit deploy/webnginx

si deseo listar los pods con labels
>kubectl get pod --show-labels

si deseo solo listar el pod en ejecución:
>kubectl get pods -w -l run=webnginx

escalemos en 8 replicas:
>kubectl scale deploy/webnginx --replicas=8

*mirar la otra consola*

y miramos los deployments:
>kubectl get deploy

si borramos un pod, vemos que se vuelve a crear, esto debido al replicaset que ha sido creado vía el deployment

listo el historico :
>kubectl rollout history deploy/webnginx

actualizamos la imagen:
>kubectl set image deploy/webnginx webnginx=bitnami/nginx:1.14--all

se presenta un error como se presenta, entonces hacemos el rollback

>kubectl get rs -l run=webnginx

antes de hacer el rollback vamos a dejar corriendo el estado del replicaset:

>kubectl get rs -w -l run=webnginx
```
kubectl rollout undo deploy/webnginx
deployment.extensions/webnginx rolled back
```

si borramos el replicaset:
>kubectl delete replicaset.apps/webnginx-5fdb9bd66b

este se vuelve a regenerar debido a que el deployment se mantiene activo.

OBS.

para que siempre guarde las actividades, hay que indicarlo:
>kubectl run nginx --image=bitnami/nginx:1.12 --record

para ver los logs de un pod en base a su label:
>kubectl logs -f -l run=webnginx
