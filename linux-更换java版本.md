看到你安装了 `bellsoft-java25`，这是第三方提供的 Java 25。现在需要配置系统使用它。

## 🔧 配置 bellsoft-java25 为默认 Java

### **步骤1：查找安装位置**

```bash
# bellsoft Java 通常安装在 /usr/lib/jvm/
ls -la /usr/lib/jvm/ | grep -i bellsoft
# 或者
find /usr -name "*bellsoft*" -type d 2>/dev/null

# 常见的安装路径：
# /usr/lib/jvm/bellsoft-java25/
# /usr/lib/jvm/jdk-25/
# /opt/bellsoft/jdk-25/
```

### **步骤2：使用 alternatives 配置**

```bash
# 查看当前 alternatives 配置
sudo alternatives --config java
sudo alternatives --config javac

# 添加 bellsoft-java25 到 alternatives
sudo alternatives --install /usr/bin/java java /usr/lib/jvm/bellsoft-java25/bin/java 3000
sudo alternatives --install /usr/bin/javac javac /usr/lib/jvm/bellsoft-java25/bin/javac 3000

# 或者如果不知道具体路径，先查找
find /usr/lib/jvm -name "java" -type f | xargs ls -la
```

### **步骤3：验证安装**

```bash
# 检查 bellsoft Java 是否正常工作
/usr/lib/jvm/bellsoft-java25/bin/java -version
/usr/lib/jvm/bellsoft-java25/bin/javac -version

# 如果上面命令有效，设置为默认
sudo alternatives --set java /usr/lib/jvm/bellsoft-java25/bin/java
sudo alternatives --set javac /usr/lib/jvm/bellsoft-java25/bin/javac
```

## 📝 快速诊断脚本

运行这个脚本找出正确路径：

```bash
#!/bin/bash
echo "=== 查找 bellsoft-java25 ==="

# 方法1：检查常见路径
PATHS=(
    "/usr/lib/jvm/bellsoft-java25"
    "/usr/lib/jvm/jdk-25"
    "/opt/bellsoft/jdk-25"
    "/usr/lib/jvm/java-25-bellsoft"
)

for path in "${PATHS[@]}"; do
    if [ -d "$path" ]; then
        echo "找到: $path"
        echo "Java 版本:"
        $path/bin/java -version 2>&1 | head -3
        echo ""
    fi
done

# 方法2：搜索
echo "=== 搜索 Java 安装 ==="
find /usr/lib/jvm -type d -name "*25*" 2>/dev/null
find /opt -type d -name "*bellsoft*" 2>/dev/null 2>/dev/null

echo ""
echo "=== 当前 alternatives 配置 ==="
sudo alternatives --display java | grep -A5 "link currently points"
```

## 🎯 针对 Gradle 的配置

### **方案A：设置 JAVA_HOME**

```bash
# 找到 bellsoft-java25 的安装路径后
export JAVA_HOME=/usr/lib/jvm/bellsoft-java25  # 根据实际路径调整
echo "JAVA_HOME=$JAVA_HOME"

# 验证
$JAVA_HOME/bin/java -version
$JAVA_HOME/bin/javac -version

# 运行 Gradle
cd ~/Documents/github/HMCL
/opt/gradle-9.2.0/bin/gradle clean makeExecutables
```

### **方案B：创建 shell 别名（永久生效）**

```bash
# 编辑 ~/.bashrc 或 ~/.zshrc
nano ~/.bashrc

# 添加以下内容
export BELLSOFT_JAVA_HOME=/usr/lib/jvm/bellsoft-java25
export JAVA_HOME=$BELLSOFT_JAVA_HOME
export PATH=$JAVA_HOME/bin:$PATH

# 添加 Gradle 别名
alias gradle25="JAVA_HOME=$BELLSOFT_JAVA_HOME /opt/gradle-9.2.0/bin/gradle"

# 使配置生效
source ~/.bashrc

# 测试
gradle25 --version
```

### **方案C：项目级配置**

在 HMCL 项目目录中：

```bash
# 创建 gradle.properties 文件
echo "org.gradle.java.home=/usr/lib/jvm/bellsoft-java25" > gradle.properties

# 或者使用环境变量
cd ~/Documents/github/HMCL
JAVA_HOME=/usr/lib/jvm/bellsoft-java25 /opt/gradle-9.2.0/bin/gradle clean makeExecutables
```

## 🔄 如果仍然失败的处理方法

### **检查 bellsoft-java25 是否包含开发工具**

```bash
# 检查是否有 javac
ls -la /usr/lib/jvm/bellsoft-java25/bin/javac

# 如果没有，可能需要安装开发包
# bellsoft 通常有单独的 jdk 包
dnf list installed | grep bellsoft
```

### **使用 dnf 重新配置**

```bash
# 查看 bellsoft 相关包
dnf list installed "*bellsoft*"

# 如果只安装了 JRE，需要 JDK
# 卸载重装
sudo dnf remove bellsoft-java25
# 安装完整的 JDK（如果有的话）
sudo dnf install bellsoft-java25-full  # 名称可能不同
```

## ✅ 最终测试命令

```bash
# 1. 先找到 bellsoft-java25
BELLSOFT_PATH=$(find /usr/lib/jvm -type d -name "*bellsoft*" 2>/dev/null | head -1)
echo "Bellsoft 路径: $BELLSOFT_PATH"

# 2. 测试 Java
$BELLSOFT_PATH/bin/java -version

# 3. 测试 javac
$BELLSOFT_PATH/bin/javac -version

# 4. 设置环境变量
export JAVA_HOME=$BELLSOFT_PATH
echo "JAVA_HOME: $JAVA_HOME"

# 5. 测试 Gradle
cd ~/Documents/github/HMCL
/opt/gradle-9.2.0/bin/gradle --version

# 6. 尝试构建
/opt/gradle-9.2.0/bin/gradle clean makeExecutables
```

## 📌 重要提醒

如果 `bellsoft-java25` 仍然缺少 `javac` 或其他开发工具，你可能需要：

1. **安装 OpenJDK 17 作为备选**

   ```bash
   sudo dnf install java-17-openjdk-devel
   export JAVA_HOME=/usr/lib/jvm/java-17-openjdk
   ```

2. **使用 HMCL 项目的 wrapper**（它可能自己管理 Java 版本）

   ```bash
   cd ~/Documents/github/HMCL
   ./gradlew clean makeExecutables
   ```

**请先运行查找命令**，告诉我 bellsoft-java25 的实际安装路径，这样我可以提供更精确的配置命令。