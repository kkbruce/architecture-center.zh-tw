---
title: "將 Azure 雲端服務應用程式移轉至 Azure Service Fabric"
description: "如何將應用程式從 Azure 雲端服務移轉至 Azure Service Fabric。"
author: MikeWasson
ms.date: 04/27/2017
ms.openlocfilehash: 22b6cca0d4714dd4cde0fd7449340d6e1f45e65b
ms.sourcegitcommit: fbcf9a1c25db13b2627a8a58bbc985cd01ea668d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/16/2017
---
# <a name="migrate-an-azure-cloud-services-application-to-azure-service-fabric"></a><span data-ttu-id="1d8ab-103">將 Azure 雲端服務應用程式移轉至 Azure Service Fabric</span><span class="sxs-lookup"><span data-stu-id="1d8ab-103">Migrate an Azure Cloud Services application to Azure Service Fabric</span></span> 

<span data-ttu-id="1d8ab-104">[![GitHub](../_images/github.png) 範例程式碼][sample-code]</span><span class="sxs-lookup"><span data-stu-id="1d8ab-104">[![GitHub](../_images/github.png) Sample code][sample-code]</span></span>

<span data-ttu-id="1d8ab-105">本文說明如何將應用程式從 Azure 雲端服務移轉至 Azure Service Fabric。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-105">This article describes migrating an application from Azure Cloud Services to Azure Service Fabric.</span></span> <span data-ttu-id="1d8ab-106">內容重點放在架構決策和建議的做法。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-106">It focuses on architectural decisions and recommended practices.</span></span> 

<span data-ttu-id="1d8ab-107">在此專案中，我們從稱為「Surveys」的雲端服務應用程式來開始著手，並已將其移植到 Service Fabric。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-107">For this project, we started with a Cloud Services application called Surveys and ported it to Service Fabric.</span></span> <span data-ttu-id="1d8ab-108">我們的目標是要以最少的變動移轉應用程式。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-108">The goal was to migrate the application with as few changes as possible.</span></span> <span data-ttu-id="1d8ab-109">在之後的文章中，我們會採用微服務架構，讓 Service Fabric 的應用程式獲得最佳效能。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-109">In a later article, we will optimize the application for Service Fabric by adopting a microservices architecture.</span></span>

<span data-ttu-id="1d8ab-110">在閱讀本文之前，先大致了解 Service Fabric 和微服務架構的基本概念，會對您有所幫助。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-110">Before reading this article, it will be useful to understand the basics of Service Fabric and microservices architectures in general.</span></span> <span data-ttu-id="1d8ab-111">請參閱下列文章：</span><span class="sxs-lookup"><span data-stu-id="1d8ab-111">See the following articles:</span></span>

- <span data-ttu-id="1d8ab-112">[Azure Service Fabric 概觀][sf-overview]</span><span class="sxs-lookup"><span data-stu-id="1d8ab-112">[Overview of Azure Service Fabric][sf-overview]</span></span>
- <span data-ttu-id="1d8ab-113">[為何要用微服務方式建置應用程式？][sf-why-microservices]</span><span class="sxs-lookup"><span data-stu-id="1d8ab-113">[Why a microservices approach to building applications?][sf-why-microservices]</span></span>


## <a name="about-the-surveys-application"></a><span data-ttu-id="1d8ab-114">關於 Surveys 應用程式</span><span class="sxs-lookup"><span data-stu-id="1d8ab-114">About the Surveys application</span></span>

<span data-ttu-id="1d8ab-115">在 2012 年時，Patterns & Practices 群組針對《[Developing Multi-tenant Applications for the Cloud][tailspin-book]》一書建立了名為「Surveys」的應用程式。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-115">In 2012, the patterns & practices group created an application called Surveys, for a book called [Developing Multi-tenant Applications for the Cloud][tailspin-book].</span></span> <span data-ttu-id="1d8ab-116">本書說明 Tailspin 這家虛構公司所設計並實作的 Surveys 應用程式。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-116">The book describes a fictitious company named Tailspin that designs and implements the Surveys application.</span></span>

<span data-ttu-id="1d8ab-117">Surveys 是多租用戶應用程式，可讓客戶建立問卷。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-117">Surveys is a multitenant application that allows customers to create surveys.</span></span> <span data-ttu-id="1d8ab-118">客戶註冊該應用程式之後，客戶組織的成員即可建立和發佈問卷，並收集結果以進行分析。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-118">After a customer signs up for the application,  members of the customer's organization can create and publish surveys, and collect the results for analysis.</span></span> <span data-ttu-id="1d8ab-119">該應用程式並有一個公用網站可供眾人進入問卷。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-119">The application includes a public website where people can take a survey.</span></span> <span data-ttu-id="1d8ab-120">您可以在[這裡][tailspin-scenario]深入了解原始的 Tailspin 案例。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-120">Read more about the original Tailspin scenario [here][tailspin-scenario].</span></span>

<span data-ttu-id="1d8ab-121">現在，Tailspin 想要使用 Azure 上所執行的 Service Fabric，將 Surveys 應用程式移往微服務架構。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-121">Now Tailspin wants to move the Surveys application to a microservices architecture, using Service Fabric running on Azure.</span></span> <span data-ttu-id="1d8ab-122">由於該應用程式已部署為雲端服務應用程式，Tailspin 採用了多階段方式：</span><span class="sxs-lookup"><span data-stu-id="1d8ab-122">Because the application is already deployed as a Cloud Services application, Tailspin adopts a multi-phase approach:</span></span>

1.  <span data-ttu-id="1d8ab-123">將雲端服務植入 Service Fabric，同時將應用程式的變更降至最低。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-123">Port the cloud services to Service Fabric, while minimizing changes to the application.</span></span>
2.  <span data-ttu-id="1d8ab-124">將 Service Fabric 的應用程式移往微服務架構，讓其獲得最佳效能。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-124">Optimize the application for Service Fabric, by moving to a microservices architecture.</span></span>

<span data-ttu-id="1d8ab-125">本文會說明第一個階段。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-125">This article describes the first phase.</span></span> <span data-ttu-id="1d8ab-126">之後的文章會說明第二個階段。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-126">A later article will describe the second phase.</span></span> <span data-ttu-id="1d8ab-127">在真實世界的專案中，這兩個階段可能會重疊。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-127">In a real-world project, it's likely that both stages would overlap.</span></span> <span data-ttu-id="1d8ab-128">在移植到 Service Fabric 的同時，您也會開始重新設計應用程式的架構，使其能夠進入微服務。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-128">While porting to Service Fabric, you would also start to re-architect the application into micro-services.</span></span> <span data-ttu-id="1d8ab-129">之後，您可能會進一步改善架構，或許是將較大型的服務分割成眾多較小的服務。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-129">Later you might refine the architecture further, perhaps dividing coarse-grained services into smaller services.</span></span>  

<span data-ttu-id="1d8ab-130">該應用程式的程式碼可於 [GitHub][sample-code] 取得。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-130">The application code is available on [GitHub][sample-code].</span></span> <span data-ttu-id="1d8ab-131">此存放庫內含雲端服務應用程式與 Service Fabric 版本。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-131">This repo contains both the Cloud Services application and the Service Fabric version.</span></span> 

> <span data-ttu-id="1d8ab-132">雲端服務是《Developing Multi-tenant Applications》一書中所提供之原始應用程式的更新版本。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-132">The cloud service is an updated version of the original application from the *Developing Multi-tenant Applications* book.</span></span>

## <a name="why-microservices"></a><span data-ttu-id="1d8ab-133">使用微服務的理由？</span><span class="sxs-lookup"><span data-stu-id="1d8ab-133">Why Microservices?</span></span>

