---
layout: post
title: "Migrando a easyengine v4"
featured-img: easyengine
categories: [Guides]
---

¡[easyengine](https://easyengine.io/) se ha actualizado y ahora funciona con docker!
Es una actualización muy potente, pero que cambia mucho la forma de trabajar (especialmente con la base de datos).

Empecemos con la migración a otro servidor paso a paso.

# Paso 1: Exportar el contenido y la base de datos.

Todo el contenido relevante de un sitio web wordpress (entradas, plugins, temas, imágenes...) está almacenado dentro del directorio **wp-content**.
Lo podemos comprimir utilizando *tar* fácilmente:

```
tar -zcvf ~/site_name.tar.gz ./wp-content
```

Es importante el directorio desde el que hacemos la compresión, ya que al descomprimir mantendrá todas los directorios tal cual se han comprimido.

Lo siguiente es exportar la BBDD. Es bastante fácil utilizando el usuario root, y ya que estamos, lo comprimimos:

```
sudo mysqldump -u root dbname > dbname.sql
tar -zcvf dbname.sql.tar.gz dbname.sql
```

# Paso 2: Copiar el contenido al servidor nuevo

Esto podemos hacerlo como más cómodo nos sea. Hay quien prefiere rsync, o ftp. Para mi lo más fácil es scp:
Desde el servidor nuevo podríamos hacer:

```
scp user@oldserver.com:~/*.tar.gz .
```

# Paso 3: Instalar easyengine y crear el sitio web

Como indican en la web de easyengine, su instalación es muy sencilla:
```
wget -qO ee rt.cx/ee4 && sudo bash ee
```

Cuando termine, podremos crear el sitio web con cache en una línea:

```
ee site create domain.com --type=wp --cache=wpredis
```

# Paso 4: Restauramos los datos
Empezamos descomprimiendo el contenido del sitio en su nuevo directorio y actualizando los permisos de la carpeta:

```
cd opt/easyengine/sites/domain.com/app/htdocs
tar -zxvf archive_name.tar.gz
chown -R www-data:www-data wp-content
```

Restaurar la BBDD tiene truco. Ya no podemos acceder a mysql como en el servidor anterior porque ahora es un contenedor de docker.
No sabemos cual es la contraseña de root configurada, pero podemos conectarnos al contenedor con algo de este estilo:

```
docker exec -i ee-global-db mysql -udbuser -pdbpass --database=dbname < dump.sql

docker exec -it f3c3a1fc727c bash -c 'mysql -uroot -p${MYSQL_ROOT_PASSWORD}'
```

# Paso 5: Activamos el SSL

Una vez hayamos cambiado el dominio y se haya propagado por los servidores de DNS, podremos activar el SSL con letsencrypt:
```
ee site update example.com --ssl=le

# O si queremos * subdominios:
# ee site update example.com --ssl=le --wildcard
```


[Fuente](https://venturedawn.com/blogging/migrate-wordpress-easyengine-v4/2667/).
