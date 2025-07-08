# 主动推送音频 - 快速集成指南

## 5分钟快速集成

### 1. 编译和部署
代码已经修改完成，直接编译部署即可：
```bash
idf.py build
idf.py flash
```

### 2. 服务端测试
使用以下Python脚本测试推送功能：

```python
import json
import base64
import websocket
import threading
import time

class PushTester:
    def __init__(self, device_url):
        self.device_url = device_url
        self.ws = None
        
    def connect(self):
        self.ws = websocket.WebSocketApp(
            self.device_url,
            on_message=self.on_message,
            on_error=self.on_error,
            on_close=self.on_close
        )
        self.ws.on_open = self.on_open
        self.ws.run_forever()
    
    def on_open(self, ws):
        print("Connected to device")
        
    def on_message(self, ws, message):
        print(f"Received: {message}")
        
    def on_error(self, ws, error):
        print(f"Error: {error}")
        
    def on_close(self, ws, close_status_code, close_msg):
        print("Connection closed")
    
    def push_notification_sound(self):
        """推送通知音效"""
        message = {
            "type": "push_command",
            "command": "play_sound",
            "params": {
                "sound": "notification"
            }
        }
        self.ws.send(json.dumps(message))
        print("Pushed notification sound")
    
    def push_custom_audio(self, audio_file):
        """推送自定义音频文件"""
        try:
            with open(audio_file, 'rb') as f:
                audio_data = base64.b64encode(f.read()).decode('utf-8')
            
            message = {
                "type": "push_audio",
                "audio_data": audio_data
            }
            self.ws.send(json.dumps(message))
            print(f"Pushed custom audio: {audio_file}")
        except Exception as e:
            print(f"Error pushing audio: {e}")
    
    def push_message(self, text, emotion="neutral"):
        """推送显示消息"""
        message = {
            "type": "push_command",
            "command": "show_message",
            "params": {
                "message": text,
                "emotion": emotion
            }
        }
        self.ws.send(json.dumps(message))
        print(f"Pushed message: {text}")

# 使用示例
if __name__ == "__main__":
    tester = PushTester("ws://your-device-ip:port")
    
    # 在新线程中连接
    thread = threading.Thread(target=tester.connect)
    thread.daemon = True
    thread.start()
    
    # 等待连接建立
    time.sleep(2)
    
    # 测试推送功能
    tester.push_notification_sound()
    time.sleep(1)
    
    tester.push_message("Hello from server!", "happy")
    time.sleep(1)
    
    # 如果有音频文件，可以推送自定义音频
    # tester.push_custom_audio("notification.opus")
    
    # 保持连接
    try:
        while True:
            time.sleep(1)
    except KeyboardInterrupt:
        print("Exiting...")
```

### 3. 验证功能
1. 设备启动后，查看日志确认推送通道已启动
2. 运行测试脚本
3. 设备应该立即播放推送的音频，无需唤醒词

### 4. 配置推送通道（可选）
如果需要独立的推送通道，在设备配置中添加：
```json
{
  "push": {
    "url": "wss://your-push-server.com/device",
    "token": "your_token"
  }
}
```

## 常见问题

### Q: 推送音频没有播放？
A: 检查以下几点：
1. 音频格式是否为Base64编码的Opus
2. 网络连接是否正常
3. 设备日志是否有错误信息

### Q: 推送通道连接失败？
A: 检查：
1. 推送服务器地址是否正确
2. 网络是否可达
3. 认证token是否有效

### Q: 推送音频质量差？
A: 建议：
1. 使用16kHz采样率
2. 调整Opus编码比特率
3. 检查网络带宽

### Q: 如何优化推送性能？
A: 
1. 常用音频预先缓存
2. 使用较低的音频比特率
3. 控制推送频率

## 高级功能

### 1. 定时推送
```python
import schedule
import time

def push_hourly_reminder():
    tester.push_message("整点提醒", "neutral")
    tester.push_notification_sound()

schedule.every().hour.do(push_hourly_reminder)

while True:
    schedule.run_pending()
    time.sleep(1)
```

### 2. 条件推送
```python
# 根据设备状态推送
if device_state == "idle":
    tester.push_message("设备空闲中", "neutral")
elif device_state == "busy":
    tester.push_notification_sound()
```

### 3. 批量推送
```python
# 批量推送多个设备
devices = ["device1", "device2", "device3"]
for device in devices:
    tester = PushTester(f"ws://{device}:port")
    tester.push_notification_sound()
```

这个快速集成指南让您能够在5分钟内开始使用主动推送音频功能！