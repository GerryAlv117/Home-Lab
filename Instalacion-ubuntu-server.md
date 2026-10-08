# Instalación de Ubuntu Server

## Objetivo
Explicar todo el proceso que hice al descargar/instalar Ubuntu Server en mi máquina.

## Hardware utilizado
Laptop Inspiron 3520
- Intel Core i3-2370M
    - Núcleos: 2
    - Subprocesos: 4
    - Frecuencia básica: 2.40 GHz
    - Cache 3 MB
    - Almacenamiento: Adata SU630 500 GB
    - RAM: 6 GB (DDR3)
- Memoria USB 32 GB 
> [!NOTE]
> La memoria USB debe estar vacÍa o de lo contrario perderemos toda la información que esta almacenada.

## Software utilizado
[Rufus](https://rufus.ie/es/) para el booteo de la memoria USB.

## Descarga
Lo primero que se debe de hacer es descargar la ISO de Ubuntu Server desde su [página oficial](https://ubuntu.com/download/server), descargamos la "*Intel or AMD 64-bit architecture*" y damos click al botón verde y esperamos a que se complete la descarga.

## Preparación de la USB con Rufus
Seleccione la memoria que había escogido para el booteo, la iso, seleccioné MBR y BIOS o UEFI. Para esta instalación seleccioné el esquema de partición MBR y el destino BIOS o UEFI, de acuerdo con la configuración de arranque utilizada por el equipo.
Las demás opciones las deje como venian y empecé el booteo.


![Booteo de la USB con Rufus](/home-lab/img/Instalacion_US/Preparacion_USB_con_rufus.jpeg)

## Instalación

### Paso 1.
Con la memoria USB ya conectada, encendi la laptop y accedi al boot menu pulsando `F2`.
Seleccione "*USB STORAGE DEVICE*"

![Boot Menu](/home-lab/img/Instalacion_US/Boot_menu.jpeg)

### Paso 2.
Marque solo la opción "*Ubuntu Server*"

![Tipo de US](/home-lab/img/Instalacion_US/Tipo_ubuntu_server.jpeg)

**¿por qué?** Seleccioné esta opción porque se ajustaba mejor al objetivo de este Home Lab y al hardware disponible.

### Paso 3.
Marque la opción "*Done*"

En esta etapa el instalador mostró la configuración de red disponible. Decidí no realizar aquí la configuración definitiva y continuar con la instalación para realizarla posteriormente.

La opción se encuentra en la parte de abajo de la imagen, solo que no se logra apreciar.

![Configuración de red](/home-lab/img/Instalacion_US/Configuracion_red.jpeg)

Para eso recomiendo usar una conexión por cable, en mi caso use un cable cat5e conectado a mi switch que va del router.

> [!NOTE]
> La configuración de red se realizará posteriormente y se documenta en:
[Configuración de red](./Configuracion_red.md)

### Paso 4.
Deje vacío la opción ya que no cuento con la configuración de un proxy, por lo que solo presion "*Done*"

![Configuración de proxy](/home-lab/img/Instalacion_US/Configuracion_proxy.jpeg)

### Paso 5.
Como no configure la red desde el principio el "*Archive mirror*" lo deje en blanco y solo baje con las flechas y seleccioné la opción de "*done*"

![Configuración de archivo mirror](/home-lab/img/Instalacion_US/Archivo_mirror.jpeg)

### Paso 6.
Llegando a la parte más importante de la instalación de Ubuntu Server que es seleccionar el disco y partición en donde se instalara US, en mi caso como ya no le daria un uso adicional a la laptop más que funcionar de manera dedicada a mi Home Lab seleccione todo el disco sin importar que toda la información que tuviera se perdiera. Por lo que deje marcadas las opciones:
- "*use an entire disk*"
- "*ADATA SU630 ...*"
- "*Set up this disk as an LVM group*"

![Selección de disco/partición para instalación](/home-lab/img/Instalacion_US/Seleccion_particion.jpeg)

Confirme las opciones escogidas y espere a que se formateé el SSD e instalé US en el. 

![Confirmación de opciones seleccionadas](/home-lab/img/Instalacion_US/confirmacion_instalacion_SSD.jpeg)

### Paso 7.
En este paso configuré el usuario administrador, el nombre del equipo (hostname) y la contraseña que utilizaría posteriormente para iniciar sesión y administrar el servidor.

![Creamos nuestro nombre de usuario, de la máquina y la contraseña](/home-lab/img/Instalacion_US/Usuario_contraseña.jpeg)

### Paso 8.
Omití Ubuntu Pro por no tener la red configurada.

![Omisión de Ubuntu Pro](/home-lab/img/Instalacion_US/Ubuntu_pro.jpeg)

### Paso 9.
Instalé OpenSSH Server para poder usarla laptop que tiene el servidor instalado de forma remota con otro dispositivo.

Marque la casilla "*Instalar servidor OpenSHH*"

![Instalación de SSH](/home-lab/img/Instalacion_US/Instalacion_SSH.jpeg)

### Paso 10.
Una vez terminado la instalación reinicie la laptop.

![Reinicio del dispositivo](/home-lab/img/Instalacion_US/Reinicio_laptop.jpeg "")

### Paso 11.
Al final solo inicie sesión con las credenciales creadas anteriormente.
"*Homelab login:*"

Inicie sesion y posteriormente apague la máquina servidor mientras preparaba el cable Ethernet para la máquina.

## Pruebas
Hice las siguientes pruebas para verificar el estado inicial del sistema
y comprobar que los principales componentes y servicios funcionan correctamente.

### Prueba 1.
El primer comando que ejecute fue:

```
    hostnamectl
```
Para consultar la información del servidor instalado;
- Nombre del equipo: *homelab*
- Id de máquina: *identificador generado automáticamente por Ubuntu*
- Versión del kernel: *Linux 7.0.0-30-generic*
- SO: *Ubuntu 26.04.1 LTS*

![Información sobre detallada del equipo](/home-lab/img/Instalacion_US/Comando_hostnamectl.jpg)

### Prueba 2.
Con el comando puedo ver con precisión el kernel que tiene cargado mi máquina.
```
    uname -a
```

![Información detallada del kernel y SO](/home-lab/img/Instalacion_US/Comando_uname_-a.jpg)

### Prueba 3.
Para saber la información detallada de la distribución de Linux que estoy usando
```
    cat /etc/os-release
```

![Información detallada del SO](/home-lab/img/Instalacion_US/Comando_cat_etc_os-release.jpg)

### Prueba 4.
Para mostrar información detallada sobre las especificaciones técnicas del procesador (CPU)
```
    lscpu
```

![Información detallada de la CPU](/home-lab/img/Instalacion_US/Comando_lscpu.jpg)

### Prueba 5.
Para ver el estado actual de la memoria RAM
```
    free -h
```

![Información detallada sobre la RAM](/home-lab/img/Instalacion_US/Comando_free_-h.jpg)

### Prueba 6.
```
    lsblk
```
Muestra la estructura física y las particiones de tu disco

```
    df -h
```
Muestra el espacio libre y ocupado únicamente del disco y las particiones que están activas

![Información detallada sobre el disco](/home-lab/img/Instalacion_US/Comando_lsblk_y_df_-h.jpg)

### Prueba 7.
sirve para listar de forma inmediata todos los servicios y unidades del sistema que han fallado
```
    systemctl --failed
```

![Información sobre un posible fallo](/home-lab/img/Instalacion_US/Comando_systemctl_--failed.jpg)

### Prueba 8.
Se verificó que OpenSSH se encuentra disponible para aceptar
conexiones remotas mediante el puerto 22.

```
    sudo systemctl status ssh
```
Muestra el estado operativo actual del servicio SSH ✅

```
    sudo ss -tlnp | grep :22
```
Sirve para comprobar si el servidor está escuchando activamente conexiones en el puerto 22 ✅

![Verificación de OpenSSH funcionando de forma correcta](/home-lab/img/Instalacion_US/Comando_sudo_systemctl_status_ssh_y_sudo_ss_-tlnp_grep_22.jpg)

Conexión exitosa de forma remota:

![Usando OpenSSH de forma correcta con la IP del servidor](/home-lab/img/Instalacion_US/Ssh_remoto_resultado.jpg)

Las pruebas fueron realizadas con la intención de mostrar que todo esta funcionando de manera correcta.

## Problemas encontrados
Durante el booteo de la laptop con la memoria USB me salio un error por no coincidir los medios de arranque de la computadora y la memoria USB.

![Error de booteo](/home-lab/img/Instalacion_US/Error_booteo.jpeg)

## Soluciones 
Volver a preparar la memoria USB booteable con el medio de arranque correcto.

## Lo que aprendí
- Aprendí a identificar problemas relacionados con el modo de arranque.
- Aprendí a realizar una instalación básica de Ubuntu Server.
- Aprendí la importancia de tener preparada la conexión de red antes de comenzar la instalación.
- Aprendí a identificar los principales componentes de hardware desde Linux.
- Aprendí a realizar pruebas básicas para verificar el estado del sistema.
- Aprendí a instalar OpenSSH Server para administrar el equipo remotamente.
- Aprendi a como usar OpenSSH en un equipo diferente para poder trabajar de forma remota.

## Resultado final
Ubuntu Server quedó instalado correctamente en la laptop y se realizaron pruebas para comprobar el estado inicial del sistema. También se verificó el funcionamiento de OpenSSH mediante una conexión remota.
La configuración de red se documentará por separado. 

## Comandos utilizados

| Comando | Propósito |
|---|---|
| `hostnamectl` | Información general del sistema |
| `uname -a` | Información del kernel |
| `cat /etc/os-release` | Información de la distribución |
| `lscpu` | Información del procesador |
| `free -h` | Estado de la memoria RAM |
| `lsblk` | Dispositivos y particiones |
| `df -h` | Uso del almacenamiento |
| `systemctl --failed` | Servicios/unidades con fallos |
| `systemctl status ssh` | Estado de OpenSSH |
| `ss -tlnp` | Puertos TCP en escucha |