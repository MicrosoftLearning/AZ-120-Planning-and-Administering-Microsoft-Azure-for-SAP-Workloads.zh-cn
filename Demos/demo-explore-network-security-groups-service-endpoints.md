---
ms.openlocfilehash: 1d4de1703fcc0710202ba868a8fabaf5df557ff4
ms.sourcegitcommit: 0113753baec606c586c0bdf4c9452052a096c084
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/13/2022
ms.locfileid: "137857567"
---
# <a name="demonstration-explore-network-security-groups-nsgs-and-service-endpoints"></a>演示：了解网络安全组 (NSG) 和服务终结点

## <a name="access-the-nsgs-blade"></a>访问“NSG”边栏选项卡

1. 访问 Azure 门户。
2. 搜索并访问“网络安全组”边栏选项卡。
3. 如果你正在使用虚拟机，那么可能已经拥有 NSG 了。 请注意列表筛选功能。

## <a name="add-a-new-nsg"></a>添加新 NSG

1. 添加网络安全组。

    + 名称：选择一个唯一名称
    + 订阅：选择自己的订阅
    + 资源组：创建新资源组，或选择现有的资源组
    + 位置：你的选择

2. 单击“创建”。

3. 等待部署新 NSG。

## <a name="explore-inbound-and-outbound-rules"></a>探索入站和出站规则

1. 选择新 NSG。
2. 注意，NSG 可与子网和网络接口相关联（规则上方的摘要信息）。
3. 注意三个入站 NSG 规则和三个出站 NSG 规则。
4. 在“设置”下，选择“入站安全规则”。
5. 注意，可使用“默认规则”隐藏默认规则。
6. 添加新的入站安全规则。
7. 单击“基本”以更改为“高级”模式。
8. 使用“服务”下拉列表查看可用的预定义服务。
9. 选择服务（如 HTTPS）时，端口范围（如 443）会自动填充。 这简化了规则的配置。
10. 使用“优先级”标签旁边的“信息”图标可了解如何配置优先级。
11. 退出规则，不进行任何更改。 
12. 如果有时间，请查看如何添加出站安全规则。