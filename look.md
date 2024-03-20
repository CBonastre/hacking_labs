# INFORME resolución maquina vulnyx LOOK

En primer lugar al no conocer la dirección IP de la máquina víctima, lanzo un escaneo arp-scan sobre la interfaz de red "eth3", ya que es sobre la que esta conectada dicha máquina.

```
arp-scan -I eth3 --localnet --ignoredups
```

![Captura de pantalla 2024-03-18 202326](https://github.com/CBonastre/hacking_labs/assets/151465796/ac3a53e3-c6e9-4825-8da0-fff69caff030)


Una vez conozco la dirección IP sobre la que trabajar, procedo a lanzar un escaneo nmap con las siguientes opciones.

```
nmap -p- --open -sS -sC -sV --min-rate 2000 -n -Pn -vvv 10.40.2.45
```

![Captura de pantalla 2024-03-18 202604](https://github.com/CBonastre/hacking_labs/assets/151465796/7cf41404-5521-4eac-baa2-88b1073486af)


Tras finalizar el escaneo, observo que los puertos que se encuentran abiertos son el puerto 80 (http) y el puerto 22 (ssh).

Conociendo ya que puertos se encuentran abiertos procedo a acceder a la aplicación web que corre por el puerto 80 para ver con que me encuentro.

![Captura de pantalla 2024-03-18 202724](https://github.com/CBonastre/hacking_labs/assets/151465796/70b13606-0c9b-4d88-8227-b6a4f38643e1)


Al acceder a la web se muestra la pagina por defecto que muestran los servidores Apache. No me da nada de información útil, así que voy a lanzar un gobuster para tratar de encontrar subdirectorios web que puedan darme mas información.

```
gobuster dir -u 10.40.2.45/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .php,.txt,.html
```

El escaneo me encuentra un archivo llamado "look.php".

![Captura de pantalla 2024-03-18 203144](https://github.com/CBonastre/hacking_labs/assets/151465796/99e85991-019a-46cc-a3ef-88a7a2a76b05)


Voy a probar de encontrar mas información en este archivo.

![Captura de pantalla 2024-03-18 203633](https://github.com/CBonastre/hacking_labs/assets/151465796/8b7275b1-df4f-4e75-a043-6f3b3d5859e8)


No parece haber información importante, así que voy a mirar en el archivo "info.php" que me ha mostrado el escaneo.

![Captura de pantalla 2024-03-18 204021](https://github.com/CBonastre/hacking_labs/assets/151465796/e241557c-a43c-4a59-847c-a1c984c0a773)


Aquí puedo ver que hay un usuario llamado "axel". Así que voy a intentar hacer un ataque de fuerza bruta al puerto 22 (ssh).

```
hydra -l axel -P /home/kali/Escritorio/Diccionarios/Diccionaris2/rockyou.txt ssh://10.40.2.45
```

![Captura de pantalla 2024-03-18 211522](https://github.com/CBonastre/hacking_labs/assets/151465796/88c0565d-5650-4fc0-93a1-9a7f3c9f2096)


Encuentra la contraseña para el usuario "axel", y me conecto al servidor mediante ssh con estas credenciales.

```
ssh axel@10.40.2.45
```

![Captura de pantalla 2024-03-18 211742](https://github.com/CBonastre/hacking_labs/assets/151465796/a1058587-e1e9-4d22-8597-01963e353475)


Ya puedo leer la primera flag que se encuentra en el directorio "/home/axel".

![Captura de pantalla 2024-03-18 225338](https://github.com/CBonastre/hacking_labs/assets/151465796/99a8f11d-a497-4372-991d-e14cd0b64ee1)

Visualizo el archivo "passwd", en busca de otros usuarios. Encuentro dos interesantes, obviamente el usuario "root", y otro usuario llamado "dylan". Así que voy a tratar de encontrar la contraseña para este usuario buscando por el sistema.

![Captura de pantalla 2024-03-18 231725](https://github.com/CBonastre/hacking_labs/assets/151465796/933ea7cf-e0ba-4a21-9689-a0899b14fbd8)


No encuentro ningún archivo o directorio que pueda darme esta información, así que voy a listar las variables de entorno por si hubiese alguna cosa que me pueda interesar.

![Captura de pantalla 2024-03-18 232032](https://github.com/CBonastre/hacking_labs/assets/151465796/89445e49-56cc-436c-8e7b-d35a79ab065a)


Encuentro la contraseña para el usuario "dylan". Ahora puedo pivotar al usuario "dylan" y desde ese usuario tratar de escalar hasta el usuario "root".

![Captura de pantalla 2024-03-18 232232](https://github.com/CBonastre/hacking_labs/assets/151465796/fcb26308-66f4-4527-9f4d-aeeb1da53f8e)


Ejecuto el comando "sudo -l" para encontrar algún  binario que pueda ejecutar con permisos del usuario "root" y así tratar de aprovecharlo para escalar privilegios.

![Captura de pantalla 2024-03-18 232409](https://github.com/CBonastre/hacking_labs/assets/151465796/36a6e079-0965-41c1-bae0-e3b0fdc3ebb7)


Puedo ejecutar el binario "nokogiri" como el usuario "root". Por lo visto puedo ejecutar comandos, con la función "system", así que voy a probar de llamar una shell y al estar ejecutando el binario como "root" deberia llamar a una shell de "root".

![Captura de pantalla 2024-03-18 233954](https://github.com/CBonastre/hacking_labs/assets/151465796/7403a2e1-918e-4806-ac8c-df2f914661ba)


Ahora ya soy "root", ya puedo buscar el resto de flags.

![Captura de pantalla 2024-03-18 234104](https://github.com/CBonastre/hacking_labs/assets/151465796/126798f7-4a56-4393-b2ce-74f516e0b421)

**¡Y con esto concluimos la resolución!🎉🚀💻**






