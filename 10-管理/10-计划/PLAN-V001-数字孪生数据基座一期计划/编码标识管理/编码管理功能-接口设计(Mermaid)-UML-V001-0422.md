---
type: uml-design
title: 编码管理功能 UML
aliases:
  - 数据基座/编码标识管理/编码管理功能/UML/V001-0422
status: draft
created: 2026-04-23
version: V001
module: 编码标识管理
function_name: 编码管理功能
lifecycle_stage: 设计
stage: uml-design
source:
  - "[[编码管理功能-接口设计-V001-0422]]"
render_engine: Mermaid
obsidian_support: Obsidian 内置 Mermaid 或常见 Mermaid 插件
tags:
  - module-design
  - uml-design
  - mermaid
  - code-data
---

# 编码管理功能 UML

本文档基于《编码管理功能接口设计》转换，使用 Mermaid 表达 UML 视图，便于在 Obsidian 中直接预览和维护。

## 1. 用例图

```mermaid
flowchart LR
    Maintainer["数据维护人员"]
    ExternalSystem["外部系统"]
    Downstream["后续映射/校验/消费链路"]

    subgraph CodeManagement["编码管理功能"]
        UC_Page(("分页查询编码记录"))
        UC_Detail(("查询编码详情"))
        UC_Create(("新增编码"))
        UC_Update(("编辑编码"))
        UC_Delete(("删除编码"))
        UC_Import(("批量导入编码"))
        UC_Batch(("查询导入批次结果"))
        UC_Clean(("查询清洗建议"))
        UC_Duplicate(("重复冲突识别"))
        UC_Validate(("基础规则校验"))
    end

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

    UC_Create -. include .-> UC_Duplicate
    UC_Update -. include .-> UC_Duplicate
    UC_Import -. include .-> UC_Duplicate
    UC_Create -. include .-> UC_Validate
    UC_Update -. include .-> UC_Validate
    UC_Import -. include .-> UC_Validate
    UC_Clean -. include .-> UC_Validate
```

## 2. 组件图

```mermaid
flowchart LR
    User["数据维护人员"]
    UI["编码管理界面"]

    subgraph Backend["编码管理后端"]
        CodeApi["CodeManagementApi<br/>REST Controller"]
        CodeSvc["CodeManagementService<br/>编码维护服务"]
        BatchSvc["ImportBatchService<br/>导入批次服务"]
        ImportProcessor["ImportProcessor<br/>异步导入处理"]
        ValidationSvc["CodeValidationService<br/>基础校验/判重"]
        CleaningSvc["CleaningSuggestionService<br/>清洗建议计算"]
    end

    CodeTable[("z_code_record<br/>编码记录表")]
    BatchTable[("z_code_import_batch<br/>导入批次表")]

    User --> UI
    UI -->|"调用编码管理接口"| CodeApi

    CodeApi -->|"单条维护/查询"| CodeSvc
    CodeApi -->|"导入受理/批次查询"| BatchSvc
    CodeApi -->|"查询清洗建议"| CleaningSvc

    CodeSvc -->|"基础校验/重复冲突识别"| ValidationSvc
    CodeSvc -->|"读写编码记录"| CodeTable

    BatchSvc -->|"创建/查询批次"| BatchTable
    BatchSvc -->|"提交异步任务"| ImportProcessor
    ImportProcessor -->|"校验导入行"| ValidationSvc
    ImportProcessor -->|"写入有效编码"| CodeTable
    ImportProcessor -->|"更新批次状态和问题明细"| BatchTable

    CleaningSvc -->|"读取编码记录"| CodeTable
    CleaningSvc -->|"实时规则计算"| ValidationSvc
```

## 3. 类图

