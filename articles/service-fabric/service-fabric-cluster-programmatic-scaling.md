---
title: "프로그래밍 방식으로 Azure Service Fabric 크기 조정 | Microsoft Docs"
description: "사용자 지정 트리거에 따라 Azure Service Fabric 클러스터를 프로그래밍 방식으로 규모 확장 또는 규모 감축"
services: service-fabric
documentationcenter: .net
author: mjrousos
manager: jonjung
editor: 
ms.assetid: 
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/10/2017
ms.author: mikerou
translationtype: Human Translation
ms.sourcegitcommit: afe143848fae473d08dd33a3df4ab4ed92b731fa
ms.openlocfilehash: 8d7052fabeb348b4bba744b43d9af78f058175a8
ms.lasthandoff: 03/17/2017


---

# <a name="scale-a-service-fabric-cluster-programmatically"></a>프로그래밍 방식으로 Service Fabric 클러스터의 크기 조정 

Azure에서 Service Fabric 클러스터의 크기를 조정하는 방법에 대한 기본적인 내용은 [클러스터 크기 조정](./service-fabric-cluster-scale-up-down.md)의 문서에 설명되어 있습니다. 이 문서에서는 가상 컴퓨터 크기 집합 위에 Service Fabric 클러스터를 구축하고 수동으로 또는 자동 크기 조정 규칙을 사용하여 규모를 조정하는 방법을 다룹니다. 이 문서에서는 고급 시나리오를 위해 Azure 크기 조정 작업을 프로그래밍 방식으로 조정하는 방법을 살펴봅니다. 

## <a name="reasons-for-programmatic-scaling"></a>프로그래밍 방식으로 크기를 조정하는 이유
여러 시나리오에서 수동으로 또는 자동 크기 조정 규칙을 통해 크기를 조정해도 아무 문제 없습니다. 하지만 일부 시나리오에서는 좋은 방법이 아닐 수 있습니다. 이러한 접근 방식의 잠재적 단점은 다음과 같습니다.

- 수동으로 크기를 조정하려면 로그인하여 크기 조정 작업을 명시적으로 요청해야 합니다. 크기 조정 작업을 자주 또는 예기치 않은 시점에 수행해야 하는 경우에는 이 방법이 좋지 않을 수 있습니다.
- 자동 크기 조정 규칙은 가상 컴퓨터 크기 집합에서 인스턴스를 제거할 때 노드 형식의 내구성 수준이 Silver 또는 Gold가 아닌 한 연결된 Service Fabric 클러스터에서 해당 노드의 지식을 제거하지 않습니다. 자동 크기 조정 규칙은 Service Fabric 수준이 아닌 크기 집합 수준에서 작동하기 때문에 Service Fabric 노드를 정상적으로 종료하지 않아도 자동 크기 조정 규칙이 Service Fabric 노드를 제거할 수 있습니다. 이 강제 노드 제거는 규모 감축 작업 후 'ghost' Service Fabric 노드 상태를 남깁니다. 개인(또는 서비스)은 Service Fabric 클러스터에서 제거된 노드 상태를 주기적으로 정리해야 합니다.
  - 내구성 수준이 Gold 또는 Silver인 노드 유형은 제거된 노드를 자동으로 정리합니다.  
- 자동 크기 조정 규칙이 지원되는 [여러 메트릭](../monitoring-and-diagnostics/insights-autoscale-common-metrics.md)이 있지만 아직은 제한된 집합입니다. 이 집합에 포함되지 않는 일부 메트릭을 기반으로 하는 크기 조정이 필요한 시나리오의 경우 자동 크기 조정 규칙은 좋은 옵션이 아닐 수 있습니다.

이러한 제한에 따라 보다 사용자 지정된 자동 크기 조정 모델을 구현하고자 할 수 있습니다. 

## <a name="scaling-apis"></a>크기 조정 API
응용 프로그램이 가상 컴퓨터 크기 집합 및 Service Fabric 클러스터를 프로그래밍 방식으로 작업할 수 있는 Azure API가 있습니다. 기존 자동 크기 조정 옵션이 시나리오에 적합하지 않은 경우 이러한 API를 사용하여 사용자 지정 크기 조정 논리를 구현할 수 있습니다. 

