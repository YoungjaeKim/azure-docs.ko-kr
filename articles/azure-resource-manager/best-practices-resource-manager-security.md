---
title: "Resource Manager에 대한 보안 고려 사항 | Microsoft Docs"
description: "키 및 암호, 역할 기반 액세스 및 네트워크 보안 그룹을 사용하여 리소스를 보호하기 위해 Azure 리소스 관리자에서 권장되는 방식을 보여줍니다."
services: azure-resource-manager
documentationcenter: 
author: george-moore
manager: georgem
editor: tysonn
ms.assetid: c862e9c7-276b-46bf-bc0a-11868ac11459
ms.service: azure-resource-manager
ms.workload: multiple
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/01/2016
ms.author: georgem;tomfitz
translationtype: Human Translation
ms.sourcegitcommit: e841c21a15c47108cbea356172bffe766003a145
ms.openlocfilehash: 0113b745a1ee719108ca86cd5a05b11c67cc6599


---
# <a name="security-considerations-for-azure-resource-manager"></a>Azure 리소스 관리자에 대한 보안 고려 사항
Azure 리소스 관리자 템플릿에 대한 보안 측면을 살펴보면, 키 및 암호, 역할 기반 액세스 제어 및 네트워크 보안 그룹 등 고려해야 할 여러 영역이 있습니다.

이 항목에서는 Azure 리소스 관리자의 역할 기반 액세스 제어(RBAC)에 대해 잘 알고 있다고 가정합니다. 자세한 내용은 [Azure 역할 기반 액세스 제어](../active-directory/role-based-access-control-configure.md)를 참조하세요.

