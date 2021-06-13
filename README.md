# **UNIVERSIDAD DE SAN CARLOS DE GUATEMALA**
## FACULTAD DE INGENIERIA
## ESCUELA DE CIENCIAS Y SISTEMAS 
## SISTEMAS DE BASES DE DATOS 2
---
## PRACTICA 1
### **201612331**
### **JOSÉ ORLANDO WANNAN ESCOBAR**
---

**INDICE**

- [GESTIÓN DINAMICA DE LA MEMORIA](#GESTIÓN-DINAMICA-DE-LA-MEMORIA)
	- [SGA](#SGA)
	- [PGA](#PGA)
	- [SESSIONS Y PROCESSES](#SESSIONS-Y-PROCESSES)
- [PERMISOS Y AUTENTICACIÓN](#PERMISOS-Y-AUTENTICACIÓN)
	- [CREACION DE TABLESPACE y DATAFILE](#CREACION-DE-TABLESPACE-y-DATAFILE)
	- [USUARIO DEL ESQUEMA](#USUARIO-DEL-ESQUEMA)
	- [ASIGNACION DE PERMISOS](#ASIGNACION-DE-PERMISOS)
	- [CREACION TABLAS PARA ESQUEMA](#CREACION-TABLAS-PARA-ESQUEMA)
	- [CREACIÓN DE ROLES](#CREACIÓN-DE-ROLES)
	- [GESTION DE PERMISOS PARA ROLES](#GESTION-DE-PERMISOS-PARA-ROLES)
	- [CREACIÓN DE USUARIOS](#CREACIÓN-DE-USUARIOS)
	- [DATOS EN TABLAS](#DATOS-EN-TABLAS)
	- [VISTAS](#VISTAS)
- [BACKUP](#BACKUP)



# **GESTIÓN DINAMICA DE LA MEMORIA**

## **SGA**

Oracle recomienda que su espacio de intercambio sea como mínimo tres o cuatro veces el tamaño de su memoria física RAM.

* RAM = 8G
* SGA = RAM/4 → 8/4 = 2G

Para establecer un valor entre el valor minimo de nuestra memoria compartida estableceremos el valor en 1G

Debido a que SGA_TARGET no debe ser mayor a MEMORY_TARGET
* **MEMORY_TARGET** = 1G
* **SGA_MAX_SIZE** = 1G
* **SGA_TARGET** = 750M

```sql

ALTER SYSTEM SET SGA_TARGET=750M scope=spfile;

```

## **PGA**

```sql

ALTER SYSTEM SET pga_aggregate_target=104857600 scope=spfile;
```

## **SESSIONS Y PROCESSES**

Si la aplicación frecuentemente presenta errores que indican que se ha excedido en número máximo de sesiones en la base de datos, se sugiere ampliar los parámetros: processes y sessions.

```sql

ALTER SYSTEM SET processes=1500 scope=spfile;
ALTER SYSTEM SET sessions=500 scope=spfile;

```

Al finalizar todas las configuraciones es necesario ejecutar un reinicio del servicio de Oracle.

```sql

STARTUP FORCE;

```

# **PERMISOS Y AUTENTICACIÓN**

## **CREACION DE TABLESPACE y DATAFILE**

```sql

CREATE TABLESPACE ELECCIONESTBS
DATAFILE 'C:\oraclexe\app\oracle\tablespacess\ELECCIONESDTF.tbs'
SIZE 250M
AUTOEXTEND ON NEXT 250M
MAXSIZE 580M;

```

## **USUARIO DEL ESQUEMA** 

```sql

CREATE USER Elecciones PROFILE DEFAULT IDENTIFIED BY Elecciones 
DEFAULT TABLESPACE ELECCIONESTBS 
TEMPORARY TABLESPACE TEMP 
ACCOUNT UNLOCK;

```

## **ASIGNACION DE PERMISOS**

```sql

GRANT CONNECT TO Elecciones;
GRANT RESOURCE TO Elecciones;
ALTER USER Elecciones QUOTA UNLIMITED ON ELECCIONESTBS;
GRANT ALL PRIVILEGES TO Elecciones;

```

## **CREACION TABLAS PARA ESQUEMA**

```sql

CREATE TABLE DEPARTAMENTO(
	CODIGO_DEPTO NUMBER NOT NULL, 
	NOMBRE_DEPTO VARCHAR(100), 
	CONSTRAINT DEPTA_PK PRIMARY KEY (CODIGO_DEPTO)
) TABLESPACE ELECCIONESTBS ;

CREATE TABLE MUNICIPIO(
	CODIGO_MUNI NUMBER NOT NULL, 
	DEPTO_MUNI NUMBER NOT NULL, 
	NOMBRE_MUNI VARCHAR(100), 
	CONSTRAINT MUNI_PK PRIMARY KEY (CODIGO_MUNI, DEPTO_MUNI),
	CONSTRAINT DEPTO_MUNI_FK
    FOREIGN KEY (DEPTO_MUNI)
    REFERENCES DEPARTAMENTO(CODIGO_DEPTO)
) TABLESPACE ELECCIONESTBS;

CREATE TABLE PARTIDO(
	CODIGO_PART NUMBER NOT NULL, 
	NOMBRE_PART VARCHAR(100), 
	CONSTRAINT PART_PK PRIMARY KEY (CODIGO_PART)
) TABLESPACE ELECCIONESTBS;

CREATE  TABLE ELECCION(
	CODIGO_ELE NUMBER NOT NULL, 
	NOMBRE_ELE VARCHAR(100), 
	CONSTRAINT ELE_PK PRIMARY KEY (CODIGO_ELE)
) TABLESPACE ELECCIONESTBS;

CREATE TABLE ACTA(
	NUMERO_MESA NUMBER NOT NULL, 
	TIPO_ELECCION NUMBER NOT NULL, 
	DEPARTAMENTO NUMBER NOT NULL,
	MUNICIPIO NUMBER NOT NULL, 
	PAPELETAS_RECIBIDAS NUMBER, 
	TOTAL_VOTOS_VALIDOS NUMBER, 
	VOTOS_NULOS NUMBER, 
	VOTOS_BLANCO NUMBER, 
	VOTOS_VALIDOS_EMITIDOS  NUMBER,
	VOTOS_INVALIDOS NUMBER, 
	IMAGEN BLOB, 
	ESTADO_IMAGEN VARCHAR(3), -- VAL, ANU, ACT
	CUADRA_ACTA VARCHAR(1), -- S o N 
	CONTEO_IMPUGNA NUMBER, 
	CONTEO_INSCRITOS NUMBER, 
	STACTA VARCHAR(3), -- VAL, ANU, ACT
	STESCAN VARCHAR(3), 
	STSFISIC VARCHAR(3), 
	STSTRANS VARCHAR(3), 
	CONSTRAINT ACTA_PK PRIMARY KEY (NUMERO_MESA, TIPO_ELECCION),
	CONSTRAINT DEPTO_ACTA_FK FOREIGN KEY (DEPARTAMENTO, MUNICIPIO) REFERENCES MUNICIPIO(DEPTO_MUNI, CODIGO_MUNI), 
	CONSTRAINT ELE_ACTA_FK FOREIGN KEY (TIPO_ELECCION) REFERENCES ELECCION(CODIGO_ELE)
) TABLESPACE ELECCIONESTBS;

CREATE TABLE VOTO(
	VOTO_PARTIDO NUMBER NOT NULL, 
	VOTO_MESA NUMBER NOT NULL, 
	VOTO_ELECCION NUMBER NOT NULL, 
	VOTO_CANTIDAD NUMBER, 
	CONSTRAINT VOTO_PK PRIMARY KEY (VOTO_PARTIDO, VOTO_MESA, VOTO_ELECCION),
	CONSTRAINT PARTIDO_VOTO_FK FOREIGN KEY (VOTO_PARTIDO) REFERENCES PARTIDO(CODIGO_PART),
	CONSTRAINT ACTA_VOTO_FK FOREIGN KEY (VOTO_MESA, VOTO_ELECCION) REFERENCES ACTA(NUMERO_MESA, TIPO_ELECCION)
) TABLESPACE ELECCIONESTBS;

```

## **CREACIÓN DE ROLES**

```sql

CREATE ROLE GUEST;
CREATE ROLE MESAS;
CREATE ROLE IT;
CREATE ROLE ADMIN;

```

## **GESTION DE PERMISOS PARA ROLES**

#### GUEST

```sql

GRANT SELECT ON Elecciones.DEPARTAMENTO TO GUEST;
GRANT SELECT ON Elecciones.VOTO TO GUEST;
GRANT SELECT ON Elecciones.MUNICIPIO TO GUEST;
GRANT SELECT ON Elecciones.ACTA TO GUEST;
GRANT SELECT ON Elecciones.ELECCION TO GUEST;
GRANT SELECT ON Elecciones.PARTIDO TO GUEST;
GRANT CREATE SESSION TO GUEST;
GRANT CONNECT TO GUEST;
GRANT RESOURCE TO GUEST;

```

#### MESAS

```sql

GRANT INSERT,SELECT ON Elecciones.DEPARTAMENTO TO MESAS;
GRANT INSERT,SELECT ON Elecciones.VOTO TO MESAS;
GRANT INSERT,SELECT ON Elecciones.MUNICIPIO TO MESAS;
GRANT INSERT,SELECT ON Elecciones.ACTA TO MESAS;
GRANT INSERT,SELECT ON Elecciones.ELECCION TO MESAS;
GRANT INSERT,SELECT ON Elecciones.PARTIDO TO MESAS;
GRANT CREATE SESSION TO MESAS;
GRANT CONNECT TO MESAS;
GRANT RESOURCE TO MESAS;

```

#### IT

```sql

GRANT SELECT ON Elecciones.DEPARTAMENTO TO IT;
GRANT SELECT ON Elecciones.VOTO TO IT;
GRANT SELECT ON Elecciones.MUNICIPIO TO IT;
GRANT SELECT ON Elecciones.ACTA TO IT;
GRANT SELECT ON Elecciones.ELECCION TO IT;
GRANT SELECT ON Elecciones.PARTIDO TO IT;
GRANT CREATE USER TO IT;
GRANT CREATE TABLE TO IT;
GRANT CREATE SESSION TO IT;
GRANT CONNECT TO IT;
GRANT RESOURCE TO IT;

```

#### ADMIN

```sql

GRANT INSERT,SELECT,DELETE,UPDATE ON Elecciones.DEPARTAMENTO TO ADMIN;
GRANT INSERT,SELECT,DELETE,UPDATE ON Elecciones.VOTO TO ADMIN;
GRANT INSERT,SELECT,DELETE,UPDATE ON Elecciones.MUNICIPIO TO ADMIN;
GRANT INSERT,SELECT,DELETE,UPDATE ON Elecciones.ACTA TO ADMIN;
GRANT INSERT,SELECT,DELETE,UPDATE ON Elecciones.ELECCION TO ADMIN;
GRANT INSERT,SELECT,DELETE,UPDATE ON Elecciones.PARTIDO TO ADMIN;
GRANT CREATE USER TO ADMIN;
GRANT CREATE SESSION TO ADMIN;
GRANT CONNECT TO ADMIN;
GRANT RESOURCE TO ADMIN;

```

## **CREACIÓN DE USUARIOS**

### GUEST'S

```sql

CREATE USER guest1 IDENTIFIED BY guest1
DEFAULT TABLESPACE ELECCIONESTBS
TEMPORARY TABLESPACE TEMP
ACCOUNT UNLOCK;

CREATE USER guest2 IDENTIFIED BY guest2
DEFAULT TABLESPACE ELECCIONESTBS
TEMPORARY TABLESPACE TEMP
ACCOUNT UNLOCK;

CREATE USER guest3 IDENTIFIED BY guest3
DEFAULT TABLESPACE ELECCIONESTBS
TEMPORARY TABLESPACE TEMP
ACCOUNT UNLOCK;

ALTER USER guest1 QUOTA UNLIMITED ON ELECCIONESTBS;
ALTER USER guest2 QUOTA UNLIMITED ON ELECCIONESTBS;
ALTER USER guest3 QUOTA UNLIMITED ON ELECCIONESTBS;

GRANT GUEST TO guest1;
GRANT GUEST TO guest2;
GRANT GUEST TO guest3;

ALTER USER guest1 DEFAULT ROLE GUEST;
ALTER USER guest2 DEFAULT ROLE GUEST;
ALTER USER guest3 DEFAULT ROLE GUEST;

```

### MESAS'S

```sql

CREATE USER mesas1 IDENTIFIED BY mesas1
DEFAULT TABLESPACE ELECCIONESTBS
TEMPORARY TABLESPACE TEMP
QUOTA UNLIMITED ON ELECCIONESTBS
ACCOUNT UNLOCK;

CREATE USER mesas2 IDENTIFIED BY mesas2
DEFAULT TABLESPACE ELECCIONESTBS
TEMPORARY TABLESPACE TEMP
QUOTA UNLIMITED ON ELECCIONESTBS
ACCOUNT UNLOCK;

CREATE USER mesas3 IDENTIFIED BY mesas3
DEFAULT TABLESPACE ELECCIONESTBS
TEMPORARY TABLESPACE TEMP
QUOTA UNLIMITED ON ELECCIONESTBS
ACCOUNT UNLOCK;

CREATE USER mesas4 IDENTIFIED BY mesas4
DEFAULT TABLESPACE ELECCIONESTBS
TEMPORARY TABLESPACE TEMP
QUOTA UNLIMITED ON ELECCIONESTBS
ACCOUNT UNLOCK;

GRANT MESAS TO mesas1;
GRANT MESAS TO mesas2;
GRANT MESAS TO mesas3;
GRANT MESAS TO mesas4;

ALTER USER mesas1 DEFAULT ROLE MESAS;
ALTER USER mesas2 DEFAULT ROLE MESAS;
ALTER USER mesas3 DEFAULT ROLE MESAS;
ALTER USER mesas4 DEFAULT ROLE MESAS;

```

### IT'S

```sql

CREATE USER it1 IDENTIFIED BY it1
DEFAULT TABLESPACE ELECCIONESTBS
TEMPORARY TABLESPACE TEMP
ACCOUNT UNLOCK;

CREATE USER it2 IDENTIFIED BY it2
DEFAULT TABLESPACE ELECCIONESTBS
TEMPORARY TABLESPACE TEMP
ACCOUNT UNLOCK;

CREATE USER it3 IDENTIFIED BY it3
DEFAULT TABLESPACE ELECCIONESTBS
TEMPORARY TABLESPACE TEMP
ACCOUNT UNLOCK;

ALTER USER it1 QUOTA UNLIMITED ON ELECCIONESTBS;
ALTER USER it2 QUOTA UNLIMITED ON ELECCIONESTBS;
ALTER USER it3 QUOTA UNLIMITED ON ELECCIONESTBS;

GRANT IT TO it1;
GRANT IT TO it2;
GRANT IT TO it3;

ALTER USER it1 DEFAULT ROLE IT;
ALTER USER it2 DEFAULT ROLE IT;
ALTER USER it3 DEFAULT ROLE IT;

```

### ADMIN'S

```sql

CREATE USER admin1 IDENTIFIED BY admin1
DEFAULT TABLESPACE ELECCIONESTBS
TEMPORARY TABLESPACE TEMP
QUOTA UNLIMITED ON ELECCIONESTBS
ACCOUNT UNLOCK;

CREATE USER admin2 IDENTIFIED BY admin2
DEFAULT TABLESPACE ELECCIONESTBS
TEMPORARY TABLESPACE TEMP
QUOTA UNLIMITED ON ELECCIONESTBS
ACCOUNT UNLOCK;

GRANT ADMIN TO admin1;
GRANT ADMIN TO admin2;

ALTER USER admin1 DEFAULT ROLE ADMIN;
ALTER USER admin2 DEFAULT ROLE ADMIN;

```


## **DATOS EN TABLAS**

```sql

--INSERCION DE DATOS PARA LAS TABLAS
INSERT INTO Elecciones.departamento (codigo_depto, nombre_depto) VALUES(1,'Guatemala');
INSERT INTO Elecciones.departamento (CODIGO_DEPTO, NOMBRE_DEPTO) VALUES(2,'Quetzaltenango');
INSERT INTO Elecciones.departamento (CODIGO_DEPTO, NOMBRE_DEPTO) VALUES(3,'Alta Verapaz');
INSERT INTO Elecciones.departamento (CODIGO_DEPTO, NOMBRE_DEPTO) VALUES(4,'Chimaltenango');
INSERT INTO Elecciones.departamento (CODIGO_DEPTO, NOMBRE_DEPTO) VALUES(5,'Escuintla');
COMMIT;

INSERT INTO Elecciones.municipio (CODIGO_MUNI, DEPTO_MUNI, NOMBRE_MUNI) VALUES(1,1,'Guatemala');
INSERT INTO Elecciones.municipio (CODIGO_MUNI, DEPTO_MUNI, NOMBRE_MUNI) VALUES(2,1,'Villa Canales');
INSERT INTO Elecciones.municipio (CODIGO_MUNI, DEPTO_MUNI, NOMBRE_MUNI) VALUES(3,1,'Palencia');
INSERT INTO Elecciones.municipio (CODIGO_MUNI, DEPTO_MUNI, NOMBRE_MUNI) VALUES(4,1,'San Miguel Petapa');
INSERT INTO Elecciones.municipio (CODIGO_MUNI, DEPTO_MUNI, NOMBRE_MUNI) VALUES(5,1,'San Jose Pinula');
INSERT INTO Elecciones.municipio (CODIGO_MUNI, DEPTO_MUNI, NOMBRE_MUNI) VALUES(6,2,'Quetzaltenango');
INSERT INTO Elecciones.municipio (CODIGO_MUNI, DEPTO_MUNI, NOMBRE_MUNI) VALUES(7,2,'Olintepeque');
INSERT INTO Elecciones.municipio (CODIGO_MUNI, DEPTO_MUNI, NOMBRE_MUNI) VALUES(8,2,'Zunil');
INSERT INTO Elecciones.municipio (CODIGO_MUNI, DEPTO_MUNI, NOMBRE_MUNI) VALUES(9,2,'La Esperanza');
INSERT INTO Elecciones.municipio (CODIGO_MUNI, DEPTO_MUNI, NOMBRE_MUNI) VALUES(10,2,'Coban');
INSERT INTO Elecciones.municipio (CODIGO_MUNI, DEPTO_MUNI, NOMBRE_MUNI) VALUES(11,3,'Tactic');
INSERT INTO Elecciones.municipio (CODIGO_MUNI, DEPTO_MUNI, NOMBRE_MUNI) VALUES(12,3,'Chisec');
INSERT INTO Elecciones.municipio (CODIGO_MUNI, DEPTO_MUNI, NOMBRE_MUNI) VALUES(13,3,'Panzoz');
INSERT INTO Elecciones.municipio (CODIGO_MUNI, DEPTO_MUNI, NOMBRE_MUNI) VALUES(14,3,'San Pedro Carcha');
INSERT INTO Elecciones.municipio (CODIGO_MUNI, DEPTO_MUNI, NOMBRE_MUNI) VALUES(15,3,'Fray Bartolome de las Cases');
INSERT INTO Elecciones.municipio (CODIGO_MUNI, DEPTO_MUNI, NOMBRE_MUNI) VALUES(16,4,'Chimaltenango');
INSERT INTO Elecciones.municipio (CODIGO_MUNI, DEPTO_MUNI, NOMBRE_MUNI) VALUES(17,4,'Tecpan');
INSERT INTO Elecciones.municipio (CODIGO_MUNI, DEPTO_MUNI, NOMBRE_MUNI) VALUES(18,4,'El Tejar');
INSERT INTO Elecciones.municipio (CODIGO_MUNI, DEPTO_MUNI, NOMBRE_MUNI) VALUES(19,4,'Patzicia');
INSERT INTO Elecciones.municipio (CODIGO_MUNI, DEPTO_MUNI, NOMBRE_MUNI) VALUES(20,4,'Zaragoza');
INSERT INTO Elecciones.municipio (CODIGO_MUNI, DEPTO_MUNI, NOMBRE_MUNI) VALUES(21,5,'Escuintla');
INSERT INTO Elecciones.municipio (CODIGO_MUNI, DEPTO_MUNI, NOMBRE_MUNI) VALUES(22,5,'Iztapa');
INSERT INTO Elecciones.municipio (CODIGO_MUNI, DEPTO_MUNI, NOMBRE_MUNI) VALUES(23,5,'Palin');
INSERT INTO Elecciones.municipio (CODIGO_MUNI, DEPTO_MUNI, NOMBRE_MUNI) VALUES(24,5,'Santa Lucia Cotzumalguapa');
INSERT INTO Elecciones.municipio (CODIGO_MUNI, DEPTO_MUNI, NOMBRE_MUNI) VALUES(25,5,'La Democracia');
COMMIT;

INSERT INTO Elecciones.partido (CODIGO_PART, NOMBRE_PART) VALUES(1,'Vamos');
INSERT INTO Elecciones.partido (CODIGO_PART, NOMBRE_PART) VALUES(2,'Todos');
INSERT INTO Elecciones.partido (CODIGO_PART, NOMBRE_PART) VALUES(3,'Pan');
INSERT INTO Elecciones.partido (CODIGO_PART, NOMBRE_PART) VALUES(4,'FRG');
INSERT INTO Elecciones.partido (CODIGO_PART, NOMBRE_PART) VALUES(5,'Mas');
COMMIT;

INSERT INTO Elecciones.eleccion (CODIGO_ELE, NOMBRE_ELE) VALUES(1,'Presidente');
INSERT INTO Elecciones.eleccion (CODIGO_ELE, NOMBRE_ELE) VALUES(2,'Alcalde');
COMMIT;

--ACTAS PRESIDENTE
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(1,1,1,1,107,100,5,2,100,0);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(2,1,1,2,153,150,1,2,150,0);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(3,1,1,3,105,102,2,0,102,0);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(4,1,1,4,91,90,0,1,91,0);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(5,1,1,5,123,122,0,0,122,1);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(6,1,2,6,110,101,6,3,101,0);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(7,1,2,7,157,151,2,3,151,1);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(8,1,2,8,108,103,3,1,103,1);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(9,1,2,9,91,91,0,1,91,0);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(10,1,2,10,124,123,0,0,123,1);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(11,1,3,11,111,101,6,4,101,0);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(12,1,3,12,158,151,3,3,151,1);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(13,1,3,13,109,103,3,1,103,2);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(14,1,3,14,56,25,0,1,25,29);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(15,1,3,15,45,40,5,0,40,0);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(16,1,4,16,35,30,5,0,30,0);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(17,1,4,17,75,50,20,5,50,1);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(18,1,4,18,66,63,3,0,63,0);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(19,1,4,19,56,25,0,0,25,31);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(20,1,4,20,44,40,4,0,44,0);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(21,1,5,21,36,31,5,0,30,0);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(22,1,5,22,76,51,20,5,51,1);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(23,1,5,23,67,64,3,0,64,0);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(24,1,5,24,55,25,0,0,25,30);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(25,1,5,25,45,40,5,0,44,0);
--ACTAS ALCALDE
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(26,2,1,1,107,100,5,2,100,0);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(27,2,1,2,153,150,1,2,150,0);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(28,2,1,3,105,102,2,0,102,0);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(29,2,1,4,91,90,0,1,91,0);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(30,2,1,5,123,122,0,0,122,1);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(31,2,2,6,110,101,6,3,101,0);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(32,2,2,7,157,151,2,3,151,1);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(33,2,2,8,108,103,3,1,103,1);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(34,2,2,9,91,91,0,1,91,0);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(35,2,2,10,124,123,0,0,123,1);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(36,2,3,11,111,101,6,4,101,0);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(37,2,3,12,158,151,3,3,151,1);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(38,2,3,13,109,103,3,1,103,2);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(39,2,3,14,56,25,0,1,25,29);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(40,2,3,15,45,40,5,0,40,0);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(41,2,4,16,35,30,5,0,30,0);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(42,2,4,17,75,50,20,5,50,1);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(43,2,4,18,66,63,3,0,63,0);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(44,2,4,19,56,25,0,0,25,31);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(45,2,4,20,44,40,4,0,44,0);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(46,2,5,21,36,31,5,0,30,0);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(47,2,5,22,76,51,20,5,51,1);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(48,2,5,23,67,64,3,0,64,0);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(49,2,5,24,55,25,0,0,25,30);
INSERT INTO Elecciones.acta (numero_mesa, tipo_eleccion, departamento, municipio, papeletas_recibidas, total_votos_validos, votos_nulos, votos_blanco, votos_validos_emitidos, votos_invalidos) VALUES(50,2,5,25,45,40,5,0,44,0);
COMMIT;
--VOTOS PRESIDENTE
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,1,1,15);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,1,1,34);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,1,1,22);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,1,1,5);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,1,1,3);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,2,1,22);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,2,1,13);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,2,1,12);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,2,1,15);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,2,1,13);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,3,1,21);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,3,1,11);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,3,1,15);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,3,1,25);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,3,1,33);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,4,1,27);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,4,1,11);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,4,1,26);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,4,1,5);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,4,1,3);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,5,1,16);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,5,1,7);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,5,1,35);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,5,1,66);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,5,1,1);

INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,6,1,12);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,6,1,15);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,6,1,3);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,6,1,1);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,6,1,17);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,7,1,21);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,7,1,33);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,7,1,5);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,7,1,6);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,7,1,11);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,8,1,8);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,8,1,7);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,8,1,5);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,8,1,15);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,8,1,14);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,9,1,7);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,9,1,1);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,9,1,6);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,9,1,25);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,9,1,13);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,10,1,6);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,10,1,27);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,10,1,5);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,10,1,16);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,10,1,12);

INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,11,1,32);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,11,1,17);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,11,1,32);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,11,1,11);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,11,1,1);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,12,1,2);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,12,1,35);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,12,1,12);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,12,1,15);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,12,1,1);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,13,1,18);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,13,1,17);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,13,1,51);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,13,1,5);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,13,1,24);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,14,1,8);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,14,1,18);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,14,1,16);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,14,1,15);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,14,1,1);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,15,1,61);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,15,1,17);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,15,1,35);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,15,1,26);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,15,1,2);

INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,16,1,12);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,16,1,27);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,16,1,33);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,16,1,21);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,16,1,11);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,17,1,32);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,17,1,5);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,17,1,2);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,17,1,35);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,17,1,11);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,18,1,8);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,18,1,1);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,18,1,1);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,18,1,5);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,18,1,4);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,19,1,5);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,19,1,8);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,19,1,6);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,19,1,5);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,19,1,12);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,20,1,1);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,20,1,7);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,20,1,31);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,20,1,6);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,20,1,21);

INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,21,1,2);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,21,1,2);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,21,1,3);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,21,1,25);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,21,1,18);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,22,1,3);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,22,1,52);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,22,1,27);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,22,1,3);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,22,1,19);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,23,1,8);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,23,1,9);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,23,1,3);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,23,1,9);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,23,1,41);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,24,1,55);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,24,1,28);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,24,1,16);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,24,1,15);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,24,1,1);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,25,1,13);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,25,1,7);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,25,1,1);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,25,1,36);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,25,1,51);
--VOTOS ALCALDA
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,1,2,5);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,1,2,34);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,1,2,2);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,1,2,5);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,1,2,3);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,2,2,2);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,2,2,3);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,2,2,2);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,2,2,5);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,2,2,3);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,3,2,1);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,3,2,1);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,3,2,5);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,3,2,5);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,3,2,3);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,4,2,7);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,4,2,1);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,4,2,6);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,4,2,25);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,4,2,39);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,5,2,6);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,5,2,73);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,5,2,5);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,5,2,6);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,5,2,17);

INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,6,2,2);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,6,2,5);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,6,2,34);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,6,2,61);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,6,2,7);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,7,2,1);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,7,2,73);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,7,2,25);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,7,2,46);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,7,2,1);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,8,2,81);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,8,2,57);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,8,2,52);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,8,2,5);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,8,2,4);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,9,2,73);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,9,2,15);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,9,2,62);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,9,2,5);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,9,2,3);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,10,2,61);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,10,2,72);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,10,2,55);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,10,2,6);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,10,2,2);

INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,11,2,2);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,11,2,7);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,11,2,3);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,11,2,1);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,11,2,19);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,12,2,27);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,12,2,3);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,12,2,2);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,12,2,5);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,12,2,1);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,13,2,8);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,13,2,7);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,13,2,1);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,13,2,45);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,13,2,4);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,14,2,48);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,14,2,68);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,14,2,6);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,14,2,1);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,14,2,18);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,15,2,6);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,15,2,7);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,15,2,5);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,15,2,2);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,15,2,23);

INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,16,2,42);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,16,2,2);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,16,2,3);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,16,2,21);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,16,2,1);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,17,2,3);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,17,2,25);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,17,2,24);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,17,2,3);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,17,2,1);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,18,2,81);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,18,2,11);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,18,2,11);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,18,2,15);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,18,2,41);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,19,2,5);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,19,2,28);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,19,2,63);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,19,2,5);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,19,2,2);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,20,2,11);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,20,2,47);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,20,2,3);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,20,2,26);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,20,2,1);

INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,21,2,23);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,21,2,25);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,21,2,36);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,21,2,5);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,21,2,8);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,22,2,36);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,22,2,2);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,22,2,7);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,22,2,43);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,22,2,9);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,23,2,38);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,23,2,91);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,23,2,34);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,23,2,92);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,23,2,4);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,24,2,5);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,24,2,8);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,24,2,6);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,24,2,5);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,24,2,19);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(1,25,2,3);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(2,25,2,71);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(3,25,2,18);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(4,25,2,6);
INSERT INTO Elecciones.voto (voto_partido, voto_mesa, voto_eleccion, voto_cantidad) VALUES(5,25,2,5);
COMMIT;
```

## **VISTAS**

```sql

CREATE VIEW VOTOSPRESIDENTE
AS
SELECT  D.NOMBRE_DEPTO AS "DEPARTAMENTO", M.NOMBRE_MUNI AS "MUNICIPALIDAD", P.NOMBRE_PART AS "PARTIDO", SUM(V.VOTO_CANTIDAD) AS "CANTIDAD_VOTOS"
FROM ELECCIONES.DEPARTAMENTO D
INNER JOIN ELECCIONES.MUNICIPIO M ON M.DEPTO_MUNI = D.CODIGO_DEPTO
INNER JOIN ELECCIONES.ACTA A ON A.MUNICIPIO = M.CODIGO_MUNI
INNER JOIN ELECCIONES.VOTO V ON V.VOTO_MESA = A.NUMERO_MESA
INNER JOIN ELECCIONES.PARTIDO P ON P.CODIGO_PART = V.VOTO_PARTIDO
INNER JOIN ELECCIONES.ELECCION E ON E.CODIGO_ELE = A.TIPO_ELECCION
WHERE E.NOMBRE_ELE = 'Presidente'
GROUP BY D.NOMBRE_DEPTO,M.NOMBRE_MUNI, P.NOMBRE_PART
ORDER BY D.NOMBRE_DEPTO, M.NOMBRE_MUNI;

```

### **ASIGNAR PERMISO A LA VISTA A USUARIO GUEST1**

```sql

GRANT SELECT
ON ELECCIONES.VOTOSPRESIDENTE
TO guest1;

```

# **BACKUP**

