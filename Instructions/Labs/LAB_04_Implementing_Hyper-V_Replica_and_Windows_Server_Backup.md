---
lab:
  title: '랩: Hyper-V 복제본 및 Windows Server 백업 구현'
  module: 'Module 4: Disaster Recovery in Windows Server'
ms.openlocfilehash: 9f668ce6b8f9f2c6802de4a03ee0038b3066f34e
ms.sourcegitcommit: d2e9d886e710729f554d2ba62d1abe3c3f65fcb6
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/10/2022
ms.locfileid: "147046984"
---
# <a name="lab-implementing-hyper-v-replica-and-windows-server-backup"></a>랩: Hyper-V 복제본 및 Windows Server 백업 구현

## <a name="scenario"></a>시나리오

Contoso, Ltd.에서 관리자로 일하고 있습니다. Contoso는 새로운 재해 복구 및 백업 기능과 기술을 평가하고 구성하려고 합니다. 시스템 관리자로서 귀하는 해당 평가 및 구현을 수행하는 임무를 받았습니다. **Hyper-V 복제본** 및 Windows Server 백업을 평가하기로 결정했습니다.

## <a name="objectives"></a>목표

이 랩을 완료하면 다음을 수행할 수 있습니다.

- **Hyper-V 복제본** 구성 및 구현
- Windows Server 백업을 사용하여 백업을 구성하고 구현합니다.

## <a name="estimated-time-45-minutes"></a>예상 시간: 45분

## <a name="lab-setup"></a>랩 설정

가상 머신: **AZ-801T00A-SEA-DC1**, **AZ-801T00A-SEA-SVR1**, **AZ-801T00A-SEA-SVR2** 를 실행해야 합니다. 다른 VM을 실행할 수도 있지만 이 랩에서는 필요하지 않습니다.

> **참고**: **AZ-801T00A-SEA-DC1**, **AZ-801T00A-SEA-SVR1**, **AZ-801T00A-SEA-SVR2** 가상 머신은 **SEA-DC1**, **SEA-SVR1**, **SEA-SVR2** 의 설치를 호스팅합니다.

1. **SEA-SVR2** 를 선택합니다.
1. 다음 자격 증명을 사용하여 로그인합니다.

   - 사용자 이름: **Administrator**
   - 암호: **Pa55w.rd**
   - 도메인: **CONTOSO**

이 랩에서는 사용 가능한 VM 환경을 사용합니다.

## <a name="exercise-1-implementing-hyper-v-replica"></a>연습 1: Hyper-V 복제본 구현

### <a name="scenario"></a>시나리오

클러스터 배포를 시작하기 전에 호스트 간에 VM을 복제하기 위한 Hyper-V의 새로운 기술을 평가하기로 결정했습니다. 활성 복사본 또는 호스트가 실패하는 경우 다른 호스트에 VM의 복사본을 수동으로 탑재할 수 있어야 합니다.

이 연습의 주요 작업은 다음과 같습니다.

1. Hyper-V 복제본 설치 및 구성
1. Hyper-V 복제 구성
1. 장애 조치(failover) 유효성 검사

#### <a name="task-1-install-and-configure-hyper-v-replica"></a>작업 1: Hyper-V 복제본 설치 및 구성

1. **SEA-SVR2** 에서 관리자 권한으로 Windows PowerShell을 시작합니다.
1. **SEA-SVR2** 에서 고급 보안 **Hyper-V 복제본 HTTP 수신기(TCP-In)** 규칙을 사용하여 Windows Defender 방화벽의 상태를 식별하려면 Windows PowerShell 프롬프트에서 다음 명령을 실행합니다.

   ```powershell
   Get-NetFirewallRule -DisplayName 'Hyper-V Replica HTTP Listener (TCP-In)'
   ```

