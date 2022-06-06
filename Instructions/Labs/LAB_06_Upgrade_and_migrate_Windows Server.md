---
lab:
  title: '랩: Windows Server에서 업그레이드 및 마이그레이션'
  module: 'Module 6: Upgrade and migrate in Windows Server'
ms.openlocfilehash: 33f6ea57f4ffc31fb0f7279a9e0df9100039e29b
ms.sourcegitcommit: fb0d39e25bc0fe182037587b772d217db126d3bb
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/07/2022
ms.locfileid: "144813001"
---
# <a name="lab-upgrade-and-migrate-in-windows-server"></a>랩: Windows Server에서 업그레이드 및 마이그레이션

## <a name="lab-scenario"></a>랩 시나리오

Contoso는 온-프레미스 Windows 서버를 Azure VM(가상 머신)으로 쉽게 마이그레이션할 수 있는 인프라 서비스에 적합한 하이브리드 모델을 찾고 있습니다. 이 이니셔티브를 지원하기 위해 Azure VM에서 AD DS(Active Directory Domain Services) 도메인 컨트롤러를 배포하는 프로세스를 평가하는 임무를 맡았습니다. 현재 온-프레미스 배포에 사용되는 수동 프로세스와 Azure에서 사용할 수 있는 배포 방법 간의 차이점을 파악하려고 합니다. 또한 Storage Migration Services 기능을 테스트하고 문서화하여 온-프레미스 파일 서버 마이그레이션에 사용하는 것이 적합한지 검증하려고 합니다. 

## <a name="objectives"></a>목표

이 랩에서는 다음을 수행합니다.

- Azure에서 AD DS 도메인 컨트롤러를 배포합니다.
- Storage Migration Service를 사용하여 파일 서버를 마이그레이션합니다.

## <a name="estimated-time-60-minutes"></a>예상 시간: 60분

## <a name="lab-environment"></a>랩 환경
  
가상 머신: **AZ-801T00A-SEA-DC1**, **AZ-801T00A-SEA-SVR1**, **AZ-801T00A-SEA-SVR2** 를 실행해야 합니다. 다른 VM을 실행할 수도 있지만 이 랩에서는 필요하지 않습니다.

> **참고**: **AZ-801T00A-SEA-DC1**, **AZ-801T00A-SEA-SVR1**, **AZ-801T00A-SEA-SVR2** 가상 머신은 각각 **SEA-DC1**, **SEA-SVR1**, **SEA-SVR2** 의 설치를 호스트합니다.

1. **SEA-SVR2** 를 선택합니다.
1. 다음 자격 증명을 사용하여 로그인합니다.

   - 사용자 이름: **Administrator**
   - 암호: **Pa55w.rd**
   - 도메인: **CONTOSO**

이 랩에서는 사용 가능한 VM 환경과 Azure 구독을 사용합니다. 랩을 시작하기 전에 구독에서 Owner 역할이 있는 Azure 구독 및 사용자 계정이 있는지 확인하세요.

## <a name="exercise-1-deploying-ad-ds-domain-controllers-in-azure"></a>연습 1: Azure에서 AD DS 도메인 컨트롤러 배포

> **참고**: 하이브리드 시나리오에는 일반적으로 기존 온-프레미스 도메인에서 Azure VM으로 추가 도메인 컨트롤러를 배포하여 온-프레미스 AD DS 환경을 Azure로 확장하는 작업이 포함됩니다. 랩에서 이러한 작업을 수행하려면 Azure 가상 네트워크에 대한 사이트 간 VPN 연결을 설정하거나 Azure에서 전체 랩 환경을 프로비전해야 하며, 그 중 일부는 온-프레미스 사이트를 에뮬레이트합니다. 간단히 하기 위해 이 연습에는 Azure VM의 도메인 컨트롤러를 새 포리스트 및 도메인에 배포하는 작업이 포함됩니다. Azure VM을 사용할 때 도메인 컨트롤러 구성 및 프로비전 프로세스의 고유한 측면을 식별하는 데 중점을 둡니다.

이 연습의 주요 작업은 다음과 같습니다.

1. ARM(Azure Resource Manager) 템플릿을 사용하여 도메인 컨트롤러를 배포합니다.
1. Azure Bastion을 배포합니다. 
1. Azure Portal을 사용하여 Azure VM을 배포합니다.
1. Azure VM에서 수동으로 도메인 컨트롤러를 승격합니다.
1. 연습에 배포된 Azure 리소스를 제거합니다.

