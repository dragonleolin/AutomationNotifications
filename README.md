# [n8n 教學]在本地建立能串連不同服務的自動化工具

n8n 是款能把你從重複的例行性任務中，拯救出來的自動化工具。

使用者可以透過視覺化的介面，用拖拉節點、設定參數的方式來建立符合自己需求的工作流。

## ▋ STEP 1: 安裝 Docker

前往 Docker 官網: https://www.docker.com/

根據自己的作業系統選擇對應的版本下載。

![img](./img/init_n8n/install_docker.png)

## ▋ STEP 2: 使用 docker-compose.yml 安裝 n8n

你可以直接 git clone 筆者的 GitHub 專案，或者建立一個 `n8n` 的資料夾，新增 `docker-compose.yml` 檔案。

```yml
version: '3.8'

services:
  n8n:
    image: n8nio/n8n
    ports:
      - "5678:5678"
    env_file:
      - .env
    volumes:
      - n8n_data:/home/node/.n8n
    restart: always

  spring-app:
    build: ./spring-app
    ports:
      - "8080:8080"
    restart: always
    depends_on:
      - n8n

volumes:
  n8n_data:

```

貼上內容後，在終端機（Terminal）輸入 `docker compose up -d` 即可啟動

![img](./img/init_n8n/docker_compose_up.png)

## ▋ STEP 3: 建立與啟動
### 編譯 Spring Boot 專案
cd autoDemo
./mvnw clean package

### 回到專案根目錄
cd ..

### 啟動容器
docker-compose up --build

## ▋ STEP 4: n8n 範例流程設定
啟動 n8n，開啟 http://localhost:5678

建立一個新 Workflow，拖拉 Webhook 節點

設定：

HTTP Method: POST

Path: /myworkflow

Authentication: None（或 Basic）

接著加入你想要的自動化流程（例如寄信、呼叫 API）

部署並複製該 Webhook URL，例如 http://localhost:5678/webhook/myworkflow

# 開始做省錢版通知
## Step 1：準備 Telegram Bot
    開 Telegram 搜尋 @BotFather    輸入 /newbot，依指示建立一個 bot
    拿到你的 Bot Token（長得像 123456:ABC-DEF...）
    尋找你想接收通知的 Telegram 聊天 ID：
    打開你的 bot 聊天視窗，先傳一句話，然後用這個網址查看你的 chat ID
👉  https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getUpdates
    在回傳的 JSON 裡找 chat.id，記住id

## Step 2：n8n Workflow 範例（抓 PTT 省錢板）
    範例流程：
    Cron：每 10 分鐘執行一次
    HTTP Request：抓 PTT 省錢板的 HTML（https://www.ptt.cc/bbs/Lifeismoney/index.html）
    HTML Extract：抓出每篇文章的標題 + 連結
    IF 判斷：判斷是否是新文章（可比對儲存的上次結果）
    Telegram：推送訊息

### 📦 範例 n8n 節點設定（純文字範本）
    1. Cron（定時觸發）
       Trigger every 10 minutes
        Use built-in Cron node
    2. HTTP Request 項目	設定
        URL	https://www.ptt.cc/bbs/Lifeismoney/index.html
        Method	GET
        Response Format	String
    3. HTML Extract（用 HTML Extract 或 Cheerio 節點）
       CSS selector：div.r-ent a
        這可以抓出所有文章連結的 <a> 元素
    4. Set（把資料組成你要的格式）
       你可以用 Set node 將抓到的文章組成： json 複製 編輯
        {
        "title": "這家超商又有買一送一",
        "link": "https://www.ptt.cc/bbs/Lifeismoney/M.12345678.A.html"
        }
    5. Deduplicate（去重，避免重複通知）
       使用 n8n 的 Data Store（或 Redis）儲存已通知過的文章連結，避免重複推播。
    
    6. Telegram Node
          Bot Token：填剛剛建立的 Bot Token
        Chat ID：剛剛查到的 chat id 
    Message Text：
        text
        複製
        編輯
        📢 PTT 省錢板有新消息！
            📰 {{ $json["title"] }}
            🔗 {{ $json["link"] }}
