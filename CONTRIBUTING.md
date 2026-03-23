# Contributing

感谢你对学习帮感兴趣。

## 本地开发

1. 创建虚拟环境并安装依赖
2. 按 `README.md` 配置 `.env`
3. 启动 MySQL 或直接运行测试

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements-dev.txt
pytest -q
```

## 提交建议

- 一个 PR 聚焦一个主题
- 变更接口时同步更新 README 和 `docs/ARCHITECTURE.md`
- 涉及语音、摘要、导出逻辑时补测试

## 代码风格

- Python 保持清晰函数拆分
- 前端交互优先可用、可读
- 不引入与当前目标无关的复杂依赖
