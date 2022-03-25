---
lab:
  title: '랩: Windows Server에서 보안 구성'
  module: 'Module 1: Windows Server security'
ms.openlocfilehash: ed3fd5da4164e3fd48c9af7b6b37074c582618e0
ms.sourcegitcommit: 9a51ea796ef3806ab9e7ec1ff75034b2f929ed2a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/28/2022
ms.locfileid: "137907133"
---
# <a name="lab-configuring-security-in-windows-server"></a>랩: Windows Server에서 보안 구성

## <a name="scenario"></a>시나리오

Contoso 제약은 전 세계적으로 약 5,000명의 직원이 근무하는 의료 연구 회사입니다. 이 회사는 특히 의료 기록과 데이터를 비공개로 유지해야 합니다. 회사는 본사와 전 세계 여러 지점을 보유하고 있습니다. Contoso는 최근 Windows Server와 Windows 클라이언트 인프라를 배포했습니다. 서버 보안 구성을 개선하라는 요청을 받았습니다.

## <a name="objectives"></a>목표

이 랩을 완료하면 다음을 수행할 수 있습니다.

- Windows Defender Credential Guard를 구성합니다.
- 문제가 있는 사용자 계정을 찾습니다.
- LAPS(로컬 관리자 암호 솔루션)를 구현하고 확인합니다.

## <a name="estimated-time-40-minutes"></a>예상 시간: 40분

## <a name="lab-setup"></a>랩 설정

가상 머신: **AZ-801T00A-SEA-DC1**, **AZ-801T00A-SEA-SVR1**, **AZ-801T00A-SEA-SVR2** 를 실행해야 합니다. 다른 VM을 실행할 수도 있지만 이 랩에서는 필요하지 않습니다.

> **참고**: **AZ-801T00A-SEA-DC1**, **AZ-801T00A-SEA-SVR1**, **AZ-801T00A-SEA-SVR2** 가상 머신은 **SEA-DC1**, **SEA-SVR1**, **SEA-SVR2** 의 설치를 호스트합니다.

1. **SEA-SVR2** 를 선택합니다.
1. 다음 자격 증명을 사용하여 로그인:

   - 사용자 이름: **Administrator**
   - 암호: **Pa55w.rd**
   - 도메인: **CONTOSO**

이 랩에서는 사용 가능한 VM 환경을 사용합니다.

## <a name="exercise-1-configuring-windows-defender-credential-guard"></a>연습 1: Windows Defender Credential Guard 구성

### <a name="scenario"></a>시나리오

Pass-the-Hash 및 Pass-the-Ticket 자격 증명 도난으로부터 보호하기 위해 서버 및 관리 워크스테이션에 Windows Defender Credential Guard를 구현하기로 했습니다. 그룹 정책을 사용하여 기존 서버에서 Credential Guard를 활성화합니다. 모든 새 서버의 경우 HVCI(Hypervisor-Protected Code Integrity) 및 Windows Defender Credential Guard 하드웨어 준비 도구를 사용하여 새 서버가 도메인에 가입되기 전에 Credential Guard를 활성화합니다.

이 랩에서는 그룹 정책을 설정하고 기존 서버에서 HVCI 및 Windows Defender Credential Guard 하드웨어 준비 도구를 실행합니다.

> **참고**: 랩 환경에서는 필수 구성 요소를 충족하지 않으므로 VM에서 Credential Guard를 사용할 수 없습니다. 그러나 이렇게 해도 그룹 정책을 사용하고 해당 도구를 사용하여 준비 상태를 평가하여 구현 프로세스를 단계별로 진행할 수 있습니다.

이 연습의 주요 작업은 다음과 같습니다.

1. 그룹 정책을 사용하여 Windows Defender Credential Guard 사용
1. HVCI 및 Windows Defender Credential Guard 하드웨어 준비 도구를 사용하여 Windows Defender Credential Guard를 활성화합니다.

#### <a name="task-1-enable-windows-defender-credential-guard-using-group-policy"></a>작업 1: 그룹 정책을 사용하여 Windows Defender Credential Guard 사용

