#key_insight #THM #key_insight/web-vuln/param-tamper 

# solve

![[Pasted image 20250617211154.png]]
계정 없으면 guest쓰라고 함
![[Pasted image 20250617211232.png]]
따라서 게스트로 로그인 id: guest pw: guest - 풀 때는 그냥 찍은 건데 알고보니 저 주석의 guest:guest가 id:pw가 guest, guest라는 뜻
![[Pasted image 20250617211240.png]]
저기서 guest를 admin으로 바꿔서 로그인
![[Pasted image 20250617211251.png]]
로그인이 admin으로 성공하면서 플래그 획득

# concept
[[IDOR]]

# key insight

_**웹 페이지에서 파라미터가 보일 때, 파라미터를 조작해서 권한 상승 or 파일을 읽을 수 있는지**_
=> 특히 이 문제에서는 계정을 줘서 guest로 로그인 유도 후, 딱봐도 취약한 파라미터가 보이므로 IDOR를 이용해서 admin으로 접근이 되는지 확인해봐야 함
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
- ==`../../etc/passwd` 같이 파라미터를 조작해 원하는 파일을 읽을 수 있다 = **LFI 취약점**==
