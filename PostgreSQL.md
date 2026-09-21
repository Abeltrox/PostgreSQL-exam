[TOC]

# PostgreSQL

PostgreSQL, que significa "Postgres Structured Query Language", fue desarrollado en la Universidad de California, Berkeley, en la década de 1980. El ilustre científico de la computación Michael Stonebraker lideró un equipo de investigadores que construyó un sistema de gestión de bases de datos relacional llamado Ingres. Más tarde, algunos de los miembros de ese equipo, incluido Stonebraker, se dispersaron y formaron el proyecto Postgres para continuar innovando en el área de la tecnología de bases de datos.

# Instalación Windows

Ver video : https://youtu.be/6U1M8TdCdgs?si=48v23fpifwS6WwRW

# Trabajando desde consola

## Parámetros de conexion

## Acceder a Psql

```
psql --host=postgres-db --username=lmsback_user
o
psql --host=localhost -U lmsback_user
```
[Ir a Creacion de bases de datos](##Creación-de-bases-de-datos)

## Prompts de consola

| Indicador | Significado                                                  |
| --------- | ------------------------------------------------------------ |
| =#        | Espera una nueva sentencia. En psql, =# es el prompt que indica que uno está conectado de manera interactiva a la base de datos PostgreSQL. Cada vez que se ve =# en la terminal, significa que se puede introducir una sentencia SQL o un comando interactivo de psql para comunicarse con la base de datos. |
| -#        | La sentencia aún no se ha terminado con ";" o \g             |
| "#        | Una cadena en comillas dobles no se ha cerrado               |
| '#        | Una cadena en comillas simples no se ha cerrado              |
| (#        | Un paréntesis no se ha cerrado                               |
| \?        | Muestra todas las variantes de comandos, q para terminar     |

## Comandos Psql

| Comando                          | Descripción                                                  |
| -------------------------------- | ------------------------------------------------------------ |
| \l                               | Lista las bases de datos                                     |
| \d                               | Describe las tablas de la base de datos en uso               |
| \ds                              | Lista las secuencias, estas se crean automáticamente cuando se declaran columnas de tipo serial. |
| \di                              | Lista los índices                                            |
| \dv                              | Lista las vistas                                             |
| \dp \z                           | Lista los privilegios sobre las tablas                       |
| \da                              | Lista las funciones de agregados                             |
| \df                              | Lista las funciones                                          |
| \g archivo                       | Ejecuta los comandos de archivo                              |
| \c                               | Seleccionar base de datos                                    |
| \dn                              | Listar Schema                                                |
| SET search_path TO NombreSchema; | Seleccionar un Schema                                        |

### Demostración comandos

#### \l

| Nombre           | Dueño    | Codificación | Collate               |
| ---------------- | -------- | ------------ | --------------------- |
| dbfarm           | postgres | UTF8         | Spanish_Colombia.1252 |
| demobusqueda     | postgres | UTF8         | Spanish_Colombia.1252 |
| electrowarehouse | postgres | UTF8         | Spanish_Colombia.1252 |
| geofarm          | postgres | UTF8         | Spanish_Colombia.1252 |
| introj2          | postgres | UTF8         | Spanish_Colombia.1252 |
| postgres         | postgres | UTF8         | Spanish_Colombia.1252 |
| template0        | postgres | UTF8         | Spanish_Colombia.1252 |
| template1        | postgres | UTF8         | Spanish_Colombia.1252 |

#### \c 

> Cuando se realiza seleccion de una base de datos diferente postgres solicita nuevamente la contraseña de administrador.

```
postgres=# \c geofarm
Contraseña:

Ahora está conectado a la base de datos «geofarm» con el usuario «postgres».
geofarm=#
```

#### \d

```
geofarm=# \d
```

| Esquema | Nombre                   | Tipo      | Dueño    |
| ------- | ------------------------ | --------- | -------- |
| public  | addresscustomer          | tabla     | postgres |
| public  | addresscustomer_id_seq   | secuencia | postgres |
| public  | addressdispensary        | tabla     | postgres |
| public  | addressdispensary_id_seq | secuencia | postgres |
| public  | addressfarmacy           | tabla     | postgres |
| public  | addressfarmacy_id_seq    | secuencia | postgres |
| public  | adminmode                | tabla     | postgres |

#### \ds

| Esquema | Nombre                   | Tipo      | Dueño    |
| ------- | ------------------------ | --------- | -------- |
| public  | addresscustomer_id_seq   | secuencia | postgres |
| public  | addressdispensary_id_seq | secuencia | postgres |
| public  | addressfarmacy_id_seq    | secuencia | postgres |
| public  | adminmode_id_seq         | secuencia | postgres |
| public  | conveyor_idconve_seq     | secuencia | postgres |
| public  | laboratory_id_seq        | secuencia | postgres |
| public  | location_id_seq          | secuencia | postgres |

#### \di

| Esquema | Nombre               | Tipo   | Dueño    | Tabla             |
| ------- | -------------------- | ------ | -------- | ----------------- |
| public  | addresscustomer_pk   | Índice | postgres | addresscustomer   |
| public  | addressdispensary_pk | Índice | postgres | addressdispensary |
| public  | addressfarmacy_pk    | Índice | postgres | addressfarmacy    |
| public  | adminmode_pk         | Índice | postgres | adminmode         |
| public  | adminmode_unique     | Índice | postgres | adminmode         |
| public  | city_pk              | Índice | postgres | city              |
| public  | city_unique          | Índice | postgres | city              |

#### \dp \z

| Esquema | Nombre                   | Tipo      | Privilegios | Privilegios de acceso a columnas | Políticas |
| ------- | ------------------------ | --------- | ----------- | -------------------------------- | --------- |
| public  | addresscustomer          | tabla     |             |                                  |           |
| public  | addresscustomer_id_seq   | secuencia |             |                                  |           |
| public  | addressdispensary        | tabla     |             |                                  |           |
| public  | addressdispensary_id_seq | secuencia |             |                                  |           |
| public  | addressfarmacy           | tabla     |             |                                  |           |

# Creación de bases de datos

1. Salir de la base datos seleccionada en caso de tener una seleccionada. Use el comando **\q**

2. Inicie sesión nuevamente [Acceder a Psql](##Acceder-a-Psql)

3. Ingrese el comando DDL para la creación de la base de datos de ejemplo (college)

   ```sql
   postgres=# CREATE DATABASE college;
   CREATE DATABASE
   postgres=#
   ```


4. Seleccione la base de datos creada \c [Nombre DB]

   ```sql
   postgres=# \c college;
   Ahora está conectado a la base de datos «college» con el usuario «postgres».
   college=#
   ```

# Tipos de datos

## Tipos de Datos Numéricos

| Name               | Storage Size | Description                     | Range                                                        |
| ------------------ | ------------ | ------------------------------- | ------------------------------------------------------------ |
| `smallint`         | 2 bytes      | small-range integer             | -32768 to +32767                                             |
| `integer`          | 4 bytes      | typical choice for integer      | -2147483648 to +2147483647                                   |
| `bigint`           | 8 bytes      | large-range integer             | -9223372036854775808 to +9223372036854775807                 |
| `decimal`          | variable     | user-specified precision, exact | up to 131072 digits before the decimal point; up to 16383 digits after the decimal point |
| `numeric`          | variable     | user-specified precision, exact | up to 131072 digits before the decimal point; up to 16383 digits after the decimal point |
| `real`             | 4 bytes      | variable-precision, inexact     | 6 decimal digits precision                                   |
| `double precision` | 8 bytes      | variable-precision, inexact     | 15 decimal digits precision                                  |
| `smallserial`      | 2 bytes      | small autoincrementing integer  | 1 to 32767                                                   |
| `serial`           | 4 bytes      | autoincrementing integer        | 1 to 2147483647                                              |
| `bigserial`        | 8 bytes      | large autoincrementing integer  | 1 to 9223372036854775807                                     |

## Tipos de Datos de Caracteres

| Name                                               | Description                              |
| -------------------------------------------------- | ---------------------------------------- |
| `character varying(*`n`*)`, `varchar(*`n`*)`       | variable-length with limit               |
| `character(*`n`*)`, `char(*`n`*)`, `bpchar(*`n`*)` | fixed-length, blank-padded               |
| `bpchar`                                           | variable unlimited length, blank-trimmed |
| `text`                                             | variable unlimited length                |

## Tipos de Datos Booleanos

| Name      | Storage Size | Description            |
| --------- | ------------ | ---------------------- |
| `boolean` | 1 byte       | state of true or false |

## Tipos de Datos de Fecha y Hora(https://www.postgresql.org/docs/current/datatype-datetime.html)

| Name                                          | Storage Size | Description                           | Low Value        | High Value      | Resolution    |
| --------------------------------------------- | ------------ | ------------------------------------- | ---------------- | --------------- | ------------- |
| `timestamp [ (*`p`*) ] [ without time zone ]` | 8 bytes      | both date and time (no time zone)     | 4713 BC          | 294276 AD       | 1 microsecond |
| `timestamp [ (*`p`*) ] with time zone`        | 8 bytes      | both date and time, with time zone    | 4713 BC          | 294276 AD       | 1 microsecond |
| `date`                                        | 4 bytes      | date (no time of day)                 | 4713 BC          | 5874897 AD      | 1 day         |
| `time [ (*`p`*) ] [ without time zone ]`      | 8 bytes      | time of day (no date)                 | 00:00:00         | 24:00:00        | 1 microsecond |
| `time [ (*`p`*) ] with time zone`             | 12 bytes     | time of day (no date), with time zone | 00:00:00+1559    | 24:00:00-1559   | 1 microsecond |
| `interval [ *`fields`* ] [ (*`p`*) ]`         | 16 bytes     | time interval                         | -178000000 years | 178000000 years | 1 microsecond |

## Tipos de Datos Monetarios

| Name    | Storage Size | Description     | Range                                          |
| ------- | ------------ | --------------- | ---------------------------------------------- |
| `money` | 8 bytes      | currency amount | -92233720368547758.08 to +92233720368547758.07 |

## Tipos de Datos Binarios (https://www.postgresql.org/docs/current/datatype-binary.html)

Una cadena binaria es una secuencia de octetos (o bytes). Las cadenas binarias se distinguen de las cadenas de caracteres de dos maneras. Primero, las cadenas binarias permiten específicamente almacenar octetos con valor cero y otros octetos "no imprimibles" (generalmente, octetos fuera del rango decimal de 32 a 126). Las cadenas de caracteres no permiten octetos con valor cero, ni tampoco permiten cualquier otro valor de octeto y secuencias de valores de octetos que sean inválidos según la codificación del conjunto de caracteres seleccionados en la base de datos. En segundo lugar, las operaciones en cadenas binarias procesan los bytes reales, mientras que el procesamiento de cadenas de caracteres depende de la configuración regional. En resumen, las cadenas binarias son apropiadas para almacenar datos que el programador considera como "bytes en bruto", mientras que las cadenas de caracteres son apropiadas para almacenar texto.

El tipo `bytea` soporta dos formatos para entrada y salida: el formato "hex" y el formato "escape" histórico de PostgreSQL. Ambos formatos son siempre aceptados en la entrada. El formato de salida depende del parámetro de configuración `bytea_output`; el valor predeterminado es hex. (Tenga en cuenta que el formato hex se introdujo en PostgreSQL 9.0; las versiones anteriores y algunas herramientas no lo entienden).

| Name    | Storage Size                               | Description                   |
| ------- | ------------------------------------------ | ----------------------------- |
| `bytea` | 1 or 4 bytes plus the actual binary string | variable-length binary string |

## Tipos de Datos de Redes (https://www.postgresql.org/docs/current/datatype-net-types.html)

| Name       | Storage Size  | Description                      |
| ---------- | ------------- | -------------------------------- |
| `cidr`     | 7 or 19 bytes | IPv4 and IPv6 networks           |
| `inet`     | 7 or 19 bytes | IPv4 and IPv6 hosts and networks |
| `macaddr`  | 6 bytes       | MAC addresses                    |
| `macaddr8` | 8 bytes       | MAC addresses (EUI-64 format)    |

## Tipos de Datos Geométricos (https://www.postgresql.org/docs/current/datatype-geometric.html)

| Name      | Storage Size | Description                      | Representation                      |
| --------- | ------------ | -------------------------------- | ----------------------------------- |
| `point`   | 16 bytes     | Point on a plane                 | (x,y)                               |
| `line`    | 32 bytes     | Infinite line                    | {A,B,C}                             |
| `lseg`    | 32 bytes     | Finite line segment              | ((x1,y1),(x2,y2))                   |
| `box`     | 32 bytes     | Rectangular box                  | ((x1,y1),(x2,y2))                   |
| `path`    | 16+16n bytes | Closed path (similar to polygon) | ((x1,y1),...)                       |
| `path`    | 16+16n bytes | Open path                        | [(x1,y1),...]                       |
| `polygon` | 40+16n bytes | Polygon (similar to closed path) | ((x1,y1),...)                       |
| `circle`  | 24 bytes     | Circle                           | <(x,y),r> (center point and radius) |



## Tipos de Datos JSON y XML

- `json`: Datos en formato JSON
- `jsonb`: Datos JSON en un formato binario más eficiente
- `xml`: Datos en formato XML

## Tipos de Datos Especiales

- `uuid`: Identificador único universal
- `array`: Arreglos de cualquier tipo de datos
- `composite`: Tipo compuesto de varios tipos de datos
- `range`: Rango de valores

## Tipos de Datos Enumerados

Los tipos enumerados se crean utilizando el comando `CREATE TYPE`, por ejemplo:

- `enum`: Conjunto de valores predefinidos

```sql
CREATE TYPE mood AS ENUM ('sad', 'ok', 'happy');
CREATE TYPE estadocivil AS ENUM ('Casado(a)', 'Soltero(a)', 'Viudo(a)','Con pelo(a)');
CREATE TABLE person (
    name text,
    current_mood mood
);
INSERT INTO person VALUES ('Moe', 'happy');
SELECT * FROM person WHERE current_mood = 'happy';
 name | current_mood
------+--------------
 Moe  | happy
```



# Creación de tablas

Para la creación de tablas se usa el comando CREATE TABLE

Sintaxis

```
CREATE TABLE nombre_tabla (
    columna1 tipo_dato [CONSTRAINT restricciones],
    columna2 tipo_dato [CONSTRAINT restricciones],
    ...
    [CONSTRAINT nombre_restriccion restriccion (columna1, columna2, ...)]
);
```

Ejemplo

```sql
CREATE TABLE empleados (
    id serial PRIMARY KEY,
    nombre varchar(100) NOT NULL,
    edad integer NOT NULL,
    salario numeric(10, 2),
    fecha_contratacion date,
    activo boolean DEFAULT true
);
```

En psql

```postgresql
college=# CREATE TABLE empleados (
college(#     id serial PRIMARY KEY,
college(#     nombre varchar(100) NOT NULL,
college(#     edad integer NOT NULL,
college(#     salario numeric(10, 2),
college(#     fecha_contratacion date,
college(#     activo boolean DEFAULT true
college(# );
CREATE TABLE
```

Ejercicio:

- Valide la creación de la tabla (**\dt**)

  | Esquema | Nombre    | Tipo  | Dueño    |
  | ------- | --------- | ----- | -------- |
  | public  | empleados | tabla | postgres |

- Verifique la estructura la tabla creada (\d empleados)

  | Columna            | Tipo                   | Ordenamiento | Nulable  | Por omisión                           |
  | ------------------ | ---------------------- | ------------ | -------- | ------------------------------------- |
  | id                 | integer                |              | not null | nextval('empleados_id_seq'::regclass) |
  | nombre             | character varying(100) |              | not null |                                       |
  | edad               | integer                |              | not null |                                       |
  | salario            | numeric(10,2)          |              |          |                                       |
  | fecha_contratacion | date                   |              |          |                                       |
  | activo             | boolean                |              |          | true                                  |

# Constraints (Restricciones)

En PostgreSQL, los constraints (restricciones) se utilizan para definir reglas sobre los datos que se pueden almacenar en una tabla. Aquí te muestro cómo se crean varios tipos de constraints al momento de crear una tabla y también cómo añadirlos a una tabla existente.

## Tipos de Constraints

1. **PRIMARY KEY**
2. **FOREIGN KEY**
3. **UNIQUE**
4. **NOT NULL**
5. **CHECK**
6. **DEFAULT**

## Explicación de los Constraints

**PRIMARY KEY**: Garantiza que cada valor en la columna (o conjunto de columnas) es único y no nulo.

```postgresql
id serial PRIMARY KEY
```

**FOREIGN KEY**: Establece una relación con otra tabla. Asegura que el valor en la columna coincide con un valor en la columna de referencia.

```postgresql
CONSTRAINT fk_departamento FOREIGN KEY (departamento_id) REFERENCES departamentos(id)
```

**UNIQUE**: Garantiza que todos los valores en una columna o conjunto de columnas son únicos.

```postgresql
UNIQUE (nombre, departamento_id)
```

**NOT NULL**: Asegura que una columna no contenga valores nulos.

```postgresql
nombre varchar(100) NOT NULL
```

**CHECK**: Asegura que los valores en una columna cumplen una condición especificada.

```sql
edad integer NOT NULL CHECK (edad > 0)
```

**DEFAULT**: Establece un valor predeterminado para la columna si no se especifica ningún valor.

```sql
salario numeric(10, 2) DEFAULT 0.00
```

## Añadir Constraints a una Tabla Existente

### Añadir PRIMARY KEY

```
ALTER TABLE empleados ADD PRIMARY KEY (id);
```

### Añadir FOREIGN KEY

```
ALTER TABLE empleados ADD CONSTRAINT fk_departamento
FOREIGN KEY (departamento_id) REFERENCES departamentos(id);
```

### Añadir UNIQUE

```
ALTER TABLE empleados ADD CONSTRAINT unique_nombre_departamento UNIQUE (nombre, departamento_id);
```

### Añadir NOT NULL

```sql
ALTER TABLE empleados ALTER COLUMN nombre SET NOT NULL;
```

### Añadir CHECK

```sql
ALTER TABLE empleados ADD CONSTRAINT check_edad CHECK (edad > 0);
```

### Añadir DEFAULT

```sql
ALTER TABLE empleados ALTER COLUMN salario SET DEFAULT 0.00;
```

# Ejercicio Tablas y Constraints

Usando PSQL cree las tablas country, region y city

**Country**

```sql
CREATE TABLE country (
	id serial,
	name varchar(50)
);
```

Agregue la llave principal (Primary Key)

```sql
ALTER TABLE country ADD PRIMARY KEY (id);
```

Ejecute los siguientes comandos: 

\dt (Lista tablas)

```
 Esquema |  Nombre   | Tipo  |  Due±o
---------+-----------+-------+----------
 public  | country   | tabla | postgres
 public  | empleados | tabla | postgres
```

\d Tabla : Permite visualizar estructura de la tabla

```
college=# \d country;
                                     Tabla ½public.country╗
 Columna |         Tipo          | Ordenamiento | Nulable  |             Por omisi¾n
---------+-----------------------+--------------+----------+-------------------------------------
 id      | integer               |              | not null | nextval('country_id_seq'::regclass)
 name    | character varying(50) |              |          |
```

```
college=# \di
                  Listado de relaciones
 Esquema |     Nombre     |  Tipo  |  Due±o   |   Tabla
---------+----------------+--------+----------+-----------
 public  | country_pkey   | Ýndice | postgres | country
 public  | empleados_pkey | Ýndice | postgres | empleados
```

**region**

```sql
CREATE TABLE region (
	id serial,
	name varchar(50),
    idcountry integer
);
```

```sql
ALTER TABLE region ADD PRIMARY KEY (id);
```

```sql
ALTER TABLE region ADD CONSTRAINT fk_region_country FOREIGN KEY (idcountry) REFERENCES country(id);
```

```postgresql
college=# \d region
                                      Tabla ½public.region╗
  Columna  |         Tipo          | Ordenamiento | Nulable  |            Por omisi¾n
-----------+-----------------------+--------------+----------+------------------------------------
 id        | integer               |              | not null | nextval('region_id_seq'::regclass)
 name      | character varying(50) |              |          |
 idcountry | integer               |              |          |
═ndices:
    "region_pkey" PRIMARY KEY, btree (id)
Restricciones de llave forßnea:
    "fk_region_country" FOREIGN KEY (idcountry) REFERENCES country(id)
```

 **city**

```sql
CREATE TABLE city (
	id serial,
	name varchar(50),
    idregion integer
);
```

```sql
ALTER TABLE city ADD PRIMARY KEY (id);
```

```sql
ALTER TABLE city ADD CONSTRAINT fk_city_region FOREIGN KEY (idregion) REFERENCES region(id);
```

```postgresql
                                      Tabla ½public.city╗
 Columna  |         Tipo          | Ordenamiento | Nulable  |           Por omisi¾n
----------+-----------------------+--------------+----------+----------------------------------
 id       | integer               |              | not null | nextval('city_id_seq'::regclass)
 name     | character varying(50) |              |          |
 idregion | integer               |              |          |
═ndices:
    "city_pkey" PRIMARY KEY, btree (id)
Restricciones de llave forßnea:
    "fk_city_region" FOREIGN KEY (idregion) REFERENCES region(id)
```

## Ejemplo General

```sql
CREATE TABLE ejemplo (
    id serial PRIMARY KEY,                  -- serial: Entero con auto-incremento
    nombre varchar(100) NOT NULL,           -- varchar: Cadena de longitud variable con límite
    descripcion text,                       -- text: Cadena de longitud variable sin límite
    precio numeric(10, 2) NOT NULL,         -- numeric: Número de precisión arbitraria
    en_stock boolean NOT NULL,              -- boolean: Valor booleano
    fecha_creacion date NOT NULL,           -- date: Fecha (sin hora)
    hora_creacion time NOT NULL,            -- time: Hora del día (sin fecha)
    fecha_hora timestamp NOT NULL,          -- timestamp: Fecha y hora (sin huso horario)
    fecha_hora_zona timestamp with time zone, -- timestamp with time zone: Fecha y hora con huso horario
    duracion interval,                      -- interval: Período de tiempo
    direccion_ip inet,                      -- inet: Dirección IP
    direccion_mac macaddr,                  -- macaddr: Dirección MAC
    punto_geometrico point,                 -- point: Punto en un plano de dos dimensiones
    datos_json json,                        -- json: Datos en formato JSON
    datos_jsonb jsonb,                      -- jsonb: Datos JSON en un formato binario más eficiente
    identificador_unico uuid,               -- uuid: Identificador único universal
    estado_monetario money,                 -- money: Cantidad monetaria
    rangos int4range,                       -- range: Rango de valores
    colores_preferidos varchar(20)[]        -- array: Arreglo de cadenas
);

-- Ejemplo de tipo ENUM
CREATE TYPE estadocivil AS ENUM ('Casado(a)', 'Soltero(a)', 'Viudo(a)','Con pelo(a)');
CREATE TYPE estado_pedido AS ENUM ('pendiente', 'en_proceso', 'completado', 'cancelado');

CREATE TABLE pedidos (
    id serial PRIMARY KEY,
    estado estado_pedido NOT NULL           -- enum: Conjunto de valores predefinidos
);

-- Ejemplo de tipo compuesto
CREATE TYPE direccion_completa AS (
    calle varchar,
    ciudad varchar,
    codigo_postal integer
);

CREATE TABLE clientes (
    id serial PRIMARY KEY,
    nombre varchar(50),
    ecivil estado_pedido,
    direccion direccion_completa            -- composite: Tipo compuesto
);

```

![](https://i.ibb.co/k262gGtW/image.png)

## Ejemplo de Inserción de Datos

```sql
-- Insertando datos en la tabla 'ejemplo'
INSERT INTO ejemplo (
    nombre, descripcion, precio, en_stock, fecha_creacion, hora_creacion,
    fecha_hora, fecha_hora_zona, duracion, direccion_ip, direccion_mac,
    punto_geometrico, datos_json, datos_jsonb, identificador_unico,
    estado_monetario, rangos, colores_preferidos
) VALUES (
    'Producto A', 'Descripción del producto A', 29.99, true, '2024-07-09', '14:30:00',
    '2024-07-09 14:30:00', '2024-07-09 14:30:00+00', '1 day', '192.168.1.1', '08:00:27:00:00:00',
    '(10, 20)', '{"key": "value"}', '{"key": "value"}', '550e8400-e29b-41d4-a716-446655440000',
    'USD 100.00', '[10, 20)', ARRAY['rojo', 'verde', 'azul']
);

-- Insertando datos en la tabla 'pedidos'
INSERT INTO pedidos (estado) VALUES ('pendiente');

-- Insertando datos en la tabla 'clientes'
INSERT INTO clientes (direccion) VALUES (ROW('Calle Falsa 123', 'Springfield', 12345));

```

# 

# Taller Diseño de base de datos

![](https://i.ibb.co/0brN66k/image.png)

Teniendo en cuenta el diagrama entidad relación resuelva los siguientes requerimientos:

1. Genere los comandos DDL que permitan la creación de la base de datos del DER suministrado,
2. Genere los comandos DML que permitan insertar datos en la base de datos creada en el paso 1

# SELECT

![](https://i.ibb.co/fYGwB8BC/image.png)

# Fecha, DateTime e Intervalos

La gestión de datos temporales es una parte fundamental en el desarrollo de aplicaciones modernas, ya que permite el registro y seguimiento preciso de eventos y transacciones a lo largo del tiempo. PostgreSQL, como uno de los sistemas de gestión de bases de datos relacionales más avanzados, proporciona un conjunto robusto de herramientas y tipos de datos para trabajar con fechas, horas y intervalos de tiempo.

En este capítulo, exploraremos a fondo los tipos de datos `date`, `timestamp` y `interval` en PostgreSQL, así como las funciones y operadores que nos permiten manipular y consultar estos datos de manera eficiente. Aprenderemos a utilizar estos tipos de datos para realizar cálculos temporales, manejar zonas horarias y aplicar intervalos de tiempo en nuestras consultas SQL.

Para este capitulo se usaran datos y estructura de una base de datos que podrá encontrar en el siguiente gist : https://gist.github.com/d113a1025ce0b5d0285972196cf6773d.git (**La estructura y data fue extraída de la plataforma devtalles con fines educativos**)

Cree una base de datos llamada logistica

`Asignar hora colombia`

```
SET TIME ZONE 'America/Bogota';
```

Si quieres dejarlo de forma persistente para la base de datos:

```
ALTER DATABASE bkddb
SET timezone TO 'America/Bogota';
```

O para un usuario específico:

```
ALTER ROLE bkseducate
SET timezone TO 'America/Bogota';
```

Algunas de las zonas más utilizadas son:

| Región                | Zona IANA                        | UTC aproximado |
| --------------------- | -------------------------------- | -------------- |
| Colombia              | `America/Bogota`                 | UTC-5          |
| Perú                  | `America/Lima`                   | UTC-5          |
| Ecuador               | `America/Guayaquil`              | UTC-5          |
| Panamá                | `America/Panama`                 | UTC-5          |
| Venezuela             | `America/Caracas`                | UTC-4          |
| Chile                 | `America/Santiago`               | variable       |
| Argentina             | `America/Argentina/Buenos_Aires` | UTC-3          |
| Brasil - São Paulo    | `America/Sao_Paulo`              | UTC-3          |
| Bolivia               | `America/La_Paz`                 | UTC-4          |
| Paraguay              | `America/Asuncion`               | variable       |
| México                | `America/Mexico_City`            | UTC-6          |
| EE. UU. - Nueva York  | `America/New_York`               | UTC-5/-4       |
| EE. UU. - Chicago     | `America/Chicago`                | UTC-6/-5       |
| EE. UU. - Denver      | `America/Denver`                 | UTC-7/-6       |
| EE. UU. - Los Ángeles | `America/Los_Angeles`            | UTC-8/-7       |
| EE. UU. - Phoenix     | `America/Phoenix`                | UTC-7          |
| Canadá - Toronto      | `America/Toronto`                | UTC-5/-4       |
| Reino Unido           | `Europe/London`                  | UTC+0/+1       |
| España                | `Europe/Madrid`                  | UTC+1/+2       |
| Francia               | `Europe/Paris`                   | UTC+1/+2       |
| Alemania              | `Europe/Berlin`                  | UTC+1/+2       |
| Italia                | `Europe/Rome`                    | UTC+1/+2       |
| Portugal              | `Europe/Lisbon`                  | UTC+0/+1       |
| Grecia                | `Europe/Athens`                  | UTC+2/+3       |
| Turquía               | `Europe/Istanbul`                | UTC+3          |
| India                 | `Asia/Kolkata`                   | UTC+5:30       |
| Japón                 | `Asia/Tokyo`                     | UTC+9          |
| China                 | `Asia/Shanghai`                  | UTC+8          |
| Corea del Sur         | `Asia/Seoul`                     | UTC+9          |
| Singapur              | `Asia/Singapore`                 | UTC+8          |
| Dubái                 | `Asia/Dubai`                     | UTC+4          |
| Australia - Sídney    | `Australia/Sydney`               | UTC+10/+11     |
| Australia - Perth     | `Australia/Perth`                | UTC+8          |
| Nueva Zelanda         | `Pacific/Auckland`               | UTC+12/+13     |
| UTC                   | `UTC`                            | UTC+0          |

```
SHOW timezone;
SELECT now();
```

| Objetivo                                       | Comando                                                      |
| ---------------------------------------------- | ------------------------------------------------------------ |
| Ver la zona horaria actual                     | `SHOW timezone;`                                             |
| Ver fecha y hora actual                        | `SELECT now();`                                              |
| Cambiar zona horaria solo en la sesión actual  | `SET TIME ZONE 'America/Bogota';`                            |
| Volver a UTC en la sesión                      | `SET TIME ZONE 'UTC';`                                       |
| Ver todas las zonas horarias disponibles       | `SELECT name FROM pg_timezone_names ORDER BY name;`          |
| Ver zonas con abreviatura y offset             | `SELECT name, abbrev, utc_offset, is_dst FROM pg_timezone_names ORDER BY name;` |
| Buscar una zona específica                     | `SELECT * FROM pg_timezone_names WHERE name = 'America/Bogota';` |
| Buscar zonas por nombre                        | `SELECT name FROM pg_timezone_names WHERE name ILIKE '%Bogota%';` |
| Listar zonas de América                        | `SELECT name FROM pg_timezone_names WHERE name LIKE 'America/%' ORDER BY name;` |
| Configurar zona horaria para una base de datos | `ALTER DATABASE bkddb SET timezone TO 'America/Bogota';`     |
| Configurar zona horaria para un usuario/rol    | `ALTER ROLE bkseducate SET timezone TO 'America/Bogota';`    |
| Consultar configuración de una base            | `SELECT datname, datconfig FROM pg_database WHERE datname = 'bkddb';` |
| Consultar configuración de un rol              | `SELECT rolname, rolconfig FROM pg_roles WHERE rolname = 'bkseducate';` |
| Convertir una fecha a otra zona horaria        | `SELECT now() AT TIME ZONE 'America/Bogota';`                |
| Convertir entre zonas                          | `SELECT timestamp '2026-09-14 00:53:51' AT TIME ZONE 'UTC' AT TIME ZONE 'America/Bogota';` |

> [!NOTE]
>
> SELECT * FROM pg_timezone_names WHERE name = 'America/Bogota';
>
> > | Columna      | Valor            | Significado                                                  |
> > | ------------ | ---------------- | ------------------------------------------------------------ |
> > | `name`       | `America/Bogota` | Nombre oficial IANA de la zona horaria                       |
> > | `abbrev`     | `-05`            | Abreviatura que PostgreSQL usa para representar el desplazamiento |
> > | `utc_offset` | `-05:00:00`      | Bogotá está 5 horas detrás de UTC                            |
> > | `is_dst`     | `f`              | No está usando horario de verano                             |
>
> 



## `CURRENT_DATE`

Devuelve la fecha actual del sistema.

```sql
SELECT CURRENT_DATE;
```

## `CURRENT_TIME`

Devuelve la hora actual del sistema.

```sql
SELECT CURRENT_TIME;
```

## `CURRENT_TIMESTAMP`

Devuelve la fecha y hora actuales del sistema.

```sql
SELECT CURRENT_TIMESTAMP;
```

## `NOW()`

Función equivalente a `CURRENT_TIMESTAMP`.

```sql
SELECT NOW();
```

## `AGE()`

Calcula la diferencia entre dos fechas, o entre una fecha y la fecha actual si se proporciona solo una fecha.

```sql
SELECT AGE(DATE '2024-07-01');
SELECT AGE('2024-07-01', '2024-06-01');
```

## `EXTRACT()`

Extrae subcampos (como año, mes, día, hora) de valores de fecha/hora.

```sql
SELECT EXTRACT(YEAR FROM TIMESTAMP '2024-07-13 14:23:45');
-- Resultado: 2024

SELECT EXTRACT(MONTH FROM TIMESTAMP '2024-07-13 14:23:45');
-- Resultado: 7

SELECT EXTRACT(DAY FROM TIMESTAMP '2024-07-13 14:23:45');
-- Resultado: 13
```

# Taller: Fechas, DateTime e Intervalos en PostgreSQL

## Objetivo

Al finalizar el taller, el estudiante podrá:

- almacenar fechas y horas correctamente;
- diferenciar `DATE`, `TIME`, `TIMESTAMP` y `TIMESTAMPTZ`;
- obtener fecha y hora actual;
- sumar y restar intervalos;
- calcular diferencias entre fechas;
- extraer partes de una fecha;
- trabajar con zonas horarias;
- resolver consultas reales usando fechas.

## Contexto

Se trabajará con un pequeño sistema de reservas.

Crea una base de datos o usa una existente y ejecuta:

```
CREATE TABLE reservas (
    id SERIAL PRIMARY KEY,
    cliente VARCHAR(100) NOT NULL,
    fecha_reserva DATE NOT NULL,
    fecha_hora_inicio TIMESTAMPTZ NOT NULL,
    fecha_hora_fin TIMESTAMPTZ NOT NULL,
    fecha_creacion TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP
);
```

Inserta datos:

```
INSERT INTO reservas
(cliente, fecha_reserva, fecha_hora_inicio, fecha_hora_fin)
VALUES
(
    'Ana Torres',
    '2026-09-15',
    '2026-09-15 08:00:00-05',
    '2026-09-15 10:00:00-05'
),
(
    'Carlos Pérez',
    '2026-09-16',
    '2026-09-16 14:30:00-05',
    '2026-09-16 16:00:00-05'
),
(
    'Laura Gómez',
    '2026-09-20',
    '2026-09-20 09:00:00-05',
    '2026-09-20 12:30:00-05'
);
```

------

# Parte 1. Tipos de fecha y hora

Ejecuta:

```
SELECT CURRENT_DATE;
SELECT CURRENT_TIME;
SELECT CURRENT_TIMESTAMP;
SELECT NOW();
```

Responde:

1. ¿Qué diferencia observas entre `CURRENT_DATE` y `CURRENT_TIMESTAMP`?
2. ¿`NOW()` y `CURRENT_TIMESTAMP` muestran el mismo resultado?
3. ¿Qué zona horaria está usando PostgreSQL?

Consulta:

```
SHOW timezone;
```

------

# Parte 2. DATE

Consulta todas las fechas de reserva:

```
SELECT
    id,
    cliente,
    fecha_reserva
FROM reservas;
```

Filtra las reservas realizadas después de una fecha:

```
SELECT *
FROM reservas
WHERE fecha_reserva > '2026-09-16';
```

Busca reservas en un rango:

```
SELECT *
FROM reservas
WHERE fecha_reserva
BETWEEN '2026-09-15' AND '2026-09-20';
```

## Ejercicio

Consulta las reservas realizadas entre:

```
2026-09-16
y
2026-09-30
```

------

# Parte 3. TIMESTAMP y TIMESTAMPTZ

Consulta:

```
SELECT
    cliente,
    fecha_hora_inicio,
    fecha_hora_fin
FROM reservas;
```

Observa el offset:

```
-05
```

Esto indica la zona horaria asociada a la representación.

Cambia temporalmente la sesión:

```
SET TIME ZONE 'UTC';
```

Consulta nuevamente:

```
SELECT
    cliente,
    fecha_hora_inicio
FROM reservas;
```

Luego vuelve a Bogotá:

```
SET TIME ZONE 'America/Bogota';
```

## Pregunta

¿Cambió el instante almacenado o solamente la forma de mostrarlo?

------

# Parte 4. INTERVAL

PostgreSQL permite representar periodos de tiempo.

Ejemplo:

```
SELECT INTERVAL '2 hours';
SELECT INTERVAL '3 days';
SELECT INTERVAL '1 month';
```

También puedes combinar:

```
SELECT INTERVAL '2 days 5 hours 30 minutes';
```

------

# Parte 5. Sumar intervalos a fechas

Ejecuta:

```
SELECT CURRENT_DATE + INTERVAL '7 days';
SELECT CURRENT_TIMESTAMP + INTERVAL '3 hours';
```

Ejemplo aplicado:

```
SELECT
    cliente,
    fecha_hora_inicio,
    fecha_hora_inicio + INTERVAL '30 minutes'
FROM reservas;
```

## Ejercicio

Muestra cada reserva y calcula una nueva hora de inicio agregando:

```
15 minutos
```

------

# Parte 6. Restar intervalos

```
SELECT CURRENT_TIMESTAMP - INTERVAL '1 day';
SELECT CURRENT_DATE - INTERVAL '1 month';
```

## Ejercicio

Consulta cuál habría sido la fecha de cada reserva:

```
7 días antes
```

------

# Parte 7. Diferencia entre fechas

Puedes restar dos fechas:

```
SELECT
    DATE '2026-09-20'
    -
    DATE '2026-09-15';
```

Resultado:

```
5
```

Es decir, 5 días.

Con timestamps:

```
SELECT
    TIMESTAMP '2026-09-15 10:00:00'
    -
    TIMESTAMP '2026-09-15 08:00:00';
```

Resultado:

```
02:00:00
```

## Aplicado a la tabla

```
SELECT
    cliente,
    fecha_hora_fin - fecha_hora_inicio AS duracion
FROM reservas;
```

------

# Parte 8. EXTRACT

`EXTRACT` permite obtener partes de una fecha.

```
SELECT EXTRACT(YEAR FROM CURRENT_DATE);
SELECT EXTRACT(MONTH FROM CURRENT_DATE);
SELECT EXTRACT(DAY FROM CURRENT_DATE);
SELECT EXTRACT(HOUR FROM CURRENT_TIMESTAMP);
```

Aplicado:

```
SELECT
    cliente,
    EXTRACT(HOUR FROM fecha_hora_inicio) AS hora_inicio
FROM reservas;
```

## Ejercicio

Obtén de cada reserva:

```
cliente
año
mes
día
hora
```

------

# Parte 9. DATE_PART

Otra alternativa es:

```
SELECT DATE_PART('year', CURRENT_DATE);
SELECT DATE_PART('month', CURRENT_DATE);
SELECT DATE_PART('hour', CURRENT_TIMESTAMP);
```

Compara:

```
EXTRACT(YEAR FROM CURRENT_DATE)
```

con:

```
DATE_PART('year', CURRENT_DATE)
```

------

# Parte 10. AGE

PostgreSQL incluye `AGE()` para diferencias más legibles.

```
SELECT AGE(
    DATE '2026-09-20',
    DATE '2025-09-20'
);
```

Otro ejemplo:

```
SELECT AGE(
    CURRENT_DATE,
    DATE '2000-01-01'
);
```

## Ejercicio

Calcula cuánto tiempo ha pasado desde:

```
2020-01-01
```

hasta hoy.

------

# Parte 11. DATE_TRUNC

Sirve para truncar una fecha a una unidad.

```
SELECT DATE_TRUNC('day', CURRENT_TIMESTAMP);
SELECT DATE_TRUNC('month', CURRENT_TIMESTAMP);
SELECT DATE_TRUNC('year', CURRENT_TIMESTAMP);
```

Ejemplo:

```
SELECT
    DATE_TRUNC('day', fecha_hora_inicio)
FROM reservas;
```

------

# Parte 12. Formatear fechas

Usa `TO_CHAR`.

```
SELECT TO_CHAR(
    CURRENT_TIMESTAMP,
    'YYYY-MM-DD HH24:MI:SS'
);
```

Ejemplo:

```
SELECT
    cliente,
    TO_CHAR(
        fecha_hora_inicio,
        'DD/MM/YYYY HH24:MI'
    ) AS fecha_formateada
FROM reservas;
```

Otros formatos útiles:

```
YYYY
MM
DD
HH24
MI
SS
Day
Month
```

## Ejercicio

Muestra la fecha así:

```
15/09/2026 08:00
```

------

# Parte 13. Conversión de zona horaria

Ejemplo:

```
SELECT
    CURRENT_TIMESTAMP
    AT TIME ZONE 'America/Bogota';
```

También:

```
SELECT
    CURRENT_TIMESTAMP
    AT TIME ZONE 'America/New_York';
```

Ejercicio:

Muestra la hora actual en:

```
America/Bogota
America/New_York
Europe/Madrid
Asia/Tokyo
```

------

# Parte 14. Consultas prácticas

Resuelve:

1. Mostrar reservas futuras.

```
SELECT *
FROM reservas
WHERE fecha_hora_inicio > CURRENT_TIMESTAMP;
```

1. Mostrar reservas ya finalizadas.

```
SELECT *
FROM reservas
WHERE fecha_hora_fin < CURRENT_TIMESTAMP;
```

1. Mostrar duración de cada reserva.

```
SELECT
    cliente,
    fecha_hora_fin - fecha_hora_inicio AS duracion
FROM reservas;
```

1. Mostrar reservas que duran más de 2 horas.

```
SELECT *
FROM reservas
WHERE fecha_hora_fin - fecha_hora_inicio
      > INTERVAL '2 hours';
```

1. Mostrar reservas del mes actual.

```
SELECT *
FROM reservas
WHERE DATE_TRUNC('month', fecha_reserva)
      =
      DATE_TRUNC('month', CURRENT_DATE);
```

------

# Parte 15. Reto final

Crea una tabla:

```
CREATE TABLE prestamos (
    id SERIAL PRIMARY KEY,
    usuario VARCHAR(100),
    fecha_prestamo TIMESTAMPTZ,
    fecha_devolucion TIMESTAMPTZ
);
```

Inserta mínimo 5 registros.

Después construye consultas que permitan:

- identificar préstamos vencidos;
- calcular duración del préstamo;
- mostrar cuántos días faltan para la devolución;
- mostrar préstamos realizados este mes;
- mostrar préstamos de más de 7 días;
- sumar 3 días a la fecha de devolución;
- mostrar fecha de devolución con formato `DD/MM/YYYY HH24:MI`;
- convertir la fecha de devolución a UTC.

## Entrega esperada

El estudiante debe entregar:

```
taller_fechas.sql
```

con:

```
1. creación de tablas
2. INSERT
3. consultas
4. comentarios explicativos
```

Por ejemplo:

```
-- Ejercicio 1
-- Consultar reservas futuras

SELECT *
FROM reservas
WHERE fecha_hora_inicio > CURRENT_TIMESTAMP;
```

# Taller práctico: Consultas de fechas, DateTime e Intervalos a partir de casos de uso

## Objetivo

El estudiante deberá analizar situaciones funcionales y traducirlas a consultas PostgreSQL usando principalmente:

`DATE`, `TIMESTAMP`, `TIMESTAMPTZ`, `INTERVAL`, `CURRENT_DATE`, `CURRENT_TIMESTAMP`, `EXTRACT`, `DATE_TRUNC`, `AGE`, `TO_CHAR` y `AT TIME ZONE`.

## Escenario

Una empresa administra reservas de espacios de trabajo y salas de reunión.

Se dispone de la siguiente tabla:

```
CREATE TABLE reservas (
    id SERIAL PRIMARY KEY,
    cliente VARCHAR(100) NOT NULL,
    sala VARCHAR(80) NOT NULL,
    fecha_reserva DATE NOT NULL,
    fecha_hora_inicio TIMESTAMPTZ NOT NULL,
    fecha_hora_fin TIMESTAMPTZ NOT NULL,
    fecha_creacion TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP,
    estado VARCHAR(30) NOT NULL
);
```

Datos de prueba:

```
INSERT INTO reservas
(cliente, sala, fecha_reserva, fecha_hora_inicio, fecha_hora_fin, estado)
VALUES
('Ana Torres', 'Sala A', '2026-09-15',
 '2026-09-15 08:00:00-05',
 '2026-09-15 10:00:00-05',
 'CONFIRMADA'),

('Carlos Pérez', 'Sala B', '2026-09-16',
 '2026-09-16 14:30:00-05',
 '2026-09-16 16:00:00-05',
 'CONFIRMADA'),

('Laura Gómez', 'Sala A', '2026-09-20',
 '2026-09-20 09:00:00-05',
 '2026-09-20 12:30:00-05',
 'PENDIENTE'),

('Miguel Rojas', 'Sala C', '2026-09-10',
 '2026-09-10 13:00:00-05',
 '2026-09-10 15:00:00-05',
 'FINALIZADA'),

('Diana Ruiz', 'Sala B', '2026-09-25',
 '2026-09-25 07:30:00-05',
 '2026-09-25 11:00:00-05',
 'CONFIRMADA'),

('Andrés López', 'Sala A', '2026-10-02',
 '2026-10-02 15:00:00-05',
 '2026-10-02 18:30:00-05',
 'PENDIENTE');
```

------

# Parte 1. Consultas básicas de fecha

## Caso de uso 1

El administrador necesita ver todas las reservas cuya fecha sea posterior al 16 de septiembre de 2026.

**El estudiante debe construir una consulta que muestre:**

```
id
cliente
sala
fecha_reserva
```

Condición:

```
fecha_reserva > 2026-09-16
```

------

## Caso de uso 2

El gerente desea conocer todas las reservas realizadas entre el 15 y el 25 de septiembre de 2026, incluyendo ambas fechas.

El estudiante debe utilizar un operador adecuado para consultar un rango de fechas.

------

## Caso de uso 3

El sistema necesita mostrar únicamente las reservas correspondientes al día actual.

No se debe escribir manualmente la fecha del día.

------

# Parte 2. Fecha y hora

## Caso de uso 4

Recepción necesita visualizar las reservas que todavía no han iniciado.

El estudiante debe comparar:

```
fecha_hora_inicio
```

contra la fecha y hora actuales.

------

## Caso de uso 5

El sistema debe mostrar todas las reservas que ya finalizaron.

Se deben retornar:

```
cliente
sala
fecha_hora_inicio
fecha_hora_fin
```

------

## Caso de uso 6

El administrador necesita saber qué reservas se encuentran activas en este momento.

Una reserva está activa cuando:

```
fecha_hora_inicio <= momento actual

y

fecha_hora_fin >= momento actual
```

El estudiante debe construir la condición completa.

------

# Parte 3. Intervalos

## Caso de uso 7

La empresa desea conocer cuánto dura cada reserva.

El resultado debe mostrar:

```
cliente
sala
duracion
```

La duración debe calcularse a partir de:

```
fecha_hora_fin
-
fecha_hora_inicio
```

------

## Caso de uso 8

La empresa considera que una reserva es larga cuando dura más de 2 horas.

El estudiante debe listar exclusivamente esas reservas.

------

## Caso de uso 9

Por política interna, todas las reservas confirmadas tendrán 30 minutos adicionales.

El estudiante debe mostrar:

```
cliente
fecha_hora_fin
nueva_fecha_hora_fin
```

sin modificar todavía los datos de la tabla.

------

## Caso de uso 10

Una reserva puede cancelarse gratuitamente hasta 24 horas antes de su inicio.

El estudiante debe calcular para cada reserva:

```
fecha_hora_inicio
fecha_limite_cancelacion
```

------

# Parte 4. Extracción de componentes

## Caso de uso 11

El área de estadísticas necesita saber a qué hora del día comienzan las reservas.

El estudiante debe mostrar:

```
cliente
hora_inicio
```

La hora debe extraerse de `fecha_hora_inicio`.

------

## Caso de uso 12

El administrador desea visualizar:

```
cliente
año
mes
día
hora
```

a partir de `fecha_hora_inicio`.

------

## Caso de uso 13

La empresa desea saber cuántas reservas existen por mes.

El estudiante deberá agrupar las reservas usando el mes y el año.

Resultado esperado conceptualmente:

```
año | mes | cantidad_reservas
```

------

# Parte 5. Consultas por períodos

## Caso de uso 14

El gerente quiere ver las reservas pertenecientes al mes actual.

La consulta debe funcionar automáticamente sin importar el mes en que se ejecute.

------

## Caso de uso 15

El sistema necesita consultar todas las reservas creadas durante el año actual.

No debe escribirse manualmente el año.

------

## Caso de uso 16

El administrador quiere saber cuáles reservas corresponden a la semana actual.

Se recomienda investigar el uso de:

```
DATE_TRUNC
```

------

# Parte 6. Formato de fechas

## Caso de uso 17

En un reporte para usuarios finales, la fecha y hora de inicio debe presentarse así:

```
15/09/2026 08:00
```

El estudiante debe construir la consulta usando `TO_CHAR`.

------

## Caso de uso 18

El sistema debe mostrar un reporte con este formato:

```
Ana Torres | Sala A | 15/09/2026 08:00
```

El estudiante debe combinar columnas y aplicar formato a la fecha.

------

# Parte 7. Zonas horarias

## Caso de uso 19

La empresa tiene clientes en Colombia y España.

El administrador quiere ver para cada reserva:

```
cliente
hora_colombia
hora_madrid
```

El estudiante debe realizar la conversión utilizando:

```
AT TIME ZONE
```

Zonas sugeridas:

```
America/Bogota
Europe/Madrid
```

------

## Caso de uso 20

El área técnica necesita ver todas las fechas de inicio expresadas en UTC.

El estudiante debe transformar `fecha_hora_inicio`.

------

# Parte 8. Cálculos con fechas

## Caso de uso 21

El administrador desea saber cuántos días faltan para cada reserva.

Resultado esperado:

```
cliente
fecha_reserva
dias_faltantes
```

Las reservas pasadas pueden producir valores negativos.

------

## Caso de uso 22

El sistema necesita identificar reservas creadas con al menos 5 días de anticipación.

Se debe comparar:

```
fecha_reserva
```

contra:

```
fecha_creacion
```

El estudiante deberá pensar cuidadosamente qué tipos de datos necesita convertir.

------

## Caso de uso 23

La empresa desea identificar reservas realizadas con menos de 24 horas de anticipación.

Estas se consideran reservas de último minuto.

------

# Parte 9. AGE

## Caso de uso 24

La empresa quiere conocer cuánto tiempo ha transcurrido desde que se creó cada reserva hasta su fecha de inicio.

El estudiante debe utilizar:

```
AGE()
```

Resultado:

```
cliente
tiempo_anticipacion
```

------

# Parte 10. Consultas combinadas

## Caso de uso 25

Se necesita identificar reservas que cumplan simultáneamente:

```
estado = CONFIRMADA
duración mayor a 2 horas
fecha posterior a hoy
```

El estudiante debe combinar las tres condiciones.

------

## Caso de uso 26

La empresa desea obtener las reservas futuras de la `Sala A`, ordenadas desde la más próxima hasta la más lejana.

------

## Caso de uso 27

El gerente desea conocer la reserva futura más cercana.

El estudiante debe devolver solo un registro.

------

## Caso de uso 28

Se desea conocer la reserva futura más lejana.

------

# Parte 11. Agregaciones

## Caso de uso 29

Calcular la duración promedio de las reservas.

------

## Caso de uso 30

Calcular la duración total de todas las reservas.

------

## Caso de uso 31

Mostrar la cantidad de reservas realizadas por día.

Resultado:

```
fecha
cantidad_reservas
```

------

## Caso de uso 32

Mostrar la cantidad de reservas por sala durante septiembre de 2026.

Resultado:

```
sala
cantidad_reservas
```

------

# Parte 12. Reto de actualización

Hasta aquí las consultas han sido principalmente `SELECT`.

Ahora se realizarán modificaciones.

## Caso de uso 33

Por mantenimiento, todas las reservas futuras de la `Sala A` deben comenzar una hora más tarde.

El estudiante debe construir un:

```
UPDATE
```

Antes de ejecutarlo, debe construir primero el `SELECT` equivalente para comprobar qué registros serán modificados.

------

## Caso de uso 34

Todas las reservas pendientes deben extender su hora de finalización 30 minutos.

------

## Caso de uso 35

Cambiar a:

```
FINALIZADA
```

todas las reservas cuya `fecha_hora_fin` ya haya pasado.

------

# Parte 13. Reto integrador

Una nueva regla de negocio indica:

> Las reservas confirmadas que duren más de 3 horas y comiencen durante los próximos 15 días deben considerarse reservas prioritarias.

Construye una consulta que muestre:

```
id
cliente
sala
fecha_hora_inicio
fecha_hora_fin
duracion
```

y que únicamente incluya las reservas que cumplan las tres condiciones.

El estudiante deberá decidir cómo utilizar:

```
CURRENT_TIMESTAMP
INTERVAL
operadores relacionales
```

------

# Parte 14. Caso de uso avanzado

La empresa necesita un reporte gerencial de reservas con las siguientes columnas:

```
cliente
sala
fecha_formateada
hora_inicio
duracion
dias_para_reserva
estado
```

Reglas:

- `fecha_formateada`: `DD/MM/YYYY`
- `hora_inicio`: únicamente hora y minutos
- `duracion`: diferencia entre inicio y fin
- `dias_para_reserva`: diferencia respecto a la fecha actual
- ordenar cronológicamente por `fecha_hora_inicio`

El estudiante debe resolver toda la consulta sin instrucciones SQL adicionales.

------

# Reglas del taller

El estudiante no debe modificar los datos iniciales salvo en los ejercicios que explícitamente solicitan `UPDATE`.

Antes de cualquier actualización debe ejecutar:

```
SELECT ...
```

para verificar qué registros se verán afectados.

No se permite escribir la fecha actual manualmente cuando el caso solicite información relativa al momento de ejecución.

Se deben utilizar correctamente los tipos y funciones de PostgreSQL.

------

# Entrega

Entregar un archivo:

```
taller_fechas_casos_uso.sql
```

con esta estructura:

```
-- =====================================================
-- Caso de uso 1
-- Reservas posteriores al 16 de septiembre de 2026
-- =====================================================

SELECT ...;


-- =====================================================
-- Caso de uso 2
-- Reservas entre dos fechas
-- =====================================================

SELECT ...;
```

Cada solución debe incluir un comentario que explique brevemente la lógica utilizada.

## Criterios de evaluación

| Criterio                                 | Peso |
| ---------------------------------------- | ---- |
| Interpretación correcta del caso de uso  | 25%  |
| Uso adecuado de tipos de fecha/hora      | 20%  |
| Uso correcto de funciones e intervalos   | 20%  |
| Construcción correcta de condiciones     | 15%  |
| Calidad y legibilidad de SQL             | 10%  |
| Comentarios y explicación de la solución | 10%  |

# Funciones Agregadas https://www.postgresql.org/docs/9.5/functions-aggregate.html

1. Cree una BD
2. Haga volcado de los datos https://gist.github.com/trainingLeader/51d7aab31945e7c8f63f0d9dd098aa61 

## Between

`BETWEEN` en **PostgreSQL**. Es un operador lógico que se usa para filtrar resultados dentro de un **rango de valores**, y **sí incluye los límites**.

```
valor BETWEEN mínimo AND máximo
```

```sql
SELECT * FROM productos
WHERE precio BETWEEN 100 AND 200;
```

```sql
SELECT * FROM pedidos
WHERE fecha_pedido BETWEEN '2024-01-01' AND '2024-12-31';
```

```sql
SELECT * FROM productos
WHERE precio NOT BETWEEN 100 AND 200;
```

```sql
SELECT
    first_name,
    last_name,
    followers
FROM
    users
WHERE
    followers BETWEEN 4600 AND 4700
ORDER BY
    followers DESC;

```

## COUNT, MAX,MIN,ROUND AVG

### `COUNT()`

Cuenta el **número de filas**.

```sql
SELECT COUNT(followers) FROM users;
```

###  `MAX()`

Devuelve el **valor máximo** de una columna.

```sql
SELECT MAX(followers) FROM users;
```

###  `MIN()`

Devuelve el **valor mínimo** de una columna.

```sql
SELECT MIN(followers) FROM users;
```

### `AVG()`

Calcula el **promedio** (media aritmética) de los valores numéricos.

```sql
SELECT AVG(followers) FROM users;
```

### `ROUND()`

Redondea un número a los decimales que se le indique.

```sql
SELECT ROUND(AVG(followers), 2) FROM users;
```

```sql
SELECT country, COUNT(*) AS total_users,
       MAX(followers) AS top_followers,
       MIN(followers) AS least_followers,
       ROUND(AVG(followers), 1) AS avg_followers
FROM users
GROUP BY country;
```

### **Ejemplo completo con `HAVING`:**

Supongamos que tienes la tabla `users` con las columnas: `id`, `first_name`, `last_name`, `country`, `followers`.

Y quieres saber:

- Cuántos usuarios hay por país.
- El máximo, mínimo y promedio de seguidores.
- Pero **solo mostrar los países que tienen más de 5 usuarios y un promedio de seguidores mayor a 1000**.

```sql
SELECT 
    country,
    COUNT(*) AS total_users,
    MAX(followers) AS top_followers,
    MIN(followers) AS least_followers,
    ROUND(AVG(followers), 2) AS avg_followers
FROM 
    users
GROUP BY 
    country
HAVING 
    COUNT(*) > 5 
    AND AVG(followers) > 1000
ORDER BY 
    avg_followers DESC;
```

## Group By

Esta cláusula en PostgreSQL se usa para **agrupar filas** que tienen el mismo valor en una o más columnas, y se usa en combinación con funciones de agregación como `COUNT()`, `SUM()`, `AVG()`, `MAX()`, `MIN()`.

```sql
SELECT columna1, AGREGADO(columna2)
FROM tabla
GROUP BY columna1;
```

Usuarios por pais

```sql
SELECT country, COUNT(*) AS total_users
FROM users
GROUP BY country;
```

`GROUP BY` + `HAVING` (filtrar después del agrupamiento):

```sql
SELECT 
    country,
    COUNT(*) AS total_users
FROM users
GROUP BY country
HAVING COUNT(*) > 10;
```

> ### 🧠 Regla de oro:
>
> Si usas `GROUP BY`, **todas las columnas del `SELECT` deben estar en**:
>
> - `GROUP BY`, **o**
> - Ser una función de agregación.

```sql
SELECT COUNT(*), followers
FROM users
WHERE followers = 4 OR followers = 4999
GROUP BY followers;
```

```sql
SELECT COUNT(*), followers
FROM users
WHERE followers BETWEEN 4500 AND 4999
GROUP BY followers
ORDER BY followers DESC;
```

## Having

`HAVING` es una cláusula SQL que se usa **después de un `GROUP BY`** para **filtrar los resultados agrupados**, especialmente cuando usamos funciones de agregación como:

- `COUNT()`
- `SUM()`
- `AVG()`
- `MAX()`
- `MIN()`

> 🧨 **`WHERE` no puede usar funciones de agregación**, pero `HAVING` sí.

### ¿Cuándo usar `HAVING`?

Cuando quieres **agrupar datos** y **mostrar solo los grupos que cumplen una condición** sobre esos datos agregados.

**¿Qué productos vendieron más de 15 unidades en total?**

Primero agrupamos por `producto`:

```sql
SELECT producto, SUM(cantidad) AS total_vendido
FROM ventas
GROUP BY producto;
```

Ahora, **filtramos esos grupos** usando `HAVING`:

```sql
SELECT producto, SUM(cantidad) AS total_vendido
FROM ventas
GROUP BY producto
HAVING SUM(cantidad) > 15;
```

```sql
SELECT 
    COUNT(*), 
    country
FROM 
    users
GROUP BY 
    country
HAVING 
    COUNT(*) > 5
ORDER BY 
    COUNT(*) DESC;
```

## Distinct

`DISTINCT` es una cláusula que sirve para **eliminar los duplicados** del resultado de una consulta. En otras palabras: **te da solo los valores únicos** de las columnas que selecciones.

```sql
SELECT DISTINCT columna1, columna2
FROM tabla;
```

```sql
SELECT DISTINCT name, country FROM users;
```

### Combinando `DISTINCT` con funciones

Cuenta la cantidad de valores **únicos** en una columna.

```sql
SELECT COUNT(DISTINCT country) FROM users;
```

### ⚠️ A tener en cuenta

1. Si usas `DISTINCT` con varias columnas, se eliminan solo si **la combinación completa se repite**.
2. No se puede usar `DISTINCT` con `ORDER BY` sobre columnas que no están en el `SELECT`, a menos que las agregues ahí también.
3. `DISTINCT` va justo **después de `SELECT`**.



```sql
SELECT
	email,
    SUBSTRING(email, POSITION('@' IN email) + 1) AS domain
FROM
    users
GROUP BY email, SUBSTRING(email, POSITION('@' IN email) + 1)
HAVING count(*) > 1;
```



# Volcado de datos

Elimine las tablas country, region, city. Tenga en cuenta que la eliminación la debe realizar respetando la relacion y las llaves foraneas.

```sql
DROP TABLE city;
DROP TABLE region;
DROP TABLE country;
```

```
               Listado de relaciones
 Esquema |      Nombre      |   Tipo    |  Due±o
---------+------------------+-----------+----------
 public  | empleados        | tabla     | postgres
 public  | empleados_id_seq | secuencia | postgres
```

Cree las siguientes tablas

```sql
CREATE TABLE "public"."city" (
    "id" int4 NOT NULL,
    "name" text NOT NULL,
    "countrycode" bpchar(3) NOT NULL,
    "district" text NOT NULL,
    "population" int4 NOT NULL CHECK (population >= 0),
    PRIMARY KEY ("id")
);
CREATE TABLE "public"."country" (
    "code" bpchar(3) NOT NULL,
    "name" text NOT NULL,
    "continent" text NOT NULL CHECK ((continent = 'Asia'::text) OR (continent = 'South America'::text) OR (continent = 'North America'::text) OR (continent = 'Oceania'::text) OR (continent = 'Antarctica'::text) OR (continent = 'Africa'::text) OR (continent = 'Europe'::text) OR (continent = 'Central America'::text)),
    "region" text NOT NULL,
    "surfacearea" float4 NOT NULL CHECK (surfacearea >= (0)::double precision),
    "indepyear" int2,
    "population" int4 NOT NULL,
    "lifeexpectancy" float4,
    "gnp" numeric(10,2),
    "gnpold" numeric(10,2),
    "localname" text NOT NULL,
    "governmentform" text NOT NULL,
    "headofstate" text,
    "capital" int4,
    "code2" bpchar(2) NOT NULL,
    PRIMARY KEY ("code")
);
CREATE TABLE "public"."countrylanguage" (
    "countrycode" bpchar(3) NOT NULL,
    "language" text NOT NULL,
    "isofficial" bool NOT NULL,
    "percentage" float4 NOT NULL CHECK ((percentage >= (0)::double precision) AND (percentage <= (100)::double precision)),
    PRIMARY KEY ("countrycode","language")
);
```

Data Inicial : https://gist.github.com/454db994cfff87a9ec542a1e92f21ff6.git

Cree una tabla llamada continent

```sql
CREATE TABLE public.continent (
	code serial4 NOT NULL,
	name text NULL,
	CONSTRAINT continent_pk PRIMARY KEY (code)
);
```

Caso de uso : Se requiere llenar la tabla continent con los continentes que se encuentran en la tabla country

![](https://i.ibb.co/VcrBm5dy/image.png)

Caso de uso : Liste los continentes de la tabla country. Los continentes no se deben repetir.

![](https://i.ibb.co/LhBLfTBh/image.png)

Ejecute el siguiente comando sql para listar e insertar los registros de continentes.

```SQL
INSERT INTO continent (name)
	SELECT DISTINCT continent
	FROM country
	ORDER BY continent ASC;
```

![](https://i.ibb.co/QFPnq25T/image.png)



# Procedimientos almacenados en PostgreSQL

En PostgreSQL, los procedimientos almacenados (también conocidos como funciones almacenadas) son bloques de código que se almacenan y se ejecutan en el servidor de la base de datos. Los procedimientos almacenados permiten encapsular lógica de negocio compleja y reutilizable, mejorando la eficiencia y el mantenimiento del código.

### Características de los Procedimientos Almacenados en PostgreSQL

1. **Encapsulación de Lógica**: Permiten agrupar operaciones lógicas en un solo lugar, lo que facilita la gestión y el mantenimiento.
2. **Reutilización**: Una vez definidos, pueden ser llamados múltiples veces desde diferentes partes de una aplicación.
3. **Rendimiento**: Reducen la cantidad de datos transferidos entre el servidor de la base de datos y la aplicación cliente, mejorando el rendimiento.
4. **Seguridad**: Pueden encapsular operaciones sensibles y restringir el acceso directo a las tablas subyacentes.
5. **Soporte para Transacciones**: Pueden incluir lógica transaccional, permitiendo commit y rollback de operaciones.

### Creación procedimiento almacenado

Utilice la sentencia CREATE OR REPLACE PROCEDURE para definir su procedimiento almacenado. Esta sentencia le permite crear un nuevo procedimiento o reemplazar uno existente. A continuación se muestra una plantilla básica

```plsql
CREATE OR REPLACE PROCEDURE procedure_name(parameter1 datatype, parameter2 datatype)
LANGUAGE plpgsql
AS $$
DECLARE
    -- Declare local variables if needed
BEGIN
    -- Your SQL logic here
EXCEPTION
    WHEN OTHERS THEN
        -- Handle exceptions if needed
        RAISE EXCEPTION 'exception';
END;
$$;
```

| Parte                                      | Descripción                                                  |
| ------------------------------------------ | ------------------------------------------------------------ |
| `CREATE OR REPLACE PROCEDURE`              | Crea o reemplaza un procedimiento con el nombre dado.        |
| `procedure_name(param1 type, param2 type)` | Define el nombre del procedimiento y sus parámetros de entrada. |
| `LANGUAGE plpgsql`                         | Indica que el procedimiento usará el lenguaje procedural PL/pgSQL (el más común en PostgreSQL). |
| `DECLARE`                                  | Sección opcional donde defines variables internas del procedimiento. |
| `BEGIN ... END;`                           | El bloque principal donde escribes tu lógica en SQL.         |
| `EXCEPTION ...`                            | Sección para manejar errores que puedan ocurrir durante la ejecución del procedimiento. |
| `RAISE EXCEPTION 'mensaje';`               | Lanza una excepción personalizada. Puedes incluir variables para más detalle. |

Cree una nueva base de datos

```
postgres=# create database ferreunica;
CREATE DATABASE
postgres=#
```

Cree las siguientes tablas  

**Data y DDL de tablas** https://gist.github.com/trainingLeader/71cafd14c9b9efa086041455987339f7

```sql
CREATE TABLE full_order_info (
    order_id        INTEGER,
    product_id      INTEGER,
    quantity        INTEGER,
    price           DOUBLE PRECISION,
    orderdate       DATE,
    user_id         CHARACTER VARYING(20),
    first_name      CHARACTER VARYING(50),
    last_name       CHARACTER VARYING(50),
    email           CHARACTER VARYING(50),
    last_connection CHARACTER VARYING(100),
    website         CHARACTER VARYING(100),
    name            CHARACTER VARYING(50),
    description     TEXT,
    stock           INTEGER,
    stock_price     DOUBLE PRECISION,
    stockmin        INTEGER,
    stockmax        INTEGER
);
CREATE TABLE users (
    "id" varchar(20),
    "first_name" character varying(50) NOT NULL,
    "last_name" character varying(50) NOT NULL,
    "email" character varying(50) NOT NULL UNIQUE,
    "last_connection" character varying(100) NOT NULL,
    "website" character varying(100) NOT NULL,
    PRIMARY KEY ("id")
);

CREATE TABLE products (
    "id" serial,
    "name" character varying(50) NOT NULL,
    "description" TEXT,
    "stock" integer CHECK (stock > 0),
    "price" double precision CHECK (price > 0),
    "stockmin" integer,
    "stockmax" integer,
    PRIMARY KEY ("id")
);

CREATE TABLE orders (
    "id" serial,
    "orderdate" date,
    "user_id" character varying(50) NOT NULL,
    PRIMARY KEY ("id")
);

CREATE TABLE order_details (
    "id" serial,
    "order_id" integer, 
    "product_id" integer,
    "quantity" smallint,
    "price" double precision CHECK (price > 0),
    PRIMARY KEY ("id")
);

ferreunica=# \d
                 Listado de relaciones
 Esquema |        Nombre        |   Tipo    |  Due±o
---------+----------------------+-----------+----------
 public  | order_details        | tabla     | postgres
 public  | order_details_id_seq | secuencia | postgres
 public  | orders               | tabla     | postgres
 public  | orders_id_seq        | secuencia | postgres
 public  | products             | tabla     | postgres
 public  | products_id_seq      | secuencia | postgres
 public  | users                | tabla     | postgres
(7 filas)

ALTER TABLE orders ADD CONSTRAINT fk_customerid FOREIGN KEY (user_id) REFERENCES users(id);
ALTER TABLE order_details ADD CONSTRAINT fk_orderid FOREIGN KEY (order_id) REFERENCES orders(id);
ALTER TABLE order_details ADD CONSTRAINT fk_productid FOREIGN KEY (product_id) REFERENCES products(id);

INSERT INTO users(id, first_name, last_name, email, last_connection, website)
SELECT DISTINCT user_id, first_name, last_name, email, last_connection, website
FROM full_order_info;

INSERT INTO products(name, description, stock, price, stockmin, stockmax)
SELECT DISTINCT name, description, stock_price, price, stockmin, stockmax
FROM full_order_info;

INSERT INTO orders(id, orderdate, user_id)
SELECT DISTINCT order_id, orderdate, user_id
FROM full_order_info;

INSERT INTO order_details(order_id, product_id, quantity, price)
SELECT DISTINCT order_id, product_id, quantity, price
FROM full_order_info;

```

![](https://i.ibb.co/6c04HpmJ/image.png)

Listar lar ordenes con su total

```sql
SELECT 
    o.id AS order_id,
    o.orderdate,
    u.id,
    CONCAT(u.first_name, ' ', u.last_name) AS customer,
    ROUND(SUM(od.quantity::numeric * od.price::numeric), 2) AS total_value
FROM 
    orders o
JOIN 
    users u ON o.user_id = u.id
JOIN 
    order_details od ON o.id = od.order_id
GROUP BY 
    o.id, o.orderdate,u.id, u.first_name, u.last_name
ORDER BY 
    total_value DESC;
```

```plsql
CREATE OR REPLACE PROCEDURE calculate_total_amount_by_customer(user_id_args VARCHAR(20))
LANGUAGE plpgsql
AS $$
DECLARE
    total NUMERIC := 0;
BEGIN
    SELECT 
        ROUND(SUM(od.quantity::numeric * od.price::numeric), 2)
    INTO total
    FROM 
        orders o
    JOIN 
        order_details od ON o.id = od.order_id
    WHERE o.user_id = user_id_args;

    RAISE NOTICE 'Total amount spent by customer %: %', user_id_args, COALESCE(ROUND(total, 2), 0);

EXCEPTION
    WHEN OTHERS THEN
        RAISE EXCEPTION 'Error calculating total amount for customer %', user_id_args;
END;
$$;
```

```sql
CALL calculate_total_amount_by_customer('00006');
CALL calculate_total_amount_by_customer('6');
```

Ejemplos varios

**Registrar una nueva orden con sus detalles**

```plsql
CREATE OR REPLACE PROCEDURE create_order_with_details(
    p_user_id VARCHAR(20),
    p_order_date DATE,
    p_product_ids INTEGER[],
    p_quantities INTEGER[],
    p_prices FLOAT8[]
)
LANGUAGE plpgsql
AS $$
DECLARE
    new_order_id INTEGER;
BEGIN
    -- Insertar la orden
    INSERT INTO orders (orderdate, user_id)
    VALUES (p_order_date, p_user_id)
    RETURNING id INTO new_order_id;

    -- Insertar detalles
    FOR i IN 1..array_length(p_product_ids, 1) LOOP
        INSERT INTO order_details (order_id, product_id, quantity, price)
        VALUES (new_order_id, p_product_ids[i], p_quantities[i], p_prices[i]);
    END LOOP;

    RAISE NOTICE 'Orden creada con ID: %', new_order_id;
END;
$$;

```

```sql
CALL create_order_with_details(
    '1',
    CURRENT_DATE,
    ARRAY[1,2,3],
    ARRAY[2,1,4],
    ARRAY[1200.50, 300.00, 45.99]
);

CALL create_order_with_details(
    '3',
    CURRENT_DATE,
    ARRAY[5,2,6],
    ARRAY[2,1,4],
    ARRAY[1200.50, 300.00, 45.99]
);
```

**Calcular el total gastado por un cliente**

```plsql
CREATE OR REPLACE PROCEDURE total_amount_by_user(
    p_user_id VARCHAR(20)
)
LANGUAGE plpgsql
AS $$
DECLARE
    total NUMERIC;
BEGIN
    SELECT SUM(od.quantity::numeric * od.price::numeric)
    INTO total
    FROM orders o
    JOIN order_details od ON o.id = od.order_id
    WHERE o.user_id = p_user_id;

    RAISE NOTICE 'El total gastado por el usuario % es: %', p_user_id, COALESCE(ROUND(total, 2), 0);
END;
$$;

CALL total_amount_by_user('6');
```

```sql
SELECT COALESCE(SUM(od.price * od.quantity), 0) as total
FROM orders o
JOIN order_details od ON o.id = od.order_id
WHERE o.user_id = '00030';
```

`COALESCE()` en SQL sirve para **devolver el primer valor no nulo** entre una lista de expresiones.

# 🧪 Taller: Procedimientos Almacenados en PostgreSQL

## 🎯 Objetivo

Aplicar el uso de procedimientos almacenados para manipular y consultar datos en un sistema de gestión de ventas.

## 🧱 Tablas involucradas

- `users(id, first_name, last_name, email, last_connection, website)`
- `products(id, name, description, stock, price, stockmin, stockmax)`
- `orders(id, orderdate, user_id)`
- `order_details(id, order_id, product_id, quantity, price)`

------

## 🔥 **Reto 1: Crear una orden con múltiples productos**

### 📘 Enunciado:

Crear un procedimiento `create_order_with_details` que reciba un ID de usuario, una fecha, y tres arrays con los IDs de productos, cantidades y precios. Debe crear la orden y registrar sus detalles.

## 🔥 **Reto 2: Calcular total de una orden**

### 🎯 Enunciado:

Crea un procedimiento que reciba el ID de una orden y muestre el total gastado.

## 🔥 **Reto 3: Mostrar órdenes de un usuario entre dos fechas**

### 🎯 Enunciado:

Crea un procedimiento que reciba el ID del usuario y dos fechas, y muestre todas las órdenes realizadas por el usuario en ese rango.

## 🔥 **Reto 4: Registrar un nuevo producto si no existe**

### 🎯 Enunciado:

Crea un procedimiento que reciba nombre, descripción, precio y stock. Si el nombre del producto ya existe, debe lanzar una excepción.

## 🔥 **Reto 5: Cancelar una orden**

### 🎯 Enunciado:

Crea un procedimiento que **elimine una orden** y sus detalles, y devuelva el stock a los productos.

# Funciones Almacenadas

Las funciones en PostgreSQL (también llamadas **funciones almacenadas** o **stored functions**) permiten encapsular lógica reutilizable que se puede ejecutar desde una consulta SQL. Son muy útiles para automatizar tareas, validar datos o realizar cálculos complejos en el servidor.

Sintaxis

```plsql
CREATE [OR REPLACE] FUNCTION nombre_función(parámetros)
RETURNS tipo_retorno AS $$
BEGIN
  -- lógica aquí
  RETURN valor;
END;
$$ LANGUAGE plpgsql;
```

Ejemplo 1: función simple que suma dos números

```plsql
CREATE OR REPLACE FUNCTION sumar(a INTEGER, b INTEGER)
RETURNS INTEGER AS $$
BEGIN
  RETURN a + b;
END;
$$ LANGUAGE plpgsql;

SELECT sumar(5, 7);  -- Resultado: 12

```

Ejemplo 2: función que retorna un mensaje personalizado

```plsql
CREATE OR REPLACE FUNCTION saludar(nombre TEXT)
RETURNS TEXT AS $$
BEGIN
  RETURN 'Hola ' || nombre || ', bienvenido a PostgreSQL!';
END;
$$ LANGUAGE plpgsql;

SELECT saludar('Johlver');
```

Ejemplo 3: función que consulta datos de una tabla

```sql
CREATE OR REPLACE FUNCTION obtener_precio_producto(id_producto INT) RETURNS NUMERIC AS $$
DECLARE
  precio NUMERIC;
BEGIN
  SELECT price INTO precio
  FROM products
  WHERE id = id_producto;

  RETURN precio;
END;
$$ LANGUAGE plpgsql;

SELECT obtener_precio_producto(2);

```

## ¿`FOR` en una función?

El `FOR` se usa para **repetir una serie de instrucciones** dentro de una función. Puedes usarlo para recorrer:

1. Un **rango de números** (tipo contador)
2. El resultado de una **consulta SELECT**
3. Un **cursor** (más detallado)

## 🔢 1. FOR tipo contador (numérico)

Se usa para iterar desde un número inicial hasta uno final.

```plsql
FOR variable IN [REVERSE] inicio..fin LOOP
  -- instrucciones
END LOOP;

```

```plsql
CREATE OR REPLACE FUNCTION contar_hasta(n INT)
RETURNS VOID AS $$
DECLARE
  i INT;
BEGIN
  FOR i IN 1..n LOOP
    RAISE NOTICE 'i = %', i;
  END LOOP;
END;
$$ LANGUAGE plpgsql;
```

## 🗃️ 2. FOR sobre una consulta (tipo FOREACH de una tabla)

Recorre cada fila que devuelve un `SELECT`.

Sintaxis

```plsql
FOR fila_variable IN SELECT ... LOOP
  -- usar fila_variable.columna
END LOOP;

```

```plsql
CREATE OR REPLACE FUNCTION listar_usuarios()
RETURNS VOID AS $$
DECLARE
  usuario RECORD;
BEGIN
  FOR usuario IN SELECT id, first_name FROM users LOOP
    RAISE NOTICE 'Usuario: % - ID: %', usuario.first_na, usuario.id;
  END LOOP;
END;
$$ LANGUAGE plpgsql;
---------------------------------------------------------------------------------------------------

CREATE OR REPLACE FUNCTION obtener_nombre_precio_producto(id_producto INT) 
RETURNS TABLE(nombre VARCHAR, precio double precision) AS $$
BEGIN
	RETURN QUERY SELECT name,price FROM products WHERE id = id_producto;
END;
$$ LANGUAGE plpgsql;

---------------------------------------------------------------------------------------------------
```

Insertar datos

```plsql
CREATE FUNCTION insertar_empleado(nombre VARCHAR, edad INTEGER, salario NUMERIC)
RETURNS VOID AS $$
BEGIN
    INSERT INTO empleados (nombre, edad, salario) VALUES (nombre, edad, salario);
END;
$$ LANGUAGE plpgsql;

SELECT insertar_empleado('Juan Perez', 30, 3000.00);
```

```sql
CREATE TABLE empleados (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(50),
    edad INTEGER,
    salario NUMERIC
);
```

```plsql
CREATE FUNCTION obtener_todos_los_empleados()
RETURNS TABLE(id INTEGER, nombre VARCHAR, edad INTEGER, salario NUMERIC) AS $$
BEGIN
    RETURN QUERY SELECT id, nombre, edad, salario FROM empleados;
END;
$$ LANGUAGE plpgsql;
```

### Retornar Registros con una Condición

Vamos a crear una funcion almacenada que retorne todos los empleados con un salario mayor a un valor especificado.

```plsql
CREATE FUNCTION obtener_empleados_con_salario_mayor_a(salario_minimo NUMERIC)
RETURNS TABLE(id INTEGER, nombre VARCHAR, edad INTEGER, salario NUMERIC) AS $$
BEGIN
    RETURN QUERY 
    SELECT id, nombre, edad, salario 
    FROM empleados 
    WHERE salario > salario_minimo;
END;
$$ LANGUAGE plpgsql;
```

### Retornar Registros con JOIN

Supongamos que tenemos otra tabla llamada `departamentos`:

```sql
CREATE TABLE departamentos (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(100)
);
```

Y una tabla intermedia `empleados_departamentos` que relaciona empleados con departamentos:

```sql
CREATE TABLE empleados_departamentos (
    empleado_id INTEGER REFERENCES empleados(id),
    departamento_id INTEGER REFERENCES departamentos(id)
);
```

Vamos a crear un procedimiento almacenado que retorne los empleados junto con sus departamentos.

```plsql
CREATE FUNCTION obtener_empleados_con_departamentos()
RETURNS TABLE(empleado_nombre VARCHAR, departamento_nombre VARCHAR) AS $$
BEGIN
    RETURN QUERY 
    SELECT e.nombre, d.nombre 
    FROM empleados e
    JOIN empleados_departamentos ed ON e.id = ed.empleado_id
    JOIN departamentos d ON ed.departamento_id = d.id;
END;
$$ LANGUAGE plpgsql;
```

### Invocación de Funciones Almacenadas

Para invocar estos procedimientos y obtener los resultados, utilizamos una sentencia `SELECT`:

```
SELECT * FROM obtener_todos_los_empleados();
SELECT * FROM obtener_empleados_con_salario_mayor_a(3000);
SELECT * FROM obtener_empleados_con_departamentos();
```

# Triggers

Un **trigger** (disparador) es un mecanismo que **ejecuta automáticamente una función** en respuesta a eventos como:

- `INSERT`
- `UPDATE`
- `DELETE`
- `TRUNCATE`

Se usa para **automatizar acciones**, **validaciones**, **auditoría**, o **modificar registros relacionados**.

## ⚙️ Estructura general

Un trigger se compone de:

1. Una **función trigger**
2. Una **sentencia CREATE TRIGGER** que la asocia a una tabla y evento

## 🧰 Tipos de triggers

| Tipo            | ¿Cuándo se ejecuta?                      |
| --------------- | ---------------------------------------- |
| `BEFORE INSERT` | Antes de insertar un registro            |
| `AFTER INSERT`  | Después de insertar un registro          |
| `BEFORE UPDATE` | Antes de actualizar un registro          |
| `AFTER UPDATE`  | Después de actualizar un registro        |
| `BEFORE DELETE` | Antes de eliminar un registro            |
| `AFTER DELETE`  | Después de eliminar un registro          |
| `INSTEAD OF`    | Para vistas, reemplaza el comportamiento |

```sql
CREATE TABLE empleados (
  id SERIAL PRIMARY KEY,
  nombre CHARACTER VARYING(50),
  salario NUMERIC
);
CREATE TABLE auditoria_empleados (
  id SERIAL PRIMARY KEY,
  empleado_id INT,
  evento CHARACTER VARYING(50),
  fecha TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Crear la función trigger

```plsql
CREATE OR REPLACE FUNCTION log_auditoria_empleados()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO auditoria_empleados (empleado_id, evento)
  VALUES (
    NEW.id,
    TG_OP  -- 'INSERT', 'UPDATE' o 'DELETE'
  );
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

```

Crear el trigger

```sql
CREATE TRIGGER trg_auditoria
AFTER INSERT OR UPDATE OR DELETE ON empleados
FOR EACH ROW
EXECUTE FUNCTION log_auditoria_empleados();
```

```sql
INSERT INTO empleados (nombre, salario) VALUES ('Pedro', 1500);
UPDATE empleados SET salario = 1800 WHERE id = 1;
DELETE FROM empleados WHERE id = 1;

SELECT * FROM auditoria_empleados;

```

### 🔁 1. **Trigger de actualización automática de timestamp**

```sql
CREATE TABLE productos (
  id SERIAL PRIMARY KEY,
  nombre CHARACTER VARYING(50),
  precio double precision,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

```

⚙️ Función y trigger:

```sql
CREATE OR REPLACE FUNCTION actualizar_timestamp()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at := CURRENT_TIMESTAMP;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_actualizar_timestamp
BEFORE UPDATE ON productos
FOR EACH ROW
EXECUTE FUNCTION actualizar_timestamp();
```

### 🛑 2. **Trigger de validación: no permitir precio negativo**

🧠 **Objetivo**: Evitar que se inserten productos con precios negativos.

```sql
CREATE OR REPLACE FUNCTION validar_precio()
RETURNS TRIGGER AS $$
BEGIN
  IF NEW.precio < 0 THEN
    RAISE EXCEPTION 'El precio no puede ser negativo';
  END IF;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_validar_precio
BEFORE INSERT OR UPDATE ON productos
FOR EACH ROW
EXECUTE FUNCTION validar_precio();

```

### 🗃️ 3. **Trigger para registrar eliminación (soft delete)**

🧠 **Objetivo**: En lugar de eliminar el registro, marcarlo como eliminado.

### 🧱 Tabla con "bandera de eliminado":

```sql
ALTER TABLE productos ADD COLUMN eliminado BOOLEAN DEFAULT FALSE;
```

⚙️ Función y trigger:

```sql
CREATE OR REPLACE FUNCTION soft_delete_producto()
RETURNS TRIGGER AS $$
BEGIN
  UPDATE productos SET eliminado = TRUE WHERE id = OLD.id;
  RETURN NULL;  -- evita la eliminación real
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_soft_delete
BEFORE DELETE ON productos
FOR EACH ROW
EXECUTE FUNCTION soft_delete_producto();

```

## 📦 4. **Trigger para copiar datos a una tabla de respaldo (backup)**

🧠 **Objetivo**: Guardar un respaldo cada vez que se elimine un producto.

### 🧱 Tabla de respaldo

```sql
CREATE TABLE productos_backup (
  id INT,
  nombre CHARACTER VARYING(50),
  precio NUMERIC,
  eliminado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

⚙️ Función y trigger:

```sql
CREATE OR REPLACE FUNCTION respaldo_producto()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO productos_backup (id, nombre, precio)
  VALUES (OLD.id, OLD.nombre, OLD.precio);
  RETURN OLD;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_respaldo_producto
AFTER DELETE ON productos
FOR EACH ROW
EXECUTE FUNCTION respaldo_producto();

```

## 📘 **Taller: Automatización de procesos con Triggers en PostgreSQL**

------

### 🎯 **Objetivo general**

Diseñar e implementar triggers en PostgreSQL para automatizar validaciones, auditoría, cálculos automáticos y control de integridad en una base de datos relacional simulando un sistema de ventas.

------

### ⭐ **Nivel de dificultad**:

⭐⭐⭐☆☆ (Intermedio)

------

### ⏳ **Duración estimada**:

1.5 horas

------

### 🧱 **Especificaciones**

Tendrás 4 tablas principales:

```sql
CREATE TABLE productos (
  id SERIAL PRIMARY KEY,
  nombre TEXT NOT NULL,
  precio NUMERIC NOT NULL CHECK (precio >= 0),
  stock INT NOT NULL DEFAULT 0,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE clientes (
  id SERIAL PRIMARY KEY,
  nombre TEXT NOT NULL
);

CREATE TABLE pedidos (
  id SERIAL PRIMARY KEY,
  cliente_id INT REFERENCES clientes(id),
  total NUMERIC DEFAULT 0,
  creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE detalles (
  id SERIAL PRIMARY KEY,
  pedido_id INT REFERENCES pedidos(id),
  producto_id INT REFERENCES productos(id),
  cantidad INT NOT NULL,
  precio_unitario NUMERIC NOT NULL
);
```

------

## 🔥 **Retos a desarrollar**

------

### ✅ **Reto 1: Actualización automática del campo `updated_at` en `productos`**

**Crea un trigger que actualice la columna `updated_at` cada vez que se modifique un producto.**

------

### ✅ **Reto 2: Cálculo automático del total en `pedidos`**

**Cuando se inserte o actualice un detalle (`detalles`), actualiza el total del pedido relacionado.**

------

### ✅ **Reto 3: Validación de stock disponible**

**Previo a insertar un `detalle`, verifica que el `stock` del producto sea suficiente. Si no, lanza un error.**

------

### ✅ **Reto 4: Auditoría de eliminación de productos**

**Crea una tabla llamada `productos_eliminados` que guarde el producto eliminado con fecha y motivo.**

```sql
CREATE TABLE productos_eliminados (
  id SERIAL,
  producto_id INT,
  nombre CHARACTER VARYING(50),
  precio NUMERIC,
  eliminado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  motivo TEXT
);
```

**El trigger debe insertar un registro antes de eliminar un producto y bloquear la eliminación real.**

------

### ✅ **Reto 5 (bonus): Auditoría de cambios en precios**

**Crea una tabla `historial_precios` y un trigger que registre el precio anterior y nuevo cada vez que se actualice.**

```sql
CREATE TABLE historial_precios (
  id SERIAL PRIMARY KEY,
  producto_id INT,
  precio_anterior NUMERIC,
  precio_nuevo NUMERIC,
  actualizado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

------

## 🧪 **Ejemplo de uso esperado**

```sql
-- Insertar un cliente y un producto
INSERT INTO clientes (nombre) VALUES ('Laura');
INSERT INTO productos (nombre, precio, stock) VALUES ('Mouse Gamer', 80000, 10);

-- Crear pedido y agregar detalle
INSERT INTO pedidos (cliente_id) VALUES (1);
INSERT INTO detalles (pedido_id, producto_id, cantidad, precio_unitario)
VALUES (1, 1, 2, 80000); -- Se debe restar el stock y actualizar el total del pedido

-- Actualizar el precio del producto
UPDATE productos SET precio = 85000 WHERE id = 1;

-- Eliminar el producto
DELETE FROM productos WHERE id = 1; -- No debe borrarlo, sino registrarlo en `productos_eliminados`
```

# Vistas

Una **vista (view)** es como una **tabla virtual** basada en una **consulta SQL**. No almacena datos, pero permite consultarlos como si fuera una tabla.

Sintaxis

```sql
CREATE VIEW nombre_vista AS
SELECT columnas
FROM tabla
WHERE condiciones;

```

## 🧠 Ejemplo práctico

Supongamos que tienes una tabla `orders`:

```sql
CREATE TABLE orders (
  order_id SERIAL PRIMARY KEY,
  user_id INT,
  total NUMERIC,
  created_at DATE
);
```

Y se requiere ver solo los pedidos de este mes:

```sql
CREATE VIEW ordenes_recientes AS
SELECT order_id, user_id, total
FROM orders
WHERE created_at >= date_trunc('month', CURRENT_DATE);

```

## Vistas materializadas

Una **vista materializada** **sí almacena datos físicamente**. Es útil cuando la consulta es muy pesada.

```sql
CREATE MATERIALIZED VIEW ventas_mensuales AS
SELECT user_id, SUM(total) AS total_ventas
FROM orders
GROUP BY user_id;
```

 Para actualizarla manualmente:

```sql
REFRESH MATERIALIZED VIEW ventas_mensuales;
```

🧼 Eliminar vistas

```sql
DROP VIEW ordenes_recientes;
-- o si es materializada:
DROP MATERIALIZED VIEW ventas_mensuales;
```

## 🎯 ¿Cuándo usar una vista?

- Para **simplificar** consultas complejas
- Para aplicar **restricciones de seguridad** (ocultar columnas sensibles)
- Para **reutilizar lógica** de negocio
- Para **optimizar tiempos de consulta** (vistas materializadas)

# Seguridad y usuarios

La gestión de usuarios y permisos es un componente fundamental en cualquier sistema de bases de datos, ya que permite controlar el acceso a la información de forma segura y organizada. En PostgreSQL, esta gestión se realiza a través del sistema de roles, combinando usuarios, grupos y privilegios para definir quién puede hacer qué dentro de la base de datos.

Este módulo tiene como finalidad establecer una estructura clara para la creación, asignación y revocación de permisos utilizando comandos nativos como `CREATE ROLE`, `GRANT`, `REVOKE` y funciones auxiliares en PL/pgSQL. De esta forma, se garantiza el principio de menor privilegio y se refuerza la seguridad en entornos multiusuario o de producción.

## Objetivos

1. **Diseñar un esquema de roles y privilegios personalizado** que permita definir diferentes niveles de acceso según los perfiles de los usuarios.
2. **Implementar funciones administrativas** para facilitar el otorgamiento (`GRANT`) y la revocación (`REVOKE`) de permisos sobre objetos de la base de datos.
3. **Automatizar tareas de seguridad** mediante funciones y scripts reutilizables para la administración de roles en PostgreSQL.
4. **Auditar los permisos actuales** mediante consultas sobre las vistas del sistema como `information_schema.role_table_grants`.

## Resultados esperados

- Una estructura de roles reutilizable que permita la creación de usuarios con acceso restringido según sus funciones.
- Permisos correctamente asignados sobre tablas, funciones y otros objetos según los principios del control de acceso.
- Scripts y funciones documentadas que permitan automatizar la asignación y revocación de privilegios.
- Capacidades de auditoría para verificar quién tiene acceso a qué objetos dentro del sistema de base de datos.

⚙️ 1. Crear roles personalizados (función de administración)

```sql
-- Crear un rol sin acceso por defecto
CREATE ROLE app_user NOLOGIN;

-- Crear un rol con acceso y contraseña
CREATE ROLE juser WITH LOGIN PASSWORD 'mi_contraseña_segura';
GRANT app_user TO juser;

```

🔐 2. Otorgar permisos

Los permisos en PostgreSQL se otorgan con `GRANT` y se revocan con `REVOKE`. Puedes hacerlo a nivel de base de datos, esquema, tabla, función, etc.

```sql
-- Dar SELECT, INSERT, UPDATE a app_user sobre una tabla
GRANT SELECT, INSERT, UPDATE ON public.usuarios TO app_user;
GRANT EXECUTE ON FUNCTION mi_funcion(param_tipo) TO app_user;
```





