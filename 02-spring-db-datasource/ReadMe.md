# DataSource

> ### 설정과 사용의 분리

- 설정
  - `DataSource`를 만들고 필요한 속성을 사용한다.
  - 설정과 관련된 속성이 한 곳에 있어 향후 변경이 유연하다.
- 사용
  - 설정은 신경쓰지 않고 `getConnection()`만 호출해서 사용한다.


- 설정과 사용의 분리
  - DataSource가 만들어는 시점에 미리 다 넣어둔다면
    - DataSource를 사용하는 곳에서 datasource.getConnection()만 호출하면 된다.
    - USER, USERNAME, PASSWORD에 의존하지 않아도 된다.
    - Repository는 DataSource만 의존하면 된다.
    - 객체를 설정하는 부분과, 사용하는 부분을 더 명확하게 한다.


- DataSource 예제
  - HikariCP 커넥션 풀
    - HikariDataSource는 DataSource 인터페이스를 구현하고 있다.
  - 커넥션 풀 최대 사이즈 : `10`
  - 커넥션 풀 이름 : `MyPool`
  - 커넥션 풀에서 컨넥션을 생성하는 작업을 애플리케이션 실행 속도에 영향을 주지 않기 위해 **별도의 쓰레드에서 작동**한다.

```java
@Test
void dataSourceConnectionPool() throws SQLException, InterruptedException {
    // 커넥션 풀링
    HikariDataSource dataSource = new HikariDataSource();
    dataSource.setJdbcUrl(URL);
    dataSource.setUsername(USERNAME);
    dataSource.setPassword(PASSWORD);
    dataSource.setMaximumPoolSize(10);
    dataSource.setPoolName("MyPool");

    userDataSource(dataSource);
    Thread.sleep(1000);
}
```
- dataSourceConnectionPool() 실행 결과

```text
#커넥션 풀 초기화 정보 출력
HikariConfig - MyPool - configuration:
HikariConfig - maximumPoolSize................................10 HikariConfig - poolName................................"MyPool"
        
#커넥션 풀 전용 쓰레드가 커넥션 풀에 커넥션을 10개 채움
[MyPool connection adder] MyPool - Added connection conn0: url=jdbc:h2:..user=SA
[MyPool connection adder] MyPool - Added connection conn1: url=jdbc:h2:..user=SA
[MyPool connection adder] MyPool - Added connection conn2: url=jdbc:h2:..user=SA
[MyPool connection adder] MyPool - Added connection conn3: url=jdbc:h2:..user=SA
[MyPool connection adder] MyPool - Added connection conn4: url=jdbc:h2:..user=SA
...
[MyPool connection adder] MyPool - Added connection conn9: url=jdbc:h2:..user=SA
        
#커넥션 풀에서 커넥션 획득1
ConnectionTest - connection=HikariProxyConnection@446445803 wrapping conn0: url=jdbc:h2:tcp://localhost/~/test user=SA, class=class com.zaxxer.hikari.pool.HikariProxyConnection
        
#커넥션 풀에서 커넥션 획득2
ConnectionTest - connection=HikariProxyConnection@832292933 wrapping conn1: url=jdbc:h2:tcp://localhost/~/test user=SA, class=class com.zaxxer.hikari.pool.HikariProxyConnection
        
MyPool - After adding stats (total=10, active=2, idle=8, waiting=0)
```

> ### HikariConfig

- HikariCP 관련 설정을 확인할 수 있다.
  - 풀의 이름
  - 최대 풀 수


> ### Pool Connection Adder

- 별도의 Thread를 사용해서 커넥션 풀에 커넥션을 채운다.
- 별도의 Thread를 사용해서 커넥션 풀을 채워야 애플리케이션 실행 시간에 영향을 주지 않는다.

> ### DriverManagerDataSource

- DriverManagerDataSource를 사용하면 conn0~5 번호를 통해서 항상 새로운 커넥션이 생성되어 사용된다.

```text
 get connection=conn0: url=jdbc:h2:.. user=SA class=class
org.h2.jdbc.JdbcConnection
get connection=conn1: url=jdbc:h2:.. user=SA class=class
org.h2.jdbc.JdbcConnection
get connection=conn2: url=jdbc:h2:.. user=SA class=class
org.h2.jdbc.JdbcConnection
get connection=conn3: url=jdbc:h2:.. user=SA class=class
org.h2.jdbc.JdbcConnection
get connection=conn4: url=jdbc:h2:.. user=SA class=class
org.h2.jdbc.JdbcConnection
get connection=conn5: url=jdbc:h2:.. user=SA class=class
org.h2.jdbc.JdbcConnection
```

> ### HikariDataSource

- 커넥션 풀 사용시 conn0 커넥션이 재사용된다.
- 웹 애플리케이션에 동시 요청이 들어오면 여러 쓰레드에서 커넥션 풀의 커넥션을 다양하게 가져가는 것을 확인할 수 있다.

```text
get connection=HikariProxyConnection@xxxxxxxx1 wrapping conn0: url=jdbc:h2:...user=SA
get connection=HikariProxyConnection@xxxxxxxx2 wrapping conn0: url=jdbc:h2:...user=SA
get connection=HikariProxyConnection@xxxxxxxx3 wrapping conn0: url=jdbc:h2:...user=SA
get connection=HikariProxyConnection@xxxxxxxx4 wrapping conn0: url=jdbc:h2:...user=SA
get connection=HikariProxyConnection@xxxxxxxx5 wrapping conn0: url=jdbc:h2:...user=SA
get connection=HikariProxyConnection@xxxxxxxx6 wrapping conn0: url=jdbc:h2:...user=SA
```