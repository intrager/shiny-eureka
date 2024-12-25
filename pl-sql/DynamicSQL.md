# Dynamic SQL 개요

> **Dynamic SQL** is a programming methodology for generating and running SQL statements at run time.

> 동적 SQL은 런타임 시 SQL문을 생성하고 실행하기 위한 프로그래밍 방법론이다.

## Static SQL vs Dynamic SQL

> If it can be done in static SQL, do it in static SQL.

1. Static SQL provides compile time checking. Dynamic SQL does not.
2. Static SQL creates schema object dependencies. Dynamic SQL does not.
3. Dynamic SQL comes with greater security risks.
4. Static SQL usually performs better than dynamic SQL
5. Static SQL is easier to read and maintain than Dynamic SQL.

## dynamic SQL 개념 맛보기 (vs Static SQL)

### Static SQL

```sql
-- 기본 구문
declare
    v_birth customer_info.birth%type;
begin
    -- 생년 가져오기
    select birth
      into v_birth
    from customer_info
    where customer_id = 'C001'
    ;
    dbms_output.put_line('v_birth : ' || v_birth);
end;
```

### Dynamic SQL

```sql
-- 기본 구문
declare
    v_birth customer_info.birth%type;
begin
    -- 생년 가져오기
    execute immediate
       'select birth'
    || 'from customer_info'
    || 'where customer_id = "C001"'
    into v_birth
    ;
    dbms_output.put_line('v_birth : ' || v_birth);
end;
```

변수를 따로 만들어서 그 안에 쿼리문을 집어넣어도 된다. JDBC를 생각하면 쉽다. 다만 아래와 같이 `||`로 바인딩하는 방법은 자주 사용하지는 않는다.

```sql
declare
    v_birth customer_info.birth%type;
    v_query varchar2(1000);
    p_customer_id customer_info.customer_id%type := 'C001';
begin
    v_query := ' select birth from customer_info where customer_id='''||p_customer_id||'''';
    -- 생년 가져오기
    execute immediate v_query
    into v_birth
    ;
end;
```

대신에 `:변수명`을 통해서 변수를 바인딩할 수 있다.

```sql
declare
    v_birth customer_info.birth%type;
    v_query varchar2(1000);
    p_customer_id customer_info.customer_id%type := 'C001';
begin
    -- v_query := ' select birth from customer_info where customer_id='''||p_customer_id||'''';
    v_query := ' select birth from customer_info where customer_id = :cst';

    -- 생년 가져오기
    execute immediate v_query
    into v_birth
    using p_customer_id
    ;
end;
```

`:변수명`으로 할 때는 아래 execute immediate문에서 using을 통해 p_customer_id를 집어넣는다.

#### 변수 두 개 이상을 돌려보고 싶을 때

정적 쿼리로는 변수 한 번 세팅, 쿼리 한 번 세팅, 변수 한 번 세팅, 쿼리 한 번 세팅 총 2쌍씩 구문을 만들어야 한다.

이 구문이 너무 많으면 수정이 필요할 때 어려워서 재활용하고자 동적 쿼리를 응용할 수 있다.

```sql
declare
    v_birth customer_info.birth%type;
    v_query varchar2(1000);
    p_customer_id customer_info.customer_id%type := 'C001';
begin
    -- v_query := 'select birth from customer_info where customer_id='''||p_customer_id||'''';
    v_query := 'select birth from customer_info where customer_id = :cst';

    
    -- 생년 가져오기
    execute immediate v_query into v_birth using p_customer_id;
    dbms_output.put_line(p_customer_id || ' : ' || v_birth);
    
    p_customer_id := 'C002';
    execute immediate v_query into v_birth using p_customer_id;
    dbms_output.put_line(p_customer_id || ' : ' || v_birth);
end;
```

주의할 점은 동적 쿼리의 컬럼, 테이블에는 `:변수`로 세팅하면 안 된다.

```sql
declare
    v_birth customer_info.birth%type;
    v_query varchar2(1000);
    p_customer_id customer_info.customer_id%type := 'C001';
    v_col varchar2(100);
begin
    v_col := 'birth';
    v_query := 'select :v_col from customer_info where customer_id = :cst';
    
    -- 생년 가져오기
    execute immediate v_query into v_birth using v_col, p_customer_id; -- ORA-01722: invalid number
    dbms_output.put_line(p_customer_id || ' : ' || v_birth);
end;
```

런타임 시에 실행한다고 해도, 이 SQL문은 SQL엔진에 가서 실행된다. 이 바인딩은 where 조건에 있는 바인딩은 바인딩이 가능하지만, 컬럼과 테이블, 오라클 DB 내부에서 사용하는 것들은 바인딩할 수 없다.
`||`로 변수를 넣어서 세팅해야 한다.

# Dynamic SQL - DML 실습

## Function을 활용한 Dynamic SQL

익명 블록에서 팡숀을 만들어 실습해보았다.

```sql
declare
--    v_col varchar2(100); -- 전역변수, 로컬변수 있어도 사용가능
    v_val varchar2(100);
    
    function f_get_val(p_key in varchar2, p_col in varchar2)
    return varchar2
    is
        r_value varchar2(100);
        v_query varchar2(1000); -- 로컬변수
    begin
        v_query := 'select ' || p_col || ' from menu where menu_id = :key';
        execute immediate v_query into r_value using p_key;
        return r_value;
    end f_get_val;
begin
    v_val := f_get_val('M003', 'menu_name');
    
    dbms_output.put_line('v_val : ' || v_val);
end;
```

