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
[text](target.csv) [text](icon) [text](actions.db)

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
使用 `--console` 重新编译并运行以查看输出。

4. **检查日志**：
- 打开 `pydirector.log`（与可执行文件位于同一目录中或配置为写入的任何位置）以查看数据库操作是否已成功记录。

---

### d.为什么
- **设计意图**：通过将 `DB_PATH` 设置为主目录，我们确保数据库存储在临时目录或构建目录之外的持久、可写位置。这允许数据在运行期间持续存在，与使用 `sys._MEIPASS` 的原始 `resource_path('actions.db')` 方法不同。
- **无需 `dist`**：`dist` 文件夹仅用于可执行文件和捆绑资源，而不是运行时生成的文件，如 `actions.db`。

---

## 5.`将“actions.db”保存在“dist”文件夹中`
要将 `actions.db` 保存在 `dist` 文件夹中，与 `pydirector.exe` 一起保存（例如，为了便于移植或组织），我们可以修改 `DB_PATH` 以指向那里。但是，这有一些注意事项：

#### 修改后的代码
```python
import os
import sys

# 将 DB_PATH 设置为包含可执行文件的目录
if getattr(sys, 'frozen',False): # 作为 PyInstaller 可执行文件运行
BASE_DIR = os.path.dirname(sys.executable) # pydirector.exe 的目录
else:
BASE_DIR = os.path.dirname(os.path.abspath(__file__)) # pydirector.py 的目录
DB_PATH = os.path.join(BASE_DIR, "actions.db")
```

- **说明**:
- `sys.executable`: 运行可执行文件的路径（例如，`C:\path\to\project\dist\pydirector.exe`）。
- `os.path.dirname(sys.executable)`: 提取目录（例如，`C:\path\to\project\dist`）。
- `DB_PATH`: 编译后变为 `C:\path\to\project\dist\actions.db`。
- 使用 Python 运行时，它使用脚本的目录。

- **警告**：
- `dist` 文件夹必须是可写的。如果我们将可执行文件分发到另一个位置（例如 `C:\Program Files`），除非以管理员身份运行，否则它可能会因权限问题而失败。
- 为了便于移植，用户需要复制整个 `dist` 文件夹，包括 `actions.db`，以保留数据。

#### 重新编译和测试
使用现有命令重新编译并从 `dist` 运行 `pydirector.exe`。检查 `actions.db` 是否出现在那里。

---

### 建议
- **将其保存在主目录中**：将 `actions.db` 存储在 `~/actions.db` 中更安全且更方便用户使用，因为它可以避免权限问题并将数据与用户绑定，而不是可执行文件的位置。
- **无需采取任何措施**：既然它现在就可以运行，除非我们有特殊原因将其移动到`dist`，否则无需更改任何内容。

---

### 最终确认
将所有内容（包括`actions.db`、`target.csv`和`icon/gear.png`）保存在与可执行文件（`pydirector.exe`）相同的文件夹中。这使得应用程序可移植，这意味着用户可以将可执行文件及其相关文件移动到任何目录，只要该文件夹保持可写，它仍将正常工作。以下是如何修改代码和 PyInstaller 命令以实现此目的。

---

### 为什么有效
- **可执行文件目录**：使用 PyInstaller 编译时，`sys.executable` 提供正在运行的可执行文件的路径（例如，`C:\path\to\dist\pydirector.exe`）。使用 `os.path.dirname(sys.executable)`，我们可以动态引用包含可执行文件的文件夹。
- **一致性**：通过调整所有文件路径（`actions.db`、`target.csv`、`gear.png`）以使用此目录，所有内容都保持在一起。
- **可移植性**：用户可以将整个文件夹（可执行文件 + 文件）复制到任何位置，应用程序仍将运行，而无需硬编码路径。

---

### 分步解决方案

#### 1. 修改代码
更新脚本以使用可执行文件的目录作为所有文件（`actions.db`、`target.csv` 和 `gear.png`）的基本路径。用新函数替换 `resource_path()` 并调整 `DB_PATH`。

以下是关键部分的修订代码片段：

