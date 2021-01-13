搭建基礎line bot(Python)
===
## 基本流程＆工具介紹
#### Line Bot運作流程圖
![](https://i.imgur.com/YX3ARN5.png)

#### Heroku介紹
![](https://i.imgur.com/VwS8uFd.png)

Heroku是一個支援多種程式語言的雲平台即服務(PaaS)，可以在Heroku平台上開發或部署各種網站，並不需太著墨在管理或是維護機器。目前Heroku可以透過註冊免費帳號，在使用有限的額度及一些條件限制下開發你的應用程式。

## 前置作業
1. 建立[line官方帳號](https://tw.linebiz.com/login/)
2. 建立[Heroku帳號](https://signup.heroku.com/login)
3. [下載範例檔案](https://github.com/luching0823/line-bot-tutorial/archive/master.zip)

## 範例檔案說明
1. Procfile：heroku的執行命令，web: {語言} {檔案}，這邊因為我們的語言為 python，要自動執行的檔案為 app.py，因此我們改成 ```web: python app.py```  
>有更動的話記得不要加副檔名(但編輯時可先改成.txt方便編輯)
2. requirements.txt：列出所有需要用到的套件，heroku會依據這份文件來安裝指定好的套件
>之後程式如果有新增套件，要記得也新增到 requitements.txt 裡面

## 設定範例line bot
1. 進入[line後台](https://developers.line.biz/en/)，選擇剛剛創建的帳號，
將**Use Webhook**打開
![](https://i.imgur.com/JEtwNPu.png)
2. 將自動回覆訊息功能關掉
![](https://i.imgur.com/klPdMSR.png)
3. 產生Channel access Token
![](https://i.imgur.com/CGbgCcw.png)
4. 產生Channel secret
![](https://i.imgur.com/CAgLEYi.png)
5. 將範例檔案內的app.py打開，將剛剛取得的token跟secret輸入進去
![](https://i.imgur.com/b78IxWK.png)

## 將範例line bot推上去Heroku
1. 安裝[Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli)，[Git](https://git-scm.com)
2. 打開命令提示字元/終端機，移動(cd)到範例檔案的資料夾內
![](https://i.imgur.com/Z9eyTHW.png)
3. 輸入以下指令登入到Heroku
```
heroku login
```
4. 初始化git
```
git config --global user.name "輸入你的名字"
git config --global user.email 輸入你的信箱
```
5. **第一次使用**git要輸入以下指令，之後不用輸入
```
git init
```
6. 將資料夾與Heroku連接起來
```
heroku git:remote -a {你的Heroku App的名稱}
```
>注意:{你的Heroku App的名稱}是你之前在Heroku上新增的Heroku App的名稱
7. 輸入以下指令，將範例程式碼推上去Heroku，**如果中間有錯誤要重新輸入**
```
git add .
git commit -m "Add code"
git push -f heroku master
```
**每次更新程式碼時，都要新輸入以上指令**(記得先heroku login)

## 將Heroku與Line綁定
1. 進入line後台，選擇要綁定的bot
2. 在webhook URL輸入Heroku網址，按下Verify直到Success出現
```
{你的Heroku App的名稱}.herokuapp.com/callback
```
![](https://i.imgur.com/Paay4wQ.png)
>注意:{你的Heroku App的名稱}是你之前在Heroku上新增的Heroku App的名稱
### 錯誤解決
Webhook URL認證的時候可能會有狀況，像下面這樣：**(但是聊天機器人可以正常回覆)**
```
The webhook returned an invalid HTTP status code.
(The expected status code is 200.)
```
**查看錯誤訊息：**
1. 進入Heroku並點擊與line bot連接的專案
2. 按下More並點擊View Logs查看工作日誌
![](https://i.imgur.com/nSHlkJS.png)
3. 仔細看log應該會有以下訊息：
```
linebot.exceptions.LineBotApiError: LineBotApiError: status_code=400,
error_response={"details": [], "message": "Invalid reply token"}
```
注意上面這句：「**Invalid reply token**」，表示**沒辦法用line給的reply token**  
要知道更詳細的原因就必須更動程式碼，印出更仔細的資訊  
程式碼更動如下：  
![](https://i.imgur.com/NUJIQEf.png)  
重新推向 Heroku後，再重新按下 Verify，回到 Heroku 應用程式工作日誌，
然後又重新看到了**linebot.exceptions.LineBotApiError**
但這次會看到以下：  
```
{
   "events": [
     {
       "replyToken": "00000000000000000000000000000000",
       "type": "message",
       "timestamp": 1568894460464,
       "source": {
         "type": "user",
         "userId": "Udeadbeefdeadbeefdeadbeefdeadbeef"
       },
       "message": {
         "id": "100001",
         "type": "text",
         "text": "Hello, world"
       }
     },
     {
       "replyToken": "ffffffffffffffffffffffffffffffff",
       "type": "message",
       "timestamp": 1568894460464,
       "source": {
         "type": "user",
         "userId": "Udeadbeefdeadbeefdeadbeefdeadbeef"
       },
       "message": {
         "id": "100002",
         "type": "sticker",
         "packageId": "1",
         "stickerId": "1"
       }
     }
   ]
 }
```
前面"Invalid reply token"，說的就是:
"replyToken": "00000000000000000000000000000000"
"replyToken": "ffffffffffffffffffffffffffffffff"
可能因為這是line測試的罐頭訊息，想解決的話只要在圖中部分加入以下程式：  
```
if event.source.user_id != "Udeadbeefdeadbeefdeadbeefdeadbeef":
```
![](https://i.imgur.com/Bh9cu5L.png)  
就會看到Success囉！
![](https://i.imgur.com/T4QtJwe.png)


## 測試成果
1. 將bot加入好友
2. 向bot發送訊息，並且確認bot會學你說話
![](https://i.imgur.com/4NEGxNn.png)
3. **完成簡單的line bot囉！**


## 進階應用可以參考以下
1. [簡明 Python LINE Bot & LIFF JS SDK 入門教學](https://blog.techbridge.cc/2020/01/12/簡明-python-line-bot-&-liff-js-sdk入門教學/)
2. [[Python+LINE Bot+PostgreSQL教學]一篇搞懂LINE Bot讀取資料庫的方法](https://www.learncodewithmike.com/2020/07/python-line-bot-connect-postgresql.html)
3. [只要有心，人人都可以做卡米狗](https://ithelp.ithome.com.tw/users/20107309/ironman/1253)
4. [Line Develepoers Documents](https://developers.line.biz/en/docs/messaging-api/)

## Git應用與原理可以參考以下
1. [為你自己學Git](https://gitbook.tw)
2. [連猴子都能懂的Git入門指南](https://backlog.com/git-tutorial/tw/)

## 參考文章
1. [LineBot+Python，輕鬆建立聊天機器人](https://blackmaple.me/line-bot-tutorial/)
2. [第 11 天：LINE BOT SDK：應用程式編程介面](https://ithelp.ithome.com.tw/articles/10217767)
