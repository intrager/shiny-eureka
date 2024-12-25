
# 테이블 생성

```sql
CREATE TABLE customer_info (
    customer_id VARCHAR2(20) not null, 
	NAME VARCHAR2(20) not null, 
	BIRTH NUMBER(8) not null, 
	MOBILE VARCHAR2(20) not null, 
	POINT NUMBER(10) default 0 not null, 
	register_day DATE default sysdate, 
	 CONSTRAINT PK_customer_INFO PRIMARY KEY (customer_id)
);

COMMENT ON COLUMN customer_info.customer_id IS '고객ID';
COMMENT ON COLUMN customer_info.NAME IS '고객명';
COMMENT ON COLUMN customer_info.BIRTH IS '출생년도';
COMMENT ON COLUMN customer_info.MOBILE IS '핸드폰번호';
COMMENT ON COLUMN customer_info.POINT IS '포인트적립';
COMMENT ON COLUMN customer_info.register_day IS '등록일자';
COMMENT ON TABLE customer_info  IS '고객정보 테이블';

-----

CREATE TABLE MENU (
    MENU_ID VARCHAR2(20) not null, 
	MENU_TYPE VARCHAR2(20) not null, 
	MENU_NAME VARCHAR2(20) not null, 
	MENU_PRICE NUMBER(10)  default 0 not null, 
	USE_YN CHAR(1) DEFAULT 'Y' not null, 
	register_day DATE DEFAULT sysdate not null, 
	CONSTRAINT PK_MENU PRIMARY KEY (MENU_ID)
);

COMMENT ON COLUMN MENU.MENU_ID IS '메뉴ID';
COMMENT ON COLUMN MENU.MENU_TYPE IS ' 메뉴 타입:커피/맥주';
COMMENT ON COLUMN MENU.MENU_NAME IS '메뉴 명칭';
COMMENT ON COLUMN MENU.MENU_PRICE IS '메뉴 가격';
COMMENT ON COLUMN MENU.USE_YN IS '사용유무';
COMMENT ON COLUMN MENU.register_day IS '등록일자';
COMMENT ON TABLE MENU IS '메뉴 테이블';


-- 메뉴 옵션 정보
CREATE TABLE MENU_OPTION (
    MENU_TYPE VARCHAR2(20) NOT NULL, 
	MENU_OPTION VARCHAR2(20) NOT NULL, 
	OPTION_PRICE NUMBER(10) NOT NULL, 
	register_day DATE DEFAULT sysdate, 
	 CONSTRAINT PK_MENU_OPT PRIMARY KEY (MENU_TYPE, MENU_OPTION)
);

   COMMENT ON COLUMN MENU_OPTION.MENU_TYPE IS '메뉴 타입:커피/맥주';
   COMMENT ON COLUMN MENU_OPTION.MENU_OPTION IS '옵션정보:Size(CC) , Ice';
   COMMENT ON COLUMN MENU_OPTION.OPTION_PRICE IS '옵션가격';
   COMMENT ON COLUMN MENU_OPTION.register_day IS '등록일자';
   COMMENT ON TABLE MENU_OPTION  IS '메뉴 옵션 테이블 : 기본 메뉴금액에 옵션가를 추가';


-- Cart (장바구니)
CREATE TABLE TEMP_ORDER (
    CUSTOMER_ID VARCHAR2(20) NOT NULL, 
	MENU_ID VARCHAR2(20) NOT NULL, 
	MENU_SIZE VARCHAR2(30) default 'N' NOT NULL, 
	MENU_ICE VARCHAR2(20) default 'N'  NOT NULL, 
	QUANTITY NUMBER(10) NOT NULL CONSTRAINT quantity_nozero CHECK(QUANTITY > 0), 
    POINT_USE NUMBER(10) default 0 not null, 
	register_day DATE DEFAULT sysdate, 
	 CONSTRAINT PK_TEMP_ORDER PRIMARY KEY (CUSTOMER_ID, MENU_ID)  
);

   COMMENT ON COLUMN TEMP_ORDER.CUSTOMER_ID IS '고객ID : cst_info.cst_id';
   COMMENT ON COLUMN TEMP_ORDER.MENU_ID IS '메뉴ID : menu.mnu_id';
   COMMENT ON COLUMN TEMP_ORDER.MENU_SIZE IS '메뉴 size :  menu_opt.mnu_opt';
   COMMENT ON COLUMN TEMP_ORDER.MENU_ICE IS '메뉴 Ice : menu_opt.mnu_opt';
   COMMENT ON COLUMN TEMP_ORDER.QUANTITY IS '주문수량 , 0 보다 커야 함';
   COMMENT ON COLUMN TEMP_ORDER.register_day IS '등록일자';
   COMMENT ON TABLE TEMP_ORDER  IS 'Cart (장바구니) 테이블';

----
CREATE SEQUENCE sequence_real_order
       INCREMENT BY 1
       START WITH 1000000
       MINVALUE 1
       MAXVALUE 9999999
       NOCYCLE
       CACHE 20
       NOORDER;


----
CREATE TABLE real_order (
    ORDER_NO NUMBER(10) DEFAULT NOT NULL, -- 11g 기준 
	ORDER_SEQUENCE NUMBER(5) NOT NULL, 
	CUSTOMER_ID VARCHAR2(20), 
	MENU_ID VARCHAR2(20), 
	MENU_SIZE VARCHAR2(20) DEFAULT 'N' NOT NULL , 
	MENU_ICE VARCHAR2(20) DEFAULT 'N' NOT NULL , 
	QUANTITY NUMBER(10) NOT NULL ENABLE, 
	PRICE NUMBER(10), 
	TOTAL_PRICE NUMBER(10), 
    POINT_USE NUMBER(10,0) DEFAULT 0 NOT NULL , 
    POINT_ADD NUMBER(10,0) DEFAULT 0 NOT NULL ,
	REGISTER_DAY DATE DEFAULT sysdate, 
	 CONSTRAINT PK_REAL_ORD PRIMARY KEY (ORDER_NO, ORDER_SEQUENCE)
);

   COMMENT ON COLUMN real_order.ORDER_SEQUENCE IS '주문순차';
   COMMENT ON COLUMN real_order.CUSTOMER_ID IS '고객ID';
   COMMENT ON COLUMN real_order.MENU_ID IS '메뉴ID';
   COMMENT ON COLUMN real_order.MENU_SIZE IS '메뉴 size :  menu_opt.mnu_opt';
   COMMENT ON COLUMN real_order.MENU_ICE IS '메뉴 Ice : menu_opt.mnu_opt';
   COMMENT ON COLUMN real_order.QUANTITY IS '주문수량 , 0 보다 커야 함';
   COMMENT ON COLUMN real_order.PRICE IS '단가';
   COMMENT ON COLUMN real_order.TOTAL_PRICE IS '주문금액 : 수량 * 단가';
   COMMENT ON COLUMN real_order.POINT_USE IS '포인트 : 사용포인트';
   COMMENT ON COLUMN real_order.POINT_ADD IS '포인트 : 추가포인트';
   COMMENT ON COLUMN real_order.REGISTER_DAY IS '등록일자';
```


