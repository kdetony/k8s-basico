# Introduccion a kubernetes

## Comandos a usar

* Listart todos los componentes para kubernetes
  * kubectl get all 

* Ejecutar un archivo yaml 
  * kubectl apply -f file.yml
  * kubectl apply -f /DIR/

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
los pods son DESCARTABLES ojo! no es la funcion de un pod ser ETERNO ... 

Para crear nuestro primer namespace: 
kubectl apply -f 00-config-namespaces.yml 

Esto nos creara un "espacio de trabajo" llamado TEST, y con esto podemos crear labels que hagan referencia a este namespace para que todo los deployments y/o cambios
solo afecten a los pods relacionados a este Namespace. 

OBSERVACION:
* Una Namespace es un "Sub-Cluster"  
  * Si queremos crear varios namespaces en un solo fichero: 
  * kubectl apply -f 00-config-namespaces-all.yml