```python
import os
import sys

# 确定基目录（可执行文件或脚本所在的位置）
def get_base_path():
if getattr(sys, 'frozen', False): # 作为 PyInstaller 可执行文件运行
return os.path.dirname(sys.executable) # pydirector.exe 的目录
return os.path.dirname(os.path.abspath(__file__)) # pydirector.py 的目录

BASE_PATH = get_base_path()
DB_PATH = os.path.join(BASE_PATH, "actions.db")

# 更新 resource_path 以使用 BASE_PATH 而不是 sys._MEIPASS
def resource_path(relative_path):
return os.path.join(BASE_PATH,relative_path)

# 数据库函数（使用 DB_PATH）
def create_database(db_name='actions.db'): # 为保持一致性，忽略 db_name
conn = sqlite3.connect(DB_PATH)
cursor = conn.cursor()
cursor.execute('''
CREATE TABLE IF NOT EXISTS action (
id INTEGER PRIMARY KEY AUTOINCREMENT,
action_name TEXT,
action_type TEXT,
parameters TEXT,
timestamp TEXT
)
''')
conn.commit()
conn.close()
logs.debug(f"数据库创建于：{DB_PATH}")

def add_action(action_name, action_type, parameters):
conn = sqlite3.connect(DB_PATH)
cursor = conn.cursor()
timestamp = datetime.datetime.now().isoformat()
cursor.execute('''
INSERT INTO action (action_name, action_type, parameters, timestamp)
VALUES (?, ?, ?, ?)
''', (action_name, action_type, parameters, timestamp))
conn.commit()
conn.close()
logging.debug(f"Action Added: {action_name}")

# 类似地更新其他数据库函数 (get_actions, update_action, delete_action)

# CSV 读取 (更新以使用 resource_path)
def read_target_ids(csv_file='target.csv'): # 调整默认值以匹配用法
csv_path = resource_path(csv_file)
try:
with open(csv_path, 'r') as f:
reader = csv.reader(f)
target_ids = [row[0] for row in reader]
return target_ids
except FileNotFoundError:
logging.error(f"未找到 CSV 文件：{csv_path}")
print("Error: target.csv 未找到。”)
return []

# 更新 MainWindow 中的托盘图标。__init__
self.tray_icon = QSystemTrayIcon(QIcon(resource_path("icon/gear.png")), self)
```

- **关键更改**:
- 已替换`resource_path()` 逻辑与 `BASE_PATH` 结合，后者指向捆绑时的可执行文件目录或使用 Python 运行时的脚本目录。
- 将 `DB_PATH` 设置为 `BASE_PATH + "actions.db"`。
- 更新 `read_target_ids()` 和托盘图标以使用带有新基本路径的 `resource_path()`。
- 保留使用 `DB_PATH` 的数据库函数。

#### 2. 更新 PyInstaller 命令
由于我们希望 `target.csv` 和 `icon/gear.png` 与可执行文件放在同一个文件夹中，我们仍需要使用 `--add-data` 将它们包含在内，但它们将在运行时从可执行文件的目录中访问。

使用此命令：
```
pyinstaller --onefile --windowed --add-data "icon/gear.png;icon" --add-data "target.csv;." --hidden-import=pyautogui --hidden-import=PySide6.QtWidgets --hidden-import=PySide6.QtCore --hidden-import=PySide6.QtGui --icon=icon/gear.png .\pydirector.py
```

- **注意**:
- `--add-data "icon/gear.png;icon"`: 在包的 `icon` 子文件夹中包含 `gear.png`，`resource_path("icon/gear.png")` 将解析为该文件夹。
- `--add-data "target.csv;."`: 在包的根文件夹中包含 `target.csv`，`resource_path("target.csv")` 将解析为该文件夹。
- `actions.db` 未包含在内，因为它是在运行时动态创建的。

#### 3. 测试设置
1. **使用 Python 运行**:
- 确保 `target.csv` 和 `icon/gear.png` 与 `pydirector.py` 位于同一目录中。
- 运行 `python pydirector.py`。
- 添加操作并检查 `actions.db` 是否出现在与 `pydirector.py` 相同的目录中。

