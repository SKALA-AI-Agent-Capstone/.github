<div align="center">

<img src="./assets/vero.webp" width="140" alt="명탐정 VeRo" />

# 명탐정 VeRo

**화장품 광고, 올리기 전에 먼저 수사합니다.**

광고 파일을 올리면 AI Agent가 위반 소지 문구를 찾아<br/>
근거 조문과 고쳐 쓴 문구를 함께 내놓습니다. 최종 판단은 사람이 합니다.

<br/>

<a href="https://meow-vero.vercel.app"><img src="https://img.shields.io/badge/서비스_열기-2563EB?style=for-the-badge&logo=vercel&logoColor=white" /></a>
<a href="https://github.com/SKALA-AI-Agent-Capstone/frontend"><img src="https://img.shields.io/badge/Frontend-0B1B3F?style=for-the-badge&logo=github&logoColor=white" /></a>
<a href="https://github.com/SKALA-AI-Agent-Capstone/backend"><img src="https://img.shields.io/badge/Backend-0B1B3F?style=for-the-badge&logo=github&logoColor=white" /></a>

</div>

<br/>

<img width="100%" alt="명탐정 VeRo 첫 화면" src="./assets/landing.png" />

<br/>

## 사건 개요

> [!IMPORTANT]
> 식약처 화장품 행정처분 **427건 중 324건(76%)** 이 표시광고 위반입니다.

협회 광고자문이 있지만 캠페인 일정에 맞춰 쓰기에는 걸림돌이 많습니다. VeRo는 자문을 대체하지 않고, **게시 전에 브랜드와 대행사가 직접, 바로 돌려 보는 사전검토**를 맡습니다.

| 막히는 지점                 | 지금 벌어지는 일                                                              |
| --------------------------- | ----------------------------------------------------------------------------- |
| **기다릴 수 없는 리드타임** | 자문 결과까지 5일에서 3주. 빠른 처리 경로가 없습니다.                         |
| **한 번 반려되면 끝**       | 부적합이면 재자문이 어려워, 자신 없으면 신청을 건너뛰고 게시합니다.           |
| **법무 지식의 공백**        | 마케터는 위반을 스스로 판단하기 어렵고, 법무는 모든 문구를 손으로 대조합니다. |
| **증빙과 주장의 어긋남**    | 미백만 보고된 제품이 주름 개선을 말해도 게시 전에 잡을 장치가 없습니다.       |
| **승인 절차의 생략**        | 일정에 쫓겨 내부 승인 없이 먼저 게시합니다.                                   |

<br/>

## 수사 과정

```mermaid
flowchart LR
    A["사건 접수<br/>PDF, 이미지, 증빙"] --> B["현장 감식<br/>OCR·VLM 추출과 정합성 검증"]
    B --> C["근거 확보<br/>법령 검색, 식약처, 특허 대조"]
    C --> D["대안 제시<br/>검증 통과한 수정 문구"]
    D --> E{"최종 판결<br/>사람이 승인"}
    E -->|승인| F["기록 보관<br/>버전별 이력"]
    E -->|재작성 요청| D
```

| 단계          | 하는 일                                                                                                         |
| ------------- | --------------------------------------------------------------------------------------------------------------- |
| **사건 접수** | 상세페이지 PDF나 광고 이미지, 시험성적서 같은 증빙을 올리면 바로 접수하고, 진행 상황은 SSE로 실시간 전달합니다. |
| **현장 감식** | OCR로 문구와 위치를 추출하고 VLM 결과와 정합성을 검증해 "7일 만에 주름 개선", "아토피 치료" 같은 표현을 찾습니다. |
| **근거 확보** | 화장품법과 관련 고시에서 근거 조문을 찾고, 식약처 기능성 보고와 특허 등록을 공공 API로 대조합니다.              |
| **대안 제시** | 규칙 검사와 모델 검증을 모두 통과한 수정 문구만 제안합니다.                                                     |
| **최종 판결** | 담당자가 문구마다 원문 유지, 수정안 채택, 직접 수정을 고릅니다. 재작성한 문구는 다시 검증합니다.                 |
| **기록 보관** | 판정, 근거, 문구별 결정과 재작성 이력을 남기고, 광고를 고치면 새 버전으로 다시 검토합니다.                       |

