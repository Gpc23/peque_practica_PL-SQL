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


