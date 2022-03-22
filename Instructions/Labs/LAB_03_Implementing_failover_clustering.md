---
lab:
  title: '랩: 장애 조치(failover) 클러스터링 구현'
  module: 'Module 3: High availability in Windows Server'
ms.openlocfilehash: dce41bd9411fc821c35e903255156bc6f8b558b3
ms.sourcegitcommit: 9a51ea796ef3806ab9e7ec1ff75034b2f929ed2a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/28/2022
ms.locfileid: "137907129"
---
# <a name="lab-implementing-failover-clustering"></a>랩: 장애 조치(failover) 클러스터링 구현

## <a name="scenario"></a>시나리오

Contoso, Ltd.의 비즈니스가 성장함에 따라 네트워크의 많은 애플리케이션과 서비스를 항상 사용할 수 있다는 것이 점점 더 중요해지고 있습니다. Contoso에는 전 세계의 다른 표준 시간대에서 작업하는 내부 및 외부 사용자가 사용할 수 있어야 하는 많은 서비스 및 애플리케이션이 있습니다. 이러한 애플리케이션의 대부분은 NLB(네트워크 부하 분산)를 사용하여 고가용성으로 만들 수 없습니다. 따라서 다른 기술을 사용하여 이러한 애플리케이션을 고가용성으로 만들어야 합니다.

Contoso의 선임 네트워크 관리자 중 한 명인 사용자는 Windows Server를 실행하는 서버에서 장애 조치(failover) 클러스터링을 구현하여 네트워크 서비스 및 애플리케이션에 고가용성을 제공합니다. 또한 장애 조치 클러스터 구성을 계획하고 장애 조치 클러스터에 애플리케이션 및 서비스를 배포해야 합니다.

## <a name="objectives"></a>목표

이 랩을 완료하면 다음을 수행할 수 있습니다.

- 장애 조치 클러스터를 구성합니다.
- 장애 조치 클러스터에서 고가용성 파일 서버를 배포하고 구성합니다.
- 고가용성 파일 서버 배포 유효성 확인

## <a name="estimated-time-60-minutes"></a>예상 시간: **60분**

## <a name="lab-setup"></a>랩 설정

가상 머신: **AZ-801T00A-SEA-DC1**, **AZ-801T00A-SEA-SVR1**, **AZ-801T00A-SEA-SVR2** 를 실행해야 합니다. 다른 VM을 실행할 수도 있지만 이 랩에서는 필요하지 않습니다.

> **참고**: **AZ-801T00A-SEA-DC1**, **AZ-801T00A-SEA-SVR1**, **AZ-801T00A-SEA-SVR2** 가상 머신은 **SEA-DC1**, **SEA-SVR1**, **SEA-SVR2** 의 설치를 호스팅합니다.

1. **SEA-SVR2** 를 선택합니다.
1. 다음 자격 증명을 사용하여 로그인합니다.

   - 사용자 이름: **Administrator**
   - 암호: **Pa55w.rd**
   - 도메인: **CONTOSO**

이 랩에서는 사용 가능한 VM 환경을 사용합니다.

## <a name="exercise-1-configuring-iscsi-storage"></a>연습 1: iSCSI 스토리지 구성

### <a name="scenario"></a>시나리오

Contoso에는 고가용성으로 만들려는 중요한 애플리케이션 및 서비스가 있습니다. 이러한 서비스 중 일부는 NLB를 사용할 수 없으므로 장애 조치 클러스터링을 구현하기로 결정했습니다. 장애 조치 클러스터링에 iSCSI(Internet Small Computer System Interface) 스토리지를 사용하기로 결정했습니다. 먼저 장애 조치 클러스터를 지원하도록 iSCSI 스토리지를 구성합니다.

이 연습의 주요 작업은 다음과 같습니다.

- 장애 조치 클러스터링 설치
- iSCSI 가상 디스크를 구성

#### <a name="task-1-install-failover-clustering"></a>작업 1: 장애 조치 클러스터링 설치

1. **SEA-SVR2** 에서 관리자 권한으로 Windows PowerShell을 시작합니다.
1. **SEA-SVR2** 에서 Windows PowerShell 사용하여 **SEA-SVR1** 및 **SEA-SVR2** 에 관리 도구를 포함한 **장애 조치 클러스터링** 기능을 설치합니다.
1. **SEA-SVR2** 에서 Windows PowerShell 사용하여 **SEA-DC1** 에 **FS-iSCSITarget-Server** 기능을 설치합니다.
 
