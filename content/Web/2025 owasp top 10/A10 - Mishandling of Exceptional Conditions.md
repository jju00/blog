# 개념
---






# List of Mapped CWEs
---
## CWE-209

> **Generation of Error Message Containing Sensitive Information**
> : 에러메세지에 environment, users, or associated data 등의 민감한 정보를 포함하는 경우, 공격자에게 다음 공격의 성공 확률을 올려준다.
> -> 즉, 에러 처리에서 *'민감 정보를 에러에 포함하는 것'* 이 문제 

![[Pasted image 20251230044911.png]]
- sql injection 시도 시 DB 이름, 칼럼 명 등 노출
- path traversal 시 정확한 경로 노출 (타겟 파일까지 적절한 `../`의 개수를 고르게 할 수 있음)

### 예시

- 설정 파일 위치 경로를 그대로 노출하는 경우 
	-> 설정파일을 path traversal, lfi 등 취약점으로 직접 다운로드하여 읽게 되면 DB 장악으로 이어지는 발판이 될 수 있다.
```php
try {
openDbConnection();
}  
_//print exception message that includes exception message and configuration file location_  
catch (Exception $e) {

echo 'Caught exception: ', $e->getMessage(), '\n';  
echo 'Check credentials in config file at: ', $Mysql_config_location, '\n';
}
```

- 서버의 디렉터리 구조가 노출되는 경우 
	-> `$ConfigDir = "/home/myprog/config"`와 같은 서버의 디렉터리 구조가 노출되는 경우 symbolic link following 취약점이나 path traversal 공격에 악용될 수 있다. 
```perl
$ConfigDir = "/home/myprog/config";  
$uname = GetUserInput("username");  
  
_# avoid [CWE-22](https://cwe.mitre.org/data/definitions/22.html), [CWE-78](https://cwe.mitre.org/data/definitions/78.html), others._  
ExitError("Bad hacker!") if ($uname !~ /^\w+$/);  
$file = "$ConfigDir/$uname.txt";  
if (! (-e $file)) {

ExitError("Error: $file does not exist");

}  
...
```


# 프로젝트 적용
---
> thm 주소: 

```python
@app.route('/upload', methods=['GET', 'POST'])
def upload():
    try:
        # 업로드 요청 시 임시 파일명으로 서버에 저장
        tmp_filename = f"tmp_{random.randint(100000, 999999)}"
        save_path = os.path.join(UPLOAD_ROOT, tmp_filename)
        raise OSError(28, "No space left on device")

    except OSError as e:
        # 에러 메시지에 내부 경로 노출
        error_message = f"Error: {save_path} upload failed\n"
        return Response(error_message, status=500, mimetype='text/plain')
```
- 호스트 내부에 돌아가는 백업 파일 서비스의 코드 일부 (flask app.py)
- `/upload`는 백업할 파일을 업로드하는 기능
- 업로드 시 파일명은 임시 파일명을 무작위로 지정하여 업로드
- 이때, except 에러 처리에서 에러 발생 시 에러 메세지에 실수로 `save_path` - 서버 내 파일 구조를 노출하도록 잘못 설계됨 

