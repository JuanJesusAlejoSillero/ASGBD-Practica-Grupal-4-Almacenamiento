# **ASGBD - Práctica Grupal 4: Almacenamiento**

## **Alumno 2**

**Tabla de contenidos:**

- [**ASGBD - Práctica Grupal 4: Almacenamiento**](#asgbd---práctica-grupal-4-almacenamiento)
  - [**Alumno 2**](#alumno-2)
  - [**Oracle**](#oracle)
    - [**Ejercicio 1**](#ejercicio-1)
    - [**Ejercicio 2**](#ejercicio-2)
    - [**Ejercicio 3**](#ejercicio-3)
    - [**Ejercicio 4**](#ejercicio-4)
    - [**Ejercicio 5**](#ejercicio-5)
    - [**Ejercicio 6**](#ejercicio-6)
  - [**PostgreSQL**](#postgresql)
    - [**Ejercicio 7**](#ejercicio-7)
  - [**MySQL**](#mysql)
    - [**Ejercicio 8**](#ejercicio-8)
  - [**MongoDB**](#mongodb)
    - [**Ejercicio 9**](#ejercicio-9)

---

## **Oracle**

### **Ejercicio 1**

> **1. Establece que los objetos que se creen en el TS1 (creado por Alumno 1) tengan un tamaño inicial de 200K, y que cada extensión sea del doble del tamaño que la anterior. El número máximo de extensiones debe ser de 3.**

1. Creamos el TS1 creado por mi compañero Juanje.

    ```txt
    SQL> CREATE TABLESPACE TS1
    DATAFILE 'TS1.dbf'
    SIZE 2M
    AUTOEXTEND ON
    EXTENT MANAGEMENT LOCAL;  2    3    4    5  

    Tablespace creado.
    ```

2. Realizamos la modificación.

    ```txt
    SQL> ALTER TABLESPACE TS1
        DEFAULT STORAGE (
           INITIAL 200K
           NEXT 400K
           MINEXTENTS 1
           MAXEXTENTS 3);  2    3    4    5    6  
    ALTER TABLESPACE TS1
    *
    ERROR en linea 1:
    ORA-25143: la clausula de almacenamiento por defecto no es compatible con la
    politica de asignacion
    ```

    Como vemos nos ha dado un fallo, eso es debido a que no podemos cambiar el tamaño inicial de la extensión una vez se ha creado el tablespace.

3. Borramos el tablespace TS1.

    ```txt
    SQL> DROP TABLESPACE TS1;

    Tablespace borrado.
    ```

4. Creamos el nuevo TS1.

    ```txt
    SQL> CREATE TABLESPACE TS1
      2  DATAFILE 'ts1.dbf'
      3  SIZE 2M
      4  DEFAULT STORAGE (
      5  INITIAL 200K
      6  NEXT 400K
      7  MINEXTENTS 1
      8  MAXEXTENTS 3);

    Tablespace creado.
    ```

### **Ejercicio 2**

> **2. Crea dos tablas en el tablespace recién creado e inserta un registro en cada una de ellas. Comprueba el espacio libre existente en el tablespace. Borra una de las tablas y comprueba si ha aumentado el espacio disponible en el tablespace. Explica la razón.**

1. Creamos las 2 tablas en el tablespace TS1.

    ```txt
    SQL> CREATE TABLE my_table1 (
      2  id NUMBER,
      3  my_blob BLOB,
      4  my_clob CLOB,
      5  my_char CHAR(1000) 
      6  ) TABLESPACE TS1;

    Tabla creada.

    SQL> CREATE TABLE my_table2 (
          id NUMBER,
          my_blob BLOB,
          my_clob CLOB,
          my_char CHAR(1000)
        ) TABLESPACE TS1;  2    3    4    5    6  

    Tabla creada.
    ```

2. Insertamos un registro en cada tabla.

    ```txt
    SQL> INSERT INTO my_table1 (id, my_blob, my_clob, my_char) VALUES (1, HEXTORAW  ('1f1e1d1c1b1a191817161514131211100f0e0d0c0b0a09080706050403020100'), 'CLOB data 1', 'Char    data 1');

    1 fila creada.

    SQL> INSERT INTO my_table2 (id, my_blob, my_clob, my_char) VALUES (1, HEXTORAW  ('1f1e1d1c1b1a191817161514131211100f0e0d0c0b0a09080706050403020100'), 'CLOB data 1', 'Char    data 1');

    1 fila creada.
    ```

3. Comprobamos el espacio libre del tablespace.

    ```txt
    SQL> SELECT BYTES FROM DBA_FREE_SPACE WHERE TABLESPACE_NAME='TS1';

         BYTES
    ----------
        131072
    ```

4. Borramos una tabla.

    ```txt
    SQL> DROP TABLE my_table2;

    Tabla borrada.
    ```

5. Vuelvo a comprobar el espacio libre del tablespace:

    ```txt
    SQL> SELECT BYTES FROM DBA_FREE_SPACE WHERE TABLESPACE_NAME='TS1';

         BYTES
    ----------
        131072
         65536
        131072
         65536
        131072
         65536

    6 filas seleccionadas.
    ```

### **Ejercicio 3**

> **3. Convierte a TS1 en un tablespace de sólo lectura. Intenta insertar registros en la tabla existente. ¿Qué ocurre?. Intenta ahora borrar la tabla. ¿Qué ocurre? ¿Porqué crees que pasa eso?**

1. Lo convertimos a modo lectura.

    ```txt
    SQL> ALTER TABLESPACE TS1 READ ONLY;

    Tablespace modificado.
    ```

2. Insertamos un registro.

    ```txt
    SQL> INSERT INTO my_table1 (id, my_blob, my_clob, my_char) VALUES (1, HEXTORAW      ('1f1e1d1c1b1a191817161514131211100f0e0d0c0b0a09080706050403020100'), 'CLOB data 1',    'Char    data 1');
    INSERT INTO my_table1 (id, my_blob, my_clob, my_char) VALUES (1, HEXTORAW   ('1f1e1d1c1b1a191817161514131211100f0e0d0c0b0a09080706050403020100'), 'CLOB data 1',     'Char	 data 1')
                *
    ERROR en linea 1:
    ORA-00372: el archivo 17 no se puede modificar en este momento
    ORA-01110: archivo de datos 17: '/opt/oracle/product/19c/dbhome_1/dbs/ts1.dbf'
    ```

    Como vemos, no ha podido realizar el insert ya que el tablespace está en modo lectura y no puede realizar operaciones de escritura.

3. Borramos la tabla my_table.

    ```txt
    SQL> DROP TABLE my_table;

    Tabla borrada.
    ```

    Si el usuario tiene permisos para borrar tablas, lo puede hacer. Esto es debido a que aunque el tablespace esté en solo lectura, esto no impide que se puedan realizar operaciones sobre los objetos almacenados en el tablespace. Unicamente no se puede escribir, como no hemos indicado nada de protección contra borrado pues lo puede realizar.

### **Ejercicio 4**

> **4. Crea un espacio de tablas TS2 con dos ficheros en rutas diferentes de 1M cada uno no autoextensibles. Crea en el citado tablespace una tabla con la clausula de almacenamiento que quieras. Inserta registros hasta que se llene el tablespace. ¿Qué ocurre?**

1. Lo primero que haremos será crear el tablespace TS2.

    ```txt
    SQL> CREATE TABLESPACE TS2 DATAFILE '/opt/oracle/oradata/ORCLCDB/ts2.dbf' SIZE 1M, '/opt/oracle/oradata/ts2.dbf' SIZE 1M AUTOEXTEND OFF;

    Tablespace creado.
    ```

2. Creamos una tabla en el tablespace TS2.

    La tabla creada es la siguiente:

    ```txt
    CREATE TABLE my_table (
      id NUMBER,
      my_blob BLOB,
      my_clob CLOB,
      my_char CHAR(1000)
    ) TABLESPACE TS2;
    ```

3. Insertamos registros en la tabla hasta llenar el tablespace:

    Como la tabla no tiene ningún tipo de restricción y es meramente de pruebas, he usado un procedimiento que inserta el mismo registro 700 veces para llenar el tablespace:

    ```txt
    CREATE OR REPLACE PROCEDURE add_707_insert
    AS
    BEGIN
        FOR i IN 1..707 LOOP
            INSERT INTO my_table (id, my_blob, my_clob, my_char) VALUES (1, HEXTORAW('1f1e1d1c1b1a191817161514131211100f0e0d0c0b0a09080706050403020100'), 'CLOB data 1', 'Char data 1');
        END LOOP;
    END;
    /
    ```

4. Comprobamos el fallo al llenar el tablespace.

    ```txt
    SQL> INSERT INTO my_table (id, my_blob, my_clob, my_char) VALUES (1, HEXTORAW('1f1e1d1c1b1a191817161514131211100f0e0d0c0b0a09080706050403020100'), 'CLOB data 1', 'Char data 1');

    1 fila creada.

    SQL> INSERT INTO my_table (id, my_blob, my_clob, my_char) VALUES (1, HEXTORAW('1f1e1d1c1b1a191817161514131211100f0e0d0c0b0a09080706050403020100'), 'CLOB data 1', 'Char data 1');
    	    INSERT INTO my_table (id, my_blob, my_clob, my_char) VALUES (1, HEXTORAW('1f1e1d1c1b1a191817161514131211100f0e0d0c0b0a09080706050403020100'), 'CLOB data 1', 'Char data 1')
                            *
    ERROR en linea 1:
    ORA-01653: no se ha podido ampliar la tabla SYS.MY_TABLE con 128 en el
    tablespace TS2
    ```

### **Ejercicio 5**

> **5. Hacer un procedimiento llamado MostrarUsuariosporTablespace que muestre por pantalla un listado de los tablespaces existentes con la lista de usuarios que tienen asignado cada uno de ellos por defecto y el número de los mismos, así:**

```txt
Tablespace xxxx:
    Usr1
    Usr2
    ...
Total Usuarios Tablespace xxxx: n1
Tablespace yyyy:
    Usr1
    Usr2
    ...
Total Usuarios Tablespace yyyy: n2
....
Total Usuarios BD: nn
```

**No olvides incluir los tablespaces temporales y de undo.**

1. Nos conectamos como sysdba y habilitamos que imprima texto por pantalla:

    ```txt
    sqlplus / as sysdba
    set serveroutput on
    ```

2. Creamos los módulos de programación necesarios para la correcta realización del listado.

    Adjunto los procedimientos realizado:

    ```sql
    CREATE OR REPLACE PROCEDURE MOSTRARUSUARIOS(V_TABLESPACENAME VARCHAR2,V_TOTAL IN OUT NUMBER)
    IS 
        V_USER VARCHAR2(50);
        V_CONT NUMBER:=0; 
        CURSOR C_USERS IS SELECT USERNAME 
        FROM DBA_USERS 
        WHERE DEFAULT_TABLESPACE=V_TABLESPACENAME; 
        V_USERS C_USERS%ROWTYPE; 
    BEGIN 
        DBMS_OUTPUT.PUT_LINE('TABLESPACE ' || V_TABLESPACENAME||': '); 
            FOR V_USERS IN C_USERS LOOP 
            V_USER:=V_USERS.USERNAME; 
            V_CONT:=V_CONT + 1; 
            DBMS_OUTPUT.PUT_LINE(RPAD('-',6)||V_USER); 
        END LOOP; 
        V_TOTAL:=V_TOTAL + V_CONT; 
        DBMS_OUTPUT.PUT_LINE('TOTAL USUARIOS TABLESPACE '||V_TABLESPACENAME||': '||V_CONT); 
        DBMS_OUTPUT.PUT_LINE(' '); 
    END; 
    /

    CREATE OR REPLACE PROCEDURE MOSTRARUSUARIOSPORTABLESPACE 
    IS 
        V_TABLESPACENAME VARCHAR2(50); 
        V_TOTAL NUMBER:=0; 
        CURSOR C_CURSOR IS 
        SELECT TABLESPACE_NAME 
        FROM DBA_TABLESPACES; 
        V_CURSOR C_CURSOR%ROWTYPE; 
    BEGIN 
        FOR V_CURSOR IN C_CURSOR LOOP 
            V_TABLESPACENAME:=V_CURSOR.TABLESPACE_NAME; 
            MOSTRARUSUARIOS(V_TABLESPACENAME,V_TOTAL); 
        END LOOP; 
        DBMS_OUTPUT.PUT_LINE('TOTAL USUARIOS BD: '||V_TOTAL); 
    END; 
    /
    ```

3. Realizo las comprobaciones pertinentes.

    ```txt
    SQL> exec MOSTRARUSUARIOSPORTABLESPACE
    TABLESPACE SYSTEM:
    -     SYS
    -     SYSTEM
    -     XS$NULL
    -     OJVMSYS
    -     LBACSYS
    -     OUTLN
    -     SYS$UMF
    TOTAL USUARIOS TABLESPACE SYSTEM: 7
    TABLESPACE SYSAUX:
    -     DBSNMP
    -     APPQOSSYS
    -     DBSFWUSER
    -     GGSYS
    -     ANONYMOUS
    -     CTXSYS
    -     DVSYS
    -     DVF
    -     GSMADMIN_INTERNAL
    -     MDSYS
    -     OLAPSYS
    -     XDB
    -     WMSYS
    TOTAL USUARIOS TABLESPACE SYSAUX: 13
    TABLESPACE UNDOTBS1:
    TOTAL USUARIOS TABLESPACE UNDOTBS1: 0
    TABLESPACE TEMP:
    TOTAL USUARIOS TABLESPACE TEMP: 0
    TABLESPACE USERS:
    -     GSMCATUSER
    -     MDDATA
    -     SYSBACKUP
    -     REMOTE_SCHEDULER_AGENT
    -     GSMUSER
    -     SYSRAC
    -     GSMROOTUSER
    -     ALEMD
    -     SI_INFORMTN_SCHEMA
    -     AEROPUERTO
    -     AUDSYS
    -     DIP
    -     ORDPLUGINS
    -     EJ7PASSWD
    -     SYSKM
    -     ORDDATA
    -     ORACLE_OCM
    -     C##SERVIDOR1
    -     SCOTT
    -     SYSDG
    -     ORDSYS
    -     RAUL
    TOTAL USUARIOS TABLESPACE USERS: 22
    TABLESPACE TA_INDICES:
    TOTAL USUARIOS TABLESPACE TA_INDICES: 0
    TABLESPACE TS2:
    TOTAL USUARIOS TABLESPACE TS2: 0
    TOTAL USUARIOS BD: 42

    Procedimiento PL/SQL terminado correctamente.
    ```

### **Ejercicio 6**

> **6. Realiza un procedimiento llamado MostrarDetallesIndices que reciba el nombre de una tabla y muestre los detalles sobre los índices que hay definidos sobre las columnas de la misma.**

1. Nos conectamos como sysdba y habilitamos que imprima texto por pantalla:

    ```txt
    sqlplus / as sysdba
    set serveroutput on
    ```

2. Creamos los módulos de programación necesarios para la correcta realización del procedimiento.

    ```sql
    CREATE OR REPLACE PROCEDURE MOSTRARDETALLESINDICES(P_TABLE VARCHAR2)
    IS 
        V_INDEXNAME VARCHAR2(50); 
        V_TABLESPACENAME VARCHAR2(50); 
        V_OWNER VARCHAR2(50);
        V_NOMBRETABLA VARCHAR2(100);
        CURSOR C_CURSOR IS SELECT INDEX_NAME, TABLE_NAME, TABLESPACE_NAME, OWNER
        FROM DBA_INDEXES 
        WHERE TABLE_NAME=P_TABLE; 
        V_CURSOR C_CURSOR%ROWTYPE; 
    BEGIN 
        FOR V_CURSOR IN C_CURSOR LOOP 
            V_INDEXNAME:=V_CURSOR.INDEX_NAME; 
            V_TABLESPACENAME:=V_CURSOR.TABLESPACE_NAME; 
            V_OWNER:=V_CURSOR.OWNER;
        END LOOP;
        DBMS_OUTPUT.PUT_LINE('NOMBRE DE LA TABLA: '||P_TABLE||' NOMBRE DEL INDICE: '||V_INDEXNAME||' NOMBRE DEL TABLESPACE: '||V_TABLESPACENAME||' PROPIETARIO: '||V_OWNER);
    END; 
    /
    ```

3. Ejecutamos el procedimiento.

    ```txt
    SQL> EXEC MOSTRARDETALLESINDICES('MY_TABLE');
    NOMBRE DE LA TABLA: MY_TABLE NOMBRE DEL INDICE: SYS_IL0000077067C00003$$ NOMBRE
    DEL TABLESPACE: TS2 PROPIETARIO: SYS

    Procedimiento PL/SQL terminado correctamente.

    ```

## **PostgreSQL**

### **Ejercicio 7**

> **7. Averigua si existe el concepto de segmento y el de extensión en Postgres, en qué consiste y las diferencias con los conceptos correspondientes de ORACLE.**

1. Segmentos en Postgres y diferencias con Oracle.

    **Definición.**

    En PostgreSQL existe el concepto de segmento, pero el término se utiliza en diferentes contextos y con diferentes significados, dependiendo del área de la base de datos en la que se esté trabajando.  Un segmento se refiere a una porción de datos de una tabla particionada que se almacena en un conjunto específico de particiones. Los segmentos pueden ser definidos por el usuario para controlar cómo se distribuyen los datos en las particiones y mejorar el rendimiento de las consultas.

    Por otro lado, en PostgreSQL también utiliza el concepto de segmento en la gestión del almacenamiento, PostgreSQL divide los archivos de la base de datos en piezas más pequeñas llamadas "bloques". Cada segmento de archivo contiene varios bloques de datos y pueden ser asignado a un proceso específico para su acceso. La gestión de estos segmentos la realiza automáticamente PostgreSQL, y por regla general, no es necesario que el usuario tenga que interactuar con ellos directamente.

    **Diferencias con Oracle.**

    En Oracle, los segmentos se utilizan principalmente para la gestión d e almacenamiento y se definen como una porción lógica de una tabla o índice que se asigna a un área de almacenamiento específica en disco. Cada segmento puede ser asignado a un tablespace. Los segmentos de Oracle pueden incluir tablas, índices, particiones y otros objetos de la base de datos.

    Mientras que en PostgreSQL, los segmentos también se utilizan para la gestión de almacenamiento, pero se dividen en bloques, que son la unidad básica de almacenamiento de datos en el sistema. Los segmentos en PostgreSQL también se utilizan para particionar tablas como vimos en su definción. 

    Otra diferencia es qie en Oracle, los segmentos pueden ser asignados a diferentes tablespaces y pueden moverse de un tablespace a otro, muentras que en PostgreSQL, los segmentos de archivos están asociados a un directorio de almacenamiento específico y no se pueden mover de manera independiente.

    Resumiendo, aunque ambos sistemas usen el término de segmento, tienen diferentes enfoques en el uso y gestión. En Oracle, los segmentos son usados para la gestión de almacenamiento y se pueden mover entre tablespaces mientras que en PostgreSQL, los segmentos se utilizan para la gestión de almacenamiento y la partición de tablas, y no se pueden mover de tablespaces ya que están asociados a un directorio de almacenamiento específico.

2. Extensiones en Postgres y diferencias con Oracle.

    **Definición.**

    En PostgreSQL, también existe el concepto de extensión. Una extensión es un componente que proporciona una funcionalidad específica a PostgreSQL. Las extensiones se utilizan para ampliar las capacidades de la base de datos, permitiendo a los usuarios agregar características y herramientas adicionales que no se incluyen en la distribución principal de PostgreSQL.

    Las extensiones se pueden desarrollar e instalar en una base de datos de PostgreSQL. Una vez instalada una extensión, se puede utilizar la funcionalidad adicional que proporciona en la base de datos.

    Las extensiones se utilizan para ampliar la funcionalidad de la base de datos en áreas como la administración de bases de datos, la seguridad, la gestión de almacenamiento, la optimización del rendimiento y la compatibilidad con otros sistemas de bases de datos.

    Si ejecutamos la siguiente consulta, vemos cuales son las extensión que tenemos habilitadas en nuestro Servidor y que podemos instalar para hacer uso de ellas.

    ```sql
    postgres=#  SELECT * FROM pg_available_extensions ORDER BY name;
    ```

    ![Alumno 2 - PostgreSQL - 1](img/Alumno%202/PostgreSQL/1.png)

    Las extensiones se pueden instalar utilizando el comando "CREATE EXTENSION" en la consola de comandos de PostgreSQL o mediante herramientas de administración como pgAdmin. Una vez instalada una extensión, se puede utilizar la funcionalidad adicional que proporciona en la base de datos.

    **Diferencias con Oracle.**

    En Oracle, el concepto de extensión es un poco diferente al de PostgreSQL. En Oracle, una extensión se refiere a una funcionalidad adicional que se instala en un esquema de base de datos específico. Las extensiones en Oracle son componentes opcionales que se utilizan para agregar características y herramientas adicionales a la base de datos.

    A diferencia de PostgreSQL, las extensiones en Oracle no se pueden instalar en toda la base de datos, sino que se deben instalar en un esquema de base de datos específico. Además, en Oracle, las extensiones se denominan también "paquetes" o "bibliotecas", que se instalan en un esquema de base de datos mediante el comando "CREATE PACKAGE" o "CREATE LIBRARY".

    Los paquetes en Oracle se utilizan para definir objetos y métodos que se pueden llamar desde otros objetos en la base de datos. Los paquetes pueden contener procedimientos, funciones, tipos de datos y variables que se pueden utilizar en otras partes de la base de datos.

    En resumen, aunque tanto PostgreSQL como Oracle tienen el concepto de extensiones, hay algunas diferencias significativas en cómo se utilizan. En PostgreSQL, las extensiones se utilizan para proporcionar funcionalidades adicionales a la base de datos en general, mientras que en Oracle, las extensiones se instalan en esquemas de base de datos específicos y se utilizan para definir objetos y métodos que se pueden llamar desde otros objetos en la base de datos.

## **MySQL**

### **Ejercicio 8**

> **8. Averigua si existe el concepto de espacio de tablas en MySQL y las diferencias con los tablespaces de ORACLE.**

1. Definición de concepto de tablespace en MySQL.

    En MySQL, el espacio de tablas se refiere al espacio físico que se asigna a una tabla en el disco duro o en otro tipo de almacenamiento persistente. El espacio de tablas es importante porque afecta directamente el rendimiento y la capacidad de almacenamiento de la base de datos.

    MySQL almacena los datos en archivos separados en el sistema de archivos del sistema operativo. Cada tabla de MySQL tiene al menos dos archivos asociados:

    - El archivo ".frm" que contiene la definición de la tabla, como su estructura de campos, índices y restricciones.

    - El archivo de datos ".ibd" que almacena los datos de la tabla.

    El tamaño del archivo de datos ".ibd" representa el tamaño total del espacio de tablas utilizado por una tabla. El tamaño de los índices también puede aumentar el espacio de tablas.

    MySQL proporciona varias herramientas y comandos para gestionar y monitorizar el espacio de tablas, como:

    - El comando "SHOW TABLE STATUS" que muestra información detallada sobre el estado de una tabla, incluyendo su tamaño actual en disco y su tamaño máximo.

    - La herramienta "mysqlcheck" que se utiliza para optimizar, reparar y analizar las tablas de la base de datos, y también puede ayudar a reducir el espacio de tablas.

    - El motor de almacenamiento InnoDB, que es el motor de almacenamiento predeterminado en MySQL, proporciona características avanzadas para gestionar el espacio de tablas, como la compresión de datos y la asignación de espacio dinámico.

    En resumen, el espacio de tablas en MySQL es el espacio físico asignado a una tabla en el disco duro o en otro tipo de almacenamiento persistente. Es importante gestionar y monitorizar el espacio de tablas para garantizar el rendimiento y la capacidad de almacenamiento de la base de datos.

2. Diferencias con los tablespaces de Oracle.

    En Oracle, el concepto de tablespaces es un poco diferente al de MySQL. En Oracle, un tablespace es un conjunto lógico de uno o más archivos de datos que se utilizan para almacenar los datos de la base de datos. Cada tablespace puede contener una o más tablas, índices y otros objetos de la base de datos.

    A diferencia de MySQL, donde cada tabla tiene su propio archivo de datos, en Oracle, una tabla puede estar almacenada en un tablespace compartido con otras tablas. Además, los tablespaces en Oracle se utilizan para controlar la asignación y el uso del espacio en disco y se pueden crear, modificar y eliminar según sea necesario.

    Oracle también proporciona diferentes tipos de tablespaces, como tablespaces de sistema, tablespaces temporales y tablespaces de usuario. Los tablespaces de sistema se utilizan para almacenar los objetos del diccionario de datos y otros objetos importantes del sistema, mientras que los tablespaces temporales se utilizan para almacenar datos temporales generados por consultas o transacciones.

    Los tablespaces de usuario se utilizan para almacenar los objetos de usuario, como tablas, índices y otros objetos. Cada usuario de Oracle puede tener su propio tablespace predeterminado, lo que permite a los administradores de bases de datos controlar y gestionar el espacio de almacenamiento de cada usuario de la base de datos.

    Otra diferencia importante entre los tablespaces de Oracle y el espacio de tablas de MySQL es que Oracle permite mover una tabla de un tablespace a otro utilizando el comando ALTER TABLE. Esto permite a los administradores de bases de datos mover objetos de la base de datos entre tablespaces para optimizar el rendimiento y la capacidad de almacenamiento de la base de datos.

    En resumen, mientras que en MySQL el espacio de tablas se refiere al espacio físico asignado a una tabla en el disco, en Oracle, los tablespaces son conjuntos lógicos de uno o más archivos de datos que se utilizan para almacenar los objetos de la base de datos. Los tablespaces en Oracle se utilizan para controlar la asignación y el uso del espacio en disco y se pueden crear, modificar y eliminar según sea necesario. Además, los tablespaces permiten a los administradores de bases de datos mover objetos de la base de datos entre tablespaces para optimizar el rendimiento y la capacidad de almacenamiento de la base de datos.

## **MongoDB**

### **Ejercicio 9**

> **9. Averigua si existe la posibilidad en MongoDB de decidir en qué archivo se almacena una colección.**

En MongoDB, no es posible decidir directamente en qué archivo se almacena una colección específica. MongoDB utiliza una estructura de almacenamiento llamada "data files" que se dividen en archivos de tamaño fijo llamados "extent", y una colección puede estar distribuida en varios de estos archivos.

Sin embargo, MongoDB ofrece la posibilidad de configurar los directorios de datos utilizados por el servidor, lo que puede afectar indirectamente a la ubicación de los archivos que almacenan las colecciones. Al iniciar el servidor de MongoDB, se puede especificar una o varias rutas de directorio donde se almacenarán los datos. Dentro de cada uno de estos directorios, MongoDB crea varios subdirectorios para almacenar diferentes tipos de datos, como datos de registro, datos temporales y datos de colección.

En resumidas cuentas, en MongoDB no es posible decidir directamente en qué archivo se almacena una colección específica, ya que la colección puede estar distribuida en varios archivos.

---

✒️ **Documentación realizada por Alejandro Montes Delgado.**
