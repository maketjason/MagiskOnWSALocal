# 接口字段映射与清洗规则

本表根据抓包样例梳理字段结构，标注来源与转换要求。若后续对接系统提供正式文档，应以正式文档为准并同步更新。

## IF-001 SyncUpdates

| 字段路径 | 含义 | 数据来源/生成方式 | 清洗与转换规则 |
| --- | --- | --- | --- |
| `Header/Action` | SOAP Action 标识 | 固定值 `.../SyncUpdates` | 保持与服务端定义一致，使用 SOAP 1.2 命名空间。 |
| `Header/MessageID` | 请求唯一标识 | 运行时生成 UUID | 使用 `urn:uuid:` 前缀，确保每次请求唯一。 |
| `Header/Security/Timestamp/Created` | 请求时间戳 | 本地系统时间 | 需与 UTC 时间同步，偏差控制在 5 分钟内。 |
| `Header/Security/WindowsUpdateTicketsToken/TicketType/user` | 账号票据 | MSA 授权上下文 | 发送前注入有效的用户令牌，避免记录明文。 |
| `Body/SyncUpdates/cookie/Expiration` | Cookie 过期时间 | 来自上次 GetCookie 响应 | 格式为 ISO8601 UTC，保持原值。 |
| `Body/SyncUpdates/cookie/EncryptedData` | 加密 Cookie 数据 | 上次 GetCookie 响应 | Base64 字符串，禁止修改。 |
| `Body/SyncUpdates/parameters/ExpressQuery` | 快速查询标识 | 业务配置 | 布尔值，缺省 `false`。 |
| `Body/SyncUpdates/parameters/InstalledNonLeafUpdateIDs/int[]` | 已安装更新 ID 列表 | 本地更新缓存 | 应去重并限制列表长度，按数值排序可提升可读性。 |
| `Body/SyncUpdates/parameters/OtherCachedUpdateIDs/int[]` | 其他缓存更新 ID | 本地更新缓存 | 需剔除超期或无效的更新 ID，避免冗余。 |
| `Body/SyncUpdates/parameters/FilterAppCategoryIds/CategoryIdentifier/Id` | 应用分类过滤 | 配置中心 | 若无过滤需求，传空集合。 |
| `Body/SyncUpdates/parameters/ExtendedUpdateInfoParameters/Locales/string[]` | 语言偏好 | 系统区域设置 | 使用两位或五位语言代码，建议按优先级排序。 |
| `Body/SyncUpdates/parameters/ProductsParameters/DeviceAttributes` | 设备属性串 | 本地系统信息 | 使用分号分隔的 `key=value` 列表，需 URL 安全字符。 |
| `Body/SyncUpdates/parameters/ProductsParameters/CallerAttributes` | 调用方属性 | 固定配置 | 字符串形式 `Interactive=1;IsSeeker=0;`，按需调整。 |

## IF-002 GetExtendedUpdateInfo2

| 字段路径 | 含义 | 数据来源/生成方式 | 清洗与转换规则 |
| --- | --- | --- | --- |
| `Header/Action` | SOAP Action | 固定值 `.../GetExtendedUpdateInfo2` | 确保命名空间及大小写准确。 |
| `Header/MessageID` | 请求 ID | UUID | 与 SyncUpdates 一致。 |
| `Header/To` | 安全端点 | 固定为 secured 路径 | 如需区域镜像，需与服务方确认。 |
| `Body/GetExtendedUpdateInfo2/updateIDs/UpdateIdentity/UpdateID` | 更新主键 | 来自 SyncUpdates 响应 | 不能为空，使用 GUID。 |
| `Body/GetExtendedUpdateInfo2/updateIDs/UpdateIdentity/RevisionNumber` | 更新修订号 | 来自 SyncUpdates 响应 | 数字类型，保持与响应一致。 |
| `Body/GetExtendedUpdateInfo2/infoTypes/XmlUpdateFragmentType` | 请求片段类型 | 配置为 `FileUrl`、`FileDecryption` | 组合传递时保持数组结构。 |
| `Body/GetExtendedUpdateInfo2/deviceAttributes` | 设备属性串 | 同 SyncUpdates | 需与同步请求保持一致，确保服务端识别。 |

## IF-003 GetCookie

| 字段路径 | 含义 | 数据来源/生成方式 | 清洗与转换规则 |
| --- | --- | --- | --- |
| `Header/Action` | SOAP Action | 固定值 `.../GetCookie` | 使用 SOAP 1.2 Action。 |
| `Header/MessageID` | 请求 ID | UUID | `urn:uuid` 格式。 |
| `Body/GetCookie/oldCookie` | 旧 Cookie | 取自上次响应 | 初次调用可为空字符串。 |
| `Body/GetCookie/lastChange` | 上次更新时间 | 来自存量数据 | ISO8601 UTC；若未知，使用首个同步时间。 |
| `Body/GetCookie/currentTime` | 当前时间 | 系统时间 | 需与 `Timestamp/Created` 保持一致。 |
| `Body/GetCookie/protocolVersion` | 协议版本 | 固定值 `1.40`（样例） | 需向微软文档确认可支持版本范围。 |

> **后续工作建议**
> 1. 根据实际对接系统的正式协议文档核实字段含义与必填规则。
> 2. 与安全团队确认票据生成与加密方式，避免在流程中泄露敏感信息。
> 3. 在数据管道中实现上述清洗逻辑，并通过单元测试验证。 
