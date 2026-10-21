# A2A Cloud Relay

This repository is the cross-network relay between the NAS A2A root and the
Tencent Cloud VM that runs 星期五-新Hermes.

- Authority root: `/Volumes/2-AI/A2A` (Mac) = `/volume2/2-AI/A2A` (NAS)
- Cloud mirror root: `/home/agentuser/A2A` (VM `VM-43-229-ubuntu`)
- Relay dirs: `inbox/`, `outbox/`, `tasks/`, `results/`, `agents/`, `processed/`
- Not relayed: `archive/`, `secrets/`

Rules:

1. Mac writes tasks to `inbox/`; the VM writes receipts to `outbox/`.
2. Both sides sync with `git pull --rebase` + `rsync --update`.
3. Never commit tokens, passwords, or `secrets/`.
4. The VM uses SSH over port 443 (`ssh.github.com`) because HTTPS is blocked.
5. A task with a terminal receipt or a same-named file in `processed/` is
   removed from `inbox/` so it cannot be delivered twice.