이 항목은 더 큰 백서의 일부입니다. 전체 문서를 읽으려면 [세계적인 ARM 템플릿 고려 사항 및 입증 사례](http://download.microsoft.com/download/8/E/1/8E1DBEFA-CECE-4DC9-A813-93520A5D7CFE/World Class ARM Templates - Considerations and Proven Practices.pdf)를 다운로드합니다.

## <a name="secrets-and-certificates"></a>암호 및 인증서
Azure 가상 컴퓨터, Azure 리소스 관리자 및 Azure 키 자격 증명 모음이 완전히 통합되어 VM에 배포되는 인증서의 안전한 처리를 위한 지원을 제공합니다.  리소스 관리자와 함께 Azure 키 자격 증명 모음을 활용하여 VM 암호와 인증서를 오케스트레이션하고 저장하는 것이 가장 좋은 방법으로, 다음과 같은 이점이 있습니다.

* 템플릿에만 암호에 대한 URI 참조가 포함되므로 코드, 구성 또는 소스 코드 리포지토리에는 실제 암호가 없게 됩니다. 따라서 GitHub의 harvest-bots 등과 같은 내부 또는 외부 리포지토리에 대한 키 피싱 공격을 차단할 수 있습니다.
* 키 자격 증명 모음에 저장된 암호는 신뢰할 수 있는 작업자의 완전한 RBAC 제어 하에 있습니다.  신뢰할 수 있는 작업자가 퇴사하거나 회사 내 다른 그룹으로 이동하는 경우에는 자격 증명 모음에 생성된 키에 대한 더 이상의 액세스 권한이 부여되지 않습니다.
* 모든 자산에 대한 완벽한 구획화:
  * 키를 배포하는 템플릿
  * 키에 대한 참조를 사용하여 VM을 배포하는 템플릿
  * 자격 증명 모음의 실제 키 자료  
    각 템플릿(및 작업)은 직무를 완전히 분리하기 위해 여러 RBAC 역할 하에 있도록 할 수 있습니다.
* 배포 시 VM으로의 암호 로드는 Microsoft 데이터 센터 범위 내에서 Azure 패브릭 및 키 자격 증명 모음 간의 직접 채널을 통해 수행됩니다.  키가 키 자격 증명 모음에 들어가면 데이터 센터 외부에 있는 신뢰할 수 없는 채널을 통해서는 절대 '드러나지' 않습니다.  
* 키 자격 증명 모음은 항상 지역 단위이므로 암호는 항상 VM에서 지역성(및 독립성)을 갖습니다. 전역 키 자격 증명 모음은 없습니다.

### <a name="separation-of-keys-from-deployments"></a>배포에서 키 분리
가장 좋은 방법은 별도의 템플릿을 유지 관리하는 것입니다.

1. 자격 증명 모음(키 자료가 포함되는 곳) 만들기
2. VM 배포(자격 증명 모음에 포함된 키에 대한 URI 참조)

일반적인 엔터프라이즈 시나리오는 VM 배포를 작성 또는 업데이트할 수 있는 광범위한 개발자/작업자와 더불어, 배포된 워크로드 내의 중요한 비밀에 액세스할 수 있는 소규모의 신뢰할 수 있는 작업자 그룹을 만드는 것입니다.  아래 예제에는 Azure Active Directory에서 현재 인증된 사용자 ID의 컨텍스트에서 새 자격 증명 모음을 생성 및 구성하는 ARM 템플릿이 나와 있습니다.  이 사용자는 이 새 키 자격 증명 모음에 있는 공용 키의 절반에 대해 만들기, 삭제, 나열, 업데이트, 백업, 복원 및 가져오기 작업을 수행할 수 있는 기본 권한을 갖습니다.

이 템플릿의 필드 대부분이 따로 설명이 필요 없는 반면, **enableVaultForDeployment** 설정은 더 자세한 설명이 필요합니다. 자격 증명 모음은 다른 Azure 인프라 구성 요소에 의한 기본적인 고정 액세스 권한이 없습니다. 이 값을 설정하면 Azure 계산 인프라 구성 요소에서 특정하게 명명된 이 자격 증명 모음에 읽기 전용으로 액세스할 수 있습니다. 따라서 더 좋은 방법은 가상 컴퓨터 비밀과 동일한 자격 증명 모음에 중요한 회사 기밀 데이터를 함께 두지 않는 것입니다.

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "keyVaultName": {
                "type": "string",
                "metadata": {
                    "description": "Name of the Vault"
                }
            },
            "location": {
                "type": "string",
                "allowedValues": ["East US", "West US", "West Europe", "East Asia", "South East Asia"],
                "metadata": {
                    "description": "Location of the Vault"
                }
            },
            "tenantId": {
                "type": "string",
                "metadata": {
                    "description": "Tenant Id of the subscription. Get using Get-AzureSubscription cmdlet or Get Subscription API"
                }
            },
            "objectId": {
                "type": "string",
                "metadata": {
                    "description": "Object Id of the AD user. Get using Get-AzureADUser cmdlet"
                }
            },
            "skuName": {
                "type": "string",
                "allowedValues": ["Standard", "Premium"],
                "metadata": {
                    "description": "SKU for the vault"
                }
            },
            "enableVaultForDeployment": {
                "type": "bool",
                "allowedValues": [true, false],
                "metadata": {
                    "description": "Specifies if the vault is enabled for a VM deployment"
                }
            }
        },
        "resources": [{
            "type": "Microsoft.KeyVault/vaults",
            "name": "[parameters('keyVaultName')]",
            "apiVersion": "2014-12-19-preview",
            "location": "[parameters('location')]",
            "properties": {
                "enabledForDeployment": "[parameters('enableVaultForDeployment')]",
                "tenantid": "[parameters('tenantId')]",
                "accessPolicies": [{
                    "tenantId": "[parameters('tenantId')]",
                    "objectId": "[parameters('objectId')]",
                    "permissions": {
                        "secrets": ["all"],
                        "keys": ["all"]
                    }
                }],
                "sku": {
                    "name": "[parameters('skuName')]",
                    "family": "A"
                }
            }
        }]
    }

자격 증명 모음을 만든 후 다음 단계는 새 VM의 배포 템플릿에 있는 해당 자격 증명 모음을 참조하는 것입니다.  위에서 설명한 것처럼 가장 좋은 방법은 VM 배포를 관리하는 서로 다른 개발자/작업자 그룹을 만들어, 이 그룹은 해당 그룹 자격 증명 모음에 저장된 키에 직접 액세스할 수 없도록 하는 것입니다.

아래 템플릿 조각은 높은 순서의 배포 구성으로 통합되며, 매우 중요한 비밀 정보를 안전하게 참조하는 것은 작업자가 직접 제어할 수 없습니다.

    "vaultName": {
        "type": "string",
        "metadata": {
            "description": "Name of Key Vault that has a secret"
        }
    },
    {
        "apiVersion": "2015-05-01-preview",
        "type": "Microsoft.Compute/virtualMachines",
        "name": "[parameters('vmName')]",
        "location": "[parameters('location')]",
        "properties": {
            "osProfile": {
                "secrets": [{
                    "sourceVault": {
                        "id": "[resourceId('vaultrg', 'Microsoft.KeyVault/vaults', 'kayvault')]"
                    },
                    "vaultCertificates": [{
                        "certificateUrl": "[parameters('secretUrlWithVersion')]",
                        "certificateStore": "My"
                    }]
                }]
            }
        }
    }

