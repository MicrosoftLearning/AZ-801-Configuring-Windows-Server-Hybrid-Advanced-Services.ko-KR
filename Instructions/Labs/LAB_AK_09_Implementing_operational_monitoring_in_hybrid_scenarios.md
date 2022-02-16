---
lab:
  title: '랩: 하이브리드 시나리오에서 운영 모니터링 구현'
  type: Answer Key
  module: Module 9 - Implementing operational monitoring in hybrid scenarios
ms.openlocfilehash: a0a3474c4db7bcebb3668e6728c3c4e002fc7487
ms.sourcegitcommit: 9a51ea796ef3806ab9e7ec1ff75034b2f929ed2a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/28/2022
ms.locfileid: "137907164"
---
# <a name="lab-answer-key-implementing-operational-monitoring-in-hybrid-scenarios"></a>랩 응답 키: 하이브리드 시나리오에서 운영 모니터링 구현

## <a name="exercise-1-preparing-a-monitoring-environment"></a>연습 1: 모니터링 환경 준비

#### <a name="task-1-deploy-an-azure-virtual-machine"></a>작업 1: Azure 가상 머신 배포

1. **SEA-SVR2** 에 연결하고 필요한 경우 **Pa55w.rd** 암호를 이용해 **CONTOSO\\Administrator** 로 로그인합니다.
1. **SEA-SVR2** 에서 Microsoft Edge를 시작하고 **[Azure Portal](https://portal.azure.com)** 로 이동한 후 이 랩에서 사용할 구독에서 소유자 역할이 있는 사용자 계정의 자격 증명을 사용하여 로그인합니다.
1. **SEA-SVR2** 의 Azure Portal을 표시하는 Microsoft Edge 창에서 Azure Portal에서 Cloud Shell 단추를 선택하여 Azure Cloud Shell 창을 엽니다.
1. **Bash** 와 **PowerShell** 중 선택하라는 메시지가 표시되면 **PowerShell** 을 선택합니다.

   > **참고:** Cloud Shell을 처음 시작하는 경우 **탑재된 스토리지가 없음** 메시지가 표시되면 이 랩에서 사용 중인 구독을 선택한 후 **스토리지 만들기** 를 선택합니다.

1. Cloud Shell 창의 도구 모음에서 **파일 업로드/다운로드** 아이콘을 선택하고 드롭다운 메뉴에서 **업로드** 를 선택한 후 **C:\\Labfiles\\Lab09\\L09-rg_template.json** 파일을 Cloud Shell 홈 디렉터리에 업로드합니다.
1. 이전 단계를 반복하여 **C:\\Labfiles\\Lab09\\L09-rg_template.parameters.json** 파일을 Cloud Shell 홈 디렉터리에 업로드합니다.
1. 랩 환경을 호스트할 리소스 그룹을 만들려면 Cloud Shell 창의 **PowerShell** 세션에서 다음 명령을 입력하고 각 명령을 입력한 후 Enter 키를 누릅니다(`<Azure_region>` 자리 표시자를 이 랩에서 리소스를 배포하려는 Azure 지역의 이름으로 바꿈).

   >**참고**: **(Get-AzLocation).Location** 명령을 사용하여 사용 가능한 Azure 지역의 이름을 나열할 수 있습니다.

   ```powershell 
   $location = '<Azure_region>'
   $rgName = 'AZ801-L0901-RG'
   New-AzResourceGroup -ResourceGroupName $rgName -Location $location
   ```

1. 새로 만든 리소스 그룹에 Azure VM(가상 머신)을 배포하려면 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell 
   New-AzResourceGroupDeployment -Name az801l0901deployment -ResourceGroupName $rgName -TemplateFile ./L09-rg_template.json -TemplateParameterFile ./L09-rg_template.parameters.json -AsJob
   ```

   >**참고**: 배포가 완료될 때까지 기다리지 말고 다음 작업을 진행하세요. 배포는 3분 정도 걸립니다.

#### <a name="task-2-register-the-microsoftinsights-and-microsoftalertsmanagement-resource-providers"></a>작업 2: Microsoft.Insights 및 Microsoft.AlertsManagement 리소스 공급자 등록

1. Microsoft.Insights 및 Microsoft.AlertsManagement 리소스 공급자를 등록하려면 **SEA-SVR2** 의 Cloud Shell 창에서 다음 명령을 입력하고 각 명령을 입력한 후 Enter 키를 누릅니다.

   ```powershell
   Register-AzResourceProvider -ProviderNamespace Microsoft.Insights
   Register-AzResourceProvider -ProviderNamespace Microsoft.AlertsManagement
   ```

   >**참고**: 등록 상태를 확인하려면 **Get-AzResourceProvider** cmdlet을 사용하면 됩니다.

   >**참고**: 등록 프로세스가 완료될 때까지 기다리지 말고 다음 작업을 진행하세요. 등록은 3분 정도 걸립니다.

1. Cloud Shell를 닫습니다.

#### <a name="task-3-create-and-configure-an-azure-log-analytics-workspace"></a>작업 3: Azure Log Analytics 작업 영역 만들기 및 구성

1. **SEA-SVR2** 의 Azure Portal에 있는 **리소스, 서비스 및 문서 검색** 텍스트 상자의 도구 모음에서 **Log Analytics 작업 영역** 을 검색하여 선택한 후 **Log Analytics 작업 영역** 페이지에서 **+ 만들기** 를 선택합니다.
1. **Log Analytics 작업 영역 만들기** 페이지의 **기본 사항** 탭에서 다음 설정을 입력하고 **검토 + 만들기** 를 선택한 후 **만들기** 를 선택합니다.

   | 설정 | 값 |
   | --- | --- |
   | Subscription | 이 랩에서 사용 중인 Azure 구독의 이름 |
   | Resource group | 새 리소스 그룹 **AZ801-L0902-RG** 의 이름 |
   | Log Analytics 작업 영역 | 고유 이름 |
   | 지역 | 이전 작업에서 가상 머신을 배포한 Azure 지역의 이름 |

   >**참고**: 이전 작업에서 가상 머신을 배포한 지역과 같은 지역을 지정해야 합니다.

   >**참고**: 배포가 완료될 때까지 기다리세요. 배포는 1분 정도 걸립니다.

## <a name="exercise-2-configuring-monitoring-of-on-premises-servers"></a>연습 2: 온-프레미스 서버 모니터링 구성

#### <a name="task-1-register-windows-admin-center-with-azure"></a>작업 1: Windows Admin Center를 Azure에 등록

1. **SEA-SVR2** 에서 **시작** 을 선택한 후 **Windows PowerShell(관리자)** 을 선택합니다.

   >**참고**: **SEA-SVR2** 에 Windows Admin Center를 아직 설치하지 않았다면 다음 두 단계를 수행합니다.

1. **Windows PowerShell** 콘솔에서 다음 명령을 입력한 후 Enter 키를 눌러 Windows Admin Center의 최신 버전을 다운로드합니다.
    
   ```powershell
   Start-BitsTransfer -Source https://aka.ms/WACDownload -Destination "$env:USERPROFILE\Downloads\WindowsAdminCenter.msi"
   ```
1. 다음 명령을 입력한 후 Enter 키를 눌러 Windows Admin Center를 설치합니다.
    
   ```powershell
   Start-Process msiexec.exe -Wait -ArgumentList "/i $env:USERPROFILE\Downloads\WindowsAdminCenter.msi /qn /L*v log.txt REGISTRY_REDIRECT_PORT_80=1 SME_PORT=443 SSL_CERTIFICATE_OPTION=generate"
   ```

   > **참고**: 설치가 완료될 때까지 기다리세요. 이 작업은 2분 정도 걸립니다.

1. **SEA-SVR2** 에서 Microsoft Edge를 시작한 후 **https://SEA-SVR2.contoso.com** 으로 이동합니다. 
1. 메시지가 표시되면 **Windows 보안** 대화 상자에 다음 자격 증명을 입력한 후 **확인** 을 선택합니다.

   - 사용자 이름: **CONTOSO\\Administrator**
   - 암호: **Pa55w.rd**

1. **모든 연결** 창에서 페이지의 오른쪽 위 모서리에 있는 **설정**(톱니 모양) 아이콘을 선택합니다.
1. Windows Admin Center의 **설정** 페이지에 있는 **Azure 계정** 섹션에서 **Azure에 등록** 을 선택한 후 **등록** 을 선택합니다.
1. **Windows Admin Center에서 Azure 시작** 창에서 **복사** 를 선택하여 등록 절차의 단계 목록에 표시된 코드를 복사합니다. 
1. 등록 절차의 단계 목록에서 **코드 입력** 링크를 선택합니다.

   >**참고**: Microsoft Edge 창에 **코드 입력** 페이지가 표시되는 다른 탭이 열립니다.

1. **코드 입력** 텍스트 상자에서 복사한 코드를 클립보드에 붙여넣은 후 **다음** 을 선택합니다.
1. **로그인** 페이지에 이전 연습에서 Azure 구독에 로그인하는 데 사용한 것과 같은 사용자 이름을 입력하고 **다음** 을 선택한 후 해당 암호를 입력하고 **로그인** 을 선택합니다.
1. **Windows Admin Center에 로그인하시겠습니까?** 라는 메시지가 표시되면 **계속** 을 선택합니다.
1. Windows Admin Center에서 로그인이 성공했는지 확인하고 Microsoft Edge 창에 새로 열린 탭을 닫습니다.
1. **Windows Admin Center에서 Azure 시작** 창에서 **Azure Active Directory 애플리케이션** 이 **새로 만들기** 로 설정되어 있는지 확인하고 **연결** 을 선택합니다.
1. 등록 절차의 단계 목록에서 **로그인** 을 선택합니다. 그러면 **요청된 권한** 이라는 레이블이 지정된 팝업 창이 열립니다.
1. **요청된 권한** 팝업 창에서 **조직 대신 동의** 를 선택한 후 **수락** 을 선택합니다.

#### <a name="task-2-integrate-an-on-premises-windows-server-with-azure-monitor"></a>작업 2: Azure Monitor와 온-프레미스 Windows Server 통합

1. **SEA-SVR2** 의 Windows Admin Center를 표시하는 Microsoft Edge 창에서 **모든 연결** 창으로 이동합니다.
1. **모든 연결** 창에서 **sea-svr2.contoso.com** 항목을 선택합니다. 
1. **sea-svr2.contoso.com** 페이지의 **도구** 메뉴에서 **Azure Monitor** 를 선택한 후 **Azure에 로그인 및 설정** 을 선택합니다.
1. **Azure Monitor 설정** 페이지에서 다음 설정을 지정하고 **설정** 을 선택합니다.

   | 설정 | 값 |
   | --- | --- |
   | Subscription | 이 랩에서 사용 중인 Azure 구독의 이름 |
   | Resource group | **AZ801-L0901-RG** |
   | 리소스 그룹 지역 | 이전 연습에서 가상 머신을 배포한 Azure 지역의 이름 |
   | Log Analytics 작업 영역 | 이전 연습에서 만든 작업 영역의 이름 |
   | Azure Arc 사용 | 선택 |

   >**참고**: 설정이 완료될 때까지 기다리지 말고 다음 연습을 진행하세요. 설정은 3분 정도 걸립니다.

   >**참고**: 이 프로세스에서는 Log Analytics 에이전트 및 Dependency Agent를 자동으로 설치합니다.

## <a name="exercise-3-configuring-monitoring-of-azure-vms"></a>연습 3: Azure VM 모니터링 구성

#### <a name="task-1-review-host-based-monitoring"></a>작업 1: 호스트 기반 모니터링 검토

1. Azure Portal에서 **가상 머신** 을 검색하여 선택하고 **가상 머신** 페이지에서 **az801l09-vm0** 을 선택합니다.
1. **az801l09-vm0** 페이지의 **모니터링** 섹션에서 **메트릭** 을 선택합니다.
1. **az801l09-vm0 \| 메트릭** 페이지의 기본 차트에 있는 **메트릭 네임스페이스** 드롭다운 목록에서는 **가상 머신 호스트** 메트릭만 사용할 수 있습니다.

   >**참고**: 게스트 수준 진단 설정이 아직 구성되지 않았으므로 그렇습니다. 그러나 **메트릭 네임스페이스** 드롭다운 목록에서 게스트 메모리 메트릭을 직접 사용하도록 설정하는 옵션이 있습니다. 이 연습의 뒷부분에서 사용하도록 설정합니다.

1. **메트릭** 드롭다운 목록에서 사용 가능한 메트릭 목록을 검토합니다.

   >**참고**: 목록에는 게스트 수준 메트릭에 액세스할 필요 없이 가상 머신 호스트에서 수집할 수 있는 다양한 CPU, 디스크, 네트워크 관련 메트릭이 포함됩니다.

1. **메트릭** 드롭다운 목록에서 **CPU 백분율** 을 선택하고 **집계** 드롭다운 목록에서 **평균** 항목이 선택되어 있는지 확인한 후 결과 차트를 검토합니다.

#### <a name="task-2-configure-diagnostic-settings-and-vm-insights"></a>작업 2: 진단 설정 및 VM Insights 구성

1. **az801l09-vm0** 페이지의 **모니터링** 섹션에서 **진단 설정** 을 선택합니다.
1. **az801l09-vm0 \| 진단 설정** 페이지의 **개요** 탭에서 **게스트 수준 모니터링 사용** 을 선택합니다.

   >**참고**: 작업이 적용될 때까지 기다리세요. 3분 정도 걸릴 수 있습니다.

1. **az801l09-vm0 \| 진단 설정** 페이지의 **성능 카운터** 탭으로 전환하고 사용 가능한 카운터를 검토합니다.

   >**참고**: 기본적으로 CPU, 메모리, 디스크, 네트워크 카운터가 사용하도록 설정되어 있습니다. 자세한 목록을 보려면 **사용자 지정** 보기로 전환하면 됩니다.

1. **az801l09-vm0 \| 진단 설정** 페이지의 **로그** 탭으로 전환하고 **게스트 수준 모니터링 사용** 단추를 선택합니다. 

   >**참고**: 게스트 수준 모니터링 진단이 사용하도록 설정될 때까지 기다리세요. 이 작업은 3분 정도 걸립니다.

1. **az801l09-vm0 \| 진단 설정** 페이지의 **개요** 탭에서 사용 가능한 이벤트 로그 수집 옵션을 검토합니다.

   >**참고**: 기본적으로 로그 수집에는 애플리케이션 로그 및 시스템 로그의 위험, 오류, 경고 항목과 보안 로그의 감사 실패 항목이 포함됩니다. **로그** 탭에서 항목을 사용자 지정할 수 있습니다.

1. **az801l09-vm0 \| 진단 설정** 페이지에서 **로그** 탭을 선택하고 사용 가능한 구성 설정을 검토합니다.
1. **az801l09-vm0 \| 로그** 페이지의 왼쪽 세로 메뉴에 있는 **모니터링** 섹션에서 **메트릭** 을 선택합니다.
1. **az801l09-vm0 \| 메트릭** 페이지의 기본 차트에서 이때 **메트릭 네임스페이스** 드롭다운 목록에 **가상 머신 호스트** 항목 외에 **게스트(클래식)** 항목도 포함됩니다.

   >**참고**: 게스트 수준 진단 설정을 사용하도록 설정했으므로 그렇습니다. **새 게스트 메모리 메트릭 사용** 옵션도 있습니다.

1. **메트릭 네임스페이스** 드롭다운 목록에서 **게스트(클래식)** 항목을 선택합니다.
1. **메트릭** 드롭다운 목록에서 사용 가능한 메트릭 목록을 검토하면 메모리 및 논리 디스크와 관련된 다양한 메트릭이 포함되어 있음을 알 수 있습니다.

   >**참고**: 목록에는 호스트 수준 모니터링만 사용할 경우 사용할 수 없는 추가 게스트 수준 메트릭이 포함됩니다.

1. **메트릭 네임스페이스** 드롭다운 목록에서 **새 게스트 메모리 메트릭 사용** 항목을 선택합니다.
1. **게스트 메트릭 사용(미리 보기)** 창에서 제공된 정보를 검토합니다.
1. **az801l09-vm0 \| 진단 설정** 페이지에서 **싱크** 탭을 선택하고 **Azure Monitor(미리 보기)** 섹션에서 **사용** 을 선택한 후 **저장** 을 선택합니다.
1. **az801l09-vm0 \| 메트릭** 페이지로 돌아가 보면 기본 차트에서 이때 **메트릭 네임스페이스** 드롭다운 목록에 **가상 머신 호스트** 및 **게스트(클래식)** 항목 외에 **가상 머신 게스트** 항목도 포함됩니다.

   >**참고**: **가상 머신 게스트** 항목을 표시하려면 페이지를 새로 고쳐야 할 수 있습니다.

1. **az801l09-vm0 \| 메트릭** 페이지의 왼쪽 세로 메뉴에 있는 **모니터링** 섹션에서 **로그** 를 선택합니다.
1. **az801l09-vm0 \| 로그** 페이지에서 **사용** 을 선택합니다.
1. **Log Analytics 작업 영역 선택** 드롭다운 목록에서 이 랩의 앞부분에서 만든 Log Analytics 작업 영역을 선택한 후 **사용** 을 선택합니다.
1. **az801l09-vm0 \| 로그** 페이지의 왼쪽 세로 메뉴에 있는 **모니터링** 섹션에서 **인사이트** 를 선택합니다.
1. 필요한 경우 **az801l09-vm0 \| 인사이트** 페이지에서 **사용** 을 선택합니다.

   >**참고**: 이 설정은 Azure VM Insights 기능을 제공합니다. VM Insights는 Azure VM과 Windows 또는 Linux를 실행하는 온-프레미스 컴퓨터의 성능과 상태 모니터링을 지원하는 Azure Monitor 솔루션입니다.

1. **SEA-SVR2** 의 Azure Portal에 있는 **리소스, 서비스 및 문서 검색** 텍스트 상자의 도구 모음에서 **모니터** 를 검색하여 선택한 후 **모니터 \| 개요** 페이지의 **인사이트** 에서 **가상 머신** 을 선택합니다.
1. **모니터 \| 가상 머신** 페이지에서 **성능** 탭을 선택한 후 **업그레이드** 를 선택합니다.
1. **업그레이드** 창에서 **업그레이드** 를 선택합니다.
1. **모니터 \| 가상 머신** 페이지에서 **맵** 탭을 선택한 후 **지금 사용해 보기** 를 선택합니다.
1. **적용 범위 관리** 페이지에서 **작업 영역 구성** 을 선택합니다.
1. **Azure Monitor** 페이지의 **Log Analytics 작업 영역 선택** 드롭다운 메뉴에서 이 랩의 앞부분에서 만든 작업 영역을 선택한 후 **구성** 을 선택합니다.

   >**참고**: 이 옵션은 VM용 Azure Monitor에서 내보낸 메트릭을 사용하여 빌드된 상태 모니터 계층 구조로 구성된 상태 모델을 사용하여 모니터링 및 경고 기능을 사용하도록 설정합니다.

## <a name="exercise-4-evaluating-monitoring-services"></a>연습 4: 모니터링 서비스 평가

#### <a name="task-1-review-azure-monitor-monitoring-and-alerting-functionality"></a>작업 1: Azure Monitor 모니터링 및 경고 기능 검토

1. **SEA-SVR2** 의 Azure Portal에서 **모니터 \| 메트릭** 페이지로 돌아갑니다.
1. **범위 선택** 페이지의 **찾아보기** 탭에서 **AZ801-L0901-RG** 리소스 그룹으로 이동하여 이 그룹을 확장하고 해당 리소스 그룹 내 **az801l09-vm0** 가상 머신 항목 옆에 있는 확인란을 선택한 후 **적용** 을 선택합니다.

   >**참고**: 목록에 **az801l09-vm0** 이 표시되지 않으면 **검색하여 항목 필터링...** 상자에서 **az801l09-vm0** 을 검색합니다.
   
   >**참고**: **az801l09-vm0 \| 메트릭** 페이지에서 사용할 수 있는 것과 같은 보기 및 옵션이 제공됩니다.

1. **메트릭** 드롭다운 목록에서 **CPU 백분율** 을 선택하고 **집계** 드롭다운 목록에 **평균** 이 표시되는지 확인한 후 결과 차트를 검토합니다.
1. **모니터 \| 메트릭** 페이지의 **az801l09-vm0 평균 CPU 백분율** 창에서 **새 경고 규칙** 을 선택합니다.

   >**참고**: 메트릭에서 경고 규칙을 만드는 기능은 게스트(클래식) 메트릭 네임스페이스의 메트릭에 대해 지원되지 않습니다. 이 경우에 메트릭에서 경고 규칙을 만들려면 **[Windows 가상 머신에 대해 Resource Manager 템플릿을 사용하여 Azure Monitor 메트릭 저장소에 게스트 OS 메트릭 보내기](https://docs.microsoft.com/en-us/azure/azure-monitor/platform/collect-custom-metrics-guestos-resource-manager-vm)** 에 설명된 대로 Azure Resource Manager 템플릿을 사용하면 됩니다.

1. **경고 규칙 만들기** 페이지의 **조건** 섹션에서 기존 조건 항목을 선택합니다.
1. **신호 논리 구성** 페이지의 신호 목록에 있는 **경고 논리** 섹션에서 다음 설정을 지정하고(다른 설정은 기본값으로 둠) **완료** 를 선택합니다.

   | 설정 | 값 |
   | --- | --- |
   | 임계값 | **정적** |
   | 연산자 | **보다 큼** |
   | 집계 유형 | **평균** |
   | 임계값 | **2** |
   | 집계 세분성(기간) | **1분** |
   | 평가 빈도 | **1분마다** |

1. **경고 규칙 만들기** 페이지의 **작업** 탭에서 **작업 그룹 추가** 를 선택한 후 **+ 작업 그룹 만들기** 단추를 선택합니다.
1. **작업 그룹 만들기** 페이지의 **기본 사항** 탭에서 다음 설정을 지정하고(다른 설정은 기본값으로 둠) **다음: 알림 >** 을 선택합니다.

   | 설정 | 값 |
   | --- | --- |
   | Subscription | 이 랩에서 사용 중인 Azure 구독의 이름 |
   | Resource group | **AZ801-L0902-RG** |
   | 작업 그룹 이름 | **az801l09-ag1** |
   | 표시 이름 | **az801l09-ag1** |

1. **작업 그룹 만들기** 페이지의 **알림** 탭에 있는 **알림 유형** 드롭다운 목록에서 **메일/SMS 메시지/푸시/음성** 을 선택합니다. **이름** 텍스트 상자에 **관리자 메일** 을 입력한 후 **세부 정보 편집**(연필) 아이콘을 선택합니다.
1. **메일/SMS 메시지/푸시/음성** 페이지에서 **메일** 확인란을 선택하고 **메일** 텍스트 상자에 메일 주소를 입력한 후 다른 설정은 기본값으로 두고 **확인** 을 선택합니다. **작업 그룹 만들기** 페이지의 **알림** 탭으로 돌아가서 **다음: 작업 >** 을 선택합니다.
1. **작업 그룹 만들기** 페이지의 **작업** 탭에서 아무것도 변경하지 않고 **작업 유형** 드롭다운 목록에서 사용할 수 있는 항목을 검토하고 **검토 + 만들기** 를 선택합니다.
1. **작업 그룹 만들기** 페이지의 **검토 + 만들기** 탭에서 **만들기** 를 선택합니다.
1. **경고 규칙 만들기** 페이지로 돌아가서 **경고 규칙 세부 정보** 섹션에서 다음 설정을 지정합니다(다른 설정은 기본값으로 둠).

   | 설정 | 값 |
   | --- | --- |
   | 경고 규칙 이름 | **CPU Percentage above the test threshold** |
   | 설명 | **테스트 임계값을 초과하는 CPU 백분율** |
   | Resource group | **AZ801-L0902-RG** |
   | 심각도 | **Sev 3** |
   | 규칙을 만들면 바로 사용 | **예** |

1. **경고 규칙 만들기** 를 선택합니다.

   >**참고**: 메트릭 경고 규칙이 활성화되는 데 최대 10분이 걸릴 수 있습니다.

1. Azure Portal에서 **가상 머신** 을 검색하여 선택하고 **가상 머신** 페이지에서 **az801l09-vm0** 을 선택합니다.
1. **az801l09-vm0** 페이지의 **작업** 섹션에서 **명령 실행** 을 선택한 후 **RunPowerShellScript** 를 선택합니다.
1. **명령 실행 스크립트** 페이지의 **PowerShell 스크립트** 섹션에서 다음 명령을 입력하고 **실행** 을 선택하여 대상 운영 체제 내에서 CPU 사용률을 높입니다.

   ```powershell
   $vCpuCount = Get-WmiObject Win32_Processor | Select-Object -ExpandProperty NumberOfLogicalProcessors
   ForEach ($vCpu in 1..$vCpuCount){ 
      Start-Job -ScriptBlock{
         $result = 1;
         ForEach ($loopCount in 1..2147483647){
            $result = $result * $loopCount
         }
      }
   }
   ```

   >**참고**: 이렇게 하면 새로 만든 경고 규칙의 임계값보다 CPU 사용률이 높아집니다.

1. **SEA-SVR2** 의 Azure Portal을 표시하는 Microsoft Edge 창에서 다른 탭을 열고 **모니터** 페이지로 이동한 후 **경고** 를 선택합니다.
1. **Sev 3** 경고 수를 기록해 두고 **Sev 3** 행을 선택합니다.

   >**참고**: 몇 분 정도 기다렸다가 **새로 고침** 을 선택해야 할 수 있습니다.

1. **모든 경고** 페이지에서 생성된 경고를 검토합니다.

#### <a name="task-2-review-azure-monitor-vm-insights-functionality"></a>작업 2: Azure Monitor VM Insights 기능 검토

1. **SEA-SVR2** 의 Azure Portal에서 **az801l09-vm0** 가상 머신 페이지로 돌아갑니다.
1. **az801l09-vm0** 가상 머신 페이지의 왼쪽 세로 메뉴에 있는 **모니터링** 섹션에서 **인사이트** 를 선택합니다.
1. **az801l09-vm0 \| 인사이트** 페이지의 **성능** 탭에서 논리 디스크 성능, CPU 사용률, 사용 가능한 메모리, 전송 및 수신된 바이트 속도를 비롯한 기본 메트릭 세트를 검토합니다.
1. **az801l09-vm0 \| 인사이트** 페이지에서 **맵** 탭을 선택하고 자동 생성된 맵을 검토합니다.
1. **az801l09-vm0 \| 인사이트** 페이지에서 **상태** 탭을 선택하고 해당 콘텐츠를 검토합니다.

   >**참고**: 상태 정보의 가용성은 작업 영역 업그레이드 완료에 따라 달라집니다.

#### <a name="task-3-review-azure-log-analytics-functionality"></a>작업 3: Azure Log Analytics 기능 검토

1. **SEA-SVR2** 의 Azure Portal에서 **모니터** 페이지로 돌아가서 **로그** 를 선택합니다.

   >**참고**: Log Analytics에 처음 액세스하는 경우 **Log Analytics 시작** 창을 닫아야 할 수 있습니다.

1. **범위 선택** 페이지에서 **최근 항목** 탭을 선택하고 **az801l09-vm0** 을 선택한 후 **적용** 을 선택합니다.
1. 쿼리 창에서 다음 쿼리를 붙여넣고 **실행** 을 선택한 후 결과 차트를 검토합니다.

   ```kql
   // Virtual Machine available memory
   // Chart the VM's available memory over the last hour.
   InsightsMetrics
   | where TimeGenerated > ago(1h)
   | where Name == "AvailableMB"
   | project TimeGenerated, Name, Val
   | render timechart
   ```

1. 도구 모음에서 **쿼리** 를 선택하고 **쿼리** 창에서 **가용성** 노드를 확장한 후 **VM 가용성 추적** 타일을 선택하고 **실행** 단추를 선택한 후 결과를 검토합니다.
1. **새 쿼리 1** 탭에서 **테이블** 헤더를 선택하고 **가상 머신** 섹션의 테이블 목록을 검토합니다.

   >**참고**: 여러 테이블의 이름은 이 랩의 앞부분에서 설치한 솔루션에 해당합니다. 특히, **InsightMetrics** 는 Azure VM Insights에서 성능 메트릭을 저장하는 데 사용됩니다.

1. 커서를 **VMComputer** 항목 위로 이동하고 **미리 보기 데이터 참조** 아이콘을 선택한 후 결과를 검토합니다.

   >**참고**: 데이터에 **az801l09-vm0** 과 **SEA-SVR2.contoso.com** 에 대한 항목이 모두 포함되어 있는지 확인하세요.

   >**참고**: 업데이트 데이터를 사용할 수 있게 되기까지 몇 분 정도 기다려야 할 수 있습니다.

1. **SEA-SVR2** 의 Azure Portal에 있는 **리소스, 서비스 및 문서 검색** 텍스트 상자의 도구 모음에서 **Log Analytics 작업 영역** 을 검색하여 선택한 후 **Log Analytics 작업 영역** 페이지에서 이 랩의 앞부분에서 만든 작업 영역을 나타내는 항목을 선택합니다.
1. 작업 영역 페이지의 왼쪽 세로 메뉴에 있는 **일반** 섹션에서 **솔루션** 을 선택합니다.
1. 솔루션 목록에서 **ServiceMap** 을 선택한 후 **요약** 페이지에서 **서비스 맵** 타일을 선택합니다.
1. **ServiceMap** 페이지의 **머신** 탭에서 **SEA-SVR2** 를 선택하여 서비스 맵을 표시합니다.
1. 확대하여 **SEA-SVR2** 에서 사용할 수 있는 네트워크 포트를 보여 주는 맵을 검토하고 다른 포트를 선택한 후 해당 연결 정보를 검토합니다.
1. 각 연결에 대해 **요약** 보기와 **속성** 보기 간에 전환합니다. 속성 보기는 연결 대상에 대한 더 자세한 정보를 제공합니다.

## <a name="exercise-5-deprovisioning-the-azure-environment"></a>연습 5: Azure 환경 프로비전 해제

#### <a name="task-1-start-a-powershell-session-in-cloud-shell"></a>작업 1: Cloud Shell에서 PowerShell 세션 시작

1. **SEA-SVR2** 의 Azure Portal을 표시하는 Microsoft Edge 창에서 **Cloud Shell** 아이콘을 선택하여 Cloud Shell 창을 엽니다.

#### <a name="task-2-delete-all-azure-resources-provisioned-in-the-lab"></a>작업 2: 랩에서 프로비저닝된 모든 Azure 리소스 삭제

1. Cloud Shell 창에서 다음 명령을 실행하여 이 랩에서 만든 리소스 그룹을 모두 나열합니다.

   ```powershell
   Get-AzResourceGroup -Name 'AZ801-L09*'
   ```

   > **참고**: 출력에 이 랩에서 만든 리소스 그룹만 포함되어 있는지 확인하세요. 이 그룹은 이 작업에서 삭제됩니다.

1. 다음 명령을 실행하여 이 랩에서 만든 모든 리소스 그룹을 삭제합니다.

   ```powershell
   Get-AzResourceGroup -Name 'AZ801-L09*' | Remove-AzResourceGroup -Force -AsJob
   ```

   >**참고**: 이 명령은 *-AsJob* 매개 변수에 의해 결정된 대로 비동기적으로 실행됩니다. 따라서 같은 PowerShell 세션에서 즉시 다른 PowerShell 명령을 실행할 수 있지만, 리소스 그룹이 실제로 제거되려면 몇 분 정도 걸립니다.
