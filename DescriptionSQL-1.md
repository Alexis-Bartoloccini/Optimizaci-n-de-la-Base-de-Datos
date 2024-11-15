# Optimización de la Base de Datos de Empleados.

## Descripción General.

El proyecto **Optimización de la Base de Datos de Empleados** tiene como propósito fundamental elevar la calidad y la utilidad de la base de datos de empleados de la empresa. En un entorno laboral donde los datos son clave para la toma de decisiones, se llevó a cabo un proceso exhaustivo de mejora que abarca las siguientes acciones:

- **Depuración de datos**: Se realizó una minuciosa revisión para identificar y eliminar registros duplicados y datos inconsistentes, garantizando la integridad de la información.
- **Renombrado de columnas**: Se mejoró la claridad y la comprensión de los nombres de las columnas, facilitando la interpretación de la base de datos por parte de los usuarios.
- **Creación de nuevas columnas**: Se incorporaron campos adicionales, como la edad de los empleados y un email predeterminado, que brindan un valor añadido para el análisis demográfico y de recursos humanos.

Este esfuerzo no solo optimiza la estructura de la base de datos, sino que también proporciona una base sólida para futuros análisis, reportes y la toma de decisiones informadas. Gracias a estas mejoras, la base de datos se convierte en una herramienta más eficiente y valiosa para la gestión del talento humano en la empresa.

## Herramientas y tecnologías aplicadas.
    -MySQL para la manipulación y optimización de datos.
    -SQL avanzado para la creación de procedimientos almacenados, triggers y consultas complejas.
    -Data cleaning: técnicas avanzadas para la depuración y normalización de datos.
    -Gestión de bases de datos: renombrado de columnas, eliminación de duplicados, corrección de formatos de fechas y otros ajustes estructurales."

## Metodología y Consultas SQL.

En esta sección se detallan las estrategias implementadas durante el proceso de optimización de la base de datos, junto con las consultas SQL utilizadas y notas explicativas que facilitan la comprensión del trabajo realizado.

#### Pasos.

#### 1. Creación de un Stored Procedure: 
<!-- Se creó un procedimiento almacenado para visualizar cambios y trabajar sobre una muestra de la base de datos. -->

### Consulta SQL.

DELIMITER //
CREATE PROCEDURE muestra()
BEGIN
    SELECT * FROM limpieza LIMIT 10;
END //
DELIMITER;


### 2. Renombrar Columnas: 
<!-- En este paso, se realizaron cambios en los nombres de las columnas para mejorar la claridad y consistencia de la base de datos. -->

#### Consultas SQL.

ALTER TABLE limpieza CHANGE COLUMN `Id_empleado` Id_empl varchar(20) null;
ALTER TABLE limpieza CHANGE COLUMN `gÃ©nero` Gender varchar(20) null;
ALTER TABLE limpieza CHANGE COLUMN `Apellido` Last_Name varchar(30) null;

### 3. Identificación e eliminación de valores duplicados.

#### 3.1. Identificación.
<!-- Se utiliza la siguiente consulta para identificar los valores duplicados en la columna `Id_empl`: -->

SELECT Id_empl, count(*) AS cantidad_duplicados
FROM limpieza
GROUP BY Id_empl
HAVING count(*) > 1;

#### 3.2. Contabilización de duplicados.
<!-- Se detectan valores duplicados ya que esta consulta trae resultados. Para contabilizar la cantidad de elementos duplicados se hace la siguiente subconsulta: -->

SELECT COUNT(*) AS cantidad_duplicados2 FROM (
    SELECT Id_empl, count(*) AS cantidad_duplicados
    FROM limpieza
    GROUP BY Id_empl
    HAVING count(*) > 1) AS subquery;

### 3.3.
<!-- Creamos una consulta que elimine los valores duplicados utilizando DISTINCT: -->

SELECT DISTINCT * FROM limpieza;

#### 3.4.
<!-- Verificamos que la nueva consulta no tiene duplicados, es decir, tiene menos elementos: -->

SELECT COUNT(*) AS unicos FROM (SELECT DISTINCT * FROM limpieza) AS verificacion_de_filtro;

#### 3.5.
<!-- Creamos una nueva tabla que contenga los resultados de la consulta anterior: -->

CREATE TABLE Base2 AS (SELECT DISTINCT * FROM limpieza);

#### 3.6.
<!-- Verificación de paso correcto -->

SELECT Id_empl, count(*) AS cantidad_duplicados
FROM Base2
GROUP BY Id_empl
HAVING count(*) > 1;