템플릿을 배포하는 동안 키 자격 증명 모음의 값을 매개 변수로 전달하려면 [배포 중 보안 값 전달](resource-manager-keyvault-parameter.md)을 참조하세요.

## <a name="service-principals-for-cross-subscription-interactions"></a>구독 간 상호 작용에 대한 서비스 주체
서비스 ID는 Active Directory에서 서비스 주체로 표시됩니다. 서비스 주체는 엔터프라이즈 IT 조직, 시스템 통합 업체(SI) 및 클라우드 서비스 공급 업체(CSV)에 대한 주요 시나리오를 사용할 때 중심이 됩니다. 특히, 사용 사례 중 이러한 조직 중 하나가 고객 중 하나의 구독에 대해 상호 작용해야 하는 경우가 있습니다.  

조직은 고객 환경 및 구독에 배포되는 솔루션을 모니터링하는 제품을 제공할 수 있습니다. 이 경우 모니터링 솔루션에서 사용할 수 있도록 고객 계정 내 로그 및 다른 데이터에 대한 액세스 권한을 얻어야 합니다. 기업 IT 조직이나 시스템 통합 업체의 경우 제품이 고객 소유 구독에 상주하는 데이터 분석 플랫폼과 같이 기능을 배포 및 관리하는 제품을 고객에게 제공할 수 있습니다.

이러한 사용 사례에서 조직은 고객 구독의 컨텍스트 내에서 이러한 작업을 수행하는 데 필요한 액세스 권한을 얻기 위해 ID가 있어야 합니다.  

이러한 시나리오는 고객에 대해 고려할 사항이 무엇인지 알려줍니다.

* 보안상의 이유로 액세스 범위를 읽기 전용 액세스 등과 같은 특정 유형의 작업으로 지정해야 할 수 있습니다.
* 배포된 리소스가 유상으로 제공되면 재정적 이유로 액세스에 비슷한 제약 조건이 필요할 수 있습니다.
* 보안상의 이유로 특정 리소스(저장소 계정 또는 환경이나 솔루션이 포함된 리소스 그룹)만으로 액세스 범위를 제한해야 할 수도 있습니다.
* 공급 업체와의 관계가 변할 수 있으므로 고객이 SI 또는 CSV에 대한 액세스를 활성화/비활성화하는 기능을 원할 수도 있습니다.
* 이 계정에 대한 작업에 요금이 청구되면 고객은 요금 청구에 대한 감사 가능성 및 책임에 대한 지원 기능을 원하게 됩니다.
* 규정 준수 관점에서 고객은 자신의 환경 내에서 사용자의 동작을 감사할 수 있는 기능을 원하게 됩니다.

서비스 주체와 RBAC를 결합하여 이러한 요구 사항을 해결하는 데 사용할 수 있습니다.

## <a name="network-security-groups"></a>네트워크 보안 그룹
많은 시나리오에서 가상 네트워크에 있는 하나 이상의 VM 인스턴스에 대해 트래픽을 제어하는 방법을 지정하는 요구 사항을 나타나게 됩니다. 네트워크 보안 그룹(NSG)를 사용하면 ARM 템플릿 배포의 일부로 이 작업을 수행할 수 있습니다.

네트워크 보안 그룹은 구독과 연결된 최상위 개체입니다. NSG에는 VM 인스턴스에 대한 트래픽을 허용하거나 거부할 수 있는 액세스 제어 규칙이 포함됩니다. NSG의 규칙은 언제든지 변경할 수 있으며, 변경 내용은 연결된 모든 인스턴스에 적용됩니다. NSG를 사용하려면 지역(위치)과 연결된 가상 네트워크가 있어야 합니다. NSG는 선호도 그룹과 연결된 가상 네트워크와는 호환되지 않습니다. 지역 가상 네트워크가 없고 끝점에 대한 트래픽을 제어하려는 경우 [네트워크 ACL(액세스 제어 목록) 정보](../virtual-network/virtual-networks-acl.md)를 참조하세요.

NSG를 VM에 연결하거나 가상 네트워크 내에 있는 서브넷에 연결할 수 있습니다. NSG가 VM과 연결된 경우 VM 인스턴스에서 보내고 받는 모든 트래픽에 적용됩니다. NSG가 가상 네트워크 내에 있는 서브넷에 적용된 경우 서브넷의 모든 VM 인스턴스에서 보내고 받는 모든 트래픽에 적용됩니다. VM 또는 서브넷은 오직 1개의 NSG와 연결될 수 있지만 각 NSG에는 규칙이 200개까지 포함될 수 있습니다. 구독당 100개의 NSG가 있을 수 있습니다.

