---
lab:
  title: '랩: Hyper-V 복제본 및 Windows Server 백업 구현'
  type: Answer Key
  module: 'Module 4: Disaster Recovery in Windows Server'
ms.openlocfilehash: 65c442ed6c2de7650e5ab4888fb080d38e8bc5d2
ms.sourcegitcommit: 9a51ea796ef3806ab9e7ec1ff75034b2f929ed2a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/28/2022
ms.locfileid: "137907135"
---
# <a name="lab-answer-key-implementing-hyper-v-replica-and-windows-server-backup"></a>랩 해답: Hyper-V 복제본 및 Windows Server 백업 구현

## <a name="exercise1-implementing-hyper-v-replica"></a>연습 1: Hyper-V 복제본 구현

#### <a name="task-1-install-and-configure-hyper-v-replica"></a>작업 1: Hyper-V 복제본 설치 및 구성

1. **SEA-SVR2** 에 연결한 다음, 필요하다면 **Pa55w.rd** 암호를 이용해 **Contoso\\Administrator** 로 로그인합니다.
1. **SEA-SVR2** 에서 **시작** 을 선택한 후 **Windows PowerShell(관리자)** 을 선택합니다.
1. **SEA-SVR2** 에서 고급 보안 **Hyper-V 복제본 HTTP 수신기(TCP-In)** 규칙을 사용하여 Windows Defender 방화벽의 상태를 식별하려면 Windows PowerShell 프롬프트에서 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Get-NetFirewallRule -DisplayName 'Hyper-V Replica HTTP Listener (TCP-In)'
   ```

1. **SEA-SVR2** 에서 고급 보안 **Hyper-V 복제본 HTTP 수신기(TCP-In)** 규칙을 사용하여 Windows Defender 방화벽을 사용하려면 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Enable-NetFirewallRule -DisplayName 'Hyper-V Replica HTTP Listener (TCP-In)'
   ```

1. **SEA-SVR2** 를 **Hyper-V 복제본** 용 복제본 서버로 구성하려면 다음 명령을 입력하고 각 명령을 입력한 후 Enter 키를 누릅니다.

   ```powershell
   New-Item -ItemType Directory -Path C:\ReplicaStorage -Force
   Set-VMReplicationServer -ReplicationEnabled $true -AllowedAuthenticationType Kerberos -ReplicationAllowedFromAnyServer $true -DefaultStorageLocation C:\ReplicaStorage
   ```

1. **Hyper-V 복제본** 에 대한 복제본 서버로 **SEA-SVR2** 가 구성되어 있는지 확인하려면 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Get-VMReplicationServer
   ```
  
   > **참고**: 명령의 출력에 다음 설정이 포함되어 있는지 확인합니다.

   - **RepEnabled: True**
   - **AuthType: Kerb**
   - **KerAuthPort: 80**
   - **CertAuthPort: 443**
   - **AllowAnyServer: True**

1. **SEA-SVR2** 에 있는 가상 머신을 식별하려면 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Get-VM
   ```

   > **참고**: 명령의 출력에 **SEA-CORE1** 이 포함되어 있는지 확인합니다. 

   > **참고**: **관리자: Windows PowerShell** 창을 열어 둡니다.

1. 다른 **관리자: Windows PowerShell** 창을 열려면 **SEA-SVR2** 에서 **시작** 을 선택하고 **Windows PowerShell(관리자)** 를 선택합니다.
1. **SEA-SVR1** 에 대한 PowerShell 원격 세션을 설정하려면 새로 연 Windows PowerShell 창에서 다음 명령을 입력한 다음, Enter 키를 눌러 합니다.

   ```powershell
   Enter-PSSession -ComputerName SEA-SVR1.contoso.com
   ```
 
   > **참고**: 이 경우 **[SEA-SVR1.contoso.com]** 접두사가 포함된 PowerShell 프롬프트를 기반으로 PowerShell 원격 세션을 인식할 수 있습니다.

