## Exámen-SRI

1. Explica métodos para 'abrir' una consola/shell a un contenedor que se está ejecutando

Para un contenedor que se está ejecutando en VisualCode tenemos varias opciones la primera y mas fácil es click derecho en el contenedor y le damos a la opción attach shell. Ahora ya pasando a comandos si el contenedor ya se está ejecutando en la consola de el equipo principal hacemos un:
```

docker container exec -it [nombre del contenedor] [interprete que vayamos a utilizar]
```

2. En el contenedor anterior con que opciones tiene que haber sido arrancado para poder interactuar con las entradas y salidas del contenedor


3. ¿Cómo sería un fichero docker-compose para que dos contenedores se comuniquen entre si en una red solo de ellos?
```

services:
  asir_bind9:
    container_name: asir_bind9
    image: ubuntu/bind9
    # Este seria el primer contenedor
    platform: linux/amd64
    ports:
      - 53:53
      #Mapeo de puertos
    networks:
    #Creamos la red con la que se van a conectar ambos y le damos ip fija al DNS
      bind9_subnet:
        ipv4_address: 172.28.5.1
    volumes:
    #Mapeado de volumenes para asi poder cambiar algunos archivos de configuración
      - ./config:/etc/bind
      - ./lib:/var/lib/bind
      - ./logs:/var/log/bind
  cliente:
    container_name: asir_cliente
    image: ubuntu
    platform: linux/amd64
    #Aqui se crea el segundo contenedor en este caso un equipo normal
    tty: true
    stdin_open: true
    networks:
      - bind9_subnet
      #Estas opciones se añaden para que una vez creado el contenedor la red principal a la que se conecte sea la creada anteriormente
networks:
  bind9_subnet:
    external: true
    #Indicamos la red a utilizar
```


4. ¿Qué hay que añadir al fichero anterior para que un contenedor tenga la IP fija?

Pues coma ya lo añadí en el fichero anterior no hace falta repetirlo pero en el caso en el que no esté puesto sería la línea de: ipv4_address: 172.28.5.1

5. ¿Que comando de consola puedo usar para saber las ips de los contenedores anteriores? Filtra todo lo que puedas la salida.



6. ¿Cual es la funcionalidad del apartado "ports" en docker compose?

El apartado ports sirve para mapear el puerto que quieras del contenedor con el de tu equipo siempre que no esté ya ocupado con otro servicio.

7. ¿Para que sirve el registro CNAME? Pon un ejemplo

El registro CNAME es un tipo de registro DNS que asigna un alias a un nombre de dominio. 

Ejemplo:
www.examen.com puede ser mapeado a www1.examensri.com y www2.examensri.com para que todos resuelvan a la misma dirección IP que en este caso seria la primera.

8. ¿Como puedo hacer para que la configuración de un contenedor DNS no se borre si creo otro contenedor?
