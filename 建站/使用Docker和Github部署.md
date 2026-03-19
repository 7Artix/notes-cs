# Github 配置

## Github Token

GitHub -> Settings -> Developer settings -> Personal access tokens -> Tokens (classic)

Generate new token (classic)

名称可选 `GHCR_TOKEN` .

权限选择: `write:packages` , `delete:packages` .

保存Token(只会显示一次).

## 仓库配置

代码仓库 -> Settings -> Secrets and variables -> Actions

New repository secret

Name: `CR_PAT` , Secret: `刚刚的Token`

# Docker Compose

```yaml
services:
  backend:
    image: ghcr.io/你的用户名/7artix-backend:latest
    container_name: 7artix-backend
    restart: always
    volumes:
      - ./data:/app/data
    environment:
      - PORT=3000
    # 加上这个标签，明确告诉 watchtower 这个容器需要更新
    labels:
      - "com.centurylinklabs.watchtower.enable=true"

  frontend:
    image: ghcr.io/你的用户名/7artix-frontend:latest
    container_name: 7artix-frontend
    restart: always
    ports:
      - "5555:80"
    depends_on:
      - backend
    labels:
      - "com.centurylinklabs.watchtower.enable=true"

  # --- 新增的部分 ---
  watchtower:
    image: containrrr/watchtower
    container_name: watchtower
    restart: always
    volumes:
      # 必须挂载 docker.sock 才能控制其他容器
      - /var/run/docker.sock:/var/run/docker.sock
      # 挂载这个文件是为了让 watchtower 读取你刚才 docker login 的凭证，才有权限下载私有镜像
      - /root/.docker/config.json:/config.json
    command: --interval 60 --cleanup --label-enable
    # 解释:
    # --interval 60: 每 60 秒检查一次更新
    # --cleanup: 更新后删除旧的镜像，节省空间
    # --label-enable: 只更新那些带有 "watchtower.enable=true" 标签的容器（防止它乱更新别的）
```
