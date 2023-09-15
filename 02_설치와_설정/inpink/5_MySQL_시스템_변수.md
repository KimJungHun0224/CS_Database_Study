## MySQL 시스템 변수

MySQL 서버는 설정 파일의 내용을 읽어 필요한 설정들을 한다.　   
이 때, 필요한 값들을 **별도의 변수에 저장**해두는데, 이를 **시스템 변수(System Variables)** 라고 한다.　   
　   
시스템 변수들은 아래 명령어를 통해 확인할 수 있다.　   
~~~
mysql > SHOW GLOBAL VARIABLES;
또는
mysql > SHOW VARIABLES;
~~~
　   
내가 원하는 변수 명만 검색하려면, 아래와 같이 Like 연산자를 이용하자. 　   
![image](https://github.com/inpink/CS_Database_Study/assets/108166692/831d9637-0a49-41eb-a04b-a6f991a056bd)　   
　   
　   
> 시스템 변수 중 'GLOBAL VARIABLES'과 'VARIABLES'의 차이는 무엇일까?

우선 시스템 변수들의 목록이 담긴 documentataion를 보고오자.　   
https://dev.mysql.com/doc/refman/8.0/en/server-system-variable-reference.html 　   　   
　   
위 docs 에 적혀있는 시스템 변수의 5가지 속성을 알아보자.　   
- Cmd-Line : Yes면, MySQL 서버의 명령어를 통해 이 시스템 변수의 값을 변경하는 것이 가능하다.　   
- Option file : 설정 파일(my.cnf)로 제어할 수 있는 변수인지의 여부이다.　   
- System Var : 시스템 변수인지 유무이다. 당연히 위에선 다 Yes이다.　   
  예전에는 변수명에 '-'를 쓰기도 하고 '_'를 쓰기도 했는데, MySQL 8.0버전부터 ' _ '를 구분자로 사용하도록 통일되었다.　   
- Var Scope(변수 범위): 변수가 어떤 범위에서 설정될 수 있는지를 나타낸다. Global, Session(커넥션), Both가 있다.　   
  자세한 내용은 아래에서 더 알아보자.　   
- Dynamic :  런타임 중에 값을 변경할 수 있는지를 나타낸다. Dynamic과 Static이 있으며,　   
  자세한 내용은 아래에서 더 알아보자.　   
　   
🌟시스템 변수는 크게 아래와 같이 분류할 수 있다. (Var Scope, Dynamic 옵션)　   
![image](https://github.com/inpink/CS_Database_Study/assets/108166692/02c8df40-2bf8-45a1-9bb9-32a21703802a)　   


> MySQL 시스템 변수 Var Scope(변수 범위)

변수가 어떤 범위에서 설정될 수 있는지를 나타낸다. Global, Session(커넥션), Both가 있다.

- GLOBAL 범위
  : 해당 변수는 MySQL 서버 전체에 영향을 미친다. 즉, 모든 세션 및 연결에 대한 공통 설정이다.
  MySQL 서버가 시작될 때 설정되며, **SET GLOBAL** 명령을 사용하여 런타임 중에 변경할 수 있다.
  my.cnf 또는 my.ini와 같은 MySQL 설정 파일에 설정을 지정할 수 있다.
  예) InnoDB 버퍼 풀 크기(innodb_buffer_pool_size)

- SESSION 범위
  : 하나의 MySQL 서버에는 여러 클라이언트가 접속할 수 있는데, 해당 변수는 각 MySQL 세션(커넥션, 클라이언트)마다 다르게 설정하여 각 세션에만 영향을 준다. 즉, 해당 세션에서만 사용 가능한 설정이다.
  세션이 시작될 때 설정되며, 세션이 종료될 때까지 유지된다. **SET** 명령을 사용하여 현재 세션 내에서 값을 변경할 수 있다.
  **커넥션별로 설정값을 서로 다르게 지정할 수 있으며, 한번 연결된 커넥션의 세션 변수는 서버에서 강제로 변경할 수 없다. MySQL 서버 설정 파일에 초기값을 명시할 수 없다는 것이다.** 
  예) innodb_parallel_read_threads

- BOTH 범위
  : 설정된 변수는 GLOBAL과 SESSION 두 범위에 모두 영향을 미친다.
  MySQL 서버의 전역 설정 값과 현재 세션의 설정 값 모두를 나타낸다.
  SET GLOBAL로 전역 설정 값을 변경하면 모든 세션에 적용되고, SET으로는 현재 세션의 설정 값을 변경할 수 있다.
  MySQL 서버 설정 파일에 초기값을 명시할 수 있다.

  
