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

