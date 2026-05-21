# 코드 리뷰 리포트

## 검사 대상

* 파일명: `vulnerable_system.py`
* 총 라인 수: 약 230줄

---

# 스타일 검사 결과

## 1. 네이밍 규칙 위반

**문제점:**
`addUser`, `getUser`, `initDB` 등 CamelCase 사용

**이유:**
Python은 snake_case를 권장 (PEP8)

**개선 방법:**

```python
def add_user():
def get_user():
def init_db():
```

---

## 2. import 문제

**문제점:**

```python
import os,sys,sqlite3,subprocess,pickle,json,logging
```

* 여러 모듈 한 줄 import ❌
* `sys` 사용되지 않음 ❌

**개선 방법:**

```python
import os
import sqlite3
import subprocess
import pickle
import json
import logging
```

---

## 3. 함수 책임 과다

**문제점:**
`runMenu()` 함수가 모든 기능 담당

* 입력 처리
* 비즈니스 로직
* 출력

**개선 방법:**

```python
def handle_add_user():
def handle_login():
def handle_backup():
```

---

## 4. docstring 부족

**문제점:**
모든 함수 설명 없음

**개선 방법:**

```python
def login(username: str, password: str) -> bool:
    """사용자 로그인 검증"""
```

---

# 보안 검사 결과

## 🔴 [위험도: 높음] SQL Injection

**문제 코드:**

```python
query = "INSERT INTO users(...) VALUES('" + username + ...
```

**원인:**
사용자 입력을 SQL에 직접 삽입

**공격 예시:**

```
' OR '1'='1
```

**개선 방법:**

```python
cursor.execute(
    "INSERT INTO users(username,password,role) VALUES (?, ?, ?)",
    (username, password, role)
)
```

---

## 🔴 [위험도: 높음] 평문 비밀번호 저장

**문제점:**
비밀번호를 암호화 없이 저장

**위험:**
DB 유출 시 계정 전체 탈취

**개선 방법:**

```python
import bcrypt

hashed = bcrypt.hashpw(password.encode(), bcrypt.gensalt())
```

---

## 🔴 [위험도: 높음] 하드코딩된 Secret

**문제 코드:**

```python
ADMIN_PASSWORD = "admin123"
SECRET_KEY = "my-secret-key"
```

**위험:**
GitHub 업로드 시 즉시 노출

**개선 방법:**

```python
import os

ADMIN_PASSWORD = os.getenv("ADMIN_PASSWORD")
SECRET_KEY = os.getenv("SECRET_KEY")
```

---

## 🔴 [위험도: 높음] Command Injection

**문제 코드:**

```python
os.system("cp " + DB_PATH + " " + filename)
subprocess.check_output(command, shell=True)
```

**공격 예시:**

```
backup.db; rm -rf /
```

**개선 방법:**

```python
subprocess.run(["cp", DB_PATH, filename])
subprocess.check_output(["ping", "-c", "1", host])
```

---

## 🔴 [위험도: 높음] Unsafe Deserialization

**문제 코드:**

```python
pickle.load()
```

**위험:**
악성 파일 실행 가능 (RCE)

**개선 방법:**

```python
json.load()
```

---

## 🟠 [위험도: 중간] 민감정보 노출

**문제 코드:**

```python
SELECT * FROM users
```

**문제점:**
비밀번호 포함 조회

**개선 방법:**

```python
SELECT id, username, role FROM users
```

---

## 🟠 [위험도: 중간] 입력값 검증 부족

**문제점:**

* host
* filename
* username

검증 없음

**개선 방법:**

```python
import re

def is_safe_input(value):
    return bool(re.match(r"^[a-zA-Z0-9._-]+$", value))
```

---

## 🟡 [위험도: 낮음] 예외 처리 부족

**문제점:**
에러 발생 시 프로그램 종료

**개선 방법:**

```python
try:
    ...
except Exception as e:
    logging.error(e)
```

---

# 📋 취약점 요약 (핵심)

| # | 취약점 유형        | 위치                               | 심각도   |
| - | ------------- | -------------------------------- | ----- |
| 1 | 하드코딩된 비밀번호    | `__init__`, `debug_print_secret` | 🔴 높음 |
| 2 | SQL Injection | `execute_search_query`           | 🔴 높음 |
| 3 | XSS           | `unsafe_html_report`             | 🔴 높음 |
| 4 | 입력값 검증 미흡     | `main()`                         | 🟡 중간 |

---