1. **SEA-SVR1** 에서 고급 보안 **Hyper-V 복제본 HTTP 수신기(TCP-In)** 규칙을 사용하여 Windows Defender 방화벽의 상태를 식별하려면 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 호스트하는 Windows PowerShell 창에서 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Get-NetFirewallRule -DisplayName 'Hyper-V Replica HTTP Listener (TCP-In)'
   ```

   > **참고**: 출력을 검토하고 **Enabled** 속성이 **False** 로 설정되어 있는지 확인합니다. **Hyper-V 복제본** 을 사용하려면 이 방화벽 규칙을 사용해야 합니다.

1. **SEA-SVR1** 에서 고급 보안 **Hyper-V 복제본 HTTP 수신기(TCP-In)** 규칙으로 Windows Defender 방화벽을 사용하려면 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 호스트하는 Windows PowerShell 창에서 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Enable-NetFirewallRule -DisplayName 'Hyper-V Replica HTTP Listener (TCP-In)'
   ```

1. **SEA-SVR1** 을 **Hyper-V 복제본** 용 복제본 서버로 구성하려면 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 호스트하는 Windows PowerShell 창에서 다음 명령을 입력하고 각 명령을 입력한 후 Enter 키를 누릅니다.

   ```powershell
   New-Item -ItemType Directory -Path C:\ReplicaStorage -Force
   Set-VMReplicationServer -ReplicationEnabled $true -AllowedAuthenticationType Kerberos -ReplicationAllowedFromAnyServer $true -DefaultStorageLocation C:\ReplicaStorage
   ```

   > **참고**: **관리자: Windows PowerShell** 창을 열어 둡니다.

#### <a name="task-2-configure-hyper-v-replication"></a>작업 2: Hyper-V 복제 구성

1. **SEA-SVR2** 에서 로컬 PowerShell 세션을 표시하는 **관리자: Windows PowerShell** 창으로 전환합니다.
1. **SEA-SVR2** 에서 **SEA-SVR1** 로 가상 머신 **SEA-CORE1** 의 복제를 사용하려면 **SEA-SVR2** 의 로컬 세션 Windows PowerShell 프롬프트에서 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Enable-VMReplication SEA-CORE1 -ReplicaServerName SEA-SVR1.contoso.com -ReplicaServerPort 80 -AuthenticationType Kerberos -ComputerName SEA-SVR2.contoso.com
   ```

1. **SEA-SVR2** 에서 **SEA-SVR1** 로 가상 머신 **SEA-CORE1** 의 복제를 시작하려면 **SEA-SVR2** 에서 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Start-VMInitialReplication SEA-CORE1
   ```

1. **SEA-SVR2** 에서 **SEA-SVR1** 로 가상 머신 **SEA-CORE1** 의 복제 상태를 확인하려면 **SEA-SVR2** 에서 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Get-VMReplication
   ```

   > **참고**: 명령의 출력에서 **상태** 값을 식별하고 **InitialReplicationInProgress** 로 나열되었는지 확인합니다. 약 5분 동안 기다렸다가 동일한 명령을 다시 실행하고 **상태** 값이 **복제** 로 변경되었는지 확인합니다. 다음 단계를 진행하기 전에 이 일이 발생할 때까지 기다립니다. 또한 **주 서버** 가 **SEA-SVR2** 로 나열되고 **ReplicaServer** 가 **SEA-SVR1** 로 나열되었는지 확인합니다.

1. **SEA-SVR2** 에서 PowerShell 원격 세션을 **SEA-SVR1** 로 표시하는 **관리자: Windows PowerShell** 창으로 전환합니다.
1. **SEA-CORE1** 의 복제본이 **SEA-SVR1** 에 있는지 확인하려면 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 호스트하는 Windows PowerShell 창에서 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Get-VM
   ```

   > **참고**: 명령의 출력에 **SEA-CORE1** 이 나열되는지 확인합니다.

   > **참고**: 두 Windows PowerShell 세션을 모두 열어 둡니다.

#### <a name="task-3-validate-a-failover"></a>작업 3: 장애 조치(failover) 유효성 검사

