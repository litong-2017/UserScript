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
        <h3>
          验证码识别设置 <span>{{ packageJson.version }}</span>
        </h3>
        <div class="captcha-settings-item">
          <label>API 类型：</label>
          <select v-model="settings.apiType">
            <option value="openai">OpenAI</option>
            <option value="gemini">Google Gemini</option>
            <option value="qwen">阿里云通义千问</option>
          </select>
        </div>

        <div v-if="settings.apiType === 'openai'">
          <div class="captcha-settings-item">
            <label>OpenAI API Key:</label>
            <div class="input-with-button">
              <input type="text" v-model="settings.openaiKey" placeholder="sk-..." />
              <button
                type="button"
                class="test-api-button"
                :class="{
                  'test-loading': apiTestStatus.openai === 'loading',
                  'test-success': apiTestStatus.openai === 'success',
                  'test-error': apiTestStatus.openai === 'error',
                }"
                @click="testApiConnection('openai')"
              >
                <span v-if="apiTestStatus.openai === ''">测试连接</span>
                <span v-else-if="apiTestStatus.openai === 'loading'"></span>
                <span v-else-if="apiTestStatus.openai === 'success'">成功</span>
                <span v-else-if="apiTestStatus.openai === 'error'">失败</span>
              </button>
            </div>
          </div>
          <div class="captcha-settings-item">
            <label>自定义 API 地址 (可选):</label>
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
            <div class="textarea-with-button">
              <textarea
                v-model="settings.openaiPrompt"
                placeholder="输入自定义提示词，或点击右侧按钮使用默认提示词"
                rows="3"
              ></textarea>
              <button
                type="button"
                class="use-default-prompt"
                @click="settings.openaiPrompt = DEFAULT_PROMPT"
              >
                使用默认
              </button>
            </div>
            <small>留空使用默认提示词</small>
          </div>
        </div>

        <div v-if="settings.apiType === 'gemini'">
          <div class="captcha-settings-item">
            <label>Google Gemini API Key:</label>
            <div class="input-with-button">
              <input
                type="text"
                v-model="settings.geminiKey"
                placeholder="输入Gemini API Key"
              />
              <button
                type="button"
                class="test-api-button"
                :class="{
                  'test-loading': apiTestStatus.gemini === 'loading',
                  'test-success': apiTestStatus.gemini === 'success',
                  'test-error': apiTestStatus.gemini === 'error',
                }"
                @click="testApiConnection('gemini')"
              >
                <span v-if="apiTestStatus.gemini === ''">测试连接</span>
                <span v-else-if="apiTestStatus.gemini === 'loading'"></span>
                <span v-else-if="apiTestStatus.gemini === 'success'">成功</span>
                <span v-else-if="apiTestStatus.gemini === 'error'">失败</span>
              </button>
            </div>
          </div>
          <div class="captcha-settings-item">
            <label>自定义 API 地址 (可选):</label>
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
            <div class="textarea-with-button">
              <textarea
                v-model="settings.geminiPrompt"
                placeholder="输入自定义提示词，或点击右侧按钮使用默认提示词"
                rows="3"
              ></textarea>
              <button
                type="button"
                class="use-default-prompt"
                @click="settings.geminiPrompt = DEFAULT_PROMPT"
              >
                使用默认
              </button>
            </div>
            <small>留空使用默认提示词</small>
          </div>
        </div>

        <div v-if="settings.apiType === 'qwen'">
          <div class="captcha-settings-item">
            <label>阿里云通义千问 API Key:</label>
            <div class="input-with-button">
              <input type="text" v-model="settings.qwenKey" placeholder="API Key" />
              <button
                type="button"
                class="test-api-button"
                :class="{
                  'test-loading': apiTestStatus.qwen === 'loading',
                  'test-success': apiTestStatus.qwen === 'success',
                  'test-error': apiTestStatus.qwen === 'error',
                }"
                @click="testApiConnection('qwen')"
              >
                <span v-if="apiTestStatus.qwen === ''">测试连接</span>
                <span v-else-if="apiTestStatus.qwen === 'loading'"></span>
                <span v-else-if="apiTestStatus.qwen === 'success'">成功</span>
                <span v-else-if="apiTestStatus.qwen === 'error'">失败</span>
              </button>
            </div>
          </div>
          <div class="captcha-settings-item">
            <label>自定义 API 地址 (可选):</label>
            <input
              type="text"
              v-model="settings.qwenApiUrl"
              placeholder="https://dashscope.aliyuncs.com/compatible-mode/v1/chat/completions"
            />
            <small>留空使用默认地址</small>
          </div>
          <div class="captcha-settings-item">
            <label>模型 (可选):</label>
            <input
              type="text"
              v-model="settings.qwenModel"
              placeholder="qwen-vl-max-2025-04-02"
            />
            <small>留空使用默认模型</small>
          </div>
          <div class="captcha-settings-item">
            <label>自定义提示词 (可选):</label>
            <div class="textarea-with-button">
              <textarea
                v-model="settings.qwenPrompt"
                placeholder="输入自定义提示词，或点击右侧按钮使用默认提示词"
                rows="3"
              ></textarea>
              <button
                type="button"
                class="use-default-prompt"
                @click="settings.qwenPrompt = DEFAULT_PROMPT"
              >
                使用默认
              </button>
            </div>
            <small>留空使用默认提示词</small>
          </div>
        </div>

        <div class="captcha-settings-item">
          <label>自动识别：</label>
          <div style="display: flex; align-items: center">
            <input
              type="checkbox"
              v-model="settings.autoRecognize"
              id="autoRecognize"
              style="width: auto; margin-right: 8px !important"
            />
            <label for="autoRecognize" style="margin-bottom: 0"
              >验证码图片变化时自动识别</label
            >
          </div>
        </div>

        <div class="captcha-settings-item">
          <label>自动复制到剪贴板：</label>
          <div style="display: flex; align-items: center">
            <input
              type="checkbox"
              v-model="settings.copyToClipboard"
              id="copyToClipboard"
              style="width: auto; margin-right: 8px !important"
            />
            <label for="copyToClipboard" style="margin-bottom: 0">自动复制到剪贴板</label>
          </div>
        </div>

        <div class="captcha-settings-item">
          <label>右上角通知：</label>
          <div style="display: flex; align-items: center">
            <input
              type="checkbox"
              v-model="settings.showNotification"
              id="showNotification"
              style="width: auto; margin-right: 8px !important"
            />
            <label for="showNotification" style="margin-bottom: 0"
              >显示右上角通知提示</label
            >
          </div>
        </div>

        <!-- 高级设置折叠面板 -->
        <div class="captcha-settings-item">
          <div class="advanced-settings-header" @click="toggleAdvancedSettings">
            <span
              >高级设置
              <a
                href="https://github.com/ezyshu/UserScript/tree/main/CAPTCHA-automatic-recognition/docs/advanced-settings.md"
                target="_blank"
                class="tutorial-link"
                @click.stop
                >教程</a
              ></span
            >
            <span class="toggle-icon" :class="{ expanded: showAdvancedSettings }">▶</span>
          </div>

          <div v-if="showAdvancedSettings" class="advanced-settings-content">
            <div class="advanced-settings-warning">
              ⚠️ 警告：如果您不了解CSS选择器，请不要修改这些设置，可能导致识别功能失效
            </div>

            <div class="captcha-settings-item">
              <label>自定义验证码图片选择器：</label>
              <div class="custom-selectors">
                <div
                  v-for="(selector, index) in settings.customCaptchaSelectors"
                  :key="'captcha-' + index"
                  class="selector-item"
                >
                  <input
                    type="text"
                    v-model="settings.customCaptchaSelectors[index]"
                    placeholder="例如: img[src*='captcha']"
                  />
                  <button
                    type="button"
                    class="remove-selector"
                    @click="removeSelector('captcha', index)"
                  >
                    ×
                  </button>
                </div>
                <button
                  type="button"
                  class="add-selector"
                  @click="addSelector('captcha')"
                >
                  添加选择器
                </button>
              </div>
            </div>

            <div class="captcha-settings-item">
              <label>自定义输入框选择器：</label>
              <div class="custom-selectors">
                <div
                  v-for="(selector, index) in settings.customInputSelectors"
                  :key="'input-' + index"
                  class="selector-item"
                >
                  <input
                    type="text"
                    v-model="settings.customInputSelectors[index]"
                    placeholder="例如: input[name*='captcha']"
                  />
                  <button
                    type="button"
                    class="remove-selector"
                    @click="removeSelector('input', index)"
                  >
                    ×
                  </button>
                </div>
                <button type="button" class="add-selector" @click="addSelector('input')">
                  添加选择器
                </button>
              </div>
            </div>
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
import { DEFAULT_PROMPT } from "./assets/prompts.js";

