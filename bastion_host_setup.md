# 🧭 Bastion Host 구성 정리 (OCI 환경)

## 📌 개요
배스천 호스트는 **퍼블릭 서브넷에 위치한 SSH 중계 서버**로,  
외부에서 직접 접근할 수 없는 **프라이빗 서브넷 내 VM**으로의 접속을 안전하게 중계하기 위해 사용됩니다.  
OCI의 네트워크 구조 상 Bastion Host는 Internet Gateway를 통해 외부와 연결됩니다.

---

## 🏗️ 인프라 구성 요약

| 항목 | 구성 값 |
|------|----------|
| **VCN 이름** | `FTS_vcn` |
| **VCN CIDR** | `10.0.0.0/16` |
| **퍼블릭 서브넷 이름** | `FTS_public_subnet` |
| **서브넷 CIDR** | `10.0.1.0/24` |
| **라우트 테이블** | `RT_Public` (IGW 연결) |
| **보안 그룹(NSG)** | `NSG_Public_Bastion` |
| **게이트웨이** | `FTS_internet_gateway` |
| **인스턴스 이름** | `bastionHost` |
| **이미지** | Oracle Linux 8 |
| **Shape (사양)** | `VM.Standard.E2.1.Micro` *(Always Free Eligible)* |
| **퍼블릭 IP** | 할당됨 (`161.33.26.203`) |
| **프라이빗 IP** |`-`|

---

## 🌐 네트워크 설정

### 🔸 Route Table: `RT_Public`

| 목적지 (CIDR) | 대상 타입 | 대상 리소스 | 설명 |
|----------------|-------------|--------------|------|
| `0.0.0.0/0` | Internet Gateway | `FTS_igw` | 외부 인터넷 트래픽 라우팅 |

---

### 🔸 Network Security Group: `NSG_Public_Bastion`

#### **Ingress 규칙**

| 방향 | 프로토콜 | 포트 | 소스 CIDR | 설명 |
|------|-----------|------|------------|------|
| Ingress | TCP | 22 | `0.0.0.0/0` | 외부 SSH 접속 허용 |
| Ingress | ICMP | All | `0.0.0.0/0` | Ping 허용 |

#### **Egress 규칙**

| 방향 | 프로토콜 | 포트 | 목적지 CIDR | 설명 |
|------|-----------|------|--------------|------|
| Egress | All | All | `0.0.0.0/0` | 모든 아웃바운드 허용 |

---

## 🖥️ 인스턴스 세부정보

| 항목 | 값 |
|------|----|
| **Hostname** | `bastionhost` |
| **Public IP** | `161.33.26.203` |
| **Private IP** | `-` |
| **Primary VNIC Subnet** | `FTS_public_subnet` |
| **Route Table 연결** | `RT_Public` |
| **상태** | Running |
| **FQDN** | `bastionhost.ftspublicsubnet.ftsvcn.oraclevcn.com` |

---

## 🔐 SSH 접속 설정 (PuTTY 기준)

1. **PuTTYgen 실행**
   - `bastion-key.key` → “Import Key”  
   - “Save private key”로 `.ppk` 파일 저장 (예: `bastion-key.ppk`)

2. **PuTTY 설정**
   - **Host Name:** `opc@161.33.26.203`
   - **Port:** `22`
   - **Connection → SSH → Auth → Private key file:**  
     → `bastion-key.ppk` 지정
   - **Session → Save → Open**

3. **접속 시 콘솔 예시**:
   - login as: opc
   - Authenticating with public key "imported-openssh-key"
   - [opc@bastionhost ~]$
  
 ## 💰 비용 요약 (예상)

| 리소스 | 요금 | 비고 |
|--------|------|------|
| Bastion Host (`E2.1.Micro`) | **무료** | Always Free Eligible |
| Internet Gateway | 무료 | 단, **아웃바운드 트래픽 사용량**에 따라 네트워크 요금 발생 |
| Storage (Boot Volume 50GB) | 무료 | Always Free 한도 내 |
