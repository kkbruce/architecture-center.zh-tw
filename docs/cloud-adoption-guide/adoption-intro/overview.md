---
title: 採用 Azure：基本
description: 說明企業要採用 Azure 所需具備的基本知識
author: petertay
ms.openlocfilehash: e9421b610e4eb07a3ed37bca56e513b0689484ef
ms.sourcegitcommit: 9ba82cf84cee06ccba398ec04c51dab0e1ca8974
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/13/2018
---
# <a name="adopting-azure-foundational"></a>採用 Azure：基本

採用 Azure 是企業追求組織成熟度的第一個階段。 這個階段結束時，組織中的人員即可將簡單的工作負載部署到 Azure。

下列清單包含完成基本採用階段所需執行的工作。 此清單是漸進式的，因此請依序完成每項工作。 如果您先前已完成某項工作，請移至清單中的下一項工作。 

1. 了解 Azure 內部項目：
    - **說明：**[Azure 如何運作？](azure-explainer.md)
2. 了解 Azure 中的企業數位身分識別：
    - **說明：**[什麼是 Azure Active Directory 租用戶？](tenant-explainer.md)
    - **作法：**[取得 Azure Active Directory 租用戶](/azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - **指引：**[Azure AD 租用戶設計指引](tenant.md)
    - **作法：**[將新的使用者新增至 Azure Active Directory](/azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json)    
3. 了解 Azure 中的訂用帳戶：
    - **說明：**[什麼是 Azure 訂用帳戶？](subscription-explainer.md)
    - **指引：**[Azure 訂用帳戶設計](subscription.md)
4. 了解 Azure 中的資源管理： 
    - **說明：**[什麼是 Azure Resource Manager？](resource-manager-explainer.md)
    - **說明：**[什麼是 Azure 資源群組？](resource-group-explainer.md)
    - **說明：**[了解 Azure 中的資源存取](/azure/active-directory/active-directory-understanding-resource-access?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - **作法：**[使用 Azure 入口網站建立 Azure 資源群組](/azure/azure-resource-manager/resource-group-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - **指引：**[Azure 資源群組設計指引](resource-group.md)
    - **指引：**[Azure 資源的命名慣例](/azure/architecture/best-practices/naming-conventions?toc=/azure/architecture/cloud-adoption-guide/toc.json)
5. 部署基本 Azure 架構：
    - 在 [Azure 計算選項概觀](/azure/architecture/guide/technology-choices/compute-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json)中了解不同類型的 Azure 計算選項，例如基礎結構即服務 (IaaS) 和平台即服務 (PaaS)。
    - 現在，您已了解不同類型的 Azure 計算選項，接下來請挑選 PaaS Web 應用程式或 IaaS 虛擬機器，作為您在 Azure 中的第一項資源：
    - PaaS：平台即服務簡介：
        - **作法：**[將基本 Web 應用程式部署到 Azure](/azure/app-service/app-service-web-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **指引：**[將基本 Web 應用程式](/azure/architecture/reference-architectures/app-service-web-app/basic-web-app?toc=/azure/architecture/cloud-adoption-guide/toc.json)部署到 Azure 的實證作法
    - IaaS：虛擬網路簡介：
        - **說明：**[Azure 虛擬網路](/azure/virtual-network/virtual-networks-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **作法：**[使用入口網站將虛擬網路部署到 Azure](/azure/virtual-network/virtual-networks-create-vnet-arm-pportal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - IasS：部署單一虛擬機器 (VM) 工作負載 (Windows 和 Linux)：
        - **作法：**[使用入口網站將 Windows VM 部署到 Azure](/azure/virtual-machines/windows/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **指引：**[在 Azure 上執行 Windows VM 的實證作法](/azure/architecture/reference-architectures/virtual-machines-windows/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **作法：**[使用入口網站將 Linux VM 部署到 Azure](/azure/virtual-machines/linux/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **指引：**[在 Azure 上執行 Linux VM 的實證作法](/azure/architecture/reference-architectures/virtual-machines-linux/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json)
