# Homebrew 自动备份

## 备份内容

- **Brewfile**: 完整的依赖清单（推荐恢复方式）
- **brew-formulae.txt**: 命令行工具列表
- **brew-casks.txt**: GUI 应用列表
- **brew-backup.json**: 详细安装信息（含版本、依赖）

## 定时任务

已配置 launchd 定时任务，**每周三下午 3:30** 自动更新备份并提交到 Git。

### 管理命令

```bash
# 查看任务状态
launchctl list | grep com.homebrew.backup

# 停止定时任务
launchctl unload ~/Library/LaunchAgents/com.homebrew.backup.plist

# 重新加载定时任务
launchctl load ~/Library/LaunchAgents/com.homebrew.backup.plist

# 手动执行备份
/Users/ggg/private/homebrew/update-backup.sh
```

### 日志文件

- **backup.log**: 备份脚本执行日志
- **launchd.log**: launchd 标准输出
- **launchd-error.log**: launchd 错误日志

## 恢复方法

### 📥 第一步：获取备份文件

#### 从 GitHub 克隆备份仓库

```bash
# 克隆备份仓库到本地
git clone https://github.com/17359898647/homebrew.git ~/homebrew-backup
cd ~/homebrew-backup
```

#### 或从已有备份目录

```bash
cd /Users/ggg/private/homebrew
```

### 🔄 第二步：恢复 Homebrew 软件

#### 方式 1：使用 Brewfile 一键恢复（推荐✅）

**最快捷的恢复方式，自动安装所有软件和依赖：**

```bash
# 进入备份目录
cd ~/homebrew-backup  # 或 /Users/ggg/private/homebrew

# 一键恢复所有软件
brew bundle install --file=./Brewfile

# 如果遇到已安装的软件，跳过错误继续安装
brew bundle install --file=./Brewfile --no-upgrade
```

**Brewfile 优势：**
- ✅ 自动处理依赖关系
- ✅ 同时安装命令行工具和 GUI 应用
- ✅ 支持 Homebrew Taps（第三方仓库）
- ✅ 幂等操作，可重复执行

#### 方式 2：逐步恢复（精细控制）

**2.1 仅恢复命令行工具（Formulae）**

```bash
# 查看将要安装的工具列表
cat brew-formulae.txt

# 批量安装命令行工具
cat brew-formulae.txt | xargs brew install

# 或逐个安装（更安全）
while read formula; do
    echo "正在安装: $formula"
    brew install "$formula"
done < brew-formulae.txt
```

**2.2 仅恢复 GUI 应用（Casks）**

```bash
# 查看将要安装的应用列表
cat brew-casks.txt

# 批量安装 GUI 应用
cat brew-casks.txt | xargs brew install --cask

# 或逐个安装
while read cask; do
    echo "正在安装: $cask"
    brew install --cask "$cask"
done < brew-casks.txt
```

**2.3 选择性恢复特定软件**

```bash
# 从列表中选择需要的软件安装
grep -i "docker" brew-formulae.txt | xargs brew install
grep -i "vscode" brew-casks.txt | xargs brew install --cask
```

#### 方式 3：查看详细信息后再决定

```bash
# 查看 JSON 格式的完整备份信息（包含版本号）
cat brew-backup.json | jq '.formulae[] | {name: .name, version: .installed[0].version}'
```

### ✅ 第三步：验证恢复结果

```bash
# 检查已安装的命令行工具
brew list --formula

# 检查已安装的 GUI 应用
brew list --cask

# 验证特定软件是否安装成功
brew list | grep -i "软件名"

# 查看 Homebrew 整体状态
brew doctor
```

### 🔧 常见问题处理

#### 问题1：某些软件安装失败

```bash
# 查看失败原因
brew install --verbose 软件名

# 更新 Homebrew 后重试
brew update && brew install 软件名
```

#### 问题2：跳过已安装的软件

```bash
# Brewfile 方式会自动跳过
brew bundle install --file=./Brewfile --no-upgrade

# 手动方式需要检查
brew list | grep "软件名" || brew install 软件名
```

#### 问题3：清理旧版本和缓存

```bash
# 清理旧版本
brew cleanup

# 清理下载缓存
brew cleanup --prune=all
```

### 📊 对比不同恢复方式

| 方式 | 速度 | 精确度 | 依赖处理 | 适用场景 |
|------|------|--------|----------|----------|
| **Brewfile** | ⭐⭐⭐ | ⭐⭐⭐ | ✅ 自动 | 全新系统恢复 |
| **列表批量安装** | ⭐⭐ | ⭐⭐ | ⚠️ 手动 | 快速恢复主要软件 |
| **逐个安装** | ⭐ | ⭐⭐⭐ | ⚠️ 手动 | 选择性恢复 |
| **JSON 查看后安装** | ⭐ | ⭐⭐⭐ | ⚠️ 手动 | 需要版本控制 |

## 修改备份时间

编辑 `~/Library/LaunchAgents/com.homebrew.backup.plist`，修改 `StartCalendarInterval` 部分：

```xml
<key>StartCalendarInterval</key>
<dict>
    <key>Weekday</key>
    <integer>3</integer>  <!-- 星期几 (0=周日, 1=周一, ..., 6=周六) 不设置则每天 -->
    <key>Hour</key>
    <integer>15</integer>  <!-- 修改小时 (0-23) -->
    <key>Minute</key>
    <integer>30</integer>  <!-- 修改分钟 (0-59) -->
</dict>
```

修改后重新加载：
```bash
launchctl unload ~/Library/LaunchAgents/com.homebrew.backup.plist
launchctl load ~/Library/LaunchAgents/com.homebrew.backup.plist
```

---

**作者**: yangyang.huang  
**邮箱**: yangyang@weimill.com  
**创建时间**: 2025-10-20