> [!NOTE]
> 끝점 기반 ACL과 네트워크 보안 그룹은 동일한 VM 인스턴스에서 지원되지 않습니다. NSG를 사용하려는데 끝점 ACL이 이미 있는 경우 먼저, 끝점 ACL을 제거합니다. 이 작업을 수행하는 방법에 대한 자세한 내용은 [PowerShell을 사용하여 끝점에 대한 ACL(액세스 제어 목록) 관리](../virtual-network/virtual-networks-acl-powershell.md)를 참조하세요.
> 
> 

### <a name="how-network-security-groups-work"></a>네트워크 보안 그룹의 작동 방법
네트워크 보안 그룹은 끝점 기반 ACL과 다릅니다. 끝점 ACL은 입력 끝점을 통해 노출되는 공용 포트에서만 작동합니다. NSG는 하나 이상의 VM 인스턴스에서 작동하며 VM의 모든 인바운드 및 아웃바운드 트래픽을 제어합니다.

네트워크 보안 그룹에는 *이름*이 있으며 *하위 지역*(지원되는 Azure 위치 중 하나)에 연결되며 설명 레이블이 있습니다. 그리고 인바운드 및 아웃바운드 유형의 규칙이 포함되어 있습니다. 인바운드 규칙은 VM으로 들어오는 패킷에 적용되며, 아웃바운드 규칙은 VM에서 나가는 패킷에 적용됩니다.
규칙은 VM이 있는 서버 컴퓨터에서 적용됩니다. 들어오는 패킷이나 나가 패킷은 허용 규칙이 동일하게 허용되어야 하며, 그렇지 않은 경우 패킷이 삭제됩니다.

규칙은 우선 순위에 따라 처리됩니다. 예를 들어 낮은 우선 순위 번호(예: 100)를 가진 규칙이 더 높은 우선 순위 번호(예: 200)를 가진 규칙보다 먼저 처리됩니다. 일치하는 항목이 발견되면 추가 규칙은 처리되지 않습니다.

규칙에서는 다음을 지정합니다.

* 이름: 규칙의 고유 식별자
* 유형: 인바운드/아웃바운드
* 우선 순위: 100과 4096 사이의 정수(규칙은 낮은 순위에서 높은 순위로 처리됩니다.)
* 원본 IP 주소: 원본 IP 범위의 CIDR
* 원본 포트 범위: 0과 65536 사이의 정수 또는 범위
* 대상 IP 범위: 대상 IP 범위의 CIDR
* 대상 포트 범위: 0과 65536 사이의 정수 또는 범위
* 프로토콜: TCP, UDP 또는 ‘\*’
* 액세스: 허용/거부

### <a name="default-rules"></a>기본 규칙
NSG에는 기본 규칙이 있습니다. 기본 규칙은 삭제할 수 없지만, 가장 낮은 우선순위가 할당되기 때문에 직접 만든 규칙으로 재정의할 수 있습니다. 기본 규칙은 플랫폼에서 권장하는 기본 설정을 설명합니다. 아래 기본 규칙에 설명된 대로, 가상 네트워크에서 시작하고 끝나는 트래픽은 인바운드와 아웃바운드 방향 둘 다에서 허용됩니다.

인터넷에 대한 연결은 아웃바운드 방향에 대해 허용되지만, 기본적으로 인바운드 방향에 대해서는 차단됩니다. 기본 규칙을 사용하면 Azure 부하 분산 장치에서 VM의 상태를 검색할 수 있습니다. NSG의 VM 또는 VM 집합이 부하 분산된 집합에 참여하지 않는 경우 이 규칙을 재정의할 수 있습니다.

기본 규칙은 아래 표에 나와 있습니다.

**인바운드 기본 규칙**

| Name | 우선 순위 | 원본 IP | 원본 포트 | 대상 IP | 대상 포트 | 프로토콜 | Access |
| --- | --- | --- | --- | --- | --- | --- | --- |
| VNET 인바운드 허용 |65000 |VIRTUAL_NETWORK |\* |VIRTUAL_NETWORK |\* |\* |허용 |
| AZURE 부하 분산 장치 인바운드 허용 |65001 |AZURE_LOADBALANCER |\* |\* |\* |\* |허용 |
| 모든 인바운드 거부 |65500 |\* |\* |\* |\* |\* |거부 |

