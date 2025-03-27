# LOTUS的TIL BLOG

基于Gin框架构建的高性能博客系统，支持用户认证、文章管理，集成Prometheus监控，采用Docker容器化部署。

## 技术栈

- **后端**: Golang, Gin
- **认证**: JWT (双Token机制)
- **数据存储**: MySQL, Redis
- **监控**: Prometheus, Grafana
- **部署**: Docker
- **API风格**: RESTful

## 主要功能

### 用户认证系统
- 双Token无感刷新机制
  - Access Token: 短期有效，用于API访问认证
  - Refresh Token: 长期有效，存储于Redis，用于刷新Access Token
- 安全机制
  - JWT签名防篡改
  - Token过期机制
  - 用户登出时Token黑名单

### 博客管理
- 文章列表与详情查看
- 个人文章管理
- 基于RESTful风格的API设计

### 系统监控
- 自定义Prometheus中间件
  - 接口调用次数统计
  - 请求响应时间测量
- URL路径规范化处理，提高监控效率
- Grafana可视化监控面板

## API文档

### 认证相关
- `GET /login` - 访问登录页面
- `POST /login/submit` - 提交登录信息
- `POST /token` - 刷新认证Token

### 博客相关
- `GET /blog/belong` - 获取博客归属信息
- `GET /blog/list/:uid` - 获取指定用户的博客列表
- `GET /blog/:bid` - 获取博客详情
- `POST /blog/update` - 更新博客内容 (需要认证)

## 如何运行

### 数据库设置

项目使用MySQL作为主要数据存储。以下是初始化数据库的SQL脚本:

```sql
-- 创建数据库和用户
CREATE DATABASE blog;
CREATE USER 'tester' IDENTIFIED BY '123456';
GRANT ALL ON blog.* TO tester;
USE blog;

-- 用户表
CREATE TABLE IF NOT EXISTS user(
    id INT AUTO_INCREMENT COMMENT '用户id，自增',
    name VARCHAR(20) NOT NULL COMMENT '用户名',
    password CHAR(32) NOT NULL COMMENT '密码的md5',
    PRIMARY KEY (id),
    UNIQUE KEY idx_name (name)
)DEFAULT CHARSET=utf8mb4 COMMENT '用户登录';

-- 创建测试用户
INSERT INTO user (name,password) VALUES 
    ("lotus","e10adc3949ba59abbe56e057f20f883e"),
    ("admin","e10adc3949ba59abbe56e057f20f883e");

-- 博客表
CREATE TABLE IF NOT EXISTS blog(
    id INT AUTO_INCREMENT COMMENT '博客id',
    user_id INT NOT NULL COMMENT '作者id',
    title VARCHAR(100) NOT NULL COMMENT '标题',
    article TEXT NOT NULL COMMENT '正文',
    create_time DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    update_time DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '最后修改时间',
    PRIMARY KEY (id),
    KEY idx_user (user_id)
)DEFAULT CHARSET=utf8mb4 COMMENT '博客内容';

-- 插入TIL (Today I Learned) 风格的博客文章
INSERT INTO blog (user_id,title,article) VALUES (1,"[TIL] Gin框架中优雅处理for循环中的goroutine","今天在重构API处理器时，发现了一个关于Go中for循环与goroutine交互的重要问题。在循环中直接使用goroutine会导致变量捕获问题，所有goroutine可能会共享同一个循环变量。解决方法是在循环内创建局部变量或使用参数传递。这个细节在处理并发请求时尤其重要，避免了数据竞争和不可预期的行为。");

INSERT INTO blog (user_id,title,article) VALUES (1,"Go 1.21新特性实践：泛型切片操作","今天实现了一个通用的数据处理函数，利用了Go 1.21中新增的泛型特性和切片操作函数。特别是slices包中的Filter、Map和Reduce等函数，使代码比传统的for循环更简洁易读。这些函数式编程特性让Go在保持高性能的同时，代码可读性大大提升。项目中的数据转换逻辑因此减少了约30%的代码量。");

INSERT INTO blog (user_id,title,article) VALUES (1,"实现高性能微服务框架的关键点","今天完成了微服务框架的核心组件设计。关键收获：1) 使用上下文传播确保请求追踪和超时控制；2) 基于接口而非具体实现进行依赖注入，提高了测试性和可扩展性；3) 在服务发现方面，结合了DNS和专用注册中心的优点；4) 采用断路器模式处理下游服务故障。这些设计模式确保了系统在高并发下的稳定性和可维护性。");
```

执行上述SQL脚本后，数据库将包含测试用户和示例博客文章，可以直接开始使用系统。

### 本地开发环境

1. 克隆仓库
```bash
git clone https://github.com/LOTUSSSB/tilblog.git
cd tilblog
```

2. 安装依赖
```bash
go mod download
```

3. 运行应用
```bash
go run main.go
```
应用将在 `http://0.0.0.0:5678` 启动

### Docker部署

1. 构建Docker镜像
```bash
docker build -t tilblog:latest .
```

2. 运行容器
```bash
docker run -p 5678:5678 tilblog:latest
```

## 监控系统

项目集成了Prometheus监控，可以通过以下步骤启用:

1. 访问 `/metrics` 端点查看原始监控数据
2. 设置Prometheus抓取配置:
```yaml
scrape_configs:
  - job_name: 'tilblog'
    scrape_interval: 15s
    static_configs:
      - targets: ['0.0.0.0:5678']
```
3. 可导入Grafana监控面板

## 项目亮点

- **高效的认证机制**：双Token设计提高安全性的同时实现用户无感刷新
- **RESTful API设计**：符合行业最佳实践的API接口
- **全面的监控**：自定义中间件收集性能指标，实现系统可观测性
- **容器化部署**：多阶段构建减小镜像体积，保证环境一致性
- **URL路径映射优化**：将动态RESTful请求路径规范化，提升监控效率

## 许可证

MIT

