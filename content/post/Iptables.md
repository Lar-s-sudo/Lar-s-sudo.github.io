+++
title = 'Iptables'
date = 2024-08-10T11:12:00+02:00
draft = false
+++

Iptables es un módulo del núcleo de Linux que se encarga de filtrar los paquetes de red, es decir,
es la parte que se encarga de determinar qué paquetes de datos queremos que lleguen hasta el 
sistema y cuáles no.

El usuario mediante sencillas instrucciones indica al firewall el tipo de paquetes que debe
permitir entrar, los puertos por donde se pueden recibir esos paquetes, el protocolo utilizado
para el envío de datos y cualquier otra información relacionada con el intercambio de datos entre
redes. 

Existen diferentes tablas dentro de las cuales puede haber varias cadenas. Cada cadena consiste  
en una lista de reglas con las que se comparan los paquetes que pasan por el cortafurgos y 
especifican qué se hace con los paquetes que se ajustan a ellas (target).

Cuando en el sistema se recibe o se envía un paquete, se recorren en orden las distintas 
reglas hasta dar con una que cumpla las condiciones. Una vez localizada, esa regla se activa
realizando sobre el paquete la acción indicada.
Target: 
    - Drop: el paquete de descarta no puede pasar
    - Accept: el paquete continúa su camino normal

Una pieza clave es la utilización de tablas. Depende de la regla que queramos aplicar a iptables
esta necesita la tabla asociada. 

Tipos de cadenas: PREROUTING, POSTROUTING, INPUT, OUTPUT, FORWARD
Tipos de tablas: MANGLE, NAT, FILTER

Para gestionar esto usamos el comando
```bash
iptables
```
Para mirar su documentación
```
man iptables
```
Para revisar cada una de las tablas, con permisos de superusuario
```
iptables -t filter -L
iptable -t nat -L
iptable -t mangle -L
iptable -t raw -L

```
**Cambio de políticas generales de una cadena*

Para ver para qué están configuradas actualmente sus cadenas de políticas con tráfico 
no coincidente, ejecute el comando:
(por defecto siempre muestra la tabla filter)
```
iptables -L
```
Cambiar política de la cadena INPUT en la tabla filter:
```
-P INPUT DROP
```
Esto implica denegar todos los datos de entrada a tu sistema. Para cambiarlo:
```
-P INPUT ACCEPT
```
Creamos un script que niege todas las reglas
```
#!/bin/bash

#nombre del fichero: ipt-drop.sh

#Especificamos tabla para que no haya ninguna duda
iptables -t filter -F

#Reiniciar contadores
iptables -t filter -Z

#Cambiar las politicas denegando todo
iptables -t filter -P INPUT DROP
iptables -t filter -P OUTPUT DROP
iptables -t filter -P FORWARD DROP

```
Damos al fichero permisos de administrador y ejecutamos

```
sudo chmod 744 ipt-drop.sh

./ipt-drop.sh
```
**Agregar reglas**
Opciones:
    - -A: añadir la relga al final de la cadena especificada
    - -o: Interfaz de salida
    - -i: Interfaz de entrada
    - -p: Protocolo
    - -j: Acción que se va a realizar si cumple con la regla

Permitir ping:
 Para permitir ping debemos habilitar el protocolo ICMP tanto para INPUT como para OUTPUT

 ```
 sudo iptables -A OUTPUT -o enp0s3 -p icmp -j ACCEP 
 sudo iptables -A INPUT -i enp0s3 -p icmp -j ACCEPT

 ```
Habilitar DNS:
 DNS (acrónimo de Domain Name System) es una base de datos distribuida y jerárquica,
 que almacena la información necesaria para los nombres de dominio. 

Los Servidores DNS utilizan TCP y UDP, en el puerto 53 para responder las consultas. 
Casi todas las consultas consisten de una sola solicitud UDP desde un Cliente DNS, 
seguida por una sola respuesta UDP del servidor. 
    --dport: Puerto de destino en cadena de salida (OUTPUT).
    --sport: Puerto de origen en cadena de entrada (INPUT).

```
iptable -A OUTPUT -o enp0s3 -p udp --sport 53 -j ACCPET 
iptable -A INPUT -i enp0s3 -p udp --dport 53 -j ACCPET

```
Lo mismo pata tcp usando -p tcp
 
 Habilitar la navegación web:

La navegación web es posible a través del protocolo tcp en los puertos 443 y 80. 
En el puerto 80 está http y en el puerto 443 está https que es la versión segura de esta tecnología.
 ```
 iptable -A OUTPUT -o enp0s3 -p tcp --sport 443 -j ACCEPT
 iptable -A INPUT -i enp0s3 -p tcp --dport 443 -j ACCEPT
 ```
 Con esto ya tendríamos habilitado la navegación web.

 