Global, Session, Both들의 차이점을 더욱 구분하자면,
Global은 서버 전체적으로 영향을 미치므로 **서버 자체에 관련된 설정**이 많다. 설정 파일에 지정해둘 수 있다.
Session은 설정 파일에 지정해둘 수 없으며, 커넥션이 만들어지는 순간부터 **해당 커넥션에서만 유효**한 변수이다.
Both는 MySQL 서버의 설정 파일에 초기값을 명시해두고, 커넥션이 만들어지는 순간에 각 커넥션으로 값이 할당된다. **변수의 값을 변경하면 전역 설정 값과 현재 세션의 설정 값이 모두 변경된다.**


> MySQL 시스템 변수 Dynamic

서버가 기동 중인 상태에서 변수 값을 변경 가능한지 여부에 따라
가능하면 동적 변수(Dynamic), 불가능하면 정적 변수(Static)이다.

우리는 위에서 SHOW 명령을 이용하여 MySQL 서버에 적용된 변수값을 확인했다.
그리고 SET을 이용하여 값을 바꿀 수 있었다.
이 때, 바로 값이 적용되는 것이 Dynamic, 서버를 재시작해야만 적용되는 것이 Static이다.

DYNAMIC (동적 변수)
  : 런타임 중에 값을 변경할 수 있는 변수.
  변수 값을 변경하려면 MySQL 서버를 다시 시작하지 않고도 SET 명령을 사용하여 값을 변경할 수 있다.
  이러한 변수는 변경이 빈번하게 발생하거나 런타임 중에 최적화나 튜닝을 수행해야 할 때 유용하다.

STATIC (정적 변수):
  MySQL 서버가 시작될 때 설정되며, 런타임 중에 변경할 수 없는 변수.
  변수 값을 변경하려면 MySQL 서버를 다시 시작하여 새로운 설정 값을 적용해야 한다.
  이러한 변수는 MySQL 서버가 안정적으로 동작해야 하거나 변경 빈도가 낮을 때 유용하다.

예를 들어, max_connections 변수는 DYNAMIC 변수다.
따라서 MySQL 서버를 다시 시작하지 않고도 런타임 중에 연결 제한 값을 변경할 수 있다.
반면 datadir 변수는 STATIC 변수로, 이 변수 값을 변경하려면 MySQL 서버를 다시 시작해야 한다.

🌟 하지만, SET 명령을 통해 변경되는 시스템 변수값은 설정파일에 반영되지 않는다. 현재 기동 중인 MySQL 인스턴스에만 유효하다는 것이다.
=> MySQL 서버가 재시작되면, 다시 설정 파일에 있는 값으로 돌아가버린다! 
=> 영구히 적용해주려면, 설정 파일에 있는 값도 변경해줘야 한다.
=> MySQL 8.0 버전부터는 **SET PERSIST, SET PERSIST ONLY** 명령어를 제공해줘서, **영구적으로 변경값이 적용될 수 있도록 지원한다**


> SET PERSIST

동적 변수의 경우 변경하면, 현재 실행 중인 서버에 바로 적용된다고 했다.
그런데 SET으로 변경한 뒤 설정파일에도 값을 바꿔주는 것을 잊어서 서비스 장애가 일어날 때가 있다.
이를 방지하기 위해 사용하기 좋은 명령어가 SET PERSIST이다.

**동적 변수의 값을 즉시 변경하며, 별도의 mysqld-autu.cnf 파일에 저장해둔다. 서버가 구동될 때 이 파일또한 참고하기 때문에, 영구히 변경이 적용되게 된다**

아래와 같이 사용할 수 있다.
~~~
set persist max_connections=5000;
~~~


> SET PERSIST ONLY

**SET PERSIST, SET PERSIST ONLY** 명령어를 사용하여 변경된 시스템 변수는, my.cnf 파일이 아닌 mysqld-autu.cnf 파일에 저장된다.
조회하는 방법


44p set persist 관련  조회
performance_schema.varaibles_info  : performance_schema.variables_info 테이블은 이러한 시스템 변수의 메타데이터를 포함하고 있으며, 이를 통해 각 시스템 변수의 이름, 데이터 형식, 설명 등을 조회할 수 있습니다.


> 데이터베이스 vs 테이블

10. 데이터베이스 생성
CREATE DATABASE TESTDB;

11. 데이터베이스 목록 확인
SHOW DATABASES;




set persist_only로 max_connections를 바꾼 뒤, 
set persist_only나 set persist로 변경된 시스템 변수를 조회함. 
![image](https://github.com/inpink/CS_Database_Study/assets/108166692/572a9878-d395-4932-9c3a-540327f20f0d)
9번 참고해서 설명 추가 예정


![image](https://github.com/inpink/CS_Database_Study/assets/108166692/3776479e-13c1-4c4c-a308-892f12fa7860)
![image](https://github.com/inpink/CS_Database_Study/assets/108166692/9026aba3-0b31-4a13-9721-d288e51d4677)

![image](https://github.com/inpink/CS_Database_Study/assets/108166692/b38ee026-0693-4483-8035-7a2c0c818974)
