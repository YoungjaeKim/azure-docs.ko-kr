---
title: "Azure Storage에서 Azure CLI 2.0 사용 | Microsoft Docs"
description: "Azure Storage에서 Azure 명령줄 인터페이스(Azure CLI) 2.0을 사용하여 저장소 계정을 만들어 관리하고 Azure blob과 파일 작업을 수행하는 방법에 대해 알아봅니다. Azure CLI 2.0은 Python으로 작성된 플랫폼 간 도구입니다."
services: storage
documentationcenter: na
author: mmacy
manager: timlt
editor: tysonn
ms.assetid: 
ms.service: storage
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: azurecli
ms.topic: article
ms.date: 02/18/2017
ms.author: marsma
translationtype: Human Translation
ms.sourcegitcommit: 432752c895fca3721e78fb6eb17b5a3e5c4ca495
ms.openlocfilehash: be44ca9d14d6dbb7a50d5c42c163bc66531bb90f
ms.lasthandoff: 03/30/2017


---
# <a name="using-the-azure-cli-20-with-azure-storage"></a>Azure Storage에서 Azure CLI 2.0 사용

플랫폼 간 오픈 소스 Azure CLI 2.0은 Azure 플랫폼을 사용하기 위한 명령 집합을 제공합니다. 풍부한 데이터 액세스를 포함하여 [Azure Portal](https://portal.azure.com)과 동일한 기능을 제공합니다.

이 가이드에서는 [Azure CLI 2.0](https://docs.microsoft.com/cli/azure/get-started-with-az-cli2)을 통해 Azure Storage 계정의 리소스를 사용하는 몇 가지 작업을 수행하는 방법을 설명합니다. 이 가이드를 사용하기 전에 최신 버전의 CLI 2.0을 다운로드하여 설치하거나 업그레이드하는 것이 좋습니다.

이 가이드의 예제에서는 Ubuntu에서 Bash 셸을 사용한다고 가정하지만 다른 플랫폼에서도 비슷하게 수행해야 합니다. 

[!INCLUDE [storage-cli-versions](../../includes/storage-cli-versions.md)]

## <a name="prerequisites"></a>필수 조건
이 가이드에서는 Azure 저장소의 기본 개념을 이해하고 있다고 가정합니다. 또한 Azure와 저장소 서비스에 대해 아래에 지정된 계정 만들기 요구 사항을 충족할 수 있다고 가정합니다.

### <a name="accounts"></a>계정
* **Azure 계정**: Azure 구독이 아직 없는 경우 [무료 Azure 계정을 만듭니다](https://azure.microsoft.com/free/).
* **저장소 계정**: [Azure 저장소 계정 정보](../storage/storage-create-storage-account.md)의 [저장소 계정 만들기](../storage/storage-create-storage-account.md#create-a-storage-account) 섹션을 참조하세요.

### <a name="install-the-azure-cli-20"></a>Azure CLI 2.0 설치

[Azure CLI 2.0 설치](/cli/azure/install-az-cli2)에 설명된 지침에 따라 Azure CLI 2.0을 다운로드하여 설치합니다.

> [!TIP]
> 설치하는 데 문제가 있으면 관련 문서의 [설치 문제 해결](/cli/azure/install-az-cli2#installation-troubleshooting) 섹션 및 GitHub의 [설치 문제 해결](https://github.com/Azure/azure-cli/blob/master/doc/install_troubleshooting.md) 가이드를 참조하세요.
>

## <a name="working-with-the-cli"></a>CLI 사용

CLI를 설치했으면 명령줄 인터페이스(Bash, 터미널, 명령 프롬프트)에서 `az` 명령을 사용하여 Azure CLI 명령에 액세스할 수 있습니다. `az` 명령을 입력하면 다음과 비슷한 결과가 표시됩니다.

```
     /\
    /  \    _____   _ _ __ ___
   / /\ \  |_  / | | | \'__/ _ \
  / ____ \  / /| |_| | | |  __/
 /_/    \_\/___|\__,_|_|  \___|


Welcome to the cool new Azure CLI!

Here are the base commands:

    account   : Commands to manage subscriptions.
    acr       : Commands to manage Azure container registries.
    acs       : Commands to manage Azure container services.
    ad        : Synchronize on-premises directories and manage Azure Active Directory (AAD)
                resources.
    appservice: Commands to manage your Azure web apps and App Service plans.
    cloud     : Manage the Azure clouds registered.
    component : Commands to manage and update Azure CLI 2.0 components.
    configure : Configure Azure CLI 2.0 or view your configuration. The command is
                interactive, so just type `az configure` and respond to the prompts.
    container : Set up automated builds and deployments for multi-container Docker applications.
    disk      : Commands to manage 'Managed Disks'.
    feature   : Commands to manage resource provider features, such as previews.
    feedback  : Loving or hating the CLI?  Let us know!
    group     : Commands to manage resource groups.
    image     : Commands to manage custom virtual machine images based on managed disks/snapshots.
    lock
    login     : Log in to access Azure subscriptions.
    logout    : Log out to remove access to Azure subscriptions.
    network   : Manages Network resources.
    policy    : Commands to manage resource policies.
    provider  : Manage resource providers.
    resource  : Generic commands to manage Azure resources.
    role      : Use role assignments to manage access to your Azure resources.
    snapshot  : Commands to manage snapshots.
    storage   : Durable, highly available, and massively scalable cloud storage.
    tag       : Manage resource tags.
    vm        : Provision Linux and Windows virtual machines in minutes.
    vmss      : Create highly available, auto-scalable Linux or Windows virtual machines.
```

명령줄 인터페이스에서 `az storage -h` 명령을 실행하여 `storage` 그룹 명령과 그 하위 그룹을 나열합니다. 하위 그룹에 대한 설명은 Azure CLI에서 저장소 리소스 작업을 위해 제공하는 기능에 대한 개요입니다.

```
Group
    az storage: Durable, highly available, and massively scalable cloud storage.

Subgroups:
    account  : Manage storage accounts.
    blob     : Object storage for unstructured data.
    container: Manage blob storage containers.
    cors     : Manage Storage service Cross-Orgin Resource Sharing (CORS).
    directory: Manage file storage directories.
    entity   : Manage table storage entities.
    file     : File shares that use the standard SMB 3.0 protocol.
    logging  : Manage Storage service logging information.
    message  : Manage queue storage messages.
    metrics  : Manage Storage service metrics.
    queue    : Effectively scale apps according to traffic using queues.
    share    : Manage file shares.
    table    : NoSQL key-value storage using semi-structured datasets.
```

## <a name="connect-the-cli-to-your-azure-subscription"></a>Azure 구독에 CLI 연결

Azure 구독에 있는 리소스를 사용하려면 먼저 `az login`으로 Azure 계정에 로그인해야 합니다. 로그인할 수 있는 몇 가지 방법은 다음과 같습니다.

* **대화형 로그인**: `az login`
* **사용자 이름 및 암호로 로그인**: `az login -u johndoe@contoso.com -p VerySecret`
  * Microsoft 계정 또는 다단계 인증을 사용하는 계정에서는 작동하지 않습니다.
* **서비스 주체로 로그인**: `az login --service-principal -u http://azure-cli-2016-08-05-14-31-15 -p VerySecret --tenant contoso.onmicrosoft.com`

## <a name="azure-cli-20-sample-script"></a>Azure CLI 2.0 샘플 스크립트

다음으로 Azure Storage 리소스와 상호 작용하는 몇 가지 기본 Azure CLI 2.0 명령을 발급하는 작은 셸 스크립트를 사용합니다. 이 스크립트는 먼저 저장소 계정에 새 컨테이너를 만든 다음 해당 컨테이너에 기존 파일(Blob)을 업로드합니다. 그런 다음 컨테이너에 있는 모든 Blob을 나열하고, 마지막으로 사용자가 지정한 로컬 컴퓨터의 대상에 파일을 다운로드합니다.

```bash
#!/bin/bash
# A simple Azure Storage example script

export AZURE_STORAGE_ACCOUNT=<storage_account_name>
export AZURE_STORAGE_ACCESS_KEY=<storage_account_key>

export container_name=<container_name>
export blob_name=<blob_name>
export file_to_upload=<file_to_upload>
export destination_file=<destination_file>

echo "Creating the container..."
az storage container create -n $container_name

echo "Uploading the file..."
az storage blob upload -f $file_to_upload -c $container_name -n $blob_name

echo "Listing the blobs..."
az storage blob list -c $container_name

echo "Downloading the file..."
az storage blob download -c $container_name -n $blob_name -f $destination_file

echo "Done"
```

**스크립트 구성 및 실행**

1. 원하는 텍스트 편집기를 연 다음 앞서의 스크립트를 복사하여 편집기에 붙여넣습니다.

2. 다음으로 구성 설정을 반영하도록 스크립트의 변수를 업데이트합니다. 다음 값을 지정한 대로 바꿉니다.

   * **\<storage_account_name\>** 저장소 계정의 이름입니다.
   * **\<storage_account_key\>** 저장소 계정의 기본 또는 보조 액세스 키입니다.
   * **\<container_name\>** 새로 만들 컨테이너의 이름입니다(예: "azure-cli-sample-container").
   * **\<Blob_name\>** 컨테이너에 있는 대상 Blob의 이름입니다.
   * **\<file_to_upload\>** 로컬 컴퓨터의 작은 파일 경로입니다(예: "~/images/HelloWorld.png").
   * **\<destination_file\>** 대상 파일 경로입니다(예: "~/downloadedImage.png").

3. 필요한 변수를 업데이트한 후 해당 스크립트를 저장하고 편집기를 종료합니다. 다음 단계에서는 스크립트 이름으로 **my_storage_sample.sh**를 지정했다고 가정합니다.

4. 필요한 경우 다음과 같이 스크립트를 실행 가능으로 표시합니다. `chmod +x my_storage_sample.sh`

5. 스크립트를 실행합니다. 예를 들어 Bash에서는 `./my_storage_sample.sh`입니다.

다음과 비슷한 출력이 표시되며, 스크립트에 지정한 **\<destination_file\>**이 로컬 컴퓨터에 나타납니다.

```
Creating the container...
Success
---------
True
Uploading the file...                                           Percent complete: %100.0
Listing the blobs...
Name           Blob Type      Length  Content Type              Last Modified
-------------  -----------  --------  ------------------------  -------------------------
test_blob.txt  BlockBlob         771  application/octet-stream  2016-12-21T15:35:30+00:00
Downloading the file...
Name
-------------
test_blob.txt
Done
```

> [!TIP]
> 앞서의 출력은 **테이블** 형식입니다. CLI 명령에서 `--output` 인수를 지정하여 사용할 출력 형식을 지정하거나 `az configure`를 사용하여 전역으로 설정할 수 있습니다.
>

## <a name="manage-storage-accounts"></a>저장소 계정 관리

### <a name="create-a-new-storage-account"></a>새 저장소 계정 만들기
Azure Storage를 사용하려면 저장소 계정이 필요합니다. [구독에 연결](#connect-to-your-azure-subscription)하도록 컴퓨터를 구성한 후 새 Azure Storage 계정을 만들 수 있습니다.

```azurecli
az storage account create -l <location> -n <account_name> -g <resource_group> --sku <account_sku>
```

* `-l`[필수]: 위치입니다. 예를 들어 "미국 서부"를 선택합니다.
* `-n`[필수]: 저장소 계정 이름입니다. 이름의 길이는 3-24자여야 하며, 소문자 영숫자만 사용합니다.
* `-g`[필수]: 리소스 그룹의 이름입니다.
* `--sku`[필수]: 저장소 계정 SKU입니다. 허용되는 값은 다음과 같습니다.
  * `Premium_LRS`
  * `Standard_GRS`
  * `Standard_LRS`
  * `Standard_RAGRS`
  * `Standard_ZRS`

### <a name="set-default-azure-storage-account-environment-variables"></a>기본 Azure Storage 계정 환경 변수 설정
Azure 구독에서 여러 저장소 계정을 사용할 수 있습니다. 모든 후속 저장소 명령에 사용하기 위해 이러한 계정 중 하나를 선택하려면 환경 변수를 다음과 같이 설정할 수 있습니다.

```azurecli
export AZURE_STORAGE_ACCOUNT=<account_name>
export AZURE_STORAGE_ACCESS_KEY=<key>
```

기본 저장소 계정을 설정하는 또 다른 방법은 연결 문자열을 사용하는 것입니다. 먼저 `show-connection-string` 명령으로 연결 문자열을 가져옵니다.

```azurecli
az storage account show-connection-string -n <account_name> -g <resource_group>
```

그런 다음 출력 연결 문자열을 복사하고 `AZURE_STORAGE_CONNECTION_STRING` 환경 변수를 설정합니다(연결 문자열을 따옴표로 묶어야 할 수도 있음).

```azurecli
export AZURE_STORAGE_CONNECTION_STRING=<connection_string>
```

> [!NOTE]
> 이 문서의 다음 섹션에 나오는 모든 예제에서는 `AZURE_STORAGE_ACCOUNT` 및 `AZURE_STORAGE_ACCESS_KEY` 환경 변수를 설정했다고 가정합니다.
>

## <a name="create-and-manage-blobs"></a>Blob 만들기 및 관리
Azure Blob 저장소는 HTTP 또는 HTTPS를 통해 전 세계 어디에서든 액세스할 수 있는 다량의 구조화되지 않은 데이터(예: 텍스트 또는 이진 데이터)를 저장할 수 있는 서비스입니다. 이 섹션에서는 Azure Blob 저장소 개념에 이미 익숙하다고 가정합니다. 자세한 내용은 [.NET을 사용하여 Azure Blob Storage 시작](storage-dotnet-how-to-use-blobs.md) 및 [Blob Service 개념](/rest/api/storageservices/fileservices/blob-service-concepts)을 참조하세요.

### <a name="create-a-container"></a>컨테이너 만들기
Azure 저장소의 모든 Blob은 컨테이너에 있어야 합니다. `az storage container create` 명령을 사용하면 컨테이너를 만들 수 있습니다.

```azurecli
az storage container create -n <container_name>
```

선택적인 `--public-access` 인수를 지정하면 새 컨테이너에 대한 읽기 액세스의 다음 세 가지 수준 중 하나를 설정할 수 있습니다.

* `off`(기본값): 컨테이너 데이터가 계정 소유자 전용입니다.
* `blob`: Blob에 대한 공용 읽기 액세스입니다.
* `container`: 전체 컨테이너에 대한 공용 읽기 및 목록 액세스입니다.

자세한 내용은 [컨테이너 및 Blob에 대한 익명 읽기 권한 관리](storage-manage-access-to-resources.md)를 참조하세요.

### <a name="upload-a-blob-to-a-container"></a>컨테이너에 Blob 업로드
Azure Blob 저장소는 블록 Blob, 추가 Blob 및 페이지 Blob을 지원합니다. `blob upload` 명령을 사용하여 컨테이너에 Blob을 업로드합니다.

```azurecli
az storage blob upload -f <local_file_path> -c <container_name> -n <blob_name>
```

 기본적으로 `blob upload` 명령은 페이지 Blob에 *.vhd 파일을 업로드하며, 그렇지 않으면 블록 Blob에 업로드합니다. Blob을 업로드할 때 다른 유형을 지정하려면 `--type` 인수를 사용할 수 있으며, 허용되는 값은 `append`, `block` 및 `page`입니다.

 Blob에 대한 자세한 내용은 [블록 Blob, 추가 Blob 및 페이지 Blob 이해](/rest/api/storageservices/fileservices/Understanding-Block-Blobs--Append-Blobs--and-Page-Blobs)를 참조하세요.

### <a name="download-blobs-from-a-container"></a>컨테이너에서 Blob 다운로드
다음 예제에서는 컨테이너에서 Blob을 다운로드하는 방법을 보여 줍니다.

```azurecli
az storage blob download -c mycontainer -n myblob.png -f ~/mydownloadedblob.png
```

### <a name="copy-blobs"></a>Blob 복사
저장소 계정 및 지역 내 또는 전체에 걸쳐 비동기적으로 Blob을 복사할 수 있습니다.

다음 예제에서는 한 저장소 계정에서 다른 계정으로 Blob을 복사하는 방법을 보여줍니다. 먼저 다른 계정에 컨테이너를 만들고, 해당 Blob을 공개적으로 익명으로 액세스할 수 있도록 지정합니다. 그런 다음 컨테이너에 파일을 업로드하고, 마지막으로 해당 컨테이너의 Blob을 현재 계정의 **mycontainer** 컨테이너에 복사합니다.

```azurecli
az storage container create -n mycontainer2 --account-name <accountName2> --account-key <accountKey2> --public-access blob

az storage blob upload -f ~/Images/HelloWorld.png -c mycontainer2 -n myBlockBlob2 --account-name <accountName2> --account-key <accountKey2>

az storage blob copy start -u https://<accountname2>.blob.core.windows.net/mycontainer2/myBlockBlob2 -b myBlobBlob -c mycontainer
```

원본 Blob URL(`-u`로 지정됨)은 공개적으로 액세스할 수 있거나 SAS(공유 액세스 서명) 토큰을 포함해야 합니다.

### <a name="delete-a-blob"></a>Blob 삭제
Blob을 삭제하려면 `blob delete` 명령을 사용합니다.

```azurecli
az storage blob delete -c <container_name> -n <blob_name>
```

## <a name="create-and-manage-file-shares"></a>파일 공유 만들기 및 관리
Azure File Storage는 SMB(서버 메시지 블록) 프로토콜을 사용하는 응용 프로그램을 위한 공유 저장소를 제공합니다. Microsoft Azure 가상 컴퓨터 및 클라우드 서비스 그리고 온-프레미스 응용 프로그램은 탑재된 공유를 통해 파일 데이터를 공유할 수 있습니다. Azure CLI를 통해 파일 공유 및 파일 데이터를 관리할 수 있습니다. Azure File Storage에 대한 자세한 내용은 [Windows에서 Azure File Storage 시작](storage-dotnet-how-to-use-files.md) 또는 [Linux에서 Azure File Storage 사용 방법](storage-how-to-use-files-linux.md)을 참조하세요.

### <a name="create-a-file-share"></a>파일 공유 만들기
Azure에서 Azure 파일 공유는 SMB 파일 공유입니다. 모든 디렉터리 및 파일을 파일 공유에서 만들어야 합니다. 계정에 포함할 수 있는 공유 수에는 제한이 없으며, 공유에 저장할 수 있는 파일 수에는 저장소 계정의 최대 용량 한도까지 제한이 없습니다. 다음 예제에서는 **myshare**라는 파일 공유를 만듭니다.

```azurecli
az storage share create -n myshare
```

### <a name="create-a-directory"></a>디렉터리 만들기
디렉터리는 Azure 파일 공유에 대한 선택적 계층적 구조를 제공합니다. 다음 예에서는 파일 공유에 **myDir** 이라는 디렉터리를 만듭니다.

```azurecli
az storage directory create -n myDir -s myshare
```

해당 디렉터리 경로에 여러 수준이 포함 될 수 있습니다( *예:***a/b**). 그러나 모든 부모 디렉터리가 존재하는지 확인해야 합니다. 예를 들어, 경로 **a/b**의 경우, 먼저 디렉터리 **a**를 만든 다음, **b** 디렉터리를 만듭니다.

### <a name="upload-a-local-file-to-a-share"></a>공유에 로컬 파일 업로드
다음 예제에서는 **~/temp/samplefile.txt**에서 **myshare** 파일 공유의 루트로 파일을 업로드합니다. `--source` 인수는 업로드할 기존 로컬 파일을 지정합니다.

```azurecli
az storage file upload --share-name myshare --source ~/temp/samplefile.txt
```

디렉터리를 만드는 것과 마찬가지로 다음과 같이 공유 내의 디렉터리 경로를 지정하여 공유 내의 기존 디렉터리에 파일을 업로드할 수 있습니다.

```azurecli
az storage file upload --share-name myshare/myDir --source ~/temp/samplefile.txt
```

공유에 있는 파일의 크기는 각각 최대 1TB일 수 있습니다.

### <a name="list-the-files-in-a-share"></a>공유에 있는 파일 나열
`az storage file list` 명령을 사용하여 공유에 있는 파일과 디렉터리를 나열할 수 있습니다.

```azurecli
# List the files in the root of a share
az storage file list -s myshare

# List the files in a directory within a share
az storage file list -s myshare/myDir

# List the files in a path within a share
az storage file list -s myshare -p myDir/mySubDir/MySubDir2
```

### <a name="copy-files"></a>파일 복사        
파일을 다른 파일로, 파일을 Blob으로 또는 Blob을 파일로 복사할 수 있습니다. 예를 들어 파일을 다른 공유의 디렉터리에 복사하려면 다음과 같이 수행합니다.        
        
```azurecli
az storage file copy start \
--source-share share1 --source-path dir1/file.txt \
--destination-share share2 --destination-path dir2/file.txt        
```

## <a name="next-steps"></a>다음 단계
Azure CLI 2.0을 사용하는 방법에 대해 자세히 알아볼 수 있는 몇 가지 추가 리소스는 다음과 같습니다.

* [Azure CLI 2.0 시작](https://docs.microsoft.com/cli/azure/get-started-with-az-cli2)
* [Azure CLI 2.0 명령 참조](/cli/azure)
* [GitHub의 Azure CLI 2.0](https://github.com/Azure/azure-cli)