2. **编译并运行**:
- 运行上面的 PyInstaller 命令。
- 转到 `dist`，我们可以看到 `pydirector.exe`。
- 构建后，手动将“target.csv”和“icon/gear.png”从项目文件夹复制到“dist”（或“gear.png”的子文件夹，如“icon/”），因为“--onefile”不会自动将它们留在那里：
```
dist/
═── pydirector.exe
═── target.csv
└── icon/
└── gear.png
```
- 运行“pydirector.exe”。
- 添加操作并确认“actions.db”与“pydirector.exe”一起在“dist”中创建。

3. **验证可移植性**：
- 将整个“dist”文件夹移动到另一个位置（例如“D:\TestFolder”）。
- 从那里运行“pydirector.exe”并确保它仍然有效（读取“target.csv”，使用“gear.png”，并在新位置写入“actions.db”）。

---

### 运行时发生的情况
- **脚本模式（`python pydirector.py`）**：
- `BASE_PATH` = `pydirector.py` 的目录。
- `DB_PATH` = `<script_dir>/actions.db`。
- `resource_path("target.csv")` = `<script_dir>/target.csv`。
- `resource_path("icon/gear.png")` = `<script_dir>/icon/gear.png`。

- **可执行模式（`pydirector.exe`）**：
- `BASE_PATH` = `pydirector.exe` 的目录（例如，`C:\path\to\dist`）。
- `DB_PATH` = `C:\path\to\dist\actions.db`。
- `resource_path("target.csv")` = `C:\path\to\dist\target.csv`（从包中提取）。
- `resource_path("icon/gear.png")` = `C:\path\to\dist\icon\gear.png`（从包中提取）。

---

### 注意事项
- **写入权限**：可执行文件的目录必须可写，才能创建/更新 `actions.db`。如果我们将其放在受限制的位置（例如，`C:\Program Files`），除非以管理员身份运行，否则可能会失败。
- **手动文件放置**：使用 `--onefile` 构建后，PyInstaller 不会将 `target.csv` 和 `gear.png` 留在 `dist` 中 - 它们嵌入在可执行文件中。我们需要手动将它们复制到 `dist`（或将它们与可执行文件一起分发），以便应用程序在运行时找到它们。

#### 替代方案：`--onedir` 模式
如果我们希望 PyInstaller 自动将所有文件放置在 `dist` 中而无需手动复制，请改用 `--onedir`：
```
pyinstaller --onedir --windowed --add-data "icon/gear.png;icon" --add-data "target.csv;." --hidden-import=pyautogui --hidden-import=PySide6.QtWidgets --hidden-import=PySide6.QtCore --hidden-import=PySide6.QtGui --icon=icon/gear.png .\pydirector.py
```
- 结果：`dist/pydirector/` 包含 `pydirector.exe`、`target.csv`、`icon/gear.png` 和依赖项。`actions.db` 也将在那里创建。
- 权衡：使用更大的文件夹而不是单个文件，但更简单

___

让我们比较一下在 PyInstaller 编译的应用程序中处理文件路径的两种方法：我们使用的原始 `resource_path()` 函数与我建议的更新方法 `get_base_path()`，该方法用于将所有内容保存在与可执行文件相同的文件夹中。我将分解它们的机制、含义和适用性，以实现将所有文件（`actions.db`、`target.csv`、`icon/gear.png`）与 `pydirector.exe` 一起保存的目标。

---

### 原始方法：`resource_path()`
```python
def resource_path(relative_path):
if hasattr(sys, '_MEIPASS'):
return os.path.join(sys._MEIPASS,relative_path)
return os.path.join(os.path.abspath("."),relative_path)
```

#### 工作原理
- **作为脚本运行时（`python pydirector.py`）**:
- `hasattr(sys, '_MEIPASS')` 为 `False`，因为 `_MEIPASS` 仅存在于 PyInstaller 包中。
- `os.path.abspath(".")`：返回当前工作目录的绝对路径（我们从哪里运行脚本，例如，如果我们在那里运行 `python pydirector.py`，则为 `C:\path\to\project`）。
- `os.path.join(os.path.abspath("."),relative_path)`：将当前目录与 `relative_path` 合并（例如，`"actions.db"`、`"target.csv"`、`"icon/gear.png"`）。
- 示例：如果从 `C:\path\to\project` 运行，则 `resource_path("actions.db")` → `C:\path\to\project\actions.db`。

