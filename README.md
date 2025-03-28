# JMeter + Ant + Mail 自动化测试框架

## 项目简介
这是一个基于 JMeter、Ant 和邮件通知的自动化测试框架，用于执行JMeter接口测试并自动发送测试报告。框架支持详细日志记录，方便排查测试过程中的问题。

## 项目环境
1. JDK 22
2. Apache JMeter 5.6.3
3. Apache Ant 1.10.15
4. 163邮件服务器（支持 SMTP）

## 项目结构
```
项目根目录/
├── build.xml         # Ant 构建脚本
├── lib/              # 依赖库目录，这些Jar包都要拷贝到ant安装目录下的lib目录中
│   ├── angus-activation-2.0.0.jar
│   ├── angus-mail-2.0.1.jar
│   ├── jakarta.mail-api-2.1.2.jar
│   └── jakarta.activation-api-2.1.2.jar
├── logs/             # 日志输出目录，存放测试执行过程的日志文件
├── report/           # 测试报告输出目录
├── rar/              # 测试报告压缩包目录
└── script/           # JMeter 测试脚本目录
    └── *.jmx         # JMeter 测试脚本文件
```

## Jakarta Mail 相关 JAR 包详细说明

### 1. angus-activation-2.0.0.jar
**功能**: Eclipse Angus Activation 是 Jakarta Activation 规范的实现。
**作用**: 提供 MIME 数据类型识别和处理功能，允许程序根据数据类型执行不同操作。
**重要性**: 是 Jakarta Mail 的基础组件，处理邮件附件和多媒体内容必不可少。
**依赖关系**: 实现了 jakarta.activation-api 中定义的接口。

### 2. angus-mail-2.0.1.jar
**功能**: Eclipse Angus Mail 是 Jakarta Mail 规范的实现库。
**作用**: 提供完整的电子邮件收发功能实现，支持 SMTP、POP3、IMAP 等协议。
**重要性**: 是项目中实际执行邮件发送操作的核心组件。
**依赖关系**: 依赖 jakarta.mail-api 和 angus-activation，实现了邮件规范中定义的所有接口。

### 3. jakarta.mail-api-2.1.2.jar
**功能**: Jakarta Mail API 定义了用于发送和接收电子邮件的标准接口。
**作用**: 提供邮件处理的抽象层，定义了 Session、Message、Transport 等关键接口。
**重要性**: 确保应用程序和具体的邮件实现之间解耦，提高代码可维护性。
**依赖关系**: 依赖 jakarta.activation-api，需要具体实现库（如 angus-mail）来提供功能。

### 4. jakarta.activation-api-2.1.2.jar
**功能**: Jakarta Activation API 定义了数据类型处理的标准接口。
**作用**: 提供 MIME 类型识别和数据处理的框架，支持邮件附件和多媒体内容处理。
**重要性**: 是 Jakarta Mail 的基础 API，处理不同类型邮件内容的必要组件。
**依赖关系**: 需要具体实现库（如 angus-activation）来提供功能。


## build.xml文件配置说明

### 1. JMeter 配置
在 `build.xml` 中配置 JMeter 安装路径：
```xml
<property name="jmeter.home" value="/Users/jane/main/apache-jmeter-5.6.3" />
```

### 2. 项目路径配置
在 `build.xml` 中配置项目根路径：
```xml
<property name="jmeter.root.dir" value="/Users/jane/Documents/Code/JMeter/ant/jmeter_ant" />
```

### 3. 邮件配置
在 `build.xml` 中配置邮件发送参数：
```xml
<mail
    mailhost="smtp.163.com"
    mailport="465"
    ssl="true"
    user="your-email@163.com"
    password="授权码"
    subject="JMeter接口测试报告 - ${time}"
    messagemimetype="text/html"
    charset="UTF-8"
    from="your-email@163.com"
    tolist="recipient@example.com">
```



### 5. 测试脚本配置
在 `build.xml` 中配置测试脚本参数，用于运行单个测试脚本：
```xml
<!-- 单个测试脚本文件名称，默认运行所有脚本 -->
<property name="test.script.file" value="*.jmx" />
```

## 使用方法

### 1. 准备测试脚本
将 JMeter 测试脚本（.jmx 文件）放在 `script` 目录下。

### 2. 执行测试
#### 2.1 执行所有测试脚本
在项目根目录下执行以下命令：
```bash
ant -f build.xml all
```
或
```bash
ant
```

#### 2.2 执行单个测试脚本
在项目根目录下执行以下命令：
```bash
ant -f build.xml run-single -Dtest.script.file=your-test-script.jmx
```

### 3. 查看结果
- 测试报告将生成在 `report` 目录下
- 测试报告压缩包将生成在 `rar` 目录下
- 测试报告将通过邮件发送给指定收件人
- 执行日志将保存在 `logs` 目录下，使用无特殊字符的时间戳格式命名

## 构建任务说明

### 1. init
初始化。

### 2. clear
清理 report 目录和 rar 目录下的所有文件，并重新创建这些目录。

### 3. test
执行所有 JMeter 测试脚本。

### 4. report
生成 HTML 格式的测试报告，包括汇总报告和详细报告。

### 5. sendmail
发送带有美化格式的测试报告邮件，并附加测试报告压缩包。

### 6. all
执行整个测试流程（init → clear → test → report → sendmail）。


## 邮件通知系统

### 邮件格式
系统发送的邮件包含以下内容：
1. **测试报告说明文本**：解释邮件的目的和联系信息
2. **详细测试报告**：嵌入HTML格式的测试报告
3. **附件**：完整测试报告的压缩包

### 邮件模板
邮件使用格式化的HTML模板，提高了可读性。模板包含以下部分：
- 说明文本区域（含联系信息）
- 详细报告区域（带有分隔线和标题）

## 注意事项
1. 确保 JMeter 安装路径配置正确
2. 确保邮件服务器配置正确，包括服务器地址、端口、用户名和密码
3. 如果使用 163 邮箱，需要使用授权码而不是登录密码
4. 确保网络连接正常，能够访问邮件服务器
5. 确保目录权限正确，允许创建日志文件和测试报告


