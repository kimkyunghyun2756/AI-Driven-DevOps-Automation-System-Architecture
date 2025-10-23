# 🌐 OCI Gateway 설정 문서

## 📘 개요

이 문서는 `FTS_vcn` 네트워크 내에서 구성된 **Internet Gateway (IGW)** 와  
**NAT Gateway (NGW)** 설정 정보를 정리한 문서입니다.  

- **Internet Gateway** → Public Subnet 외부 통신용  
- **NAT Gateway** → Private Subnet 아웃바운드 전용 통신용  

---

## 🚀 Internet Gateway 설정

| 항목 | 설정값 |
|------|---------|
| **Gateway 이름** | `FTS_internet_gateway` |
| **Gateway 유형** | Internet Gateway |
| **Compartment** | `kekim00 (root)` |
| **소속 VCN** | `FTS_vcn` |
| **상태** | Available |
| **Public IP 할당** | 자동 (OCI 관리) |
| **사용 목적** | Public Subnet (Bastion, AI Agent 등) 외부 인터넷 연결 |
| **연결된 라우트 테이블** | Default Route Table for `FTS_vcn` |
| **라우팅 규칙** | `0.0.0.0/0 → FTS_internet_gateway` |

### 🏷️ 태그 (Free-form Tags)

| Key | Value | 설명 |
|------|--------|------|
| `Project` | `AI-Agent` | 프로젝트명 |
| `Environment` | `Dev` | 개발 환경 |
| `Role` | `InternetGateway` | 외부 통신용 게이트웨이 |
| `Owner` | `kyunghyun` | 담당자 |

---

## 🔒 NAT Gateway 설정

| 항목 | 설정값 |
|------|---------|
| **Gateway 이름** | `FTS_nat_gateway` |
| **Gateway 유형** | NAT Gateway |
| **Compartment** | `kekim00 (root)` |
| **소속 VCN** | `FTS_vcn` |
| **Public IP 설정** | Ephemeral Public IP Address (임시) |
| **상태** | Available |
| **사용 목적** | Private Subnet의 아웃바운드 인터넷 접근 (yum, apt 등) |
| **연결된 라우트 테이블** | Default Route Table for `FTS_vcn` |
| **라우팅 규칙** | `0.0.0.0/0 → FTS_nat_gateway` |

### 🏷️ 태그 (Free-form Tags)

| Key | Value | 설명 |
|------|--------|------|
| `Project` | `AI-Agent` | 프로젝트명 |
| `Environment` | `Dev` | 개발 환경 |
| `Role` | `NATGateway` | 내부 아웃바운드 전용 게이트웨이 |
| `Owner` | `kyunghyun` | 담당자 |

---

## 🧭 라우팅 요약

| 서브넷 | 목적지 CIDR | Target Type | Target |
|----------|--------------|--------------|---------|
| **FTS_public_subnet** | `0.0.0.0/0` | Internet Gateway | `FTS_internet_gateway` |
| **FTS_private_subnet** | `0.0.0.0/0` | NAT Gateway | `FTS_nat_gateway` |

---

## 💡 구성 흐름 요약 다이어그램

```text
[ Internet Gateway ] ←→ [ Public Subnet (10.0.1.0/24) ]
                               ↓
                          Bastion Host
                               ↓ (SSH Tunnel)
[ NAT Gateway ] ←→ [ Private Subnet (10.0.2.0/24) ]
```

---

📅 **작성일**: 2025-10-23
✍️ **작성자**: kyunghyun
💾 **버전**: v1.0 — Internet / NAT Gateway 설정 기록
