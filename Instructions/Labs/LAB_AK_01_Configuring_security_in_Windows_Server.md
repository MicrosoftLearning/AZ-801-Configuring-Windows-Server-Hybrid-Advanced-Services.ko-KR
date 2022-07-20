---
lab:
  title: '랩: Windows Server에서 보안 구성'
  type: Answer Key
  module: 'Module 1: Windows Server security'
ms.openlocfilehash: 42202d9eb5bf7c9a371c3215120ec8b7f6da0d46
ms.sourcegitcommit: d2e9d886e710729f554d2ba62d1abe3c3f65fcb6
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/10/2022
ms.locfileid: "147046981"
---
# <a name="lab-answer-key-configuring-security-in-windows-server"></a>랩 응답 키: Windows Server에서 보안 구성

## <a name="exercise-1-configuring-windows-defender-credential-guard"></a>연습 1: Windows Defender Credential Guard 구성

> **참고**: 랩 환경에서는 필수 구성 요소를 충족하지 않으므로 VM에서 Credential Guard를 사용할 수 없습니다. 그러나 이렇게 해도 그룹 정책을 사용하고 해당 도구를 사용하여 준비 상태를 평가하여 구현 프로세스를 단계별로 진행할 수 있습니다.

#### <a name="task-1-enable-windows-defender-credential-guard-using-group-policy"></a>작업 1: 그룹 정책을 사용하여 Windows Defender Credential Guard 사용

1. **SEA-SVR2** 에 연결한 다음, 필요하다면 **Pa55w.rd** 암호를 이용해 **CONTOSO\\Administrator** 로 로그인합니다.
1. **시작** 단추 옆에 있는 **검색하려면 여기에 입력** 텍스트 상자에 **그룹 정책 관리** 를 입력합니다.
1. 결과 목록에서 **그룹 정책 관리** 를 선택합니다.
1. **그룹 정책 관리** 콘솔에서 **포리스트: contoso.com** 확장, **도메인** 확장, **contoso.com** 확장, **IT** OU( 조직 구성 단위)의 **상황에 맞는** 메뉴를 마우스 오른쪽 단추로 클릭하거나 액세스한 다음, **이 도메인에서 GPO를 만들어 여기에 연결** 을 선택합니다.
1. **새 GPO** 대화 상자의 **이름** 텍스트 상자에 **CredentialGuard_GPO** 를 입력한 다음, **확인** 을 선택합니다.
1. **그룹 정책 관리** 창의 **IT** 에서 **CredentialGuard_GPO** 의 **상황에 맞는** 메뉴를 마우스 오른쪽 단추로 클릭하거나 액세스한 다음, **편집** 을 선택합니다.
1. **그룹 정책 관리 편집기** 에서 **컴퓨터 구성\\정책\\관리 템플릿\\시스템\\Device Guard** 로 이동합니다.
1. **가상화 기반 보안 켜기** 를 선택한 다음, **정책 설정** 링크를 선택합니다.
1. **가상화 기반 보안 켜기** 창에서 **사용** 옵션을 선택합니다.
1. **플랫폼 보안 수준 선택** 드롭다운 목록에서 **보안 부팅 및 DMA 보호** 항목이 선택되어 있는지 확인합니다.
1. **Credential Guard 구성** 드롭다운 목록에서 **UEFI 잠금을 사용하여 설정** 항목을 선택합니다.
1. **보안 시작 구성** 드롭다운 목록에서 **사용** 항목을 선택한 다음, **확인** 을 선택합니다.
1. **그룹 정책 관리 편집기** 창을 닫습니다.
1. **그룹 정책 관리** 콘솔 창을 닫습니다.

#### <a name="task-2-enable-windows-defender-credential-guard-using-the-hypervisor-protected-code-integrity-hvci-and-windows-defender-credential-guard-hardware-readiness-tool"></a>작업 2: HVCI(Hypervisor-Protected 코드 무결성) 및 Windows Defender Credential Guard 하드웨어 준비 도구를 사용하여 Windows Defender Credential Guard 사용

