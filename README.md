基于当前工作区文件结构和功能设计：
![alt text](image.png)

# Docker 镜像自动同步工具

基于 GitHub Actions 的 Docker 镜像自动同步工具，支持将指定镜像从源仓库同步
到目标仓库（如 Docker Hub、阿里云容器镜像服务、GHCR 等），支持多仓库认证、失
败通知及手动触发功能。

---

## 功能特性
- **自动触发**：推送代码到 `main` 分支或手动触发（通过 GitHub Actions 界
面）
- **多仓库支持**：支持 Docker Hub、NVCR、GHCR、Harbor 等主流容器仓库
- **安全认证**：通过 GitHub Secrets 管理仓库密码，避免明文暴露
- **状态通知**：同步成功/失败时通过钉钉机器人推送通知（含触发分支、提交 SHA、
日志链接）
- **错误处理**：自动检查配置文件和执行文件是否存在，避免无意义失败

---

## 快速开始

### 1. 依赖准备
- 一个 GitHub 仓库（用于存放代码和运行 Actions）
- 需要同步的镜像源仓库和目标仓库的账号（如 Docker Hub、阿里云等）
- 钉钉机器人（用于接收通知，需提前创建并获取 `access_token`）

### 2. 配置文件说明

#### `configs/auth.json`（仓库认证信息）
定义各目标仓库的认证信息，`password` 字段为占位符（会被 GitHub Secrets 替
换）：
```json
{
  "registry.cn-hangzhou.aliyuncs.com": {
    "username": "Liufugui1",
    "password": "DOCKERHUB_PASSWORD"  // 对应 Secrets: 
    DOCKERHUB_PASSWORD
  },
  "docker.io": {
    "username": "2467802439@qq.com",
    "password": "DOCKERIO_PASSWORD"    // 对应 Secrets: 
    DOCKERIO_PASSWORD
  },
  "nvcr.io": {
    "username": "$oauthtoken",
    "password": "NVCRIO_PASSWORD"      // 对应 Secrets: 
    NVCRIO_PASSWORD
  },
  "ghcr.io": {  
    "username": "liu-fu-gui",
    "password": "GHCRIO_PASSWORD"      // 对应 Secrets: 
    GHCRIO_PASSWORD
  },
  "harbor.xiaoliu.xn--6qq986b3xl": {  
    "username": "admin",
    "password": "HARBOR_PASSWORD",     // 对应 Secrets: 
    HARBOR_PASSWORD
    "insecure": true                   // 非 HTTPS 仓库需设置为 true
  }
}
``` configs/images.json （镜像映射关系）
定义需要同步的镜像源与目标路径（键为源镜像，值为目标镜像）：

{
  "docker.io/nacos/nacos-server:latest": "registry.cn-hangzhou.
  aliyuncs.com/imagessync/nacos-server:latest"
}
```
### 3. 配置 GitHub Secrets
在仓库的 Settings → Secrets → Actions 中添加以下变量（与 auth.json 占位符一一对应）：

- DOCKERHUB_PASSWORD ：阿里云容器镜像服务密码
- DOCKERIO_PASSWORD ：Docker Hub 密码
- NVCRIO_PASSWORD ：NVCR 密码（或 Token）
- GHCRIO_PASSWORD ：GHCR 密码（或 PAT）
- HARBOR_PASSWORD ：Harbor 密码
- DINGTALK_TOKEN ：钉钉机器人 access_token （用于接收通知）
### 4. 运行同步任务
- 自动触发 ：推送代码到 main 分支时，工作流自动执行
- 手动触发 ：在 GitHub 仓库的 Actions 页面 → 选择 image-sync 工作流 → 点击 Run workflow
## 工作流说明
工作流文件 .github/workflows/docker-image-sync.yaml 包含以下核心步骤：

1. Checkout ：检出仓库代码
2. 生成配置文件 ：用 GitHub Secrets 替换 auth.json 中的占位符
3. 检查目录结构 （可选）：安装 tree 并打印目录结构（调试用）
4. 执行同步 ：调用 image-syncer 工具同步镜像（自动检查执行文件是否存在）
5. 钉钉通知 ：根据任务状态（成功/失败）推送不同内容的通知
## 注意事项
- 确保 image-syncer155/image-syncer 执行文件存在且有可执行权限（工作流会自动添加权限）
- 若无需查看目录结构，可删除 检查架构 步骤（避免安装 tree 的额外耗时）
- 钉钉通知链接会附带触发分支、提交 SHA 和日志地址，方便快速定位问题
## 贡献与反馈
如有功能建议或 BUG 反馈，欢迎提交 Issues 或 Pull Requests！

### 说明：
- 内容覆盖了项目核心功能、配置细节、操作步骤和注意事项，用户或贡献者可直接根据
文档完成部署。
- 重点标注了 GitHub Secrets 的配置要求（与 `auth.json` 强关联），避免因 
Secrets 缺失导致同步失败。
- 结合当前 `auth.json` 和 `images.json` 的实际内容（如 `nacos-server` 
镜像示例），使文档更贴合项目实际。