### 4. Nueva inspeccion a la base de datos

DESCRIBE base2;

### 5. Remoción de espacios extras.

#### 5.1. Remoción de espacios extras en columna "Name".

##### 5.1.1. Búsqueda de espacios extras.

<!-- Este codigo traer consigo los nombres que al restar "largo de nombre con espacios" y "largo de nombres sin espacios" de mayor a 1, es decir, tiene espacios en blanco. -->

SELECT Name 
FROM base2
WHERE LENGTH(name)-LENGTH(TRIM(name))>1;

##### 5.1.2. Chequeo de instrucción que eliminaría los espacios (compara nombre vs nombre sin espacio).

SELECT Name, TRIM(name) 
FROM base2
WHERE LENGTH(name)-LENGTH(TRIM(name))>1;

##### 5.1.3. Eliminación de espacios extras en nombre.

UPDATE base2
SET Name = TRIM(Name)
WHERE LENGTH(name)-LENGTH(TRIM(name))>1;

#### 5.2. Remoción de espacios extras en columna "Last_Name".

##### 5.2.1. #Búsqueda de espacios extras en columna "Last_Name".

SELECT Last_Name 
FROM base2
WHERE LENGTH(Last_Name)-LENGTH(TRIM(Last_Name))>1;

##### 5.2.2. Chequeo de instrucción que eliminaría los espacios (compara apellido vs apellido sin espacio).

SELECT Last_Name, TRIM(Last_Name)
FROM base2
WHERE LENGTH(Last_Name)-LENGTH(TRIM(Last_Name))>1;

##### 5.2.3. Eliminación de espacios extras en nombre.

UPDATE base2
SET Last_Name = TRIM(Last_Name)
WHERE LENGTH(Last_Name)-LENGTH(TRIM(Last_Name))>1;

#### 5.3. Remoción de espacios extras entre palabras en columnas compuestas por datos con mas de 1 palabra (columna "area").

##### 5.3.1. Búsqueda de espacios extras en columna "area"

SELECT area 
FROM base2
WHERE area REGEXP '\\s{2,}';
<!-- 
EXPLICACION DEL FUNCIONAMIENTO DEL REGEXP:
 REGEXP: Es el operador de expresión regular que permite buscar patrones en una columna de texto. En este caso se buscaran las "areas" que tengan dos o mas espacios.

 '\\s{2,}': Es el patrón de expresión regular utilizado para buscar en la columna area. Vamos a desglosar este patrón:

 \\s: Aquí, \s es un carácter de espacio en blanco (como espacios, tabulaciones, etc.). En las cadenas de MySQL y muchos lenguajes de programación, el backslash \ se usa como carácter de escape, por lo que necesitas escribirlo como \\ para representar un solo backslash. Por lo tanto, \\s representa un espacio en blanco.

 {2,}: Esto indica que queremos encontrar dos o más ocurrencias del carácter de espacio en blanco. {2,} es una forma de especificar una cantidad mínima de ocurrencias en expresiones regulares. En este caso, está buscando dos o más espacios consecutivos. -->

##### 5.3.2. Chequeo de instrucción que eliminaría espacios extras entre palabras en columnas compuestas por datos con mas de 1 palabra. (columna "area")

<!--"regexp_replace" una función que se utiliza para buscar y reemplazar patrones dentro de un texto, utilizando expresiones regulares (regex). Su función principal es encontrar coincidencias en una cadena de texto que correspondan a un patrón específico y luego reemplazarlas con una cadena de texto diferente. -->

SELECT area, TRIM(regexp_replace(area,'\\s{2,}',' ')) AS area2
FROM base2
WHERE area REGEXP '\\s{2,}';

##### 5.3.3. Eliminación de espacios extras entre palabras en columna "area".

UPDATE base2
SET area = TRIM(regexp_replace(area,'\\s{2,}',' '))
WHERE area REGEXP '\\s{2,}';

### 6. Cambio de idioma en columna "gender".

#### 6.1. Chequeo de instrucción que haría los cambios.

SELECT Gender,
	CASE
		WHEN Gender='hombre' THEN 'male'
		WHEN Gender='mujer' THEN 'women'
		ELSE 'other'
	END AS Gender2
FROM base2;

#### 6.2. Instrucción para el cambio.

UPDATE base2 
SET gender = 	
	CASE
		WHEN Gender='hombre' THEN 'male'
		WHEN Gender='mujer' THEN 'female'
		ELSE 'other'
