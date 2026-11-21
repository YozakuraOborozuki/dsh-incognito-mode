[README.md](https://github.com/user-attachments/files/35218529/README.md)
# 无痕模式 · dsh-incognito-mode

给 DeepSeek Harness（DSH）加一个“无痕模式”。

按钮在输入框下方、权限模式「工作区内修改」和 Plan 控件旁边。无痕会话仍然留在侧栏里，可以和其他会话自由切换；切回无痕会话时按钮仍显示「无痕中」。

> ⚠️ 这个插件会**真的删除**无痕会话的聊天记录，以及它用写作工具新建的文件；如需要保存请将文件拖出到工作区外的其他文件夹。建议先用一个不重要的会话试一次。删除发生在**下一次启动 DSH、载入会话列表之前**。

---

## 功能

- **手动开启**：无痕模式不会自动开启，每次都要点「无痕」。
- **没有上文时**：把当前空白会话设为无痕。
- **有上文时**：在同一工作区新建一个空白会话并切过去，原会话保持原样。
- **多个无痕会话并存**：可以从不同普通会话各开一个；每个都独立保持无痕状态。
- **侧栏可见、可切换**：其他会话照常显示；无痕会话也显示，点它切回来仍是「无痕中」。
- **关闭无痕**：
  - 未使用的空白无痕会话：点一下「无痕中」直接关闭，变回普通空白会话；
  - 已有内容的无痕会话：点「无痕中」会先确认，确认后变普通，退出时不再清理。
- **会话内容搜索**：Host 层会过滤掉无痕会话的搜索结果。
- **首次提示**：弹窗里可勾选「不再提示（这台设备）」，存在浏览器 `localStorage`，以后直接开启。
- **退出后清理**：下一次启动、会话列表构建之前，删除所有无痕会话的历史记录和它们新建的文件。


### 本地目录安装

1. 把 `dsh-incognito-mode` 整个目录放到任意位置；
2. 打开 DSH → **Plugins → Add plugin**，选择这个目录；
3. 或让 `plugin_manager` 用 `install_bundle` 指向目录的绝对路径；
4. 重启 DSH（Client bundle 在启动时构建）。

### 从 GitHub 安装

把本目录推到 GitHub 后，可以用 DSH 的 Git 安装目标：

```sh
dsh plugin --profile desktop add github:<your-github-user>/dsh-incognito-mode
```

或在 **Plugins → Add plugin** 中选择从 Git 安装并填写同一个 `github:<user>/<repo>` 目标。安装后重启。

### 从 npm 安装（发布后）

```sh
dsh plugin --profile desktop add dsh-incognito-mode@1.0.0
```

## 数据与隐私边界

- marker 在 `$DSH_HOME/plugins-data/dsh-incognito-mode/<profile>/state.json`，按 DSH profile 隔离。只记录 Session id、工作目录、时间戳、新建文件路径；不记录消息正文。
- 无痕会话本身仍然是一个正常的 DSH Session：模型推理会把对话发送给你配置的模型服务商。**这不是“对模型服务商隐身”**，只是不长期保留在 DSH 的会话记录里。
- 插件不会把会话内容上传到别的地方，也不写入长期记忆；退出后的下一次启动会删除会话日志。
- 侧栏和搜索过滤在 Client / Session 搜索层完成；**DSH 没有 Session 级 ACL**，绕过本插件的 API 调用仍可能按 Session id 读到正在运行的会话。

## 必须告知使用者的限制

1. **会删除文件，且删除时机是下一次启动前**。退出 DSH 本身不做同步删除，是为了避免 Windows 上会话日志句柄未释放时卡住 Host shutdown。下次启动、列表加载前才会清理。
2. **只追踪写作工具创建的文件**：`write` / `edit` / `str_replace_editor` 等工具调用的目标路径，并且只删除“调用时原本不存在”、真实路径在该会话工作目录内、且不在 `$DSH_HOME` 内的普通文件。
   - 被修改的原有文件不会还原；
   - `pwsh` / `bash` 里用重定向或脚本直接创建的文件**不在追踪范围内**；
   - 目录不会被删除；如需要保存某个文件，请在清理前把它移到工作区之外。
3. **假设默认的 JSONL 会话存储布局**：`<DSH_HOME>/sessions/<项目>/<会话id>/`。如果部署配置了自定义 session persistence root，清理可能找不到文件；升级 DSH 后也要重新验证。
4. **依赖当前版本的内部接口**：`ctx.connection.rpc`、`ctx.sessionController.search`、`uiWorkspace`、`conversation.input.left` / `conversation.composer.dock` 槽位。其他 DSH 版本（尤其跨 minor）可能失效，`engines.dsh` 只是声明，不保证兼容。
5. **桌面端原生退出对话框不能改文案**：DSH Desktop 的「退出 DeepSeek Harness？」由 Electron 主进程显示，插件无法替换。删除警示出现在开启时的确认弹窗和会话期间的状态条；不要声称退出时会弹出带删除文案的原生框。
6. **不是安全隔离**：其他会话在 UI 中看不到无痕会话的内容搜索命中，但 DSH 没有 Session ACL；有本机文件/进程访问权的人仍可能读到运行中的会话日志或 marker。
7. **只在 Windows Desktop 实测**：代码使用 Node 内置 API，理论上跨平台，但 macOS/Linux、DSH Web profile、其他 DSH 版本均未验证。
8. **多 profile 不再共享 marker**：marker 按 `DSH_PROFILE` 分目录；同一 profile 仍应只运行一个 DSH 进程。
9. **卸载前先清理**：如果还有无痕会话，先点「无痕中 → 关闭无痕模式」，或重启一次让启动清理完成，再卸载插件；否则 marker/会话可能留在磁盘上，需要手工处理。
10. **没有自动迁移旧版 marker**：旧的本地预览版 marker 在 `plugins-data/dsh-incognito-mode/state.json`，公开版 1.0.0 起写到 `<profile>/state.json`；如果你从本地预览版升级，请先让旧版完成一次启动清理，或手工检查旧 marker。

## 卸载

在 **Plugins** 里移除 `dsh-incognito-mode`，然后重启 DSH。若卸载时仍有活跃无痕会话，建议先关闭它们或重启一次完成启动清理。

## 开发与验证

- 无构建步骤：Host 是普通 ESM `index.js`，Client 是手写 `window.__ModuleLoader__` bundle（`client.js`）。
- Host 只用 Node 内置模块；Client 只用 DSH 页面提供的 `react` / `react-dom`。
- 本机离线 smoke：`smoke-host`、`smoke-recover`、`smoke-search`、`smoke-client`（不随包发布）。




