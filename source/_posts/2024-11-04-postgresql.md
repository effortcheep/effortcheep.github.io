---
title: postgresql
date: 2024-11-04 20:57:24
tags:
- sql
- postgre
---


### 数据类型

#### Integres 只支持整数 速度快 有边界

```sql
DROP TABLE IF EXISTS Users;

CREATE TABLE Users (
  name text,
  age  smallint
);

-- int2 smallint
-- int4 integer
-- int8 bigint
```

#### Numeric 精准的数字 速度慢 无边界
```sql
CREATE TABLE numeric_example(
  description text,
  interest_rate numeric
);

select 12.345::numeric(5,3)

numeric(precision, scale) precision 整数位多少个，scale小数位多少个  会四舍五入
```

### Floating-point 小数 会丢失进度 速度快
```sql
CREATE TABLE real_example(
  sensor_name text,
  reading float8
);

-- real 4bytes 1E-37 - 1E37 6 digits
-- double precision 8bytes 1E-307 - 1E+308 15 digits

```


#### money 货比问题，精度问题 不建议使用 推荐使用 int * 100
```sql
CREATE TABLE money_example(
  item_name text,
  price money
);

--
INSERT INTO money_example (item_name, price) VALUES
('Laptop', 1999.99),
('Smartphone', 799.50),
('Pen', .25),
('Headphones', '$199.99'),
('Smartwatch', 249.75),
('Gaming Console', 299.95);
```


#### NaN not a number and Infiity 无穷大


#### text  建议使用 text  不要使用固定长度 char


#### check-constraints
```sql
create table check_example(
  price numeric CONSTRANINT price_must_be_positive CHECK (price > 0),   -- > 0
  discount_price numeric CHECK (discoount_price > 0),
  abbr text,
  CHECK (LENGTH(abbr) = 5),
  CHECK (price > discount_price)
);
```


#### Domain
```sql
CREATE DOMAIN us_postal_code as text constraint format check (
  VALUE ~ '^\d{5}$' OR VALUE ~ '^\d{5}-\d{4}$'
);

CREATE TABLE domain_example(
  street TEXT NOT NULL,
  city TEXT NOT NULL,
  postal us_postal_code
);
```