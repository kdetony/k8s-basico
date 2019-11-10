# Intro Kubernetes
#######

Para empezar a trabajar con Kubernetes, vamos a plantear la siguiente arquitectura:

1 Master

2 Nodes - Workers

### OBS.
Para trabajar con kubernetes, debemos contar con un contenedor como Docker, RKT, ContainerD, Cri-O

En el Nodo Master:

1) Inicializamos el Rol Master
>kubeadm init --apiserver-advertise-address $(hostname -i)

2) Descargamos un CNI, en este caso: Weave
>kubectl apply -n kube-system -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

3) Ahora los pasos finales
>mkdir -p $HOME/.kube
>cp -i /etc/kubernetes/admin.conf $HOME/.kube/config 

>kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/master/content/en/examples/application/nginx-app.yaml


4) Usaremos este TOKEN para añadir los nodos al cluster:
>kubeadm join 192.168.0.28:6443 --token j4bv4x.5ipdxeifs99jyh1p --discovery-token-ca-cert-hash sha256:f232f016816d999ce48597b367d4585bade1749dc9c2462e4fc48200ceb58be6

## Chuletas para K8S:

Para listar los nodos que son parte del cluster:
>kubectl get Nodes

Para obtener un mayor detalle, por ejm. ver la version del runtime del contenedor:
>kubectl get nodes -o wide
