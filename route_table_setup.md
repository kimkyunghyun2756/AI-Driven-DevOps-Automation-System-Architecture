# 🧭 OCI Route Table Configuration  
**Project:** AI-Agent  
**Environment:** Dev  
**VCN:** FTS_vcn  
**Compartment:** kekim00 (root)

---

## 📘 개요  
VCN 내부 트래픽이 외부와 통신할 수 있도록  
Public / Private Subnet별 라우트 테이블을 분리하여 구성했습니다.

- Public Subnet: 외부 인터넷 직접 통신 (Internet Gateway 경유)  
- Private Subnet: 외부로 나가되, 내부 보호 (NAT Gateway 경유)  

---

## 🌐 1. RT_Public

| 항목 | 설정값 |
|------|--------|
| **Name** | RT_Public |
| **Compartment** | kekim00 (root) |
| **VCN** | FTS_vcn |
| **대상 Subnet** | FTS_public_subnet |
| **목적** | 외부 인터넷 직접 통신 (예: Bastion Host, AI Agent) |
| **Route Rule** | |
| → Destination CIDR | `0.0.0.0/0` |
| → Target Type | Internet Gateway |
| → Target | `FTS_internet_gateway` |
| **Tags** | Project=AI-Agent, Environment=Dev, Role=RouteTable-Public, Owner=kyunghyun |

✅ **동작 결과:**  
Public Subnet 내 인스턴스는 외부 인터넷과 HTTPS/SSH로 직접 통신 가능.

---

## 🔒 2. RT_Private

| 항목 | 설정값 |
|------|--------|
| **Name** | RT_Private |
| **Compartment** | kekim00 (root) |
| **VCN** | FTS_vcn |
| **대상 Subnet** | FTS_private_subnet |
| **목적** | 내부 시스템이 NAT를 통해 외부 통신만 수행 |
| **Route Rule** | |
| → Destination CIDR | `0.0.0.0/0` |
| → Target Type | NAT Gateway |
| → Target | `FTS_nat_gateway` |
| **Tags** | Project=AI-Agent, Environment=Dev, Role=RouteTable-Private, Owner=kyunghyun |

✅ **동작 결과:**  
Private Subnet 내 인스턴스(DB, K8s 등)는 외부로 나갈 수 있으나,  
외부에서 직접 접근은 불가능.

---

## ⚙️ 3. 연결 관계 요약

| Subnet 이름 | 연결된 Route Table | 외부 접근 | 주요 게이트웨이 |
|--------------|--------------------|------------|------------------|
| FTS_public_subnet | RT_Public | ✅ 가능 | Internet Gateway |
| FTS_private_subnet | RT_Private | 🚫 불가능 (Outbound Only) | NAT Gateway |

---

📅 **작성일**: 2025-10-23
✍️ **작성자**: kyunghyun
💾 **버전**: v1.0 — Public / Private Route Table 설정 기록

---
