# Exception - Predefined Exceptions

## 시스템 정의 예외처리(Predefined Exceptions) & 사용자 정의 예외처리 (User-Defined Exceptions)


### 전체 구문 Exception 처리

```sql
declare
    -- 변수 선언, 서브 프로그램
begin
    -- Statements
exception when 에러유형 then
            -- Error 메시지
          when 에러유형 then
            -- Error 메시지
          when others then
            -- Error 메시지
end;
```

PL/SQL에서 select문은 반드시 하나의 값만 나와야 한다. 여기서 select 했는데 값이 없다던지, 다른 값이 들어간다던지 등에는 에러가 발생한다.  
또 한 가지는 데이터가 두 개 이상 나올 때이다.

```sql
declare
  v_name customer_info.name%type;
  v_menu_name menu.menu_name%type;
begin
  -- 메뉴명 가져오기
  select menu_name
  into v_menu_name
  from menu
  where menu_id = 'M001'
  ;
  
  -- 고객명 가져오기
  select name
  into v_name
  from customer_info
  where customer_id = 'C001'
  ;
  
  dbms_output.put_line('이름 : ' || v_name);
  dbms_output.put_line('메뉴 : ' || v_menu_name);

--  exception when 에러유형 then
    -- error 메시지 when others then
    -- error 메시지
end;
```

나는 어떤 오류든지 여기서 잡아내겠다 -> `others`

```sql
...
EXCEPTION when others then
...
```


### 실습

```sql
declare
  r_return_code varchar2(500);
  r_return_message varchar2(500);
begin
  pkg_orders.psp_make_order(null, r_return_code, r_return_message);
  
  dbms_output.put_line('r_return_code -> ' || r_return_code);
  dbms_output.put_line('r_return_message -> ' || r_return_message);
  
  if r_return_code = '0' then
    commit;
  else
    rollback;
  end if;
end;
```

```sql
create or replace PACKAGE BODY PKG_ORDERS AS

  FUNCTION PF_GET_ADDPOINT (
      P_PRICE IN NUMBER
  ) RETURN NUMBER AS
  -- 적립 포인트
  v_point customer_info.point%type;
  BEGIN
      v_point := round(p_price * 0.1);
      RETURN v_point;
  Exception when others then
      return 0;
  END PF_GET_ADDPOINT;

  PROCEDURE PSP_MAKE_ORDER (
      P_CUSTOMER_ID IN VARCHAR2 
    , R_RETURN_CODE OUT VARCHAR2 
    , R_RETURN_MESSAGE OUT VARCHAR2 
  ) AS
    -- real_order 테이블 컬럼 값을 변수로 사용
    r_real_order real_order%rowtype;
    
    -- 옵션가 변수
    v_price_size menu.menu_price%type;
    v_price_ice menu_option.option_price%type;
    BEGIN
        -- 실행구문
        -- 1. 장바구니의 자료를 가져온다.
        -- 2. 장바구니의 개수만큼 loop를 진행한다.
        for fc in (select * from temp_order where customer_id = nvl(p_customer_id, customer_id))
        loop
            -- 3. 주문서에 필요한 기본정보를 가져온다.
            -- 3-1. 맥주일 경우 미성년자인지 체크한다.
            dbms_output.put_line(fc.customer_id);
            -- 기준금액
            select x.menu_price
            into r_real_order.price
            from menu x
--            where x.menu_id = fc.menu_id;
        ;
            -- 4. 옵션가를 가져온다.
            -- 사이즈 옵션 가격
            v_price_size := f_get_option_price(fc.menu_id, fc.menu_size);
            -- 아이스 옵션 가격
            v_price_ice := f_get_option_price(fc.menu_id, fc.menu_ice);
        
            -- 5. 주문금액을 계산한다. (수량 * (기준단가 + 옵션가))
            -- 주문 단가
            r_real_order.price := r_real_order.price + v_price_size + v_price_ice;
            r_real_order.total_price := r_real_order.price * fc.quantity;
        
            -- 6. 포인트를 10% 추가한다.
            r_real_order.point_add := f_get_addpoint(r_real_order.total_price);
            -- 7. 주문서를 생성한다.
            insert into real_order (
                order_sequence
              , customer_id
              , menu_id
              , menu_size
              , menu_ice
              , quantity
              , price
              , total_price
              , point_use
              , point_add
            ) values (
                1
              , fc.customer_id
              , fc.menu_id
              , fc.menu_size
              , fc.menu_ice
              , fc.quantity
              , r_real_order.price
              , r_real_order.total_price
              , fc.point_use
              , r_real_order.point_add
            );
            -- 8. 개인정보에 포인트를 넣어준다.
            update customer_info
            set point = point + r_real_order.point_add
            where customer_id = fc.customer_id;
        end loop;
        r_return_code := '0';
        r_return_message := 'OK';
        
      exception when others then
        r_return_code := 0;
        r_return_message := '오류가 발생했습니다.';
    END PSP_MAKE_ORDER;
END PKG_ORDERS;
```

결과

```log
C001
r_return_code -> 0
r_return_message -> 오류가 발생했습니다.
```

### 다른 예외 처리 (Predefined Exceptions)

