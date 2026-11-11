---
name: safe-lab-change
description: Use before any operation that could alter the host, VirtualBox VMs, disks, network, firewall, audit policy or endpoint configuration.
---

# Safe lab change review

Before any potentially impactful operation:
1. Identify the exact target machine, VM, disk, adapter, file or service.
2. Explain what the command/action does.
3. Classify it as read-only, reversible change, or potentially destructive.
4. Explain privileges required and possible side effects.
5. Identify backup/snapshot/rollback method.
6. Show the exact command/action before execution.
7. Obtain explicit owner approval.
8. Execute only the approved operation.
9. Verify the result and document rollback instructions.
Never modify HoneyWatch/Cowrie assets without explicit approval. Never run destructive tests or target public systems.
