---
layout: post
title: '[Deploy] AWS 서버에 Django 프로젝트 띄우기 - 5. RDS'
subtitle: Setting Up A Database To A Separate Server 
category: Django
author: Che1
---

{% include /deploy/deploy-toc-base.html %}

- - -

이번 포스트에서는 프로젝트가 사용할 데이터베이스를 별개의 서버에 두고 운영하는 방법을 알아본다.  
각자의 기능이 최적으로 운영되도록 하기 위해서는 기능별로 별도의 서버를 두고 관리하는 것이 효율적이다.  
아마존에서 제공하는 `RDS` 는 데이터베이스 운영에 최적화된 환경에서 효율적으로 데이터베이스를 관리하기 위해 별도로 운영하는 서버이다.

- - -

## Relational Database Server 시작하기

아마존 웹 서버 콘솔 페이지로 접속한 다음, 메인화면의 검색창에 `RDS` 를 검색한다.

AWS: [https://aws.amazon.com/ko/?nc2=h_lg](https://aws.amazon.com/ko/?nc2=h_lg)

<img width="600px" src="/img/AWS_deploy/search_rds.png">

- - -

왼쪽의 메뉴에서 `인스턴스` 를 클릭한 다음 `DB 인스턴스 시작` 버튼을 눌러 데이터베이스 인스턴스를 생성한다.

<img width="500px" src="/img/AWS_deploy/start_db_instance.png">

- - -

데이터베이스 엔진을 선택하는 화면에서 원하는 엔진을 선택한다. 이번 포스트에서는 `PostgreSQL` 을 사용한다.  

<img width="700px" src="/img/AWS_deploy/select_db.png">

- - -

데이터베이스 사용 목적을 물어보는 화면이 나타나면 `개발/테스트` 를 선택하고 `다음 단계` 버튼을 누른다.  
`프로덕션` 을 선택하면 과금이 될 수 있으니 주의.

<img width="700px" src="/img/AWS_deploy/purpose_db.png">

- - -

DB 세부정보 지정 화면에서 `RDS 프리 티어가 지원하는 옵션만 표시하기` 에 체크한다.  
`인스턴스 사양` 의 항목들이 아래와 같이 설정된 것을 확인한다.

<img width="700px" src="/img/AWS_deploy/free_tier_check.png">

- - -

화면 하단의 `설정` 항목을 채워준 다음 `다음 단계` 를 눌러 넘어간다.

<img width="500px" src="/img/AWS_deploy/config_db.png">

`DB 인스턴스 식별자` 는 해당 지역의 AWS 사용자들이 각자 고유한 값을 가지므로 다른 사용자의 DB 인스턴스 식별자와 중복된 식별자를 생성할 수 없다.  

- - -

`고급 설정 구성` 의 `VPC 보안 그룹` 항목에서 `새 보안 그룹 생성` 을 선택한다.

<img width="700px" src="/img/AWS_deploy/advanced_setting_1.png">

- - -

아래의 `데이터베이스 옵션` 항목에서 `데이터베이스 이름` 을 지정해준 뒤 다음으로 넘어간다.  
설정에서 볼 수 있듯이 `PostgreSQL` 은 `5432` 번 포트를 사용한다.

<img width="500px" src="/img/AWS_deploy/advanced_setting_2.png">

- - -

인스턴스 생성을 완료한 다음 `인스턴스` 화면으로 이동해서 어느 정도 기다리고 나면 아래와 같이 `사용 가능` 상태가 되어 데이터베이스 서버를 사용할 수 있게 된다.  

<img width="950px" src="/img/AWS_deploy/available_db.png">

- - -

이제 `EC2` 의 `보안 그룹` 으로 이동해보면 데이터베이스 보안 그룹이 추가되어 있고 `5432` 번 포트가 등록되어 있는 것을 볼 수 있다.

<img width="950px" src="/img/AWS_deploy/security_db.png">

- - -

## Django에 연결하기


이제 생성한 DB 인스턴스를 장고 프로젝트에 연결시켜줄 차례이다.  

- - -

#### settings.py 설정

`settings.py` 를 열어 `DATABASES` 변수를 아래와 같이 수정해주자.

```py
# settings.py

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'HOST': 'che1-db.czxnimwgemge.ap-northeast-2.rds.amazonaws.com',
        'PORT': '5432',
        'NAME': 'deploy',
        'USER': 'Che1',
        'PASSWORD': '*******',
    }
}
```

각각의 항목에 들어가야할 값은 `RDS` 인스턴스 화면에서 인스턴스를 체크한뒤 `세부 정보` 아이콘을 클릭하여 나타나는 창에서 확인할 수 있다.

<img width="950px" src="/img/AWS_deploy/detail_db.png">

- `ENGINE`: sqlite3를 postgresql로 바꿔준다.
- `HOST`: `엔드포인트` 항목을 복사-붙여넣기 한다.
- `PORT`: `포트` 항목을 입력한다.
- `NAME`: `DB 이름` 항목을 입력한다.
- `USER`: `사용자 이름` 항목을 입력한다.
- `PASSWORD`: DB 인스턴스를 생성할 때 설정했던 비밀번호를 입력한다.

- - -

이제 터미널에서 `manage.py` 가 있는 경로로 이동한 다음 `showmigrations` 명령을 실행한다.  

```
./manage.py showmigrations
```
```re
admin
 [ ] 0001_initial
 [ ] 0002_logentry_remove_auto_add
auth
 [ ] 0001_initial
 [ ] 0002_alter_permission_name_max_length
 [ ] 0003_alter_user_email_max_length
 [ ] 0004_alter_user_username_opts
 [ ] 0005_alter_user_last_login_null
 [ ] 0006_require_contenttypes_0002
 [ ] 0007_alter_validators_add_error_messages
 [ ] 0008_alter_user_username_max_length
contenttypes
 [ ] 0001_initial
 [ ] 0002_remove_content_type_name
sessions
 [ ] 0001_initial
```

마이그레이션 목록이 나타나면 데이터베이스에 연결이 잘 된 것이다.  

만약 `psycopg2` 가 없다는 에러가 발생할 경우 아래 명령어로 설치해준다.  
`psycopg2` 는 데이터베이스와 장고를 연결해주는 역할을 한다.

```
pip install psycopg2
```

- - -

#### psql 접속 해보기

PostgreSQL 설치 및 사용법에 대한 내용은 별도의 포스트에서 다루도록 하겠다.  

- - -

터미널에서 아래와 같이 입력하여 생성한 데이터베이스에 접속해보자.

```
psql \
--host=che1-db.czxnimwgemge.ap-northeast-2.rds.amazonaws.com \
--user=Che1 \
--port=5432 \
deploy
```

비밀번호를 입력하고 나면 아래와 같이 psql에 접속하게 된다.

```re
psql (9.5.9, server 9.6.2)
WARNING: psql major version 9.5, server major version 9.6.
         Some psql features might not work.
SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
Type "help" for help.

deploy=> 
```

이제 데이터베이스를 별도의 서버에서 운영할 수 있게 되었다.

- - -

###### Reference

이한영 강사님 블로그: [https://lhy.kr/ec2-ubuntu-deploy](https://lhy.kr/ec2-ubuntu-deploy)  