#### NO_DATA_FOUND & TOO_MANY_ROWS

```sql
create or replace PACKAGE BODY PKG_ORDERS AS

  FUNCTION PF_GET_ADDPOINT (
      P_PRICE IN NUMBER
  ) RETURN NUMBER AS
  -- 적립 포인트
  v_point customer_info.point%type;
  BEGIN
      v_point := round(p_price * 0.1);
      RETURN v_point;
  Exception when others then
      return 0;
  END PF_GET_ADDPOINT;

  PROCEDURE PSP_MAKE_ORDER (
      P_CUSTOMER_ID IN VARCHAR2 
    , R_RETURN_CODE OUT VARCHAR2 
    , R_RETURN_MESSAGE OUT VARCHAR2 
  ) AS
    -- real_order 테이블 컬럼 값을 변수로 사용
    r_real_order real_order%rowtype;
    
    -- 옵션가 변수
    v_price_size menu.menu_price%type;
    v_price_ice menu_option.option_price%type;
    BEGIN
        -- 실행구문
        -- 1. 장바구니의 자료를 가져온다.
        -- 2. 장바구니의 개수만큼 loop를 진행한다.
        for fc in (select * from temp_order where customer_id = nvl(p_customer_id, customer_id))
        loop
            -- 3. 주문서에 필요한 기본정보를 가져온다.
            -- 3-1. 맥주일 경우 미성년자인지 체크한다.
            dbms_output.put_line(fc.customer_id);
            -- 기준금액
            select x.menu_price
            into r_real_order.price
            from menu x
--            where x.menu_id = fc.menu_id;
        ;
            -- 4. 옵션가를 가져온다.
            -- 사이즈 옵션 가격
            v_price_size := f_get_option_price(fc.menu_id, fc.menu_size);
            -- 아이스 옵션 가격
            v_price_ice := f_get_option_price(fc.menu_id, fc.menu_ice);
        
            -- 5. 주문금액을 계산한다. (수량 * (기준단가 + 옵션가))
            -- 주문 단가
            r_real_order.price := r_real_order.price + v_price_size + v_price_ice;
            r_real_order.total_price := r_real_order.price * fc.quantity;
        
            -- 6. 포인트를 10% 추가한다.
            r_real_order.point_add := f_get_addpoint(r_real_order.total_price);
            -- 7. 주문서를 생성한다.
            insert into real_order (
                order_sequence
              , customer_id
              , menu_id
              , menu_size
              , menu_ice
              , quantity
              , price
              , total_price
              , point_use
              , point_add
            ) values (
                1
              , fc.customer_id
              , fc.menu_id
              , fc.menu_size
              , fc.menu_ice
              , fc.quantity
              , r_real_order.price
              , r_real_order.total_price
              , fc.point_use
              , r_real_order.point_add
            );
            -- 8. 개인정보에 포인트를 넣어준다.
            update customer_info
            set point = point + r_real_order.point_add
            where customer_id = fc.customer_id;
        end loop;
        r_return_code := '0';
        r_return_message := 'OK';
        
      exception when others then
        r_return_code := 0;
        r_return_message := '오류가 발생했습니다.';
    END PSP_MAKE_ORDER;
END PKG_ORDERS;
```

```sql
declare
  v_name customer_info.name%type;
  v_menu_name menu.menu_name%type;
begin
  -- 메뉴명 가져오기
  select menu_name -- 데이터가 2개 이상입니다.
  into v_menu_name
  from menu
--  where menu_id = 'M0011234' -- 데이터를 찾지 못했습니다.
  ;
  
  -- 고객명 가져오기
  select name
  into v_name
  from customer_info
  where customer_id = 'C001'
  ;
  
  dbms_output.put_line('이름 : ' || v_name);
  dbms_output.put_line('메뉴 : ' || v_menu_name);

  exception when NO_DATA_FOUND then
      dbms_output.put_line('데이터를 찾지 못했습니다.');
    when TOO_MANY_ROWS then
      dbms_output.put_line('데이터가 2개 이상입니다.');
end;
```

#### logs 테이블을 따로 만들어서 오류 내용 넣기

```sql
create table logs (
    name varchar2(200),
    memo varchar2(200),
    register_date date default sysdate
);
```

```sql
declare
  v_name customer_info.name%type;
  v_menu_name menu.menu_name%type;
begin
  -- 메뉴명 가져오기
  select menu_name
  into v_menu_name
  from menu
  where menu_id = 'M0011234'
  ;
  
  -- 고객명 가져오기
  select name
  into v_name
  from customer_info
  where customer_id = 'C001'
  ;
  
  dbms_output.put_line('이름 : ' || v_name);
  dbms_output.put_line('메뉴 : ' || v_menu_name);

    exception when NO_DATA_FOUND then
      dbms_output.put_line('데이터를 찾지 못 했습니다.');
      insert into logs (name, memo)
      values ('익명PL/SQL', '데이터를 찾지 못 하는 오류 발생');
    when TOO_MANY_ROWS then
      dbms_output.put_line('데이터가 2개 이상입니다.');
      insert into logs (name, memo)
      values ('익명PL/SQL', '2개 이상의 데이터 추출 오류');
    when others then
      dbms_output.put_line('오류 발생');
      insert into logs (name, memo)
      values ('익명PL/SQL', '알 수 없는 오류가 발생했습니다.');
end;
```