1. *SEA-SVR2**에서 **그룹 정책 관리 콘솔** 을 엽니다.
1. **그룹 정책 관리 콘솔** 에서 **Forest: Contoso.com**, **Domains**, **Contoso.com** 을 탐색한 다음, **IT** OU(조직 단위)에 연결된 **CredentialGuard_GPO** 라는 GPO(그룹 정책 개체)를 만듭니다.
1. 그룹 정책 관리 편집기에서 **CredentialGuard_GPO** 를 열고 **컴퓨터 구성\\정책\\관리 템플릿\\시스템\\디바이스 가드** 노드로 이동합니다.
1. 다음 설정을 사용하여 **가상화 기반 보안 켜기** 옵션을 활성화합니다.

   - 플랫폼 보안 수준 선택: **보안 부팅 및 DMA 보호**.
   - Credential Guard 구성: **UEFI 잠금으로 사용**
   - 보안 시작 구성: **사용**

1. **그룹 정책 관리 편집기** 창을 닫습니다.
1. **그룹 정책 관리 콘솔** 을 닫습니다.

#### <a name="task-2-enable-windows-defender-credential-guard-using-the-hvci-and-windows-defender-credential-guard-hardware-readiness-tool"></a>작업 2: HVCI 및 Windows Defender Credential Guard 하드웨어 준비 도구를 사용하여 Windows Defender Credential Guard를 활성화합니다.

1. **SEA-SVR2** 에서 관리자 권한으로 Windows PowerShell을 시작합니다.
1. HVCI 및 Windows Defender Credential Guard 하드웨어 준비 도구를 실행하려면 Windows PowerShell 명령 프롬프트에서 다음 명령을 실행합니다.

   ```powershell
   Set-Location -Path C:\Labfiles\Lab01\
   .\DG_Readiness_Tool.ps1 -Enable -AutoReboot
   ```

1. 도구가 실행을 완료할 때까지 기다렸다가 프롬프트에 따라 운영 체제를 다시 시작합니다.
1. 다시 시작이 완료되면 암호 **Pa55w.rd** 를 사용하여 **SEA-SVR2** 에 **CONTOSO\\Administrator** 로 다시 로그인합니다.

### <a name="results"></a>결과

이 연습을 완료하면 다음을 할 수 있게 됩니다.

1. 그룹 정책을 사용하여 조직의 모든 컴퓨터에서 Windows Defender Credential Guard를 구현할 수 있습니다.
1. 로컬 컴퓨터에서 Windows Defender 자격 증명 가드를 즉시 활성화할 수 있습니다.

## <a name="exercise-2-locating-problematic-accounts"></a>연습 2: 문제가 있는 계정 찾기

### <a name="scenario"></a>시나리오

조직에 만료되지 않도록 구성된 암호가 있는 사용자 계정이 있는지 확인하고 이 설정을 수정하려고 합니다. 또한 90일 이상 로그인하지 않은 계정을 확인하고 이를 비활성화하려고 합니다.

이 연습의 주요 작업은 다음과 같습니다.

1. 만료되지 않은 암호를 사용하여 도메인 계정 찾기 및 다시 구성
1. 90일 이상 로그인에 사용되지 않은 도메인 계정 찾기 및 사용하지 않도록 설정

#### <a name="task-1-locate-and-reconfigure-domain-accounts-with-non-expiring-passwords"></a>작업 1: 만료되지 않은 암호를 사용하여 도메인 계정 찾기 및 다시 구성

1. **SEA-SVR2** 에서 관리자 권한으로 Windows PowerShell을 시작합니다.
1. 만료되지 않는 암호를 사용하여 Active Directory 사용 사용자 계정을 나열하려면 Windows PowerShell 명령 프롬프트에서 다음 명령을 실행합니다.

   ```powershell
   Get-ADUser -Filter {Enabled -eq $true -and PasswordNeverExpires -eq $true}
   ```

1. 반환된 사용자 계정 목록을 검토합니다.
1. 결과 집합의 모든 사용자 계정의 암호를 만료하려면 Windows PowerShell 명령 프롬프트에서 다음 명령을 실행합니다.

   ```powershell
   Get-ADUser -Filter {Enabled -eq $true -and PasswordNeverExpires -eq $true} | Set-ADUser -PasswordNeverExpires $false
   ```

1. 결과를 확인하려면 2단계에서 명령을 다시 실행하고 결과가 반환되지 않음을 확인합니다.

#### <a name="task-2-locate-and-disable-domain-accounts-that-have-not-been-used-to-sign-in-for-at-least-90-days"></a>작업 2: 90일 이상 로그인에 사용되지 않은 도메인 계정을 찾아 비활성화