#### <a name="task-2-configure-iscsi-virtual-disks"></a>작업 2: iSCSI 가상 디스크 구성

> **중요:** 랩은 AD DS(Active Directory Domain Services) 도메인 컨트롤러 역할을 하는 **SEA-DC1** 을 사용하여 Windows 서버 기반 클러스터에 대한 공유 iSCI 스토리지를 호스트합니다. 이는 권장 구성을 나타내기 위한 것이 아니라 랩 구성을 단순화하고 랩 가상 머신 수를 최소화하기 위해 수행됩니다. 모든 프로덕션 환경에서는 장애 조치 클러스터에 대한 공유 스토리지를 호스트하는 데 도메인 컨트롤러를 사용하면 안 됩니다. 대신 이러한 스토리지는 고가용성 인프라에서 호스트되어야 합니다. 

1. **SEA-SVR2** 에서 두 개의 Windows PowerShell 창을 관리자 권한으로 시작합니다.
1. 두 번째 및 세 번째 Windows PowerShell 창을 사용하여 각각 **SEA-DC1** 및 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 설정합니다.
1. **SEA-DC1** 에 대한 PowerShell 원격 세션을 사용하여 **C:\\Storage** 디렉터리를 만들고 다음 매개 변수를 사용하여 **New-IscsiVirtualDisk** cmdlet을 실행하여 **SEA-DC1** 에 세 개의 iSCSI 가상 디스크를 만듭니다.

   - Disk1:
      - 스토리지 위치: **C:\\Storage**
      - 디스크 이름: **Disk1**
      - 크기: **10GB**
   - Disk2:
      - 스토리지 위치: **C:\\Storage**
      - 디스크 이름: **Disk2**
      - 크기: **10GB**
   - Disk3:
      - 스토리지 위치: **C:\\Storage**
      - 디스크 이름: **Disk3**
      - 크기: **10GB**

1. **SEA-SVR2** 및 **SEA-SVR1** 에서 iSCSI 초기자(MSiSCSI) 서비스를 시작하고 **SEA-SVR2** 에 대한 로컬 Windows PowerShell 세션 및 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 사용하여 **Start-Service** 및 **Set-Service** cmdlets를 실행하여 자동으로 시작되도록 구성합니다.
1. 다음 매개 변수를 사용하여 **New-IscsiServerTarget** cmdlet을 실행하기 위해 **SEA-DC1** 에 대한 PowerShell 원격 세션을 사용하여 **SEA-DC1** 에서 iSCSI 대상을 만듭니다.

   - 대상 이름: **ISCSI-L03**
   - InitiatorsIds: 
      - **"IQN:iqn.1991-05.com.microsoft:sea-svr1.contoso.com"**
      - **"IQN:iqn.1991-05.com.microsoft:sea-svr2.contoso.com"**

### <a name="results"></a>결과

이 연습을 완료한 후에는 **SEA-SVR1** 및 **SEA-SVR2** 에 장애 조치 클러스터링 기능을 성공적으로 설치하고 **SEA-DC1**, **SEA-SVR1**, **SEA-SVR2** 에 iSCSI 구성 요소를 초기화해야 합니다.

## <a name="exercise-2-configuring-a-failover-cluster"></a>연습 2: 장애 조치 클러스터 구성

### <a name="scenario"></a>시나리오

이 연습에서는 클러스터 설치를 준비한 다음 클러스터를 만듭니다.

이 연습의 주요 작업은 다음과 같습니다.

1. 클라이언트를 iSCSI 대상에 연결
1. 디스크 초기화
1. 장애 조치 클러스터 유효성 검사 및 만들기

#### <a name="task-1-connect-clients-to-the-iscsi-targets"></a>작업 1: iSCSI 대상에 클라이언트 연결