- **当作为 PyInstaller 可执行文件 (`pydirector.exe`) 运行时**:
- `hasattr(sys, '_MEIPASS')` 为 `True`，因为 PyInstaller 将 `_MEIPASS` 设置为提取捆绑文件的临时目录 (例如，`C:\Users\Lenovo\AppData\Local\Temp\_MEIxxxx`)。
- `os.path.join(sys._MEIPASS,relative_path)`: 将临时目录与 `relative_path` 合并。
- 示例：`resource_path("actions.db")` → `C:\Users\Lenovo\AppData\Local\Temp\_MEIxxxx\actions.db`。

#### 含义
1. **脚本模式**:
- 如果 `target.csv` 和 `icon/gear.png` 位于当前目录中，则可正常工作。
- `actions.db` 在当前工作目录（例如 `C:\path\to\project`）中创建，该目录是持久且可写的，因此数据在运行期间会保存。

2. **可执行模式（`--onefile`）**：
- **优点**：如果使用 `--add-data` 包含，则可正确定位临时目录中的捆绑文件（`target.csv`、`icon/gear.png`）。
- **缺点**：
- `actions.db` 在 `sys._MEIPASS` 中创建，该目录：
- **临时**：应用关闭时删除，因此数据不会保留。
- **SQLite 只读**：由于其性质，SQLite 通常无法在此目录中可靠地写入，从而导致最初的问题。
- 文件分散：捆绑文件位于 `sys._MEIPASS` 中，但 `actions.db` 不在 `dist` 文件夹中或可执行文件旁边。

3. **可移植性**:
- `actions.db` 较差：数据丢失，因为它与临时位置绑定。
- 需要在构建后手动将 `target.csv` 和 `gear.png` 放置在可执行文件旁边，但它们可以从 `sys._MEIPASS` 访问。

---

### 新方法：`get_base_path()`
```python
def get_base_path():
if getattr(sys, 'frozen', False): # 作为 PyInstaller 可执行文件运行
return os.path.dirname(sys.executable) # pydirector.exe 的目录
return os.path.dirname(os.path.abspath(__file__)) # pydirector.py 的目录

BASE_PATH = get_base_path()

def resource_path(relative_path):
return os.path.join(BASE_PATH,relative_path)

DB_PATH = os.path.join(BASE_PATH, "actions.db")
```

#### 工作原理
- **作为脚本运行时（`python pydirector.py`）**：
- `getattr(sys, 'frozen', False)` 为 `False`（PyInstaller 的标志捆绑应用程序的路径未设置）。
- `os.path.abspath(__file__)`：返回 `pydirector.py` 的绝对路径（例如，`C:\path\to\project\pydirector.py`）。
- `os.path.dirname(...)`：提取目录（例如，`C:\path\to\project`）。
- `BASE_PATH`：`C:\path\to\project`。
- 示例：
- `DB_PATH` → `C:\path\to\project\actions.db`。
- `resource_path("target.csv")` → `C:\path\to\project\target.csv`。
- `resource_path("icon/gear.png")` → `C:\path\to\project\icon\gear.png`。

- **当作为 PyInstaller 可执行文件 (`pydirector.exe`) 运行时**:
- `getattr(sys, 'frozen', False)` 为 `True` (由 PyInstaller 设置)。
- `sys.executable`：可执行文件的路径 (例如，`C:\path\to\dist\pydirector.exe`)。
- `os.path.dirname(sys.executable)`：可执行文件的目录 (例如，`C:\path\to\dist`)。
- `BASE_PATH`：`C:\path\to\dist`。
- 示例：
- `DB_PATH` → `C:\path\to\dist\actions.db`。
- `resource_path("target.csv")` → `C:\path\to\dist\target.csv`。
- `resource_path("icon/gear.png")` → `C:\path\to\dist\icon\gear.png`。

