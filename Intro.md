# Intro Kubernetes
#####

Para empezar a trabajar con Kubernetes, vamos a plantear la siguiente arquitectura:

1 Master

2 Nodes - Workers

Vamos a trabajar con Minikube o tambien en Play With Kubernetes:

[Play With K8s](https://labs.play-with-k8s.com)

### OBS.
Para trabajar con kubernetes, debemos contar con un contenedor como Docker, RKT, ContainerD, Cri-O


## Diagrama K8s Base

![Arquitectura](https://github.com/kdetony/k8s-basico/blob/master/images/k8s-base.jpg)

## ESTRUCTURA DE KUBERNETES

Para entender el funcionamiento de Kubernetes lo mejor es repasar los conceptos y verlo de una forma gráfica:

### Pod:
es una abstracción de un grupo de contenedores, agrupados en torno a elementos comunes como almacenamientos compartidos, una misma IP  e información compartida como qué puertos usar.
    
### Nodo: 
Los nodos son las máquinas en las que se ejecutan los contenedores.

### Master: 
Es el coordinador del cluster, se encarga de escalar los contenedores, mantener las aplicaciones en el estado deseado, hacer actualizaciones de los despliegues.


## Trabajando con K8S


### En el Nodo Master:

1) Inicializamos el Rol Master
>kubeadm init --apiserver-advertise-address $(hostname -i)

2) Descargamos un CNI, en este caso: Weave
>kubectl apply -n kube-system -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

3) Ahora los pasos finales
>mkdir -p $HOME/.kube
>cp -i /etc/kubernetes/admin.conf $HOME/.kube/config 

>kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/master/content/en/examples/application/nginx-app.yaml


4) Usaremos este TOKEN para añadir los nodos al cluster:
>kubeadm join IP_MASTER:6443 --token j4bv4x.5ipdxeifs99jyh1p --discovery-token-ca-cert-hash sha256:f232f016816d999ce48597b367d4585bade1749dc9c2462e4fc48200ceb58be6
