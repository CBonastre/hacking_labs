## Resolución de la máquina Fruits de Thehackerlabs

### 1. **Escaneo de Puertos con Nmap**
Comenzamos realizando un escaneo de puertos con `nmap` para identificar los servicios abiertos en la máquina objetivo.

```bash
nmap -sC -sV -oN nmap_scan.txt <IP>
```

El escaneo muestra que los puertos **22 (SSH)** y **80 (HTTP)** están abiertos.

![Captura de pantalla 2024-10-22 185917](https://github.com/user-attachments/assets/c7e0faef-a745-4d9c-8b3b-875b512b2674)

### 2. **Exploración del Servidor Web**
Accedemos al puerto **80**, donde encontramos un sitio web con un buscador. Observamos que las URLs contienen parámetros que podrían ser vulnerables a Local File Inclusion (LFI).

![image](https://github.com/user-attachments/assets/7ad1d8de-2c55-4287-9860-147f72add905)

### 3. **Pruebas de LFI**
Probamos la inyección de rutas comunes para tratar de visualizar archivos sensibles, pero no obtenemos resultados significativos. Sin embargo, decidimos buscar directorios y archivos adicionales en el servidor.

### 4. **Enumeración de Archivos con Gobuster**
Ejecutamos **Gobuster** para buscar archivos y directorios ocultos en el servidor web:

```bash
gobuster dir -u http://<IP> -w /usr/share/wordlists/dirb/common.txt
```

Encontramos un archivo interesante llamado **fruits.php**.

![image](https://github.com/user-attachments/assets/09b07f14-ceab-4005-8b08-336125479a9d)

### 5. **Explotación de LFI en fruits.php**
Accedemos al archivo `fruits.php` y probamos si es vulnerable a LFI:

```bash
http://<IP>/fruits.php?file=/etc/passwd
```

Logramos visualizar el contenido del archivo `/etc/passwd`, donde encontramos un usuario llamado **bananaman**.

![image](https://github.com/user-attachments/assets/22b1cddf-3376-4d74-b73f-1a7538f67af8)

### 6. **Ataque de Fuerza Bruta con Hydra**
Con el nombre de usuario **bananaman**, realizamos un ataque de fuerza bruta sobre el servicio SSH para encontrar la contraseña:

![image](https://github.com/user-attachments/assets/dbebc978-619b-46a3-a78e-72719af3a4ec)

Hydra nos revela que la contraseña es **celtic**.

![Captura de pantalla 2024-10-22 202631](https://github.com/user-attachments/assets/746402cd-4c77-474c-a827-abc6ed9272c1)

### 7. **Acceso a la Máquina por SSH**
Con las credenciales obtenidas, nos conectamos al servicio SSH:

![Captura de pantalla 2024-10-22 202615](https://github.com/user-attachments/assets/fdfb1fdd-fb22-46c7-9757-44b15f2587fc)

Una vez dentro, encontramos la primera flag en el sistema.

![image](https://github.com/user-attachments/assets/e49e4d36-c111-49b9-845c-2c5dee2c8f57)

### 8. **Escalada de Privilegios a Root**
Ejecutamos el comando `sudo -l` para ver qué comandos podemos ejecutar con privilegios elevados. Descubrimos que podemos ejecutar el comando **find** como root sin necesidad de contraseña. 

![Captura de pantalla 2024-10-22 202908](https://github.com/user-attachments/assets/2d820a91-5775-4e4b-a26e-c74fd0613b9b)

Consultamos **GTFOBins** y encontramos una forma de obtener una shell con el comando `find`:

![Captura de pantalla 2024-10-22 202915](https://github.com/user-attachments/assets/a7ccc60a-301c-4ec8-b7e9-f31b6a5c3d69)

Esto nos da acceso root.

![Captura de pantalla 2024-10-22 202929](https://github.com/user-attachments/assets/3eb84f1f-7ac2-4d87-901b-5e99ccd970d3)

### 9. **Búsqueda de la Flag Final**
Navegamos al directorio **/root** y encontramos la flag final:

![Captura de pantalla 2024-10-22 202958](https://github.com/user-attachments/assets/03451952-f548-460e-9d8b-b63963b9ef65)


### ¡Máquina completada!
