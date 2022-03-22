---
lab:
  title: '랩: Azure 기반 복구 서비스 구현'
  module: 'Module 5: Planning and implementing migration and recovery services in hybrid scenarios'
ms.openlocfilehash: 5d2a3e40b82d90b132e588fede27229275862047
ms.sourcegitcommit: 9a51ea796ef3806ab9e7ec1ff75034b2f929ed2a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/28/2022
ms.locfileid: "137907130"
---
# <a name="lab-implementing-azure-based-recovery-services"></a>랩: Azure 기반 복구 서비스 구현

## <a name="scenario"></a>시나리오 
오래된 운영 모델, 제한된 자동화 사용, 복원 및 재해 복구를 대비한 테이프 백업 의존성에 대한 우려를 해결하기 위해 Microsoft Azure 기반 복구 서비스를 사용하기로 결정했습니다. 첫 번째 단계로 Azure Site Recovery 및 Azure Backup을 구현합니다.

## <a name="objectives"></a>목표
이 랩을 완료하면 다음을 수행할 수 있습니다. 

- Azure Site Recovery 자격 증명 모음을 만들고 구성할 수 있습니다.
- Azure Site Recovery 자격 증명 모음을 사용하여 Hyper-V VM 보호를 구현할 수 있습니다.
- Azure Backup을 구현할 수 있습니다.
- Azure 랩 환경의 프로비전을 해제할 수 있습니다.

## <a name="estimated-time-60-minutes"></a>예상 시간: 60분

## <a name="lab-setup"></a>랩 설정

가상 머신: **AZ-801T00A-SEA-DC1**, **AZ-801T00A-SEA-SVR1**, **AZ-801T00A-SEA-SVR2** 를 실행해야 합니다. 다른 VM을 실행할 수도 있지만 이 랩에서는 필요하지 않습니다.

> **참고**: **AZ-801T00A-SEA-DC1**, **AZ-801T00A-SEA-SVR1**, **AZ-801T00A-SEA-SVR2** 가상 머신은 **SEA-DC1**, **SEA-SVR1**, **SEA-SVR2** 의 설치를 호스트합니다.

1. **SEA-SVR2** 를 선택합니다.
1. 다음 자격 증명을 사용하여 로그인:

   - 사용자 이름: **Administrator**
   - 암호: **Pa55w.rd**
   - 도메인: **CONTOSO**

이 랩에서는 사용 가능한 VM 환경과 Azure 구독을 사용합니다. 랩을 시작하기 전에 구독에서 Owner 또는 Contributor 역할이 있는 Azure 구독 및 사용자 계정이 있는지 확인하세요.

## <a name="exercise-1-creating-and-configuring-an-azure-site-recovery-vault"></a>연습 1: Azure Site Recovery 자격 증명 모음 만들기 및 구성

### <a name="scenario"></a>시나리오

Azure를 재해 복구 사이트로 사용하여 Azure VM에서 실행되는 VM용 Azure Site Recovery를 구현하려면 먼저 Azure Site Recovery 자격 증명 모음을 만들고 구성해야 합니다.

이 연습의 주요 작업은 다음과 같습니다.

1. Azure Site Recovery 자격 증명 모음을 만듭니다.
1. Azure Site Recovery 자격 증명 모음을 구성합니다.

#### <a name="task-1-create-an-azure-site-recovery-vault"></a>작업 1: Azure 사이트 복구 자격 증명 모음 만들기