# 데이터 입력

```sql
-- 고객정보 - customer_info

Insert into customer_info (customer_id,NAME,BIRTH,MOBILE,POINT,REGister_DAY) 
values ('C001','홍길동1',20080304,'010-0000-1111',100,sysdate);

Insert into customer_info (customer_id,NAME,BIRTH,MOBILE,POINT,REGister_DAY)
values ('C002','홍길동2',20000304,'010-0000-1112',0,sysdate);

Insert into customer_info (customer_id,NAME,BIRTH,MOBILE,POINT,REGister_DAY)
values ('C003','홍길동3',19950304,'010-0000-1113',5000,sysdate);

Insert into customer_info (customer_id,NAME,BIRTH,MOBILE,POINT,REGister_DAY)
values ('C004','홍길동4',19700304,'010-0000-1114',0,sysdate);

Insert into customer_info (customer_id,NAME,BIRTH,MOBILE,POINT,REGister_DAY)
values ('C005','홍길동5',19600304,'010-0000-1115',0,sysdate);


-- 메뉴 - menu
insert into menu(menu_id, menu_type, menu_name, menu_price, use_yn, REGister_DAY)
values ('M001','커피','아메리카노',3000,'Y',sysdate);

insert into menu(menu_id, menu_type, menu_name, menu_price, use_yn, REGister_DAY)
values ('M002','커피','카페라떼',4000,'Y',sysdate);

insert into menu(menu_id, menu_type, menu_name, menu_price, use_yn, REGister_DAY)
values ('M003','커피','카푸치노',4000,'Y',sysdate);

insert into menu(menu_id, menu_type, menu_name, menu_price, use_yn, REGister_DAY)
values ('M004','맥주','생맥주',5000,'Y',sysdate);

insert into menu(menu_id, menu_type, menu_name, menu_price, use_yn, REGister_DAY)
values ('M005','맥주','흑맥주',6000,'Y',sysdate);

-- 메뉴 옵션 - menu_option
insert into menu_option(menu_type, menu_option,  option_price, register_day)
values ('커피','M',500,sysdate);

insert into menu_option(menu_type, menu_option,  option_price, register_day)
values ('커피','L',1000,sysdate);

insert into menu_option(menu_type, menu_option,  option_price, register_day)
values ('커피','ICE',500,sysdate);

insert into menu_option(menu_type, menu_option,  option_price, register_day)
values ('맥주','1000', 5000,sysdate);

insert into menu_option(menu_type, menu_option,  option_price, register_day)
values ('맥주','3000', 10000,sysdate);

insert into menu_option(menu_type, menu_option,  option_price, register_day)
values ('맥주','5000', 20000,sysdate);

-- Cart(장바구니)
insert into temp_order (customer_id, menu_id, menu_size, menu_ice, quantity, point_use, register_day)
values ('C001','M001','M','N',2,0,sysdate);

insert into temp_order (customer_id, menu_id, menu_size, menu_ice, quantity, point_use, register_day)
values ('C002','M002','L','ICE',3,0,sysdate);

insert into temp_order (customer_id, menu_id, menu_size, menu_ice, quantity, point_use, register_day)
values ('C003','M004','3000','N',1,3000,sysdate);
```