1. 최소 90일 동안 로그인에 사용되지 않은 Active Directory 사용자 계정을 식별하려면 Windows PowerShell 명령 프롬프트에서 다음 명령을 실행합니다.

   ```powershell
   $days = (Get-Date).AddDays(-90)
   Get-ADUser -Filter {LastLogonTimeStamp -lt $days -and enabled -eq $true} -Properties LastLogonTimeStamp
   ```

   > **참고**: 랩 환경에서는 결과가 반환되지 않습니다.

1. 90일 이상 로그인하는 데 사용되지 않은 Active Directory 사용자 계정을 비활성화하려면 다음 명령을 실행합니다.

   ```powershell
   Get-ADUser -Filter {LastLogonTimeStamp -lt $days -and enabled -eq $true} -Properties LastLogonTimeStamp | Disable-ADAccount
   ```

   > **참고**: 랩 환경에서는 결과가 반환되지 않습니다.

## <a name="exercise-3-implementing-laps"></a>연습 3: LAPS 구현

### <a name="scenario"></a>시나리오

현재 Contoso의 모든 서버 및 워크스테이션에 동일한 로컬 관리자 계정 암호가 사용됩니다. 이 문제를 해결하려면 LAP를 구성하고 배포합니다.

이 연습의 주요 작업은 다음과 같습니다.

1. LAPS(로컬 관리자 암호 솔루션)를 구현하기 위한 컴퓨터 계정을 준비합니다.
1. LAPS용 Active Directory(AD DS)를 준비합니다.
1. LAPS 클라이언트 쪽 확장을 배포합니다.
1. LAPS를 확인합니다.

#### <a name="task-1-prepare-computer-accounts-for-implementing-laps-local-administrator-password-solution"></a>작업 1: LAPS(로컬 관리자 암호 솔루션)를 구현하기 위한 컴퓨터 계정 준비

1. 지정된 OU를 만들고 **SEA-SVR1** 컴퓨터 계정을 해당 계정으로 이동하려면 **SEA-SVR2** 의 Windows PowerShell 명령 프롬프트에서 다음 명령을 실행합니다.

   ```powershell
   New-ADOrganizationalUnit -Name "Seattle_Servers"
   Get-ADComputer SEA-SVR1 | Move-ADObject –TargetPath "OU=Seattle_Servers,DC=Contoso,DC=com"
   ```

1. LAPS를 설치하려면 Windows PowerShell 명령 프롬프트에서 다음 명령을 실행합니다.

   ```powershell
   Msiexec /i C:\Labfiles\Lab01\LAPS.x64.msi
   ```

1. 메시지가 표시되면 **사용자 지정 설정** 페이지에 도달할 때까지 기본 설정을 사용하여 **로컬 관리자 암호 솔루션 설정 마법사 시작** 을 진행합니다.
1. **로컬 관리자 암호 솔루션 설정** 마법사의 **사용자 지정 설정** 페이지에서 **관리 도구** 옆에 있는 드롭다운 메뉴에서 **모든 기능이 로컬 하드 드라이브에 설치됩니다.** 를 선택한 다음, 기본 설정을 수락하고 설정을 실행하여 마법사를 완료합니다. 
1. 설치가 완료될 때까지 기다립니다. 
1. 다른 도메인에 가입된 서버에서 들어오는 SMB(서버 메시지 블록) 연결을 허용하는 고급 보안 규칙으로 Windows Defender 방화벽을 활성화하려면 Windows PowerShell 명령 프롬프트에서 다음 명령을 각각 입력한 후 Enter 키를 누릅니다.

   ```
   $rule = Get-NetFirewallRule | Where-Object DisplayName -eq 'File and Printer Sharing (SMB-In)' 
   $rule | Set-NetFirewallRule -Profile Domain
   $rule | Enable-NetFirewallRule
   ```

   > **참고**: 이 랩에서 나중에 **SEA-SVR1** 에서 **SEA-SVR2** 에 연결하는 데 필요합니다.

#### <a name="task-2-prepare-ad-ds-for-laps"></a>작업 2: LAPS용 AD DS 준비