**아웃바운드 기본 규칙**

| Name | 우선 순위 | 원본 IP | 원본 포트 | 대상 IP | 대상 포트 | 프로토콜 | Access |
| --- | --- | --- | --- | --- | --- | --- | --- |
| VNET 아웃바운드 허용 |65000 |VIRTUAL_NETWORK |\* |VIRTUAL_NETWORK |\* |\* |허용 |
| 인터넷 아웃바운드 허용 |65001 |\* |\* |인터넷 |\* |\* |허용 |
| 모든 아웃바운드 거부 |65500 |\* |\* |\* |\* |\* |거부 |

### <a name="special-infrastructure-rules"></a>특별 인프라 규칙
NSG 규칙은 명시적입니다. NSG 규칙에서 지정한 트래픽 이외의 트래픽은 허용되거나 거부되지 않습니다. 그러나 네트워크 보안 그룹 사양과 관계없이 항상 허용되는 트래픽 유형이 두 가지 있습니다. 이러한 프로비전은 인프라를 지원하도록 구성되었습니다.

* **호스트 노드의 가상 IP:**DHCP, DNS 및 상태 모니터링과 같은 기본 인프라 서비스는 168.63.129.16의 가상화된 호스트 IP 주소를 통해 제공됩니다. 이 공용 IP 주소는 Microsoft에 속하며, 이 목적을 위해 모든 지역에서 유일하게 사용되는 가상화된 IP 주소입니다. 이 IP 주소는 VM을 호스트하는 서버 컴퓨터(호스트 노드)의 실제 IP 주소에 매핑됩니다. 호스트 노드는 DHCP 릴레이, DNS 재귀 확인자, 부하 분산 장치 상태 검색 및 컴퓨터 상태 검색에 대한 검색 소스 등의 역할을 합니다. 이 IP 주소에 대한 통신은 공격으로 간주되지 않아야 합니다.
* **라이선싱(키 관리 서비스)**: VM에서 실행되는 Windows 이미지는 사용이 허가되어 있어야 합니다. 사용 허가를 위해 라이선싱 요청이 해당 쿼리를 처리하는 키 관리 서비스 호스트 서버로 전송됩니다. 그리고 이 통신은 항상 아웃바운드 포트 1688을 사용합니다.

### <a name="default-tags"></a>기본 태그
기본 태그는 IP 주소의 범주를 다루기 위해 시스템에서 제공한 식별자입니다. 기본 태그는 사용자 정의 규칙에서 지정할 수 있습니다.

**NSG의 기본 태그**

| 태그 | 설명 |
| --- | --- |
| VIRTUAL_NETWORK |모든 네트워크 주소 공간을 나타냅니다. 여기에는 연결된 모든 온-프레미스 주소 공간(로컬 네트워크)뿐만 아니라 가상 네트워크 주소 공간(Azure의 IP CIDR)도 포함됩니다. 또한 가상 네트워크 간 공간도 포함됩니다. |
| AZURE_LOADBALANCER |Azure 인프라 부하 분산 장치를 나타내며 Azure의 상태 검색이 시작되는 Azure 데이터 센터 IP로 변환됩니다. 이 기본 태그는 NSG와 연결된 VM 또는 VM 집합에서 부하 분산된 집합에 참여하는 경우에만 필요합니다. |
| 인터넷 |가상 네트워크 외부에 있으며 공용 인터넷에서 연결할 수 있는 IP 주소 공간을 표시합니다. 이 범위에는 Azure 소유의 공용 IP 공간도 포함됩니다. |

### <a name="ports-and-port-ranges"></a>포트 및 포트 범위
NSG 규칙은 단일 원본 또는 대상 포트, 혹은 포트 범위에서 지정할 수 있습니다. 이 방식은 FTP와 같은 응용 프로그램을 위해 다양한 범위의 포트를 열려는 경우에 특히 유용합니다. 범위는 순차적이어야 하며, 개별 포트 사양과 혼합할 수 없습니다.
포트의 범위를 지정하려면 하이픈(–) 문자를 사용하세요. 예: **100-500**

### <a name="icmp-traffic"></a>ICMP 트래픽
현재 NSG 규칙을 사용하여 TCP 또는 UDP를 프로토콜로 지정할 수 있지만 ICMP은 지정할 수 있습니다. 그러나 ICMP 트래픽은 기본적으로 가상 네트워크 내에서 모든 포트 및 프로토콜(\*) 간의 트래픽을 지원하는 인바운드 규칙을 통해 가상 네트워크 내에서 허용됩니다.

