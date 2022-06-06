---
lab:
  title: '랩: 장애 조치(failover) 클러스터링 구현'
  type: Answer Key
  module: 'Module 3: High availability in Windows Server'
ms.openlocfilehash: 9f61985bbfde16e2653daff8032ebfb785cac219
ms.sourcegitcommit: fb0d39e25bc0fe182037587b772d217db126d3bb
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/07/2022
ms.locfileid: "144813005"
---
# <a name="lab-answer-key-implementing-failover-clustering"></a>랩 응답 키: 장애 조치 클러스터링 구현

## <a name="exercise-1-configuring-iscsi-storage"></a>연습 1: iSCSI 스토리지 구성

#### <a name="task-1-install-failover-clustering"></a>작업 1: 장애 조치 클러스터링 설치

1. **SEA-SVR2** 에 연결한 다음, 필요하다면 **Pa55w.rd** 암호를 이용해 **Contoso\\Administrator** 로 로그인합니다.
1. **SEA-SVR2** 에서 **시작** 을 선택한 후 **Windows PowerShell(관리자)** 을 선택합니다.
1. **SEA-SVR1** 및 **SEA-SVR2** 에 관리 도구를 포함하여 장애 조치 클러스터링 서버 기능을 설치하려면 Windows PowerShell 명령 프롬프트에서 다음 명령을 입력하고 각 명령을 입력한 후 Enter 키를 누릅니다.

   ```powershell
   Install-WindowsFeature –Name Failover-Clustering –IncludeManagementTools
   Install-WindowsFeature -ComputerName 'SEA-SVR1.contoso.com' –Name Failover-Clustering –IncludeManagementTools
   ```

   > **참고**: 설치 프로세스가 완료될 때까지 기다립니다. 설치하는 데 1분 정도 걸립니다.

1. **SEA-DC1** 에 iSCSI 대상 서버 역할 서비스를 설치하려면 Windows PowerShell 명령 프롬프트에서 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Install-WindowsFeature -ComputerName 'SEA-DC1.contoso.com' –Name FS-iSCSITarget-Server –IncludeManagementTools
   ```

   > **참고**: 설치 프로세스가 완료될 때까지 기다립니다. 설치하는 데 1분 정도 걸립니다.

#### <a name="task-2-configure-iscsi-virtual-disks"></a>작업 2: iSCSI 가상 디스크 구성

> **중요:** 랩은 AD DS(Active Directory Domain Services) 도메인 컨트롤러 역할을 하는 **SEA-DC1** 을 사용하여 Windows 서버 기반 클러스터에 대한 공유 iSCI 스토리지를 호스트합니다. 이는 권장 구성을 나타내기 위한 것이 아니라 랩 구성을 단순화하고 랩 가상 머신 수를 최소화하기 위해 수행되는 것입니다. 모든 프로덕션 환경에서는 장애 조치 클러스터에 대한 공유 스토리지를 호스트하는 데 도메인 컨트롤러를 사용하면 안 됩니다. 대신 이러한 스토리지는 고가용성 인프라에서 호스트되어야 합니다. 

1. **SEA-SVR2** 에서 **시작** 을 선택한 후 **Windows PowerShell(관리자)** 을 선택합니다.
1. **SEA-DC1** 에 대한 PowerShell 원격 세션을 설정하려면 새로 연 **Windows PowerShell** 창에서 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Enter-PSSession -ComputerName SEA-DC1.contoso.com
   ```

1. **SEA-DC1** 에 iSCSI 가상 디스크를 만들려면 **SEA-SVR2** 의 PowerShell 원격 세션에서 **SEA-DC1** 에 다음 명령을 입력하고 각 명령을 입력한 후 Enter 키를 누릅니다.

   ```powershell
   New-Item -ItemType Directory C:\Storage -Force
   New-IscsiVirtualDisk C:\Storage\disk1.VHDX –size 10GB
   New-IscsiVirtualDisk C:\Storage\disk2.VHDX –size 10GB
   New-IscsiVirtualDisk C:\Storage\disk3.VHDX –size 10GB
   ```

