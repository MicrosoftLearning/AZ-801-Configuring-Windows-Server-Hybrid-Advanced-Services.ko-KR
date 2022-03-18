---
lab:
  title: '랩: Windows Server 모니터링 및 문제 해결'
  type: Answer Key
  module: 'Module 8: Monitoring, performance, and troubleshooting'
ms.openlocfilehash: 5067b69c33fa81f9ba2d7d4efb10b717fe11924f
ms.sourcegitcommit: 9a51ea796ef3806ab9e7ec1ff75034b2f929ed2a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/28/2022
ms.locfileid: "137907168"
---
# <a name="lab-answer-key-monitoring-and-troubleshooting-windows-server"></a>랩 응답 키: Windows Server 모니터링 및 문제 해결

### <a name="exercise-1-establishing-a-performance-baseline"></a>연습 1: 성능 기준 설정

> **참고**: 데이터 수집기 집합을 시작한 후 결과가 나타나기까지 10분이 지연될 수 있습니다.

#### <a name="task-1-create-and-start-a-data-collector-set"></a>작업 1: 데이터 수집기 집합 만들기 및 시작

1. **SEA-SVR2** 에 연결한 후 필요한 경우 **Pa55w.rd** 암호를 이용해 **CONTOSO\\Administrator** 로 로그인합니다.
1. **SEA-SVR2** 에서 **시작** 단추 옆에 있는 **검색하려면 여기에 입력하십시오.** 텍스트 상자에 **성능** 을 입력한 후 **가장 정확** 목록에서 **성능 모니터** 를 선택합니다.
1. 성능 모니터의 탐색 창에서 **데이터 수집기 집합** 노드를 확장한 후 **사용자 정의** 를 선택합니다.
1. **사용자 정의** 를 마우스 오른쪽 단추로 클릭하거나 해당 상황에 맞는 메뉴에 액세스하고 **새로 만들기** 를 선택한 후 **데이터 수집기 집합** 을 선택합니다. 그러면 **새 데이터 수집기 집합 만들기** 마법사가 시작됩니다.
1. **새 데이터 수집기 집합 만들기** 마법사의 **이 새 데이터 수집기 집합을 어떻게 만드시겠습니까?** 페이지에서 **이름** 상자에 **SEA-SVR2 Performance** 를 입력합니다.
1. **수동으로 만들기(고급)** 옵션을 선택한 후 **다음** 을 선택합니다.
1. **어떤 형식의 데이터를 포함하시겠습니까?** 페이지에서 **성능 카운터** 확인란을 선택한 후 **다음** 을 선택합니다.
1. **어떤 성능 카운터를 기록하시겠습니까?** 페이지에서 **추가** 를 선택합니다.
1. **사용 가능한 카운터** 목록에서 **Processor** 를 확장하고 **% Processor Time** 을 선택한 후 **선택한 개체의 인스턴스** 섹션에서 **_Total** 항목이 선택되어 있는지 확인하고 **추가** 를 선택합니다.
1. **사용 가능한 카운터** 목록에서 **Memory** 를 확장하고 **Pages/sec** 를 선택한 후 **추가** 를 선택합니다.
1. **사용 가능한 카운터** 목록에서 **PhysicalDisk** 를 확장하고 **% Disk Time** 을 선택한 후 **선택한 개체의 인스턴스** 섹션에서 **_Total** 항목이 선택되어 있는지 확인하고 **추가** 를 선택합니다.
1. **Avg. Disk Queue Length** 를 선택하고 **선택한 개체의 인스턴스** 섹션에서 **_Total** 항목이 선택되어 있는지 확인한 후 **추가** 를 선택합니다.
1. **사용 가능한 카운터** 목록에서 **System** 을 확장하고 **Processor Queue Length** 를 선택한 후 **추가** 를 선택합니다.
1. **사용 가능한 카운터** 목록에서 **Network Interface** 를 확장하고 **Bytes Total/sec** 를 선택한 후 **선택한 개체의 인스턴스** 섹션에서 **<All instances>** 항목이 선택되어 있는지 확인하고 **추가** 를 선택한 후 **확인** 을 선택합니다.
1. **어떤 성능 카운터를 기록하시겠습니까?** 페이지에서 **샘플 간격** 상자에 **1** 을 입력한 후 **다음** 을 선택합니다.
1. **데이터를 어디에 저장하시겠습니까?** 페이지에서 **다음** 을 선택합니다.
1. **데이터 수집기 집합을 만드시겠습니까?** 페이지에서 **저장 후 닫기** 옵션이 선택되어 있는지 확인하고 **마침** 을 선택합니다.
1. 성능 모니터의 결과 창에서 **SEA-SVR2 Performance** 를 마우스 오른쪽 단추로 클릭하거나 해당 상황에 맞는 메뉴에 액세스한 후 **시작** 을 선택합니다.

