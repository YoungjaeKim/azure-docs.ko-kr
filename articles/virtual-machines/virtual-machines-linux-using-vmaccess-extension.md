---
title: "VMAccess 확장 및 Azure CLI 2.0을 사용하여 액세스 다시 설정 | Microsoft Docs"
description: "VMAccess 확장 및 Azure CLI 2.0을 사용하여 사용자를 관리하고 Linux VM에 대한 액세스를 다시 설정하는 방법"
services: virtual-machines-linux
documentationcenter: 
author: vlivech
manager: timlt
editor: 
tags: azure-resource-manager
ms.assetid: 261a9646-1f93-407e-951e-0be7226b3064
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 02/16/2017
ms.author: v-livech
translationtype: Human Translation
ms.sourcegitcommit: debdb8a16c8cfd6a137bd2a7c3b82cfdbedb0d8c
ms.openlocfilehash: 4fac98d37dde195af69d8bd03fd796c6eeae3734
ms.lasthandoff: 02/27/2017


---
# <a name="manage-users-ssh-and-check-or-repair-disks-on-linux-vms-using-the-vmaccess-extension-with-the-azure-cli-20"></a>Azure CLI 2.0에서 VMAccess 확장을 사용하여 사용자, SSH 관리 및 Linux VM의 디스크 검사 또는 복구
Linux VM의 디스크에 오류가 표시되어 있습니다. 사용자가 Linux VM의 루트 암호를 재설정했거나 SSH 개인 키를 실수로 삭제했습니다. 데이터 센터를 사용할 때는 이러한 경우 데이터 센터로 직접 가서 KVM을 열어 서버 콘솔에 액세스해야 했습니다. Azure VMAccess 확장을 콘솔에 액세스하여 Linux에 대한 액세스 권한을 재설정하거나 디스크 수준 유지 관리를 수행할 수 있는 이 KVM 스위치로 생각하세요.

이 문서는 VMAccess VM 확장을 사용하여 디스크를 검사 또는 복구하거나, 사용자 액세스를 다시 설정하거나, 사용자 계정을 관리하거나, Linux의 SSHD 구성을 다시 설정하는 방법을 설명합니다. [Azure CLI 1.0](virtual-machines-linux-using-vmaccess-extension-nodejs.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)에서 이러한 단계를 수행할 수도 있습니다.


## <a name="ways-to-use-the-vmaccess-extension"></a>VMAccess 확장을 사용하는 방법
Linux VM에서 두 가지 방법으로 VMAccess 확장을 사용할 수 있습니다.