> [!NOTE]
> AI 결과는 내부 검토 초안입니다. 협회 자문이나 정부 인증을 대체하지 않으며 그를 보완하기 위한 서비스임을 알립니다.

<br/>

## 시스템 아키텍처

[<img width="100%" alt="VeRo 시스템 아키텍처 1차 개선안" src="./assets/architecture_fin.png" />](./assets/architecture_fin.png)

OCR·VLM 직렬 처리와 정합성 검증 → Agent 도구 선택 → 독립 모델의 타당성 검증 → 문구 재작성과 사람의 결정으로 이어지는 **1차 개선안**입니다. 과금은 Spring 내부에서 처리하고, 업무·지식·Agent 상태는 PostgreSQL 안에서 구분합니다.

<details>
<summary><b>계층별로 자세히 보기</b></summary>
<br/>

| 계층              | 맡는 일                                                                                                                                                                                                                |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Presentation**  | Vue 화면은 Spring API만 부릅니다. 모델 키와 DB 정보는 브라우저에 오지 않습니다.                                                                                                                                        |
| **Application**   | Spring Boot BFF가 JWT 인증, 기업 격리, SSE와 요청 접수를 맡습니다. 검토·증빙 요청은 DB outbox와 RabbitMQ로 전달합니다. Spring 내부 과금 모듈은 잔액을 사전 확인하고 사용량에 따라 차감을 확정합니다. |
| **Orchestration** | OCR·VLM 정합성 검증 뒤 Agent가 법령 검색과 증빙 대조 도구를 선택합니다. 도구 선택은 최대 8회·동일 도구 연속 2회·60초로 제한합니다. 독립 모델이 근거가 부족한 위반 후보를 제외하고, 남은 위반은 문구 재작성·자가검증과 HITL로 이어집니다. 재작성은 최대 3회이며, 회당 법령 검색 도구 호출은 1회로 제한합니다. |
| **Tool**          | PaddleOCR가 PDF·이미지의 문구와 위치를 추출합니다. `text-embedding-3-small`로 법령·지침을 임베딩하고, `rag-mcp`가 검색합니다. `business-mcp`는 저장된 증빙 결과와 식약처·KIPRIS Plus 조회를 제공합니다. |
| **Data**          | PostgreSQL의 `app`은 광고 원본·사용자·제품·증빙·크레딧을, `knowledge`는 법령·지침과 pgvector 벡터를 저장합니다. Agent 체크포인트는 별도로 관리합니다. 도표의 `state`는 이 상태 저장 영역을 뜻합니다. Redis는 처리 중 원본을 임시 보관하며 TTL은 30분입니다. |

</details>

<br/>

## ERD


### RAG 지식 저장소 · knowledge

[<img width="100%" alt="RAG 지식 저장소 ERD: document_versions, chunks, article_links와 문서·부모 조각 관계" src="./assets/erd/01-knowledge.svg" />](./assets/erd/01-knowledge.svg)

문서 버전과 청크, 청크의 부모·자식 관계를 저장합니다. 임베딩은 pgvector `vector(1024)`를 사용합니다. `article_links`는 법령명과 조문 문자열로 연결하며 문서·청크에 대한 DB FK는 없습니다.


### 인증과 크레딧 · app

[<img width="100%" alt="인증과 크레딧 ERD: 기업, 사용자, 세션, 갱신 토큰, 크레딧 주문·적립·차감·정책" src="./assets/erd/02-auth-credit.svg" />](./assets/erd/02-auth-credit.svg)

기업과 사용자, 로그인 세션, 크레딧 구매·적립·검토별 차감을 관리합니다. `review_credit_charges.review_id`는 검토 삭제 후에도 과금 기록을 보존하기 위해 DB FK로 연결하지 않습니다.


### 검토와 제품 및 증빙 · app

[<img width="100%" alt="검토와 제품 및 증빙 ERD: 제품, 증빙과 개정본, 검토, 원본 파일, 비동기 작업, 위반과 문구 재작성" src="./assets/erd/03-review-product.svg" />](./assets/erd/03-review-product.svg)

제품과 시험성적서 등 증빙, 광고 검토, 비동기 작업, 문구별 결정과 재작성 이력을 관리합니다. 버전별 입력과 분석 결과는 `reviews.document` JSONB에 저장합니다. 검토 전체 승인 컬럼은 V24에서 제거됐으며, 문구별 결정은 `violations`에 유지됩니다.

