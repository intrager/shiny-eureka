# 일반 변수 / ROWTYPE

> 학습목표
>
> Procedure or Function에서 일반변수 / ROWTYPE을 파라미터 / 리턴 값으로 사용할 수 있다.


> 실습
>
> 1. 일반변수를 파라미터와 리턴 값으로 사용한다.
> 2. ROWTYPE 변수를 파라미터와 리턴 값으로 사용한다.
> 3. 위 둘을 Function & Procedure에서 모두 사용해본다.
> 4. Function <-> Procedure를 변환해본다.

## SAMPLE 일반변수 & ROWTYPE 사용

### SP_SECTION_SIX_ONE 프로시저

```sql
CREATE OR REPLACE PROCEDURE SP_SECTION_SIX_ONE AS 
    -- 일반 변수
    p_customer_id customer_info.customer_id%type;
    v_name customer_info.name%type;

    -- rowtype 변수
    p_customer_info customer_info%rowtype;
    r_customer_info customer_info%rowtype;

begin
    p_customer_id := 'C003';

    -- 일반변수 이용
    select name
      into v_name
    from customer_info
    where customer_id = p_customer_id
    ;

    dbms_output.put_line('v_name > ' || v_name);

    -- rowtype 이용
    select *
      into r_customer_info
    from customer_info
    where customer_id = p_customer_id
    ;

    dbms_output.put_line('r_customer_info > ' || r_customer_info.name);
END SP_SECTION_SIX_ONE;
```

파라미터가 없는 프로시저는 바로 사용할 수 있다.

```sql
begin
  sp_section_six_one;
end;
```

### 팡숀 사용 f_get_name1

위 프로시저에 팡숀(`f_get_name1`) 하나 넣어서 사용해보겠다.

이 팡숀은 고객의 아이디를 파라미터로 받아 그의 이름을 조회한 후 반환하는 팡숀이다.

```sql
create or replace function f_get_name1 (
  p_customer_id in customer_info.customer_id%type
) return varchar2
as
  v_name customer_info.name%type;
begin
  select name
    into v_name
  from customer_info
  where customer_id = p_customer_id;
  
  return v_name;
end f_get_name1;
```

### 팡숀 사용 f_get_name2

이번에는 테이블의 컬럼을 직접 참조한 타입이 아닌 rowtype 변수를 만들어볼 것이다.

```sql
CREATE OR REPLACE PROCEDURE SP_SECTION_SIX_ONE AS 
    -- 일반 변수
    p_customer_id customer_info.customer_id%type;
    v_name customer_info.name%type;

    -- rowtype 변수
    p_customer_info customer_info%rowtype;
    r_customer_info customer_info%rowtype;

begin
    p_customer_id := 'C003';

    -- 일반변수 이용
    v_name := f_get_name1(p_customer_id);

    dbms_output.put_line('v_name > ' || v_name);

    -- rowtype 이용
    r_customer_info := f_get_name2(p_customer_info);

    dbms_output.put_line('r_customer_info > ' || r_customer_info.name);
END SP_SECTION_SIX_ONE;
```

프로시저는 이렇게 바꿀 수 있고

```sql
begin
  sp_section_six_one;
end;
```

이렇게 실행하면

```log
오류 보고 -
ORA-01403: no data found
ORA-06512: at "INFLEARN.F_GET_NAME2", line 7
ORA-06512: at "INFLEARN.SP_SECTION_SIX_ONE", line 19
ORA-06512: at line 2
01403. 00000 -  "no data found"
*Cause:    No data was found from the objects.
*Action:   There was no data from the objects which may be due to end of fetch.
```

파라미터로 넣은 p_customer_id 값을 `C003`으로 했는데 해당하는 적당한 값이 없는 듯하다.

p_customer_id를 `C002`로 바꾸고 p_customer_info의 customer_id에다가 `C004`값을 넣고 다시 실행해본다.