<span data-ttu-id="1d8ab-134">本文不會深入討論微服務，但 Tailspin 希望藉由移往微服務架構而能獲得的優勢如下：</span><span class="sxs-lookup"><span data-stu-id="1d8ab-134">An in-depth discussion of microservices is beyond scope of this article, but here are some of the benefits that Tailspin hopes to get by moving to a microservices architecture:</span></span>

- <span data-ttu-id="1d8ab-135">**應用程式升級**。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-135">**Application upgrades**.</span></span> <span data-ttu-id="1d8ab-136">服務可以獨立地部署，以便能夠採取累加方式升級應用程式。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-136">Services can be deployed independently, so you can take an incremental approach to upgrading an application.</span></span>
- <span data-ttu-id="1d8ab-137">**復原功能和錯誤隔離**。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-137">**Resiliency and fault isolation**.</span></span> <span data-ttu-id="1d8ab-138">如果服務失敗，其他服務會繼續執行。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-138">If a service fails, other services continue to run.</span></span>
- <span data-ttu-id="1d8ab-139">**延展性**。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-139">**Scalability**.</span></span> <span data-ttu-id="1d8ab-140">服務可以獨立調整。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-140">Services can be scaled independently.</span></span>
- <span data-ttu-id="1d8ab-141">**彈性**。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-141">**Flexibility**.</span></span> <span data-ttu-id="1d8ab-142">服務是圍繞商務案例 (而不是技術堆疊) 來設計的，因此可讓您更輕鬆地將服務移轉至新的技術、架構或資料存放區。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-142">Services are designed around business scenarios, not technology stacks, making it easier to migrate services to new technologies, frameworks, or data stores.</span></span>
- <span data-ttu-id="1d8ab-143">**敏捷式開發**。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-143">**Agile development**.</span></span> <span data-ttu-id="1d8ab-144">相較於單體式應用程式，個別服務的程式碼較少，因此其程式碼基底較容易理解、知道原由和進行測試。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-144">Individual services have less code than a monolithic application, making the code base easier to understand, reason about, and test.</span></span>
- <span data-ttu-id="1d8ab-145">**小型焦點小組**。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-145">**Small, focused teams**.</span></span> <span data-ttu-id="1d8ab-146">應用程式分成了許多小型服務，因此可由小型的聚焦小組建置各個服務。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-146">Because the application is broken down into many small services, each service can be built by a small focused team.</span></span>

## <a name="why-service-fabric"></a><span data-ttu-id="1d8ab-147">使用 Service Fabric 的理由？</span><span class="sxs-lookup"><span data-stu-id="1d8ab-147">Why Service Fabric?</span></span>
      
<span data-ttu-id="1d8ab-148">Service Fabric 適合用於微服務架構，因為分散式系統所需的大部分功能都會內建在 Service Fabric 中，包括：</span><span class="sxs-lookup"><span data-stu-id="1d8ab-148">Service Fabric is a good fit for a microservices architecture, because most of the features needed in a distributed system are built into Service Fabric, including:</span></span>

- <span data-ttu-id="1d8ab-149">**叢集管理**。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-149">**Cluster management**.</span></span> <span data-ttu-id="1d8ab-150">Service Fabric 會自動處理節點的容錯移轉、健康情況監視和其他叢集管理功能。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-150">Service Fabric automatically handles node failover, health monitoring, and other cluster management functions.</span></span>
- <span data-ttu-id="1d8ab-151">**水平調整**。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-151">**Horizontal scaling**.</span></span> <span data-ttu-id="1d8ab-152">當您將節點新增至 Service Fabric 叢集時，應用程式會自動調整，因為服務會分散到這些新節點。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-152">When you add nodes to a Service Fabric cluster, the application automatically scales, as services are distributed across the new nodes.</span></span>
- <span data-ttu-id="1d8ab-153">**服務探索**。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-153">**Service discovery**.</span></span> <span data-ttu-id="1d8ab-154">Service Fabric 會提供可解析具名服務端點的探索服務。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-154">Service Fabric provides a discovery service that can resolve the endpoint for a named service.</span></span>
- <span data-ttu-id="1d8ab-155">**無狀態服務和具狀態服務**。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-155">**Stateless and stateful services**.</span></span> <span data-ttu-id="1d8ab-156">具狀態服務會使用[可靠集合][sf-reliable-collections]，這些集合不僅可取代快取或佇列，還能加以分割。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-156">Stateful services use [reliable collections][sf-reliable-collections], which can take the place of a cache or queue, and can be partitioned.</span></span>
- <span data-ttu-id="1d8ab-157">**應用程式生命週期管理**。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-157">**Application lifecycle management**.</span></span> <span data-ttu-id="1d8ab-158">服務可以獨立升級，應用程式不必停機。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-158">Services can be upgraded independently and without application downtime.</span></span>
- <span data-ttu-id="1d8ab-159">跨機器叢集的**服務協調流程**。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-159">**Service orchestration** across a cluster of machines.</span></span>
- <span data-ttu-id="1d8ab-160">**更高的密度**以獲得最佳的資源耗用量。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-160">**Higher density** for optimizing resource consumption.</span></span> <span data-ttu-id="1d8ab-161">單一節點可以裝載多個服務。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-161">A single node can host multiple services.</span></span>

<span data-ttu-id="1d8ab-162">Service Fabric 可供各種 Microsoft 服務使用，包括 Azure SQL Database、Cosmos DB、Azure 事件中樞等等，因此是經過證實可供建置分散式雲端應用程式的平台。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-162">Service Fabric is used by various Microsoft services, including Azure SQL Database, Cosmos DB, Azure Event Hubs, and others, making it a proven platform for building distributed cloud applications.</span></span> 

## <a name="comparing-cloud-services-with-service-fabric"></a><span data-ttu-id="1d8ab-163">比較雲端服務與 Service Fabric</span><span class="sxs-lookup"><span data-stu-id="1d8ab-163">Comparing Cloud Services with Service Fabric</span></span>

<span data-ttu-id="1d8ab-164">下表摘要列出雲端服務和 Service Fabric 應用程式的一些重要差異。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-164">The following table summarizes some of the important differences between Cloud Services and Service Fabric applications.</span></span> <span data-ttu-id="1d8ab-165">如需更深入的套論，請參閱[移轉應用程式之前，先了解雲端服務與 Service Fabric 之間的差異][sf-compare-cloud-services]。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-165">For a more in-depth discussion, see [Learn about the differences between Cloud Services and Service Fabric before migrating applications][sf-compare-cloud-services].</span></span>

