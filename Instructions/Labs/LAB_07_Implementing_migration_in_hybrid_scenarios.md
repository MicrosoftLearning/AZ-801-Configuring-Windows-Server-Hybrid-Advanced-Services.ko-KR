---
lab:
  title: '랩: Azure Migrate를 사용하여 Hyper-V VM을 Azure로 마이그레이션'
  module: 'Module 7: Design for Migration'
ms.openlocfilehash: e97b3ecfbeac22c3eafe8caccd3cac3ed0081ffc
ms.sourcegitcommit: d2e9d886e710729f554d2ba62d1abe3c3f65fcb6
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/10/2022
ms.locfileid: "147046980"
---
# <a name="lab-migrating-hyper-v-vms-to-azure-by-using-azure-migrate"></a>랩: Azure Migrate를 사용하여 Hyper-V VM을 Azure로 마이그레이션

## <a name="lab-scenario"></a>랩 시나리오

Adatum Enterprise Architecture 팀은 Azure로의 마이그레이션의 일환으로 워크로드를 현대화하려는 포부에도 불구하고 공격적인 타임라인으로 인해 많은 경우 리프트 앤 시프트 접근 방식을 따라야 한다는 것을 알고 있습니다. 이 작업을 간소화하기 위해 팀은 Azure Migrate의 기능을 탐색하기 시작합니다. Azure Migrate는 온-프레미스 서버, 인프라, 애플리케이션 및 데이터를 평가하고 Azure로 마이그레이션할 수 있는 중앙 허브를 제공합니다.

Azure Migrate는 다음과 같은 기능을 제공합니다.

- 통합 마이그레이션 플랫폼: Azure로의 마이그레이션을 시작, 실행 및 추적하는 단일 포털입니다.
- 도구 범위: 평가 및 마이그레이션에 사용할 도구 범위입니다. 도구에는 Azure Migrate: Server Assessment 및 Azure Migrate: Server Migration에 연결합니다. Azure Migrate는 다른 Azure 서비스, 기타 도구 및 ISV(독립 소프트웨어 공급업체) 제품과 통합됩니다.
- 평가 및 마이그레이션: Azure Migrate 허브에서 다음을 평가하고 마이그레이션할 수 있습니다.
    - 서버: 온-프레미스 서버를 평가하고 Azure 가상 머신으로 마이그레이션합니다.
    - 데이터베이스: 온-프레미스 데이터베이스를 평가하고 Azure SQL Database 또는 SQL Managed Instance로 마이그레이션합니다.
    - 웹 애플리케이션: Azure App Service Migration Assistant를 사용하여 온-프레미스 웹 애플리케이션을 평가하고 Azure App Service로 마이그레이션합니다.
    - 가상 데스크톱: 온-프레미스 VDI(Virtual Desktop Infrastructure)를 평가하고 Azure의 Windows Virtual Desktop으로 마이그레이션합니다.
    - Data: Azure Data Box 제품을 사용하여 대량의 데이터를 Azure로 빠르고 비용 효율적으로 마이그레이션합니다.

데이터베이스, 웹앱, 가상 데스크톱은 마이그레이션 이니셔티브의 다음 단계 범위 내에 있지만 Adatum Enterprise Architecture 팀은 온-프레미스 Hyper-V 가상 머신을 Azure VM으로 마이그레이션하기 위해 Azure Migrate의 사용을 평가하는 것으로 시작하려고 합니다.

## <a name="objectives"></a>목표
  
이 랩을 완료하면 다음을 수행할 수 있습니다.

-  Azure Migrate를 사용하여 평가 및 마이그레이션 준비
-  Azure Migrate를 사용하여 마이그레이션에 대한 Hyper-V 평가
-  Azure Migrate를 사용하여 Hyper-V VM 마이그레이션

## <a name="estimated-time-120-minutes"></a>예상 시간: 120분

## <a name="lab-environment"></a>랩 환경
  
가상 머신: **AZ-801T00A-SEA-DC1** 및 **AZ-801T00A-SEA-SVR2** 를 실행해야 합니다. 다른 VM을 실행할 수도 있지만 이 랩에서는 필요하지 않습니다.

> **참고**: **AZ-801T00A-SEA-DC1**, **AZ-801T00A-SEA-SVR2** 가상 머신은 **SEA-DC1** 및 **SEA-SVR2** 의 설치를 호스팅합니다.

1. **SEA-SVR2** 를 선택합니다.
1. 다음 자격 증명을 사용하여 로그인합니다.

   - 사용자 이름: **Administrator**
   - 암호: **Pa55w.rd**
   - 도메인: **CONTOSO**

이 랩에서는 사용 가능한 VM 환경과 Azure 구독을 사용합니다. 랩을 시작하기 전에 구독에서 Owner 역할이 있는 Azure 구독 및 사용자 계정이 있는지 확인하세요.

