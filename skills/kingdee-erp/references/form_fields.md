# Kingdee ERP Common Field Reference

## Common Fields by Form Type

### BD_Supplier (供应商)
| Field Key | Display Name | Type | Notes |
|-----------|-------------|------|-------|
| FName | 名称 | String | Supplier name |
| FNumber | 编码 | String | Supplier code |
| FShortName | 简称 | String | |
| FTaxNumber | 纳税登记号 | String | |
| FAddress | 通讯地址 | String | |
| FPhone | 联系电话 | String | |
| FContact | 联系人 | String | |
| FForbidden | 禁用状态 | String | A=可用, B=禁用 |

### BD_Customer (客户)
| Field Key | Display Name | Type | Notes |
|-----------|-------------|------|-------|
| FName | 名称 | String | Customer name |
| FNumber | 编码 | String | Customer code |
| FShortName | 简称 | String | |
| FAddress | 通讯地址 | String | |
| FPhone | 联系电话 | String | |
| FContact | 联系人 | String | |
| FForbidden | 禁用状态 | String | A=可用, B=禁用 |

### BD_Material (物料)
| Field Key | Display Name | Type | Notes |
|-----------|-------------|------|-------|
| FName | 名称 | String | Material name |
| FNumber | 编码 | String | Material code |
| FSpecification | 规格型号 | String | |
| FMaterialGroup | 物料分组 | Reference | |
| FBaseUnit | 基本单位 | Reference | |
| FForbidden | 禁用状态 | String | |

### PUR_PurchaseOrder (采购订单)
| Field Key | Display Name | Type | Notes |
|-----------|-------------|------|-------|
| FBillNo | 单据编号 | String | |
| FDate | 日期 | Date | |
| FSupplyId | 供应商 | Reference | {FNumber: 'S001'} |
| FDocumentStatus | 单据状态 | String | A/B/C |
| FCreatorId | 创建人 | String | |
| FCreateDate | 创建日期 | DateTime | |
| FBillEntry | 明细 | Array | Child entries |
| FBillEntry.FMaterialId | 物料编码 | Reference | |
| FBillEntry.FQty | 数量 | Decimal | |
| FBillEntry.FPrice | 单价 | Decimal | |
| FBillEntry.FAmount | 金额 | Decimal | |
| FBillEntry.FEntryNote | 备注 | String | |

### SAL_SaleOrder (销售订单)
| Field Key | Display Name | Type | Notes |
|-----------|-------------|------|-------|
| FBillNo | 单据编号 | String | |
| FDate | 日期 | Date | |
| FCustId | 客户 | Reference | {FNumber: 'C001'} |
| FDocumentStatus | 单据状态 | String | A/B/C |
| FBillEntry | 明细 | Array | Child entries |
| FBillEntry.FMaterialId | 物料编码 | Reference | |
| FBillEntry.FQty | 数量 | Decimal | |
| FBillEntry.FPrice | 单价 | Decimal | |
| FBillEntry.FAmount | 金额 | Decimal | |

### STK_Inventory (即时库存)
| Field Key | Display Name | Type | Notes |
|-----------|-------------|------|-------|
| FMaterialNumber | 物料编码 | String | |
| FMaterialName | 物料名称 | String | |
| FStockName | 仓库名称 | String | |
| FStockPlaceName | 仓位名称 | String | |
| FBaseQty | 基本数量 | Decimal | |
| FBaseUnit | 基本单位 | String | |
| FQty | 库存数量 | Decimal | |

## Filter String Patterns

Common filter expressions for ExecuteBillQuery:

```
# Status filters
FDocumentStatus = 'C'      # 已审核
FDocumentStatus != 'C'     # 非审核状态
FDocumentStatus = 'A'      # 暂存

# Date filters
FCreateDate >= '2024-01-01' and FCreateDate <= '2024-12-31'
FDate = '2024-06-15'

# Name/Number filters (fuzzy)
FName like '%科技有限公司%'
FNumber = 'C001'

# Compound filters
FDocumentStatus = 'C' and FDate >= '2024-01-01'
FSupplyId.FNumber = 'S001' and FDocumentStatus = 'B'
```