이 '홈 수행 '수제작' 자동 크기 조정 기능을 구현하는 한 가지 방법은 크기 조정 작업을 관리하는 새로운 상태 비저장 서비스를 Service Fabric 응용 프로그램에 추가하는 것입니다. 서비스의 `RunAsync` 메서드 내에서 트리거 집합은 크기 조정이 필요한지 여부(최대 클러스터 크기 및 크기 조정 휴지 시간 같은 매개 변수 확인 포함)를 확인할 수 있습니다.   

가상 컴퓨터 크기 집합 상호 작용(현재 가상 컴퓨터 인스턴스의 수를 확인하고 수정하는 작업 포함)에 사용되는 API는 fluent [Azure Management Compute 라이브러리](https://www.nuget.org/packages/Microsoft.Azure.Management.Compute.Fluent/1.0.0-beta50)입니다. fluent compute 라이브러리는 가상 컴퓨터 크기 집합과 상호 작용하는 간편한 API를 제공합니다.

Service Fabric 클러스터 자체와 상호 작용하려면 [System.Fabric.FabricClient](/dotnet/api/system.fabric.fabricclient)를 사용하세요.

물론 크기를 조정하기 위해 클러스터에서 크기 조정 코드를 서비스로 실행할 필요는 없습니다. `IAzure` 및 `FabricClient`는 각각 연결된 Azure 리소스에 원격으로 연결하기 때문에 Service Fabric 응용 프로그램 외부에서 실행되는 콘솔 응용 프로그램 또는 Windows 서비스는 손쉽게 크기 조정 서비스가 될 수 있습니다. 

## <a name="credential-management"></a>자격 증명 관리
크기 조정을 처리하는 서비스를 작성할 때 마주치는 한 가지 어려움은 대화형 로그인 없이 서비스가 가상 컴퓨터 크기 집합에 액세스할 수 있어야 한다는 점입니다. 크기 조정 서비스가 자체 Service Fabric 응용 프로그램을 수정하는 경우 Service Fabric 클러스터에 손쉽게 액세스할 수 있지만 자격 증명이 크기 집합에 액세스해야 합니다. 로그인하려면 [Azure CLI 2.0](https://github.com/azure/azure-cli)으로 만든 [서비스 주체](https://github.com/Azure/azure-sdk-for-net/blob/Fluent/AUTH.md#creating-a-service-principal-in-azure)를 사용하면 됩니다.

서비스 주체는 다음 단계에 따라 만들 수 있습니다.

1. 가상 컴퓨터 크기 집합에 액세스할 수 있는 사용자로 Azure CLI(`az login`)에 로그인합니다.
2. `az ad sp create-for-rbac`을 사용하여 서비스 주체를 만듭니다.
    1. 나중에 사용할 수 있도록 appId(다른 곳에서는 'client ID'라고도 함), 이름, 암호, 테넌트를 기록해 둡니다.
    2. 구독 ID도 필요하며 `az account list`를 사용하여 확인할 수 있습니다.

fluent compute 라이브러리는 다음과 같이 자격 증명을 사용하여 로그인할 수 있습니다.

```C#
var credentials = AzureCredentials.FromServicePrincipal(AzureClientId, AzureClientKey, AzureTenantId, AzureEnvironment.AzureGlobalCloud);
IAzure AzureClient = Azure.Authenticate(credentials).WithSubscription(AzureSubscriptionId);

if (AzureClient?.SubscriptionId == AzureSubscriptionId)
{
    ServiceEventSource.Current.ServiceMessage(Context, "Successfully logged into Azure");
}
else
{
    ServiceEventSource.Current.ServiceMessage(Context, "ERROR: Failed to login to Azure");
}
```

로그인되면 `AzureClient.VirtualMachineScaleSets.GetById(ScaleSetId).Capacity`를 통해 크기 집합 인스턴스 수를 쿼리할 수 있습니다.

## <a name="scaling-out"></a>확장
fluent Azure compute SDK를 사용하면 호출 몇 번으로 가상 컴퓨터 크기 집합에 인스턴스를 추가할 수 있습니다.

```C#
var scaleSet = AzureClient?.VirtualMachineScaleSets.GetById(ScaleSetId);
var newCapacity = Math.Min(MaximumNodeCount, NodeCount.Value + 1);
scaleSet.Update().WithCapacity(newCapacity).Apply(); 
``` 

**현재 이 코드가 작동하지 않는 [버그](https://github.com/Azure/azure-sdk-for-net/issues/2716)가 있지만** 픽스가 병합되었기 때문에 게시된 Microsoft.Azure.Management.Compute.Fluent 버전에서는 곧 이 문제가 해결될 것입니다. 이 버그는 fluent compute API를 사용하여 용량 같은 가상 컴퓨터 크기 집합 속성을 수정하면 크기 집합의 Resource Manager 템플릿에서 보호되는 설정이 손실되는 버그입니다. 다른 이유도 있지만 주로 이렇게 사라진 설정 때문에 Service Fabric 서비스가 새로운 가상 컴퓨터 인스턴스에서 올바르게 설정되지 않습니다.

임시 방편으로 크기 조정 서비스에서 PowerShell cmdlet을 호출하여 동일한 변경 내용을 적용하는 방법이 있습니다(단, 이 방법을 사용하려면 PowerShell 도구가 있어야 함).

```C#
using (var psInstance = PowerShell.Create())
{
    psInstance.AddScript($@"
        $clientId = ""{AzureClientId}""
        $clientKey = ConvertTo-SecureString -String ""{AzureClientKey}"" -AsPlainText -Force
        $Credential = New-Object -TypeName ""System.Management.Automation.PSCredential"" -ArgumentList $clientId, $clientKey
        Login-AzureRmAccount -Credential $Credential -ServicePrincipal -TenantId {AzureTenantId}
        
        $vmss = Get-AzureRmVmss -ResourceGroupName {ResourceGroup} -VMScaleSetName {NodeTypeToScale}
        $vmss.sku.capacity = {newCapacity}
        Update-AzureRmVmss -ResourceGroupName {ResourceGroup} -Name {NodeTypeToScale} -VirtualMachineScaleSet $vmss
    ");

    psInstance.Invoke();

    if (psInstance.HadErrors)
    {
        foreach (var error in psInstance.Streams.Error)
        {
            ServiceEventSource.Current.ServiceMessage(Context, $"ERROR adding node: {error.ToString()}");
        }
    }                
}
```

노드를 수동으로 추가하는 경우 크기 집합 인스턴스만 추가하면 새로운 Service Fabric 노드를 시작할 수 있습니다. 크기 집합 템플릿에는 새 인스턴스를 자동으로 Service Fabric 클러스터에 조인하는 확장 기능이 포함되어 있기 때문입니다. 

## <a name="scaling-in"></a>규모 감축

규모 감축은 규모 확장과 비슷합니다. 실제 가상 컴퓨터 크기 집합 변경은 실질적으로 동일합니다. 하지만 앞서 살펴본 것처럼 Service Fabric은 제거된 노드 중에서 내구성 수준이 Gold 또는 Silver인 노드만 자동으로 정리합니다. 따라서 내구성 수준이 Bronze인 규모 감축에서는 제거할 노드를 종료한 후 상태를 제거하도록 Service Fabric 클러스터와 상호 작용이 필요합니다.

종료할 노드를 준비하는 과정에는 제거할 노드(가장 최근에 추가된 노드)를 찾아서 비활성화하는 작업이 포함됩니다. 비-시드 노드의 경우 `NodeInstanceId`를 비교하여 보다 최근의 노드를 찾을 수 있습니다. 

```C#
using (var client = new FabricClient())
{
    var mostRecentLiveNode = (await client.QueryManager.GetNodeListAsync())
        .Where(n => n.NodeType.Equals(NodeTypeToScale, StringComparison.OrdinalIgnoreCase))
        .Where(n => n.NodeStatus == System.Fabric.Query.NodeStatus.Up)
        .OrderByDescending(n => n.NodeInstanceId)
        .FirstOrDefault();
```

*시드* 노드는 더 큰 인스턴스 ID가 먼저 제거되는 규칙을 따르지 않는 경우도 있다는 점에 주의해야 합니다.

제거할 노드를 찾았으면 이전과 동일한 `FabricClient` 인스턴스 및 `IAzure` 인스턴스를 사용하여 노드를 비활성화하고 제거할 수 있습니다.

```C#
var scaleSet = AzureClient?.VirtualMachineScaleSets.GetById(ScaleSetId);

// Remove the node from the Service Fabric cluster
ServiceEventSource.Current.ServiceMessage(Context, $"Disabling node {mostRecentLiveNode.NodeName}");
await client.ClusterManager.DeactivateNodeAsync(mostRecentLiveNode.NodeName, NodeDeactivationIntent.RemoveNode);

// Wait (up to a timeout) for the node to gracefully shutdown
var timeout = TimeSpan.FromMinutes(5);
var waitStart = DateTime.Now;
while ((mostRecentLiveNode.NodeStatus == System.Fabric.Query.NodeStatus.Up || mostRecentLiveNode.NodeStatus == System.Fabric.Query.NodeStatus.Disabling) &&
        DateTime.Now - waitStart < timeout)
{
    mostRecentLiveNode = (await client.QueryManager.GetNodeListAsync()).FirstOrDefault(n => n.NodeName == mostRecentLiveNode.NodeName);
    await Task.Delay(10 * 1000);
}

// Decrement VMSS capacity
var newCapacity = Math.Max(MinimumNodeCount, NodeCount.Value - 1); // Check min count 

scaleSet.Update().WithCapacity(newCapacity).Apply(); 
```

가상 컴퓨터 인스턴스가 제거되면 Service Fabric 노드 상태를 제거할 수 있습니다.

```C#
await client.ClusterManager.RemoveNodeStateAsync(mostRecentLiveNode.NodeName);
```

이전과 마찬가지로 [Azure/azure-sdk-for-net#2716](https://github.com/Azure/azure-sdk-for-net/issues/2716)이 해결되기 전에는 `IVirtualMachineScaleSet.Update()`가 작동하지 않는 문제를 해결해야 합니다.

## <a name="potential-drawbacks"></a>잠재적 단점

위의 코드 조각에서 살펴본 것처럼 사용자 고유의 크기 조정 서비스를 만들면 응용 프로그램 동작을 최대한 원하는 대로 제어하고 사용자 지정할 수 있습니다. 응용 프로그램을 규모 감축 또는 규모 확장하는 시기와 방법을 정교하게 제어해야 하는 시나리오에는 이 방법이 유용할 수 있습니다. 그러나 이 제어 방법은 코드가 복잡해지는 단점이 있습니다. 이 방법을 사용하려면 특수한 크기 조정 코드가 필요합니다.

Service Fabric 크기 조정에 접근하는 방식은 시나리오에 달렸습니다. 크기 조정을 자주 하지 않는다면 노드를 수동으로 추가 또는 제거할 수만 있으면 충분할 것입니다. 좀 더 복잡한 시나리오의 경우 프로그래밍 방식으로 확장하는 기능을 노출하는 자동 크기 조정 규칙과 SDK가 강력한 대안이 될 수 있습니다.

## <a name="next-steps"></a>다음 단계

사용자 고유의 자동 크기 조정 논리를 구현하려면 먼저 다음과 같은 개념과 유용한 API부터 숙지해야 합니다.

- [수동으로 또는 자동 크기 조정 규칙을 사용하여 크기 조정](./service-fabric-cluster-scale-up-down.md)
- [.NET용 Fluent Azure Management 라이브러리](https://github.com/Azure/azure-sdk-for-net/tree/Fluent)(Service Fabric 클러스터의 기본 가상 컴퓨터 크기 집합과 상호 작용하는 데 유용함)
- [System.Fabric.FabricClient](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient)(Service Fabric 클러스터 및 그 노드와 상호 작용하는 데 유용함)
