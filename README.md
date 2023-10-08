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