1. **SEA-SVR2** 에서 로컬 PowerShell 세션을 표시하는 **관리자: Windows PowerShell** 창으로 전환합니다.
1. **SEA-SVR1** 에 대해 **SEA-CORE1** 가상 머신의 장애 조치(failover)를 준비하려면 **SEA-SVR2** 의 로컬 세션을 호스트하는 Windows PowerShell 창에서 다음 명령을 입력하고 각 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Start-VMFailover -Prepare -VMName SEA-CORE1 -ComputerName SEA-SVR2.contoso.com
   ```

   > **참고**: 메시지가 표시되면 **Y** 를 입력한 다음, Enter 키를 누릅니다. 이 명령은 보류 중인 변경 내용의 복제를 트리거하여 **SEA-CORE1** 의 계획된 장애 조치(failover)를 준비합니다.

1. **SEA-SVR2** 에서 PowerShell 원격 세션을 **SEA-SVR1** 로 표시하는 **관리자: Windows PowerShell** 창으로 전환합니다.
1. **SEA-SVR1** 에 대해 **SEA-CORE1** 가상 머신의 장애 조치(failover)를 시작하려면 **SEA-SVR2** 의 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 호스트하는 Windows PowerShell 창에서 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Start-VMFailover -VMName SEA-CORE1 -ComputerName SEA-SVR1.contoso.com
   ```

   > **참고**: 메시지가 표시되면 **Y** 를 입력한 다음, Enter 키를 누릅니다.

1. 복제본 VM을 주 VM으로 구성하려면 **SEA-SVR2** 의 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 호스트하는 Windows PowerShell 창에서 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Set-VMReplication -Reverse -VMName SEA-CORE1 -ComputerName SEA-SVR1.contoso.com
   ```

1. **SEA-SVR1** 에서 새로 지정된 주 VM을 시작하려면 **SEA-SVR2** 의 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 호스트하는 Windows PowerShell 창에서 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Start-VM -VMName SEA-CORE1 -ComputerName SEA-SVR1.contoso.com
   ```

1. VM을 시작하려면 **SEA-SVR2** 의 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 호스트하는 Windows PowerShell 창에서 다음 명령을 입력하고 각 명령을 입력한 후 Enter 키를 누릅니다.

   ```powershell
   Get-VM
   ```

   > **참고**: 결과 테이블에서 **상태** 가 **실행 중** 으로 나열되었는지 확인합니다.

1. **SEA-SVR1** 에서 **SEA-SVR2** 로의 가상 머신 **SEA-CORE1** 복제 상태를 식별하려면 **SEA-SVR2** 의 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 호스트하는 Windows PowerShell 창에서 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Get-VMReplication
   ```

   > **참고**: 명령의 출력에서 **상태** 값을 식별하고 **복제** 로 나열되었는지 확인합니다. 또한 **주 서버** 가 **SEA-SVR1** 로 나열되고 **ReplicaServer** 가 **SEA-SVR2** 로 나열되었는지 확인합니다.

1. 주 서버에서 VM 복제를 중지하려면 **SEA-SVR2** 의 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 호스트하는 Windows PowerShell 창에서 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Stop-VM -VMName SEA-CORE1
   ```

1. 두 Windows PowerShell 창을 모두 열어 둡니다.

   > **참고**: 그래픽 도구를 사용하여 이 연습의 결과를 확인하려면 **SEA-SVR2** 에서 Hyper-V 관리자를 사용한 다음, **Hyper-V** 콘솔에 **SEA-SVR1** 및 **SEA-SVR2** 서버를 추가할 수 있습니다. 그럼 다음, **SEA-CORE1** VM이 **SEA-SVR1** 및 **SEA-SVR2** 모두에 있고 **SEA-SVR2** 에서 **SEA-SVR1** 로 복제가 실행 중인지 확인할 수 있습니다.

## <a name="exercise-2-implementing-backup-and-restore-with-windows-server-backup"></a>연습 2: Windows Server 백업을 사용하여 백업 및 복원 구현

### <a name="task1-configure-windows-server-backup-settings"></a>작업 1: Windows Server 백업 설정 구성

