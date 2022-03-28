ICCI
'I'mage is a 'C'ontainer as 'C'lass is a 'I'nstance --> ICCI

Referencia
https://docs.docker.com/engine/reference/builder/

Construir la imagen
docker build -t "alex-hello" .

run exponiendo el purto 8000
docker run --rm -it -p 8000:80 alex-hello 


run exponiendo el purto ??
docker run --rm -it -p 80 alex-hello

ejecutar con modo de fallos
docker run --rm -it -p 80 --restart=unless-stopped alex-hello

limitar cpu & Memoria
--memory limita la mem
--cpu cuantas cpu puedes utilizar n de cores. max
--cpu-shares cpus a compartir cuando cpus estan a tope. es un peso.
docker run --rm -it -p 80 --memory=500m alex-hello


persistiendo datos. fuera del container.
el flag es el -v
 docker run -it --name vol-test -v /data ubuntu

 al hacer ls -lha para listar veremos dentro de ubunto data.

 inspeccionamos para ver donde hemos guardado /data en nuestro host local
 docker inspect -f "{{json .Mounts}}" vol-test

 [
   {
      "Type":"volume",
      "Name":"f6edb4e2cdf7f2c1fe8311b23dd6757eee26acad12d99b76643f5729172bbfa4",
      "Source":"/var/lib/docker/volumes/f6edb4e2cdf7f2c1fe8311b23dd6757eee26acad12d99b76643f5729172bbfa4/_data",
      "Destination":"/data",
      "Driver":"local",
      "Mode":"",
      "RW":true,
      "Propagation":""
   }
]

especificar la ruta donde crear el volumen en mi ordenador local
 docker run -it --name vol-test -v $PWD:/data ubuntu

Montar en el directorio actual/DATA un volumen para guardar los datos.
docker run -it --name vol-test3 -v $PWD/data:/data ubuntu 

# subir la imagen
- construir
   docker build -t "dindev/alex-hello:2" .
- taggear - Optional. renombrar sin buildear.
   docker tag alex-hello:3 dindev/alex-hello
- publicar
   docker push dindev/alex-hello:3

Salir 
exit

Run directo en apache compartiendo datos.

docker run --rm -it -p 8000:80 -v $PWD/data:/var/www/html php:7.2-apache 

# KUBERNETES

## POD
la unidad mínima que puedo desplegar en un cluster.
Un pod puede contener mas de un container.
Todos los containers desplegados dentro de un mismo pod comparten la misma IP.


Para ejecutar con powershell

commandos

- kubectl version

   nos da la version de kubernetes y del cluster. que deberían de coincidir.
- minikube start

   para arrancar nuestro cluster virtual. Tenemos que ejecutarlo en una shell para ir mandando comandos con kubectl 
- minikube status
   nos da informacion del cluster

- minikube ip
   nos da la ip del clustser.

- kubectl get nodes
   nos dice cuantos nodos tenemos en el cluster.
- kubectl cluster-info

   summary del estado del cluster.

- crear un pod de nuestra aplicación alex-hello

kubectl run alex-hello --image=dindev/alex-hello:latest --port=80

se baja de nuestro docker hub la app para ponerla a funcionar.

- kubectl get pods
nos da una lista de pods creados
- port forward

 kubectl port-forward pod/alex-hello 8000:80

para decirle que nuestro puerto 8000 se vaya al 80 del cluster.

trouble shotting
- kubectl describe pod/alex-hello
si no podemos comp

- eval $(minikube docker-env -u)

Para poder utilizar los comandos de Docker y que funcionen utilizando minikube como host para Docker, tenemos que lanzar esto y los comandos de Docker empezarán a funcionar

## Comandos generadores.

Son comandos que nos generar el ym de un pod. 
Que es un pod ela especificación de nuestro conjunto de contarines que forman nuestra aplicación.

ejecuta en nuestro cluster este container
kubectl run hello-world --image=codely-docker:latest --port=80
para poder acceder nos faltaría un port forwarding de nuestro pc local al cluster.

