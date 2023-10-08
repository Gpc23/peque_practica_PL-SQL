# peque_practica_PL-SQL

## Hacer un procedimiento que muestre el nombre y el salario del empleado cuyo código es 7082

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