```sql
declare
  v_name customer_info.name%type;
  v_menu_name menu.menu_name%type;
begin
  -- 메뉴명 가져오기
  select menu_name
  into v_menu_name
  from menu
  where menu_id = 'M0011234'
  ;
  
  -- 고객명 가져오기
  select name
  into v_name
  from customer_info
  where customer_id = 'C001'
  ;
  
  dbms_output.put_line('이름 : ' || v_name);
  dbms_output.put_line('메뉴 : ' || v_menu_name);

    exception when others then
      dbms_output.put_line('오류 발생');
      insert into logs (name, memo)
      values ('익명PL/SQL', '알 수 없는 오류가 발생했습니다.');
end;
```

### 블록단위 Exception 처리

```sql
declare
  -- 변수선언, 서브 프로그램
begin
  declare
    -- 블록의 블록 안에서만 사용 가능한 변수 선언
  begin
    -- statements
  exception when 에러유형 then
    -- error 메시지
  end;
exception when 에러유형 then
    -- error 메시지
  when 에러유형 then
    -- error 메시지
  when others then
    -- error 메시지
end;
```

이 오류는 발생하더라도 굳이 exception 처리를 전체에 영향주고 싶지 않다.
너 혼자 오류나고 없어져라. 라는 뜻으로 활용할 수 있다.

```sql
declare
  v_name customer_info.name%type;
  v_menu_name menu.menu_name%type;
begin
  declare
  begin
    -- 메뉴명 가져오기
    select menu_name
      into v_menu_name
    from menu
    where menu_id = 'M001'
    ;
    exception when 에러유형 then
      -- error 메시지
  end;

  -- 고객명 가져오기
  select name
    into v_name
  from customer_info
  where customer_id = 'C001'
  ;

  dbms_output.put_line('이름 : ' || v_name);
  dbms_output.put_line('메뉴 : ' || v_menu_name);

  exception when 에러유형 then
    -- error 메시지
end;
```

#### 실습

```sql
declare
  v_name customer_info.name%type;
  v_menu_name menu.menu_name%type;
begin
    begin
      -- 메뉴명 가져오기
      select menu_name
      into v_menu_name
      from menu
      where menu_id = 'M0011234'
      ;
      dbms_output.put_line('메뉴 : ' || v_menu_name);
      exception when others then
      dbms_output.put_line('메뉴 가져오는 구간 오류 발생');
    end;
  
  -- 고객명 가져오기
  select name
  into v_name
  from customer_info
  where customer_id = 'C001'
  ;
  
  dbms_output.put_line('이름 : ' || v_name);
  dbms_output.put_line('메뉴 : ' || v_menu_name);

    exception when others then
      dbms_output.put_line('오류 발생');
      insert into logs (name, memo)
      values ('익명PL/SQL', '알 수 없는 오류가 발생했습니다.');
end;
```

패키지 내에 있던 예외 처리를 빼놓고 돌리면 에러가 난다.

```sql
create or replace PACKAGE BODY PKG_ORDERS AS

  FUNCTION PF_GET_ADDPOINT (
      P_PRICE IN NUMBER
  ) RETURN NUMBER AS
  -- 적립 포인트
  v_point customer_info.point%type;
  BEGIN
      v_point := round(p_price * 0.1);
      RETURN v_point;
  Exception when others then
      return 0;
  END PF_GET_ADDPOINT;

  PROCEDURE PSP_MAKE_ORDER (
      P_CUSTOMER_ID IN VARCHAR2 
    , R_RETURN_CODE OUT VARCHAR2 
    , R_RETURN_MESSAGE OUT VARCHAR2 
  ) AS
    -- real_order 테이블 컬럼 값을 변수로 사용
    r_real_order real_order%rowtype;
    
    -- 옵션가 변수
    v_price_size menu.menu_price%type;
    v_price_ice menu_option.option_price%type;
    BEGIN
        -- 실행구문
        -- 1. 장바구니의 자료를 가져온다.
        -- 2. 장바구니의 개수만큼 loop를 진행한다.
        for fc in (select * from temp_order where customer_id = nvl(p_customer_id, customer_id))
        loop
            -- 3. 주문서에 필요한 기본정보를 가져온다.
            -- 3-1. 맥주일 경우 미성년자인지 체크한다.
            dbms_output.put_line(fc.customer_id);
            -- 기준금액
            select x.menu_price
            into r_real_order.price
            from menu x
--            where x.menu_id = fc.menu_id;
        ;
            -- 4. 옵션가를 가져온다.
            -- 사이즈 옵션 가격
            v_price_size := f_get_option_price(fc.menu_id, fc.menu_size);
            -- 아이스 옵션 가격
            v_price_ice := f_get_option_price(fc.menu_id, fc.menu_ice);
        
            -- 5. 주문금액을 계산한다. (수량 * (기준단가 + 옵션가))
            -- 주문 단가
            r_real_order.price := r_real_order.price + v_price_size + v_price_ice;
            r_real_order.total_price := r_real_order.price * fc.quantity;
        
            -- 6. 포인트를 10% 추가한다.
            r_real_order.point_add := f_get_addpoint(r_real_order.total_price);
            -- 7. 주문서를 생성한다.
            insert into real_order (
                order_sequence
              , customer_id
              , menu_id
              , menu_size
              , menu_ice
              , quantity
              , price
              , total_price
              , point_use
              , point_add
            ) values (
                1
              , fc.customer_id
              , fc.menu_id
              , fc.menu_size
              , fc.menu_ice
              , fc.quantity
              , r_real_order.price
              , r_real_order.total_price
              , fc.point_use
              , r_real_order.point_add
            );
            -- 8. 개인정보에 포인트를 넣어준다.
            update customer_info
            set point = point + r_real_order.point_add
            where customer_id = fc.customer_id;
        end loop;
        r_return_code := '0';
        r_return_message := 'OK';
        
--      exception when others then
--        r_return_code := 0;
--        r_return_message := '오류가 발생했습니다.';
    END PSP_MAKE_ORDER;
END PKG_ORDERS;
```