# 왜래키, 조인 쿼리 실습

```sql
-- 외래키 생성
alter table temp_order
add constraint fk_temp_order_01 foreign key(customer_id) references customer_info(customer_id);

alter table temp_order
add constraint fk_temp_order_02 foreign key(menu_id) references menu(menu_id);

alter table real_order
add constraint fk_real_order_01 foreign key(customer_id) references customer_info(customer_id);

alter table real_order
add constraint fk_real_order_02 foreign key(menu_id) references menu(menu_id);

-- 외래키 삭제
alter table temp_order drop constraint fk_temp_order_01;
alter table temp_order drop constraint fk_temp_order_02;
alter table real_order drop constraint fk_real_order_01;
alter table real_order drop constraint fk_real_order_02;

-- 장바구니 쿼리
select temp_order.*, customer_info.name, customer_info.birth, menu.menu_name, menu.menu_price
from temp_order
inner join customer_info
        on temp_order.customer_id = customer_info.customer_id
inner join menu
        on temp_order.menu_id = menu.menu_id;
        
-- 주문서 쿼리
select real_order.order_no, real_order.order_sequence, customer_info.name, customer_info.birth, menu.menu_name, real_order.price, real_order.quantity, real_order.total_price
from real_order
inner join customer_info
        on real_order.customer_id = customer_info.customer_id
inner join menu
        on real_order.menu_id = menu.menu_id;
```