### <a name="associating-an-nsg-with-a-vm"></a>VM과 NSG 연결
NSG가 VM에 직접 연결된 경우 NSG의 네트워크 액세스 규칙은 VM에 전송되는 모든 트래픽에 직접 적용됩니다. 규칙 변경을 위해 NSG가 업데이트될 때마다 몇 분 내에 업데이트가 트래픽 처리에 반영됩니다. VM에서 NSG를 연결 해제하면 상태가 이전 NSG 상태 즉, NSG를 도입하기 전의 시스템 기본 상태로 돌아갑니다.

### <a name="associating-an-nsg-with-a-subnet"></a>서브넷과 NSG 연결
NSG가 서브넷에 연결된 경우 NSG의 네트워크 액세스 규칙이 서브넷의 모든 VM에 적용됩니다. NSG의 액세스 규칙이 업데이트될 때마다 변경 내용이 몇 분 내에 서브넷에 있는 모든 VM에 적용됩니다.

### <a name="associating-an-nsg-with-a-subnet-and-a-vm"></a>서브넷 및 VM과 NSG 연결
VM과 한 NSG를 연결하고 해당 VM이 있는 서브넷과 다른 NSG를 연결할 수 있습니다. 이 시나리오는 VM에 두 개의 보호 계층을 제공하기 위해 지원됩니다.
인바운드 트래픽의 경우 패킷은 서브넷에 지정된 액세스 규칙을 따른 다음 VM의 규칙을 따릅니다. 아웃바운드인 경우 패킷은 아래와 같이 먼저 VM에 지정된 규칙을 따른 다음 서브넷에 지정된 규칙을 따릅니다.

![서브넷 및 VM에 NSG 연결](./media/best-practices-resource-manager-security/nsg-subnet-vm.png)

NSG가 VM 또는 서브넷과 연결된 경우 네트워크 액세스 제어 규칙은 매우 명시적이 됩니다. 플랫폼은 특정 포트에 대한 트래픽을 허용하는 어떠한 암시적 규칙도 삽입하지 않습니다. 이 경우에 VM의 끝점을 만들면 인터넷에서의 트래픽을 허용하는 규칙도 만들어야 합니다. 이렇게 하지 않으면 외부에서 *VIP:{Port}* 에 액세스할 수 없습니다.

예를 들어 새 VM 및 NSG를 만들 수 있습니다. 그런 다음, NSG와 VM을 연결합니다. VM은 VNET 인바운드 허용 규칙을 통해 가상 네트워크에 있는 다른 VM과 통신할 수 있습니다. 또한 VM은 인터넷 아웃바운드 허용 규칙을 사용하여 인터넷에 아웃바운드 연결할 수도 있습니다. 나중에 포트 80에 끝점을 만들어 VM에서 실행 중인 웹사이트에 대한 트래픽을 수신합니다. 다음 표와 비슷한 규칙을 NSG에 추가할 때까지 인터넷에서 VIP(공용 가상 IP 주소)의 포트 80에 보내는 패킷은 VM에 도달하지 않습니다.

**특정 포트에 대한 트래픽을 허용하는 명시적 규칙**

| 이름 | 우선 순위 | 원본 IP | 원본 포트 | 대상 IP | 대상 포트 | 프로토콜 | Access |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 웹 |100 |인터넷 |* |* |80 |TCP |허용 |

## <a name="user-defined-routes"></a>사용자 정의 경로
Azure에서는 경로 테이블을 사용하여 각 패킷의 대상에 따라 IP 트래픽을 전달하는 방법을 결정합니다. Azure에서는 가상 네트워크 설정에 따라 기본 경로 테이블을 제공하지만 해당 테이블에 사용자 지정 경로를 추가해야 할 수도 있습니다.

경로 테이블에 사용자 지정 항목이 필요한 가장 일반적인 경우는 Azure 환경에서 가상 어플라이언스를 사용하는 경우입니다. 아래 그림에 나와 있는 시나리오를 살펴보겠습니다. 프런트 엔드 서브넷에서 시작되어 중간 계층 이하의 서브넷으로 전달되는 모든 트래픽이 가상 방화벽 어플라이언스를 통과하는지 확인하려는 경우를 가정해 보겠습니다. 어플라이언스를 가상 네트워크에 추가하고 다른 서브넷에 연결하는 것만으로는 이 기능이 제공되지 않습니다.
서브넷에 적용된 라우팅 테이블을 변경하여 패킷이 가상 방화벽 어플라이언스로 전달되도록 해야 합니다.

