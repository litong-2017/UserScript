<template>
  <div class="captcha-recognition-container">
    <!-- 开发环境设置按钮 -->
    <div
      v-if="process.env.NODE_ENV === 'development' && !showSettings"
      class="dev-settings-button"
      @click="openSettings"
    >
      打开设置
    </div>

    <!-- 设置弹窗 -->
    <div v-if="showSettings" class="captcha-settings-modal">
      <div class="captcha-settings-content">
        <h3>验证码识别设置</h3>
        <div class="captcha-settings-item">
          <label>API类型:</label>
          <select v-model="settings.apiType">
            <option value="openai">OpenAI</option>
            <option value="gemini">Google Gemini</option>
          </select>
        </div>

        <div v-if="settings.apiType === 'openai'">
          <div class="captcha-settings-item">
            <label>OpenAI API Key:</label>
            <input
              type="text"
              v-model="settings.openaiKey"
              placeholder="sk-..."
            />
          </div>
          <div class="captcha-settings-item">
            <label>自定义API地址 (可选):</label>
            <input
              type="text"
              v-model="settings.openaiApiUrl"
              placeholder="https://api.openai.com/v1/chat/completions"
            />
            <small>留空使用默认地址</small>
          </div>
          <div class="captcha-settings-item">
            <label>模型 (可选):</label>
            <input
              type="text"
              v-model="settings.openaiModel"
              placeholder="gpt-4.1-mini"
            />
            <small>留空使用默认模型</small>
          </div>
          <div class="captcha-settings-item">
            <label>自定义提示词 (可选):</label>
            <textarea
              v-model="settings.openaiPrompt"
              placeholder="1.这是一个验证码图片,请识别图片中的文字,只返回识别结果,不要有任何其他文字或解释；
2.如果你识别到了是一道数学计算题（加减乘除），请进行计算，然后直接输出数字，不要有任何其他文字或解释；"
              rows="3"
            ></textarea>
            <small>留空使用默认提示词</small>
          </div>
        </div>

        <div v-if="settings.apiType === 'gemini'">
          <div class="captcha-settings-item">
            <label>Google Gemini API Key:</label>
            <input
              type="text"
              v-model="settings.geminiKey"
              placeholder="输入Gemini API Key"
            />
          </div>
          <div class="captcha-settings-item">
            <label>自定义API地址 (可选):</label>
            <input
              type="text"
              v-model="settings.geminiApiUrl"
              placeholder="https://generativelanguage.googleapis.com/v1beta/models"
            />
            <small>留空使用默认地址</small>
          </div>
          <div class="captcha-settings-item">
            <label>模型 (可选):</label>
            <input
              type="text"
              v-model="settings.geminiModel"
              placeholder="gemini-2.5-flash-lite"
            />
            <small>留空使用默认模型</small>
          </div>
          <div class="captcha-settings-item">
            <label>自定义提示词 (可选):</label>
            <textarea
              v-model="settings.geminiPrompt"
              placeholder="1.这是一个验证码图片,请识别图片中的文字,只返回识别结果,不要有任何其他文字或解释；
2.如果你识别到了是一道数学计算题（加减乘除），请进行计算，然后直接输出数字，不要有任何其他文字或解释；"
              rows="3"
            ></textarea>
            <small>留空使用默认提示词</small>
          </div>
        </div>

        <div class="captcha-settings-item">
          <label>自动识别:</label>
          <div style="display: flex; align-items: center">
            <input
              type="checkbox"
              v-model="settings.autoRecognize"
              id="autoRecognize"
              style="width: auto; margin-right: 8px"
            />
            <label for="autoRecognize" style="margin-bottom: 0"
              >验证码图片变化时自动识别</label
            >
          </div>
        </div>

        <div class="captcha-settings-item">
          <label>自动复制到剪贴板:</label>
          <div style="display: flex; align-items: center">
            <input
              type="checkbox"
              v-model="settings.copyToClipboard"
              id="copyToClipboard"
              style="width: auto; margin-right: 8px"
            />
            <label for="copyToClipboard" style="margin-bottom: 0"
              >自动复制到剪贴板</label
            >
          </div>
        </div>

        <div class="captcha-settings-buttons">
          <button @click="saveSettings">保存设置</button>
          <button @click="closeSettings">取消</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import packageJson from "../package.json";
