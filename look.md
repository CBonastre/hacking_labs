# INFORME resolución maquina vulnyx LOOK

Inicialmente, dado que no se dispone de la dirección IP de la máquina víctima, procedo a realizar un escaneo arp-scan a través de la interfaz de red "eth3", dado que dicha máquina está conectada a esta interfaz.

```
arp-scan -I eth3 --localnet --ignoredups
```

![Captura de pantalla 2024-03-18 202326](https://github.com/CBonastre/hacking_labs/assets/151465796/ac3a53e3-c6e9-4825-8da0-fff69caff030)


Una vez identificada la dirección IP sobre la cual trabajar, procedo a ejecutar un escaneo nmap con las siguientes opciones:

```
nmap -p- --open -sS -sC -sV --min-rate 2000 -n -Pn -vvv 10.40.2.45
```

![Captura de pantalla 2024-03-18 202604](https://github.com/CBonastre/hacking_labs/assets/151465796/7cf41404-5521-4eac-baa2-88b1073486af)


Tras la finalización del escaneo, se observa la apertura de los puertos 80 (HTTP) y 22 (SSH).

Con la información sobre los puertos abiertos, accedo a la aplicación web alojada en el puerto 80 para evaluar su contenido:

![Captura de pantalla 2024-03-18 202724](https://github.com/CBonastre/hacking_labs/assets/151465796/70b13606-0c9b-4d88-8227-b6a4f38643e1)


La página web muestra la configuración predeterminada de los servidores Apache, proporcionando escasa información útil. Ante esta situación, decido lanzar un escaneo con Gobuster para buscar posibles subdirectorios que puedan contener información relevante.

```
gobuster dir -u 10.40.2.45/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .php,.txt,.html
```

El escaneo revela un archivo llamado "look.php":

![Captura de pantalla 2024-03-18 203144](https://github.com/CBonastre/hacking_labs/assets/151465796/99e85991-019a-46cc-a3ef-88a7a2a76b05)


Al inspeccionar su contenido, no se encuentra información relevante:

![Captura de pantalla 2024-03-18 203633](https://github.com/CBonastre/hacking_labs/assets/151465796/8b7275b1-df4f-4e75-a043-6f3b3d5859e8)

Posteriormente, se revisa el archivo "info.php" obtenido del escaneo, donde se identifica la existencia de un usuario llamado "axel":

![Captura de pantalla 2024-03-18 204021](https://github.com/CBonastre/hacking_labs/assets/151465796/e241557c-a43c-4a59-847c-a1c984c0a773)


Con la información obtenida, se procede a intentar un ataque de fuerza bruta al puerto 22 (SSH).

El comando utilizado para llevar a cabo el ataque de fuerza bruta al puerto 22 (SSH) utilizando el usuario "axel" y el diccionario "rockyou.txt" es el siguiente:

```
hydra -l axel -P /home/kali/Escritorio/Diccionarios/Diccionaris2/rockyou.txt ssh://10.40.2.45
```

![Captura de pantalla 2024-03-18 211522](https://github.com/CBonastre/hacking_labs/assets/151465796/88c0565d-5650-4fc0-93a1-9a7f3c9f2096)


El ataque tiene éxito al encontrar la contraseña para el usuario "axel", lo que permite establecer una conexión SSH al servidor utilizando estas credenciales:

```
ssh axel@10.40.2.45
```

Se confirma la conexión exitosa al servidor mediante SSH con las credenciales obtenidas:

![Captura de pantalla 2024-03-18 211742](https://github.com/CBonastre/hacking_labs/assets/151465796/a1058587-e1e9-4d22-8597-01963e353475)


Ahora se puede acceder al directorio "/home/axel" y visualizar la primera flag.

![Captura de pantalla 2024-03-18 225338](https://github.com/CBonastre/hacking_labs/assets/151465796/99a8f11d-a497-4372-991d-e14cd0b64ee1)

Después de visualizar el archivo "passwd" en busca de otros usuarios, se identifican dos usuarios relevantes: "root" y "dylan". Se procede a buscar la contraseña para el usuario "dylan" explorando el sistema:

![Captura de pantalla 2024-03-18 231725](https://github.com/CBonastre/hacking_labs/assets/151465796/933ea7cf-e0ba-4a21-9689-a0899b14fbd8)


No se encuentra ningún archivo o directorio que proporcione esta información, por lo que se decide listar las variables de entorno en busca de datos relevantes:

![Captura de pantalla 2024-03-18 232032](https://github.com/CBonastre/hacking_labs/assets/151465796/89445e49-56cc-436c-8e7b-d35a79ab065a)


Se localiza la contraseña para el usuario "dylan" en las variables de entorno. Con esta información, se puede pivotar al usuario "dylan" y posteriormente intentar escalar privilegios hasta alcanzar el usuario "root".

![Captura de pantalla 2024-03-18 232232](https://github.com/CBonastre/hacking_labs/assets/151465796/fcb26308-66f4-4527-9f4d-aeeb1da53f8e)



Después de ejecutar el comando "sudo -l" para identificar cualquier binario que pueda ejecutarse con permisos del usuario "root", se encuentra que se puede ejecutar el binario "nokogiri" como usuario "root":

![Captura de pantalla 2024-03-18 232409](https://github.com/CBonastre/hacking_labs/assets/151465796/36a6e079-0965-41c1-bae0-e3b0fdc3ebb7)


Aprovechando la capacidad de ejecutar comandos mediante la función "system" del binario, se decide llamar a una shell, aprovechando el hecho de que se está ejecutando como usuario "root":

![Captura de pantalla 2024-03-18 233954](https://github.com/CBonastre/hacking_labs/assets/151465796/7403a2e1-918e-4806-ac8c-df2f914661ba)


Con éxito, ahora se tiene acceso como usuario "root" y se puede proceder a buscar el resto de las flags.

![Captura de pantalla 2024-03-18 234104](https://github.com/CBonastre/hacking_labs/assets/151465796/126798f7-4a56-4393-b2ce-74f516e0b421)

**¡Y con esto concluimos la resolución!🎉🚀💻**






