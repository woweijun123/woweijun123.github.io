---
{"dg-publish":true,"permalink":"/Work/Tools/Composer/Composer satis/","title":"Composer satis Docker搭建代码仓库","tags":["flashcards"],"noteIcon":"","created":"2026-04-11T08:24:33.000+08:00","updated":"2026-05-21T11:11:59.574+08:00","dg-note-properties":{"title":"Composer satis Docker搭建代码仓库","tags":["flashcards"],"reference linking":"[satis搭建composer包仓库 - 简书](https://www.jianshu.com/p/712bf3c07191)","reference linking2":"[3.4. 使用 Satis 处理私有仓库 | 官方文章 |《Composer 中文文档 2018》| PHP 技术论坛](https://learnku.com/docs/composer/2018/handling-private-packages-with-satis/2092#a9f94d)","reference linking3":"[Docker搭建内网 composer satis 代码仓库](https://www.cnblogs.com/zhanghuizong/p/12693303.html)","reference linking4":"[composer私有仓库搭建之系列一：创建自己的私有仓库](https://www.cnblogs.com/joshua317/p/13111672.html)"}}
---

# 目录结构
```shell
tree -L 3
.
├── docker-compose.yml
├── data
│   └── satis
│       └── composer # 存放satis 容器运行时使用到的配置文件的文件夹
│           ├── cache
│           ├── auth.json
│           ├── composer.json
│           └── satis.json
├── www
│   └── satis # 编译包输出的文件夹
│       ├── dist  # 编译后会自动生成的文件夹
│       ├── include # 编译后会自动生成的文件夹
│       ├── index.html # 编译后会自动生成的文件
│       ├── p2 # 编译后会自动生成的文件夹
│       └── packages.json# 编译后会自动生成的文件
├── README.md
├── satis.sh
```
# 分别创建文件
`auth.json`
```json
{}
```
`composer.json`
```json
{}
```
`satis.json`
```json
{
  "name": "riven/repository",
  "homepage": "http://10.0.0.6:83",
  "repositories": [
    {
      "name": "riven/hyperf-generator",
      "type": "vcs",
      "url": "ssh://git@10.0.0.6:222/my/hyperf-generator.git"
    },
    {
      "name": "amnuts/opcache-gui",
      "type": "vcs",
      "url": "ssh://git@10.0.0.6:222/my/opcache-gui.git"
    },
    {
      "name": "peehaa/opcachegui",
      "type": "vcs",
      "url": "ssh://git@10.0.0.6:222/my/opcache-gui-new.git"
    }
  ],
  "require-all": true,
  "archive": {
    "directory": "dist",
    "format": "zip",
    "skip-dev": false,
    "prefix-url": "http://10.0.0.6:83"
  },
  "config": {
    "secure-http": false
  }
}
```
# 构建模块扩展
composer 官网说明：[用 Satis 处理私有资源包 | Composer 中文文档 | Composer 中文网](https://docs.phpcomposer.com/articles/handling-private-packages-with-satis.html)
satis GitHub代码：[GitHub - composer/satis: Simple static Composer repository generator - For a full private Composer repo use Private Packagist · GitHub](https://github.com/composer/satis)

| 关键词                | 描述                                         |
| ------------------ | ------------------------------------------ |
| repositories       | 指定去哪获取包                                    |
| require            | 指定获取哪些包，如果想获取所有包，使用require-all: true       |
| archive.directory  | dist 文件存放的路径(取代 output-dir)                |
| archive.format     | 可选，默认：zip， 支持两种压缩格式：zip，tar。build时采用的压缩格式  |
| archive.skip-dev   | 可选，默认情况下为 false，启用时（true）satis 不会为分支机构创建下载 |
| archive.prefix-url | 可选,下载位置，主页（来自satis.json），默认情况下是目录          |
### shell 脚本
`satis.sh`
```shell
#!/usr/bin/env bash
docker run --rm --init -it  \
-v /dnmp/data/satis/composer/satis.json:/satis.json:ro \
-v /dnmp/data/satis/composer:/composer:rw \
-v /dnmp/www/satis:/build \
-v ~/.ssh/:/root/.ssh/ \
composer/satis build /satis.json /build "$@"
```
添加执行权限
```shell
chmod +x satis.sh
```
### 增加快捷访问
编辑文件： `vim ~/.bashrc`
```shell
alias satis='bash /dnmp/satis.sh'
```
用法
```shell
# 构建所有代码仓库模块
sh satis.sh
# 更新指定包「会扫描所有包进行查找」
sh satis.sh riven/hyperf-generator
# 更新指定URL下的指定包「只扫描指定包进行查找」
php bin/satis build --repository-url https://only.my/repo.git satis.json web/
```
> **注意**  
> 指定模块安装， `repositories` 节点中必须配置 `name` 字段并保持和项目中的`composer.json`的name一致，同时与 `require` 节点配置对应上
# nginx 配置
```nginx
server {
    listen 10000;
    server_name  _;
    root /www/satis;
    index index.html;
	# 必须开启 Gzip 提升 Composer 检索速度
    gzip on;
    gzip_types application/json;
}
```
可以通过 `http://10.0.0.6:83` 访问到私有仓库界面了，类似访问 [https://packagist.org/](https://packagist.org/)
# 使用
在自己项目中的`composer.json` 中添加或修改类似如下内容
```json
{
    "require": {
        "riven/hyperf-grpc-sdk": "v1.0"
    },
    "config": {
        "secure-http": false
    },
    "repositories": {
        "packagist": {
            "type": "composer",
            "url": "http://10.0.0.6:83"
        }
    }
}
```
# 补充
## 可以配置定时任务，定时拉取
```shell
# 每5分钟拉取
*/5 * * * * root zsh satis.sh >> /dev/null 2>&1
```
## 配置webhook触发更新指定包
```
webhooks 触发重建
```
## 便捷编辑`satis.json`工具
[https://github.com/ludofleury/satisfy](https://github.com/ludofleury/satisfy)
# 公私有包分流
## 结论
生产环境推荐：
- 私有包：走 Satis `http://10.0.0.6:83`
- 公有包：走 [[Work/Tools/Composer/Composer nexus\|Composer nexus]] `http://10.0.0.6:38081/repository/composer-proxy/`
- 禁止直连官方 Packagist `"packagist.org": false`
- HTTP 源开启 `secure-http=false`
## 推荐配置
````json
{
  "repositories": [
    {
      "type": "composer",
      "url": "http://10.0.0.6:83",
      "only": [
        "riven/*"
      ]
    },
    {
      "type": "composer",
      "url": "http://10.0.0.6:38081/repository/composer-proxy/",
      "exclude": [
        "riven/*"
      ]
    },
    {
      "packagist.org": false
    }
  ]
}
````
## 分流规则
- `Satis`：只存放公司私有 Composer 包，例如 `riven/*`。
- `Nexus Composer Proxy`：代理官方 Packagist，缓存公有依赖。
- `Packagist 官方源`：禁用直连，避免生产环境绕过 Nexus。
## 为什么这样配置
### Satis 只处理私有包
私有包应统一命名空间，例如：
```text
riven/*
company/*
internal/*
```
通过 `only` 限制后，Composer 不会拿 `hyperf/*`、`firebase/*`、`google/*` 等公有包去请求 Satis，避免出现 `package not found`。
### Nexus 只处理公有包
Nexus Composer Proxy 用来代理官方 Packagist，负责缓存公有依赖，提高构建速度，并减少外网依赖。
### 禁止直连 Packagist
````json
{
  "packagist.org": false
}
````
可以避免生产构建绕过 Nexus 直接访问公网，保证依赖来源一致。
## 生产建议
1. 私有包必须使用统一命名空间，例如 `riven/*`。
2. Satis 仓库必须加 `only`，避免公有包误查 Satis。
3. Nexus 仓库建议加 `exclude`，避免私有包误走 Packagist 代理。
4. 生产环境建议使用 HTTPS。
5. 如果仍使用 HTTP，需要配置 `secure-http: false`。
6. Jenkins、本地开发、Docker 构建应保持同一套 Composer 源配置。
## 当前项目建议
````json
"repositories": [
  {
    "type": "composer",
    "url": "http://10.0.0.6:83",
    "only": [
      "riven/*"
    ]
  },
  {
    "type": "composer",
    "url": "http://10.0.0.6:38081/repository/composer-proxy/",
    "exclude": [
      "riven/*"
    ]
  },
  {
    "packagist.org": false
  }
]
````