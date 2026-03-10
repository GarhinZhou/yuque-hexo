---
title: ESP32
date: '2025-11-17 21:48:06'
updated: '2025-12-07 17:34:12'
---
将用户添加到有串口访问权限的用户组：

```rust
sudo usermod -a -G dialout $USER
```

```rust
Sketch uses 271400 bytes (20%) of program storage space. Maximum is 1310720 bytes.
Global variables use 11920 bytes (3%) of dynamic memory, leaving 315760 bytes for local variables. Maximum is 327680 bytes.
                                                                                
 Usage: esptool [OPTIONS] COMMAND [ARGS]...                                     
                                                                                
 Try 'esptool -h' for help                                                      
╭─ Error ──────────────────────────────────────────────────────────────────────╮
│ Invalid value for '--port' / '-p': Path '/dev/ttyACM0' is not readable.      │
╰──────────────────────────────────────────────────────────────────────────────╯
                                                                                
Failed uploading: uploading error: exit status 2
```

