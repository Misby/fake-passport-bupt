# fake-passport-bupt

网页版北邮出入校通行证的代码实现，可以**随机生成信息**或**自定义各种字段**。

本项目仅为代码实现，你可以学习从代码到网站上线的过程，请勿用于非法用途

> 2021 年 9 月 13 日，目前疫情加重，请自觉配合学校疫情防控检查！请勿将本项目用于学习之外的其他用途

> 该软件“按原样”提供，不提供任何形式的明示或暗示的保证，包括但不限于适销性、特定用途的适用性和不侵权的保证。在任何情况下，无论是在合同诉讼、侵权行为或其他方面，作者或版权持有人均不对直接或间接产生于本软件、使用本软件的过程中或对本软件做其他处理产生的任何索赔、损害或其他后果承担任何责任。（译者（本人）不对中文译文的准确性做任何保证，任何信息请以原文为准，详见 LICENSE 文件或 MIT 许可协议。）
> 
> THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

示例：

<img src="readme_assets/example_screenshot.jpg" height="540" />

## 使用方法

### 配置环境变量和后端设置

- 将 `.example.env` 复制一份为 `.env`（其中内容详见文末 [dotenv 中环境变量的说明](#dotenv-中环境变量的说明)）

- 将 `config/example.config.json` 复制一份为 `config/config.json` 

    它是配置文件（需要修改的话可以直接修改它，或者通过 API 修改）

### 接下来你有两种运行方法（任选其一）：

> 如果你不知道选啥，使用第一种即可

#### 一：直接运行（使用 `Node.js`）

1. 确保 `node` 已正常安装（版本需要`>= 14`），通过 `node --version` 检查是否安装成功

2. 进入本项目根目录，`npm install` 或 `yarn`（若你使用 `yarn`）安装依赖

3. `npm run start` 或 `yarn start`（若你使用 `yarn`）开始运行，你应该会看到控制台输出 `listening at 10985`（`10985` 是默认端口，可以在 `.env` 中配置）

4. 现在你可以在浏览器 `localhost:10985` 访问它（详细说明见 API）。桌面端样式不对和字体不对是正常现象，因为学校官方网站也是这样的，真正的完全相同（笑😂，微信访问就正常了

5. 根据你的需要部署至你自己的服务器，可以使用 `./runner.sh start prod --verbose --detach` 运行，方便在后台运行。（`./runner.sh stop dev` 来停止）

    > 如果你好奇那个 runner.sh 是个啥玩意，你可以在这里找到它的详细说明 [charlie0129/server-app-runner](https://github.com/charlie0129/server-app-runner)

#### 二：使用 Docker 容器化

1. 确保安装了 `docker` 和 `docker-compose`
2. `docker-compose --file docker-compose-general.yml up --build` 来 build 并运行
    - 如果你的服务器上使用 `traefik` 做反向代理，你可以参考默认的 docker-compose 配置运行（`docker-compose up --build `）
        - 记得在 `.env` 文件中的 `WEBSITE_URL` 说明你的域名和路由，docker-compose 配置文件中指定 `traefik` 所在的 docker network （你的名称可能和我的不一样）
3. 如果你使用 `docker-compose-general.yml` 运行，那么你可以直接访问主机上你配置的端口。如果你使用了 `traefik`，你可以通过配置的 URL 访问 `traefik` 所在的容器来将流量代理到本项目的容器
4. 在服务器上部署时加上 `--detach` 参数即可在后台运行，`docker-compose down` 或 `docker-compose rm -s -f` 来停止容器

### 在你成功运行后，就可以开始测试访问了！

1. 如果你使用仓库中的默认配置（随机信息生成开、匿名访问开、白名单关），那么你甚至可以不用看 API 说明，因为服务器会随机生成信息。如果你想继续自定义配置，可以继续往下看
2. 参考下面的 GET `/` API 说明，准备一个 URL 来让你自己访问
3. 你可以在手机上打开这串 URL 查看效果（记得使用正确的 host）

## API

### 通行证页面 GET `/`

| URL Param | 含义                     | 默认值（仅在开启随机信息生成时有效） | 是否必填 |
| --------- | ------------------------ | -------- | --------- |
| name      | 你的名字                 | 随机姓名 | 否，关闭匿名访问或开启白名单时必填 |
| school    | 你的学院                 | 随机学院 | 否，关闭匿名访问必填 |
| type      | 出校或入校，填 `出` 或 `入` | 入       | 否 |
| id        | 你的学号                 | 随机学号 | 否，关闭匿名访问或开启白名单时必填 |
| avatar        | 头像 URL 或 xxxx/xxxxxxx            | 无（你能在单次访问网页时更换你手机中的照片） | 否 |
| auth | 额外身份验证信息 | 无 | 否，开启白名单时必填 |

解释：xxxx/xxxxxxx 可以通过查看官方页面的 HTML 中 el-image 类下 img 的 src 找到，形如 1234/1234567

最终你的 URL 看起来会是这样：`http://localhost:10985/?school=你的学院&type=出&id=你的学号&name=你的名字&avatar=https://dummyimage.com/100x100  `（当然这不能直接访问，请看下面的注意点）

#### 注意：

在你通过网址访问时，URL param 中任何字段都不能包含非法字符（比如中文、正斜杠、空格等都属于非法字符，上面的示例 URL 其实是不合法的），你需要转义它们，这里提供两种转义方法：

1. 对于中文来说，在现代浏览器（如 Chrome）中的地址栏完整输入上面的示例 URL 后，在复制出来，你会发现你剪贴板中 URL 的中文部分已经是被转义过的了（当然正斜杠是不会被转义的，也就有了第二种方法）
2. 使用 JavaScript 中的 `encodeURIComponent()` 方法。比如我们想转义 "什么/ &"，可以执行 `encodeURIComponent('什么/ &')`，你会得到 `%E4%BB%80%E4%B9%88%2F%20%26`，只需把这串字符填入对应的字段即可。每个字段的值都去转义一下即可，比如你的 `avatar` 字段的 URL 需要使用这种方法转义。

**示例：**在经过正确的转义后，上面的示例 URL 会变成这样：`http://localhost:10985/?school=%E4%BD%A0%E7%9A%84%E5%AD%A6%E9%99%A2&type=%E5%87%BA&id=%E4%BD%A0%E7%9A%84%E5%AD%A6%E5%8F%B7&name=%E4%BD%A0%E7%9A%84%E5%90%8D%E5%AD%97&avatar=https%3A%2F%2Fdummyimage.com%2F100x100`

### 获取日志 GET `/logs`

> 需要鉴权，记得设置 `AUTH_USERNAME` 和 `AUTH_PASSWORD` 环境变量或在 `.env` 中配置用户名和密码

### 发送全局提醒 POST `/config/alert`

>  需要鉴权，记得设置 `AUTH_USERNAME` 和 `AUTH_PASSWORD` 环境变量或在 `.env` 中配置用户名和密码

开启后，这个提醒会在任何人访问 GET `/` 时弹出

Request body 使用 JSON

| JSON Property | Type   | 含义                 |
| ------------- | ------ | -------------------- |
| alert         | `string` | 用户能看到的提醒信息 |

### 删除全局提醒 DELETE `/config/alert`

>  需要鉴权，记得设置 `AUTH_USERNAME` 和 `AUTH_PASSWORD` 环境变量或在 `.env` 中配置用户名和密码

### 开启/关闭随机信息生成 PUT `/config/random-identity`

>  需要鉴权，记得设置 `AUTH_USERNAME` 和 `AUTH_PASSWORD` 环境变量或在 `.env` 中配置用户名和密码

开启后，在用户 GET `/` 时不设置姓名学号学院等信息时，服务器将随机生成信息

Request body 使用 JSON

| JSON Property | Type   | 含义                 |
| ------------- | ------ | -------------------- |
| enabled       | `boolean` | 开启/关闭随机信息生成 |

### 开启/关闭匿名访问 PUT `/config/anonymous-access`

>  需要鉴权，记得设置 `AUTH_USERNAME` 和 `AUTH_PASSWORD` 环境变量或在 `.env` 中配置用户名和密码

关闭后，任何 GET `/` 的请求都需要有姓名、学号、学院信息，否则 400

Request body 使用 JSON

| JSON Property | Type   | 含义                 |
| ------------- | ------ | -------------------- |
| enabled       | `boolean` | 开启/关闭匿名访问 |

### 设置白名单 POST `/config/whitelist`

>  需要鉴权，记得设置 `AUTH_USERNAME` 和 `AUTH_PASSWORD` 环境变量或在 `.env` 中配置用户名和密码

开启后（下面的 `enabled` ），任何 GET `/` 的请求都需要姓名、学号和 auth 字段与whitelist 中的信息匹配，否则 403 （whitelist 的添加见相关 API）

Request body 使用 JSON

| JSON Property | Type   | 含义                 |
| ------------- | ------ | -------------------- |
| enabled       | `boolean` | 开启/关闭白名单 |
| whitelist       | `{id: string, name: string, auth: string}[]` | 在白名单中的身份（将覆盖原有的，若不想覆盖则使用 PUT） |

### 添加白名单 PUT `/config/whitelist`

>  需要鉴权，记得设置 `AUTH_USERNAME` 和 `AUTH_PASSWORD` 环境变量或在 `.env` 中配置用户名和密码

Request body 使用 JSON

| JSON Property | Type   | 含义                 |
| ------------- | ------ | -------------------- |
| whitelist       | ``{id: string, name: string, auth: string}[]`` | 在白名单中的身份（将添加至原有的） |

### 获取配置信息 GET `/config`

>  需要鉴权，记得设置 `AUTH_USERNAME` 和 `AUTH_PASSWORD` 环境变量或在 `.env` 中配置用户名和密码

获取上面能配置的所有配置项的状态

### dotenv 中环境变量的说明

| ENV             | DESCRIPTION                           |
| --------------- | ------------------------------------- |
| TZ              | 时区                                  |
| WEBSITE_URL     | 使用 `traefik` 反向代理时本容器的 URL |
| PORT            | 后端监听的端口                        |
| AUTH_USERNAME   | 鉴权时的用户名                        |
| AUTH_PASSWORD   | 鉴权时的密码                          |
| LOG_FILENAME    | 日志文件文件名                        |
| CONFIG_FILENAME | 配置文件文件名                        |

