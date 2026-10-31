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

```shell
huawei signing-env --github
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
