/dev/ttyS4", "/dev/ttyS5

# Run this on NUC980 before sending data
stty -F /dev/ttyS5 115200 cs8 -parenb -cstopb -crtscts raw

echo -e "Test123\r\n" > /dev/ttyS5

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
sub-board [GRN 5-rx 5-tx 4-rx 4-tx]
USB-serial [x x tx rx]
nRF52 DK [p0.08-rx p0.06-tx]

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
<info> app: UART init result: 0
<info> app: uart_event_handle
<error> app: Communication error occurred while handling UART.
<error> app: Fatal error