```sql
declare
  r_return_code varchar2(500);
  r_return_message varchar2(500);
begin
  pkg_orders.psp_make_order(null, r_return_code, r_return_message);
  
  dbms_output.put_line('r_return_code -> ' || r_return_code);
  dbms_output.put_line('r_return_message -> ' || r_return_message);
  
  if r_return_code = '0' then
    commit;
  else
    rollback;
  end if;
    
end;
```

> ORA-01422: exact fetch returns more than requested number of rows
> ORA-06512: at "INFLEARN.PKG_ORDERS", line 36
> ORA-06512: at line 5
> 01422. 00000 -  "exact fetch returns more than requested number of rows"
> *Cause:    The number specified in exact fetch is less than the rows returned.
> *Action:   Rewrite the query or change number of rows requested

패키지 내에서 예외처리를 하던 것을 없애면 위와 같은 오류가 발생한다.

블록 안의 블록을 만들어 패키지의 오류를 예외처리해보자.

```sql
declare
  r_return_code varchar2(500);
  r_return_message varchar2(500);
begin
  begin
    pkg_orders.psp_make_order(null, r_return_code, r_return_message);
  exception when others then
    dbms_output.put_line('블록 안에서 패키지 예외 처리');
  end;
  
    dbms_output.put_line('r_return_code -> ' || r_return_code);
    dbms_output.put_line('r_return_message -> ' || r_return_message);
  
  if r_return_code = '0' then
    commit;
  else
    rollback;
  end if;
    
end;
```

블록 안의 블록에 변수 선언해서 넣어보기

```sql
declare
  v_name customer_info.name%type;
  v_menu_name menu.menu_name%type;
begin
    declare
      block_v varchar2(100) := '블록 변수'; -- 이 inner 블록 안에서만 사용 가능
    begin
      -- 메뉴명 가져오기
      select menu_name
      into v_menu_name
      from menu
      where menu_id = 'M0011234'
      ;
      dbms_output.put_line('메뉴 : ' || v_menu_name);
      exception when others then
      dbms_output.put_line('메뉴 가져오는 구간 오류 발생 - ' || block_v);
    end;
  
  -- 고객명 가져오기
  select name
  into v_name
  from customer_info
  where customer_id = 'C001'
  ;
  
  dbms_output.put_line('이름 : ' || v_name);
  dbms_output.put_line('메뉴 : ' || v_menu_name);

    exception when others then
      dbms_output.put_line('오류 발생');
      insert into logs (name, memo)
      values ('익명PL/SQL', '알 수 없는 오류가 발생했습니다.');
end;
```

# SQLCODE / SQLERRM (코드와 메시지를 가져오자)

## SQLCODE / SQLERRM function 정의

### SQLCODE

> In an exception handler, the `SQLCODE` function returns the numeric code of the exception being handled. (Outside an exception handler, `SQLCODE` returns `0`)
> For an internally defined exception, the numeric code is the number of the associated Oracle Database error.
> This number is negative except For a user-defined exception, the numeric code is either + 1 (default) or the error code associated with the exception by the `EXCEPTION_INT` pragma.

> 예외 처리기에서 SQLCODE 함수는 처리 중인 예외의 숫자 코드를 반환한다. (예외 핸들러 외부에서 SQLCODE는 0을 반환한다.)
> 내부적으로 정의된 예외의 경우 숫자 코드는 연관된 Oracle 데이터베이스 오류의 번호다.
> 숫자 코드가 100인 "데이터를 찾을 수 없음" 오류를 제외하고 이 숫자는 음수이다.

### SQLERRM

> The SQLERRM function returns the error message associated with an error code.
>
> Note:
> `DBMS_UTILITY.FORMAT_ERROR_STACK` is recommended over `SQLERRM`, unless you see the `FORALL` statement with its `SAVE EXCEPTIONS` clause.
> For more information, see "Retrieving Error Code and Error Message".