## <a name="exercise-1-prepare-the-lab-environment"></a>연습 1: 랩 환경 준비

이 연습의 주요 작업은 다음과 같습니다.

1. Azure Resource Manager 빠른 시작 템플릿을 사용하여 Azure VM 배포
1. Azure Bastion 배포
1. Azure VM에 중첩된 VM 배포

#### <a name="task-1-deploy-an-azure-vm-by-using-an-azure-resource-manager-quickstart-template"></a>작업 1: Azure Resource Manager 빠른 시작 템플릿을 사용하여 Azure VM 배포

1. **SEA-SVR2** 에서 Microsoft Edge를 시작하고, [301-nested-vms-in-virtual-network Azure QuickStart 템플릿](https://github.com/az140mp/azure-quickstart-templates/tree/master/demos/nested-vms-in-virtual-network)을 찾아보고 **Azure에 배포** 를 선택합니다. 템플릿에서 만든 리소스 목록 뒤에 있는 `README.md` 파일에 **Azure에 배포** 단추가 있습니다.
1. 메시지가 나타나면 Azure Portal에서 이 랩에서 사용할 구독에서 Owner 역할이 있는 사용자 계정의 자격 증명을 사용하여 로그인합니다.
1. Azure Portal의 **중첩된 VM이 있는 Hyper-V 호스트 가상 머신** 페이지에서 다음 설정을 사용하여 배포를 수행합니다(다른 설정은 기본값으로 유지)

   | 설정 | 값 | 
   | --- | --- |
   | 구독 | 이 랩에서 사용 중인 Azure 구독의 이름 |
   | Resource group | 새 리소스 그룹 **AZ801-L0701-RG** 의 이름 |
   | 지역 | Azure VM을 프로비전할 수 있는 Azure 지역의 이름 |
   | Virtual Network 이름 | **az801l07a-hv-vnet** |
   | 호스트 네트워크 Interface1Name | **az801l07a-hv-vm-nic1** |
   | 호스트 네트워크 Interface2Name | **az801l07a-hv-vm-nic2** |
   | 호스트 가상 머신 이름 | **az801l07a-hv-vm** |
   | 호스트 관리자 사용자 이름 | **학생** |
   | 호스트 관리자 암호 | **Pa55w.rd1234** |

   > **참고**: 배포가 완료될 때까지 기다리세요. 배포는 10분 정도 걸릴 수 있습니다.

#### <a name="task-2-deploy-azure-bastion"></a>작업 2: Azure Bastion 배포 

> **참고**: Azure Bastion은 이 연습의 이전 작업에서 배포한 퍼블릭 엔드포인트 없이 Azure VM에 연결할 수 있으며 운영 체제 수준 자격 증명을 무차별 악용하지 못하도록 보호합니다.

1. **SEA-SVR2** 에서 Azure Portal을 표시하는 브라우저 창의 **Cloud Shell** 창에서 PowerShell 세션을 엽니다.
1. **Cloud Shell** 창의 PowerShell 세션에서 다음 명령을 실행하여 **AzureBastionSubnet** 이라는 서브넷을 이 연습에서 이전에 만든 가상 네트워크 **az801l07a-hv-vnet** 에 추가합니다.

   ```powershell
   $resourceGroupName = 'AZ801-L0701-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az801l07a-hv-vnet'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
    -Name 'AzureBastionSubnet' `
    -AddressPrefix 10.0.7.0/24 `
    -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. **Cloud Shell** 창을 닫습니다.
1. Azure Portal에서 다음 설정을 사용하여 Azure Bastion 인스턴스를 만듭니다.

   | 설정 | 값 | 
   | --- | --- |
   | 구독 | 이 랩에서 사용 중인 Azure 구독의 이름 |
   | Resource group | **AZ801-L0701-RG** |
   | 이름 | **az801l07a-bastion** |
   | 지역 | 이 연습의 이전 작업에서 리소스를 배포한 동일한 Azure 지역 |
   | 계층 | **기본** |
   | 가상 네트워크 | **az801l07a-hv-vnet** |
   | 서브넷 | **AzureBastionSubnet(10.0.7.0/24)** |
   | 공용 IP 주소 | **새로 만들기** |
   | 공용 IP 이름 | **az801l07a-hv-vnet-ip** |

   > **참고**: 배스천은 가상 네트워크와 동일한 Azure 지역에 만들어야 합니다.
   
   > **참고**: 배포가 완료될 때까지 기다린 후 다음 작업을 진행하세요. 배포는 5분 정도 걸릴 수 있습니다.

#### <a name="task-3-deploy-a-nested-vm-in-the-azure-vm"></a>작업 3: Azure VM에 중첩된 VM 배포

1. Azure Portal에서 **az801l07a-hv-vm** 가상 머신 페이지를 찾아봅니다.
1. **az801l07a-hv-vm** 페이지에서 다음 자격 증명을 사용하여 Azure Bastion을 통해 가상 머신 내에서 실행되는 운영 체제에 연결합니다.

   | 설정 | 값 | 
   | --- | --- |
   | 사용자 이름 |**학생** |
   | 암호 |**Pa55w.rd1234** |

1. **az801l07a-hv-vm** 에 대한 원격 데스크톱 세션 내의 **서버 관리자** 창에서 **IE 보안 강화 구성** 을 사용하지 않도록 설정합니다.
1. 파일 탐색기를 사용하여 두 개의 폴더 **F:\\VHDs** 및 **F:\\VMs** 를 만듭니다.
1. Microsoft Edge 사용하여 [Windows Server Evaluations](https://techcommunity.microsoft.com/t5/windows-11/accessing-trials-and-kits-for-windows-eval-center-workaround/m-p/3361125)에서 Windows Server 2022 **VHD** 파일을 다운로드하고 **F:\VHDs** 폴더에 복사합니다. 
1. 다음 설정을 사용하여 새 가상 머신을 만들려면 **Hyper-V 관리자** 를 사용합니다.

   | 설정 | 값 | 
   | --- | --- |
   | 이름 | **az801l07a-vm1** | 
   | 가상 컴퓨터를 다른 위치에 저장 | 선택 | 
   | 위치 | **F:\VMs** |
   | Generation | **1** |
   | 메모리 | static set to **2048** MB |
   | 연결 | **NestedSwitch** |
   | 가상 하드 디스크 | **F:\VHDs** 폴더에 다운로드된 VHD 파일 |

1. 새로 프로비전된 VM을 시작하고 가상 머신 연결을 사용하여 연결합니다. 
1. **가상 머신 연결** 창에서 운영 체제를 초기화합니다. 
1. 기본 제공 관리자 계정의 암호를 **Pa55w.rd** 로 설정합니다. 
1. **가상 머신 연결** 창에서 **az801l07a-vm1** 에 로그인합니다. 
1. **Windows PowerShell** 창을 열고 실행하여 컴퓨터 이름을 **az801l07a-vm1** 으로 설정합니다.

   ```powershell
   Rename-Computer -NewName 'az801l07a-vm1' -Restart
   ```

## <a name="exercise-2-prepare-for-assessment-and-migration-by-using-azure-migrate"></a>연습 2: Azure Migrate를 사용하여 평가 및 마이그레이션 준비
  
이 연습의 주요 작업은 다음과 같습니다.

1. Hyper-V 환경 구성
1. Azure Migrate 프로젝트를 만듭니다.
1. 대상 Azure 환경 구현

#### <a name="task-1-configure-hyper-v-environment"></a>작업 1: Hyper-V 환경 구성

1. **az801l07a-hv-vm** 에 대한 원격 데스크톱 세션 내의 브라우저 창에서 [https://aka.ms/migrate/script/hyperv](https://aka.ms/migrate/script/hyperv)로 이동하여 Azure Migrate 구성 PowerShell 스크립트를 다운로드합니다.

   >**참고**: 스크립트는 다음 작업을 수행합니다. 
   >- 지원되는 PowerShell 버전에서 스크립트를 실행 중인지 확인
   >- Hyper-V 호스트에 대한 관리 권한이 있는지 확인
   >- Azure Migrate 서비스가 Hyper-V 호스트와 통신하는 데 사용하는 로컬 사용자 계정을 만들 수 있습니다(이 사용자 계정은 Hyper-V 호스트의 **원격 관리 사용자**, **Hyper-V 관리자**, **성능 모니터 사용자** 그룹에 추가됨). 
   >- 호스트에서 지원되는 버전의 Hyper-V 및 Hyper-V 역할을 실행하는지 확인합니다.
   >- WinRM 서비스를 사용하도록 설정하고 호스트에서 포트 5985(HTTP) 및 5986(HTTPS)을 엽니다(메타데이터 수집에 필요) 
   >- 호스트에서 PowerShell 원격을 사용하도록 설정합니다.
   >- 호스트에서 관리하는 모든 VM에 Hyper-V Integration Services를 사용하도록 설정되어 있는지 확인합니다.
   >- 필요한 경우 호스트에서 CredSSP를 사용하도록 설정합니다.

1. **az801l07a-hv-vm** 에 대한 원격 데스크톱 세션 내에서 **Windows PowerShell ISE** 를 시작합니다. 
1. **관리자: Windows PowerShell ISE** 창에서 다음 명령을 실행하여 스크립트를 **C:\\Labfiles\\Lab07** 폴더에 복사하고 Zone.Identifier 대체 데이터 스트림을 제거합니다. 이는 파일이 인터넷에서 다운로드되었음을 나타냅니다.

   ```powershell
   New-Item -ItemType Directory -Path C:\Labfiles\Lab07 -Force
   Copy-Item -Path "$env:USERPROFILE\Downloads\MicrosoftAzureMigrate-Hyper-V.ps1" -Destination 'C:\Labfiles\Lab07'
   Unblock-File -Path C:\Labfiles\Lab07\MicrosoftAzureMigrate-Hyper-V.ps1
   Set-Location -Path C:\Labfiles\Lab07
   ```

1. **관리자: Windows PowerShell ISE** 창에서 **C:\\Labfiles\\Lab07** 폴더에 있는 **MicrosoftAzureMigrate-Hyper-V.ps1** 스크립트를 열고 실행합니다. 확인 메시지가 표시되면 **Y** 를 입력하고 Enter 키를 누릅니다. 단, 다음 메시지는 예외입니다. 이 경우에는 **N** 을 입력하고 Enter 키를 누릅니다.

   - SMB 공유를 사용하여 VHD를 저장하나요?
   - Azure Migrate 및 Hyper-V 호스트 통신에 대해 관리자가 아닌 로컬 사용자를 만들겠습니까? 

#### <a name="task-2-create-an-azure-migrate-project"></a>작업 2: Azure Migrate 프로젝트 만들기

1. **az801l07a-hv-vm** 에 대한 원격 데스크톱 세션 내의 브라우저 창에서 [Azure portal](https://portal.azure.com)로 이동하여 이 랩에서 사용 중인 구독에서 Owner 역할이 있는 사용자 계정의 자격 증명을 사용하여 로그인합니다.
1. Azure Portal에서 **Azure Migrate** 페이지를 찾아보고 다음 설정을 사용하여 프로젝트를 만듭니다(다른 설정은 기본값으로 유지)

   | 설정 | 값 | 
   | --- | --- |
   | 구독 | 이 랩에서 사용 중인 Azure 구독의 이름 |
   | Resource group | 새 리소스 그룹 **AZ801-L0702-RG** 의 이름 |
   | 프로젝트 마이그레이션 | **az801l07a-migrate-project** |
   | Geography | 국가 또는 지리적 지역의 이름 |

#### <a name="task-3-implement-the-target-azure-environment"></a>작업 3: 대상 Azure 환경 구현

1. Azure Portal에서 다음 설정을 사용하여 가상 네트워크를 만듭니다(다른 설정은 기본값으로 유지)

   | 설정 | 값 |
   | --- | --- |
   | 구독 | 이 랩에서 사용 중인 Azure 구독의 이름 |
   | Resource group | 새 리소스 그룹 **AZ801-L0703-RG** 의 이름 |
   | 이름 | **az801l07a-migration-vnet** |
   | 지역 | 이 랩의 앞부분에서 가상 머신을 배포한 Azure 지역의 이름 |
   | 서브넷 이름 | **subnet0** |
   | 서브넷 주소 범위 | **10.7.0.0/24** |

1. Azure Portal에서 다음 설정을 사용하여 다른 가상 네트워크를 만듭니다(다른 설정은 기본값으로 유지)

   | 설정 | 값 |
   | --- | --- |
   | 구독 | 이 랩에서 사용 중인 Azure 구독의 이름 |
   | Resource group | **AZ801-L0703-RG** |
   | 이름 | **az801l07a-test-vnet** |
   | 지역 | 이 랩의 앞부분에서 가상 머신을 배포한 Azure 지역의 이름 |
   | 설정 | 값 |
   | --- | --- |
   | 서브넷 이름 | **subnet0** |
   | 서브넷 주소 범위 | **10.7.0.0/24** |

1. Azure Portal에서 다음 설정을 사용하여 스토리지 계정을 만듭니다(다른 설정은 기본값으로 유지)

   | 설정 | 값 | 
   | --- | --- |
   | 구독 | 이 랩에서 사용 중인 Azure 구독의 이름 |
   | Resource group | **AZ801-L0703-RG** |
   | 스토리지 계정 이름 | 문자 및 숫자로 구성되고 3~24자 사이의 전역적으로 고유한 이름 |
   | 위치 | 이 작업 앞부분에서 가상 네트워크를 만든 Azure 지역의 이름 |
   | 성능 | **Standard** |
   | 중복 | **LRS(로컬 중복 스토리지)** |
   | Blob에 일시 삭제를 사용하도록 설정 | 사용 안 함 |
   | 컨테이너에 대해 일시 삭제 사용 | 사용 안 함 |

## <a name="exercise-3-assess-hyper-v-for-migration-by-using-azure-migrate"></a>연습 3: Azure Migrate를 사용하여 마이그레이션에 대한 Hyper-V 평가
  
이 연습의 주요 작업은 다음과 같습니다.

1. Azure Migrate 어플라이언스 배포 및 구성
1. 평가 구성, 실행, 보기

#### <a name="task-1-deploy-and-configure-the-azure-migrate-appliance"></a>작업 1: Azure Migrate 어플라이언스 배포 및 구성

1. **az801l07a-hv-vm** 에 대한 원격 데스크톱 세션 내 브라우저 창의 Azure Portal에서 새로 만든 Azure Migrate 프로젝트를 찾아봅니다. 
1. **Azure Migrate | 시작** 페이지의 **서버, 데이터베이스 및 웹앱** 섹션에서 **검색, 평가 및 마이그레이션** 을 선택한 다음, **검색** 페이지로 이동합니다.
1. **검색** 페이지에서 **어플라이언스를 사용한 검색** 옵션이 선택되어 있는지 확인하고 **서버가 가상화되었습니까?** 설정의 값은 **예, Hyper-V 사용** 으로 설정합니다. 
1. **검색** 페이지에서 어플라이언스 값을 **az801l07a-vma1** 로 설정하고 해당 키를 생성합니다.

   >**참고**: 키 생성이 완료될 때까지 기다렸다 값을 기록합니다. 나중에 이 연습에서 필요할 것입니다.

1. **검색** 페이지에서 어플라이언스 VM의 **.VHD 파일** 을 **F:\VMs** 폴더에 다운로드합니다.

   >**참고**: 다운로드가 완료될 때까지 기다립니다. 5분 정도 걸릴 수 있습니다.

1. 다운로드가 완료되면 다운로드한 .ZIP 파일의 콘텐츠를 **F:\VMs** 폴더로 추출합니다.

   >**참고**: Microsoft Edge에는 기본적으로 프롬프트가 표시되지 않으므로 수동으로 .VHD 파일을 F:\VMs 폴더로 복사해야 할 수 있습니다.

1. **az801l07a-hv-vm** 에 대한 원격 데스크톱 세션 내에서 **Hyper-V 관리자** 콘솔을 사용하여 VM 파일을 복사한 가상 머신을 **F:\VMs** 폴더로 가져옵니다. **가상 머신을 현재 위치에 등록(기존 고유 ID 사용)** 옵션을 사용하고 **가상 프로세서 수** 를 **4** 로 설정한 다음, **NestedSwitch** 에 연결합니다.

   >**참고**: 랩 환경에서는 가상 프로세서 수의 변경과 관련된 오류 메시지를 무시할 수 있습니다. 프로덕션 시나리오에서는 가상 어플라이언스에 충분한 수의 컴퓨팅 리소스가 할당되어 있는지 확인해야 합니다.

1. 새로 가져온 가상 머신의 이름을 **az801l07a-vma1** 로 변경합니다.
1. 새로 가져온 가상 머신을 시작하고 **가상 머신 연결** 창을 사용하여 연결합니다. 
1. 가상 어플라이언스에 대한 **가상 머신 연결** 창에서 **사용 조건** 에 동의하고 기본 제공 관리자 계정의 암호를 **Pa55w.rd** 로 설정합니다. 
1. 가상 어플라이언스에 대한 **가상 머신 연결** 창의 **작업** 메뉴에서 **Ctrl + Alt + Delete** 를 선택한 다음, 메시지가 표시되면 새로 설정된 암호를 사용하여 로그인합니다.

   >**참고**: 가상 어플라이언스에 대한 **가상 머신 연결** 창 내에서 **어플라이언스 구성 관리자** 를 표시하는 브라우저 창이 자동으로 열립니다.

1. **어플라이언스 구성 관리자** 페이지에서 **동의** 단추를 선택하고 설치 필수 구성 요소가 성공적으로 확인될 때까지 기다립니다. 
1. **어플라이언스 구성 관리자** 페이지의 **여기에 키를 붙여넣어 Hyper-V 어플라이언스를 등록하세요.** 텍스트 상자에 이 연습의 앞부분에서 복사한 키를 메모장 붙여넣고 **로그인** 을 선택한 다음, 디바이스 코드를 사용하여 구독에 로그인합니다. 
1. **Microsoft Azure PowerShell 로그인하려고 하시나요?** 라는 메시지가 표시되면 **계속** 을 선택하고 새로 연 브라우저 탭을 닫습니다.
1. **어플라이언스 구성 관리자** 페이지에서 등록이 성공했는지 확인합니다.
1. **어플라이언스 구성 관리자** 페이지의 **자격 증명 및 검색 원본 관리** 섹션에서 다음 설정을 사용하여 자격 증명을 추가합니다.

   | 설정 | 값 | 
   | --- | --- |
   | 친숙한 이름 | **az801l07ahvcred** | 
   | 사용자 이름 | **학생** |
   | 암호 | **Pa55w.rd1234** |

1. 브라우저 창 내에 있는 **어플라이언스 구성 관리자** 페이지의 **Hyper-V 호스트/클러스터 세부 정보 제공** 섹션에서 **Hyper-V 호스트/클러스터** 에 검색 원본 집합을 추가하고, **식별 이름** 을 **az801l07ahvcred** 로 설정하고, **IP 주소/FQDN** 을 **10.0.2.1** 로 설정합니다.

   >**참고**: **10.0.2.1** 은 내부 스위치에 연결된 Hyper-V 호스트의 네트워크 인터페이스 IP 주소입니다.

1. **어플라이언스 Configuration Manager** 페이지의 **Hyper-V 호스트/클러스터 세부 정보 제공** 섹션에서 **이러한 기능을 수행하지 않으려면 슬라이더 사용 안 함** 의 토글 단추를 사용하도록 설정한 다음 **검색 시작** 을 선택합니다. 

   >**참고**: 검색된 서버의 메타데이터가 Azure Portal에 표시되려면 호스트당 약 15분이 걸릴 수 있습니다.

#### <a name="task-2-configure-run-and-view-an-assessment"></a>작업 2: 평가 구성, 실행, 보기

1. 가상 어플라이언스에 대한 **가상 머신 연결** 창에서 **으로 az801l07a-hv-vm** 에 대한 원격 데스크톱 세션으로 전환합니다. Azure Portal을 표시하는 브라우저 창에서 **Azure Migrate | 서버, 데이터베이스 및 웹앱** 페이지로 다시 돌아가서 **새로 고침** 을 선택합니다. **Azure Migrate: 검색 및 평가** 섹션에서 **평가** 를 선택한 다음, 드롭다운 메뉴에서 **Azure VM** 을 선택합니다.
1. **평가 만들기** 페이지에서 다음 설정을 지정합니다(다른 설정은 기본값으로 유지).

   | 설정 | 값 | 
   | --- | --- |
   | 대상 위치 | 이 랩에서 사용 중인 Azure 지역의 이름 |
   | 스토리지 유형 | **프리미엄 관리 디스크** |
   | 예약 인스턴스 | **예약 인스턴스 없음** |
   | 크기 조정 기준 | **온-프레미스로** |
   | VM 시리즈 | **Dsv3_series** |
   | 쾌적 인자 | **1** |
   | 제안 | **종량제** |
   | 통화 | 미국 달러($) | 
   | 할인 | **0** |
   | VM 작동 시간 | 매월 **31** 일 및 매일 **24** 시간 | 
   | 평가 이름 | az801l07a-assessment** |
   | 그룹 선택 또는 만들기 | **az801l07a-assessment-group** |
   | 그룹에 추가할 머신 목록 | **az801l07a-vm1** |

   >**참고**: 랩 환경에 내재된 제한된 시간을 고려할 때 이 경우 실행 가능한 유일한 옵션은 **온-프레미스로** 평가입니다. 

1. **Azure Migrate \| 서버, 데이터베이스, 웹앱** 페이지로 돌아가서 **새로 고침** 을 선택합니다. **Azure Migrate: 검색 및 평가** 섹션에서 **평가** **합계** 줄에 **1** 개의 항목이 포함되어 있는지 확인한 다음 선택합니다.
1. **Azure Migrate: 검색 및 평가 \| 평가** 페이지에서 새로 만든 평가 **az801l07a-assessment** 를 선택합니다. 
1. **az801l07a-assessment** 페이지에서 컴퓨팅 및 스토리지 모두에 대한 Azure 준비 상태 및 월별 예상 비용을 나타내는 정보를 검토합니다. 

   >**참고**: 실제 시나리오에서는 종속성 에이전트를 설치하여 평가 단계에서 서버 종속성에 대한 더 많은 인사이트를 제공하는 것이 좋습니다.

## <a name="exercise-4-migrate-hyper-v-vms-by-using-azure-migrate"></a>연습 4: Azure Migrate를 사용하여 Hyper-V VM 마이그레이션
  
이 연습의 주요 작업은 다음과 같습니다.

1. Hyper-V VM 마이그레이션 준비
1. Hyper-V VM의 복제 구성
1. Hyper-V VM 마이그레이션 수행
1. 랩에 배포된 Azure 리소스 제거

#### <a name="task-1-prepare-for-migration-of-hyper-v-vms"></a>작업 1: Hyper-V VM 마이그레이션 준비

1. **az801l07a-hv-vm** 에 대한 원격 데스크톱 세션 내에서 Azure Portal을 표시하는 브라우저 창에서 **Azure Migrate | 서버, 데이터베이스 및 웹앱** 페이지로 다시 돌아갑니다. 
1. **Azure Migrate | 서버, 데이터베이스 및 웹앱** 페이지의 **Azure Migrate: 서버 마이그레이션** 섹션에서 **검색** 링크를 선택합니다. 
1. **평가** 페이지에서 다음 설정을 지정하여 리소스를 만듭니다(다른 설정은 기본값으로 유지).

   | 설정 | 값 | 
   | --- | --- |
   | 컴퓨터가 가상화되어 있나요? | **예, Hyper-V를 사용하겠습니다.** |
   | 대상 지역 | 이 랩에서 사용 중인 Azure 지역의 이름 | 
   | 마이그레이션의 대상 지역을 확인합니다. | 선택 |

   >**참고**: 이 단계는 Azure Site Recovery 자격 증명 모음의 프로비저닝을 자동으로 트리거합니다.

1. **검색** 페이지의 **1. Hyper-V 호스트 서버를 준비합니다.** 에서 Hyper-V 복제 공급자 소프트웨어 설치 관리자를 다운로드하려면 첫 번째 **다운로드** 링크(**다운로드** 단추 아님)를 선택합니다.

   > **참고:** **AzureSiteRecoveryProvider.exe를 안전하게 다운로드할 수 없습니다** 라는 브라우저 알림을 받으면 **다운로드** 링크의 상황에 맞는 메뉴를 표시한 다음, 메뉴에서 **링크 복사** 를 선택합니다. 동일한 브라우저 창에서 다른 탭을 열고 복사한 링크를 붙여넣은 다음, Enter 키를 누릅니다.

1. 다운로드한 파일을 사용하여 기본 설정으로 **Azure Site Recovery 공급자** 를 설치합니다.
1. 설치하는 동안 Azure Portal로 전환한 다음, **머신 검색** 페이지의 온-프레미스 Hyper-V 호스트 준비 절차 1단계에서 **다운로드** 단추를 선택하여 자격 증명 모음 등록 키를 다운로드하고 이를 사용하여 **Azure Site Recovery 공급자** 를 등록합니다.
1. **검색** 페이지를 표시하는 브라우저 창을 새로 고칩니다. 
1. **Azure Migrate | 서버, 데이터베이스 및 웹앱** 페이지의 **Azure Migrate: 서버 마이그레이션** 섹션에서 **검색** 링크를 선택합니다. 
1. **검색** 페이지에서 등록을 완료합니다.

   >**참고**: 가상 머신 검색을 완료하는 데 최대 15분이 걸릴 수 있습니다.

#### <a name="task-2-configure-replication-of-hyper-v-vms"></a>작업 2: Hyper-V VM 복제 구성

1. 등록이 완료되었다는 확인 메시지가 표시되면 **Azure Migrate | 서버, 데이터베이스 및 웹앱** 페이지로 다시 이동하여 **Azure Migrate: 서버 마이그레이션** 섹션에서 **복제** 링크를 선택합니다. 

   >**참고**: **Azure Migrate | 서버, 데이터베이스 및 웹앱** 페이지를 표시하는 브라우저 페이지를 새로 고쳐야 할 수 있습니다.

1. **복제** 페이지의 **머신이 가상화되었습니까?** 드롭다운 목록에서 **예, Hyper-V 사용** 을 선택합니다.
1. **복제** 페이지의 **가상 머신** 탭에서 다음 설정을 지정합니다(다른 설정은 기본값으로 유지).

   | 설정 | 값 | 
   | --- | --- |
   | Azure 마이그레이션 평가에서 마이그레이션 설정 가져오기 | **예, Azure 마이그레이션 평가에서 마이그레이션 설정 적용** |
   | 그룹 선택 | **az801l07a-assessment-group** |
   | 평가 선택 | **az801l07a-assessment** |
   | 가상 머신 | **az801l07a-vm1** |

1. **복제** 페이지의 **대상 설정** 탭에서 다음 설정을 지정합니다(다른 설정은 기본값으로 유지).

   | 설정 | 값 | 
   | --- | --- |
   | 구독 | 이 랩에서 사용 중인 Azure 구독의 이름 |
   | Resource group | **AZ801-L0703-RG** |
   | 복제 스토리지 계정 | 이 랩의 앞부분에서 만든 스토리지 계정의 이름 | 
   | Virtual Network | **az801l07a-migration-vnet** |
   | 서브넷 | **subnet0** |

1. **복제** 페이지의 **컴퓨팅** 탭에서 **Azure VM 크기** 드롭다운 목록에서 **Standard_D2s_v3** 이 선택되어 있는지 확인합니다. **OS 유형** 드롭다운 목록에서 **Windows** 를 선택합니다.
1. 복제 상태를 모니터링하려면 **Azure Migrate | 서버, 데이터베이스 및 웹앱** 페이지로 돌아가서 **새로 고침** 을 선택한 다음, **Azure Migrate: 서버 마이그레이션** 섹션에서 **서버 복제** 항목을 선택합니다. **Azure Migrate: 서버 마이그레이션 | 머신 복제** 페이지에서 복제 머신 목록에서 **상태** 열을 검사합니다. 
1. 상태가 **보호됨** 으로 변경될 때까지 기다립니다. 이 작업은 15분이 더 걸릴 수 있습니다.

   >**참고**: **Azure Migrate: 서버 마이그레이션 | 머신 복제** 를 새로 고쳐 **상태** 정보를 업데이트합니다.

#### <a name="task-3-perform-migration-of-hyper-v-vms"></a>작업 3: Hyper-V VM 마이그레이션 수행

1. Azure Portal의 **Azure Migrate: 서버 마이그레이션 | 머신 복제** 페이지에서 **az801l07a-vm1** 가상 머신을 나타내는 항목을 선택합니다.
1. **az801l07a-vm1** 페이지에서 **az801l07a-test-vnet** 가상 네트워크를 대상으로 사용하여 **테스트 마이그레이션** 을 시작합니다.

   >**참고**: 테스트 마이그레이션이 완료되기를 기다립니다. 5분 정도 걸릴 수 있습니다.

1. Azure Portal에서 **가상 머신** 페이지로 이동하여 새로 복제된 가상 머신 **az801l07a-vm1-test** 를 나타내는 항목을 확인합니다.

   > **참고:** 처음에 가상 머신의 이름은 **asr-** 접두사와 임의로 생성된 접미사로 구성되지만 결국 **az801l07a-vm1-test** 로 이름이 바뀝니다.

1. Azure Portal에서 **Azure Migrate: 서버 마이그레이션 | 머신 복제** 페이지로 다시 이동하여 **새로 고침** 을 선택한 다음, **az801l07a-vm1** 가상 머신이 **테스트 장애 조치(failover) 정리 보류 중** 상태로 나열되어 있는지 확인합니다.
1. **Azure Migrate: 서버 마이그레이션 | 머신 복제** 페이지에서 **az801l07a-vm1** 복제 머신 페이지로 이동한 다음, **테스트 마이그레이션 정리** 작업을 트리거하여 **테스트가 완료되었습니다. 테스트 가상 머신을 삭제하세요.** 를 지정합니다.
1. 테스트 장애 조치 정리 작업이 완료된 후에는 **az801l07a-vm1** 머신 복제 페이지를 표시하는 브라우저 페이지를 새로 고치면 도구 모음의 **마이그레이션** 아이콘을 자동으로 사용할 수 있게 됩니다.
1. **az801l07a-vm1** 머신 복제 페이지에서 **마이그레이션** 작업을 트리거합니다. 
1. **마이그레이션** 페이지에서 **마이그레이션 전에 머신을 종료하여 데이터 손실을 최소화합니까?** 옵션이 선택되어 있는지 확인합니다.
1. 마이그레이션 상태를 모니터링하려면 **Azure Migrate | 서버, 데이터베이스 및 웹앱** 페이지로 다시 이동합니다. **Azure Migrate: 서버 마이그레이션** 섹션에서 **서버 복제** 항목을 선택한 다음, **Azure Migrate: 서버 마이그레이션 | 머신 복제** 페이지에서 머신 복제 목록에서 **상태** 열을 검사합니다. 상태가 **계획된 장애 조치(failover) 완료** 상태를 표시하는지 확인합니다.

   >**참고**: 마이그레이션은 되돌릴 수 없는 작업이어야 합니다. 완료된 정보를 보려면 **Azure Migrate | 서버, 데이터베이스 및 웹앱** 페이지로 다시 이동하여 페이지를 새로 고친 다음, **Azure Migrate: 서버 마이그레이션** 섹션의 **마이그레이션된 서버** 항목 값이 **1** 인지 확인합니다.

#### <a name="task-4-remove-azure-resources-deployed-in-the-lab"></a>작업 4: 랩에 배포된 Azure 리소스 제거

1. **SEA-SVR2** 에서 Azure Portal을 표시하는 브라우저 창의 **Azure Cloud Shell** 창에서 PowerShell 세션을 엽니다.
1. **Cloud Shell** 창에서 다음 명령을 실행하여 이 랩에서 만든 리소스 그룹을 모두 나열합니다.

   ```powershell
   Get-AzResourceGroup -Name 'AZ801-L070*'
   ```

   > **참고**: 출력에 이 랩에서 만든 리소스 그룹만 포함되어 있는지 확인합니다. 이 그룹은 이 작업에서 삭제됩니다.

1. **Cloud Shell** 창에서 다음을 실행하여 이 랩에서 만든 리소스 그룹을 삭제합니다.

   ```powershell
   Get-AzResourceGroup -Name 'AZ801-L070*' | Remove-AzResourceGroup -Force -AsJob
   ```

   > **참고:** 이 명령은 *-AsJob* 매개 변수에 의해 결정되어 비동기로 실행되므로, 동일한 PowerShell 세션 내에서 이 명령을 실행한 직후 다른 PowerShell 명령을 실행할 수 있지만 리소스 그룹이 실제로 제거되기까지는 몇 분 정도 걸립니다.
