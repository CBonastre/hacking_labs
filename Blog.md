## Resolución de la máquina Blog de Vulnyx

### 1. **Escaneo de Puertos con Nmap**
Comenzamos realizando un escaneo con `nmap` para identificar los servicios abiertos en la máquina. Usamos el siguiente comando:

![Captura de pantalla 2024-10-10 165523](https://github.com/user-attachments/assets/5a71977f-a580-43f2-a178-ac50e9831957)

Identificamos que los puertos **SSH (22)** y **HTTP (80)** están abiertos.

![Captura de pantalla 2024-10-10 165535](https://github.com/user-attachments/assets/f8120f18-3867-4582-a248-ee950a5433f5)

### 2. **Exploración del Servidor Web**
Accedemos al servidor web y observamos lo que parece ser la ejecución de un ping. No encontramos nada interesante en esta fase, por lo que decidimos usar **Gobuster** para encontrar subdirectorios ocultos:

![Captura de pantalla 2024-10-10 165953](https://github.com/user-attachments/assets/4f802c82-894e-45c0-bbaa-3fac1949f8db)

Encontramos un subdirectorio llamado **/my_weblog**, que contiene un blog perteneciente a un usuario llamado **admin**, lo que podría ser útil más adelante.

![Captura de pantalla 2024-10-10 165923](https://github.com/user-attachments/assets/b23ba602-29a1-4ba8-b3b6-29dfac5608e3)

### 3. **Búsqueda de Archivos en /my_weblog**
Ejecutamos nuevamente **Gobuster** sobre el subdirectorio **/my_weblog** para descubrir más contenido oculto:

![Captura de pantalla 2024-10-10 170206](https://github.com/user-attachments/assets/30f02502-c666-4504-9ff4-61707858f5bd)

Descubrimos un archivo **admin.php**, que al acceder muestra un portal de inicio de sesión de **Nibbleblog**, un CMS.

![Captura de pantalla 2024-10-10 170357](https://github.com/user-attachments/assets/326900f3-3701-4ff1-8dae-1a94ce8393ea)

### 4. **Ataque de Fuerza Bruta con Hydra**
Decidimos realizar un ataque de fuerza bruta sobre el portal de inicio de sesión utilizando **Hydra** para encontrar la contraseña del usuario **admin**:

![image](https://github.com/user-attachments/assets/7c59b766-0637-4974-8bdf-5025ad7cb032)

Hydra nos revela que la contraseña es **kisses**.

![Captura de pantalla 2024-10-10 184052](https://github.com/user-attachments/assets/1364130a-4a36-4a1e-b230-3d61ea17e842)

### 5. **Acceso al Panel de Administración**
Con las credenciales obtenidas, accedemos al panel de administración de Nibbleblog como **admin**.

![Captura de pantalla 2024-10-10 184450](https://github.com/user-attachments/assets/60401fcd-d076-4d3a-af18-eec87162fd11)

### 6. **Explotación de Vulnerabilidad en Plugin de Imágenes**
Investigamos vulnerabilidades conocidas del CMS Nibbleblog y encontramos una relacionada con el plugin de imágenes, que permite subir archivos PHP renombrados como imágenes. 

![Captura de pantalla 2024-10-10 172637](https://github.com/user-attachments/assets/8968e697-5201-4912-bfea-0399cb741999)

Decidimos aprovechar esto para subir una reverse shell en PHP con el nombre `image.php`:

![Captura de pantalla 2024-10-10 184513](https://github.com/user-attachments/assets/ede5cdcb-4421-463c-a68c-36a174023556)

Nos ponemos en escucha en nuestra máquina con **Netcat**:

```bash
nc -lnvp 444
```

Accedemos al archivo subido a través del navegador (conocemos la ruta gracias a la descripción de la vulnerabilidad) y obtenemos una shell reversa.

![Captura de pantalla 2024-10-10 174032](https://github.com/user-attachments/assets/b5680105-9cf8-4fdb-b3e7-093410ed7bfd)

### 7. **Escalada Horizontal a Admin**
Estamos dentro del sistema como el usuario **www-data**. Utilizamos el comando `sudo -l` para verificar qué privilegios podemos usar sin contraseña.
Descubrimos que podemos ejecutar **git** como el usuario **admin** sin necesidad de una contraseña. Aprovechamos esto para cambiar a **admin**:

![Captura de pantalla 2024-10-10 174540](https://github.com/user-attachments/assets/d31bbd16-04a0-470d-aeb4-cf45e3059d0a)

### 8. **Primera Flag: /home/admin/user.txt**
Una vez en el sistema como **admin**, encontramos la primera flag en el directorio **/home/admin/user.txt**:

![Captura de pantalla 2024-10-10 174607](https://github.com/user-attachments/assets/94c0738f-3898-4b96-9bf7-a10d616ebce2)

### 9. **Escalada de Privilegios a Root**
Volvemos a utilizar el comando `sudo -l` para ver qué más podemos ejecutar. 

![Captura de pantalla 2024-10-10 175023](https://github.com/user-attachments/assets/0c24590f-fe18-4aff-bdf4-e15c9c7a9f2c)

Vemos que podemos usar **mcedit** como **root** sin contraseña:

```bash
sudo mcedit
```

Exploramos la interfaz de **mcedit** y encontramos una opción que nos permite ejecutar una shell. 

![Captura de pantalla 2024-10-10 174821](https://github.com/user-attachments/assets/ec3bddff-f018-4c96-838c-b3ae6548c3fe)

Ejecutamos esa opción y obtenemos acceso como **root**.

![Captura de pantalla 2024-10-10 174858](https://github.com/user-attachments/assets/01426e16-cb15-4515-a34b-2a869db03b06)

### 10. **Búsqueda de la Flag Final: /root/root.txt**
Finalmente, navegamos al directorio **/root** y encontramos la flag final:

![Captura de pantalla 2024-10-10 174944](https://github.com/user-attachments/assets/cfafb784-cd78-4795-9142-e52418945ac4)


### ¡Máquina completada!
