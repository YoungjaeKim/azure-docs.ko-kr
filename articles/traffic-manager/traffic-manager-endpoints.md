---
title: "Azure 트래픽 관리자에서 끝점 관리 | Microsoft Docs"
description: "이 문서는 Azure 트래픽 관리자에서 끝점을 추가, 제거 및 사용하거나 사용하지 않도록 설정하는 데 도움이 됩니다."
services: traffic-manager
documentationcenter: 
author: kumudd
manager: timlt
editor: tysonn
ms.assetid: 038270d1-28ba-4078-9c5d-37fc5d683be6
ms.service: traffic-manager
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/17/2016
ms.author: kumud
translationtype: Human Translation
ms.sourcegitcommit: 8827793d771a2982a3dccb5d5d1674af0cd472ce
ms.openlocfilehash: 22729a7164b8998e92147e418ce96ff9b09896b9

---

# <a name="add-disable-enable-or-delete-endpoints"></a>끝점 추가, 사용 안 함, 사용 또는 삭제

Azure 앱 서비스의 웹앱은 웹 사이트 모드에 관계없이 데이터 센터 내의 웹 사이트에 대해 이미 장애 조치(Failover) 및 라운드 로빈 트래픽 라우팅 기능을 제공합니다. Azure 트래픽 관리자를 통해 다른 데이터 센터의 웹 사이트와 클라우드 서비스에 대해 장애 조치(Failover) 및 라운드 로빈 트래픽 라우팅을 지정할 수 있습니다. 해당 기능을 제공하는 데 필요한 첫 번째 단계는 트래픽 관리자에 클라우드 서비스 또는 웹 사이트 끝점을 추가하는 것입니다.

