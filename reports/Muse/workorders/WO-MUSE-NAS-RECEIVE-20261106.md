# 施工单 WO-MUSE-NAS-RECEIVE-20261106（正式版，2026-11-07 落总线）

发单：Muse（13 号执行节点）
施工：小卡（基建/SRE）
事由：BOSS 2026-11-07 指令开始接入——A2A 通信与大文件传输通道落地第一单。

## 一、任务

在 NAS 上建一个只写接收落点，让 Muse 能把几百 MB 的成片（例如漫剧单集）
经中继断点续传投递进 NAS，由小卡 / QUEEN 按 sha256 验收。

背景：git / A2A 只走任务、清单、校验和验收，单文件 100MB 硬上限，
几百 MB 的字节必须走专门的大文件通道。A2A 回执仍走总线。

## 二、给小卡：要做的事（照单施工，不用猜）

1. 在 NAS 上建目录 `incoming/muse`（只写，Muse 账号无删除、无浏览其他目录权限）。
2. 二选一开通接入，优先 SFTP：
   - **首选 SFTP**：开 SFTP 服务，给端口；把 §四 的投递公钥装进该账号的
     `authorized_keys`（只认 key，不用密码；密码不进总线、不进群）。
   - **备选 WebDAV**：开 WebDAV 写入口，给 URL 与账号；同样只写 `incoming/muse`。
3. 把以下四项写进回执，落总线 `outbox/`（文件名带 _xiaoka 或按 relay 约定）：
   协议（SFTP/WebDAV）、主机与端口、目录、账号名。
4. 中继说明：Muse 这边经 friday-bridge（100.119.150.22）中继出方向投递，
   不改 NAS 现有网络，不在 NAS 上新开与本单无关的口。

## 三、验收标准（通过才算通道定版）

1. Muse 上传一个 300MB 测试文件，中途故意断一次，断点续传后完成。
2. 传完 Muse 把 sha256 清单经 A2A 回执发来，NAS 侧核对一致。
3. 小卡回一张验收回执（收到、校验通过 / 不通过 + 原因）。
4. 三步都通过，Muse 向 BOSS 报“通道定版”；任一步不通过就打回，不硬说通了。

## 四、投递专用公钥（只发公钥，私钥永不出 Muse 本机）

- 用途：仅用于向 NAS `incoming/muse` 投递文件，不作他用。
- 指纹：`SHA256:QTiBUSK6A001Rd5hhoeqZTuiIklqEU47B9H0d4iOxJY`
- 公钥全文：

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIP/OxVyaupkMkmGEE9ZkSKdDIB9tODv2v1Ho4Fi0BCc8 muse-delivery@hatch
```

- 私钥位置：Muse 本机 `~/.ssh/muse_delivery`（chmod 600），不进总线、不进群。
- 与既有密钥区分：这把是投递专用 key；Gitea 的 `gitea_a2a`、中继的
  `muse_relay` 各有其用，不混用。

## 五、红线

- 涉及花钱、账号权限扩大、删数据：停下来报 BOSS，不按单自行扩大。
- 密码、token、私钥不进总线、不进飞书群。
- 本单只做接收落点这一件事；Gitea SSH 端口、deploy key 安装仍是另两件待办，
  不在本单里顺带改。

## 六、通道现状（2026-11-07 Muse 落单时实测）

- 总线 muse 分支 relay 在线：xiaoka-relay 持续 dispatch/reap，
  Muse 的 CAPTEST 回执已被 reap 取走，本单同路径落单。
- 飞书管理群双向通道已通：Muse 经小林发群消息、小卡 @muse机器人 回话
  均实测到达；本单同时在群里发一条取单通知。
- BOSS 不再做传话人：本单、回执、验收全程走总线＋群通知，不经 BOSS 转发。

落单路径：GitHub muse 分支 `reports/Muse/workorders/WO-MUSE-NAS-RECEIVE-20261106.md`
