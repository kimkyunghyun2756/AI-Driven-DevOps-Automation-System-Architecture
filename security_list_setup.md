# 🔒 OCI Security List Configuration  
**Project:** AI-Agent  
**Environment:** Dev  
**VCN:** FTS_vcn  
**Compartment:** kekim00 (root)

---

## 📘 개요
VCN 내부 트래픽 보안을 위해 Public / Private 서브넷별로  
각기 다른 Security List를 구성했습니다.

- **SL_Public:** 외부 접근 허용 (Bastion Host, AI Agent 등)
- **SL_Private:** 내부 통신만 허용 (DB, K8s, Prometheus 등)

---

## 🌐 1. SL_Public

| 항목 | 설정값 |
|------|--------|
| **Name** | SL_Public |
| **Compartment** | kekim00 (root) |
| **VCN** | FTS_vcn |
| **대상 Subnet** | FTS_public_subnet |
| **역할** | Bastion / Agent 접근 허용용 방화벽 |
| **Tags** | Project=AI-Agent, Environment=Dev, Role=SecurityList-Public, Owner=kyunghyun |

### ✅ Ingress Rules (들어오는 트래픽)

| Source CIDR | Protocol | Port | 설명 |
|--------------|-----------|------|------|
| 0.0.0.0/0 | TCP | 22 | SSH (Bastion 접속 허용) |
| 0.0.0.0/0 | TCP | 80 | HTTP (웹 서비스용) |
| 0.0.0.0/0 | TCP | 443 | HTTPS (보안 웹 통신용) |
| 0.0.0.0/0 | ICMP | ALL | Ping 테스트 허용 |

### ✅ Egress Rules (나가는 트래픽)

| Destination CIDR | Protocol | Port | 설명 |
|------------------|-----------|------|------|
| 0.0.0.0/0 | ALL | ALL | 모든 외부 트래픽 허용 (Internet Gateway 경유) |

---

## 🔒 2. SL_Private

| 항목 | 설정값 |
|------|--------|
| **Name** | SL_Private |
| **Compartment** | kekim00 (root) |
| **VCN** | FTS_vcn |
| **대상 Subnet** | FTS_private_subnet |
| **역할** | 내부 통신 / 모니터링 / DB 전용 방화벽 |
| **Tags** | Project=AI-Agent, Environment=Dev, Role=SecurityList-Private, Owner=kyunghyun |

### ✅ Ingress Rules (내부 통신만 허용)

| Source CIDR | Protocol | Port | 설명 |
|--------------|-----------|------|------|
| 10.0.1.0/24 | TCP | 22 | SSH from Bastion (Public → Private SSH) |
| 10.0.2.0/24 | TCP | 9090 | Prometheus Monitoring |
| 10.0.0.0/16 | ICMP | ALL | 내부 피어 간 Ping 허용 |
| 10.0.2.0/24 | TCP | 27017 | inner DB connection (MongoDB 등) |

### ✅ Egress Rules (나가는 트래픽)

| Destination CIDR | Protocol | Port | 설명 |
|------------------|-----------|------|------|
| 0.0.0.0/0 | ALL | ALL | 모든 아웃바운드 트래픽 허용 (NAT Gateway 경유) |

---

## ⚙️ 3. 연결 관계 요약

| Security List | 연결된 Subnet | 주요 통신 포트 | 주요 게이트웨이 |
|----------------|----------------|----------------|------------------|
| SL_Public | FTS_public_subnet | 22, 80, 443, ICMP | Internet Gateway |
| SL_Private | FTS_private_subnet | 22, 9090, 27017, ICMP | NAT Gateway |

---

## 📂 파일 정보  
**파일명:** `security_list_setup.md`  
**작성자:** kyunghyun  
**마지막 수정:** 2025-10-23  