|        | <span data-ttu-id="1d8ab-166">雲端服務</span><span class="sxs-lookup"><span data-stu-id="1d8ab-166">Cloud Services</span></span> | <span data-ttu-id="1d8ab-167">Service Fabric</span><span class="sxs-lookup"><span data-stu-id="1d8ab-167">Service Fabric</span></span> |
|--------|---------------|----------------|
| <span data-ttu-id="1d8ab-168">應用程式組合</span><span class="sxs-lookup"><span data-stu-id="1d8ab-168">Application composition</span></span> | <span data-ttu-id="1d8ab-169">角色</span><span class="sxs-lookup"><span data-stu-id="1d8ab-169">Roles</span></span>| <span data-ttu-id="1d8ab-170">服務</span><span class="sxs-lookup"><span data-stu-id="1d8ab-170">Services</span></span> |
| <span data-ttu-id="1d8ab-171">密度</span><span class="sxs-lookup"><span data-stu-id="1d8ab-171">Density</span></span> |<span data-ttu-id="1d8ab-172">每個 VM 一個角色執行個體</span><span class="sxs-lookup"><span data-stu-id="1d8ab-172">One role instance per VM</span></span> | <span data-ttu-id="1d8ab-173">單一節點可有多個服務</span><span class="sxs-lookup"><span data-stu-id="1d8ab-173">Multiple services in a single node</span></span> |
| <span data-ttu-id="1d8ab-174">最小節點數目</span><span class="sxs-lookup"><span data-stu-id="1d8ab-174">Minimum number of nodes</span></span> | <span data-ttu-id="1d8ab-175">每一角色 2 個</span><span class="sxs-lookup"><span data-stu-id="1d8ab-175">2 per role</span></span> | <span data-ttu-id="1d8ab-176">每一叢集 5 個，用於生產部署</span><span class="sxs-lookup"><span data-stu-id="1d8ab-176">5 per cluster, for production deployments</span></span> |
| <span data-ttu-id="1d8ab-177">狀態管理</span><span class="sxs-lookup"><span data-stu-id="1d8ab-177">State management</span></span> | <span data-ttu-id="1d8ab-178">無狀態</span><span class="sxs-lookup"><span data-stu-id="1d8ab-178">Stateless</span></span> | <span data-ttu-id="1d8ab-179">無狀態或具狀態*</span><span class="sxs-lookup"><span data-stu-id="1d8ab-179">Stateless or stateful*</span></span> |
| <span data-ttu-id="1d8ab-180">裝載</span><span class="sxs-lookup"><span data-stu-id="1d8ab-180">Hosting</span></span> | <span data-ttu-id="1d8ab-181">Azure</span><span class="sxs-lookup"><span data-stu-id="1d8ab-181">Azure</span></span> | <span data-ttu-id="1d8ab-182">雲端或內部部署</span><span class="sxs-lookup"><span data-stu-id="1d8ab-182">Cloud or on-premises</span></span> |
| <span data-ttu-id="1d8ab-183">Web 裝載</span><span class="sxs-lookup"><span data-stu-id="1d8ab-183">Web hosting</span></span> | <span data-ttu-id="1d8ab-184">IIS**</span><span class="sxs-lookup"><span data-stu-id="1d8ab-184">IIS**</span></span> | <span data-ttu-id="1d8ab-185">自我裝載</span><span class="sxs-lookup"><span data-stu-id="1d8ab-185">Self-hosting</span></span> |
| <span data-ttu-id="1d8ab-186">部署模型</span><span class="sxs-lookup"><span data-stu-id="1d8ab-186">Deployment model</span></span> | <span data-ttu-id="1d8ab-187">[傳統部署模型][azure-deployment-models]</span><span class="sxs-lookup"><span data-stu-id="1d8ab-187">[Classic deployment model][azure-deployment-models]</span></span> | <span data-ttu-id="1d8ab-188">[Resource Manager][azure-deployment-models]</span><span class="sxs-lookup"><span data-stu-id="1d8ab-188">[Resource Manager][azure-deployment-models]</span></span>  |
| <span data-ttu-id="1d8ab-189">包裝中</span><span class="sxs-lookup"><span data-stu-id="1d8ab-189">Packaging</span></span> | <span data-ttu-id="1d8ab-190">雲端服務套件檔案 (.cspkg)</span><span class="sxs-lookup"><span data-stu-id="1d8ab-190">Cloud service package files (.cspkg)</span></span> | <span data-ttu-id="1d8ab-191">應用程式和服務套件</span><span class="sxs-lookup"><span data-stu-id="1d8ab-191">Application and service packages</span></span> |
| <span data-ttu-id="1d8ab-192">應用程式更新</span><span class="sxs-lookup"><span data-stu-id="1d8ab-192">Application update</span></span> | <span data-ttu-id="1d8ab-193">VIP 交換或輪流更新</span><span class="sxs-lookup"><span data-stu-id="1d8ab-193">VIP swap or rolling update</span></span> | <span data-ttu-id="1d8ab-194">輪流更新</span><span class="sxs-lookup"><span data-stu-id="1d8ab-194">Rolling update</span></span> |
| <span data-ttu-id="1d8ab-195">自動調整</span><span class="sxs-lookup"><span data-stu-id="1d8ab-195">Auto-scaling</span></span> | <span data-ttu-id="1d8ab-196">[內建服務][cloud-service-autoscale]</span><span class="sxs-lookup"><span data-stu-id="1d8ab-196">[Built-in service][cloud-service-autoscale]</span></span> | <span data-ttu-id="1d8ab-197">具有 VM 擴展集可供自動相應放大</span><span class="sxs-lookup"><span data-stu-id="1d8ab-197">VM Scale Sets for auto scale out</span></span> |
| <span data-ttu-id="1d8ab-198">Debugging</span><span class="sxs-lookup"><span data-stu-id="1d8ab-198">Debugging</span></span> | <span data-ttu-id="1d8ab-199">本機模擬器</span><span class="sxs-lookup"><span data-stu-id="1d8ab-199">Local emulator</span></span> | <span data-ttu-id="1d8ab-200">本機叢集</span><span class="sxs-lookup"><span data-stu-id="1d8ab-200">Local cluster</span></span> |


<span data-ttu-id="1d8ab-201">\*具狀態服務會使用[可靠集合][sf-reliable-collections]來跨複本儲存狀態，讓所有讀取皆是在叢集節點的本機進行。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-201">\* Stateful services use [reliable collections][sf-reliable-collections] to store state across replicas, so that all reads are local to the nodes in the cluster.</span></span> <span data-ttu-id="1d8ab-202">寫入會複寫到各個節點，以提供可靠性。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-202">Writes are replicated across nodes for reliability.</span></span> <span data-ttu-id="1d8ab-203">無狀態服務可以使用資料庫或其他外部儲存體而具有外部狀態。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-203">Stateless services can have external state, using a database or other external storage.</span></span>

<span data-ttu-id="1d8ab-204">** 背景工作角色也可以使用 OWIN 自我裝載 ASP.NET Web API。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-204">** Worker roles can also self-host ASP.NET Web API using OWIN.</span></span>

## <a name="the-surveys-application-on-cloud-services"></a><span data-ttu-id="1d8ab-205">雲端服務上的 Surveys 應用程式</span><span class="sxs-lookup"><span data-stu-id="1d8ab-205">The Surveys application on Cloud Services</span></span>

<span data-ttu-id="1d8ab-206">下圖顯示雲端服務上所執行之 Surveys 應用程式的架構。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-206">The following diagram shows the architecture of the Surveys application running on Cloud Services.</span></span> 

![](./images/tailspin01.png)

<span data-ttu-id="1d8ab-207">此應用程式是由兩個 Web 角色和一個背景工作角色所組成。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-207">The application consists of two web roles and a worker role.</span></span>

- <span data-ttu-id="1d8ab-208">**Tailspin.Web** Web 角色可裝載 ASP.NET 網站，供 Tailspin 客戶用來建立和管理問卷。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-208">The **Tailspin.Web** web role hosts an ASP.NET website that Tailspin customers use to create and manage surveys.</span></span> <span data-ttu-id="1d8ab-209">客戶也會使用此網站來註冊應用程式，並管理其訂用帳戶。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-209">Customers also use this website to sign up for the application and manage their subscriptions.</span></span> <span data-ttu-id="1d8ab-210">最後，Tailspin 管理員可用它來查看租用戶清單，並管理租用戶資料。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-210">Finally, Tailspin administrators can use it to see the list of tenants and manage tenant data.</span></span> 

- <span data-ttu-id="1d8ab-211">**Tailspin.Web.Survey.Public** Web 角色可裝載 ASP.NET 網站，供眾人進入 Tailspin 客戶所發佈的問卷。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-211">The **Tailspin.Web.Survey.Public** web role hosts an ASP.NET website where people can take the surveys that Tailspin customers publish.</span></span> 

