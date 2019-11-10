# Introduccion a kubernetes

## Comandos a usar

* Listar todos los componentes para kubernetes
> kubectl get all 

* Ejecutar un archivo yaml 
> kubectl apply -f file.yml
> kubectl apply -f /DIR/

* Listar Nodos:
  * kubectl get nodes
  * kubectl get nodes -o wide 

* Listar Namespaces:
  * kubectl get ns 

* Listart Servicios:
  * kubectl get svc

## Desplegando componentes para Kubernetes

Para poder empezar a trabajar con kubernetes, debemos de tener en claro toda la infraestructura que vamos a necesitar,
para este primer ejm. vamos a definir 2 ambientes: test y dev para ello vamos a usar los archivos: *00-config-namespaces.yml* y *00-config-namespaces-all.yml*

OBSERVACION:
* Los pods son DESCARTABLES ojo! no es la funcion de un pod ser ETERNO ... 

Para crear nuestro primer namespace: 
**kubectl apply -f 00-config-namespaces.yml**

Esto nos creara un "espacio de trabajo" llamado TEST, y con esto podemos crear labels que hagan referencia a este namespace para que todo los deployments y/o cambios
solo afecten a los pods relacionados a este Namespace. 

OBSERVACION:
* Una Namespace es un "Sub-Cluster"  
  * Si queremos crear varios namespaces en un solo fichero: 
  * kubectl apply -f 00-config-namespaces-all.yml

Con los namespaces creados, para listarlos usamos:
> kubectl get ns 

Ahora, vamos a crear un Pod, para ello vamos a utilizar el archivo 00-config-replica.yml

> kubectl apply -n test -f 00-config-replica.yml

OBSERVACION: 

* si usamos: kubectl apply -f XXX.yml, esto crea el ReplicaController en el namespace **default**
* ReplicaController nos permite crear una cantidad inicial de Pods, ante lo cual, si uno presenta problemas, k8s siempre podrá
crear la cantidad inicial que se le indico.

Con nuestro pod creado, vamos a crear un service, que nos permita acceder a nuestro pod: 

> kubectl apply -n test -f 00-config-service.yml

Con esto, mediante la opcion NodePort estamos exponiendo el puerto 80 del contenedor por el puerto 30000 para el host.
