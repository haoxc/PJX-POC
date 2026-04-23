---
type: uml-design
title: 编码管理功能 UML PlantUML
aliases:
  - 数据基座/编码标识管理/编码管理功能/UML/PlantUML/V001-0422
status: draft
created: 2026-04-23
version: V001
module: 编码标识管理
function_name: 编码管理功能
lifecycle_stage: 设计
stage: uml-design
source:
  - "[[编码管理功能-接口设计-V001-0422]]"
  - "[[编码管理功能-接口设计(Mermaid)-UML-V001-0422]]"
render_engine: PlantUML
tags:
  - module-design
  - uml-design
  - plantuml
  - code-data
---

# 编码管理功能 UML PlantUML

本文档基于《编码管理功能接口设计》转换，使用 PlantUML 表达 UML 视图，适用于支持 `@startuml` 的 Obsidian 插件或外部 PlantUML 渲染器。

## 1. 用例图

```plantuml
@startuml code-management-use-case
left to right direction
skinparam packageStyle rectangle

actor "数据维护人员" as Maintainer
actor "外部系统" as ExternalSystem
actor "后续映射/校验/消费链路" as Downstream

rectangle "编码管理功能" {
  usecase "分页查询编码记录" as UC_Page
  usecase "查询编码详情" as UC_Detail
  usecase "新增编码" as UC_Create
  usecase "编辑编码" as UC_Update
  usecase "删除编码" as UC_Delete
  usecase "批量导入编码" as UC_Import
  usecase "查询导入批次结果" as UC_Batch
  usecase "查询清洗建议" as UC_Clean
  usecase "重复冲突识别" as UC_Duplicate
  usecase "基础规则校验" as UC_Validate
}

Maintainer --> UC_Page
Maintainer --> UC_Detail
Maintainer --> UC_Create
Maintainer --> UC_Update
Maintainer --> UC_Delete
Maintainer --> UC_Import
Maintainer --> UC_Batch
Maintainer --> UC_Clean

ExternalSystem --> UC_Create
ExternalSystem --> UC_Import
Downstream --> UC_Page
Downstream --> UC_Detail

UC_Create .> UC_Duplicate : <<include>>
UC_Update .> UC_Duplicate : <<include>>
UC_Import .> UC_Duplicate : <<include>>
UC_Create .> UC_Validate : <<include>>
UC_Update .> UC_Validate : <<include>>
UC_Import .> UC_Validate : <<include>>
UC_Clean .> UC_Validate : <<include>>
@enduml
```

## 2. 组件图

```plantuml
@startuml code-management-component
skinparam componentStyle rectangle

actor "数据维护人员" as User
component "编码管理界面" as UI

package "编码管理后端" {
  component "CodeManagementApi\nREST Controller" as CodeApi
  component "CodeManagementService\n编码维护服务" as CodeSvc
  component "ImportBatchService\n导入批次服务" as BatchSvc
  component "ImportProcessor\n异步导入处理" as ImportProcessor
  component "CodeValidationService\n基础校验/判重" as ValidationSvc
  component "CleaningSuggestionService\n清洗建议计算" as CleaningSvc
}

database "z_code_record\n编码记录表" as CodeTable
database "z_code_import_batch\n导入批次表" as BatchTable

User --> UI
UI --> CodeApi : 调用编码管理接口
CodeApi --> CodeSvc : 单条维护/查询
CodeApi --> BatchSvc : 导入受理/批次查询
CodeApi --> CleaningSvc : 查询清洗建议
CodeSvc --> ValidationSvc : 基础校验/重复冲突识别
CodeSvc --> CodeTable : 读写编码记录
BatchSvc --> BatchTable : 创建/查询批次
BatchSvc --> ImportProcessor : 提交异步任务
ImportProcessor --> ValidationSvc : 校验导入行
ImportProcessor --> CodeTable : 写入有效编码
ImportProcessor --> BatchTable : 更新批次状态和问题明细
CleaningSvc --> CodeTable : 读取编码记录
CleaningSvc --> ValidationSvc : 实时规则计算
@enduml
```

