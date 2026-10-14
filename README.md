# 金蝶云星空 ERP MCP 查询技能

本仓库提供连接金蝶云星空（Kingdee Cloud Star）ERP 的 MCP 使用技能，供接入金蝶 MCP Server 的 Agent 自动加载，快速掌握查询工具用法、字段规则与常见坑。

## 目录结构

```
.
├── README.md                     # 本说明
└── skills/
    └── kingdee-erp/              # 技能目录（WorkBuddy 从此目录安装）
        ├── SKILL.md              # 技能主文件
        └── references/
            ├── api_reference.md  # 金蝶 WebAPI 接口参考
            └── form_fields.md    # 常用字段定义速查
```

## 安装方式

在 WorkBuddy 中，让 Agent 执行：

```
帮我安装这个 skill 里 skills/kingdee-erp 目录下的技能：https://github.com/wo-am-i/kingdee-erp-mcp-skill
```

或手动将 `skills/kingdee-erp` 整个目录复制到用户级技能目录 `~/.workbuddy/skills/` 下。

## 配套 MCP 配置

技能需要配合金蝶 MCP Server 使用，MCP 配置示例（`~/.workbuddy/mcp.json`）：

```json
{
  "mcpServers": {
    "金蝶云星空": {
      "url": "http://<服务器IP>:3000/sse",
      "type": "sse",
      "headers": {
        "x-api-key": "<API_KEY>"
      }
    }
  }
}
```

## 技能覆盖内容

- 18 个 MCP 工具用法（13 查询 + 5 写入）
- 科目余额表专用查询方式（GetSysReportData 端点）
- 单据查询与分页技巧（100 行限制的规避方法）
- 常用 formId 对照表（基础资料 / 供应链 / 生产 / 库存 / 报表）
- 字段书写规则与高频易错字段（FSalesManID vs FSalerID、FRealQty vs FQty 等）
- 故障排查速查

## 许可

供内部团队与合作伙伴使用。
