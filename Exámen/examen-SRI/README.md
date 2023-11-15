# Exámen-SRI

1. **Explica métodos para 'abrir' una consola/shell a un contenedor que se está ejecutando**

Para un contenedor que se está ejecutando en VisualCode tenemos varias opciones la primera y mas fácil es click derecho en el contenedor y le damos a la opción attach shell. Ahora ya pasando a comandos si el contenedor ya se está ejecutando en la consola de el equipo principal hacemos un:
```

docker container exec -it [nombre del contenedor] [interprete que vayamos a utilizar]
```

2. **En el contenedor anterior con que opciones tiene que haber sido arrancado para poder interactuar con las entradas y salidas del contenedor**
Para que podamos interactuar con las entradas y salidas en el docker run hay que poner -it para que puedas interactuar en el contenedor.


3. **¿Cómo sería un fichero docker-compose para que dos contenedores se comuniquen entre si en una red solo de ellos?**
```
services:
  asir_bind9:
    container_name: asir_bind9
    image: internetsystemsconsortium/bind9:9.16
    ports:
      - 53:53
    networks:
      bind9_subnet:
        ipv4_address: 172.28.5.1
    volumes:
      - ./config:/etc/bind
      - ./lib:/var/lib/bind
  asir_cliente:
    container_name: asir_cliente
    image: alpine
    networks:
      - bind9_subnet
    stdin_open: true # docker run -i
    tty: true # docker run -t
    dns:
      - 172.28.5.1 # el contenedor dns server
networks:
  bind9_subnet:
    external: true

```

4. **¿Qué hay que añadir al fichero anterior para que un contenedor tenga la IP fija?**

Pues como ya lo añadí en el fichero anterior no hace falta repetirlo pero en el caso en el que no esté puesto sería la línea de: 
```

ipv4_address: 172.28.5.1
```
5. **¿Que comando de consola puedo usar para saber las ips de los contenedores anteriores? Filtra todo lo que puedas la salida.**

Hay varias opciones pero las que hemos visto han sido una desde dentro del contenedor y en la consola que ponemos el comando `ip a` y se nos muestran las redes conectadas y buscamos la del equipo y vemos la ip.

Luego también se puede desde fuera pero es más complejo como puede ser:
```
docker exec [nombre contenedor] cat /etc/hosts
```
Nos aparecería algo como esto donde se ve la ip del contenedor:
```

127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.18.0.3      607b00c25f29
```
6. **¿Cual es la funcionalidad del apartado "ports" en docker compose?**

El apartado ports sirve para mapear el puerto que quieras del contenedor con el de tu equipo siempre que no esté ya ocupado con otro servicio.

7. **¿Para que sirve el registro CNAME? Pon un ejemplo**

El registro CNAME es un tipo de registro DNS que asigna un alias a un nombre de dominio. 

Ejemplo:

*www.examen.com puede ser mapeado a www1.examensri.com y www2.examensri.com para que todos resuelvan a la misma dirección IP que en este caso seria la primera.*

8. **¿Como puedo hacer para que la configuración de un contenedor DNS no se borre si creo otro contenedor?**
Pues puedes mapear los volumenes fuera del contenedor y asi tenerlos ya guardados si los vas a utilizarlos mas tarde y solo habria que mapearlos otra vez en el compose.
9. **Añade una zona tiendadeelectronica.int en tu docker DNS que tenga**

    1. www a la IP 172.16.0.1
    2. owncloud sea un CNAME de www
    3. un registro de texto con el contenido "1234ASDF"
    4. Comprueba que todo funciona con el comando "dig"
    5. Muestra en los logs que el servicio arranca correctamente

Primero para añadir la zona tenemos que cambiar algunos archivos de configuración como por ejemplo:

(named.conf)
```
include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
```
(named.conf.options)
```
options {
	directory "/var/cache/bind";

	forwarders {
	 	8.8.8.8;
		1.1.1.1;
	 };
	 forward only;

	listen-on { any; };
	listen-on-v6 { any; };

	allow-query {
		any;
	};
};
```
 (name.conf.local)
 ```
 zone "tiendadeelectronica.int" {
	type master;
	file "/var/lib/bind/db.tiendadeelectronica.int";
	allow-query {
		any;
		};
	};
 ```

 El archivo db.tiendadeelectronica.int le añadimos los registros que se nos piden:
 ```
 $TTL 38400	; 10 hours 40 minutes
@		IN SOA	ns.tiendadeelectronica.int. some.email.address. (
				10000003   ; serial
				10800      ; refresh (3 hours)
				3600       ; retry (1 hour)
				604800     ; expire (1 week)
				38400      ; minimum (10 hours 40 minutes)
				)
@		IN NS	ns.tiendadeelectronica.int.
ns		IN A		172.28.5.1
www		IN A		172.16.0.1
alias	IN CNAME	owncloud
texto	IN TXT		1234ASDF
 ```

 Una vez creados y modificado los archivos de conf y lib hacemos el `docker compose up` y nos deberia de crear el dns.

 ahora desde el equipo si todo funciona correctamente hacemos un `dig @172.28.5.1 tiendadeelectronica.int TEXT` y nos debería de aparecer en el answer section esto:

```
;; ANSWER SECTION:
tiendadeelectronica.int. 60 IN TXT "1234ASDF"
```
`dig @172.28.5.1 tiendadeelectronica.int CNAME`:
```
;; ANSWER SECTION:
www.tiendadeelectronica.int. 27 IN CNAME owncloud.tiendadeelectronica.int
```

Logs de que arranca correctamente:

```
13-Nov-2023 16:15:12.002 loading configuration from '/etc/bind/named.conf'
13-Nov-2023 16:15:12.002 unable to open '/etc/bind/bind.keys'; using built-in keys instead
13-Nov-2023 16:15:12.002 looking for GeoIP2 databases in '/usr/share/GeoIP'
13-Nov-2023 16:15:12.002 using default UDP/IPv4 port range: [32768, 60999]
13-Nov-2023 16:15:12.002 using default UDP/IPv6 port range: [32768, 60999]
13-Nov-2023 16:15:12.002 listening on IPv4 interface lo, 127.0.0.1#53
13-Nov-2023 16:15:12.002 listening on IPv4 interface eth0, 172.28.5.1#53
13-Nov-2023 16:15:12.002 generating session key for dynamic DNS
13-Nov-2023 16:15:12.002 sizing zone task pool based on 1 zones
13-Nov-2023 16:15:12.002 none:99: 'max-cache-size 90%' - setting to 14187MB (out of 15764MB)
13-Nov-2023 16:15:12.002 using built-in root key for view _default
13-Nov-2023 16:15:12.002 set up managed keys zone for view _default, file 'managed-keys.bind'
13-Nov-2023 16:15:12.006 configuring command channel from '/etc/bind/rndc.key'
13-Nov-2023 16:15:12.006 command channel listening on 127.0.0.1#953
13-Nov-2023 16:15:12.006 configuring command channel from '/etc/bind/rndc.key'
13-Nov-2023 16:15:12.006 command channel listening on ::1#953
13-Nov-2023 16:15:12.006 not using config file logging statement for logging due to -g option
13-Nov-2023 16:15:12.006 managed-keys-zone: loaded serial 0
13-Nov-2023 16:15:12.006 zone tiendadeelectronica.int/IN: loaded serial 10000003
13-Nov-2023 16:15:12.006 all zones loaded
13-Nov-2023 16:15:12.006 running
```
10. **Realiza el apartado 9 en la máquina virtual con DNS**
Para este apartado es repetir lo anterior pero en maquina de linux y ubuntu.

Para editar los archivos hacemso un `sudo nano [ruta del fichero a modificar] y una vez se nos haya abierto hacemos estas configuraciones.

(named.conf)
```
include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
```

(named.conf.local)
```
zone "tiendadeelectronica.int" {
	type master;
	file "/var/lib/bind/db.tiendadeelectronica.int";
	allow-query {
		any;
		};
	};
```
(named.conf.options)
```
options {
	directory "/var/cache/bind";

	forwarders {
	 	8.8.8.8;
		1.1.1.1;
	 };
	 forward only;

	listen-on { any; };
	listen-on-v6 { any; };

	allow-query {
		any;
	};
};
```
(db.tiendadeelectronica.int)
```
$TTL 38400	; 10 hours 40 minutes
@		IN SOA	ns.tiendadeelectronica.int. some.email.address. (
				10000003   ; serial
				10800      ; refresh (3 hours)
				3600       ; retry (1 hour)
				604800     ; expire (1 week)
				38400      ; minimum (10 hours 40 minutes)
				)
@		IN NS	ns.tiendadeelectronica.int.
ns		IN A		10.0.2.15
www		IN A		172.16.0.1
alias	IN CNAME	owncloud
texto	IN TXT		1234ASDF
```
Por último hacemos el dig otra vez y nos tiene que aparecer lo mismo o al menos parecido ya que puede haber pequeñas variaciones en la interfaz de ubuntu a debian pero en el answer aparece igual
```
;; ANSWER SECTION:
tiendadeelectronica.int. 60 IN TXT "1234ASDF"
```
`dig @172.28.5.1 tiendadeelectronica.int CNAME`:

```
;; ANSWER SECTION:
www.tiendadeelectronica.int. 27 IN CNAME owncloud.tiendadeelectronica.int
```