## 3. 类图

```plantuml
@startuml code-management-class
skinparam classAttributeIconSize 0

class CodeRecord {
  +string codeId
  +string codeValue
  +string systemCode
  +string subjectType
  +CodeStatus status
  +SourceType source
  +QualityFlag qualityFlag
  +object attributes
  +string createdBy
  +datetime createdTime
  +string updatedBy
  +datetime updatedTime
  +int deletedFlag
}

class CodeDetail {
  +string codeId
  +string codeValue
  +string systemCode
  +string subjectType
  +CodeStatus status
  +SourceType source
  +QualityFlag qualityFlag
  +object attributes
  +string createdBy
  +datetime createdTime
  +string updatedBy
  +datetime updatedTime
}

class CreateCodeRequest {
  +string codeValue
  +string systemCode
  +string subjectType
  +SourceType source
  +object attributes
}

class UpdateCodeRequest {
  +string codeValue
  +string subjectType
  +CodeStatus status
  +SourceType source
  +object attributes
}

class ImportCodeRequest {
  +string systemCode
  +string subjectType
  +List<ImportCodeRow> rows
}

class ImportCodeRow {
  +string codeValue
  +SourceType source
}

class ImportBatchResult {
  +string batchId
  +BatchStatus batchStatus
  +string systemCode
  +string subjectType
  +int successCount
  +int warningCount
  +int errorCount
  +List<DataIssue> issues
  +datetime startedTime
  +datetime finishedTime
  +string failReason
  +string createdBy
  +datetime createdTime
  +string updatedBy
  +datetime updatedTime
  +int deletedFlag
}

class DataIssue {
  +int rowNo
  +string codeValue
  +string issueType
  +string issueMessage
  +Severity severity
  +string suggestion
}

class CleaningSuggestionResult {
  +string codeId
  +List<CleaningSuggestion> suggestions
}

class CleaningSuggestion {
  +string suggestionType
  +Severity severity
  +string currentValue
  +string suggestedValue
  +string issueMessage
}

enum CodeStatus {
  draft
  ready
  warning
  blocked
  deleted
}

enum QualityFlag {
  normal
  format_risk
  duplicate_risk
  missing_attribute
  manual_reviewed
}

enum SourceType {
  manual
  excel
  api
  sync
}

enum BatchStatus {
  pending
  processing
  completed
  failed
}

enum Severity {
  info
  warning
  error
}

CodeRecord <|-- CodeDetail
CreateCodeRequest ..> CodeRecord : create
UpdateCodeRequest ..> CodeRecord : update
ImportCodeRequest *-- ImportCodeRow
ImportBatchResult *-- DataIssue
CleaningSuggestionResult *-- CleaningSuggestion
CodeRecord --> CodeStatus
CodeRecord --> QualityFlag
CodeRecord --> SourceType
ImportBatchResult --> BatchStatus
DataIssue --> Severity
CleaningSuggestion --> Severity

note right of QualityFlag
接口真实取值：
format-risk
duplicate-risk
missing-attribute
manual-reviewed
end note
@enduml
```

## 4. 接口时序图

### 4.1 新增或编辑编码

```plantuml
@startuml code-management-save-sequence
autonumber

actor "数据维护人员" as User
participant "编码管理界面" as UI
participant "CodeManagementApi" as Api
participant "CodeManagementService" as Service
participant "CodeValidationService" as Validator
database "z_code_record" as CodeTable

User -> UI : 填写编码信息
UI -> Api : POST /codes 或 PUT /codes/{codeId}
Api -> Service : saveCode(request)
Service -> Validator : validateRequiredFields(request)
Validator --> Service : 字段校验结果
Service -> Validator : checkDuplicate(systemCode, subjectType, codeValue, deletedFlag=0)
Validator -> CodeTable : 查询有效唯一键
CodeTable --> Validator : 冲突结果

alt 存在重复或业务冲突
  Service --> Api : 409 conflict
  Api --> UI : 编码重复或数据冲突
else 校验通过
  Service -> CodeTable : 新增/更新编码记录
  CodeTable --> Service : 保存结果
  Service --> Api : 201 created 或 200 updated
  Api --> UI : 返回 codeId 和处理结果
end
@enduml
```

