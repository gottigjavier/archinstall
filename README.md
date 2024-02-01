# Instalación de Arch.

## Pasos previos.

- Crear un medio externo booteable. Aconsejo Ventoy ya que permite cargar varios SO y luego elegir cuál se va a instalar.

- Si la instalación va a ser en una máquina que ya posee SO:
``` 
~$ ip add
```
y anotar la dirección MAC que figura en wlan0 (wlp2s0) link/ether, por ejemplo, 8a:94:2f:a7:4c:e2
A veces no tenemos acceso al setup de nuestro router y el broadcast filtra las direcciones MAC y bloquea algunas, entre ellas, la que viene por defecto en la .iso de Arch, entonces al conectarnos podemos hacer un ping al gateway del router (192.168.0.1) pero no a una ip o direciión externa.

- Hacer un backup de la información relevante, obviamente.

---

## Arrancamos desde el booteable.

El SO se carga en la RAM y nos da una pequeña bienvenida en la que nos informa un par de cosas, entre ellas instrucciones para conectarmos a la web (iwctl y nmcli). Luego usaremos iwctl.

- La distribución de teclado es inglés. Cambiarlo a español:
```
~$ loadkeys es
```

- Conectarnos al Wifi accedediendo al entorno iwd:
```
~$ iwctl
~#
```

- Obtenemos los datos de nuestra interfaz:
```
~# device list
```
nos interesa el nombre, probablemente "wlan0"


- Escaneamos las redes wifi:
```
~# station wlan0 scan
```

- Pedimos que nos muestre la SSID de las redes
```
~# station wlan0 get-networks
```
Aparece la lista de la redes.

- Nos cconectamos a la que nos corresponde:
```
~# station wlan0 connect "Mi Red wifi"
```

>Si no usamos comillas:
```
~# station wlan0 connect Mi\ Red\ wifi
```

Nos pedirá la clave.

- Salimos del entorno iwd:
```
~# exit
~$ 
```

- Hacemos in ping a un dirección cualquiera
```
~$ ping -c 2 google.com
```
Si no funciona es por el filtrado de la MAC, entonces procedemos a cambiarla:

- Primero debemos desactivar la interfaz
```
~$ ip link set dev wlan0 down
```

- Introducimos la nueva dirección MAC
```
~$ ip link set dev wlan0 address 8a:94:2f:a7:4c:e2
```

- Activamos la interfaz:
```
~$ ip link set dev wlan0 up
```

- Otra forma de ver los datos de la interfaz:
```
~$ ip link show wlan0
```


> Se puede usar el método antiguo
>
> ``` 
> ~$ ifconfig wlan0 down 
> 
> ~$ ifconfig wlan0 hw ether 8a:94:2f:a7:4c:e2
>
> ~$ ifconfig wlan0 up
> ```
>
> Si ahora hecemos un ping debería funcionar.


*Durante la instalación se bajan muchos paquetes que pequeños donde se demora más en la preparación a bajar el paquete que la descarga en sí.
Conviene entonces habilitar descargas en paralelo. Para ello vamos a la configuración de pacman:*
```
~$ nano /etc/pacman.conf
```

- Descomentamos la línea

**ParallelDownloads = 5**

Si tenemos buen ancho de banda, podemos aumentar el número de hilos: 

**ParallelDownloads = 8**

- Podemos darle algo de color a la tty descomentando la línea **#Color**

- Guardamos:

**CTRL + o**
 
**Enter**

**CTRL + x**

>Otro problema que puede aparecer si vamos a instalar en una máquina que ya tenía SO es algún conflicto con la tabla de particiones, incluso si establecemos una nueva. La primera sospecha que de algo de esto ocurre es que no nos pregunte el esquema de partición (mbr, gpt, etc.) que queremos.
>
>Para evitar inconvenientes borraremos toda tabla de partición previa.

- Listamos los discos:
```
~$ fdisk -l
```
aparecerán, al menos, nuestro medio externo y el disco sobre el cual haremos la instalación, que generalmente aparece como /dev/sda

- Borramos toda tabla de partición:
```
~$ sgdisk --zap-all /dev/sda
```

Con esto resuelto, podemos establecer la nueva tabla de partición.
Como ejemplo, voy a elegir 5 particiones ( /boot, [swap], / , /home, /tmp ) en un disco de 1TB
Para una RAM de 8Gb resrvaré 8Gb de swap.

