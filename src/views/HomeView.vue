<template>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>My Chat</title>
  <link
    href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600&display=swap"
    rel="stylesheet"
  />
  <div class="home">
    <h1 class="title">My Chat</h1>
  </div>
  <div class="chat-container">
    <div class="messages" id="messages"></div>
    <div id="loading" class="loading" style="display: none">加载中...</div>
    <div class="chat-send-container">
      <div class="input-container">
        <textarea
          id="chat-input"
          placeholder="输入消息..."
          @keyup="handleKeyPress"
          @input="textAreaInput"
        ></textarea>
      </div>
      <div class="input-container">
        <select id="chat-select">
          <option value="deepseek-chat" selected>DeepSeek-V3</option>
          <option value="deepseek-reasoner">DeepSeek-R1</option>
        </select>
        <button @click="sendMessage()" style="margin-left: auto">
          发送（Ctrl+Enter）
        </button>
      </div>
    </div>
  </div>
</template>

<script>
// @ is an alias to /src
import { ref } from "vue";

export default {
  name: "HomeView",
  setup() {
    return {
      formatMessage,
      displayMessage,
      sendMessage,
      handleKeyPress,
      sendFlag,
      textAreaInput,
    };
  },
};

// 是否正在回复标志
// 0-未在回复   1-在回复
const sendFlag = ref(0);

// 复制代码的函数
window.copyCode = (button) => {
  // 调试：打印 button 和其父元素、兄弟元素
  // console.log("Button:", button);
  // console.log("Parent Element:", button.parentElement);
  // console.log(
  //   "Next Sibling:",
  //   button.parentElement.nextElementSibling
  // );
  const codeContent = button.parentElement.nextElementSibling.innerText; // 获取代码内容
  // console.log("Code Content:", codeContent); // 调试：打印代码内容
  // fallbackCopyCode(codeContent, button);
  if (navigator.clipboard) {
    navigator.clipboard
      .writeText(codeContent)
      .then(() => {
        button.textContent = "已复制"; // 修改按钮文本
        setTimeout(() => {
          button.textContent = "复制"; // 恢复按钮文本
        }, 2000); // 2 秒后恢复
      })
      .catch((err) => {
        console.error("复制失败:", err);
        // 降级方案
        fallbackCopyCode(codeContent, button);
      });
  } else {
    // 降级方案
    fallbackCopyCode(codeContent, button);
  }
};

// 降级方案：使用 document.execCommand 复制代码
const fallbackCopyCode = (text, button) => {
  const textarea = document.createElement("textarea");
  textarea.value = text;
  document.body.appendChild(textarea);
  textarea.select();

  try {
    const success = document.execCommand("copy");
    if (success) {
      button.textContent = "已复制"; // 修改按钮文本
      setTimeout(() => {
        button.textContent = "复制"; // 恢复按钮文本
      }, 2000); // 2 秒后恢复
    } else {
      console.error("复制失败（document.execCommand）");
    }
  } catch (err) {
    console.error("复制失败（document.execCommand）:", err);
  } finally {
    document.body.removeChild(textarea); // 移除临时 textarea
  }
};

