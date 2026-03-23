# StudyBang

StudyBang（学习帮）是一个面向大学课堂场景的实时转写助手：不落盘音频，只保留实时文本记录，自动标记老师提问与关键知识点，并在课后生成可搜索、可导出的结构化笔记。

![学习帮封面](docs/assets/cover.svg)

## 这版已经能做什么

- 浏览器直接使用麦克风做实时语音识别
- 自动创建课程会话，并把课堂片段实时写入 MySQL
- 支持两种转写方案
  - 浏览器实时识别
  - OpenAI Realtime / 服务端专业转写
- 标记老师提问、重点语句、关键词
- 识别到老师提问后自动生成候选答案与解释
- 结课后生成课后总结
  - 可选 AI 摘要
  - AI 不可用时自动回退到规则摘要
- 可在项目内直接切换“摘要 / 答题助手”模型
- 搜索历史课程
- 导出 Markdown / PDF
- 前端已升级为 React + Vite + Ant Design 仪表盘界面

## 技术栈

- 后端：FastAPI
- 数据库：MySQL 8
- ORM：SQLAlchemy 2
- 前端：React + Vite + Ant Design
- 实时转写：浏览器 `SpeechRecognition` / `webkitSpeechRecognition`
- 低延迟转写：OpenAI Realtime WebRTC
- 服务端转写：OpenAI Speech-to-Text（fallback 分段近实时）
- AI 摘要：OpenAI Responses API（可选）

## 工作流

![学习帮工作流](docs/assets/workflow.svg)

## 目录结构

```text
app/
  config.py
  database.py
  exporters.py
  main.py
  models.py
  schemas.py
  services.py
  summarizer.py
  static/
    index.html
    assets/
frontend/
  src/
  package.json
  vite.config.js
docs/
  ARCHITECTURE.md
  ROADMAP.md
tests/
  test_app.py
main.py
requirements.txt
requirements-dev.txt
docker-compose.yml
```

## 快速启动

### 1. 创建虚拟环境并安装后端依赖

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

### 2. 安装前端依赖并构建界面

```powershell
cd frontend
npm install
npm run build
cd ..
```

生产构建会输出到 `D:\PyCharm\StudyBang\app\static`，FastAPI 会直接托管这套页面。

### 3. 准备环境变量

复制 `D:\PyCharm\StudyBang\.env.example` 为 `.env`，或直接设置环境变量。

最小配置如下：

```env
DATABASE_URL=mysql+pymysql://root:123456@localhost:3306/xuexibang?charset=utf8mb4
SUMMARY_MODE=auto
OPENAI_API_KEY=
OPENAI_MODEL=gpt-5-mini
TRANSCRIPTION_MODE=browser
OPENAI_TRANSCRIPTION_MODEL=gpt-4o-mini-transcribe
TRANSCRIPTION_CHUNK_MS=4000
```

### 4. 启动 MySQL

如果本机已经有 MySQL，直接确认库和账号可用即可；如果本机没有安装，也可以使用 Docker：

```powershell
docker compose up -d mysql
```

### 5. 启动项目

```powershell
python main.py
```

浏览器访问 [http://127.0.0.1:8000](http://127.0.0.1:8000)

如果你刚改过前端样式，记得先重新执行一次 `cd frontend && npm run build`。

启动后可以在页面右上角打开“模型中心”，直接配置摘要 / 答题助手模型，不需要手改 `.env`。

## 前端开发

开发前端时可以单独启动 Vite：

```powershell
cd frontend
npm run dev
```

如果只是想让 FastAPI 页面生效，最终还是要执行一次 `npm run build`。

## AI 摘要模式

可以通过 `SUMMARY_MODE` 控制摘要行为：

- `rules`：只使用规则总结
- `auto`：有 `OPENAI_API_KEY` 时使用 AI，否则回退规则
- `ai`：优先用 AI，总结失败也会自动回退规则

如果你暂时不想接任何 AI 服务，直接把 `SUMMARY_MODE=rules` 就能稳定运行。

## AI 模型中心

- 页面里可以直接切换“摘要 / 答题助手”所使用的文本模型
- 当前内置预设：
  - `OpenAI 官方`
  - `小米 MiMo V2`
  - `自定义兼容接口`
- 对于 `小米 MiMo V2` 和其他兼容平台，通常还需要补上平台提供的 `Base URL`
- 文本接口支持在 `Responses API` 和 `Chat Completions` 之间切换
- 这一块只影响摘要和答题助手，不影响下方的语音转写链路

## 转写方案

- `TRANSCRIPTION_MODE=browser`
  - 使用浏览器内置 Web Speech API
  - 部署简单，但浏览器差异较大
- `TRANSCRIPTION_MODE=openai`
  - 优先使用 OpenAI Realtime WebRTC 做更低延迟转写
  - Realtime 不可用时回退到分段上传 + Speech-to-Text
  - 需要 `OPENAI_API_KEY`

页面里也可以在开始课程前手动切换转写方案。

## 问题自动解答

- 当课堂片段被识别为老师提问时，前端会自动请求后端生成候选答案
- 系统会优先结合最近几段课堂上下文生成：
  - 直接答案
  - 更易懂的解释
  - 相关知识点
- 如果 AI 不可用，系统会自动回退到基于课堂上下文的保守解答

## 导出与搜索

- 历史课程支持按课程名、老师名、课堂关键词搜索
- 已结束课程支持导出：
  - Markdown
  - PDF
- 示例导出可参考 `D:\PyCharm\StudyBang\examples\sample_note.md`

## 开发测试

安装开发依赖：

```powershell
pip install -r requirements-dev.txt
```

运行测试：

```powershell
pytest -q
```

测试默认使用临时 SQLite，不依赖 MySQL。

## 浏览器兼容

实时语音识别依赖浏览器内置 Web Speech API，推荐：

- Chrome
- Edge

如果浏览器提示不支持实时识别，请优先使用 Chromium 内核浏览器验证 MVP。

## 隐私说明

- 当前版本默认不保存原始音频文件
- 项目定位是“个人课堂学习整理助手”
- 实际使用时请遵守老师、学校以及当地对课堂采集的要求