## Procedure를 활용한 Dynamic SQL

익명 블록에서 프로시저를 만들어 실습해보았다.

### 동적 쿼리를 활용한 update

```sql
declare
    p_menu_id menu.menu_id%type := 'M001';
    p_price menu.menu_price%type := 9494;
    v_query varchar2(1000);
begin
    v_query := 'update menu set menu_price = :price where menu_id = :key';
    dbms_output.put_line(v_query);
    execute immediate v_query using p_price, p_menu_id;
    dbms_output.put_line(sql%rowcount);
end;
```

### 프로시저와 동적 쿼리를 활용한 update

```sql
declare
    procedure sp_update_menu(p_key in varchar2, p_col in varchar2, p_val in varchar2)
    is
        v_query varchar2(1000);
    begin
        v_query := 'update menu set ' || p_col || ' = :param where menu_id = :key';
        execute immediate v_query using p_val, p_key;
    end sp_update_menu;
begin
    sp_update_menu('M004', 'menu_price', 10000);
    dbms_output.put_line(sql%rowcount);
end;
```

# Dynamic SQL - DDL 실습

```sql
declare
begin
    delete from temp_order2;

    insert into temp_order2
    select * from temp_order;

    commit;
end;
```

delete문을 truncate로 바꾼다면 아래처럼 바꿀 수 있다.

```sql
delete
    v_query varchar2(1000);
begin
    -- 테이블 데이터 삭제
    v_query := 'truncate table temp_order2 drop storage';
    execute immediate v_query;
    
    -- 인덱스 삭제
    v_query := 'drop index idx_temp_order2_01';
    execute immediate v_query;

    -- 데이터 입력
    insert into temp_order2 select * from temp_order;

    -- 인덱스 생성
    v_query := 'create index idx_temp_order2_01 on temp_order2(customer_id)';
    execute immediate v_query;

    insert into temp_order2 select * from temp_order;

    commit;
end;
```

## DDL을 활용하는 사례

> 매일의 루틴 작업이 있다.
> ex) 장바구니에 있는 내용을 매일 저녁에 비워야 한다. 새 장바구니로 만들어야 한다.  
> ex) 어제까지의 주문 내용을 항상 새로운 장바구니에 넣는다. 등

운영에 사용하는 테이블은 트랜잭션이 많이 일어난다. 그런 테이블을 많이 쓰면 성능에 문제가 있어, 이를 해결하고자 테이블을 별도로 만들어 놓는 경우가 있다.

```sql
create table real_order2
as select * from real_order;
```

새로 만든 `real_order2` 테이블에는 아래처럼 데이터를 집어넣을 수 있다.

이렇게 데이터를 집어넣는 예는 다음이 있다.
- ex) 매일 주문서를 갱신한다.
- eX) 증분만 넣으면 되지 않나? 다 지우고 넣나?
    - 어제꺼, 그제꺼, 10일 전에 변경된 데이터가 존재할 수 있으므로, 감안할 수 없어 기존의 테이블을 비우고 다시 만드는 경우가 있다.

```sql
insert into real_order2
select * from real_order;
```

작은 수량의 데이터라면 위와 같은 쿼리로 해도 되지만, 100만 건 등의 대량의 데이터라면 다른 방법으로 해야 성능 등 문제가 발생하지 않게 처리할 수 있다.

먼저 인덱스를 활용하는 방법이 있는데, dml을 먼저 한 뒤 조회를 하기 전에 인덱스를 만들면 좋다.
인덱스를 없애고 dml을 해주는 게 성능상 문제가 덜하다.

```sql
create index idx_real_order2_01 on real_order2(order_no, order_sequence);
create index idx_real_order2_02 on real_order2(customer_id);
```

```sql
declare
begin
    delete from real_order2;

    insert into real_order2
    select * from real_order;

    commit;
end;
```

이 쿼리에 있는 delete문이 만약 100만 건을 지운다고 하면, 이게 공간을 확보해주는 게 아니고 테이블이 잡고 있다. 그 공간은 다른 곳에서 쓸 수 없다.  
다른 곳에서 insert 하는 것도 기존 사용하던 것을 재사용하는 거라 사이즈에 제약도 생기고, 여러 조건/성능에 문제가 생긴다.  
이를 해결하기 위해서 `truncate`를 사용할 수 있다.

```sql
declare
    v_query varchar2(1000);
begin
    -- 테이블 데이터 삭제
    v_query := 'truncate table real_order2 drop storage';
    execute immediate v_query;

    -- 인덱스 삭제1
    v_query := 'drop index idx_real_order2_01';
    execute immediate v_query;

    -- 인덱스 삭제2
    v_query := 'drop index idx_real_order2_02';
    execute immediate v_query;

    insert into real_order2
    select * from real_order;

    -- 인덱스 생성1
    v_query := 'create index idx_real_order2_01 on real_order2(order_no, order_sequence)';
    execute immediate v_query;

    -- 인덱스 생성2
    v_query := 'create index idx_real_order2_02 on real_order2(customer_id)';
    execute immediate v_query;

    commit;
end;
```

위의 쿼리처럼 만들어놓으면, 이용량이 적은 밤에 daemon 등을 활용하여 주문서의 내용을 넣을 수 있다. 

`drop`이 부담스러우면 `alter unuseable` 및 `rebuild`등을 활용할 수 있다.