```mermaid
classDiagram
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
        +List~ImportCodeRow~ rows
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
        +List~DataIssue~ issues
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
        +List~CleaningSuggestion~ suggestions
    }

    class CleaningSuggestion {
        +string suggestionType
        +Severity severity
        +string currentValue
        +string suggestedValue
        +string issueMessage
    }

    class CodeStatus {
        <<enumeration>>
        draft
        ready
        warning
        blocked
        deleted
    }

    class QualityFlag {
        <<enumeration>>
        normal
        format_risk
        duplicate_risk
        missing_attribute
        manual_reviewed
    }

    class SourceType {
        <<enumeration>>
        manual
        excel
        api
        sync
    }

    class BatchStatus {
        <<enumeration>>
        pending
        processing
        completed
        failed
    }

    class Severity {
        <<enumeration>>
        info
        warning
        error
    }

    CodeDetail --|> CodeRecord
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
```

说明：`QualityFlag` 在 Mermaid 类图中使用下划线形式以增强 Obsidian 渲染兼容性；接口真实取值仍为 `format-risk`、`duplicate-risk`、`missing-attribute`、`manual-reviewed`。

## 4. 接口时序图

### 4.1 新增或编辑编码

```mermaid
sequenceDiagram
    autonumber
    actor User as 数据维护人员
    participant UI as 编码管理界面
    participant Api as CodeManagementApi
    participant Service as CodeManagementService
    participant Validator as CodeValidationService
    participant CodeTable as z_code_record

    User->>UI: 填写编码信息
    UI->>Api: POST /codes 或 PUT /codes/{codeId}
    Api->>Service: saveCode(request)
    Service->>Validator: validateRequiredFields(request)
    Validator-->>Service: 字段校验结果
    Service->>Validator: checkDuplicate(systemCode, subjectType, codeValue, deletedFlag=0)
    Validator->>CodeTable: 查询有效唯一键
    CodeTable-->>Validator: 冲突结果

    alt 存在重复或业务冲突
        Service-->>Api: 409 conflict
        Api-->>UI: 编码重复或数据冲突
    else 校验通过
        Service->>CodeTable: 新增/更新编码记录
        CodeTable-->>Service: 保存结果
        Service-->>Api: 201 created 或 200 updated
        Api-->>UI: 返回 codeId 和处理结果
    end
```

### 4.2 删除编码

```mermaid
sequenceDiagram
    autonumber
    actor User as 数据维护人员
    participant UI as 编码管理界面
    participant Api as CodeManagementApi
    participant Service as CodeManagementService
    participant Validator as CodeValidationService
    participant CodeTable as z_code_record
    participant Downstream as 映射/校验/消费链路

    User->>UI: 删除编码
    UI->>Api: DELETE /codes/{codeId}
    Api->>Service: deleteCode(codeId)
    Service->>CodeTable: 查询编码记录
    CodeTable-->>Service: CodeRecord
    Service->>Validator: checkDeleteConstraint(codeId)
    Validator->>Downstream: 检查映射、构建前校验、稳定输出引用
    Downstream-->>Validator: 引用检查结果

    alt 存在受保护引用
        Service-->>Api: 409 conflict
        Api-->>UI: 禁止直接删除
    else 可删除
        Service->>CodeTable: deletedFlag=1, status=deleted
        CodeTable-->>Service: 更新成功
        Service-->>Api: 200 deleted
        Api-->>UI: 返回 deletedFlag=1
    end
```

### 4.3 批量导入编码

```mermaid
sequenceDiagram
    autonumber
    actor User as 数据维护人员
    participant UI as 编码管理界面
    participant Api as CodeManagementApi
    participant BatchSvc as ImportBatchService
    participant BatchTable as z_code_import_batch
    participant Processor as ImportProcessor
    participant Validator as CodeValidationService
    participant CodeTable as z_code_record

    User->>UI: 上传导入数据
    UI->>Api: POST /codes/import
    Api->>BatchSvc: acceptImport(request)
    BatchSvc->>BatchTable: 创建 pending 批次
    BatchTable-->>BatchSvc: batchId
    BatchSvc->>Processor: 提交异步导入任务
    BatchSvc-->>Api: 202 accepted
    Api-->>UI: 返回 batchId

    Processor->>BatchTable: batchStatus=processing
    loop 遍历导入行
        Processor->>Validator: 校验格式、必填、批内重复、库内重复
        Validator->>CodeTable: 查询有效唯一键
        CodeTable-->>Validator: 判重结果
        alt 行数据有效
            Processor->>CodeTable: 写入编码记录
        else 行数据异常
            Processor->>Processor: 记录 DataIssue
        end
    end

    alt 系统级异常
        Processor->>BatchTable: batchStatus=failed, failReason
    else 导入处理完成
        Processor->>BatchTable: batchStatus=completed, success/warning/errorCount, issues
    end
```

