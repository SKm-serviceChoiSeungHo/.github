# 🙋‍♂️ About Me

안녕하세요!  

SK엠앤서비스 개발자 직무에 지원한 **최승호**입니다.

이 저장소는 이름 검색 시스템과 GPS 기반 경로 판단 시스템을 구현한 프로젝트입니다.

실제 서비스 환경에서도 견딜 수 있도록 대량의 데이터를 효율적으로 처리하고, 사용자 친화적인 검색 경험을 제공하는 것에 집중했습니다.  

단순히 작동하는 코드가 아닌, **확장 가능하고 유지보수하기 쉬운 구조**를 항상 고민하며 개발합니다.

- 📫 Email: chltmdgh517@naver.com  
- 💻 Blog: [https://balhae.tistory.com](https://balhae.tistory.com/)  
- 🗂️ Portfolio: [https://drive.google.com/file/d/1TrbDxxXgEowxcRZOPU5DKS4e2TO0gvOJ/view](https://drive.google.com/file/d/1TrbDxxXgEowxcRZOPU5DKS4e2TO0gvOJ/view)  

---


# 🛰️ 과제 1 - GPS 도로 매칭 및 경로 이탈 판단 시스템

## 📌 과제 개요

이 시스템은 사용자가 업로드한 GPS 로그(CSV 파일)와 OSM 도로 데이터를 기반으로 주행 경로가 유효한 도로를 따르고 있는지 판단합니다.  
또한 GPS 신호의 품질(거리/방향 기준)에 따라 오차 여부도 판별합니다.

---

## 🧭 전체 설계 흐름

### 1. 도로 데이터 파싱

- `resources/roads.osm` 파일을 최초 1회 파싱하여 도로(Way)와 노드(Node) 정보를 메모리에 로드
- 유효 도로 ID(Set<Long>)를 기준으로 매칭 여부 판단

### 2. GPS CSV 업로드

- 사용자는 Postman 등으로 CSV 파일을 업로드 (예: `POST /api/gps/match`)
- CSV 형식: `Latitude,Longitude,Angle,Speed,HDOP`

### 3. GPS 처리 로직

- 각 GPS 좌표마다 다음 항목을 계산:
  - 가장 가까운 도로 선분과의 거리
  - 도로 진행 방향 각도 vs GPS 진행 방향 각도 차이

- **경로 이탈 여부 판단**:
  - 매칭된 도로 ID가 유효 도로 ID 목록에 없는 경우 → 이탈로 간주

- **GPS 오차 여부 판단**:
  - 도로와의 거리 > 30m  
  - 각도 차이 > 45도

### 4. 결과 반환

- CSV 스타일 텍스트 목록으로 결과 반환  
  (매칭된 도로 ID, 이탈 여부, GPS 오차 여부 포함)
- 마지막 줄에 전체 판정 요약 포함:
  - 경로 이탈 여부
  - GPS 오차 여부

---

## 🧪 실행 방법(서버 비용이 없어서 배포는 못했습니다..)

1. Spring Boot 프로젝트 실행  
2. Postman을 사용한 CSV 파일 업로드 요청  
   `POST /api/gps/uploadPI`   
   멀티파트 폼 데이터 형식으로 CSV 파일을 업로드하는 엔드포인트입니다.

---

###  요청 정보

- **URL**: `http://localhost:8080/api/gps/upload`
- **Method**: `POST`
- **Content-Type**: `multipart/form-data`
- **요청 파라미터**:
  - `file`: 업로드할 CSV 파일 (예: `sample_data.csv`)

---
###  결과

![image](https://github.com/user-attachments/assets/79055105-da7a-47c7-8251-99ee47edd780)

---


# 과제2: 이름 검색 시스템 [🔗 바로가기](https://sktask2.netlify.app/)

이 프로젝트는 사용자가 이름을 입력하면 관련 이름 목록을 자동완성으로 보여주고,  
선택 시 해당 인물의 상세 정보를 조회하는 **이름 검색 시스템**입니다.

프론트는 **React**, 백엔드는 **Spring Boot + JPA**로 구현했으며,  
이름 데이터가 **10만 개 이상으로 증가**해도 원활히 동작하도록 설계하였습니다.


---

## 문제 분석 및 개선 방향

### 📌 기존 HTML 코드의 문제점 (제공된 원본 기준)

| 문제점 | 설명 | 개선 방향 |
|--------|------|-----------|
| ❌ `container` DOM 전체 삭제 및 재생성 (`i()`, `hQuery()`) | 입력이 바뀔 때마다 화면 전체를 지우고 다시 그림 → 비효율적 | ✅ React 컴포넌트로 분리 렌더링 최적화 |
| ❌ 이름 데이터 하드코딩 | `data()` 함수 내에 직접 문자열로 삽입 | ✅ DB에서 동적으로 조회하도록 변경 |
| ❌ 중복 로직 다수 | `nameListQuery1()`/`2()` 여러 번 호출 | ✅ 1회 호출 + 상태값으로 처리 |
| ❌ 느린 성능 | 입력마다 필터링 → 결과 많을수록 렉 | ✅ 백엔드에서 상위 20개만 필터링해 전송 |
| ❌ 자동완성 힌트 복잡 | overlay 레이어 및 cloneNode 처리 반복 | ✅ 단순 `state`로 처리 (React 상태관리) |

---

## 주요 기능

- 🔎 **이름 자동완성**: 입력값이 포함된 이름 목록을 자동 추천 (최대 20개)
- 👤 **이름 클릭 시 상세 조회**: 해당 이름을 가진 인물들의 상세 정보 표시
- 🧠 **대용량 데이터 대응**: 이름 데이터 10만 개 이상이어도 성능 저하 없음
- ⚙️ **프론트-백엔드 연동**: Axios를 통한 REST API 통신
- 🧼 **디자인 유지**: 원본 HTML의 CSS 스타일을 그대로 React로 이식

---

## 🛠️ 기술 스택

- **Frontend**: React
- **Backend**: Spring Boot (Java)
- **DB**: RDS(MySQL) 
- **배포**: Netlify (Frontend), AWS EC2 Dockekr (Backend)

---

## ✅ 실행 화면
![20250703_171205](https://github.com/user-attachments/assets/b95a96b1-7ef2-4892-a4ac-d036d37b3a80)

---

## ✅ 대용량 데이터 대응 전략

### 1. DB에 이름 10만 개를 초기 데이터로 삽입
- `@PostConstruct`를 활용한 자동 초기화 방식
- CSV를 따로 분리하지 않고, 아래와 같이 코드로 직접 삽입함:

```java
IntStream.range(1, 100001).forEach(i -> {
    People person = People.builder()
        .id(UUID.randomUUID().toString())
        .name("이름" + i)
        .gender(i % 2 == 0 ? "남" : "여")
        .age(String.valueOf(20 + (i % 30)))
        .country(i % 2 == 0 ? "대한민국" : "일본")
        .build();

    peopleRepository.save(person);
});
```

### 2. 백엔드에서 이름 검색 시 상위 20개만 조회
```java
List<People> findTop20ByNameContainingIgnoreCase(String query);