1. **SEA-SVR2** 에서 고급 보안 **Hyper-V 복제본 HTTP 수신기(TCP-In)** 규칙을 사용하여 Windows Defender 방화벽을 사용하려면 Windows PowerShell 프롬프트에서 다음 명령을 실행합니다.

   ```powershell
   Enable-NetFirewallRule -DisplayName 'Hyper-V Replica HTTP Listener (TCP-In)'
   ```

1. **HYPER-V 복제본** 에 대한 복제본 서버로 **SEA-SVR2** 를 구성하려면 다음 명령을 실행합니다.

   ```powershell
   New-Item -ItemType Directory -Path C:\ReplicaStorage -Force
   Set-VMReplicationServer -ReplicationEnabled $true -AllowedAuthenticationType Kerberos -KerberosAuthenticationPort 8080 -ReplicationAllowedFromAnyServer $true -DefaultStorageLocation C:\ReplicaStorage
   ```

1. **HYPER-V 복제본** 에 대한 복제본 서버로 **SEA-SVR2** 가 구성되어 있는지 확인하려면 다음 명령을 실행합니다.

   ```powershell
   Get-VMReplicationServer
   ```
  
   > **참고**: 명령의 출력에 다음 설정이 포함되어 있는지 확인합니다.

   - **RepEnabled: True**
   - **AuthType: Kerb**
   - **KerAuthPort: 8080**
   - **CertAuthPort: 443**
   - **AllowAnyServer: True**

1. **SEA-SVR2** 에 있는 가상 머신을 식별하려면 Windows PowerShell 프롬프트에서 다음 명령을 실행합니다.

   ```powershell
   Get-VM
   ```

   > **참고**: 명령의 출력에 **SEA-CORE1** 이 포함되어 있는지 확인합니다. 

   > **참고**: **관리자: Windows PowerShell** 창을 열어 둡니다.

1. **SEA-SVR2** 에서 관리자 권한으로 다른 Windows PowerShell 창을 엽니다.
1. **SEA-SVR1** 에 대한 PowerShell 원격 세션을 설정하려면 새로 연 Windows PowerShell 창에서 다음 명령을 입력한 다음, Enter 키를 누릅니다.

   ```powershell
   Enter-PSSession -ComputerName SEA-SVR1.contoso.com
   ```
 
   > **참고**: 이 경우 **[SEA-SVR1.contoso.com]** 접두사가 포함된 PowerShell 프롬프트를 기반으로 PowerShell 원격 세션을 인식할 수 있습니다.

1. **SEA-SVR1** 에서 고급 보안 **Hyper-V 복제본 HTTP 수신기(TCP-In)** 규칙을 사용하여 Windows Defender 방화벽의 상태를 식별하려면 PowerShell 원격 세션을 **SEA-SVR1** 로 호스트하는 Windows PowerShell 창에서 다음 명령을 실행합니다.

   ```powershell
   Get-NetFirewallRule -DisplayName 'Hyper-V Replica HTTP Listener (TCP-In)'
   ```

   > **참고**: 출력을 검토하고 **Enabled** 속성이 **False** 로 설정되어 있는지 확인합니다. **Hyper-V 복제본** 을 사용하려면 이 방화벽 규칙을 사용하도록 설정해야 합니다.

1. **SEA-SVR1** 에서 고급 보안 **Hyper-V 복제본 HTTP 수신기(TCP-In)** 규칙으로 Windows Defender 방화벽을 사용하려면 PowerShell 원격 세션을 **SEA-SVR1** 로 호스트하는 Windows PowerShell 창에서 다음 명령을 실행합니다.

   ```powershell
   Enable-NetFirewallRule -DisplayName 'Hyper-V Replica HTTP Listener (TCP-In)'
   ```