> SQLERRM 함수는 오류 코드와 관련된 오류 메시지를 반환한다.
>
> Note:
> `SAVE EXCEPTIONS` 절과 함께 `FORALL` 문을 사용하지 않는 한 `SQLERRM`보다 `DBMS_UTILITY.FORMAT_ERROR_STACK`을 사용하는 것이 좋다.

![SQLCODE](sqlcode.png)

SQLCODE의 0은 정상처리라고 약속된 상태이다.


## 실습

```sql
declare
  v_name customer_info.name%type;
  v_menu_name menu.menu_name%type;
begin
    declare
      block_v varchar2(100) := '블록 변수';
    begin
      -- 메뉴명 가져오기
      select menu_name
      into v_menu_name
      from menu
      where menu_id = 'M0011234'
      ;
      dbms_output.put_line('메뉴 : ' || v_menu_name);
      exception when others then
      dbms_output.put_line('메뉴 가져오는 구간 오류 발생 - ' || block_v);
    end;
  
  -- 고객명 가져오기
  select name
  into v_name
  from customer_info
  where customer_id = 'C001'
  ;
  
  dbms_output.put_line('이름 : ' || v_name);
  dbms_output.put_line('sqlcode : ' || sqlcode); -- sqlcode : 0
  dbms_output.put_line('sqlerrm : ' || sqlerrm); -- sqlerrm : ORA-0000: normal, successful completion
  dbms_output.put_line('sqlerrm : ' || sqlerrm(sqlcode)); -- sqlerrm : ORA-0000: normal, successful completion

    exception when others then
      dbms_output.put_line('오류 발생');
      insert into logs (name, memo)
      values ('익명PL/SQL', '알 수 없는 오류가 발생했습니다.');
end;
```

`sqlcode`는 정상 실행되었으므로 0을 반환하고, `sqlerrm`은 `sqlcode`값에 따라 반환한다.

sqlerrm은 sqlerrm(sqlcode)와 같다.

각 예외 코드에 따라 에러 메시지도 달라진다.

```sql
declare
  v_name customer_info.name%type;
  v_menu_name menu.menu_name%type;
begin

      -- 메뉴명 가져오기
      select menu_name
      into v_menu_name
      from menu
      where menu_id = 'M0011234'
      ;
      dbms_output.put_line('메뉴 : ' || v_menu_name);

  -- 고객명 가져오기
  select name
  into v_name
  from customer_info
  where customer_id = 'C001'
  ;
  
  dbms_output.put_line('이름 : ' || v_name);

    exception when others then
      dbms_output.put_line('sqlcode : ' || sqlcode); -- sqlcode : 100
      dbms_output.put_line('sqlerrm : ' || sqlerrm); -- sqlerrm : ORA-01403: no data found
end;
```

패키지 내 예외 처리에 sqlcode, sqlerrm을 활용할 수 있다.

```sql
create or replace PACKAGE BODY PKG_ORDERS AS

  FUNCTION PF_GET_ADDPOINT (
      P_PRICE IN NUMBER
  ) RETURN NUMBER AS
  -- 적립 포인트
  v_point customer_info.point%type;
  BEGIN
      v_point := round(p_price * 0.1);
      RETURN v_point;
  Exception when others then
      return 0;
  END PF_GET_ADDPOINT;

  PROCEDURE PSP_MAKE_ORDER (
      P_CUSTOMER_ID IN VARCHAR2 
    , R_RETURN_CODE OUT VARCHAR2 
    , R_RETURN_MESSAGE OUT VARCHAR2 
  ) AS
    -- real_order 테이블 컬럼 값을 변수로 사용
    r_real_order real_order%rowtype;
    
    -- 옵션가 변수
    v_price_size menu.menu_price%type;
    v_price_ice menu_option.option_price%type;
    BEGIN
        -- 실행구문
        -- 1. 장바구니의 자료를 가져온다.
        -- 2. 장바구니의 개수만큼 loop를 진행한다.
        for fc in (select * from temp_order where customer_id = nvl(p_customer_id, customer_id))
        loop
            -- 3. 주문서에 필요한 기본정보를 가져온다.
            -- 3-1. 맥주일 경우 미성년자인지 체크한다.
            dbms_output.put_line(fc.customer_id);
            -- 기준금액
            select x.menu_price
            into r_real_order.price
            from menu x
--            where x.menu_id = fc.menu_id;
        ;
            -- 4. 옵션가를 가져온다.
            -- 사이즈 옵션 가격
            v_price_size := f_get_option_price(fc.menu_id, fc.menu_size);
            -- 아이스 옵션 가격
            v_price_ice := f_get_option_price(fc.menu_id, fc.menu_ice);
        
            -- 5. 주문금액을 계산한다. (수량 * (기준단가 + 옵션가))
            -- 주문 단가
            r_real_order.price := r_real_order.price + v_price_size + v_price_ice;
            r_real_order.total_price := r_real_order.price * fc.quantity;
        
            -- 6. 포인트를 10% 추가한다.
            r_real_order.point_add := f_get_addpoint(r_real_order.total_price);
            -- 7. 주문서를 생성한다.
            insert into real_order (
                order_sequence
              , customer_id
              , menu_id
              , menu_size
              , menu_ice
              , quantity
              , price
              , total_price
              , point_use
              , point_add
            ) values (
                1
              , fc.customer_id
              , fc.menu_id
              , fc.menu_size
              , fc.menu_ice
              , fc.quantity
              , r_real_order.price
              , r_real_order.total_price
              , fc.point_use
              , r_real_order.point_add
            );
            -- 8. 개인정보에 포인트를 넣어준다.
            update customer_info
            set point = point + r_real_order.point_add
            where customer_id = fc.customer_id;
        end loop;
        r_return_code := to_char(sqlcode);
        r_return_message := sqlerrm;
        
      exception when others then
        r_return_code := sqlcode;
        r_return_message := sqlerrm;
    END PSP_MAKE_ORDER;
END PKG_ORDERS;
```

