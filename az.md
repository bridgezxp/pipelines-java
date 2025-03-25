# infra

## login

powershell

```powershell
# azureProfile.json clouds.config PSConfig.json 
Connect-AzAccount -EnvironmentName AzureChinaCloud

Install-Module -Name Az -Scope CurrentUser -Repository PSGallery -Force
Import-Module Az
```

**bash**

适用于Linux的Windows子系统 （[Windows Subsystem for Linux](https://zhida.zhihu.com/search?content_id=252464055&content_type=Article&match_order=1&q=Windows+Subsystem+for+Linux&zhida_source=entity)， WSL ）\

****程序**> 适用于 Linux 的 Windows 子系统** ”与“**虚拟机平台**

再下载ubuntu， 重启即可

?wsl --install

[在 Windows 上安装 Azure CLI](https://learn.microsoft.com/zh-cn/cli/azure/install-azure-cli-windows?pivots=msi)

```bash
az cloud set --name AzureChinaCloud
Switched active cloud to 'AzureChinaCloud'.
Use 'az login' to log in to this cloud.
Use 'az account set' to set the active subscription.

az login --allow-no-subscriptions
```

## Cosmos DB

Azure Cosmos DB is a fully managed NoSQL and relational database service for building scalable, high performance applications.

一种全局分布式数据库系统，使你能够从数据库的本地副本读取和写入数据，且它以透明方式将数据复制到与 Cosmos 帐户关联的所有区域。

Azure Cosmos DB 是一个完全托管的 NoSQL 数据库，旨在提供低延迟、吞吐量的弹性可伸缩性、定义良好的数据一致性语义和高可用性。

可将数据库配置为全局分布，并使其可在任何 Azure 区域中使用。

## storage

在将数据保存到云时，Azure 存储会使用服务端加密 (SSE) 自动对数据加密。

Azure 存储了提供四项可以使用 Azure 存储帐户访问的数据服务：

* Azure Blob 存储（容器）：一种适用于文本和二进制数据的可大规模缩放的对象存储。
* **Azure 文件** ：适用于云或本地部署的托管文件共享。
* **Azure 队列存储** ：用于在应用程序组件之间进行可靠的消息传送的消息传送存储。
* **Azure 表存储** ：存储非关系结构化数据（也称为结构化 NoSQL 数据）的服务。

Azure 存储加密会针对所有新的和现有的存储帐户启用，并且不能禁用。

使用云分层可以在本地服务器上缓存经常访问的文件。 根据所创建的策略，不常访问的文件会分层或存档到 Azure 文件共享。

```bash
sudo mount -t cifs //dsffds.file.core.windows.net/sdagfd /mnt/mystoragefileshare -o vers=3.0,username=<storage-account-name>,password=<your-key>,sec=ntlmssp



```

则 Blob 存储的默认终结点为：

```bash
http://mystorageaccount.blob.core.windows.net
容器的 URI 类似于：
https://myaccount.blob.core.windows.net/mycontainer
Blob 的 URI 类似于：
https://myaccount.blob.core.windows.net/mycontainer/myblob 
or:
https://myaccount.blob.core.windows.net/mycontainer/myvirtualdirectory/myblob
```

数据集具有独特的生命周期。 在生命周期的早期，人们经常访问某些数据。 但随着数据的老化，访问需求急剧下降。 有些数据在云中保持空闲状态，并且在存储后很少被访问。 有些数据在创建后的数日或者数月即会过期，还有一些数据集在其整个生存期会频繁受到读取和修改。

* **热存储层** ：该联机层经过优化，可存储经常访问的数据。
* **冷存储层** ：该联机层经过优化，可存储不经常访问且存储至少 30 天的数据。
* **寒存储层** ：该联机层经过优化，可存储不经常访问且存储至少 90 天的数据。 与冷层相比，寒层的存储成本较低，访问成本较高。
* **存档存储层** ：该联机层经过优化，可存储极少访问、存储至少 180 天且延迟要求（以小时计）不严格的数据。当 Blob 位于存档访问层时，它被视为处于脱机状态，无法读取或修改。 若要读取或修改存档 Blob 中的数据，须先将 Blob 解除冻结到联机层，即热层或冷层。可复制或更改。

```bash
export rgName=eShopOnWeb-test
az configure --defaults location=centralus
export location=centralus
az group create --name $rgName
saName=mysa0217
az storage account create --resource-group $rgName --name $saName --location $location --sku Standard_LRS

# 创建存储访问策略的示例
az storage container policy create --name <stored access policy identifier> --container-name <container name> --start <start time UTC datetime> --expiry <expiry time UTC datetime> --permissions <(a)dd, (c)reate, (d)elete, (l)ist, (r)ead, or (w)rite> --account-key <storage account key> --account-name <storage account name> 
```

# app

## Create a static HTML web app

```ba
export rgName=eShopOnWeb-test
az configure --defaults location=centralus
export location=centralus
az group create --name $rgName
```

```bash
mkdir htmlapp
cd htmlapp
git clone https://github.com/Azure-Samples/html-docs-hello-world.git
#rgName=$(az group list --query "[].{id:name}" -o tsv)
appName=az204app0217

cd html-docs-hello-world
az webapp up -g $rgName -n $appName --html
{
  "URL": "http://az204app0217.azurewebsites.net",
  "appserviceplan": "down.dlu_asp_3237",
  "location": "centralus",
  "name": "az204app0217",
  "os": "Windows",
  "resourcegroup": "eShopOnWeb-test",
  "runtime_version": "-",
  "runtime_version_detected": "-",
  "sku": "FREE",
  "src_path": "//home//down//htmlapp//html-docs-hello-world"
}


#modify index.html and then re-deploy
az webapp up -g $rgName -n $appName --html

```

**access log file**

* Linux/容器应用：`https://<app-name>.scm.azurewebsites.net/api/logs/docker/zip`
* Windows 应用：`https://<app-name>.scm.azurewebsites.net/api/dump`

## slot

https://learn.microsoft.com/zh-cn/training/modules/understand-app-service-deployment-slots/4-swap-deployment-slots

Deployment slots are live apps with their own hostnames. App content and configurations elements can be swapped between two deployment slots, including the production slot.

每种应用服务计划层支持不同数量的部署槽

staging

https://az204app0217.azurewebsites.net/?x-ms-routing-name=staging

production

https://az204app0217.azurewebsites.net/?x-ms-routing-name=self

## key vault

Azure Key Vault 是用于存储应用程序机密（如加密密钥、证书和服务器端令牌）的集中式云服务。 Key Vault 通过将应用程序密钥保存在一个中心位置，并提供安全访问、权限控制和访问日志记录，来帮助控制这些密钥。

Azure Key Vault 中有三个主要的概念：保管库、密钥和机密。:  *vaults* ,  *keys* , and  *secrets* .

https://learn.microsoft.com/zh-cn/training/modules/configure-and-manage-azure-key-vault/4-store-secrets-in-akv

```bash
export rgName=eShopOnWeb-test
az configure --defaults location=eastus
export location=eastus
az group create --name $rgName
```

**Add a secret, and grant permission to create secret**

```bash
export kvName=eShopOnWeb-kv0217
az keyvault create --resource-group $rgName --name $kvName
az keyvault secret set --name SecretPassword --value just_for_test --vault-name $kvName
az keyvault secret show --name SecretPassword --vault-name $kvName

down [ ~ ]$ # 获取当前用户的租户ID
tenant_id=$(az account show --query tenantId -o tsv)

# 使用AAD Graph API获取当前用户的对象ID
user_id=$(az rest --method GET --uri "https://graph.microsoft.com/v1.0/me" --headers '{"Authorization": "Bearer '"$(az account get-access-token --resource-type ms-graph --query accessToken -o tsv)"'"}' --query id -o tsv)

echo "Current User ID (Object ID): $user_id"

Current User ID (Object ID): 00cd02da-0a8e-40d9-b60a-bed24594553d

az role assignment create \
  --assignee-object-id 00cd02da-0a8e-40d9-b60a-bed24594553d \
  --role "Key Vault Secrets Officer" \
  --scope "/subscriptions/41a65feb-3aea-4e29-8177-719b70c2407e/resourceGroups/eShopOnWeb-test/providers/Microsoft.KeyVault/vaults/eShopOnWeb-kv0217"
az keyvault secret set --name SecretPassword --value just_for_test --vault-name $kvName
az keyvault secret show --vault-name $kvName  --name SecretPassword

?　？？
az role assignment create --assignee 02bef979-7e8c-46de-998d-7d5ddbb7a7b3 --role "Key Vault Secrets User" --scope "
/subscriptions/41a65feb-3aea-4e29-8177-719b70c2407e/resourceGroups/eShopOnWeb-test/providers/Microsoft.KeyVault/vaults/VaultamortDiarymxvcz"
```

Access policies not available.
The access configuration for this key vault is set to role-based access control. To add or manage your access policies, go to the Access control (IAM) page.

RBAC

#### Deploy nodejs app

##### prepare service app

create service plan and service app, then add conf and grant access to keyvault

```bash
export spName=keyvault-exercise-plan
export saName=eShopOnWeb-app
az appservice plan create --name $spName --sku FREE --resource-group $rgName --location centralus
az webapp create --plan $spName --runtime "node|16LTS" --resource-group $rgName --name $saName
az webapp config appsettings set --resource-group  $rgName --name $saName --settings "VaultName=$kvName" "SCM_DO_BUILD_DURING_DEPLOYMENT=true"

#　启用托管标识
$ az webapp identity assign --resource-group $rgName --name $saName
  "principalId": "2f59cb19-df7b-43e9-8661-7e3a86efa6ff",
  "tenantId": "7b96c184-569a-423f-8f01-a96340f87442",

授予对保管库的访问权限
###az keyvault set-policy --secret-permissions get list --name $kvName --object-id 2f59cb19-df7b-43e9-8661-7e3a86efa6ff
az role assignment create \
  --assignee-object-id 2f59cb19-df7b-43e9-8661-7e3a86efa6ff \
  --role "Key Vault Secrets Officer" \
  --scope "/subscriptions/41a65feb-3aea-4e29-8177-719b70c2407e/resourceGroups/eShopOnWeb-test/providers/Microsoft.KeyVault/vaults/eShopOnWeb-kv0217"

```

##### code

```bash
mkdir KeyVaultDemoApp
cd KeyVaultDemoApp
npm init -y
npm install @azure/identity @azure/keyvault-secrets express
```

vim app.js

```js
// Importing dependencies
const { DefaultAzureCredential } = require("@azure/identity");
const { SecretClient } = require("@azure/keyvault-secrets");
const express = require('express');
const app = express();

// Initialize port
const port = process.env.PORT || 3000;

// Create Vault URI from App Settings
const vaultUri = `https://${process.env.VaultName}.vault.azure.net/`;

// Map of key vault secret names to values
let vaultSecretsMap = {};

// Function to get secrets from Key Vault
const getKeyVaultSecrets = async () => {
  // Create a key vault secret client
  let secretClient = new SecretClient(vaultUri, new DefaultAzureCredential());
  try {
    // Iterate through each secret in the vault
    let listPropertiesOfSecrets = secretClient.listPropertiesOfSecrets();
    while (true) {
      let { done, value } = await listPropertiesOfSecrets.next();
      if (done) {
        break;
      }
      // Only load enabled secrets - getSecret will return an error for disabled secrets
      if (value.enabled) {
        const secret = await secretClient.getSecret(value.name);
        vaultSecretsMap[value.name] = secret.value;
      }
    }
  } catch (err) {
    console.log(err.message);
  }
};

// API endpoint to test if secrets are loaded
app.get('/api/SecretTest', (req, res) => {
  let secretName = 'SecretPassword';
  let response;
  if (secretName in vaultSecretsMap) {
    response = `Secret value: ${vaultSecretsMap[secretName]}\n\nThis is for testing only! Never output a secret to a response or anywhere else in a real app!`;
  } else {
    response = `Error: No secret named ${secretName} was found...`;
  }
  res.type('text');
  res.send(response);
});

// Initialize and start the server
(async () => {
  await getKeyVaultSecrets();  // Load secrets from Key Vault
  app.listen(port, () => {
    console.log(`Server running at http://localhost:${port}`);
  });
})().catch(err => console.log(err));


```

##### 部署

```bash
zip site.zip * -x node_modules/
# powershell
Compress-Archive -Path * -DestinationPath site.zip 

az webapp deploy --src-path site.zip --resource-group $rgName --name $saName
Deployment type: zip. To override deployment type, please specify the --type parameter. Possible values: war, jar, ear, zip, startup, script, static
Initiating deployment
Deploying from local path: site.zip
Polling the status of sync deployment. Start Time: 2025-03-10 02:26:58.144124+00:00 UTC
Deployment has completed successfully
You can visit your app at: http://eshoponweb-app.azurewebsites.net
{
  "active": true,
  "author": "N/A",
  "author_email": "N/A",
  "complete": true,
  "deployer": "OneDeploy",
  "end_time": "2025-03-10T02:26:57.8730302Z",
  "id": "68139f3d60e1404eb870c71ef4e9dc5b",
  "is_readonly": true,
  "is_temp": false,
  "last_success_end_time": "2025-03-10T02:26:57.8730302Z",
  "log_url": "https://eshoponweb-app.scm.azurewebsites.net/api/deployments/latest/log",
  "message": "OneDeploy",
  "progress": "",
  "provisioningState": "Succeeded",
  "received_time": "2025-03-10T02:26:00.4138722Z",
  "site_name": "eShopOnWeb-app",
  "start_time": "2025-03-10T02:26:00.5544994Z",
  "status": 4,
  "status_text": "",
  "url": "https://eshoponweb-app.scm.azurewebsites.net/api/deployments/latest"
}
```

##### test

curl https://$saName.azurewebsites.net/api/SecretTest --insecure

**certificate Azure 应用服务集成**

1. 在“设置”下选择“TLS/SSL 设置”。
2. 选择“私钥证书 (.pfx)”选项卡。
3. 选择“导入密钥保管库证书”

## DevOps

### 使用 Azure DevOps 部署应用程序

#### 在 Azure 管道中创建发布管道

**将 Web 应用程序部署到 Azure 应用服务**

project: Space Game - web - Release

repo:　https://github.com/MicrosoftDocs/mslearn-tailspin-spacegame-web-deploy

branch: release-pipeline

创建应用服务实例

```bash
RG=eShopOnWeb-test
az configure --defaults location=eastasia
export location=eastasia
az group create --name $RG
ASP=app-svc-plan
az appservice plan create --name $ASP --resource-group $RG --sku B1 --is-linux
az appservice plan list

WA=$RG-WA
az webapp create --resource-group $RG --name $WA --plan $ASP --runtime "DOTNETCORE:6.0"


```

**配置service connection**

```bash
# 安装 Azure DevOps CLI 扩展
az extension add --name azure-devops
az login
# 创建服务主体,这会返回服务主体的信息，包括 appId（客户端ID），password（客户端密钥）和 tenant（租户ID）
az ad sp create-for-rbac --name mySPN --role contributor --scopes /subscriptions/41a65feb-3aea-4e29-8177-719b70c2407e/resourceGroups/eShopOnWeb-test
{
  "appId": "4a16087a-ca11-4ce9-8473-be12fde4e196",
  "displayName": "mySPN",
  "password": "VZY8Q~y2WtJLsDKgMTnRlesiJYflnMVBVJlihcwB",
  "tenant": "7b96c184-569a-423f-8f01-a96340f87442"
}

# 设置 Azure DevOps 组织的详细信息
az devops configure --defaults organization=https://dev.azure.com/downdlu

# 创建服务连接
az devops service-connection create \
  --name "mySC" \
  --type azurerm \
  --service-endpoint-type azureResourceManager \
  --azure-subscription 41a65feb-3aea-4e29-8177-719b70c2407e \
  --azure-resource-group eShopOnWeb-test \
  --service-principal-id 4a16087a-ca11-4ce9-8473-be12fde4e196 \
  --service-principal-key VZY8Q~y2WtJLsDKgMTnRlesiJYflnMVBVJlihcwB \
  --tenant-id 7b96c184-569a-423f-8f01-a96340f87442
# 
```

create an ARM Service Connection：　mySC

create **dev** env

variable group: **Release *WebAppName* eShopOnWeb-test-WA**

https://learn.microsoft.com/zh-cn/training/modules/create-release-pipeline/5-deploy-to-appservice

#### 使用 Azure Pipelines 创建多阶段管道

创建应用服务实例

```bash
RG=eShopOnWeb-test
export location=eastasia
az configure --defaults location=$location
az group create --name $RG

```

app service plan 创建一个名为 tailspin-space-game-asp 的应用服务计划

```bash
ASP=app-svc-plan
az appservice plan create --name $ASP --resource-group $RG --sku B1 --is-linux
az appservice plan list

webappsuffix=$RANDOM
WA=$RG-WA
az webapp create --resource-group $RG --name $WA-dev --plan $ASP --runtime "DOTNETCORE:6.0"
az webapp create --resource-group $RG --name $WA-test --plan $ASP --runtime "DOTNETCORE:6.0"
az webapp create --resource-group $RG --name $WA-staging --plan $ASP --runtime "DOTNETCORE:6.0"
# check
az webapp list --resource-group $RG --query "[].{hostName: defaultHostName, state: state}" --output table
```

project: Space Game - web - Multistage

repo:　https://github.com/MicrosoftDocs/mslearn-tailspin-spacegame-web-deploy.git

branch: release-pipeline

创建环境  dev test staging

将 Web 应用名称存储在管道变量中

Release WebAppName tailspin-space-game

ServiceConnection: mySC

need to change condition of the dev deploy

releaseBranchName: 'release-pipeline'

https://learn.microsoft.com/zh-cn/training/modules/create-multi-stage-pipeline/6-promote-staging

#### [部署模式](https://learn.microsoft.com/zh-cn/training/modules/manage-release-cadence/)

A *deployment pattern* is an automated way to smoothly roll out new application features to your users. An appropriate deployment pattern helps you minimize downtime. Some patterns also enable you to roll out new features progressively. That way, you can validate new features with select users before you make those features available to everyone.

Because we're using Azure App Service, we can take advantage of  *deployment slots* . Deployment slots are running apps that have their own host names.

I know we're not yet ready to deploy the *Space Game* website to production as part of the automated pipeline. But as a test, we can add a deployment slot to our **staging** environment.

Instead of setting up a load balancer or a router, we can just add a second slot to the App Service instance that we use in our existing *Staging* environment. We can call the primary slot *blue* and the secondary slot  *green* .

![Diagram of applications swapping IP addresses.](https://learn.microsoft.com/en-us/training/azure-devops/manage-release-cadence/media/2-zero-downtime-deployment.png)

This way we can deploy new features without any downtime. We swap an application and its configuration between the two deployment slots. Basically we're swapping the IP addresses of the two slots.

You might call this variation of a blue-green deployment a *zero-downtime deployment*

创建应用服务实例

```bash
rgName=eShopOnWeb-test
export Location=eastasia
az configure --defaults location=$Location
az group create --name $rgName

```

app service plan 创建一个名为 tailspin-space-game-asp 的应用服务计划

```bash

ASP=app-svc-plan
az appservice plan create --name $ASP-test --resource-group $rgName --sku B1 --is-linux
az appservice plan create --name $ASP-prod --resource-group $rgName --sku P1V2 --is-linux
az appservice plan list --output table

webappsuffix=$RANDOM
suf=WA-0217
WA=$rgName-$suf
az webapp create --name $WA-dev --resource-group $rgName --plan $ASP-test --runtime "DOTNET|6.0"
az webapp create --name $WA-test --resource-group $rgName --plan $ASP-test --runtime "DOTNET|6.0"
az webapp create --name $WA-staging --resource-group $rgName --plan $ASP-prod --runtime "DOTNET|6.0"
# check
az webapp list --resource-group $rgName --query "[].{hostName: defaultHostName, state: state}" --output table
```

```bash

staging=$(az webapp list --resource-group $rgName --query "[?contains(@.name, 'staging')].{name: name}" --output tsv)
echo $staging
eShopOnWeb-test-WA-0217-staging

# 创建与staging应用相关的“swap”槽
az webapp deployment slot create --name $staging --resource-group $rgName --slot swap


#列出部署槽位的主机名, 供稍后使用
az webapp deployment slot list --name $staging --resource-group $rgName --query [].hostNames --output tsv
eshoponweb-test-wa-0217-staging-swap.azurewebsites.net
```

project: Space Game - web - Deployment patterns

repo: https://github.com/MicrosoftDocs/mslearn-tailspin-spacegame-web-deploy.git

branch: release-pipeline

**创建环境**  dev test staging

**将 Web 应用名称存储在管道变量中**

Release

WebAppNameDev eshoponweb-test-wa-0217-dev
WebAppNameTest eshoponweb-test-wa-0217-test
WebAppNameStaging eshoponweb-test-wa-0217-staging

**ServiceConnection**

 mySC

need to change condition of the dev deploy

releaseBranchName: 'blue-green'

change the resourcegroup and service name in pipeline.yaml

这两步任务共同实现了  **蓝绿部署** ：

1. **先将新版本部署到 `swap` 槽** （而不是直接影响生产环境）。
2. **然后执行槽交换** (`swap → production`)，让新版本无缝替换旧版本。

##### update the content

Tailspin.SpaceGame.Web/Views/Home 目录中打开 Index.cshtml。

az webapp deployment slot swap  --resource-group eshoponweb-test --name eshoponweb-test-wa-0217-staging  --slot production   --target-slot swap

#### [通过 Azure Pipelines 实现 Azure Functions 部署自动化](https://learn.microsoft.com/zh-cn/training/modules/deploy-azure-functions/)

Azure Functions 是更广泛的 Azure 无服务器计算技术范围中的特定产品。 它让开发人员能够轻松地生成存在于无状态无服务器环境中的简单函数。 这些函数可由各种方法触发，例如 HTTP 请求、更改存储中的数据、接收来自队列的消息等等.

Azure Functions 是一种无服务器解决方案，用户通过它可以减少代码编写、减少需要维护的基础结构并节省成本。 无需担心部署和维护服务器，云基础结构提供保持应用程序运行所需的所有最新资源。

**Azure Functions** is a specific offering within the broader spectrum of Azure serverless computing technologies. It provides an easy way for developers to build straightforward functions that exist in a stateless, serverless environment. Functions can be triggered using various methods, such as HTTP requests, changes to data in storage, receipt of a message from a queue, and more.

Azure Function 是一种无服务器计算服务，属于 Azure 的计算平台。它允许你编写小块的代码（称为“函数”），然后将这些代码托管在 Azure 上，按需执行。这意味着你只需为代码实际执行的时间付费，而不需要像传统服务器那样为预先配置的资源付费。

Azure Functions 的底层架构实际上是基于 **容器** 的，而不是传统的虚拟机（VM）。

我们将创建一个简单的 Azure Function，它响应 HTTP 请求，并返回一个欢迎消息。

![1741771371415](image/az/1741771371415.png)![1740972613614](image/terms/1740972613614.png)

**Create function in Azure portal**

Runtime stack: Node.js 20 LTS

Basic authentication: Disable

Hosting options and plans: Functions premium

create a function:

HTTP trigger: A function that will be run whenever it receives an HTTP request, responding based on data in the body or query string

Authorization level: anonymous

Authorization level controls whether the function requires an API key and which key to use; Function uses a function key; Admin uses your master key. The function and master keys are found in the 'keys' management panel on the portal, when your function is selected. For user-based authentication, go to Function App Settings.

```javascript
module.exports = async function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');

    const name = (req.query.name || (req.body && req.body.name));
    const responseMessage = name
        ? "Hello, " + name + ". This HTTP triggered function executed successfully."
        : "This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.";

    context.res = {
        // status: 200, /* Defaults to 200 */
        body: responseMessage
    };
}
```

https://funcapp1-dyghfsh6h7gqbwb4.canadacentral-01.azurewebsites.net/api/HttpTrigger1?name=bridge

https://eshoponwebzxp.azurewebsites.net/api/HttpTrigger1?code=MWyelrUuocBI1r7rPkWfJjv9r0zhzr3DoWfzcUAo4Sb0AzFuIWoi-g==**&name=bridge**

创建应用服务实例

```bash
RG=eShopOnWeb-test
export location=eastasia
az configure --defaults location=$location
az group create --name $RG

```

app service plan 创建一个名为 tailspin-space-game-asp 的应用服务计划

```bash
ASP=app-svc-plan
az appservice plan create --name $ASP --resource-group $RG --sku B1 --is-linux
az appservice plan list

suf=$RANDOM
suf=0217
WA=$RG-$suf
# to check the available versions
az webapp list-runtimes --os-type Linux  
az webapp create --name $WA --resource-group $RG --plan $ASP --runtime "DOTNET|6.0"
az webapp list --resource-group $RG --query "[].{hostName: defaultHostName, state: state}" --output table

#Azure Functions 需要一个存储帐户来进行部署
saName="eshoponwebsa${suf}"
az storage account create --name $saName --resource-group $RG --sku Standard_LRS
#创建 Azure Functions 应用示例
leaderboardName="${RG}-SA-${suf}-leaderboard"
az functionapp create --name $leaderboardName --resource-group $RG --storage-account $saName --functions-version 4 --consumption-plan-location eastasia

az functionapp list --resource-group $RG --query "[].{hostName: defaultHostName, state: state}" --output table
HostName                                                State
------------------------------------------------------  -------
tailspin-space-game-leaderboard-0217.azurewebsites.net  Running

```

project: Space Game - web - Azure Functions

repo: https://github.com/MicrosoftDocs/mslearn-tailspin-spacegame-web-deploy.git

branch: release-pipeline

project: Space Game - web - Azure Functions

https://github.com/MicrosoftDocs/mslearn-tailspin-spacegame-web-azure-functions.git

create **spike** environment

**Library** Release

WebAppName: eshoponweb-test-wa-0217
LeaderboardAppName: eShopOnWeb-test-SA-0217-leaderboard
ResourceGroupName: eShopOnWeb-test

**service connection** arm-sc

#### [执行 Docker 容器部署 ](https://learn.microsoft.com/zh-cn/training/modules/deploy-docker/)

##### ACR

```bash
export RG=eShopOnWeb-test
export Location=eastasia
az configure --defaults location=$Location

az group create --name $RG

az acr create --resource-group $rgName --name myacr0217 --sku Basic
echo FROM mcr.microsoft.com/hello-world > Dockerfile
az acr build --image sample/hello-world:v1 --registry myacr0217 --file Dockerfile .

az acr repository list --name myacr0217 --output table
az acr repository show-tags --name myacr0217 --repository sample/hello-world --output table
az acr run --registry myacr0217 --cmd '$Registry/sample/hello-world:v1' /dev/null
```

deploy a container

```bash
DNS_NAME_LABEL=aci-example-$RANDOM
DNS_NAME_LABEL=aci-example-0217
down [ ~ ]$ az provider register --namespace Microsoft.ContainerInstance
Registering is still on-going. You can monitor using 'az provider show -n Microsoft.ContainerInstance'
az container create --resource-group $rgName \
    --name mycontainer \
    --image mcr.microsoft.com/azuredocs/aci-helloworld \
    --ports 80 --os-type Linux --cpu 1 --memory 1 \
    --dns-name-label $DNS_NAME_LABEL --location $location
az container show --resource-group $rgName \
    --name mycontainer \
    --query "{FQDN:ipAddress.fqdn,ProvisioningState:provisioningState}" \
    --out table

az container create \
    --resource-group  $rgName \
    --name mycontainer2 \
    --image mcr.microsoft.com/azuredocs/aci-wordcount:latest \
    --os-type Linux --cpu 1 --memory 1 \
    --restart-policy OnFailure \
    --environment-variables 'NumWords'='5' 'MinLength'='8'
```

**env variables need test**

```yaml
cat > secure-env.yaml << EOF
apiVersion: 2018-10-01
location: eastus
name: securetest
properties:
  containers:
  - name: mycontainer
    properties:
      environmentVariables:
        - name: 'NOTSECRET'
          value: 'my-exposed-value'
        - name: 'SECRET'
          secureValue: 'my-secret-value'
      image: nginx
      ports: []
      resources:
        requests:
          cpu: 1.0
          memoryInGB: 1.5
  osType: Linux
  restartPolicy: Always
tags: null
type: Microsoft.ContainerInstance/containerGroups
EOF
az container create --resource-group $rgName --file secure-env.yaml 
```

Azure 容器应用是一种无服务器容器服务，支持微服务应用程序和强大的自动缩放功能，而无需管理复杂的基础结构。

```bash
az extension add --name containerapp --upgrade
az provider register --namespace Microsoft.App
az provider register --namespace Microsoft.OperationalInsights
myRG=eShopOnWeb-test
myLocation=centralus
myAppContEnv=az204-env-$RANDOM
myAppContEnv=az204-env-0217
#创建环境
az containerapp env create \
    --name $myAppContEnv \
    --resource-group $myRG \
    --location $myLocation
＃应用环境完成部署后，可以部署到 Azure 容器应用。
az containerapp create \
    --name my-container-app \
    --resource-group $myRG \
    --environment $myAppContEnv \
    --image mcr.microsoft.com/azuredocs/containerapps-helloworld:latest \
    --target-port 80 \
    --ingress 'external' \
    --query properties.configuration.ingress.fqdn
＃通过将 --ingress 设置为 external，可以将容器应用提供给公共请求。 该命令返回用于访问应用的链接。
```

upgrade container app

```bash
az containerapp update \
  --name my-container-app \
  --resource-group $myRG \
  --image httpd
az containerapp revision list \
  --name my-container-app \
  --resource-group $myRG \
  -o table
```

!!!!!!

##### 创建应用服务实例

```bash
RG=eShopOnWeb-test
export location=eastasia
az configure --defaults location=$location
az group create --name $RG

```

app service plan 创建一个名为 tailspin-space-game-asp 的应用服务计划

```bash
regName=reg0217
az acr create --name $regName --resource-group $RG --sku Standard --admin-enabled true
az acr list --resource-group $RG --query "[].{loginServer: loginServer}" --output table
#reg0217.azurecr.io

ASP=app-svc-plan
az appservice plan create --name $ASP --resource-group $RG --sku B1 --is-linux
az appservice plan list

suf=$RANDOM
suf=0217
WA=$RG-$suf
# to check the available versions
az webapp list-runtimes --os-type Linux  
az webapp create --name $WA --resource-group $RG --plan $ASP --deployment-container-image-name $regName.azurecr.io/web:latest
az webapp list --resource-group $RG --query "[].{hostName: defaultHostName, state: state}" --output table



```

project: Space Game - web - Docker

repo: https://github.com/MicrosoftDocs/mslearn-tailspin-spacegame-web-docker

**Library** Release

WebAppName: eshoponweb-test-0217
RegistryName: reg0217.azurecr.io

**service connection** arm-sc acr-sc2

#### [执行多容器 Kubernetes 部署](https://learn.microsoft.com/zh-cn/training/modules/deploy-kubernetes/)

```bash
rgName=eShopOnWeb-test
export Location=eastasia
az configure --defaults location=$Location
az group create --name $rgName

```

 创建acr aks

```bash
regName=reg0217
az acr create --name $regName --resource-group $rgName --sku Standard --admin-enabled true
az acr list --resource-group $rgName --query "[].{loginServer: loginServer}" --output table
#reg0217.azurecr.io

# az aks get-versions
az aks create --name $rgName-aks --resource-group $rgName --kubernetes-version 1.31.5 --generate-ssh-keys # --enable-addons monitoring
az aks get-credentials --resource-group $rgName --name $rgName-aks
# 创建一个变量，用于存储为 AKS ACR实例配置的服务主体的 ID, 授权 AKS 群集访问 ACR
aksId=$(az aks show --resource-group $rgName --name $rgName-aks --query "identityProfile.kubeletidentity.clientId" --output tsv)

acrId=$(az acr show --name $regName --resource-group $rgName --query "id" --output tsv)

az role assignment create --assignee $aksId --scope $acrId --role AcrPull

```

project: Space Game - Web - Kubernetes

repo: https://github.com/MicrosoftDocs/mslearn-tailspin-spacegame-web-kubernetes

**Library** Release

RegistryName: reg0217.azurecr.io

**service connection** arm-sc3 acr-sc3

#### [Canary](https://learn.microsoft.com/zh-cn/azure/devops/pipelines/ecosystems/kubernetes/canary-demo?view=azure-devops&tabs=yaml)

based on the above infra

project: azure pipelines canary k8s

repo: https://github.com/MicrosoftDocs/azure-pipelines-canary-k8s.git
? branch: **azure-pipelines-canary-k8s**

**Library** Release

RegistryName: reg0217.azurecr.io

**service connection** arm-sc acr-sc

ENV:  akscanary **Kubernetes new namespace: *canarydemo
*akspromote* use the above namespace***

1. Create a new Kubernetes environment called  ***akspromote*** .
2. Open the new **akspromote** environment from the list of environments, and select **Approvals** on the **Approvals and checks** tab.
3. On the **Approvals** screen, add your own user account under  **Approvers** .
4. Expand  **Advanced** , and make sure **Allow approvers to approve their own runs** is selected.
5. Select  **Create** .

**create your pipeline**
starter pipeline

##### Add manual approval for promoting or rejecting

1. Create a new Kubernetes environment called  *akspromote* .
2. Open and select **Approvals** on the **Approvals and checks** tab.
3. On the **Approvals** screen, add your own user account under  **Approvers** .
4. Expand  **Advanced** , and make sure **Allow approvers to approve their own runs** is selected.


```bash
kubectl run client --image busybox -- sleep infinity

```

https://learn.microsoft.com/zh-cn/azure/devops/pipelines/ecosystems/kubernetes/canary-demo?view=azure-devops&tabs=yaml

https://dev.azure.com/downdlu/_git/azure%20pipelines%20canary%20k8s?path=/azure-pipelines.yml

https://portal.azure.com/#view/HubsExtension/BrowseResourceGroups.ReactView

https://blog.51cto.com/u_13236892/13082693