실선은 DB FK, 점선은 논리 참조입니다. `PK`는 기본 키, `UK`는 고유 키, `NN`은 NOT NULL이며, 복합 키 표시는 구성 컬럼 각각에 붙습니다. Agent 라이브러리가 관리하는 체크포인트 테이블은 이 ERD에서 제외했습니다. 아키텍처는 개선 설계, ERD는 소스 코드 기준이며 실제 DB 적용 상태는 별도로 확인해야 합니다.


<br/>

## 저장소

| 저장소 | 맡는 일 | 주요 기술 |
| --- | --- | --- |
| [**frontend**](https://github.com/SKALA-AI-Agent-Capstone/frontend) | 랜딩, 로그인, 광고 업로드와 진행 화면, 문구별 결정과 승인, 크레딧 결제, 한국어와 영어 | Vue 3, Vite, Pinia |
| [**backend**](https://github.com/SKALA-AI-Agent-Capstone/backend) | 인증과 기업 격리, 검토 이력과 버전, 작업 큐, SSE 중계, 크레딧, 결정 저장 | Java 21, Spring Boot 3, RabbitMQ |
| [**ai_agent**](https://github.com/SKALA-AI-Agent-Capstone/ai_agent) | LangGraph Agent, rag-mcp, business-mcp, 평가셋 | Python, FastAPI, LangGraph, FastMCP |

<br/>

## 기술 스택

| 영역 | 기술 | 적용 범위 |
| :---: | --- | --- |
| **Web** | <img src="https://img.shields.io/badge/Vue_3-4FC08D?style=flat&logo=vuedotjs&logoColor=white" height="26" />&nbsp;&nbsp;<img src="https://img.shields.io/badge/Vite-646CFF?style=flat&logo=vite&logoColor=white" height="26" />&nbsp;&nbsp;<img src="https://img.shields.io/badge/Pinia-FFD859?style=flat&logo=pinia&logoColor=black" height="26" />&nbsp;&nbsp;<img src="https://img.shields.io/badge/vue--i18n-26A69A?style=flat&logo=i18next&logoColor=white" height="26" /> | 화면, 상태 관리, 다국어 |
| **Server** | <img src="https://img.shields.io/badge/Java_21-007396?style=flat&logo=openjdk&logoColor=white" height="26" />&nbsp;&nbsp;<img src="https://img.shields.io/badge/Spring_Boot_3-6DB33F?style=flat&logo=springboot&logoColor=white" height="26" />&nbsp;&nbsp;<img src="https://img.shields.io/badge/Spring_Security-6DB33F?style=flat&logo=springsecurity&logoColor=white" height="26" />&nbsp;&nbsp;<img src="https://img.shields.io/badge/Flyway-CC0200?style=flat&logo=flyway&logoColor=white" height="26" /> | REST API, 인증과 권한, DB 마이그레이션 |
| **Messaging** | <img src="https://img.shields.io/badge/RabbitMQ-FF6600?style=flat&logo=rabbitmq&logoColor=white" height="26" />&nbsp;&nbsp;<img src="https://img.shields.io/badge/SSE-1E3A8A?style=flat" height="26" /> | 작업 큐, 재시도와 DLQ, 진행 이벤트 |
| **AI Agent** | <img src="https://img.shields.io/badge/Python_3.12-3776AB?style=flat&logo=python&logoColor=white" height="26" />&nbsp;&nbsp;<img src="https://img.shields.io/badge/FastAPI-009688?style=flat&logo=fastapi&logoColor=white" height="26" />&nbsp;&nbsp;<img src="https://img.shields.io/badge/LangGraph-1C3C3C?style=flat&logo=langgraph&logoColor=white" height="26" />&nbsp;&nbsp;<img src="https://img.shields.io/badge/OpenAI-412991?style=flat&logo=data:image/svg%2bxml;base64,PHN2ZyBmaWxsPSJ3aGl0ZSIgcm9sZT0iaW1nIiB2aWV3Qm94PSIwIDAgMjQgMjQiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyI+PHRpdGxlPk9wZW5BSTwvdGl0bGU+PHBhdGggZD0iTTIyLjI4MTkgOS44MjExYTUuOTg0NyA1Ljk4NDcgMCAwIDAtLjUxNTctNC45MTA4IDYuMDQ2MiA2LjA0NjIgMCAwIDAtNi41MDk4LTIuOUE2LjA2NTEgNi4wNjUxIDAgMCAwIDQuOTgwNyA0LjE4MThhNS45ODQ3IDUuOTg0NyAwIDAgMC0zLjk5NzcgMi45IDYuMDQ2MiA2LjA0NjIgMCAwIDAgLjc0MjcgNy4wOTY2IDUuOTggNS45OCAwIDAgMCAuNTExIDQuOTEwNyA2LjA1MSA2LjA1MSAwIDAgMCA2LjUxNDYgMi45MDAxQTUuOTg0NyA1Ljk4NDcgMCAwIDAgMTMuMjU5OSAyNGE2LjA1NTcgNi4wNTU3IDAgMCAwIDUuNzcxOC00LjIwNTggNS45ODk0IDUuOTg5NCAwIDAgMCAzLjk5NzctMi45MDAxIDYuMDU1NyA2LjA1NTcgMCAwIDAtLjc0NzUtNy4wNzI5em0tOS4wMjIgMTIuNjA4MWE0LjQ3NTUgNC40NzU1IDAgMCAxLTIuODc2NC0xLjA0MDhsLjE0MTktLjA4MDQgNC43NzgzLTIuNzU4MmEuNzk0OC43OTQ4IDAgMCAwIC4zOTI3LS42ODEzdi02LjczNjlsMi4wMiAxLjE2ODZhLjA3MS4wNzEgMCAwIDEgLjAzOC4wNTJ2NS41ODI2YTQuNTA0IDQuNTA0IDAgMCAxLTQuNDk0NSA0LjQ5NDR6bS05LjY2MDctNC4xMjU0YTQuNDcwOCA0LjQ3MDggMCAwIDEtLjUzNDYtMy4wMTM3bC4xNDIuMDg1MiA0Ljc4MyAyLjc1ODJhLjc3MTIuNzcxMiAwIDAgMCAuNzgwNiAwbDUuODQyOC0zLjM2ODV2Mi4zMzI0YS4wODA0LjA4MDQgMCAwIDEtLjAzMzIuMDYxNUw5Ljc0IDE5Ljk1MDJhNC40OTkyIDQuNDk5MiAwIDAgMS02LjE0MDgtMS42NDY0ek0yLjM0MDggNy44OTU2YTQuNDg1IDQuNDg1IDAgMCAxIDIuMzY1NS0xLjk3MjhWMTEuNmEuNzY2NC43NjY0IDAgMCAwIC4zODc5LjY3NjVsNS44MTQ0IDMuMzU0My0yLjAyMDEgMS4xNjg1YS4wNzU3LjA3NTcgMCAwIDEtLjA3MSAwbC00LjgzMDMtMi43ODY1QTQuNTA0IDQuNTA0IDAgMCAxIDIuMzQwOCA3Ljg3MnptMTYuNTk2MyAzLjg1NThMMTMuMTAzOCA4LjM2NCAxNS4xMTkyIDcuMmEuMDc1Ny4wNzU3IDAgMCAxIC4wNzEgMGw0LjgzMDMgMi43OTEzYTQuNDk0NCA0LjQ5NDQgMCAwIDEtLjY3NjUgOC4xMDQydi01LjY3NzJhLjc5Ljc5IDAgMCAwLS40MDctLjY2N3ptMi4wMTA3LTMuMDIzMWwtLjE0Mi0uMDg1Mi00Ljc3MzUtMi43ODE4YS43NzU5Ljc3NTkgMCAwIDAtLjc4NTQgMEw5LjQwOSA5LjIyOTdWNi44OTc0YS4wNjYyLjA2NjIgMCAwIDEgLjAyODQtLjA2MTVsNC44MzAzLTIuNzg2NmE0LjQ5OTIgNC40OTkyIDAgMCAxIDYuNjgwMiA0LjY2ek04LjMwNjUgMTIuODYzbC0yLjAyLTEuMTYzOGEuMDgwNC4wODA0IDAgMCAxLS4wMzgtLjA1NjdWNi4wNzQyYTQuNDk5MiA0LjQ5OTIgMCAwIDEgNy4zNzU3LTMuNDUzN2wtLjE0Mi4wODA1TDguNzA0IDUuNDU5YS43OTQ4Ljc5NDggMCAwIDAtLjM5MjcuNjgxM3ptMS4wOTc2LTIuMzY1NGwyLjYwMi0xLjQ5OTggMi42MDY5IDEuNDk5OHYyLjk5OTRsLTIuNTk3NCAxLjQ5OTctMi42MDY3LTEuNDk5N1oiLz48L3N2Zz4=" height="26" /> | 분석 그래프, 도구 선택, HITL 재개 |
| **MCP** | <img src="https://img.shields.io/badge/FastMCP-000000?style=flat&logo=modelcontextprotocol&logoColor=white" height="26" />&nbsp;&nbsp;<img src="https://img.shields.io/badge/Pydantic-E92063?style=flat&logo=pydantic&logoColor=white" height="26" /> | 법령 검색, 증빙과 공공 API 조회 |
| **Data** | <img src="https://img.shields.io/badge/PostgreSQL_17-4169E1?style=flat&logo=postgresql&logoColor=white" height="26" />&nbsp;&nbsp;<img src="https://img.shields.io/badge/pgvector-336791?style=flat&logo=postgresql&logoColor=white" height="26" />&nbsp;&nbsp;<img src="https://img.shields.io/badge/Redis-DC382D?style=flat&logo=redis&logoColor=white" height="26" /> | 업무 데이터, Agent 상태, 법령 벡터 검색, 원본 임시 보관 |
| **Ops** | <img src="https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white" height="26" />&nbsp;&nbsp;<img src="https://img.shields.io/badge/Vercel-000000?style=flat&logo=vercel&logoColor=white" height="26" />&nbsp;&nbsp;<img src="https://img.shields.io/badge/Render-46E3B7?style=flat&logo=render&logoColor=black" height="26" />&nbsp;&nbsp;<img src="https://img.shields.io/badge/LangSmith-1C3C3C?style=flat&logo=langchain&logoColor=white" height="26" /> | 로컬 실행, 배포, 실행 추적 |
| **외부 연동** | <img src="https://img.shields.io/badge/식약처_공공데이터-0055A4?style=flat" height="26" />&nbsp;&nbsp;<img src="https://img.shields.io/badge/KIPRIS_Plus-00A0E9?style=flat" height="26" />&nbsp;&nbsp;<img src="https://img.shields.io/badge/Toss_Payments-0064FF?style=flat" height="26" /> | 기능성 보고 조회, 특허 조회, 테스트 결제 |

<br/>

## 수사팀

<div align="center">
<table>
  <tr>
    <td align="center" width="16%"><a href="https://github.com/naneunmina"><img src="https://github.com/naneunmina.png" width="88" /><br/><b>김민아</b></a><br/><sub>Agent</sub></td>
    <td align="center" width="16%"><a href="https://github.com/S-Gihun"><img src="https://github.com/S-Gihun.png" width="88" /><br/><b>손기훈</b></a><br/><sub>FE · BE</sub></td>
    <td align="center" width="16%"><a href="https://github.com/YunhuiHwang"><img src="https://github.com/YunhuiHwang.png" width="88" /><br/><b>황윤희</b></a><br/><sub>Agent · Harness</sub></td>
    <td align="center" width="16%"><a href="https://github.com/kyeonghyunLee"><img src="https://github.com/kyeonghyunLee.png" width="88" /><br/><b>이경현</b></a><br/><sub>BE</sub></td>
    <td align="center" width="16%"><a href="https://github.com/uemy6unoh"><img src="https://github.com/uemy6unoh.png" width="88" /><br/><b>김영환</b></a><br/><sub>RAG · Agent</sub></td>
    <td align="center" width="16%"><a href="https://github.com/largeredjade"><img src="https://github.com/largeredjade.png" width="88" /><br/><b>이홍규</b></a><br/><sub>DB · RAG</sub></td>
  </tr>
</table>
</div>

<br/>

<div align="center">
<img src="./assets/vero-outline.webp" width="72" alt="VeRo" /><br/>
<sub><i>모든 광고에는 단서가 남는다.</i></sub>
</div>

<img width="100%" src="https://capsule-render.vercel.app/api?type=waving&color=0:0b1b3f,100:1e3a8a&height=120&section=footer" />
