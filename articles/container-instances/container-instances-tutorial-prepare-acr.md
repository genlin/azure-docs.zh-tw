---
title: "Azure 容器執行個體教學課程 - 準備 Azure Container Registry"
description: "Azure 容器執行個體教學課程第 2 部分 (共 3 部分) - 準備 Azure Container Registry"
services: container-instances
author: neilpeterson
manager: timlt
ms.service: container-instances
ms.topic: tutorial
ms.date: 01/02/2018
ms.author: seanmck
ms.custom: mvc
ms.openlocfilehash: 94ecba44b8281460da4518c146aab814d2eaa850
ms.sourcegitcommit: 1fbaa2ccda2fb826c74755d42a31835d9d30e05f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/22/2018
---
# <a name="deploy-and-use-azure-container-registry"></a>部署和使用 Azure Container Registry

這是三段式教學課程的第二段。 在[上一個步驟](container-instances-tutorial-prepare-app.md)中，我們已針對使用 [Node.js][nodejs] 所編寫的簡單 Web 應用程式建立容器映像。 在本教學課程中，您會將此映像推送至 Azure Container Registry。 如果您尚未建立容器映像，請回到[教學課程 1 – 建立容器映像](container-instances-tutorial-prepare-app.md)。

Azure Container Registry 是以 Azure 為基礎的私人登錄，用於裝載 Docker 容器映像。 本教學課程逐步解說如何部署 Azure Container Registry 執行個體，以及將容器映像推送至該執行個體。

在本文 (本系列的第二部分) 中，您將：

> [!div class="checklist"]
> * 部署 Azure Container Registry 執行個體
> * 標記 Azure Container Registry 的容器映像
> * 將映像上傳至您的登錄

在下一篇文章 (本系列的最後一個教學課程) 中，您會從私人登錄將容器部署至 Azure 容器執行個體。

## <a name="before-you-begin"></a>開始之前

本教學課程需要您執行 Azure CLI 2.0.23 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI 2.0][azure-cli-install]。

若要完成本教學課程，您需要在本機安裝 Docker 開發環境。 Docker 提供可輕鬆在 [Mac][docker-mac]、[Windows][docker-windows] 或 [Linux][docker-linux] 系統上設定 Docker 的套件。

Azure Cloud Shell 不包括完成本教學課程每個步驟所需的 Docker 元件。 您必須在本機電腦上安裝 Azure CLI 和 Docker 開發環境，才能完成本教學課程。

## <a name="deploy-azure-container-registry"></a>部署 Azure Container Registry

部署 Azure Container Registry 時，您必須先有資源群組。 Azure 資源群組是在其中部署與管理 Azure 資源的邏輯集合。

使用 [az group create][az-group-create] 命令來建立資源群組。 在此範例中，會在 eastus 區域中建立名為 myResourceGroup 的資源群組。

```azurecli
az group create --name myResourceGroup --location eastus
```

使用 [az acr create][az-acr-create] 命令建立 Azure Container Registry。 容器登錄名稱在 Azure 內必須是唯一的，且必須包含 5-50 個英數字元。 以登錄的唯一名稱取代 `<acrName>`：

```azurecli
az acr create --resource-group myResourceGroup --name <acrName> --sku Basic
```

例如，建立名為 *mycontainerregistry082* 的 Azure Container Registry：

```azurecli
az acr create --resource-group myResourceGroup --name mycontainerregistry082 --sku Basic --admin-enabled true
```

在本教學課程的其餘部分，我們使用 `<acrName>` 作為您選擇之容器登錄名稱的預留位置。

## <a name="container-registry-login"></a>Container Registry 登入

您必須先登入 Azure Container Registry 執行個體，才能將映像推送給它。 請使用 [az acr login][az-acr-login] 命令完成此作業。 您必須在建立容器登錄時，為它提供唯一名稱。

```azurecli
az acr login --name <acrName>
```

完成後，此命令會傳回 `Login Succeeded` 訊息。

## <a name="tag-container-image"></a>標記容器映像

若要從私人登錄部署容器映像，您必須為此映像標記登錄的 `loginServer` 名稱。

若要查看目前的映像清單，請使用 [docker images][docker-images] 命令。

```bash
docker images
```

輸出：

```bash
REPOSITORY                   TAG                 IMAGE ID            CREATED              SIZE
aci-tutorial-app             latest              5c745774dfa9        39 seconds ago       68.1 MB
```

