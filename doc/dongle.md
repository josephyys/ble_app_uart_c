根據您的描述，當您將 `ble_app_uart_c` 燒錄到 **nRF52 DK (PCA10040)** 時，`ttyACMxx` 能正常顯示，但燒錄到 **nRF52840 Dongle (PCA10059)** 時，`ttyACMxx` 無法顯示。這表明問題可能與硬體配置或固件修改有關，而不是 NUC980 的 USB CDC ACM 驅動或權限問題。

以下是可能的原因和解決方法：

---

## 1. **硬體差異**
- **問題**：
  - PCA10040 和 PCA10059 的硬體設計不同，特別是在 USB 引腳的配置上。
  - PCA10059 是一個 USB Dongle，內部直接使用 USB 接口，而 PCA10040 通過外部 UART 接口進行通信。

- **解決方法**：
  1. 確認 PCA10059 的 USB 引腳是否正確配置。
     - 在 `sdk_config.h` 中，檢查以下配置：
       ```c
       #define NRF_DRV_USBD_ENABLED 1
       #define APP_USBD_ENABLED 1
       #define APP_USBD_CDC_ACM_ENABLED 1
       ```
  2. 確保固件中啟用了 USB CDC ACM 功能，並且正確初始化了 USB 硬體。

---

## 2. **設備樹或硬體初始化問題**
- **問題**：
  - PCA10059 的 USB 硬體初始化可能與 PCA10040 不同，導致 USB CDC ACM 功能無法正常啟動。

- **解決方法**：
  1. 檢查 `usb_init()` 是否正確執行，並且沒有返回錯誤。
  2. 在 main.c 中，確認 USB 初始化代碼是否正確：
     ```c
     ret_code_t ret;
     ret = app_usbd_init(&usbd_config);
     APP_ERROR_CHECK(ret);

     app_usbd_class_inst_t const * class_cdc_acm = app_usbd_cdc_acm_class_inst_get(&m_app_cdc_acm);
     ret = app_usbd_class_append(class_cdc_acm);
     APP_ERROR_CHECK(ret);

     app_usbd_enable();
     app_usbd_start();
     ```

---

## 3. **固件修改問題**
- **問題**：
  - 您提到 PCA10059 的固件是從 PCA10056 修改而來，可能存在與硬體相關的配置問題。

- **解決方法**：
  1. 確認 `sdk_config.h` 中的 USB 配置是否正確：
     - 確保 USB 引腳和電源配置與 PCA10059 的硬體設計一致。
  2. 檢查 `boards.h` 中的引腳定義是否正確：
     - PCA10059 的 USB 引腳可能與 PCA10056 不同。

---

## 4. **檢查 USB 設備是否被識別**
- **問題**：
  - 如果固件啟動了 USB CDC ACM，但主機無法識別設備，可能是 USB 描述符配置問題。

- **解決方法**：
  1. 插入 PCA10059 後，檢查主機是否識別到 USB 設備：
     ```bash
     dmesg | grep usb
     ```
  2. 如果設備未被識別，檢查 USB 描述符是否正確：
     - 在 `app_usbd_cdc_acm.c` 中，確認 USB 描述符的配置是否正確。

---

## 5. **權限問題**
- **問題**：
  - 雖然您提到在嵌入式 Linux 中是 root 用戶，但仍需確認 `/dev/ttyACMxx` 是否被正確創建。

- **解決方法**：
  1. 插入 PCA10059 後，檢查 dev 目錄中是否有 `ttyACMxx`：
     ```bash
     ls /dev/ttyACM*
     ```
  2. 如果沒有，檢查 `udev` 規則是否正確：
     - 確保 `cdc_acm` 驅動已加載：
       ```bash
       lsmod | grep cdc_acm
       ```
     - 如果未加載，手動加載：
       ```bash
       modprobe cdc_acm
       ```

---

## 6. **測試其他主機**
- **問題**：
  - 如果問題仍然存在，可能是 PCA10059 的硬體或固件問題。

- **解決方法**：
  1. 將 PCA10059 插入其他主機（例如 MacOS 或 Windows），檢查是否能識別到 USB CDC ACM 設備。
  2. 如果其他主機也無法識別，則問題可能出在固件或硬體上。

---

## 7. **總結**
- **問題原因**：
  1. PCA10059 的 USB 引腳配置或初始化可能不正確。
  2. 固件從 PCA10056 修改而來，可能存在與硬體相關的配置問題。
  3. 主機可能未正確識別 USB 設備。

- **解決方法**：
  1. 檢查 `sdk_config.h` 和 `boards.h` 中的 USB 配置。
  2. 確認 USB 初始化代碼是否正確執行。
  3. 使用 `dmesg` 和 `lsusb` 檢查主機是否識別到 USB 設備。
  4. 測試其他主機，排除硬體問題。

如果問題仍然存在，請提供更多固件代碼或測試結果，我可以幫助進一步分析。