1. **SEA-SVR2** 에서 Microsoft Edge를 시작하고 **[Azure Portal](https://portal.azure.com)** 로 이동한 후 이 랩에서 사용할 구독에서 Owner 역할이 있는 사용자 계정의 자격 증명을 사용하여 로그인합니다.
1. Azure Portal에서 다음 설정을 사용하여 Recovery Services 자격 증명 모음을 만듭니다(다른 설정은 기본값으로 유지).

   |설정|값|
   |---|---|
   |구독|이 랩에서 사용할 Azure 구독의 이름|
   |리소스 그룹|새 리소스 그룹 **AZ801-L0501-RG** 의 이름|
   |자격 증명 모음 이름|**az801l05a-rsvault**|
   |위치|Azure Recovery Services 자격 증명 모음을 만들 수 있고 랩 환경의 위치에 가까운 Azure 지역의 이름|

   > **참고:** Recovery Services 자격 증명 모음이 프로비전될 때까지 기다립니다. 이 작업은 2분 정도 걸립니다.

   > **참고:** 기본적으로 자격 증명 모음의 스토리지 복제 유형은 지역 중복(GRS)으로 설정되고 일시 삭제가 사용됩니다. 프로비전 해제를 간소화하기 위해 랩에서 이러한 설정을 변경하지만 프로덕션 환경에서 활성화해야 합니다.

#### <a name="task-2-configure-the-azure-site-recovery-vault"></a>작업 2: Azure Site Recovery 자격 증명 모음 구성

1. **SEA-SVR2** 의 Azure Portal을 표시하는 Microsoft Edge 창에서 새로 만든 Azure Recovery Services 자격 증명 모음으로 이동합니다. 
1. Azure Portal에서 자격 증명 모음의 **스토리지 복제 유형** 을 **로컬 중복** 으로 설정합니다.

   > **참고:** 보호를 구현한 후에는 스토리지 복제 유형을 변경할 수 없습니다.

1. Azure Portal에서 자격 증명 모음의 **일시 삭제** 및 **보안 기능** 설정을 비활성화합니다.

## <a name="exercise-2-implementing-hyper-v-vm-protection-by-using-azure-site-recovery-vault"></a>연습 2: Azure Site Recovery 자격 증명 모음을 사용하여 Hyper-V VM 보호 구현

### <a name="scenario"></a>시나리오

테스트 Hyper-V VM 및 Recovery Services 자격 증명 모음이 만들어지면 이제 Azure Site Recovery를 사용하여 Hyper-V VM 보호 구현을 진행할 수 있습니다. 테스트 장애 조치(failover)를 수행하고 계획되거나 계획되지 않은 장애 조치(failover)의 설정을 검토합니다. 

이 연습의 주요 작업은 다음과 같습니다.

1. Azure 복구 사이트를 구현합니다.
1. Hyper-V 가상 머신의 보호를 준비합니다.
1. Hyper-V 가상 머신의 복제를 사용합니다.
1. Azure VM 복제 설정을 검토합니다.
1. Hyper-V 가상 머신의 장애 조치(failover)를 수행합니다.

#### <a name="task-1-implement-an-azure-recovery-site"></a>작업 1: Azure 복구 사이트 구현

1. **SEA-SVR2** 의 Azure Portal에서 다음 설정을 사용하여 가상 네트워크를 만듭니다(다른 설정은 기본값으로 유지).

   |설정|값|
   |---|---|
   |구독|이 랩에서 사용 중인 Azure 구독의 이름|
   |리소스 그룹|새 리소스 그룹 **AZ801-L0502-RG** 의 이름|
   |이름|**az801l05-dr-vnet**|
   |지역|이 랩의 앞부분에서 Recovery Services 자격 증명 모음을 배포한 Azure 지역의 이름|

1. 새 가상 네트워크 내에서 다음 설정을 사용하여 서브넷을 만듭니다(다른 설정은 기본값으로 유지).

   |설정|값|
   |---|---|
   |서브넷 이름|**subnet0**|
   |서브넷 주소 범위|**10.5.0.0/24**|

1. **SEA-SVR2** 의 Azure Portal에서 다음 설정을 사용하여 다른 가상 네트워크를 만듭니다(다른 설정은 기본값으로 유지).

   |설정|값|
   |---|---|
   |구독|이 랩에서 사용 중인 Azure 구독의 이름|
   |리소스 그룹|**AZ801-L0502-RG**|
   |이름|**az801l05-test-vnet**|
   |지역|이 랩의 앞부분에서 Recovery Services 자격 증명 모음을 배포한 Azure 지역의 이름|

   > **참고:** 겹치는 IP 주소 공간에 대한 경고는 무시합니다. 이는 의도적인 것이므로 테스트 환경의 IP 주소 공간은 재해 복구 환경의 IP 주소 공간과 일치합니다.

1. 새 가상 네트워크 내에서 다음 설정을 사용하여 서브넷을 만듭니다(다른 설정은 기본값으로 유지).

   |설정|값|
   |---|---|
   |서브넷 이름|**subnet0**|
   |서브넷 주소 범위|**10.5.0.0/24**|

1. **SEA-SVR2** 의 Azure Portal에서 다음 설정을 사용하여 스토리지 계정을 만듭니다(다른 설정은 기본값으로 유지).

   |설정|값|
   |---|---|
   |구독|이 랩에서 사용 중인 Azure 구독의 이름|
   |리소스 그룹|**AZ801-L0502-RG**|
   |스토리지 계정 이름|문자 및 숫자로 구성되고 문자로 시작하는 3~24자 사이의 전역적으로 고유한 이름|
   |지역|이 랩의 앞부분에서 Recovery Services 자격 증명 모음을 배포한 Azure 지역의 이름|
   |성능|표준|
   |중복|LRS(로컬 중복 스토리지)|
   |Blob에 일시 삭제를 사용하도록 설정|사용 안 함|
   |컨테이너에 대해 일시 삭제 사용|사용 안 함|

   > **참고:** Azure Site Recovery에 스토리지 계정을 사용하는 경우 Blob 및 컨테이너에 대한 일시 삭제 기능을 비활성화해야 합니다.

#### <a name="task-2-prepare-protection-of-a-hyper-v-virtual-machine"></a>작업 2: Hyper-V 가상 머신 보호 준비

1. **SEA-SVR2** 의 Azure Portal에서 **az801l05a-rsvault** Recovery Services 자격 증명 모음 블레이드로 이동합니다. 
1. **az801l05a-rsvault** 블레이드의 세로 메뉴에서 **Site Recovery** 구성을 시작합니다.
1. **az801l05a-rsvault \| Site Recovery** 블레이드의 **Hyper-V 컴퓨터에서 Azure로** 섹션에서 **1. 인프라를 준비** 를 선택하고 다음 설정을 지정합니다.

   |설정|값|
   |---|---|
   |배포 계획이 완료되었습니까?|**예, 완료되었습니다.**|
   |System Center VMM을 사용하여 Hyper-V 호스트를 관리하고 있습니까?|**아니요**|
   |원본 설정: Hyper-V 사이트|**az801l05-site**|

1. **인프라 준비** 블레이드의 **원본 설정** 탭에서 **Hyper-V 서버 추가** 링크를 선택합니다. 
1. **서버 추가** 블레이드에서 온-프레미스 Hyper-V 호스트를 추가하는 절차의 3단계에서 **다운로드** 링크를 선택하여 Microsoft Azure Site Recovery 공급자를 다운로드합니다.
1. **Microsoft 업데이트** 옵션을 비활성화한 상태로 **AzureSiteRecoveryProvider.exe** 를 설치합니다. 
1. Azure Portal에서 자격 증명 모음 등록 키를 **다운로드** 폴더에 다운로드합니다.
1. **공급자 설치** 마법사를 완료하고 **Microsoft Azure Site Recovery 등록 마법사** 를 시작합니다.
1. 메시지가 표시되면 **Microsoft Azure Site Recovery 등록 마법사** 에서 자격 증명 모음 자격 증명 파일 위치를 제공합니다.
1. 기본 설정을 사용하여 **Microsoft Azure Site Recovery 등록 마법사** 를 완료합니다. 
1. Azure Portal을 표시하는 브라우저 페이지를 새로 고치고 **1. 인프라 준비** 절차를 초기 단계를 반복합니다.
1. **인프라 준비** 블레이드의 **원본 설정** 탭에 도달한 후 **Hyper-V 사이트** 및 **Hyper-V 서버** 설정이 올바르게 설정되었는지 확인하고 다음 단계로 계속 진행합니다. 
1. **인프라 준비** 페이지의 **대상 설정** 탭에서 기본 설정을 적용합니다.
1. **인프라 준비** 블레이드의 **복제 정책** 탭에서 다음 설정을 사용하여 새 정책을 만들고 Hyper-V 사이트와 연결합니다.

   |설정|값|
   |---|---|
   |이름|**az801l05-replication-policy**|
   |복사 빈도|**30초**|

1. **인프라 준비** 절차를 완료하고 연결 프로세스가 완료될 때까지 기다립니다.

#### <a name="task-3-enable-replication-of-a-hyper-v-virtual-machine"></a>작업 3: Hyper-V 가상 머신 복제 사용

1. **SEA-SVR2** 의 Azure Portal에서 **az801l05a-rsvault \| Site Recovery** 페이지의 **Hyper-V 컴퓨터에서 Azure로** 섹션에서 **2. 복제 사용** 을 선택합니다. 
1. **복제 사용** 페이지의 **원본 환경** 탭에 있는 **원본 위치** 드롭다운 목록에서 **az801l05-site** 를 선택합니다.
1. **복제 사용** 블레이드의 **대상 환경** 탭에서 다음 설정을 지정합니다(다른 설정은 기본값으로 유지).

   |설정|값|
   |---|---|
   |구독|이 랩에서 사용 중인 Azure 구독의 이름|
   |사후 장애 조치(failover) 리소스 그룹|**AZ801-L0502-RG**|
   |사후 장애 조치(failover) 배포 모델|**Resource Manager**|
   |스토리지 계정|이 연습의 첫 번째 작업에서 만든 스토리지 계정의 이름|
   |Azure 네트워크|선택한 머신에 대해 지금 구성|
   |가상 네트워크|**az801l05-dr-vnet**|
   |서브넷|**subnet0(10.5.0.0/24)**|

1. **복제 사용** 블레이드의 **가상 머신 선택** 탭에서 **SEA-CORE1** 항목을 선택합니다.
1. **복제 사용** 블레이드의 **복제 설정** 탭에서 **기본값** 및 **OS 유형** 을 **Windows** 로 설정합니다.
1. 기본 설정을 사용하여 **복제 사용** 절차를 완료합니다.

#### <a name="task-4-review-azure-vm-replication-settings"></a>작업 4: Azure VM 복제 설정 검토

1. Azure Portal에서 **az801l05a-rsvault \| Site Recovery** 블레이드로 돌아가서 **az801l05a-rsvault \| 복제된 항목** 블레이드로 이동합니다.
1. **az801l05a-rsvault \| 복제된 항목** 블레이드에서 **SEA-CORE1** 가상 머신을 나타내는 항목이 있는지 확인하고 **복제 상태** 가 **정상** 으로 나열되고 **상태** 가 **보호 사용** 으로 나열되거나 동기화 진행률의 현재 백분율을 표시하는지 확인합니다.

   > **참고:** **SEA-CORE1** 항목이 **az801l05a-rsvault \| 복제된 항목** 블레이드에 나타날 때까지 몇 분 정도 기다려야 할 수 있습니다.

1. **az801l05a-rsvault \| 복제된 항목** 블레이드에서 **SEA-CORE1** 복제된 항목 블레이드로 이동합니다.
1. **SEA-CORE1** 복제된 항목 블레이드에서 **상태**, **장애 조치(failover) 준비 상태**, **최신 복구 지점** 및 **인프라 보기** 섹션을 검토합니다. **계획된 장애 조치(failover)** , **장애 조치(failover)** 및 **테스트 장애 조치(failover)** 도구 모음 아이콘을 확인합니다.

   > **참고:** 상태가 **보호됨** 으로 변경될 때까지 기다립니다. 이 작업을 수행하는 데 필요한 시간은 랩 환경과 Recovery Services 자격 증명 모음을 호스트하는 Azure 지역 간에 사용 가능한 연결 대역폭에 따라 달라집니다. 상태를 업데이트하려면 브라우저 페이지를 새로 고쳐야 합니다.

1. **SEA-CORE1** 복제된 항목 블레이드에서 **최신 복구 지점** 을 선택하고 **최신 크래시 일치** 및 **최신 앱 일치** 복구 지점을 검토합니다. 

#### <a name="task-5-perform-a-failover-of-the-hyper-v-virtual-machine"></a>작업 5: Hyper-V 가상 머신의 장애 조치(failover) 수행

1. **SEA-SVR2** 의 Azure Portal을 표시하는 브라우저 창의 **SEA-CORE1** 복제된 항목 블레이드에서 다음 설정을 사용하여 **테스트 장애 조치(failover)** 를 시작하고(다른 설정은 기본값으로 유지) **확인** 을 선택합니다.

   |설정|값|
   |---|---|
   |복구 지점 선택|기본 옵션|
   |Azure 가상 네트워크|**az801l05-test-vnet**|

1. Azure Portal에서 **az801l05a-rsvault** 블레이드로 돌아가서 여기에서 **Site Recovery 작업** 목록으로 이동합니다. **테스트 장애 조치(failover)** 작업의 상태가 **성공** 으로 나열될 때까지 기다립니다.

   > **참고:** 테스트 장애 조치(failover)가 완료되는 데 필요한 시간은 랩 환경과 Recovery Services 자격 증명 모음을 호스트하는 Azure 지역 간에 사용 가능한 연결 대역폭에 따라 달라집니다. 상태를 업데이트하려면 브라우저 페이지를 새로 고쳐야 합니다. 

   > **참고:** 테스트 장애 조치(failover)가 완료되기를 기다리는 동안 연습 3을 진행하고 완료되면 이 연습의 나머지 부분을 단계별로 진행합니다.

1. Azure Portal에서 **가상 머신** 블레이드로 이동하여 새로 프로비전된 가상 머신을 나타내는 항목을 확인합니다.

   > **참고:** 처음에 가상 머신의 이름은 **asr-** 접두사와 임의로 생성된 접미사로 구성되지만 결국 **SEA-CORE1-test** 로 이름이 바뀝니다.

1. Azure Portal에서 **SEA-CORE1** 복제된 항목 블레이드로 돌아가서 **테스트 장애 조치(failover) 정리** 를 시작합니다.
1. 테스트 장애 조치(failover) 정리 작업이 완료되면 **SEA-CORE1** 복제된 항목 블레이드를 표시하는 브라우저 페이지를 새로 고치고 계획되거나 계획되지 않은 장애 조치(failover)를 수행할 수 있는 옵션이 있음을 확인합니다.
1. **SEA-CORE1** 복제된 항목 블레이드에서 **계획된 장애 조치(failover)** 블레이드로 이동합니다.
1. **계획된 장애 조치(failover)** 블레이드에서 장애 조치 방향 설정이 이미 정해져 있으며 수정할 수 없음을 확인합니다. 
1. 장애 조치(failover)를 시작하지 않고 **계획된 장애 조치(failover)** 블레이드를 닫습니다. 
1. **SEA-CORE1** 복제된 항목 블레이드에서 **장애 조치(failover)** 블레이드로 이동합니다.
1. **장애 조치(failover)** 블레이드에서 복구 지점을 선택할 수 있는 옵션이 있음을 확인합니다. 
1. 장애 조치(failover)를 시작하지 않고 **장애 조치(failover)** 블레이드를 닫습니다.

## <a name="exercise-3-implementing-azure-backup"></a>연습 3: Azure Backup 구현

### <a name="scenario"></a>시나리오

중첩된 VM의 복제가 완료될 때까지 기다리는 동안 Azure VM 에이전트와 세 번째 Azure VM의 Azure VM 수준 백업을 사용하여 두 번째 Azure VM의 Azure Backup을 구현합니다.

이 연습의 주요 작업은 다음과 같습니다.

1. Azure Recovery Services 에이전트를 설정합니다.
1. Azure Backup을 예약합니다.
1. 주문형 백업을 수행합니다.
1. Azure Recovery Services 에이전트를 사용하여 파일 복구를 수행합니다.

#### <a name="task-1-set-up-the-azure-recovery-services-agent"></a>작업 1: Azure Recovery Services 에이전트 설정

> **참고:** 일반적으로 동일한 자격 증명 모음을 사용하여 Azure Site Recovery 및 Azure Backup 기능을 구현할 수 있습니다. 재해 복구 및 백업용 자격 증명 모음을 호스트할 Azure 지역을 선택할 때는 지역 재해의 영향 범위와 네트워크 대기 시간 고려 사항을 포함하여 복구 목표를 고려해야 합니다. 이 랩에서는 사이트 복구 및 백업에 동일한 자격 증명 모음을 사용하여 중복 단계 수를 최소화합니다. 

> **참고:** Azure Backup을 구현하려면 이미 **Microsoft Azure Site Recovery 공급자** 역할을 하는 **SEA-SVR2** 에 Azure Recovery Services 에이전트를 설치합니다. 종속성 문제를 제거하기 위해 먼저 Azure Recovery Services 에이전트의 기존 설치를 제거합니다.

1. **SEA-SVR2** 에서 **설정** 앱을 사용하여 **Microsoft Azure Recovery Services 에이전트** 를 제거합니다.
   > **참고:** 설치에 실패했다는 오류 메시지가 표시되면 VM을 다시 시작하고 다시 시도합니다. 
1. **SEA-SVR2** 의 Azure Portal을 표시하는 Microsoft Edge 창의 **az801l05a-rsvault Recovery Services** 자격 증명 모음 블레이드에서 다음 설정을 사용하여 **백업** 구성을 시작합니다.

   |설정| 값|
   |---|---|
   |워크로드는 어디에서 실행되고 있습니까?|**온-프레미스**|
   |어떤 항목을 백업하시겠습니까?|**파일 및 폴더**|

1. **az801l05a-rsvault \|백업** 블레이드에서 **인프라 준비** 절차를 시작합니다.
1. **인프라 준비** 블레이드에서 Azure Recovery Services 에이전트를 **SEA-SVR2** 로 다운로드하고, **Microsoft Azure Recovery Services 에이전트 설치 마법사** 를 시작하고, Microsoft 업데이트 옵션을 비활성화하고, 기본 설정을 사용하여 설치를 완료합니다. 
1. 설치가 완료되면 **서버 등록 마법사** 를 시작합니다.
1. Azure Portal을 표시하는 Microsoft Edge 창으로 전환하고 **인프라 준비** 블레이드에서 자격 증명 모음 자격 증명 파일을 로컬 다운로드 폴더로 다운로드합니다.
1. **서버 등록 마법사** 창으로 다시 전환하고 자격 증명 모음 자격 증명을 제공하라는 메시지가 표시되면 새로 다운로드한 파일을 가리킵니다. 
1. **서버 등록 마법사** 의 **암호화 설정** 페이지에서 암호를 생성하고 로컬 **문서** 폴더에 저장합니다. 
1. **Microsoft Azure Backup** 경고를 검토하고 등록을 완료합니다. 그러면 **Microsoft Azure Backup** 콘솔이 자동으로 열립니다.

   > **참고:** 프로덕션 환경에서는 백업 중인 서버가 아닌 안전한 위치에 암호 파일을 저장해야 합니다.

#### <a name="task-2-schedule-azure-backup"></a>작업 2: Azure Backup 예약

1. **SEA-SVR2** 의 **Microsoft Azure Backup** 콘솔에서 다음 설정을 사용하여 백업을 예약합니다(다른 설정은 기본값으로 유지).

   |설정| 값|
   |---|---|
   |백업할 항목|C:\\Windows\\System32\\drivers\\etc\\hosts|
   |백업 일정|매일 오전 4:30|
   |보존 정책|default|
   |초기 백업 유형|default|

#### <a name="task-3-perform-an-on-demand-backup"></a>작업 3: 주문형 백업 수행

> **참고:** 예약된 백업을 만든 후 요청 시 백업을 실행하는 옵션을 사용할 수 있습니다.

1. **Microsoft Azure Backup** 콘솔에서 기본 설정을 사용하여 주문형 백업을 시작합니다.
1. Azure Portal을 표시하는 Microsoft Edge 창으로 전환하고 **az801l05a-rsvault** Recovery Services 자격 증명 모음 블레이드로 돌아가서 **백업 항목** 을 표시합니다. 
1. **az801l05a-rsvault \| 백업 항목** 블레이드에서 **백업 항목(Azure Backup 에이전트)** 블레이드로 이동하여 **sea-svr2.contoso.com** 의 드라이브 **C** 를 참조하는 항목이 있는지 확인합니다.

#### <a name="task-4-perform-file-recovery-by-using-azure-recovery-services-agent"></a>작업 4: Azure Recovery Services 에이전트를 사용하여 파일 복구 수행

1. **SEA-SVR2** 에서 파일 탐색기를 열고 **C:\\Windows\\System32\\drivers\\etc\\** 폴더로 이동하여 **hosts** 파일을 삭제합니다.
1. Microsoft Azure Backup 창으로 전환하고 다음 설정을 사용하여 **데이터 복구 마법사** 를 시작합니다(다른 설정은 기본값으로 유지).

   |설정| 값|
   |---|---|
   |대상 복원|**이 서버(sea-svr2.contoso.com)**|
   |항목 복원|**개별 파일 및 폴더**|
   |볼륨 선택|**C:\\**|

   > **참고:** 탑재 작업이 완료될 때까지 기다립니다. 2분 정도 걸릴 수 있습니다.

1. **파일 찾아보기 및 복구** 페이지에서 복구 볼륨의 드라이브 문자를 확인하고, **찾아보기** 를 선택하고, **Robocopy** 사용에 대한 팁을 검토합니다. 
1. **SEA-SVR2** 에서 **명령 프롬프트** 를 시작합니다.
1. **관리자: 명령 프롬프트** 창에서 다음을 실행하여 **호스트** 파일을 복사하여 원래 위치로 복원합니다(`<recovery_volume>`을 앞서 식별한 복구 볼륨의 드라이브 문자로 대체).

   ```cmd
   robocopy <recovery_volume>:\Windows\System32\drivers\etc C:\Windows\system32\drivers\etc hosts /r:1 /w:1
   ```

1. **관리자: 명령 프롬프트** 창에서 다음을 실행하여 파일이 복원되었는지 확인합니다.

   ```cmd
   dir C:\Windows\system32\drivers\etc\hosts
   ```

1. **데이터 복구 마법사** 로 다시 전환하고 탑재된 백업 파일을 분리합니다. 

## <a name="exercise-4-deprovisioning-the-azure-lab-environment"></a>연습 4: Azure 랩 환경 프로비전 해제

### <a name="scenario"></a>시나리오

Azure 관련 요금을 최소화하기 위해 이 랩 전체에서 프로비전한 Azure 리소스의 프로비전을 해제하려 합니다.

이 연습의 주요 작업은 다음과 같습니다.

1. 보호된 항목을 제거합니다.
1. 랩 리소스 그룹을 삭제합니다.

#### <a name="task-1-remove-the-protected-items"></a>작업 1: 보호된 항목 제거

1. **SEA-SVR2** 에서 Azure Portal의 **백업 항목(Azure Backup 에이전트)** 블레이드를 표시하는 Microsoft Edge 창으로 전환하고 **sea-svr2.contoso.com** 의 **C** 드라이브를 참조하는 항목을 선택합니다.
1. **C:\\ sea-svr2.contoso.com** 블레이드에서 **sea-svr2.contoso.com** 블레이드로 이동합니다.
1. **sea-svr2.contoso.com** 블레이드에서 다음 정보를 지정하고 백업을 삭제합니다. 

   |설정| 값|
   |---|---|
   |서버 이름 입력|**sea-svr2.contoso.com**|
   |이유|**서비스 해제됨**|
   |주석|**서비스 해제됨**|

1. **SEA-SVR2** 의 Azure Portal을 표시하는 Microsoft Edge에서 **az801l05a-rsvault | 복제된 항목** 블레이드로 이동하고 **SEA-CORE1** 항목을 선택합니다.
1. **SEA-CORE1** 복제 항목 블레이드에서 복제를 사용하지 않고 피드백을 제공하지 않고 복제된 항목을 제거합니다. 

#### <a name="task-2-delete-the-lab-resource-groups"></a>작업 2: 랩 리소스 그룹 삭제

1. **SEA-SVR2** 에서 Azure Portal을 표시하는 Microsoft Edge 창의 Azure Cloud Shell 창에서 PowerShell 세션을 엽니다.

    > **참고:** Cloud Shell을 처음 사용하고 **탑재된 스토리지가 없음** 메시지가 표시되면 이 랩에서 사용 중인 구독을 선택한 후 **스토리지 만들기** 를 선택합니다.

1. Cloud Shell 창에서 다음 명령을 실행하여 이 랩에서 만든 리소스 그룹을 모두 나열합니다.

   ```powershell
   Get-AzResourceGroup -Name 'AZ801-L070*'
   ```

   > **참고**: 출력에 이 랩에서 만든 리소스 그룹만 포함되어 있는지 확인하세요. 이 그룹은 이 작업에서 삭제됩니다.

1. Cloud Shell 창에서 다음 명령을 실행하여 이 랩에서 만든 리소스 그룹을 모두 삭제합니다.

   ```powershell
   Get-AzResourceGroup -Name 'AZ801-L050*' | Remove-AzResourceGroup -Force -AsJob
   ```

   > **참고:** 이 명령은 *-AsJob* 매개 변수에 의해 결정되어 비동기로 실행됩니다. 따라서 동일한 PowerShell 세션에서 즉시 다른 PowerShell 명령을 실행할 수 있지만, 리소스 그룹이 실제로 제거되려면 몇 분 정도 걸립니다.
