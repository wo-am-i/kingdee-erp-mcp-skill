---
name: kingdee-erp
description: 金蝶云星空 ERP 查询技能。通过 MCP Server 查询金蝶云星空（Kingdee Cloud Star）中的客户、供应商、物料、库存、采购/销售订单、出入库单据、科目余额等数据。当用户提到金蝶、ERP、物料、库存、采购订单、销售订单、科目余额、出入库等词时使用。
---

# 金蝶云星空 ERP 查询技能

通过 MCP Server（`kingdee-erp`）连接金蝶云星空私有云/公有云系统，提供单据与报表的查询能力。

> 本技能面向「接入金蝶 MCP 的任意账套」通用，具体连接的账套、组织、权限由 MCP 服务端配置决定。

## 一、可用工具总览（18 个）

### 查询工具（13 个）

| 工具名 | 用途 | 关键参数 |
|--------|------|---------|
| `kingdee_ping` | 测试连接与认证 | 无 |
| `kingdee_list_forms` | 搜索业务表单 | keyword |
| `kingdee_get_fields` | 获取表单字段定义 | formId |
| `kingdee_query_bills` | 通用单据查询（支持分页） | formId, fieldKeys, filterString, startRow, limit |
| `kingdee_view_bill` | 查看单条单据详情 | formId, number/id |
| `kingdee_query_purchase_orders` | 查询采购订单 | filterString 等 |
| `kingdee_query_sale_orders` | 查询销售订单 | filterString 等 |
| `kingdee_query_stock_bills` | 查询出入库单据 | billType, filterString 等 |
| `kingdee_query_inventory` | 查询即时库存 | materialNumber, materialName, stockName |
| `kingdee_query_materials` | 查询物料档案 | materialNumber, materialName |
| `kingdee_query_partners` | 查询客户/供应商 | partnerType, number, name |
| `kingdee_query_account_balance` | 查询科目余额表（专用） | FSTARTYEAR, FENDPERIOD 等 |
| `kingdee_common_forms` | 常用 formId 对照表 | 无 |

### 写入工具（5 个，需完整读写权限）

| 工具名 | 用途 | 风险 |
|--------|------|------|
| `kingdee_save_bill` | 新建/修改单据 | 高 |
| `kingdee_submit_bills` | 提交单据 | 高 |
| `kingdee_audit_bills` | 审核单据 | 高 |
| `kingdee_unaudit_bills` | 反审核单据 | 高 |
| `kingdee_delete_bills` | 删除单据（仅未审核） | 高 |

> ⚠️ 只读模式（readonly）的 MCP 不会暴露这 5 个写入工具。

---

## 二、⭐ 科目余额表（最常用，接口特殊）

- 工具名：`kingdee_query_account_balance`（专用工具，**不要**用 `kingdee_query_bills` 查）
- 后端端点：`GetSysReportData`（**不是** ExecuteBillQuery）
- FormId：`GL_RPT_AccountBalance`

**最简调用：**
```json
{ "FSTARTYEAR": "2026", "FENDPERIOD": "7" }
```

**完整参数：**

| 参数 | 默认值 | 说明 |
|------|--------|------|
| FACCTBOOKID | {"FNumber":""} | 账簿 |
| FCURRENCY | "1" | 币别，"1"=人民币（**不是** "PRE001"） |
| FSTARTYEAR | "2026" | 起始年度 |
| FSTARTPERIOD | — | 起始期间 |
| FENDYEAR | "2026" | 截止年度 |
| FENDPERIOD | — | 截止期间 |
| FBALANCELEVEL | "1" | 1=一级科目，2=二级 |
| FSTARTBALANCE | {"FNumber":""} | 起始科目编码筛选 |
| FENDBALANCE | {"FNumber":""} | 截止科目编码筛选 |
| FBALANCEZERO | "false" | 是否含零余额科目 |

**返回值：** `Result.Rows` 数组，每行 6 列：[科目编码, 科目名称, 年初借方, 年初贷方, 期末借方, 期末贷方]

---

## 三、单据查询与分页

### 3.1 通用查询 `kingdee_query_bills`

**参数：**

| 参数 | 必填 | 说明 |
|------|------|------|
| formId | ✅ | 单据 formId |
| fieldKeys | | 字段列表，逗号分隔，默认 "FNumber,FName" |
| filterString | | 过滤条件，SQL 风格 |
| orderString | | 排序 |
| topRowCount | | 总行数上限 |
| startRow | | 起始行（分页用） |
| limit | | 每页行数 |

### 3.2 ⚠️ 分页技巧（重要）

金蝶 `ExecuteBillQuery` 接口 **每页（Limit）最多返回 100 行**。

- `kingdee_query_materials`、`kingdee_query_inventory`、`kingdee_query_partners` 等**专用工具没有分页参数**，只能拿到前 100 行（且多组织账套会因同一物料每组织一条而出现大量重复）。
- 需要**全量数据**时，改用 `kingdee_query_bills` 传 `startRow` + `limit` 循环分页：`startRow=0, 100, 200...`，直到返回行数 < 100。
- 多组织账套查基础资料（物料/客户/供应商）会重复，需按编码去重。

---

## 四、常用 formId 对照表

### 基础资料

