## 🔍 프로젝트 소개

망분리(Network Segregation)는 기업과 정부기관에서 내부망과 인터넷망을 물리적으로 분리해 사이버 보안을 강화하는 중요한 전략입니다. 하지만 망분리 환경에서도 내부자의 의도적인 보안 위반이나 사용자 실수로 인한 네트워크 혼용이 빈번히 발생하며, 기존의 방화벽(Firewall), 침입탐지시스템(IDS), 네트워크 접근제어(NAC)와 같은 보안 솔루션만으로는 이런 위협을 완벽히 차단하기 어렵습니다.

본 프로젝트는 망분리 환경에서 발생 가능한 내부 보안 위협을 조기에 탐지하고 대응하기 위한 **실시간 이상 트래픽 탐지 및 차단 시스템**을 제안합니다. 본 연구는 특히 **Variational Autoencoder(VAE)**, **Anomaly Transformer**, **나이브 베이즈(Naive Bayes)** 기법을 결합하여 비정상 트래픽을 정확히 탐지하고, 단말의 이상행위를 확률적으로 평가하여 효과적으로 관리할 수 있는 앙상블 기법을 제시합니다.

본 시스템의 주요 기능은 다음과 같습니다.

* SNMP, TAP, Arkime 기반의 다차원 네트워크 트래픽 실시간 수집 및 분석
* VAE를 활용한 세션 단위 이상행위 탐지 및 차원 축소(latent vector) 분석
* Anomaly Transformer를 이용한 시계열 기반 세션 흐름 이상행위 탐지
* 나이브 베이즈 기법을 이용한 단말별 이상행위 확률적 평가 및 우선순위 지정

본 앙상블 접근법을 통해 내부자 위협, 네트워크 혼용, 암호화된 트래픽 내 이상징후 등을 기존 방식보다 훨씬 정밀하게 탐지하며, 궁극적으로 물리적 망분리 환경에서 강력한 **실시간 보안 관제 인프라 구축**에 기여하고자 합니다.


