# AWS Bedrock 기반 OpenClaw

> AWS에서 실행되는 나만의 AI 비서 — WhatsApp, Telegram, Discord, Slack 연결 지원. Amazon Bedrock 기반. API 키 불필요. 원클릭 배포. 월 ~$30부터.

[English](README.md) | [简体中文](README_CN.md) | 한국어

[![License](https://img.shields.io/badge/License-MIT--0-yellow?style=for-the-badge)](https://opensource.org/licenses/MIT)
[![Amazon Bedrock](https://img.shields.io/badge/Powered_by-Amazon_Bedrock-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)](https://aws.amazon.com/bedrock/)
[![CloudFormation](https://img.shields.io/badge/IaC-CloudFormation-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white)](https://aws.amazon.com/cloudformation/)

## 왜 이 프로젝트가 필요한가

[OpenClaw](https://github.com/openclaw/openclaw)는 가장 빠르게 성장하는 오픈소스 AI 비서입니다 — 자체 하드웨어에서 실행되며, 메시징 앱에 연결되고, 실제로 작업을 수행합니다: 이메일 관리, 웹 브라우징, 명령 실행, 일정 예약 등.

문제점: 설정하려면 여러 제공업체의 API 키를 관리하고, VPN을 구성하고, 보안을 직접 처리해야 합니다.

이 프로젝트가 해결책을 제공합니다. 하나의 CloudFormation 스택으로 다음을 제공합니다:

- **Amazon Bedrock** 모델 액세스 — 10개 모델, 하나의 통합 API, IAM 인증 (API 키 불필요)
- **Graviton ARM 인스턴스** — x86보다 20-40% 저렴
- **SSM Session Manager** — 포트 개방 없이 안전한 액세스
- **VPC 엔드포인트** — AWS 프라이빗 네트워크 내 트래픽 유지
- **CloudTrail** — 모든 API 호출 자동 감사

8분 만에 배포. 휴대폰에서 액세스.

## 빠른 시작

### 원클릭 배포

1. 해당 리전의 "Launch Stack" 클릭
2. EC2 키 페어 선택
3. 약 8분 대기
4. Outputs 탭 확인

| 리전 | 시작 |
|--------|--------|
| **미국 서부 (오리건)** | [![Launch Stack](https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/create/review?stackName=openclaw-bedrock&templateURL=https://raw.githubusercontent.com/YonghoChoi/sample-OpenClaw-on-AWS-with-Bedrock/main/clawdbot-bedrock.yaml) |
| **미국 동부 (버지니아)** | [![Launch Stack](https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/create/review?stackName=openclaw-bedrock&templateURL=https://raw.githubusercontent.com/YonghoChoi/sample-OpenClaw-on-AWS-with-Bedrock/main/clawdbot-bedrock.yaml) |
| **유럽 (아일랜드)** | [![Launch Stack](https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=eu-west-1#/stacks/create/review?stackName=openclaw-bedrock&templateURL=https://raw.githubusercontent.com/YonghoChoi/sample-OpenClaw-on-AWS-with-Bedrock/main/clawdbot-bedrock.yaml) |
| **아시아 태평양 (도쿄)** | [![Launch Stack](https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=ap-northeast-1#/stacks/create/review?stackName=openclaw-bedrock&templateURL=https://raw.githubusercontent.com/YonghoChoi/sample-OpenClaw-on-AWS-with-Bedrock/main/clawdbot-bedrock.yaml) |

> **사전 요구사항**: 대상 리전에 EC2 키 페어를 생성하세요. Bedrock 모델 액세스는 자동으로 활성화됩니다 — 수동 활성화 불필요.

### 배포 후

![CloudFormation Outputs](images/20260305-215111.png)

> 🦞 **웹 UI를 열고 인사하기만 하면 됩니다.** 모든 메시징 플러그인(WhatsApp, Telegram, Discord, Slack, Feishu)이 사전 설치되어 있습니다. OpenClaw에게 연결하고 싶은 플랫폼을 말하면 — 전체 설정 과정을 단계별로 안내합니다. 수동 구성 불필요.

```bash
# 1. SSM Session Manager 플러그인 설치 (1회)
#    https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html

# 2. 포트 포워딩 시작 (터미널 열어둠)
INSTANCE_ID=$(aws cloudformation describe-stacks \
  --stack-name openclaw-bedrock \
  --query 'Stacks[0].Outputs[?OutputKey==`InstanceId`].OutputValue' \
  --output text --region ap-northeast-1)

aws ssm start-session \
  --target $INSTANCE_ID \
  --region ap-northeast-1  # 배포 리전으로 변경 \
  --document-name AWS-StartPortForwardingSession \
  --parameters '{"portNumber":["18789"],"localPortNumber":["18789"]}'

# 3. 토큰 가져오기 (두 번째 터미널에서)
TOKEN=$(aws ssm get-parameter \
  --name /openclaw/openclaw-bedrock/gateway-token \
  --with-decryption \
  --query Parameter.Value \
  --output text --region YOUR_REGION)

# 4. 브라우저에서 열기
echo "http://localhost:18789/?token=$TOKEN"
```

### CLI 배포 (대안)

```bash
aws cloudformation create-stack \
  --stack-name openclaw-bedrock \
  --template-body file://clawdbot-bedrock.yaml \
  --parameters ParameterKey=KeyPairName,ParameterValue=your-keypair \
  --capabilities CAPABILITY_IAM \
  --region us-west-2

aws cloudformation wait stack-create-complete \
  --stack-name openclaw-bedrock --region us-west-2
```

### 🎯 Kiro AI로 배포하기

가이드 방식을 선호하시나요? [Kiro](https://kiro.dev/)가 대화형으로 배포 과정을 안내합니다 — 이 저장소를 워크스페이스로 열고 "OpenClaw 배포를 도와줘"라고 말하면 됩니다.

**[→ Kiro 배포 가이드](QUICK_START_KIRO.md)**

---

## 메시징 플랫폼 연결

배포 후 웹 UI의 "Channels"에서 원하는 플랫폼을 연결하세요:

| 플랫폼 | 설정 | 가이드 |
|----------|-------|-------|
| **WhatsApp** | 휴대폰에서 QR 코드 스캔 | [문서](https://docs.openclaw.ai/channels/whatsapp) |
| **Telegram** | [@BotFather](https://t.me/botfather)로 봇 생성, 토큰 입력 | [문서](https://docs.openclaw.ai/channels/telegram) |
| **Discord** | Developer Portal에서 앱 생성, 봇 토큰 입력 | [문서](https://docs.openclaw.ai/channels/discord) |
| **Slack** | api.slack.com에서 앱 생성, 워크스페이스에 설치 | [문서](https://docs.openclaw.ai/channels/slack) |
| **Microsoft Teams** | Azure Bot 설정 필요 | [문서](https://docs.openclaw.ai/channels/msteams) |
| **Lark / Feishu** | 커뮤니티 플러그인: [openclaw-feishu](https://www.npmjs.com/package/openclaw-feishu) | — |

**전체 플랫폼 문서**: [docs.openclaw.ai](https://docs.openclaw.ai/)

---

## OpenClaw로 무엇을 할 수 있나요?

연결 후 메시지만 보내면 됩니다:

```
나: 도쿄 날씨 알려줘
나: 이 PDF 요약해줘 [파일 첨부]
나: 매일 오전 9시에 이메일 확인하라고 알림
나: google.com 열어서 "AWS Bedrock pricing" 검색해줘
```

| 명령어 | 기능 |
|---------|-------------|
| `/status` | 모델, 사용한 토큰, 비용 표시 |
| `/new` | 새 대화 시작 |
| `/think high` | 심층 추론 모드 활성화 |
| `/help` | 모든 명령어 목록 |

음성 메시지는 WhatsApp과 Telegram에서 작동합니다 — OpenClaw가 전사하고 응답합니다.

---

## 아키텍처

```
사용자 (WhatsApp/Telegram/Discord)
  │
  ▼
┌─────────────────────────────────────────────┐
│  AWS Cloud                                  │
│                                             │
│  EC2 (OpenClaw)  ──IAM──▶  Bedrock         │
│       │                   (Nova/Claude)     │
│       │                                     │
│  VPC Endpoints        CloudTrail            │
│  (프라이빗 네트워크)    (감사 로그)          │
└─────────────────────────────────────────────┘
  │
  ▼
사용자 (응답 수신)
```

- **EC2**: OpenClaw 게이트웨이 실행 (~1GB RAM)
- **Bedrock**: IAM을 통한 모델 추론 (API 키 불필요)
- **SSM**: 안전한 액세스, 공개 포트 없음
- **VPC 엔드포인트**: Bedrock으로의 프라이빗 네트워크 (선택사항, +$22/월)

---

## 모델

하나의 CloudFormation 파라미터로 모델 전환 — 코드 변경 불필요:

| 모델 | 100만 토큰당 입력/출력 | 최적 용도 |
|-------|---------------------------|----------|
| **Nova 2 Lite** (기본) | $0.30 / $2.50 | 일상 작업, Claude보다 90% 저렴 |
| Nova Pro | $0.80 / $3.20 | 균형잡힌 성능, 멀티모달 |
| Claude Opus 4.6 | $15.00 / $75.00 | 가장 강력, 복잡한 에이전트 작업 |
| Claude Opus 4.5 | $15.00 / $75.00 | 심층 분석, 확장된 사고 |
| Claude Sonnet 4.5 | $3.00 / $15.00 | 복잡한 추론, 코딩 |
| Claude Sonnet 4 | $3.00 / $15.00 | 안정적인 코딩 및 분석 |
| Claude Haiku 4.5 | $1.00 / $5.00 | 빠르고 효율적 |
| DeepSeek R1 | $0.55 / $2.19 | 오픈소스 추론 |
| Llama 3.3 70B | — | 오픈소스 대안 |
| Kimi K2.5 | $0.60 / $3.00 | 멀티모달 에이전트, 262K 컨텍스트 |

> [Global CRIS 프로필](https://docs.aws.amazon.com/bedrock/latest/userguide/cross-region-inference.html) 사용 — 모든 리전에 배포 가능, 요청이 최적 위치로 자동 라우팅됩니다.

---

## 비용

### 일반적인 월간 비용 (가벼운 사용)

| 구성 요소 | 비용 |
|-----------|------|
| EC2 (c7g.large, Graviton) | $58 |
| EBS (30GB gp3) | $2.40 |
| VPC 엔드포인트 (선택사항) | $22 |
| Bedrock (Nova 2 Lite, ~100회 대화/일) | $5-8 |
| **합계** | **$65-90** |

> 추가 CPU 여유가 필요하지 않다면 `t4g.medium` ($24/월) 사용으로 총 비용을 ~$31-56으로 낮출 수 있습니다.

### 비용 절감

- Claude 대신 Nova 2 Lite 사용 → 90% 저렴
- x86 대신 Graviton (ARM) 사용 → 20-40% 저렴
- VPC 엔드포인트 제외 → $22/월 절감 (보안성 낮음)
- AWS Savings Plans → EC2 30-40% 할인

### 대안과의 비교

| 옵션 | 비용 | 제공 내용 |
|--------|------|-------------|
| ChatGPT Plus | $20/인/월 | 단일 사용자, 통합 기능 없음 |
| 이 프로젝트 (5명 사용자) | ~$10/인/월 | 다중 사용자, WhatsApp/Telegram/Discord, 완전한 제어 |
| 로컬 Mac Mini | $0 서버 + $20-30 API | 하드웨어 비용, 직접 관리 |

---

## 구성

### 인스턴스 타입

| 타입 | 월간 | RAM | 아키텍처 | 사용 사례 |
|------|---------|-----|-------------|----------|
| t4g.small | $12 | 2GB | Graviton ARM | 개인용 |
| t4g.medium | $24 | 4GB | Graviton ARM | 소규모 팀 |
| t4g.large | $48 | 8GB | Graviton ARM | 중간 규모 팀 |
| **c7g.large** | **$58** | **4GB** | **Graviton ARM** | **균형잡힌 성능 (기본)** |
| c7g.xlarge | $108 | 8GB | Graviton ARM | 고성능 |
| t3.medium | $30 | 4GB | x86 | x86 호환성 |

### 파라미터

| 파라미터 | 기본값 | 설명 |
|-----------|---------|-------------|
| `OpenClawModel` | Nova 2 Lite | Bedrock 모델 ID |
| `OpenClawVersion` | 2026.3.24 | `2026.3.24` (기본, 모델 승인 불필요, WeChat 호환), `2026.4.5` (자동 검색, 임베딩), 또는 `latest` |
| `InstanceType` | c7g.large | EC2 인스턴스 타입 |
| `CreateVPCEndpoints` | false | 프라이빗 네트워킹 (+$22/월) |
| `EnableSandbox` | true | 코드 실행을 위한 Docker 격리 |
| `EnableDataProtection` | false | 스택 삭제 시 EBS 볼륨 유지 |
| `KeyPairName` | 없음 | EC2 키 페어 (선택사항, 긴급 SSH용) |
| `AllowedSSHCIDR` | _(비어있음)_ | SSH 액세스용 CIDR — 비활성화하려면 비워둠 |

---

## 배포 옵션

### 표준 (EC2) — 이 README

대부분의 사용자에게 최적. 고정 비용, 완전한 제어, 24/7 가용성.

### 멀티테넌트 플랫폼 (AgentCore 런타임) — [README_ENTERPRISE.md](README_ENTERPRISE.md)

> ✅ **E2E 검증 완료** — 전체 파이프라인 실행 중: IM → 게이트웨이 → Bedrock H2 프록시 → 테넌트 라우터 → AgentCore Firecracker microVM → OpenClaw CLI → Bedrock → 응답. [데모 가이드 →](demo/README.md)

OpenClaw를 단일 사용자 도구에서 엔터프라이즈 플랫폼으로 전환: 모든 직원이 Firecracker microVM에서 격리된 AI 비서를 받으며, 공유 스킬, 중앙 집중식 거버넌스, 테넌트별 권한 제공. OpenClaw 코드 변경 제로.

```
Telegram/WhatsApp 메시지
  → OpenClaw 게이트웨이 (IM 채널, 웹 UI)
  → Bedrock H2 프록시 (AWS SDK HTTP/2 호출 가로채기)
  → 테넌트 라우터 (직원별 tenant_id 도출)
  → AgentCore 런타임 (Firecracker microVM, 테넌트별 격리)
  → OpenClaw CLI → Bedrock Nova 2 Lite
  → 직원의 IM으로 응답 반환
```

| 제공 내용 | 방법 | 상태 |
|---|---|---|
| 테넌트 격리 | 사용자별 Firecracker microVM (AgentCore 런타임) | ✅ 검증됨 |
| 공유 모델 액세스 | 하나의 Bedrock 계정, 테넌트별 측정 (~$1-2/인/월) | ✅ 검증됨 |
| 테넌트별 권한 프로필 | SSM 기반 규칙, Plan A (프롬프트 인젝션) + Plan E (감사) | ✅ 검증됨 |
| IM 채널 관리 | 단일 사용자와 동일한 설정 (WhatsApp/Telegram/Discord) | ✅ 검증됨 |
| OpenClaw 코드 변경 제로 | 외부 레이어(프록시, 라우터, 진입점)를 통한 모든 관리 | ✅ 검증됨 |
| 번들 SaaS 키로 공유 스킬 | 한 번 설치, 테넌트별 인증 | 🔜 다음 |
| 사람 승인 워크플로 | Auth Agent → 관리자 알림 → 승인/거부 | 🔜 다음 |
| 탄력적 컴퓨팅 | 자동 확장 microVM, 버스트 용량, 사용량 기반 과금 | ✅ 검증됨 |

| 지표 | 값 |
|--------|-------|
| 콜드 스타트 (사용자 체감) | ~3초 (빠른 경로 직접 Bedrock) |
| 콜드 스타트 (실제 microVM) | ~22-25초 (백그라운드, 사용자는 대기하지 않음) |
| 웜 요청 | ~5-10초 |
| 50명 사용자 비용 | ~$65-110/월 (~$1.30-2.20/인) |
| vs ChatGPT Plus (50명 사용자) | $1,000/월 |

**[→ 전체 멀티테넌트 가이드](README_ENTERPRISE.md)** · **[→ 로드맵](ROADMAP.md)**

### 🏢 엔터프라이즈 디지털 워크포스 플랫폼 — [enterprise/](enterprise/)

> **NEW** — OpenClaw를 조직 전체를 위한 중앙 관리형 디지털 워크포스로 전환하세요. 각 직원은 고유한 신원, 권한, 메모리, 지식을 가진 역할별 AI 에이전트를 받으며 — 모두 IT에서 관리하고, OpenClaw 코드 한 줄도 수정하지 않습니다.

멀티테넌트 AgentCore 런타임 위에 구축된 엔터프라이즈 플랫폼은 다음을 추가합니다:

```
┌─────────────────────────────────────────────────────────┐
│  관리 콘솔 (19페이지) + 직원 포털 (5페이지)              │
│  React + Tailwind + FastAPI + DynamoDB + S3              │
├─────────────────────────────────────────────────────────┤
│  3계층 SOUL 아키텍처                                     │
│  글로벌 (IT 잠금) → 직책 (부서 관리자) → 개인            │
│  동일한 LLM, 완전히 다른 에이전트 정체성                 │
├─────────────────────────────────────────────────────────┤
│  엔터프라이즈 제어                                       │
│  RBAC (관리자/매니저/직원) · 스킬 거버넌스               │
│  감사 추적 + AI 이상 탐지 · 사용량 추적                 │
│  메모리 지속성 · 지식 베이스 (S3의 Markdown)            │
└─────────────────────────────────────────────────────────┘
```

| 설계 원칙 | 의미 |
|-----------------|--------------|
| 제로 침습 | 워크스페이스 파일(SOUL.md, TOOLS.md)을 통해 OpenClaw 제어. 포크 없음, 패치 없음. OpenClaw 독립적으로 업그레이드. |
| 서버리스 우선 | AgentCore를 통한 요청당 Firecracker microVM. 20개 에이전트 = ~$65/월 (vs ChatGPT Team $500/월). |
| 설계 단계부터 보안 | 개방 포트 없음, 하드코딩된 자격 증명 없음, 테넌트 격리, IAM 최소 권한, 포괄적 감사. |
| 파일 우선 지식 | S3의 Markdown, 벡터 DB 아님. 인프라 비용 제로, 사람이 읽을 수 있음, 범위 제어됨. |

| 포함 내용 | 세부사항 |
|----------------|---------|
| 24개 페이지 | 대시보드, 조직도, 에이전트, SOUL 편집기, 워크스페이스, 스킬, 지식, 모니터, 감사, 사용량, 승인, 설정, 플레이그라운드 + 5개 포털 페이지 |
| 35개 이상 API 엔드포인트 | DynamoDB 단일 테이블 설계, S3 작업, JWT 인증을 사용하는 FastAPI |
| 3역할 RBAC | 관리자 (전체), 매니저 (부서 범위), 직원 (포털만) |
| 10개 SOUL 템플릿 | SA, SDE, DevOps, QA, AE, PM, Finance, HR, CSM, Legal |
| 26개 스킬 | `allowedRoles`/`blockedRoles` 매니페스트로 역할 필터링 |
| 샘플 조직 | 20명 직원, 20개 에이전트, 13개 부서 — 시드 스크립트 포함 |

**[→ 엔터프라이즈 플랫폼 가이드](README_ENTERPRISE.md)** · **[→ 엔터프라이즈 로드맵](enterprise/ROADMAP.md)**

### EKS (Kubernetes) — 컨테이너 네이티브 배포

Terraform으로 Amazon EKS에서 OpenClaw 에이전트 실행. AWS Global 및 China 리전 지원.

**[→ EKS 배포 가이드 (EN)](docs/DEPLOYMENT_EKS.md)** · **[→ EKS 部署指南 (中文)](docs/DEPLOYMENT_EKS_CN.md)**

### macOS (Apple Silicon) — iOS/macOS 개발용

| 타입 | 칩 | RAM | 월간 |
|------|------|-----|---------|
| mac2.metal | M1 | 16GB | $468 |
| mac2-m2.metal | M2 | 24GB | $632 |
| mac2-m2pro.metal | M2 Pro | 32GB | $792 |

> 24시간 최소 할당. Apple 개발 워크플로우 전용 사용 — Linux가 일반 용도로 12배 저렴합니다.

| 리전 | 시작 |
|--------|--------|
| **미국 서부 (오리건)** | [![Launch Stack](https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/create/review?stackName=openclaw-mac&templateURL=https://sharefile-jiade.s3.cn-northwest-1.amazonaws.com.cn/clawdbot-bedrock-mac.yaml) |
| **미국 동부 (버지니아)** | [![Launch Stack](https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/create/review?stackName=openclaw-mac&templateURL=https://sharefile-jiade.s3.cn-northwest-1.amazonaws.com.cn/clawdbot-bedrock-mac.yaml) |

### 🇨🇳 AWS China (베이징/닝샤)

Bedrock 대신 SiliconFlow (DeepSeek, Qwen, GLM) 사용. SiliconFlow API 키 필요.

| 리전 | 시작 |
|--------|--------|
| **cn-north-1 (베이징)** | [![Launch Stack](https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png)](https://cn-north-1.console.amazonaws.cn/cloudformation/home?region=cn-north-1#/stacks/create/review?stackName=openclaw-china&templateURL=https://sharefile-jiade.s3.cn-northwest-1.amazonaws.com.cn/clawdbot-china.yaml) |
| **cn-northwest-1 (닝샤)** | [![Launch Stack](https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png)](https://cn-northwest-1.console.amazonaws.cn/cloudformation/home?region=cn-northwest-1#/stacks/create/review?stackName=openclaw-china&templateURL=https://sharefile-jiade.s3.cn-northwest-1.amazonaws.com.cn/clawdbot-china.yaml) |

**[→ China 배포 가이드 (中国区部署指南)](docs/DEPLOYMENT_CHINA_REGION.md)**

---

## 보안

| 계층 | 기능 |
|-------|-------------|
| **IAM 역할** | API 키 없음 — 자동 자격 증명 순환 |
| **SSM Session Manager** | 공개 포트 없음, 세션 로깅 |
| **VPC 엔드포인트** | Bedrock 트래픽이 프라이빗 네트워크에 유지됨 |
| **SSM Parameter Store** | 게이트웨이 토큰이 SecureString으로 저장됨, 디스크에 저장 안됨 |
| **공급망 보호** | GPG 서명된 저장소를 통한 Docker, 다운로드 후 실행 방식의 NVM (`curl \| sh` 없음) |
| **Docker 샌드박스** | 그룹 채팅에서 코드 실행 격리 |
| **CloudTrail** | 모든 Bedrock API 호출 감사됨 |

**[→ 전체 보안 가이드](SECURITY.md)**

---

## 커뮤니티 스킬

OpenClaw를 위한 선택적 확장:

- [S3 Files Skill](skills/s3-files-skill/) — 사전 서명된 URL로 S3를 통해 파일 업로드 및 공유 (기본 자동 설치)
- [Kiro CLI Skill](skills/openclaw-kirocli-skill/) — Kiro CLI를 통한 AI 기반 코딩
- [AWS Backup Skill](https://github.com/genedragon/openclaw-aws-backup-skill) — 선택적 KMS 암호화를 사용한 S3 백업/복원

---

## SSM을 통한 SSH와 유사한 액세스

```bash
# 대화형 세션 시작
aws ssm start-session --target i-xxxxxxxxx --region us-east-1

# ubuntu 사용자로 전환
sudo su - ubuntu

# OpenClaw 명령 실행
openclaw --version
openclaw gateway status
```

---

## 문제 해결

일반적인 문제 및 해결책: [TROUBLESHOOTING.md](TROUBLESHOOTING.md)

단계별 배포 가이드: [DEPLOYMENT.md](DEPLOYMENT.md)

---

## 기여

우리는 엔터프라이즈 OpenClaw 플랫폼을 공개적으로 구축하고 있습니다 — 단일 사용자 배포에서 멀티테넌트 디지털 워크포스까지. 엔터프라이즈 아키텍트든, 스킬 개발자든, 보안 연구원이든, 아니면 더 나은 AI 비서를 원하는 누구든지 환영합니다.

도움이 가장 필요한 영역:
- 엔터프라이즈 플랫폼 테스트 (RBAC, SOUL 인젝션, 권한 경계)
- E2E 멀티테넌트 테스트
- 번들 SaaS 자격 증명을 사용하는 스킬 (Jira, Salesforce, SAP)
- 에이전트 간 오케스트레이션
- 비용 벤치마킹 (AgentCore vs EC2)
- 보안 감사 및 침투 테스트

**[→ 로드맵](ROADMAP.md)** · **[→ 기여 가이드](CONTRIBUTING.md)** · **[→ GitHub Issues](https://github.com/aws-samples/sample-OpenClaw-on-AWS-with-Bedrock/issues)**

## 리소스

- [OpenClaw 문서](https://docs.openclaw.ai/) · [OpenClaw GitHub](https://github.com/openclaw/openclaw)
- [Amazon Bedrock 문서](https://docs.aws.amazon.com/bedrock/) · [SSM Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html)
- [OpenClaw on Lightsail](https://aws.amazon.com/blogs/aws/introducing-openclaw-on-amazon-lightsail-to-run-your-autonomous-private-ai-agents/) (공식 AWS 블로그)

## 지원

- **이 프로젝트**: [GitHub Issues](https://github.com/aws-samples/sample-OpenClaw-on-AWS-with-Bedrock/issues)
- **OpenClaw**: [GitHub Issues](https://github.com/openclaw/openclaw/issues) · [Discord](https://discord.gg/openclaw)
- **AWS Bedrock**: [AWS re:Post](https://repost.aws/tags/bedrock)

---

**Built with Kiro** 🦞