기존에 호출 구문에서 예외처리 하던 거는 없애고 돌리면 다음과 같다.

```sql
declare
  r_return_code varchar2(500);
  r_return_message varchar2(500);
begin
    pkg_orders.psp_make_order(null, r_return_code, r_return_message);
  
    dbms_output.put_line('r_return_code -> ' || r_return_code); -- r_return_code -> -1422
    dbms_output.put_line('r_return_message -> ' || r_return_message); -- r_return_message -> ORA-01422: exact fetch returns more than requested number of rows
  
  if r_return_code = '0' then
    commit;
  else
    rollback;
  end if; 
end;
```

패키지 내부를 다시 정상으로 돌아가게 수정해놓고 돌리면 다음과 같다.

```sql
create or replace PACKAGE BODY PKG_ORDERS AS

  FUNCTION PF_GET_ADDPOINT (
      P_PRICE IN NUMBER
  ) RETURN NUMBER AS
  -- 적립 포인트
  v_point customer_info.point%type;
  BEGIN
      v_point := round(p_price * 0.1);
      RETURN v_point;
  Exception when others then
      return 0;
  END PF_GET_ADDPOINT;

  PROCEDURE PSP_MAKE_ORDER (
      P_CUSTOMER_ID IN VARCHAR2 
    , R_RETURN_CODE OUT VARCHAR2 
    , R_RETURN_MESSAGE OUT VARCHAR2 
  ) AS
    -- real_order 테이블 컬럼 값을 변수로 사용
    r_real_order real_order%rowtype;
    
    -- 옵션가 변수
    v_price_size menu.menu_price%type;
    v_price_ice menu_option.option_price%type;
    BEGIN
        -- 실행구문
        -- 1. 장바구니의 자료를 가져온다.
        -- 2. 장바구니의 개수만큼 loop를 진행한다.
        for fc in (select * from temp_order where customer_id = nvl(p_customer_id, customer_id))
        loop
            -- 3. 주문서에 필요한 기본정보를 가져온다.
            -- 3-1. 맥주일 경우 미성년자인지 체크한다.
            dbms_output.put_line(fc.customer_id);
            -- 기준금액
            select x.menu_price
            into r_real_order.price
            from menu x
            where x.menu_id = fc.menu_id
            ;
        
            -- 4. 옵션가를 가져온다.
            -- 사이즈 옵션 가격
            v_price_size := f_get_option_price(fc.menu_id, fc.menu_size);
            -- 아이스 옵션 가격
            v_price_ice := f_get_option_price(fc.menu_id, fc.menu_ice);
        
            -- 5. 주문금액을 계산한다. (수량 * (기준단가 + 옵션가))
            -- 주문 단가
            r_real_order.price := r_real_order.price + v_price_size + v_price_ice;
            r_real_order.total_price := r_real_order.price * fc.quantity;
        
            -- 6. 포인트를 10% 추가한다.
            r_real_order.point_add := f_get_addpoint(r_real_order.total_price);
            -- 7. 주문서를 생성한다.
            insert into real_order (
                order_sequence
              , customer_id
              , menu_id
              , menu_size
              , menu_ice
              , quantity
              , price
              , total_price
              , point_use
              , point_add
            ) values (
                1
              , fc.customer_id
              , fc.menu_id
              , fc.menu_size
              , fc.menu_ice
              , fc.quantity
              , r_real_order.price
              , r_real_order.total_price
              , fc.point_use
              , r_real_order.point_add
            );
            -- 8. 개인정보에 포인트를 넣어준다.
            update customer_info
            set point = point + r_real_order.point_add
            where customer_id = fc.customer_id;
        end loop;
        r_return_code := to_char(sqlcode);
        r_return_message := sqlerrm;
        
      exception when others then
        r_return_code := sqlcode;
        r_return_message := sqlerrm;
    END PSP_MAKE_ORDER;
END PKG_ORDERS;
```

```log
C001
C002
C003
r_return_code -> 0
r_return_message -> ORA-0000: normal, successful completion
```

`sqlerrm`말고 `DBMS_UTILITY.FORMAT_ERROR_STACK`을 활용할 수 있다.

