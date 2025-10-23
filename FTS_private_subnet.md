# 🔒 OCI Subnet 설정 문서 — FTS_private_subnet

## 📘 기본 정보

| 항목 | 설정값 |
|------|---------|
| **Subnet 이름** | `FTS_private_subnet` |
| **IPv4 CIDR Block** | `10.0.2.0/24` |
| **IPv6 Prefix** | `—` (비활성화) |
| **Subnet Access** | `Private Subnet` |
| **소속 VCN** | `FTS_vcn (10.0.0.0/16)` |
| **DNS Domain Name** | `ftsprivatesubnet.ftsvcn.oraclevcn.com` |
| **Compartment** | `kekim00 (root)` |
| **게이트웨이** | NAT Gateway |
| **라우트 테이블** | Default Route Table for `FTS_vcn` (또는 별도 Route) |

---

## 🏷️ 태그 (Free-form Tags)

| Key | Value | 설명 |
|------|--------|------|
| `Project` | `AI-Agent` | 프로젝트명 |
| `Environment` | `Dev` | 개발 환경 |
| `Role` | `PrivateSubnet` | 내부 전용 네트워크 |
| `AccessLevel` | `Private` | 외부 접근 차단 |
| `Owner` | `kyunghyun` | 서브넷 담당자 |

---

## 🧭 네트워크 구성 요약

| 항목 | 내용 |
|------|------|
| **게이트웨이 연결** | NAT Gateway (아웃바운드 전용) |
| **라우팅 경로** | `0.0.0.0/0` → NAT Gateway |
| **주요 리소스** | K8s Master / Worker Node, MongoDB, Prometheus, Grafana |
| **보안 규칙** | Bastion Host IP만 SSH(22) 허용, 내부망(10.0.0.0/16) 전체 허용 |

---

📅 **작성일:** 2025-10-23  
✍️ **작성자:** kyunghyun  
💾 **버전:** v1.0 — Private Subnet 설정 기록