#### 含义
1. **脚本模式**:
- 所有文件（`actions.db`、`target.csv`、`gear.png`）都在脚本目录中创建/访问（例如，`C:\path\to\project`）。
- 与使用 Python 运行时的原始工作设置一致。

2. **可执行模式（`--onefile`）**:
- **优点**:
- 所有文件都绑定到可执行文件的目录（例如，`C:\path\to\dist`），该目录是持久的并且可以写入（如果不在像`C:\Program Files`这样的受限位置）。
- `actions.db` 与 `pydirector.exe` 一起创建，解决了持久性问题。
- 可移植性：将 `dist` 文件夹移动到任何地方，所有内容都会保持在一起。
- **缺点**：
- 需要在构建后将 `target.csv` 和 `gear.png` 手动放置在 `dist` 中（或与可执行文件一起分发），因为 `--onefile` 将它们嵌入可执行文件中，但不会将它们留在 `dist` 中。
- 目录必须是可写的，否则 SQLite 写入将失败（例如，需要在受限文件夹中拥有管理员权限）。

3. **可移植性**：
- 优秀：整个应用程序（可执行文件 + 文件）可以作为一个单元移动，只要文件夹可写，它就可以工作。

---

### 比较表

| **方面** | **原始（`resource_path`）** | **新（`get_base_path`）** |
|---------------------------- | ------------------------------------------------------ | --------------------------------------------- |
| **脚本模式位置** | 当前工作目录（例如，`C:\path\to\project`）| 脚本目录（例如，`C:\path\to\project`）|
| **可执行模式位置** | 临时目录（`sys._MEIPASS`）| 可执行目录（例如，`C:\path\to\dist`）|
| **数据库持久性** | 否（退出时删除临时目录）| 是（与可执行文件一起存储）|
| **文件访问** | `sys._MEIPASS` 中的捆绑文件，DB 在其他位置 | 可执行文件的目录中的所有文件 |
| **可移植性** | 差（DB 丢失，文件分散）| 高（所有内容一起移动）|
| **设置工作量** | 简单但对 DB 有缺陷 | 构建后需要手动放置文件 |
| **写入权限问题** | 高（临时目录通常是只读的） | 可能（如果目录受到限制） |


---

### 为什么 `get_base_path()` 满足目标
我们希望将所有文件保存在可执行文件所在的同一文件夹中。但是，原始的 `resource_path()` 在这方面失败了，因为：
- 它将 `actions.db` 放在已删除的临时目录中，而不是与 `pydirector.exe` 放在一起。
- 虽然它从 `sys._MEIPASS` 正确访问捆绑的 `target.csv` 和 `gear.png`，但它没有将它们与数据库的位置对齐。

`get_base_path()` 方法：
- 确保 `actions.db`、`target.csv` 和 `gear.png` 都是相对于可执行文件的目录引用的（例如，`C:\path\to\dist`）。
- 通过将所有内容放在一个持久文件夹中，使应用程序可移植。

---

### 解决 `--onefile` 限制
使用 `--onefile`，PyInstaller 将 `target.csv` 和 `gear.png` 嵌入 `pydirector.exe` 中，并在运行时将它们提取到 `sys._MEIPASS`。但是，代码在 `BASE_PATH`（例如 `C:\path\to\dist`）中查找它们，而不是 `sys._MEIPASS`。

#### 选项 1：手动文件放置
- 构建后，将 `target.csv` 和 `icon/gear.png` 复制到 `dist`：
```
dist/
═── pydirector.exe
═── target.csv
└── icon/
└── gear.png
```
- 运行 `pydirector.exe`，也会在那里创建 `actions.db`。

#### 选项 2：使用 `--onedir` 模式
- 命令：
```
pyinstaller --onedir --windowed --add-data "icon/gear.png;icon" --add-data "target.csv;." --hidden-import=pyautogui --hidden-import=PySide6.QtWidgets --hidden-import=PySide6.QtCore --hidden-import=PySide6.QtGui --icon=icon/gear.png .\pydirector.py
```
- 结果：`dist/pydirector/` 自动包含所有文件：
```
dist/pydirector/
═── pydirector.exe
═── target.csv
═── icon/
│ └── gear.png
└── [其他 PyInstaller 依赖项]
```
- `actions.db` 将在同一文件夹中创建。

