#concept/web

# concept

```cardlink
url: https://www.hahwul.com/cullinan/attack/idor/#tools
title: "IDOR Attack | HAHWUL"
description: "Insecure Direct Object Reference"
host: www.hahwul.com
favicon: https://www.hahwul.com/favicon.ico
image: https://www.hahwul.com/images/card.jpg
```
- reference

**개념**: 어플리케이션 처리 로직에서 사용자 입력 값이 Object에 직접 참조되는 부분들이 모두 영향받는다. 일반적으로 _**ID, Username**_ 등 식별 값이 포인트가 되며, 파라미터가 아닌 Static File 등에서도 식별 정보를 기반으로한 데이터 참조가 있는 경우 IDOR의 영향을 받는다.

**예시**
![[Pasted image 20250617213842.png]]
위 예시에서 account와 같이 *사용자의 Object에 참조될만한 파라미터에 다른 사용자의 값을 넣어* 어플리케이션의 처리, 리턴되는 반응을 살펴서 만약 account 값을 1110을 사용하는 유저의 데이터가 변경되었다면 IDOR에 취약한 것으로 판단 가능하다.

아니면 username같은 파라미터를 바꿨을 때 다른 사용자 페이지로 들어가서 로그인되는 경우도 해당된다. 영향을 받기 쉬운 파라미터는 아래와 같다. GF-pattern에 명시되어있는 파라미터들.
```bash
{
    [
        "id=",
        "user=",
        "account=",
        "number=",
        "order=",
        "no=",
        "doc=",
        "key=",
        "email=",
        "group=",
        "profile=",
        "edit=",
        "report="
    ]
}
```

**탐지 tool**
ZAP, Burpsuite에선 HUNT라는 AddOn을 통해 IDOR의 가능성을 가지는 Request를 Passive Scan 형태로 식별할 수 있다. 자세한 내용은 아래 글 참고
- [https://www.hahwul.com/blog/2018/bugcrowd-hunt-burp-extension/](https://www.hahwul.com/blog/2018/bugcrowd-hunt-burp-extension/)
- [https://www.hahwul.com/blog/2018/bugcrowd-hunt-burp-extension/](https://www.hahwul.com/blog/2018/bugcrowd-hunt-burp-extension/)


**종류**

1. __*수평적 권한 상승*

수평 권한 상승은 IDOR를 통해 *유사한 권한을 가진 다른 사용자의 데이터를 처리*하는 형태의 방법이다.
![[Pasted image 20250617214002.png]]


2. __*수직적 권한 상승*

수직적 권한 상승은 IDOR를 통해 *상위 권한을 가진 다른 사용자의 데이터를 처리*하는 형태의 방법으로, 보통 일반 계정간의 권한 문제가 아닌 Admin 등 상위 권한의 데이터를 침범하는 형태의 방법이다.
![[Pasted image 20250617214028.png]]
![[Pasted image 20250617214038.png]]


# key insight

_**웹 페이지에서 자주 사용되는 파라미터가 보일 때, 파라미터를 조작해 다른 사용자 데이터에 접근할 수 있는지**_
```bash
{
    [
        "id=",
        "user=",
        "account=",
        "number=",
        "order=",
        "no=",
        "doc=",
        "key=",
        "email=",
        "group=",
        "profile=",
        "edit=",
        "report="
    ]
}
```
=> 특히 사용자의 object에 참조되며, 영향을 받기 쉬운 파라미터들. *위 파라미터들이 보이면 한 번 쯤 조작(tamper)을 생각*해보자.

- ==만약 다른 사용자 데이터에 접근할 수 있다 = **IDOR**==
- ==`../../etc/passwd` 같이 파라미터를 조작해 원하는 파일을 읽을 수 있다 = **LFI 취약점


# in wargame
[[[THM] neighbor]]








