# Java 客户端与工具库安装

## 安装客户端库

运行以下命令，确定 `com.google.api-ads:data-manager` 的最新版本：
```shell
curl -s "https://repo1.maven.org/maven2/com/google/api-ads/data-manager/maven-metadata.xml" \
  | sed -n 's/.*<release>\(.*\)<\/release>.*/\1/p'
```

如果使用 Maven，请将以下依赖添加到 `pom.xml` 文件。
将 `{latest_version}` 替换为上面获取的版本：

```xml
<dependency>
  <groupId>com.google.api-ads</groupId>
  <artifactId>data-manager</artifactId>
  <version>{latest_version}</version>
</dependency>
```

如果使用 Gradle，请将以下内容添加到依赖中。
将 `{latest_version}` 替换为上面获取的版本：

```groovy
implementation 'com.google.api-ads:data-manager:{latest_version}'
```

## 安装工具库

运行以下命令，确定 `com.google.api-ads:data-manager-util` 的最新版本：
```shell
curl -s "https://repo1.maven.org/maven2/com/google/api-ads/data-manager-util/maven-metadata.xml" \
  | sed -n 's/.*<release>\(.*\)<\/release>.*/\1/p'
```

如果使用 Maven，请将以下依赖添加到 `pom.xml` 文件。
将 `{latest_version}` 替换为上面获取的版本：

```xml
<dependency>
  <groupId>com.google.api-ads</groupId>
  <artifactId>data-manager-util</artifactId>
  <version>{latest_version}</version>
</dependency>
```

如果使用 Gradle，请将以下内容添加到依赖中。
将 `{latest_version}` 替换为上面获取的版本：

```groovy
implementation 'com.google.api-ads:data-manager-util:{latest_version}'
```