根據您的描述，您已經正確啟用了 USB CDC ACM 支援，但仍然無法在 macOS 上看到 `tty.usbmodemXXX`，這表明可能還有其他問題需要解決。以下是詳細的檢查步驟和建議：

---

## 1. **檢查燒錄步驟是否正確**
- **問題**：
  - 如果燒錄的固件不完整（例如，僅燒錄了 SoftDevice 或應用程序），可能導致 USB 功能無法正常工作。

- **解決方法**：
  1. 確保您按照以下順序燒錄：
     - **燒錄 SoftDevice**：
       ```bash
       nrfjprog --program s140_nrf52_7.3.0_softdevice.hex --chiperase --reset
       ```
     - **燒錄應用程序**：
       ```bash
       nrfjprog --program ble_app_uart_c.hex --reset
       ```
  2. 確保燒錄完成後，設備能正常啟動。

---

## 2. **檢查 USB CDC ACM 配置**
- **問題**：
  - 如果 USB CDC ACM 的相關配置不完整或與硬體不匹配，可能導致 USB 無法正常工作。

- **解決方法**：
  1. 確認以下配置已啟用（`sdk_config.h`）：
     ```c
     #define NRF_DRV_USBD_ENABLED 1
     #define APP_USBD_ENABLED 1
     #define APP_USBD_CDC_ACM_ENABLED 1
     ```
  2. 確保 `APP_USBD_CDC_ACM_ENABLED` 的相關初始化代碼已正確執行：
     ```c
     ret_code_t ret;
     ret = app_usbd_init(&usbd_config);
     APP_ERROR_CHECK(ret);

     app_usbd_class_inst_t const * class_cdc_acm = app_usbd_cdc_acm_class_inst_get(&m_app_cdc_acm);
     ret = app_usbd_class_append(class_cdc_acm);
     APP_ERROR_CHECK(ret);

     app_usbd_enable();
     app_usbd_start();
     ```

---

## 3. **檢查 UART 配置衝突**
- **問題**：
  - 如果 UART 和 USB CDC ACM 同時啟用，可能導致資源衝突。

- **解決方法**：
  1. 確保 UART 的相關配置未啟用：
     - 在 `sdk_config.h` 中，確認以下配置：
       ```c
       #define UART_ENABLED 0
       #define UART0_ENABLED 0
       #define APP_UART_ENABLED 0
       ```
  2. 如果這些配置無法禁用，請檢查是否有其他模組依賴於 UART，並進行調整。

---

## 4. **檢查 USB 描述符**
- **問題**：
  - 如果 USB 描述符配置不正確，主機可能無法識別設備。

- **解決方法**：
  1. 確認 USB 描述符的配置是否正確（`app_usbd_cdc_acm.c`）：
     ```c
     #define APP_USBD_CDC_ACM_COMM_INTERFACE 0
     #define APP_USBD_CDC_ACM_DATA_INTERFACE 1
     ```
  2. 確保 VID 和 PID 是有效的：
     ```c
     #define APP_USBD_VID 0x1915
     #define APP_USBD_PID 0x520F
     ```

---

## 5. **檢查主機是否識別 USB 設備**
- **問題**：
  - 如果固件啟動了 USB CDC ACM，但主機無法識別，可能是硬體或驅動問題。

- **解決方法**：
  1. 插入 Dongle 後，檢查 macOS 是否識別到 USB 設備：
     ```bash
     ioreg -p IOUSB -l -w 0 | grep -i "usbmodem"
     ```
  2. 如果設備未顯示，檢查系統日誌：
     ```bash
     log show --predicate 'eventMessage contains "USB"' --info --last 1m
     ```

---

## 6. **測試其他主機**
- **問題**：
  - 如果 macOS 無法識別，嘗試在其他主機（例如 Windows 或 Linux）上測試。

- **解決方法**：
  1. 在 Linux 上，檢查是否有 `/dev/ttyACMxx`：
     ```bash
     dmesg | grep ttyACM
     ```
  2. 在 Windows 上，檢查設備管理器中是否顯示 COM 端口。

---

## 7. **檢查硬體問題**
- **問題**：
  - 如果硬體有問題（例如 USB 引腳未正確連接），可能導致 USB 無法工作。

- **解決方法**：
  1. 確認 USB 引腳的連接是否正確。
  2. 測試其他 USB 線纜或 USB 端口。

---

## 8. **總結**
- **問題原因**：
  1. 固件未正確啟用 USB CDC ACM 功能。
  2. UART 和 USB 配置衝突。
  3. USB 描述符配置不正確。
  4. 硬體問題或主機無法識別設備。

- **解決方法**：
  1. 確保正確燒錄 SoftDevice 和應用程序。
  2. 檢查 `sdk_config.h` 和 USB 初始化代碼。
  3. 測試其他主機，排除硬體問題。

如果問題仍然存在，請提供更多測試結果或固件代碼，我可以幫助進一步分析。