// 格式化消息文本
const formatMessage = (text) => {
  if (!text) return "";

  // 处理代码块（```标题\n代码内容```）
  const codeBlockRegex = /```([^\n]*)\n([\s\S]*?)```/g;
  text = text.replace(codeBlockRegex, (match, title, code) => {
    // 如果有标题，则将标题和代码内容分别处理
    const codeTitle = title.trim()
      ? `<div class="code-title"><span>${title.trim()}</span><button class="copy-button" onclick="copyCode(this)">复制</button></div>
        `
      : "";
    const codeContent = `<pre class="code-block"><code>
                         ${code.trim()}
                         </code></pre>`;
    return `${codeTitle}${codeContent}`;
  });

  // 处理标题和换行
  let lines = text.split("\n");
  let formattedLines = lines.map((line) => {
    // 处理标题（**文本**）
    line = line.replace(/\*\*(.*?)\*\*/g, '<span class="bold-text">$1</span>');
    return line;
  });

  // 将 ### 替换为换行，并确保每个部分都是一个段落
  let processedText = formattedLines.join("\n");
  let sections = processedText
    .split("###")
    .filter((section) => section.trim())
    .map((section) => {
      // 移除多余的换行和空格
      let lines = section.split("\n").filter((line) => line.trim());

      if (lines.length === 0) return "";

      // 处理每个部分
      let result = "";
      let currentIndex = 0;

      // 是否为代码内部内容标志
      let isCode = false;

      while (currentIndex < lines.length) {
        let line = lines[currentIndex].trim();
        // console.log(line);

        // 如果是数字开头（如 "1.")
        if (/^\d+\./.test(line)) {
          result += `<p class="section-title">${line}</p>`;
        }
        // 如果是小标题（以破折号开头）
        else if (line.startsWith("-")) {
          result += `<p class="subsection"><span class="bold-text">${line
            .replace(/^-/, "")
            .trim()}</span></p>`;
        }
        // 如果是正文（包含冒号的行）
        else if (line.includes(":")) {
          let [subtitle, content] = line.split(":").map((part) => part.trim());
          result += `<p><span class="subtitle">${subtitle}</span>: ${content}</p>`;
        }
        // 如果是代码块标题（通过正则表达式处理后，代码块已经被替换为 HTML 结构）
        else if (line.startsWith(`<div class="code-title">`)) {
          // 直接添加代码块的 HTML 结构
          result += line;
        }
        // 如果是代码块开始（通过正则表达式处理后，代码块已经被替换为 HTML 结构）
        else if (line.startsWith(`<pre class="code-block"><code>`)) {
          // 直接添加代码块的 HTML 结构
          result += line;
          isCode = true;
        }
        // 如果是代码块结束（通过正则表达式处理后，代码块已经被替换为 HTML 结构）
        else if (line.startsWith(`</code></pre>`)) {
          // 直接添加代码块的 HTML 结构
          result += line;
          isCode = false;
        }
        // 如果是代码块（通过正则表达式处理后，代码块已经被替换为 HTML 结构）
        else if (isCode) {
          // 直接添加代码块的 HTML 结构
          result += `<span style="display: block">${line}</span>`;
        }
        // 普通文本
        else {
          result += `<p>${line}</p>`;
        }
        currentIndex++;
      }
      return result;
    });

  return sections.join("");
};

// 显示消息
const displayMessage = (role, message) => {
  const messagesContainer = document.getElementById("messages");
  const messageElement = document.createElement("div");
  messageElement.className = `message ${role}`;

  const avatar = document.createElement("img");
  avatar.src =
    role === "user"
      ? require("../assets/user-avatar.png")
      : require("../assets/bot-avatar.png");
  avatar.alt = role === "user" ? "User" : "Bot";

  const messageContent = document.createElement("div");
  messageContent.className = "message-content";

  // 用户消息直接显示，机器人消息需要格式化
  messageContent.innerHTML = role === "user" ? message : formatMessage(message);

  messageElement.appendChild(avatar);
  messageElement.appendChild(messageContent);
  messagesContainer.appendChild(messageElement);

  // 平滑滚动到底部
  messageElement.scrollIntoView({ behavior: "smooth" });
};