#### <a name="task-2-create-a-typical-workload-on-the-server"></a>작업 2: 서버에서 일반적인 워크로드 만들기

1. **SEA-SVR2** 에서 **시작** 을 선택하고 Windows PowerShell의 상황에 맞는 메뉴를 마우스 오른쪽 단추로 클릭하거나 액세스한 다음, **관리자 권한으로 실행** 을 선택합니다.
1. 특정 크기의 파일을 만들려면 Windows PowerShell 명령 프롬프트에서 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   fsutil file createnew bigfile 104857600
   ```

1. 새로 만든 파일을 **SEA-DC1** 의 **C** 드라이브 루트에 복사하려면 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Copy-Item -Path .\bigfile -Destination \\SEA-DC1.contoso.com\c$\ -Force
   ```

1. **SEA-DC1** 의 **C** 드라이브 루트에 있는 파일을 현재 작업 디렉터리에 복사하려면 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Copy-Item -Path \\SEA-DC1.contoso.com\c$\bigfile -Destination .\bigfile2 -Force
   ```

1. 현재 작업 디렉터리에서 두 파일을 모두 삭제하려면 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Remove-Item -Path .\bigfile* -Force
   ```

1. **SEA-DC1** 의 **C** 드라이브 루트에 있는 파일을 삭제하려면 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Remove-Item -Path \\SEA-DC1.contoso.com\c$\bigfile -Force
   ```

1. Windows PowerShell 창을 열어 둡니다.

#### <a name="task-3-analyze-the-collected-data"></a>작업 3: 수집된 데이터 분석

1. **SEA-SVR2** 에서 성능 모니터로 전환합니다.
1. 탐색 창에서 **SEA-SVR2 Performance** 를 마우스 오른쪽 단추로 클릭하거나 해당 상황에 맞는 메뉴에 액세스한 후 **중지** 를 선택합니다.
1. 성능 모니터의 탐색 창에서 **보고서**, **사용자 정의**, **SEA-SVR2 Performance** 를 차례로 확장하고 **SEA-SVR2\_*DateTime*-000001**(여기서 *DateTime* 은 현재 타임스탬프를 지정함)을 선택한 후 보고서 데이터를 검토합니다.
1. 메뉴 모음에서 **그래프 형식 변경** 을 선택하거나 Ctrl+G를 누른 후 **보고서** 를 선택합니다.
1. 나중에 분석할 수 있도록 보고서에 나열된 값을 기록합니다. 기록된 값은 다음과 같습니다.

   - **Memory\\Pages/sec**
   - **Network Interface\\Bytes Total/sec**
   - **PhysicalDisk\\% Disk Time**
   - **PhysicalDisk\\Avg. Disk Queue Length**
   - **Processor\\% Processor Time**
   - **System\\Processor Queue Length**

### <a name="exercise-2-identifying-the-source-of-a-performance-problem"></a>연습 2: 성능 문제 원인 식별

#### <a name="task-1-create-additional-workload-on-the-server"></a>작업 1: 서버에서 추가 워크로드 만들기

1. **SEA-SVR2** 에서 파일 탐색기를 엽니다.
1. 파일 탐색기에서 **C:\Labfiles\Lab08** 로 이동합니다.
1. **CPUSTRES64.EXE** 를 두 번 클릭하거나 선택한 후 Enter 키를 누릅니다.

   > **참고**: **CPUSTRES64.EXE** 는 루프에서 최대 64개의 스레드를 실행하여 CPU 작업을 시뮬레이션하는 데 사용할 수 있는 SysInternals 유틸리티입니다.

1. **CPUSTRESS License Agreement**(CPUSTRESS 사용권 계약) 대화 상자에서 **Agree**(동의)를 선택합니다.
1. **CPU Stress**(CPU 스트레스) 대화 상자에서 실행 중인 스레드 목록 맨 위에 있는 강조 표시된 스레드를 마우스 오른쪽 단추로 클릭하거나 해당 상황에 맞는 메뉴에 액세스하고 **Activity Level**(작업 수준)을 선택한 후 **Busy (75%)** (사용률(75%))를 선택합니다.

#### <a name="task-2-capture-performance-data-by-using-a-data-collector-set"></a>작업 2: 데이터 수집기 집합을 사용하여 성능 데이터 캡처

1. **SEA-SVR2** 에서 성능 모니터로 전환합니다.
1. 성능 모니터에서 **데이터 수집기 집합** 을 확장하고 **사용자 정의** 를 선택합니다.
1. 결과 창에서 **SEA-SVR2 Performance** 를 마우스 오른쪽 단추로 클릭하거나 해당 상황에 맞는 메뉴에 액세스한 후 **시작** 을 선택합니다.

   > **참고**: 데이터 캡처가 수행되도록 1분 정도 기다리세요.

#### <a name="task-3-remove-the-workload-and-review-the-performance-data"></a>작업 3: 워크로드 제거 및 성능 데이터 검토

1. **SEA-SVR2** 에서 **CPU Stress**(CPU 스트레스) 대화 상자로 전환하고 이 대화 상자를 닫습니다. 
1. 성능 모니터로 전환합니다.
1. 탐색 창에서 **SEA-SVR2 Performance** 를 마우스 오른쪽 단추로 클릭하거나 해당 상황에 맞는 메뉴에 액세스한 후 **중지** 를 선택합니다.
1. 성능 모니터의 탐색 창에서 **보고서**, **사용자 정의**, **SEA-SVR2 Performance** 를 차례로 확장하고 **SEA-SVR2\_*DateTime*-000002**(여기서 *DateTime* 은 현재 타임스탬프를 지정함)를 선택한 후 보고서 데이터를 검토합니다.
1. 메뉴 모음에서 **그래프 형식 변경** 을 선택하거나 Ctrl+G를 누른 후 **보고서** 를 선택합니다.
1. 이전과 마찬가지로 보고서에 나열된 값을 기록합니다. 기록된 값은 다음과 같습니다.

   - **Memory\\Pages/sec**
   - **Network Interface\\Bytes Total/sec**
   - **PhysicalDisk\\% Disk Time**
   - **PhysicalDisk\\Avg. Disk Queue Length**
   - **Processor\\% Processor Time**
   - **System\\Processor Queue Length**

### <a name="exercise-3-viewing-and-configuring-centralized-event-logs"></a>연습 3: 중앙 집중식 이벤트 로그 보기 및 구성

#### <a name="task-1-configure-subscription-prerequisites"></a>작업 1: 구독 필수 구성 요소 구성

1. **SEA-SVR2** 에서 Windows PowerShell로 전환합니다.
1. **SEA-SVR2** 로 전달된 이벤트의 구독을 만들고 관리할 수 있으려면 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   WECUtil qc /q
   ```