#### 选项 3：混合方法
- 保留 `--onefile` 并修改 `resource_path()`，以便对只读文件（`target.csv`、`gear.png`）使用 `sys._MEIPASS`，但对 `actions.db` 使用 `BASE_PATH`：
```python
def resource_path(relative_path):
if hasattr(sys, '_MEIPASS'):
return os.path.join(sys._MEIPASS,relative_path) # 对于捆绑文件
return os.path.join(BASE_PATH,relative_path)

DB_PATH = os.path.join(BASE_PATH,"actions.db") # 始终与可执行文件一起
```
- 这会将 `target.csv` 和 `gear.png` 捆绑在一起，但将 `actions.db` 放在 `dist` 中。

---

### 建议
- **为简单起见**：使用 `--onedir` 和 `get_base_path()`。所有内容都会自动放置在 `dist/pydirector/` 中，并且可移植。
- **对于单个文件**：使用 `--onefile` 和手动放置（选项 1）或混合方法（选项 3）。

## 5. 代码如何工作???

### 代码如何工作

“PyDirector”是一个基于 GUI 的自动化工具，使用 PySide6（Qt for Python）、PyAutoGUI、SQLite 和其他库构建。它允许用户定义、管理和执行存储在数据库中的鼠标/键盘自动化操作，可选择循环遍历 CSV 文件中的目标 ID 列表。以下是详细分类：

#### 1. **导入和设置**
- **库**：
- `sys`、`os`：处理文件路径和可执行行为（例如，PyInstaller 支持）。
- `sqlite3`：管理用于存储操作的数据库。
- `datetime`、`time`：处理时间戳和延迟。
- `csv`：从 CSV 文件中读取目标 ID。
- `logging`：将事件记录到 `pydirector.log` 以供调试。
- `PySide6`：构建 GUI（QMainWindow、QTableWidget 等）。
- `pyautogui`：执行自动化任务（鼠标移动、点击、打字等）。
- **日志记录**：配置为将调试/信息/错误消息写入 `pydirector.log`。
- **路径**：
- `get_base_path()` 和 `resource_path()` 确保脚本无论是作为 Python 文件还是 PyInstaller 可执行文件运行都能正常工作。
- `DB_PATH` 指向基础目录中的 `actions.db`。

#### 2. **数据库管理**
- **架构**：`actions.db` 中的 `actions` 表包含以下列：`id`、`action_name`、`action_type`、`parameters`、`timestamp`。
-**函数**：
-`create_database()`：如果表不存在，则创建表。
-`add_action()`、`update_action()`、`delete_action()`：操作的 CRUD 操作。
-`get_actions()`：检索所有操作以进行显示或执行。


#### 3.**CSV 处理**
-`read_target_ids()`：读取单列 CSV 文件（`target.csv`）以获取目标 ID 列表。这些 ID 可用于操作（例如，键入它们）。

#### 4. **自动化逻辑**
- **核心函数**：`execute_action(action, target_id=None)`：
- 从数据库获取一个动作元组（`id`、`name`、`type`、`parameters`、`timestamp`）和一个可选的`target_id`。
- 支持以下`action_type`：
- `move`：将鼠标移动到`(x, y)`，`duration=0.5`。
- `click`：单击（`left` 或 `right`）。
- `double_click`：双击（`left` 或 `right`）。
- `right_click`：在当前位置单击。
- `drag`：将鼠标拖动到`(x, y)`，`duration=0.5`。
- `hotkey`：按下组合键（例如，`ctrl,c`）。
- `type`：输入文本，可选择用 CSV 值替换 `{target_id}`。
- `wait`：暂停指定的秒数。
- 如果操作失败，则记录并打印错误。
- **运行函数**：`run_automation(loop_count=0, stop_flag=None)`：
- 加载目标 ID 和操作。
- 循环遍历所有目标 ID（`loop_count=0`）或指定次数。
- 对于每个循环：
- 选择一个 `target_id`（来自 CSV 或循环索引）。
- 按顺序执行所有操作，传递 `target_id`。
- 在操作之间添加 `time.sleep(1.0)`。
- 如果 `stop_flag()` 返回 `True`（例如，按下 Esc），则停止。