- <span data-ttu-id="1d8ab-212">**Tailspin.Workers.Survey** 背景工作角色負責進行幕後處理。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-212">The **Tailspin.Workers.Survey** worker role does background processing.</span></span> <span data-ttu-id="1d8ab-213">Web 角色會將工作項目放入佇列，背景工作角色則負責處理這些項目。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-213">The web roles put work items onto a queue, and the worker role processes the items.</span></span> <span data-ttu-id="1d8ab-214">系統會定義兩個背景工作：將問卷的答覆匯出到 Azure SQL Database，並計算問卷答覆的統計資料。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-214">Two background tasks are defined: Exporting survey answers to Azure SQL Database, and calculating statistics for survey answers.</span></span>

<span data-ttu-id="1d8ab-215">除了雲端服務外，Surveys 應用程式還會使用一些其他的 Azure 服務：</span><span class="sxs-lookup"><span data-stu-id="1d8ab-215">In addition to Cloud Services, the Surveys application uses some other Azure services:</span></span>

- <span data-ttu-id="1d8ab-216">**Azure 儲存體**，以存放問卷、問卷答覆和租用戶資訊。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-216">**Azure Storage** to store surveys, surveys answers, and tenant information.</span></span>

- <span data-ttu-id="1d8ab-217">**Azure Redis 快取**，以快取 Azure 儲存體中儲存的部分資料，來加快讀取存取的速度。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-217">**Azure Redis Cache** to cache some of the data that is stored in Azure Storage, for faster read access.</span></span> 

- <span data-ttu-id="1d8ab-218">**Azure Active Directory** (Azure AD)，以驗證客戶與 Tailspin 管理員。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-218">**Azure Active Directory** (Azure AD) to authenticate customers and Tailspin administrators.</span></span>

- <span data-ttu-id="1d8ab-219">**Azure SQL Database**，以儲存問卷答覆來進行分析。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-219">**Azure SQL Database** to store the survey answers for analysis.</span></span> 

## <a name="moving-to-service-fabric"></a><span data-ttu-id="1d8ab-220">移往 Service Fabric</span><span class="sxs-lookup"><span data-stu-id="1d8ab-220">Moving to Service Fabric</span></span>

<span data-ttu-id="1d8ab-221">如前所述，此階段的目標是要移轉至 Service Fabric，且所需的變更要最少。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-221">As mentioned, the goal of this phase was migrating to Service Fabric with the minimum necessary changes.</span></span> <span data-ttu-id="1d8ab-222">為此，我們建立了對應至原始應用程式中各個雲端服務角色的無狀態服務：</span><span class="sxs-lookup"><span data-stu-id="1d8ab-222">To that end, we created stateless services corresponding to each cloud service role in the original application:</span></span>

![](./images/tailspin02.png)

<span data-ttu-id="1d8ab-223">此架構刻意設計成非常類似原始應用程式。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-223">Intentionally, this architecture is very similar to the original application.</span></span> <span data-ttu-id="1d8ab-224">不過，圖表中會隱藏一些重要差異。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-224">However, the diagram hides some important differences.</span></span> <span data-ttu-id="1d8ab-225">在本文的其餘部分，我們會探討這些差異。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-225">In the rest of this article, we'll explore those differences.</span></span> 


## <a name="converting-the-cloud-service-roles-to-services"></a><span data-ttu-id="1d8ab-226">將雲端服務角色轉換為服務</span><span class="sxs-lookup"><span data-stu-id="1d8ab-226">Converting the cloud service roles to services</span></span>

<span data-ttu-id="1d8ab-227">如前所述，我們已將每個雲端服務角色移轉到 Service Fabric 服務。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-227">As mentioned, we migrated each cloud service role to a Service Fabric service.</span></span> <span data-ttu-id="1d8ab-228">由於雲端服務角色是無狀態的，因為在此階段中，我們必須在 Service Fabric 中建立無狀態服務。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-228">Because cloud service roles are stateless, for this phase it made sense to create stateless services in Service Fabric.</span></span> 

<span data-ttu-id="1d8ab-229">在移轉時，我們遵循了[將 Web 角色和背景工作角色轉換成 Service Fabric 無狀態服務的指南][sf-migration]中概述的步驟。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-229">For the migration, we followed the steps outlined in [Guide to converting Web and Worker Roles to Service Fabric stateless services][sf-migration].</span></span> 

### <a name="creating-the-web-front-end-services"></a><span data-ttu-id="1d8ab-230">建立 Web 前端服務</span><span class="sxs-lookup"><span data-stu-id="1d8ab-230">Creating the web front-end services</span></span>

<span data-ttu-id="1d8ab-231">在 Service Fabric 中，服務會在 Service Fabric 執行階段所建立的處理序內執行。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-231">In Service Fabric, a service runs inside a process created by the Service Fabric runtime.</span></span> <span data-ttu-id="1d8ab-232">對於 Web 前端來說，這表示服務未在 IIS 內執行。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-232">For a web front end, that means the service is not running inside IIS.</span></span> <span data-ttu-id="1d8ab-233">相反地，服務必須裝載 Web 伺服器。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-233">Instead, the service must host a web server.</span></span> <span data-ttu-id="1d8ab-234">這種方式稱為「自我裝載」，因為在處理序內執行的程式碼會作為 Web 伺服器主機。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-234">This approach is called *self-hosting*, because the code that runs inside the process acts as the web server host.</span></span> 

<span data-ttu-id="1d8ab-235">自我裝載的需求代表 Service Fabric 服務無法使用 ASP.NET MVC 或 ASP.NET Web Form，因為這些架構需要 IIS，並不支援自我裝載。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-235">The requirement to self-host means that a Service Fabric service can't use ASP.NET MVC or ASP.NET Web Forms, because those frameworks require IIS and do not support self-hosting.</span></span> <span data-ttu-id="1d8ab-236">自我裝載選項包含：</span><span class="sxs-lookup"><span data-stu-id="1d8ab-236">Options for self-hosting include:</span></span>

