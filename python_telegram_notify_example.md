# Python发送Telegram通知示例

## 前期准备

### 1. 创建Telegram Bot
1. 在Telegram中搜索 `@BotFather`
2. 发送 `/newbot` 命令
3. 按照提示设置bot名称和用户名
4. 创建成功后，BotFather会发送一个包含API Token的消息，保存此Token

### 2. 获取Chat ID
1. 在Telegram中搜索并向你创建的bot发送一条消息
2. 访问 `https://api.telegram.org/bot<YourBOTToken>/getUpdates`
   - 将`<YourBOTToken>`替换为你的Bot Token
3. 在返回的JSON数据中找到 `"chat":{"id":123456789}` 中的数字，这就是你的Chat ID

## 代码实现

### 安装依赖
```bash
pip install requests
```

### 配置代理
由于网络环境限制，可能需要通过代理访问Telegram API，以下示例默认使用`127.0.0.1:7890`作为代理。

```python
# [ai]: 代理设置
proxies = {
    'http': 'http://127.0.0.1:7890',
    'https': 'http://127.0.0.1:7890'
}
```

### 简单发送文本消息
```python
import requests

def send_telegram_message(bot_token, chat_id, message, proxies=None):
    """
    发送Telegram消息
    
    参数:
        bot_token (str): Bot的API Token
        chat_id (str/int): 聊天ID
        message (str): 要发送的消息内容
        proxies (dict, optional): 代理设置，默认为None
    
    返回:
        bool: 是否发送成功
    """
    # [ai]: 构建API URL
    url = f"https://api.telegram.org/bot{bot_token}/sendMessage"
    
    # [ai]: 默认代理设置
    if proxies is None:
        proxies = {
            'http': 'http://127.0.0.1:7890',
            'https': 'http://127.0.0.1:7890'
        }
    
    # [ai]: 设置请求参数
    params = {
        "chat_id": chat_id,
        "text": message,
        "parse_mode": "Markdown"  # 支持Markdown格式
    }
    
    try:
        # [ai]: 发送POST请求，使用代理
        response = requests.post(url, params=params, proxies=proxies, timeout=10)
        
        # [ai]: 检查是否发送成功
        if response.status_code == 200:
            print("消息发送成功！")
            return True
        else:
            print(f"消息发送失败，状态码: {response.status_code}")
            print(f"错误信息: {response.text}")
            return False
    except Exception as e:
        print(f"发送消息时出错: {e}")
        return False

# 示例使用
if __name__ == "__main__":
    # [ai]: 替换为你的Bot Token和Chat ID
    BOT_TOKEN = "你的Bot Token"
    CHAT_ID = "你的Chat ID"
    
    # [ai]: 代理设置
    PROXIES = {
        'http': 'http://127.0.0.1:7890',
        'https': 'http://127.0.0.1:7890'
    }
    
    # [ai]: 构建消息内容
    message = """
*网站操作通知*

✅ 已完成操作
网站: `https://example.com`
操作类型: Connect Wallet, Sign Permit
执行时间: 2023-06-15 14:30:45
    """
    
    # [ai]: 发送消息，使用代理
    send_telegram_message(BOT_TOKEN, CHAT_ID, message, PROXIES)
```

### 发送带图片的消息
```python
def send_telegram_photo(bot_token, chat_id, photo_path, caption="", proxies=None):
    """
    发送带图片的Telegram消息
    
    参数:
        bot_token (str): Bot的API Token
        chat_id (str/int): 聊天ID
        photo_path (str): 图片文件路径
        caption (str): 图片说明文字
        proxies (dict, optional): 代理设置，默认为None
    
    返回:
        bool: 是否发送成功
    """
    url = f"https://api.telegram.org/bot{bot_token}/sendPhoto"
    
    # [ai]: 默认代理设置
    if proxies is None:
        proxies = {
            'http': 'http://127.0.0.1:7890',
            'https': 'http://127.0.0.1:7890'
        }
    
    # [ai]: 准备文件和数据
    files = {
        'photo': open(photo_path, 'rb')
    }
    
    data = {
        'chat_id': chat_id,
        'caption': caption,
        'parse_mode': 'Markdown'
    }
    
    try:
        # [ai]: 发送POST请求，使用代理
        response = requests.post(url, files=files, data=data, proxies=proxies, timeout=30)
        
        if response.status_code == 200:
            print("图片消息发送成功！")
            return True
        else:
            print(f"图片消息发送失败，状态码: {response.status_code}")
            print(f"错误信息: {response.text}")
            return False
    except Exception as e:
        print(f"发送图片消息时出错: {e}")
        return False
    finally:
        # [ai]: 关闭文件
        files['photo'].close()
```

## 集成到项目中的示例

```python
def notify_operation_completed(website_url, operation_type, screenshot_path=None):
    """
    操作完成后发送通知
    
    参数:
        website_url (str): 操作的网站URL
        operation_type (str): 操作类型
        screenshot_path (str, optional): 截图路径
    """
    import time
    
    # [ai]: 替换为实际的Token和Chat ID
    BOT_TOKEN = "你的Bot Token"
    CHAT_ID = "你的Chat ID"
    
    # [ai]: 代理设置
    PROXIES = {
        'http': 'http://127.0.0.1:7890',
        'https': 'http://127.0.0.1:7890'
    }
    
    # [ai]: 获取当前时间
    current_time = time.strftime("%Y-%m-%d %H:%M:%S")
    
    # [ai]: 构建消息
    message = f"""
*操作完成通知*

✅ 已完成网站操作
网站: `{website_url}`
操作类型: {operation_type}
执行时间: {current_time}
    """
    
    # [ai]: 如果有截图，发送带截图的消息
    if screenshot_path:
        send_telegram_photo(BOT_TOKEN, CHAT_ID, screenshot_path, message, PROXIES)
    else:
        # [ai]: 否则只发送文本消息
        send_telegram_message(BOT_TOKEN, CHAT_ID, message, PROXIES)
```

## 注意事项

1. 保持Bot Token安全，不要将其硬编码在代码中或公开在代码仓库里
2. 考虑将Token和Chat ID存储在环境变量或配置文件中
3. 网络问题可能导致消息发送失败，建议添加重试机制
4. Telegram API可能受到网络限制，确保你的服务器能够访问Telegram API
5. 代理设置可能需要根据实际网络环境调整，默认使用`127.0.0.1:7890`
6. 如果无法连接代理或连接超时，可尝试修改代理地址或检查代理服务是否正常运行