```sql
declare
  v_name customer_info.name%type;
  v_menu_name menu.menu_name%type;
begin

      -- 메뉴명 가져오기
      select menu_name
      into v_menu_name
      from menu
      where menu_id = 'M0011234'
      ;
      dbms_output.put_line('메뉴 : ' || v_menu_name);

  -- 고객명 가져오기
  select name
  into v_name
  from customer_info
  where customer_id = 'C001'
  ;
  
  dbms_output.put_line('이름 : ' || v_name);

    exception when others then
      dbms_output.put_line('sqlcode : ' || sqlcode);
      dbms_output.put_line('sqlerrm : ' || sqlerrm);
      dbms_output.put_line('sqlerrm : ' || DBMS_UTILITY.FORMAT_ERROR_STACK);
end;
```

# Exception : User-Defined Exceptions (RAISE, RAISE_APPLICATION_ERROR)

## 전체 구문 Exception 처리

```sql
declare
  -- 변수 선언, 서브 프로그램
  e_nocoffee EXCEPTION; -- 커피는 예외처리
begin
  -- statements
  RAISE past_due; -- 사용자 정의 오류가 발생함
exception
  when 사용자정의 오류 then
    -- error 메시지
  when 에러유형 then
    -- error 메시지
  when others then
    -- error 메시지
end;
```

```sql
declare
    v_name customer_info.name%type;
    v_menu_name menu.menu_name%type;
begin
    -- 메뉴명 가져오기
    select menu_name
      into v_menu_name
    from menu
    where menu_id = 'M001'
    ;

    if v_menu_name = '아메리카노' then
        raise v_exception;
    end if;

    dbms_output.put_line('메뉴 : ' || v_menu_name);
exception
    when v_excpetion then
        -- 사용자 정의 예외 처리
    when others then
        -- Error 메시지
end;
```

### RAISE 실습

```sql
declare
  v_name customer_info.name%type;
  v_menu_name menu.menu_name%type;
  e_beer EXCEPTION; -- 맥주는 예외처리
begin
  -- 메뉴명 가져오기
  select menu_name
    into v_menu_name
  from menu
  where menu_id in ('M005')
  ;
  
  if v_menu_name like '%맥주%' then
    dbms_output.put_line('예외처리 : ' || v_menu_name); -- 예외처리 : 흑맥주
    raise e_beer;-- 발생시키는 것이기 때문에 raise, 시스템이 예외처리한 것처럼 예외처리됨
  end if;  
  dbms_output.put_line('메뉴 : ' || v_menu_name);

  -- 고객명 가져오기
  select name
    into v_name
  from customer_info
  where customer_id = 'C001'
  ; 
  dbms_output.put_line('이름 : ' || v_name);
  
  exception
    when others then
      dbms_output.put_line('sqlcode : ' || sqlcode); -- sqlcode : 1
      dbms_output.put_line('sqlerrm : ' || sqlerrm); -- sqlerrm : User-Defined Exception
    insert into logs (name, memo)
    values ('익명 PL/SQL', '알 수 없는 오류가 발생했습니다.');
end;
```

### RAISE_APPLICATION_ERROR

#### RAISE Statement

> The `RAISE` statement explicitly raises an exception.
> Outside an exception handler, you must specify the exception name.
> Inside an exception handler, if you omit the expcetion name, the `RAISE` statement reraises the current exception.

> RAISE문은 명시적으로 예외를 발생시킨다. 예외 처리기 외부에서는 예외 이름을 지정해야 한다.  
> 예외 처리기 내에서 예외 이름을 생략하면 RAISE문이 현재 예외를 다시 발생시킨다.

#### RAISE_APPLICATION_ERROR Procedure

> You can invoke the `RAISE_APPLICATION_ERROR` procedure (defined in the `DBMS_STANDARD` package) only from a stored subprogram or method.
> Typically, you invoke this procedure to raise a user-defined exception and return its error code and error message to the invoker.

> `RAISE_APPLICATION_ERROR` 프로시저 (`DBMS_STANDARD` 패키지에 정의됨)는 저장된 하위 프로그램이나 메서드에서만 호출할 수 있다.
> 일반적으로 이 프로시저를 호출하여 사용자 정의 예외를 발생시키고 해당 오류 코드와 오류 메시지를 호출자에게 반환한다.
> The error_code is an integer in the range -20000 .. -20999

```sql
declare
  v_name customer_info.name%type;
  v_menu_name menu.menu_name%type;
  e_beer EXCEPTION; -- 맥주는 예외처리
begin
  -- 메뉴명 가져오기
  select menu_name
    into v_menu_name
  from menu
  where menu_id in ('M005')
  ;
  
  if v_menu_name like '%맥주%' then
    dbms_output.put_line('예외처리 : ' || v_menu_name);
    RAISE_APPLICATION_ERROR(-20055, '맥주는 안 됨');-- 발생시키는 것이기 때문에 raise, 시스템이 예외처리한 것처럼 예외처리됨
  end if;  
  dbms_output.put_line('메뉴 : ' || v_menu_name);

  -- 고객명 가져오기
  select name
    into v_name
  from customer_info
  where customer_id = 'C001'
  ; 
  dbms_output.put_line('이름 : ' || v_name);
  
  exception
    when others then
      dbms_output.put_line('sqlcode : ' || sqlcode);
      dbms_output.put_line('sqlerrm : ' || sqlerrm);
    insert into logs (name, memo)
    values ('익명 PL/SQL', '알 수 없는 오류가 발생했습니다.');
end;
```