1. **SEA-SVR2** 에서 **시작** 을 선택하고 **Windows PowerShell** 의 **상황에 맞는** 메뉴를 마우스 오른쪽 단추로 클릭하거나 액세스한 다음, **관리자 권한으로 실행** 을 선택합니다.
1. HVCI 및 Windows Defender Credential Guard 하드웨어 준비 도구를 실행하려면 Windows PowerShell 명령 프롬프트에서 다음 명령을 입력하고 첫 번째 프롬프트에서 **[R] 한 번 실행** 을 선택한 다음 나머지 프롬프트에 대해 Enter 키를 누릅니다.

   ```powershell
   Set-Location -Path C:\Labfiles\Lab01\
   .\DG_Readiness_Tool.ps1 -Enable -AutoReboot
   ```

1. 도구가 실행을 완료할 때까지 기다렸다가 메시지가 표시되면 **You're about to be signed out**(로그아웃하려고 합니다) 대화 상자에서 **확인** 을 선택합니다.

   > **참고**: 운영 체제가 다시 시작됩니다. 

1. 다시 시작이 완료되면 암호 **Pa55w.rd** 를 사용하여 **SEA-SVR2** 에 **CONTOSO\\Administrator** 로 다시 로그인합니다.

## <a name="exercise-2-locating-problematic-accounts"></a>연습 2: 문제가 있는 계정 찾기

#### <a name="task-1-locate-and-reconfigure-domain-accounts-with-non-expiring-passwords"></a>작업 1: 만료되지 않은 암호를 사용하여 도메인 계정 찾기 및 다시 구성

1. **SEA-SVR2** 에서 **시작** 을 선택하고 **Windows PowerShell** 의 **상황에 맞는** 메뉴를 마우스 오른쪽 단추로 클릭하거나 액세스한 다음, **관리자 권한으로 실행** 을 선택합니다.
1. 만료되지 않는 암호를 사용하여 Active Directory 사용 사용자 계정을 나열하려면 Windows PowerShell 명령 프롬프트에서 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Get-ADUser -Filter {Enabled -eq $true -and PasswordNeverExpires -eq $true}
   ```

1. 반환된 사용자 계정 목록을 검토합니다.
1. 결과 집합의 모든 사용자 계정에 대해 암호 만료를 사용하도록 설정하려면 Windows PowerShell 명령 프롬프트에서 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Get-ADUser -Filter {Enabled -eq $true -and PasswordNeverExpires -eq $true} | Set-ADUser -PasswordNeverExpires $false
   ```

1. 결과를 확인하려면 2단계에서 명령을 다시 실행하고 결과가 반환되지 않음을 확인합니다.

#### <a name="task-2-locate-and-disable-domain-accounts-that-have-not-been-used-to-sign-in-for-at-least-90-days"></a>작업 2: 90일 이상 로그인에 사용되지 않은 도메인 계정 찾기 및 사용하지 않도록 설정

1. 최소 90일 동안 로그인에 사용되지 않은 Active Directory 사용자 계정을 식별하려면 Windows PowerShell 명령 프롬프트에서 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   $days = (Get-Date).AddDays(-90)
   Get-ADUser -Filter {LastLogonTimeStamp -lt $days -and enabled -eq $true} -Properties LastLogonTimeStamp
   ```

   > **참고**: 랩 환경에서는 결과가 반환되지 않습니다.

1. 최소 90일 동안 로그인에 사용되지 않은 Active Directory 사용자 계정을 사용하지 않도록 설정하려면 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Get-ADUser -Filter {LastLogonTimeStamp -lt $days -and enabled -eq $true} -Properties LastLogonTimeStamp | Disable-ADAccount
   ```

   > **참고**: 랩 환경에서는 결과가 반환되지 않습니다.