Azure 가상 네트워크와 인터넷 간의 트래픽을 제어하기 위해 가상 NAT 어플라이언스를 구현한 경우도 마찬가지입니다. 가상 어플라이언스를 사용하려면 인터넷으로 전달되는 모든 트래픽이 가상 어플라이언스로 전달되도록 지정하는 경로를 만들어야 합니다.

### <a name="routing"></a>라우팅
패킷은 물리적 네트워크의 각 노드에 정의된 경로 테이블을 기반으로 TCP/IP 네트워크를 통해 라우팅됩니다. 경로 테이블은 대상 IP 주소에 따라 패킷을 전달할 위치를 결정하는 데 사용되는 개별 경로의 컬렉션입니다. 경로는 다음으로 구성됩니다.

* 주소 접두사. 경로가 적용되는 대상 CIDR(예: 10.1.0.0/16)입니다.
* 다음 홉 유형. 패킷을 전송해야 하는 대상 Azure 홉의 유형입니다. 가능한 값은 다음과 같습니다.
  * 로컬. 로컬 가상 네트워크를 나타냅니다. 예를 들어 10.1.0.0/16 및 10.2.0.0/16의 두 서브넷이 같은 가상 네트워크에 있는 경우 경로 테이블의 각 서브넷에 대한 경로는 다음 홉 값이 로컬로 설정됩니다.
  * VPN 게이트웨이. Azure S2S VPN 게이트웨이를 나타냅니다.
  * 인터넷. Azure 인프라에서 제공하는 기본 인터넷 게이트웨이를 나타냅니다.
  * 가상 어플라이언스. Azure 가상 네트워크에 추가한 가상 어플라이언스를 나타냅니다.
  * NULL. 블랙 홀을 나타냅니다. 블랙 홀로 전달된 패킷은 아무 곳에도 전달되지 않습니다.
* 다음 홉 값. 다음 홉 값에는 패킷을 전달해야 하는 IP 주소가 포함됩니다. 다음 홉 값은 다음 홉 유형이 *가상 어플라이언스*인 경로에서만 허용됩니다. 다음 홉은 원격 서브넷이 아닌 서브넷(네트워크 ID에 따른 가상 어플라이언스의 로컬 인터페이스) 상에 있어야 합니다.

![라우팅](./media/best-practices-resource-manager-security/routing.png)

### <a name="default-routes"></a>기본 경로
가상 네트워크에서 생성된 모든 서브넷은 다음 기본 경로 규칙을 포함하는 경로 테이블과 자동으로 연결됩니다.

* 로컬 Vnet 규칙:이 규칙은 가상 네트워크에 있는 모든 서브넷에 대해 자동으로 생성됩니다. VNet의 VM들이 직접 연결되어 있으며 다음 홉으로 연결되는 매개가 없음을 명시합니다. 이렇게 하면 동일한 서브넷의 VM은(VM이 존재하는 네트워크 ID에 상관없이) 기본 게이트웨이 주소를 요구하지 않고 서로 통신할 수 있게 됩니다.
* 온-프레미스 규칙: 이 규칙은 온-프레미스 주소 범위로 전달되는 모든 트래픽에 적용되며, VPN 게이트웨이를 다음 홉 대상으로 사용합니다.
* 인터넷 규칙: 이 규칙은 공용 인터넷에 유입되는 모든 트래픽을 처리하며 인프라 인터넷 게이트웨이를 인터넷에 유입되는 모든 트래픽에 대한 다음 홉으로 사용합니다.

### <a name="bgp-routes"></a>BGP 경로
이 내용을 작성할 당시에는 Azure Resource Manager의 [네트워크 리소스 공급자](../virtual-network/resource-groups-networking.md)에서 [ExpressRoute](../expressroute/expressroute-introduction.md)가 아직 지원되지 않습니다.  온-프레미스 네트워크와 Azure 간에 ExpressRoute 연결이 있는 경우 BGP를 사용하도록 설정하여 ExpressRoute가 NRP에서 지원되고 나서 온-프레미스 네트워크에서 Azure로 경로를 전파할 수 있습니다. 이러한 BGP 경로는 각 Azure 서브넷의 사용자 정의 경로 및 기본 경로와 동일한 방식으로 사용됩니다. 자세한 내용은 [Express 경로 소개](../expressroute/expressroute-introduction.md)를 참조하세요.

