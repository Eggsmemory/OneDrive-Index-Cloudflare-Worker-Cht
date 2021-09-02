<!--
 * @Author: your name
 * @Date: 2020-05-05 14:09:52
 * @LastEditTime: 2020-05-05 17:18:57
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: \OneDrive-Index-Cloudflare-Worker\readme.md
 -->

中文 | [English](readme.en.md)
--- 
# OneDrive Index ( Cloudflare Worker ) 

## 🌈 演示地址

[storage.idx0.workers.dev](https://storage.idx0.workers.dev)

## 咋用

1. 去這里新建一個 APP https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade 
   `redirect_uri` 設置成 `https://eggsmemory.github.io/tools/microsoft-graph-api-auth` 。

2. 在 `Certificates & secrets` 面板創建一個新的 `secret`。

3. 在 `API permissions` 面板， 添加以下權限 `offline_access, Files.Read, Files.Read.All`。（此權限可以在Microsoft Graph中找到）

4. 使用這個工具 [microsoft-graph-api-auth](https://eggsmemory.github.io/tools/microsoft-graph-api-auth) 獲取 `refresh_token` 參數。

5. 在 `Cloudflare Worker` 管理頁面創建一個新的 `Worker` ,粘貼 `index.js` 中的代碼並替換相關參數。

*6. 訪問密碼設置（默認關閉）：

```
const AUTH_ENABLED = true
const NAME = "admin"
const PASS = "password"
```

## 🔥 新特性 V1.1

### ⏬ 中轉下載 
利用 `Cloudflare` 服務器中轉 `OneDrive` 中文件的下載，以提高中國大陸的下載體驗。已知問題，無法顯示下載進度。

在配置中開啟 `proxyDownload` 功能，在文件直鏈路徑後面加 `?proxied` 即可開啟，例如：
https://storage.idx0.workers.dev/Other/zero_file?proxied

( Cloudflare 的速度也挺隨緣的... )

### ☁️ 緩存功能
利用 `Cloudflare CDN` 來緩存 `OneDrive` 中文件，目前有兩種緩存模式：
- 整個文件緩存： 文件會先完整傳輸到 `Cloudflare` 的服務器後再返回給客戶端。文件太大可能超過 `Cloudflare Worker` 限制的單次請求運行時間。
- chunk 緩存： 流式傳輸與緩存，無法正確顯示 `Content-Length`。

在配置中開啟 `cache` 功能，可以配置兩種緩存模式的選擇以及啟用緩存的路徑地址。

### ⏫ 小文件上傳
可以利用這個工具直接上傳小文件到 `OneDrive` 上 ( 小於 4MB ，OneDrive API 的限制，比這個大就得創建 upload session 反正很麻煩 )

在配置中開啟 `upload` 功能，並設置一個密鑰 `key` ( 防止遊客上傳文件 )。

比如： 
```
POST https://storage.idx0.workers.dev/Images/?upload=<filename>&key=<key>
```

**注意：開啟該功能需要 `Files.ReadWrite` 權限**

### 🖼️ 縮略圖
對於圖片文件，可以直接獲取不同尺寸的縮略圖。
比如：https://storage.idx0.workers.dev/Images/public-md-image-20191010113652775.png?thumbnail=mediumSquare

![](https://storage.idx0.workers.dev/Images/public-md-image-20191010113652775.png?thumbnail=mediumSquare)

可用的取值參見：https://docs.microsoft.com/en-us/onedrive/developer/rest-api/api/driveitem_list_thumbnails?view=odsp-graph-online#size-options


### 👍 沒錯，這就是個好用的部落格圖床！

同時開啟**緩存功能**和**小文件上傳功能**後，這就是個自建圖床。
配合**縮略圖**功能，亦可提升博客頁面在不同場景下的加載體驗。

例如 https://blog.idx0.dev 在首頁文章列表配圖使使用了 `large` 尺寸的縮略圖，在側欄文章列表中使用了 `smallSquare` 尺寸的縮略圖。
