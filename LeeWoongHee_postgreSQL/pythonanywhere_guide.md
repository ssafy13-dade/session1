# PostgreSQL for Everyone - PythonAnywhere에서 psql 사용하기

## 1. PythonAnywhere 소개
- **PythonAnywhere**는 무료로 사용할 수 있는 클라우드 기반 Linux 환경을 제공합니다.
- 계정을 만들면 3개월 동안 사용하지 않으면 만료되지만, 계속 사용하면 **영구적으로 무료**입니다.
- PostgreSQL을 실행할 수 있으며, 과제 수행에 필요한 환경을 제공합니다.

---

## 2. PythonAnywhere에서 Bash 콘솔 사용하기
### 2.1. 콘솔 실행하기
1. PythonAnywhere에 로그인합니다.
2. `Consoles` 탭을 열어 **Bash 콘솔**을 실행합니다.
3. 기본적으로 Linux 환경이 제공되며, 다음과 같은 명령어를 사용할 수 있습니다.

### 2.2. 기본적인 Linux 명령어
```bash
pwd        # 현재 작업 디렉터리 확인
ls         # 현재 디렉터리 내 파일 목록 보기
cat file   # 파일 내용 확인 (예: cat lesson1.sql)
cd folder  # 특정 폴더로 이동
cd ..      # 상위 폴더로 이동
cd ~       # 홈 디렉터리로 이동
clear      # 화면 정리
```
---
## PythonAnywhere에서 파일 편집하기.
### 3.1 웹 기반 텍스트 편집기 사용
1. files: 탭을 열어 원하는 파일을 찾음.
    e.g. lesson1.sql (단, README.txt와 같은 설정 파일은 기본 제공이기에 삭제 않하는 것을 권장)

### 3.2 터미널에서 파일 편집하기.
```bash
nano lesson1.sql
vi lesson1.sql
```