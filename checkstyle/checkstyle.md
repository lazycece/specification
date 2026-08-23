# 风格配置

## 官方地址
https://github.com/checkstyle/checkstyle

## 自定义配置
./google_checks_indent4.xml

google原生的是缩进两个，这里保持过往风格，即缩进4格，修改内容如下：
- basicOffset: 2 -> 4
- braceAdjustment = basicOffset
- caseIndent = basicOffset
- arrayInitIndent = basicOffset
- throwsIndent, lineWrappingIndentation  = basicOffset × 2

```xml
    <module name="Indentation">
      <property name="basicOffset" value="4"/>
      <property name="braceAdjustment" value="4"/>
      <property name="caseIndent" value="4"/>
      <property name="throwsIndent" value="8"/>
      <property name="lineWrappingIndentation" value="8"/>
      <property name="arrayInitIndent" value="4"/>
    </module>
```


# maven集成
pom.xml 加插件
- 如果用原生的google_checks，configLocation 直接填 google_checks.xml（Maven 插件会从 Checkstyle 依赖里解析内置资源）
- 如果要挂本地修改版，把 configLocation 改成 具体路径：如config/checkstyle/google_checks.xml，checkstyle文件jar版本和依赖版本要对应

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-checkstyle-plugin</artifactId>
  <version>3.6.0</version>
                  <!-- <dependencies>
                    <dependency>
                        <groupId>com.puppycrawl.tools</groupId>
                        <artifactId>checkstyle</artifactId>
                        <version>11.1.0</version>
                    </dependency>
                </dependencies> -->
  <configuration>
    <configLocation>google_checks.xml</configLocation>
    <encoding>UTF-8</encoding>
    <consoleOutput>true</consoleOutput>
    <failsOnError>true</failsOnError>
    <includeTestSourceDirectory>true</includeTestSourceDirectory>
  </configuration>
  <executions>
    <execution>
      <phase>validate</phase>
      <goals><goal>check</goal></goals>
    </execution>
  </executions>
</plugin>
```
本地校验：
```bash
mvn checkstyle:check      # 只检查，不编译
mvn validate              # 走生命周期，违规直接 fail
```


# IntelliJ IDEA 实时检查

- Settings → Plugins 搜 CheckStyle-IDEA​ 安装并重启
- Settings → Tools → Checkstyle
- Configuration File 点 +：
  - 选 Use a local file​ → 指到 config/checkstyle/google_checks.xml
  - 或选内置 Google Checks（插件自带）
- 勾选设为 Active Configuration
- 编辑器里违规会直接波浪线提示，也可右键 Checkstyle → Check Module
> 顺手建议：再把 Google 的 IntelliJ 格式化方案（intellij-java-google-style.xml）导入 Editor → Code Style，让“自动格式化”和“checkstyle 检查”不打架。
> 如果使用了自定的checkstyle，也要处理 codestyle

# CI / 门禁（GitHub Actions 示例）
在 validate 或 check 阶段挂一个步骤即可，PR 里格式不对直接红：
```yaml
- name: Set up JDK 21
  uses: actions/setup-java@v3
  with:
    java-version: '21'
    distribution: 'temurin'
    cache: maven

- name: Checkstyle (Google)
  run: mvn -B checkstyle:check
```