# Kingdee Cloud Star WebAPI Reference

## Authentication (AppSecret Mode)

### LoginByAppSecret

```
POST {BASE_URL}/Kingdee.BOS.WebApi.ServicesStub.AuthService.LoginByAppSecret.common.kdsvc
Content-Type: application/json

{
  "parameters": ["AcctID", "Username", "AppId", "AppSecret", LCID]
}
```

Returns session cookie. LCID defaults to 2052 (Simplified Chinese).

## Core API Endpoints

### View (Query Single Record)
```
POST /Kingdee.BOS.WebApi.ServicesStub.DynamicFormService.View.common.kdsvc
{
  "parameters": [
    "formId",
    "{\"CreateOrgId\": 0, \"Number\": \"BILL001\"}"
  ]
}
```
OR by inner ID:
```
"{\"Id\": 12345}"
```

### ExecuteBillQuery (Query Multiple Records)
```
POST /Kingdee.BOS.WebApi.ServicesStub.DynamicFormService.ExecuteBillQuery.common.kdsvc
{
  "parameters": [
    "formId",           // Business object ID
    "FieldKeys",        // Comma-separated field list, empty = all
    "FilterString",     // Filter expression
    "OrderString",      // Sort expression
    "TopRowCount",      // Max rows to return
    "StartRow",         // Start offset
    "Limit"             // Page size
  ]
}
```

### Save (Create/Update)
```
POST /Kingdee.BOS.WebApi.ServicesStub.DynamicFormService.Save.common.kdsvc
{
  "parameters": [
    "formId",
    "{\"NeedUpDateFields\":[],\"NeedReturnFields\":[],...\"Model\":{...}}"
  ]
}
```

Save payload structure:
```json
{
  "NeedUpDateFields": [],
  "NeedReturnFields": [],
  "IsDeleteEntry": true,
  "SubSystemId": "",
  "IsVerifyBaseDataField": false,
  "IsEntryBatchFill": true,
  "ValidateFlag": true,
  "NumberSearch": true,
  "InterationFlags": "",
  "IsAutoSubmitAndAudit": false,
  "Model": {
    "FName": "...",
    "FNumber": "...",
    "FBillEntry": []
  }
}
```

### Submit
```
POST /Kingdee.BOS.WebApi.ServicesStub.DynamicFormService.Submit.common.kdsvc
{
  "parameters": [
    "formId",
    "{\"CreateOrgId\": 0, \"Numbers\": [\"BILL001\", \"BILL002\"]}"
  ]
}
```

### Audit
```
POST /Kingdee.BOS.WebApi.ServicesStub.DynamicFormService.Audit.common.kdsvc
{
  "parameters": [
    "formId",
    "{\"CreateOrgId\": 0, \"Numbers\": [\"BILL001\", \"BILL002\"]}"
  ]
}
```

### UnAudit
```
POST /Kingdee.BOS.WebApi.ServicesStub.DynamicFormService.UnAudit.common.kdsvc
```
Same parameter format as Audit.

### Delete
```
POST /Kingdee.BOS.WebApi.ServicesStub.DynamicFormService.Delete.common.kdsvc
```
Same parameter format as Submit. Only works on non-audited bills.

## Reference Field Format

For FNumber-based references (customer, supplier, material, department, etc.):
```json
"FCustId": {"FNumber": "C001"}
```

NOT:
```json
"FCustId": "C001"  // WRONG
```

For FId-based references (less common):
```json
"FCustId": {"FId": 12345}
```

## Response Format

### Success
```json
{
  "Result": {
    "ResponseStatus": {
      "IsSuccess": true,
      "Errors": [],
      "SuccessEntitys": [],
      "SuccessMessages": []
    },
    "Id": 12345,
    "Number": "BILL001"
  }
}
```

### Failure
```json
{
  "Result": {
    "ResponseStatus": {
      "IsSuccess": false,
      "Errors": [
        {
          "FieldName": "FCustId",
          "Message": "客户不存在",
          "DIndex": 0
        }
      ]
    }
  }
}
```

## Common Error Codes

| Error Message | Cause | Solution |
|--------------|-------|----------|
| 认证失败 | Wrong AppID/AppSecret | Check KINGDEE_APP_ID and KINGDEE_APP_SEC |
| 客户不存在 | Invalid customer code | Verify FNumber exists in BD_Customer |
| 物料不存在 | Invalid material code | Verify FNumber exists in BD_Material |
| 单据状态不允许操作 | Wrong status for action | Check FDocumentStatus |
| 连接超时 | Server unreachable | Check KINGDEE_BASE_URL and network |