1. LAPS용 도메인을 준비하려면 **SEA-SVR2** 의 Windows PowerShell 명령 프롬프트에서 다음 명령을 실행합니다.

   ```powershell
   Import-Module admpwd.ps
   Update-AdmPwdADSchema
   Set-AdmPwdComputerSelfPermission -Identity "Seattle_Servers"
   ```

1. *SEA-SVR2**에서 **그룹 정책 관리 콘솔** 을 엽니다.
1. **그룹 정책 관리 콘솔** 에서 **Forest: Contoso.com**, **Domains**, **Contoso.com** 을 탐색한 다음, **Seattle_Servers** OU에 연결된 **LAPS_GPO** 라는 GPO(그룹 정책 개체)를 만듭니다.
1. 그룹 정책 관리 편집기에서 **LAPS_GPO** 를 열고 **컴퓨터 구성\\정책\\관리 템플릿\\LAPS** 노드로 이동합니다.
1. **로컬 관리자 암호 관리 사용** 옵션을 활성화합니다.
1. 다음 설정을 사용하여 **암호 설정** 옵션을 구성합니다.

   - 플랫폼 보안 수준 선택: **보안 부팅 및 DMA 보호**.
   - Credential Guard 구성: **UEFI 잠금으로 사용**
   - 보안 시작 구성: **사용**

1. 그룹 정책 관리 편집기 창을 닫습니다.

   - 암호 길이: **20**
   - 암호 사용 기간(일): **30**

1. 그룹 정책 관리 편집기를 닫습니다.
1. **그룹 정책 관리 콘솔** 을 닫습니다.

#### <a name="task-3-deploy-laps-client-side-extension"></a>작업 3: LAPS 클라이언트 쪽 확장 배포

1. 콘솔 세션을 **SEA-SVR1** 로 전환한 다음, 필요한 경우 **Pa55w.rd** 암호를 사용하여 **CONTOSO\\Administrator** 로 로그인합니다.

   > **참고:** 이전 연습에서 암호 만료를 활성화하는 스크립트를 실행한 결과 암호를 변경하라는 메시지가 표시됩니다. 임의의 암호를 선택하여 랩의 나머지 부분에서 사용합니다.

1. 로그인하면 Windows PowerShell 명령 프롬프트에 액세스하려면 **SConfig** 메뉴를 종료합니다.
1. 기본 설정을 사용하여 자동으로 LAPS를 설치하려면 Windows PowerShell 명령 프롬프트에서 다음 명령을 실행합니다.

   ```powershell
   Start-Process msiexec.exe -Wait -ArgumentList '/i \\SEA-SVR2.contoso.com\c$\Labfiles\Lab01\LAPS.x64.msi /quiet'
   ```

1. LAPS 설정을 로컬로 적용할 그룹 정책 처리를 트리거하려면 Windows PowerShell 명령 프롬프트에서 다음 명령을 실행합니다.

   ```powershell
   gpupdate /force
   ```

#### <a name="task-4-verify-laps"></a>작업 4: LAPS 확인

1. **SEA-SVR2** 에 대한 콘솔 세션으로 전환합니다.
1. **시작** 메뉴에서 **LAPS UI** 를 시작합니다.
1. **LAPS UI** 대화 상자의 **ComputerName** 텍스트 상자에 **SEA-SVR1** 을 입력한 다음, **검색** 을 선택합니다.
1. **암호** 및 **암호 만료** 값을 검토한 다음, **종료** 를 선택합니다.
1. **Windows PowerShell** 콘솔로 전환한 다음, 암호 값을 확인하려면 Windows PowerShell 명령 프롬프트에서 다음 명령을 실행합니다.

   ```powershell
   Get-ADComputer -Identity SEA-SVR1 -Properties ms-Mcs-AdmPwd
   ```

1. SEA-SVR1에 할당된 암호를 검토하고 **LAPS UI** 도구에 표시된 암호와 일치하는지 확인합니다.

   > **참고:** 이 경우 암호 값은 중괄호 쌍으로 묶입니다.

### <a name="results"></a>결과

이 연습을 완료하면 다음을 할 수 있게 됩니다.

- LAPS용 OU 및 컴퓨터 계정을 준비할 수 있습니다.
- LAPS용 AD DS를 준비할 수 있습니다.
- LAPS 클라이언트 쪽 확장을 배포할 수 있습니다.
- LAPS를 성공적으로 구현했는지 확인할 수 있습니다.