```sql
create or replace PROCEDURE SP_SECTION_SIX_ONE AS 
    -- 일반 변수
    p_customer_id customer_info.customer_id%type;
    v_name customer_info.name%type;

    -- rowtype 변수
    p_customer_info customer_info%rowtype;
    r_customer_info customer_info%rowtype;

begin
    p_customer_id := 'C002';

    -- 일반변수 이용
    v_name := f_get_name1(p_customer_id);

    dbms_output.put_line('v_name > ' || v_name);

    -- rowtype 이용
    p_customer_info.customer_id := 'C004';
    r_customer_info := f_get_name2(p_customer_info);

    dbms_output.put_line('r_customer_info > ' || r_customer_info.name);
END SP_SECTION_SIX_ONE;
```

추가한 함수는 아래와 같다.

```sql
create or replace FUNCTION f_get_name2 (
  p_customer_info in customer_info%rowtype
) return customer_info%rowtype
as
r_customer_info customer_info%rowtype;
begin
  select *
    into r_customer_info
  from customer_info
  where customer_id = p_customer_info.customer_id
  ;
  return r_customer_info;
end f_get_name2;
```

### 프로시저 사용 sp_get_name1

```sql
create or replace procedure sp_get_name1 (
    p_customer_id in customer_info.customer_id%type
  , v_name out customer_info.name%type
)
as

begin
  select name
    into v_name
  from customer_info
  where customer_id = p_customer_id;
end sp_get_name1;
```

```sql
create or replace PROCEDURE SP_SECTION_SIX_ONE AS 
    -- 일반 변수
    p_customer_id customer_info.customer_id%type;
    v_name customer_info.name%type;

    -- rowtype 변수
    p_customer_info customer_info%rowtype;
    r_customer_info customer_info%rowtype;

begin
    p_customer_id := 'C002';

    -- 일반변수 이용
    v_name := f_get_name1(p_customer_id);

    dbms_output.put_line('v_name > ' || v_name);

    -- rowtype 이용
    p_customer_info.customer_id := 'C004';
    r_customer_info := f_get_name2(p_customer_info);

    dbms_output.put_line('r_customer_info > ' || r_customer_info.name);
END SP_SECTION_SIX_ONE;
```

### 패키지로 합치기

만들었던 팡숀과 프로시저들은 바디 안에 감추고 `PSP_SECTION_SIX_ONE`만으로 호출할 수 있도록 만들 것이다.

#### Specification

```sql
create or replace PACKAGE PKG_SECTION_SIX
AS
    PROCEDURE PSP_SECTION_SIX_ONE;
END PKG_SECTION_SIX;
```

#### Body