* Azure CLI 2.0 및 필수 매개 변수를 사용합니다.
* [VMAccess 확장을 처리하고 관련 작업을 수행하는 원시 JSON 파일을 사용](#use-json-files-and-the-vmaccess-extension)합니다.

다음 예제에서는 적절한 매개 변수와 함께 [az vm access](/cli/azure/vm/access)를 사용합니다. 이러한 단계를 수행하려면 최신 [Azure CLI 2.0](/cli/azure/install-az-cli2)을 설치하고 [az login](/cli/azure/#login)을 사용하여 Azure 계정에 로그인해야 합니다.

## <a name="reset-ssh-key"></a>SSH 키 다시 설정
다음 예제에서는 VM `myVM`에서 사용자 `azureuser`에 대한 SSH 키를 다시 설정합니다.

```azurecli
az vm access set-linux-user \
  --resource-group myResourceGroup \
  --name myVM \
  --username azureuser \
  --ssh-key-value ~/.ssh/id_rsa.pub
```

## <a name="reset-password"></a>암호 재설정
다음 예제에서는 VM `myVM`에서 사용자 `azureuser`에 대한 암호를 다시 설정합니다.

```azurecli
az vm access set-linux-user \
  --resource-group myResourceGroup \
  --name myVM \
  --username azureuser \
  --password myNewPassword
```

## <a name="reset-sshd"></a>SSHD 재설정
다음 예제에서는 VM `myVM`에서 SSHD 구성을 다시 설정합니다.

```azurecli
az vm access reset-linux-ssh \
  --resource-group myResourceGroup \
  --name myVM
```

## <a name="create-a-user"></a>사용자 만들기
다음 예제에서는 VM `myVM`에서 인증을 위해 SSH 키를 사용하여 사용자 `myNewUser`을 만듭니다.

```azurecli
az vm access set-linux-user \
  --resource-group myResourceGroup \
  --name myVM \
  --username myNewUser \
  --ssh-key-value ~/.ssh/id_rsa.pub
```

## <a name="deletes-a-user"></a>사용자 삭제
다음 예제에서는 VM `myVM`에서 사용자 `myNewUser`을 삭제합니다.

```azurecli
az vm access delete-linux-user \
  --resource-group myResourceGroup \
  --name myVM \
  --username myNewUser
```


## <a name="use-json-files-and-the-vmaccess-extension"></a>JSON 파일 및 VMAccess 확장 사용
다음 예제에서는 원시 JSON 파일을 사용합니다. [az vm extension set](/cli/azure/vm/extension#set)을 사용하여 JSON 파일을 호출합니다. 이러한 JSON 파일은 Azure 템플릿에서도 호출할 수 있습니다. 

### <a name="reset-user-access"></a>사용자 액세스 다시 설정
Linux VM의 루트에 액세스할 수 없게 된 경우 VMAccess 스크립트를 시작하여 사용자 암호를 다시 설정할 수 있습니다.

사용자의 SSH 키를 다시 설정하려면 파일 `reset_ssh_key.json`을 만들고 다음 콘텐츠를 추가합니다.

```json
{
  "username":"azureuser",
  "ssh_key":"ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCZ3S7gGp3rcbKmG2Y4vGZFMuMZCwoUzZNG1vHY7P2XV2x9FfAhy8iGD+lF8UdjFX3t5ebMm6BnnMh8fHwkTRdOt3LDQq8o8ElTBrZaKPxZN2thMZnODs5Hlemb2UX0oRIGRcvWqsd4oJmxsXa/Si98Wa6RHWbc9QZhw80KAcOVhmndZAZAGR+Wq6yslNo5TMOr1/ZyQAook5C4FtcSGn3Y+WczaoGWIxG4ZaWk128g79VIeJcIQqOjPodHvQAhll7qDlItVvBfMOben3GyhYTm7k4YwlEdkONm4yV/UIW0la1rmyztSBQIm9sZmSq44XXgjVmDHNF8UfCZ1ToE4r2SdwTmZv00T2i5faeYnHzxiLPA3Enub7iUo5IdwFArnqad7MO1SY1kLemhX9eFjLWN4mJe56Fu4NiWJkR9APSZQrYeKaqru4KUC68QpVasNJHbuxPSf/PcjF3cjO1+X+4x6L1H5HTPuqUkyZGgDO4ynUHbko4dhlanALcriF7tIfQR9i2r2xOyv5gxJEW/zztGqWma/d4rBoPjnf6tO7rLFHXMt/DVTkAfn5woYtLDwkn5FMyvThRmex3BDf0gujoI1y6cOWLe9Y5geNX0oj+MXg/W0cXAtzSFocstV1PoVqy883hNoeQZ3mIGB3Q0rIUm5d9MA2bMMt31m1g3Sin6EQ== azureuser@myVM"
}
```

다음을 사용하여 VMAccess 스크립트를 실행합니다.

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name VMAccessForLinux \
  --publisher Microsoft.OSTCExtensions \
  --version 1.4 \
  --protected-settings reset_ssh_key.json
```

사용자 암호를 다시 설정하려면 파일 `reset_user_password.json`을 만들고 다음 콘텐츠를 추가합니다.

```json
{
  "username":"azureuser",
  "password":"myNewPassword" 
}
```

다음을 사용하여 VMAccess 스크립트를 실행합니다.

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name VMAccessForLinux \
  --publisher Microsoft.OSTCExtensions \
  --version 1.4 \
  --protected-settings reset_user_password.json
```

### <a name="reset-ssh"></a>SSH 다시 설정
Linux VM SSHD 구성을 변경하고 변경 내용을 확인하기 전에 SSH 연결을 닫을 경우 SSH에 다시 로그인하지 못할 수 있습니다.  VMAccess를 사용하면 SSH를 통해 로그인하지 않고도 SSHD 구성을 알려진 정상 구성으로 다시 설정할 수 있습니다.

SSHD 구성을 다시 설정하려면 파일 `reset_sshd.json`을 만들고 다음 콘텐츠를 추가합니다.

```json
{
  "reset_ssh": true
}
```

다음을 사용하여 VMAccess 스크립트를 실행합니다.

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name VMAccessForLinux \
  --publisher Microsoft.OSTCExtensions \
  --version 1.4 \
  --protected-settings reset_sshd.json
```

### <a name="manage-users"></a>사용자 관리
VMAccess는 로그인하고 sudo 또는 루트 계정을 사용하지 않고 Linux VM의 사용자를 관리하는 데 사용할 수 있는 Python 스크립트입니다.

사용자를 만들려면 파일 `create_new_user.json`을 만들고 다음 콘텐츠를 추가합니다.

```json
{
  "username":"myNewUser",
  "ssh_key":"ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCZ3S7gGp3rcbKmG2Y4vGZFMuMZCwoUzZNG1vHY7P2XV2x9FfAhy8iGD+lF8UdjFX3t5ebMm6BnnMh8fHwkTRdOt3LDQq8o8ElTBrZaKPxZN2thMZnODs5Hlemb2UX0oRIGRcvWqsd4oJmxsXa/Si98Wa6RHWbc9QZhw80KAcOVhmndZAZAGR+Wq6yslNo5TMOr1/ZyQAook5C4FtcSGn3Y+WczaoGWIxG4ZaWk128g79VIeJcIQqOjPodHvQAhll7qDlItVvBfMOben3GyhYTm7k4YwlEdkONm4yV/UIW0la1rmyztSBQIm9sZmSq44XXgjVmDHNF8UfCZ1ToE4r2SdwTmZv00T2i5faeYnHzxiLPA3Enub7iUo5IdwFArnqad7MO1SY1kLemhX9eFjLWN4mJe56Fu4NiWJkR9APSZQrYeKaqru4KUC68QpVasNJHbuxPSf/PcjF3cjO1+X+4x6L1H5HTPuqUkyZGgDO4ynUHbko4dhlanALcriF7tIfQR9i2r2xOyv5gxJEW/zztGqWma/d4rBoPjnf6tO7rLFHXMt/DVTkAfn5woYtLDwkn5FMyvThRmex3BDf0gujoI1y6cOWLe9Y5geNX0oj+MXg/W0cXAtzSFocstV1PoVqy883hNoeQZ3mIGB3Q0rIUm5d9MA2bMMt31m1g3Sin6EQ== myNewUser@myVM",
  "password":"myNewUserPassword"
}
```

다음을 사용하여 VMAccess 스크립트를 실행합니다.

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name VMAccessForLinux \
  --publisher Microsoft.OSTCExtensions \
  --version 1.4 \
  --protected-settings create_new_user.json
```

사용자를 삭제하려면 파일 `delete_user.json`을 만들고 다음 콘텐츠를 추가합니다.

```json
{
  "remove_user":"myDeleteUser"
}
```

다음을 사용하여 VMAccess 스크립트를 실행합니다.

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name VMAccessForLinux \
  --publisher Microsoft.OSTCExtensions \
  --version 1.4 \
  --protected-settings delete_user.json
```

### <a name="check-or-repair-the-disk"></a>디스크 확인 또는 복구
VMAccess를 사용하면 Linux VM에 있는 디스크에 fsck를 실행할 수 있습니다. VMAccess를 사용하여 디스크 검사와 디스크 복구를 수행할 수도 있습니다.

이 VMAccess 스크립트를 사용하는 디스크를 확인화고 복구하려면 하는 파일 `disk_check_repair.json`을 만들고 다음 콘텐츠를 추가합니다.

```json
{
  "check_disk": "true",
  "repair_disk": "true, user-disk-name"
}
```

다음을 사용하여 VMAccess 스크립트를 실행합니다.

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name VMAccessForLinux \
  --publisher Microsoft.OSTCExtensions \
  --version 1.4 \
  --protected-settings disk_check_repair.json
```

## <a name="next-steps"></a>다음 단계
실행 중인 Linux VM에서 변경을 수행하는 한 가지 방법은 Azure VMAccess 확장을 사용하여 Linux를 업데이트하는 것입니다. cloud-init 및 Azure Resource Manager 템플릿 등의 도구를 사용하여 부팅 시 Linux VM을 수정할 수도 있습니다.

[가상 컴퓨터 확장 및 기능 정보](virtual-machines-linux-extensions-features.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

[Linux VM 확장을 사용하여 Azure Resource Manager 템플릿 작성](virtual-machines-linux-extensions-authoring-templates.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

[cloud-init를 사용하여 생성 중인 Linux VM 사용자 지정](virtual-machines-linux-using-cloud-init.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)


