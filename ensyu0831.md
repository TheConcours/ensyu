<meta charset="utf8">
<style>
    .hidden {
        background: gray;
        color: gray;
    }
</style>

<!--
# Part{{{counter}}}. Azure Log Analytics and KQL
[Log Analytics のチュートリアル](https://learn.microsoft.com/ja-jp/azure/azure-monitor/logs/log-analytics-tutorial)(日本語版)<br>
[Log Analytics tutorial](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/log-analytics-tutorial)(英語版)
## Intro
### List of Query Languages you should know
- XPATH
- XQuery
- jq
- SQL
- GraphQL
- KQL
### Azure Monitor Overview
### KQLと戯れる。
-->


# Part1. 小問集合
## Guidelines
下記の問題の中には、AZ900の範囲を越えて、クラウドエンジニアとして知っておいてもらいたい知識に関する問題も取り込んでおいた。
授業で扱っていない内容を扱った問題も存在するが、しっかり考えて、定型業務以外にも対応できる思考力を養ってほしい。

### Question1

  下記のコマンドを`bash`で実行できるように書き直せ。
  ```pwsh
  PS> az vm show -g "myResourceGroup" -n "myVM" -d `
  PS> | ConvertFrom-Json `
  PS> | select privateIps, publicIps, fqdns | fl
  ```

### Question2
  Azure MarketPlace で利用できるOS imageとして正しいものをすべて選びなさい。
  <table>
  <tr>
    <td>ubuntu</td><td>Slackware </td><td> Kali</td>
  </tr>
  <tr>
   <td>FreeBSD</td><td>Alma </td><td>MX Linux</td>
  </tr>
  <tr>
    <td>RHEL</td><td>AIX</td><td>Endeavour </td>
  </tr>
  <tr>
    <td>Cent</td><td> MacOS </td><td></td>
  </tr>
  </table>

### Question3

下記のコマンドをいれてもうまくloginできない。
```pwsh
PS> az login
PS> ^C
PS> Connect-AzAccount
PS> ^C
```
何を試すべきか？

### Question4
Azure <span class="hidden">*******</span>を用いれば、vmにpubic ipは必要ない。

### Question5
azure vm をdeployし、下記の設定を入れた。
```pwsh
PS> Invoke-AzVMRunCommand -ResourceGroupName "myResourceGroup" -VMName "myVM" `
PS> -CommandId "RunPowerShellScript" -ScriptString "Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0"
PS> Get-AzNetworkSecurityGroup -Name "myNSG" -ResourceGroupName "myResourceGroup" `
PS>  | Add-NetworkSecurityRuleConfig -Name allow-SSH -access Allow -Direction Inbound `
PS>    -Priority 1000 -SourceAddressPrefix "*" -SourcePortRange "*" `
PS>    -DestinationAddressPrefix "*" -DestinationPortRange "22" -Protocol TCP
PS>  | Set-AzNetworkSecurityGroup
PS> $MYPUBLICKEY = Get-Content ~/.ssh/id_rsa.pub
PS> Invoke-AzVMRunCommand -ResourceGroupName myResourceGroup -VMName myVM `
PS>  -CommandId 'RunPowerShellScript' -ScriptString $MYPUBLICKEY  + " | Add-Content 'C:\ProgramData\ssh\administrators_authorized_keys';icacls.exe 'C:\ProgramData\ssh\administrators_authorized_keys' /inheritance:r /grant 'Administrators:F' /grant 'SYSTEM:F'"
PS> Invoke-AzVMRunCommand -ResourceGroupName "myResourceGroup" -VMName "myVM" `
PS> -CommandId "RunPowerShellScript" -ScriptString "Start-Service sshd"
```
また自分のlocal pcでは、下記を確かめた。
```pwsh
PS> Get-Service ssh-agent
Status     Name            DisplayName
------     ----            -----------
Runniing   ssh-agent       OpenSSH Authentication Agent
```
`.ssh`下にはちゃんと`id_rsa`, `id_rsa.pub`もできている。どんな原因が考えられるだろうか。

### Question6
```pwsh
PS C:\Users\dts> systeminfo

Host Name:                 vm045
OS Name:                   Microsoft Windows Server 2019 Datacenter
OS Version:                10.0.17763 N/A Build 17763
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Multiprocessor Free
Registered Owner:          N/A
Registered Organization:   N/A
Product ID:                00430-00000-00000-AA325
Original Install Date:     8/23/2023, 4:12:11 AM
System Boot Time:          8/28/2023, 2:46:40 AM
System Manufacturer:       Microsoft Corporation
System Model:              Virtual Machine
System Type:               x64-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: Intel64 Family 6 Model 85 Stepping 4 GenuineIntel ~2095 Mhz
BIOS Version:              Microsoft Corporation Hyper-V UEFI Release v4.1, 5/9/2022
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume3
System Locale:             en-us;English (United States)
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC) Coordinated Universal Time
Total Physical Memory:     8,191 MB
Available Physical Memory: 6,624 MB
Virtual Memory: Max Size:  10,111 MB
Virtual Memory: Available: 8,618 MB
Virtual Memory: In Use:    1,493 MB
Page File Location(s):     D:\pagefile.sys
Domain:                    WORKGROUP
Logon Server:              N/A
Hotfix(s):                 4 Hotfix(s) Installed.
                           [01]: KB5028960
                           [02]: KB5004424
                           [03]: KB5029247
                           [04]: KB5028316
Network Card(s):           1 NIC(s) Installed.
                           [01]: Microsoft Hyper-V Network Adapter
                                 Connection Name: Ethernet
                                 DHCP Enabled:    Yes
                                 DHCP Server:     168.63.129.16
                                 IP address(es)
                                 [01]: 10.0.0.4
                                 [02]: fe80::50ef:eedd:9da4:9cce
Hyper-V Requirements:      A hypervisor has been detected. Features required for Hyper-V will not be displayed.
```
上の状況のもと下記のコマンドを打ったがうまくいかなかった。
```pwsh
PS　C:\Users\dts> wsl --install
```
原因として考えられるものはなにか。

# Part2. Azure App Service
## Intro
Code を簡単にdeployできる様をみよう。
いまから、すでにできあがったdjangoのアプリをgit cloneするが、
意欲のあるひとは、開発の部分に挑戦して、codeをいじくってくれて構わない。

## 要件
たとえば、Team1 member2のひとは、下記のようになるようにせよ。
|| Web App  |App Service Plan | Resource Group |
|-|--------------|--------------|---------------------|
|name| team1mem2wapp |team1asp| team1-wapp-rg     |
|tag| "framework"="django"; "team"=1;| "team"=1; | "team"=1;
```pwsh
$wapp="team1mem2wapp"
$asp="team1asp"
$rg="team1-wapp-rg"
$location="eastasia"
$tagswapp=@{"framework"="django"; "team"=1;}
$tagsasp=@{"team"=1;}
$tagsrg=@{ "team"=1;}
if ($(Get-AzResourceGroupName -Name $rg 2> $null) -eq $null ) {
    New-AzResourceGroup -Name $rg -Location $location -Tag $tagsrg
}
```

## 手順
[Quickstart: Deploy a Python (Django or Flask) web app to Azure App Service](https://learn.microsoft.com/en-us/azure/app-service/quickstart-python?tabs=django)

```pwsh
git clone https://github.com/Azure-Samples/msdocs-python-django-webapp-quickstart
```
#### Question このコードは何をしているか？
webapp 名やAppServicePlan名を勝手に決められていいなら下記のコマンドをこのまま実行すればよい。
非常に簡単である！

```pwsh
az webapp up --runtime PYTHON:3.9 --sku B1 --logs
```
しかし、要件に従うためには、下記の手順を踏んだほうがよい。

```pwsh
New-AzAppServicePlan -ResourceGroupName $rg -Name $asp -Tier "Basic" -NumberofWorkers 2 -WorkerSize "Small"
New-AzWebApp -Name $wapp -Location $location -AppServicePlan $asp -ResourceGroupName $rg
$Properties = @{
  repoUrl = "https://github.com/Azure-Samples/msdocs-python-django-webapp-quickstart";
  branch = "main";
  isManualIntegration = "true";
}
Set-Resource -Properties $Properties -ResourceGroupName $rg -ResourceType Microsoft.Web/sites/sourcecontrols -ResourceName $wapp/web -ApiVersion 2015-08-01 -Force
```