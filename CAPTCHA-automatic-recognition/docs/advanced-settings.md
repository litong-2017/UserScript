# 高级设置

需要具备基础的前端知识，譬如 HTML 标签, CSS 选择器之类的。

> ⚠️ 警告：如果您不了解 CSS 选择器，请不要修改这些设置，可能导致识别功能失效

---

**教程：**

打开 F12 查看你需要识别的网站验证码结构，越详细越好，避免与其他网站冲突。

### 1. 查看验证码图案结构

如图所示，这个图片的标签为 img，根据上述原则，这里我可以通过下面的选择器获取到图片信息。

```css
/* 这种方式获取或许会影响其他的网站显示 */
img
.test-code img

/* 推荐：比较准确的获取到当前网站的验证码标签 */
.test-code img[alt="code1"]
.test-code img[title="code1"]
.test-code img[class="code1"]

/* 只要alt、title、class中包含 `code1` 字符即可获取 */
.test-code img[alt*="code1"]
.test-code img[title*="code1"]
.test-code img[class*="code1"]
```

![image](https://github.com/user-attachments/assets/76f513af-7373-4a9d-9e58-3605ff396aae)

### 2. 查看输入框结构

一样的道理，可以获取到输入框的 HTML 结构；

```css
input[name="captcha"]
```

---

切记，如果你不知道如何使用这些功能，关闭高级设置！
