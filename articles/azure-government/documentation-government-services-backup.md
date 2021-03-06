---
title: "Azure Government Backup | Microsoft 문서"
description: "이 문서에서는 Azure Government에서 사용할 수 있는 Azure Backup 기능에 대한 개요를 제공합니다."
services: azure-government
documentationcenter: 
author: markgalioto
manager: carmonm
ms.assetid: a7622135-8790-4be4-a02a-7b9ac8a4996f
ms.service: azure-government
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: azure-government
ms.date: 1/5/2017
ms.author: markgal;
translationtype: Human Translation
ms.sourcegitcommit: fa00142a9e89c5ad2630f688ea9771a1a542c052
ms.openlocfilehash: e5f89f845302ecb890caa50dd8f86503b29f1154
ms.lasthandoff: 01/06/2017


---
# <a name="azure-government-backup"></a>Azure Government Backup

이 문서에서는 Azure Backup 서비스의 개요를 제공하고 Azure Government에서 사용할 수 있는 백업 기능을 나열합니다. Azure Backup은 데이터를 Microsoft 클라우드에 백업(또는 보호)하는 데 사용할 수 있는 Azure 기반 서비스입니다. Azure에서 데이터를 보호하는 것은 클라우드로 백업하는 것뿐만 아니라 클라우드 또는 온-프레미스 환경에 데이터를 복원하는 것을 의미합니다. Azure Backup은 다음과 같은 주요 이점을 제공합니다.

- 자동 저장소 관리
- 무제한 확장
- 여러 저장소 옵션
- 무제한 데이터 전송
- 데이터 암호화.
- 응용 프로그램 일치 백업
- 장기 보존

Azure Backup을 처음 사용하고 사용할 수 있는 기능에 대한 개요를 원하면 [Azure Backup이란](../backup/backup-introduction-to-azure-backup.md) 문서를 참조하세요.

[!INCLUDE [learn-about-backup-deployment models](../../includes/backup-deployment-models.md)]

## <a name="azure-backup-components-available-in-azure-government-backup"></a>Azure Government Backup에서 Azure Backup 구성 요소를 사용할 수 있습니다.

Azure Backup을 사용하여 파일, 폴더, 볼륨, 가상 컴퓨터, 응용 프로그램 및 워크로드를 보호 할 수 있습니다. 보호할 대상과 해당 데이터가 있는 위치에 따라 별도의 Azure Backup 구성 요소를 사용합니다. 다음 섹션에는 각 구성 요소에 대한 Azure Backup 공개 문서에 대한 링크가 있습니다. 섹션은 클래식 포털 또는 Azure Portal로 세분화됩니다. Resource Manager 배포를 사용하려고 계획하고 있는 경우 Azure Portal 버전을 사용하세요.

### <a name="using-windows-server-and-windows-computers-in-azure-portal"></a>Azure Portal에서 Windows Server 및 Windows 컴퓨터 사용

- [Windows Server 및 Windows 클라이언트 컴퓨터 백업](../backup/backup-configure-vault.md)
- [Windows Server 및 Windows 클라이언트 컴퓨터 복원](../backup/backup-azure-restore-windows-server.md)
- [Windows Server 및 Windows 클라이언트 컴퓨터 백업 관리](../backup/backup-azure-manage-windows-server.md)
- [PowerShell을 사용하여 Windows Server 백업](../backup/backup-client-automation.md)

### <a name="using-windows-server-and-windows-computers-in-classic-portal"></a>클래식 포털에서 Windows Server 및 Windows 컴퓨터 사용

- [Windows Server 및 Windows 클라이언트 컴퓨터 백업](../backup/backup-configure-vault-classic.md)
- [Windows Server 및 Windows 클라이언트 컴퓨터 복원](../backup/backup-azure-restore-windows-server-classic.md)
- [Windows Server 및 Windows 클라이언트 컴퓨터 백업 관리](../backup/backup-azure-manage-windows-server-classic.md)
- [Powershell을 사용하여 Windows Server 백업](../backup/backup-client-automation-classic.md)

### <a name="using-virtual-machines-in-azure-portal"></a>Azure Portal에서 가상 컴퓨터 사용

- [가상 컴퓨터 환경 준비](../backup/backup-azure-arm-vms-prepare.md)
- [가상 컴퓨터 설정](../backup/backup-azure-vms-first-look-arm.md)
- [가상 컴퓨터 복원](../backup/backup-azure-arm-restore-vms.md)
- [가상 컴퓨터 관리](../backup/backup-azure-manage-vms.md)
- [PowerShell을 사용하여 가상 컴퓨터 백업](../backup/backup-azure-vms-automation.md)

### <a name="using-virtual-machines-in-classic-portal"></a>클래식 포털에서 가상 컴퓨터 사용

- [가상 컴퓨터 환경 준비](../backup/backup-azure-vms-prepare.md)
- [가상 컴퓨터 설정](../backup/backup-azure-vms-first-look.md)
- [가상 컴퓨터 복원](../backup/backup-azure-restore-vms.md)
- [가상 컴퓨터 관리](../backup/backup-azure-manage-vms-classic.md)
- [PowerShell을 사용하여 가상 컴퓨터 백업](../backup/backup-azure-vms-classic-automation.md)

### <a name="using-system-center-data-protection-manager-in-azure-portal"></a>Azure Portal에서 System Center Data Protection Manager 사용

- [System Center Data Protection Manager 백업](../backup/backup-azure-dpm-introduction.md)

### <a name="using-system-center-data-protection-manager-in-classic-portal"></a>클래식 포털에서 System Center Data Protection Manager 사용

- [System Center Data Protection Manager 백업](../backup/backup-azure-dpm-introduction-classic.md)

### <a name="using-azure-backup-server-in-azure-portal"></a>Azure Portal에서 Azure Backup Server 사용

Azure Backup Server는 System DPM(Center Data Protection Manager)과 비슷한 기능을 하는 Azure Backup 구성 요소지만, Azure Backup Server는 데이터를 테이프로 백업할 수 없다는 한 가지 예외가 있습니다. Azure Backup Server는 단일 콘솔에서 클라우드로의 응용 프로그램 작업 부하(예: Hyper-V VM, Microsoft SQL Server, SharePoint Server, Microsoft Exchange 및 Windows 클라이언트)를 보호 할 수 있습니다. Azure Backup Server는 System Center 라이선스가 필요하지 않습니다.

- [Azure 백업 서버](../backup/backup-azure-microsoft-azure-backup.md)

### <a name="using-azure-backup-server-in-classic-portal"></a>클래식 포털에서 Azure Backup Server 사용

- [Azure 백업 서버](../backup/backup-azure-microsoft-azure-backup-classic.md)


## <a name="next-steps"></a>다음 단계

어디서부터 시작해야 할지 모르는 경우 [클래식 배포 모델을 사용하여 Azure에 Windows 서버 또는 클라이언트 백업](../backup/backup-configure-vault-classic.md) 문서로 시작합니다. 이 자습서는 Windows Server 또는 컴퓨터에서 백업 프로젝트를 설정하는 단계를 안내합니다.

Azure Backup을 사용할 수 있지만 비용을 알고 싶다면 [백업 가격 책정 페이지](http://azure.microsoft.com/pricing/details/backup/)를 참조하세요. 여기에는 유용한 정보를 제공할 수 있는 질문과 대답 목록이 있습니다. 또한 **지역** 드롭다운 메뉴에는 두 개의 Azure Government 지역이 있습니다.

