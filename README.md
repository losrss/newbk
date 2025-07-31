# 博客聚合平台 - 宝塔面板部署教程

## 目录
- [1. 准备工作](#1-准备工作)
- [2. 本地项目构建](#2-本地项目构建)
- [3. 宝塔面板配置](#3-宝塔面板配置)
- [4. Nginx配置](#4-nginx配置)
- [5. SSL证书配置](#5-ssl证书配置)
- [6. 网站访问与测试](#6-网站访问与测试)
- [7. 常见问题解决](#7-常见问题解决)

## 1. 准备工作

### 1.1 环境要求
- 已安装宝塔面板的服务器(推荐CentOS 7+)
- 服务器至少1GB内存，20GB存储空间
- 已注册域名(可选但推荐)
- 本地开发环境(Node.js 16+, pnpm)

### 1.2 登录宝塔面板
1. 通过服务器IP和端口访问宝塔面板(默认端口: 8888)
2. 使用用户名和密码登录(首次登录需修改默认密码)

![宝塔登录界面](https://space.coze.cn/api/coze_space/gen_image?image_size=landscape_16_9&prompt=Baota%20panel%20login%20interface%2C%20clean%20UI%2C%20white%20background&sign=b503923fcbbfd9fe1d0066605147180c)

### 1.3 安装必要软件
1. 登录后进入「软件商店」
2. 搜索并安装以下软件:
   - Nginx 1.21+ (推荐使用编译安装)
   - PM2管理器(用于Node.js应用，可选)

![安装Nginx](https://space.coze.cn/api/coze_space/gen_image?image_size=landscape_16_9&prompt=Baota%20panel%20software%20store%2C%20install%20Nginx%2C%20screenshot%20style&sign=7ae1013b04216ab4ccc88829d5a9f0be)

## 2. 本地项目构建

### 2.1 生成生产版本
1. 在本地项目根目录打开终端
2. 运行以下命令安装依赖:
   ```bash
   pnpm install
   ```
3. 构建生产版本:
   ```bash
   pnpm run build
   ```
4. 构建成功后，项目根目录会生成`dist`文件夹，包含所有静态文件

### 2.2 压缩构建文件
1. 将`dist`文件夹压缩为ZIP格式(命名为`blog-client.zip`)
2. 准备好用于上传到服务器

## 3. 宝塔面板配置

### 3.1 创建网站
1. 在宝塔面板左侧菜单中，点击「网站」→「添加站点」
2. 填写网站信息:
   - 域名: 输入您的域名(如无域名可输入服务器IP)
   - 根目录: `/www/wwwroot/blog-client`(自动创建)
   - 创建数据库: 无需(本项目为纯前端应用)
   - PHP版本: 纯静态(选择此项)
3. 点击「提交」创建网站

![创建网站](https://space.coze.cn/api/coze_space/gen_image?image_size=landscape_16_9&prompt=Baota%20panel%20create%20website%20form%2C%20Chinese%20interface&sign=c4818e7b7d00164e7a3f950b8f03abfb)

### 3.2 上传并解压文件
1. 进入刚创建的网站管理页面，点击「文件」
2. 进入网站根目录(`/www/wwwroot/blog-client`)
3. 点击「上传」→「选择文件」，上传之前准备的`blog-client.zip`
4. 上传完成后，选中ZIP文件，点击「解压」，确认解压路径

![上传文件](https://space.coze.cn/api/coze_space/gen_image?image_size=landscape_16_9&prompt=Baota%20panel%20file%20manager%2C%20upload%20and%20extract%20ZIP%20file&sign=41fe12bb69dbb1ffc0eb697f9f04306b)

## 4. Nginx配置

### 4.1 基础配置
1. 返回网站管理页面，点击「设置」→「配置文件」
2. 替换Nginx配置为以下内容:

```nginx
server {
    listen 80;
    server_name yourdomain.com;  # 替换为您的域名或服务器IP
    
    # 网站根目录
    root /www/wwwroot/blog-client/dist/static;
    index index.html;
    
    # 支持前端路由
    location / {
        try_files $uri $uri/ /index.html;
        expires 12h;
        add_header Cache-Control "public, max-age=43200";
    }
    
    # 管理后台路径
    location /admin {
        try_files $uri $uri/ /index.html;
    }
    
    # 静态资源缓存设置
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
        expires 7d;
        add_header Cache-Control "public, max-age=604800";
    }
    
    # 禁止访问隐藏文件
    location ~ /\. {
        deny all;
    }
    
    # 日志配置
    access_log /www/wwwlogs/blog-client-access.log;
    error_log /www/wwwlogs/blog-client-error.log;
}
```

3. 点击「保存」应用配置

### 4.2 配置生效
1. 点击「服务」→「Nginx」→「重载配置」
2. 或在终端执行:
   ```bash
   nginx -s reload
   ```

## 5. SSL证书配置

### 5.1 申请SSL证书
1. 在网站管理页面，点击「SSL」
2. 选择「Let's Encrypt」
3. 勾选域名，选择「文件验证」
4. 点击「申请」

![SSL配置](https://space.coze.cn/api/coze_space/gen_image?prompt=Baota%20panel%20SSL%20configuration%2C%20Let&sign=29ead04c3653766e6acdf8983fc34bde's%20Encrypt%20certificate%20application&image_size=landscape_16_9&sign=17fb6f368481f6bab2216c9603d8e6a8)

### 5.2 强制HTTPS
1. 申请成功后，勾选「强制HTTPS」
2. 点击「保存」

## 6. 网站访问与测试

### 6.1 访问网站
1. 打开浏览器，访问您的域名或服务器IP
2. 您应该能看到博客聚合平台的首页

### 6.2 测试管理后台
1. 访问`https://yourdomain.com/admin`
2. 使用默认管理员账号登录:
   - 用户名: admin
   - 密码: admin123
3. 登录后建议立即修改密码

### 6.3 功能测试
1. 测试博客列表显示
2. 测试搜索功能
3. 测试排序功能
4. 测试管理后台的添加/编辑/删除功能

## 7. 常见问题解决

### 7.1 404错误
- **原因**: Nginx配置未正确处理前端路由
- **解决**: 确保Nginx配置中包含`try_files $uri $uri/ /index.html;`

### 7.2 静态资源加载失败
- **原因**: 根目录配置错误或文件权限问题
- **解决**: 
  1. 检查Nginx配置中的`root`路径是否正确
  2. 确保文件权限正确:
     ```bash
     chmod -R 755 /www/wwwroot/blog-client
     chown -R www:www /www/wwwroot/blog-client
     ```

### 7.3 管理后台无法登录
- **原因**: 本地存储或权限问题
- **解决**:
  1. 清除浏览器缓存和Cookie
  2. 检查Nginx是否正确配置了管理后台路径

### 7.4 网站加载缓慢
- **原因**: 未启用Gzip压缩
- **解决**: 在Nginx配置中添加Gzip压缩:
  ```nginx
  gzip on;
  gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
  ```

## 8. 维护与更新

### 8.1 定期备份
1. 在宝塔面板中，点击「网站」→「备份」
2. 选择需要备份的网站，设置备份周期
3. 建议开启「自动备份」和「备份到云端」

### 8.2 更新网站内容
1. 本地修改代码并重新构建
2. 上传新的`dist`文件夹覆盖服务器上的旧文件
3. 清除浏览器缓存或执行Nginx缓存清理

## 结语
恭喜您成功使用宝塔面板部署了博客聚合平台！如有其他问题，请参考宝塔面板官方文档或提交issue。