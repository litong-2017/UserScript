## CAPTCHA-automatic-recognition（AI 验证码自动识别填充）

[GitHub](https://github.com/ezyshu/UserScript/tree/main/CAPTCHA-automatic-recognition) |
[Greasyfork](https://greasyfork.org/scripts/540822) |
[更新日志](https://github.com/ezyshu/UserScript/tree/main/CAPTCHA-automatic-recognition/version-log.md)

![Greasy Fork Version](https://img.shields.io/greasyfork/v/540822)
![Greasy Fork Downloads](https://img.shields.io/greasyfork/dt/540822)

AI 自动识别网页验证码，支持各大主流厂商，限制视觉模型。（推荐 gemini-2.5-flash-lite，速度较快、稳定性较高）

- OpenAI
- Gemini
- 阿里云通义千问

![image](https://github.com/user-attachments/assets/c3a24ea0-7ce9-4e2f-930b-73f3951197d5)

### 1. 功能

- 自动识别网页验证码（支持图片变化自动识别）
- 验证码图片后出现识别按钮，一键识别并填充
- 支持主流AI服务商（OpenAI / Gemini / 通义千问），可自定义API地址和模型
- 全局配置一次，所有网站通用
- 可设置禁用特定网站的验证码识别（支持通配符/正则）
- 适配各种网站UI，界面自适应且不被网站样式影响

### 2. 安装方法

1. 安装 [Tampermonkey](https://www.tampermonkey.net/) 或 [Violentmonkey](https://violentmonkey.github.io/)
2. 安装本脚本

### 3. 使用方法

1. 进入任意有验证码的网页
2. 点击验证码图片后的小图标，首次使用会弹出设置面板
3. 配置好 API Key 和模型，保存即可
4. 如需自动识别功能，请在设置面板中勾选"验证码图片变化时自动识别"
5. 对于使用框架模板语法的网站，可开启"自动复制到剪贴板"功能
6. 之后所有网站都可直接用，无需重复配置

### 4. LICENSE

[Apache-2.0](https://github.com/ezyshu/UserScript/blob/main/CAPTCHA-automatic-recognition/LICENSE)

---

If you think this feature is good, please give it a star to show your support~

如果你认为这个功能不错，可以点个 star 支持一下~