> [!NOTE]
> NRP에서 Express 경로가 지원되는 경우 VPN 게이트웨이를 다음 홉으로 사용하는 서브넷 0.0.0.0/0에 대한 사용자 정의 경로를 만들어 온-프레미스 네트워크를 통한 강제 터널링을 사용하도록 Azure 환경을 구성할 수 있습니다. 그러나 이 구성은 VPN 게이트웨이를 사용하는 경우에만 작동하고 ExpressRoute를 사용하는 경우에는 작동하지 않습니다. ExpressRoute의 경우 강제 터널링은 BGP를 통해 구성됩니다.
> 
> 

### <a name="user-defined-routes"></a>사용자 정의 경로
Azure 환경에서는 위에서 지정한 기본 경로를 볼 수 없으며, 대부분의 환경에는 이러한 경로만 있으면 됩니다.
그러나 다음과 같은 특정한 경우에는 경로 테이블을 만들고 하나 이상의 경로를 추가해야 할 수 있습니다.

* 온-프레미스 네트워크를 통해 인터넷으로 강제 터널링하는 경우
* Azure 환경에서 가상 어플라이언스를 사용하는 경우

위 시나리오에서는 경로 테이블을 만들어 사용자 정의 경로를 추가해야 합니다. 여러 경로 테이블을 만들 수 있으며, 동일한 경로 테이블을 하나 이상의 서브넷에 연결할 수 있습니다. 또한 각 서브넷은 단일 경로 테이블에만 연결할 수 있습니다. 서브넷의 모든 VM 및 클라우드 서비스는 해당 서브넷에 연결된 경로 테이블을 사용합니다.

경로 테이블이 서브넷에 연결될 때까지 서브넷은 기본 경로에 의존합니다. 연결이 설정되면 사용자 정의 경로 및 기본 경로 간에 [LPM(가장 긴 접두사 일치)](https://en.wikipedia.org/wiki/Longest_prefix_match) 을 기반으로 라우팅이 수행됩니다. LPM 일치가 동일한 경로가 두 개 이상 있으면 다음 순서대로 해당 원점에 따라 경로가 선택됩니다.

1. 사용자 정의 경로
2. BGP 경로(ExpressRoute를 사용하는 경우)
3. 기본 경로

> [!NOTE]
> 사용자 정의 경로는 Azure VM 및 클라우드 서비스에만 적용됩니다. 예를 들어 온-프레미스 네트워크와 Azure 간에 방화벽 가상 어플라이언스를 추가하려면 온-프레미스 주소 공간으로 이동하는 모든 트래픽을 가상 어플라이언스로 전달하는 Azure 경로 테이블에 대한 사용자 정의 경로를 만들어야 합니다. 그러나 온-프레미스 주소 공간에서 들어오는 트래픽은 가상 어플라이언스를 우회하여 VPN 게이트웨이 또는 ExpressRoute 회로를 통해 Azure 환경으로 직접 이동합니다.
> 
> 

### <a name="ip-forwarding"></a>IP 전달
위에서 설명한 것처럼 사용자 정의 경로를 만드는 주된 이유 중 하나는 트래픽을 가상 어플라이언스로 전달하기 위한 것입니다. 가상 어플라이언스는 방화벽이나 NAT 장치와 같이 네트워크 트래픽을 처리하는 데 사용되는 응용 프로그램을 실행하는 VM일 뿐입니다.

이 가상 어플라이언스 VM은 주소가 자신으로 지정되지 않은 들어오는 트래픽을 받을 수 있어야 합니다. VM이 다른 대상으로 주소가 지정된 트래픽을 받을 수 있도록 하려면 해당 VM에서 IP 전달을 사용하도록 설정해야 합니다.

## <a name="next-steps"></a>다음 단계
* 조직에서 리소스로 작업하는 데 필요한 정확한 액세스 권한으로 보안 주체를 설정하는 방법을 알아보려면 [Azure 리소스 관리자를 사용하여 서비스 사용자 인증](resource-group-authenticate-service-principal.md)
* 리소스에 대한 액세스를 잠가야 하는 경우 관리 잠금을 사용할 수 있습니다.  [Azure 리소스 관리자를 사용하여 리소스 잠그기](resource-group-lock-resources.md)
* 라우팅 및 IP 전달을 구성하려면 [템플릿을 사용하여 Resource Manager에서 UDR(사용자 정의 경로) 만들기](../virtual-network/virtual-network-create-udr-arm-template.md)
* 역할 기반 액세스 제어에 대한 개요는 [Microsoft Azure 포털에서 역할 기반 액세스 제어](../active-directory/role-based-access-control-configure.md)




<!--HONumber=Nov16_HO3-->


