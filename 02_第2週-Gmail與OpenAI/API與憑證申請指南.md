# 第 2 週：Gmail OAuth2 與 OpenAI API 憑證申請指南

本章以自架 n8n（`http://localhost:5678`）為例，完成 Gmail OAuth2 與 OpenAI API 的端到端設定。憑證只儲存在 n8n 的 Credentials 中；請勿將 API key、Client Secret、下載的 OAuth JSON 或 `.env` 提交到 Git。

## 課前檢查

- n8n 已啟動並可開啟 `http://localhost:5678`。
- 使用自己的 Google 帳號登入 [Google Cloud Console](https://console.cloud.google.com/)。
- 準備一個不含重要資料的 Gmail 測試信箱或測試標籤。
- 第一次驗證只做讀取測試；確認成功後，才啟用「加上標籤」等會修改信件的節點。

## 從零開始申請 Google Cloud Platform

Google Cloud Console 使用 Google 帳號登入；若學生還沒有 Google 帳號，請先在 [建立 Google 帳號](https://accounts.google.com/signup) 完成姓名、生日、帳號名稱與驗證。這個帳號日後就是 OAuth 同意畫面與測試 Gmail 的管理者。

### 0.1 開啟 Cloud Console 與同意條款

1. 開啟 [Google Cloud Console](https://console.cloud.google.com/)，以自己的 Google 帳號登入。
2. 初次使用時，依畫面選擇國家／地區並同意服務條款。
3. Console 可能提供免費試用或要求建立 Billing account。這門課的 Gmail API 設定通常不會直接計費，但是否需要付款方式、免費額度與可用服務以帳戶當下畫面為準；若出現費用確認，請以google 官方說明為主。
4. 不要使用他人的 Cloud 專案或 Client Secret；每位學生應使用自己的專案與自己的測試 Gmail。

### 0.2 建立課程專案

1. 在 Console 頂端點選專案選擇器，按 **新增專案**。
2. Project name 可填 `n8n-course-姓名縮寫`，Location 維持 **No organization**（個人帳號通常如此）。
3. 點選 **建立**，等待 Console 顯示專案已建立，再由專案選擇器切換到它。
4. 後續所有操作都先核對頂端顯示的專案名稱；API、OAuth 同意畫面與 OAuth 用戶端必須在同一個專案內。

> 截圖中的專案名稱與網址只是示範，請勿照抄或嘗試使用他人的專案、Client ID 或密鑰。

## Gmail OAuth2：端到端設定

### 1. 建立或選取 Google Cloud 專案

1. 開啟 [Google Cloud Console](https://console.cloud.google.com/)。
2. 點選頁面頂端的專案選擇器，建立新專案或選取課程專案。
3. 記下專案名稱，避免在錯誤的專案啟用 API 或建立用戶端。

### 2. 啟用 Gmail API

1. 左側選單選 **API 和服務 > 程式庫**。
2. 搜尋 `Gmail API`，打開結果頁。
3. 點選 **啟用**。若頁面顯示「管理」，表示 API 已啟用。
4. 回到 **API 和服務 > 已啟用的 API 和服務**，確認清單有 `Gmail API`。

畫面核對：此頁的表格列會顯示 `Gmail API`；這是後續 OAuth scope 能成功要求 Gmail 權限的前提。

### 3. 設定 OAuth 同意畫面

1. 在 Google Cloud Console 開啟 **Google Auth Platform > 品牌**，填寫應用程式名稱、使用者支援電子郵件與開發人員聯絡資訊。
2. 開啟 **Google Auth Platform > 目標對象**。
3. 個人或課堂測試通常選 **外部**；若仍是測試狀態，將實際要登入授權的 Gmail 位址加入 Test users。
4. 若應用程式已是「實際運作中」，不需另外加入 Test users，但要求敏感 Gmail 權限時 Google 仍可能顯示未驗證提示。

### 4. 建立 Web application OAuth 用戶端

1. 開啟 **Google Auth Platform > 用戶端**。
2. 選 **建立用戶端**，類型選 **網頁應用程式（Web application）**。
3. 為用戶端取名，例如 `n8n-course-local`。
4. 在 **已授權的重新導向 URI** 選 **新增 URI**。
5. 回到 n8n 的 Gmail Credential 視窗，複製它顯示的 `OAuth Redirect URL`，一字不改貼到 Google Cloud。

本機預設應為：

```
http://localhost:5678/rest/oauth2-credential/callback
```

6. 點選 **儲存**。Google Cloud 介面會提示設定可能需要數分鐘才生效；若立刻授權失敗，先等候後再試。

畫面核對：在用戶端詳情的「已授權的重新導向 URI」清單中，必須能看到上述完整網址，包含 `/rest/oauth2-credential/callback`。

![Google Cloud Console：在 OAuth 用戶端加入 n8n 回呼 URI](images/google-cloud-oauth-redirect-uri.png)

截圖重點：左側是 **已授權的重新導向 URI**，按 **新增 URI** 後貼入 n8n 顯示的網址；完成後按左下角 **儲存**。右側的密鑰區塊僅示範位置，學生不應將完整密鑰截圖、複製到作業或提交 Git。

### 5. 建立並保護 Client Secret

1. 在同一個用戶端詳情頁的 **用戶端密鑰** 區塊，選 **新增用戶端密鑰**。
2. 立即複製新密鑰，因 Google 之後不會再顯示完整內容。
3. 保留 Client ID 與 Client Secret，僅用於下一節的 n8n Credential。

> Google 允許輪換密鑰。新密鑰能正常連線後，再停用與刪除不再使用的舊密鑰；不要同時長期保留多把密鑰。

![Google Cloud Console：用戶端密鑰安全提示](images/google-cloud-client-secret-safety.png)

截圖重點：Google 只以遮罩顯示既有密鑰，遺失後無法再次查看或下載。新增密鑰後要立即安全保存，確認 n8n 已改用新密鑰後再停用舊密鑰。

### 6. 在 n8n 建立 Gmail OAuth2 Credential

1. 開啟本週的 Gmail 工作流，點選 **Gmail Trigger** 或任一 Gmail 節點。
2. 在 **Credential** 選 **Create new credential**，類型為 **Gmail OAuth2 API**。
3. 將 Google Cloud 的 Client ID 填入 **Client ID**，Client Secret 填入 **Client Secret**。
4. 先按 **Save**，再按 **Sign in with Google**。
5. 選擇要授權的 Google 帳號，閱讀權限範圍後按 **允許**。
6. 成功時瀏覽器會回到 n8n，Credential 會顯示已連線；同一工作流的 Gmail 節點可選取這一個 Credential。

畫面核對：n8n Credential 視窗會先顯示 `OAuth Redirect URL`、`Client ID`、`Client Secret`，填妥並儲存後才會出現 **Sign in with Google**。

### 7. 安全測試與排錯

1. 不要先按整個工作流的 **Execute workflow**，因範本中的 Add labels 節點會改變信件。
2. 在 Gmail Trigger 點 **Fetch Test Event**，或在 `Get all labels` 節點按 **Execute step**。兩者皆為讀取操作。
3. 看到 label 或測試事件資料即代表 Gmail OAuth2 連線成功。

| 現象 | 原因與處理 |
| --- | --- |
| `redirect_uri_mismatch` | Google Cloud 的 URI 與 n8n 顯示的 Callback URL 不完全相同。重新複製貼上並儲存。 |
| `access blocked` 或未驗證畫面 | 到 OAuth 目標對象加入測試帳號，或確認應用程式發布狀態。 |
| `Gmail API has not been used` | 在正確的 Cloud 專案啟用 Gmail API，等待設定生效後再試。 |
| Sign in 後回不到 n8n | 確認 n8n 的公開網址、`N8N_EDITOR_BASE_URL` 與 Google redirect URI 使用相同網域；本機課堂環境請使用 localhost 範例。 |

官方參考：[n8n Google OAuth2 credentials](https://docs.n8n.io/integrations/builtin/credentials/google/)、[Google OAuth 2.0 設定](https://support.google.com/googleapi/answer/6158849)。

## OpenAI API key

1. 登入 [OpenAI Platform](https://platform.openai.com/)，建立或選取課程 project，並確認該 project 有可用的 billing／額度。
2. 在 project 的 API keys 建立新 key，命名例如 `n8n-course-w2`。
3. 完整 key 只顯示一次；立即貼入 n8n 的 **Credentials > New > OpenAI**，不要存到筆記、工作流 JSON 或 Git。
4. 在 OpenAI Chat Model 節點選取此 Credential，以低成本、非敏感的分類文字測試。
5. 若 key 外洩，立刻在 Platform revoke，建立新 key 後更新 n8n Credential。

官方參考：[Project API keys](https://platform.openai.com/docs/api-reference/project-api-keys)。

## 學生交付前檢核

- [ ] Gmail API 已在正確 Cloud 專案啟用。
- [ ] Google Cloud 與 n8n 的 Redirect URL 完全一致。
- [ ] Gmail Credential 已完成 Google 授權。
- [ ] 已用讀取節點成功測試，尚未誤執行會修改所有信件的流程。
- [ ] API key、Client Secret、OAuth JSON 與 `.env` 不在 Git 或作業截圖中。
