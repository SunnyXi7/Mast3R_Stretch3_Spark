# 跨端介面摘要

本頁是用於交接的 profile 對照，不是變更契約。正式修改前，仍須核對該端的 launch/config、實際 publisher 與 Unity 序列化場景。以下用既有主機別名表示機器；實際 Unity 端設定可能使用內網位址而非 SSH alias。

| 路徑或 profile | 端點／domain | 職責 |
| --- | --- | --- |
| Robot → SparkX live 感測 | Robot TCP `50011`（HRS1） | D455 JPEG 與 CameraInfo，保留標頭；感測來源，不是控制。 |
| Workspace map | SparkX domain **97**、ROS-TCP **10005** | Unity `workspace.unity` 接收重建資料。 |
| Workspace control | SparkX TCP **10006** | JSON-lines `hri_workspace_control_v1`：`calibrate`、`begin`、`finish`、`status`；使用既有 session/request/phase/revision 檢查及 ACK。只控制重建階段。 |
| Pure RGB delta | SparkX domain **95**、ROS-TCP **10003** | 與 workspace 分離；典型 Unity scene 是 `sunntxi——recon 1`，不接 workspace 階段控制。 |
| Unity ↔ Robot | Robot rosbridge **9090** | 動作指令及狀態回授。不可拿來測重建連線或重播控制資料。 |
| Robot → Unity RGB-D | Robot raw TCP **50010** | 已選相機的直接 RGB-D 顯示；不是 MASt3R map 端點。依所選 Robot profile 啟用。 |
| Robot → LBK | Humble native DDS domain **0**，`rmw_fastrtps_cpp`，非 localhost-only | 僅接收既有 streams。用實際 offered QoS 驗證相容，不擅自改 publisher。 |

## Workspace map 傳輸與幾何

| ROS topic | ROS 型別 | 用途 |
| --- | --- | --- |
| `/koch/mast3r/delta/keyframe_metadata` | `std_msgs/msg/String` | JSON：UID、姿態 Sim(3)、版本和 snapshot 控制。 |
| `/koch/mast3r/delta/keyframe_cloud_local` | `sensor_msgs/msg/PointCloud2` | 每 UID 的局部 XYZRGB 資料；三個 float32 座標與打包 RGB。 |
| `/koch/mast3r/camera_trajectory` | `nav_msgs/msg/Path` | 相機軌跡，工作區 patch 不當作相機軌跡節點。 |

同一樣本請保留來源整數秒／奈秒，不要用主機 wall clock 假裝相機時間。Workspace 點雲 `header.frame_id` 承載 UID/revision 識別，不是一般 TF frame。配對同 UID／revision，再以元資料中的 Sim(3) 與 Unity 場景變換放置局部點。不要再對 XYZ 額外乘一次已在輸出 pose 套用的 Tag 尺度。校正成功後先用新 revision 更新對應點雲／pose 與軌跡；保留重連 snapshot、較舊版本拒絕及完整替換語義。QoS 深度與舊 profile 不可互抄，需查實際 publisher。

## Camera topic 查找

- 正式 live D455：`/mast3r_camera/camera/color/image_raw/compressed`（`CompressedImage`）與 `/mast3r_camera/camera/color/camera_info`（`CameraInfo`）。
- LBK 也可在使用者指定且 Robot 真正發布時，接收 head `/camera/camera` 或 D405 `/gripper_camera/gripper_camera` 下的相機流；D405 彩色影像是 `color/image_rect_raw`，不是 `color/image_raw`。不要因此啟動其他相機。
- 已記錄的 workspace 離線入口使用 `/camera/color/image_raw` 與 `/camera/color/camera_info`，且要求原始 `Image`，不能把 live JPEG topic 名直接代入。換 bag 或 topic 必須核對腳本與實際 bag。