1. **SEA-SVR2** 에서 **시작** 을 선택한 후 **Windows PowerShell(관리자)** 을 선택합니다.
1. **SEA-SVR1** 에 대한 PowerShell 원격 세션을 설정하려면 새로 연 **Windows PowerShell** 창에서 다음 명령을 입력한 다음, Enter 키를 누릅니다.

   ```powershell
   Enter-PSSession -ComputerName SEA-SVR1.contoso.com
   ```

   > **참고:** 이때 세 개의 **Windows PowerShell** 창이 열려 있어야 합니다. 첫 번째 명령을 사용하여 **SEA-SVR2** 에서 로컬로 명령을 실행하고 다른 두 명령을 사용하여 **SEA-DC1** 및 **SEA-SVR1** 과 상호 작용합니다. PowerShell 프롬프트를 식별하면 각 프롬프트를 쉽게 인식할 수 있습니다(두 번째 및 세 번째 프롬프트에는 각각 **[SEA-DC1.contoso.com]** 및 **[SEA-SVR1.contoso.com]** 접두사 포함).

1. **SEA-SVR2** 에서 Microsoft iSCSI 초기자 서비스를 시작하려면 로컬 세션에 대한 액세스를 제공하는 **Windows PowerShell** 프롬프트에서 다음 명령을 입력하고 각 명령을 입력한 후 Enter 키를 누릅니다.

   ```powershell
   Start-Service -ServiceName MSiSCSI
   Set-Service -ServiceName MSiSCSI -StartupType Automatic
   ```

1. **SEA-SVR1** 에서 Microsoft iSCSI 초기자 서비스를 시작하려면 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 호스트하는 **Windows PowerShell** 창으로 전환하고, 다음 명령을 입력하고 각 명령을 입력한 후 Enter 키를 누릅니다.

   ```powershell
   Start-Service -ServiceName MSiSCSI
   Set-Service -ServiceName MSiSCSI -StartupType Automatic
   ```

1. **SEA-DC1** 에서 Microsoft iSCSI 대상을 만들려면 **SEA-DC1** 에 대한 PowerShell 원격 세션을 호스트하는 **Windows PowerShell** 창으로 전환하고 다음 명령을 입력한 다음, Enter 키를 누릅니다.

   ```powershell
   New-IscsiServerTarget iSCSI-L03 –InitiatorIds “IQN:iqn.1991-05.com.microsoft:sea-svr1.contoso.com","IQN:iqn.1991-05.com.microsoft:sea-svr2.contoso.com"
   ```

## <a name="exercise-2-configuring-a-failover-cluster"></a>연습 2: 장애 조치 클러스터 구성

#### <a name="task-1-connect-clients-to-the-iscsi-targets"></a>작업 1: iSCSI 대상에 클라이언트 연결

1. **SEA-SVR2** 에서 **SEA-DC1** 에서 iSCSI 디스크를 탑재하려면 **SEA-DC1** 에 대한 PowerShell 원격 세션을 호스트하는 **Windows PowerShell** 창에서 다음 명령을 입력하고 각 명령을 입력한 후 Enter 키를 누릅니다.

   ```powershell
   Add-IscsiVirtualDiskTargetMapping iSCSI-L03 C:\Storage\Disk1.VHDX
   Add-IscsiVirtualDiskTargetMapping iSCSI-L03 C:\Storage\Disk2.VHDX
   Add-IscsiVirtualDiskTargetMapping iSCSI-L03 C:\Storage\Disk3.VHDX
   ```

1. **SEA-SVR2** 에서 **SEA-DC1** 에 호스트된 iSCSI 대상에 연결하려면 로컬 세션에 대한 액세스를 제공하는 **Windows PowerShell** 프롬프트로 전환하고, 다음 명령을 입력하고 후 각 명령을 입력한 후 Enter 키를 누릅니다.

   ```powershell
   New-iSCSITargetPortal –TargetPortalAddress SEA-DC1.contoso.com  
   Connect-iSCSITarget –NodeAddress iqn.1991-05.com.microsoft:sea-dc1-iSCSI-L03-target
   Get-iSCSITarget | fl
   ```

   > **참고:** 마지막 명령을 실행한 후 *IsConnected* 변수의 값이 True인지 확인합니다.

