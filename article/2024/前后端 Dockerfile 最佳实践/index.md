## Dockerfile 构建基础
Dockerfile 是一个文本文件，其中包含了自动化构建 Docker 镜像的所有命令。<br />构建过程是在 Docker 守护进程（docker daemon）中进行的，而非命令行工具。<br />执行 docker build 命令时，命令行工具会将 Dockerfile 和其构建上下文（通常是包含 Dockerfile 的目录）发送给守护进程，以生成镜像。<br />![](https://cdn.nlark.com/yuque/0/2024/jpeg/21596389/1710578768922-6458a4e4-0da3-4152-8e32-3e274c8c4b90.jpeg)

## .dockerignore
.dockerignore 文件用于指定在构建过程中应该忽略的文件和目录，以减小镜像体积并提高构建速度。<br />其语法与 .gitignore 类似，支持通配符和排除规则：

- .eslintignore：忽略 .eslintignore 文件
- *.md：忽略所有以 .md 结尾的文件（除了 README.md）
- !README.md：不忽略 README.md 文件
- node_modules/：忽略 node_modules 目录下的所有文件
- [a-c].txt：忽略文件名以 a, b, c 开头并且扩展名为 .txt 的文件

## React 项目实践
新建项目：
```bash
npx craete-react-app dockerfile-test-app
```
根目录创建 .dockerignore 文件，排除不必要的文件：
```bash
# 忽略 node_modules 目录，因为它包含大量的依赖项，可以在构建镜像时通过 npm/yarn 安装  
node_modules  
  
# 忽略构建生成的目录，例如 build 或 dist，因为这些文件可以在构建镜像时重新生成  
build  
dist  
  
# 忽略开发环境相关的配置文件  
.env  
.env.development  
.env.local  
.env.*.local  
  
# 忽略编辑器生成的临时文件  
.DS_Store  
.idea  
.vscode  
  
# 忽略日志文件  
npm-debug.log*  
yarn-debug.log*  
yarn-error.log*  
  
# 忽略测试报告和覆盖率文件  
coverage  
  
# 忽略其他不需要的文件或目录  
.git  
.gitignore  
.dockerignore  
README.md  
LICENSE
```
在根目录创建 Dockerfile 文件，定义镜像构建过程：
```dockerfile
# 选择一个带有Node.js的轻量级基础镜像，使用 as 多阶段构建
FROM node:16-alpine as build-stage

# 设置工作目录
WORKDIR /app

# 复制package.json和package-lock.json（如果存在）
COPY package*.json ./

# 安装项目生产依赖
# 利用 Docker 缓存层，如果 package.json 没有变化，则不会重新安装node modules
RUN npm install --only=production

# 复制项目文件到工作目录
COPY . .

# 构建应用
RUN npm run build

# 阶段2：运行
# 使用 Nginx 镜像作为基础来提供前端静态文件服务
FROM nginx:stable-alpine as production-stage

# 定义环境变量，例如NODE_ENV
# ARG NODE_ENV=production
# ENV NODE_ENV=${NODE_ENV}

WORKDIR /app

# 从构建阶段拷贝构建出的文件到Nginx目录
COPY --from=build-stage /app/build /usr/share/nginx/html

# 配置nginx，如果有自定义的nginx配置可以取消注释并修改下面的行
# COPY nginx.conf /etc/nginx/conf.d/default.conf

# 暴露80端口
EXPOSE 80

# 启动Nginx服务器
CMD ["nginx", "-g", "daemon off;"]
```
启动 docker dekstop，打包镜像：
```bash
docker build -t dockerfile-test-app:v1.0 .
```
![image.png](https://cdn.nlark.com/yuque/0/2024/png/21596389/1710590501684-d8cbb46f-5ec3-49df-b754-4769631bbdb8.png#averageHue=%232c2c2c&clientId=u31789297-685c-4&from=paste&height=369&id=u37583f74&originHeight=738&originWidth=2100&originalType=binary&ratio=2&rotation=0&showTitle=false&size=230947&status=done&style=none&taskId=ucf8ce3ab-81a8-4e8d-be47-0d57a9e1289&title=&width=1050)<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/21596389/1710590516541-73700f4e-6e97-45bc-b172-20f0b75b961c.png#averageHue=%23f4f5f8&clientId=u31789297-685c-4&from=paste&height=54&id=u45790162&originHeight=108&originWidth=2160&originalType=binary&ratio=2&rotation=0&showTitle=false&size=22448&status=done&style=none&taskId=u656947cd-f053-4c1c-84ad-0a17370dcff&title=&width=1080)<br />运行镜像：
```bash
docker run -d -p 8088:80 dockerfile-test-app:v1.0
```
-d 表示在后台运行容器<br />-p 8088:80 表示将宿主机的 8088 端口映射到容器的 80 端口。<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/21596389/1710590564822-3ef0e90f-6bb4-419e-a1fc-897ad533694f.png#averageHue=%234b4b4b&clientId=u31789297-685c-4&from=paste&height=31&id=u0f8fd3ec&originHeight=62&originWidth=1528&originalType=binary&ratio=2&rotation=0&showTitle=false&size=19086&status=done&style=none&taskId=ucf9a6d57-6e28-40e8-afe4-8ed5991d61c&title=&width=764)<br />打开浏览器，访问 http://localhost:8088：<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/21596389/1710590615004-31e49e88-f9b0-4ce1-ab4d-4a800eed87fc.png#averageHue=%232d3c45&clientId=u31789297-685c-4&from=paste&height=280&id=u76420f3d&originHeight=706&originWidth=818&originalType=binary&ratio=2&rotation=0&showTitle=false&size=71544&status=done&style=none&taskId=u4ead03d3-fc8c-42c0-98fc-5ce1d876068&title=&width=324)<br />没问题。

## Nest 项目实践
新建项目：
```bash
nest new dockerfile-test-nest -p npm
```
根目录创建 .dockerignore：
```bash
*.md
node_modules/
.git/
.DS_Store
.vscode/
.dockerignore
```
根目录创建 Dockerfile：
```dockerfile
# Step 1: 使用带有 Node.js 的基础镜像
FROM node:18-alpine as builder

# 设置工作目录
WORKDIR /usr/src/app

# 复制 package.json 和 package-lock.json（如果可用）
COPY package*.json ./

# 安装项目依赖
RUN npm install --only=production

# 安装 nest CLI 工具（确保它作为项目依赖被安装）  
RUN npm install @nestjs/cli -g

# 复制所有文件到容器中
COPY . .

# 构建应用程序
RUN npm run build

# Step 2: 运行时使用更精简的基础镜像
FROM node:18-alpine

# 创建 runc 的符号链接
RUN ln -s /sbin/runc /usr/bin/runc

WORKDIR /usr/src/app

# 从 builder 阶段复制构建好的 /usr/src/app/dist 和 /usr/src/app/node_modules 的文件到当工作目录
COPY --from=builder /usr/src/app/dist ./dist
COPY --from=builder /usr/src/app/node_modules ./node_modules

# 暴露 3000 端口
EXPOSE 3000

# 运行 Nest.js 应用程序
CMD ["node", "dist/main"]
```
打包镜像：
```bash
docker build -t dockerfile-test-nest:v1.0 .
```
![image.png](https://cdn.nlark.com/yuque/0/2024/png/21596389/1709349718917-65490675-d269-4f8e-bbea-20d91dd365c5.png#averageHue=%23eceef1&clientId=u6d937af0-bedb-4&from=paste&height=50&id=u2ec1c8ed&originHeight=100&originWidth=2020&originalType=binary&ratio=2&rotation=0&showTitle=false&size=22131&status=done&style=none&taskId=u1f0d6848-780d-4f7c-b924-40960ab97ac&title=&width=1010)<br />运行镜像：
```dockerfile
docker run -d -p 3002:3000 dockerfile-test-nest:v1.0
```
-p 3002:3000: 将宿主机的 3002 端口映射到容器的 3000 端口<br />访问 [http://localhost:3002/](http://localhost:3002/)：<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/21596389/1709349895089-45cd33a0-90f5-41f6-9918-93d6fb047f53.png#averageHue=%23efefef&clientId=u6d937af0-bedb-4&from=paste&height=85&id=u9127a01e&originHeight=170&originWidth=510&originalType=binary&ratio=2&rotation=0&showTitle=false&size=13128&status=done&style=none&taskId=u24197179-1db9-46e1-a327-6273005a37a&title=&width=255)<br />代表 Nest 应用启动了，没问题。



## dockerfile 最佳优化实践
优化 Dockerfile 不仅可以减少镜像的大小，还可以加快构建速度，提高容器的运行效率。<br />以下是一些 Dockerfile 最佳优化实践：

### **使用官方、轻量级、精简镜像**
选择一个适合应用的官方轻量级基础镜像，例如 `alpine` 或 `debian-slim`结尾，可以显著减少最终镜像的大小。<br />避免使用 latest 标签，而是使用具体的版本号，保持一致性。

### 最小化镜像层数

- **合并命令行**：通过使用逻辑运算符 (&&) 合并命令行，减少镜像层的数量。
- **清理缓存**：在同一 `RUN` 指令中安装软件后立即清理缓存和不必要的文件。
```dockerfile
# 不推荐
RUN apt-get update
RUN apt-get install -y package1
RUN apt-get install -y package2

# 推荐
RUN apt-get update && \
    apt-get install -y package1 package2 && \
    rm -rf /var/lib/apt/lists/*
```

### 利用缓存
**优化指令顺序**：将不经常变化的指令放在 Dockerfile 的前面，这样可以利用 Docker 的层缓存，加速后续构建过程。
```dockerfile
# 先复制不经常变动的文件
COPY requirements.txt /app/

# 再复制经常变动的文件
COPY . /app
```

### 清理不必要的文件
在同一个 RUN 指令中安装软件包后，应该清理缓存和不必要的文件，以减少镜像大小。
```dockerfile
RUN apt-get update && \
    apt-get install -y package && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

### 使用多阶段构建
通过在一个 Dockerfile 中使用多个 FROM 指令，可以在一个阶段构建应用，然后在另一个轻量级的基础镜像中运行该应用，从而减少最终镜像的体积。<br />多阶段构建可以将构建过程分为多个阶段，每个阶段使用不同的基础镜像，最终只将必要的文件复制到最终镜像中。

### 使用 .dockerignore 文件
通过在 `.dockerignore` 文件中列出不需要复制到镜像中的文件和目录，可以减少构建上下文的大小，加快构建速度，并减小镜像体积。

### 避免安全漏洞

- **定期更新基础镜像**：定期更新基础镜像以获取安全补丁。
- **使用非 root 用户**：通过 USER 指令切换到非 root 用户，以减少安全风险。
```dockerfile
RUN adduser -D myuser
USER myuser
```

### 善用环境变量
新建目录，创建 test.js：
```javascript
console.log(process.env.name);
console.log(process.env.age);
```
创建 Dockerfile：
```dockerfile
FROM node:18-alpine3.14

WORKDIR /app

COPY ./test.js .

ARG name=Yun
ARG age=20

ENV name=${name} \
    age=${age}

CMD ["node", "/app/test.js"]
```
打包镜像：
```bash
docker build -t env-test:v1.0 .
```
这里的 `-t` 参数用于给镜像命名<br />`.` 表示 Dockerfile 所在的当前目录。<br />运行镜像：
```bash
docker run -it --rm env-test:v1.0
```
`-it` 参数表示运行一个带有交互式 shell 的 Docker 容器。<br />`--rm` 参数表示在容器退出时自动删除容器。<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/21596389/1709352212849-16092558-0c0d-41bf-bcd7-7587dc1f0279.png#averageHue=%23373737&clientId=ud18305a7-83e5-4&from=paste&height=46&id=ub7184449&originHeight=92&originWidth=1018&originalType=binary&ratio=2&rotation=0&showTitle=false&size=10986&status=done&style=none&taskId=u7d60a605-f4fb-49ea-80f8-ade9b227a17&title=&width=509)<br />或者我们可以手动指定：<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/21596389/1709352435472-c0547a8b-5ce0-4ea7-94f4-247b8c726c70.png#averageHue=%23353535&clientId=ud18305a7-83e5-4&from=paste&height=49&id=ue946ca7e&originHeight=98&originWidth=1350&originalType=binary&ratio=2&rotation=0&showTitle=false&size=12809&status=done&style=none&taskId=ubf52ab80-a8b7-4de7-bb15-f74ed404e3d&title=&width=675)<br />如果有很多环境变量，可以存储在文件中，然后在使用 `docker run` 命令时通过 `--env-file` 选项指定该文件。 <br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/21596389/1709352695039-3c80d8af-7e13-4d95-9a13-3a72ede320a7.png#averageHue=%233d3e3f&clientId=ud18305a7-83e5-4&from=paste&height=108&id=uaa698f2c&originHeight=216&originWidth=316&originalType=binary&ratio=2&rotation=0&showTitle=false&size=14635&status=done&style=none&taskId=ud2104dc7-b46f-4e26-b055-dfed9fd20a2&title=&width=158)<br />![image.png](https://cdn.nlark.com/yuque/0/2024/png/21596389/1709352686223-995fff52-dc5c-4565-b415-8e59a1be6c3d.png#averageHue=%23353535&clientId=ud18305a7-83e5-4&from=paste&height=48&id=ue5acc26f&originHeight=96&originWidth=1388&originalType=binary&ratio=2&rotation=0&showTitle=false&size=11463&status=done&style=none&taskId=ua1e779c8-35f2-4015-99e1-8f8745f27d3&title=&width=694)