Salvo que nuestra máquina sea un dinosaurio, usará boot UEFI. 

- Para comprobarlo:
```
~$ ls /sys/firmware/efi/efivars
```
si devuelve una lista de archivos o carpetas, usa UEFI


- Hay varios métodos para particionar. El más simple y efectivo para mí es usar cfdisk
```
~$ cfdisk /dev/sda
```

- *elegimos "gpt"*

Abrirá el entorno. En la parte superior se irá viendo la información de nustras particiones y en la parte inferior las acciones.
Con las flechas vamos seleccionando, arriba y abajo para las particiones, derecha e izquierda para las acciones.
Inicialmente aparece todo el dico como "free".

>Por prolijidad, generalmente el orden de las particiones es:
>
>/dev/sda1 --> /boot   <128M>
>
>/dev/sda2 --> /        <50G>
>
>/dev/sda3 --> /home   <936G>
>
>/dev/sda4 --> /tmp      <5G>
>
>/dev/sda5 --> [swap]   <~9G>

Es un cálculo para un disco ideal. Depende también del estado de disco. Es frecuente que el tamaño utilizable sea menor al que dice que tiene y habrá que adaptarse a esto.

- Posicionados en "New" --> Enter

establecemos el tamaño de la partición (a la izquierda sobre el menú de acciones). Por defecto aparecerá el tamaño completo del dico o lo libre restante.

### Para /boot 

Partition size: 128M
 
Notar las unidades (M para megabyte y G para gigabyte). Para mí 128M es suficiente si se va a instalar un solo SO. Se puede elegir cualquier valor por encima de éste. En general conviene, cuando se pueda, elegir múltiplos de 1024b. Muchos recomiendan como estándar 512M.

- Con la flecha horizontal nos posicionammos en **Type**
- en el menú que se despliega elegir el primero: **EFI System**

>(en caso de no utilizar UEFI, se peude optar por **BIOS boot** u otro que especifique el fabricante del hardware)

En la parte superior ya aparece la información de nuestra primera partición. Lo más relevante es

*Device: /dev/sda1*

*Size: 128M*

*Type: EFI System*

- Con la felcha abajo nos pocicionamos en el siguiente renglón, *Free Space --> Enter*

**Acción: New**

### Para la raíz del sistema

*Partition size: 50G* (aconsejo más de 20G, siempre)

Aparece:

*Device: /dev/sda2*

*Size: 50G*

*Type: Linux filesystem*


>En caso de que no aparezca el Type o aparezca como "unknow", ir a Type y en el menú seleccionarlo.

- Felcha abajo, nos pocicionamos en el siguiente renglón, *Free Space --> Enter*

**Acción: New**

### Para home

*Partition size: 936G*

Aparece:

*Device: /dev/sda3*

*Size: 936G*

*Type: Linux filesystem*


- Felcha abajo, nos pocicionamos en el siguiente renglón, *Free Space --> Enter*

**Acción: New**

### Para tmp

*Partition size: 5G*

Aparece:

*Device: /dev/sda4*

*Size: 5G*

*Type: Linux filesystem*


- Felcha abajo, nos pocicionamos en el siguiente renglón, *Free Space --> Enter*

**Acción: New**

### Para swap

*Partition size: 9G* (o lo que reste de espacio)

- Seleccionar *Type --> menú --> Linux swap --> Enter*

Aparece:

*Device: /dev/sda5*

*Size: 9G*

*Type: Linux swap*


- Flechas horizontales *--> Write --> Enter*

Nos pide que confirmemos escribiendo **"yes"**

- Salimos

Flechas horizontales *--> Quit --> Enter*

En este punto, están hechas las particiones pero no están formateadas. Esto se puede hacer manualmente desde la línea de comandos o ya en el proceso de instalación.
Yo recomiendo esta última ya que con la primera me han surgido errores que abortan la instalación.
No obstante, mostaré cómo hacerlo. Si siguen el proceso de instalación, verán que estos comandos aparecen.
Obviaré las opciones de agregar etiquetas, etc. 

