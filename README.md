**Labs - 웹 서비스 개발/DevOps에 AWS 서비스 활용**

---

serverless.yml

```yml
service: task-user-service

frameworkVersion: '4'

# 프로바이더 설정
provider:
  name: aws
  runtime: nodejs20.x
  region: ap-northeast-2  # 서울 리전
  stage: ${opt:stage, 'dev'}  # 기본 스테이지는 dev
  
  # IAM 역할 권한 설정
  iam:
    role:
      statements:
        - Effect: Allow
          Action:
            - logs:CreateLogGroup
            - logs:CreateLogStream
            - logs:PutLogEvents
          Resource: '*'

# 함수 정의
functions:
  # Task 서비스 함수
  taskService:
    handler: TaskService/index.handler
    events:
      - http:
          path: /tasks
          method: get
          cors: true  # CORS 활성화
      - http:
          path: /tasks
          method: post
          cors: true
    environment:
      STAGE: ${self:provider.stage}

  # User 서비스 함수
  userService:
    handler: UserService/index.handler
    events:
      - http:
          path: /users
          method: get
          cors: true
      - http:
          path: /users
          method: post
          cors: true
    environment:
      STAGE: ${self:provider.stage}

# 사용자 정의 리소스
resources:
  Resources:
    # API Gateway 응답 헤더 설정
    GatewayResponseDefault4XX:
      Type: 'AWS::ApiGateway::GatewayResponse'
      Properties:
        ResponseParameters:
          gatewayresponse.header.Access-Control-Allow-Origin: "'*'"
          gatewayresponse.header.Access-Control-Allow-Headers: "'*'"
        ResponseType: DEFAULT_4XX
        RestApiId:
          Ref: 'ApiGatewayRestApi'
    
    GatewayResponseDefault5XX:
      Type: 'AWS::ApiGateway::GatewayResponse'
      Properties:
        ResponseParameters:
          gatewayresponse.header.Access-Control-Allow-Origin: "'*'"
          gatewayresponse.header.Access-Control-Allow-Headers: "'*'"
        ResponseType: DEFAULT_5XX
        RestApiId:
          Ref: 'ApiGatewayRestApi'

# 커스텀 설정
custom:
  # 스테이지별 설정
  stages:
    dev:
      logLevel: DEBUG
    prod:
      logLevel: ERROR

# 플러그인 설정
plugins:
  - serverless-offline  # 로컬 테스트용 플러그인

package:
  patterns:
    - '!node_modules/**'  # node_modules 제외
    - '!.git/**'          # .git 폴더 제외

```
