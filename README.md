vad_v2代码：

import pyaudio
import numpy as np
import pyautogui
import time

# --- 配置参数 ---
THRESHOLD = 500  # 灵敏度阈值，如果没反应就调低点，如果太敏感就调高点
CHUNK = 1024
FORMAT = pyaudio.paInt16
CHANNELS = 1
RATE = 44100
SILENCE_DURATION = 2  # 停止说话后多久关闭监听(秒)

p = pyaudio.PyAudio()
stream = p.open(format=FORMAT, channels=CHANNELS, rate=RATE, input=True, frames_per_buffer=CHUNK)

is_speaking = False
last_sound_time = time.time()

print("Echo VAD F2 自动化脚本已启动...")
print("警告：开启此模式将极速消耗 AIB 余额！")

try:
    while True:
        try:
            data = np.frombuffer(stream.read(CHUNK, exception_on_overflow=False), dtype=np.int16).astype(np.float32)
            rms = np.sqrt(np.mean(data**2))
            
            # 可视化音量条 (0-2000 范围映射到 40 个字符)
            bar_len = min(40, int(rms / 50))
            print(f"\r音量: [{'#' * bar_len}{'-' * (40 - bar_len)}] {rms:.1f} ", end="", flush=True)
            
            if rms > THRESHOLD:
                last_sound_time = time.time()
                if not is_speaking:
                    print("[Voice Detected] 触发 F2 开启监听")
                    pyautogui.press('f2')
                    is_speaking = True
            else:
                if is_speaking and (time.time() - last_sound_time > SILENCE_DURATION):
                    print("[Silence] 触发 F2 关闭监听")
                    pyautogui.press('f2')
                    is_speaking = False
        except Exception as e:
            print(f"错误: {e}")
            continue
except KeyboardInterrupt:
    print("\n脚本已停止。")
finally:
    stream.stop_stream()
    stream.close()
    p.terminate()


桌面快捷

@echo off
chcp 936 >nul
title Echo VAD 启动器
echo [Echo] 正在唤醒顺风耳...
python "G:\SteamLibrary\steamapps\common\你的ai桌宠\resources\backend\workspace\skills\vad_f2.py"
if 0 neq 0 (
    echo.
    echo [错误] 脚本启动失败！请确保 Python 3.12 安装正确。
    pause
)
pause





