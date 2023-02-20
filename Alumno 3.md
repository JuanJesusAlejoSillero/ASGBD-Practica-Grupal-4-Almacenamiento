# **ASGBD - Práctica Grupal 4: Almacenamiento**

## **Alumno 3**

**Tabla de contenidos:**

- [**ASGBD - Práctica Grupal 4: Almacenamiento**](#asgbd---práctica-grupal-4-almacenamiento)
  - [**Alumno 3**](#alumno-3)
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

> **1. Muestra los objetos a los que pertenecen las extensiones del tablespace TS2 (creado por Alumno 2) y el tamaño de cada una de ellas.**

```sql
SELECT SEGMENT_NAME, BYTES 
FROM DBA_EXTENTS
WHERE TABLESPACE_NAME = 'TS2';
```

![Alumno 3 - Oracle - 1](img/Alumno%203/Oracle/1.png)

### **Ejercicio 2**

> **2. Borra la tabla que está llenando TS2 consiguiendo que vuelvan a existir extensiones libres. Añade después otro fichero de datos a TS2.**

```sql
DROP TABLE my_table;

ALTER TABLESPACE TS2
ADD DATAFILE '/opt/oracle/oradata/ORCLCDB/TS2-ext.dbf' 
SIZE 2M;
```

![Alumno 3 - Oracle - 2](img/Alumno%203/Oracle/2.png)

### **Ejercicio 3**

> **3. Crea el tablespace TS3 gestionado localmente con un tamaño de extension uniforme de 128K y un fichero de datos asociado. Cambia la ubicación del fichero de datos y modifica la base de datos para que pueda acceder al mismo. Crea en TS3 dos tablas e inserta registros en las mismas. Comprueba que segmentos tiene TS3, qué extensiones tiene cada uno de ellos y en qué ficheros se encuentran.**

```sql
CREATE TABLESPACE TS3
DATAFILE '/opt/oracle/oradata/ORCLCDB/TS3.dbf'
SIZE 500M
EXTENT MANAGEMENT LOCAL
UNIFORM SIZE 128K;
```

```sql
ALTER TABLESPACE TS3 OFFLINE;
```

![Alumno 3 - Oracle - 3-1](img/Alumno%203/Oracle/3-1.png)

Creamos un directorio NEW-TS y movemos el archivo de datos a esta ruta:

```bash
mkdir /opt/oracle/oradata/ORCLCDB/NEW-TS/
mv /opt/oracle/oradata/ORCLCDB/TS3.dbf /opt/oracle/oradata/ORCLCDB/NEW-TS/TS3.dbf
```

```sql
ALTER DATABASE
RENAME FILE '/opt/oracle/oradata/ORCLCDB/TS3.dbf' TO '/opt/oracle/oradata/ORCLCDB/NEW-TS/TS3.dbf';

ALTER TABLESPACE TS3 ONLINE;
```

![Alumno 3 - Oracle - 3-2](img/Alumno%203/Oracle/3-2.png)

Creamos dos nuevas tablas con datos dentro del nuevo tablespace:

```sql
CREATE TABLE tabla1 (
  id NUMBER PRIMARY KEY,
  nombre VARCHAR2(50),
  apellido VARCHAR2(50)
) TABLESPACE TS3;
```

```sql
CREATE TABLE tabla2 (
  id NUMBER PRIMARY KEY,
  edad NUMBER,
  direccion VARCHAR2(100)
) TABLESPACE TS3;
```

```sql
INSERT INTO tabla1 (id, nombre, apellido)
VALUES (1, 'Juan', 'Pérez');

INSERT INTO tabla1 (id, nombre, apellido)
VALUES (2, 'María', 'González');

INSERT INTO tabla2 (id, edad, direccion)
VALUES (1, 30, 'Calle 1');

INSERT INTO tabla2 (id, edad, direccion)
VALUES (2, 35, 'Calle 2');
```

![Alumno 3 - Oracle - 3-3](img/Alumno%203/Oracle/3-3.png)

```sql
SELECT segment_name, bytes / 1024 / 1024 AS TAMANO_MB
FROM dba_extents
WHERE tablespace_name = 'TS3';
```

![Alumno 3 - Oracle - 3-4](img/Alumno%203/Oracle/3-4.png)

### **Ejercicio 4**

> **4. Redimensiona los ficheros asociados a los tres tablespaces que has creado de forma que ocupen el mínimo espacio posible para alojar sus objetos.**

Esto solo funciona para TABLESPACES NON-SYSTEM.

```sql
ALTER TABLESPACE TS1 SHRINK SPACE KEEP;
ALTER TABLESPACE TS2 SHRINK SPACE KEEP;
ALTER TABLESPACE TS3 SHRINK SPACE KEEP;
```

Comprobar si es NON-SYSTEM o SYSTEM:

```sql
SELECT TABLESPACE_NAME, CONTENTS FROM DBA_TABLESPACES;
```

![Alumno 3 - Oracle - 4](img/Alumno%203/Oracle/4.png)

Como podemos ver esta definido como PERMANENT, es decir, pertenece a SYSTEM y por lo cual no podemos utlizarlo. Para redimensionarlo al tamaño mínimo que en mi caso son 3M tendremos que ejecutar:

```sql
ALTER DATABASE DATAFILE '/opt/oracle/oradata/ORCLCDB/ts1.dbf' RESIZE 3M;
ALTER DATABASE DATAFILE '/opt/oracle/oradata/ORCLCDB/ts2.dbf' RESIZE 3M;
ALTER DATABASE DATAFILE '/opt/oracle/oradata/ORCLCDB/NEW-TS/TS3.dbf' RESIZE 3M;
```

### **Ejercicio 5**

> **5. Realiza un procedimiento llamado InformeRestricciones que reciba el nombre de una tabla y muestre los nombres de las restricciones que tiene, a qué columna o columnas afectan y en qué consisten exactamente.**

```sql
SET SERVEROUTPUT ON;

-- Procedimiento para mostrar objetivo de cada restricción:
CREATE OR REPLACE PROCEDURE mostrar_column_objetivo(p_cons DBA_CONSTRAINTS.CONSTRAINT_NAME%TYPE, p_nomtabla DBA_CONSTRAINTS.TABLE_NAME%TYPE)
IS
  v_objetivo VARCHAR2(1);
BEGIN
  SELECT CONSTRAINT_TYPE INTO v_objetivo
  FROM DBA_CONSTRAINTS
  WHERE CONSTRAINT_NAME=p_cons AND TABLE_NAME=p_nomtabla;
  IF v_objetivo IS NULL THEN
    dbms_output.put_line(chr(9)||'La restricción no cuenta con una descripción que muestre en que consiste.');
  ELSIF v_objetivo = 'P' THEN
    dbms_output.put_line(chr(9)||'Consiste en: Clave primaria');
  ELSIF v_objetivo = 'R' THEN
    dbms_output.put_line(chr(9)||'Consiste en: Clave foranea');
  ELSIF v_objetivo = 'C' THEN
    dbms_output.put_line(chr(9)||'Consiste en: Check');  
  ELSE
    dbms_output.put_line(chr(9)||'Consiste en: Unique');
  END IF;
END;
/

-- Procedimiento para mostrar columnas a las que afecta una restricción:
CREATE OR REPLACE PROCEDURE mostrar_column(p_cons DBA_CONS_COLUMNS.CONSTRAINT_NAME%TYPE)
IS
  CURSOR c_mos_column IS
  SELECT DISTINCT COLUMN_NAME, TABLE_NAME 
  FROM DBA_CONS_COLUMNS 
  WHERE CONSTRAINT_NAME=p_cons;
BEGIN
  FOR i IN c_mos_column LOOP
      dbms_output.put_line(chr(9)||'Columna afectada: '||i.COLUMN_NAME);
      mostrar_column_objetivo(p_cons, i.TABLE_NAME);
  END LOOP;
END;
/

-- Procedimiento para mostrar las restricciones que tiene la tabla que introducimos:
CREATE OR REPLACE PROCEDURE mostrar_restricciones(p_nomtabla DBA_TABLES.TABLE_NAME%TYPE)
IS
	CURSOR c_mos_restri IS
	SELECT DISTINCT CONSTRAINT_NAME 
  FROM DBA_CONSTRAINTS 
  WHERE TABLE_NAME=p_nomtabla;
BEGIN
	FOR i IN c_mos_restri LOOP
	dbms_output.put_line(chr(10)||'Restriccion: '||i.CONSTRAINT_NAME);
	mostrar_column(i.CONSTRAINT_NAME);
	END LOOP;
END;
/

-- Procedimiento principal (InformeRestricciones):
CREATE OR REPLACE PROCEDURE InformeRestricciones (p_nomtabla DBA_TABLES.TABLE_NAME%TYPE)
IS
BEGIN
  mostrar_restricciones(p_nomtabla);
END InformeRestricciones;
/
```

Comprobación de procedimientos compilados:

![Alumno 3 - Oracle - 5-1](img/Alumno%203/Oracle/5-1.png)

![Alumno 3 - Oracle - 5-2](img/Alumno%203/Oracle/5-2.png)

Para la comprobación creamos la siguiente tabla:

```sql
CREATE TABLE ALUMNOS (
  DNI NUMBER(4),
  Nombre VARCHAR2(10),
  Telefono VARCHAR2(9),
  Nacionalidad NUMBER(4),
  DEPTNO NUMBER(2),
  CONSTRAINT pk_dni PRIMARY KEY (DNI),
  CONSTRAINT fk_DEPTNO FOREIGN KEY (DEPTNO) REFERENCES DEPT(DEPTNO),
  CONSTRAINT uk_telf UNIQUE (Telefono),
  CONSTRAINT ck_nacionalidad CHECK (Nacionalidad IN ('Española', 'Francesa', 'Italiana')));

exec InformeRestricciones ('ALUMNOS');
```

![Alumno 3 - Oracle - 5-3](img/Alumno%203/Oracle/5-3.png)

### **Ejercicio 6**

> **6. Realiza un procedimiento llamado MostrarAlmacenamientoUsuario que reciba el nombre de un usuario y devuelva el espacio que ocupan sus objetos agrupando por dispositivos y archivos:**

```sql
SET SERVEROUTPUT ON;


-- Procedimiento para mostrar tablas......tamañoK

CREATE OR REPLACE PROCEDURE tablas_archivos(p_nomuser VARCHAR2, p_idArchivo NUMBER)
IS
  CURSOR c_mostrar_tablas IS
  SELECT DISTINCT t.table_name, e.BYTES/1024 AS K FROM DBA_TABLES t, DBA_EXTENTS e WHERE t.owner=UPPER(p_nomuser) and t.owner=e.owner and e.file_id=p_idArchivo;
BEGIN
  FOR i IN c_mostrar_tablas LOOP
    dbms_output.put_line(chr(10)||chr(9)||chr(9)||chr(9)||i.TABLE_NAME||'......'||i.K||' K');
  END LOOP;
END;
/


-- Procedimiento para mostrar index......tamañoK

CREATE OR REPLACE PROCEDURE index_archivos(p_nomuser VARCHAR2, p_idArchivo NUMBER)
IS
  CURSOR c_mostrar_tablas IS
  SELECT DISTINCT i.index_name, e.BYTES/1024 AS KI FROM DBA_INDEXES i, DBA_EXTENTS e WHERE i.owner=UPPER(p_nomuser) and i.owner=e.owner and e.file_id=p_idArchivo;
BEGIN
  FOR i IN c_mostrar_tablas LOOP
    dbms_output.put_line(chr(9)||chr(9)||chr(9)||i.INDEX_NAME||'......'||i.KI||' K');
  END LOOP;
END;
/


-- Funcion para total espacio archivos

CREATE OR REPLACE FUNCTION f_total_archivo (p_nomuser VARCHAR2,p_idfile NUMBER)
RETURN NUMBER
IS
  v_total NUMBER;
BEGIN
  SELECT DISTINCT d.USER_BYTES/1024 INTO v_total
  FROM DBA_DATA_FILES d, DBA_EXTENTS e 
  WHERE e.owner=upper(p_nomuser) AND d.FILE_ID=e.FILE_ID AND d.FILE_ID=p_idfile;
  RETURN v_total;
END;
/


-- Funcion para total espacio en dispositivo

CREATE OR REPLACE FUNCTION f_total_dispositivo (p_nomuser VARCHAR2,p_idfile NUMBER)
RETURN NUMBER
IS
  v_total NUMBER;
BEGIN
  SELECT DISTINCT SUM(d.USER_BYTES)/1024 INTO v_total
  FROM DBA_DATA_FILES d, DBA_EXTENTS e 
  WHERE e.owner=upper(p_nomuser) AND d.FILE_ID=e.FILE_ID AND d.FILE_ID=p_idfile;
  RETURN v_total;
END;
/


-- Funcion para total espacio en bd

CREATE OR REPLACE FUNCTION f_total_usuario (p_nomuser VARCHAR2)
RETURN NUMBER
IS
  v_total NUMBER;
BEGIN
  SELECT SUM(bytes)/1024 INTO v_total
  FROM DBA_SEGMENTS 
  WHERE OWNER=upper(p_nomuser);
  RETURN v_total;
END;
/


-- Procedimiento para mostrar dispositivos y archivos

CREATE OR REPLACE PROCEDURE dispositivo_archivo (p_nomuser VARCHAR2)
IS
  CURSOR c_mostrar_dis_arch IS
  SELECT DISTINCT REGEXP_SUBSTR(f.file_name, '^/[^/]+') AS DISPOSITIVO, f.file_name AS ARCHIVO, f.file_id, REGEXP_SUBSTR(f.file_name, '[^/]+$') AS NOMBRE_ARCHIVO FROM DBA_EXTENTS e, DBA_DATA_FILES f WHERE f.file_id=e.file_id AND OWNER=UPPER(p_nomuser);
BEGIN
  FOR i IN c_mostrar_dis_arch LOOP
    dbms_output.put_line(chr(10)||chr(9)||'Dispositivo: '||i.DISPOSITIVO||chr(10)||chr(10)||chr(9)||chr(9)||'Archivo: '||i.ARCHIVO);
    tablas_archivos(p_nomuser,i.FILE_ID);
    dbms_output.put_line(chr(9));
    index_archivos(p_nomuser,i.FILE_ID);
    dbms_output.put_line(chr(10)||chr(9)||chr(9)||'Total Espacio en Archivo '||i.NOMBRE_ARCHIVO||': '||f_total_archivo(p_nomuser,i.FILE_ID)||' K');
    dbms_output.put_line(chr(10)||chr(9)||'Total Espacio en Dispositivo '||i.DISPOSITIVO||': '||f_total_dispositivo(p_nomuser,i.FILE_ID)||' K');
  END LOOP;
  dbms_output.put_line(chr(10)||'Total Espacio Usuario en la BD: '||f_total_usuario(p_nomuser)||' K');
END;
/


-- Procedimiento principal MostrarAlmacenamientoUsuario

CREATE OR REPLACE PROCEDURE MostrarAlmacenamientoUsuario (p_nomuser VARCHAR2)
IS
BEGIN
dbms_output.put_line(chr(10)||'Usuario: '||p_nomuser);
dispositivo_archivo(p_nomuser);
END MostrarAlmacenamientoUsuario;
/


-- COMPROBACIÓN

exec MostrarAlmacenamientoUsuario('pacodiz');
```

![Alumno 3 - Oracle - 6](img/Alumno%203/Oracle/6.png)


## **PostgreSQL**

### **Ejercicio 7**

> **7. Averigua si es posible establecer cuotas de uso sobre los tablespaces en Postgres.**

PostgreSQL no tiene un mecanismo incorporado para establecer cuotas en los tablespaces. Sin embargo, existen algunas formas de lograr un comportamiento similar a tráves de restricciones o algunas herramientas de terceros como LVM o cuotas.

## **MySQL**

### **Ejercicio 8**

> **8. Averigua si existe el concepto de extensión en MySQL y si coincide con el existente en ORACLE.**

En MySQL, una extensión se refiere al tamaño en el que se asigna inicialmente un archivo de datos al crear una tabla. Cada tabla creada se almacena en un archivo de datos separado del sistema de archivos. Este archivo de datos se divide en páginas de un tamaño fijo, y cuando se crea una tabla, se le asigna una extensión inicial de varias páginas.

Cada extensión se compone de un número fijo de páginas (por defecto, 1 página). A medida que la tabla crece y se insertan más filas, se añaden más extensiones para acomodar el crecimiento de la tabla.

El tamaño de las extensiones generalmente se ajusta en función del tamaño esperado de las tablas y la cantidad de datos que se espera que se inserten en ellas. Esto puede llegar a afectar al rendimiento, ya que MySQL debe realizar operaciones de lectura y escritura en las extensiones cada vez que se accede a la tabla, por lo que un tamaño de extensión adecuado puede mejorar el rendimiento.

Ambos no coinciden ya que en oracle se utilizan las extensiones para gestionar el espacio de almacenamiento y permitir un crecimiento dinámico de las tablas, mientras que en MySQL, las extensiones se utilizan principalmente para asignar un tamaño inicial al archivo de datos de la tabla.

## **MongoDB**

### **Ejercicio 9**

> **9. Averigua si en MongoDB puede saberse el espacio disponible para almacenar nuevos documentos.**

En mongodb podemos saber el espacio del que disponemos en disco consultando la información de estadísticas de la base de datos, que incluye información sobre el espacio utilizado y disponible en el servidor.

Podemos ver esta información usando el comando *db.stats()*.

![Alumno 3 - MongoDB - 1](img/Alumno%203/MongoDB/1.png)

También la obtenemos con el comando *db.runCommand({dbStats: 1})*.

---

✒️ **Documentación realizada por Paco Diz Ureña.**