#### 5. **GUI（主窗口）**
- **组件**：
- **鼠标位置标签**：通过 `QTimer` 每 100 毫秒更新一次。
- **表格**：显示数据库中的操作。
- **按钮**：添加、编辑、删除操作；启动自动化。
- **循环输入**：QSpinBox 用于设置循环次数（0 = 使用所有目标 ID）。
- **系统托盘**：最小化到托盘，显示/退出选项。
- **快捷方式**：Esc 键停止自动化。
- **行为**：
- 关闭时最小化到托盘；双击托盘图标恢复。
- 通过 `ActionDialog` 弹出窗口管理操作。
- 自动化隐藏窗口、运行并在完成/中断时显示托盘通知。

#### 6. **执行流程**
1. 用户通过 GUI 添加操作（例如，“移动到 100,200”、“输入 Hello”）。
2. 用户设置循环计数并单击“启动自动化”。
3. 脚本读取 `target.csv`，为每个目标 ID 执行操作并记录进度。
4. 用户可以按 Esc 停止；托盘通知报告结果。

---

### 控制操作之间的间隔

#### 当前时间控制
- **操作之间**：在 `run_automation()` 中，每次 `execute_action()` 调用后都有一个硬编码的 `time.sleep(1.0)`。这意味着无论操作类型如何，一个操作的结束与下一个操作的开始之间都有 **1 秒的延迟**。
- **在操作中**：对于 `move` 和 `drag` 操作，`pyautogui.moveTo()` 和 `pyautogui.dragTo()` 使用 `duration=0.5`，它控制鼠标到达目的地所需的时间（0.5 秒）。其他操作（例如 `click`、`type`）立即执行或以 PyAutoGUI 的默认速度执行。

#### `duration` 是一种控制间隔的方法吗？
- **不直接用于操作之间的间隔**：`moveTo` 和 `dragTo` 中的 `duration` 参数控制**移动本身的持续时间**，而不是操作之间的暂停。例如：
- `pyautogui.moveTo(100, 200, duration=0.5)` 需要 0.5 秒来移动鼠标，然后脚本在下一个操作之前立即继续执行 `time.sleep(1.0)`。
- 更改 `duration` 会影响移动看起来的“平滑”或“缓慢”，而不是动作之间的间隙。
- **适用于特定动作**：它只是针对 `move` 和 `drag` 的计时控制，不适用于 `click`、`type` 等不使用 `duration` 的动作。

#### 如何控制操作之间的间隔
要调整操作之间的**时间**（一个操作完成后，下一个操作开始之前的暂停），您有以下选择：

1. **修改 `run_automation` 中的 `time.sleep(1.0)`**：
- 当前代码：
```python
for action in action:
if stop_flag and stop_flag():
return False
logging.debug(f"Executing action: {action}")
execute_action(action, target_id)
time.sleep(1.0) # 固定 1 秒延迟
```
- 将 `1.0` 更改为您想要的延迟（例如，`0.5` 表示 0.5 秒，`2.0` 表示 2 秒）：
```python
time.sleep(0.5) # 现在操作之间间隔 0.5 秒
```

2. **使延迟可配置**：
- 添加 UI 元素（例如 QSpinBox）供用户设置延迟：
```python
# 在 MainWindow.__init__ 中
self.delay_label = QLabel("操作之间的延迟（秒）：")
self.delay_input = QSpinBox()
self.delay_input.setMinimum(0)
self.delay_input.setMaximum(10) # 最大 10 秒
self.delay_input.setValue(1) # 默认 1 秒
self.layout.addWidget(self.delay_label)
self.layout.addWidget(self.delay_input)
```