1. **SEA-SVR1** 을 **Hyper-V 복제본** 용 복제본 서버로 구성하려면 PowerShell 원격 세션을 **SEA-SVR1** 로 호스트하는 Windows PowerShell 창에서 다음 명령을 실행합니다.

   ```powershell
   New-Item -ItemType Directory -Path C:\ReplicaStorage -Force
   Set-VMReplicationServer -ReplicationEnabled $true -AllowedAuthenticationType Kerberos -ReplicationAllowedFromAnyServer $true -DefaultStorageLocation C:\ReplicaStorage
   ```

   > **참고**: 두 번째 **관리자: Windows PowerShell** 창을 열어 둡니다.

#### <a name="task-2-configure-hyper-v-replication"></a>작업 2: Hyper-V 복제 구성

1. **SEA-SVR2** 에서 로컬 PowerShell 세션을 표시하는 **관리자: Windows PowerShell** 창으로 전환합니다.
1. **SEA-SVR2** 에서 **SEA-SVR1** 로 가상 머신 **SEA-CORE1** 의 복제를 사용하도록 설정하려면 **SEA-SVR2** 의 로컬 세션 Windows PowerShell 프롬프트에서 다음 명령을 실행합니다.

   ```powershell
   Enable-VMReplication SEA-CORE1 -ReplicaServerName SEA-SVR1.contoso.com -ReplicaServerPort 80 -AuthenticationType Kerberos -ComputerName SEA-SVR2.contoso.com
   ```

1. **SEA-SVR2** 에서 **SEA-SVR1** 로 가상 머신 **SEA-CORE1** 의 복제를 시작하려면 **SEA-SVR2** 에서 다음 명령을 실행합니다.

   ```powershell
   Start-VMInitialReplication SEA-CORE1
   ```

1. **SEA-SVR2** 에서 **SEA-SVR1** 로 가상 머신 **SEA-CORE1** 의 복제 상태를 식별하려면 다음 명령을 실행합니다.

   ```powershell
   Get-VMReplication
   ```

   > **참고**: 명령의 출력에서 **상태** 값을 식별하고 **InitialReplicationInProgress** 로 나열되었는지 확인합니다. 약 5분 동안 기다렸다가 동일한 명령을 다시 실행하고 **상태** 값이 **복제** 로 변경되었는지 확인합니다. 다음 단계를 진행하기 전에 이 일이 발생할 때까지 기다립니다. 또한 **주 서버** 가 **SEA-SVR2** 로 나열되고 **ReplicaServer** 가 **SEA-SVR1** 로 나열되었는지 확인합니다.

1. **SEA-SVR2** 에서 PowerShell 원격 세션을 **SEA-SVR1** 로 표시하는 **관리자: Windows PowerShell** 창으로 전환합니다.
1. **SEA-CORE1** 의 복제본이 **SEA-SVR1** 에 있는지 확인하려면 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 호스트하는 Windows PowerShell 창에서 다음 명령을 실행합니다.

   ```powershell
   Get-VM
   ```

   > **참고**: 명령의 출력에 **SEA-CORE1** 이 나열되는지 확인합니다.

   > **참고**: Windows PowerShell 세션을 모두 열어 둡니다.

##### <a name="task-3-validate-a-failover"></a>작업 3: 장애 조치(failover) 유효성 검사

1. **SEA-SVR2** 에서 로컬 PowerShell 세션을 표시하는 **관리자: Windows PowerShell** 창으로 전환합니다.
1. **SEA-CORE1** 가상 머신에서 **SEA-SVR1** 로의 장애 조치를 준비하려면 **SEA-SVR2** 의 로컬 세션을 호스트하는 Windows PowerShell 창에서 다음 명령을 실행합니다.

   ```powershell
   Start-VMFailover -Prepare -VMName SEA-CORE1 -ComputerName SEA-SVR2.contoso.com
   ```

   > **참고**: 메시지가 표시되면 **Y** 를 입력한 다음, Enter 키를 누릅니다. 이 명령은 보류 중인 변경 내용의 복제를 트리거하여 **SEA-CORE1** 의 계획된 장애 조치를 준비합니다.

