# SSH solo con llaves

Esta es mi guía de cómo configuré SSH para que mi servidor (Ubuntu Server) **solo deje entrar con llaves** y ya no con contraseña. Es la primera vez que hago esto, así que lo escribí paso a paso, con los errores que tuve y lo que aprendí.

---

## ¿Qué es esto y por qué lo hice?

Antes entraba a mi servidor con mi usuario y contraseña. El problema es que una contraseña se puede adivinar o robar. Con llaves es más seguro:

- Tengo una **llave privada** (se queda en mi laptop, nunca se comparte).
- Tengo una **llave pública** (la pongo en el servidor, es como un candado).
- Solo quien tenga la llave privada que encaje con ese candado puede entrar.

Además, la llave privada está protegida con una **passphrase** (una contraseña larga que inventé yo), por si alguien llegara a robarme el archivo.

---

##  Lo que necesitaba

- La **IP** del servidor (en el servidor: `ip a`).
- Mi **usuario** del servidor (en mi caso `luis`).
- Mi laptop y mi servidor en la **misma red**.
- Comprobar que ya podía entrar con contraseña antes de empezar:

``` PowerShell
  ssh luis@IP_DEL_SERVIDOR
```

---

##  Paso a paso

### Paso 1: Crear mis llaves (en mi laptop, NO en el servidor)

Abrí **PowerShell** en mi laptop con Windows y ejecuté:

``` PowerShell
   ssh-keygen -t ed25519 -C "laptop-luis"
```

   <br>

- `-t ed25519` es el tipo de llave (moderno y seguro).
- `-C "laptop-luis"` es solo una **etiqueta** para saber de qué equipo es la llave. No es la contraseña.

Me hizo tres preguntas:

1. Dónde guardar la llave → **Enter** (la ruta por defecto está bien).
2. Passphrase → escribí una frase larga (mientras la escribes **no se ve nada**, lo cual es normal).
3. Repetir la passphrase.

Se crearon dos archivos en la carpeta `.ssh`:

| Archivo | Qué es |
|---|---|
| `id_ed25519` | Llave **privada**. Nunca la subo a GitHub ni la comparto por ningun medio. |
| `id_ed25519.pub` | Llave **pública**. Esta es la que va al servidor |

![Generacion de lleves publica, privada y passphrase](img/ssh-llaves/ssh-keygen%20-t%20ed25519%20-C%20laptop-luis.png)

### Paso 2: Pasar la llave pública al servidor

En Windows no existe `ssh-copy-id`, así que lo hice a mano.

1. Mostré mi llave pública (es **una sola línea** que empieza con `ssh-ed25519`):
   ```powershell
      type $env:USERPROFILE\.ssh\id_ed25519.pub
   ```
   La copié completa.

![Mostrar llave publica y copiarla completa](img/ssh-llaves/type%20envUSERPROFILE.sshid_ed25519.pub.png)

   <br>

2. Entré al servidor con mi contraseña (todavía se podía):
   ```powershell
      ssh luis@IP_DEL_SERVIDOR
   ```

   <br>

3. Ya dentro del servidor, preparé la carpeta y el archivo:
   ```bash
      mkdir -p ~/.ssh
      chmod 700 ~/.ssh
      nano ~/.ssh/authorized_keys
   ```

   <br>

1. Pegué la línea, guardé con `Ctrl+O` → `Enter` y salí con `Ctrl+X`.

   <br>

2. Ajusté los permisos (SSH es muy estricto con esto):
   ```bash
      chmod 600 ~/.ssh/authorized_keys
   ```

   <br>

3. Salí con `exit`.


### Paso 3: Probar que entro con llave

Desde mi laptop:

```bash
   ssh luis@IP_DEL_SERVIDOR
```

Ahora ya no me pidió la contraseña del usuario, sino la **passphrase de mi llave**. y logre Entrar.

>[!IMPORTANT]
>
> **No pasar al siguiente paso hasta que esto funcione.** Si no, me puedo quedar fuera del servidor.
> Antes de tocar algo, **dejé una sesión SSH abierta** como salvavidas y usé otra terminal para probar.

### Paso 4: Desactivar las contraseñas


1. Hice un respaldo de la configuración:
   ```bash
      sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
   ```
![Copia de respaldo](img/ssh-llaves/sudo%20cp%20etcsshsshd_config%20etcsshsshd_config.bak.png)

   <br>

2. Busqué dónde estaba definida la opción de contraseñas:
   ```bash
      sudo grep -ri "PasswordAuthentication" /etc/ssh/sshd_config /etc/ssh/sshd_config.d/
   ```

   Encontré que el archivo `/etc/ssh/sshd_config.d/50-cloud-init.conf` tenía `PasswordAuthentication yes` activo (sin `#`).