import axios from "axios";

export default {
  data() {
    return {
      // 设置项
      settings: {
        apiType: "openai", // openai, gemini
        // OpenAI设置
        openaiKey: "",
        openaiApiUrl: "",
        openaiModel: "",
        openaiPrompt: "", // 自定义提示词
        // Gemini设置
        geminiKey: "",
        geminiApiUrl: "",
        geminiModel: "",
        geminiPrompt: "", // 自定义提示词
        // 自动识别设置
        autoRecognize: false, // 是否启用自动识别
        // 剪贴板设置
        copyToClipboard: true, // 是否自动复制到剪贴板
      },
      // 是否显示设置面板
      showSettings: false,
      // 配置选项
      config: {
        // 验证码图片选择器
        captchaSelector:
          'img[src*="captcha"], img[src*="verify"], img[alt*="验证码"], img[title*="点击刷新验证码"], img[alt*="captcha"], img[id="captchaPic"], .validate-code img',
        // 相关输入框选择器 (通常在验证码图片附近的输入框)
        inputSelector:
          'input[name*="captcha"], input[name*="verify"], input[placeholder*="验证码"], input[placeholder*="captcha"]',
      },
      // 用于在模板中访问环境变量
      process: {
        env: {
          NODE_ENV: process.env.NODE_ENV || "development",
        },
      },
    };
  },
  methods: {
    /**
     * 加载用户设置
     */
    loadSettings() {
      try {
        // 从油猴脚本存储中加载设置
        if (typeof GM_getValue !== "undefined") {
          const savedSettings = GM_getValue("captchaSettings");
          if (savedSettings) {
            this.settings = JSON.parse(savedSettings);
          }
        } else {
          // console.log('未检测到油猴环境，无法加载设置');
          // 尝试从localStorage加载设置（开发环境使用）
          const localSettings = localStorage.getItem("captchaSettings");
          if (localSettings) {
            this.settings = JSON.parse(localSettings);
          }
        }
      } catch (error) {
        console.error("加载设置失败:", error);
      }
    },

    /**
     * 显示设置面板
     */
    openSettings() {
      // 先添加类阻止body滚动
      document.body.classList.add("captcha-settings-open");
      // 显示设置面板
      this.showSettings = true;
    },

    /**
     * 关闭设置面板
     */
    closeSettings() {
      // 移除阻止body滚动的类
      document.body.classList.remove("captcha-settings-open");
      // 关闭设置面板
      this.showSettings = false;
    },

    /**
     * 保存用户设置
     */
    saveSettings() {
      try {
        // 保存设置到油猴脚本存储
        if (typeof GM_setValue !== "undefined") {
          GM_setValue("captchaSettings", JSON.stringify(this.settings));
        } else {
          // console.log('未检测到油猴环境，将设置保存到localStorage');
          // 保存到localStorage（开发环境使用）
          localStorage.setItem(
            "captchaSettings",
            JSON.stringify(this.settings)
          );
        }
        this.closeSettings();
        this.showToast("设置已保存！", "success");
      } catch (error) {
        console.error("保存设置失败:", error);
        this.showToast("保存设置失败，请查看控制台获取更多信息。", "error");
      }
    },

    /**
     * 使用AI识别验证码
     * @param {HTMLImageElement} captchaImg - 验证码图片元素
     * @returns {Promise<string>} - 识别结果
     */
    async recognizeCaptcha(captchaImg) {
      // 检查是否配置了API
      if (!this.isApiConfigured()) {
        this.showToast("请先配置验证码识别API", "error");
        this.openSettings();
        return "";
      }

      // 将图片转换为base64
      const base64Image = this.imageToBase64(captchaImg);

      // 根据API类型调用不同的识别方法
      let result = "";
      try {
        this.showToast("正在识别验证码...", "info");

        switch (this.settings.apiType) {
          case "openai":
            result = await this.recognizeWithOpenAI(base64Image);
            break;
          case "gemini":
            result = await this.recognizeWithGemini(base64Image);
            break;
        }

        if (result) {
          this.showToast(`识别成功: ${result}`, "success");
        } else {
          this.showToast("识别结果为空", "error");
        }

        return result;
      } catch (error) {
        console.error("验证码识别失败:", error);
        this.showToast("识别失败: " + (error.message || "未知错误"), "error");
        return "";
      }
    },

    /**
     * 检查API是否配置
     */
    isApiConfigured() {
      switch (this.settings.apiType) {
        case "openai":
          return !!this.settings.openaiKey;
        case "gemini":
          return !!this.settings.geminiKey;
        default:
          return false;
      }
    },

    /**
     * 将图片转换为base64格式
     */
    imageToBase64(imgElement) {
      // 创建canvas
      const canvas = document.createElement("canvas");
      canvas.width = imgElement.naturalWidth || imgElement.width;
      canvas.height = imgElement.naturalHeight || imgElement.height;

      // 在canvas上绘制图片
      const ctx = canvas.getContext("2d");
      ctx.drawImage(imgElement, 0, 0);

      // 转换为base64
      return canvas.toDataURL("image/png").split(",")[1];
    },

    /**
     * 使用OpenAI API识别验证码
     */
    async recognizeWithOpenAI(base64Image) {
      // 使用自定义API地址或默认地址
      const apiUrl =
        this.settings.openaiApiUrl ||
        "https://api.openai.com/v1/chat/completions";
      // 使用自定义模型或默认模型
      const model = this.settings.openaiModel || "gpt-4.1-mini";
      // 使用自定义提示词或默认提示词
      const prompt =
        this.settings.openaiPrompt ||
        `1.这是一个验证码图片,请识别图片中的文字,只返回识别结果,不要有任何其他文字或解释；
2.如果你识别到了是一道数学计算题（加减乘除），请进行计算，然后直接输出数字，不要有任何其他文字或解释；`;

      const response = await this.request({
        method: "POST",
        url: apiUrl,
        data: {
          model: model,
          messages: [
            {
              role: "user",
              content: [
                {
                  type: "text",
                  text: prompt,
                },
                {
                  type: "image_url",
                  image_url: {
                    url: `data:image/png;base64,${base64Image}`,
                  },
                },
              ],
            },
          ],
          max_tokens: 300,
        },
        headers: {
          "Content-Type": "application/json",
          Authorization: `Bearer ${this.settings.openaiKey}`,
        },
      });

      // 提取结果
      return response.data.choices[0].message.content.trim();
    },

    /**
     * 使用Google Gemini API识别验证码
     */
    async recognizeWithGemini(base64Image) {
      // 使用自定义模型或默认模型
      const model = this.settings.geminiModel || "gemini-2.5-flash-lite";
      // 使用自定义API地址或默认地址
      const baseApiUrl =
        this.settings.geminiApiUrl ||
        "https://generativelanguage.googleapis.com/v1beta/models";
      const apiUrl = `${baseApiUrl}/${model}:generateContent`;
      // 使用自定义提示词或默认提示词
      const prompt =
        this.settings.geminiPrompt ||
        `1.这是一个验证码图片,请识别图片中的文字,只返回识别结果,不要有任何其他文字或解释；
2.如果你识别到了是一道数学计算题（加减乘除），请进行计算，然后直接输出数字，不要有任何其他文字或解释；`;

      const response = await this.request({
        method: "POST",
        url: `${apiUrl}?key=${this.settings.geminiKey}`,
        data: {
          contents: [
            {
              parts: [
                {
                  text: prompt,
                },
                {
                  inline_data: {
                    mime_type: "image/png",
                    data: base64Image,
                  },
                },
              ],
            },
          ],
          generationConfig: {
            temperature: 0,
            maxOutputTokens: 100,
          },
        },
        headers: {
          "Content-Type": "application/json",
        },
      });

      // 提取结果
      if (response.data.candidates && response.data.candidates.length > 0) {
        const candidate = response.data.candidates[0];
        if (
          candidate.content &&
          candidate.content.parts &&
          candidate.content.parts.length > 0
        ) {
          const text = candidate.content.parts[0].text || "";
          // 只保留数字和字母
          return text.replace(/[^a-zA-Z0-9]/g, "").trim();
        }
      }

      return "";
    },

    /**
     * 查找页面上的验证码图片和相关输入框
     */
    findCaptchaElements() {
      const captchaImages = document.querySelectorAll(
        this.config.captchaSelector
      );
      if (captchaImages.length === 0) {
        // console.log('未找到验证码图片');
        return [];
      }

      const elements = [];
      captchaImages.forEach((img) => {
        // 查找最近的输入框
        let inputField = null;

        // 方法1：尝试通过选择器查找相关输入框
        const inputs = document.querySelectorAll(this.config.inputSelector);
        if (inputs.length > 0) {
          // 找到距离验证码图片最近的输入框
          let minDistance = Infinity;
          let closestInput = null;

          inputs.forEach((input) => {
            const distance = this.getDistance(img, input);
            if (distance < minDistance) {
              minDistance = distance;
              closestInput = input;
            }
          });

          // 如果找到的输入框距离验证码图片不太远，则认为它是相关的
          if (minDistance < 300) {
            inputField = closestInput;
          }
        }

        // 方法2：如果方法1未找到，尝试检查验证码图片附近的输入框
        if (!inputField) {
          // 向上查找
          let parent = img.parentElement;
          let level = 0;
          while (parent && level < 5) {
            const nearInputs = parent.querySelectorAll('input[type="text"]');
            if (nearInputs.length > 0) {
              inputField = nearInputs[0];
              break;
            }
            parent = parent.parentElement;
            level++;
          }
        }

        if (inputField) {
          elements.push({ captchaImg: img, inputField });
        }
      });

      return elements;
    },

    /**
     * 计算两个元素之间的距离
     */
    getDistance(el1, el2) {
      const rect1 = el1.getBoundingClientRect();
      const rect2 = el2.getBoundingClientRect();

      const x1 = rect1.left + rect1.width / 2;
      const y1 = rect1.top + rect1.height / 2;
      const x2 = rect2.left + rect2.width / 2;
      const y2 = rect2.top + rect2.height / 2;

      return Math.sqrt(Math.pow(x1 - x2, 2) + Math.pow(y1 - y2, 2));
    },

    /**
     * 在验证码图片旁添加识别图标
     */
    addIconsToCaptchas() {
      const elements = this.findCaptchaElements();

      elements.forEach(({ captchaImg, inputField }) => {
        // 检查是否已经添加过图标
        const existingIcon = captchaImg.nextElementSibling;
        if (
          existingIcon &&
          existingIcon.classList.contains("captcha-recognition-icon")
        ) {
          return;
        }

        // 创建识别图标
        const icon = document.createElement("div");
        icon.classList.add("captcha-recognition-icon");
        icon.title = "点击识别验证码";

        // 将图标放在验证码图片后面（紧邻其后）
        if (captchaImg.nextSibling) {
          captchaImg.parentNode.insertBefore(icon, captchaImg.nextSibling);
        } else {
          captchaImg.parentNode.appendChild(icon);
        }

        // 添加点击事件
        icon.addEventListener("click", async () => {
          this.processCaptcha(captchaImg, inputField, icon);
        });
      });
    },

    /**
     * 处理验证码识别
     */
    async processCaptcha(captchaImg, inputField, icon) {
      try {
        // 更新图标状态为加载中
        icon.classList.add("captcha-recognition-loading");

        // 识别验证码
        const text = await this.recognizeCaptcha(captchaImg);

        // 如果识别结果为空，显示错误
        if (!text) {
          icon.classList.remove("captcha-recognition-loading");
          icon.classList.add("captcha-recognition-error");
          setTimeout(() => {
            icon.classList.remove("captcha-recognition-error");
          }, 2000);
          return;
        }

        // 填充结果到输入框
        inputField.value = text;
        // 触发输入事件，以便触发表单验证
        inputField.dispatchEvent(new Event("input", { bubbles: true }));
        inputField.dispatchEvent(new Event("change", { bubbles: true }));

        // 将识别结果复制到剪贴板
        if (this.settings.copyToClipboard) {
          try {
            await navigator.clipboard.writeText(text);
            this.showToast(`验证码已识别: ${text} (已复制到剪贴板)`, "success");
          } catch (clipboardError) {
            // 如果剪贴板API不可用，使用传统方法
            const textarea = document.createElement("textarea");
            textarea.value = text;
            textarea.style.position = "fixed";
            textarea.style.opacity = "0";
            document.body.appendChild(textarea);
            textarea.select();
            document.execCommand("copy");
            document.body.removeChild(textarea);
            this.showToast(`验证码已识别: ${text} (已复制到剪贴板)`, "success");
          }
        } else {
          this.showToast(`验证码已识别: ${text}`, "success");
        }

        // 更新图标状态为成功
        icon.classList.remove("captcha-recognition-loading");
        icon.classList.add("captcha-recognition-success");
        setTimeout(() => {
          icon.classList.remove("captcha-recognition-success");
        }, 2000);
      } catch (error) {
        console.error("验证码识别失败:", error);
        icon.classList.remove("captcha-recognition-loading");
        icon.classList.add("captcha-recognition-error");
        setTimeout(() => {
          icon.classList.remove("captcha-recognition-error");
        }, 2000);

        this.showToast(
          "处理验证码失败: " + (error.message || "未知错误"),
          "error"
        );
      }
    },

    /**
     * 监听DOM变化，自动为新添加的验证码添加识别图标
     */
    setupMutationObserver() {
      const observer = new MutationObserver((mutations) => {
        let hasNewCaptcha = false;
        let newCaptchaElements = [];

        mutations.forEach((mutation) => {
          // 检查新增节点
          if (mutation.type === "childList" && mutation.addedNodes.length) {
            mutation.addedNodes.forEach((node) => {
              if (node.nodeType === Node.ELEMENT_NODE) {
                const captchas = node.querySelectorAll(
                  this.config.captchaSelector
                );
                if (captchas.length > 0) {
                  hasNewCaptcha = true;
                  captchas.forEach((captcha) => {
                    newCaptchaElements.push(captcha);
                  });
                }
              }
            });
          }

          // 检查属性变化 - 特别是src属性变化的验证码图片
          if (
            mutation.type === "attributes" &&
            mutation.attributeName === "src" &&
            mutation.target.matches &&
            mutation.target.matches(this.config.captchaSelector)
          ) {
            hasNewCaptcha = true;
            newCaptchaElements.push(mutation.target);
          }
        });

        if (hasNewCaptcha) {
          this.addIconsToCaptchas();

          // 对新检测到的验证码元素进行自动识别处理
          // 只有当启用了自动识别功能时才执行
          if (this.settings.autoRecognize) {
            setTimeout(() => {
              const elements = this.findCaptchaElements();
              elements.forEach(({ captchaImg, inputField }) => {
                // 检查这个验证码图片是否是新添加或变化的
                if (newCaptchaElements.includes(captchaImg)) {
                  // 查找或创建识别图标
                  let icon;
                  const existingIcon = captchaImg.nextElementSibling;
                  if (
                    existingIcon &&
                    existingIcon.classList.contains("captcha-recognition-icon")
                  ) {
                    icon = existingIcon;
                  } else {
                    icon = document.createElement("div");
                    icon.classList.add("captcha-recognition-icon");
                    if (captchaImg.nextSibling) {
                      captchaImg.parentNode.insertBefore(
                        icon,
                        captchaImg.nextSibling
                      );
                    } else {
                      captchaImg.parentNode.appendChild(icon);
                    }
                  }

                  // 自动进行识别
                  this.processCaptcha(captchaImg, inputField, icon);
                }
              });
            }, 500); // 延迟500ms，确保图片已经加载完成
          }
        }
      });

      // 监听整个文档的变化，包括子树和属性变化
      observer.observe(document.body, {
        childList: true,
        subtree: true,
        attributes: true,
        attributeFilter: ["src"], // 只监听src属性变化
      });
    },

    /**
     * 注册油猴菜单
     */
    registerMenuCommands() {
      // 检查是否在油猴环境中
      if (typeof GM_registerMenuCommand !== "undefined") {
        GM_registerMenuCommand("验证码识别设置", () => {
          this.openSettings();
        });
      } else {
        // console.log('未检测到油猴环境，跳过菜单注册');
      }
    },

    /**
     * 初始化插件
     */
    init() {
      this.registerMenuCommands();
      this.loadSettings();
      setTimeout(() => {
        this.addIconsToCaptchas(); // 步骤1: 确保所有图标都已创建
        this.setupMutationObserver();
        const elements = this.findCaptchaElements();

        if (elements.length > 0) {
          // 新增逻辑: 检查初始加载时的自动识别设置
          if (this.settings.autoRecognize) {
            this.showToast(
              `检测到 ${elements.length} 个验证码，正在自动识别...`,
              "info"
            );
            // 遍历所有找到的验证码并处理
            elements.forEach(({ captchaImg, inputField }) => {
              const icon = captchaImg.nextElementSibling;
              // 确保图标元素存在
              if (icon && icon.classList.contains("captcha-recognition-icon")) {
                // 直接调用处理函数，无需等待点击
                this.processCaptcha(captchaImg, inputField, icon);
              }
            });
          } else {
            // 保留原始行为: 如果未开启自动识别，则提示用户点击
            this.showToast(
              `检测到 ${elements.length} 个验证码，点击识别图标开始识别`,
              "info"
            );
          }
        }
      }, 1e3);
    },

    /**
     * 通用请求函数，自动根据环境使用GM_xmlhttpRequest或axios
     * @param {Object} config - 请求配置
     * @returns {Promise} - 请求结果
     */
    request(config) {
      // 检查是否在油猴环境中
      if (typeof GM_xmlhttpRequest !== "undefined") {
        return new Promise((resolve, reject) => {
          GM_xmlhttpRequest({
            method: config.method || "GET",
            url: config.url,
            data: config.data
              ? typeof config.data === "string"
                ? config.data
                : JSON.stringify(config.data)
              : undefined,
            headers: config.headers || {},
            responseType: config.responseType || "json",
            onload: (response) => {
              if (response.status >= 200 && response.status < 300) {
                let responseData;
                try {
                  responseData =
                    config.responseType === "json" &&
                    typeof response.response === "string"
                      ? JSON.parse(response.response || response.responseText)
                      : response.response || response.responseText;
                } catch (e) {
                  responseData = response.response || response.responseText;
                }
                resolve({ data: responseData });
              } else {
                reject(new Error(`请求失败，状态码: ${response.status}`));
              }
            },
            onerror: (error) => {
              reject(error);
            },
          });
        });
      } else {
        // 在非油猴环境中，移除可能导致问题的头部
        const safeHeaders = { ...config.headers };
        const unsafeHeaders = ["Host", "Origin", "Referer", "Cookie"];

        unsafeHeaders.forEach((header) => {
          if (safeHeaders[header]) {
            delete safeHeaders[header];
          }
        });

        // 使用axios
        return axios({
          method: config.method || "GET",
          url: config.url,
          data: config.data,
          params: config.params,
          headers: safeHeaders,
          responseType: config.responseType,
        });
      }
    },

    /**
     * 显示Toast提示
     * @param {string} message - 提示信息
     * @param {string} type - 提示类型 (success, error, info)
     */
    showToast(message, type = "info") {
      // 创建toast容器（如果不存在）
      let toastContainer = document.getElementById("captcha-toast-container");
      if (!toastContainer) {
        toastContainer = document.createElement("div");
        toastContainer.id = "captcha-toast-container";
        document.body.appendChild(toastContainer);
      }

      // 创建新的toast元素
      const toast = document.createElement("div");
      toast.className = `captcha-toast captcha-toast-${type}`;
      toast.textContent = message;

      // 将新toast插入到容器的最前面
      if (toastContainer.firstChild) {
        toastContainer.insertBefore(toast, toastContainer.firstChild);
      } else {
        toastContainer.appendChild(toast);
      }

      // 添加显示动画
      setTimeout(() => {
        toast.classList.add("captcha-toast-show");
      }, 10);

      // 3秒后移除
      setTimeout(() => {
        toast.classList.remove("captcha-toast-show");
        toast.classList.add("captcha-toast-hide");

        // 动画结束后移除元素
        setTimeout(() => {
          if (toast.parentNode) {
            toast.parentNode.removeChild(toast);
          }
        }, 300); // 300ms是动画持续时间
      }, 3000);
    },
  },
  mounted() {
    this.init();
  },
};
</script>
