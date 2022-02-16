---
lab:
  title: '랩: Windows Server 모니터링 및 문제 해결'
  module: 'Module 8: Monitoring, performance, and troubleshooting'
ms.openlocfilehash: be85a4a949d34c194ba76cd56cbb81f350707685
ms.sourcegitcommit: 9a51ea796ef3806ab9e7ec1ff75034b2f929ed2a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/28/2022
ms.locfileid: "137907127"
---
# <a name="lab-monitoring-and-troubleshooting-windows-server"></a>랩: Windows Server 모니터링 및 문제 해결

## <a name="scenario"></a>시나리오

Contoso, Ltd.는 미국 워싱턴 주 시애틀에 본사가 있는 글로벌 엔지니어링 및 제조 회사입니다. IT 사무실과 데이터 센터는 시애틀에서 시애틀 위치 및 기타 위치를 지원합니다. Contoso는 최근 Windows 서버와 클라이언트 인프라를 배포했습니다.

조직에서 새 서버를 배포했기 때문에 이러한 새 서버에 대한 일반적인 부하를 사용하여 성능 기준을 설정하는 것이 중요합니다. 이 프로젝트에 참여하라는 요청을 받았습니다. 또한 모니터링 및 문제 해결 프로세스를 더 쉽게 수행하기 위해 이벤트 로그의 중앙 집중식 모니터링을 수행하기로 결정했습니다.

## <a name="objectives"></a>목표

이 랩을 완료하면 다음을 수행할 수 있습니다.

- 성능 기준선을 설정합니다.
- 성능 문제 원인 식별
- 중앙 집중식 이벤트 로그 검토 및 구성

## <a name="estimated-time-40-minutes"></a>예상 시간: **40분**

## <a name="lab-setup"></a>랩 설정

가상 머신: **AZ-801T00A-SEA-DC1** 및 **AZ-801T00A-SEA-SVR2** 를 실행해야 합니다. 다른 VM을 실행할 수도 있지만 이 랩에서는 필요하지 않습니다.

> **참고**: **AZ-801T00A-SEA-DC1**, **AZ-801T00A-SEA-SVR2** 가상 머신은 **SEA-DC1** 및 **SEA-SVR2** 의 설치를 호스팅합니다.

1. **SEA-SVR2** 를 선택합니다.
1. 다음 자격 증명을 사용하여 로그인:

   - 사용자 이름: **Administrator**
   - 암호: **Pa55w.rd**
   - 도메인: **CONTOSO**

이 랩에서는 사용 가능한 VM 환경을 사용합니다.

## <a name="exercise-1-establishing-a-performance-baseline"></a>연습 1: 성능 기준 설정

### <a name="scenario"></a>시나리오

이 연습에서는 서버의 성능 모니터를 사용하고 일반적인 성능 카운터를 사용하여 기준을 만듭니다.

이 연습의 주요 작업은 다음과 같습니다.

1. 데이터 수집기 집합 만들기 및 시작
1. 서버에서 일반적인 워크로드 만들기
1. 수집된 데이터 분석

> **참고**: 데이터 수집기 집합을 시작한 후 결과가 나타나기까지 10분이 지연될 수 있습니다.

#### <a name="task-1-create-and-start-a-data-collector-set"></a>작업 1: 데이터 수집기 집합 만들기 및 시작

1. **SEA-SVR2** 에서 성능 모니터를 엽니다.
1. 성능 모니터를 사용하여 다음 설정을 사용하여 새 **사용자 정의** 데이터 수집기 집합을 만듭니다.

   - 이름: **SEA-SVR2 성능**
   - 만들기: **수동으로 만들기(고급)**
   - 데이터 형식: **성능 카운터**
   - 성능 카운터(기본 인스턴스 옵션 사용):

      - **Memory\\Pages/sec**
      - **Network Interface\\Bytes Total/sec**
      - **PhysicalDisk\\% Disk Time**
      - **PhysicalDisk\\Avg. Disk Queue Length**
      - **Processor\\% Processor Time**
      - **System\\Processor Queue Length**

   - 샘플 간격: **1** 초

1. 성능 모니터에서 **SEA-SVR2 성능** 데이터 수집기를 시작합니다. 

#### <a name="task-2-create-a-typical-workload-on-the-server"></a>작업 2: 서버에서 일반적인 워크로드 만들기