## <a name="exercise-3-implementing-laps"></a>연습 3: LAPS 구현

#### <a name="task-1-prepare-computer-accounts-for-implementing-laps-local-administrator-password-solution"></a>작업 1: LAPS(로컬 관리자 암호 솔루션)를 구현하기 위한 컴퓨터 계정 준비

1. 지정된 OU를 만들고 **SEA-SVR1** 컴퓨터 계정을 해당 계정으로 이동하려면 **SEA-SVR2** 의 Windows PowerShell 명령 프롬프트에서 다음 명령을 입력하고 Enter 키를 누릅니다. 

   ```powershell
   New-ADOrganizationalUnit -Name "Seattle_Servers"
   Get-ADComputer SEA-SVR1 | Move-ADObject –TargetPath "OU=Seattle_Servers,DC=Contoso,DC=com"
   ```

1. LAPS를 설치하려면 Windows PowerShell 명령 프롬프트에서 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Msiexec /i C:\Labfiles\Lab01\LAPS.x64.msi
   ```

1. **로컬 관리자 암호 솔루션 설정** 마법사의 **로컬 관리자 암호 솔루션 설정 마법사 시작** 페이지에서 **다음** 을 선택합니다.
1. **로컬 관리자 암호 솔루션 설정** 마법사의 **최종 사용자 사용권 계약** 페이지에서 **동의함** 을 선택한 다음, **다음** 을 선택합니다.
1. **로컬 관리자 암호 솔루션 설정** 마법사의 **사용자 지정 설정** 페이지에서 **관리 도구** 옆에 있는 드롭다운 메뉴에서 **모든 기능이 로컬 하드 드라이브에 설치됩니다.** 를 선택한 다음, **다음** 을 선택합니다.
1. **로컬 관리자 암호 솔루션 설정** 마법사의 **로컬 관리자 암호 솔루션 설치 준비** 페이지에서 **설치** 를 선택합니다. 
1. 설치가 완료되면 **로컬 관리자 암호 솔루션 설정** 마법사의 마지막 페이지에서 **마침** 을 선택합니다.
1. 다른 도메인에 가입된 서버에서 들어오는 SMB(서버 메시지 블록) 연결을 허용하는 고급 보안 규칙으로 Windows Defender 방화벽을 사용하도록 설정하려면 Windows PowerShell 명령 프롬프트에서 다음 명령을 각각 입력한 후 Enter 키를 누릅니다.

   ```
   $rule = Get-NetFirewallRule | Where-Object DisplayName -eq 'File and Printer Sharing (SMB-In)' 
   $rule | Set-NetFirewallRule -Profile Domain
   $rule | Enable-NetFirewallRule
   ```

   > **참고**: 이 랩에서 나중에 **SEA-SVR1** 에서 **SEA-SVR2** 에 연결하는 데 필요합니다.

#### <a name="task-2-prepare-ad-ds-for-laps"></a>작업 2: LAPS용 AD DS 준비

1. LAPS용 도메인을 준비하려면 **SEA-SVR2** 의 Windows PowerShell 명령 프롬프트에서 다음 명령을 각각 입력한 후 Enter 키를 누릅니다.

   ```powershell
   Import-Module admpwd.ps
   Update-AdmPwdADSchema
   Set-AdmPwdComputerSelfPermission -Identity "Seattle_Servers"
   ```

1. **SEA-SVR2** 에서 **시작** 단추 옆에 있는 **검색하려면 여기에 입력** 텍스트 상자에 **그룹 정책 관리** 를 입력합니다.
1. 결과 목록에서 **그룹 정책 관리** 를 선택합니다.
1. **그룹 정책 관리** 콘솔에서 **포리스트: contoso.com** 확장, **도메인** 확장, **contoso.com** 확장, **Seattle_Servers** OU의 **상황에 맞는** 메뉴를 마우스 오른쪽 단추로 클릭하거나 액세스한 다음, **이 도메인에서 GPO를 만들어 여기에 연결** 을 선택합니다.
1. **새 GPO** 대화 상자의 **이름** 텍스트 상자에 **LAPS_GPO** 를 입력한 다음, **확인** 을 선택합니다.
1. **그룹 정책 관리** 창의 **Seattle_Servers** 에서 **LAPS_GPO** 의 **상황에 맞는** 메뉴를 마우스 오른쪽 단추로 클릭하거나 액세스한 다음, **편집** 을 선택합니다.
1. **그룹 정책 관리 편집기** 창의 **컴퓨터 구성** 에서 **정책** 노드를 확장하고 **관리 템플릿** 노드를 확장한 다음, **LAPS** 를 선택합니다.
1. **로컬 관리자 암호 관리 사용** 정책을 선택한 다음, **정책 설정** 링크를 선택합니다.
1. **로컬 관리자 암호 관리 사용** 창에서 **사용** 을 선택한 다음, **확인** 을 선택합니다.
1. **암호 설정** 정책을 선택한 다음, **정책 설정** 링크를 선택합니다.
1. **암호 설정** 정책 대화 상자에서 **사용** 을 선택한 다음, **암호 길이** 를 **20** 으로 구성합니다.
1. **암호 사용 기간(일)** 이 **30** 으로 구성되었는지 확인한 다음, **확인** 을 선택합니다.
1. 그룹 정책 관리 편집기를 닫습니다.

#### <a name="task-3-deploy-laps-client-side-extension"></a>작업 3: LAPS 클라이언트 쪽 확장 배포

1. 콘솔 세션을 **SEA-SVR1** 로 전환한 다음, 필요한 경우 **Pa55w.rd** 암호를 사용하여 **CONTOSO\\Administrator** 로 로그인합니다.

   > **참고:** 이전 연습에서 암호 만료를 사용하도록 설정하는 스크립트를 실행한 결과 암호를 변경하라는 메시지가 표시됩니다. 임의의 암호를 선택하여 랩의 나머지 부분에서 사용합니다.

1. 로그인한 후 Windows PowerShell 명령 프롬프트에 액세스하려면 **SConfig** 메뉴 프롬프트에서 **15** 를 입력하고 Enter 키를 누릅니다.
1. 기본 설정을 사용하여 자동으로 LAPS를 설치하려면 Windows PowerShell 명령 프롬프트에서 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Start-Process msiexec.exe -Wait -ArgumentList '/i \\SEA-SVR2.contoso.com\c$\Labfiles\Lab01\LAPS.x64.msi /quiet'
   ```

