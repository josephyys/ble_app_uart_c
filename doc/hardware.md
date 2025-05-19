/dev/ttyS4", "/dev/ttyS5

dongle soft device pca10059 s140

# Run this on NUC980 before sending data
stty -F /dev/ttyS4 115200 cs8 -parenb -cstopb -crtscts raw
stty -F /dev/ttyS5 115200 cs8 -parenb -cstopb -crtscts raw
cat /dev/ttyS5 > /dev/null
dmesg | grep tty

minicom -D /dev/ttyS5 -b 115200
busybox hexdump -C /dev/ttyS5

echo -e "Test123\r\n" > /dev/ttyS5
echo -e "Test123\r\n" > /dev/ttyS4

檢查 UART 的 RX 和 TX 數量：
cat /proc/tty/driver/serial 
cat /sys/kernel/debug/serial

檢查 UART 或相關中斷是否正常運作
dmesg | grep tty
檢查中斷計數是否正常增加：
cat /proc/interrupts | grep ttyS5
檢查波特率、數據位、停止位、校驗位是否與對端一致。
stty -F /dev/ttyS5
嘗試重新加載 UART 驅動
modprobe -r 8250
modprobe 8250

# Configure the UART port with proper settings
stty -F /dev/ttyS5 115200 cs8 -parenb -cstopb -crtscts raw
# Use cat to display incoming data
cat /dev/ttyS5
# Dump hex values of received data
hexdump -C /dev/ttyS5

ncu98 tx(red) -->P0.08
rx(bladck) --> p0.06
<info> app_timer: RTC: initialized.
<info> app: UART pins - RX: 8, TX: 6
<info> app: UART init result: 0
<info> app: BLE UART central example started. 20250518
<info> app: app_uart_put sent
<warning> ble_nus_c: Connection handle invalid.
<warning> ble_nus_c: Connection handle invalid.
<warning> ble_nus_c: Connection handle invalid.


minicom -D /dev/tty.usbserial-14110 -b 115200
screen /dev/tty.usbserial-14110 115200
ps aux | grep minicom



===========================
the pin of each hardware
muc980 sub-board [GRN 5-rx 5-tx 4-rx 4-tx]
USB-serial [x x tx rx]
nRF52 DK [p0.08-rx p0.06-tx]

test:
macos <==> nuc980 5.10
tx(black) -->  rx(black) ok (cat /dev/ttyS5)
rx(red)  <--   tx(red) fail (echo -e "Test123\r\n" > /dev/ttyS5)

macos <==> nuc980 4.4
tx(black) -->  rx(black) fail (cat /dev/ttyS5)
rx(red)  <--   tx(red) ok/fail(....) (echo -e "Test123\r\n" > /dev/ttyS5)


nRF52 <==> nuc980 4.4
tx(white) -->  rx(black) fail (cat /dev/ttyS5)
rx(red)  <--   tx(red) ok/fail(....) (echo -e "Test123\r\n" > /dev/ttyS5)
<info> app: app_uart_put sent
<info> app: uart_event_handle
<info> app: uart_event_handle
<error> app: Communication error occurred while handling UART.
<error> app: Fatal error

nRF52 <==> nuc980 5.10
tx(white) -->  rx(black) fail (cat /dev/ttyS5)
rx(red)  <--   tx(red) fail (echo -e "Test123\r\n" > /dev/ttyS5)

nRF52 <==> macos
tx(white) -->  rx(red)  (cat /dev/ttyS5)
rx(red)  <--   tx(black) (echo -e "Test123\r\n" > /dev/ttyS5)
plug in usb-serial, error==>
<info> app_timer: RTC: initialized.
<info> app: UART pins - RX: 8, TX: 6
<info> app: uart_event_handle
<error> app: Communication error occurred while handling UART.
<error> app: Fatal error



# Test Records Summary

## Hardware Pin Mapping
- **NUC980 Sub-board**:
  - GRD
  - UART 5-rx
  - UART 5-tx
  - UART 4-rx
  - UART 4-tx
