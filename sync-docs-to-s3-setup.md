# 将 docs 自动同步到 AWS S3 的配置说明

推送代码时（包括本地 `git push` 和在 GitHub 网页上的 commit/push），`docs/` 目录会自动同步到指定的 AWS S3 桶。

## 触发方式

- **自动**：向 `main` 分支 push 且本次提交涉及 `docs/**` 时触发
- **手动**：在 GitHub 仓库 → Actions → “Sync docs to AWS S3” → Run workflow

## 1. 在 AWS 侧准备

1. 创建一个 S3 桶（或使用已有桶）。
2. 创建用于 GitHub 的 IAM 用户，并为其添加策略（将 `your-bucket-name` 换成你的桶名）：

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:ListBucket",
        "s3:DeleteObject"
      ],
      "Resource": [
        "arn:aws:s3:::your-bucket-name",
        "arn:aws:s3:::your-bucket-name/*"
      ]
    }
  ]
}
```

3. 为该 IAM 用户创建 **访问密钥**（Access Key），记下 **Access Key ID** 和 **Secret Access Key**。

## 2. 在 GitHub 仓库中配置 Secrets

打开仓库：**Settings → Secrets and variables → Actions**，新建以下 Repository secrets：

| Secret 名称 | 必填 | 说明 |
|-------------|------|------|
| `AWS_ACCESS_KEY_ID` | 是 | 上面创建的 Access Key ID |
| `AWS_SECRET_ACCESS_KEY` | 是 | 上面创建的 Secret Access Key |
| `AWS_REGION` | 是 | 桶所在区域，如 `us-east-1`、`ap-northeast-1` |
| `AWS_S3_BUCKET` | 是 | S3 桶名称 |
| `AWS_S3_DOCS_PREFIX` | 否 | 对象前缀，默认为 `docs`。例如不填则同步到 `s3://桶名/docs/`；填 `''` 需在 value 里留空或单独处理 |

配置完成后，下次对 `main` 的 push（或手动运行 workflow）就会把 `docs/` 同步到 S3。

## 同步行为

- 使用 `aws s3 sync ./docs <目标> --delete`：本地 `docs/` 与 S3 目标保持一致，S3 上多出的文件会被删除。
- 若希望**每次 push 都同步**（不限制为仅 `docs/` 变更），可编辑 `.github/workflows/sync-docs-to-s3.yml`，删掉其中的 `paths: - 'docs/**'` 配置。

## 若使用阿里云 OSS 而非 AWS S3

本 workflow 针对 AWS S3 编写。若对象存储是阿里云 OSS，需要改用 OSS 的 API/CLI 或第三方 Action（例如 `manyuanrong/setup-ossutil`），在仓库里另建一个 workflow 并配置 OSS 的 AccessKey ID/Secret。
