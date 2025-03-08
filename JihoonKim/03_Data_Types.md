# Data Types

### String Types (문자열)

- `CHAR (N)`: 길이가 N으로 고정된 문자열을 저장할 수 있는 필드
  - 해시 값처럼 고정된 길이를 가지는 문자열을 저장할 때 사용
  - 입력 값이 N보다 작은 경우 부족한 부분을 공백으로 채움
  - 입력 값이 N보다 큰 경우 오류 발생
- `VARCHAR (N)`: 길이가 N 이하인 문자열을 저장할 수 있는 필드
- 참고사항
  - PostgreSQL에서는 N의 제한이 없어 N을 생략하는 경우 Text Fields와 같은 역할을 수행
  - 다른 DBMS에서는 N의 제한이 있는 경우가 많음

### Text (텍스트)

- `TEXT`: 여러 길이의 문자열을 저장할 수 있는 필드
  - `WHILE`문, 인덱싱, 정렬 등에서 사용하면 전체 테이블을 스캔하기 때문에 성능이 감소
    - `LIKE`문은 원래 전체 테이블을 스캔하기 때문에 관계 없음
- 참고사항
  - 사용하는 Character Set (UTF-8, latin1 등)에 따라 문자를 정렬하는 방식이 달라질 수 있음

### Binary Type (이진수)

- `BYTEA`: 이진수 데이터를 저장할 수 있는 필드
- 이미지, 파일, 암호화된 데이터 등을 저장할 때 사용할 수 있지만, 잘 사용되지는 않음

### Integer Numbers (정수)

- `SMALLINT`: 16비트 정수형(-2<sup>15</sup> ~ 2<sup>15</sup>, ±3만 정도)까지 저장할 수 있는 필드
- `INTEGER`: 32비트 정수형(-2<sup>31</sup> ~ 2<sup>31</sup>, ±20억 정도)까지 저장할 수 있는 필드
- `BIGINT`: 64비트 정수형(-2<sup>65</sup> ~ 2<sup>65</sup>)까지 저장할 수 있는 필드

### Floating Point Numbers (부동 소수점)

- `REAL`: 32비트 부동소수점을 저장할 수 있는 필드
- `DOUBLE PRECISION`: 64비트 부동소수점을 저장할 수 있는 필드
- `NUMERIC (precision, scale)`: 정확한 계산을 위한 소수점을 저장할 수 있는 필드
  - Precision: 소수점을 포함한 전체 자릿수
  - Scale: 소수점 이하 자릿수
  - Precision과 Scale을 명시하지 않는 경우 시스템 메모리가 허용하는 한도까지 저장할 수 있음
- 컴퓨터에서 소수점 연산을 하는 경우 비트 제한에 따라 대략적인 값만 계산할 수 있음

### Dates (날짜 및 시간)

- `TIMESTAMP`: `YYYY-MM-DD HH:MM:SS`를 저장할 수 있음
- `DATE`: `YYYY-MM-DD`를 저장할 수 있음
- `TIME`: `HH:MM:SS`를 저장할 수 있음
- `NOW`: PostgreSQL의 내장 함수이며, 현재 시간을 반환