1. **SEA-SVR1** 에서 **SEA-DC1** 에 호스트된 iSCSI 대상에 연결하려면 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 호스트하는 Windows PowerShell 창으로 전환하고, 다음 명령을 입력하고 각 명령을 입력한 후 Enter 키를 누릅니다.

   ```powershell
   New-iSCSITargetPortal –TargetPortalAddress SEA-DC1.contoso.com 
   Connect-iSCSITarget –NodeAddress iqn.1991-05.com.microsoft:sea-dc1-iSCSI-L03-target
   Get-iSCSITarget | fl
   ```

   > **참고:** 마지막 명령을 실행한 후 *IsConnected* 변수의 값이 True인지 확인합니다.

#### <a name="task-2-initialize-the-disks"></a>작업 2: 디스크 초기화

1. **SEA-SVR2** 에서 디스크를 나열하려면 로컬 세션에 대한 액세스를 제공하는 Windows PowerShell 프롬프트로 전환하고 다음 명령을 입력한 다음 Enter 키를 누릅니다.

   ```powershell
   Get-Disk
   ```

   > **참고:** 세 개의 iSCSI 디스크가 **오프라인** 작동 상태로 나열되어 있는지 확인합니다. 숫자 1, 2, 3이 있는 디스크여야 합니다.

1. 디스크를 초기화하려면 로컬 세션에 대한 액세스를 제공하는 Windows PowerShell 프롬프트에서 다음 명령을 입력하고 각 명령을 입력한 후 Enter 키를 누릅니다.

   ```powershell
   Get-Disk | Where OperationalStatus -eq 'Offline' | Initialize-Disk -PartitionStyle MBR
   New-Partition -DiskNumber 1 -Size 5gb -AssignDriveLetter
   New-Partition -DiskNumber 2 -Size 5gb -AssignDriveLetter
   New-Partition -DiskNumber 3 -Size 5gb -AssignDriveLetter
   Format-Volume -DriveLetter E -FileSystem NTFS
   Format-Volume -DriveLetter F -FileSystem NTFS
   Format-Volume -DriveLetter G -FileSystem NTFS
   ```

   > **참고:** 명령을 실행하기 전에 디스크 번호가 이전 명령 출력과 일치하는지 확인합니다. 각 명령이 성공적으로 완료되었는지 확인합니다.

#### <a name="task-3-create-a-failover-cluster"></a>작업 3: 장애 조치 클러스터 만들기

1. **SEA-SVR2** 에서 장애 조치(failover) 클러스터를 만들려면 로컬 세션에 대한 액세스를 제공하는 Windows PowerShell 프롬프트에서 다음 명령을 입력한 다음, Enter 키를 누릅니다.

   ```powershell
   New-Cluster -Name SEA-CL03 -Node SEA-SVR2.contoso.com -StaticAddress 172.16.10.125
   ```

   > **참고:** 이 명령은 새로 만든 클러스터의 이름(**SEA-CL03**)을 반환해야 합니다. 

1. **SEA-SVR1** 을 새로 만든 클러스터에 다른 노드로 추가하려면 **SEA-SVR2** 에서 로컬 세션에 대한 액세스를 제공하는 Windows PowerShell 프롬프트에서 다음 명령을 입력한 후 Enter 키를 누릅니다.

   ```powershell
   Add-ClusterNode -Cluster SEA-CL03 -Name SEA-SVR1.contoso.com
   ```

   > **참고:** 명령이 성공적으로 완료되었는지 확인합니다.

## <a name="exercise-3-deploying-and-configuring-a-highly-available-file-server"></a>연습 3: 고가용성 파일 서버 배포 및 구성

#### <a name="task-1-add-the-file-server-application-to-the-failover-cluster"></a>작업 1: 장애 조치 클러스터에 파일 서버 애플리케이션 추가

1. **SEA-SVR2** 의 **시작** 을 선택하고 **시작** 메뉴에서 **서버 관리자** 를 선택한 다음, **서버 관리자** 에서 **도구** 메뉴에 있는 **장애 조치(failover) 클러스터 관리자** 를 선택합니다.

   > **참고:** **SEA-SVR2** 는 클러스터 노드 중 하나이므로 **장애 조치(Failover) 클러스터 관리자** 콘솔이 **SEA-CL03** 에 자동으로 연결됩니다.

