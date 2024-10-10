## Resolución de la máquina Doctor de Vulnyx

### 1. **Escaneo de Puertos con Nmap**
Comenzamos realizando un escaneo con `nmap` para identificar los servicios abiertos en la máquina objetivo.
Identificamos que los puertos **SSH (22)** y **HTTP (80)** están abiertos.

![Captura de pantalla 2024-10-10 135202](https://github.com/user-attachments/assets/b20e0a3e-f588-4fc2-b2a8-c54fa1dd1b75)


### 2. **Acceso a la Web**
Accedemos al servidor web y comenzamos a navegar por sus pestañas. Notamos un indicio de que podría haber una vulnerabilidad de **Local File Inclusion (LFI)**.

![Captura de pantalla 2024-10-10 135650](https://github.com/user-attachments/assets/18bf1211-3984-4426-9cba-181ace3784e4)


### 3. **Comprobación de LFI**
Probamos a visualizar el archivo **/etc/passwd** para verificar la vulnerabilidad:

![Captura de pantalla 2024-10-10 135710](https://github.com/user-attachments/assets/034eb689-2405-40c4-9fbf-c1c4a72f1cd1)

Confirmamos que podemos visualizar el archivo y encontramos el usuario **admin**.

### 4. **Obtención de la Clave RSA**
Ahora, intentamos obtener la clave privada **id_rsa** del usuario **admin** en el directorio **/home/admin/.ssh/**:

![Captura de pantalla 2024-10-10 140015](https://github.com/user-attachments/assets/711f25b2-875c-47af-99fd-7727870f4370)

Al visualizar el archivo, logramos obtener la clave privada.

### 5. **Crackeo de la Clave RSA**
Utilizamos **John the Ripper** para crackear la clave RSA obtenida:

![Captura de pantalla 2024-10-10 140411](https://github.com/user-attachments/assets/200237a8-9d7b-4f96-8ec1-129c84c3350a)
![Captura de pantalla 2024-10-10 140423](https://github.com/user-attachments/assets/a2fa14d6-47e6-4ed1-9832-eb1810acceb4)

Una vez obtenida la contraseña, iniciamos sesión mediante SSH:

![Captura de pantalla 2024-10-10 143256](https://github.com/user-attachments/assets/28f2a052-b42c-4e61-bd93-52d4bc2143d2)

### 6. **Primera Flag: /home/admin/user.txt**
Dentro del sistema, encontramos la primera flag en **/home/admin/user.txt**:

![Captura de pantalla 2024-10-10 140843](https://github.com/user-attachments/assets/19478dc5-7414-495b-b825-33cb8cf48d43)

### 7. **Escalada de Privilegios: Verificación de Sudo**
Para escalar privilegios, ejecutamos el siguiente comando para verificar si podemos ejecutar tareas con permisos de sudo:

```bash
sudo -l
```
Además, realizamos un `find / -perm -4000 2>/dev/null` para buscar binarios con los que poder escalar privilegios, pero no encontramos ninguno.

### 8. **Modificación del Archivo /etc/passwd**
Dado que pudimos visualizar el archivo **/etc/passwd** anteriormente, decidimos intentar editarlo. Abrimos el archivo con permisos de sudo:

```bash
sudo nano /etc/passwd
```

Pudimos modificarlo, así que generamos un hash de la contraseña "123456" con **openssl**:

![Captura de pantalla 2024-10-10 141149](https://github.com/user-attachments/assets/b97dddc7-b606-4ccf-96f5-e1ba224cde67)

Sustituimos la contraseña de root con el hash generado y guardamos los cambios.

![Captura de pantalla 2024-10-10 141635](https://github.com/user-attachments/assets/52e432a8-a16c-40a2-b295-4b10a113f6d6)

### 9. **Acceso como Root**
Una vez modificado el archivo, cambiamos a la cuenta de root:

![Captura de pantalla 2024-10-10 141722](https://github.com/user-attachments/assets/4e8d1ae9-e315-4f27-bf86-c7e5a333684b)

### 10. **Búsqueda de la Flag Final: /root/root.txt**
Al acceder al directorio **/root**, encontramos la flag final:

![Captura de pantalla 2024-10-10 141814](https://github.com/user-attachments/assets/c524cc05-f07f-4439-aa9b-e059dd278b4c)

### ¡Máquina completada!
