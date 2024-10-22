# Práctica DNS Maestro-Esclavo con Vagrant

Este proyecto implementa un sistema DNS maestro-esclavo usando BIND9 en dos máquinas virtuales creadas con Vagrant. La configuración se automatiza mediante scripts de Shell para facilitar la instalación y despliegue de la infraestructura.

## Tabla de contenido

1. [Arquitectura del Proyecto](#arquitectura-del-proyecto)
2. [Requisitos](#requisitos)
3. [Instalación](#instalación)
4. [Configuracion Maestro](#configuracion-maestro)
5. [Decisiones Técnicas](#decisiones-técnicas)
6. [Licencia](#licencia)

## Arquitectura del Proyecto

El sistema se basa en dos servidores DNS configurados en una red privada:

- **Servidor Maestro**: `tierra.sistema.test` con IP `192.168.57.103`
- **Servidor Esclavo**: `venus.sistema.test` con IP `192.168.57.102`

Ambos servidores gestionan las zonas directa e inversa para el dominio `sistema.test`, y la transferencia de zona entre ellos está habilitada. Las consultas para las que no tienen autoridad se reenvían al DNS de OpenDNS.

## Requisitos

Antes de comenzar, asegúrate de tener instalados los siguientes requisitos:

- [Vagrant](https://www.vagrantup.com/downloads)
- [VirtualBox](https://www.virtualbox.org/)

## Instalación

Sigue estos pasos para desplegar el sistema:

1. **Crear archivo de configuracion en Vagrant**:

Utilizando Visual Studio Code o otro software a tu gusto crea un archivo Vagrant configurando las maquinas como el que tenemos en el repositorio.

2. **Levantar las máquinas virtuales con Vagrant**:

Ejecuta el siguiente comando para crear y configurar las máquinas virtuales:

```bash
vagrant up
```

3.**Verificar el estado de las máquinas**:

Para acceder a cada máquina y verificar el estado de la configuración:

```bash
vagrant ssh master   # Para acceder al servidor maestro
vagrant ssh slave    # Para acceder al servidor esclavo
```
## Configuracion Maestro y Esclavo

Para configurar el maestro correctamente debemos hacer estos cambios:

1. **Acceder al archivo named.conf.options y darle esta configuracion**:
   ```bash
   options {
	directory "/var/cache/bind";
	recursion yes;
        allow-recursion { 127.0.0.1; 192.168.57.0/24; };

	// If there is a firewall between you and nameservers you want
	// to talk to, you may need to fix the firewall to allow multiple
	// ports to talk.  See http://www.kb.cert.org/vuls/id/800113

	// If your ISP provided one or more IP addresses for stable 
	// nameservers, you probably want to use them as forwarders.  
	// Uncomment the following block, and insert the addresses replacing 
	// the all-0's placeholder.

	 forwarders {
	 	208.67.222.222;
	 };

	//========================================================================
	// If BIND logs error messages about the root key being expired,
	// you will need to update your keys.  See https://www.isc.org/bind-keys
	//========================================================================
	dnssec-validation yes;
        listen-on { any; };
	listen-on-v6 { any; };

   recursion yes;:
**Explicacion**

```bash
recursion yes;
```

Permite al servidor resolver consultas recursivas, lo que significa que el servidor puede buscar una respuesta DNS en nombre del cliente si no conoce directamente la respuesta. Esto es útil cuando los clientes necesitan resolver dominios fuera de las zonas que administra el servidor.

```bash
allow-recursion { 127.0.0.1; 192.168.57.0/24; };:
```

Esta opción especifica qué hosts pueden realizar consultas recursivas. Aquí se ha limitado a:
127.0.0.1: Permite que el servidor mismo haga consultas recursivas (loopback).
192.168.57.0/24: Permite a todos los dispositivos de la red local (192.168.57.x) realizar consultas recursivas.

``` bash
dnssec-validation yes;:
```

Habilita la validación de DNSSEC (DNS Security Extensions). DNSSEC asegura que las respuestas DNS estén firmadas criptográficamente, lo que evita ataques de suplantación o envenenamiento de caché. Si la validación está habilitada, el servidor rechazará las respuestas no firmadas o aquellas con firmas inválidas.

``` bash
listen-on { any; };:
```

Instruye a BIND para que escuche en todas las interfaces de red (cualquier dirección IP). Esto incluye tanto las interfaces de red externas como internas. Puede restringirse a direcciones específicas si, por ejemplo, solo se quiere que BIND escuche en una red particular.

``` bash
forwarders { 208.67.222.222; };:
```

Define los servidores DNS a los que se reenviarán las consultas para las que el servidor no tiene autoridad (es decir, cuando el servidor no puede responder directamente a una consulta, la envía a un reenviador). En este caso, se está usando OpenDNS, que es un servicio DNS público confiable y rápido. Esta configuración permite reenviar consultas externas fuera de la zona sistema.test a OpenDNS.

2. **Configurar la zona directa e inversa**:
``` bash
//
// Do any local configuration here
//

zone "sistema.test" {
    type master;
    file "/etc/bind/db.sistema.test";
    allow-transfer { 192.168.57.102; };
};

zone "57.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.192.168.57";
};

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";
```
3. **Creamos archivo zona directa**:
``` bash
\$TTL 86400
@   IN  SOA ns1.sistema.test. root.sistema.test. (
            2         ; Serial
            604800    ; Refresh
            86400     ; Retry
            2419200   ; Expire
            604800 )  ; Negative Cache TTL
;
@       IN  NS      ns1.sistema.test.
@       IN  NS      ns2.sistema.test.

ns1     IN  A       192.168.57.103
ns2     IN  A       192.168.57.102
```
