# 개념
---






# List of Mapped CWEs




## CWE-209

> **Generation of Error Message Containing Sensitive Information**
> : 에러메세지에 environment, users, or associated data 등의 민감한 정보를 포함하는 경우, 공격자에게 다음 공격의 성공 확률을 올려준다.
> -> 즉, 에러 처리에서 *'민감 정보를 에러에 포함하는 것'* 이 문제 

![[Pasted image 20251230044911.png]]
- sql injection 시도 시 DB 이름, 칼럼 명 등 노출
- path traversal 시 정확한 경로 노출 (타겟 파일까지 적절한 `../`의 개수를 고르게 할 수 있음)





