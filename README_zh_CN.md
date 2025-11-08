
思源笔记 AI 助手插件，实现基于思源笔记内容进行问答和编辑等丰富功能。

**注意**：使用本插件需要自备 AI 平台的 API 密钥，插件本身不提供 AI 服务。请遵守各平台的使用条款和隐私政策。

> 如果有需要使用GPT5、Gemini 2.5 Pro、Claude 4.5等顶级模型的朋友
> - 推荐[V3 API网站](https://api.gpt.ge/register?aff=fQIZ)，非常好用，按量付费，各个AI模型都能使用，比用官方的API能省不少钱，使用我的[邀请链接](https://api.gpt.ge/register?aff=fQIZ)注册可以获得$0.3余额奖励试用。

## 📝更新日志

见[CHANGELOG.md](https://cdn.jsdelivr.net/gh/Achuan-2/siyuan-plugin-copilot@main/CHANGELOG.md)


## ✨主要功能

- 多平台AI支持：
  - 插件内置常见平台（OpenAI、Google Gemini、DeepSeek、火山引擎）
  - 也支持添加任意兼容 OpenAI API 的平台，灵活切换聊天模型
- 模型设置
  - 支持独立配置每个模型的参数（温度、最大 tokens）
  - 标识模型特殊能力（思考模式、视觉支持）
- 三种聊天模式切换：ask、edit、agent三种聊天模式切换
  - ask 模式：日常问答，支持选择多个模型同时回复，选择满意回答
  - edit 模式：编辑修改笔记，支持编辑查看差异、撤回
  - agent 模式：提供工具，让AI实现自助查询笔记内容、编辑笔记、创建文档等功能
- 会话管理
  - 支持保存对话历史，支持对历史记录进行置顶和删除
  - 支持复制对话为Markdown
  - 支持保存对话为文档
-  多模态支持
  - 思源笔记内容：可通过拖拽块、拖拽页签、拖拽文档树的文档实现笔记内容到上传
  - 图片上传：支持粘贴、上传图片，还支持拖动图片块直接上传
  - 文件上传（支持 Markdown、文本文件等）
- 提示词管理
  - 支持创建和保存常用提示词，快速插入到输入框

## 🔧 开发相关

### 本地开发

```bash
pnpm install
pnpm run dev
```

### 文件
- `src\tools\index.ts`: agent 模式调用的工具实现代码

## 📄 许可证

GPL3 License

## 🙏 致谢

- 基于 [plugin-sample-vite-svelte](https://github.com/siyuan-note/plugin-sample-vite-svelte/) 模板开发
- 参考 [sy-f-misc](https://github.com/frostime/sy-f-misc)的GPT对话功能实现


## 📮 反馈与建议

如有问题或建议，欢迎在 [GitHub Issues](https://github.com/Achuan-2/siyuan-plugin-ai-sidebar/issues) 中提出。

## ❤️用爱发电

如果喜欢我的插件，欢迎给GitHub仓库点star和微信赞赏，这会激励我继续完善此插件和开发新插件。

维护插件费时费力，个人时间和精力有限，开源只是分享，不等于我要浪费我的时间免费帮用户实现ta需要的功能，

我需要的功能我会慢慢改进（打赏可以催更），有些我觉得可以改进、但是现阶段不必要的功能需要打赏才改进（会标注打赏标签和需要打赏金额），而不需要的功能、实现很麻烦的功能会直接关闭issue不考虑实现

累积赞赏50元的朋友如果想加我微信，可以发邮件到<span data-type="a" data-href="mailto:achuan-2@outlook.com">achuan-2@outlook.com</span>来进行好友申请（赞赏达不到50元的，我不会回复邮件和加好友哦，因为不想当免费客服）

![image](https://camo.githubusercontent.com/8052f6f2e7dafba534e781934efa9bcb084fa3a9dfa5c221a85ac63db8b043cb/68747470733a2f2f6173736574732e62336c6f6766696c652e636f6d2f73697975616e2f313631303230353735393030352f6173736574732f6e6574776f726b2d61737365742d696d6167652d32303235303631343132333535382d667568697235762e706e67)
