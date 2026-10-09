# 백엔드 API 명세서 규칙

- 모든 API에 대해 OpenAPI 3.1 Json 방식으로 API 명세서를 작성한다
- 기존 API의 변경이 이루어졌을시 기존 OpenAPI 파일에도 반영한다 
- OpenAPI Json 파일은 프로젝트 내 도메인별로 구분한다
- 파일은 docs/api/{version}/{project-name}-{domain}-openapi-{version}.json에 작성한다
- description에는 설명을, example에는 예시 값을 작성한다
- 각각의 API에 대해 요청/응답 타입별로 예시 값을 모두 작성한다 (하나의 API에서 요청 타입이 A, B, C일때.. / 응답 코드가 200일때, 404일때, 500일때..) 