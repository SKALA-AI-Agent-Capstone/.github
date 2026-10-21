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
    A["사건 접수<br/>PDF, 이미지, 증빙"] --> B["현장 감식<br/>문구와 사진을 따로 분석"]
    B --> C["근거 확보<br/>법령 검색, 식약처, 특허 대조"]
    C --> D["대안 제시<br/>검증 통과한 수정 문구"]
    D --> E{"최종 판결<br/>사람이 승인"}
    E -->|승인| F["기록 보관<br/>버전별 이력"]
    E -->|수정 요청| A
```

| 단계          | 하는 일                                                                                                         |
| ------------- | --------------------------------------------------------------------------------------------------------------- |
| **사건 접수** | 상세페이지 PDF나 광고 이미지, 시험성적서 같은 증빙을 올리면 바로 접수하고, 진행 상황은 SSE로 실시간 전달합니다. |
| **현장 감식** | 문구와 사진을 각각 따로 읽어 "7일 만에 주름 개선", "아토피 치료" 같은 표현을 찾습니다.                          |
| **근거 확보** | 화장품법과 관련 고시에서 근거 조문을 찾고, 식약처 기능성 보고와 특허 등록을 공공 API로 대조합니다.              |
| **대안 제시** | 규칙 검사와 모델 검증을 모두 통과한 수정 문구만 제안합니다.                                                     |
| **최종 판결** | 담당자가 문구마다 원문 유지, 수정안 채택, 직접 수정을 고르고 검토자가 승인합니다.                               |
| **기록 보관** | 판정, 근거, 승인 이력을 버전별로 남기고, 광고를 고치면 새 버전으로 다시 검토합니다.                             |

> [!NOTE]
> AI 결과는 내부 검토 초안입니다. 협회 자문이나 정부 인증을 대체하지 않으며 그를 보완하기 위한 서비스임을 알립니다.

<br/>

## 시스템 아키텍처

<img width="100%" alt="시스템 아키텍처" src="./assets/architecture.webp" />

<details>
<summary><b>계층별로 자세히 보기</b></summary>
<br/>

| 계층              | 맡는 일                                                                                                                                                                                                                |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Presentation**  | Vue 화면은 Spring API만 부릅니다. 모델 키와 DB 정보는 브라우저에 오지 않습니다.                                                                                                                                        |
| **Application**   | Spring Boot가 인증, 기업 격리, 파일과 버전, 크레딧, 사람의 결정을 맡습니다. 분석과 승인 후 재개는 DB outbox와 RabbitMQ로 처리해 재시도와 DLQ를 보장합니다.                                                             |
| **Orchestration** | LangGraph Agent가 문구와 사진 분석, 법령 검색, 도구 선택 loop(최대 8회, 같은 도구 2회, 60초), 수정안 생성, 사람 승인 대기를 실행합니다. 승인 대기 상태는 PostgreSQL checkpoint에 저장했다가 결정 후 이어서 실행합니다. |
| **Tool**          | Agent는 업무 데이터에 직접 접근하지 않고 MCP 도구만 거칩니다. `rag-mcp`는 법령 검색과 원문, `business-mcp`는 증빙 읽기와 식약처, KIPRIS 조회를 맡습니다.                                                               |
| **Data**          | PostgreSQL 하나에 업무, Agent 상태, 법령 벡터를 스키마로 나눠 담고, 처리 중인 원본만 Redis에 잠시 둡니다.                                                                                                              |

</details>

<br/>

## ERD

<img width="100%" alt="ERD" src="./assets/erd.webp" />

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