#### <a name="task-1-deploy-a-domain-controller-by-using-an-azure-resource-manager-arm-template"></a>작업 1: ARM(Azure Resource Manager) 템플릿을 사용하여 도메인 컨트롤러 배포

1. **SEA-SVR2** 에서 Microsoft Edge를 시작하고 **[Azure Portal](https://portal.azure.com)** 로 이동한 후 이 랩에서 사용할 구독에서 Owner 역할이 있는 사용자 계정의 자격 증명을 사용하여 로그인합니다.
1. **SEA-SVR2** 에서 Microsoft Edge를 시작하고 **[새 Windows VM 만들기 및 새 AD 포리스트, 도메인 및 DC 만들기](https://github.com/az140mp/azure-quickstart-templates/tree/master/application-workloads/active-directory/active-directory-new-domain)** 에서 사용자 지정 버전의 QuickStart 템플릿으로 이동합니다. 
1. **새 Windows VM 만들기 및 새 AD 포리스트, 도메인 및 DC 만들기** 페이지에서 Azure에 배포를 시작합니다. 
1. **새 AD 포리스트로 Azure VM 만들기** 페이지에서 **템플릿 편집** 을 선택합니다.
1. **템플릿 편집** 페이지에서 **storageProfile** 섹션(**195** 행부터)으로 이동하여 **sku**(**199** 행)이 **2022-Datacenter** 로, **dataDisks** **캐싱**(**213** 행)이 **없음** 으로 설정되어 있는지 확인합니다.

   > **참고**: AD DS 데이터베이스 및 로그 파일을 호스트하는 디스크의 캐싱은 **없음** 으로 설정되어야 합니다.

1. **템플릿 편집** 페이지에서 **확장** 섹션(**233** 행부터)으로 이동하여 템플릿이 PowerShell Desired State Configuration을 사용하여 배포된 Azure VM(가상 머신) 내에서 **CreateADPDC.ps1** 스크립트를 실행함을 확인합니다.

   > **참고**: 스크립트를 검토하려면 다음 단계를 사용할 수 있습니다.

   1. **SEA-SVR2** 에서 Microsoft Edge 창에서 다른 탭을 열고 **[새 Windows VM 만들기 및 새 AD 포리스트, 도메인 및 DC 만들기](https://github.com/az140mp/azure-quickstart-templates/tree/master/application-workloads/active-directory/active-directory-new-domain)** 에서 사용자 지정 버전의 QuickStart 템플릿으로 이동합니다.
   1. **새 Windows VM 만들기 및 새 AD 포리스트, 도메인 및 DC** 페이지의 리포지토리 콘텐츠 목록에서 **DSC** 폴더를 선택한 다음, **CreateADPDC.ps1** 파일을 선택합니다.
   1. **azure-quickstart-templates/application-workloads/active-directory/active-directory-new-domain/DSC/CreateADPDC.ps1** 페이지에서 스크립트의 콘텐츠를 검토하고 Active Directory Domain Services 및 DNS를 비롯한 여러 서버 역할을 설치하여 NTDS 데이터베이스와 로그는 물론 SYSOVL 공유를 드라이브 **F** 에 배치합니다. 
   1. Microsoft Edge 탭을 닫고 Azure Portal에서 **템플릿 수정** 페이지를 표시하는 탭으로 다시 전환합니다.

1. **템플릿 수정** 페이지에서 가용성 집합을 프로비전하는 섹션(**110** 행부터)으로 이동하여 템플릿이 가용성 집합을 만들고 VM을 여기에 배포함을 확인합니다(**181** 행 **dependsOn** 요소로 표시됨).

   > **참고**: 이 연습의 뒷부분에서는 다른 Azure VM을 동일한 가용성 집합에 배포하고 동일한 도메인에서 추가 도메인 컨트롤러로 구성합니다. 가용성 집합을 사용하면 복원력이 추가됩니다.

1. Azure VM의 네트워크 인터페이스를 프로비전하는 섹션(**110** 행부터)으로 이동하여 개인 IP 주소 할당 방법이 **고정**(**164** 행)으로 설정됨을 확인합니다.

   > **참고**: 정적 할당은 도메인 컨트롤러를 배포할 때 일반적으로 사용되지만 DNS 서버 역할을 호스트하는 서버에는 필수적으로 사용해야 합니다.

1. 중첩된 템플릿을 배포하는 섹션(**266** 행부터)으로 이동하여 템플릿이 DNS 서버 역할이 설치된 도메인 컨트롤러로 작동하는 Azure VM을 호스트하는 가상 네트워크 내에서 DNS 서버 주소를 업데이트함을 확인합니다.

   > **참고**: DNS 서버 역할을 사용하여 도메인 컨트롤러를 실행하는 Azure VM을 가리키는 사용자 지정 DNS 서버 가상 네트워크 설정을 구성하면 이후에 동일한 가상 네트워크에 배포된 모든 Azure VM이 이름 확인을 위해 해당 DNS 서버를 자동으로 사용하여 도메인 조인 기능을 효과적으로 제공할 수 있습니다.

1. 템플릿에 변경 내용을 적용하지 않고 **템플릿 수정** 페이지를 닫습니다.
1. **새 AD 포리스트로 Azure VM 만들기** 페이지로 돌아가서 **매개 변수 편집** 을 선택합니다.
1. **매개 변수 편집** 페이지에서 **C:\\Labfiles\\Lab06\\L06-rg_template.parameters.json** 파일을 업로드하여 기본 매개 변수를 바꿉니다.
1. 다음 설정을 사용하여 배포를 시작합니다(다른 설정은 기본값으로 유지).

   | 설정 | 값 | 
   | --- | --- |
   | 구독 | 이 랩에서 사용 중인 Azure 구독의 이름 |
   | 리소스 그룹 | 새 리소스 그룹 **AZ801-L0601-RG** 의 이름 |
   | 지역 | Azure VM을 프로비전할 수 있는 Azure 지역의 이름 |
   | 관리자 사용자 이름 | **학생** |
   | 관리자 암호 | **Pa55w.rd1234** |
   | 도메인 이름 | **contoso.com** |
   | Vm 크기 | **Standard_DS2_v2** |
   | 가상 컴퓨터 이름 | **az801l06a-dc1** |
   | Virtual Network 이름 | **az801l06a-vnet** |
   | 가상 네트워크 주소 범위 | **10.6.0.0/16** |
   | 네트워크 인터페이스 이름 | **az801l06a-dc1-nic1** |
   | 개인 IP 주소 | **10.6.0.4** |
   | 서브넷 이름 | **adSubnet** |
   | 서브넷 범위 | **10.6.0.0/24** |
   | 가용성 집합 이름 | **adAvailabilitySet** |

   > **참고**: 배포가 완료될 때까지 기다린 후 다음 작업을 진행하세요. 15분 정도 걸릴 수 있습니다. 

#### <a name="task-2-deploy-azure-bastion"></a>작업 2: Azure Bastion 배포 

> **참고**: Azure Bastion은 이 연습의 이전 작업에서 배포한 퍼블릭 엔드포인트 없이 Azure VM에 연결할 수 있으며 운영 체제 수준 자격 증명을 무차별 암호 대입으로 악용하지 못하도록 보호합니다.

1. **SEA-SVR2** 의 Azure Portal을 표시하는 Microsoft Edge 창에서 Cloud Shell의 PowerShell 세션을 엽니다.
1. Cloud Shell 창의 PowerShell 세션에서 다음 명령을 실행하여 **AzureBastionSubnet** 이라는 서브넷을 이 연습의 앞부분에서 만든 가상 네트워크 **az801l06a-vnet** 에 추가합니다.

   ```powershell
   $resourceGroupName = 'AZ801-L0601-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az801l06a-vnet'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
    -Name 'AzureBastionSubnet' `
    -AddressPrefix 10.6.255.0/24 `
    -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. Azure Portal에서 다음 설정을 사용하여 Azure Bastion을 배포합니다.

   | 설정 | 값 | 
   | --- | --- |
   | 구독 | 이 랩에서 사용 중인 Azure 구독의 이름 |
   | 리소스 그룹 | 새 리소스 그룹 **AZ801-L0602-RG** 의 이름 |
   | 이름 | **az801l06a-bastion** |
   | 지역 | 이 연습의 이전 작업에서 리소스를 배포한 동일한 Azure 지역 |
   | 계층 | **기본** |
   | 가상 네트워크 | **az801l06a-vnet** |
   | 서브넷 | **AzureBastionSubnet(10.6.255.0/24)** |
   | 공용 IP 주소 | **새로 만들기** |
   | 공용 IP 이름 | **az801l06a-vnet-ip** |

   > **참고**: 배포가 완료될 때까지 기다리지 말고 다음 작업을 진행하세요. 배포는 5분 정도 걸릴 수 있습니다.

#### <a name="task-3-deploy-an-azure-vm-by-using-the-azure-portal"></a>작업 3: Azure Portal을 사용하여 Azure VM을 배포합니다.

> **참고**: 이 연습의 첫 번째 작업에서 프로비전한 첫 번째 도메인과 동일한 도메인에 있는 추가 도메인 컨트롤러로서 두 번째 Azure VM 및 설정의 배포를 완전히 자동화할 수 있습니다. 그러나 이 경우 그래픽 인터페이스를 사용하면 온-프레미스 시나리오와 Azure 기반 시나리오에서 도메인 컨트롤러 프로비전 간의 차이점에 대한 추가 지침을 제공해야 합니다.

1. **SEA-SVR2** 의 Azure Portal을 표시하는 Microsoft Edge 창에서 다음 설정을 사용하여 가상 머신을 만듭니다(다른 설정은 기본값으로 유지).

   | 설정 | 값 |
   | --- | --- |
   | 구독 | 이 랩에서 사용 중인 Azure 구독의 이름 |
   | 리소스 그룹 | 새 리소스 그룹 **AZ801-L0601-RG** 의 이름 |
   | 가상 머신 이름 | **az801l06a-dc2** |
   | 지역 | 이 연습의 앞부분에서 첫 번째 가상 머신을 배포한 동일한 Azure 지역을 선택합니다. |
   | 가용성 옵션 | **가용성 집합** |
   | 가용성 집합 | **adAvailabilitySet** |
   | 이미지 | **Windows Server 2022 Datacenter - Gen2** |
   | Azure Spot 인스턴스 | **아니요** |
   | 크기 | **표준 D2s v3** |
   | 사용자 이름 | **학생** |
   | 암호 | **Pa55w.rd1234** |
   | 공용 인바운드 포트 | **없음** |
   | 기존 Windows Server 라이선스를 사용하시겠습니까? | **아니요** |
   | OS 디스크 유형 | **표준 SSD** |
   | 데이터 디스크 이름 | **az801l06a-dc2_DataDisk_0** |
   | 데이터 디스크 원본 유형 | **없음(빈 디스크)** |
   | 데이터 디스크 크기 | **32GiB** **프리미엄 SSD** |
   | 가상 네트워크 | **az801l06a-vnet** |
   | 서브넷 | **adSubnet(10.6.0.0/24)** |
   | 공용 IP | **없음** |
   | NIC 네트워크 보안 그룹 추가 | **없음** |
   | 가속화된 네트워킹 | 사용 |
   | 기존 부하 분산 솔루션 뒤에 이 가상 머신을 배치하시겠습니까? | 사용 안 함 |
   | 부트 진단 | **관리형 스토리지 계정으로 활성화(권장)** |
   | 패치 오케스트레이션 옵션 | **수동 업데이트** |  

   > **참고**: 배포가 완료될 때까지 기다립니다. 배포는 3분 정도 걸릴 수 있습니다.

#### <a name="task-4-manually-promote-a-domain-controller-in-an-azure-vm"></a>작업 4: Azure VM에서 수동으로 도메인 컨트롤러 승격

1. **SEA-SVR2** 의 Azure Portal을 표시하는 Microsoft Edge 창에서 고정 할당을 사용하도록 **az801l06a-dc2** 가상 머신의 네트워크 인터페이스에 할당된 개인 IP 주소를 구성합니다. 

   > **참고**: 정적 할당은 도메인 컨트롤러를 배포할 때 일반적으로 사용되지만 DNS 서버 역할을 호스트하는 서버에는 필수적으로 사용해야 합니다.

   > **참고**: Azure VM의 네트워크 인터페이스에 고정 IP 주소를 할당하면 운영 체제가 다시 시작됩니다.

1. **SEA-SVR2** 의 Azure Portal을 표시하는 Microsoft Edge 창에서 **az801l06a-dc2** 페이지로 이동합니다.
1. **az801l06a-dc2** 페이지에서 Bastion 서비스를 통해 **az801l06a-dc2** 에 RDP 세션을 설정하고 다음 자격 증명을 사용하여 인증합니다.

   | 설정 | 값 | 
   | --- | --- |
   | 사용자 이름 |**학생** |
   | 암호 |**Pa55w.rd1234** |

1. **az801l06a-dc2** 에 대한 원격 데스크톱 세션 내에서 Windows PowerShell 세션을 시작합니다.
1. AD DS 및 DNS 서버 역할을 설치하려면 Windows PowerShell 프롬프트에서 다음 명령을 실행합니다.
    
   ```powershell
   Install-WindowsFeature -Name AD-Domain-Services,DNS -IncludeManagementFeatures
   ```

   > **참고**: 설치가 완료될 때까지 기다립니다. 3분 정도 걸릴 수 있습니다.

1. 데이터 디스크를 구성하려면 Windows PowerShell 프롬프트에서 다음 명령을 실행합니다.

   ```powershell
   Get-Disk | Where PartitionStyle -eq 'RAW' |  Initialize-Disk -PartitionStyle MBR
   New-Partition -DiskNumber 2 -UseMaximumSize -AssignDriveLetter
   Format-Volume -DriveLetter F -FileSystem NTFS
   ```

1. **az801l06a-dc2** 에 대한 원격 데스크톱 세션 내에서 **서버 관리자** 창으로 전환합니다.
1. **서버 관리자** 에서 **Active Directory Domain Services 구성 마법사** 를 시작하여 도메인 컨트롤러 승격을 수행합니다.
1. **Active Directory Domain Services 구성 마법사** 에서 **기존 도메인에 도메인 컨트롤러 추가** 옵션을 선택하고 **contoso.com** 을 대상 도메인으로 지정합니다.
1. **CONTOSO\\Student** 사용자 이름과 **Pa55w.rd1234** 암호를 자격 증명으로 사용하여 승격을 수행합니다.
1. 새 도메인 컨트롤러를 쓰기 가능으로 지정하는 옵션을 지정하고 **DNS(Domain Name System) 서버** 및 **GC(글로벌 카탈로그)** 구성 요소를 포함합니다.
1. **DSRM(Directory Services Restore Mode) 암호** 를 **Pa55w.rd1234** 로 설정합니다.
1. AD DS 데이터베이스, 로그 파일 및 SYSVOL을 호스트하는 폴더를 호스트하는 드라이브를 드라이브 **C** 에서 드라이브 **F** 로 변경합니다.
1. **필수 조건 확인** 페이지에서 고정 IP 주소가 없는 네트워크 어댑터에 대한 경고를 확인하고 승격을 시작합니다. 

   > **참고**: 고정 IP 주소가 운영 체제가 아닌 플랫폼 수준에서 할당되기 때문에 발생할 수 있습니다.

   > **참고**: 승격 프로세스를 완료하기 위해 운영 체제가 자동으로 다시 시작됩니다.

1. **SEA-SVR2** 에서 Bastion 서비스를 통해 **az801l06a-dc2** 에 다시 연결합니다.
1. **az801l06a-dc2** 에 대한 원격 데스크톱 세션 내에서 **서버 관리자** 를 사용하여 로컬에 설치된 역할에 **AD DS** 및 **DNS** 가 포함되어 있는지 확인합니다.

#### <a name="task-5-remove-azure-resources-deployed-in-the-exercise"></a>작업 5: 연습에 배포된 Azure 리소스 제거

#### <a name="task-1-start-a-powershell-session-in-cloud-shell"></a>작업 1: Cloud Shell에서 PowerShell 세션 시작

1. **SEA-SVR2** 의 Azure Portal을 표시하는 Microsoft Edge 창에서 Cloud Shell 아이콘을 선택하여 Cloud Shell 창을 엽니다.
1. Cloud Shell 창에서 다음 명령을 실행하여 이 랩 전체에서 만든 리소스 그룹을 모두 나열합니다.

   ```powershell
   Get-AzResourceGroup -Name 'AZ801-L060*'
   ```

   > **참고**: 출력에 이 연습에서 만든 리소스 그룹만 포함되어 있는지 확인하세요. 이 그룹은 이 작업에서 삭제됩니다.

1. 다음 명령을 실행하여 이 연습에서 만든 리소스 그룹을 모두 삭제합니다.

   ```powershell
   Get-AzResourceGroup -Name 'AZ801-L060*' | Remove-AzResourceGroup -Force -AsJob
   ```

   > **참고:** 이 명령은 *-AsJob* 매개 변수에 의해 결정되어 비동기로 실행되므로, 동일한 PowerShell 세션 내에서 다른 PowerShell 명령을 즉시 실행할 수 있지만 리소스 그룹이 실제로 제거되기까지는 몇 분 정도 걸립니다.

## <a name="exercise-2-migrating-file-servers-by-using-storage-migration-service"></a>연습 2: Storage Migration Service를 사용하여 파일 서버 마이그레이션

이 연습의 주요 작업은 다음과 같습니다.

1. Windows Admin Center를 설치합니다.
1. 파일 서비스를 설정합니다.
1. Storage Migration Service를 사용하여 마이그레이션을 수행합니다.
1. 마이그레이션 결과의 유효성을 검사합니다.

#### <a name="task-1-install-windows-admin-center"></a>작업 1: Windows Admin Center 설치

1. **SEA-ADM1** 에서 관리자 권한으로 **Windows PowerShell** 을 시작합니다.

   >**참고**: **SEA-ADM1** 에 Windows Admin Center를 아직 설치하지 않았다면 다음 두 단계를 수행합니다.

1. **Windows PowerShell** 콘솔에서 다음 명령을 실행하여 Windows Admin Center 최신 버전을 다운로드합니다.
    
   ```powershell
   Start-BitsTransfer -Source https://aka.ms/WACDownload -Destination "$env:USERPROFILE\Downloads\WindowsAdminCenter.msi"
   ```
1. 다음 명령을 실행하여 Windows Admin Center를 설치합니다.
    
   ```powershell
   Start-Process msiexec.exe -Wait -ArgumentList "/i $env:USERPROFILE\Downloads\WindowsAdminCenter.msi /qn /L*v log.txt REGISTRY_REDIRECT_PORT_80=1 SME_PORT=443 SSL_CERTIFICATE_OPTION=generate"
   ```

   > **참고**: 설치가 완료될 때까지 기다리세요. 이 작업은 2분 정도 걸립니다.

#### <a name="task-2-set-up-file-services"></a>작업 2: 파일 서비스 설정

1. **SEA-SVR2** 에서 Windows PowerShell ISE의 **C:\\Labfiles\\Lab06\\L06_SetupFS.ps1** 파일을 엽니다.
1. Windows PowerShell ISE 스크립트 창에서 스크립트를 검토한 다음, 이를 실행합니다. 

   >**참고:** 스크립트가 완료될 때까지 기다립니다. 이 작업은 1분 정도 걸립니다.

   >**참고:** 스크립트는 **SEA-SVR1** 및 **SEA-SVR2** 에서 추가 데이터 디스크를 초기화하고, 각각에 NTFS 볼륨을 만들고, 각 볼륨에 **S:** 드라이브 문자를 할당하고, **SEA-SVR1** 의 **S:\Data** 폴더를 사용하여 **Data** 라는 공유를 만들고, 이를 총 크기가 약 1GB인 샘플 파일에 추가합니다. 

#### <a name="task-3-perform-migration-by-using-storage-migration-service"></a>작업 3: Storage Migration Service를 사용하여 마이그레이션 수행

1. **SEA-ADM1** 에서 Microsoft Edge 시작하고 **https://SEA-ADM1.contoso.com** 에 있는 Windows Admin Center의 로컬 인스턴스에 연결합니다. 
1. 메시지가 표시되면 다음 자격 증명으로 인증합니다.

   - 사용자 이름: **CONTOSO\\Administrator**
   - 암호: **Pa55w.rd**

1. **SEA-ADM1** 의 Windows Admin Center에서 설치된 확장을 검토하고 목록에 **Storage Migration Service** 확장이 포함되어 있는지 확인합니다.

   >**참고:** 사용 가능한 업데이트가 있는 경우 **Storage Migration Service** 확장 항목을 선택하고 **업데이트** 를 선택하세요.

1. **모든 연결** 창에서 **sea-svr2.contoso.com** 에 연결합니다.
1. **sea-svr2.contoso.com** 페이지의 **도구** 메뉴에서 **Storage Migration Service** 를 시작하고 **설치** 작업을 호출합니다.

   >**참고:** 그러면 Storage Migration Service 및 필수 구성 요소가 자동으로 설치됩니다.

1. **스토리지 마이그레이션을 위한 세 단계** 창을 닫습니다.
1. **Storage Migration Service** 창에서 **원본 디바이스** 가 **Windows 서버 및 클러스터** 로 설정된 **SVR1toSVR2** 라는 마이그레이션 작업을 만듭니다.
1. **Storage Migration Service > SVR1toSVR2** 창의 **인벤토리 서버** 탭에서 **필수 구성 요소 확인** 창을 검토합니다.
1. **인벤토리 서버** 탭의 **자격 증명 입력** 창에서 필요한 경우 **CONTOSO\\Administrator** 사용자 계정의 자격 증명을 입력하고 **장애 조치(failover) 클러스터에서 마이그레이션** 확인란의 선택을 취소합니다.
1. **인벤토리 서버** 탭의 **디바이스 추가 및 검사** 창에서 다음 자격 증명을 사용하여 **SEA-SVR1.contoso.com** 서버를 추가합니다.

   - 사용자 이름: **CONTOSO\\Administrator**
   - 암호: **Pa55w.rd**

   > **참고**: Single Sign-On을 수행하려면 Kerberos 제한 위임을 설정해야 합니다.

1. 디바이스 목록에서 새로 추가된 **SEA-SVR1.contoso.com** 항목을 선택하고 검색을 시작합니다.

   >**참고:** 검사가 성공적으로 완료될 때까지 기다립니다. 이 작업은 1분 정도 걸립니다.

   >**참고:** 검사가 완료된 후 **Storage Migration Service > SVR1toSVR2** 창의 **데이터 전송** 탭을 통해 액세스할 수 있는 마이그레이션 작업의 두 번째 단계로 진행합니다.

1. **CONTOSO\\Administrator** 사용자 계정이 데이터 전송에 사용되고 있는지 확인합니다.
1. 대상 디바이스를 **SEA-SVR2.contoso.com** 으로 설정합니다.

   >**참고:** 검사가 성공적으로 완료될 때까지 기다립니다. 이 작업은 1분 정도 걸립니다.

   >**참고:** 하이브리드 시나리오에서는 마이그레이션 작업의 대상 역할을 하는 Azure VM을 자동으로 만드는 옵션도 있습니다.

1. 검사가 완료되면 **대상 지정: sea-svr1.contoso.com** 창에서 **각 원본 볼륨을 대상 볼륨에 매핑** 섹션을 검토하고 **S:** 원본 볼륨이 **S:** 대상 볼륨에 매핑되었는지 확인합니다.
1. **대상 지정: sea-svr1.contoso.com** 창에서 **전송할 공유 선택 섹션** 을 검토하고 **데이터** 원본 공유가 전송에 포함되어 있는지 확인합니다.
1. **데이터 전송** 탭의 **전송 설정 조정** 창에서 다음 설정을 지정합니다(다른 설정은 기본값으로 유지).

   | 설정 | 값 | 
   | --- | --- |
   | 덮어쓸 폴더 백업(Azure 파일 동기화 지원 공유는 백업되지 않음) | 사용 |
   | 유효성 검사 방법 | **CRC 64** |
   | 최대 기간(분) | **60** |
   | 사용자 및 그룹 마이그레이션 | **동일한 이름의 계정을 다시 사용** |
   | 최대 재시도 횟수 | **3** |
   | 재시도 간 지연(초) | **60** |

   >**참고:** **필수 기능 설치** 창의 **데이터 전송** 탭에서 **SEA-SVR2.contoso.com** **의 SMS-Proxy** 설치가 완료될 때까지 기다립니다.

1. 검사가 완료되면 **데이터 전송** 탭의 **원본 및 대상 디바이스 유효성 검사** 창에서 유효성 검사를 시작하고 성공적으로 완료될 때까지 기다립니다.
1. **데이터 전송** 탭의 **전송 시작** 창에서 데이터 전송을 시작합니다.

   >**참고:** 전송이 성공적으로 완료될 때까지 기다립니다. 이 작업은 1분 이내에 완료됩니다.

   >**참고:** 그러면 **Storage Migration Service > SVR1toSVR2** 창의 **새 서버로 전환** 탭을 통해 액세스할 수 있는 마이그레이션 작업의 세 번째 단계로 전환됩니다.

1. **새 서버로 전환** 탭에서, **원본 디바이스의 자격 증명 입력** 및 **대상 디바이스의 자격 증명 입력** 섹션에서 **CONTOSO\\관리자** 사용자 계정의 저장된 자격 증명을 수락합니다.
1. **새 서버로 전환** 탭에서, **sea-svr1.contoso.com에서 sea-svr2.contoso.com으로 전환 구성** 창의 **원본 네트워크 어댑터** 섹션에서 다음 설정을 지정합니다.

   | 설정 | 값 | 
   | --- | --- |
   | DHCP 사용 | 사용 안 함 |
   | IP 주소 | **172.16.10.111** |
   | 서브넷 | **255.255.0.0** |
   | 게이트웨이 | **172.16.10.1** |

1. **새 서버로 전환** 탭에서, **sea-svr1.contoso.com에서 sea-svr2.contoso.com으로 전환 구성** 창의 **대상 네트워크 어댑터** 드롭다운 목록에서 **시애틀** 을 선택합니다.
1. **새 서버로 전환** 탭에서, **sea-svr1.contoso.com에서 sea-svr2.contoso.com으로 전환 구성** 창의 **전환 후 원본 디바이스 이름 바꾸기** 섹션에서 **새 이름 선택** 옵션을 선택한 다음, **새 원본 컴퓨터 이름** 대화 상자에 **SEA-SVR1-OLD** 를 입력합니다.
1. **새 서버로 전환** 탭에서, **컷오버 설정 조정** 창의 **컷오버 시간 초과(분)** 텍스트 상자에서 **30** 을 입력한 다음, **AD 자격 증명 입력** 섹션에서 **저장된 자격 증명 정보** 옵션을 활성화된 상태로 둡니다.
1. **새 서버로 전환** 탭의 **원본 및 대상 디바이스 유효성 검사** 창에서 유효성 검사를 시작합니다.
1. 유효성 검사가 성공적으로 완료되면 **새 서버로 전환** 탭에서 전환 단계를 시작합니다.

   >**참고:** 컷오버를 수행하면 **SEA-SVR1** 및 **SEA-SVR2** 가 두 번 연속으로 다시 시작됩니다.

#### <a name="task-4-validate-migration-outcome"></a>작업 4: 마이그레이션 결과 유효성 검사

1. **SEA-ADM1** 에서 관리자 권한으로 **Windows PowerShell** 을 시작합니다.
1. **SEA-SVR2** 의 네트워크 인터페이스에 할당된 IPv4 주소를 식별하려면 **Windows PowerShell** 콘솔에서 다음 명령을 실행합니다.
    
   ```powershell
   Get-NetIPAddress | Where-Object AddressFamily -eq 'IPv4' | Select-Object IPAddress
   ```

   >**참고:** 출력에 **172.16.10.11** 및 **172.16.10.12** 가 모두 포함되어 있는지 확인합니다.

1. **SEA-SVR2** 에 할당된 NetBIOS 이름을 식별하려면 **Windows PowerShell** 콘솔에서 다음 명령을 실행합니다.
    
   ```powershell
   nbtstat -n
   ```

   >**참고:** 출력에 **SEA-SVR1** 및 **SEA-SVR2** 가 모두 포함되어 있는지 확인합니다.

1. **SEA-SVR2** 의 로컬 공유를 식별하려면 **Windows PowerShell** 콘솔에서 다음 명령을 실행합니다.
    
   ```powershell
   Get-SMBShare
   ```

   >**참고:** 출력에 **S:\Data** 폴더에 호스트된 **데이터** 공유가 포함되어 있는지 확인합니다.

1. **SEA-SVR2** 에서 **Data** 공유의 콘텐츠를 식별하려면 **Windows PowerShell** 콘솔에서 다음 명령을 실행합니다.
    
   ```powershell
   Get-ChildItem -Path 'S:\Data'
   ```

#### <a name="review"></a>검토

이 랩에서 다음을 수행했습니다.

- Azure에 AD DS 도메인 컨트롤러를 배포했습니다.
- Storage Migration Service를 사용하여 파일 서버를 마이그레이션했습니다.
