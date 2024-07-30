# application.yaml의 로깅(logging)

로그(log) : **정보를 제공**하는 일련의 **기록**

로깅(logging) : **로그(log)**를 **생성**하도록 하는 **시스템**을 말한다.

로깅를 사용하는 이유는 로그를 바탕으로 데이터를 분석하기 위해 사용합니다.


Spring에서 application.yaml을 통해 logging을 하는 방법

```yaml
debug : true
```

이렇게 debug : false 가 되어있으면 진행하는 모든 데이터에서 로그를 뽑아냅니다.

이러면 문제점이, 모든 데이터를 뽑음으로서 쓸모없는 로그를 계속해서 뽑아내어, 성능이 저하되는 문제가 됩니다.

따라서 일반적으로는 특정부분을 한정하여 로그를 뽑아내어야 합니다.


```yaml
logging:
  com.example.exam: true
  org.springframework.web.servlet: true
  org.hibernate.type.descriptor.sql.BasicBinder: trace
```

logging.com.example.exam : true
logging.org.springframework.web.servlet: true

* 해당 패키지에 작동되는 동작들의 로그를 저장 할 수 있습니다.

logging.org.hibernate.type.descriptor.sql.BasicBinder: trace

* 쿼리문 로그에 출력되어 있는 파라미터에 바인딩 되는 값을 확인할 수 있습니다.
* trace로 설정해야 바인딩 파라미터를 확인할 수 있습니다.


```yaml
logging:
  file:
    path: /Users/jsc/Logs/spring
    max-size: 100MB
    max-history: 10
```

logging.file.path : 로그 파일의 경로를 설정합니다

logging.file.max-size : 로그 파일의 최대 크기를 설정합니다

logging.file.max-history : 몇일동안 로그를 저장할 것인지를 지정할 수 있습니다.
