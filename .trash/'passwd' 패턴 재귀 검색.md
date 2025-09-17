---
cheat: " "
---
[[정보 수집 - 윈도우]]

```
# 위에서 출력한 파일들의 내용 중 passw 라는 비밀번호(password / passwd)와 관련된 문자열을 대/소문자 구분하지 않고 Grep 함. 
Select-String -Pattern 'passw' -CaseSensitive:$false


# passw 문자열이 들어간 파일이있다면 Path로 오브젝트들을 그룹화 한 뒤, 해당 파일의 Name 특성만 출력 
Group-Object Path | ForEach-Object { "[+] Potential plaintext creds: $($_.Name)" }
```