1. 이벤트 원본과 수집기의 로컬 날짜와 시간을 동기화하려면 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   w32tm /resync /computer:SEA-DC1.contoso.com
   ```

1. Kerberos 인증 문제가 발생하는 경우 WinRM 연결을 허용하려면 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Set-Item WSMan:localhost\client\trustedhosts -Value *.contoso.com -Force
   ```

1. **SEA-DC1** 에 대한 PowerShell 원격 세션을 설정하려면 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Enter-PSSession -ComputerName SEA-DC1.contoso.com
   ```

1. **SEA-DC1** 에서 WinRM(Windows 원격 관리)이 사용하도록 설정되어 있는지 확인하려면 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   winrm qc
   ```

   > **참고**: WinRM 서비스가 이미 실행 중이고 원격 관리를 위해 설정되어 있는지 확인하세요.

1. **SEA-DC1** 에서 고급 보안 규칙이 포함된 관련 Windows Defender 방화벽이 사용하도록 설정되어 있는지 확인하려면 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Set-NetFirewallRule -DisplayGroup 'Remote Event Log Management' -Enabled True -Profile Domain -PassThru
   ```

   > **참고**: Windows PowerShell 창을 열어 둡니다.

1. **SEA-SVR2** 에서 **시작** 단추 옆에 있는 **검색하려면 여기에 입력하십시오.** 텍스트 상자에 **Active** 를 입력한 후 **가장 정확** 목록에서 **Active Directory 사용자 및 컴퓨터** 를 선택합니다.
1. **Active Directory 사용자 및 컴퓨터** 콘솔에서 **Builtin** 컨테이너를 선택합니다. 
1. **Builtin** 컨테이너에서 **Event Log Readers** 그룹을 선택합니다.
1. **Event Log Readers** 그룹을 마우스 오른쪽 단추로 클릭하거나 해당 상황에 맞는 메뉴에 액세스하고 **속성** 을 선택한 후 **Event Log Readers 속성** 대화 상자에서 **구성원** 탭을 선택합니다.
1. **구성원** 탭에서 **추가** 를 선택합니다.
1. **사용자, 연락처, 컴퓨터, 서비스 계정 또는 그룹 선택** 대화 상자에서 **개체 유형** 을 선택합니다.
1. **개체 유형** 대화 상자에서 **컴퓨터** 확인란을 선택한 후 **확인** 을 선택합니다.
1. **사용자, 컴퓨터, 서비스 계정 또는 그룹 선택** 대화 상자에서 **선택할 개체 이름을 입력하십시오.** 상자에 **SEA-SVR2** 를 입력한 후 **확인** 을 선택합니다.
1. **Event Log Readers 속성** 대화 상자에서 **확인** 을 선택합니다.

#### <a name="task-2-create-a-subscription-and-verify-the-results"></a>작업 2: 구독 만들기 및 결과 확인

1. **SEA-SVR2** 에서 **시작** 단추 옆에 있는 **검색하려면 여기에 입력하십시오.** 텍스트 상자에 **이벤트** 를 입력한 후 **가장 정확** 목록에서 **이벤트 뷰어** 를 선택합니다.
1. **이벤트 뷰어** 의 탐색 창에서 **구독** 을 선택합니다.
1. **구독** 을 마우스 오른쪽 단추로 클릭하거나 해당 상황에 맞는 메뉴에 액세스한 후 **구독 만들기** 를 선택합니다.
1. **구독 속성** 대화 상자의 **구독 이름** 상자에 **SEA-DC1 Events** 를 입력합니다.
1. **수집기 시작** 옵션이 선택되어 있는지 확인한 후 **컴퓨터 선택** 을 선택합니다.
1. **컴퓨터** 대화 상자에서 **도메인 컴퓨터 추가** 를 선택합니다.
1. **컴퓨터 선택** 대화 상자에서 **선택할 개체 이름을 입력하십시오.** 상자에 **SEA-DC1** 을 입력한 후 **확인** 을 선택합니다.
1. **컴퓨터** 대화 상자에서 **확인** 을 선택합니다.
1. **구독 속성 – SEA-DC1 Events** 대화 상자에서 **이벤트 선택** 을 선택합니다.
1. **로그 기간** 드롭다운 목록에서 **지난 24시간** 을 선택합니다.
1. **쿼리 필터** 대화 상자에서 **위험**, **오류**, **경고**, **정보** 확인란을 선택합니다.
1. **이벤트 로그** 드롭다운 목록에서 **Windows 로그** 를 확장한 후 **시스템** 및 **응용 프로그램** 확인란을 선택합니다.
1. **쿼리 필터** 대화 상자에서 **확인** 을 선택합니다.
1. **구독 속성 – SEA-DC1 Events** 대화 상자로 돌아가서 **확인** 을 선택합니다.
1. **SEA-SVR2** 에서 **이벤트 뷰어** 창으로 전환한 후 탐색 창에서 **Windows 로그** 를 확장합니다.
1. **Forwarded Events** 를 선택하고 전달된 이벤트에 **SEA-DC1** 에서 생성된 이벤트가 포함되어 있는지 확인합니다.