```sql
RAISE_APPLICATION_ERROR(-20055, '맥주는 안 됨');
```

이렇게 에러코드와 에러메시지를 넣어서 활용할 수 있다.

#### 맥주 판매 전 미성년자 유효성 체크하기

```sql
create or replace PACKAGE BODY PKG_ORDERS AS

  FUNCTION PF_GET_ADDPOINT (
      P_PRICE IN NUMBER
  ) RETURN NUMBER AS
  -- 적립 포인트
  v_point customer_info.point%type;
  BEGIN
      v_point := round(p_price * 0.1);
      RETURN v_point;
  Exception when others then
      return 0;
  END PF_GET_ADDPOINT;
  
  /* 미성년자는 음주 판매 불가 유효성 체크 */
  function pf_valid_beer (
      p_customer_id in varchar2 -- 고객 ID
    , p_menu_id in varchar2 -- 메뉴 ID  
  ) return varchar2
  AS
    r_return_message varchar2(10) := 'OK'; -- OK, NO
    v_birth customer_info.birth%type;
  begin
  
    -- 미성년자 여부 체크
    select birth
      into v_birth
    from customer_info
    where customer_id = p_customer_id
    ;
    
    -- 미성년자, 메뉴가 맥주인 것 체크
    if to_number(to_char(sysdate, 'yyyy'))-to_number(v_birth) < 19 -- 미성년자 이면서
      and p_menu_id in ('M004', 'M005') -- 맥주를 구매하면
      then
      r_return_message := 'NO';
    end if;

  return r_return_message;
  
  exception when others then
  return r_return_message;
  end pf_valid_beer;

  PROCEDURE PSP_MAKE_ORDER (
      P_CUSTOMER_ID IN VARCHAR2 
    , R_RETURN_CODE OUT VARCHAR2 
    , R_RETURN_MESSAGE OUT VARCHAR2 
  ) AS
    -- real_order 테이블 컬럼 값을 변수로 사용
    r_real_order real_order%rowtype;
    
    -- 옵션가 변수
    v_price_size menu.menu_price%type;
    v_price_ice menu_option.option_price%type;    
    BEGIN
        -- 실행구문
        -- 1. 장바구니의 자료를 가져온다.
        -- 2. 장바구니의 개수만큼 loop를 진행한다.
        -- 3. 주문서에 필요한 기본정보를 가져온다.
        for fc in (select * from temp_order where customer_id = nvl(p_customer_id, customer_id))
        loop
            -- 3-1. 맥주일 경우 미성년자인지 체크한다.
            dbms_output.put_line(fc.customer_id);
            if pkg_orders.pf_valid_beer(fc.customer_id, fc.menu_id) = 'NO' then
              RAISE_APPLICATION_ERROR(-20055, '미성년자에게 음주 판매는 불가합니다.');
            end if;
            
            -- 기준금액
            select x.menu_price
            into r_real_order.price
            from menu x
            where x.menu_id = fc.menu_id
            ;
        
            -- 4. 옵션가를 가져온다.
            -- 사이즈 옵션 가격
            v_price_size := f_get_option_price(fc.menu_id, fc.menu_size);
            -- 아이스 옵션 가격
            v_price_ice := f_get_option_price(fc.menu_id, fc.menu_ice);
        
            -- 5. 주문금액을 계산한다. (수량 * (기준단가 + 옵션가))
            -- 주문 단가
            r_real_order.price := r_real_order.price + v_price_size + v_price_ice;
            r_real_order.total_price := r_real_order.price * fc.quantity;
        
            -- 6. 포인트를 10% 추가한다.
            r_real_order.point_add := f_get_addpoint(r_real_order.total_price);
            -- 7. 주문서를 생성한다.
            insert into real_order (
                order_sequence
              , customer_id
              , menu_id
              , menu_size
              , menu_ice
              , quantity
              , price
              , total_price
              , point_use
              , point_add
            ) values (
                1
              , fc.customer_id
              , fc.menu_id
              , fc.menu_size
              , fc.menu_ice
              , fc.quantity
              , r_real_order.price
              , r_real_order.total_price
              , fc.point_use
              , r_real_order.point_add
            );
            -- 8. 개인정보에 포인트를 넣어준다.
            update customer_info
            set point = point + r_real_order.point_add
            where customer_id = fc.customer_id;
        end loop;
        r_return_code := to_char(sqlcode);
        r_return_message := sqlerrm;
        
      exception when others then
        r_return_code := sqlcode;
        r_return_message := sqlerrm;
    END PSP_MAKE_ORDER;
END PKG_ORDERS;
```

```sql
declare
  r_return_code varchar2(500);
  r_return_message varchar2(500);
begin
  pkg_orders.psp_make_order(null, r_return_code, r_return_message);

  dbms_output.put_line('r_return_code -> ' || r_return_code);
  dbms_output.put_line('r_return_message -> ' || r_return_message);
  
  if r_return_code = '0' then
    commit;
  else
    rollback;
  end if;
end;
```

```log
C001
C002
C003
r_return_code -> -20055
r_return_message -> ORA-20055: 미성년자에게 음주 판매는 불가합니다.
```
