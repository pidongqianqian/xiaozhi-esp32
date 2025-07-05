# 主动推送音频解决方案

## 方案概述

该方案支持设备主动接收服务端下发的音频并播放，不再依赖"你好小智"唤醒词才能播放音频。

## 核心特性

1. **独立推送通道**：与语音交互通道分离，保持长连接
2. **优先级播放**：推送音频优先播放，不被设备状态影响
3. **多种推送方式**：支持音频数据推送和命令推送
4. **兼容性**：可复用现有的WebSocket/MQTT协议

## 架构设计

### 1. 推送通道管理
- `StartPushChannel()`: 启动推送通道
- `StopPushChannel()`: 停止推送通道
- `IsPushChannelActive()`: 检查推送通道状态

### 2. 消息处理
- `HandlePushAudio()`: 处理推送音频数据
- `HandlePushCommand()`: 处理推送命令

### 3. 音频播放优先级
- 推送音频队列优先于正常音频队列
- 支持中断当前播放，立即播放推送音频

## 服务端接口

### 1. 推送音频消息格式
```json
{
  "type": "push_audio",
  "audio_data": "base64编码的Opus音频数据"
}
```

### 2. 推送命令消息格式
```json
{
  "type": "push_command",
  "command": "play_sound",
  "params": {
    "sound": "notification"
  }
}
```

支持的命令：
- `play_sound`: 播放预定义音效
- `show_message`: 显示消息
- `wake_device`: 主动唤醒设备

## 配置说明

### 1. 独立推送通道配置
在设备配置中添加`push`节点：
```json
{
  "push": {
    "url": "wss://push.example.com/device",
    "token": "your_push_token"
  }
}
```

### 2. 复用主通道配置
如果不配置独立推送地址，系统会自动复用主协议通道。

## 使用场景

1. **通知播报**：服务端主动推送通知音频
2. **定时提醒**：定时推送提醒音频
3. **紧急广播**：紧急情况下主动播放警报音频
4. **智能音效**：根据环境变化推送对应音效

## 实现细节

### 1. 音频解码
- 支持Base64编码的Opus音频数据
- 自动重采样适配设备输出采样率
- 独立的推送音频解码器

### 2. 线程安全
- 使用独立的推送音频队列和互斥锁
- 条件变量通知音频播放线程

### 3. 错误处理
- 网络错误自动重连
- 音频解码失败忽略
- 推送通道异常不影响主功能

## 示例代码

### 服务端推送音频
```python
import json
import base64
import websocket

# 读取音频文件并编码
with open('notification.opus', 'rb') as f:
    audio_data = base64.b64encode(f.read()).decode('utf-8')

# 构造推送消息
message = {
    "type": "push_audio",
    "audio_data": audio_data
}

# 发送到设备
ws.send(json.dumps(message))
```

### 服务端推送命令
```python
import json
import websocket

# 播放音效
message = {
    "type": "push_command",
    "command": "play_sound",
    "params": {
        "sound": "notification"
    }
}

ws.send(json.dumps(message))
```

## 部署建议

1. **网络稳定性**：确保推送通道网络稳定
2. **音频质量**：使用16kHz采样率的Opus编码
3. **推送频率**：避免过于频繁的推送影响用户体验
4. **权限控制**：在服务端实现推送权限管理

## 性能优化

1. **音频压缩**：使用低比特率Opus编码减少网络传输
2. **缓存机制**：常用音频可预先缓存到设备
3. **批量推送**：多个短音频可合并推送
4. **流量控制**：限制推送音频的大小和频率

## 兼容性

- 支持现有的WebSocket和MQTT协议
- 向后兼容，不影响现有唤醒词功能
- 可选择启用或禁用推送功能

## 监控和调试

- 推送通道状态监控
- 音频播放成功率统计
- 网络连接质量监控
- 详细的日志记录

这个解决方案提供了灵活的推送音频能力，同时保持了与现有架构的兼容性。