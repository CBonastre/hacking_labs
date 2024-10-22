## Resolución de la máquina Microchoft de Thehackerlabs

### 1. **Escaneo de Vulnerabilidades con Nmap**
Comenzamos realizando un escaneo con `nmap` utilizando scripts para buscar vulnerabilidades en la máquina objetivo. Usamos el siguiente comando:

```bash
nmap -sV --script vuln <IP>
```

El escaneo identifica la vulnerabilidad **CVE-2017-0143** (EternalBlue).

![Captura de pantalla 2024-10-22 223310](https://github.com/user-attachments/assets/c0c83dd2-eae5-4d12-905a-a9d3604bd952)


### 2. **Investigación de la CVE en Metasploit**
Decidimos buscar el exploit correspondiente a la CVE en **Metasploit**. 

![Captura de pantalla 2024-10-22 230157](https://github.com/user-attachments/assets/61e8b7eb-4d2a-4326-bae0-647e2f22f0e2)

Encontramos varios resultados y seleccionamos el exploit número **0**.

![Captura de pantalla 2024-10-22 230301](https://github.com/user-attachments/assets/2fb8d9e5-373d-4e1f-91ce-15555f99355c)


### 3. **Configuración del Exploit**
Configuramos el exploit con la dirección IP de la máquina víctima:

![Captura de pantalla 2024-10-22 230335](https://github.com/user-attachments/assets/90c1e048-8884-4eb8-85bc-b0430ef9e167)


### 4. **Ejecutar el Exploit**
Ejecutamos el exploit y logramos obtener una sesión de **Meterpreter** en la máquina víctima:

![Captura de pantalla 2024-10-22 230353](https://github.com/user-attachments/assets/2d6a1e7f-7979-43f4-a5b0-84648145a038)


### 5. **Acceso a Shell**
Una vez dentro, ejecutamos el comando `shell` para lanzar una shell:

![Captura de pantalla 2024-10-22 230727](https://github.com/user-attachments/assets/acf22e30-b79b-4f47-a6df-369730569f95)


### 6. **Búsqueda de Flags**
Buscamos las flags en el sistema. La primera flag la encontramos en el directorio **/users/lola**:

![Captura de pantalla 2024-10-22 231004](https://github.com/user-attachments/assets/4967c359-93c1-490d-867d-2e729785b5ac)


La segunda flag la encontramos dentro del directorio **/users/admin**:

![Captura de pantalla 2024-10-22 231054](https://github.com/user-attachments/assets/ce85d49e-c3d0-413b-9052-83f0651242b6)


### ¡Máquina completada!