![PasswordAuthentication no](img/ssh-llaves/sudo%20grep%20-ri%20PasswordAuthentication%20etcsshsshd_config%20etcsshsshd_config.d.png)

   <br>

3. Edité ese archivo y cambié `yes` por `no`:
   ```bash
      sudo nano /etc/ssh/sshd_config.d/50-cloud-init.conf
   ```
   ```
   PasswordAuthentication no
   ```
![PasswordAuthentication no](img/ssh-llaves/sudo%20nano%20etcsshsshd_config.d50-cloud-init.conf.png)


   <br>

4. Edité también el archivo principal:
   ```bash
      sudo nano /etc/ssh/sshd_config
   ```

![Nano config](img/ssh-llaves/sudo%20nano%20etcsshsshd_config.png)

   Con `Ctrl+W` busqué cada opción, le quité el `#` del inicio y la dejé así:
   ```
      PubkeyAuthentication yes
      PasswordAuthentication no
      PermitRootLogin no
      KbdInteractiveAuthentication no
   ```

![Valores cambiados](img/ssh-llaves/confirmacion%20de%20valores%20cambiados.png)

>[!NOTE]
>
>**Regla del `#`:** si la línea empieza con `#`, SSH la ignora. Para que cuente, hay que quitárselo.

### Paso 5: Revisar y reiniciar

1. Revisé que no hubiera errores (si no imprime nada, está bien):
   ```bash
   sudo sshd -t
   ```
![Sin errores](img/ssh-llaves/sudo%20sshd%20-t.png)


   <br>

1. Confirmé qué configuración quedó realmente activa:
   ```bash
   sudo sshd -T | grep -E "passwordauthentication|pubkeyauthentication|permitrootlogin|kbdinteractiveauthentication"
   ```

   <br>

   Tenía que salir `passwordauthentication no`, `pubkeyauthentication yes` y `kbdinteractiveauthentication no`.

![Confirmacion](img/ssh-llaves/sudo%20sshd%20-T%20%20grep%20-E%20passwordauthenticationpubkeyauthenticationpermitrootloginkbdinteractiveauthentication.png)

1. Reinicié el servicio:
   ```bash
   sudo systemctl restart ssh
   ```

### Paso 6: Prueba final (desde mi laptop)

Con la otra sesión todavía abierta, probé en una terminal nueva:

```bash
   ssh luis@IP_DEL_SERVIDOR
```
Me pidió la passphrase y entré.

![passphrase](img/ssh-llaves/frase%20en%20vez%20de%20contraseña.png)


Y para comprobar que ya no deja entrar con contraseña:

```bash
   ssh -o PubkeyAuthentication=no luis@IP_DEL_SERVIDOR
```
Respondió `Permission denied (publickey)`. Eso significa que quedó bien cerrado.

![Permission denied (publickey)](img/ssh-llaves/ssh%20-o%20PubkeyAuthentication=no%20luis@IP_DEL_SERVIDOR.png)


---

##  Errores que tuve (para no repetirlos)

- **Probé la conexión desde el propio servidor** en vez de desde mi laptop. Me salió `Permission denied (publickey)` y pensé que lo había arruinado, pero era normal: las llaves están en mi laptop, no en el servidor. Las pruebas siempre se hacen desde el **cliente**.
- **Un archivo de `sshd_config.d/` mandaba sobre el principal.** Si solo hubiera editado `sshd_config`, el servidor habría seguido aceptando contraseñas.
- **Se me olvidaba el `#`.** Una línea comentada no cuenta aunque tenga el valor correcto.
- **Revisar `KbdInteractiveAuthentication`.** Si queda en `yes`, a veces se puede entrar con contraseña por otro camino.

---

##  Cómo agregar otro dispositivo

Cada dispositivo necesita **su propia llave**. Como ya apagué las contraseñas, `ssh-copy-id` no sirve desde el dispositivo nuevo. Lo que hago:

1. En el dispositivo nuevo genero su llave (Paso 1) y copio la línea de su `.pub`.

   <br>

2. Desde un dispositivo que ya tenga acceso, entro al servidor y agrego esa línea:
   ```bash
   echo "ssh-ed25519 AAAA...(línea completa)... laptop-2" >> ~/.ssh/authorized_keys
   ```

   <br>

3. Pruebo desde el dispositivo nuevo.

En `authorized_keys` va **una línea por dispositivo**. Si pierdo uno, borro solo su línea.

---

##  ¿Qué contraseña se usa y cuándo?

| Contraseña | Dónde se usa |
|---|---|
| **Contraseña del usuario** (la de la instalación) | Al iniciar sesión directo en el servidor (monitor y teclado) y para `sudo` |
| **Passphrase de la llave** (la que inventé) | En mi laptop, cada vez que me conecto por SSH |

