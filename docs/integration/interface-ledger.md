# 接口台账

本文档整理了仓库中现存的 Windows Update 相关 SOAP 抓包样例，形成接口台账，便于后续确认与补充。

| 接口编号 | 接口名称 | 目标系统/用途 | Endpoint | SOAP Action | 请求样例 | 鉴权方式 | 请求格式 | 备注 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| IF-001 | SyncUpdates | Windows Update ClientWebService 同步更新 | `https://fe3.delivery.mp.microsoft.com/ClientWebService/client.asmx` | `http://www.microsoft.com/SoftwareDistribution/Server/ClientWebService/SyncUpdates` | [`xml/WUIDRequest.xml`](../../xml/WUIDRequest.xml) | WS-Security + WindowsUpdateTicketsToken (MSA) | SOAP 1.2 (`application/soap+xml`) | 抓包样例包含大量缓存更新 ID，需结合实际环境裁剪。 |
| IF-002 | GetExtendedUpdateInfo2 (FileUrl) | 拉取指定更新的文件 URL 与解密信息 | `https://fe3.delivery.mp.microsoft.com/ClientWebService/client.asmx/secured` | `http://www.microsoft.com/SoftwareDistribution/Server/ClientWebService/GetExtendedUpdateInfo2` | [`xml/FE3FileUrl.xml`](../../xml/FE3FileUrl.xml) | WS-Security + WindowsUpdateTicketsToken (MSA) | SOAP 1.2 (`application/soap+xml`) | 需携带更新 ID/Revision 等参数，与 SyncUpdates 响应保持一致。 |
| IF-003 | GetCookie | 获取/刷新更新同步 Cookie | `https://fe3.delivery.mp.microsoft.com/ClientWebService/client.asmx` | `http://www.microsoft.com/SoftwareDistribution/Server/ClientWebService/GetCookie` | [`xml/GetCookie.xml`](../../xml/GetCookie.xml) | WS-Security + WindowsUpdateTicketsToken (MSA) | SOAP 1.2 (`application/soap+xml`) | oldCookie 可为空；时间戳需与服务器时间同步。 |

> **说明**
> - 抓包样例中的 `{}` 为运行时注入的动态内容（如用户标识、更新 ID 等）。
> - 若存在更多接口或协议文档，应在补充信息后更新本台账。
