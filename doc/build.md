# SES version
v5.42a

JLINK 7.x

# 清除設備
nrfjprog --eraseall

# 燒錄 SoftDevice（調整路徑到您的 s132 hex 檔案）
nrfjprog --program <path_to_sdk>/components/softdevice/s132/hex/s132_nrf52_7.2.0_softdevice.hex --verify

# 燒錄應用程式
nrfjprog --program <output_directory>/ble_app_uart_c_pca10040_s132.hex --verify

# 重置以開始執行
nrfjprog --reset



=== wiring

# nRF52 DK 與 NUC980 的正確連接與調試方法

您目前的連接方式（USB到USB）是不正確的。USB-USB直連通常不適用於設備間通信，這就是為什麼您沒有收到任何響應。

## 正確的連接方式

### 選項1：通過UART引腳連接（推薦）
1. **硬體連接**:
   - 將nRF52 DK的TX引腳連接到NUC980的RX引腳
   - 將nRF52 DK的RX引腳連接到NUC980的TX引腳
   - 連接兩個設備的GND引腳

2. **nRF52 DK UART引腳**:
   - P0.06 (RX)
   - P0.08 (TX)
   - GND

3. **連接圖**:
   ```
   nRF52 DK                NUC980
   --------                ------
   P0.08 (TX) ---------> RX
   P0.06 (RX) <--------- TX
   GND        <--------> GND
   ```

### 選項2：使用兩條USB線
1. 將nRF52 DK通過USB連接到PC
2. 將NUC980通過USB連接到PC
3. 使用PC作為橋樑處理數據交換

## 查看nRF52 DK調試信息的方法

### 方法1：RTT查看器（需要連接PC）
1. 將nRF52 DK通過USB連接到PC
2. 運行SEGGER的J-Link RTT Viewer
3. 選擇正確的目標設備(nRF52832)並連接

### 方法2：通過UART查看（適用於當前場景）
1. 確保正確連接UART引腳（如上所述）
2. 在NUC980上運行串口監視程序：
   ```python
   import serial
   ser = serial.Serial('/dev/ttyS0', 115200)  # 使用適當的串口
   while True:
       if ser.in_waiting:
           print(ser.readline().decode().strip())
   ```

### 方法3：使用邏輯分析儀
如果有邏輯分析儀，可以連接到UART線路上監控通信

## 確認連接和通信

1. **驗證韌體**：確保nRF52 DK上已正確燒錄ble_app_uart_c示例
2. **硬體驗證**：測試UART針腳是否有信號輸出（使用萬用表或邏輯分析儀）
3. **簡單測試**：先讓nRF52 DK輸出固定模式的數據（如每秒發送"TEST"）確認連接

如果按照上述方法進行連接，您應該能夠正確接收到nRF52 DK的調試信息和串口數據。