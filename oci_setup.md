# 🌐 OCI Virtual Cloud Network (VCN) 설정 문서

## 📘 기본 정보

| 항목 | 설정값 |
|------|---------|
| **VCN 이름** | `FTS_vcn` |
| **CIDR 블록** | `10.0.0.0/16` |
| **DNS 레이블** | `FTSvcn.oraclevcn.com` |
| **컴파트먼트** | `kekim00 (root)` |

---

## 🏷️ 태그 (Free-form Tags)

| Key | Value | 설명 |
|------|--------|------|
| `Project` | `AI-Agent` | 프로젝트명 (AI 기반 Agent 서비스용 네트워크) |
| `Environment` | `Dev` | 개발 환경용 네트워크 (운영 전용 아님) |
| `Role` | `Network` | 네트워크 인프라 리소스 구분용 태그 |
| `Owner` | `kyunghyun` | 리소스 관리 책임자 |

---

## 🧭 추가 참고사항

- **IPv6 Prefix**: 사용 안 함 (기본값 유지)
- **보안 및 접근 정책**: 생성 후 Public / Private Subnet에 맞춰 Security List 구성 예정
- **확장 계획**:
  - Public Subnet → `10.0.1.0/24`
  - Private Subnet → `10.0.2.0/24`
- **리전**: `ap-seoul-1`

---

## ✅ 생성 요약

```text
VCN: FTS_vcn
CIDR: 10.0.0.0/16
DNS: FTSvcn.oraclevcn.com
Compartment: kekim00 (root)
Tags:
  - Project=AI-Agent
  - Environment=Dev
  - Role=Network
  - Owner=kyunghyun
```

---

#### 📅 작성일: 2025-10-23
#### ✍️ 작성자: kyunghyun
#### 💾 버전: v1.0 — Initial VCN setup record

---