END;

### 7. Cambio de formato y presentación del dato en columna "type".

#### 7.1. Instrucción para cambio de tipo de dato de la columna.

ALTER TABLE base2
MODIFY COLUMN type text;

#### 7.2. Instrucción para el cambio.

UPDATE base2 
SET type = 	
	CASE
		WHEN type='1' THEN 'Remote'
		WHEN type='0' THEN 'Hybrid'
		ELSE 'other'
END;

### 8. Cambio de formato y presentación del dato en columna "salary".
 
UPDATE base2
SET salary = CAST(TRIM(REPLACE(REPLACE(salary, '$',''),',','')) AS INT);

### 9. Normalización de columnas con fechas.

#### 9.1. Columna  "birth_date"

##### 9.1.1. Chequeo de instrucción que haría los cambios. 

<!-- Se utiliza str_to_date para darle formato fecha al dato de cada celda, luego con date_format se le da el formato de fecha específico. -->

SELECT birth_date, 
	CASE
		WHEN birth_date LIKE '%/%' THEN date_format(str_to_date(birth_date, '%m/%d/%y'),'%Y-%m-%d')
		WHEN birth_date LIKE '%-%' THEN date_format(str_to_date(birth_date, '%m-%d-%y'),'%Y-%m-%d')
        ELSE NULL
END AS new_birth_date
FROM base2;

#### 9.1.2. Instrucción para cambiar el formato en tabla.

UPDATE base2
	SET birth_date = 
	CASE
		WHEN birth_date LIKE '%/%' THEN date_format(str_to_date(birth_date, '%m/%d/%Y'),'%Y-%m-%d')
		WHEN birth_date LIKE '%-%' THEN date_format(str_to_date(birth_date, '%m-%d-%Y'),'%Y-%m-%d')
        ELSE NULL
	END;

#### 9.1.3. Cambio de tipo de dato a la columna.

ALTER TABLE base2 MODIFY COLUMN birth_date date;

#### 9.2. Columna "Start_date"

#### 9.2.1. Chequeo de instrucción que haría los cambios. 

SELECT Start_date, 
	CASE
		WHEN Start_date LIKE '%/%' THEN date_format(str_to_date(Start_date, '%m/%d/%y'),'%Y-%m-%d')
		WHEN Start_date LIKE '%-%' THEN date_format(str_to_date(Start_date, '%m-%d-%y'),'%Y-%m-%d')
        ELSE NULL
END AS new_start_date
FROM base2;

#### 9.2.2. Instrucción para cambiar el formato en tabla.

UPDATE base2
	SET Start_date = CASE
		WHEN Start_date LIKE '%/%' THEN date_format(str_to_date(Start_date, '%m/%d/%Y'),'%Y-%m-%d')
		WHEN Start_date LIKE '%-%' THEN date_format(str_to_date(Start_date, '%m-%d-%Y'),'%Y-%m-%d')
        ELSE NULL
	END;
#### 9.2.3. Cambio de tipo de dato a la columna.

ALTER TABLE base2 MODIFY COLUMN Start_date date;

#### 9.3. Columna  "finish_date"

##### 9.3.1. Backup de columna finish_date

<!-- Creacion de columna. -->

ALTER TABLE base2
	ADD finish_date_copy TEXT;

<!-- -- Copiado de datos. -->
UPDATE base2
	SET finish_date_copy = finish_date;

##### 9.3.2. Chequeo de instrucción que haría los cambios. 

SELECT finish_date, 
		  date_format(str_to_date(finish_date, '%Y-%m-%d %H:%i:%s'),'%Y/%m/%d %H:%i:%s')
          as new_finish_date
FROM base2 ;

#### 9.3.3. Instrucción para cambiar el formato en tabla.

UPDATE  base2
	SET finish_date = date_format(str_to_date(finish_date, '%Y-%m-%d %H:%i:%s UTC'),'%Y/%m/%d %H:%i:%s')
    WHERE finish_date != '';

#### 9.3.4. A continuación, otra alternativa para cambiar la columna finish date. 

<!-- Elimina el UTC y luego selecciona solo año mes y dias para dejar solo eso en la columna. -->

UPDATE  base2
	SET finish_date =
		DATE_FORMAT(str_to_date(REPLACE(finish_date, ' UTC', ''), '%Y-%m-%d %H:%i:%s'),'%Y-%m-%d')
    WHERE finish_date != '' AND finish_date IS NOT NULL;

