# OCI A1 Recovery via GitHub Actions

GitHub Actions(Public Repository 무료 Runner)를 활용하여 Oracle Cloud Infrastructure (OCI) ARM(A1.Flex) 인스턴스를 지속적으로 생성/복구 시도하는 워크플로우입니다.

## 동작 방식
- **3시간 주기 스케줄 (`cron`)**: UTC 기준 매 3시간마다 GitHub Runner가 실행됩니다.
- **워크플로우 내부 루프**: 1회 실행당 60초 간격으로 최대 170회(약 170분 = 2시간 50분) 재시도합니다.
- **24/7 지속 시도**: 한 실행이 끝나면 다음 3시간 주기 스케줄이 바통을 이어받아 거의 빈틈없이 24시간 재시도합니다.
- **성공 시 자동 비활성화**: A1 인스턴스가 성공적으로 생성되면 워크플로우를 스스로 비활성화(`disable`)하여 더 이상 불필요한 러너가 돌지 않습니다.

---

## 1. GitHub Secrets 등록

GitHub 레포지토리의 **Settings** → **Secrets and variables** → **Actions** → **Repository secrets**에 다음 항목들을 등록합니다:

| Secret Name | 설명 | 필수 여부 |
| :--- | :--- | :---: |
| `OCI_USER_OCID` | OCI 사용자 OCID (`ocid1.user...`) | 필수 |
| `OCI_TENANCY_OCID` | OCI 테넌시 OCID (`ocid1.tenancy...`) | 필수 |
| `OCI_FINGERPRINT` | OCI API Key Fingerprint | 필수 |
| `OCI_PRIVATE_KEY` | OCI API Private Key 내용 전체 (`-----BEGIN RSA PRIVATE KEY...`) | 필수 |
| `BOOT_VOLUME_ID` | 복구/연결할 Boot Volume OCID (`ocid1.bootvolume...`) | 필수 |
| `COMPARTMENT_ID` | 인스턴스가 생성될 구획 OCID (`ocid1.compartment...` 또는 테넌시 OCID) | 필수 |
| `SUBNET_ID` | VCN 서브넷 OCID (`ocid1.subnet...`) | 필수 |
| `NSG_ID` | 네트워크 보안 그룹 OCID (사용하는 경우에만 입력) | 선택 |
| `OCI_REGION` | 리전 식별자 (미입력 시 기본값 `ap-seoul-1`) | 선택 |

---

## 2. GitHub Actions 권한 설정 (중요)

워크플로우가 성공 시 스스로 비활성화(`disable`)할 수 있도록 권한을 부여해야 합니다:
1. 레포지토리 **Settings** 이동
2. 좌측 메뉴 **Actions** → **General** 클릭
3. **Workflow permissions** 섹션에서 **Read and write permissions** 선택 후 Save

---

## 3. 실행 방법

1. 위 Secrets 및 권한 설정 완료 후 레포지토리에 커밋 & 푸시
2. GitHub의 **Actions** 탭 → **OCI A1 Recovery** 워크플로우 클릭
3. **Run workflow** 버튼을 눌러 즉시 시작하거나 스케줄에 의해 자동 실행 대기

> **참고**: GitHub 정책상 커밋/푸시 등의 활동이 60일 동안 전혀 없는 레포지토리는 scheduled workflow가 일시 정지될 수 있습니다. 장기 실행 시 간혹 상태를 확인해 주세요.
