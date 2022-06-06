---
lab:
  title: '랩: 하이브리드 시나리오에서 보안 솔루션 구현'
  module: 'Module 2: Implementing Security Solutions in Hybrid Scenarios'
ms.openlocfilehash: 475c539c6792c3a50a41c27ec5c293e895f0f07a
ms.sourcegitcommit: fb0d39e25bc0fe182037587b772d217db126d3bb
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/07/2022
ms.locfileid: "144812993"
---
# <a name="lab-implementing-security-solutions-in-hybrid-scenarios"></a>랩: 하이브리드 시나리오에서 보안 솔루션 구현

## <a name="scenario"></a>시나리오

온-프레미스 및 클라우드 보안 환경을 더욱 강화할 수 있는 Microsoft Azure 보안 관련 통합 기능을 식별하기 위해 개념 증명 환경에 있는 Windows 서버를 클라우드용 Microsoft Defender에 온보딩하기로 결정했습니다. 또한 Windows Server를 실행하는 온-프레미스 서버 및 Azure VM을 인벤토리, 변경 내용 추적 및 업데이트 관리를 비롯한 Azure Automation 기반 솔루션과 통합하려고 합니다.

## <a name="objectives"></a>목표

이 랩을 완료하면 다음을 수행할 수 있습니다.

- Azure Log Analytics 작업 영역 및 Azure Automation 계정을 만들 수 있습니다.
- 클라우드용 Defender를 구성할 수 있습니다.
- Windows Server를 실행하는 Azure VM을 프로비전할 수 있습니다.
- 온-프레미스 Windows Server를 클라우드용 Defender 및 Azure Automation에 온보딩할 수 있습니다.
- 클라우드용 Defender 및 Azure Automation 솔루션의 하이브리드 기능을 확인할 수 있습니다.

## <a name="estimated-time-75-minutes"></a>예상 시간: 75분

## <a name="lab-setup"></a>랩 설정

가상 머신: **AZ-801T00A-SEA-DC1**, **AZ-801T00A-SEA-SVR1**, **AZ-801T00A-SEA-SVR2** 를 실행해야 합니다. 다른 VM을 실행할 수도 있지만 이 랩에서는 필요하지 않습니다.

> **참고**: **AZ-801T00A-SEA-DC1**, **AZ-801T00A-SEA-SVR1**, **AZ-801T00A-SEA-SVR2** 가상 머신은 각각 **SEA-DC1**, **SEA-SVR1**, **SEA-SVR2** 의 설치를 호스트합니다.

1. **SEA-SVR2** 를 선택합니다.
1. 다음 자격 증명을 사용하여 로그인합니다.

   - 사용자 이름: **Administrator**
   - 암호: **Pa55w.rd**
   - 도메인: **CONTOSO**

이 랩에서는 사용 가능한 VM 환경과 Azure 구독을 사용합니다. 랩을 시작하기 전에 구독에서 Owner 또는 Contributor 역할이 있는 Azure 구독 및 사용자 계정이 있는지 확인하세요.

## <a name="exercise-1-creating-an-azure-log-analytics-workspace-and-an-azure-automation-account"></a>연습 1: Azure Log Analytics 작업 영역 및 Azure Automation 계정 만들기

### <a name="scenario"></a>시나리오

평가를 준비하려면 인벤토리, 변경 내용 추적 및 업데이트 관리를 포함하여 보안 관련 솔루션에 대한 핵심 기능을 제공할 Azure Log Analytics 작업 영역 및 Azure Automation 계정을 만드는 것으로 시작합니다.

이 연습의 주요 작업은 다음과 같습니다.

1. Azure Log Analytics 작업 영역 만들기 
1. Azure Automation 계정 만들기 및 구성

#### <a name="task-1-create-an-azure-log-analytics-workspace"></a>작업 1: Azure Log Analytics 작업 영역 만들기 