1. **SEA-SVR2** 에서 관리자 권한으로 Windows PowerShell을 시작합니다.
1. 일반적인 워크로드를 에뮬레이트하려면 Windows PowerShell 명령 프롬프트에서 다음 명령을 실행합니다.


   ```powershell
   fsutil file createnew bigfile 104857600
   Copy-Item -Path .\bigfile -Destination \\SEA-DC1.contoso.com\c$\ -Force
   Copy-Item -Path \\SEA-DC1.contoso.com\c$\bigfile -Destination .\bigfile2 -Force
   Remove-Item -Path .\bigfile* -Force
   Remove-Item -Path \\SEA-DC1.contoso.com\c$\bigfile -Force
   ```

1. Windows PowerShell 창을 열어 둡니다.

#### <a name="task-3-analyze-the-collected-data"></a>작업 3: 수집된 데이터 분석

1. **SEA-SVR2** 에서 성능 모니터로 전환합니다.
1. **SEA-SVR2 성능** 데이터 수집기 집합을 중지합니다.
1. 성능 모니터의 탐색 창에서 **보고서**, **사용자 정의**, **SEA-SVR2**, **SEA-SVR2\_DateTime-000001** 을 찾아본 다음, 보고서 보기를 사용하여 보고서 데이터를 검토합니다.
1. 나중에 분석할 수 있도록 보고서에 나열된 값을 기록합니다. 기록된 값은 다음과 같습니다.

   - **Memory\\Pages/sec**
   - **Network Interface\\Bytes Total/sec**
   - **PhysicalDisk\\% Disk Time**
   - **PhysicalDisk\\Avg. Disk Queue Length**
   - **Processor\\% Processor Time**
   - **System\\Processor Queue Length**

### <a name="results"></a>결과

이 연습 후에는 성능 비교를 위한 기준을 설정해야 합니다.

## <a name="exercise-2-identifying-the-source-of-a-performance-problem"></a>연습 2: 성능 문제 원인 식별

### <a name="scenario"></a>시나리오

이 연습에서는 부하를 시뮬레이션하여 실시간 사용 중인 시스템을 나타내고 데이터 수집기 집합을 사용하여 성능 데이터를 수집한 다음, 성능 문제의 잠재적 원인을 확인합니다.

이 연습의 주요 작업은 다음과 같습니다.

1. 서버에서 추가 워크로드 만들기
1. 데이터 수집기 집합을 사용하여 성능 데이터 캡처
1. 워크로드 제거 및 성능 데이터 검토

#### <a name="task-1-create-additional-workload-on-the-server"></a>작업 1: 서버에서 추가 워크로드 만들기

1. **SEA-SVR2** 에서 파일 탐색기를 엽니다.
1. 파일 탐색기에서 **C:\Labfiles\Lab08** 을 찾아보고 **CPUSTRES64** 를 시작합니다.
1. 첫 번째 강조 표시된 작업을 구성하여 **BUSY(75%)** 를 실행합니다.

   > **참고**: **CPUSTRES64.EXE** 는 루프에서 최대 64개의 스레드를 실행하여 CPU 작업을 시뮬레이션하는 데 사용할 수 있는 SysInternals 유틸리티입니다.

#### <a name="task-2-capture-performance-data-by-using-a-data-collector-set"></a>작업 2: 데이터 수집기 집합을 사용하여 성능 데이터 캡처

1. **SEA-SVR2** 에서 성능 모니터로 전환합니다.
1. 성능 모니터에서 **데이터 수집기 집합**, **사용자 정의** 를 찾아본 다음, 결과 창에서 **SEA-SVR2 성능** 데이터 수집기 집합을 시작합니다.

   > **참고**: 데이터 캡처가 수행되도록 1분 정도 기다리세요.

#### <a name="task-3-remove-the-workload-and-review-the-performance-data"></a>작업 3: 워크로드 제거 및 성능 데이터 검토