### 4.2 删除编码

```plantuml
@startuml code-management-delete-sequence
autonumber

actor "数据维护人员" as User
participant "编码管理界面" as UI
participant "CodeManagementApi" as Api
participant "CodeManagementService" as Service
participant "CodeValidationService" as Validator
database "z_code_record" as CodeTable
participant "映射/校验/消费链路" as Downstream

User -> UI : 删除编码
UI -> Api : DELETE /codes/{codeId}
Api -> Service : deleteCode(codeId)
Service -> CodeTable : 查询编码记录
CodeTable --> Service : CodeRecord
Service -> Validator : checkDeleteConstraint(codeId)
Validator -> Downstream : 检查映射、构建前校验、稳定输出引用
Downstream --> Validator : 引用检查结果

alt 存在受保护引用
  Service --> Api : 409 conflict
  Api --> UI : 禁止直接删除
else 可删除
  Service -> CodeTable : deletedFlag=1, status=deleted
  CodeTable --> Service : 更新成功
  Service --> Api : 200 deleted
  Api --> UI : 返回 deletedFlag=1
end
@enduml
```

### 4.3 批量导入编码

```plantuml
@startuml code-management-import-sequence
autonumber

actor "数据维护人员" as User
participant "编码管理界面" as UI
participant "CodeManagementApi" as Api
participant "ImportBatchService" as BatchSvc
database "z_code_import_batch" as BatchTable
queue "ImportProcessor" as Processor
participant "CodeValidationService" as Validator
database "z_code_record" as CodeTable

User -> UI : 上传导入数据
UI -> Api : POST /codes/import
Api -> BatchSvc : acceptImport(request)
BatchSvc -> BatchTable : 创建 pending 批次
BatchTable --> BatchSvc : batchId
BatchSvc -> Processor : 提交异步导入任务
BatchSvc --> Api : 202 accepted
Api --> UI : 返回 batchId

Processor -> BatchTable : batchStatus=processing
loop 遍历导入行
  Processor -> Validator : 校验格式、必填、批内重复、库内重复
  Validator -> CodeTable : 查询有效唯一键
  CodeTable --> Validator : 判重结果
  alt 行数据有效
    Processor -> CodeTable : 写入编码记录
  else 行数据异常
    Processor -> Processor : 记录 DataIssue
  end
end

alt 系统级异常
  Processor -> BatchTable : batchStatus=failed, failReason
else 导入处理完成
  Processor -> BatchTable : batchStatus=completed, success/warning/errorCount, issues
end
@enduml
```

### 4.4 查询清洗建议

```plantuml
@startuml code-management-cleaning-sequence
autonumber

actor "数据维护人员" as User
participant "编码管理界面" as UI
participant "CodeManagementApi" as Api
participant "CleaningSuggestionService" as CleaningSvc
database "z_code_record" as CodeTable
participant "CodeValidationService" as Validator

User -> UI : 查看清洗建议
UI -> Api : GET /codes/{codeId}/clean-suggestions
Api -> CleaningSvc : getSuggestions(codeId)
CleaningSvc -> CodeTable : 查询编码记录
CodeTable --> CleaningSvc : CodeRecord
CleaningSvc -> Validator : 实时计算清洗建议
Validator --> CleaningSvc : trim-space / case-normalize / invalid-char / duplicate / missing-segment
CleaningSvc --> Api : CleaningSuggestionResult
Api --> UI : 返回建议值和问题说明
@enduml
```

## 5. 状态机图

### 5.1 编码记录状态机