<!-- ##Cambio de tipo de dato a la columna.
<!-- -- Primero se le asigna valor nulo a los campos vacíos para poder asignar el typo de dato "date". -->
UPDATE base2 SET finish_date = null WHERE finish_date =''
<!-- -- Luego se asigna el tipo de dato nulo. -->
ALTER TABLE base2 MODIFY COLUMN finish_date date;

### 10. Cálculo de la edad: Operaciones con fechas.

#### 10.1. Adición de columnas necesarias.

ALTER TABLE  limpieza ADD COLUMN age INT;

#### 10.2. Cálculo de edad actual.

##### 10.2.1. Chequeo de instrucción que haría los cambios.

SELECT Name , birth_date , timestampdiff( year , birth_date , curdate()) AS Age
FROM base2;

#### 10.2.2. Inserción de datos.

UPDATE  base2
	SET Age = timestampdiff( year , birth_date , curdate());

#### 11. Creación de correo electrónico predeterminado utilizando información de la tabla.

##### 11.1. Chequeo de instrucción que haría los cambios.

<!-- Formar correo que se componga el nombre y las primeras 2 letras de lapellido, separados por un guin bajo y finalizado por "@ -->

SELECT concat(substring_index(Name,' ', 1),'_',substring(Last_name, 1 , 2),'.',substring(type, 1 , 1), '@genericcompany.com') 
AS 'email'
FROM base2;

##### 11.2. Adición de columnas necesarias.

ALTER TABLE  base2 ADD COLUMN Email varchar(100);

##### 11.3. Adición de columnas necesarias.

UPDATE base2 SET Email = concat(substring_index(Name,' ', 1),'_',substring(Last_name, 1 , 2),'.',substring(type, 1 , 1), '@genericcompany.com');


#### 12. Exportación de db.

SELECT Id_empl,Name,Last_Name,birth_date,Gender,area,salary,Start_date,finish_date,promotion_date, type, Age,Email
FROM base2
WHERE finish_date <= curdate();

## Conclusiones:

El proceso de optimización ha mejorado significativamente la calidad, precisión y organización de los datos, haciendo que la base de datos sea más accesible y fácil de interpretar. La depuración y normalización de datos permite un análisis más eficiente, mientras que las nuevas columnas y formatos unificados favorecen la creación de reportes más detallados. Estas mejoras fortalecen la integridad de los datos y proporcionan una base sólida para análisis futuros y la toma de decisiones estratégicas.

## Visualización de Datos:
Con la base de datos optimizada, los datos están preparados para ser visualizados mediante herramientas como Power BI o Tableau. Esto permitirá crear dashboards que ofrezcan:

    -Distribución de empleados por género y área.
    -Análisis de salarios por departamento.
    -Comparación de antigüedad de empleados en relación con promociones o contrataciones.

## Impacto en la Toma de Decisiones:

Gracias a la optimización, ahora es posible generar reportes más precisos que impulsan la toma de decisiones en áreas clave como la gestión del talento, la evaluación de salarios y el análisis de rendimiento. Por ejemplo, los datos optimizados permiten identificar empleados con alto potencial para promociones o detectar áreas con alta rotación de personal, lo que facilita acciones estratégicas más efectivas.

## Conclusiones:

    El proceso de Optimización de la Base de Datos de Empleados ha logrado mejorar la calidad, precisión y organización de los datos, asegurando que la información sea más accesible y comprensible para los usuarios. La depuración y normalización de los datos permite un análisis más eficiente, mientras que la creación de nuevas columnas y la unificación de formatos facilita futuros reportes y análisis detallados. Estas mejoras no solo aseguran una mayor integridad en los datos actuales, sino que también proporcionan una base sólida para el crecimiento de la empresa.

## Visualización de Datos

Tras la optimización de la base de datos, los datos están listos para ser utilizados en herramientas de visualización como Power BI o Tableau. Un ejemplo de visualización sería un dashboard que muestre:

    - Distribución de empleados por género y área.
    - Análisis de salarios por departamento.
    - Comparación de antigüedad de empleados en relación con su fecha de promoción o contratación.

## Impacto en la Toma de Decisiones.

    La optimización realizada en la base de datos no solo mejora la calidad de los datos, sino que también permite generar reportes más precisos que facilitan la toma de decisiones estratégicas en áreas como la gestión del talento humano, evaluación de salarios, y análisis de rendimiento. Por ejemplo, con los nuevos campos y correcciones aplicadas, ahora es posible identificar empleados con alto potencial para promociones o identificar áreas con altos niveles de rotación.
