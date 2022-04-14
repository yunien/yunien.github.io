# webapp service

## 一、使用gitAction，部署artifact至slot中
### 須在Git Repo設定secrets(github為例)
> secrets路徑：repository > Settings > Secrets > "Add a new secret"
> secrets來源：兩種方法，搭配不同的gitAction workflow寫法

#### 1. 使用service principal in Azure
Azure CLI
```
az ad sp create-for-rbac --name "myApp" --role contributor \
--scopes /subscriptions/<subscription-id>/resourceGroups/<group-name>/providers/Microsoft.Web/sites/<app-name> \
--sdk-auth
```

> output(下方json全部貼到secrets中)

```json
{
  "clientId": "bf5984d0-xxxx-xxxx-xxxx-1e8fad657f58",
  "clientSecret": "ptWGxK0MoDX~ead9837hd.qqqqqqqqqqqq",
  "subscriptionId": "286789e6-xxxx-xxxx-xxxx-c23c70a2728f",
  "tenantId": "92ff4b4f-xxxx-xxxx-xxxx-7f73aecdca0c",
  "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
  "resourceManagerEndpointUrl": "https://management.azure.com/",
  "activeDirectoryGraphResourceId": "https://graph.windows.net/",
  "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
  "galleryEndpointUrl": "https://gallery.azure.com/",
  "managementEndpointUrl": "https://management.core.windows.net/"
}
```

> gitAction workflow寫法yaml, 加上login & logout
```yaml
# 設定
name: Workflow to Azure Web App

jobs:
  deploy:
    runs-on: ubuntu-latest
    needs: build
    environment:
      name: 'production'
      url: ${{ steps.deploy-to-webapp.outputs.webapp-url }}
    
    steps:
      - uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Deploy to Azure Web App
        id: deploy-to-webapp
        uses: azure/webapps-deploy@v2
        with:
          app-name: 'appName'
          slot-name: 'production'
          package: '*.jar'
      
      # Azure logout 
      - name: logout
        run: |
          az logout
```

#### 2. 使用 publish-profile
> 下載 publish-profile(文件內容加到全部貼到secrets中)

```markdown
![image][https://github.com/yunien/yunien.github.io/blob/main/azure/app-service/getPublishProfile.png]
```

> gitAction workflow寫法yaml, azure/webapps-deploy@v2 內加上 publish-profile
```yaml
name: Workflow to Azure Web App

jobs:
  deploy:
    runs-on: ubuntu-latest
    needs: build
    environment:
      name: 'production'
      url: ${{ steps.deploy-to-webapp.outputs.webapp-url }}
    
    steps:
      - name: Download artifact from build job
        uses: actions/download-artifact@v2
        with:
          name: java-app

      - name: Deploy to Azure Web App
        id: deploy-to-webapp
        uses: azure/webapps-deploy@v2
        with:
          app-name: 'appName'
          slot-name: 'production'
          publish-profile: ${{ secrets.AZURE_CREDENTIALS }}
          package: '*.jar'

```