- <span data-ttu-id="1d8ab-237">[ASP.NET Core][aspnet-core]，使用 [Kestrel][kestrel] Web 伺服器自我裝載。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-237">[ASP.NET Core][aspnet-core], self-hosted using the [Kestrel][kestrel] web server.</span></span> 
- <span data-ttu-id="1d8ab-238">[ASP.NET Web API][aspnet-webapi]，使用 [OWIN][owin] 自我裝載。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-238">[ASP.NET Web API][aspnet-webapi], self-hosted using [OWIN][owin].</span></span>
- <span data-ttu-id="1d8ab-239">[Nancy](http://nancyfx.org/) 之類的第三方架構。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-239">Third-party frameworks such as [Nancy](http://nancyfx.org/).</span></span>

<span data-ttu-id="1d8ab-240">原始的 Surveys 應用程式使用 ASP.NET MVC。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-240">The original Surveys application uses ASP.NET MVC.</span></span> <span data-ttu-id="1d8ab-241">因為 ASP.NET MVC 無法自我裝載在 Service Fabric 中，我們考慮了下列移轉選項：</span><span class="sxs-lookup"><span data-stu-id="1d8ab-241">Because ASP.NET MVC cannot be self-hosted in Service Fabric, we considered the following migration options:</span></span>

- <span data-ttu-id="1d8ab-242">將 Web 角色植入 ASP.NET Core，以便能夠自我裝載。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-242">Port the web roles to ASP.NET Core, which can be self-hosted.</span></span>
- <span data-ttu-id="1d8ab-243">將網站轉換為單一頁面應用程式 (SPA)，以使用 ASP.NET Web API 呼叫 Web API 實作。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-243">Convert the web site into a single-page application (SPA) that calls a web API implemented using ASP.NET Web API.</span></span> <span data-ttu-id="1d8ab-244">這會需要徹底重新設計的 Web 前端。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-244">This would have required a complete redesign of the web front end.</span></span>
- <span data-ttu-id="1d8ab-245">保留現有的 ASP.NET MVC 程式碼，並將 Windows Server 容器中的 IIS 部署到 Service Fabric。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-245">Keep the existing ASP.NET MVC code and deploy IIS in a Windows Server container to Service Fabric.</span></span> <span data-ttu-id="1d8ab-246">這個方式只需要稍微變更程式碼，或完全不必變更。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-246">This approach would require little or no code change.</span></span> <span data-ttu-id="1d8ab-247">不過，Service Fabric 中的[容器支援][sf-containers]目前仍是預覽本。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-247">However, [container support][sf-containers] in Service Fabric is currently still in preview.</span></span>

<span data-ttu-id="1d8ab-248">根據這些考量，我們選擇了第一個選項，也就是移植到 ASP.NET Core。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-248">Based on these considerations, we selected the first option, porting to ASP.NET Core.</span></span> <span data-ttu-id="1d8ab-249">為了這麼做，我們遵循了[從 ASP.NET MVC 移轉到 ASP.NET Core MVC][aspnet-migration] 所述的步驟。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-249">To do so, we followed the steps described in [Migrating From ASP.NET MVC to ASP.NET Core MVC][aspnet-migration].</span></span> 

> [!NOTE]
> <span data-ttu-id="1d8ab-250">為求安全，在搭配使用 ASP.NET Core 與 Kestrel 時，您應該在 Kestrel 前面放置反向 Proxy 以處理來自網際網路的流量。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-250">When using ASP.NET Core with Kestrel, you should place a reverse proxy in front of Kestrel to handle traffic from the Internet, for security reasons.</span></span> <span data-ttu-id="1d8ab-251">如需詳細資訊，請參閱 [ASP.NET Core 中的 Kestrel Web 伺服器實作][kestrel]。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-251">For more information, see [Kestrel web server implementation in ASP.NET Core][kestrel].</span></span> <span data-ttu-id="1d8ab-252">[部署應用程式](#deploying-the-application)一節會說明建議的 Azure 部署。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-252">The section [Deploying the application](#deploying-the-application) describes a recommended Azure deployment.</span></span>

### <a name="http-listeners"></a><span data-ttu-id="1d8ab-253">HTTP 接聽程式</span><span class="sxs-lookup"><span data-stu-id="1d8ab-253">HTTP listeners</span></span>

<span data-ttu-id="1d8ab-254">在雲端服務中，Web 角色或背景工作角色會藉由在[服務定義檔][cloud-service-endpoints]宣告 HTTP 端點，以將其公開。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-254">In Cloud Services, a web or worker role exposes an HTTP endpoint by declaring it in the [service definition file][cloud-service-endpoints].</span></span> <span data-ttu-id="1d8ab-255">Web 角色必須至少有一個端點。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-255">A web role must have at least one endpoint.</span></span>

```xml
<!-- Cloud service endpoint -->
<Endpoints>
    <InputEndpoint name="HttpIn" protocol="http" port="80" />
</Endpoints>
```

<span data-ttu-id="1d8ab-256">同樣地，Service Fabric 端點會於服務資訊清單中宣告：</span><span class="sxs-lookup"><span data-stu-id="1d8ab-256">Similarly, Service Fabric endpoints are declared in a service manifest:</span></span> 

```xml
<!-- Service Fabric endpoint -->
<Endpoints>
    <Endpoint Protocol="http" Name="ServiceEndpoint" Type="Input" Port="8002" />
</Endpoints>
```

<span data-ttu-id="1d8ab-257">不過，不同於雲端服務角色，Service Fabric 服務可共置於相同節點內。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-257">Unlike a cloud service role, however, Service Fabric services can be co-located within the same node.</span></span> <span data-ttu-id="1d8ab-258">因此，每個服務都必須接聽不同連接埠。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-258">Therefore, every service must listen on a distinct port.</span></span> <span data-ttu-id="1d8ab-259">在本文稍後，我們會討論連接埠 80 或連接埠 443 上的用戶端要求是如何路由至服務的正確連接埠。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-259">Later in this article, we'll discuss how client requests on port 80 or port 443 get routed to the correct port for the service.</span></span>

<span data-ttu-id="1d8ab-260">服務必須明確地建立每個端點的接聽程式。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-260">A service must explicitly create listeners for each endpoint.</span></span> <span data-ttu-id="1d8ab-261">原因是 Service Fabric 對於通訊堆疊是無從驗證的。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-261">The reason is that Service Fabric is agnostic about communication stacks.</span></span> <span data-ttu-id="1d8ab-262">如需詳細資訊，請參閱[使用 ASP.NET Core 建置應用程式的 Web 服務前端][sf-aspnet-core]。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-262">For more information, see [Build a web service front end for your application using ASP.NET Core][sf-aspnet-core].</span></span>

## <a name="packaging-and-configuration"></a><span data-ttu-id="1d8ab-263">封裝和設定</span><span class="sxs-lookup"><span data-stu-id="1d8ab-263">Packaging and configuration</span></span>

 <span data-ttu-id="1d8ab-264">雲端服務包含下列組態和套件檔案：</span><span class="sxs-lookup"><span data-stu-id="1d8ab-264">A cloud service contains the following configuration and package files:</span></span>

| <span data-ttu-id="1d8ab-265">檔案</span><span class="sxs-lookup"><span data-stu-id="1d8ab-265">File</span></span> | <span data-ttu-id="1d8ab-266">說明</span><span class="sxs-lookup"><span data-stu-id="1d8ab-266">Description</span></span> |
|------|-------------|
| <span data-ttu-id="1d8ab-267">服務定義 (.csdef)</span><span class="sxs-lookup"><span data-stu-id="1d8ab-267">Service definition (.csdef)</span></span> | <span data-ttu-id="1d8ab-268">Azure 用來設定雲端服務的設定。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-268">Settings used by Azure to configure the cloud service.</span></span> <span data-ttu-id="1d8ab-269">定義角色、端點、啟動工作和組態設定名稱。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-269">Defines the roles, endpoints, startup tasks, and the names of configuration settings.</span></span> |
| <span data-ttu-id="1d8ab-270">服務組態 (.cscfg)</span><span class="sxs-lookup"><span data-stu-id="1d8ab-270">Service configuration (.cscfg)</span></span> | <span data-ttu-id="1d8ab-271">每一部署的設定，包括角色執行個體數目、端點連接埠號碼，以及組態設定的值。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-271">Per-deployment settings, including the number of role instances, endpoint port numbers, and the values of configuration settings.</span></span> 
| <span data-ttu-id="1d8ab-272">服務套件 (.cspkg)</span><span class="sxs-lookup"><span data-stu-id="1d8ab-272">Service package (.cspkg)</span></span> | <span data-ttu-id="1d8ab-273">包含應用程式程式碼和組態以及服務定義檔。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-273">Contains the application code and configurations, and the service definition file.</span></span>  |

<span data-ttu-id="1d8ab-274">整個應用程式有一個 .csdef 檔案。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-274">There is one .csdef file for the entire application.</span></span> <span data-ttu-id="1d8ab-275">您可以有多個用於不同環境的 .cscfg 檔案，例如本機、測試或生產環境。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-275">You can have multiple .cscfg files for different environments, such as local, test, or production.</span></span> <span data-ttu-id="1d8ab-276">當服務執行時，您可以更新 .cscfg，但不能更新 .csdef。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-276">When the service is running, you can update the .cscfg but not the .csdef.</span></span> <span data-ttu-id="1d8ab-277">如需詳細資訊，請參閱[什麼是雲端服務模型？如何封裝？][cloud-service-config]</span><span class="sxs-lookup"><span data-stu-id="1d8ab-277">For more information, see [What is the Cloud Service model and how do I package it?][cloud-service-config]</span></span>

<span data-ttu-id="1d8ab-278">Service Fabric 在服務「定義」和服務「設定」之間有相似的劃分，但結構更為細微。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-278">Service Fabric has a similar division between a service *definition* and service *settings*, but the structure is more granular.</span></span> <span data-ttu-id="1d8ab-279">若要了解 Service Fabric 的組態模型，了解 Service Fabric 應用程式的封裝方式會有所幫助。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-279">To understand Service Fabric's configuration model, it helps to understand how a Service Fabric application is packaged.</span></span> <span data-ttu-id="1d8ab-280">結構如下：</span><span class="sxs-lookup"><span data-stu-id="1d8ab-280">Here is the structure:</span></span>

```
Application package
  - Service packages
    - Code package
    - Configuration package
    - Data package (optional)
```

<span data-ttu-id="1d8ab-281">應用程式套件是您部署的項目。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-281">The application package is what you deploy.</span></span> <span data-ttu-id="1d8ab-282">它包含一或多個服務套件。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-282">It contains one or more service packages.</span></span> <span data-ttu-id="1d8ab-283">服務套件包含程式碼、組態和資料套件。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-283">A service package contains code, configuration, and data packages.</span></span> <span data-ttu-id="1d8ab-284">程式碼套件包含服務的二進位檔，組態套件則包含組態設定。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-284">The code package contains the binaries for the services, and the configuration package contains configuration settings.</span></span> <span data-ttu-id="1d8ab-285">此模型可讓您不需重新部署整個應用程式，就能升級個別的服務。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-285">This model allows you to upgrade individual services without redeploying the entire application.</span></span> <span data-ttu-id="1d8ab-286">它也可讓您只更新組態設定，而不需重新部署程式碼或重新啟動服務。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-286">It also lets you update just the configuration settings, without redeploying the code or restarting the service.</span></span>

<span data-ttu-id="1d8ab-287">Service Fabric 應用程式包含下列組態檔：</span><span class="sxs-lookup"><span data-stu-id="1d8ab-287">A Service Fabric application contains the following configuration files:</span></span>

| <span data-ttu-id="1d8ab-288">檔案</span><span class="sxs-lookup"><span data-stu-id="1d8ab-288">File</span></span> | <span data-ttu-id="1d8ab-289">位置</span><span class="sxs-lookup"><span data-stu-id="1d8ab-289">Location</span></span> | <span data-ttu-id="1d8ab-290">說明</span><span class="sxs-lookup"><span data-stu-id="1d8ab-290">Description</span></span> |
|------|----------|-------------|
| <span data-ttu-id="1d8ab-291">ApplicationManifest.xml</span><span class="sxs-lookup"><span data-stu-id="1d8ab-291">ApplicationManifest.xml</span></span> | <span data-ttu-id="1d8ab-292">應用程式套件</span><span class="sxs-lookup"><span data-stu-id="1d8ab-292">Application package</span></span> | <span data-ttu-id="1d8ab-293">定義撰寫應用程式的服務。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-293">Defines the services that compose the application.</span></span> |
| <span data-ttu-id="1d8ab-294">ServiceManifest.xml</span><span class="sxs-lookup"><span data-stu-id="1d8ab-294">ServiceManifest.xml</span></span> | <span data-ttu-id="1d8ab-295">服務套件</span><span class="sxs-lookup"><span data-stu-id="1d8ab-295">Service package</span></span>| <span data-ttu-id="1d8ab-296">說明一或多個服務。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-296">Describes one or more services.</span></span> |
| <span data-ttu-id="1d8ab-297">Settings.xml</span><span class="sxs-lookup"><span data-stu-id="1d8ab-297">Settings.xml</span></span> | <span data-ttu-id="1d8ab-298">組態套件</span><span class="sxs-lookup"><span data-stu-id="1d8ab-298">Configuration package</span></span> | <span data-ttu-id="1d8ab-299">包含服務套件中定義之服務的組態設定。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-299">Contains configuration settings for the services defined in the service package.</span></span> |

<span data-ttu-id="1d8ab-300">如需詳細資訊，請參閱[在 Service Fabric 中模型化應用程式][sf-application-model]。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-300">For more information, see [Model an application in Service Fabric][sf-application-model].</span></span>

<span data-ttu-id="1d8ab-301">若要支援多個環境的不同組態設定，請使用下列方式，如[管理多個環境的應用程式參數][sf-multiple-environments]所述：</span><span class="sxs-lookup"><span data-stu-id="1d8ab-301">To support different configuration settings for multiple environments, use the following approach, described in [Manage application parameters for multiple environments][sf-multiple-environments]:</span></span>

1. <span data-ttu-id="1d8ab-302">定義服務的 Setting.xml 檔案中的設定。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-302">Define the setting in the Setting.xml file for the service.</span></span>
2. <span data-ttu-id="1d8ab-303">在應用程式資訊清單中，定義設定的覆寫。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-303">In the application manifest, define an override for the setting.</span></span>
3. <span data-ttu-id="1d8ab-304">將環境專屬設定放入應用程式參數檔案中。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-304">Put environment-specific settings into application parameter files.</span></span>


## <a name="deploying-the-application"></a><span data-ttu-id="1d8ab-305">部署應用程式</span><span class="sxs-lookup"><span data-stu-id="1d8ab-305">Deploying the application</span></span>

<span data-ttu-id="1d8ab-306">Azure 雲端服務是受管理的服務，而 Service Fabric 是執行階段。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-306">Whereas Azure Cloud Services is a managed service, Service Fabric is a runtime.</span></span> <span data-ttu-id="1d8ab-307">您可以在許多環境 (包括 Azure 和內部部署環境) 建立 Service Fabric 叢集。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-307">You can create Service Fabric clusters in many environments, including Azure and on premises.</span></span> <span data-ttu-id="1d8ab-308">在本文中，我們聚焦在部署至 Azure。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-308">In this article, we focus on deploying to Azure.</span></span> 

<span data-ttu-id="1d8ab-309">下圖顯示建議的部署：</span><span class="sxs-lookup"><span data-stu-id="1d8ab-309">The following diagram shows a recommended deployment:</span></span>

![](./images/tailspin-cluster.png)

<span data-ttu-id="1d8ab-310">Service Fabric 叢集會部署到 [VM 擴展集][vm-scale-sets]。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-310">The Service Fabric cluster is deployed to a [VM scale set][vm-scale-sets].</span></span> <span data-ttu-id="1d8ab-311">擴展集是可用來部署及管理一組相同 VM 的 Azure 計算資源。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-311">Scale sets are an Azure Compute resource that can be used to deploy and manage a set of identical VMs.</span></span> 

<span data-ttu-id="1d8ab-312">如前所述，Kestrel Web 伺服器需要反向 Proxy 以確保安全。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-312">As mentioned, the Kestrel web server requires a reverse proxy for security reasons.</span></span> <span data-ttu-id="1d8ab-313">此圖會顯示 [Azure 應用程式閘道][application-gateway]，這是一項 Azure 服務，可提供各種第 7 層的負載平衡功能。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-313">This diagram shows [Azure Application Gateway][application-gateway], which is an Azure service that offers various layer 7 load balancing capabilities.</span></span> <span data-ttu-id="1d8ab-314">它會用來當做反向 Proxy 服務，終止用戶端連線，並將要求轉寄到後端端點。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-314">It acts as a reverse-proxy service, terminating the client connection and forwarding requests to back-end endpoints.</span></span> <span data-ttu-id="1d8ab-315">您可以使用不同的反向 Proxy 解決方案，例如 nginx。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-315">You might use a different reverse proxy solution, such as nginx.</span></span>  

### <a name="layer-7-routing"></a><span data-ttu-id="1d8ab-316">第 7 層路由</span><span class="sxs-lookup"><span data-stu-id="1d8ab-316">Layer 7 routing</span></span>

<span data-ttu-id="1d8ab-317">在[原始的 Surveys 應用程式](https://msdn.microsoft.com/en-us/library/hh534477.aspx#sec21)中，一個 Web 角色接聽連接埠 80，另一個 Web 角色接聽連接埠 443。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-317">In the [original Surveys application](https://msdn.microsoft.com/en-us/library/hh534477.aspx#sec21), one web role listened on port 80, and the other web role listened on port 443.</span></span> 

| <span data-ttu-id="1d8ab-318">公用網站</span><span class="sxs-lookup"><span data-stu-id="1d8ab-318">Public site</span></span> | <span data-ttu-id="1d8ab-319">問卷管理網站</span><span class="sxs-lookup"><span data-stu-id="1d8ab-319">Survey management site</span></span> |
|-------------|------------------------|
| `http://tailspin.cloudapp.net` | `https://tailspin.cloudapp.net` |

<span data-ttu-id="1d8ab-320">另一個選項是使用第 7 層路由。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-320">Another option is to use layer 7 routing.</span></span> <span data-ttu-id="1d8ab-321">在此方式中，不同的 URL 路徑會路由至後端上的不同連接埠號碼。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-321">In this approach, different URL paths get routed to different port numbers on the back end.</span></span> <span data-ttu-id="1d8ab-322">例如，公用網站可能會使用開頭為 `/public/` 的 URL 路徑。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-322">For example, the public site might use URL paths starting with `/public/`.</span></span> 

<span data-ttu-id="1d8ab-323">第 7 層路由的選項包括：</span><span class="sxs-lookup"><span data-stu-id="1d8ab-323">Options for layer 7 routing include:</span></span>

- <span data-ttu-id="1d8ab-324">使用應用程式閘道。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-324">Use Application Gateway.</span></span> 

- <span data-ttu-id="1d8ab-325">使用網路虛擬設備 (NVA)，例如 nginx。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-325">Use a network virtual appliance (NVA), such as nginx.</span></span>

- <span data-ttu-id="1d8ab-326">將自訂閘道撰寫為無狀態服務。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-326">Write a custom gateway as a stateless service.</span></span>

<span data-ttu-id="1d8ab-327">如果您有兩個以上的服務具有公開的 HTTP 端點，但想要讓它們顯示為每個站台皆具有單一網域名稱，請考慮這種方式。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-327">Consider this approach if you have two or more services with public HTTP endpoints, but want them to appear as one site with a single domain name.</span></span>

> <span data-ttu-id="1d8ab-328">有一個方式我們並「不」建議，那就是讓外部用戶端透過 Service Fabric [反向 Proxy][sf-reverse-proxy] 傳送要求。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-328">One approach that we *don't* recommend is allowing external clients to send requests through the Service Fabric [reverse proxy][sf-reverse-proxy].</span></span> <span data-ttu-id="1d8ab-329">雖然這是可行方式，但反向 Proxy 應該要用於服務對服務的通訊。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-329">Although this is possible, the reverse proxy is intended for service-to-service communication.</span></span> <span data-ttu-id="1d8ab-330">對外部用戶端公開，將會暴露具有 HTTP 端點之叢集中執行的「任何」服務。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-330">Opening it to external clients exposes *any* service running in the cluster that has an HTTP endpoint.</span></span>

### <a name="node-types-and-placement-constraints"></a><span data-ttu-id="1d8ab-331">節點類型和放置條件約束</span><span class="sxs-lookup"><span data-stu-id="1d8ab-331">Node types and placement constraints</span></span>

<span data-ttu-id="1d8ab-332">在如上所示的部署中，所有服務會在所有節點上執行。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-332">In the deployment shown above, all the services run on all the nodes.</span></span> <span data-ttu-id="1d8ab-333">不過，您也可以將服務分組，讓特定服務只在叢集內的特定節點上執行。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-333">However, you can also group services, so that certain services run only on particular nodes within the cluster.</span></span> <span data-ttu-id="1d8ab-334">會使用此方式的原因包括：</span><span class="sxs-lookup"><span data-stu-id="1d8ab-334">Reasons to use this approach include:</span></span>

- <span data-ttu-id="1d8ab-335">要在不同 VM 類型上執行某些服務。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-335">Run some services on different VM types.</span></span> <span data-ttu-id="1d8ab-336">例如，某些服務可能需要大量計算或需要 GPU。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-336">For example, some services might be compute-intensive or require GPUs.</span></span> <span data-ttu-id="1d8ab-337">您可以在 Service Fabric 叢集中混合使用不同的 VM 類型。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-337">You can have a mix of VM types in your Service Fabric cluster.</span></span>
- <span data-ttu-id="1d8ab-338">隔離前端服務與後端服務，以確保安全。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-338">Isolate front-end services from back-end services, for security reasons.</span></span> <span data-ttu-id="1d8ab-339">所有前端服務會在一組節點上執行，後端服務則會在相同叢集內的不同節點上執行。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-339">All the front-end services will run on one set of nodes, and the back-end services will run on different nodes in the same cluster.</span></span>
- <span data-ttu-id="1d8ab-340">不同的調整需求。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-340">Different scale requirements.</span></span> <span data-ttu-id="1d8ab-341">某些服務可能需要比其他服務在更多節點上執行。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-341">Some services might need to run on more nodes than other services.</span></span> <span data-ttu-id="1d8ab-342">例如，如果您定義前端節點和後端節點，每一組都可以獨立調整。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-342">For example, if you define front-end nodes and back-end nodes, each set can be scaled independently.</span></span>

<span data-ttu-id="1d8ab-343">下圖顯示分隔前端和後端服務的叢集：</span><span class="sxs-lookup"><span data-stu-id="1d8ab-343">The following diagram shows a cluster that separates front-end and back-end services:</span></span>

![](././images/node-placement.png)

<span data-ttu-id="1d8ab-344">若要實作這種方式：</span><span class="sxs-lookup"><span data-stu-id="1d8ab-344">To implement this approach:</span></span>

1.  <span data-ttu-id="1d8ab-345">在建立叢集時，請定義兩個以上的節點類型。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-345">When you create the cluster, define two or more node types.</span></span> 
2.  <span data-ttu-id="1d8ab-346">對每個服務使用[放置條件約束][sf-placement-constraints]以將服務指派給某個節點類型。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-346">For each service, use [placement constraints][sf-placement-constraints] to assign the service to a node type.</span></span>

<span data-ttu-id="1d8ab-347">當您部署至 Azure 時，每個節點類型都會部署到不同的 VM 擴展集。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-347">When you deploy to Azure, each node type is deployed to a separate VM scale set.</span></span> <span data-ttu-id="1d8ab-348">Service Fabric 叢集橫跨所有節點類型。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-348">The Service Fabric cluster spans all node types.</span></span> <span data-ttu-id="1d8ab-349">如需詳細資訊，請參閱 [Service Fabric 節點類型與虛擬機器調整集之間的關聯性][sf-node-types]。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-349">For more information, see [The relationship between Service Fabric node types and Virtual Machine Scale Sets][sf-node-types].</span></span>

> <span data-ttu-id="1d8ab-350">如果叢集有多個節點類型，系統會將某種節點類型指定為「主要」節點類型。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-350">If a cluster has multiple node types, one node type is designated as the *primary* node type.</span></span> <span data-ttu-id="1d8ab-351">Service Fabric 執行階段服務 (例如叢集管理服務) 會在主要節點類型上執行。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-351">Service Fabric runtime services, such as the Cluster Management Service, run on the primary node type.</span></span> <span data-ttu-id="1d8ab-352">在生產環境中，請為主要節點類型佈建至少 5 個節點。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-352">Provision at least 5 nodes for the primary node type in a production environment.</span></span> <span data-ttu-id="1d8ab-353">其他節點類型則至少應該有 2 個節點。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-353">The other node type should have at least 2 nodes.</span></span>

## <a name="configuring-and-managing-the-cluster"></a><span data-ttu-id="1d8ab-354">設定和管理叢集</span><span class="sxs-lookup"><span data-stu-id="1d8ab-354">Configuring and managing the cluster</span></span>

<span data-ttu-id="1d8ab-355">您必須保護叢集，以免未經授權的使用者連線到您的叢集。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-355">Clusters must be secured to prevent unauthorized users from connecting to your cluster.</span></span> <span data-ttu-id="1d8ab-356">建議您使用 Azure AD 來驗證用戶端，並使用 X.509 憑證以確保節點對節點的安全性。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-356">It is recommended to use Azure AD to authenticate clients, and X.509 certificates for node-to-node security.</span></span> <span data-ttu-id="1d8ab-357">如需詳細資訊，請參閱 [Service Fabric 叢集安全性案例][sf-security]。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-357">For more information, see [Service Fabric cluster security scenarios][sf-security].</span></span>

<span data-ttu-id="1d8ab-358">若要設定公用 HTTPS 端點，請參閱[在服務資訊清單中指定資源][sf-manifest-resources]。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-358">To configure a public HTTPS endpoint, see [Specify resources in a service manifest][sf-manifest-resources].</span></span>

<span data-ttu-id="1d8ab-359">您可以在叢集中新增 VM，以將應用程式相應放大。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-359">You can scale out the application by adding VMs to the cluster.</span></span> <span data-ttu-id="1d8ab-360">VM 擴展集支援使用自動調整規則，根據效能計數器來進行自動調整。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-360">VM scale sets support auto-scaling using auto-scale rules based on performance counters.</span></span> <span data-ttu-id="1d8ab-361">如需詳細資訊，請參閱[使用自動調整規則相應縮小或放大 Service Fabric 叢集][sf-auto-scale]。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-361">For more information, see [Scale a Service Fabric cluster in or out using auto-scale rules][sf-auto-scale].</span></span>

<span data-ttu-id="1d8ab-362">在叢集執行時，您應該將所有節點的記錄收集到集中位置。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-362">While the cluster is running, you should collect logs from all the nodes in a central location.</span></span> <span data-ttu-id="1d8ab-363">如需詳細資訊，請參閱[使用 Azure 診斷收集記錄][sf-logs]。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-363">For more information, see [Collect logs by using Azure Diagnostics][sf-logs].</span></span>   


## <a name="conclusion"></a><span data-ttu-id="1d8ab-364">結論</span><span class="sxs-lookup"><span data-stu-id="1d8ab-364">Conclusion</span></span>

<span data-ttu-id="1d8ab-365">將 Surveys 應用程式移植到 Service Fabric 的做法相當簡單。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-365">Porting the Surveys application to Service Fabric was fairly straightforward.</span></span> <span data-ttu-id="1d8ab-366">總結來說，我們已完成下列工作：</span><span class="sxs-lookup"><span data-stu-id="1d8ab-366">To summarize, we did the following:</span></span>

- <span data-ttu-id="1d8ab-367">將角色轉換為無狀態服務。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-367">Converted the roles to stateless services.</span></span>
- <span data-ttu-id="1d8ab-368">將 Web 前端轉換為 ASP.NET Core。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-368">Converted the web front ends to ASP.NET Core.</span></span>
- <span data-ttu-id="1d8ab-369">將封裝和設定檔變更為 Service Fabric 模型。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-369">Changed the packaging and configuration files to the Service Fabric model.</span></span>

<span data-ttu-id="1d8ab-370">此外，部署已從雲端服務變更為 VM 擴展集內執行的 Service Fabric 叢集。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-370">In addition, the deployment changed from Cloud Services to a Service Fabric cluster running in a VM Scale Set.</span></span>

<span data-ttu-id="1d8ab-371">不過，此時應用程式並未獲得微服務的所有優勢，例如獨立的服務部署和版本設定。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-371">However, at this point the application does not get all the benefits of microservices, such as independent service deployment and versioning.</span></span> <span data-ttu-id="1d8ab-372">若要獲得 Service Fabric 的所有優勢，Tailspin 需要再進行一些最佳化。</span><span class="sxs-lookup"><span data-stu-id="1d8ab-372">To take full advantage of Service Fabric, Tailspin needs to optimize a bit further.</span></span>



<!-- links -->

[application-gateway]: /azure/application-gateway/
[aspnet-core]: /aspnet/core/
[aspnet-webapi]: https://www.asp.net/web-api
[aspnet-migration]: /aspnet/core/migration/mvc
[aspnet-hosting]: /aspnet/core/fundamentals/hosting
[aspnet-webapi]: https://www.asp.net/web-api
[azure-deployment-models]: /azure/azure-resource-manager/resource-manager-deployment-model
[cloud-service-autoscale]: /azure/cloud-services/cloud-services-how-to-scale-portal
[cloud-service-config]: /azure/cloud-services/cloud-services-model-and-package
[cloud-service-endpoints]: /azure/cloud-services/cloud-services-enable-communication-role-instances#worker-roles-vs-web-roles
[kestrel]: https://docs.microsoft.com/aspnet/core/fundamentals/servers/kestrel
[lb-probes]: /azure/load-balancer/load-balancer-custom-probe-overview
[owin]: https://www.asp.net/aspnet/overview/owin-and-katana
[sample-code]: https://github.com/mspnp/cloud-services-to-service-fabric
[sf-application-model]: /azure/service-fabric/service-fabric-application-model
[sf-aspnet-core]: /azure/service-fabric/service-fabric-add-a-web-frontend
[sf-auto-scale]: /azure/service-fabric/service-fabric-cluster-scale-up-down
[sf-compare-cloud-services]: /azure/service-fabric/service-fabric-cloud-services-migration-differences
[sf-connect-and-communicate]: /azure/service-fabric/service-fabric-connect-and-communicate-with-services
[sf-containers]: /azure/service-fabric/service-fabric-containers-overview
[sf-logs]: /azure/service-fabric/service-fabric-diagnostics-how-to-setup-wad
[sf-manifest-resources]: /azure/service-fabric/service-fabric-service-manifest-resources
[sf-migration]: /azure/service-fabric/service-fabric-cloud-services-migration-worker-role-stateless-service
[sf-multiple-environments]: /azure/service-fabric/service-fabric-manage-multiple-environment-app-configuration
[sf-node-types]: /azure/service-fabric/service-fabric-cluster-nodetypes
[sf-overview]: /azure/service-fabric/service-fabric-overview
[sf-placement-constraints]: /azure/service-fabric/service-fabric-cluster-resource-manager-cluster-description
[sf-reliable-collections]: /azure/service-fabric/service-fabric-reliable-services-reliable-collections
[sf-reliable-services]: /azure/service-fabric/service-fabric-reliable-services-introduction
[sf-reverse-proxy]: /azure/service-fabric/service-fabric-reverseproxy
[sf-security]: /azure/service-fabric/service-fabric-cluster-security
[sf-why-microservices]: /azure/service-fabric/service-fabric-overview-microservices
[tailspin-book]: https://msdn.microsoft.com/en-us/library/ff966499.aspx
[tailspin-scenario]: https://msdn.microsoft.com/en-us/library/hh534482.aspx
[unity]: https://msdn.microsoft.com/en-us/library/ff647202.aspx
[vm-scale-sets]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview