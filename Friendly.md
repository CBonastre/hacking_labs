## Resolución de la máquina CTF

### 1. **Escaneo de Puertos con Nmap**
Comenzamos realizando un escaneo con `nmap` para identificar los servicios abiertos en la máquina objetivo. Usamos el siguiente comando para un escaneo básico:

Identificamos que los puertos **FTP (21)** y **HTTP (80)** están abiertos. Además, el servidor FTP permite el acceso anónimo, lo que puede ser aprovechado para subir archivos.

![Captura de pantalla 2024-10-10 120830](https://github.com/user-attachments/assets/ddb78279-337d-486c-a730-26aff86776a4)


### 2. **Acceso FTP con Anonymous**
Nos conectamos al servidor FTP utilizando las credenciales de acceso anónimo:

![Captura de pantalla 2024-10-10 121159](https://github.com/user-attachments/assets/07043543-7666-48ab-bd76-5f658024427c)

Inicialmente, no encontramos ningún archivo relevante en los directorios del servidor FTP. Sin embargo, decidimos probar la carga de una **reverse shell** en PHP para ganar acceso al sistema.

### 3. **Subida de Reverse Shell en PHP**
Creamos una reverse shell en PHP y la subimos al servidor FTP:

```bash
put php-reverse-shell.php
```

Luego, iniciamos una escucha en nuestra máquina con **Netcat** en el puerto 444, que es el configurado en nuestra reverse shell:

```bash
nc -lnvp 444
```

### 4. **Ejecución de la Reverse Shell**
Accedemos al archivo de la reverse shell a través del navegador web en el servidor HTTP:

```
http://<IP>/php-reverse-shell.php
```

Esto nos permite obtener una shell inversa en nuestra máquina.

### 5. **Primera Flag: /home/Rijaba1/user.txt**
Con la shell establecida, navegamos al directorio **/home/Rijaba1/** y encontramos la primera flag en **user.txt**:

![Captura de pantalla 2024-10-10 124845](https://github.com/user-attachments/assets/81e72556-f604-4593-a7bf-8848d8b450c0)

### 6. **Escalada de Privilegios: Vim con Sudo**
Para escalar privilegios, ejecutamos el siguiente comando para verificar si podemos ejecutar alguna tarea con permisos de sudo:

![Captura de pantalla 2024-10-10 131048](https://github.com/user-attachments/assets/1332c799-c290-42bb-8120-17d5b5dcba12)


Descubrimos que podemos ejecutar **vim** como sudo. Aprovechamos esto para editar el archivo **/etc/passwd** y cambiar la contraseña de root.

### 7. **Modificación del Archivo /etc/passwd**
Generamos un hash de la contraseña "123456" con **openssl**:

```bash
openssl passwd -1 123456
```

Abrimos el archivo **/etc/passwd** usando `vim` con permisos de sudo:

```bash
sudo vim /etc/passwd
```

![Captura de pantalla 2024-10-10 131104](https://github.com/user-attachments/assets/eb0843df-35fc-4be9-8d92-056b0f082dfd)


Sustituimos la contraseña de root con el hash generado. Guardamos los cambios y luego cambiamos a la cuenta de root:

```bash
su root
```

### 8. **Búsqueda de root.txt**
Al acceder al directorio **/root**, encontramos un archivo llamado **root.txt**, pero al leerlo, el contenido es **"Not yet! Find root.txt"**.

![Captura de pantalla 2024-10-10 131346](https://github.com/user-attachments/assets/3113d0e1-3ca5-4590-9a99-66999d5a14da)


Utilizamos el comando `find` para buscar otro archivo **root.txt** en todo el sistema:

![Captura de pantalla 2024-10-10 131728](https://github.com/user-attachments/assets/af09fdf1-4fdc-4080-a1f8-f22a03700179)


Finalmente, encontramos el archivo correcto y obtenemos la flag final. ¡Máquina pwned!

---

### ¡Máquina completada!