- **USB-Serial**:
  - X
  - GRD
  - TX
  - RX
- **nRF52 DK**:
  - P0.08 (RX)
  - P0.06 (TX)

---

## Test Results

### **MacOS <==> NUC980 (Kernel 5.10)**
1. **TX (MacOS, black) --> RX (NUC980, black)**:
   - **Result**: OK
   - **Command**: `cat /dev/ttyS5`
2. **RX (MacOS, red) <-- TX (NUC980, red)**:
   - **Result**: FAIL
   - **Command**: `echo -e "Test123\r\n" > /dev/ttyS5`

---

### **MacOS <==> NUC980 (Kernel 4.4)**
1. **TX (MacOS, black) --> RX (NUC980, black)**:
   - **Result**: FAIL
   - **Command**: `cat /dev/ttyS5`
2. **RX (MacOS, red) <-- TX (NUC980, red)**:
   - **Result**: OK/FAIL (Unstable)
   - **Command**: `echo -e "Test123\r\n" > /dev/ttyS5`

---

### **nRF52 DK <==> NUC980 (Kernel 4.4)**
1. **TX (nRF52, white) --> RX (NUC980, black)**:
   - **Result**: FAIL
   - **Command**: `cat /dev/ttyS5`
2. **RX (nRF52, red) <-- TX (NUC980, red)**:
   - **Result**: OK/FAIL (Unstable)
   - **Command**: `echo -e "Test123\r\n" > /dev/ttyS5`
   - **Logs**:
     ```
     <info> app: app_uart_put sent
     <info> app: uart_event_handle
     <info> app: uart_event_handle
     <error> app: Communication error occurred while handling UART.
     <error> app: Fatal error
     ```

---

### **nRF52 DK <==> NUC980 (Kernel 5.10)**
1. **TX (nRF52, white) --> RX (NUC980, black)**:
   - **Result**: FAIL
   - **Command**: `cat /dev/ttyS5`
2. **RX (nRF52, red) <-- TX (NUC980, red)**:
   - **Result**: FAIL
   - **Command**: `echo -e "Test123\r\n" > /dev/ttyS5`

---

### **nRF52 DK <==> MacOS**
1. **TX (nRF52, white) --> RX (MacOS, red)**:
   - **Result**: fail
   - **Command**: `cat /dev/ttyS5`
2. **RX (nRF52, red) <-- TX (MacOS, black)**:
   - **Result**: fail
   - **Command**: `echo -e "Test123\r\n" > /dev/ttyS5`

when USB-Serial Connection
- **Error Logs**:
  ```
  <info> app_timer: RTC: initialized.
  <info> app: UART pins - RX: 8, TX: 6
  <info> app: UART init result: 0
  <info> app: uart_event_handle
  <error> app: Communication error occurred while handling UART.
  <error> app: Fatal error
  ```

---

## Observations
1. **MacOS <==> NUC980**:
   - Communication is inconsistent, with Kernel 5.10 showing better results than Kernel 4.4.
2. **nRF52 DK <==> NUC980**:
   - Communication fails in most cases, with unstable results when using Kernel 4.4.
3. **nRF52 DK <==> MacOS**:
   - Communication works correctly in both directions.
4. **USB-Serial**:
   - Errors occur during UART communication, indicating potential configuration or hardware issues.

---

## Recommendations
1. **Verify UART Configuration**:
   - Ensure baud rate, parity, stop bits, and flow control settings match on both ends.
   - Example: `stty -F /dev/ttyS5 115200 cs8 -parenb -cstopb -crtscts raw`
2. **Check Wiring**:
   - Confirm TX/RX connections are correct and stable.
   - Ensure GND is properly connected between devices.
3. **Test with Known Working Setup**:
   - Use MacOS as a baseline to verify nRF52 DK functionality.
4. **Debug UART Errors**:
   - Investigate the "Communication error occurred while handling UART" issue in the nRF52 DK firmware.
   - Add more detailed logging to identify the root cause.---
