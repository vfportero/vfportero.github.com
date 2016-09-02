---
layout: post
title:  "Configurar Réplica Maestro – Esclavo en MySql Server"
date:   2011-10-07
categories: [MySql]
tags: [MySql, Bases de datos, réplica, sysadmin, tutorial]
---
Sé que este artículo se aleja un poco de lo que suelo escribir por aquí pero la clase magistral que me ha dado mi gran amigo [**Pedro Sanz**](https://twitter.com/psanzm "El Twitter de Pedro") sobre MySql bien se merece una entrada en el blog ! :-)

Hoy hemos tenido que montar para un cliente un servidor de **réplica de MySql** en el que era imperioso que los datos se **espejeran de forma inmediata entre el servidor maestro y el esclavo**.

Una **réplica** de MySql Server hace uso del **fichero binario de transacciones** para almacenar en el maestro todos los cambios reaizados (UPDATES, DELETES, CREATE, etc..) para que un **servidor externo lo lea y replique** exactamente los mismos cambios en su propia base de datos.

De este modo tenemos **uno o más servidores MySql esclavos** **haciendo las mismas transacciones que el maestro** para así tener los mismos datos en diferentes servidores cosa que se realmente útil y, no sólo ante caídas, ya que es posible configurar nuestra aplicación para compartir las SELECT entre distintos nodos de MySql y mejorar así el rendimiento.

Bueno, menos teoría y vamos **al lío**!.

Lo primero que debemos hacer es **configurar el servidor maestro para que almacene el log binario** y asignarle un identificador. Para ello editamos el **my.cnf** añadiendo (o editando si ya las tiene) las siguientes entradas:

```
#Identificador único del servidor maestro
server-id=1
#Nombre del fichero binario donde se almacenarán las transacciones
log-bin=mysql-bin
sync_binlog=1
#Tamaño del fichero de log tras lo que se truncara
max-binlog-size=500M
expire_logs_days=4
innodb_flush_log_at_trx_commit=1
```

Como siempre que modificamos un my.cnf hay que **reiniciar el servicio de MySql** para que acepte los cambios.

Luego tenemos que hacer lo propio con el (o los) esclavo(s). Modificar el my.cnf con los siguientes parámetros y luego reiniciar el servicio:

```
#Indentificador único del esclavo
server-id=2
relay-log=mysqld-relay-bin
max-relay-log-size=500M
relay_log_purge=1
```

**¡Muy bien!** Ya tenemos un servidor maestro y un esclavo pero ahora necesitamos **crear un usuario** para que el esclavo se conecte al maestro y pueda leer el log de transacciones. Para ello vamos a crear un nuevo usuario llamado “replicador” (el nombre y pass puede variar, jeje) en el master con privilegios de “**REPLICATION SLAVE**“:

```sql
CREATE USER replicador IDENTIFIED BY 'elpassword';
```

Y luego le damos los permisos de REPLICATION SLAVE:

```sql
GRANT REPLICATION SLAVE ON *.* TO 'replicador'@'%' IDENTIFIED BY 'elpassword';
FLUSH PRIVILEGES;
```

Ok! Ya tenemos los servidores bien configurados y el usuario que usaremos como replicador por lo que lo próximo que tenemos que hacer es **crear una copia inicial o “snapshot”** de la base de datos que queremos replicar para luego poder indicar al servidor esclavo desde dónde tiene que empezar a leer.

Para hacer el snapshot primero ejecutamos las siguientes consultas:

```sql
FLUSH TABLES;
FLUSH TABLES WITH READ LOCK;
SHOW MASTER STATUS;
```

Ten en cuenta que al hacer “**READ LOCK**” estamos bloqueando la tabla para que nadie cambie nada por lo que lo que viene a continuación deberíamos hacerlo lo más rápidamente posible.

El **SHOW MASTER STATUS** muestra dos valores que debemos anotar que son el “**File**” y “**Position**“. Necesitaremos indicarselos al servidor de réplica una vez hayamos cargado la copia inicial.

```
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000001 |     492 |              |                  |
+------------------+----------+--------------+------------------+
1 row in set (0.00 sec)
```

¡Muy bien! Vamos al servidor esclavo y lo terminamos de configurar.

Lo primero es **cargar una copia de seguridad** de base de datos que queremos replicar del master al esclavo (puedes usar el método que quieras, mysqldump, HeidSql, MySql Workbench…) y, una vez restaurada, **configurar el esclavo e iniciarlo.**

En este ejemplo vamos a suponer que el servidor maestro está alojado en 10.0.1.10\. Fíjate que tenemos que indicar el **File** y la **Position** que hemos obtenido antes del SHOW MASTER STATUS:

```sql
CHANGE MASTER TO MASTER_HOST='10.0.1.10', MASTER_USER='replicador', MASTER_PASSWORD='elpassword', MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=492, MASTER_PORT=3306;

START SLAVE;
```

Para terminar volvemos al maestro y desbloqueamos de nuevo las tablas para que puedan volver a editar datos:

```sql
UNLOCK TABLES;
```

**¡Ya está!** Si no ha pasado nada raro tendremos el servidor esclavo con la carga inicial funcionando y **todo cambio en el master se replicara por arte de magia.**

Si queremos saber el estado del servidor de réplica podemos usar la consulta:

```sql
SHOW SLAVE STATUS\G
```

Que nos mostrará un listado de datos. Yo miro el valor “**Seconds_Behind_Master**” que indica que “retraso” tiene el servidor esclavo respecto al maestro (si es NULL es que no va. Revisa el “Slave_IO_State” y “Last_Error”).

Ahora empieza a meter datos en la base de datos maestra y verás como ellos solitos aparecen en la esclava… ¡**brujería**!
Espero que esto os sea útil.

Nos vemos **Compilando**!!