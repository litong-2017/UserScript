## CAPTCHA-automatic-recognition（AI验证码自动识别填充）

自动识别网页验证码并填充输入框，支持 OpenAI 和 Google Gemini。

![image](https://github.com/user-attachments/assets/a2987157-e6cd-47cb-92d4-0a060b5b37c8)

### 功能
- 自动检测验证码图片
- 验证码图片后出现识别按钮，点击即可识别并填充
- 支持 OpenAI / Gemini，支持自定义API地址和模型
- 全局配置一次，所有网站通用

### 安装方法
1. 安装 [Tampermonkey](https://www.tampermonkey.net/) 或 [Violentmonkey](https://violentmonkey.github.io/)
2. 安装本脚本

### 使用方法
1. 进入任意有验证码的网页
2. 点击验证码图片后的小图标，首次使用会弹出设置面板
3. 配置好API Key和模型，保存即可
4. 之后所有网站都可直接用，无需重复配置

### 许可证
Apache-2.0
