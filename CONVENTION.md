# Spring 미니 프로젝트 컨벤션

환경: Java 21, Spring Boot 3, Gradle, JPA, H2

## 1. Git

### 브랜치

| 브랜치 | 용도 |
|---|---|
| `main` | 기본 브랜치. 초기 세팅과 `docs/`만 유지 |
| `student-a/작업명` (예: `student-a/post-crud`) | 학습자 A의 작업 브랜치. `main`에서 분기 |
| `student-b/작업명` (예: `student-b/post-crud`) | 학습자 B의 작업 브랜치. `main`에서 분기 |

- 본인 디렉토리(`student-a/` 또는 `student-b/`) **안의 파일만** 수정한다.
- 상대방의 디렉토리는 참고용으로 볼 수 있지만 수정하지 않는다.
- 별도의 `dev` 브랜치 없이, `main`에서 분기한 본인 작업 브랜치에서 구현하고 PR로 제출한다.

```bash
git checkout main
git pull origin main
git checkout -b student-a/작업명
# 작업 후
git add student-a
git commit -m "feat: 게시글 CRUD 추가"
git push origin student-a/작업명
```

학습자 B는 위 명령의 `student-a`를 `student-b`로 바꿔서 사용한다.

### 커밋

- 형식: `type: 한글 제목` (명사형 종결). 예: `feat: 프로젝트 생성 API 추가`
- type: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`
- 커밋 하나에는 목적 하나만 담고, 기능과 그 테스트는 함께 커밋한다.

### PR

- 본인 작업 브랜치에서 `main`을 대상으로 리뷰용 PR을 올린다. `main`에는 초기 세팅과 `docs/`만 유지하므로 구현 PR은 병합하지 않는다.
- 제목은 커밋과 같은 형식으로 쓰고, PR 하나에는 기능 하나만 담는다.
- 서로의 PR을 리뷰해보는 것을 추천한다.
- 본문은 아래 형식을 쓴다.

```markdown
## 요약
## 작업 내용
## 확인 방법   (./gradlew build 통과, Swagger로 확인한 내용)
## 리뷰 요청
```

## 2. 패키지 구조

도메인별로 나누고, 각 도메인 안에 계층 패키지를 둔다.

```
com.example.assignment
└── {project | product | option | cart | order}
    ├── controller/admin/    # /api/admin/** 
    ├── controller/client/   # /api/**
    ├── service/
    ├── repository/
    ├── entity/
    ├── dto/request/, dto/response/
    └── exception/
```

- 호출 흐름은 `Controller → Service → Repository` 한 방향으로만 간다.
- Controller는 요청 수신, 검증, Service 호출, 응답 구성만 한다. 비즈니스 로직과 Repository 직접 호출은 Service에서 한다.
- Entity를 API 응답으로 직접 반환하지 않는다. Service에서 DTO로 변환한다.

## 3. 코드 스타일

- 클래스는 `UpperCamelCase`, 메서드와 변수는 `lowerCamelCase`, 상수는 `UPPER_SNAKE_CASE`로 쓴다.
- 클래스 이름에는 `XxxController`, `XxxService`, `XxxRepository`처럼 역할 접미사를 붙인다.
- 의존성은 `private final` 필드와 `@RequiredArgsConstructor`로 주입한다. `@Autowired` 필드 주입은 금지한다.
- Lombok은 엔티티에 `@Getter`와 `@NoArgsConstructor(access = PROTECTED)`만 쓴다. 엔티티의 `@Data`와 `@Setter`는 금지한다.
- IDE 기본 포맷(들여쓰기 4칸)을 따르고, 작업과 무관한 파일을 일괄 포맷하지 않는다.

## 4. DTO와 입력 검증

- 요청과 응답 DTO는 `record`로 쓰고, 이름은 `XxxRequestDto`, `XxxResponseDto`로 한다.
- 생성용과 수정용 필드가 다르면 요청 DTO도 나눈다.
- 응답 DTO는 Service에서 직접 생성한다(`new XxxResponseDto(...)`).
- 검증 어노테이션(`@NotBlank`, `@NotNull`, `@Min` 등)은 **필드마다 별도 줄**에 쓰고, Controller에서 `@Valid`로 검증한다.
- 중복 여부나 재고 부족처럼 저장된 데이터에 의존하는 검증은 Service에서 한다.

## 5. API와 예외 처리

- URL은 복수 명사 중심으로 쓴다. 예: `/api/admin/projects/{projectId}/products`
- HTTP 메서드는 조회 `GET`, 생성 `POST`, 수정 `PUT`/`PATCH`, 삭제 `DELETE`로 쓴다.
- 상태 코드는 성공 `200`, 생성 `201`, 입력 오류 `400`, 대상 없음 `404`, 충돌(재고 부족 등) `409`로 통일한다.
- 성공/실패 응답 형식은 하나로 통일하고, 응답 본문의 상태 코드와 실제 HTTP 상태를 일치시킨다.
- 업무 오류는 도메인별 예외(예: `ProductException`)로 던진다. `RuntimeException`과 `IllegalArgumentException`을 직접 던지지 않는다.
- `@RestControllerAdvice` 한 곳에서 예외를 응답으로 변환한다. Controller마다 `try-catch`를 반복하지 않는다.
- 예상 못 한 오류는 `500`으로 응답하고 내부 예외 메시지를 노출하지 않는다.

## 6. JPA와 트랜잭션

- 엔티티는 `@Id Long`, 보호된 기본 생성자, 필요한 값을 받는 생성자를 갖는다. Enum은 `@Enumerated(EnumType.STRING)`으로 저장한다.
- setter 대신 `update(...)`, `decreaseStock(...)`처럼 의미 있는 메서드로 상태를 바꾼다. 수정은 변경 감지를 사용한다.
- 연관관계는 `@ManyToOne(fetch = LAZY)`로 시작하고, 단방향으로 충분하면 단방향으로 둔다. `cascade`와 `orphanRemoval`은 자식이 부모에 완전히 종속될 때만 쓴다.
- Service의 쓰기 메서드에는 `@Transactional`, 조회 메서드에는 `@Transactional(readOnly = true)`를 붙인다. 재고 차감 같은 변경은 반드시 트랜잭션 안에서 처리한다.
- `open-in-view`가 꺼져 있으므로, 연관 데이터 접근과 DTO 변환은 트랜잭션 안에서 끝낸다.
