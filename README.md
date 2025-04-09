> #### 📁 원본 저장소  
> 본 프로젝트는 팀 프로젝트로 진행되었으며, 아래 저장소에서 협업하였습니다.  
> 🔗 https://github.com/LGCoreNova/Kitcha

<br>

## 시스템 아키텍처
![image](https://github.com/user-attachments/assets/7d65d571-7e07-40e6-b606-d7bdfd7d2f12)

## 1. 프로젝트 개요

- **프로젝트 명**: CoreNova(Kitcha.ver2)
- **기술 원칙**: Microservices Architecture (MSA)
- **프로젝트 목적**:  
  기존 단일 서비스 구조였던 [`Kitcha`](https://github.com/Team4-Kitcha/Kitcha) 프로젝트를 마이크로서비스 구조로 재구성하였습니다.  
  이를 통해 **각 서비스의 독립 배포**, **클라우드 환경에서의 유연한 운영**, **CI/CD 자동화**를 달성하는 것을 목표로 했습니다.
- **구성 기술**: Spring Boot, Spring Cloud, Eureka, Config Server, AWS ECS, ECR, RDS, S3, Docker, Jenkins, GitHub

## 2. 서비스 구성
| 구조명 | 서비스명 | 설명 |
| --- | --- | --- |
| Outer-Architecture | API Gateway | 개발자/회원 등에게 하나의 주소를 제공 |
|  | Eureka | 서비스 등록과 정보 검색을 제공 |
|  | Config Server | GitHub에서개별 설정 파일을 가져와서 공유 |
| Inner-Architecture | authentication | 회원 가입, 로그인, JWT 검사 |
|  | board | 게시판 CRUD |
|  | article | 게시물 CRUD, S3 PDF upload |
| Front-End | Web Page | S3를 통한 정적 웹 호스팅 |
- 의존 서비스 구성
    
    
    | 서비스 | 의존 서비스 | 설명 |
    | --- | --- | --- |
    | `auth` | RDS (MySQL) | 사용자 인증 정보 저장 |
    | `board` | RDS (MySQL), S3 | 게시글 및 PDF 파일 저장 |
    | 모든 서비스 | `config-service`, `eureka-service` | 설정 및 서비스 디스커버리 기능 |

## 3. 아키텍처 및 서비스 흐름

Kitcha 프로젝트는 MSA 기반으로 구성되어 있으며, 클라이언트 요청부터 서비스 처리, 배포와 모니터링까지 다음과 같은 흐름으로 운영됩니다:

- **프론트엔드 웹사이트**는 S3에 정적 호스팅되어 있으며, 사용자의 요청은 **API Gateway**를 거쳐 내부 마이크로서비스로 전달됩니다.
- Gateway는 **Eureka**를 통해 서비스 위치를 확인하고, **ALB(Application Load Balancer)** 의 DNS를 통해 **ECS Fargate**에서 동작 중인 마이크로서비스 태스크로 요청을 라우팅합니다.
- 각 마이크로서비스(`auth`, `board`, `article`)는 Eureka에 등록되며, 필요에 따라 서로 통신하거나 RDS, S3 등 외부 리소스를 활용합니다.
- **CI/CD 파이프라인**은 GitHub Webhook → Jenkins → Docker 이미지 빌드 및 ECR 푸시 → ECS 태스크 정의 등록 및 서비스 업데이트 순서로 자동화되어 있습니다.
- Fargate 기반 서비스는 개별 인스턴스에 접근할 수 없기 때문에, **로그는 AWS CloudWatch Logs**를 통해 수집·모니터링합니다.

## 4. 설정 파일 구성 정보

  [config](https://github.com/LGCoreNova/Config)

## 5. 트러블 슈팅 가이드

  [troubleshooting](https://github.com/LGCoreNova/Kitcha/blob/main/troubleShooting.md)