```sql
CREATE OR REPLACE
PACKAGE BODY PKG_SECTION_SIX AS
    -- functions
    function pf_get_name1 (
      p_customer_id in customer_info.customer_id%type
    ) return varchar2
    as
      v_name customer_info.name%type;
    begin
      select name
        into v_name
      from customer_info
      where customer_id = p_customer_id;
      
      return v_name;
    end pf_get_name1;

    function pf_get_name2 (
      p_customer_info in customer_info%rowtype
    ) return customer_info%rowtype
    as
    r_customer_info customer_info%rowtype;
    begin
      select *
        into r_customer_info
      from customer_info
      where customer_id = p_customer_info.customer_id
      ;
      return r_customer_info;
    end pf_get_name2;

    -- procedures
    procedure psp_get_name1 (
        p_customer_id in customer_info.customer_id%type
      , v_name out customer_info.name%type
    )
    as
    
    begin
      select name
        into v_name
      from customer_info
      where customer_id = p_customer_id;
    end psp_get_name1;

    procedure psp_get_name2 (
        p_customer_info in customer_info%rowtype
      , r_customer_info out customer_info%rowtype
    )
    as
    
    begin
      select *
        into r_customer_info
      from customer_info
      where customer_id = p_customer_info.customer_id
      ;
    end psp_get_name2;

    PROCEDURE PSP_SECTION_SIX_ONE AS 
    -- 일반 변수
    p_customer_id customer_info.customer_id%type;
    v_name customer_info.name%type;

    -- rowtype 변수
    p_customer_info customer_info%rowtype;
    r_customer_info customer_info%rowtype;
    begin
        -- 팡숀 구문
        p_customer_id := 'C005';
        -- 일반변수 이용
        v_name := PKG_SECTION_SIX.pf_get_name1(p_customer_id);
        dbms_output.put_line('v_name > ' || v_name);
    
        -- rowtype 이용
        p_customer_info.customer_id := 'C005';
        r_customer_info := PKG_SECTION_SIX.pf_get_name2(p_customer_info);
        dbms_output.put_line('r_customer_info.v_name > ' || r_customer_info.name);
        
        -- 프로시저 구문
        p_customer_id := 'C001';
        -- 일반변수 이용
        PKG_SECTION_SIX.psp_get_name1(p_customer_id, v_name);
        dbms_output.put_line('sp - v_name > ' || v_name);
    
        -- rowtype 이용
        p_customer_info.customer_id := 'C001';
        PKG_SECTION_SIX.psp_get_name2(p_customer_info, r_customer_info);
        dbms_output.put_line('sp - r_customer_info.v_name > ' || r_customer_info.name);
    END PSP_SECTION_SIX_ONE;
END PKG_SECTION_SIX;
```

# RECORD 개념 / 활용


- 단일필드 변수
  - v_name varchar2(20)
  - v_name customer_info.name%type;
- 전체필드 (ROWTYPE record)
  - r_customer_info customer_info%type
- Type Record (사용자 정의)
  - 
    ```sql
      Type recordName IS Record (
          customer_id varchar2(20)
        , name customer_info.name%type;
      )
    ```

RECORD는 패키지 안에 전역처럼 사용하도록 맨 위에 둘 수도 있고, 필요한 프로시저/팡숀 바로 위에 두고 쓸 수도 있다.


예제는 다음과 같다.
```sql
declare
  type DepartmentRecordType IS RECORD (
    dept_id number(4) not null := 10,
    dept_name varchar2(30) not null := 'Administration',
    mgr_id number(6) := 200,
    loc_id number(4) := 1700
  );
  department_record DepartmentRecordType;
begin
    dbms_output.put_line('dept_id : ' || department_record.dept_id);
    dbms_output.put_line('dept_name : ' || department_record.dept_name);
    dbms_output.put_line('mgr_id : ' || department_record.mgr_id);
    dbms_output.put_line('loc_id : ' || department_record.loc_id);
end;
```

```sql
CREATE OR REPLACE
PACKAGE BODY PKG_SECTION_SIX AS

    /* 패키지 내부 전역으로도 사용 가능한 RECORD */
    TYPE CustomerInfoRecordType IS RECORD (   
        v_name customer_info.name%type
      , v_birth customer_info.birth%type
      , v_mobile customer_info.mobile%type
      , v_favorite varchar2(100)
    );
    ...
```

프로시저 내에 둔, 즉 다른 위치에 둔 RECORD를 활용한 전체 소스는 다음과 같다.

