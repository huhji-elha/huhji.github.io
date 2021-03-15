---
layout: post
title:  "MYSQL basic code with Django"
date:   2020-01-16 17:02:11 +0530
categories: MYSQL MariaDB Django
---

🐢 Mysql 기본 구문과 Django에서 사용하기

_____________________________________


# MYSQL basic code

### Structured Query Language

관계형 데이터베이스에서 데이터를 읽거나 생성 및 수정하기 위해 사용하는 언어입니다. 
기본적으로 **CRUD**라고 하여 데이터를 Create, Read, Update, Delete 하는 기능을 제공하는 관계형 데이터베이스 시스템 전용언어이다.

다음 내용은 관계형 데이터베이스에서 데이터를 처리하는 기본적인 구문입니다.

### SELECT

데이터를 읽어 들일 때 사용하는 구문.

```sql
# 기본 사용
SELECT
    column1,
    column2,
    column3,
    column4
FROM table_name

# WHERE로 조건을 넣어서 이렇게 불러올 수도 있다.
SELECT
    id,
    name,
    age,
    gender
FROM users
WHERE name = "huhji"
```

### INSERT

데이터를 생성할 때 사용하는 구문. 

```sql
# 기본 사용
INSERT INTO table_name (
    column1,
    column2,
    column3,
) VALUES (
    column1_value,
    column2_value,
    column3_value
)

# 만약 users 테이블에 다음과 같은 값을 생성해야 한다면
{
    "id" : 1,
    "name" : "huhji",
    "age" : "100",
    "gender" : None
}

# 이렇게 작성할 수 있다.
INSERT INFO users (
    id,
    name,
    age,
    gender
) VALUES (
    1,
    "huhji",
    "100",
    None
)
# 두개 이상을 추가하고 싶다면 VALUE(..), (..) 이렇게 추가하면 된다.
```

### UPDATE

기존의 데이터를 수정할 때 사용한다.

```sql
# 기본 구문
UPDATE table_name SET column1 = value1 WHERE column2 = value2

# 나이를 25로 수정해야 한다면
UPDATE users SET age = 25 WHERE name = "huhji"
```

### DELETE

데이터를 삭제할 때 사용한다.

```sql
# 기본 구문
DELETE FROM table_name WHERE column1 = value1

# 나이가 20세 이하인 사용자들을 삭제해야 한다면
DELETE FROM users WHERE age < 20
```

### JOIN

여러 테이블을 연결할 때 사용한다. 관계형 데이터베이스에서는 원하는 정보를 전부 얻기 위해 하나 이상의 테이블에서 값을 읽어 들여야 할 필요가 있다. 
관계형 데이터베이스의 특성인 테이블의 고유한 외부 아이디를 이용한다.

```sql
# 기본 구문
SELECT
    table1.column1,
    table2.column2
FROM table1
JOIN table2 ON table1.id = table2.table1_id

# 사용자의 이름을 users 테이블에서 읽어들이고, 해당 사용자의 주소를 user_address에서 읽어오자
SELECT
    users.name,
    user_address.address
FROM users
JOIN user_address ON users.id = user_address.user_id
```

### start mysql

```bash
# start
$ service mysql start

# status
$ service mysql status

# stop
$ service mysql stop

# start mysql
$ sudo mysql

# start mysql as root admin
$ sudo -u root -p
```

### > mysql

```sql
# check user info.
mysql> SELECT user, plugin, host FROM mysql.user;
```

