# Mast3R / Stretch 3 / SparkX — HRI Coordinator

這是跨主機總控的交接倉庫，不是任何一端的程式碼鏡像或部署包。Claude Code 在此目錄開啟後，從 [CLAUDE.md](CLAUDE.md) 開始；人類操作者可從 [架構與資料流](ARCHITECTURE.md) 開始。

| 文件 | 用途 |
| --- | --- |
| [CLAUDE.md](CLAUDE.md) | 角色、操作邊界、跨端修改及驗證流程 |
| [ARCHITECTURE.md](ARCHITECTURE.md) | 主機、專案位置、live/offline 路徑和查找入口 |
| [INTERFACE.md](INTERFACE.md) | Profile、連線端點、ROS 訊息、座標和版本規則 |

Unity 專案另存於 [hrc-pme/Mast3R_delta_workspace_unity](https://github.com/hrc-pme/Mast3R_delta_workspace_unity)，實際部署工作樹仍以本機為準。此倉庫不包含 SSH 設定或金鑰、憑證、模型權重、rosbag、實驗資料、錄影、測試日誌或主機備份。已配置的 SSH 別名由操作者自己的設定提供。

交接整理日期：2026-11-16。內容轉述既有本機架構、介面及已記錄部署，不代表整理當天重新驗證了設備連線或實機功能。
