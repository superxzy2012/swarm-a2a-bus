# A2A 共享 vault 回执协议 v1
- 指令消息含 [NEED_RECEIPT] 时，接收方须写回执到 `A2A/receipts/`
- 文件名: `<日期>-<发送方>-<接收方>-<主题>.md`
- 内容: 时间戳、发送方、接收方、原指令摘要、执行结果/状态 (ACK/DONE/REPORT)
- 回执经 git 同步总线共享 (sync.sh, 5min)