1. **SEA-CL03.contoso.com** 노드를 확장하여 **역할** 을 선택하고 클러스터가 이 시점에서 어떤 역할도 호스트하지 않는지 확인합니다.
1. **Nodes** 노드를 선택하고 **SEA-SVR1** 및 **SEA-SVR2** 노드가 **Up** 상태로 표시되는지 확인합니다.
1. **Storage** 노드를 확장하고 **디스크** 를 선택합니다. 세 개의 클러스터 디스크가 **온라인** 상태로 표시됩니다.
1. **장애 조치(failover) 클러스터 관리자** 페이지에서 **역할** 의 상황에 맞는 메뉴를 마우스 오른쪽 단추로 클릭하거나 액세스한 다음 **역할 구성** 을 선택합니다. 그러면 **고가용성 마법사** 가 시작됩니다.
1. **고가용성 마법사** 의 **시작하기 전에** 페이지에서 **다음** 을 선택합니다.
1. **고가용성 마법사** 의 **역할 선택** 페이지에서 **파일 서버** 를 선택한 다음, **다음** 을 선택합니다.
1. **고가용성 마법사** 의 **파일 서버 유형** 페이지에서 **File Server for general use**(일반 용도의 파일 서버) 옵션이 선택되어 있는지 확인하고 **다음** 을 선택합니다.
1. **고가용성 마법사** 에서 **클라이언트 액세스 지점** 페이지의 **이름** 상자에 **FSCluster** 를 입력합니다.
1. **주소** 상자에 **172.16.10.130** 을 입력하고 **다음** 을 선택합니다.
1. **고가용성 마법사** 의 **스토리지 선택** 페이지에서 **클러스터 디스크 1** 및 **클러스터 디스크 2** 를 선택한 다음, **다음** 을 선택합니다.
1. **고가용성 마법사** 의 **확인** 페이지에서 **다음** 을 선택합니다.
1. **고가용성 마법사** 의 **요약** 페이지에서 **마침** 을 선택합니다.

   > **참고:** **Storage** 노드에서 **디스크** 노드를 선택한 상태에서 세 개의 클러스터 디스크가 온라인 상태인지 확인합니다. **클러스터 디스크 1** 및 **클러스터 디스크 2** 를 **FSCluster** 에 할당해야 합니다.

#### <a name="task-2-add-a-shared-folder-to-a-highly-available-file-server"></a>작업 2: 고가용성 파일 서버에 공유 폴더 추가

1. **SEA-SVR2** 의 **장애 조치(failover) 클러스터 관리자** 에서 **역할** 을 선택하고 **FSCluster** 를 선택한 다음, 작업 창에서 **파일 공유 추가** 를 선택합니다. 

   > **참고:** 그러면 **새 공유 마법사** 가 시작됩니다.

1. **프로필 선택** 페이지에서 **SMB 공유 - 빠른** 프로필이 선택되어 있는지 확인하고 **다음** 을 선택합니다.
1. **공유 위치** 페이지에서 **다음** 을 선택합니다.
1. **공유 이름** 페이지에서 공유 이름에 대한 **문서** 를 입력한 다음, **다음** 을 선택합니다.
1. **기타 설정** 페이지에서 **다음** 을 선택합니다.
1. **사용 권한** 페이지에서 **다음** 을 선택합니다.
1. **확인** 페이지에서 **만들기** 를 선택합니다.
1. **결과 보기** 페이지에서 **닫기** 를 선택합니다.

#### <a name="task-3-configure-the-failover-and-failback-settings"></a>작업 3: 장애 조치 및 장애 복구 설정 구성

1. **SEA-SVR2** 의 **장애 조치(failover) 클러스터 관리자** 콘솔에서 **Roles** 노드의 **FSCluster** 를 선택한 상태에서 작업 창에 있는 **속성** 을 선택합니다.
1. **장애 조치(failover)** 탭을 선택한 다음, **Allow failback**(장애 복구 허용) 옵션을 선택합니다.
1. 옵션 **간 장애 복구(Failback)** 를 선택한 다음, 다음 값을 입력합니다.

   - 첫 번째 텍스트 상자: **4**
   - 두 번째 텍스트 상자: **5**

1. **일반** 탭을 선택합니다.
1. **기본 설정 소유자** 섹션에서 **SEA-SVR1** 이 첫 번째 항목으로 나열되었는지 확인한 다음, **확인** 을 선택합니다.


## <a name="exercise-4-validating-the-deployment-of-the-highly-available-file-server"></a>연습 4: 고가용성 파일 서버 배포 유효성 검사