kubectl run hello-world --image=codely-docker:latest --port=80 --dry-run -o yaml > pod-file-name.yaml

 --dry-run 
 esto indica a kubctl que no queremos mandarlo a la api de kubernets. solo generar el pod yaml

-yaml 
esto nos genera el pod por consola.

- kubectl delete pods/alex-hello
- kubectl delete pods alex-hello
   nos borrar un pod.
   listar antes para ver el nombre exacto. 
- kubectl create -f pod.yaml

crea un pod a partir de un archivo.

- minikube dashboard
nos da una interfaz gráfica para nuestros pods dentro de minikube


## Generar un service

se generar con kubectl expose.

 directamente a la API
 - kubectl expose pod/alex-hello --port 80 --type=NodePort
 a ficheo
 - kubectl expose pod/alex-hello --port 80 --dry-run=client -o yaml > service-alex-hello.yaml

los exposes en windows no funcionan muy bien. Hay que decirle a minikube quelos abra.

- minikube service alex-hello

esto nos abrirá la URL directamente y genera un tunnel que redirije el tráfico.

## cosas
 PAra lanzar containers dentro del cluster es necesario en cada shell ejecutar esto;
   eval $(minikube docker-env)

   pero en windows 10 no va.
   el equivalente es ejecutar:
   - minikube docker-env
   - minikube docker-env | Invoke-Expression
   luego ya poodemos meter el alpine por ejemplo y attacharnos a su shell:
   bajamos la alpine dentro de miniube. luego de ejecutar los comandos anteriores.
   - docker run alpine
   ahora ya la tenemos la imagen dentro del cluster i la podemos ejecutar.

   - kubectl run --rm -i --tty my-client-app --image=alpine --restart=Never -- sh
  para generar en nuestro cluster un node de alpine i conectar a su shell y así usarlo de client.

   dentro de la shell:
   
   wget alex-hello/index.php -qSO-

   nos saluda.

 ### cosas

   Ejecutar algo dentro del cluster
   - kubectl exec -t alex-hello-- /bin/bash
   dentro de un shell de linus ver las variables de entornoe definidas.
   - env
 ## Deployments

 para crer deployments que kubernetes mantendrá mas allá del ciclo de vida de los nodos.

 Generator para generar un deployment:

Al indicair --restart=always se creará um kind:Deployment

 -  kubectl create deployment alex-hello --image=dindev/alex-hello:latest --port=80 --dry-run=client -o yaml > deploy-alex-hello.yaml

 si un nodo cae y contiene un de los nodos indicados en el deployment. se crea uno nuevo.

 El número de réplicas indica el número de pods que se mantienen.

 ## habilitar ingress.
 ingres es el que reconciliador que nos gestionará el traffico que llega al cluster.
 hay varias implementacione. se va a utilizar aquí la de nginx.


habilitar el addon en minikube

para listar los addons.
- minikube addons list
habilitar.
- minikube addons enable ingress


generador para el ingress
- kubectl create ingress alex-hello --default-backend alex-hello:80 --dry-run=client -o yaml > ingress.yaml


 ## config maps & secrets

 sirven para pasar configuración variable y sensible a k8s.
 No hardcodear valores sensibles de "perfil" o "configuracin".

se pueden pasar con utilizando variables de entorno.

 separa definición de configruarción.

### Config map.
 Parejas clave valor.

 Generador
 - kubectl create configmap special-config --from-literal=example.property1=hello --from-literal=example.property2=world

 ver el datalle 
 - kubectl get configmaps special-config -o yaml

 - kubectl exec -t alex-hello-- /bin/bash
   dentro de un shell de linus ver las variables de entornoe definidas.
 - env


### KUBECTL con diferentes contextos.
  para saber los diferentes cluster que tenemos o a los que podemos mandar órdenes desde kubectl:
   - kubectl config view
  para cambiar de contextos:
   Primero listar los cotextos
   - kubectl config get-contexts
   luego cambiar de contexto
   - kubectl config use-context <context>   
   - ejemplo: 
      ''' kubectl config use-context minikube   '''
  para cambiar el namespace actual sin cambiar el contexto
   - '''  kubectl config set-context --current --namespace=<namespace>  '''



- ver la version del linyx
   cat /etc/os-release