1. **SEA-SVR2** 에서 **SEA-DC1** 에 PowerShell 원격 세션을 호스트하는 **Windows PowerShell** 창을 사용해 **SEA-DC1** 에 iSCSI 디스크를 탑재하여 **Add-IscsiVirtualDiskTargetMapping** cmdlet을 실행합니다.
1. **SEA-SVR2** 에서 로컬 Windows PowerShell 세션을 사용하여 **SEA-SVR2** 에서 **SEA-DC1** 에 호스트된 iSCSI 대상에 연결하여 다음 명령을 실행합니다.

   ```powershell
   New-iSCSITargetPortal -TargetPortalAddress SEA-DC1.contoso.com  
   Connect-iSCSITarget -NodeAddress iqn.1991-05.com.microsoft:sea-dc1-iSCSI-L03-target
   Get-iSCSITarget | fl
   ```

   > **참고:** 마지막 명령을 실행한 후 **IsConnected** 변수의 값이 True인지 확인합니다.

1. **SEA-SVR2** 에서 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 사용하여 **SEA-SVR1** 에서 **SEA-DC1** 에 호스트된 iSCSI 대상에 연결하여 다음 명령을 실행합니다.

   ```powershell
   New-iSCSITargetPortal -TargetPortalAddress SEA-DC1.contoso.com  
   Connect-iSCSITarget -NodeAddress iqn.1991-05.com.microsoft:sea-dc1-iSCSI-L03-target
   Get-iSCSITarget | fl
   ```

   > **참고:** 마지막 명령을 실행한 후 **IsConnected** 변수의 값이 True인지 확인합니다.


#### <a name="task-2-initialize-the-disks"></a>작업 2: 디스크 초기화

1. **SEA-SVR2** 에서 로컬 Windows PowerShell 세션을 사용해 **Get-Disk** cmdlet을 실행하여 로컬 디스크를 나열합니다.
1. **SEA-SVR2** 에서 로컬 Windows PowerShell 세션을 사용하여 다음 설정으로 세 개의 iSCSI 디스크를 초기화합니다.

   - PartitionStyle: **MBR**
   - New-Partition 크기: **5GB**
   - 파일 시스템: **NTFS**
   - 드라이브 문자 지정: **E**, **F**, **G**

#### <a name="task-3-create-a-failover-cluster"></a>작업 3: 장애 조치 클러스터 만들기

1. **SEA-SVR2** 에서 로컬 Windows PowerShell 세션을 사용하여 첫 번째 노드로 IP 주소가 **172.16.10.125** 및 **SEA-SVR2.contoso.com** 으로 설정된 **SEA-CL03** 이라는 장애 조치 클러스터를 만듭니다.
1. **SEA-SVR2** 에서 로컬 Windows PowerShell 세션을 사용하여 새로 만든 클러스터에 **SEA-SVR1** 을 두 번째 노드로 추가합니다.

### <a name="results"></a>결과

이 연습을 완료한 후에는 디스크를 구성하고 장애 조치 클러스터를 만들어야 합니다.

## <a name="exercise-3-deploying-and-configuring-a-highly-available-file-server"></a>연습 3: 고가용성 파일 서버 배포 및 구성

### <a name="scenario"></a>시나리오

Contoso에서 파일 서비스는 고가용성으로 만들어야 하는 중요한 서비스입니다. 클러스터 인프라를 만든 후에는 고가용성 파일 서버를 구성한 다음 장애 조치 및 장애 복구(failback)에 대한 설정을 구현하기로 결정합니다.

이 연습의 주요 작업은 다음과 같습니다.

1. 파일 서버 애플리케이션을 장애 조치 클러스터에 추가합니다.
1. 고가용성 파일 서버에 공유 폴더를 추가합니다.
1. 장애 조치 및 장애 복구 설정을 구성합니다.

#### <a name="task-1-add-the-file-server-application-to-the-failover-cluster"></a>작업 1: 장애 조치 클러스터에 파일 서버 애플리케이션 추가

1. **SEA-SVR2** 에서 **장애 조치(Failover) 클러스터 관리자** 콘솔을 열고 **SEA-CL03.contoso.com** 클러스터에 자동으로 연결되어 있는지 확인합니다.
1. **Nodes** 노드에서 **SEA-SVR2** 및 **SEA-SVR1** 노드가 모두 실행 중인지 확인합니다.
1. **Storage** 노드에서 **디스크** 를 선택한 다음, 세 개의 클러스터 디스크가 온라인 상태인지 확인합니다.
1. **File Server for general use**(일반 용도의 파일 서버) 옵션 및 다음 설정을 사용하여 **파일 서버** 를 클러스터 역할로 추가합니다.

   - 클라이언트 액세스 이름: **FSCluster**
   - 주소: **172.16.0.130**
   - 스토리지: **클러스터 디스크 1, 클러스터 디스크 2**

