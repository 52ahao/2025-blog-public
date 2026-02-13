# 1 分钟教会你：桌面图标变白如何恢复

桌面快捷方式图标突然变成一片白，只剩文件名？多半是 **图标缓存坏了**。按下面做，一两分钟就能恢复。

**说明**：在 Windows 10/11 上，图标缓存不仅可能在旧路径 `%localappdata%\IconCache.db`，更主要存放在 **`%localappdata%\Microsoft\Windows\Explorer`** 目录下的 **`iconcache_*.db`** 等文件中。最佳做法不是直接删这些文件，而是先让系统自己重建：执行 **`ie4uinit.exe -show`** 会触发系统重建图标缓存的逻辑，比手动删库更稳妥。

---

## 方法一：触发重建图标缓存（推荐）

1. **关掉所有文件夹窗口**，减少占用。
2. 按 **Win + R** 打开「运行」，输入：
   ```text
   ie4uinit.exe -show
   ```
   回车执行。系统会收到「重建图标缓存」的通知并自动处理（可能会闪一下，属正常）。
3. 等几秒，看桌面图标是否恢复正常。若已恢复，到此即可。

若执行后仍有一片白：

4. 按 **Win + R**，输入：
   ```text
   %localappdata%\Microsoft\Windows\Explorer
   ```
   回车打开该文件夹。
5. 删除其中的 **`iconcache_*.db`**（以及如需要可删 `thumbcache_*.db`）。
6. 按 **Ctrl + Shift + Esc** 打开任务管理器，找到 **Windows 资源管理器**，右键 → **重新启动**。

桌面会重新加载，图标会再次生成。

---

## 方法二：命令行先触发重建，再按需清缓存

1. **以管理员身份**打开「命令提示符」或 PowerShell。
2. 先触发系统重建图标缓存：
   ```bat
   ie4uinit.exe -show
   ```
3. 若仍无效，再执行（进入 Explorer 缓存目录、删缓存、重启资源管理器）：
   ```bat
   cd /d %localappdata%\Microsoft\Windows\Explorer
   del iconcache*.db
   del thumbcache*.db
   taskkill /f /im explorer.exe
   start explorer.exe
   ```

---

## 方法三：只针对某个软件图标变白

若是**某一个**快捷方式图标变白：

1. 右键该快捷方式 → **属性**。
2. 点 **「更改图标」**。
3. 先选一个系统自带的图标，确定；再点一次「更改图标」，改回该软件原来的图标，确定。

有时这样就能把该图标的显示修好。

---

## 为啥会变白？

Windows 会把桌面、开始菜单等处的图标存进 **图标缓存**。在 Win10/11 上，这些缓存主要在 **`%localappdata%\Microsoft\Windows\Explorer`** 的 **`iconcache_*.db`** 中（旧系统还有 `%localappdata%\IconCache.db`）。缓存损坏或冲突后，系统就只显示白块。**优先用 `ie4uinit.exe -show` 触发重建**，必要时再手动删缓存并重启资源管理器，图标即可恢复。

---

按上面顺序来：先 `ie4uinit.exe -show`，一般 1 分钟内就见效；若仍有个别图标是白的，用方法三单独修即可。
