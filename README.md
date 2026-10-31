## 安卓签名

### 生成本地签名证书

```shell
keytool -genkey -v -keystore .arkui-x/android/publish_apk_demo.keystore -alias publish_apk_demo -keyalg RSA -keysize 2048 -validity 36500
```

> keystore 和 Gradle 工程放在一起；`*.keystore*` 已被 `.gitignore` 忽略，不会入库

### 配置 Gradle 签名

```shell
cp .arkui-x/android/keystore.properties.template .arkui-x/android/keystore.properties
```

编辑 `.arkui-x/android/keystore.properties`，`storeFile` 相对 `.arkui-x/android` 目录解析：

```properties
storeFile=publish_apk_demo.keystore
keyAlias=publish_apk_demo
storePassword=<keystore 口令>
keyPassword=<私钥口令>
```

`app/build.gradle` 打 release 包时读取该文件；文件不存在时回退 Gradle 的 debug 签名并打印警告。
`keystore.properties` 同样已被忽略，口令不会入库。

> 也可以用 `publish-apk keystore-config` 从环境变量生成，CI 用的就是它（需要 publish-apk 0.0.1-alpha.20 及以上），见下一节

### 在 GitHub Actions 中使用

workflow 里的「生成安卓签名配置」步骤在 `.arkui-x/android` 下执行 `publish-apk keystore-config`：
把 `ANDROID_KEYSTORE_BASE64` 解码成 keystore（`publish_apk_demo.keystore`）并生成 `keystore.properties`，
然后才开始打 APK；secrets 没配置时这一步会直接失败。

## 使用 publish-apk 获取 CI/CD 所需环境变量

### 全局安装依赖

```shell
npm install -g publish-apk
```

### 获取环境变量

> 可以获取到 build-profile.json5 文件中的 `keyPassword`、`storePassword` 原始密码

```shell
publish-apk huawei signing-env
```

> 屏蔽关键内容

```shell
publish-apk huawei signing-env | \
  sed -E -e 's/^([^=]+=)"(.{0,2})(.*)(.)"$/\1"\2...\4"/' \
         -e 's/^([^=]+=)(.{0,2})(.*)(.)$/\1\2...\4/'
```

> 执行结果示例

```text
警告: stdout 包含明文口令，请勿写入日志、提交仓库或公开分享
HARMONY_STORE_FILE_BASE64="M..."
HARMONY_PROFILE_BASE64="M..."
HARMONY_CERTPATH_BASE64="L..."
HARMONY_STORE_PASSWORD="s..."
HARMONY_KEY_PASSWORD="s..."
HARMONY_KEY_ALIAS="d..."
HARMONY_SIGN_ALG="S..."
```

### 自动配置 GitHub Actions secrets

> 鸿蒙证书

```shell
publish-apk huawei signing-env --github
```

> 执行结果示例

```text
GitHub 仓库: xuxiaowei-io/publish_apk_demo
密钥范围: 仓库
已写入 7 个密钥:
- HARMONY_STORE_FILE_BASE64
- HARMONY_PROFILE_BASE64
- HARMONY_CERTPATH_BASE64
- HARMONY_STORE_PASSWORD
- HARMONY_KEY_PASSWORD
- HARMONY_KEY_ALIAS
- HARMONY_SIGN_ALG
```

> 安卓证书

```shell
publish-apk keystore-env --file .arkui-x/android/publish_apk_demo.keystore --github
```

> 执行结果示例

```text
请输入 keystore 口令（明文，不回显）：
GitHub 仓库: xuxiaowei-io/publish_apk_demo
密钥范围: 仓库
已写入 4 个密钥:
- ANDROID_KEYSTORE_BASE64
- ANDROID_KEYSTORE_PASSWORD
- ANDROID_KEY_ALIAS
- ANDROID_KEY_PASSWORD
```

### 使用环境变量，在 CI/CD 中配置 build-profile.json5

> 定义变量

```shell
export HARMONY_STORE_FILE_BASE64=""
export HARMONY_PROFILE_BASE64=""
export HARMONY_CERTPATH_BASE64=""
export HARMONY_STORE_PASSWORD=""
export HARMONY_KEY_PASSWORD=""
export HARMONY_KEY_ALIAS=""
export HARMONY_SIGN_ALG="SHA256withECDSA"
```

> 配置 build-profile.json5

```shell
# 使用 --product 指定产品
publish-apk huawei signing-material-env
```