1. **SEA-SVR2** 에서 작업 표시줄에서 **파일 탐색기** 아이콘을 선택하여 파일 탐색기를 엽니다.
1. 파일 탐색기에서 **탐색** 창에서 **로컬 디스크(C:)** 를 선택합니다.
1. 세부 정보 창의 빈 공간에 있는 상황에 맞는 메뉴를 마우스 오른쪽 단추로 클릭하거나 액세스하고 **새로 만들기** 를 선택한 다음, **폴더** 를 선택합니다. 
1. **BackupShare** 폴더의 이름을 지정합니다. **BackupShare** 폴더의 상황에 맞는 메뉴를 마우스 오른쪽 단추로 클릭하거나 액세스하고 **액세스 권한 부여** 를 선택한 다음, **특정 사용자** 를 선택합니다.
1. **네트워크 액세스** 창에서 **인증된 사용자** 를 입력한 다음, **추가** 를 선택합니다. **사용 권한 수준** 열에서 **인증된 사용자** 의 값을 **읽기/쓰기** 로 설정하고 **공유** 를 선택한 다음, **완료** 를 선택합니다.
1. **SEA-SVR2** 에서 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 호스트하는 **관리자: Windows PowerShell** 창으로 전환합니다. 
1. **SEA-SVR1** 에 **Windows Server Backup** 역할을 설치하려면 **SEA-SVR2** 의 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 호스트하는 Windows PowerShell 창에서 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Install-WindowsFeature Windows-Server-Backup
   ```

   > **참고**: 설치가 완료될 때까지 기다리세요.

1. **wbadmin** 명령줄 유틸리티의 기능을 검토하려면 **SEA-SVR2** 의 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 호스트하는 Windows PowerShell 창에서 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   wbadmin /?
   ```

1. **WindowsServerBackup** 모듈에 포함된 Windows PowerShell cmdlet의 기능을 검토하려면 **SEA-SVR2** 의 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 호스트하는 Windows PowerShell 창에서 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Get-Command -Module WindowsServerBackup -CommandType Cmdlet
   ```

#### <a name="task-2-perform-a-backup-to-a-network-share"></a>작업 2: 네트워크 공유에 백업 수행

1. **SEA-SVR1** 에서 백업할 폴더와 파일을 만들려면 **SEA-SVR2** 의 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 호스트하는 Windows PowerShell 창에서 다음 명령을 입력하고 각 명령을 입력한 후 Enter 키를 누릅니다.

   ```powershell
   New-Item -ItemType Directory -Path 'C:\Files' -Force
   fsutil file createnew C:\Files\report1.docx 25432108
   fsutil file createnew C:\Files\report2.docx 25432108
   fsutil file createnew C:\Files\report3.docx 25432108
   fsutil file createnew C:\Files\report4.docx 25432108
   ```

1. Windows Server 백업을 사용하여 백업 정책 및 백업할 파일 경로에 대한 변수를 정의하려면 **SEA-SVR2** 의 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 호스트하는 Windows PowerShell 창에서 다음 명령을 입력하고 각 명령을 입력한 후 Enter 키를 누릅니다.

   ```powershell
   $policy = New-WBPolicy
   $fileSpec = New-WBFileSpec -FileSpec 'C:\Files'
   ```

1. 이전 단계에서 정의한 변수를 참조하는 Windows 서버 백업 정책을 정의하려면 **SEA-SVR2** 의 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 호스트하는 Windows PowerShell 창에서 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Add-WBFileSpec -Policy $policy -FileSpec $fileSpec
   ```

1. 이전 작업에서 만든 네트워크 공유를 사용하여 **SEA-SVR2** 에 백업 위치를 구성하려면 **SEA-SVR2** 의 **SEA-SVR1** 에 대한 PowerShell Remoting 세션을 호스트하는 Windows PowerShell 창에서 다음 명령을 입력하고 각 명령을 입력한 후 Enter 키를 누릅니다(로그인하라는 메시지가 표시되면 **CONTOSO\\Administrator** 사용자 이름과 **Pa55w.rd** 암호를 입력).

   ```powershell
   $cred = Get-Credential
   $networkBackupLocation = New-WBBackupTarget -NetworkPath "\\SEA-SVR2.contoso.com\BackupShare" -Credential $cred
   ```

1. 백업 정책에 백업 위치를 추가하려면 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 호스트하는 Windows PowerShell 창에서 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Add-WBBackupTarget -Policy $policy -Target $networkBackupLocation
   ```

1. 볼륨 섀도 복사본 서비스를 활성화하려면 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 호스트하는 Windows PowerShell 창에서 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Set-WBVssBackupOptions -Policy $policy -VssCopyBackup
   ```

1. 백업 작업을 시작하려면 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 호스트하는 Windows PowerShell 창에서 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Start-WBBackup -Policy $policy
   ```

   > **참고**: 백업이 완료될 때까지 기다립니다. 이 작업은 1분 정도 걸립니다.

1. **SEA-SVR2** 에서 파일 탐색기로 전환하고 **C:\\BackupShare** 로 이동하여 폴더에 **WindowsImageBackup** 하위 폴더에 새로 만든 백업이 포함되어 있는지 확인합니다.