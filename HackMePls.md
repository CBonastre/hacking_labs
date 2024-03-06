# Guía Rápida para HackMePls de VulnHub
Aquí tienes la guía relámpago para conquistar la máquina virtual HackMePls en VulnHub.


## Escaneo y enumeración

### Dirección IP

**1-** En primer lugar al no conocer la dirección IP de la máquina víctima, lanzo un escaneo arp-scan sobre la interfaz de red "eth3", ya que es sobre la que esta conectada dicha máquina.

```
arp-scan -I eth3 --localnet --ignoredups
```

![Captura de pantalla 2024-03-06 160411](https://github.com/CBonastre/hacking_labs/assets/151465796/08ae2de8-0868-4dbd-9788-7ba012553075)


### Puertos

**2-** Una vez conozco la dirección IP sobre la que trabajar, procedo a lanzar un escaneo nmap con las siguientes opciones.

![Captura de pantalla 2024-03-06 162719](https://github.com/CBonastre/hacking_labs/assets/151465796/ff26abdc-70f7-4141-afb9-9947f8b6d629)


Tras finalizar el escaneo, observo que los puertos que se encuentran abiertos son el puerto 80 (http) y el puerto 3306 (mysql).

### Fuzzing WEB

**3-** Conociendo ya que puertos se encuentran abiertos procedo a acceder a la aplicación web para ver con que me encuentro. Reviso el código fuente, y veo un etiqueta "href" en html que parece interesante, "/js/main.js".

![Captura de pantalla 2024-03-06 165912](https://github.com/CBonastre/hacking_labs/assets/151465796/6a8821fa-be4b-48af-8e8e-b2f90c66491c)

**4-** Revisando el código encuentro el siguiente comentario:

![Captura de pantalla 2024-03-06 165958](https://github.com/CBonastre/hacking_labs/assets/151465796/f9fce7ad-f729-420a-9a35-92fc607f63c0)

**5-** Al acceder al subdirectorio, encuentro un panel de inicio a la plataforma de seeddms.

![Captura de pantalla 2024-03-06 170228](https://github.com/CBonastre/hacking_labs/assets/151465796/ef920372-efbf-4e15-b1b8-64ce4bf70688)

**6-** Como he encontrado este nuevo subdirectorio, voy a lanzar un gobuster primero a la url 10.40.2.42/seeddms/seeddms-5.1.22.

![Captura de pantalla 2024-03-06 170406](https://github.com/CBonastre/hacking_labs/assets/151465796/3f3419b2-78fe-4e94-8dde-101708cc222b)

**7-** No parece haber nada que me pueda dar información extra así que voy a probar con un gobuster a la url 10.40.2.42/seeddms.

![Captura de pantalla 2024-03-06 170648](https://github.com/CBonastre/hacking_labs/assets/151465796/c16ad2b0-ee13-4fad-8564-673ea55b8ee3)

**8-** En este caso no parece haber gran cosa interesante quitando un subdirectorio llamado conf.
Así que lanzo un gobuster a 10.40.2.42/seeddms/conf buscando archivos con las extensiones .php, .html, .txt, .xml para ver si encuentro algún archivo que me de algún tipo de información relevante acerca de la configuración del seeddms.

![Captura de pantalla 2024-03-06 170906](https://github.com/CBonastre/hacking_labs/assets/151465796/dcf3abfb-5d10-47ea-be36-3c8b0b96d71d)

**9-** Al terminar el escaneo, encuentro un archivo llamado setting.xml, y al acceder al él entre otros muchos datos que pueden ser relevantes, encuentro un campo de dbuser="seeddms" y otro de dbpass="seeddms".

![Captura de pantalla 2024-03-06 171111](https://github.com/CBonastre/hacking_labs/assets/151465796/542673a3-f157-4436-8451-ad4872bfa652)

## Explotación y conexión

### Conexión a MySQL

**1-** Con las credenciales encontadas pruebo de conectarme a la base de datos.

```
mysql -h 10.40.2.42 -u seeddms -pseeddms
```

![Captura de pantalla 2024-03-06 171156](https://github.com/CBonastre/hacking_labs/assets/151465796/2d6b78e6-5a93-41c9-9213-d7095e59b1e6)

Efectivamente son las credenciales de acceso y ya estoy dentro de la base de datos. 

**2-** Voy a listar las tablas de la base de datos para tratar de encontrar nombres de usuario y sus respectivas contraseñas.

![Captura de pantalla 2024-03-06 171306](https://github.com/CBonastre/hacking_labs/assets/151465796/7e142d82-9dcc-4c17-a655-518003cb7990)

![Captura de pantalla 2024-03-06 171321](https://github.com/CBonastre/hacking_labs/assets/151465796/963c77cf-a8b8-4a0b-b6ff-5ab54123e500)
![Captura de pantalla 2024-03-06 171335](https://github.com/CBonastre/hacking_labs/assets/151465796/292f809a-68fc-4b47-a0cc-c523fdf16ba3)


**3-** En la tabla users encuentro el usuario saket y su password.

![Captura de pantalla 2024-03-06 171403 1](https://github.com/CBonastre/hacking_labs/assets/151465796/23ceccee-6c14-4ee4-bf3b-141bc123ba8b)


**4-** Revisando que otras tablas pueden contener otros usuarios, encuentro una llamda tblUsers en la que hay un registro con el campo login con valor "admin" y en el campo pwd lo que parece ser un hash MD5, así que decido probar de cambiar el campo de pwd en la base de datos.
Para ello genero un nuevo hasd MD5 con la contraseña "admin", ya con el hash uso el siguiente comando para modificar la contraseña.

![Captura de pantalla 2024-03-06 171941](https://github.com/CBonastre/hacking_labs/assets/151465796/51bd1f53-59bc-43c1-b4f5-f8bb460e1564)



```
UPDATE tblUsers SET pwd="21232f297a57a5a743894a0e4a801fc3" WHERE id=1;
```



![Captura de pantalla 2024-03-06 171624](https://github.com/CBonastre/hacking_labs/assets/151465796/92ce28fd-7fa5-45ac-8525-46ba9f9073cc)


**5-** Ahora que ya conozco un nombre de usuario y que he conseguido modificarle la contraseña, vuelvo al portal de login de /seeddms/seeddms-5.1.22/ y inicio sesión con las credenciales.

![Captura de pantalla 2024-03-06 172056](https://github.com/CBonastre/hacking_labs/assets/151465796/fb004226-83f7-4d09-96ae-a6dbb4dd1f23)


### Explotar el sitio web


**6-** Una vez dentro del portal reviso un poco que pestañas y opciones tiene. En una de ellas veo que hay un apartado que permite subir archivos, el cual puedo aprovechar para subir una reverse shell en php. Descargo un archivo de reverse shell de la web de pentestmonkey, modifico los parámetros de la ip y al puerto al que quiero recibir la reverse shell. 
Ya configurado el archivo, lo subo desde el portal de seeddms. 

![Captura de pantalla 2024-03-06 172124](https://github.com/CBonastre/hacking_labs/assets/151465796/64887386-69ec-4ad5-8ac6-00c047898a06)

![Captura de pantalla 2024-03-06 172147](https://github.com/CBonastre/hacking_labs/assets/151465796/d892cb91-8b3d-49cf-8d5a-6a72867ab95e)


Ahora debo encontrar la ruta donde se ha almacenado el archivo para poder ejecutarlo.

**7-** Busco en internet y encuentro un reporte de una vulnerabilidad la cual me dice donde se encuentran los archivos subidos.

![Captura de pantalla 2024-03-06 173536](https://github.com/CBonastre/hacking_labs/assets/151465796/b4cb4a84-7db9-4fb4-b20d-e81b07248706)

![Captura de pantalla 2024-03-06 173523](https://github.com/CBonastre/hacking_labs/assets/151465796/e0b83505-ca33-44bc-a458-be0bc8dfcab8)


**8-** Como ya conozco la ruta para ejecutar el archivo me pongo en escucha para poder recibir la shell.

```
nc -lnvp 443
```

![Captura de pantalla 2024-03-06 172514](https://github.com/CBonastre/hacking_labs/assets/151465796/a820e090-fb2e-4e0a-b195-835a75b492d1)


**9-** Ejecuto el archivo desde la ruta en la que se encuentra y con el formato encontrado en el reporte de la CVE y recibo la shell.

![Captura de pantalla 2024-03-06 182724](https://github.com/CBonastre/hacking_labs/assets/151465796/8662540c-bcf0-4a64-a566-82d14f8668a9)

![Captura de pantalla 2024-03-06 172559](https://github.com/CBonastre/hacking_labs/assets/151465796/469a0dcc-1cb5-447e-8b9c-911e40136fd9)


## Escalada de privilegios

**1-** Ya dentro de la máquina veo que soy el usuario www-data, así que hago un cat /etc/passwd para ver que otros usuarios tiene el sistema.

![Captura de pantalla 2024-03-06 172636](https://github.com/CBonastre/hacking_labs/assets/151465796/882833ff-b461-4ce2-ba28-baca79bfcc5b)

![Captura de pantalla 2024-03-06 172657](https://github.com/CBonastre/hacking_labs/assets/151465796/507fba91-a37b-4eef-a6b3-67553b3cd2bc)


**2-** Veo que hay un usuario llamado **saket** que parece ser el mismo del cual había encontrado la contraseña dentro de la base de datos.

![Captura de pantalla 2024-03-06 171403 1](https://github.com/CBonastre/hacking_labs/assets/151465796/2bc273c3-3c1d-4b11-b512-4514e8de6608)

Conociendo las credenciales del usuario cambio al usuario **saket**.

![Captura de pantalla 2024-03-06 172843](https://github.com/CBonastre/hacking_labs/assets/151465796/7975cc1e-fe7f-4a92-80fb-75af44b41c43)


**3-** Llegado a este punto me interesa escalar privilegios, y conseguir ser el usuario root. Para ello ejecuto el comando el comando "sudo -l" y veo que siendo el usuario saket puedo ejecutar cualquier comando.

![Captura de pantalla 2024-03-06 173049](https://github.com/CBonastre/hacking_labs/assets/151465796/d8835523-9d52-4bf7-966b-4b2787e67b48)


**4-** Sabiendo que tengo permisos para ejecutar cualquier comando, cambio al usuario root.

![Captura de pantalla 2024-03-06 173147](https://github.com/CBonastre/hacking_labs/assets/151465796/0eacb349-8175-432f-9523-24138b631755)

