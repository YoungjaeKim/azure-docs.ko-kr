---
title: "데이터 원본을 검색하는 방법 | Microsoft Docs"
description: "검색 및 필터링을 포함하는 Azure 데이터 카탈로그 및 Azure 데이터 카탈로그 포털의 적중 항목 강조 표시 기능을 사용하여 등록된 데이터 자산을 검색하는 방법을 강조 표시한 방법 문서"
services: data-catalog
documentationcenter: 
author: steelanddata
manager: NA
editor: 
tags: 
ms.assetid: f72ae3a3-6573-4710-89a7-f13555e1968c
ms.service: data-catalog
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: data-catalog
ms.date: 01/23/2017
ms.author: maroche
translationtype: Human Translation
ms.sourcegitcommit: 219dcbfdca145bedb570eb9ef747ee00cc0342eb
ms.openlocfilehash: 0c90aff38ebe33e8c26f9e46db7e61786ea9a7dd


---
# <a name="how-to-discover-data-sources"></a>데이터 원본을 검색하는 방법
## <a name="introduction"></a>소개
**Microsoft Azure 데이터 카탈로그** 는 등록 시스템 및 기업 데이터 원본을 위한 검색 시스템 역할을 하는 완전히 관리되는 클라우드 서비스입니다. 다시 말해서 **Azure 데이터 카탈로그** 는 사람들이 데이터 원본을 검색하고 이해하고 사용하도록 도우면서 조직의 기존 데이터로부터 더 많은 가치를 얻어내도록 돕는 역할을 합니다. **Azure 데이터 카탈로그**를 사용하여 데이터 원본이 등록되면 해당 메타데이터는 서비스로 인덱싱되어 사용자가 쉽게 검색하여 필요한 데이터를 검색할 수 있습니다.

## <a name="searching-and-filtering"></a>검색 및 필터링
**Azure 데이터 카탈로그** 에서 검색은 검색 및 필터링이라는 두 가지 기본 메커니즘을 사용합니다.

기본적으로 검색은 직관적이고 강력하게 설계되었으며 사용자가 제공한 주석을 포함한 검색 용어는 카탈로그의 모든 속성과 일치합니다.

필터링은 검색을 보완하도록 설계되었습니다. 사용자는 전문가, 데이터 원본 유형, 개체 유형 및 태그와 같은 특정 특성을 선택하여 일치하는 데이터 자산만 볼 수 있으며 일치하는 자산으로 검색 결과를 제한할 수도 있습니다.

검색 및 필터링 조합을 사용하여 사용자는 **Azure 데이터 카탈로그** 로 등록된 데이터 원본으로 빠르게 이동하여 필요한 데이터 원본을 검색할 수 있습니다.

## <a name="search-syntax"></a>검색 구문
기본 무료 텍스트 검색은 단순하고 직관적이지만 사용자는 **Azure 데이터 카탈로그**의 검색 구문을 사용하여 검색 결과를 더 잘 제어할 수도 있습니다. **Azure 데이터 카탈로그** 는 다음과 같은 기법을 지원합니다.

| 기법 | 사용 | 예 |
| --- | --- | --- |
| 기본 검색 |기본 검색은 하나 이상의 검색 용어를 사용합니다. 결과는 지정된 하나 이상의 용어와 속성에 대해 일치하는 모든 자산입니다. |판매 데이터 |
| 속성 범위 |지정된 속성에서 일치하는 검색 용어가 있는 데이터 원본만 반환 |name:finance |
| 부울 연산자 |부울 연산을 사용하여 검색 확대 또는 축소 |finance NOT corporate |
| 괄호로 그룹화 |특별히 부울 연산자와 함께 논리적 격리를 수행하도록 쿼리의 일부를 그룹화하기 위해 괄호 사용 |name:finance AND(태그:Q1 또는 태그:Q2) |
| 비교 연산자 |숫자 및 날짜 데이터 유형이 있는 속성에 대한 일치가 아닌 비교 사용 |modifiedTime > "11/05/2014" |

**Azure 데이터 카탈로그** 검색에 대한 자세한 내용은 [https://msdn.microsoft.com/library/azure/mt267594.aspx](https://msdn.microsoft.com/library/azure/mt267594.aspx)를 참조하세요.

## <a name="hit-highlighting"></a>적중 항목 강조 표시
검색 결과를 볼 때 데이터 자산 이름, 설명 및 태그와 같은 지정된 검색 용어와 일치하는 표시된 모든 속성은 지정된 데이터 자산이 지정된 검색에서 반환된 이유를 쉽게 확인할 수 있도록 강조 표시됩니다.

> [!NOTE]
> 사용자는 **Azure 데이터 카탈로그** 포털의 “강조 표시" 스위치를 사용하여 원하는 경우 적중 항목 강조 표시를 해제할 수 있습니다.
>
>

검색 결과를 볼 때 적중 항목 강조 표시가 활성화되어 있더라도 데이터 자산이 포함된 이유가 항상 명확하지는 않을 수도 있습니다. 모든 속성은 기본적으로 검색되므로 열 수준 속성과 일치로 인해 데이터 자산이 반환될 수 있습니다. 또한 여러 사용자가 자신의 태그 및 설명을 사용하여 등록된 데이터 자산에 주석을 지정할 수 있으므로 모든 메타데이터는 검색 결과 목록에 표시되지 않을 수도 있습니다.

기본 타일 보기에서 검색 결과에 표시된 각 타일은 사용자가 일치 수와 해당 위치를 신속하게 보고 원하는 경우 이동할 수 있도록 하는 "검색 용어 일치 항목 보기" 아이콘을 포함합니다.

 ![적중 항목 강조 표시 및 Azure 데이터 카탈로그 포털에서 일치하는 항목 검색](./media/data-catalog-how-to-discover/search-matches.png)

## <a name="summary"></a>요약
데이터 원본을 **Azure 데이터 카탈로그** 에 등록하면 구조적 메타데이터 및 설명이 포함된 메타데이터를 데이터 원본에서 카탈로그 서비스로 복사하여 데이터 원본을 보다 쉽게 검색하고 이해할 수 있게 됩니다. 데이터 원본이 등록되면 사용자는 **Azure 데이터 카탈로그** 포털 내에서 필터링 및 검색을 사용하여 데이터 원본을 검색할 수 있습니다.

## <a name="see-also"></a>참고 항목
* [Azure 데이터 카탈로그 시작](data-catalog-get-started.md) 자습서.



<!--HONumber=Nov16_HO3-->


