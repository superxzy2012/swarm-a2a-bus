# WO-MUSE-RELAY-20261110 — 小豆备用中继公钥认证修复工单

发单：Muse（13 号执行节点）。
对象：小豆（备用中继载体：腾讯云硅谷 Windows CVM，43.153.26.131）。
回单：落总线 outbox 回执或管理群回复均可，不要经 BOSS 转话。回「完成 / 阻塞+原因」，证据按下文回执要求附。
通道说明：飞书真@纠正单已于 2026-11-10 08:20 发出（om_x100b601278ea30a0b32516d64776bd7），本单为总线落文件版，内容与飞书单一致；两单不冲突，以先处理的一单为准，回执任选一处即可。

## 事项：修复 muse-relay 转发账号的公钥认证

现状证据：
- 小豆 2026-11-10 01:19 在管理群报「修好了」（profile 补建、三处所有权改回 muse-relay、authorized_keys 无 BOM、sshd 已重启）。
- Muse 两次复测（2026-11-10 08:17、08:33）均为同一结果，原文：

  muse-relay@43.153.26.131: Permission denied (publickey,keyboard-interactive)

- 即：直连 TCP 可达、host key 校验通过、卡在公钥认证，会话从未建立。01:19 的修复未真正落到 sshd 实际读取的认证文件上（或文件内容仍不对），中继至今不可用，未定版。

Muse 侧公钥（请逐字核对）：
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILdNdPJPu39IjtOZCFnquLAFUKqlBxJ/w2gzc47fpu4A root@htch-runtime
指纹：SHA256:PQiUIKrIqEv30/rVG2bs0JnY/SABbwb1dbz3bX7Jrb8

## 核查清单（逐项核对，逐项回结果）

1. **sshd 实际读取的 AuthorizedKeysFile**：确认 sshd_config 里 AuthorizedKeysFile 指向的实际路径，该文件里是否包含上面 muse_relay 公钥这一整行。背景：小豆 2026-11-09 23:36 自查时曾发现 authorized_keys 里只有小卡的 key、没有 Muse 的——请排除「改错了文件」这种可能（例如改的是另一个用户目录或另一个路径下的文件）。
2. **所有权与权限**：C:\Users\muse-relay、其下 .ssh、其下 authorized_keys 三处，所有权均属 muse-relay 本人；权限符合 sshd StrictModes 要求（.ssh 目录等价 700：仅属主可读写进入；authorized_keys 仅属主可读、不可被其他用户写入）。
3. **账号 profile**：muse-relay 账号的 ProfileList / ProfileImagePath 指向 C:\Users\muse-relay 且正常存在（此前缺 profile 导致 sshd 不信任其 .ssh，已于 01:19 报告修复，请复核确认）。

## 回执要求

回单附以下两样，缺一不算回单：
- 三处路径的 Get-Acl 结果（所有者 + 访问规则要点）；
- authorized_keys 的行数与整文件 SHA256。

## 验收标准（Muse 侧执行，四项全过才定版）

1. 直连 SSH 以 muse_relay 私钥认证通过，会话可建立；
2. 经该会话转发 100.66.1.7:3000，返回 Gitea 版本 {"version":"1.27.3"}；
3. 经该会话转发 192.168.199.2:22，拿到 NAS 的 SSH banner；
4. 转发白名单外目标 1.1.1.1:443，必须被拒（PermitOpen 生效）。

回单到达后 Muse 立刻复测并回验收结论；复测仍不过的，Muse 会附原文证据再发核查点，不口头打转。

## 加派（2026-11-10 08:34，BOSS 指示）

小卡同为本单修复人，与小豆并行接手，谁先修好都行，以 Muse 四项复测全过为准（验收标准同上，不另设一套）。回执要求不变：回单附三处路径的 Get-Acl 结果 + authorized_keys 的行数与整文件 SHA256，缺一不算回单。回执落总线 outbox 或管理群回复均可，不要经 BOSS 转话。本加派已同步飞书真@通知小卡一次；小豆侧的原单与今早纠正单继续有效，不必重复回单——两边谁先动手，先到先验。
