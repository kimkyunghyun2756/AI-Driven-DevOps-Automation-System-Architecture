# 🌐 OCI Subnet 설정 문서 — FTS_public_subnet

## 📘 기본 정보

| 항목 | 설정값 |
|------|---------|
| **Subnet 이름** | `FTS_public_subnet` |
| **OCID** | `ocid1.subnet.oc1.ap-osaka-1.<masked>` |
| **IPv4 CIDR Block** | `10.0.1.0/24` |
| **IPv6 Prefix** | `—` (비활성화) |
| **Virtual Router MAC Address** | `00:00:17:23:5F:0F` |
| **Subnet Type** | `Regional` |
| **Availability Domain** | `—` |
| **Compartment** | `kekim00 (root)` |
| **DNS Domain Name** | `ftspublicsubnet.ftsvcn.oraclevcn.com` |
| **Subnet Access** | `Public Subnet` |
| **DHCP Options** | `Default DHCP Options for FTS_vcn` |
| **Route Table** | `Default Route Table for FTS_vcn` |

---

## 🏷️ 태그 (Free-form Tags)

| Key | Value | 설명 |
|------|--------|------|
| `Project` | `AI-Agent` | 프로젝트명 (AI Agent 네트워크 인프라용) |
| `Environment` | `Dev` | 개발 환경용 서브넷 |
| `Role` | `PublicSubnet` | 외부 접근 가능한 영역 (Bastion, AI Agent 등 배치 예정) |
| `AccessLevel` | `Public` | 외부 통신 허용 서브넷 |
| `Owner` | `kyunghyun` | 서브넷 관리 책임자 |

---

## 🧭 네트워크 구성 요약

| 항목 | 내용 |
|------|------|
| **소속 VCN** | `FTS_vcn (10.0.0.0/16)` |
| **서브넷 대역** | `10.0.1.0/24` |
| **게이트웨이 연결** | Internet Gateway (외부 통신용) |
| **라우팅 테이블** | Default Route Table for `FTS_vcn` |
| **보안리스트** | 초기 기본값 (추후 SSH/HTTPS 허용 예정) |

---

## 🧩 비고
- 이 서브넷은 **Public Zone**에 속하며, Bastion Host와 AI Agent VM이 배치될 예정입니다.  
- 외부 접근이 필요한 리소스는 반드시 **공인 IP를 할당받고** 이 서브넷에 연결해야 합니다.  
- 내부(Private Subnet) 리소스와의 통신은 VCN 내부 라우팅으로 연결됩니다.

---

📅 **작성일:** 2025-10-23  
✍️ **작성자:** kyunghyun  
💾 **버전:** v1.0 — Public Subnet 설정 기록