![](https://user-images.githubusercontent.com/59910975/111102648-f3152980-858f-11eb-9c0b-03e7a7cac8d2.png)

```sql
# create database
mysql> CREATE DATABASE toy_service;
# Query OK, 1 row affected (0.00 sec)

# use database
mysql> USE toy_service;
# Database changed

# create table
mysql> CREATE TABLE users(
    -> id INT NOT NULL AUTO_INCREMENT, 
    -> name VARCHAR(255) NOT NULL, 
    -> email VARCHAR(255) NOT NULL,
    -> hashed_password VARCHAR(255) NOT NULL,
    -> profile VARCHAR(2000) NOT NULL,
    -> created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP, 
    -> updated_at TIMESTAMP NULL DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP,
    -> PRIMARY KEY (id),
    -> UNIQUE KEY email (email)
    -> );
# Query OK, 0 rows affected (0.06 sec)
mysql> CREATE TABLE users_follow_list(
    -> user_id INT NOT NULL,
    -> follow_user_id INT NOT NULL,
    -> created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    -> PRIMARY KEY (user_id, follow_user_id),
    -> CONSTRAINT users_follow_list_follow_user_id_fkey FOREIGN KEY 
        (follow_user_id) REFERENCES users(id)
    -> );
# Query OK, 0 rows affected (0.06 sec)

mysql> CREATE TABLE tweets(
    -> id INT NOT NULL AUTO_INCREMENT,
    -> user_id INT NOT NULL,
    -> tweet VARCHAR(300) NOT NULL,
    -> created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    -> PRIMARY KEY (id),
    -> CONSTRAINT tweets_user_id_fkey FOREIGN KEY (user_id) REFERENCES
    -> users(id)
    -> );
# Query OK, 0 rows affected (0.06 sec)
```

### 기본 명령어 참고

- `NOT NULL` : 해당 칼럼은 NULL값이 될 수 없다.
- `AUTO_INCREMENT` : 해당 칼럼의 값이 자동으로 1씩 증가한다. 주로 id 값을 생성해주기 위해 사용한다.
- `CURRENT_TIMESTAMP` : 해당 칼럼의 값이 없으면 디폴트 값으로 현재 시간 값을 사용한다.
주로 created_at 처럼 해당 값이 생성된 시점을 기록하는 칼럼에 사용된다.
- `ON UPDATE CURRENT TIMESTAMP` : 만일 해당 row의 값이 업데이트되면 해당 column의 값을 수정이 이루어진 시간의 값으로 자동 생성해 준다는 뜻이다.
row가 언제 업데이트되었는지 자동으로 기록되므로 편리하다.
- `PRIMARY KEY` : 고유 키로 사용될 column을 정해준다. 
고유 키는 한 개의 column으로 정할 수도, 여러 개의 column을 정할 수도 있다. 여러 column을 고유 키로 정해주면 해당 column 값들을 합한 값이 고유 키가 된다.
- `UNIQUE KEY` : 해당 column의 값에 중복되는 row가 없어야 한다는 뜻.
이메일의 경우 이미 등록된 이메일로 중복 등록이 안되도록 방지 할 때 사용한다.
- `CONSTRAINT ... FOREIGN KEY ... REFERENCES ...` : 구문을 통해 외부 키를 걸 수 있다.
users_follow_list 테이블과 tweets 테이블 둘 다 users 테이블에 외부 키를 통해 연결된다.

```sql
# created table check
mysql> EXPLAIN table_name;
```

![](https://user-images.githubusercontent.com/59910975/111102650-f4465680-858f-11eb-9190-de4819d0e23d.png)

`MUL`은 내가 foriegn key로 설정한 id를 표현하는 방식이다. 다른 테이블의 기본 키를 참조하는 외래키는 이렇게 `MUL(Multiple)`이라고 표시된다.

### 저장된 데이터 확인

```sql
SELECT * from Tablename
```

![](https://user-images.githubusercontent.com/59910975/111102652-f4deed00-858f-11eb-8c4a-c0598a889670.png)

![](https://user-images.githubusercontent.com/59910975/111102653-f4deed00-858f-11eb-91cb-064812c9c428.png)


## Django에서 SQL 작성하기

```python
# models.py
# Question Table
class Question(models.Model):
    question_text = models.CharField(max_length=200)
    create_date = models.DateTimeField(blank=True, null=True)

    class Meta:
        managed = False
        db_table = 'Question'

    # 객체를 문자열로 표현할 때 사용하는 함수
    # 테이블에 저장된 데이터를 question_text로 보여준다.
    def __str__(self) :
        return self.question_text

# Choice Table
class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0, blank=True, null=True)

    class Meta:
        managed = False
        db_table = 'Choice'
    def __str__(self) :
        return self.choice_text
```

```sql
## TB_DATASET
mysql > CREATE TABLE TB_DATASET( 
        dataset_id INT(11) NOT NULL AUTO_INCREMENT, 
        filename VARCHAR(100) NOT NULL,
        file_addr VARCHAR(255) NOT NULL, 
        file_size FLOAT(11) NOT NULL, 
        user_id VARCHAR(20) NOT NULL, 
        create_date TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP, 
        PRIMARY KEY (dataset_id));

## TB_MODELS
mysql > CREATE TABLE TB_MODELS( 
        models_id INT(11) NOT NULL AUTO_INCREMENT, 
        model_name VARCHAR(100) NOT NULL, 
        model_addr VARCHAR(255) NOT NULL, 
        model_size FLOAT(11) NOT NULL, 
        train_data_name VARCHAR(255) NOT NULL, 
        train_data_type VARCHAR(20) NOT NULL, 
        docker_image VARCHAR(255) NOT NULL, 
        user_id VARCHAR(20) NOT NULL, 
        create_date TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP, 
        PRIMARY KEY (models_id));
```


## Django에 기존 SQL 연결하기 (MariaDB 사용, Mysql과 동일)

- settings.py에 DB 정보 입력

```python
# .env
MARIA_DB_ENGINE=django.db.backends.mysql
MARIA_DB_NAME=toy_service
MARIA_DB_USER=huhji
MARIA_DB_PASSWORD=1234
MARIA_DB_HOST=123.456.78.9
MARIA_DB_PORT=3306
```

```python
# settings.py
# python environ 모듈 사용
from pathlib import Path

import environ
env = environ.Env(
    DEBUG=(bool, False)
)
environ.Env.read_env()

DATABASES = {
    "default": {
        "ENGINE": env("MARIA_DB_ENGINE"),
        "NAME": env("MARIA_DB_NAME"),
        "USER": env("MARIA_DB_USER"),
        "PASSWORD": env("MARIA_DB_PASSWORD"),
        "HOST": env("MARIA_DB_HOST"),
        "PORT": env("MARIA_DB_PORT"),
        "OPTIONS": {"init_command": "SET sql_mode='STRICT_TRANS_TABLES'"},
    },
}
```

Mysql은 유효하지 않은 값을 넣더라도 가장 근접한 값으로 변환하여 입력되고,
값이 누락되더라도 implicit default 값을 넣어 에러가 아닌 경고 메세지를 반환한다는 특성이 있다.

STRICT_TRANS_TABLES mode는 명령문 안에 데이터가 유효하지 않거나 누락이 되면 에러를 발생시키는 옵션이다. 이는 코드 단계가 아닌 DBMS 단계에서 데이터 무결성을 체크할 수 있다.

- 다음 명령 실행 (settings.py에 입력한 테이블이 이미 생성되어 있어야함)

```bash
# 입력한 데이터베이스를 검사하여 모델 생성
python manage.py inspectdb > models.py

# Django 테이블 설치
python manage.py migrate
```

### Reference.

Django document: [https://docs.djangoproject.com/ko/3.1/](https://docs.djangoproject.com/ko/3.1/)

Mysql manual: [http://www.mysqlkorea.com/sub.html?mcode=manual&scode=01_1&lang=k&ver_name=5.1](http://www.mysqlkorea.com/sub.html?mcode=manual&scode=01_1&lang=k&ver_name=5.1)