1. **SEA-SVR2** 에서 Microsoft Edge를 시작하고 **[Azure Portal](https://portal.azure.com)** 로 이동한 후 이 랩에서 사용할 구독에서 Owner 역할이 있는 사용자 계정의 자격 증명을 사용하여 로그인합니다.
1. Azure Portal에서 다음 설정을 사용하여 Log Analytics 작업 영역을 만듭니다.

   | 설정 | 값 |
   | --- | --- |
   | Subscription | 이 랩에서 사용 중인 Azure 구독의 이름 |
   | 리소스 그룹 | 새 리소스 그룹 **AZ801-L0201-RG** 의 이름 |
   | Log Analytics 작업 영역 | 고유한 이름 |
   | 지역 | 이전 작업에서 가상 머신을 배포한 Azure 지역의 이름 |

   >**참고**: 배포가 완료될 때까지 기다립니다. 배포는 1분 정도 걸립니다.

#### <a name="task-2-create-and-configure-an-azure-automation-account"></a>작업 2: Azure Automation 계정 만들기 및 구성

1. **SEA-SVR2** 의 Azure Portal에서 다음 설정을 사용하여 Azure Automation 계정을 만듭니다.

   | 설정 | 값 |
   | --- | --- |
   | Subscription | 이 랩에서 사용 중인 Azure 구독의 이름 |
   | 리소스 그룹 | **AZ801-L0201-RG** |
   | 이름 | 고유한 이름 |
   | 지역 | [작업 영역 매핑 설명서](https://docs.microsoft.com/en-us/azure/automation/how-to/region-mappings)에 따라 결정되는 Azure 지역의 이름 |

   >**참고**: [작업 영역 매핑 설명서](https://docs.microsoft.com/en-us/azure/automation/how-to/region-mappings)에 따라 Azure 지역을 지정해야 합니다.

   >**참고**: 배포가 완료될 때까지 기다립니다. 배포는 3분 정도 걸릴 수 있습니다.

1. Azure Portal에서 새로 만든 Azure Automation 계정으로 이동합니다. 
1. **Automation 계정** 페이지에서 **인벤토리** 페이지로 이동합니다.
1. **인벤토리** 페이지에서 이 작업의 앞부분에서 만든 Log Analytics 작업 영역과 연결하여 솔루션을 사용하도록 설정합니다.

   >**참고**: 해당 Log Analytics 솔루션의 설치가 완료되기를 기다립니다. 3분 정도 걸릴 수 있습니다. 

   >**참고**: **변경 내용 추적** 솔루션도 자동으로 설치됩니다.

1. **Automation 계정** 페이지에서 **업데이트 관리** 페이지로 이동합니다.
1. **Automation 계정** 페이지에서 이 작업의 앞부분에서 만든 Log Analytics 작업 영역과 연결하여 솔루션을 사용하도록 설정합니다.

   >**참고**: 설치가 완료될 때까지 기다립니다. 5분 정도 걸릴 수 있습니다.

## <a name="exercise-2-configuring-microsoft-defender-for-cloud"></a>연습 2: 클라우드용 Microsoft Defender 구성

### <a name="scenario"></a>시나리오

다음으로 클라우드용 Defender에서 제공하는 보안 강화 기능을 활용할 수 있는지 확인해야 합니다. 이렇게 하면 Windows Server 하이브리드 시나리오에 적용되는 기능을 적절하게 평가할 수 있습니다.

이 연습의 주요 작업은 다음과 같습니다.

1. 클라우드용 Defender 및 자동 에이전트 설치를 사용하도록 설정합니다.
1. 클라우드용 Defender의 보안 강화를 사용하도록 설정합니다.

#### <a name="task-1-enable-defender-for-cloud-and-automatic-agent-installation"></a>작업 1: 클라우드용 Microsoft Defender 및 자동 에이전트 설치 사용

이 작업에서는 Azure 구독에 연결하고 클라우드용 Defender의 보안 강화를 사용하도록 설정합니다.

1. **SEA-SVR2** 의 Azure Portal에서 **클라우드용 Microsoft Defender** 페이지로 이동합니다.
1. 구독을 클라우드용 Defender로 업그레이드하고 자동 에이전트 설치를 사용하도록 설정합니다. 

   > **참고:** 구독에 이미 클라우드용 Defender의 보안 강화가 활성화되어 있을 수 있습니다. 이 경우 다음 작업을 계속합니다.

#### <a name="task-2-enable-enhanced-security-of-defender-for-cloud"></a>작업 2: 클라우드용 Defender의 보안 강화 활성화

이 작업에서는 클라우드용 Defender의 보안 강화를 사용하도록 설정합니다.

1. **SEA-SVR2** 의 Azure Portal을 표시하는 Microsoft Edge 창의 **클라우드용 Microsoft Defender | 개요** 페이지에서 **환경 설정** 페이지로 이동합니다.
1. **환경 설정** 페이지에서 Azure 구독에서 사용할 수 있는 클라우드용 Defender 플랜을 검토합니다.

   > **참고:** 동일한 페이지에 나열된 개별 Microsoft Defender 플랜을 선택적으로 비활성화할 수 있습니다.

1. **서버용 Microsoft Defender** 플랜을 사용하도록 설정합니다. 
1. **설정 \| 자동 프로비전** 페이지로 이동합니다. 
1. 확장 목록에서 다음 작업을 수행합니다.

   - 이전 연습에서 만든 Log Analytics 작업 영역을 활용하여 **Azure VM용 Log Analytics 에이전트** 를 사용하도록 설정합니다.
   - 이전 연습에서 만든 Log Analytics 작업 영역을 활용하여 **Azure Arc Machine용 Log Analytics 에이전트(미리 보기)** 를 사용하도록 설정합니다.
   - **Microsoft 위협 및 취약성 관리** 옵션을 사용하여 **컴퓨터에 대한 취약성 평가** 를 사용하도록 설정합니다.


1. 클라우드용 Defender의 **클라우드 환경 설정** 페이지로 이동합니다.
1. **환경 설정** 페이지에서 Azure 구독을 나타내는 항목을 확장하고 이전 연습에서 만든 Log Analytics 작업 영역을 나타내는 항목을 검토합니다.
1. **설정 \| Defender 플랜** 페이지에서 작업 영역에서 사용할 수 있는 모든 클라우드용 Defender 플랜을 사용하도록 설정합니다.

   > **참고:** 위협 방지 기능을 포함한 모든 클라우드용 Defender 기능을 사용하도록 설정하려면 해당 워크로드를 포함하는 구독에서 보안 강화 기능을 사용하도록 설정해야 합니다. 작업 영역 수준에서 사용하도록 설정해도 Just-In-Time VM 액세스, 적응형 애플리케이션 제어 및 Azure 리소스에 대한 네트워크 검색 기능은 사용하도록 설정되지 않습니다. 또한 작업 영역 수준에서 사용할 수 있는 유일한 Microsoft Defender 플랜은 서버용 Microsoft Defender와 컴퓨터의 SQL Server용 Microsoft Defender입니다.

1. 이전 연습에서 만든 작업 영역의 경우 **데이터 수집** 범위를 **모든 이벤트** 로 설정합니다.

   > **참고:** 클라우드용 Defender에서 데이터 수집 계층을 선택하면 Log Analytics 작업 영역의 보안 이벤트 스토리지에만 영향을 줍니다. Log Analytics 에이전트는 작업 영역에 저장하도록 선택한 보안 이벤트 수준에 관계없이 클라우드용 Defender의 위협 보호에 필요한 보안 이벤트를 계속 수집하고 분석합니다. 보안 이벤트를 저장하도록 선택하면 작업 영역에서 해당 이벤트를 조사, 검색 및 감사할 수 있습니다.

## <a name="exercise-3-provisioning-azure-vms-running-windows-server"></a>연습 3: Windows Server에서 실행하는 Azure VM 프로비전

### <a name="scenario"></a>시나리오

Windows Server를 실행하는 Azure VM에 대한 이점을 포함하여 하이브리드 시나리오에서 클라우드용 Defender 기능을 테스트해야 합니다. 이를 위해 Azure Resource Manager 템플릿을 사용하여 Windows Server를 실행하는 Azure VM을 프로비전합니다.

이 연습의 주요 작업은 다음과 같습니다.

1. Azure Cloud Shell을 시작합니다.
1. Azure Resource Manager 템플릿을 사용하여 Azure VM을 배포합니다.

#### <a name="task-1-start-azure-cloud-shell"></a>작업 1: Azure Cloud Shell 시작

이 작업에서는 Azure Cloud Shell을 시작합니다.

1. **SEA-SVR2** 에서 Azure Portal의 Azure Cloud Shell 창에서 PowerShell 세션을 엽니다.
1. Cloud Shell을 처음 시작하는 경우 기본 구성 설정을 적용합니다.

#### <a name="task-2-deploy-an-azure-vm-by-using-an-azure-resource-manager-template"></a>작업 2: Azure Resource Manager 템플릿을 사용하여 Azure VM 배포

이 작업에서는 Azure Resource Manager 템플릿을 사용하여 Azure VM을 배포합니다.

1. **SEA-SVR2** 에서 Azure Portal의 Cloud Shell 창에서 **C:\\Labfiles\\Lab02\\L02-sub_template.json**, **C:\\Labfiles\\Lab02\\L02-rg_template.json**, **C:\\Labfiles\\Lab02\\L02-rg_template.parameters.json** 파일을 Cloud Shell 홈 디렉터리에 업로드합니다.
1. 랩 환경을 호스트할 리소스 그룹을 만들려면 Cloud Shell 창의 **PowerShell** 세션에서 다음 명령을 실행합니다(`<Azure_region>` 자리 표시자를 이 랩에서 리소스를 배포하려는 Azure 지역의 이름으로 교체).

   >**참고**: **(Get-AzLocation).Location** 명령을 사용하여 사용 가능한 Azure 지역의 이름을 나열할 수 있습니다.

   ```powershell 
   $location = '<Azure_region>'
   New-AzSubscriptionDeployment -Location $location -Name az801l2001deployment -TemplateFile ./L02-sub_template.json -rgLocation $location -rgName 'AZ801-L0202-RG'
   ```

1. 새로 만든 리소스 그룹에 Azure VM(가상 머신)을 배포하려면 다음 명령을 실행합니다.

   ```powershell 
   New-AzResourceGroupDeployment -Name az801l2002deployment -ResourceGroupName AZ801-L0202-RG -TemplateFile ./L02-rg_template.json -TemplateParameterFile ./L02-rg_template.parameters.json
   ```

   >**참고**: 배포가 완료될 때까지 기다리세요. 이 작업은 3분 정도 걸립니다.

1. Cloud Shell를 닫습니다.

## <a name="exercise-4-onboarding-on-premises-windows-server-into-microsoft-defender-for-cloud-and-azure-automation"></a>연습 4: 온-프레미스 Windows Server를 클라우드용 Microsoft Defender 및 Azure Automation에 온보딩

### <a name="scenario"></a>시나리오

**SEA-SVR1** 및 **SEA-SVR2** 를 클라우드용 Defender에 온보딩하여 온-프레미스 환경에서 실행되는 서버의 보안을 강화하는 데 사용할 수 있는 클라우드용 Defender 기능을 결정합니다.

이 연습의 주요 작업은 다음과 같습니다.

1. Log Analytics 에이전트의 수동 설치를 수행합니다.
1. Log Analytics 에이전트의 무인 설치를 수행합니다.
1. Azure VM용 Azure Automation 솔루션을 사용하도록 설정합니다.
1. 온-프레미스 서버에 Azure Automation 솔루션을 사용하도록 설정합니다.

#### <a name="task-1-perform-manual-installation-of-the-log-analytics-agent"></a>작업 1: Log Analytics 에이전트 수동 설치 수행

이 작업에서는 Log Analytics 에이전트의 수동 설치를 수행합니다.

1. **SEA-SVR2** 에서 Azure Portal을 표시하는 Microsoft Edge 창에서 클라우드용 Defender의 **클라우드용 Microsoft Defender \| 인벤토리** 페이지로 이동합니다.
1. **클라우드용 Microsoft Defender \| 인벤토리** 페이지에서 **+ 비 Azure 서버 추가** 를 선택합니다.
1. **클라우드용 Defender에 서버 온보딩** 페이지에서 Log Analytics 작업 영역 **az801-l02-workspace** 를 업그레이드합니다.

   > **참고:** 업그레이드가 성공적으로 완료되면 **업그레이드** 단추의 레이블이 **+ 서버 추가** 로 변경됩니다.

1. **+ 서버 추가** 단추를 사용하여 **az801-l02-workspace \| 에이전트 관리** 페이지로 이동하면, Log Analytics 에이전트 설치 프로그램을 다운로드하고 에이전트 설치를 완료하는 데 필요한 작업 영역 ID와 키를 식별할 수 있습니다.
1. **az801-l02-workspace \| 에이전트 관리** 페이지에서 **작업 영역 ID** 및 **기본 키** 의 값을 기록합니다. 이는 이 작업의 뒷부분에서 필요합니다.
1. **az801-l02-workspace \| Agents 관리** 페이지에서 Windows 에이전트(64비트)를 다운로드합니다.
1. 다운로드한 실행 파일을 사용하여 **Microsoft Monitoring Agent 설치** 마법사를 시작합니다.
1. **Microsoft Monitoring Agent 설치** 마법사의 **Azure Log Analytics** 페이지에서 이 작업 앞부분에서 기록한 **작업 영역 ID** 및 **기본 키** 값을 입력하고 기본 설정을 사용하여 Microsoft Monitoring Agent 설치를 완료합니다.

#### <a name="task-2-perform-unattended-installation-of-the-log-analytics-agent"></a>작업 2: Log Analytics 에이전트의 무인 설치 수행

이 작업에서는 Log Analytics 에이전트의 무인 설치를 수행합니다.

1. **SEA-SVR2** 에서 관리자 권한으로 **Windows PowerShell** 을 시작합니다.
1. **MMASetup-AMD64.exe** 파일의 콘텐츠를 추출하려면 **Windows PowerShell** 콘솔에서 다음 명령을 실행합니다.
    
   ```powershell
   New-Item -ItemType Directory -Path 'C:\Labfiles\L02\' -Force
   Copy-Item -Path $env:USERPROFILE\Downloads\MMASetup-amd64.exe -Destination 'C:\Labfiles\L02\' -Force
   Set-Location -Path C:\Labfiles\L02
   .\MMASetup-amd64.exe /c /t:C:\Labfiles\L02
   Remove-Item -Path .\MMASetup-amd64.exe
   ```

1. 설치 파일을 대상 **SEA-SVR1** 에 복사하려면 **Windows PowerShell** 콘솔에서 다음 명령을 실행합니다.
    
   ```powershell
   New-Item -ItemType Directory -Path '\\SEA-SVR1\c$\Labfiles\L02' -Force
   Copy-Item -Path 'C:\Labfiles\L02\*' -Destination '\\SEA-SVR1\c$\Labfiles\L02' -Recurse -Force
   ```

1. **SEA-SVR1** 에 Log Analytics 에이전트를 설치하려면 **Windows PowerShell** 콘솔에서 다음 명령을 실행합니다(`<WorkspaceID>` 및 `<PrimaryKey>` 자리 표시자를 이 연습의 이전 작업에서 기록한 **작업 영역 ID** 및 **작업 영역 키** 값으로 교체)

   ```powershell
   Invoke-Command -ComputerName SEA-SVR1.contoso.com -ScriptBlock { Start-Process -FilePath C:\Labfiles\L02\setup.exe -ArgumentList '/qn NOAPM=1 ADD_OPINSIGHTS_WORKSPACE=1 OPINSIGHTS_WORKSPACE_AZURE_CLOUD_TYPE=0 OPINSIGHTS_WORKSPACE_ID="<WorkspaceID>" OPINSIGHTS_WORKSPACE_KEY="<PrimaryKey>" AcceptEndUserLicenseAgreement=1' -Wait }
   ```

   > **참고:** 설치가 완료될 때까지 기다립니다. 이 작업은 1분 정도 걸립니다.

#### <a name="task-3-enable-azure-automation-solutions-for-azure-vms"></a>작업 3: Azure VM용 Azure Automation 솔루션 사용

1. **SEA-SVR2** 에서 Azure Portal을 표시하는 Microsoft Edge 창으로 전환하고 이 랩의 앞부분에서 프로비전한 Azure Automation 계정 페이지로 이동합니다. 
1. Automation 계정 페이지에서 **인벤토리** 페이지로 이동합니다.
1. **인벤토리** 페이지에서 도구 모음의 **+ Azure VM 추가** 단추를 사용하여 **az801l02-vm0** Azure VM에 대한 인벤토리 및 변경 내용 추적을 사용하도록 설정합니다. 

   > **참고:** **사용할 수 있도록 준비로** 나열되도록 하려면 VM을 Automation 계정 솔루션과 연결된 Log Analytics 작업 영역에 연결해야 합니다.

1. Automation 계정의 **업데이트 관리** 페이지로 이동합니다.
1. **업데이트 관리** 페이지에서 도구 모음의 **+ Azure VM 추가** 단추를 사용하여 **az801l02-vm0** Azure VM에 대한 업데이트 관리를 사용하도록 설정합니다. 

   > **참고:** 인벤토리 솔루션과 마찬가지로, **사용할 수 있도록 준비로** 나열되도록 하려면 VM을 Automation 계정 솔루션과 연결된 Log Analytics 작업 영역에 연결해야 합니다.

   > **참고:** 온-프레미스 VM은 Log Analytics 에이전트를 설치하고 이 연습의 이전 작업에서 수행한 인벤토리, 변경 내용 추적 및 업데이트 관리 솔루션을 호스트하는 Azure Automation 계정과 연결된 Azure Log Analytics 작업 영역에 등록한 결과로 온보딩됩니다.

#### <a name="task-4-enable-azure-automation-solutions-for-on-premises-servers"></a>작업 4: 온-프레미스 서버에 Azure Automation 솔루션 사용

1. **SEA-SVR2** 의 Azure Portal을 표시하는 Microsoft Edge 창에서 이 랩의 앞부분에서 프로비전한 **Azure Automation 계정** 페이지로 이동합니다. 
1. **Automation 계정** 페이지에서 **인벤토리** 페이지로 이동합니다.
1. **인벤토리** 페이지에서 **컴퓨터를 관리하려면 클릭하세요** 링크를 사용하여 온-프레미스 서버에 대한 인벤토리 및 변경 내용 추적을 사용하도록 설정합니다.

   > **참고:** 이 옵션은 인벤토리, 변경 내용 추적 및 업데이트 관리 솔루션을 호스트하는 Azure Automation 계정과 연결된 Azure Log Analytics 작업 영역에 Log Analytics 에이전트를 설치하고 등록한 온-프레미스 서버에 적용됩니다.

1. Automation 계정의 **업데이트 관리** 페이지로 이동합니다.
1. **업데이트 관리** 페이지에서 **컴퓨터를 관리하려면 클릭** 링크를 사용하여 온-프레미스 서버에 대한 업데이트 관리를 사용하도록 설정합니다.

   > **참고:** 인벤토리 및 변경 내용 추적 솔루션과 마찬가지로, 이 옵션은 인벤토리, 변경 내용 추적 및 업데이트 관리 솔루션을 호스트하는 Azure Automation 계정과 연결된 Azure Log Analytics 작업 영역에 Log Analytics 에이전트를 설치하고 등록한 온-프레미스 서버에 적용됩니다.

## <a name="exercise-5-verifying-the-hybrid-capabilities-of-microsoft-defender-for-cloud-and-azure-automation-solutions"></a>연습 5: 클라우드용 Microsoft Defender 및 Azure Automation 솔루션의 하이브리드 기능 확인

### <a name="scenario"></a>시나리오

온-프레미스 Azure VM과 Windows Server를 실행하는 서버가 혼합되어 있기 때문에 두 경우 모두에서 사용할 수 있는 클라우드용 Defender 및 Azure Automation 보안 관련 솔루션의 기능을 확인하려고 합니다. 두 리소스에 대한 사이버 공격을 시뮬레이션하고 클라우드용 Defender에서 경고를 식별합니다.

이 연습의 주요 작업은 다음과 같습니다.

1. Azure VM에 대한 위협 탐지 기능의 유효성을 검사합니다.
1. 온-프레미스 서버에 대한 위협 탐지 기능의 유효성을 검사합니다.
1. 하이브리드 시나리오에 적용되는 기능을 검토합니다.
1. Azure Automation 솔루션의 유효성을 검사합니다.

#### <a name="task-1-validate-threat-detection-capabilities-for-azure-vms"></a>작업 1: Azure VM에 대한 위협 탐지 기능 유효성 검사

이 작업에서는 Azure VM에 대한 위협 탐지 기능의 유효성을 검사합니다.

1. **SEA-SVR2** 의 Azure Portal을 표시하는 Microsoft Edge 창에서 **az801l02-vm0** 가상 머신의 페이지로 이동합니다.
1. **az801l02-vm0** 페이지에서 **명령 실행** 기능을 사용하여 다음 PowerShell 명령을 실행하여 위협 탐지 경고를 트리거합니다.

   ```powershell
   New-Item -ItemType Directory -Path 'C:\Temp' -Force
   Start-Process -FilePath powershell.exe -ArgumentList '-nop -exec bypass -EncodedCommand "cABvAHcAZQByAHMAaABlAGwAbAAgAC0AYwBvAG0AbQBhAG4AZAAgACIAJgAgAHsAIABpAHcAcgAgAGgAdAB0AHAAcwA6AC8ALwBkAG8AdwBuAGwAbwBhAGQALgBzAHkAcwBpAG4AdABlAHIAbgBhAGwAcwAuAGMAbwBtAC8AZgBpAGwAZQBzAC8AUwB5AHMAbQBvAG4ALgB6AGkAcAAgAC0ATwB1AHQARgBpAGwAZQAgAGMAOgBcAHQAZQBtAHAAXABzAHYAYwBoAG8AcwB0AC4AZQB4AGUAIAB9ACIA"' -Wait
   ```

1. **SEA-ADM1** 의 Azure Portal에서 **클라우드용 Microsoft Defender \| 보안 경고** 페이지로 이동합니다.
1. **클라우드용 Microsoft Defender \| 보안 경고** 페이지에서 **az801l02-vm0** 에서 의심스러운 PowerShell 사용을 나타내는 심각도가 높은 경고를 확인합니다.
1. 보안 경고를 선택하고 **보안 경고** 페이지에서 **작업 수행** 을 선택한 후 가능한 작업을 검토합니다.

   > **참고:** 향후 공격 가능성을 최소화하려면 보안 권장 사항을 구현하는 것이 좋습니다.

#### <a name="task-2-validate-the-threat-detection-capabilities-for-on-premises-servers"></a>작업 2: 온-프레미스 서버에 대한 위협 탐지 기능 유효성 검사

이 작업에서는 온-프레미스 서버에 대한 위협 탐지 기능의 유효성을 검사합니다.

1. **SEA-ADM1** 에서 **Windows PowerShell** 콘솔로 전환합니다.
1. 위협 탐지 경고를 트리거하려면 **Windows PowerShell** 콘솔에서 다음 명령을 실행합니다.
    
   ```powershell
   New-Item -ItemType Directory -Path 'C:\Temp' -Force
   powershell -nop -exec bypass -EncodedCommand "cABvAHcAZQByAHMAaABlAGwAbAAgAC0AYwBvAG0AbQBhAG4AZAAgACIAJgAgAHsAIABpAHcAcgAgAGgAdAB0AHAAcwA6AC8ALwBkAG8AdwBuAGwAbwBhAGQALgBzAHkAcwBpAG4AdABlAHIAbgBhAGwAcwAuAGMAbwBtAC8AZgBpAGwAZQBzAC8AUwB5AHMAbQBvAG4ALgB6AGkAcAAgAC0ATwB1AHQARgBpAGwAZQAgAGMAOgBcAHQAZQBtAHAAXABzAHYAYwBoAG8AcwB0AC4AZQB4AGUAIAB9ACIA"
   ```

1. **SEA-ADM1** 에서 Azure Portal을 표시하는 Microsoft Edge 창으로 전환하고 **클라우드용 Microsoft Defender \| 보안 경고** 페이지로 다시 이동합니다.
1. **클라우드용 Microsoft Defender \| 보안 경고** 페이지에서 **SEA-SVR2** 에서 의심스러운 PowerShell 사용을 나타내는 심각도가 높은 경고를 확인합니다.
1. 보안 경고를 선택하고 **보안 경고** 페이지에서 **작업 수행** 을 선택하고 가능한 작업을 검토합니다.

   > **참고:** 향후 공격 가능성을 최소화하려면 보안 권장 사항을 구현하는 것이 좋습니다.

#### <a name="task-3-review-the-features-and-capabilities-that-apply-to-hybrid-scenarios"></a>작업 3: 하이브리드 시나리오에 적용되는 기능 검토

이 작업에서는 하이브리드 시나리오에 적용되는 클라우드용 Defender 기능 및 기능을 검토합니다.

1. **SEA-SVR2** 에서 Azure Portal을 표시하는 Microsoft Edge 창에서 **클라우드용 Microsoft Defender \| 인벤토리** 페이지로 이동합니다.
1. **인벤토리** 페이지의 리소스 목록에서 **az801l02-vm0** Azure VM과 **SEA-SVR1.contoso.com** 및 **SEA-SVR2.contoso.com** 온-프레미스 서버를 나타내는 항목을 식별합니다.

   > **참고:** Azure 및 온-프레미스 VM을 나타내는 항목이 **인벤토리** 페이지에 표시되기까지 몇 분 정도 걸릴 수 있습니다.

   > **참고:** **az801l02-vm0** 에서 **모니터링 에이전트** 열에 **설치되지 않음** 상태를 보고하는 경우 **az801l02-vm0** 링크를 선택합니다. **리소스 상태(미리 보기)** 페이지에서 **권장 사항** 을 검토하고 **가상 머신에 Log Analytics 에이전트를 설치해야 함** 항목을 선택합니다. **가상 머신에 Log Analytics 에이전트를 설치해야 함** 페이지에서 **수정** 을 선택합니다. **리소스 수정** 페이지의 **작업 영역 ID** 드롭다운 목록에서 클라우드용 Defender에서 만든 기본 작업 영역을 선택한 후 **1개 리소스 수정** 을 선택합니다. 

1. **az801l02-vm0** 의 **리소스 상태(미리 보기)** 페이지로 이동하고 **권장 사항** 을 검토합니다.
1. **SEA-SVR2** 의 **리소스 상태(미리 보기)** 페이지로 이동하고 **권장 사항** 을 검토합니다.

#### <a name="task-4-validate-azure-automation-solutions"></a>작업 4: Azure Automation 솔루션 유효성 검사

1. **SEA-SVR2** 의 Azure Portal을 표시하는 Microsoft Edge 창에서 이 랩의 앞부분에서 프로비전한 Azure Automation 계정의 **인벤토리** 페이지로 이동합니다.
1. **인벤토리** 페이지에서 **컴퓨터** 탭을 검토하고 이 랩의 앞부분에서 Log Analytics 작업 영역에 등록한 Azure VM 및 온-프레미스 서버가 모두 포함되어 있는지 확인합니다.

   >**참고**: Azure VM 서버 또는 온-프레미스 서버 또는 둘 다 **컴퓨터** 탭에 나열되지 않은 경우 더 오래 기다려야 할 수 있습니다.

1. **인벤토리** 페이지에서 **소프트웨어**, **파일**, **Windows 레지스트리** 및 **Windows 서비스** 탭을 포함한 나머지 탭을 검토합니다.

   >**참고**: 수집된 데이터 및 파일은 **인벤토리** 페이지의 도구 모음에서 **설정 편집** 옵션을 통해 구성할 수 있습니다.

   >**참고**: **변경 내용 추적** 솔루션도 자동으로 설치됩니다.

1. 동일한 Azure Automation 계정의 **변경 내용 추적** 페이지로 이동합니다. 
1. **이벤트**, **파일**, **레지스트리**, **소프트웨어** 및 **Windows 서비스** 항목과 연결된 숫자를 식별합니다. 이 중에 0보다 큰 숫자가 있는 경우 페이지 아래쪽의 **변경 내용** 및 **이벤트** 탭에서 해당 변경 내용에 대한 자세한 내용을 확인할 수 있습니다.

   >**참고**: 이 경우에도 **변경 내용 추적** 페이지의 도구 모음에서 **설정 편집** 옵션을 통해 추적된 변경 내용을 구성할 수 있습니다.

1. 동일한 Azure Automation 계정의 **업데이트 관리** 페이지로 이동합니다.
1. **업데이트 관리** 페이지에서 **컴퓨터** 탭을 검토하고 이 랩의 앞부분에서 Log Analytics 작업 영역에 등록한 Azure VM 및 온-프레미스 서버가 모두 포함되어 있는지 확인합니다.
1. **컴퓨터** 탭에서 각 항목의 준수 상태를 확인한 다음, **누락된 업데이트** 탭으로 이동합니다.

   >**참고**: 온-프레미스 서버와 Azure VM 모두에 대한 누락된 업데이트의 자동화된 배포를 예약하는 옵션이 있습니다.

## <a name="exercise-6-deprovisioning-the-azure-environment"></a>연습 6: Azure 환경 프로비전 해제

### <a name="scenario"></a>시나리오

Azure 관련 요금을 최소화하기 위해 이 랩 전체에서 프로비전한 Azure 리소스의 프로비전을 해제합니다.

이 연습의 주요 작업은 다음과 같습니다.

1. Cloud Shell에서 PowerShell 세션을 시작합니다.
1. 랩에서 프로비전된 모든 Azure 리소스를 식별하고 제거합니다.

#### <a name="task-1-start-a-powershell-session-in-cloud-shell"></a>작업 1: Cloud Shell에서 PowerShell 세션 시작

1. **SEA-SVR2** 의 Azure Portal을 표시하는 Microsoft Edge 창에서 Cloud Shell 창을 엽니다.

#### <a name="task-2-identify-and-remove-all-azure-resources-that-were-provisioned-in-the-lab"></a>작업 2: 랩에서 프로비전된 모든 Azure 리소스 식별 및 제거

1. Cloud Shell 창에서 다음 명령을 실행하여 이 랩에서 만든 리소스 그룹을 모두 나열합니다.

   ```powershell
   Get-AzResourceGroup -Name 'AZ801-L02*'
   ```

   > **참고**: 출력에 이 랩에서 만든 리소스 그룹만 포함되어 있는지 확인하세요. 이 그룹은 이 작업에서 삭제됩니다.

1. 이 랩에서 만든 모든 리소스 그룹을 삭제하려면 다음 명령을 실행합니다.

   ```powershell
   Get-AzResourceGroup -Name 'AZ801-L02*' | Remove-AzResourceGroup -Force -AsJob
   ```

   >**참고**: 이 명령은 *-AsJob* 매개 변수에 의해 결정되어 비동기로 실행되므로, 동일한 PowerShell 세션 내에서 이 명령을 실행한 직후 다른 PowerShell 명령을 실행할 수 있지만 리소스 그룹이 실제로 제거되기까지는 몇 분 걸립니다.

## <a name="results"></a>결과

이 랩을 완료한 결과는 다음과 같습니다.

- Azure Log Analytics 작업 영역 및 Azure Automation 계정을 만들었습니다.
- 클라우드용 Defender를 구성했습니다.
- Windows Server를 실행되는 Azure VM을 프로비전했습니다.
- 온-프레미스 Windows Server를 클라우드용 Defender 및 Azure Automation에 온보딩했습니다.
- 클라우드용 Defender 및 Azure Automation 솔루션의 하이브리드 기능을 확인했습니다.
