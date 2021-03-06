---
title: "Azure 마이크로 서비스에서 메트릭 및 배치 설정 지정 | Microsoft Docs"
description: "메트릭, 배치 제약 조건 및 기타 배치 정책을 지정하여 Service Fabric 서비스를 설명합니다."
services: service-fabric
documentationcenter: .net
author: masnider
manager: timlt
editor: 
ms.assetid: 16e135c1-a00a-4c6f-9302-6651a090571a
ms.service: Service-Fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 01/05/2017
ms.author: masnider
translationtype: Human Translation
ms.sourcegitcommit: bb27d279396aa7b670187560cebe2ed074576bad
ms.openlocfilehash: c1df3b77f3fa621a60d6ab73f6dc2c24abbc3366


---
# <a name="configuring-cluster-resource-manager-settings-for-service-fabric-services"></a>Service Fabric 서비스에 대한 클러스터 리소스 관리자 설정 구성
Service Fabric Cluster Resource Manager를 사용하면 개별적으로 명명된 모든 서비스를 제어하는 규칙을 매우 정밀하게 제어할 수 있습니다. 명명된 각 서비스 인스턴스는 클러스터에서 할당되는 방식에 대한 규칙을 지정할 수 있습니다. 명명된 각 서비스는 해당 서비스에 대해 갖는 중요도를 포함하여 보고하려는 메트릭 집합도 정의할 수 있습니다. 서비스 구성은 세 가지 작업으로 구분됩니다.

1. 배치 제약 조건 구성
2. 메트릭 구성
3. 고급 배치 정책 및 기타 규칙 구성(덜 일반적임)

## <a name="placement-constraints"></a>배치 제약 조건
배치 제약 조건은 클러스터에서 서비스가 실제로 실행될 수 있는 노드를 제어하는 데 사용됩니다. 일반적으로 명명된 특정 서비스 인스턴스 또는 특정 노드 형식에서 실행되도록 제약된 지정된 형식의 모든 서비스를 볼 수 있습니다. 즉, 배치 제약 조건은 확장 가능하므로 노드 형식에 따라 속성 집합을 정의한 다음, 서비스가 생성될 때 제약 조건을 사용해서 선택할 수 있습니다. 배치 제약 조건은 서비스가 진행되는 동안 동적으로 업데이트할 수 있으므로 클러스터의 변화에 응답할 수 있습니다. 지정된 노드의 속성을 클러스터에서 동적으로 업데이트할 수도 있습니다. 배치 제약 조건 및 구성 방법에 대한 자세한 내용은 [이 문서](service-fabric-cluster-resource-manager-cluster-description.md#placement-constraints-and-node-properties)

## <a name="metrics"></a>메트릭
메트릭은 특정 명명된 서비스 인스턴스에서 필요한 리소스의 집합입니다. 서비스의 메트릭 구성에는 기본적으로 해당 서비스의 상태 저장 복제본 또는 상태 비저장 인스턴스 각각이 서비하는 리소스의 크기가 포함됩니다. 메트릭에는 조정이 필요한 경우 서비스에 대한 해당 메트릭의 부하 분산 중요도를 나타내는 가중치도 포함됩니다.

## <a name="other-placement-rules"></a>기타 배치 규칙
지리적으로 분산된 클러스터 또는 덜 일반적인 시나리오에서 유용한 몇 가지 배치 규칙이 있습니다. 다른 배치 규칙은 상관 관계 또는 정책을 통해 구성됩니다.

## <a name="next-steps"></a>다음 단계
* 메트릭은 서비스 패브릭 클러스터 리소스 관리자가 클러스터의 소비와 용량을 관리하는 방법입니다. 메트릭 및 구성 방법에 대한 자세한 내용은 [이 문서](service-fabric-cluster-resource-manager-metrics.md)를 확인하세요.
* 선호도는 서비스에 대해 구성할 수 있는 하나의 모드입니다. 일반적이지 않지만 필요한 경우 [여기](service-fabric-cluster-resource-manager-advanced-placement-rules-affinity.md)
* 추가 시나리오를 처리하기 위해 서비스에 구성할 수 있는 다양한 배치 규칙이 있습니다. 이러한 기타 배치 정책은 [여기](service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies.md)
* 처음부터 시작 및 [서비스 패브릭 클러스터 리소스 관리자 소개](service-fabric-cluster-resource-manager-introduction.md)
* 클러스터 Resource Manager가 클러스터의 부하를 관리하고 분산하는 방법을 알아보려면 [부하 분산](service-fabric-cluster-resource-manager-balancing.md)
* Cluster Resource Manager에는 클러스터를 설명하기 위한 많은 옵션이 있습니다. 이에 대해 자세히 알아보려면 [Service Fabric 클러스터 설명](service-fabric-cluster-resource-manager-cluster-description.md)에 대한 문서를 확인하세요.



<!--HONumber=Jan17_HO4-->