export default {
  data() {
    return {
      packageJson: packageJson,
      // 导入的常量
      DEFAULT_PROMPT,
      // API 测试状态
      apiTestStatus: {
        openai: "", // 可能的值：'', 'loading', 'success', 'error'
        gemini: "", // 可能的值：'', 'loading', 'success', 'error'
        qwen: "", // 可能的值：'', 'loading', 'success', 'error'
      },
      // 设置项
      settings: {
        apiType: "openai", // openai, gemini, qwen
        // OpenAI 设置
        openaiKey: "",
        openaiApiUrl: "",
        openaiModel: "",
        openaiPrompt: DEFAULT_PROMPT, // 自定义提示词，默认填充
        // Gemini 设置
        geminiKey: "",
        geminiApiUrl: "",
        geminiModel: "",
        geminiPrompt: DEFAULT_PROMPT, // 自定义提示词，默认填充
        // 通义千问设置
        qwenKey: "",
        qwenApiUrl: "",
        qwenModel: "",
        qwenPrompt: DEFAULT_PROMPT, // 自定义提示词，默认填充
        // 自动识别设置
        autoRecognize: false, // 是否启用自动识别
        // 剪贴板设置
        copyToClipboard: true, // 是否自动复制到剪贴板
        // 通知设置
        showNotification: true, // 是否显示右上角通知，默认开启
        // 自定义选择器
        customCaptchaSelectors: [],
        customInputSelectors: [],
      },
      // 是否显示设置面板
      showSettings: false,
      // 是否显示高级设置
      showAdvancedSettings: false,
      // 配置选项
      config: {
        // 验证码图片选择器
        captchaSelectors: [
          'img[src*="captcha"]',
          'img[src*="verify"]',
          'img[alt*="验证码"]',
          'img[title*="验证码"]',
          'img[alt*="captcha"]',
          'img[id="captchaPic"]',
          ".validate-code img",
          'img[src*="/login_check_code.php"]',
          'img[style="z-index: 2; position: absolute; bottom: -11px; left: 206px; width: 88px; height: 40px;"]',
          '.authcode img[id="authImage"]',
          'img[class="verification-img"]',
        ],
        // 相关输入框选择器 (通常在验证码图片附近的输入框)
        inputSelectors: [
          'input[name*="captcha"]',
          'input[name*="verify"]',
          'input[placeholder*="验证码"]',
          'input[placeholder*="captcha"]',
          'input[id="authcode"]',
        ],
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
            const parsedSettings = JSON.parse(savedSettings);
            // 合并设置，确保新增字段也有默认值
            this.settings = { ...this.settings, ...parsedSettings };
          }
        } else {
          // console.log('未检测到油猴环境，无法加载设置');
          // 尝试从 localStorage 加载设置（开发环境使用）
          const localSettings = localStorage.getItem("captchaSettings");
          if (localSettings) {
            const parsedSettings = JSON.parse(localSettings);
            // 合并设置，确保新增字段也有默认值
            this.settings = { ...this.settings, ...parsedSettings };
          }
        }
      } catch (error) {
        console.error("加载设置失败：", error);
      }
    },

    /**
     * 显示设置面板
     */
    openSettings() {
      // 先添加类阻止 body 滚动
      document.body.classList.add("captcha-settings-open");
      // 显示设置面板
      this.showSettings = true;
    },

    /**
     * 关闭设置面板
     */
    closeSettings() {
      // 移除阻止 body 滚动的类
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
          // console.log('未检测到油猴环境，将设置保存到 localStorage');
          // 保存到 localStorage（开发环境使用）
          localStorage.setItem("captchaSettings", JSON.stringify(this.settings));
        }
        this.closeSettings();
        this.showToast("设置已保存！", "success");
      } catch (error) {
        console.error("保存设置失败：", error);
        this.showToast("保存设置失败，请查看控制台获取更多信息。", "error");
      }
    },

    /**
     * 使用 AI 识别验证码
     * @param {string} base64Image - 验证码图片的 base64 编码
     * @returns {Promise<string>} - 识别结果
     */
    async recognizeCaptcha(base64Image) {
      // 检查是否配置了 API
      if (!this.isApiConfigured()) {
        this.showToast("请先配置验证码识别 API", "error");
        this.openSettings();
        return "";
      }

      // 根据 API 类型调用不同的识别方法
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
          case "qwen":
            result = await this.recognizeWithQwen(base64Image);
            break;
        }

        if (result) {
          this.showToast(`识别成功：${result}`, "success");
        } else {
          this.showToast("识别结果为空", "error");
        }

        return result;
      } catch (error) {
        console.error("验证码识别失败：", error);
        this.showToast("识别失败：" + (error.message || "未知错误"), "error");
        return "";
      }
    },

    /**
     * 检查 API 是否配置
     * @param {string} specificType - 指定要检查的 API 类型，不传则检查当前选中的 API
     * @returns {boolean} - API 是否已配置
     */
    isApiConfigured(specificType) {
      const typeToCheck = specificType || this.settings.apiType;
      switch (typeToCheck) {
        case "openai":
          return !!this.settings.openaiKey;
        case "gemini":
          return !!this.settings.geminiKey;
        case "qwen":
          return !!this.settings.qwenKey;
        default:
          return false;
      }
    },

    /**
     * 将图片转换为 base64 格式
     * @param {HTMLImageElement} imgElement - 图片元素
     * @returns {Object} - 返回包含成功状态和数据的对象
     */
    imageToBase64(imgElement) {
      try {
        // 检查图片来源
        const imgSrc = imgElement.src;

        // 如果图片来源是外部链接且跨域（非 data:image 开头，且不是同域）
        if (
          !imgSrc.startsWith("data:image") &&
          !this.isSameOrigin(imgSrc) &&
          imgElement.crossOrigin != "anonymous"
        ) {
          imgElement.crossOrigin = "anonymous";
          imgElement.src = imgSrc;
        }
        /*if (!imgSrc.startsWith("data:image") && !this.isSameOrigin(imgSrc)) {
            return {
              success: false,
              message: "抱歉，检测到跨域图片链接"
            };
          }*/

        // 创建 canvas
        const canvas = document.createElement("canvas");
        canvas.width = imgElement.naturalWidth || imgElement.width;
        canvas.height = imgElement.naturalHeight || imgElement.height;

        // 在 canvas 上绘制图片
        const ctx = canvas.getContext("2d");

        // 尝试绘制图片
        try {
          ctx.drawImage(imgElement, 0, 0);
          // 测试能否访问图片数据（如果是跨域图片会抛出错误）
          ctx.getImageData(0, 0, 1, 1);
        } catch (e) {
          return {
            success: false,
            message: "无法读取图片数据，可能是跨域限制。请尝试手动下载验证码图片后识别。",
          };
        }

        // 转换为 base64
        const base64Data = canvas.toDataURL("image/png").split(",")[1];

        // 检查 base64 数据是否有效
        if (!base64Data || base64Data.length < 100) {
          return {
            success: false,
            message: "图片转换失败或内容为空。请刷新验证码后重试。",
          };
        }

        return {
          success: true,
          data: base64Data,
        };
      } catch (error) {
        console.error("图片转base64失败:", error);
        return {
          success: false,
          message: "图片转换失败: " + (error.message || "未知错误"),
        };
      }
    },

    /**
     * 检查URL是否与当前页面同源
     * @param {string} url - 要检查的URL
     * @returns {boolean} - 是否同源
     */
    isSameOrigin(url) {
      try {
        const currentOrigin = window.location.origin;
        const urlObj = new URL(url, currentOrigin);
        return urlObj.origin === currentOrigin;
      } catch (e) {
        return false;
      }
    },

    /**
     * 格式化OpenAI API URL，如果只提供了前缀，自动补全'/v1/chat/completions'
     * @param {string} url - 原始URL
     * @returns {string} - 格式化后的URL
     */
    formatOpenAIUrl(url) {
      if (!url) {
        return "https://api.openai.com/v1/chat/completions";
      }

      if (!url.endsWith("/v1/chat/completions")) {
        // 移除末尾的斜杠（如果有）
        url = url.replace(/\/+$/, "");
        // 添加标准路径
        url = `${url}/v1/chat/completions`;
      }

      return url;
    },

    /**
     * 使用OpenAI API识别验证码
     */
    async recognizeWithOpenAI(base64Image) {
      // 使用自定义API地址或默认地址，并格式化URL
      const apiUrl = this.formatOpenAIUrl(this.settings.openaiApiUrl);

      // 使用自定义模型或默认模型
      const model = this.settings.openaiModel || "gpt-4.1-mini";
      // 使用自定义提示词或默认提示词
      const prompt = this.settings.openaiPrompt || DEFAULT_PROMPT;

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

      // 提取结果并移除所有非字母数字字符（包括空格）
      const content = response.data.choices[0].message.content.trim();
      return content.replace(/[^a-zA-Z0-9]/g, "");
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
      const prompt = this.settings.geminiPrompt || DEFAULT_PROMPT;

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
          return text.replace(/[^a-zA-Z0-9]/g, "");
        }
      }

      return "";
    },

    /**
     * 使用通义千问 API 识别验证码（新版API格式，messages/content结构）
     */
    async recognizeWithQwen(base64Image) {
      const apiUrl =
        this.settings.qwenApiUrl ||
        "https://dashscope.aliyuncs.com/compatible-mode/v1/chat/completions";
      const model = this.settings.qwenModel || "qwen-vl-max-2025-04-02";
      const prompt = this.settings.qwenPrompt || DEFAULT_PROMPT;

      const response = await this.request({
        method: "POST",
        url: apiUrl,
        data: {
          model: model,
          messages: [
            {
              role: "user",
              content: [
                { type: "text", text: prompt },
                {
                  type: "image_url",
                  image_url: { url: `data:image/png;base64,${base64Image}` },
                },
              ],
            },
          ],
          temperature: 0.1,
          top_p: 1,
          stream: false,
        },
        headers: {
          "Content-Type": "application/json",
          Authorization: `Bearer ${this.settings.qwenKey}`,
        },
      });

      // 提取结果
      if (response.data && response.data.choices && response.data.choices.length > 0) {
        const text = response.data.choices[0].message.content;
        // 只保留数字和字母
        return text.replace(/[^a-zA-Z0-9]/g, "");
      }
      return "";
    },

    /**
     * 获取合并后的选择器字符串
     * @param {Array} selectors - 选择器数组
     * @returns {string} - 合并后的选择器字符串
     */
    getCombinedSelector(selectors) {
      // 合并默认选择器和自定义选择器
      let allSelectors = [...selectors];

      // 如果是验证码选择器，添加自定义验证码选择器
      if (
        selectors === this.config.captchaSelectors &&
        this.settings.customCaptchaSelectors
      ) {
        allSelectors = [
          ...allSelectors,
          ...this.settings.customCaptchaSelectors.filter((s) => s.trim()),
        ];
      }

      // 如果是输入框选择器，添加自定义输入框选择器
      if (
        selectors === this.config.inputSelectors &&
        this.settings.customInputSelectors
      ) {
        allSelectors = [
          ...allSelectors,
          ...this.settings.customInputSelectors.filter((s) => s.trim()),
        ];
      }

      return allSelectors.join(", ");
    },

    /**
     * 查找页面上的验证码图片和相关输入框
     */
    findCaptchaElements() {
      const captchaSelector = this.getCombinedSelector(this.config.captchaSelectors);
      const captchaImages = document.querySelectorAll(captchaSelector);
      if (captchaImages.length === 0) {
        // console.log('未找到验证码图片');
        return [];
      }

      const elements = [];
      captchaImages.forEach((img) => {
        // 查找最近的输入框
        let inputField = null;

        // 方法 1：尝试通过选择器查找相关输入框
        const inputSelector = this.getCombinedSelector(this.config.inputSelectors);
        const inputs = document.querySelectorAll(inputSelector);
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
        if (existingIcon && existingIcon.classList.contains("captcha-recognition-icon")) {
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
     * @param {HTMLImageElement} captchaImg - 验证码图片元素
     * @param {HTMLInputElement} inputField - 输入框元素
     * @param {HTMLElement} icon - 识别图标元素
     * @param {Object} checkedBase64 - 可选，已经预先检查过的 base64 结果
     */
    async processCaptcha(captchaImg, inputField, icon, checkedBase64) {
      try {
        // 更新图标状态为加载中
        icon.classList.add("captcha-recognition-loading");

        // 获取 base64 编码的图片
        let base64Result;
        if (checkedBase64) {
          // 如果已经有检查过的结果，直接使用
          base64Result = checkedBase64;
        } else {
          // 否则转换图片为 base64
          base64Result = this.imageToBase64(captchaImg);

          // 如果图片转换失败，显示错误并终止识别
          if (!base64Result.success) {
            this.showToast(base64Result.message, "error");
            icon.classList.remove("captcha-recognition-loading");
            icon.classList.add("captcha-recognition-error");
            setTimeout(() => {
              icon.classList.remove("captcha-recognition-error");
            }, 2000);
            return;
          }
        }

        // 识别验证码
        const text = await this.recognizeCaptcha(base64Result.data);

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
            this.showToast(`已将验证码复制到剪贴板`, "success");
          } catch (clipboardError) {
            // 如果剪贴板 API 不可用，使用传统方法
            const textarea = document.createElement("textarea");
            textarea.value = text;
            textarea.style.position = "fixed";
            textarea.style.opacity = "0";
            document.body.appendChild(textarea);
            textarea.select();
            document.execCommand("copy");
            document.body.removeChild(textarea);
            this.showToast(`验证码已识别：${text} (已复制到剪贴板)`, "success");
          }
        } else {
          this.showToast(`验证码已识别：${text}`, "success");
        }

        // 更新图标状态为成功
        icon.classList.remove("captcha-recognition-loading");
        icon.classList.add("captcha-recognition-success");
        setTimeout(() => {
          icon.classList.remove("captcha-recognition-success");
        }, 2000);
      } catch (error) {
        console.error("验证码识别失败：", error);
        icon.classList.remove("captcha-recognition-loading");
        icon.classList.add("captcha-recognition-error");
        setTimeout(() => {
          icon.classList.remove("captcha-recognition-error");
        }, 2000);

        this.showToast("处理验证码失败：" + (error.message || "未知错误"), "error");
      }
    },

    /**
     * 监听 DOM 变化，自动为新添加的验证码添加识别图标
     */
    setupMutationObserver() {
      const observer = new MutationObserver((mutations) => {
        let hasNewCaptcha = false;
        let newCaptchaElements = [];
        const captchaSelector = this.getCombinedSelector(this.config.captchaSelectors);

        mutations.forEach((mutation) => {
          // 检查新增节点
          if (mutation.type === "childList" && mutation.addedNodes.length) {
            mutation.addedNodes.forEach((node) => {
              if (node.nodeType === Node.ELEMENT_NODE) {
                const captchas = node.querySelectorAll(captchaSelector);
                if (captchas.length > 0) {
                  hasNewCaptcha = true;
                  captchas.forEach((captcha) => {
                    newCaptchaElements.push(captcha);
                  });
                }
              }
            });
          }

          // 检查属性变化 - 特别是 src 属性变化的验证码图片
          if (
            mutation.type === "attributes" &&
            mutation.attributeName === "src" &&
            mutation.target.matches &&
            mutation.target.matches(captchaSelector)
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

              // 过滤出新检测到的元素
              const newElements = elements.filter(({ captchaImg }) =>
                newCaptchaElements.includes(captchaImg)
              );

              // 预先检测是否有不可识别的图片
              const unrecognizableImages = [];
              newElements.forEach(({ captchaImg }) => {
                // 预先检测图片是否可以转换为 base64
                const base64Result = this.imageToBase64(captchaImg);
                if (!base64Result.success) {
                  unrecognizableImages.push({
                    img: captchaImg,
                    message: base64Result.message,
                  });
                }
              });

              // 如果有不可识别的图片，显示提示
              if (unrecognizableImages.length > 0) {
                this.showToast(
                  `检测到 ${unrecognizableImages.length} 个新验证码图片无法识别：${unrecognizableImages[0].message}`,
                  "error"
                );
              }

              // 只识别可识别的图片
              const recognizableElements = newElements.filter(({ captchaImg }) => {
                const base64Result = this.imageToBase64(captchaImg);
                return base64Result.success;
              });

              // 处理可识别的图片
              if (recognizableElements.length > 0) {
                recognizableElements.forEach(({ captchaImg, inputField }) => {
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
                      captchaImg.parentNode.insertBefore(icon, captchaImg.nextSibling);
                    } else {
                      captchaImg.parentNode.appendChild(icon);
                    }
                  }

                  // 获取 base64 结果，避免重复转换
                  const base64Result = this.imageToBase64(captchaImg);
                  // 自动进行识别，传入已检查的 base64 结果
                  this.processCaptcha(captchaImg, inputField, icon, base64Result);
                });
              } else if (newElements.length > 0) {
                this.showToast(
                  `检测到 ${newElements.length} 个新验证码，但均无法自动识别`,
                  "error"
                );
              }
            }, 500); // 延迟 500ms，确保图片已经加载完成
          }
        }
      });

      // 监听整个文档的变化，包括子树和属性变化
      observer.observe(document.body, {
        childList: true,
        subtree: true,
        attributes: true,
        attributeFilter: ["src"], // 只监听 src 属性变化
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
        this.addIconsToCaptchas(); // 步骤 1: 确保所有图标都已创建
        this.setupMutationObserver();
        const elements = this.findCaptchaElements();

        if (elements.length > 0) {
          // 新增逻辑：预先检测验证码图片是否可以识别
          const unrecognizableImages = [];
          elements.forEach(({ captchaImg }) => {
            // 预先检测图片是否可以转换为 base64
            const base64Result = this.imageToBase64(captchaImg);
            if (!base64Result.success) {
              unrecognizableImages.push({
                img: captchaImg,
                message: base64Result.message,
              });
            }
          });

          // 如果有不可识别的图片，显示提示
          if (unrecognizableImages.length > 0) {
            this.showToast(
              `检测到 ${unrecognizableImages.length} 个验证码图片无法识别：${unrecognizableImages[0].message}`,
              "error"
            );
            return false;
          }

          // 检查初始加载时的自动识别设置
          if (this.settings.autoRecognize) {
            // 只识别可识别的图片
            const recognizableElements = elements.filter(({ captchaImg }) => {
              const base64Result = this.imageToBase64(captchaImg);
              return base64Result.success;
            });

            if (recognizableElements.length > 0) {
              this.showToast(
                `检测到 ${recognizableElements.length} 个可识别的验证码，正在自动识别...`,
                "info"
              );
              // 遍历所有找到的可识别验证码并处理
              recognizableElements.forEach(({ captchaImg, inputField }) => {
                const icon = captchaImg.nextElementSibling;
                // 确保图标元素存在
                if (icon && icon.classList.contains("captcha-recognition-icon")) {
                  // 获取 base64 结果，避免重复转换
                  const base64Result = this.imageToBase64(captchaImg);
                  // 直接调用处理函数，传入已检查的 base64 结果
                  this.processCaptcha(captchaImg, inputField, icon, base64Result);
                }
              });
            } else if (elements.length > 0) {
              this.showToast(
                `检测到 ${elements.length} 个验证码，但均无法自动识别`,
                "error"
              );
            }
          } else {
            // 保留原始行为：如果未开启自动识别，则提示用户点击
            this.showToast(
              `检测到 ${elements.length} 个验证码，点击识别图标开始识别`,
              "info"
            );
          }
        }
      }, 1e3);
    },

    /**
     * 通用请求函数，自动根据环境使用 GM_xmlhttpRequest 或 axios
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

        // 使用 axios
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
     * 显示 Toast 提示
     * @param {string} message - 提示信息
     * @param {string} type - 提示类型 (success, error, info)
     */
    showToast(message, type = "info") {
      // 如果用户禁用了通知，则不显示
      if (this.settings.showNotification === false) {
        return;
      }

      // 创建 toast 容器（如果不存在）
      let toastContainer = document.getElementById("captcha-toast-container");
      if (!toastContainer) {
        toastContainer = document.createElement("div");
        toastContainer.id = "captcha-toast-container";
        document.body.appendChild(toastContainer);
      }

      // 创建新的 toast 元素
      const toast = document.createElement("div");
      toast.className = `captcha-toast captcha-toast-${type}`;
      toast.textContent = message;

      // 将新 toast 插入到容器的最前面
      if (toastContainer.firstChild) {
        toastContainer.insertBefore(toast, toastContainer.firstChild);
      } else {
        toastContainer.appendChild(toast);
      }

      // 添加显示动画
      setTimeout(() => {
        toast.classList.add("captcha-toast-show");
      }, 10);

      // 3 秒后移除
      setTimeout(() => {
        toast.classList.remove("captcha-toast-show");
        toast.classList.add("captcha-toast-hide");

        // 动画结束后移除元素
        setTimeout(() => {
          if (toast.parentNode) {
            toast.parentNode.removeChild(toast);
          }
        }, 300); // 300ms 是动画持续时间
      }, 3000);
    },

    /**
     * 测试 API 连通性
     * @param {string} apiType - API 类型，'openai'或'gemini'
     */
    async testApiConnection(apiType) {
      try {
        if (!this.isApiConfigured(apiType)) {
          this.apiTestStatus[apiType] = "error";
          // 3 秒后重置状态
          setTimeout(() => {
            this.apiTestStatus[apiType] = "";
          }, 3000);
          return;
        }

        // 设置加载状态
        this.apiTestStatus[apiType] = "loading";

        // 测试图片base64
        const testBase64Image =
          "iVBORw0KGgoAAAANSUhEUgAAAGQAAAAmCAYAAAAycj4zAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsMAAA7DAcdvqGQAAAPNSURBVGhD7ZgxbuswDIZ9r1zAx8ghfICX+Q0d28EIeoACRYcOAQpkKdDuWQp0yQGKXkEvSuSYIimKsqU4Md4HCEFkiqL4W5SSarfbGdtunbmsQy1IiQVrfGrn1dpdiqHxVO4zSokFa3yGbDRjp2RofDdbsm417hizOUPmgrpk5WLKF+AWXj4iSOmgU/3njEfrK+ecqYwWhLNP9SGR6ivH3CEfOXzHEAXRBMDZ+H3fpqn+mgq35ts9p2yfK1P96Vvz5R4QtqYBdtV9azYoHi4+Ddw4+329BvPB9rx1VjzaOJIF4fp4fk1bM0Kg1nDr+GlNrVnsV+MlpX7fuwc9Urwpz/bvtTdXqNkYOL+4j7OxJB/qIUeY9ZIXgGtUlL1p7+FCm8NeoPg7qTbtj3ugRFoLfKYVw7aQIJjOBtsmC6Ji++onnZQnVMbqz4MEPjgJtGzRckX3RxwuKZZzH96tx9aYtTemj4XbpRJ4bvF3SOiZNMaybfpk1+2v68X4JY3skljZipUrNpGwnXZUt5bQesjucMJzY6xtqiCYUYLA1gPf/le21JwBO4kKJ5etaLlSCLL64NfXg2OQLhhp0LydUJWsbjB2wPbvP03dCSLcpI5EbMNla0i5QsllbmQUNA96KcjaEwiNHSUIC0zy8k0eExMvVLZQudK8tb642guAXhB1fiLkP9Rhkhcvx7cwiGI3+aXplBC/b+kliWWAgCdkQSDXK4h0hhzPjL5v3z46O+HwJ8lESVqvnWEIZO92mSaBu93GrO7AWGZn6fzoxRIFSXEECd2yvP66F6OqHk2LDoF+bvyW+i32tnM7zKJZm32+eVqA8Yd2t/J2PfQTumVp5uoYJUjwufA7BIpybszvEOjbTypscrnCl4LlRpeUjmMMHyt6WzuI4sfbvzRFr70xpLFs4tn2QHYHAZWtc5PKFboQLJ42yevs1oeFldqkgsjo/ss6NVqyfPiyJZUrTRK1ybP5IaUr0EYL4j4LEvi3d/liVgvYJ/+IpGUrfOOx5Bake2mD5TPyb68WIki5HRPAXX3Df7HMH5jzYoLk8pOTHDGVWBf0WaxkpQaeaj8EzRwxG42PMVzgDNFReqFaLhVHaJ7sglxqQddOLA/dc2zzX5BCaPLA2VxNybo1NAkfwqwFKZU0C+dbM1/MZjJBNMGP5RJzQDTzxWxEQTQTDKWkby05Y0j1FbIngkDD0KC5kHN9qb44e/tdFCQH0F9u3ykMmTtXvFo/1qb4GQKD0QbGMWasJWV8Z5syRiLFz2SHeiq5kqPhknNhJhFkygVfN8b8A/Cu2G5QVhydAAAAAElFTkSuQmCC";

        if (apiType === "openai") {
          // 测试 OpenAI API
          const apiUrl = this.formatOpenAIUrl(this.settings.openaiApiUrl);
          const model = this.settings.openaiModel || "gpt-4.1-mini";

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
                      text: "这是一个验证码图片，请识别其中的字符",
                    },
                    {
                      type: "image_url",
                      image_url: {
                        url: `data:image/png;base64,${testBase64Image}`,
                      },
                    },
                  ],
                },
              ],
            },
            headers: {
              "Content-Type": "application/json",
              Authorization: `Bearer ${this.settings.openaiKey}`,
            },
          });

          if (response && response.data) {
            this.apiTestStatus[apiType] = "success";
          }
        } else if (apiType === "gemini") {
          // 测试 Google Gemini API
          const model = this.settings.geminiModel || "gemini-2.5-flash-lite";
          const baseApiUrl =
            this.settings.geminiApiUrl ||
            "https://generativelanguage.googleapis.com/v1beta/models";
          const apiUrl = `${baseApiUrl}/${model}:generateContent`;

          const response = await this.request({
            method: "POST",
            url: `${apiUrl}?key=${this.settings.geminiKey}`,
            data: {
              contents: [
                {
                  parts: [
                    {
                      text: "这是一个验证码图片，请识别其中的字符",
                    },
                    {
                      inline_data: {
                        mime_type: "image/png",
                        data: testBase64Image,
                      },
                    },
                  ],
                },
              ],
              generationConfig: {
                temperature: 0,
              },
            },
            headers: {
              "Content-Type": "application/json",
            },
          });

          if (response && response.data) {
            this.apiTestStatus[apiType] = "success";
          }
        } else if (apiType === "qwen") {
          // 测试通义千问 API（新版API格式）
          const apiUrl =
            this.settings.qwenApiUrl ||
            "https://dashscope.aliyuncs.com/compatible-mode/v1/chat/completions";
          const model = this.settings.qwenModel || "qwen-vl-max-2025-04-02";

          const response = await this.request({
            method: "POST",
            url: apiUrl,
            data: {
              model: model,
              messages: [
                {
                  role: "user",
                  content: [
                    { type: "text", text: "这是一个验证码图片，请识别其中的字符" },
                    {
                      type: "image_url",
                      image_url: { url: `data:image/png;base64,${testBase64Image}` },
                    },
                  ],
                },
              ],
              temperature: 0.1,
              top_p: 1,
              stream: false,
            },
            headers: {
              "Content-Type": "application/json",
              Authorization: `Bearer ${this.settings.qwenKey}`,
            },
          });

          if (response && response.data) {
            this.apiTestStatus[apiType] = "success";
          }
        }

        // 3 秒后重置成功状态
        setTimeout(() => {
          if (this.apiTestStatus[apiType] === "success") {
            this.apiTestStatus[apiType] = "";
          }
        }, 3000);
      } catch (error) {
        console.error("API 连接测试失败：", error);
        this.apiTestStatus[apiType] = "error";

        // 3 秒后重置错误状态
        setTimeout(() => {
          this.apiTestStatus[apiType] = "";
        }, 3000);
      }
    },

    /**
     * 切换高级设置的显示状态
     */
    toggleAdvancedSettings() {
      this.showAdvancedSettings = !this.showAdvancedSettings;
    },

    /**
     * 添加自定义选择器
     * @param {string} type - 选择器类型，'captcha'或'input'
     */
    addSelector(type) {
      if (type === "captcha") {
        this.settings.customCaptchaSelectors.push("");
      } else if (type === "input") {
        this.settings.customInputSelectors.push("");
      }
    },

    /**
     * 删除自定义选择器
     * @param {string} type - 选择器类型，'captcha'或'input'
     * @param {number} index - 要删除的选择器索引
     */
    removeSelector(type, index) {
      if (type === "captcha") {
        this.settings.customCaptchaSelectors.splice(index, 1);
      } else if (type === "input") {
        this.settings.customInputSelectors.splice(index, 1);
      }
    },
  },
  mounted() {
    this.init();
  },
  created() {
    // Website Compatibility Handling
    if (window.location.host == "nportal.ntut.edu.tw") {
      const observer = new MutationObserver(function (mutations) {
        const authcodeElement = document.querySelector(".authcode.co");
        if (authcodeElement) {
          const captchaIcon = document.querySelector(".captcha-recognition-icon");
          if (captchaIcon) {
            captchaIcon.parentNode.removeChild(captchaIcon);
            authcodeElement.appendChild(captchaIcon);
            observer.disconnect();
          }
        }
      });
      observer.observe(document.body, { childList: true, subtree: true });
    }
  },
};
</script>