- 更新 `run_automation` 以使用此值：
```python
def start_automation(self):
loop_count = self.loop_input.value()
delay = self.delay_input.value() # 从 UI 获取延迟
self.is_running = True
self.stop_requested = False
self.hide()
logs.debug(f"以 {loop_count} 个循环和 {delay} 秒延迟启动自动化")
print("自动化已启动。按 Esc 停止。")
complete = run_automation(loop_count, delay, stop_flag=lambda: self.stop_requested)
self.is_running = False
# ... 方法的其余部分 ...

def run_automation(loop_count=0, delay=1.0, stop_flag=None):
# ... 现有代码 ...
for action in action:
if stop_flag and stop_flag():
return False
logging.debug(f"执行操作：{action}")
execute_action(action, target_id)
time.sleep(delay) # 使用可配置延迟
# ... 其余函数 ...
```

3. **将延迟存储在数据库中**:
- 向 `actions` 表添加 `delay_after` 列:
```python
def create_database(db_name='actions.db'):
conn = sqlite3.connect(DB_PATH)
cursor = conn.cursor()
cursor.execute('''
CREATE TABLE IF NOT EXISTS action (
id INTEGER PRIMARY KEY AUTOINCREMENT,
action_name TEXT,
action_type TEXT,
parameters TEXT,
delay_after REAL DEFAULT 1.0, -- 新列
timestamp TEXT
)
''')
conn.commit()
conn.close()
```
- 更新 `add_action`、`update_action` 和 `ActionDialog` 以包含 `delay_after`。
- 修改 `run_automation` 以使用每个操作的延迟：
```python
for action in action:
if stop_flag and stop_flag():
return False
logs.debug(f"Executing action: {action}")
execute_action(action, target_id)
time.sleep(action[4]) # 使用数据库中的 delay_after
```

#### 操作之间的总时间
一个操作开始和下一个操作开始之间的有效间隔是：
- **操作执行时间**（例如，`move` 为 0.5 秒，`duration=0.5`，`click` 几乎是即时的）+ **延迟**（当前为 1.0 秒）。
- 示例：
- `moveTo(100, 200, duration=0.5)`：0.5s 移动 + 1.0s 睡眠 = **1.5s 总计**。
- `click()`：~0s 执行 + 1.0s 睡眠 = **1.0s 总计**。

#### 建议
- **最佳方法**：修改 `time.sleep(1.0)` 或使其可通过 UI 进行配置（选项 2）。这可为您提供灵活性，而不会使数据库过于复杂。
- **Duration 的作用**：使用 `duration` 来微调 `move` 和 `drag` 速度，但依赖 `time.sleep()` 来确定所有操作之间的间隔。

---

### 示例修改
以下是您可以如何实现可配置的延迟：
```python
# 在 MainWindow.__init__ 中，loop_input 之后：
self.delay_label = QLabel("操作之间的延迟（秒）：")
self.delay_input = QSpinBox()
self.delay_input.setMinimum(0)
self.delay_input.setMaximum(10)
self.delay_input.setValue(1)
self.layout.addWidget(self.delay_label)
self.layout.addWidget(self.delay_input)

# 更新 start_automation：
def start_automation(self):
loop_count = self.loop_input.value()
delay = self.delay_input.value()
self.is_running = True
self.stop_requested = False
self.hide()
logs.debug(f"以 {loop_count} 循环和 {delay} 秒延迟启动自动化")
print("自动化已启动。按 Esc 键停止。")
complete = run_automation(loop_count, delay, stop_flag=lambda: self.stop_requested)
self.is_running = False
# ... 其余方法 ...

# 更新 run_automation 签名：
def run_automation(loop_count=0, delay=1.0, stop_flag=None):
# ... 现有代码 ...
for action in action:
if stop_flag and stop_flag():
return False
logs.debug(f"执行操作：{action}")
execute_action(action, target_id)
time.sleep(delay) # 可配置延迟
# ... 其余函数 ...
```

现在，用户可以在 GUI 中设置延迟（0-10 秒），并且它在操作之间统一应用。

---

### 最后的想法
要控制间隔，调整 `time.sleep()` 是最直接、最有效的方法。

## 7. `Todo`

- 在运行过程中添加 `interruption`（CTRL-C 似乎不起作用）