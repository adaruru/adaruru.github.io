---
title: "Typora"
date: 2025-04-17T00:00:00+08:00
description: "工具教學：Typora 使用與 Markdown 工作流筆記"
menu:
  sidebar:
    name: Typora
    identifier: typora
    parent: Tool
    weight: 10
tags:
- Typora
- Markdown
- Tool
---

# Typora

a nice .md file editor IDE

撰寫者：Amanda Chou 
日期：2025/04/17

## .md ?

- md 是什麼?
- Markdown 文件的副檔名
- Markdown 又是什麼?

## Markdown ? 

- **Markdown **是一種[輕量級標記式語言](https://zh.wikipedia.org/wiki/轻量级标记语言)，創始人為[約翰·格魯伯](https://zh.wikipedia.org/wiki/約翰·格魯伯)。
- 使用易讀易寫的純文字格式編寫文件，然後轉換成有效的[XHTML](https://zh.wikipedia.org/wiki/XHTML)（或者[HTML](https://zh.wikipedia.org/wiki/HTML)）文件。
- 這種語言吸收了很多在[電子郵件](https://zh.wikipedia.org/wiki/电子邮件)中已有的純文字標記的特性。
- 輕量化、易讀易寫特性，並且對於圖片，圖表、數學式都有支援

## Markdown 用途廣泛

目前許多網站都廣泛使用Markdown來撰寫說明文件或是用於[論壇](https://zh.wikipedia.org/wiki/网络论坛)上發表訊息。如[GitHub](https://zh.wikipedia.org/wiki/GitHub)、[Reddit](https://zh.wikipedia.org/wiki/Reddit)、[Discord](https://zh.wikipedia.org/wiki/Discord)、[Diaspora](https://zh.wikipedia.org/wiki/Diaspora_(社交网络))、[Stack Exchange](https://zh.wikipedia.org/wiki/Stack_Exchange)、[OpenStreetMap](https://zh.wikipedia.org/wiki/OpenStreetMap) 、[SourceForge](https://zh.wikipedia.org/wiki/SourceForge)、[簡書](https://zh.wikipedia.org/wiki/简书)等，甚至還能被用來撰寫[電子書](https://zh.wikipedia.org/wiki/電子書)。

1. 之前介紹的 Vue 文件 [重新認識 Vue.js | Kuro Hsu](https://book.vue.tw/)
2. git book 也是 [深入淺出 GitBook 寫作與自助出版，電子書也能多人協作](https://www.codedata.com.tw/social-coding/gitbook-self-publishing/)
3. HackMD 也是 [小題大作的30個 HackMD 技巧](https://ithelp.ithome.com.tw/articles/10290656)

4. 現在演示的這個投影片也是
   - md fille 怎麼變 slide? 後面針對匯出章節會提到


## Why Markdown

1. 輕量易用：Markdown 語法簡單直觀，幾乎無學習成本。(幾乎沒有，但還是有一點，語法還是需要熟悉一下)
2. 高度可移植：Markdown 文件是純文字格式，可以在任何平台上查看或編輯，方便與版本控制工具，適合開發者和團隊協作。
3. 多用途輸出：支援導出為 PDF、HTML、Word、Slides 等多種格式。特別適合技術文件、筆記、部落格文章等用途。
   - 多輸出主要是依賴 Pandoc 工具支援，後面針對匯出章節會提到

---

1. 強大的生態系統：可擴展性強，支持 LaTeX（數學公式）、圖表（如 mermaid.js）、代碼高亮等功能。
2. 開放性標準：Markdown 是一種開放格式，無專屬軟體依賴，確保文件長期可讀。
3. 參考: [寫作＆筆記神器 MarkDown 真希望我學生時期就懂](https://medium.com/@mdzeng/%E5%AF%AB%E4%BD%9C-%E7%AD%86%E8%A8%98%E7%A5%9E%E5%99%A8markdown-%E7%9C%9F%E5%B8%8C%E6%9C%9B%E6%88%91%E5%AD%B8%E7%94%9F%E6%99%82%E6%9C%9F%E5%B0%B1%E6%87%82-26becb160f6e)
   - 分享了 dillinger, Typora 還有影片介紹

___

### Why Markdown 只要可以版控，就會好找很多

如果有要找，版控，就會好找很多

如果沒有要找的需求，這段就聽聽就好

---

我可以文件內搜尋，這沒什麼了不起，word 也行

![image-20250217124735953](posts/tool/attach/typora/image-20250217124735953-17465930807531.png)



---

版控的優勢，在於可以資料夾搜尋

![image-20250217124833424](posts/tool/attach/typora/image-20250217124833424-17465930807532.png)

---

- 假設1: 我的產品修改了一個檔案上傳的 ftp 用法為 sftp
- 如果我用 word 存放我的文件，我要修改的文件可能在
  - aaa/bbb/xxxx檔案同步功能.docx
  - aaa/bbb/xxxx檔案下載功能.docx
  - ccc/xxxx檔案同步功能.docx
- 如果我有版控，我可以直接搜尋 ftp 找到所有，有 ftp 的文件

---

假設2: 我的網段修改 192.168.100.xxx 修改成 192.168.1.xxx 我要改幾個檔案 他們分別在哪裡

---

假設3: 我把 `DPARService` 打錯打成 `DAPRServe`，還複製貼上用了很多次，我要改幾個檔案 他們分別在哪裡

___

我可直接搜尋，再一個一個修改他們。

如果不能文件是不能版控的呢 ?

在資料夾海裡有 n 個 Word 檔，我要怎麼定位到這些內容 ?



![image-20250217125911585](posts/tool/attach/typora/image-20250217125911585-17465930807533.png)

---

- 也可以調整我的資料夾位置，擴大搜尋條件
- 其實只要內容可以版控，即使不用 typora, vs code 的搜尋功能其實也很好用

![image-20250217124914340](posts/tool/attach/typora/image-20250217124914340-17465930807534.png)

## Markdown 語法

<img src="posts/tool/attach/typora/markdownsyntex.png" alt="image-20250217095933173" style="zoom:67%;" />

## Markdown 效果展示

這是 [HackMD 快速入門教學](https://hackmd.io/s/quick-start-tw) 的顯示效果，而 HackMD 是幾乎所有社群活動或課程有開課就會使用的線上共筆工具，有在參加各社群年度 conference 的應該不陌生。

---

效果展示

<img src="posts/tool/attach/typora/markdowneffect.jpg" alt="img" style="zoom:50%;" />


## 可是我不想學這些

- 使用 typora ，你可以既保有 md 的好處
- 同時可以不用學會 md 的語法
- 本次分享的主要內容，就是 typora 的介紹 以及我使用 typora 的一些心得



## UI 友好

```
你不需要會 md 語法，你也可以用
想要什麼文字效果直接點
且工具上面都有顯示快捷鍵是什麼，用起來的感覺就跟 word 很像
```

![image-20250207095217939](posts/tool/attach/typora/image-20250207095217939-173976474289760-1739764830938122-17465930807537.png)



## 熟悉快捷鍵 提升編輯效率

分享幾個我常用的快捷鍵

- Ctrl + 數字 : 級數標題
- Ctrl + t : 新增 table
- Ctrl + shift + k  新增程式段落

## 不過 如果你突然間想學一下

可以隨時顯示原始 Markdown, 做中學

![image-20250207101247773](posts/tool/attach/typora/image-20250207101247773-173976474289761-1739764830938125-17465930807538.png)

## 級數標題 是連結 也是目錄

`[[_TOC_]]` 就會把所有級數標題集成目錄，可以點選

![image-20250217104357476](posts/tool/attach/typora/image-20250217104357476-173976474289762-1739764830938124-17465930807539.png)



---

不使用 toc , typora IDE 的大綱也可以點選目錄，有常在使用 word 的導覽功能，應該就很理解，一邊目錄導覽一邊查看或編輯內容的方便性

![image-20250217104610205](posts/tool/attach/typora/image-20250217104610205-173976474289763-1739764830938126-174659308075411.png)



azure 的 wiki 其實就是一個 git  repostary, clone 下來用法與 git 都一樣 

![image-20250217104944444](posts/tool/attach/typora/image-20250217104944444-173976474289764-1739764830938127-174659308075410.png)



---

azure 的 wiki 只要是級數標題，就都是連結，都可以點選、可以複製分享(直接導到內容)

![image-20250217104822610](posts/tool/attach/typora/image-20250217104822610-173976474289765-1739764830938128-174659308075413.png)

---

azure 專案 readme 也可以像 wiki 一樣查看，大多開源專案的說明文件，就是放在根目錄的 readme, gitbub 上有很多

![image-20250217105207593](posts/tool/attach/typora/image-20250217105207593-173976474289766-1739764830938130-174659308075412.png)



## Caret 註腳使用

 typora ide 可以 右鍵>>插入>>新增註腳

```
我寫了很多東西,很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西。

我寫了很多東西,很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西很多東西。

[^A]: 這個難懂的東西我另外在這裡說明
```

### 註腳效果展示

當我滑鼠上滑，就可以看到註腳說明以 tooltip 效果展示

![image-20250307113321033](posts/tool/attach/typora/image-20250307113321033-174659308075416.png)



## Checkbox 使用

![image-20250307114445166](posts/tool/attach/typora/image-20250307114445166-174659308075414.png)

---

也可以選取一段文字，讓其全部都變成 check box

![image-20250307114554264](posts/tool/attach/typora/image-20250307114554264-174659308075415.png)

---

直接點可以互動打勾，打勾的樣子

![image-20250307114614199](posts/tool/attach/typora/image-20250307114614199-174659308075417.png)



## URL 使用

直接複製連結，就是可以點擊的 link

https://hackmd.io/zh/product?utm_source=blog&utm_medium=nav-bar

---

選取文字使其轉成超連結

[hackmd](https://hackmd.io/zh/product?utm_source=blog&utm_medium=nav-bar)

![image-20250307114326637](posts/tool/attach/typora/image-20250307114326637-174659308075418.png)

---

離開編輯畫面，就會是簡短的樣子

![image-20250307114406234](posts/tool/attach/typora/image-20250307114406234-174659308075419.png)

## table 使用

Ctrl + t : 新增 table

![image-20250217112706201](posts/tool/attach/typora/image-20250217112706201-173976474289767-1739764830938129-174659308075420.png)

---

所有網頁的 tr、th、td 都可以簡單ctrl c, ctrl v 的 md 文件

![image-20250217113003581](posts/tool/attach/typora/image-20250217113003581-173976474289768-1739764830939133-174659308075421.png)

---

所有列、攔都可以拖曳變更位置

![image-20250217113254704](posts/tool/attach/typora/image-20250217113254704-173976474289769-1739764830939132-174659308075422.png)



---

也有很多快捷鍵可以使用

![image-20250217113340709](posts/tool/attach/typora/image-20250217113340709-173976474289770-1739764830939131-174659308075423.png)

## FileName Naming 注意

所有 Folder、File 上 Azure wiki、git book ，都會變成路徑，檔名就要小心

- 特殊符號在 URL 會被強制編碼，導致網址和真實檔名不同，圖片、檔案、資料夾就會讀不到。

  因為 **URL/URI 有「保留字元 (reserved characters)」與「不安全字元 (unsafe characters)」**，當檔名包含這些符號時，系統或瀏覽器在組合路徑時 **必須進行百分比編碼 (Percent-encoding)**。編碼後的路徑與實際檔名不一致，就造成圖片無法正確讀取。

- 檔名避免使用特殊符號 `#`, `?`, `&`, etc.) 會倒置路徑變成  .posts/tool/attach/typora/.C%23 使圖片顯示路徑錯誤，比如 `.C#` will be encoded as `.C%23`, which may cause the photo to fail to load.

  1. 保留字元（一定會被編碼）

     ```
     : / ? # [ ] @ ! $ & ' ( ) * + , ; =
     ```

  2. 不安全字元（常導致路徑錯誤）

     ```
     空白 " < > { } | \ ^ ` % 
     ```

  3. 建議避免的其它字元（不同平台會出問題）

     ```
     ~ 
     ```

  4.  最安全的檔名字元（白名單）

     ```
     A–Z
     a–z
     0–9
     -（dash）
     _（underscore）
     .（dot） 
     ```

### 為什麼是保留字元

這些字元在 URL 裡有特殊語義，其他特殊語義在此不加贅述。

| 符號 | 用途                  | 例子                  |
| ---- | --------------------- | --------------------- |
| `#`  | Fragment (錨點)       | `page.html#section1`  |
| `?`  | Query string 開頭     | `file.jpg?size=large` |
| `&`  | Query string 參數分隔 | `?id=3&mode=1`        |
| 空白 | 需編碼                | `%20` 或 `+`          |

只要檔名包含它們，**瀏覽器就一定會 percent-encode**，否則會破壞 URL 結構。

## image Source 圖片使用

- 由於 md 是存文字可以版控的，與 docx 是一整包壓縮檔完全不一樣

- 所以圖片，是像 img 那樣，需要透過 img url 導入來顯現

- 圖片上傳是預設存本機的固定位置，在文件共用的前提，使用上不好維護 (比如說固定存在  C:\Users\AmandaChou\Documents\typora)


- 這在文件的團隊共用編輯使用上不方便

---

- 目標: 

- 希望設置好之後，編輯圖片就是簡單的複製貼上，跟用 office 在感覺上都一樣

- 希望其他人 pull 之後，圖片都可以 render 正常，不要有多餘的動作跟溝通

### 圖片使用-相對位置處理圖片

Typora 相對位置處理圖片，路徑使用 ./.posts/tool/attach/typora/.${filename}

![image-20241114150224347](posts/tool/attach/typora/image-20241114150224347-173889219812626-173976474289771-1739764830939134-174659308075424.png)

---

![image-20241114150917531](posts/tool/attach/typora/image-20241114150917531-173889219812631-173976474289772-1739764830939135-174659308075425.png)



---

- 在這樣的配置之下，每一個檔案都會在複製上圖片的時候，新增一個資料夾，在相對路徑的地方新增與檔案同名的 (.posts/tool/attach/typora/.TyporaSlide) 資料夾
- 路徑像這樣

```
![image-20241114150917531](.posts/tool/attach/typora/.TyporaSlide/image-20241114150917531-173889219812631-173889542464476.png)
```

---

- 在維護的檔案需要搬遷位置的時候
- 就可以整個檔案與上傳的圖片一併移動
- 檔名=資料夾名，就很好維護、也很好找

![image-20250207103339022](posts/tool/attach/typora/image-20250207103339022-173976474289873-1739764830939136-174659308075426.png)

---

- 變成路徑，檔名就要小心

- 檔名避免使用特殊符號 `#`, `?`, `&`, etc.) 會倒置路徑變成  .posts/tool/attach/typora/.C%23 使圖片顯示路徑錯誤

- 比如:

  `.C#` will be encoded as `.C%23`, which may cause the photo to fail to load.

#### 為什麼資料夾要.開頭

./.posts/tool/attach/typora/.${filename}

* 避免在檔案總管或其他工具中混亂
  一般情況下，像 `.git/`、`.posts/tool/attach/typora/` 等以「`.`」開頭的資料夾會被視為系統或隱藏資料夾，從而減少開發者在檔案樹狀結構中誤操作或誤刪除。
* 和普通目錄區分
  用 `./.something` 的形式，可讓我們很快辨認「這是特定工具（Typora、Git 等）自動生成或使用的特殊目錄」。

---

 ./.posts/tool/attach/typora/.${filename}
 如果沒有加.
 那 typora 在打開左側欄查看檔案的時候 就會跑出 attach 跟 .${filename} 
 使用上會覺得介面比較混亂

---

#### 比較混亂是有多混亂，示範一下

![image-20250307121718603](posts/tool/attach/typora/image-20250307121718603-174659308075427.png)

### 圖片使用-縮小

右建圖片 調整大小

```html
# 加入屬性 width
<img src=".posts/tool/attach/typora/.XXX/image-20251015141308550.png" width="100">
# 圖片右鍵 Zoom image 選擇縮小比例
<img src="posts/tool/attach/typora/aligndemo.png" alt="image-20250605103033878" style="zoom:33%;" />
```

### 圖片使用-圖片靠左/靠右

預設置中

---

右邊加一個空格，有靠左效果 `<img src=""/>{加一個空格} `

<img src="posts/tool/attach/typora/aligndemo.png" alt="image-20250605103033878" style="zoom:33%;" /> 

左邊加很多縮排，有靠右效果 `{加  很  多  空  白}<img src=""/> `

​                                                                                      <img src="posts/tool/attach/typora/aligndemo.png" alt="image-20250605103033878" style="zoom:33%;" />

## Code Fence 使用

Ctrl Shift K

![image-20250217121619775](posts/tool/attach/typora/image-20250217121619775-174659308075428.png)

---

Code Fence highlight 過的 code 更好閱讀



![image-20250217122516889](posts/tool/attach/typora/image-20250217122516889-174659308075429.png)



Code Fence highlight 過的 code 更好閱讀



![image-20250217122520818](posts/tool/attach/typora/image-20250217122520818-174659308075430.png)





- 幾乎支援所有語言了

- 參考 [Code-Fences-Language-Support](https://support.typora.io/Code-Fences-Language-Support/)



## 流程圖使用

常見的流程圖工具

- drawio
- PlantUML
- lucid.app
- Visio



如果，流程圖的變更與維護，不需要版控 也不需要很好找到，那這段也可以聽聽就好



### 流程圖使用-Typora可以用的

參考:[Draw Diagrams With Markdown](https://support.typora.io/Draw-Diagrams-With-Markdown/)

- Sequence Diagrams
- Flowcharts
- Mermaid 



### 只重點分享 mermaid 

1. Mermaid 也可以畫 flow 、Sequence ，幾乎可以取代使用

2. git push 到專案azure 介面之後，只有 mermaid 有支援

   

#### mermaid 幹嘛又要學它呢

- 流程可版控
- 版控是人看得懂的
- drawio 有 vs code 的 可版控 exetension (但它版控的內容是一坨一坨的 xml)

- 相較之下 mermaid 的版控清楚多了
- 即便 git commit 沒有寫清楚是什麼時候的文件，對流程有印象的人看 git diff 也可以大概找到想找的歷史版本
- 看 git diff 相比 drawio 或是 Visio 一個流程、一個流程打開來檢查的時間，應該是快得多

![image-20250217111619175](posts/tool/attach/typora/image-20250217111619175-173976474289874-1739764830939137-174659308075431.png)

## mermaid

[專案整理流程範例](https://itsower-vsonline.visualstudio.com/ITSPayCollection/_wiki/wikis/wiki/212/CPMSDoc?anchor=batch-flow)

更多介紹

https://mermaid.js.org/intro/

### mermaid - 我下載來使用了，可是官方的圖片顯示不支援

- mermaid 是一個 js library
- mermaid 官方顯示的功能，是最新版的的 js
- 下載的 typora 版本 mermaid 不是使用最新的版本
- 下載最新版本mermaid  [mermaid@11.4.1](https://github.com/mermaid-js/mermaid/releases/tag/mermaid%4011.4.1)

- path:C:\Program Files\Typora\resources

- edit file : window.html

html <body> add

```html
<script>
    const interval = setInterval(() => {
        console.log('check mermaid...');
        if (window.editor &&
            window.editor.diagrams &&
            window.mermaidAPI) {
            $.getScript('file:///C:/ Program Files/Typora/resources/mermaid.min.js')
                .then(() => {
                    mermaidAPI = mermaid.mermaidAPI;
                    editor.diagrams.refreshDiagram(editor);
                    clearInterval(interval);
                });
        }
    }, 100);
</script>
```

- 這個改法主要是針對 windows 電腦
- windows 支援 參考 https://blog.csdn.net/changcheng00/article/details/134597120
- mac 支援 參考 https://www.cnblogs.com/learn-gz/p/17566881.html

#### mermaid 搭配使用 azure wiki

```pseudocode
須要提醒一點， azure 的 mermaid 支援是::: (三個冒號)
不是常見的 code Fence 寫法 ‵‵‵(三個反引號)
如果希望 azure 可以查看 mermaid render 圖片，就必須使用 ::: (三個冒號)
且不能用太新得功能
```

![image-20250217133947947](posts/tool/attach/typora/image-20250217133947947-174659308075432.png)

---

不能用太新得功能，因為目前最新的更新顯示[Wiki - 短期衝刺 200 更新](https://learn.microsoft.com/zh-tw/azure/devops/release-notes/2022/wiki/sprint-200-update)，目前只支援到 mermaid 到 8.13.9

![image-20250217135010349](posts/tool/attach/typora/image-20250217135010349-174659308075433.png)

---

- https://developercommunity.visualstudio.com/ 有相關討論，為什麼 azure wiki 要特立獨行(冒號)，不使用一般 md code block 語法 (反單引號)
- 可是 azure 沒有在 care 社群意見
- 或者可以[參考冒號包引號](https://stackoverflow.com/questions/73852774/markdown-syntax-differences-to-start-a-mermaid-block)
- 不過，如果這次的分享結束，大家都會使用 typora、或是自己找 vs code md 相關的 extensiopn 編輯 md，那也許 azure wiki 可以不再使用，也許所有文件都可以使用 ‵‵‵(三個反引號) 維護 mermaid 就不再是個問題了



## 檔案匯出 pandoc

- 使用 md 的好處，多用途輸出，主要是依賴 pandoc 來達成的
- 參考 [介紹好用工具：Pandoc ( 萬用的文件轉換器)](https://blog.miniasp.com/post/2018/10/06/Useful-tool-Pandoc-universal-document-converter)

### 檔案匯出 pandoc 下載

下載

- [About](https://pandoc.org/index.html)
- [Installing](https://pandoc.org/installing.html)



### 檔案匯出 typora 使用 pandoc

加入匯出的Pandoc.path

![image-20241114153155586](posts/tool/attach/typora/image-20241114153155586-173889219812629-173976474289876-1739764830939139-174659308075434.png)

---

- 可以簡單測試一下 匯出 word 跟 pdf 是否正常
- 其他匯出功能也可用用看

![image-20250217123246206](posts/tool/attach/typora/image-20250217123246206-174659308075435.png)



word 檔案匯出，原本的級數標題都會變成 word 格式支援的標題類

![image-20250217130938370](posts/tool/attach/typora/image-20250217130938370-174659308075436.png)

### 檔案匯出 匯出投影片 RevealJS

https://medium.com/@leshrimp/using-typora-to-create-presentations-1f4dce83b656

目前演示的這個投影片

就是用這個方法產出的

---

- 優點: 投影片二級標題可以左右橫切，投影講解的時候 這個功能還蠻好用

- 缺點1: 要改投影片主題要另外 google 支援的 css
- 缺點2: 編輯完在匯出投影片，沒辦法像 marp 一邊編輯、一邊 preview 

---

#### 有設定樣式的投影片 RevealJS + css

https://fitness.agroparistech.fr/fitness2/train-the-trainers/session-2/html-files/how-to-create-slides-from-markdown.html

這個方法包含了投影片產出可以設定樣式

MD 檔轉 SlidesTeam Jeffery md 製作投影片google 找 md to slides 有很多方法目前我用過兩個方法，比較一下區別，各有優缺 給大家參考 

### 檔案匯出 匯出投影片-有設定樣式的投影片 Marp

https://ivonblog.com/posts/vscode-marp-presentation/

- 優點: 有主題 也有深淺模式 主題蠻好看的、還可以設定字體
- 缺點1: mermaid 流程圖有自己的 code block 與 azure wiki、和 typora 都不一樣
- 缺點2: 匯出跟 Preview 需要 Vscode 變成要 typora 與 Vscode 都開著，使用上小麻煩

## 但，這軟體是要付費的 

- 破解說明 - 如何激活使用Typora（2023年最新版）

  目前大家使用的大部分是 `方法二：使用最后一个免费版本，并延长试用时间（存在一定操作难度）`

  其實沒有很難，大家自己評估挑選

- 參考 : https://juejin.cn/post/7266672372807827515

- 只要 google `typora 破解`，有各式各樣的文章可以參考，也可以自己再找找

- 舊版  typora 就會有支援舊版 mermaid js 的問題，參考前面章節 mermaid - 我下載來使用了，可是官方的圖片顯示不支援

## 其他: 添加自己喜歡的主題

預設就有很多主題可以使用

![image-20250217132832751](posts/tool/attach/typora/image-20250217132832751-174659308075437.png)





官網還有各種漂亮的樣式可以下載

[Themes 下載連結 ](https://theme.typora.io/)

![image-20250217131250440](posts/tool/attach/typora/image-20250217131250440-174659308075438.png)





下載好的主題怎麼用呢?

![image-20241114151348378](posts/tool/attach/typora/image-20241114151348378-173889219812627-173976474289877-1739764830939140-174659308075439.png)



1. 將已下載 theme 複製到 該資料夾
2. 重新開啟typora
3. 可以挑選更新的主題了

![image-20241114151738570](posts/tool/attach/typora/image-20241114151738570-173889219812628-173976474289878-1739764830939141-174659308075440.png)