### 4.4 查询清洗建议

```mermaid
sequenceDiagram
    autonumber
    actor User as 数据维护人员
    participant UI as 编码管理界面
    participant Api as CodeManagementApi
    participant CleaningSvc as CleaningSuggestionService
    participant CodeTable as z_code_record
    participant Validator as CodeValidationService

    User->>UI: 查看清洗建议
    UI->>Api: GET /codes/{codeId}/clean-suggestions
    Api->>CleaningSvc: getSuggestions(codeId)
    CleaningSvc->>CodeTable: 查询编码记录
    CodeTable-->>CleaningSvc: CodeRecord
    CleaningSvc->>Validator: 实时计算清洗建议
    Validator-->>CleaningSvc: trim-space / case-normalize / invalid-char / duplicate / missing-segment
    CleaningSvc-->>Api: CleaningSuggestionResult
    Api-->>UI: 返回建议值和问题说明
```

## 5. 状态机图

### 5.1 编码记录状态机

```mermaid
stateDiagram-v2
    [*] --> draft: 新增录入

    draft --> ready: 基础校验通过
    draft --> warning: 存在非阻断警告
    draft --> blocked: 存在阻断问题

    ready --> warning: 重新校验发现警告
    ready --> blocked: 重新校验发现阻断问题
    warning --> ready: 人工修正/复核通过
    warning --> blocked: 警告升级为阻断问题
    blocked --> ready: 问题修正并通过校验
    blocked --> warning: 阻断问题解除但仍有警告

    ready --> deleted: 逻辑删除
    warning --> deleted: 逻辑删除
    blocked --> deleted: 逻辑删除
    draft --> deleted: 逻辑删除

    deleted --> [*]
```

### 5.2 导入批次状态机

```mermaid
stateDiagram-v2
    [*] --> pending: POST /codes/import
    pending --> processing: 导入任务开始执行
    processing --> completed: 导入完成
    processing --> failed: 系统级异常

    completed --> [*]
    failed --> [*]
```

## 6. ER 图

```mermaid
erDiagram
    z_code_record {
        varchar code_id PK "编码记录主键"
        varchar code_value "编码值"
        varchar system_code "所属编码体系编码"
        varchar subject_type "对象类型"
        varchar status "编码状态"
        varchar source "数据来源"
        varchar quality_flag "质量标记"
        json attributes_json "扩展属性"
        varchar created_by "创建人"
        datetime created_time "创建时间"
        varchar updated_by "更新人"
        datetime updated_time "更新时间"
        tinyint deleted_flag "删除标记"
    }

    z_code_import_batch {
        varchar batch_id PK "导入批次主键"
        varchar system_code "导入所属编码体系"
        varchar subject_type "导入对象类型"
        varchar batch_status "导入批次状态"
        int success_count "成功数量"
        int warning_count "警告数量"
        int error_count "失败数量"
        json issues_json "问题列表"
        datetime started_time "开始处理时间"
        datetime finished_time "结束处理时间"
        varchar fail_reason "失败原因"
        varchar created_by "创建人"
        datetime created_time "创建时间"
        varchar updated_by "更新人"
        datetime updated_time "更新时间"
        tinyint deleted_flag "删除标记"
    }

    z_code_import_batch ||--o{ z_code_record : "导入成功后生成"
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

## 9. Obsidian 渲染说明

- 图块均使用 `mermaid` 代码块，Obsidian 默认支持 Mermaid 渲染。
- 若需要增强导出或样式能力，可使用常见 Mermaid/Markdown 导出插件；不依赖 PlantUML、Java 或远程渲染服务。