1. **SEA-SVR2** 에서 PowerShell 원격 세션을 **SEA-SVR1** 로 표시하는 **관리자: Windows PowerShell** 창으로 전환합니다.
1. **SEA-CORE1** 가상 머신에서 **SEA-SVR1** 로의 장애 조치를 시작하려면 **SEA-SVR2** 에서 **SEA-SVR1** 의 PowerShell 원격 세션을 호스트하는 Windows PowerShell 창에서 다음 명령을 실행합니다.

   ```powershell
   Start-VMFailover -VMName SEA-CORE1 -ComputerName SEA-SVR1.contoso.com
   ```

1. 복제본 VM을 주 VM으로 구성하려면 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 호스트하는 Windows PowerShell 창의 **SEA-SVR2** 에서 다음 명령을 실행합니다.

   ```powershell
   Set-VMReplication -Reverse -VMName SEA-CORE1 -ComputerName SEA-SVR1.contoso.com
   ```

1. **SEA-SVR1** 에서 새로 지정된 주 VM을 시작하려면 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 호스트하는 Windows PowerShell 창의 **SEA-SVR2** 에서 다음 명령을 실행합니다.

   ```powershell
   Start-VM -VMName SEA-CORE1 -ComputerName SEA-SVR1.contoso.com
   ```

1. VM을 시작하려면 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 호스트하는 Windows PowerShell 창의 **SEA-SVR2** 에서 다음 명령을 실행합니다.

   ```powershell
   Get-VM
   ```

   > **참고**: 결과 테이블에서 **상태** 가 **실행 중** 으로 나열되었는지 확인합니다.

1. **SEA-SVR1** 에서 **SEA-SVR2** 로의 가상 머신 **SEA-CORE1** 복제 상태를 식별하려면 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 호스트하는 Windows PowerShell 창의 **SEA-SVR2** 에서 다음 명령을 실행합니다.

   ```powershell
   Get-VMReplication
   ```

   > **참고**: 명령의 출력에서 **상태** 값을 식별하고 **복제** 로 나열되었는지 확인합니다. 또한 **주 서버** 가 **SEA-SVR1** 로 나열되고 **ReplicaServer** 가 **SEA-SVR2** 로 나열되었는지 확인합니다.

1. 주 서버에서 VM 복제를 중지하려면 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 호스트하는 Windows PowerShell 창의 **SEA-SVR2** 에서 다음 명령을 실행합니다.

   ```powershell
   Stop-VM -VMName SEA-CORE1
   ```

1. Windows PowerShell 창을 모두 열어 둡니다.

   > **참고**: 그래픽 도구를 사용하여 이 연습의 결과를 확인하려면 **SEA-SVR2** 에서 Hyper-V 관리자를 사용한 다음, **Hyper-V** 콘솔에 **SEA-SVR1** 및 **SEA-SVR2** 서버를 추가할 수 있습니다. **SEA-CORE1** VM이 **SEA-SVR1** 및 **SEA-SVR2** 둘 다에 있고 복제가 **SEA-SVR2** 에서 **SEA-SVR1** 로 실행되고 있는지 확인할 수 있습니다.

## <a name="exercise-2-implementing-backup-and-restore-with-windows-server-backup"></a>연습 2: Windows Server 백업을 사용하여 백업 및 복원 구현

### <a name="scenario"></a>시나리오

멤버 서버에 대한 Windows Server 백업을 평가해야 합니다. **SEA-SVR2** 서버의 Windows Server 백업을 구성하고 **SEA-SVR2** 의 네트워크 공유에 대한 평가판 백업을 수행하기로 결정했습니다.

이 연습의 주요 작업은 다음과 같습니다.

1. Windows Server 백업 설정 구성
1. 네트워크 공유에 대한 백업 수행

#### <a name="task-1-configure-windows-server-backup-settings"></a>작업 1: Windows Server 백업 설정 구성