#### <a name="task-2-add-a-shared-folder-to-a-highly-available-file-server"></a>작업 2: 고가용성 파일 서버에 공유 폴더 추가

1. **SEA-SVR2** 의 **장애 조치(Failover) 클러스터 관리자** 콘솔에서 다음 설정을 사용하여 **FSCluster** 역할에 파일 공유를 추가합니다(다른 모든 항목은 기본값으로 유지).

   - 파일 공유 프로필: **SMB 공유 - 빠른**
   - 이름: **문서**

#### <a name="task-3-configure-the-failover-and-failback-settings"></a>작업 3: 장애 조치 및 장애 복구 설정 구성

1. **SEA-SVR2** 의 **장애 조치(Failover) 클러스터 관리자** 콘솔에서 **FSCluster** 클러스터 역할의 **속성** 대화 상자를 사용하여 다음 설정을 구성합니다.

   - 장애 복구: **4~5시간**
   - 선호 소유자: **SEA-SVR2** 및 **SEA-SVR1**(목록의 맨 위에 **SEA-SVR1** 포함)

### <a name="results"></a>결과

이 연습을 완료한 후에는 고가용성 파일 서버를 구성해야 합니다.

## <a name="exercise-4-validating-the-deployment-of-the-highly-available-file-server"></a>연습 4: 고가용성 파일 서버 배포 유효성 검사

### <a name="scenario"></a>시나리오

장애 조치 클러스터를 구현할 때 장애 조치 및 장애 복구 테스트를 수행하려고 합니다. 또한 쿼럼에서 감시 디스크를 변경하려고 합니다.

이 연습의 주요 작업은 다음과 같습니다.

1. 고가용성 파일 서버 배포 유효성 확인
1. 파일 서버 역할에 대한 장애 조치 및 쿼럼 구성의 유효성 검사

#### <a name="task-1-validate-the-highly-available-file-server-deployment"></a>작업 1: 고가용성 파일 서버 배포 유효성 검사

1. **SEA-SVR2** 에서 파일 탐색기를 열고 **\\\\FSCluster\\Docs** 공유에 액세스할 수 있는지 확인합니다.
1. 공유에 테스트 텍스트 문서를 만듭니다.
1. **SEA-SVR2** 에서 **장애 조치(Failover) 클러스터 관리자** 콘솔을 사용하여 **FSCluster** 역할을 다른 노드로 이동합니다.
1. **SEA-SVR2** 의 파일 탐색기를 열고 **\\\\FSCluster\\Docs** 공유에 계속 액세스할 수 있는지 확인합니다.

#### <a name="task-2-validate-the-failover-and-quorum-configuration-for-the-file-server-role"></a>작업 2: 파일 서버 역할에 대한 장애 조치 및 쿼럼 구성의 유효성 검사

1. **SEA-SVR2** 의 **장애 조치(Failover) 클러스터 관리자** 콘솔에서 **FSCluster** 역할의 현재 소유자를 식별합니다.
1. **FSCluster** 역할의 현재 소유자인 노드에서 클러스터 서비스를 중지합니다.
1. **SEA-SVR2** 에서 파일 탐색기를 사용하여 **\\\\FSCluster\\Docs** 공유를 계속 사용할 수 있는지 확인합니다.
1. **SEA-SVR2** 에서 **장애 조치(failover) 클러스터 관리자** 콘솔을 사용하여 2단계에서 중지한 노드에서 클러스터 서비스를 시작합니다.
1. **SEA-SVR2** 의 **장애 조치(failover) 클러스터 관리자** 콘솔에서 기본 설정을 사용하여 **FSCluster** 에 대한 클러스터 쿼럼을 구성합니다.
1. **장애 조치(failover) 클러스터 관리자** 콘솔에서 **Disks** 노드를 찾아보고 **쿼럼에서 감시 디스크** 로 표시된 디스크를 오프라인으로 전환합니다.
1. **SEA-SVR2** 에서 파일 탐색기를 사용하여 **\\\\FSCluster\\Docs** 공유를 계속 사용할 수 있는지 확인합니다.
1. 감시 디스크를 온라인 상태로 설정합니다.

### <a name="results"></a>결과

이 연습을 완료한 후에는 장애 조치 클러스터링을 사용하여 고가용성 유효성을 검사해야 합니다.

