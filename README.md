# COW-Platform-Spring-Assignment

학습자 2명이 하나의 레포에서 **같은 과제를 서로 간섭 없이** 각자 구현하는 스프링 학습용 레포입니다.

과제 내용은 [docs/requirements.md](docs/requirements.md)를 확인해주세요.

## 디렉토리 구조

```
COW-Platform-Spring-Assignment/
├── README.md
├── .gitignore
├── docs/
│   └── requirements.md   # 과제 요구사항
├── student-a/            # 학습자 A 전용 Spring Boot 프로젝트
└── student-b/            # 학습자 B 전용 Spring Boot 프로젝트 (student-a와 같은 초기 상태)
```

- `student-a`와 `student-b`는 **완전히 독립된 Gradle 프로젝트**입니다. 루트에는 공용 Gradle 설정이 없어서, 한 사람이 빌드를 깨뜨려도 다른 사람에게 영향이 없습니다.
- 두 프로젝트는 빈 상태에서 시작합니다. `AssignmentApplication`과 기본 테스트만 들어 있어요.

## 기술 스택

| 항목 | 내용 |
| --- | --- |
| Java | 21 |
| Spring Boot | 3.5.16 |
| 빌드 도구 | Gradle (Groovy DSL, Wrapper 포함) |
| DB | H2 인메모리 (재시작하면 초기화) |
| 의존성 | Web, Data JPA, Validation, springdoc-openapi(Swagger UI), H2, Lombok |

## 실행 방법

각자 **자기 디렉토리에서** 실행합니다.

```bash
cd student-a   # 학습자 B는 student-b
./gradlew bootRun
```

IntelliJ에서는 레포 루트가 아니라 **`student-a` 또는 `student-b` 폴더를 프로젝트로 열어야** Gradle 프로젝트로 인식됩니다.

### 접속 주소

| 학습자 | 포트 | Swagger UI | H2 콘솔 |
| --- | --- | --- | --- |
| A | 8081 | http://localhost:8081/swagger-ui.html | http://localhost:8081/h2-console |
| B | 8082 | http://localhost:8082/swagger-ui.html | http://localhost:8082/h2-console |

포트가 달라서 두 프로젝트를 동시에 실행해도 충돌하지 않습니다.

H2 콘솔 로그인 정보

| 항목 | 값 |
| --- | --- |
| JDBC URL | `jdbc:h2:mem:assignment` |
| User Name | `sa` |
| Password | (비워두기) |

> 컨트롤러를 만들기 전에는 Swagger UI에 API가 하나도 보이지 않는 게 정상입니다.

## 브랜치 · 협업 규칙

- 기본 브랜치는 `main`이고, `main`에는 **초기 세팅과 `docs/`만** 둡니다.
- 작업은 `main`에서 분기한 **본인 브랜치**에서 하고, PR로 올립니다.
  - 학습자 A: `student-a/작업명` (예: `student-a/post-crud`)
  - 학습자 B: `student-b/작업명`
- **자기 디렉토리(`student-a/` 또는 `student-b/`) 안의 파일만** 수정합니다. 이 규칙을 지키면 두 사람의 PR이 서로 충돌하지 않습니다.
- 상대방의 디렉토리는 참고용으로 볼 수 있지만 수정하지 않습니다.
- 서로의 PR을 리뷰해보는 것을 추천합니다.

```bash
git checkout main
git pull origin main
git checkout -b student-a/작업명
# 작업 후
git add student-a
git commit -m "작업 내용"
git push origin student-a/작업명
```
