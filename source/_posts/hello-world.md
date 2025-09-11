---
title: Arch Linux 使用 systemd 限制用户进程内存
---
# Arch Linux 使用 systemd 限制用户进程内存

Arch Linux 默认使用 systemd 作为初始化系统，可直接通过其资源限制功能对特定用户的进程内存进行管控，以下是详细操作步骤。


## 1. 为用户启用 systemd slice
systemd 的 `slice` 用于对进程进行分组和资源隔离，首先需为目标用户启用持久化的 slice 会话：
```bash
sudo loginctl enable-linger USERNAME
```
- 替换 `USERNAME` 为实际用户名（如 `xiezx11`）。
- 执行后，systemd 会将该用户的所有进程归入专属的 `user-XXX.slice`（XXX 为用户 UID，如 `user-1000.slice`）。


## 2. 配置 slice 内存限制
通过 `systemctl set-property` 命令设置目标 slice 的最大内存上限：
```bash
sudo systemctl set-property user-1000.slice MemoryMax=2G
```
- **参数说明**：
  - `user-1000.slice`：替换为实际用户的 slice 名称（可通过 `id -u USERNAME` 查看 UID）。
  - `MemoryMax=2G`：限制该 slice 下所有进程的总内存使用量不超过 2GB，可根据需求调整为 `1G`、`4G` 等。
- **灵活配置**：不同用户的 slice 可设置不同内存限制（如 `user-1001.slice MemoryMax=1G`）。


## 3. 验证配置是否生效
通过 `systemctl show` 命令查看当前 slice 的内存限制设置：
```bash
systemctl show -p MemoryMax user-1000.slice
```
- 若输出 `MemoryMax=2147483648`（即 2GB 对应的字节数），说明配置已生效。


## 4. 测试内存限制
通过 Python 脚本模拟内存占用，验证限制是否生效：

### 步骤 1：创建测试脚本
新建 `memhog.py` 文件，内容如下：
```python
data = []
try:
    while True:
        # 每次循环分配约 1MB 内存（10^6 个字符，每个字符占 1 字节）
        data.append("X" * 10**6)
except MemoryError:
    print("Out of Memory!")
```

### 步骤 2：运行测试脚本
切换到目标用户（如 `xiezx11`），执行脚本：
```bash
python3 memhog.py
```
- 当进程内存占用接近 2GB 时，会触发 `MemoryError` 并打印 `Out of Memory!`，证明内存限制已生效。


## 注意事项
- 该配置为**临时生效**，重启后会恢复默认值；若需永久生效，需将配置写入 `/etc/systemd/system/user-XXX.slice.d/override.conf`（需手动创建目录和文件）。
- 内存限制针对 slice 下所有进程的总和，而非单个进程。