```sql
CREATE OR REPLACE
PACKAGE BODY PKG_SECTION_SIX AS

    -- section 6-1
    -- functions
    function pf_get_name1 (
      p_customer_id in customer_info.customer_id%type
    ) return varchar2
    as
      v_name customer_info.name%type;
    begin
      select name
        into v_name
      from customer_info
      where customer_id = p_customer_id;
      
      return v_name;
    end pf_get_name1;

    function pf_get_name2 (
      p_customer_info in customer_info%rowtype
    ) return customer_info%rowtype
    as
    r_customer_info customer_info%rowtype;
    begin
      select *
        into r_customer_info
      from customer_info
      where customer_id = p_customer_info.customer_id
      ;
      return r_customer_info;
    end pf_get_name2;

    -- procedures
    procedure psp_get_name1 (
        p_customer_id in customer_info.customer_id%type
      , v_name out customer_info.name%type
    )
    as
    
    begin
      select name
        into v_name
      from customer_info
      where customer_id = p_customer_id;
    end psp_get_name1;

    procedure psp_get_name2 (
        p_customer_info in customer_info%rowtype
      , r_customer_info out customer_info%rowtype
    )
    as
    
    begin
      select *
        into r_customer_info
      from customer_info
      where customer_id = p_customer_info.customer_id
      ;
    end psp_get_name2;

    PROCEDURE PSP_SECTION_SIX_ONE AS 
    -- 일반 변수
    p_customer_id customer_info.customer_id%type;
    v_name customer_info.name%type;

    -- rowtype 변수
    p_customer_info customer_info%rowtype;
    r_customer_info customer_info%rowtype;
    begin
        -- 팡숀 구문
        p_customer_id := 'C005';
        -- 일반변수 이용
        v_name := PKG_SECTION_SIX.pf_get_name1(p_customer_id);
        dbms_output.put_line('v_name > ' || v_name);
    
        -- rowtype 이용
        p_customer_info.customer_id := 'C005';
        r_customer_info := PKG_SECTION_SIX.pf_get_name2(p_customer_info);
        dbms_output.put_line('r_customer_info.v_name > ' || r_customer_info.name);
        
        -- 프로시저 구문
        p_customer_id := 'C001';
        -- 일반변수 이용
        PKG_SECTION_SIX.psp_get_name1(p_customer_id, v_name);
        dbms_output.put_line('sp - v_name > ' || v_name);
    
        -- rowtype 이용
        p_customer_info.customer_id := 'C001';
        PKG_SECTION_SIX.psp_get_name2(p_customer_info, r_customer_info);
        dbms_output.put_line('sp - r_customer_info.v_name > ' || r_customer_info.name);
    END PSP_SECTION_SIX_ONE;
    
    -- section 6-3
    
    -- functions
    function pf_get_record (
      p_customer_id in customer_info.customer_id%type
    ) return varchar2
    as
      v_name customer_info.name%type;
    begin
      select name
        into v_name
      from customer_info
      where customer_id = p_customer_id;
      
      return v_name;
    end pf_get_record;
    
    procedure psp_get_record (
        p_customer_id in customer_info.customer_id%type
      , v_name out customer_info.name%type
      , v_birth out customer_info.birth%type
      , v_mobile out customer_info.mobile%type
      , v_favorite out varchar2
    )
    as
      
    begin
      select name, birth, mobile, '커피' as favorite
        into v_name, v_birth, v_mobile, v_favorite
      from customer_info
      where customer_id = p_customer_id;
    end psp_get_record;
    
    
    PROCEDURE PSP_SECTION_SIX_THREE AS 
        -- 일반 변수
        p_customer_id customer_info.customer_id%type;
        
        TYPE CustomerInfoRecordType IS RECORD (   
            v_name customer_info.name%type
          , v_birth customer_info.birth%type
          , v_mobile customer_info.mobile%type
          , v_favorite varchar2(100)
        );
        -- 레코드 타입 변수
        customer_info_record CustomerInfoRecordType;
    begin
        p_customer_id := 'C004';
        
        -- 일반변수 이용
        pkg_section_six.psp_get_record(p_customer_id, customer_info_record.v_name, customer_info_record.v_birth, customer_info_record.v_mobile, customer_info_record.v_favorite);
        dbms_output.put_line('v_name : ' || customer_info_record.v_name);
        dbms_output.put_line('v_birth : ' || customer_info_record.v_birth);
        dbms_output.put_line('v_mobile : ' || customer_info_record.v_mobile);
        dbms_output.put_line('v_favorite : ' || customer_info_record.v_favorite);
    END PSP_SECTION_SIX_THREE;
END PKG_SECTION_SIX;
```

