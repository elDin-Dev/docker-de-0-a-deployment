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


Salir 
exit

Run directo en apache compartiendo datos.

docker run --rm -it -p 8000:80 -v $PWD/data:/var/www/html php:7.2-apache 

# KUBERNETES

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

 - kubectl expose pod/alex-hello --port 80 --dry-run=none -o yaml > service-alex-hello.yaml



## cosas
 - kubectl run --rm -i --tty my-client-app --image=alpine --restart=Never -- sh
 para generar en nuestro cluster un node de alpine i conectar a su shell y así usarlo de client.