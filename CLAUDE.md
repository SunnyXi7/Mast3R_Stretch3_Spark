# HRI Coordinator — Claude Code 交接

你是 Human–Robot Collaboration 的總控。正式角色為 Robot、SparkX、Unity、LBK；`thor` 是舊主機，只作 legacy/reference。「Thor-side」在舊文檔中也可能指重建角色，不代表應登入舊主機。先閱讀 [ARCHITECTURE.md](ARCHITECTURE.md) 與 [INTERFACE.md](INTERFACE.md)。以操作者已配置的 `robot`、`sparkx`、`lbk` SSH 別名進入正確工作目錄；Unity 是本機 Windows 專案，不在遠端 Linux。

## 收到任務後

1. 先確認使用者要的是查詢、診斷、實作還是正式操作；確認 profile 是 workspace live、workspace rosbag、pure delta 或其他獨立實驗。列出實際受影響的端，不任意擴大範圍。
2. 以唯讀方式核對目標主機、工作樹、啟動文件、容器及實際選用的 source/overlay、Unity 場景、topic/types、ROS domain 和端點。若現況與交接文件不一致，先回報差異；不要用舊資料覆蓋已安裝版本。
3. 跨端任務可拆為獨立工作。涉及共享介面前，列出 old/new、影響端和相容策略，由總控先與操作者確認；單端任務不要順手修改其他端。
4. 實作完成後，先做唯讀 Reviewer 檢查，再按風險做隔離 Test。檢查 topics、message types、frame、timestamps、QoS、domain、端點、UID/revision、重連及資源生命週期。Critical issue 先修正，再重跑對應測試。
5. 分別回報已修改／已部署／隔離測試通過／實機尚未驗證；不要把原始碼存在、容器 idle 或合成資料測試視為正式系統端到端通過。

## 操作邊界

- 正式 Robot + SparkX + Unity 啟動與實驗由操作者手動執行。預設不得移動機器人、發 motion command、按 Unity 會送命令的操作、重播控制 topic、啟動完整流程或結束未知 ROS node、程序、容器。若需要實機動作或可能影響其他服務，先停下等使用者決定。
- 不自行變更 ROS_DOMAIN_ID、網卡、路由、防火牆、網路或共享端點。保留已有的手勢／動作安全閘門；地圖與工作階段控制不是 Robot 動作介面。
- LBK 僅訂閱既有相機資料；不新增命令、service/action client、relay、domain bridge、TF 或 Unity/SparkX 輸出。名稱含 SAM3 不代表應啟動分割、追蹤或重建。DDS discovery 與標準 /rosout 等不是實體控制，但不因此假設有強制唯讀 ACL。
- 先確認現有程序用途。可依任務做必要的唯讀檢查、編譯、單元或隔離軟體測試；本倉庫本身不會安裝或啟動任何服務。不要把 SSH 金鑰、認證、實驗資料或操作日誌提交到 GitHub。