> [!NOTE]
> Azure 클래식 포털을 사용하여 외부 위치나 트래픽 관리자 프로필을 끝점으로 추가할 수 없습니다. REST API [정의 만들기](http://go.microsoft.com/fwlink/p/?LinkId=400772) 또는 Windows PowerShell [Add-AzureTrafficManagerEndpoint](http://go.microsoft.com/fwlink/p/?LinkId=400774)를 사용해야 합니다.

트래픽 관리자 프로필의 일부인 개별 끝점을 사용하지 않도록 설정할 수도 있습니다. 끝점에는 클라우드 서비스와 웹 사이트가 둘 다 포함됩니다. 끝점을 사용하지 않도록 설정하는 경우 프로필의 일부로 유지되지만 끝점이 없는 것처럼 프로필이 작동합니다. 이 작업은 유지 관리 모드이거나 다시 배포할 예정인 끝점을 일시적으로 제거하는 데 매우 유용합니다. 끝점이 다시 작동하여 실행되면 사용하도록 설정할 수 있습니다.

> [!NOTE]
> 끝점을 사용하지 않도록 설정하는 경우 Azure의 끝점 배포 상태에는 영향을 주지 않습니다. 정상 끝점은 실행 상태로 유지되며 트래픽 관리자에서 사용하지 않도록 설정된 경우에도 트래픽을 수신할 수 있습니다. 또한 한 프로필에서 끝점을 사용하지 않도록 설정해도 다른 프로필의 해당 끝점 상태에는 영향을 주지 않습니다.

## <a name="to-add-a-cloud-service-or-website-endpoint"></a>클라우드 서비스 또는 웹 사이트 끝점을 추가하려면

1. Azure 클래식 포털의 트래픽 관리자 창에서 수정할 끝점 설정이 포함된 트래픽 관리자 프로필을 찾은 다음 프로필 이름 옆에 있는 화살표를 클릭합니다. 프로필에 대한 설정 페이지가 열립니다.
2. 페이지 맨 위에서 **끝점** 을 클릭하여 이미 구성에 포함된 끝점을 확인합니다.
3. 페이지 맨 아래에서 **추가**를 클릭하여 **서비스 끝점 추가** 페이지에 액세스합니다. 기본적으로 페이지의 **서비스 끝점**아래에 클라우드 서비스가 나열됩니다.
4. 클라우드 서비스의 경우 목록에서 클라우드 서비스를 선택하여 이 프로필에 대한 끝점으로 사용하도록 설정합니다. 클라우드 서비스 이름을 지우면 끝점 목록에서 제거됩니다.
5. 웹 사이트의 경우 **서비스 유형** 드롭다운 목록을 클릭하고 **웹앱**을 선택합니다.
6. 목록에서 웹 사이트를 선택하여 이 프로필에 대한 끝점으로 추가합니다. 웹 사이트 이름을 지우면 끝점 목록에서 제거됩니다. Azure 데이터 센터(지역이라고도 함)당 하나의 웹 사이트만 선택할 수 있습니다. 여러 웹 사이트를 호스트하는 데이터 센터에서 웹 사이트를 선택하는 경우 첫 번째 웹 사이트를 선택하면 동일한 데이터 센터의 다른 웹 사이트는 선택할 수 없게 됩니다. 또한 표준 웹 사이트만 나열됩니다.
7. 이 프로필에 대한 끝점을 선택한 후 오른쪽 아래에 있는 확인 표시를 클릭하여 변경 내용을 저장합니다.

> [!NOTE]
> *장애 조치(Failover)* 트래픽 라우팅 방법을 사용하는 경우 끝점을 추가하거나 제거한 후 구성에 사용할 장애 조치(Failover) 순서를 반영하도록 구성 페이지의 장애 조치(Failover) 우선 순위 목록을 조정해야 합니다. 자세한 내용은 [장애 조치(Failover) 트래픽 라우팅 구성](traffic-manager-configure-failover-routing-method.md)을 참조하세요.

## <a name="to-disable-an-endpoint"></a>끝점을 사용하지 않도록 설정하려면

1. Azure 클래식 포털의 트래픽 관리자 창에서 수정할 끝점 설정이 포함된 트래픽 관리자 프로필을 찾은 다음 프로필 이름 옆에 있는 화살표를 클릭합니다. 프로필에 대한 설정 페이지가 열립니다.
2. 페이지 맨 위에서 **끝점** 을 클릭하여 구성에 포함된 끝점을 확인합니다.
3. 사용하지 않도록 설정할 끝점을 클릭한 다음 페이지 맨 아래에서 **사용 안 함** 을 클릭합니다.
4. 트래픽 관리자 도메인 이름에 대해 구성된 DNS TTL(Time-To-Live)에 따라 트래픽이 더 이상 끝점으로 흐르지 않습니다. 트래픽 관리자 프로필의 구성 페이지에서 TTL을 변경할 수 있습니다.

## <a name="to-enable-an-endpoint"></a>끝점을 사용하도록 설정하려면

1. Azure 클래식 포털의 트래픽 관리자 창에서 수정할 끝점 설정이 포함된 트래픽 관리자 프로필을 찾은 다음 프로필 이름 옆에 있는 화살표를 클릭합니다. 프로필에 대한 설정 페이지가 열립니다.
2. 페이지 맨 위에서 **끝점** 을 클릭하여 구성에 포함된 끝점을 확인합니다.
3. 사용하도록 설정할 끝점을 클릭한 다음 페이지 맨 아래에서 **사용** 을 클릭합니다.
4. 프로필에 지정된 대로 트래픽이 다시 서비스로 흐르기 시작합니다.

## <a name="to-delete-a-cloud-service-or-website-endpoint"></a>클라우드 서비스 또는 웹 사이트 끝점을 삭제하려면

1. Azure 클래식 포털의 트래픽 관리자 창에서 수정할 끝점 설정이 포함된 트래픽 관리자 프로필을 찾은 다음 프로필 이름 옆에 있는 화살표를 클릭합니다. 프로필에 대한 설정 페이지가 열립니다.
2. 페이지 맨 위에서 **끝점** 을 클릭하여 이미 구성에 포함된 끝점을 확인합니다.
3. 끝점 페이지에서, 프로필에서 삭제할 끝점의 이름을 클릭합니다.
4. 페이지 맨 아래에서 **삭제**를 클릭합니다.

> [!NOTE]
> Azure 클래식 포털을 사용하여 외부 위치나 트래픽 관리자 프로필을 끝점으로 삭제할 수 없습니다. Windows PowerShell을 사용해야 합니다. 자세한 내용은 [Remove-AzureTrafficManagerEndpoint](https://msdn.microsoft.com/library/dn690251.aspx)를 참조하세요.

## <a name="next-steps"></a>다음 단계

* [장애 조치(Failover) 라우팅 방법 구성](traffic-manager-configure-failover-routing-method.md)
* [라운드 로빈 라우팅 방법 구성](traffic-manager-configure-round-robin-routing-method.md)
* [성능 라우팅 방법 구성](traffic-manager-configure-performance-routing-method.md)
* [트래픽 관리자 성능 저하 상태 문제해결](traffic-manager-troubleshooting-degraded.md)
* [트래픽 관리자 작업(REST API 참조)](http://go.microsoft.com/fwlink/p/?LinkID=313584)




<!--HONumber=Nov16_HO5-->