RECORD를 활용한 프로시저는 아래와 같이 코드의 중복을 제거할 수 있어 보기 편하게 수정할 수 있다.

```sql
procedure psp_get_record (
    p_customer_id in customer_info.customer_id%type
  , record_customer_info out CustomerInfoRecordType
)
...
```

```sql
PROCEDURE PSP_SECTION_SIX_THREE AS 
    -- 일반 변수
    p_customer_id customer_info.customer_id%type;
    
    -- 레코드 타입 변수
    customer_info_record CustomerInfoRecordType;
begin
    p_customer_id := 'C005';
    
    -- 일반변수 이용
    pkg_section_six.psp_get_record(p_customer_id, customer_info_record);
    dbms_output.put_line('v_name : ' || customer_info_record.v_name);
    dbms_output.put_line('v_birth : ' || customer_info_record.v_birth);
    dbms_output.put_line('v_mobile : ' || customer_info_record.v_mobile);
    dbms_output.put_line('v_favorite : ' || customer_info_record.v_favorite);
END PSP_SECTION_SIX_THREE;
```

팡숀도 프로시저에서 활용한 것처럼 비슷하게 이용할 수 있다.
팡숀은 리턴 타입이 한 개이지만, "RECORD 타입"도 한 개이므로 Java에서의 DTO 타입 객체 반환하듯이 RECORD 타입 자체를 반환할 수 있다.

```sql
function pf_get_record (
    p_customer_id in customer_info.customer_id%type
) return CustomerInfoRecordType
as
    -- 레코드 타입 변수
    customer_info_record CustomerInfoRecordType;
begin
    select name, birth, mobile, '커피' as favorite
      into customer_info_record
    -- into customer_info_record.v_name, customer_info_record.v_birth, customer_info_record.v_mobile, customer_info_record.v_favorite
    from customer_info
    where customer_id = p_customer_id
    ;
    
    return customer_info_record;
end pf_get_record;

...

PROCEDURE PSP_SECTION_SIX_THREE AS 
    -- 일반 변수
    p_customer_id customer_info.customer_id%type;
    
    -- 레코드 타입 변수
    customer_info_record CustomerInfoRecordType;
begin
    p_customer_id := 'C001';
    
    -- 일반변수 이용
    customer_info_record := pkg_section_six.pf_get_record(p_customer_id);
    dbms_output.put_line('v_name : ' || customer_info_record.v_name);
    dbms_output.put_line('v_birth : ' || customer_info_record.v_birth);
    dbms_output.put_line('v_mobile : ' || customer_info_record.v_mobile);
    dbms_output.put_line('v_favorite : ' || customer_info_record.v_favorite);
END PSP_SECTION_SIX_THREE;
```

RECORD는 rowtype에 비해 테이블에 한정되지 않고 여러가지 사용자 정의로 활용할 수 있다.

RECORD는 그 안에 있는 변수들을 선언해서 매칭할 수도 있지만, 선언하지 않고 RECORD이름, 변수 타입만 넣어도 매칭할 수 있다.

또한, RECORD는 Specification 부분에도 넣어서 전역으로 활용할 수 있다.

```sql
create or replace PACKAGE PKG_SECTION_SIX
AS
    TYPE CustomerInfoRecordType IS RECORD (   
        v_name customer_info.name%type
      , v_birth customer_info.birth%type
      , v_mobile customer_info.mobile%type
      , v_favorite varchar2(100)
    );
    
    PROCEDURE PSP_SECTION_SIX_ONE;
    
    PROCEDURE PSP_SECTION_SIX_THREE;
END PKG_SECTION_SIX;
```

RECORD는 다른 패키지에 있어도 Specification에 있다면 가져와서 사용할 수 있다.