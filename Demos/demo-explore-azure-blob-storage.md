---
ms.openlocfilehash: 5673f081ce1335404538fd8bb648084293cb873c
ms.sourcegitcommit: 0113753baec606c586c0bdf4c9452052a096c084
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/13/2022
ms.locfileid: "137857517"
---
# <a name="demonstration-explore-azure-blob-storage"></a>演示：了解 Azure Blob 存储

>**注意**：本演示需要一个存储帐户。

## <a name="create-a-container"></a>创建容器

1. 前往 Azure 门户中的存储帐户。
2. 在存储帐户的左侧菜单中滚动到“Blob 服务”部分，然后选择“Blob”。
3. 选择“+ 容器”。 
4. 键入新容器的名称。 容器名称必须小写，必须以字母或数字开头，并且只能包含字母、数字和短划线 (-) 字符。 
5. 设置容器的公共访问权限级别。 默认级别为“专用(禁止匿名访问)”。 
6. 选择“确定”  创建容器。

## <a name="upload-a-block-blob"></a>上传块 Blob

1. 在 Azure 门户中，导航到在上一部分创建的容器。
2. 选择一个容器，显示其包含的 Blob 的列表。 此容器为新容器，因此不会包含任何 Blob。
3. 选择“上传”按钮将 Blob 上传到容器。
4. 展开“高级”部分。
5. 注意“身份验证类型”、“Blob 类型”、“块大小”、以及“上传到文件夹”的能力   。
6. 注意，默认的身份验证类型为 SAS。
4. 浏览本地文件系统，找到一个可作为块 Blob 上传的文件，然后选择“上传”。
5. 以这种方式根据需要上传相应数量的 Blob。 你会看到新 blob 现在在容器中列出。

## <a name="download-a-block-blob"></a>下载块 Blob

可以下载一个块 Blob，让其在浏览器中显示，或者将其保存到本地文件系统。 

1. 导航到在前一部分上传的 Blob 的列表。
2. 右键单击要下载的 Blob，然后选择“下载”。 

>**注意**：你现在可以学习使用 Azure 存储资源管理器来管理 blob 存储。 