```plantuml
@startuml code-record-state
[*] --> draft : 新增录入

draft --> ready : 基础校验通过
draft --> warning : 存在非阻断警告
draft --> blocked : 存在阻断问题
ready --> warning : 重新校验发现警告
ready --> blocked : 重新校验发现阻断问题
warning --> ready : 人工修正/复核通过
warning --> blocked : 警告升级为阻断问题
blocked --> ready : 问题修正并通过校验
blocked --> warning : 阻断问题解除但仍有警告
ready --> deleted : 逻辑删除
warning --> deleted : 逻辑删除
blocked --> deleted : 逻辑删除
draft --> deleted : 逻辑删除
deleted --> [*]
@enduml
```

### 5.2 导入批次状态机

```plantuml
@startuml import-batch-state
[*] --> pending : POST /codes/import
pending --> processing : 导入任务开始执行
processing --> completed : 导入完成
processing --> failed : 系统级异常
completed --> [*]
failed --> [*]
@enduml
```

## 6. ER 图

```plantuml
@startuml code-management-er
hide circle
skinparam linetype ortho

entity "z_code_record" as code_record {
  * code_id : varchar(64) <<PK>>
  --
  * code_value : varchar(128)
  * system_code : varchar(64)
  * subject_type : varchar(64)
  * status : varchar(32)
  source : varchar(64)
  quality_flag : varchar(64)
  attributes_json : json
  created_by : varchar(64)
  created_time : datetime
  updated_by : varchar(64)
  updated_time : datetime
  * deleted_flag : tinyint(1)
  --
  uk_z_code_record_active : system_code + subject_type + code_value + deleted_flag
}

entity "z_code_import_batch" as import_batch {
  * batch_id : varchar(64) <<PK>>
  --
  system_code : varchar(64)
  subject_type : varchar(64)
  * batch_status : varchar(32)
  * success_count : int
  * warning_count : int
  * error_count : int
  issues_json : json
  started_time : datetime
  finished_time : datetime
  fail_reason : varchar(255)
  created_by : varchar(64)
  created_time : datetime
  updated_by : varchar(64)
  updated_time : datetime
  * deleted_flag : tinyint(1)
}

import_batch ||..o{ code_record : "导入成功后生成编码记录\n通过 system_code/subject_type/code_value 形成业务关联"
@enduml
```

## 7. 接口与对象映射

| 接口 | UML 参与对象 | 主要结果 |
| --- | --- | --- |
| `GET /api/code-management/codes/page` | `CodeRecord` | 分页返回编码记录列表 |
| `GET /api/code-management/codes/{codeId}` | `CodeDetail` | 返回编码详情 |
| `POST /api/code-management/codes` | `CreateCodeRequest` `CodeRecord` | 创建编码并返回 `codeId` |
| `PUT /api/code-management/codes/{codeId}` | `UpdateCodeRequest` `CodeRecord` | 更新编码并重新校验状态 |
| `DELETE /api/code-management/codes/{codeId}` | `CodeRecord` | 逻辑删除，`deletedFlag=1` 且 `status=deleted` |
| `POST /api/code-management/codes/import` | `ImportCodeRequest` `ImportCodeRow` `ImportBatchResult` | 创建导入批次并异步处理 |
| `GET /api/code-management/import-batches/{batchId}` | `ImportBatchResult` `DataIssue` | 查询导入结果、统计数和问题明细 |
| `GET /api/code-management/codes/{codeId}/clean-suggestions` | `CleaningSuggestionResult` `CleaningSuggestion` | 实时返回清洗建议，不直接修改原始数据 |

## 8. 关键业务约束

- 有效唯一键：`systemCode + subjectType + codeValue + deletedFlag`。
- 新增、编辑、导入发现有效重复记录时返回 `409`。
- 同一导入批次内重复编码按 `duplicate-in-batch` 记入 `issues`，不落正式编码表。
- 删除采用逻辑删除；存在已确认映射、构建前校验引用或稳定消费输出引用时返回 `409`。
- 编辑接口不允许修改 `systemCode`。
- 清洗建议按实时规则计算，不单独落库，不自动覆盖原始编码。
