# HRI 系統架構與定位

| 角色 | 位置／工作目錄 | 責任 |
| --- | --- | --- |
| Robot | SSH `robot`；`/home/hrc/stretch3` | Stretch 3、RealSense、ROS 2 Humble、動作和狀態回授。 |
| SparkX | SSH `sparkx`；`/home/hrclab631/Mast3R/mast3r_openvins` | 正式重建角色，承接舊 Thor。MASt3R-SLAM、點雲和軌跡輸出、workspace 階段管理；OpenVINS 為另選流程。 |
| Unity | 本機 Windows；`C:/Users/xigua/Desktop/VR_project/Multi_robot-reconstruction`（不同使用者請代入實際 clone 路徑） | VR/XR、Avatar、直接相機回饋、點雲顯示、使用者操作與 Robot 控制／狀態。Unity 專案版本 2022.3.51f1。 |
| LBK（賴bk） | SSH `lbk`；`/home/hrclab/stretch_sam3` | 相機唯讀接收。既有證據未證實 Robot → LBK 實體收圖成功；以現場唯讀檢查為準。 |
| 舊 Thor | SSH `thor`；`/home/hrc/mast3r_openvins` | Legacy/reference；不作正式重建路由。 |

## 即時資料流

- **建圖：**Robot D455 原始 JPEG 與 CameraInfo → HRS1 → SparkX MASt3R-SLAM → delta 點雲、姿態／版本資料及相機軌跡 → Unity VR。跨機 HRS1 不等於 SparkX 加入 Robot DDS domain。
- **控制及回饋：**Unity 與 Robot 走各自的控制／狀態連線；直接 RGB-D 為獨立資料路徑。SparkX 地圖端點不代送機器人動作。
- **工作區：**Unity 可發重建階段命令。導航建立全域圖；成功 Tag 校正後可進入固定相機假設的局部工作模式。完成後保留最後有效合成結果，並用新鮮來源幀嘗試在同一 SLAM session 恢復導航建圖；恢復失敗保圖，可按既有邏輯重試。Tag 更新的是輸出的公尺尺度，不把地圖原點或方向改到 Tag 座標，也不保證消除後續尺度漂移。
- **接收支線：**LBK 訂閱 Robot 同 domain 的既有 camera topics，不經 SparkX，也不向 Unity 或 SparkX 重發。

## 入口與檔案

- **SparkX 啟動入口：**專案根目錄的 `啓動指令_new.md` 區分 workspace 與 pure；workspace 對應 `/home/hrclab631/Mast3R/.sparkx_runtime/workspace_unity_20261014/` 的啟動文件及 `sparkx_workspace.sh`。既有 workspace 容器名是 `workspace_local_map`；專案下安裝來源為 `workspace_local_map_unity_v2`。動手前比對當前掛載／文件，不使用舊 Thor 安裝命令。
- **Workspace Unity：**`Assets/Scenes/sunnyxi/workspace.unity`。接收、解碼、Sim(3) 及 GPU 緩衝：`Assets/Script/env_reconstruction/Mast3rKeyframeDeltaViewer.cs`；點渲染：同目錄 `Mast3rKeyframeDelta.shader`，GPU 將 XYZRGB 點展成面向畫面的色塊，不是 voxel 或表面 mesh。地面／碰撞／小地圖另外由 `RuntimePointCloudCollider.cs` 處理。
- **離線 workspace：**SparkX 主機的 `/home/hrclab631/Mast3R/.sparkx_runtime/workspace_unity_state_20261014/state/offline_runtime_20261029/workspace_bag.py` 是已記錄的離線入口。以唯讀 SQLite 取原始 RGB Image／CameraInfo CDR，透過本機 docker-exec stdin 餵給同一重建引擎；不連 Robot HRS1，不需要 `ros2 bag play`。初版只支援單個 SQLite `.db3`，不支援 MCAP／分卷。沿用 workspace Unity 和相同 ports，所以與 live 互斥；回放後 EOF 保圖。讀取的 topic 名稱需核對腳本與 bag 元資料，不假設跟 live 一樣。
- **尺度路徑不要混用：**目前 workspace 可用 AprilTag 尺度；Unity 相機高度／地板點為另一條校正途徑；OpenVINS stereo+IMU 為獨立 profile，並非 pure delta 或 workspace 必須啟用的元件。D455 深度比較工具是診斷，不能當作 workspace 使用真實 depth 修正重建的證據。

## 驗證狀態

來源是原 Coordinator 本機的 `AGENTS.md`、`ARCHITECTURE.md`、`INTERFACE.md`，以及 2026-10-22 Tag 與 2026-10-29 離線部署交接。文件是摘要，沒有把舊測試、部署日誌或未驗證指令當成這次操作結果。設備狀態、實際啟動與 VR 效果仍以現場為準。
