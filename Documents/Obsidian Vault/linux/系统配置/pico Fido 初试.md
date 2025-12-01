# [polhenarejos/pico-fido: FIDO Passkey for Raspberry Pico and ESP32](https://github.com/polhenarejos/pico-fido)
笔者这里用的是 微雪的RP2350 one
长按boot键，然后插入usb，放入下载好的固件

# 在[Pico Commissioner - Pico Keys](https://www.picokeys.com/pico-commissioner/) 上配置

¹All operations are carried out locally. No information is transmitted over the internet and all operations are done locally by your browser.  
¹所有操作均在本地进行。不会通过互联网传输任何信息，所有操作均由您的浏览器在本地完成。  
²Secure Boot and Secure Lock must be commissioned with WebUSB.  
²必须使用 WebUSB 调试安全启动和安全锁。  
³Only for Pico Fido.  ³仅适用于 Pico Fido。  
⁴Android does not support secp256k1 curve.  
⁴Android 不支持 secp256k1 曲线。

点击`Commission via WebUSB `进行配置

# 测试？
[WebAuthn.io](https://webauthn.io/)
[Yubico demo website](https://demo.yubico.com/playground)