Por SSH ya no se pide la contraseña del usuario. Y no necesito iniciar sesión en el servidor cuando lo enciendo: SSH arranca solo con el sistema y me conecto desde mi laptop.

---

##  Plan de rescate

Si me quedo fuera, conecto monitor y teclado al servidor, entro con mi contraseña normal y restauro la configuración:

```bash
sudo cp /etc/ssh/sshd_config.bak /etc/ssh/sshd_config
sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config.d/50-cloud-init.conf
sudo systemctl restart ssh
```

Si pierdo mi laptop o mi llave privada, desde el servidor borro la línea de ese equipo en `~/.ssh/authorized_keys`.

---

##  Checklist final

- [x] Llaves generadas en mi laptop (ed25519 con passphrase)
- [x] Llave pública agregada en `~/.ssh/authorized_keys`
- [x] Permisos correctos (`700` en `.ssh`, `600` en `authorized_keys`)
- [x] `PasswordAuthentication no` en **ambos** archivos
- [x] Poner `PermitRootLogin no`
- [x] `sshd -t` sin errores y servicio reiniciado
- [x] Entro con llave y ya no me deja con contraseña
- [x] Respaldar mi llave privada en un lugar seguro.

---

## Tabla de comandos utilizados

| Comando | Propósito |
|---|---|
| `ip a` | Muestra las interfaces de red del servidor y sus direcciones IP. |
| `ssh-keygen -t ed25519 -C "laptop-luis"` | Genera un par de llaves SSH (pública y privada) utilizando el algoritmo Ed25519. |
| `type $env:USERPROFILE\.ssh\id_ed25519.pub` | Muestra el contenido de la llave pública SSH desde PowerShell para poder copiarla al servidor. |
| `ssh luis@IP_DEL_SERVIDOR` | Establece una conexión SSH con el servidor utilizando el usuario y su dirección IP. |
| `mkdir -p ~/.ssh` | Crea el directorio `.ssh` en el directorio personal del usuario si todavía no existe. |
| `chmod 700 ~/.ssh` | Establece permisos de acceso al directorio `.ssh`, permitiendo el acceso únicamente a su propietario. |
| `nano ~/.ssh/authorized_keys` | Abre o crea el archivo donde se almacenan las llaves públicas autorizadas para conectarse mediante SSH. |
| `chmod 600 ~/.ssh/authorized_keys` | Establece permisos para que únicamente el propietario pueda leer y modificar el archivo `authorized_keys`. |
| `exit` | Cierra la sesión SSH actual. |
| `sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak` | Crea una copia de seguridad del archivo principal de configuración del servidor SSH. |
| `sudo grep -ri "PasswordAuthentication" /etc/ssh/sshd_config.d/` | Busca recursivamente la directiva `PasswordAuthentication` en los archivos de configuración adicionales de SSH. |
| `sudo nano /etc/ssh/sshd_config.d/50-cloud-init.conf` | Abre el archivo de configuración de Cloud-Init para modificar la autenticación mediante contraseña. |
| `sudo nano /etc/ssh/sshd_config` | Abre el archivo principal de configuración del servidor SSH para modificar sus parámetros de seguridad. |
| `sudo sshd -t` | Comprueba la sintaxis del archivo de configuración de SSH y detecta posibles errores. |
| `sudo sshd -T` | Muestra la configuración efectiva del servidor SSH después de procesar sus archivos de configuración. |
| `sudo sshd -T \| grep -E "passwordauthentication\|pubkeyauthentication\|permitrootlogin\|kbdinteractiveauthentication"` | Filtra y muestra los valores efectivos de las principales opciones de autenticación y acceso SSH. |
| `sudo systemctl restart ssh` | Reinicia el servicio SSH para aplicar los cambios realizados en su configuración. |
| `ssh -o PubkeyAuthentication=no luis@IP_DEL_SERVIDOR` | Intenta establecer una conexión SSH deshabilitando la autenticación mediante llave pública para comprobar que el acceso por contraseña está deshabilitado. |
| `echo "ssh-ed25519 AAAA...(línea completa)... laptop-2" >> ~/.ssh/authorized_keys` | Agrega una nueva llave pública al archivo `authorized_keys` sin sobrescribir las llaves existentes. |
| `sudo cp /etc/ssh/sshd_config.bak /etc/ssh/sshd_config` | Restaura el archivo principal de configuración SSH desde la copia de seguridad. |
| `sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config.d/50-cloud-init.conf` | Modifica la configuración de Cloud-Init para volver a habilitar la autenticación mediante contraseña SSH. |