1. **LAPS** 설정을 로컬로 적용할 그룹 정책 처리를 트리거하려면 Windows PowerShell 명령 프롬프트에서 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   gpupdate /force
   ```

#### <a name="task-4-verify-laps"></a>작업 4: LAPS 확인

1. **SEA-SVR2** 에 대한 콘솔 세션으로 전환합니다.
1. **시작** 을 선택합니다. **시작** 메뉴에서 **LAPS** 를 선택한 다음, **LAPS UI** 를 선택합니다.
1. **LAPS UI** 대화 상자의 **ComputerName** 텍스트 상자에 **SEA-SVR1** 을 입력한 다음, **검색** 을 선택합니다.
1. **암호** 및 **암호 만료** 값을 검토한 다음, **종료** 를 선택합니다.
1. **Windows PowerShell** 콘솔로 전환한 다음 암호 값을 확인하려면 Windows PowerShell 명령 프롬프트에서 다음 명령을 입력하고 Enter 키를 누릅니다.

   ```powershell
   Get-ADComputer -Identity SEA-SVR1 -Properties ms-Mcs-AdmPwd
   ```

1. SEA-SVR1에 할당된 암호를 검토하고 **LAPS UI** 도구에 표시된 암호와 일치하는지 확인합니다.

   > **참고:** 이 경우 암호 값은 중괄호 쌍으로 묶입니다.
