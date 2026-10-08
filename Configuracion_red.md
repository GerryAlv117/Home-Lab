# Configuración de red en Ubuntu Server

## Objetivo 
Documentar el diagnóstico y la configuración de red realizada en Ubuntu Server para solucionar la ausencia de conectividad IPv4 y comprobar posteriormente el funcionamiento de la conexión de red.

## Configuración de red

## Problema
Al momento de hacer pruebas de comandos y ping para saber y ver las direcciones IP solo se podía observar el ipv6 y con el IPv4 no se encontraba y al momento de hacer las pruebas de ping con IPv4 no habia respuesta y ver si podia tener conexión con x pagina de internet no fucnionaban.

## Diagnostico

#### Paso 1
Con el comando `ip a` permite consultar las direcciones IP y el estado de las interfaces de red del sistema.
El comando `ip link` me ayudo a ver si la interfaz fisica y logica estaba activa.

```
    ip a
    ip link
```

![Comando ip a e ip link](/home-lab/img/Configuracion_red/comando_IP_A_e_IP_LINK.jPeg)

En este caso se pude observar que el servidor cuenta con con una direccion IPv6 "*2006:...*" ademas me ayudo a ver que la interfaz de red esta activa "*state up*" y "*lower up*", pero no logre encontrar mi direccion IPv4.

#### Paso 2
Al ejecutar `ip route`, no se mostró ninguna ruta configurada, lo que indicaba que el servidor no tenía una ruta IPv4 disponible para comunicarse con otras redes.
Con el comando `ping -c 4 8.8.8.8` probar la conectividad de red y su tiempo de respuesta hacia un servidor publico de google pero tuve la respuesta no deseada **la red no esta disponible** (**network is unreachable**).
Pero aquí pasa algo curioso al ejecutar `ping -c 4 google.com`, la prueba fue exitosa. Debido a que el servidor únicamente mostraba una dirección IPv6 en ese momento, esto indicaba que la conectividad estaba funcionando mediante IPv6. Sin embargo, la conectividad IPv4 continuaba sin funcionar, como se comprobó posteriormente con `ping -c 4 8.8.8.8`. 

```
    ip route
    ping -c 4 8.8.8.8
    ping -c 4 google.com
```

![Comandos ip route, ping -c 4 8.8.8.8 y ping -c 4 google.com](/home-lab/img/Configuracion_red/comando_ip_route_ping_-c_48888_y_ping_-c_4_google.jpeg)

#### Paso 3
Con el comando `resolvectl status` revise la configuración actual que tiene mi servidor de Ubuntu, todo parece estar bien no encontré ningún error.

```
    resolvectl status
```

![Comando resolvectl status](/home-lab/img/Configuracion_red/comando_resolvectl_status.jpg)

#### Paso 4
al querer ver las direcciones IP (IPv4 e IPv6) que mi servidor tenia con el comando `hostname -I` solo obtuve como respuesta la dirección IPv6, no mostró la dirección IPv4

```
    hostname -I
```

![Comando hostname -I](/home-lab/img/Configuracion_red/comando_hostname_-I.jpg)

#### Paso 5
Con el comando `ip -4 addr show enp9s0` comprobé que la interfaz enp9s0 no tenía una dirección IPv4 asignada.

```
    ip -4 addr show enp9s0
```

![Comando ip -4 addr show enp9s0](/home-lab/img/Configuracion_red/comando_ip_-4_addr_show_enp9s0.jpg)

#### Paso 6
Me asegure de que la interfaz estuviera activa y funcional con el comando `networkctl status enp9s0`, en la que comprobé que si estaba bien, era la razon por la que podia hacer un ping al servidor de google y obtener una respuesta con la IPv6, pero tambien comprobé que el US no tenia una dirección IPv4, ya que solo salia la dirección IPv6

```
    networkctl status enp9s0
```

![Comando networkctl status enp9s0](/home-lab/img/Configuracion_red/comando_networkctl_status_enp9s0.jpg)

#### Paso 7 
Con el siguiente comando `sudo cat /etc/netplan/00-installer-config.yaml` encontré el porque no tenia una direccion IPv4 y es que no tenia el `DHCP`, que es lo que me asignaria mi IPv4 de manera automatica.

##### ¿Qué es DHCP4?

`dhcp4: true` indica que la interfaz de red debe utilizar DHCP para obtener automáticamente una configuración IPv4, incluyendo una dirección IP, puerta de enlace y otros parámetros proporcionados por el servidor DHCP de la red.

