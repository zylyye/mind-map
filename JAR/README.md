# Jar

## JAR包的优点

### 安全：（可签名验证）

### 方便传输

### 压缩

### 包密封

### 附带额外描述性信息

### 跨平台(portability) 

### 作为核心库的扩展包

## 如何使用

### 基本操作

- 创建 jar cvf app.jar add-files
- 查看 jar tf app.jar
- 更新 jar uf app.jar add-files
- 运行 JAR 包 java -jar app.jar

### Manifest文件

- 默认生成的Manifest
- (修改)加入自定义设置
- 设置包执行入口
- 为需要引入的包指定类路径(Class-Path) 
- 添加包的描述信息(标题、版本、生产商等) 
- 包密封

### 签名与验证

- 作用

	- 签名用来表明包发布者身份
	- 验证用来验证是否是可信的发布者以及包的完整性(例如是否被篡改) 

- 机制

	- 包内文件摘要条目，存放在manifest中
	- manifest文件摘要，存放在. SF文件中
	- 私钥产生的签名、公钥，在META-INF目录下

### JAR相关api，用于手动加载JAR包