1. **SEA-SVR2** 에서 CPUSTRES64를 닫습니다.
1. 성능 모니터로 전환합니다.
1. **SEA-SVR2 성능** 데이터 수집기 집합을 중지합니다.
1. 성능 모니터의 탐색 창에서 **보고서**, **사용자 정의**, **SEA-SVR2**, **SEA-SVR2\_DateTime-000002** 를 찾아본 다음, 보고서를 검토합니다. 
1. 보고서에 나열된 값을 기록합니다.

   - **Memory\\Pages/sec**
   - **Network Interface\\Bytes Total/sec**
   - **PhysicalDisk\\% Disk Time**
   - **PhysicalDisk\\Avg. Disk Queue Length**
   - **Processor\\% Processor Time**
   - **System\\Processor Queue Length**

### <a name="results"></a>결과

이 연습 후에는 성능 도구를 사용하여 잠재적인 성능 병목 상태를 식별해야 합니다.

### <a name="exercise-3-viewing-and-configuring-centralized-event-logs"></a>연습 3: 중앙 집중식 이벤트 로그 보기 및 구성

### <a name="scenario"></a>시나리오

이 연습에서는 **SEA-SVR2** 를 사용하여 **SEA-DC1** 에서 애플리케이션 이벤트 로그 항목을 수집합니다. 

이 연습의 주요 작업은 다음과 같습니다.

1. 구독 필수 구성 요소 구성
1. 구독 만들기 및 결과 확인

#### <a name="task-1-configure-subscription-prerequisites"></a>작업 1: 구독 필수 구성 요소 구성

1. **SEA-SVR2** 에서 Windows PowerShell로 전환합니다.
1. **SEA-SVR2** 로 전달된 이벤트의 구독을 만들고 관리할 수 있으려면 다음 명령을 실행합니다.

   ```powershell
   WECUtil qc /q
   ```

1. 이벤트 원본과 수집기의 로컬 날짜와 시간을 동기화하려면 다음 명령을 실행합니다.

   ```powershell
   w32tm /resync /computer:SEA-DC1.contoso.com
   ```

1. Kerberos 인증 문제가 발생하는 경우 WinRM 연결을 허용하려면 다음 명령을 실행합니다.

   ```powershell
   Set-Item WSMan:localhost\client\trustedhosts -Value *.contoso.com -Force
   ```

1. **SEA-SVR1** 에 대한 PowerShell 원격 세션을 설정하려면 다음을 실행합니다.

   ```powershell
   Enter-PSSession -ComputerName SEA-DC1.contoso.com
   ```

1. **SEA-DC1** 에서 WinRM(Windows 원격 관리)이 사용하도록 설정되어 있는지 확인하려면 다음 명령을 실행합니다.

   ```powershell
   winrm qc
   ```

   > **참고**: WinRM 서비스가 이미 실행 중이고 원격 관리를 위해 설정되어 있는지 확인하세요.

1. **SEA-DC1** 에서 고급 보안 규칙이 포함된 관련 Windows Defender 방화벽이 사용하도록 설정되어 있는지 확인하려면 다음 명령을 실행합니다.

   ```powershell
   Set-NetFirewallRule -DisplayGroup 'Remote Event Log Management' -Enabled True -Profile Domain -PassThru
   ```

   > **참고**: Windows PowerShell 창을 열어 둡니다.

1. **SEA-SVR2** 에서 **Active Directory 사용자 및 컴퓨터** 를 엽니다.
1. **Active Directory 사용자 및 컴퓨터** 콘솔에서 **SEA-SVR2** 컴퓨터 계정을 **이벤트 로그 판독기** 그룹에 추가합니다.

#### <a name="task-2-create-a-subscription-and-verify-the-results"></a>작업 2: 구독 만들기 및 결과 확인

1. **SEA-SVR2** 에서 **이벤트 뷰어** 를 엽니다.
1. 다음 속성을 사용하여 새 구독을 만듭니다.

   - 이름: **SEA-DC1 이벤트**
   - 컴퓨터: **SEA-DC1**
   - 수집기: **시작됨**
   - 이벤트: **위험**, **경고**, **정보**, **오류**
   - 기록됨: **지난 24시간**
   - 로그: **애플리케이션**, **시스템** **Windows 로그**

1. **SEA-SVR2** 에서 **이벤트 뷰어** 창으로 전환한 후 탐색 창에서 **Windows 로그** 를 확장합니다.
1. **Forwarded Events** 를 선택하고 전달된 이벤트에 **SEA-DC1** 에서 생성된 이벤트가 포함되어 있는지 확인합니다.