> ```
> ~$ mkfs.fat -F32 /dev/sda1
>
> ~$ mkfs.ext4 /dev/sda2
>
> ~$ mkfs.ext4 /dev/sda3
>
> ~$ mkfs.ext4 /dev/sda4
>```
>
>Aconsejo dejar la partición swap para después de la instalación
>
>formatear
>```
> ~$ mkswap /dev/sda5
>```
>activar
>```
> ~$ swapon /dev/sda5
>```

La forma recomendada por mí es hacerlo dentro del proceso de instalación.

---

## Instalación del SO.
```
~$ archinstall
```

Esto abrirá el menú de instalación, el cual es bastante intuitivo. Me centraré en la configuración de disco, pero antes un par de recomendaciones.

>- A cada apartado del menú se accede con Enter
>
>- Si en el apartado aparecen muchas opciones, se puede buscar con "/". Ejemplo: en idioma -> "/" -> "sp" --> aparece spanish seleccionado --> Enter
>
>- Para el caso de tener que seleccionar (única o múltiple) se lo hace con la barra espaciadora
>
>- En mirrors, además de seleccionar servidores cercanoa mi ubicación, selecciona el de USA, es veloz. 
>
>- En locales selesccionar sin .UTF-8, ej: es_AR en vez de es_AR.UTF-8. Algunos lo aconsejan pero me ha saltado error en el proceso de instalación. Además, debajo aparece la posibilidad de elegir la codificación.
>
>- Gestor de arranque: el confiable grub
>
>- En servidores gráficos, a menos que se sepa bien lo que se va a hacer, seleccionar all-opensource
>
>- En swap depende. Si se deja en true, se habilita la zram que usa una parte de la ram para crear un espacio swap comprimido en vez de usar el disco. Si se tiene una ram de tamaño suficiente esto podría se atractivo, más aún si se tiene un SSD y no se configura la ram para menguar la cantidad de acceso a éste. En el caso de tener un tamaño de ram no muy holgado, recomiendo elegir "false" y usar la swap en disco (ya hemos reservado unapartición para esto). En caso de tener un SSD habrá que configurar ciertos parámetros, como por ejemplo "swappiness" para reducir la frecuencia de lectura y escritura en el disco. En caso de tener HDD, si bien los tiempos de acceso son mayores, el desgaste es mucho menor.
>
>- Perfiles: si no se tiene amplia experiencia en window manager, elegir (además) un ecritorio liviano, como Mate. Ventana de bienvenida: a gusto (es la que en el arranque muestra el usuario y pide la contraseña además de permitir elegir el escritorio o wm a utilizar).
>
>- Servidor de audio: dependerá de la edad del sistema. Piewire es el más actual.
>
>- Configuración de la red: seleccionar NetworkManager 
>
>- En repositorios adicionales selecciona multilib. Esto es para aplicaciones que necesiten librerías de 32 bits.

### Ahora, la configuración del disco.

- En el apartado Disk configuration.

-> "Partición manual"

-> el disco donde se va a instalar -> Enter

>aparece el disco con las particiones. De nuevo, no hacer nada con swap por el momento

-> Enter en cada partición para modificar

-> marcar para ser formateada -> pasa de "existing" a "modify"

-> tipo de partición: ext4 o Fat32 según corresponda.

-> punto de montaje: *boot* ->/boot ... *raíz* ->/ ... *home* ->/home ... *tmp* ->/tmp

- Confirmar y salir.

- Desplazarse hasta **Instalar** -> muestra un json con las configuraciones -> *Enter* --> Comienza la instalación.

---

## Post instalación. 

>Montaremos y activaremos la swap. Instalaremos el grub. Este último se podría hacer después de reiniciar usando las ventanas de inicio por primera vez. Particularmente me gusta esta forma.

Al final de la instalación va a preguntar si queremos *chroot -> si*

- Verificamos que no hay entrada para el SO el e menú de inicio:
```
~# efibootmgr
```
veremos que no hay referencia a Arch.

- Instalamos el grub
```
~# grub-install --target=x86_64-efi --efi-directory=/boot
```
ahora, si hacemos otro efibootmgr veremos una línea agregada que muestra el acceso al SO

- Swap: esto es copia de lo que explicamos anteriormente. Ahora lo aplicaremos.

- Formatear
```
~# mkswap /dev/sda5
```

- Activar
```
~# swapon /dev/sda5 
```

### LISTO.

```
~# exit

~$ reboot now
```