若要取得 loginServer 名稱，請執行 [az acr show][az-acr-show] 命令。 以您的容器登錄名稱取代 `<acrName>`。

```azurecli
az acr show --name <acrName> --query loginServer --output table
```

範例輸出︰

```
Result
------------------------
mycontainerregistry082.azurecr.io
```

以您容器登錄的 loginServer 標記 *aci-tutorial-app*映像。 此外，將 `:v1` 新增至映像名稱的結尾。 此標籤指示映像版本號碼。 以您剛剛執行 [az acr show][az-acr-show] 命令的結果取代 `<acrLoginServer>`。

```bash
docker tag aci-tutorial-app <acrLoginServer>/aci-tutorial-app:v1
```

標記之後，執行 `docker images` 來驗證作業。

```bash
docker images
```

輸出：

```bash
REPOSITORY                                                TAG                 IMAGE ID            CREATED             SIZE
aci-tutorial-app                                          latest              5c745774dfa9        39 seconds ago      68.1 MB
mycontainerregistry082.azurecr.io/aci-tutorial-app        v1                  a9dace4e1a17        7 minutes ago       68.1 MB
```

## <a name="push-image-to-azure-container-registry"></a>將映像推送至 Azure Container Registry

使用 [docker push][docker-push] 命令將 *aci-tutorial-app* 映像推送至登錄。 以您在稍早步驟中取得的完整登入伺服器名稱取代 `<acrLoginServer>`。

```bash
docker push <acrLoginServer>/aci-tutorial-app:v1
```

視您的網際網路連線而定，`push` 作業應該會花費幾秒鐘到幾分鐘的時間，且輸出會與以下類似：

```bash
The push refers to a repository [mycontainerregistry082.azurecr.io/aci-tutorial-app]
3db9cac20d49: Pushed
13f653351004: Pushed
4cd158165f4d: Pushed
d8fbd47558a8: Pushed
44ab46125c35: Pushed
5bef08742407: Pushed
v1: digest: sha256:ed67fff971da47175856505585dcd92d1270c3b37543e8afd46014d328f05715 size: 1576
```

## <a name="list-images-in-azure-container-registry"></a>在 Azure Container Registry 中列出映像

若要傳回已推送至 Azure Container Registry 的映像清單，請使用 [az acr repository list][az-acr-repository-list] 命令。 使用容器登錄名稱來更新命令。

```azurecli
az acr repository list --name <acrName> --output table
```

輸出：

```azurecli
Result
----------------
aci-tutorial-app
```

而後若要查看特定映像的標籤，請使用 [az acr repository show-tags][az-acr-repository-show-tags] 命令。

```azurecli
az acr repository show-tags --name <acrName> --repository aci-tutorial-app --output table
```

輸出：

```azurecli
Result
--------
v1
```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您準備了 Azure Container Registry 與 Azure 容器執行個體搭配使用，並已將容器映像推送至登錄。 已完成下列步驟：

> [!div class="checklist"]
> * 已部署 Azure Container Registry 執行個體
> * 標記 Azure Container Registry 的容器映像
> * 將映像上傳至 Azure Container Registry

進入下一個教學課程，來了解如何使用 Azure 容器執行個體將容器部署至 Azure 中。

> [!div class="nextstepaction"]
> [將容器部署至 Azure 容器執行個體](./container-instances-tutorial-deploy-app.md)

<!-- LINKS - External -->
[docker-build]: https://docs.docker.com/engine/reference/commandline/build/
[docker-get-started]: https://docs.docker.com/get-started/
[docker-hub-nodeimage]: https://store.docker.com/images/node
[docker-images]: https://docs.docker.com/engine/reference/commandline/images/
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-login]: https://docs.docker.com/engine/reference/commandline/login/
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[docker-windows]: https://docs.docker.com/docker-for-windows/
[nodejs]: http://nodejs.org

<!-- LINKS - Internal -->
[az-acr-create]: /cli/azure/acr#az_acr_create
[az-acr-login]: /cli/azure/acr#az_acr_login
[az-acr-repository-list]: /cli/azure/acr/repository#az_acr_list
[az-acr-repository-show-tags]: /cli/azure/acr/repository#az_acr_repository_show_tags
[az-acr-show]: /cli/azure/acr#az_acr_show
[az-group-create]: /cli/azure/group#az_group_create
[azure-cli-install]: /cli/azure/install-azure-cli
