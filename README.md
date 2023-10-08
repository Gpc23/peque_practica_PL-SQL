# peque_practica_PL-SQL

## 1. Hacer un procedimiento que muestre el nombre y el salario del empleado cuyo código es 7082

```
CREATE OR REPLACE PROCEDURE mostrar_empleado
is
    v_nombre emp.ename%type;
    v_salario emp.sal%type;
begin
    select ename,sal into v_nombre,v_salario from emp where empno=7782;
    dbms_output.put_line('El nombre del empleado es ' || v_nombre || ' y su salario es: ' || v_salario);
end;
/

exec mostrar_empleado;
```

[![1.png](https://i.postimg.cc/3rtMM5Nh/1.png)](https://postimg.cc/hhQZxYVZ)


## 2. Hacer un procedimiento que reciba como parámetro un código de empleado y devuelva su nombre

```
CREATE OR REPLACE PROCEDURE mostrar_empleado2 (p_empno emp.empno%TYPE)
is
    v_nombre emp.ename%type;
    v_salario emp.sal%type;
begin
    select ename,sal into v_nombre,v_salario from emp where empno=p_empno;
    dbms_output.put_line('El nombre del empleado es ' || v_nombre || ' y su salario es: ' || v_salario);
end;
/

exec mostrar_empleado2(7900);
```

[![2.png](https://i.postimg.cc/2SFMxPPg/2.png)](https://postimg.cc/dh35sSQ9)



## 3. Hacer un procedimiento que devuelva los nombres de los tres empleados más antiguos

```
CREATE OR REPLACE PROCEDURE empleados_mas_antiguos
IS
    CURSOR c_top IS
        SELECT ename
        FROM emp
        ORDER BY hiredate ASC;

    v_contador NUMBER := 0;
    v_maximo NUMBER := 3;
BEGIN
    dbms_output.put_line('Los 3 empleados más antiguos son: ');
    FOR v_empleado IN c_top
    LOOP
        v_contador := v_contador + 1;
        EXIT WHEN v_contador > v_maximo;
        dbms_output.put_line(v_empleado.ename);
    END LOOP;
END;
/

exec empleados_mas_antiguos;
```

[![3.png](https://i.postimg.cc/hGDwZ2nv/3.png)](https://postimg.cc/JsFPGcGC)


## 4. Hacer un procedimiento que reciba el nombre de un tablespace y muestre los nombres de los usuarios que lo tienen como tablespace por defecto (Vista DBA_USERS)

```
CREATE OR REPLACE PROCEDURE usuarios_por_tablespace (p_tablespace dba_users.default_tablespace%type)
is
    cursor c_usuarios is
        SELECT username
        from dba_users
        where default_tablespace=p_tablespace;
begin
    dbms_output.put_line('El tablespace ' || p_tablespace || ' es el predeterminado de los siguientes usuarios:');
    for v_usuario in c_usuarios loop
        dbms_output.put_line(v_usuario.username);
    end loop;
end;
/

exec usuarios_por_tablespace('USERS');
```

[![4.png](https://i.postimg.cc/CKc23bMN/4.png)](https://postimg.cc/DWb5sJXJ)


## 5. Modificar el procedimiento anterior para que haga lo mismo pero devolviendo el número de usuarios que tienen ese tablespace como tablespace por defecto. Nota: Hay que convertir el procedimiento en función

```
CREATE OR REPLACE function f_numusuarios (p_tablespace dba_users.default_tablespace%type)
return number
is
    v_num number;
begin
    select count(username) into v_num from dba_users where default_tablespace=p_tablespace;
    return v_num;
end;
/
```

## 6. Hacer un procedimiento llamado mostrar_usuarios_por_tablespace que muestre por pantalla un listado de los tablespaces existentes con la lista de usuarios de cada uno y el número de los mismos, así: (Vistas DBA_TABLESPACES y DBA_USERS)


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


### Codigo:

```
CREATE OR REPLACE PROCEDURE mostrar_upt
is
    cursor c_tablespaces is
        SELECT tablespace_name
        from dba_tablespaces;
    v_total_usuario number;
    v_total number:=0;
begin
    for v_tablespace in c_tablespaces loop
        dbms_output.put_line('Tablespace ' || v_tablespace.tablespace_name);
        usuarios_por_tablespace(v_tablespace.tablespace_name);
        v_total_usuario:=f_numusuarios(v_tablespace.tablespace_name);
        v_total:=v_total+v_total_usuario;
        dbms_output.put_line('Total Usuarios tablespace ' || v_tablespace.tablespace_name || ': ' || v_total_usuario);
    end loop;
    dbms_output.put_line('Total Usuarios BD : ' || v_total);
end;
/

exec mostrar_upt;
```

[![6.png](https://i.postimg.cc/zXTcJsX3/6.png)](https://postimg.cc/MvKD5rB8)


## 7. Hacer un procedimiento llamado mostrar_codigo_fuente  que reciba el nombre de otro procedimiento y muestre su código fuente. (DBA_SOURCE)

```
CREATE OR REPLACE PROCEDURE mostrar_cod (p_nombre dba_source.name%type)
is
    cursor c_codigo is
        SELECT text
        from dba_source
        where name=p_nombre;
begin
    for v_codigo in c_codigo loop
        dbms_output.put_line(v_codigo.text);
    end loop;
end;
/

exec mostrar_cod('usuarios_por_tablespace');
```

### No funciona bien

## 8. Hacer un procedimiento llamado mostrar_privilegios_usuario que reciba el nombre de un usuario y muestre sus privilegios de sistema y sus privilegios sobre objetos. (DBA_SYS_PRIVS y DBA_TAB_PRIVS)

```
CREATE OR REPLACE PROCEDURE mostrar_privilegios_usuario (
    p_usuario IN VARCHAR2
)
IS
BEGIN
    DBMS_OUTPUT.PUT_LINE('Privilegios de Sistema:');
    FOR sys_priv IN (SELECT privilege
                     FROM dba_sys_privs
                     WHERE grantee = p_usuario)
    LOOP
        DBMS_OUTPUT.PUT_LINE(sys_priv.privilege);
    END LOOP;

    DBMS_OUTPUT.PUT_LINE('Privilegios sobre Objetos:');
    FOR tab_priv IN (SELECT privilege, table_name
                     FROM dba_tab_privs
                     WHERE grantee = p_usuario)
    LOOP
        DBMS_OUTPUT.PUT_LINE(tab_priv.privilege || ' en ' || tab_priv.table_name);
    END LOOP;
END;
/

exec mostrar_privilegios_usuario('SYS');
```

[![8.png](https://i.postimg.cc/BZrzfS3d/8.png)](https://postimg.cc/pyBCDtpZ)


## 9. Realiza un procedimiento llamado listar_comisiones que nos muestre por pantalla un listado de las comisiones de los empleados agrupados según la localidad donde está ubicado su departamento con el siguiente formato:

```
Localidad NombreLocalidad
	
Departamento: NombreDepartamento

		Empleado1 ……. Comisión 1
		Empleado2 ……. Comisión 2
		.	
		.
		.
		Empleadon ……. Comision n

	Total Comisiones en el Departamento NombreDepartamento: SumaComisiones

	Departamento: NombreDepartamento

		Empleado1 ……. Comisión 1
		Empleado2 ……. Comisión 2
		.	
		.		.
		Empleadon ……. Comision n

	Total Comisiones en el Departamento NombreDepartamento: SumaComisiones
	.	
	.
Total Comisiones en la Localidad NombreLocalidad: SumaComisionesLocalidad

Localidad NombreLocalidad
.
.

Total Comisiones en la Empresa: TotalComisiones

```

Nota: Los nombres de localidades, departamentos y empleados deben aparecer por orden alfabético.

Si alguno de los departamentos no tiene ningún empleado con comisiones, aparecerá un mensaje informando de ello en lugar de la lista de empleados.

El procedimiento debe gestionar adecuadamente las siguientes excepciones:

a)	La tabla Empleados está vacía.
b)	Alguna comisión es mayor que 10000.


Codigo:

```
CREATE OR REPLACE PROCEDURE listar_comisiones IS
  v_total_empresa NUMBER := 0;
  v_total_localidad NUMBER;
  v_total_departamento NUMBER;
BEGIN
  SELECT COUNT(*) INTO v_total_empresa FROM emp;

  IF v_total_empresa = 0 THEN
    DBMS_OUTPUT.PUT_LINE('La tabla Empleados está vacía.');
    RETURN;
  END IF;

  FOR loc_rec IN (SELECT DISTINCT loc FROM emp ORDER BY loc) LOOP
    v_total_localidad := 0;
    
    DBMS_OUTPUT.PUT_LINE('Localidad: ' || loc_rec.loc);

    FOR dept_rec IN (SELECT DISTINCT deptno, dname FROM dept WHERE deptno IN (SELECT DISTINCT deptno FROM emp WHERE loc = loc_rec.loc) ORDER BY dname) LOOP
      v_total_departamento := 0;

      DBMS_OUTPUT.PUT_LINE('  Departamento: ' || dept_rec.dname);

      FOR emp_rec IN (SELECT ename, comm FROM emp WHERE loc = loc_rec.loc AND deptno = dept_rec.deptno ORDER BY ename) LOOP
        v_total_departamento := v_total_departamento + NVL(emp_rec.comm, 0);
        DBMS_OUTPUT.PUT_LINE('    ' || emp_rec.ename || ' ....... Comisión ' || NVL(emp_rec.comm, 0));
      END LOOP;

      IF v_total_departamento = 0 THEN
        DBMS_OUTPUT.PUT_LINE('    No hay empleados con comisiones en este departamento.');
      ELSE
        DBMS_OUTPUT.PUT_LINE('    Total Comisiones en el Departamento ' || dept_rec.dname || ': ' || v_total_departamento);
      END IF;

      v_total_localidad := v_total_localidad + v_total_departamento;
    END LOOP;

    IF v_total_localidad = 0 THEN
      DBMS_OUTPUT.PUT_LINE('  No hay empleados con comisiones en esta localidad.');
    ELSE
      DBMS_OUTPUT.PUT_LINE('  Total Comisiones en la Localidad ' || loc_rec.loc || ': ' || v_total_localidad);
    END IF;

    v_total_empresa := v_total_empresa + v_total_localidad;
  END LOOP;

  DBMS_OUTPUT.PUT_LINE('Total Comisiones en la Empresa: ' || v_total_empresa);

  FOR emp_rec IN (SELECT ename, comm FROM emp WHERE NVL(comm, 0) > 10000) LOOP
    DBMS_OUTPUT.PUT_LINE('¡Advertencia! ' || emp_rec.ename || ' tiene una comisión mayor que 10000.');
  END LOOP;
END;
/
```

### No funciona bien


## 10. Realiza un procedimiento que reciba el nombre de una tabla y muestre los nombres de las restricciones que tiene, a qué columna afectan y en qué consisten exactamente. (DBA_TABLES, DBA_CONSTRAINTS, DBA_CONS_COLUMNS)

```
CREATE OR REPLACE PROCEDURE mostrar_restricciones(
    p_table_name IN VARCHAR2
) IS
BEGIN
    FOR constraint_info IN (
        SELECT c.constraint_name, 
               c.constraint_type,
               cols.column_name,
               c.search_condition
        FROM user_constraints c
        JOIN user_cons_columns cols ON c.constraint_name = cols.constraint_name
        WHERE c.table_name = p_table_name
    ) 
    LOOP
        DBMS_OUTPUT.PUT_LINE('Nombre de Restricción: ' || constraint_info.constraint_name);
        DBMS_OUTPUT.PUT_LINE('Tipo de Restricción: ' || constraint_info.constraint_type);
        DBMS_OUTPUT.PUT_LINE('Columna Afectada: ' || constraint_info.column_name);
        DBMS_OUTPUT.PUT_LINE('Condición: ' || constraint_info.search_condition);
        DBMS_OUTPUT.PUT_LINE(' ');
    END LOOP;
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('La tabla ' || p_table_name || ' no tiene restricciones.');
END;
/

exec mostrar_restricciones('EMP');
```

[![10.png](https://i.postimg.cc/50jrbj6M/10.png)](https://postimg.cc/hzWs0DLC)


## 11. Realiza al menos dos de los ejercicios anteriores en Postgres usando PL/pgSQL.

[![11.png](https://i.postimg.cc/Y0WyLV6n/11.png)](https://postimg.cc/D47ctxYL)

Y añadimos los inserts que no hice captura de eso.

## 11.1

```
CREATE or replace PROCEDURE mostrar_empleado() AS $$
DECLARE
    v_nombre emp.ename%type;
    v_salario emp.sal%type;
BEGIN
    select ename,sal into v_nombre,v_salario from emp where empno=7782;
    RAISE NOTICE 'El nombre del empleado es %, y su salario es %', v_nombre,v_salario;
END;
$$ LANGUAGE plpgsql;

CALL mostrar_empleado();
```

[![11-1.png](https://i.postimg.cc/Ss5vsqMY/11-1.png)](https://postimg.cc/S26rTBVm)


## 11.2

```
CREATE OR REPLACE PROCEDURE mostrar_empleado2 (p_empno emp.empno%TYPE) AS $$
DECLARE
    v_nombre emp.ename%type;
    v_salario emp.sal%type;
BEGIN
    SELECT ename, sal INTO v_nombre, v_salario FROM emp WHERE empno = p_empno;
    RAISE NOTICE 'El nombre del empleado es %, y su salario es %', v_nombre, v_salario;
END;
$$ LANGUAGE plpgsql;

CALL mostrar_empleado2(7900);
```

[![11-2.png](https://i.postimg.cc/sfQLLBkY/11-2.png)](https://postimg.cc/yWKngNYx)
