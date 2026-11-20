---
{"dg-publish":true,"permalink":"/Work/Tools/Jetbrains/PhpStorm 集成 SonarScanner/","title":"PhpStorm 集成 SonarScanner","tags":["flashcards"],"noteIcon":"","created":"2026-03-25T12:27:52.861+08:00","updated":"2026-03-26T10:59:47.180+08:00","dg-note-properties":{"title":"PhpStorm 集成 SonarScanner","tags":["flashcards"],"reference linking":null}}
---

在 macOS 环境下，如果不进行单元测试（Unit Test），操作会简单很多。你只需要关注 **静态代码扫描**（Static Analysis），它会检查代码中的 Bug、漏洞、异味（Code Smells）以及安全隐患，但不会有“代码覆盖率”这一项指标。
## 1. 安装 SonarScanner CLI
在 macOS 上，推荐使用 **Homebrew** 进行安装，这是最快捷的方式。
1.  打开终端（Terminal）。
2.  执行安装命令：
```shell
brew install sonar-scanner
```
1.  验证安装是否成功：
```shell
sonar-scanner -v
```
## 2. 获取 SonarQube 服务器 Token
为了安全地将扫描结果同步到服务器，你需要一个访问令牌：
1.  登录你的 SonarQube 服务器。
2.  点击右上角个人头像 -> **My Account** -> **Security**。
3.  在 **Generate Tokens** 中输入一个名字（如 `mac-hyperf-scan`），点击 **Generate**。
4.  **复制并保存这个 Token**，稍后会用到。
## 3. 在 Hyperf 项目中创建配置文件
在你的 Hyperf 项目根目录下（与 `app/` 文件夹同级），创建一个名为 `sonar-project.properties` 的文件。
```properties
# =====================================================
# 1. 项目基础信息
# =====================================================
sonar.host.url=http://13.214.186.243:9000
sonar.token=sqp_8f0721c1d8d11120d5d4927cb86b1dbd69dd5e70
sonar.projectKey=playerduo-test
sonar.projectName=playerduo-test
sonar.projectVersion=1.0.0

# =====================================================
# 2. 扫描范围与路径控制
# =====================================================
sonar.sources=app,config,bin
sonar.tests=test
sonar.test.inclusions=test/**/*.php
sonar.sourceEncoding=UTF-8

# 排除项精细化：跳过第三方库、缓存和日志
# 特别注意：runtime/** 必须排除，因为 Hyperf 的代理类会干扰逻辑分析
sonar.exclusions=vendor/**,runtime/**,storage/**,public/**,test/**,*.log,*.xml,*.json,database/migrations/**
sonar.cpd.exclusions=database/migrations/**

# =====================================================
# 3. PHP 深度分析与外部报告集成
# =====================================================
# 集成 PHPStan 报告，解决 IDE 报错但在 Sonar 没显示的问题
sonar.php.phpstan.reportPaths=phpstan-report.json

# 如果后续有单元测试，取消下方注释并指向 clover.xml
# sonar.php.tests.reportPath=report/clover.xml
# sonar.php.coverage.reportPaths=report/clover.xml

# 暂时屏蔽覆盖率统计，避免显示 0% 导致质量门禁不通过
sonar.coverage.exclusions=**/*

# =====================================================
# 4. 性能与限速优化
# =====================================================
# 增加扫描时的超时限制
sonar.scanner.socketTimeout=300
# 建议给扫描器分配 4-6 个线程以加速分析
sonar.analyzer.workerCount=6
```
## 4. 执行全量扫描
### 方案1「推荐」：Makefile
在项目根目录创建一个 `Makefile`。它能完美处理复杂的命令链、忽略特定错误码。
```makefile
.PHONY: analyze

# 定义变量
PHP_BIN = php -d xdebug.mode=off
PHPSTAN = ./vendor/bin/phpstan
SONAR = sonar-scanner
MEMORY_LIMIT = 2G

analyze:
	@echo "开始 PHPStan 逻辑检查..."
	# 使用 - 前缀忽略错误退出，确保即使 PHPStan 发现 Bug，也能执行下一步的 Sonar 扫描
	-$(PHP_BIN) $(PHPSTAN) analyse app --memory-limit=$(MEMORY_LIMIT) --error-format=json > phpstan-report.json
	@echo "PHPStan 扫描完成，报告已生成。"
	@echo "开始上传至 SonarQube 控制台..."
	$(SONAR)
	@echo "分析任务全部完成！请登录看板查看结果。"
```
**使用方法：** 以后在终端只需输入 `make analyze`。
### 方案2：Composer Script 适配
如果你更倾向于使用 `composer.json`，可以利用 `|| true` 来强制跳过错误码中断：
```json
"scripts": {
    "analyze": [
        "php -d xdebug.mode=off ./vendor/bin/phpstan analyse app --memory-limit=2G --error-format=json > phpstan-report.json || true",
        "sonar-scanner"
    ]
}
```
## 5. 在服务器查看结果
1.  命令执行完毕后，终端会显示 `EXECUTION SUCCESS`。
2.  打开 SonarQube 网页端，刷新项目列表。
3.  你会看到新生成的项目报告。虽然**没有单元测试覆盖率数据**，但你会看到：
	* **Bugs**: 潜在的逻辑错误。
	* **Vulnerabilities**: 安全漏洞（如 SQL 注入风险）。
	* **Code Smells**: 代码规范问题（如过长的类、未使用的变量）。
	* **Duplications**: 重复代码块。