#### <a name="task-1-validate-the-highly-available-file-server-deployment"></a>작업 1: 고가용성 파일 서버 배포 유효성 검사

1. **SEA-SVR2** 에서 파일 탐색기를 열고 **\\\\FSCluster\\Docs** 폴더를 찾아봅니다.
1. **Docs** 폴더 내에서 폴더의 빈 영역의 상황에 맞는 메뉴를 마우스 오른쪽 단추로 클릭하거나 액세스하고 **새로 만들기** 를 선택한 다음, **텍스트 문서** 를 선택합니다.
1. 문서의 기본 이름을 **New Text Document.txt** 로 그대로 사용하려면 Enter 키를 누릅니다.
1. **SEA-SVR2** 에서 **장애 조치(failover) 클러스터 관리자** 콘솔로 전환하고, **FSCluster** 의 상황에 맞는 메뉴를 마우스 오른쪽 단추로 클릭하거나 액세스하고, **이동** 을 선택하고, **노드 선택** 을 선택하고, **SEA-SVR2** 를 선택한 다음, **확인** 을 선택합니다.
1. **SEA-SVR2** 에서 파일 탐색기로 다시 전환하고 **\\\\FSCluster\\Docs** 폴더의 콘텐츠에 계속 액세스할 수 있는지 확인합니다.

#### <a name="task-2-validate-the-failover-and-quorum-configuration-for-the-file-server-role"></a>작업 2: 파일 서버 역할에 대한 장애 조치 및 쿼럼 구성의 유효성 검사

1. **SEA-SVR2** 에서 **장애 조치(Failover) 클러스터 관리자** 콘솔로 전환하고 **FSCluster** 역할의 현재 소유자를 식별합니다.
1. **Nodes** 를 선택한 다음, 이전 단계에서 식별한 노드의 상황에 맞는 메뉴를 마우스 오른쪽 단추로 클릭하거나 액세스합니다.
1. 상황에 맞는 메뉴에서 **추가 작업** 을 선택한 다음, **클러스터 서비스 중지** 를 선택합니다.
1. 파일 탐색기로 전환하고 **\\\\FSCluster\\Docs** 폴더의 콘텐츠에 계속 액세스할 수 있는지 확인합니다.
1. **장애 조치(failover) 클러스터 관리자** 콘솔로 전환한 다음, **다운된** 상태인 노드의 상황에 맞는 메뉴를 마우스 오른쪽 단추로 클릭하거나 액세스합니다.
1. 상황에 맞는 메뉴에서 **추가 작업** 을 선택한 다음, **클러스터 서비스 시작** 을 선택합니다.
1. **장애 조치(failover) 클러스터 관리자** 콘솔에서 **SEA-CL03.Contoso.com** 클러스터에 대한 상황에 맞는 메뉴를 마우스 오른쪽 단추로 클릭하거나 액세스하고 **추가 작업** 을 선택한 다음, **클러스터 쿼럼 설정 구성** 을 선택합니다. 그러면 **클러스터 쿼럼 구성 마법사** 가 시작됩니다.
1. **시작하기 전에** 페이지에서 **다음** 을 선택합니다.
1. **쿼럼 구성 옵션 선택** 페이지에서 **기본 쿼럼 구성 사용** 옵션이 선택되어 있는지 확인하고 **다음** 을 선택합니다.
1. **확인** 페이지에서 기본적으로 **클러스터 디스크 3** 이 **디스크 감시** 로 선택되면 **다음** 을 선택합니다. 
1. **요약** 페이지에서 **마침** 을 선택합니다.
1. **장애 조치(failover) 클러스터 관리자** 콘솔에서 **Disks** 노드를 찾아보고 디스크 감시로 구성된 **클러스터 디스크 3** 을 선택한 다음, 작업 창에서 **오프라인 상태로 전환** 을 선택합니다.
1. 확인하라는 메시지가 표시되면 **예** 를 선택합니다.
1. 파일 탐색기로 전환하고 **\\\\FSCluster\\Docs** 폴더의 콘텐츠에 계속 액세스할 수 있는지 확인합니다.
1. **장애 조치(failover) 클러스터 관리자** 콘솔로 전환하고 **Disks** 노드 내의 디스크 목록에서 디스크 감시로 구성된 **클러스터 디스크 3** 을 선택한 다음, 작업 창에서 **온라인 상태로 전환** 을 선택합니다.
