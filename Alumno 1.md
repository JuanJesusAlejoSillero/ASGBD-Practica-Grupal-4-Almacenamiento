# **ASGBD - Práctica Grupal 4: Almacenamiento**

## **Alumno 1**

**Tabla de contenidos:**

- [**ASGBD - Práctica Grupal 4: Almacenamiento**](#asgbd---práctica-grupal-4-almacenamiento)
  - [**Alumno 1**](#alumno-1)
  - [**Oracle**](#oracle)
    - [**Preparación del escenario**](#preparación-del-escenario)
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

### **Preparación del escenario**

1. Entrar en SQLPlus como administrador.

    ```bash
    sqlplus / as sysdba
    ```

2. Habilitar el modo script y la salida por pantalla, así como aumentar el tamaño de las líneas y páginas:

    ```sql
    ALTER SESSION SET "_ORACLE_SCRIPT"=TRUE;
    SET LINESIZE 32000;
    SET PAGESIZE 400;
    SET SERVEROUTPUT ON;
    ```

### **Ejercicio 1**

> **1. Muestra los espacios de tablas existentes en tu base de datos y la ruta de los ficheros que los componen. ¿Están las extensiones gestionadas localmente o por diccionario?**

Para ver los espacios de tablas existentes en la base de datos:

```sql
SELECT TABLESPACE_NAME, FILE_NAME
FROM DBA_DATA_FILES
UNION
SELECT TABLESPACE_NAME, FILE_NAME
FROM DBA_TEMP_FILES;
```

La primera sentencia *SELECT* muestra los *tablespaces* permanentes y la segunda los temporales. Más info sobre los tipos de espacios de tablas en [la documentación de Oracle](https://docs.oracle.com/database/121/ADMQS/GUID-3F47A659-71C8-4544-B3B6-736554805816.htm).

![Alumno 1 - Oracle - 1](img/Alumno%201/Oracle/1.png)

Para comprobar si los *extents* están siendo gestionados por diccionario o localmente:

```sql
SELECT TABLESPACE_NAME, EXTENT_MANAGEMENT
FROM DBA_TABLESPACES;
```

![Alumno 1 - Oracle - 2](img/Alumno%201/Oracle/2.png)

Podemos ver que los *extents* están siendo gestionados de forma local.

### **Ejercicio 2**

> **2. Usa la vista del diccionario de datos V$DATAFILE para mirar cuando fue la última vez que se ejecutó el proceso CKPT en tu base de datos.**

Para ver la fecha de la última vez a la que se ejecutó el proceso *CKPT* (*Oracle Checkpoint Process*) debemos consultar el valor máximo de la columna *CHECKPOINT_TIME*:

```sql
SELECT MAX(CHECKPOINT_TIME)
FROM V$DATAFILE;
```

![Alumno 1 - Oracle - 3](img/Alumno%201/Oracle/3.png)

### **Ejercicio 3**

> **3. Intenta crear el tablespace TS1 con un fichero de 2M en tu disco que crezca automáticamente cuando sea necesario. ¿Puedes hacer que la gestión de extensiones sea por diccionario? Averigua la razón.**

A la hora de crearlo podemos hacer que se cree en otra ruta diferente a la que tiene Oracle asignada por defecto para los *tablespaces* permanentes. Para ello, en el parámetro *DATAFILE* debemos indicar la ruta completa del fichero. En este caso lo crearé en la ruta por defecto por lo que solo tendré que indicar el nombre del fichero:

```sql
CREATE TABLESPACE TS1
DATAFILE 'TS1.dbf'
SIZE 2M
AUTOEXTEND ON
EXTENT MANAGEMENT LOCAL;
```

![Alumno 1 - Oracle - 4](img/Alumno%201/Oracle/4.png)

Tras crearlo, podemos verlo consultando la vista *DBA_DATA_FILES* como hicimos antes:

```sql
SELECT TABLESPACE_NAME, FILE_NAME
FROM DBA_DATA_FILES
WHERE TABLESPACE_NAME = 'TS1';
```

![Alumno 1 - Oracle - 5](img/Alumno%201/Oracle/5.png)

En mi instalación de Oracle 19c no podría crear un *tablespace* con la gestión de extensiones por diccionario ya que cuando se instaló, se configuró para que la gestión de extensiones fuera local. Si lo intento, me dará el siguiente error:

```sql
CREATE TABLESPACE TS2
DATAFILE 'TS2.dbf'
SIZE 2M
AUTOEXTEND ON
EXTENT MANAGEMENT DICTIONARY;
```

![Alumno 1 - Oracle - 6](img/Alumno%201/Oracle/6.png)

### **Ejercicio 4**

> **4. Averigua el tamaño de un bloque de datos en tu base de datos. Cámbialo al doble del valor que tenga.**

[Desde Oracle 9i, el tamaño de los bloques se puede configurar por *tablespace*](https://support.esri.com/en/technical-article/000011463) por lo que la forma más correcta de obtener esta información hoy en día sería consultando el tamaño de bloque y el nombre del *tablespace* con dicho tamaño, podemos hacerlo consultando la vista *DBA_TABLESPACES*:

```sql
SELECT BLOCK_SIZE, TABLESPACE_NAME
FROM DBA_TABLESPACES;
```

![Alumno 1 - Oracle - 7](img/Alumno%201/Oracle/7.png)

Si quisiéramos obtener el valor por defecto, podríamos consultar la vista *V$PARAMETER*:

```sql
SELECT NAME, VALUE
FROM V$PARAMETER
WHERE NAME = 'db_block_size';
```

![Alumno 1 - Oracle - 8](img/Alumno%201/Oracle/8.png)

Como se indica en [la documentación indicada anteriormente](https://support.esri.com/en/technical-article/000011463), para agregar soporte para *tablespaces* con tamaños de bloque diferentes (16K en este ejemplo), debemos ejecutar la sentencia:

```sql
ALTER SYSTEM SET DB_16K_CACHE_SIZE=60M;
```

![Alumno 1 - Oracle - 9](img/Alumno%201/Oracle/9.png)

Tras ejecutarla, el tamaño de bloque de los *tablespaces* ya existentes no variará, pero si creamos uno nuevo, podremos indicar el tamaño de bloque que queramos (8K que estaba por defecto o 16K que acabamos de habilitar):

```sql
CREATE TABLESPACE TS2
DATAFILE 'TS2.dbf'
SIZE 2M
AUTOEXTEND ON
EXTENT MANAGEMENT LOCAL
BLOCKSIZE 16384;

CREATE TABLESPACE TS3
DATAFILE 'TS3.dbf'
SIZE 2M
AUTOEXTEND ON
EXTENT MANAGEMENT LOCAL
BLOCKSIZE 8192;

SELECT BLOCK_SIZE, TABLESPACE_NAME
FROM DBA_TABLESPACES
WHERE TABLESPACE_NAME IN ('TS1', 'TS2', 'TS3')
```

![Alumno 1 - Oracle - 10](img/Alumno%201/Oracle/10.png)

En la captura podemos ver como el *tablespace* creado antes de la modificación (*TS1*) tiene un tamaño de bloque de 8K, y los dos creados después de la modificación (*TS2* y *TS3*) tienen un tamaño de bloque de 16K y 8K respectivamente.

### **Ejercicio 5**

> **5. Realiza un procedimiento *MostrarObjetosdeUsuarioenTS* que reciba el nombre de un tablespace y el de un usuario y muestre qué objetos tiene el usuario en dicho tablespace y qué tamaño tiene cada uno de ellos.**

```sql
CREATE OR REPLACE PROCEDURE MOSTRAROBJETOSDEUSUARIOENTS(P_TS VARCHAR2, P_USR VARCHAR2) IS
  CURSOR C_OBJETOS IS
    SELECT SEGMENT_NAME, BYTES
    FROM DBA_SEGMENTS
    WHERE TABLESPACE_NAME = P_TS
    AND OWNER = P_USR;
BEGIN
  DBMS_OUTPUT.PUT_LINE('Objetos del usuario ' || P_USR || ' en el tablespace ' || P_TS || ':');
  FOR OBJETO IN C_OBJETOS LOOP
    DBMS_OUTPUT.PUT_LINE('Nombre: ' || OBJETO.SEGMENT_NAME || ' - Tamaño: ' || OBJETO.BYTES);
  END LOOP;
END MOSTRAROBJETOSDEUSUARIOENTS;
/
```

![Alumno 1 - Oracle - 11](img/Alumno%201/Oracle/11.png)

Compila correctamente, si lo probamos con el usuario que creé para la práctica anterior:

```sql
EXEC MOSTRAROBJETOSDEUSUARIOENTS('USERS', 'PRACGRUPAL');
```

![Alumno 1 - Oracle - 12](img/Alumno%201/Oracle/12.png)

Funciona perfectamente.

### **Ejercicio 6**

> **6. Realiza un procedimiento llamado *MostrarUsrsCuotaIlimitada* que muestre los usuarios que puedan escribir de forma ilimitada en más de uno de los tablespaces que cuentan con ficheros en la unidad C:**

Según [la documentación oficial de Oracle](https://docs.oracle.com/cd/E18283_01/server.112/e17110/statviews_5068.htm), para que un usuario tenga cuota ilimitada en un *tablespace*, el valor de las columnas *MAX_BYTES* y *MAX_BLOCKS* de la vista *DBA_TS_QUOTAS* debe ser *-1*. Alternativamente, todo aquel usuario que tenga el privilegio *UNLIMITED TABLESPACE* también tendrá cuota ilimitada, por lo que tendremos que tener en cuenta ambas condiciones.

Además, al estar en un sistema Linux, usaremos la ruta raíz del sistema (*/*) en lugar de la unidad *C:*:

```sql
CREATE OR REPLACE PROCEDURE MOSTRARUSRSCUOTAILIMITADA IS
  CURSOR C_USUARIOCUOTAILIM IS
    SELECT USERNAME
    FROM DBA_TS_QUOTAS
    WHERE MAX_BYTES = -1
    AND MAX_BLOCKS = -1
    AND TABLESPACE_NAME IN (
      SELECT TABLESPACE_NAME
      FROM DBA_DATA_FILES
      WHERE FILE_NAME LIKE '/%')
    GROUP BY USERNAME
    HAVING COUNT(TABLESPACE_NAME) > 1
    UNION
    SELECT GRANTEE
    FROM DBA_SYS_PRIVS
    WHERE PRIVILEGE = 'UNLIMITED TABLESPACE';
BEGIN
  DBMS_OUTPUT.PUT_LINE('Usuarios con cuota ilimitada en más de un tablespace:');
  FOR USUARIO IN C_USUARIOCUOTAILIM LOOP
    DBMS_OUTPUT.PUT_LINE('Usuario: ' || USUARIO.USERNAME);
  END LOOP;
END MOSTRARUSRSCUOTAILIMITADA;
/
```

![Alumno 1 - Oracle - 13](img/Alumno%201/Oracle/13.png)

Compila correctamente, para probarlo:

```sql
EXEC MOSTRARUSRSCUOTAILIMITADA;
```

![Alumno 1 - Oracle - 14](img/Alumno%201/Oracle/14.png)

Y podemos ver que funciona debidamente.

## **PostgreSQL**

### **Ejercicio 7**

> **7. Averigua si existe el concepto de *tablespace* en PostgreSQL, en qué consiste y las diferencias con los *tablespaces* de Oracle.**

[En PostgreSQL existen los *tablespaces*](https://www.postgresql.org/docs/current/manage-ag-tablespaces.html) y, consisten en unidades lógicas de almacenamiento de datos que se corresponden con uno o más ficheros del sistema de archivos. Al usar *tablespaces*, un administrador puede controlar la distribución de una instalación de PostgreSQL. Esto es útil como mínimo en dos situaciones:

- En caso de que la partición o el volumen en el que se inicializó el clúster se quede sin espacio y no se pueda ampliar, se podrá crear un tablespace en otra partición y utilizarlo hasta que se pueda reconfigurar el sistema.

- Para optimizar el rendimiento, se pueden crear *tablespaces* en unidades de almacenamiento de alta velocidad, para almacenar en ellas los datos y objetos consultados con más frecuencia mientras que los datos menos utilizados se almacenan en unidades de almacenamiento de menor velocidad.

No obstante, los *tablespaces* de PostgreSQL cuentan con menos funciones que los de Oracle.

Las diferencias más importantes entre los *tablespaces* de ambos SGBD son:

- En Oracle se usan *datafiles* (archivos) para almacenar los datos, mientras que en PostgreSQL se usan directorios.

- Oracle ofrece un control más granular, permitiendo definir el tipo de *tablespace* (permanente, temporal o de *undo*), el tamaño inicial de los *datafiles* y el tamaño máximo de los mismos, mientras que en PostgreSQL no contamos con la mayoría de estas opciones.

Otra diferencia notable entre ambos SGBD consiste en la diferenciación que realiza Oracle de sus *tablespaces*, pudiendo ser [permanentes, temporales o de *undo*](https://www.techonthenet.com/oracle/tablespaces/create_tablespace.php), mientras que [en PostgreSQL solo se pueden crear *tablespaces* permanentes](https://www.postgresql.org/docs/13/sql-createtablespace.html). A continuación, la sintaxis a usar para cada caso:

- **Sintaxis de creación de un *tablespace* permanente en Oracle:**

Un tablespace permanente contiene objetos persistentes que se almacenan en archivos de datos.

```sql
CREATE
  [ SMALLFILE | BIGFILE ]
  TABLESPACE tablespace_name
  { DATAFILE { [ 'filename' | 'ASM_filename' ]
               [ SIZE integer [ K | M | G | T | P | E ] ]
               [ REUSE ]
               [ AUTOEXTEND
                   { OFF
                   | ON [ NEXT integer [ K | M | G | T | P | E ] ]
                   [ MAXSIZE { UNLIMITED | integer [ K | M | G | T | P | E ] } ]
                   }
               ]
             | [ 'filename | ASM_filename'
             | ('filename | ASM_filename'
                 [, 'filename | ASM_filename' ] )
             ]
             [ SIZE integer [ K | M | G | T | P | E ] ]
             [ REUSE ]
             }
     { MINIMUM EXTENT integer [ K | M | G | T | P | E ]
     | BLOCKSIZE integer [ K ]
     | { LOGGING | NOLOGGING }
     | FORCE LOGGING
     | DEFAULT [ { COMPRESS | NOCOMPRESS } ]
   storage_clause
     | { ONLINE | OFFLINE }
     | EXTENT MANAGEMENT
        { LOCAL
           [ AUTOALLOCATE
           | UNIFORM
              [ SIZE integer [ K | M | G | T | P | E ] ]
           ]
        | DICTIONARY
        }
     | SEGMENT SPACE MANAGEMENT { AUTO | MANUAL }
     | FLASHBACK { ON | OFF }
         [ MINIMUM EXTENT integer [ K | M | G | T | P | E ]
         | BLOCKSIZE integer [ K ]
         | { LOGGING | NOLOGGING }
         | FORCE LOGGING
         | DEFAULT [ { COMPRESS | NOCOMPRESS } ]
         storage_clause
         | { ONLINE | OFFLINE }
         | EXTENT MANAGEMENT
              { LOCAL
                [ AUTOALLOCATE | UNIFORM [ SIZE integer [ K | M | G | T | P | E ] ] ]
                | DICTIONARY
              }
         | SEGMENT SPACE MANAGEMENT { AUTO | MANUAL }
         | FLASHBACK { ON | OFF }
         ]
     }
```

- **Sintaxis de creación de un *tablespace* temporal en Oracle:**

Un tablespace temporal contiene objetos que se almacenan en archivos temporales que solo existen durante una sesión.

```sql
CREATE
  [ SMALLFILE | BIGFILE ]
  TEMPORARY TABLESPACE tablespace_name
    [ TEMPFILE { [ 'filename' | 'ASM_filename' ]
                 [ SIZE integer [ K | M | G | T | P | E ] ]
                 [ REUSE ]
                 [ AUTOEXTEND
                     { OFF
                     | ON [ NEXT integer [ K | M | G | T | P | E ] ]
                     [ MAXSIZE { UNLIMITED | integer [ K | M | G | T | P | E ] } ]
                     }
                 ]
               | [ 'filename | ASM_filename'
               | ('filename | ASM_filename'
                   [, 'filename | ASM_filename' ] )
               ]
               [ SIZE integer [ K | M | G | T | P | E ] ]
               [ REUSE ]
               }
    [ TABLESPACE GROUP { tablespace_group_name | '' } ]
    [ EXTENT MANAGEMENT
       { LOCAL
          [ AUTOALLOCATE | UNIFORM [ SIZE integer [ K | M | G | T | P | E ] ] ]
       | DICTIONARY
       } ]
```

- **Sintaxis de creación de un *tablespace* de *undo* en Oracle:**

Se crea un tablespace de *undo* para administrar los datos de restauración si la base de datos de Oracle se ejecuta en el modo de administración automática de *undo*.

```sql
CREATE
  [ SMALLFILE | BIGFILE ]
  UNDO TABLESPACE tablespace_name
    [ DATAFILE { [ 'filename' | 'ASM_filename' ]
                 [ SIZE integer [ K | M | G | T | P | E ] ]
                 [ REUSE ]
                 [ AUTOEXTEND
                     { OFF
                     | ON [ NEXT integer [ K | M | G | T | P | E ] ]
                     [ MAXSIZE { UNLIMITED | integer [ K | M | G | T | P | E ] } ]
                     }
                 ]
               | [ 'filename | ASM_filename'
               | ('filename | ASM_filename'
                   [, 'filename | ASM_filename' ] )
               ]
               [ SIZE integer [ K | M | G | T | P | E ] ]
               [ REUSE ]
               }
    [ EXTENT MANAGEMENT
       { LOCAL
          [ AUTOALLOCATE | UNIFORM [ SIZE integer [ K | M | G | T | P | E ] ] ]
       | DICTIONARY
       } ]
    [ RETENTION { GUARANTEE | NOGUARANTEE } ]
```

- **Sintaxis de creación de un *tablespace* en PostgreSQL:**

```sql
CREATE TABLESPACE tablespace_name
    [ OWNER { new_owner | CURRENT_USER | SESSION_USER } ]
    LOCATION 'directory'
    [ WITH ( tablespace_option = value [, ... ] ) ]
```

## **MySQL**

### **Ejercicio 8**

> **8. Averigua si pueden establecerse cláusulas de almacenamiento para las tablas o los espacios de tablas en MySQL.**

[En MySQL, de forma normal no se pueden establecer cláusulas de almacenamiento para las tablas o los espacios de tablas](https://dev.mysql.com/doc/refman/8.0/en/create-table.html), esto se debe al uso por defecto del motor de almacenamiento *InnoDB*. Para poder establecer cláusulas de almacenamiento, se debe cambiar al motor *NDB* (pensado para clústers).

En caso de usar el motor *NDB*, se pueden establecer cláusulas de almacenamiento para las tablas o los *tablespaces* de la siguiente forma:

```sql
CREATE TABLE table (
  c1 INT STORAGE DISK,
  c2 INT STORAGE MEMORY
) TABLESPACE tablespace ENGINE NDB;
```

## **MongoDB**

### **Ejercicio 9**

> **9. Averigua si existe el concepto de índice en MongoDB y las diferencias con los índices de Oracle. Explica los distintos tipos de índice que ofrece MongoDB.**

[En MongoDB existen los índices](https://www.mongodb.com/docs/manual/indexes/), y son *similares* a los de cualquier otro SGBD. Consisten en estructuras de datos que se utilizan para acelerar las consultas. Se definen a nivel de colección y pueden indexar cualquier campo o subcampo de los documentos de la colección en la que se definan.

El índice por defecto que crea MongoDB registra el campo *_id*, que es único y no puede eliminarse.

Podemos crear índices que indexen un campo (simples) o varios (compuestos) y de forma ascendente o descendente (incluso podemos indexar campos que tienen un *array*). El nombre por defecto asignado al índice será la concatenación del campo indexado y la dirección de indexación, por ejemplo, un índice creado con *{ objeto : 1, cantidad: -1 }* adquirirá el nombre *objeto_1_cantidad_-1*.

Los tipos de índices en MongoDB son:

- **[Índices simples](https://www.mongodb.com/docs/manual/core/index-single/):**

Indexan un solo campo. Su sintaxis es la siguiente:

```js
db.collection.createIndex( { campo: 1 } )
```

Si queremos especificar el nombre del índice, lo hacemos con *{ name: "nombre" }*, será igual en todos los tipos de índices:

```js
db.collection.createIndex( { campo: 1 }, { name: "nombre" } )
```

- **[Índices compuestos](https://www.mongodb.com/docs/manual/core/index-compound/):**

Indexan varios campos. Su sintaxis es la siguiente:

```js
db.collection.createIndex( { campo1: 1, campo2: -1 }, { name: "nombre" } )
```

- **[Índices multikey](https://www.mongodb.com/docs/manual/core/index-multikey/):**

Este tipo de índices es usado para campos que tienen un *array* de valores. Al indexarlos, hacemos que Mongo solo busque en campos concretos del *array* y no su totalidad. Su sintaxis es la misma que la de los índices simples, ya que el motor de Mongo detecta si el campo es un *array* de forma automática.

- **[Índices geoespaciales](https://www.mongodb.com/docs/manual/indexes/#geospatial-index):**

MongoDB ofrece índices geoespaciales que se utilizan para coordenadas geográficas en una esfera. Su sintaxis es la siguiente:

```js
db.collection.createIndex( { campo: "2dsphere" } )
```

- **[Índices de texto](https://www.mongodb.com/docs/manual/core/index-text/#std-label-index-feature-text):**

MongoDB proporciona un tipo de índice de texto que admite la búsqueda de cadenas en una colección. Solo puede existir un índice de este tipo por colección. Su sintaxis es la siguiente:

```js
db.collection.createIndex( { campo: "text" } )
```

- **[Índices hash](https://www.mongodb.com/docs/manual/core/index-hashed/):**

Los índices hash mantienen entradas con los hashes de los valores del campo indexado. Su sintaxis es la siguiente:

```js
db.collection.createIndex( { campo: "hashed" } )
```

- **[Índices agrupados](https://www.mongodb.com/docs/manual/reference/method/db.createCollection/#std-label-db.createCollection.clusteredIndex):**

A partir de MongoDB 5.3, se pueden crear colecciones con un índice agrupado. Estas colecciones se denominan colecciones agrupadas.

---

✒️ **Documentación realizada por Juan Jesús Alejo Sillero.**