1. **SEA-SVR2** 에서 파일 탐색기를 사용하여 **SEA-SVR2** 에 **C:\\BackupShare** 폴더를 만듭니다. **인증된 사용자** 에게 읽기/쓰기 권한이 있도록 폴더를 공유합니다.
1. **SEA-SVR2** 에서 PowerShell 원격 세션을 **SEA-SVR1** 로 표시하는 **관리자: Windows PowerShell** 창으로 전환합니다.
1. **SEA-SVR1** 에 대한 PowerShell 원격 세션에서 **Install-WindowsFeature** cmdlet을 사용하여 **SEA-SVR1** 에 **Windows-Server-Backup** 기능을 설치합니다.
1. **SEA-SVR1** 에 대한 PowerShell 원격 세션에서 **wbadmin /?** 및 **Get-Command** 명령을 사용하여 **wbadmin** 유틸리티의 기능과 **WindowsServerBackup** 모듈의 cmdlet을 검토합니다.

#### <a name="task-2-perform-a-backup-to-a-network-share"></a>작업 2: 네트워크 공유에 대한 백업 수행

1. **SEA-SVR2** 에서 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 사용하여 **SEA-SVR1** 에 백업할 폴더와 파일을 만들어 다음 명령을 실행합니다.

   ```powershell
   New-Item -ItemType Directory -Path 'C:\Files' -Force
   fsutil file createnew C:\Files\report1.docx 25432108
   fsutil file createnew C:\Files\report2.docx 25432108
   fsutil file createnew C:\Files\report3.docx 25432108
   fsutil file createnew C:\Files\report4.docx 25432108
   ```

1. Windows Server 백업을 사용하여 백업 정책 및 백업할 파일 경로에 대한 변수를 정의하려면 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 호스트하는 Windows PowerShell 창의 **SEA-SVR2** 에서 다음 명령을 실행합니다.

   ```powershell
   $policy = New-WBPolicy
   $fileSpec = New-WBFileSpec -FileSpec 'C:\Files'
   ```

1. 이전 단계에서 정의한 변수를 참조하는 Windows Server 백업 정책을 정의하려면 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 사용하여 **SEA-SVR2** 에서 다음 명령을 실행합니다.

   ```powershell
   Add-WBFileSpec -Policy $policy -FileSpec $fileSpec
   ```

1. 이전 작업에서 만든 네트워크 공유를 사용하여 **SEA-SVR2** 에서 백업 위치를 구성하려면 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 사용하여 **SEA-SVR2** 에서 다음 명령을 실행합니다(로그인하라는 메시지가 표시되면 **CONTOSO\\Administrator** 사용자 이름과 **Pa55w.rd** 암호 입력).

   ```powershell
   $cred = Get-Credential
   $networkBackupLocation = New-WBBackupTarget -NetworkPath "\\SEA-SVR2.contoso.com\BackupShare" -Credential $cred
   ```

1. 백업 정책에 백업 위치를 추가하려면 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 사용하여 다음 명령을 실행합니다.

   ```powershell
   Add-WBBackupTarget -Policy $policy -Target $networkBackupLocation
   ```

1. 볼륨 섀도 복사본 서비스를 사용하려면 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 사용하여 다음 명령을 실행합니다.

   ```powershell
   Set-WBVssBackupOptions -Policy $policy -VssCopyBackup
   ```

1. 백업 작업을 시작하려면 **SEA-SVR1** 에 대한 PowerShell 원격 세션을 사용하여 다음 명령을 실행합니다.

   ```powershell
   Start-WBBackup -Policy $policy
   ```

   > **참고**: 백업이 완료될 때까지 기다립니다. 이 작업은 1분 정도 걸립니다.

1. **SEA-SVR2** 에서 파일 탐색기로 전환하고 **C:\\BackupShare** 를 찾아본 다음 폴더에 **WindowsImageBackup** 하위 폴더에 새로 만든 백업이 포함되어 있는지 확인합니다.