| formId | 名称 |
|--------|------|
| BD_Customer | 客户 |
| BD_Supplier | 供应商 |
| BD_Material | 物料 |
| BD_Stock | 仓库 |
| BD_Department | 部门 |
| BD_Empinfo | 员工 |
| BD_Account | 会计科目 |
| BD_Currency | 币别 |

### 供应链单据

| formId | 名称 | 关键字段 |
|--------|------|---------|
| PUR_PurchaseOrder | 采购订单 | FSupplyId |
| PUR_ReceiveBill | 收料通知单 | — |
| PUR_InStock | 采购入库单 | FRealQty |
| PUR_ReturnBill | 退料通知单 | — |
| SAL_SaleOrder | 销售订单 | FSalerID |
| SAL_DeliveryNotice | 发货通知单 | — |
| SAL_OUTSTOCK | 销售出库单 | FSalesManID, FRealQty |
| SAL_RETURNSTOCK | 销售退货单 | FRealQty |
| AR_Receivable | 应收单 | — |
| AR_ReceiveBill | 收款单 | — |
| AP_Payable | 应付单 | — |
| AP_PayBill | 付款单 | — |

### 生产单据

| formId | 名称 |
|--------|------|
| SP_PickMtrl | 简单生产领料单 |
| SP_ReturnMtrl | 简单生产退料单 |
| SP_InStock | 简单生产入库单 |
| SP_OUTSTOCK | 简单生产退库单 |

### 库存单据（实测确认，注意方向）

| formId | 实际单据 | 单据编号前缀 | 数量字段 | 方向 |
|--------|---------|-------------|---------|------|
| SP_InStock | 简单生产入库单 | JDSCRK | FRealQty | 入库 + |
| SAL_RETURNSTOCK | 销售退货单 | XTRCCPXS | FRealQty | 入库 + |
| STK_InStock | 采购入库单 | CGRK | FRealQty | 入库 + |
| STK_Miscellaneous | 其他入库单 | QTRK | FQty | 入库 + |
| SAL_OUTSTOCK | 销售出库单 | XCRCCP | FRealQty | 出库 - |
| STK_MisDelivery | 其他出库单（⚠️非"盘盈入库"） | QTCK | FQty | 出库 - |
| SP_OUTSTOCK | 简单生产退库单 | — | 字段特殊 | 出库 - |
| STK_Inventory | 即时库存 | — | FBaseQty | — |

> ⚠️ 实测结论：`STK_MisDelivery` 是**其他出库单**（QTCK），不是代码注释里的"盘盈入库"；`STK_MisOut`、`PUR_InStock` 这两个 formId 在金蝶中**不存在**（报"业务对象不存在"）。盘盈盘亏通过其他入库/其他出库单的业务类型体现。

### 报表

| formId | 名称 | 说明 |
|--------|------|------|
| GL_RPT_AccountBalance | 科目余额表 | 走 `kingdee_query_account_balance`，**不要**用 query_bills |
| GL_Voucher | 总账凭证 | 单据，可用 query_bills |

---

## 五、字段书写规则

- **引用字段**：`FSalerID.FName`（取关联对象名称）、`FStockOrgId.FNumber`（取关联对象编码）
- **过滤条件**：SQL 风格，如 `FDocumentStatus = 'C' and FDate >= '2026-01-01'`
- **单据状态**：`FDocumentStatus` A=暂存, B=已提交, C=已审核
- **模糊查询**：`FName like '%关键字%'`

### 高频易错字段

| 场景 | 正确字段 | 错误写法 |
|------|---------|---------|
| 销售出库单销售员 | `FSalesManID` | ❌ FSalerID |
| 销售订单销售员 | `FSalerID` | — |
| 销售出库/退货数量 | `FRealQty` | ❌ FQty |
| 即时库存数量 | `FBaseQty` | — |

---

## 六、故障排查

**问题 1：科目余额表返回空**
- ❌ 用 `kingdee_query_bills(GL_RPT_AccountBalance)` → 字段名不匹配返回空
- ✅ 改用 `kingdee_query_account_balance(...)` → 内部走 GetSysReportData 端点

**问题 2：凭证能查，余额表查不到**
- 余额表是金蝶**报表**（Report），凭证是单据（Bill），报表走独立端点。

**问题 3：销售出库查不到销售员**
- 销售出库单字段名是 `FSalesManID`，销售订单才是 `FSalerID`，别混用。

**问题 4：销售出库数量为 0**
- 字段名是 `FRealQty` 不是 `FQty`。

**问题 5：库存数量为 0**
- 即时库存字段名是 `FBaseQty`，配合 `FMaterialId.FName`、`FStockId.FName` 查询。

**问题 6：物料/库存查询只返回 100 行**
- 专用查询工具不分页，改用 `kingdee_query_bills` 传 `startRow` + `limit` 循环分页。

---

## 七、关键提示（速记）

1. 看到"查询科目余额" → 立刻调用 `kingdee_query_account_balance`，不要查 GL_Voucher 拼接
2. 看到"销售出库数量" → 必用 `FRealQty + FSalesManID`
3. 看到"库存" → 必用 `FBaseQty + FMaterialId`
4. 看到"要全部数据/超过100条" → 用 `kingdee_query_bills` 分页循环
5. 查询前先确认用户要的时间范围和组织范围
