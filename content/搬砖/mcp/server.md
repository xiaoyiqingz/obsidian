1. vscode agent mode 目前只http 方式调用，远端server 提供接口 /v1/openapi.json  描述 该mcp server 支持的 tools 接口， 待验证

2. claude desktop 
配置 ~/Library/Application\ Support/Claude/claude_desktop_config.json
```
{
    "mcpServers": {
        "weather": {
            "command": "go",
            "args": [
                "run",
                "/Users/zhangzhe/Documents/test/gomod/mcp-server-test/main.go"
            ]
        }
    }
}
```
错误日志在 `~/Library/Logs/Claude/mcp.log` 查看