![Comando sudo cat /etc/netplan/00-installer-config.yaml](/home-lab/img/Configuracion_red/comando_sudo_cat_-etc-netplan-00-installer-config_yaml.jpg)

> [!NOTE]
> Antes de haber editado el archivo `...-config.yaml` se guardo una copia de respaldo por si llegara haber un problema regresar a esa configuración.

```
    sudo cp /etc/netplan/00-installer-config.yaml 
    /etc/netplan/00-installer-config.yaml.backup
```

Para comprobar que se realizo con exito el backup use el comando:
```
    ls -l /etc/netplan/
```

## Solución
Se agrego el `dhcp4: true` a la configuración del archivo `sudo cat /etc/netplan/00-installer-config.yaml`. 

```
    dhcp4: true
```

![Se agrego dhcp4: true](/home-lab/img/Configuracion_red/dhcp4_true.jpg)

Para poder obtener la interfaz IPv4. Una vez agregado presione `Ctrl + o` para guardar lo que agregue y `Ctrl + x` para salir del editor del archivo.
Posteriormente solo se aplico y guardo la configuración:

```
    sudo netplan try
    sudo netplan apply
```

## Pruebas
Se realizaron con exito los siguientes comandos:

**1.** Se verifico la ruta:
```
    ip route
```
![Comando ip route](/home-lab/img/Configuracion_red/comando_ip_route_funcional.png)

**2.** Volvi a verificar el comando `hostname -I`
 ```
    hostname -I
```

![El comando hostname -i funciona correctamente](/home-lab/img/Configuracion_red/comando_hostname_-I_funcional.png)
**Muestra la direccion IPv4 (izquierda) e IPv6 (derecha)**

**3.** Se comprueba el resultado exitoso de `ip -4 addr show enp9s0`
```
    ip -4 addr show enp9s0
```
![Comando ip -4 addr show enp9s0](/home-lab/img/Configuracion_red/Comando_ip_-4_addr_show_enp9s0_funcional.png)

**4.** Se realizo una prueba de conectividad:
```
    ping -c 4 8.8.8.8
```
![Comando ip route](/home-lab/img/Configuracion_red/comando_ping_-c_4_8_8_8_8_funcional.png)

**5.** Se comprobo el correcto funcionamiento de DNS
```
    ping -c 4 google.com
```
![Comando ip route](/home-lab/img/Configuracion_red/comando_ping_-c_4_google_com_funcional.png)

## Lo que aprendi
- Tener preparado el cable ethernet para hacer la configuracion durante la instalación.
- Si no veo el IPv4 en mis interfaces de red es posiblemente un error y falta agregar el `DHCP4` en el `config.yaml`.
- Que una de las razones del porque no puedan funcionar los comandos de conectividad es porque no tengo una IPv4 en mi interfaz de red.
- Algunos comandos de ping (como `ping -c 4 google.com`) pueden funcionar con solo la dirección IPv6.

## Resultado final
La configuración del servidor se realizo de manera correcta, realizandose pruebas para confirmar su buen funcionamiento.

## Comandos utilizados

| Comando | Propósito |
|---|---|
| `ip a` | Muestra las direcciones IP y el estado de las interfaces de red. |
| `ip link` | Muestra el estado de las interfaces de red y permite comprobar si están activas. |
| `ip route` | Muestra la tabla de enrutamiento del sistema. |
| `ping -c 4 8.8.8.8` | Comprueba la conectividad IPv4 hacia un servidor público. |
| `ping -c 4 google.com` | Comprueba la conectividad y la resolución de nombres mediante DNS. |
| `resolvectl status` | Muestra el estado y la configuración actual de DNS. |
| `hostname -I` | Muestra las direcciones IP asignadas al servidor. |
| `ip -4 addr show enp9s0` | Muestra las direcciones IPv4 asignadas a la interfaz `enp9s0`. |
| `networkctl status enp9s0` | Muestra el estado y la configuración de la interfaz `enp9s0`. |
| `sudo cat /etc/netplan/00-installer-config.yaml` | Muestra el contenido del archivo de configuración de Netplan. |
| `sudo cp /etc/netplan/00-installer-config.yaml /etc/netplan/00-installer-config.yaml.backup` | Crea una copia de respaldo de la configuración de Netplan. |
| `ls -l /etc/netplan/` | Muestra los archivos existentes dentro del directorio de configuración de Netplan. |
| `sudo netplan try` | Prueba la nueva configuración de Netplan antes de aplicarla permanentemente. |
| `sudo netplan apply` | Aplica la configuración de Netplan. |