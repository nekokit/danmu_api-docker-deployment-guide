# Docker版弹幕danmu_api图文部署教程（面板安装版）

---

## 一、准备环境

### 1. 服务器要求

- Linux服务器/NAS/VPS  
- 可访问公网网络  
- 已开放部署端口  

### 2. 安装 Docker（NAS一般已自带，无需再次安装）

#### 安装 Docker

```bash
curl -fsSL https://get.docker.com | bash
```

#### 验证安装

```bash
docker -v
```

---

## 二、安装管理面板

### 1. Linux服务器/VPS 用户推荐安装1panel面板

[1panel官方安装教程](https://1panel.cn/docs/v2/installation/online_installation)

### 2. NAS用户可直接使用自带的管理面板

![nas面板展示](images/step-1.png)

---

## 三、容器创建教程（以飞牛OS为例，其他面板安装步骤类似）

### 1. 进入创建compose页面

![进入创建compose页面](images/step-2.png)

### 2. 填写相关配置

按照图片内标注的顺序，填写相关配置。其中，项目名称可以自定义想要的内容，路径可自由选择想保存的位置，docker-compose请直接复制粘贴下方提供的内容。全部配置完成后直接点击确认。

![填写相关配置](images/step-3.png)

docker-compose配置文件：
```yaml
services:
  danmu-api:
    image: logvar/danmu-api:latest
    container_name: danmu-api
    restart: always
    network_mode: bridge
    ports:
      - 9321:9321
    environment:
      - TZ=Asia/Shanghai
    volumes:
      - ./config:/app/config  # 挂载config目录后，会自动在config目录下创建.env配置文件
      - ./cache:/app/.cache   # 挂载.chche目录后，会将缓存实时保存在本地文件，无需再配置redis
```

### 3. 构建镜像

直接点击构建镜像按钮即可。

![构建镜像](images/step-4.png)

---

## 四、配置面板的管理员权限

### 1. 打开.env配置文件

根据前面配置的路径，找到.env配置文件，并打开。

![打开.env配置文件](images/step-5.png)

### 2. 配置ADMIN_TOKEN

删除ADMIN_TOKEN前面的#，在=后填写想要配置的值

![配置ADMIN_TOKEN](images/step-6.png)

## 五、访问 API

### 1. 浏览器访问

使用下面格式的地址即可访问到普通权限的danmu_api：
```
http://服务器IP:9321/TOKEN
```

使用下面格式的地址即可访问到管理员权限的danmu_api：
```
http://服务器IP:9321/ADMIN_TOKEN
```

### 2. API测试

切换到接口调试菜单，选择搜索动漫接口，输入想要搜索的关键词，点击发送请求。下方的响应结果内能正确显示搜索的内容，说明项目配置正确。

![API测试](images/step-7.png)

---

## 六、容器自动更新教程

安装Watchtower容器，实现自动监控更新。按照图片内标注的顺序，填写相关配置。其中，项目名称可以自定义想要的内容，路径可自由选择想保存的位置，docker-compose请直接复制粘贴下方提供的内容。配置完成后直接点击确认，然后点击构建镜像按钮即可。
![Watchtower安装步骤](images/step-8.png)

docker-compose配置文件：
```yaml
services:
  watchtower:
    image: nickfedor/watchtower
    container_name: watchtower-gx
    restart: always
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - TZ=Asia/Shanghai  # 保持时区正确
    command:
      - --cleanup         # 更新后清理旧镜像
      - --interval        # 间隔参数
      - "12600"           # 30分钟（1800秒），适合测试
      - danmu-api         # 监控的目标容器名
```

---

## 七、卸载容器

直接在面板点击删除，即可完整卸载容器。
![卸载容器](images/step-9.png)

---

## 八、常见问题

### 1. 弹幕匹配错

弹幕匹配错可以考虑以下两种方案：

① 使用剧名映射表TITLE_MAPPING_TABLE，用于自动匹配时替换标题进行搜索，格式：原始标题->映射标题;原始标题->映射标题;... ，例如："唐朝诡事录->唐朝诡事录之西行;国色芳华->锦绣芳华"。

② 打开记住手动选择结果环境变量REMEMBER_LAST_SELECT。

### 2.搜索结果缺集

缺集的情况：

① 默认配置的源是360,vod,renren,hanjutv四个，其中360和VOD等采集站不一定采集了全集，请添加官方源（tencent,youku,iqiyi,imgo,bilibili,migu,sohu,leshi,xigua,maiduidui,renren,hanjutv,bahamut,dandan,animeko）或douban源后重新尝试。

② 请确认是否开启了ENABLE_EPISODE_FILTER手动搜索集标题过滤开关，以及EPISODE_TITLE_FILTER环境变量中有没有过滤关键字匹配到了集标题。

### 3. 巴哈姆特弹幕获取失败

① 巴哈姆特需要能够访问国外的网络环境，请使用PROXY_URL变量配置网络代理。

② 巴哈姆特源的标题可能与国内的不同，请配置TMDB_API_KEY变量，可以帮助巴哈姆特源进行日语原名搜索，提高成功率。