![image](https://github.com/user-attachments/assets/e59eee47-85a9-4ae6-bb57-fc14b4a7ee6d)

## 🌐 망분리(Network Segregation) 환경의 네트워크 구성도

본 그림은 망분리 환경에서 기관 및 기업이 일반적으로 사용하는 네트워크 구성을 나타내고 있습니다. 망분리 환경은 주로 아래와 같이 세 가지 영역으로 구성됩니다:

* **인터넷 네트워크(Internet Network)**: 외부 인터넷과 직접 연결되어 있으며, 다양한 보안 장치를 통해 외부 위협을 탐지하고 차단합니다. 또한 웹 서비스를 안전하게 제공하기 위해 DMZ(Demilitarized Zone) 영역을 운영합니다.
* **사설 네트워크(Private Network)**: 내부 업무망으로, 인터넷망과 완전히 격리하여 기밀성을 유지하고 데이터의 무결성을 확보합니다. 중요한 내부 서비스 및 보안 시스템이 위치합니다.
* **사용자 영역(User Area)**: 실제 사용자들이 업무를 수행하는 구역으로, 인터넷망과 내부 업무망이 동시에 제공되어 사용자 편의성을 지원합니다.

망분리 환경의 도입은 보안성 강화와 데이터 보호라는 측면에서 명확한 이점을 제공합니다. 하지만 관리적인 관점에서 볼 때는 관리해야 할 네트워크 포인트가 증가하여 관리 복잡성 및 운영 비용이 증가할 수 있습니다.

특히 사용자 영역에서는 사용자가 정확히 인지하지 못하고 업무망과 인터넷망을 혼용할 가능성이 있으며, 이는 내부 데이터가 외부로 노출되는 심각한 보안 위협을 초래할 수 있습니다. 또한 관리자의 실수나 잘못된 네트워크 설정으로 인해 네트워크 혼용이 발생할 수 있어 보안상 문제가 지속적으로 발생합니다.



## 📊 데이터 수집 및 처리 방법

본 프로젝트에서는 망분리 환경에서 정확하고 신뢰성 높은 이상 탐지 시스템을 구현하기 위해 데이터의 정확성과 무결성을 확보하는 것이 중요합니다. 따라서 다음과 같은 단계로 데이터를 수집하고 처리하였습니다.

![image](https://github.com/user-attachments/assets/8b75728e-db3f-42b2-9277-d45c483ced95)

### 🔹 데이터 수집 환경 구축

* **네트워크 트래픽 실시간 캡처**

  * **Arkime와 Wireshark** 등을 사용하여 실시간 네트워크 트래픽과 세션 데이터를 캡처·분석합니다.
  * 네트워크의 미러링 포트와 TAP(Test Access Point)를 통해 모든 VLAN의 트래픽을 수집하고, 이를 분석 서버로 집약하여 관리합니다.

* **스위치 포트 통계 정보 수집**

  * SNMP(Simple Network Management Protocol)를 이용하여 브로드캐스트, 멀티캐스트, 유니캐스트, 오류 패킷 등 다양한 포트 통계 정보를 실시간으로 수집하고 분석에 활용합니다.

### 🔹 데이터 정확성 및 무결성 보장 방법

* **다중 네트워크 혼용 탐지**

  * 여러 네트워크가 사용자 실수 또는 의도적인 혼용으로 인해 연결될 경우 심각한 보안 위협이 발생할 수 있습니다. 따라서 이를 방지하기 위해 스위치 장비의 SNMP 정보를 활용하여 네트워크 혼용 상태를 탐지합니다.
  * IP와 MAC 정보를 분석하여 비정상적 네트워크 연결 상태를 조기에 발견하고 관리자에게 경고합니다.

* **비인가 네트워크 장비 탐지 및 제거**

  * 비인가 장비(NAT 라우터 등)가 내부 네트워크에 무단으로 연결되면 보안 정책을 우회하여 침입이 가능합니다. SNMP 데이터와 TTL(Time to Live), ARP 정보 등 네트워크 프로토콜 데이터를 분석하여 비인가 장비를 탐지하고 제거합니다.
  * 이를 통해 비정상 데이터 발생을 최소화하고 데이터의 무결성을 유지합니다.

## 🗂️ 수집 데이터 종류 및 특징

본 프로젝트에서는 망분리 환경의 이상 행위를 효과적으로 탐지하기 위해 다음과 같은 데이터들을 수집하여 사용했습니다. 각 데이터의 특성과 활용 방법은 다음과 같습니다.

### 📌 1. 패킷 데이터 및 세션 정보

* **수집 방법**

  * 오픈소스 툴인 **Arkime** 및 **Wireshark**를 활용하여 네트워크 트래픽과 세션 데이터를 실시간으로 수집·저장했습니다.
  * 모든 트래픽은 세션 단위로 구성하여 분석이 용이하도록 메타데이터로 색인화했습니다.

* **주요 수집 항목**

  * 패킷 발생 시각(timestamp)
  * 세션 길이(length), 지속 시간(duration)
  * TCP/IP 프로토콜 세부 플래그(SYN, ACK 등) 수치
  * 클라이언트·서버 간 데이터 전송량 및 패킷 수 등 **총 82가지 이상의 세션 통계 지표**

* **활용 방안**

  * 수집된 세션 데이터는 **Variational Autoencoder(VAE)** 모델을 이용해 이상 트래픽 탐지 학습에 활용합니다.
  * 이 데이터를 기반으로 실시간 세션 분석을 수행하여 비정상 행위를 빠르게 식별하고 대응합니다.

---

### 📌 2. 네트워크 장비 포트별 통계 지표 데이터

* **수집 방법**

  * SNMP(Simple Network Management Protocol)를 활용하여 스위치 포트별 다양한 트래픽 통계 지표를 주기적으로 수집했습니다.

* **주요 수집 항목**

  * 브로드캐스트(Broadcast) 패킷 수
  * 멀티캐스트(Multicast) 패킷 수
  * 유니캐스트(Unicast) 패킷 수
  * 폐기(Packet Discard) 및 오류(Packet Error) 패킷 수
  * 포트별 총 트래픽 양(IN/OUT)

* **활용 방안**

  * 포트별 통계 지표를 분석하여 평상시의 정상 트래픽 패턴을 학습합니다.
  * 트래픽 지표에서 이상 징후가 발견되면 이를 즉시 탐지하여 빠르게 대응할 수 있습니다.

---

### 📌 3. 통계적 모델링을 통한 행위 수치화 및 확률화

* **수행 방법**

  * 각 단말이 서버에 접근하는 빈도 데이터를 활용하여 포아송 분포(Poisson Distribution) 기반의 통계적 모델링을 수행했습니다.
  * 서버 접근 평균값(람다, λ)을 산출하여 시간별, 단말별 접근 빈도의 정상·비정상을 통계적으로 판단합니다.

* **활용 방안**

  * 이상 행위를 보이는 단말의 접근 빈도를 수치화하여 명확하게 탐지합니다.
  * AI 기반 모델을 보완하는 빠르고 효율적인 탐지 방법으로 활용됩니다.

---

# 🔍 망분리 환경 이상 트래픽 탐지 시스템

본 프로젝트는 망분리(Network Segregation) 환경에서 발생 가능한 이상 트래픽 및 내부 보안 위협을 실시간으로 탐지하고 차단하는 시스템을 구현합니다.  
Variational Autoencoder (VAE), Anomaly Transformer, Naive Bayes를 결합한 앙상블 기반의 탐지 프레임워크를 통해 **암호화된 트래픽**, **네트워크 혼용**, **비인가 단말 접속** 등의 위협에 효과적으로 대응합니다.

---

## 📁 프로젝트 구조

```
📦project-root/
 ┣ 📂data/                 # 세션 및 포트 통계 수집 데이터
 ┣ 📂models/               # 모델 정의 (VAE, Transformer 등)
 ┣ 📂notebooks/            # 분석용 Jupyter 노트북
 ┣ main.py                # 메인 실행 스크립트
 ┗ README.md              # 프로젝트 설명서
```

---

## 📊 데이터 설명

### ✅ 세션 데이터 (session_data.csv)
- 총 82개 필드로 구성
- 주요 항목:
  - `duration`, `tcp_flags_ack`, `client_bytes`, `server_bytes`, `packets`, `bytes_total` 등
- 트래픽 흐름을 수치화한 데이터 → VAE 학습에 사용

### ✅ 포트 통계 데이터 (port_stats.csv)
- SNMP 기반 수집
- 주요 항목:
  - `broadcast_packets`, `multicast_packets`, `packet_errors`, `port_traffic_in/out`
- 스위치 포트별 이상 징후 탐지에 활용

---

## 💡  ELASTIC SERVER에서 데이터 가지고 오기 
```python
from elasticsearch import Elasticsearch
from datetime import datetime
import pandas as pd
import numpy as np
# Elasticsearch 호스트 및 포트 설정
es = Elasticsearch(['http://192.168.0.0:9200'])

# 인덱스 이름, 페이지 크기, 시간 범위 및 특정 값 설정
index_name = 'arkime_sessions3*'
page_size = 500
start_time = datetime(2024, 1, 1)  # 시작 시간 설정
end_time = datetime(2024, 12, 31)  # 종료 시간 설정
desired_value = 'your_desired_value'  # 가져오고자 하는 특정 값

# Elasticsearch Scroll API를 사용하여 데이터 검색
scroll_size = page_size
scroll_timeout = "1m"  # 스크롤 타임아웃 설정

response = es.search(
    index=index_name,
    scroll=scroll_timeout,
    size=scroll_size,
    body={
        "query": {
            "bool": {
                "must": [
                    {
                        "range": {
                            "@timestamp": {
                                "gte": start_time,
                                "lte": end_time
                            }
                        }
                    }
                ]
            }
        }
    }
)

scroll_id = response.get('_scroll_id')

hits_list = []
while True:
    # 스크롤 API를 사용하여 다음 페이지를 가져옵니다.
    scroll_response = es.scroll(scroll_id=scroll_id, scroll=scroll_timeout)
    hits = scroll_response['hits']['hits']
    if not hits:
        break  # 더 이상 데이터가 없으면 루프 종료
    for hit in hits:
        hits_list.append(hit)

```

---

## 💡 학습을 위해 세션 데이터 추출 및 전처리 작업
```python
import pandas as pd
import numpy as np
from collections import defaultdict

keys_to_extract = [
    "firstPacket", "lastPacket", "length", "ipProtocol", 'srcOuiCnt', 'dstOuiCnt',
    "tcpflags_syn", "tcpflags_syn-ack", "tcpflags_ack", "tcpflags_psh", "tcpflags_fin",
    "tcpflags_rst", "tcpflags_urg", "tcpflags_srcZero", "tcpflags_dstZero", "initRTT",
    "source_bytes", "source_packets", "destination_bytes", "destination_packets","destination_mac-cnt",
    "network_packets", "network_bytes", "client_bytes", "server_bytes", "totDataBytes",
    "segmentCnt", "http_bodyMagicCnt", "http_clientVersionCnt", "http_hostCnt",
    "http_keyCnt", "http_methodCnt", "http_pathCnt", "http_request-content-typeCnt",
    "http_request-refererCnt", "http_requestHeaderCnt", "http_response-content-typeCnt",
    "http_responseHeaderCnt", "http_serverVersionCnt", "http_statuscodeCnt", "http_uriCnt",
    "http_useragentCnt", "protocolCnt",  "http_authTypeCnt",
    "http_request-authorizationCnt", "http_userCnt", "srcDscpCnt", "ssh_hasshCnt",
    "ssh_hasshServerCnt", "ssh_versionCnt", "tls_cipherCnt", "tls_ja3Cnt",
    "tls_versionCnt",    "protocol_str_concat", 'destination_port' # Include other keys as needed
]

# 평탄화 함수 정의
def flatten_json(y, parent_key='', sep='_'):
    items = {}
    for key, value in y.items():
        new_key = f"{parent_key}{sep}{key}" if parent_key else key
        if isinstance(value, dict):
            items.update(flatten_json(value, new_key, sep=sep))
        elif isinstance(value, list):
            num_sum = sum(item for item in value if isinstance(item, (int, float)))
            str_concat = ','.join(str(item) for item in value if isinstance(item, str))
            items[f'{new_key}_num_sum'] = num_sum
            items[f'{new_key}_str_concat'] = str_concat
        else:
            items[new_key] = value
    return items

# 데이터 추출 및 평탄화를 위한 함수 정의
def extract_and_flatten(json_data):
    extracted_data = {}
    flat_data = flatten_json(json_data['_source'])
    for key in keys_to_extract:
        value = flat_data.get(key, 0)  # 값이 없을 경우 0을 입력
        extracted_data[key] = value
    return extracted_data

# 데이터 병합을 위한 함수 정의
def merge_json_data(json_list):
    data_list = []
    for json_data in json_list:
        extracted_data = extract_and_flatten(json_data)
        data_list.append(extracted_data)
    return pd.DataFrame(data_list)

# 병합할 JSON 데이터 리스트
json_data_list = hits_list #[json_data_1,json_data_2,json_data_3,json_data_4,json_data_5,json_data_6]

# 병합 함수 호출
merged_df = merge_json_data(json_data_list)
merged_df = pd.concat([merged_df] * 1, ignore_index=True)

# "firstPacket"과 "lastPacket"의 차이값 계산 및 컬럼 추가
merged_df['packetDuration'] = merged_df['lastPacket'] - merged_df['firstPacket']

# 원래 "firstPacket"과 "lastPacket" 컬럼 제거
merged_df = merged_df.drop(columns=['firstPacket', 'lastPacket'])

# 'protocol_str_concat' 컬럼을 리스트로 변환
merged_df['protocol_list'] = merged_df['protocol_str_concat'].str.split(',')
# 'protocol_str_concat' 컬럼 삭제
merged_df.drop(columns=['protocol_str_concat'], inplace=True)

# 지정된 프로토콜 목록
protocols = [
    "http", "https", "ftp", "sftp", "smtp", "pop3", "imap",
    "dns", "tcp", "udp", "ssh", "ssl", "telnet", "snmp", "icmp",
    "bgp", "ospf", "rip", "sip", "voip", "rdp", "mqtt", "ldap",
    "arp", "ip", "ethernet", "nfs", "smb", "quic", "ntp", "http/2", "http/3", "tls"
]

# 각 프로토콜을 컬럼으로 추가
for protocol in protocols:
    merged_df['pthot_'+protocol] = merged_df['protocol_list'].apply(lambda x: 1 if protocol in x else 0)

columns_to_drop = [col for col in merged_df.columns if isinstance(merged_df[col][0], (list, str))]
merged_df = merged_df.drop(columns=columns_to_drop)
```
---


## 💡 학습을 위해 스위치 포트 통계 데이터 추출 및 전처리 작업
```python
# Interface names to filter
interface_names = [ 
    "GigabitEthernet1/0/12",
    "GigabitEthernet1/0/13",
    "GigabitEthernet1/0/14",
    "GigabitEthernet1/0/17",
    "GigabitEthernet1/0/18",
    "GigabitEthernet1/0/22", #이부분이 192.168.0.17로 사용할 인터페이스 정보
    "GigabitEthernet1/0/24",
    "GigabitEthernet1/0/4",
    "GigabitEthernet1/0/5",
    "GigabitEthernet1/0/8"
]

# Filtering the DataFrame to include only rows where 'field_value' matches any of the interface names
filtered_data_all_interfaces = df_switch_sensor_data[df_switch_sensor_data['field_value'].isin(interface_names)]
# Assuming df_switch_sensor_data is already created and has a 'time' column with datetime strings

# Convert 'time' column to datetime format
filtered_data_all_interfaces['time'] = pd.to_datetime(filtered_data_all_interfaces['time'])

# Format 'time' column to 'YYYY-MM-DD HH:MM' format
filtered_data_all_interfaces['time'] = filtered_data_all_interfaces['time'].dt.strftime('%Y-%m-%d %H:%M')

filtered_data_all_interfaces.head()  # Displaying the first few rows to verify the transformation
# 'output' 컬럼을 숫자형으로 변환 (오류 발생 시 NaN으로 대체)
filtered_data_all_interfaces['output'] = pd.to_numeric(filtered_data_all_interfaces['output'], errors='coerce')

# 데이터 정렬 및 그룹화
grouped = filtered_data_all_interfaces.sort_values('time').groupby(['traffic_type', 'field_value'])

# 'output' 값의 시간대별 증가분을 계산
filtered_data_all_interfaces['output_diff'] = grouped['output'].diff().fillna(0)

# 32비트 초과 상황 감지 및 처리
max_32bit_value = 2**32 - 1

def adjust_negative_diff(row):
    if row['output_diff'] < 0:
        return row['output_diff'] + max_32bit_value + 1
    return row['output_diff']

filtered_data_all_interfaces['output_diff'] = filtered_data_all_interfaces.apply(adjust_negative_diff, axis=1)

pivot_table = filtered_data_all_interfaces.pivot_table(
    index=['field_value', 'time'],
    columns='traffic_type',
    values='output_diff',
    aggfunc='sum'  
).reset_index()
```

## 🚀 실행 방법

```bash
# 전체 파이프라인 학습 모드 실행(구현중)
$ python main.py --mode train

```

---

## 🧠 모델 구성 요약

| 구성 요소     | 기법                 | 역할                             |
|---------------|----------------------|----------------------------------|
| 🔹 세션 분석  | VAE                  | 정상/비정상 세션 구분           |
| 🔹 시계열 분석| Anomaly Transformer | 세션 흐름 기반 이상 탐지       |
| 🔹 단말 판단  | Naive Bayes          | 이상 확률 기반 단말 순위화     |

---

## 🛡️ 적용 시나리오

- ✅ 내부 업무망에 외부망 혼용 여부 자동 탐지  
- ✅ 비인가 NAT 공유기·허브 탐지 및 경고  
- ✅ 서버 접근 패턴 기반 이상 단말 탐지  
- ✅ 포트별 트래픽 급증, 오류 패턴 탐지

---
# 📌 제안 방법 및 모델 (Proposed Methods and Models)

본 프로젝트는 실시간 다차원 데이터를 기반으로 네트워크 이상 트래픽을 탐지하고 차단하는 **계층적 탐지 프레임워크**를 제안합니다.  
다음은 제안 모델의 핵심 구성 요소와 각각의 구현 방법에 대한 상세한 설명입니다.

---

## 📖 목차

1. [🔹 제안 모델 개요](#-제안-모델-개요)
2. [🔹 VAE 기반 세션 이상 탐지](#-vae-기반-세션-이상-탐지)
3. [🔹 서버 접근 패턴 기반 이상 탐지 (Poisson 분포 활용)](#-서버-접근-패턴-기반-이상-탐지-poisson-분포-활용)
4. [🔹 세션 흐름 기반 이상 탐지 (Anomaly Transformer)](#-세션-흐름-기반-이상-탐지-anomaly-transformer)
5. [🔹 스위치 포트 통계 데이터 기반 이상 탐지](#-스위치-포트-통계-데이터-기반-이상-탐지)
6. [🔹 나이브 베이즈를 이용한 이상 단말 확률적 판단](#-나이브-베이즈를-이용한-이상-단말-확률적-판단)
7. [🔹 앙상블 기반 탐지 및 최종 판단](#-앙상블-기반-탐지-및-최종-판단)

---

## 🔹 제안 모델 개요

### 🔹 제안 모델의 특징과 장점

본 프로젝트에서 제안하는 이상 트래픽 탐지 모델은 망분리 환경의 다양한 보안 위협을 효과적으로 대응하기 위해 설계되었습니다. 기존 단일 방식의 이상 탐지 기법이 갖는 한계를 보완하고, 다양한 탐지 모델을 앙상블 형태로 결합하여 높은 정확도와 신속한 탐지 능력을 제공합니다.

본 모델의 주요 특징과 장점은 다음과 같습니다:

* **다차원 데이터 기반 탐지**

  * 세션 데이터, 시계열 데이터, 포트 통계 등 다양한 형태의 데이터를 동시에 활용하여 탐지 정확도를 높입니다.

* **비지도 학습 기반 이상 탐지**

  * Variational Autoencoder(VAE)와 Anomaly Transformer를 통해 정상 패턴만을 학습하여 레이블링이 어려운 환경에서도 우수한 탐지 성능을 제공합니다.

* **실시간 확률적 판단 및 우선순위화**

  * 나이브 베이즈(Naive Bayes) 알고리즘을 통해 각 단말의 이상 행위를 실시간으로 확률적으로 평가하여, 탐지된 위협에 신속한 우선 대응이 가능합니다.

* **앙상블 기법을 통한 견고한 탐지**

  * 여러 개의 독립적인 탐지 모델의 결과를 통합(앙상블)하여, 개별 모델의 오탐지(false positive) 및 미탐지(false negative)를 효과적으로 감소시킵니다.

* **유연한 확장성 및 범용성**

  * 서비스 환경이 변경되거나 새로운 트래픽 패턴이 발생해도, 개별 모델의 가중치 조정을 통해 유연하게 대응이 가능하며 다양한 네트워크 환경에서 활용 가능합니다.

---

### 🔹 모델 구조 및 데이터 흐름도

본 프로젝트의 이상 탐지 시스템은 **4개의 주요 구성 요소**로 이루어져 있으며, 각 구성 요소는 독립적이면서도 상호보완적인 역할을 수행합니다. 전체 프로세스는 아래와 같은 단계로 진행됩니다:

#### 🖥️ 모델 구조 개요:

| 구성 요소       | 사용 모델                         | 역할 및 기능                      |
| ----------- | ----------------------------- | ---------------------------- |
| 세션 기반 이상 탐지 | Variational Autoencoder (VAE) | 세션 데이터 재구성 오차 기반 이상 행위 탐지    |
| 서버 접근 패턴 분석 | Poisson 분포 기반 통계 모델           | 서버 접근 횟수에 따른 비정상적 접근 패턴 탐지   |
| 세션 흐름 이상 탐지 | Anomaly Transformer           | 세션 흐름에서 발생하는 시계열 이상 탐지       |
| 이상 단말 확률 평가 | Naive Bayes                   | 개별 단말 이상 행위 확률적 평가 및 위험도 순위화 |

#### 📌 데이터 흐름도:

아래 흐름은 데이터 수집부터 이상 탐지, 대응까지의 전체 프로세스를 간략히 나타냅니다:

```plaintext
[네트워크 트래픽 수집]
        │
        ▼
[실시간 데이터 전처리]
(세션 데이터, 포트 통계 데이터 추출)
        │
        ▼
┌──────────────────────────────────────────┐
│        [이상 탐지 앙상블]                │
│ ┌────────────────────────────────┐       │
│ │    1. VAE (세션 이상 탐지)     │       │
│ │    2. Poisson (접근 패턴 분석) │       │
│ │    3. Transformer (세션 흐름)  │       │
│ └────────────────────────────────┘       │
│              │ 결과 통합(확률 가중치적용) │
│              ▼                           │
│         [4. Naive Bayes]                 │
│ (단말 이상 확률 계산 및 순위화)           │
└──────────────────────────────────────────┘
        │
        ▼
[관리자 알림 및 자동 대응]
(이상 단말 차단, 관리자 경고 등)
```

**📌 Note(그림4)**

『A Real-time Multidimensional Data-driven Probabilistic Network Anomaly Detection Framework』

 ![image](https://github.com/user-attachments/assets/fd4fdaa4-6884-4052-a352-b013cf90ff2e)















---

## 🔹 VAE 기반 세션 이상 탐지

- **VAE (Variational Autoencoder) 소개**
- **VAE의 사용 목적과 이유**
- **데이터 전처리 및 잠재 벡터(Latent Vector) 변환 방법**

**예시 코드 삽입**:
```python
import pandas as pd
import numpy as np
import tensorflow as tf
from tensorflow.keras.layers import Dense, Flatten, Reshape, BatchNormalization, Dropout
from tensorflow.keras.models import Model
import matplotlib.pyplot as plt
from sklearn.manifold import TSNE

# 데이터 스케일링 및 전처리
data_scaling = merged_df.astype('float32')
data = data_scaling.apply(min_max_scaling)

class DenseBlock(Model):
    def __init__(self, units, dropout_rate=0.3):
        super(DenseBlock, self).__init__()
        self.dense = Dense(units, activation=tf.nn.relu)
        self.bn = BatchNormalization()
        self.dropout = Dropout(dropout_rate)

    def call(self, x):
        x = self.dense(x)
        x = self.bn(x)
        return self.dropout(x)

class Encoder(Model):
    def __init__(self, latent_dim=10):
        super(Encoder, self).__init__()
        self.latent_dim = latent_dim
        self.dense_block1 = DenseBlock(256, dropout_rate=0.3)
        self.dense_block2 = DenseBlock(128, dropout_rate=0.3)
        self.dense_block3 = DenseBlock(64, dropout_rate=0.3)  # 추가 계층
        self.dense_output = Dense(units=self.latent_dim, activation=None)

    def call(self, x):
        x = self.dense_block1(x)
        x = self.dense_block2(x)
        x = self.dense_block3(x)  # 추가 계층 호출
        return self.dense_output(x)

class Decoder(Model):
    def __init__(self, latent_dim=10):
        super(Decoder, self).__init__()
        self.dense_block1 = DenseBlock(64, dropout_rate=0.3)
        self.dense_block2 = DenseBlock(128, dropout_rate=0.3)
        self.dense_block3 = DenseBlock(256, dropout_rate=0.3)  # 추가 계층
        self.dense_output = Dense(units=86, activation='sigmoid')  # 데이터에 따라 조정

    def call(self, x):
        x = self.dense_block1(x)
        x = self.dense_block2(x)
        x = self.dense_block3(x)  # 추가 계층 호출
        return self.dense_output(x)

class VAE(Model):
    def __init__(self, latent_dim=10):
        super(VAE, self).__init__()
        self.latent_dim = latent_dim
        self.encoder = Encoder(latent_dim=self.latent_dim)
        self.decoder = Decoder(latent_dim=self.latent_dim)
        self.dense_mu = Dense(units=self.latent_dim, activation=None)
        self.dense_log_var = Dense(units=self.latent_dim, activation=None)

    def reparameterize(self, mu, logvar):
        eps = tf.random.normal(shape=mu.shape)
        return eps * tf.exp(logvar * 0.5) + mu

    def call(self, x):
        x = self.encoder(x)
        mu = self.dense_mu(x)
        log_var = self.dense_log_var(x)
        z = self.reparameterize(mu, log_var)
        x_recon = self.decoder(z)
        return x_recon, mu, log_var
    
    def encode(self, x):
        x = self.encoder(x)
        return self.dense_mu(x)

# 손실 함수 및 학습 단계 정의
def compute_loss(data, reconstruction, mu, log_var):
    epsilon = 1e-7  # Small constant for numerical stability
    recon_loss = tf.reduce_mean(tf.reduce_sum(tf.keras.losses.mean_squared_error(data, reconstruction), axis=-1))
    kl_loss = -0.5 * (1 + log_var - tf.square(mu) - tf.exp(log_var + epsilon))
    kl_loss = tf.reduce_mean(tf.reduce_sum(kl_loss, axis=1))
    total_loss = recon_loss + kl_loss
    return total_loss, recon_loss, kl_loss

@tf.function
def train_step(model, optimizer, data):
    with tf.GradientTape() as tape:
        reconstruction, mu, log_var = model(data)
        data = tf.reshape(data, shape=(data.shape[0], -1))  # Ensuring shapes match
        reconstruction = tf.reshape(reconstruction, shape=(reconstruction.shape[0], -1))
        total_loss, recon_loss, kl_loss = compute_loss(data, reconstruction, mu, log_var)
    gradients = tape.gradient(total_loss, model.trainable_variables)
    optimizer.apply_gradients(zip(gradients, model.trainable_variables))
    return total_loss, recon_loss, kl_loss

# 모델 및 옵티마이저 초기화
latent_dim = 4  # 잠재 차원 크기 조정 가능
vae = VAE(latent_dim=latent_dim)

# 학습률 스케줄러 설정
initial_learning_rate = 1e-4
lr_schedule = tf.keras.optimizers.schedules.ExponentialDecay(
    initial_learning_rate, decay_steps=100000, decay_rate=0.96, staircase=True)
optimizer = tf.keras.optimizers.Adam(learning_rate=lr_schedule)

# 데이터셋 준비 및 학습 루프
dataset = tf.data.Dataset.from_tensor_slices(data.values).shuffle(len(data)).batch(64)

# 학습 루프
num_epochs = 100  # 조정 가능
for epoch in range(num_epochs):
    for step, batch_data in enumerate(dataset):
        total_loss, recon_loss, kl_loss = train_step(vae, optimizer, batch_data)
    print(f"Epoch: {epoch+1}, Total Loss: {total_loss.numpy()}, Recon Loss: {recon_loss.numpy()}, KL Loss: {kl_loss.numpy()}")

# 잠재 공간 시각화
def visualize_latent_space(model, data, num_samples=1000):
    subset = data.sample(n=num_samples, random_state=42)  # 무작위 샘플 선택

    mu = model.encode(subset).numpy()
    
    tsne = TSNE(n_components=2, perplexity=30, n_iter=300)
    mu_tsne = tsne.fit_transform(mu)

    plt.figure(figsize=(10, 8))
    plt.scatter(mu_tsne[:, 0], mu_tsne[:, 1], alpha=0.5)
    plt.title('Latent Space Representation using t-SNE')
    plt.xlabel('Dimension 1')
    plt.ylabel('Dimension 2')
    plt.show()

# visualize_latent_space(vae, data)
```

---

## 🔹 서버 접근 패턴 기반 이상 탐지 (Poisson 분포 활용)

- **Poisson 분포 개념과 도입 이유**
- **접근 빈도 이상 탐지 방법**
- **λ(람다) 값 산출 및 분석 방법**

**예시 코드 삽입**:
```python
import pandas as pd
import numpy as np
from scipy.stats import poisson

# 예시 데이터 생성
# data = read_csv(...)

# 데이터 프레임으로 변환 및 전처리
df = pd.DataFrame(data)
df['@timestamp'] = pd.to_datetime(df['@timestamp'], unit='ms')
df['source_ip'] = df['source'].apply(lambda x: x['ip'])
df['destination_ip'] = df['destination'].apply(lambda x: x['ip'])
df = df.drop(columns=['source', 'destination'])
df['hour'] = df['@timestamp'].dt.hour

# 시간대별 방문 빈도 계산
visits_per_hour = df.groupby(['source_ip', 'destination_ip', 'hour']).size().reset_index(name='visits')

# 시간대별 평균 방문 빈도 계산
average_visits_per_hour = visits_per_hour.groupby(['source_ip', 'destination_ip', 'hour'])['visits'].mean().reset_index(name='average_visits')

# 푸아송 분포를 사용하여 상위 95% 임계치 계산 함수
def calculate_threshold(avg_visits):
    return poisson.ppf(0.99, avg_visits)

# 시간대별 상위 99% 임계치 계산
average_visits_per_hour['threshold'] = average_visits_per_hour['average_visits'].apply(calculate_threshold)

```

---

## 🔹 세션 흐름 기반 이상 탐지 (Anomaly Transformer)

- **Anomaly Transformer 모델 소개**
- **Association Discrepancy 개념 설명**
- **세션 흐름 데이터의 모델 학습 및 이상 탐지 과정**


Anomaly-Transformer (ICLR 2022 Spotlight)

https://github.com/thuml/Anomaly-Transformer


---

## 🔹 스위치 포트 통계 데이터 기반 이상 탐지 (Anomaly Transformer)

- **스위치 포트 통계 데이터의 중요성**
- **다변량 시계열 데이터 분석 방법**
- **이상 징후 판단 기준 및 탐지 방식**

Anomaly-Transformer (ICLR 2022 Spotlight)

https://github.com/thuml/Anomaly-Transformer

---

## 🔹 나이브 베이즈를 이용한 이상 단말 확률적 판단

- **나이브 베이즈 모델 소개 및 선택 이유**
- **단말 이상 확률 계산 방법 및 공식**
- **로그 확률 변환 및 소프트맥스 활용 설명**

**예시 코드 삽입**:
```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from scipy.special import softmax

# 가중치(감마) 설정
gamma_poisson_vae = 0.5  # VAE와 POISSON에 적용할 가중치
gamma_sw_vae_mus = 1.0   # SW와 VAE_MUS에 적용할 가중치

# 파일 경로 설정 및 CSV 파일 불러오기
vae_df = pd.read_csv('NaiveBayesResults/VAE.csv', index_col=0)  # VAE.csv 파일 불러오기
poisson_df = pd.read_csv('NaiveBayesResults/POISSON.csv', index_col=0)  # POISSON.csv 파일 불러오기
sw_df = pd.read_csv('NaiveBayesResults/SW.csv', index_col=0)  # SW.csv 파일 불러오기
vae_mus_df = pd.read_csv('NaiveBayesResults/VAE_MUS.csv', index_col=0)  # VAE_MUS.csv 파일 불러오기

# 각 데이터프레임의 크기 확인 (디버깅용 출력)
print("VAE shape:", vae_df.shape)
print("POISSON shape:", poisson_df.shape)
print("SW shape:", sw_df.shape)
print("VAE_MUS shape:", vae_mus_df.shape)

# 각 데이터프레임의 인덱스 (IP 주소) 확인 (디버깅용 출력)
print("VAE IPs:", vae_df.index)
print("POISSON IPs:", poisson_df.index)
print("SW IPs:", sw_df.index)
print("VAE_MUS IPs:", vae_mus_df.index)

# IP 주소 중복 또는 누락 확인을 위한 통합 인덱스 비교
common_ips = vae_df.index.intersection(poisson_df.index).intersection(sw_df.index).intersection(vae_mus_df.index)

# 만약 공통 IP 개수가 예상보다 적을 경우 (9개인 경우), 문제 해결을 위해 결측값 처리 수행
# 이 경우 각 데이터프레임을 공통 IP로 맞추지 않고, 모든 IP를 포함하도록 결측값을 채우는 방법을 사용
combined_ips = vae_df.index.union(poisson_df.index).union(sw_df.index).union(vae_mus_df.index)

# 각 데이터프레임의 결측값을 0으로 채워서 모든 IP를 포함하도록 처리
vae_df_numeric = vae_df.reindex(combined_ips).fillna(0)
poisson_df_numeric = poisson_df.reindex(combined_ips).fillna(0)
sw_df_numeric = sw_df.reindex(combined_ips).fillna(0)
vae_mus_df_numeric = vae_mus_df.reindex(combined_ips).fillna(0)

# 데이터 가중치 적용
vae_df_weighted = vae_df_numeric * gamma_poisson_vae  # VAE 데이터에 가중치 적용
poisson_df_weighted = poisson_df_numeric * gamma_poisson_vae  # POISSON 데이터에 가중치 적용
sw_df_weighted = sw_df_numeric * gamma_sw_vae_mus  # SW 데이터에 가중치 적용
vae_mus_df_weighted = vae_mus_df_numeric * gamma_sw_vae_mus  # VAE_MUS 데이터에 가중치 적용

# 네 개의 파일을 합산하여 하나의 DataFrame 생성 (숫자 데이터만)
combined_df_numeric = vae_df_weighted + poisson_df_weighted + sw_df_weighted + vae_mus_df_weighted

# 데이터의 크기 확인 (디버깅용 출력)
print("Combined data shape:", combined_df_numeric.shape)  # (10, 200)인지 확인

# Softmax 및 Z-score 정규화를 사용한 확률 계산 함수
def calculate_initial_probabilities(data):
    if np.all(data == data[0]):  # 만약 모든 값이 동일하다면
        return np.ones_like(data) / len(data)  # 동일한 확률 분포
    normalized_scores = (data - np.mean(data)) / (np.std(data) + 1e-10)  # Z-score 정규화
    probabilities = softmax(normalized_scores)  # softmax로 확률 계산
    return probabilities

# 나이브 베이즈 업데이트 함수 (로그 공간에서 계산)
def bayesian_update_log(prior_log, likelihood):
    likelihood = np.clip(likelihood, 1e-10, 1.0)  # 확률 범위를 1e-10로 제한하여 log(0) 방지
    posterior_log = prior_log + np.log(likelihood)  # 로그 확률을 더하여 posterior 계산
    posterior_log -= np.max(posterior_log)  # overflow 방지를 위해 최대값을 빼줌
    posterior = np.exp(posterior_log)  # posterior 값을 확률로 변환
    return posterior / np.sum(posterior)  # 정규화

# 시간대별로 나이브 베이즈 업데이트를 수행하는 함수
def update_probabilities_over_time(data_df):
    probability_history = []  # 확률 기록을 저장할 리스트
    
    # 첫 번째 시간대의 데이터를 기반으로 초기 확률 계산
    initial_data = data_df.iloc[0]
    initial_probabilities = calculate_initial_probabilities(initial_data)
    
    # 초기 확률을 log-space로 변환
    prior_log = np.log(initial_probabilities + 1e-10)

    # 각 시간대별로 나이브 베이즈 업데이트 수행
    for t in range(1, len(data_df)):
        new_data = data_df.iloc[t]  # 새로운 데이터 추출
        likelihood = calculate_initial_probabilities(new_data)  # 새로운 데이터의 가능도 계산
        prior_log = bayesian_update_log(prior_log, likelihood)  # 나이브 베이즈 업데이트
        probability_history.append(prior_log)  # 업데이트된 확률 기록
    
    return np.array(probability_history)  # 확률 기록 반환

# 나이브 베이즈 업데이트 실행 (시간대별로 계산)
updated_probabilities = update_probabilities_over_time(combined_df_numeric.T)

# 확률 업데이트 히스토리를 전치하여 IP별로 행, 시간별로 열이 되도록 변환
updated_probabilities = updated_probabilities.T

# 데이터의 크기 확인 (디버깅용 출력)
print("Updated probabilities shape:", updated_probabilities.shape)  # (10, 200)인지 확인

# 시각화 설정
plt.figure(figsize=(60, 10))  # 그래프 크기 설정
sns.set(font_scale=0.7)  # 글꼴 크기 조정

# xticklabels의 뒤 두 자리만 표시 (00 ~ 99로 반복)
xticklabels = [f'{i % 100}' for i in range(updated_probabilities.shape[1])]

# 히트맵 시각화
ax = sns.heatmap(updated_probabilities, xticklabels=xticklabels, 
                 yticklabels=combined_ips, cmap='YlOrRd', vmin=0, vmax=1, 
                 cbar_kws={'label': 'Probability', 'shrink': 0.4}, 
                 linewidths=0.01, linecolor='black', square=True, annot=True, fmt=".2f", annot_kws={"size": 6})

plt.title('Naive Bayesian Update of Probabilities over Time')
plt.xlabel('Time')
plt.ylabel('IP Address')
plt.tight_layout()

plt.show()

```
---


![image](https://github.com/user-attachments/assets/8a40318e-fd02-49b3-8957-2757f669feaf)

---

## 🔹 앙상블 기반 탐지 및 최종 판단

- **앙상블 기법의 필요성 및 기대 효과**
- **각 모델의 가중치 적용 방법 (Gamma 값 활용)**
- **최종 모델의 탐지 성능 평가 및 활용 가능성**

---

## 📚 참고 문헌 및 관련 자료

- Xu, Jiehui et al. (2022). *Anomaly Transformer: Time Series Anomaly Detection with Association Discrepancy*
