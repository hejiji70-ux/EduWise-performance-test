# EduWise 智课 — 性能测试

## 项目简介

基于 JMeter 5.6 的核心 API 接口性能测试，覆盖 Admin / Teacher / App 三端，包含 4 个测试场景。

## 技术栈

- JMeter 5.6+
- HTTP 请求采样器
- JSON 断言 + 响应断言
- JSON 提取器（Token 传递）

## 文件结构

```
EduWise-performance-test/
├── README.md
├── .gitignore
├── jmeter/
│   └──EduWise_Performance_Test.jmx      # JMeter 测试脚本（主文件）
└── results/                               # 测试结果（git 忽略）
    ├── *.jtl
    └── reports/
```

## 快速开始

### 前置条件

1. JMeter 5.6+ 已安装（https://jmeter.apache.org/download_jmeter.cgi）
2. Docker 容器已启动：`docker-compose up -d`
3. 服务可访问：http://localhost:9096

### 使用 JMeter GUI 运行

1. 打开 JMeter：`jmeter.bat`（Windows）或 `./jmeter`（macOS / Linux）
2. 文件 → 打开 → 选择 `jmeter/EduWise_Performance_Test.jmx`
3. 点击工具栏运行按钮
4. 观察结果：
   - 「查看结果树」→ 看每个请求的请求/响应详情
   - 「聚合报告」→ 看 QPS、平均响应时间、错误率

### 命令行运行（无 GUI）

```bash
# 基准测试
jmeter -n -t "jmeter/EduWise_Performance_Test.jmx" \
       -l "results/baseline.jtl"

# 生成 HTML 报告
jmeter -g "results/baseline.jtl" \
       -o "results/reports/baseline"
```

## 测试场景

| 场景 | 线程数 | 循环次数 | 覆盖接口 |
|---|---|---|---|
| 场景 1：三端登录压测 | 50 | 20 | Admin/Teacher/App 登录 |
| 场景 2：Admin 后台负载 | 30 | 30 | 用户信息、学员列表、讲师列表、课程列表、统计 |
| 场景 3：App 端业务负载 | 50 | 30 | Banner、分类、课程搜索、个人信息、订单 |
| 场景 4：Teacher 端 | 30 | 30 | 登录、个人信息、学科、课程列表 |

## 全局变量

在 JMeter GUI 左侧「用户自定义变量」节点可直接修改：

| 变量名 | 默认值 | 说明 |
|---|---|---|
| SERVER_HOST | localhost | 服务器地址 |
| SERVER_PORT | 9096 | 服务器端口 |
| ADMIN_USER | admin | Admin 用户名 |
| ADMIN_PWD | 123456 | Admin 密码 |
| TEACHER_MOBILE | 13800138000 | Teacher 手机号 |
| TEACHER_PWD | 123456 | Teacher 密码 |
| APP_MOBILE | 18029240302 | App 学员手机号 |
| APP_PWD | 123456 | App 学员密码 |

## 断言策略

| 接口类型 | 断言内容 |
|---|---|
| 登录接口 | 响应体包含 `"status":200` |
| 列表查询接口 | 响应体包含 `"total"` 字段 |
| 公开接口 | 响应体包含 `[`（数组标记）或特定字段 |

## 注意事项

1. Docker 必须已启动：确保 `http://localhost:9096` 可访问
2. App 端限流严格：原系统有 `@AccessLimit(maxCount=3, seconds=120)` 限流，高并发可能触发限流返回错误（属正常行为）
3. Token 不共享：每个线程组各自登录获取独立的 Token
4. 首次运行：建议先用 5 线程 x 5 循环做冒烟测试，确认环境正常后再加压
