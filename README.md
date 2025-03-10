# 基于 PyAutoGUI 和 PySide6 的键盘鼠标自动化

## 1.`如何安装`
```bash
# 升级 pip
python -m pip install --upgrade pip

# 要求
pip install PySide6
pip install pyautogui
pip install pyinstaller
```

## 2. `如何打包 python 脚本`
```bash
pyinstaller --onefile --windowed --add-data "icon/gear.png;icon" --add-data "target.csv;." --hidden-import=pyautogui --hidden-import=PySide6.QtWidgets --hidden-import=PySide6.QtCore --hidden-import=PySide6.QtGui .\pydirector.py
```


## 3. `如何运行`
> 直接运行`dist`文件夹中的可执行文件，并按照说明安排操作。

## 4. `关于编译和数据路径`

### a. 如果`dist`文件夹中没有`actions.db`
1. **动态创建**:
- 脚本在运行时使用`create_database()`创建`actions.db`。它不是 PyInstaller 捆绑到 `dist` 文件夹中的预先存在的文件（例如 `target.csv` 或 `icon/gear.png`，我们通过 `--add-data` 明确添加）。
- `dist` 文件夹仅包含已编译的可执行文件（`pydirector.exe`）和通过 `--add-data` 明确包含的任何文件。由于我们没有使用 `--add-data` 包含 `actions.db`（也不应该这样做，因为它是动态生成的），因此它不会出现在那里。

2. **由 `DB_PATH` 定义的位置**：
- 我们使用 `DB_PATH = os.path.join(os.path.expanduser("~"), "actions.db")`，将数据库放在用户的主目录中（例如，Windows 上的 `C:\Users\Lenovo\actions.db`）。这位于 `dist` 文件夹之外，该文件夹通常位于项目目录中（例如，`C:\path\to\your\project\dist`）。

3. **PyInstaller `--onefile` 行为**：
- 使用 `--onefile`，所有捆绑文件（如 `target.csv` 和 `gear.png`）在运行时都会提取到临时目录（`sys._MEIPASS`），而不是 `dist` 文件夹。但是，`actions.db` 并未捆绑 - 它被创建并写入 `DB_PATH` 指定的位置，该位置与 `dist` 文件夹或临时提取无关。

---

### b. 在哪里找到 `actions.db`
由于 `DB_PATH` 设置为 `os.path.join(os.path.expanduser("~"), "actions.db")`，因此数据库文件存储在**用户的主目录**中，而不是 `dist` 文件夹中。以下是查找方法：

- **在 Windows 上**：
- 路径：`C:\Users\YourUsername\actions.db`
- 示例：如果用户名是 `Lenovo`，则查找 `C:\Users\Lenovo\actions.db`。
- 如何检查：
1. 打开文件资源管理器。
2. 导航到 `C:\Users\Lenovo`（或按 `Win + R`，输入 `%userprofile%`，然后按 Enter）。
3. 查找 `actions.db`。

- **在 macOS 上**：
- 路径：`/Users/YourUsername/actions.db`
- 示例：`/Users/lenovo/actions.db`
- 如何检查：打开 Finder，按 `Cmd + Shift + G`，输入 `~`，然后查找 `actions.db`。

- **在 Linux 上**：
- 路径：`/home/YourUsername/actions.db`
- 示例：`/home/lenovo/actions.db`
- 如何检查：打开终端，输入`cd ~`，然后输入`ls -a` 以查看`actions.db`。

---

### c. 验证其是否正常工作
要确认数据库位于其应在的位置并包含数据：
1. **运行可执行文件**：
- 执行`dist\pydirector.exe`。
- 通过 GUI 添加一些操作（例如，“Test Move”、“move”、“100,200”）。

2. **检查文件**：
- 导航到主目录（例如，`C:\Users\Lenovo`）。
- 查找`actions.db`。如果写入了数据，它应该存在并且文件大小不为零。

3. **检查数据库**：
- 使用 SQLite 查看器（例如，SQLite 的 DB Browser）打开 `actions.db` 并验证 `actions` 表是否包含条目。
- 或者，在代码中添加调试打印：
  ```python
  def get_actions()：
  conn = sqlite3.connect(DB_PATH)
  cursor = conn.cursor()
  cursor.execute('SELECT * FROM action ORDER BY id')
  action = cursor.fetchall()
  conn.close()
  print(f"Retrieved action from {DB_PATH}: {actions}") # Debug
  return action
  ```

4. **检查日志**：
- 打开 `pydirector.log`（与可执行文件位于同一目录中或配置为写入的任何位置）以查看数据库操作是否已成功记录。

### d. 也可以设置为将所有内容（包括`actions.db`、`target.csv`和`icon/gear.png`）保存在与可执行文件（`pydirector.exe`）相同的文件夹中。
这使得应用程序可移植，这意味着用户可以将可执行文件及其相关文件移动到任何目录，只要该文件夹保持可写，它仍将正常工作。



## 7. `Todo`

- 在运行过程中添加 `interruption`（CTRL-C 似乎不起作用）