const sendMessage = () => {
  if (sendFlag.value === 1) {
    alert("AI正在思考，请耐心等待哦！");
    return;
  }
  const inputElement = document.getElementById("chat-input");
  const message = inputElement.value;
  if (!message.trim()) return;

  displayMessage("user", message);
  inputElement.value = "";

  // 显示加载动画
  const loadingElement = document.getElementById("loading");
  if (loadingElement) {
    loadingElement.style.display = "block";
  }

  const selectElement = document.getElementById("chat-select");
  const model = selectElement.value;

  const apiKey = "sk-xxxxxxxx";
  const endpoint = "https://api.deepseek.com/chat/completions";

  const payload = {
    model: model,
    messages: [
      { role: "system", content: "You are a helpful assistant" },
      { role: "user", content: message },
    ],
    stream: false,
  };

  sendFlag.value = 1;
  fetch(endpoint, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${apiKey}`,
    },
    body: JSON.stringify(payload),
  })
    .then((response) => response.json())
    .then((data) => {
      sendFlag.value = 0;
      // 隐藏加载动画
      if (loadingElement) {
        loadingElement.style.display = "none";
      }

      if (data.choices && data.choices.length > 0) {
        displayMessage("bot", data.choices[0].message.content);
      } else {
        displayMessage("bot", "出错了，请稍后再试。");
      }
    })
    .catch((error) => {
      sendFlag.value = 0;
      // 隐藏加载动画
      if (loadingElement) {
        loadingElement.style.display = "none";
      }

      displayMessage("bot", "出错了，请稍后再试。");
      console.error("Error:", error);
    });
};

// 添加ctrl+回车发送功能
const handleKeyPress = (event) => {
  if (event.key === "Enter" && event.ctrlKey) {
    event.preventDefault();
    sendMessage();
  }
};

// 添加ctrl+回车发送功能
const textAreaInput = () => {
  const textarea = document.getElementById("chat-input");
  // 最小高度
  const minHeight = 50;
  // 最大高度
  const maxHeight = 200;
  // 上下 padding 的总和（10px + 10px）
  const padding = 20;
  // 重置高度为 auto，让浏览器重新计算 scrollHeight
  textarea.style.height = "auto";
  // 计算内容高度（scrollHeight - padding）
  const contentHeight = textarea.scrollHeight - padding;
  // 设置高度，确保不低于最小高度且不超过最大高度
  textarea.style.height = `${Math.max(
    minHeight,
    Math.min(contentHeight, maxHeight)
  )}px`;
};
</script>

<style>
.title {
  font-size: 40px;
  font-family: 华文新魏, serif;
  text-align: center;
}

body {
  font-family: "Inter", sans-serif;
  background-color: #e9ecef;
  margin: 0;
  padding: 0;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

.copy-button {
  background-color: #e0e0e0; /* 按钮背景色 */
  color: #007bff; /* 按钮文字颜色 */
  border: none;
  border-radius: 3px;
  padding: 4px 8px;
  font-size: 12px;
  cursor: pointer;
  float: right;
}

.copy-button:hover {
  background-color: #e0e0e0; /* 按钮悬停背景色 */
}

.code-title {
  background-color: #e0e0e0; /* 标题背景色 */
  padding: 8px 10px; /* 内边距 */
  border-radius: 5px 5px 0 0; /* 圆角（仅上方） */
  font-family: monospace; /* 等宽字体 */
  font-size: 14px; /* 字体大小 */
  font-weight: bold; /* 加粗 */
  border: 1px solid #ddd; /* 边框 */
  border-bottom: none; /* 去掉下方边框 */
}

.code-block {
  background-color: #f5f5f5; /* 代码块背景色 */
  padding: 10px; /* 内边距 */
  border-radius: 0 0 5px 5px; /* 圆角（仅下方） */
  border: 1px solid #ddd; /* 边框 */
  font-family: monospace; /* 等宽字体 */
  font-size: 14px; /* 字体大小 */
  overflow-x: auto; /* 水平滚动 */
  white-space: pre-wrap; /* 保留换行和空格 */
  margin: 0;
}

.chat-container {
  width: 100%;
  max-width: 600px;
  background-color: #fff;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  height: 80vh;
  backdrop-filter: blur(80px);
  margin: auto;
  margin-top: 10px;
}

/* 外层容器 */
.chat-send-container {
  max-width: 600px;
  margin: 0;
  margin-top: 0;
  border: 1px solid #e0e0e0; /* 外层边框 */
  border-radius: 32px; /* 圆角 */
  background-color: #f9f9f9; /* 背景色 */
  padding: 16px; /* 内边距 */
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1); /* 阴影 */
  overflow: hidden;
}

.messages {
  flex: 1;
  overflow-y: auto;
  border: 1px solid #ddd;
  margin-bottom: 20px;
  padding: 15px;
  border-radius: 5px;
  background-color: #f8f9fa;
  font-size: 16px;
}

.message {
  display: flex;
  align-items: flex-start;
  margin-bottom: 15px;
}

.message img {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  margin-right: 10px;
}

.message-content {
  padding: 12px 16px;
  border-radius: 12px;
  max-width: 85%;
  font-size: 16px;
  line-height: 1.6;
}

.message-content .title {
  font-size: 1.3em;
  font-weight: bold;
  display: block;
  margin: 16px 0 12px 0;
  color: #333;
}

.message.user {
  flex-direction: row-reverse;
}

.message.user img {
  margin-left: 10px;
  margin-right: 0;
}

.message.user .message-content {
  background-color: #007bff;
  backdrop-filter: blur(80px);
  color: white;
  text-align: right;
}

.message.bot .message-content {
  background-color: #f1f3f5;
}

.input-container {
  display: flex;
  gap: 8px;
  align-items: center;
}

.input-container textarea {
  background-color: #f9f9f9; /* 背景色 */
  flex: 1;
  min-width: 0;
  padding: 10px;
  border: 0 solid #ccc;
  border-radius: 5px;
  font-size: 16px;
  font-family: inherit;
  resize: none;
  height: 50px;
  min-height: 50px; /* 最小高度 */
  max-height: 200px; /* 最大高度 */
  overflow-y: auto; /* 内容超出时显示滚动条 */
  line-height: 1.5;
  transition: height 0.2s ease; /* 添加动画效果 */
  outline: none; /* 移除默认的焦点轮廓 */
}

.input-container input {
  flex: 1;
  min-width: 0;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
  font-size: 16px;
}

.input-container button {
  padding: 10px 15px;
  min-width: 60px;
  white-space: nowrap;
  background-color: #007bff;
  backdrop-filter: blur(80px);
  color: #fff;
  border: none;
  border-radius: 32px;
  cursor: pointer;
}

.input-container button:hover {
  background-color: #0056b3;
  backdrop-filter: blur(80px);
}

/* Media query for small screens */
@media (max-width: 600px) {
  .chat-container {
    height: 83vh;
    padding: 15px;
  }

  .messages {
    padding: 20px;
    font-size: 15px;
  }

  .input-container {
    gap: 8px;
  }

  .input-container input {
    font-size: 14px;
  }

  .input-container button {
    padding: 8px 12px;
    font-size: 14px;
  }

  .message img {
    width: 30px;
    height: 30px;
  }

  .message-content {
    font-size: 15px;
    max-width: 100%;
  }
}

/* 添加下拉菜单样式 */
.dropdown {
  position: relative;
  display: inline-block;
}

.dropdown-content {
  display: none;
  position: absolute;
  bottom: 100%;
  right: 0;
  background-color: #fff;
  min-width: 160px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  border-radius: 5px;
  z-index: 1;
  margin-bottom: 10px;
}

.dropdown-content.show {
  display: block;
}

.dropdown-item {
  padding: 12px 16px;
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
  transition: background-color 0.2s;
}

.dropdown-item:hover {
  background-color: #f8f9fa;
}

/* 深色模式样式 */
body.dark-mode {
  background-color: #1a1a1a;
}

.dark-mode .chat-container {
  background-color: #2d2d2d;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
  backdrop-filter: blur(80px);
}

.dark-mode .messages {
  background-color: #383838;
  border-color: #444;
  color: #fff;
  backdrop-filter: blur(80px);
}

.dark-mode .message.bot .message-content {
  background-color: #444;
  color: #fff;
}

.dark-mode .message.user .message-content {
  background-color: #0056b3;
  backdrop-filter: blur(80px);
}

.dark-mode .input-container input {
  background-color: #383838;
  border-color: #444;
  color: #fff;
}

.dark-mode .dropdown-content {
  background-color: #2d2d2d;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
}

.dark-mode .dropdown-item {
  color: #fff;
}

.dark-mode .dropdown-item:hover {
  background-color: #383838;
}

/* 段落样式 */
.message-content p {
  margin: 10px 0;
  line-height: 1.6;
}

.dark-mode .message-content p {
  color: #fff;
}

/* 调整标题和段落样式 */
.message-content .title {
  font-size: 1.3em;
  font-weight: bold;
  display: block;
  margin: 16px 0 12px 0;
  color: #333;
}

.message-content p {
  margin: 10px 0;
  line-height: 1.6;
}

.dark-mode .message-content .title {
  color: #fff;
}

/* 粗体文本样式 */
.message-content .bold-text {
  font-weight: bold;
  font-size: 1.1em;
  color: #333;
}

/* 段落标题样式 */
.message-content .section-title {
  font-size: 1.1em;
  font-weight: bold;
  margin: 16px 0 8px 0;
}

/* 子段落样式 */
.message-content .subsection {
  margin: 8px 0 8px 20px;
}

/* 正文样式 */
.message-content .subtitle {
  font-weight: bold;
}

/* 深色模式适配 */
.dark-mode .message-content .bold-text,
.dark-mode .message-content .section-title,
.dark-mode .message-content .subtitle,
.dark-mode .message-content p {
  color: #fff;
}

.loading {
  text-align: center;
  font-size: 16px;
  color: #666;
  margin-bottom: 20px;
}

.input-container select {
  flex: 1; /* 占据剩余空间 */
  min-width: 0; /* 防止内容溢出 */
  max-width: 30%;
  padding: 10px; /* 内边距 */
  border: 1px solid #ccc; /* 边框 */
  border-radius: 32px; /* 圆角 */
  font-size: 16px; /* 字体大小 */
  background-color: white; /* 背景颜色 */
  appearance: none; /* 移除默认箭头 */
  -webkit-appearance: none; /* 兼容 Safari */
  -moz-appearance: none; /* 兼容 Firefox */
  background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="%23333"><path d="M7 10l5 5 5-5z"/></svg>'); /* 自定义箭头 */
  background-repeat: no-repeat;
  background-position: right 10px center; /* 箭头位置 */
  background-size: 12px; /* 箭头大小 */
  cursor: pointer; /* 鼠标指针 */
}

/* 鼠标悬停效果 */
.input-container select:hover {
  border-color: #888; /* 边框颜色变深 */
}

/* 聚焦效果 */
.input-container select:focus {
  border-color: #1890ff; /* 聚焦时边框颜色 */
  outline: none; /* 移除默认聚焦轮廓 */
  box-shadow: 0 0 5px rgba(24, 144, 255, 0.5); /* 添加聚焦阴影 */
}
</style>
