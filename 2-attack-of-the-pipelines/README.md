# 攻克流水线

> 在这个练习环节，我们将研究一个“TODO List”应用程序，在 Jenkins 中创建流水线，从而对代码进行构建和部署。

![jenkins-time](../images/exercise2/jenkins-time.jpg)
[image-ref](https://devrant.com/rants/390132/jenkins-builds)

## 练习简介
本课将关注为应用程序创建流水线。流水线指的是将应用从源代码到最终部署的过程中的一系列步骤或阶段。流水线有可能包含多很阶段，不过一个简单的流水线也可以只包含 *构建 > 烘焙 > 部署*(`build > bake > deploy`)的简单流程。通常，第一个阶段是由 git 提交之类的事件所触发。

在各个阶段中，又可能有多个步骤；比如编译代码、运行测试和代码检查。这些操作的目的是为了保障产品的质量，并确保部署的内容与预期行为相符。在练习中，我们将通过 Jenkins 界面创建一条 Jenkins 流水线，从而创建一条通往生产环境的免检快车道。

首先，我们先研究一下示例应用，在本地运行它。我们的示例应用是一个`todolist`(待办事项)应用——它也是现如今一种新的`Hello World`应用形态。

#### 为什么要创建流水线
* 保障 - 确保产品质量的同时，不需要引入单独的部署或发布团队
* 自由 - 让开发人员掌握代码编译的方式和时机的主动性
* 可靠性 - 流水线过程并不有趣，它每次的执行过程都是相同的
* 通往生产环境的快车道:
    - 让产品更快地到达用户手中
    - 让部署过程可重复，各步骤无缝衔接
    - 与生产环境更相似的环境可以增加保障
    - “提前操练”的做法可降低部署生产环境时的风险

## 学员收获
作为学员，完成本课程之后，你将能够：

- 在本地完整地构建并运行 TODO List 应用程序
- 用 Jenkins 界面为前后端创建“免检”流水线
- 为流水线添加分支支持，从而部署到特定的命名空间

## 工具和框架
> 本练习将用到下列工具。你并不需要对他们完全熟悉，但对它们有所了解会很有帮助。你并不需要安装 Vue 和 MongoDB，它们会由我们的`todolist`应用自动管理。

1. [Jenkins](https://jenkins.io/) - 开源的构建自动化服务器；基于插件可实现高度定制化
2. [Node.js](https://nodejs.org/en/) - Node.js® 是一种基于 Chrome 的 V8 JavaScript 引擎的 JavaScript 运行时。Node.js 使用一种事件驱动的、无阻塞的 I/O 模式，因此显得轻量级而高效。Node.js 的包管理系统 npm 是全球最大型的开源库生态系统。
4. [MongoDB](https://www.mongodb.com/what-is-mongodb) - MongoDB 以一种灵活的、 类 JSON 文档的方式存储数据；这意味着其中每个文档的字段都可以不相同，其数据结构可以持续变更
5. [VueJS](https://vuejs.org/) - Vue (读作 /vjuː/, 与 view 发音类似) 是一款用于构建用户界面的框架，它本身也处于持续演进中。Vue 本身的设计深深地融入了“渐进式采用”的理念，在不同的场合中，它既可以被用作一个库，也可以被用作一个框架。它由两部分组成：一个专注于视图层的朴实无华的内核库，以及一个由支持性类库构成的生态系统，这些支持性类库可助你应对大型单页应用中的各种复杂性。
7. [Jenkins Pipeline](https://jenkins.io/doc/book/pipeline/) - 以 Jenkinsfile 的方式构建流水线简介
8. [Pipeline Syntax](https://jenkins.io/doc/book/pipeline/syntax/) - 声明式流水线的文档
9. [Groovy](http://groovy-lang.org/) - Groovy 是 Java 平台上一种强大的、可选类型的动态语言，它兼具静态类型和静态编译能力，其简洁易学的语法大大提高了开发人员的效率。与各种 Java 程序集成后，它可直接为应用增加各种强大的功能，例如脚本编程接口、特定领域专用语言(DSL)的编程平台、运行期和编译期元编程与函数式编程等。Jenkinsfile 就是用 Grovvy 语言编写的，但得益于 Jenkins 流水线 DSL，这里并不要求你非常了解 Grovvy

## 全景图
> 在前面的练习中，我们为应用创建了一些支持性的工具。现在我们开始介绍我们的示例应用，并为其创建流水线。

![big-picture](../images/big-picture/big-picture-2.jpg)

<!-- ## 10,000 Ft View
> _This lab requires users to take the sample TODO app and create a build pipeline in Jenkins by clicking your way to success ending up with an app deployed to each of the namespaces created previously_

2. Import the projects into your gitlab instance. See the README of each for build instructions

2. Deploy a `MongoDB` using the provided template to all project namespace.

2. Create 2 pipelines with three stages (`build`, `bake`, `deploy`) in Jenkins for `develop` & `master` branches on the `todolist-fe` such that:
    * a `Build` job is responsible for compiling and packaging our code:
        1. Checkout from source code (`develop` for `<yourname>-dev` & `master` for `<yourname>-test`)
        2. Install node dependencies and run a build / package
        3. Send the package to Nexus
        4. Archive the workspace to persist the workspace in case of failure
        4. Tag the GitLab repository with the `${JOB_NAME}.${BUILD_NUMBER}` from Jenkins. This is our `${BUILD_TAG}` which will be used on downstream jobs.
        5. Trigger the `bake` job with the `${BUILD_TAG}` param
    * a `Bake` job should take the package and put it in a Linux Container
        1. Take an input of the previous jobs `${BUILD_TAG}` ie `${JOB_NAME}.${BUILD_NUMBER}`.
        2. Checkout the binary from Nexus and unzip its contents
        3. Run an oc start-build of the App's BuildConfig and tag its imagestream with the provided `${BUILD_TAG}`
        4. Trigger a deploy job using the parameter `${BUILD_TAG}`
    * a `deploy` job should roll out the changes by updating the image tag in the DC:
        1. Take an input of the `${BUILD_TAG}`
        2. Patch / set the DeploymentConfig to the image's `${BUILD_TAG}`
        3. Rollout the changes
        4. Verify the deployment

2. Repeat the above setup for the backend `todolist-fe`. TIP - use the copy config to speed things up!

2. Verify that both apps and the DB are talking to one another as expected. -->

## 操作步骤

### 一、了解 Todo List 应用
> _在这个部分，我们将一起了解示例应用，在开展 OCP 中的构建和部署之前，先在本地环境建立对它的了解。_

我们的 Todolist 应用程序，是一个单代码库应用(monorepo)，也就是说在同一个代码库中存储着前端和后端的各个层次。

1. 在容器`dev-pod/main`中运行`che: init-todolist`任务，从而把`todolist`项目的代码克隆到`/projects`目录。

![init-code1](../images/exercise1/init-code2.png)

<p class="tip">
⛷️ <b>注意</b> ⛷️ - 如果你不使用云端 IDE，则可以从此处克隆代码库 https://github.com/rht-labs/todolist.git
</p>

2. 打开 GitLab 并登录。创建一个名为`todolist`的项目(访问权限选择 internal) 用于存放项目副本，复制其远程地址 ![new-gitlab-proj](../images/exercise2/new-gitlab-proj.png)

3. 在提交代码时自动触发 Jenkins 上的构建的操作在本次练习稍后才进行，不过现在我们先创建 WebHook。进入 Settings > Integrations，为新创建的项目创建 WebHook。![gitlab-integrations](../images/exercise2/gitlab-integrations.png) 

4. 在界面表单中，填入包含有 Jenkins 网址、WebHook 位置和密钥令牌的 URL；如果集群用的是自签证书，还需要禁用 SSL 验证；最后完成添加 WebHook。
```bash
https://<YOUR_JENKINS_URL>/multibranch-webhook-trigger/invoke?token=todolist
```

5. 使用`Terminal > Open Terminal in specific container`菜单命令在`dev-pod/main`容器中打开一个终端会话，然后替换下面命令中的`<YOUR_GIT_LAB_PROJECT>`的值之后执行，就可以将本地`todolist`项目副本中存储的 Git 远端替换为 GitLab 地址，并将应用代码推送到 GitLab。

```bash
cd todolist
git remote set-url origin <YOUR_GIT_LAB_PROJECT>
# 验证远端信息已更新
git remote -v
git push -u origin --all
```

6. 我们的`todolist`应用根目录有一个`package.json`文件，它的作用是定义应用的配置，包括它的依赖项、开发过程中的依赖项、脚本，以及其他一些配置。执行下面的命令可以为应用安装依赖：
```bash
npm install
```

7. 如果你使用云端托管环境，就需要在命令行环境中使用你的用户登录到 OpenShift：
```bash
oc login -u <username> -p <password> <CLUSTER_URL>
```

<p class="tip">
️🐇 <b>注意</b> 🐇- 这里需要运行的辅助脚本 fixApiUrl 有一点黑魔法。
</p>

由于我们的环境是托管在云中的，因此需要把为我们应用生成的路由更新到客户端配置中的应用 API 中。请在终端环境中运行`fixApiUrl`脚本。

<p class="tip">
🔥 <b>注意</b> 🔥 - 请确保在运行此命令时，你位于 workspace* 项目，否则的话终端将会崩溃，且脚本不会生效。
</p>

![fixApiUrl](../images/exercise2/fixApiUrl.png)

这一操作会更新配置文件`index.js`中的 API 位置。在执行命令之前，它的内容是：

<kbd>📝 *todolist/src/config/index.js*</kbd>
```
export default {
  todoEndpoint: "/api/todos"
};
```
而之后，你应该能看到类似这样的情况：

![fixApiUrl](../images/exercise2/black-magic.png)

8. 我们的`todolist`应用在其根目录的`package.json`中定义了一些脚本。下面是这些 npm 脚本部分的内容。要运行它们，请执行`npm run <脚本名称>`.

<kbd>📝 *todolist/package.json*</kbd>
```
  "scripts": {
    "serve": "vue-cli-service serve --open",
    "serve:all": "npm-run-all -p -r serve dev:server",
    "start": "node server/app.js",
    "clean": "rm -rf reports package-contents* reports dist ",
    "build:client": "vue-cli-service build",
    "build:ci": "cp src/config/openshift.js src/config/index.js && npm run build:client && mkdir -p package-contents && cp -vr dist server Dockerfile package.json package-contents",
    "package": "zip -r package-contents.zip package-contents",
```
![npm-scripts](../images/exercise2/npm-scripts.png)

9. 接下来，我们从启运数据库、运行应用开始。使用菜单`Terminal > Open Terminal in specific container`在容器`dev-pod/main`中打开一个终端，然后启动 mongo 数据库。

```bash
cd todolist
npm run mongo:start-ide
```
<p class="tip" >
<b>注意</b> - 如果不使用云端托管环境，你也可以通过 <i>npm run mongo</i> 命令来启动 mongo，它会从 <a href="https://hub.docker.com/">Docker Hub</a> 拉取 <i>mongo</i> 镜像。
</p>

在云 IDE 中，会出现一个弹窗问你“是否需要添加重定向”(`add a redirect`)，你可以直接关闭它。
![close-popup](../images/exercise2/close-popup.png)

10. 在云 IDE 中，通过菜单`Terminal > Open Terminal in specific container`在容器`dev-pod/main`中再打开一个终端。现在，可以启动我们的 todolist 应用了。

```bash
cd todolist
npm run serve:all
```

应用启动完成后，我们就会看到一个有`打开链接`(Open Link)的弹窗，点击它可以打开 todolist 应用的页面。

![8080-popup](../images/exercise2/8080-popup.png)

1.   应用启动后，云 IDE 会出现一个显示`todolist`应用首页的预览窗口。
 ![fullstack-app](../images/exercise2/fullstack-app.png)

点击 URL 旁边的箭头按钮可以在云 IDE 之外的浏览器窗口打开预览页面。

![open-in-browser](../images/exercise2/open-in-browser.png)

<p class="tip" >
<b>注意</b> - 在本地环境中，你可以通过浏览器 (http://localhost:8080) 打开应用的首页。
</p>

1.   在云 IDE 中，通过菜单`Terminal > Open Terminal in specific container`在容器`dev-pod/main`中打开第三个终端。请运行一次`curl`命令以测试各个服务都运行良好。API 应该返回预置的数据(存储在文件`server/config/seed.js`中)

```bash
cd todolist
curl -s localhost:9000/api/todos | jq
```
```json
[{
    "_id": "5ac8ff1fdfafb02138698948",
    "title": "Learn some stuff about MongoDB",
    "completed": false,
    "__v": 0
  },
  {
    "_id": "5ac8ff1fdfafb02138698949",
    "title": "Play with NodeJS",
    "completed": true,
    "__v": 0
}]
```

1.  应用启动后，云 IDE 会出现一个显示`todolist`应用首页的预览窗口。
   * 点击首页顶部的 'Todo' 返回上页的页面
   * 服务器端会自动重新启动；所以每当你修改代码，应用就会实时更新

14. 这个应用是一个基于 Vue.js 构建的待办事项管理工具，后端是 Node.js。现在可以随意体验一番应用的功能。可以看到，当添加待办事项时，就会出现在列表中；也可以清除。刷新页面后，这些待办事项还持续地存在。

15. 我们的`todolist`应用的代码结构如下：
```bash
todolist
├── Dockerfile
├── Gruntfile.js
├── jest.config.js
├── jsconfig.json
├── nightwatch.config.js
├── node_modules
├── package.json
├── public
│   ├── favicon.ico
│   ├── img
│   ├── index.html
│   └── manifest.json
├── server
│   ├── api
│   │   └── todo
│   ├── app.js
│   ├── components
│   │   └── errors
│   ├── config
│   │   ├── environment
│   │   ├── express.js
│   │   ├── local.env.sample.js
│   │   └── seed.js
│   ├── mocks
│   │   ├── mock-routes-config.json
│   │   ├── mock-routes.js
│   │   └── mock-routes.spec.js
│   ├── routes.js
│   └── views
│       └── 404.html
├── src
│   ├── App.vue
│   ├── assets
│   ├── components
│   │   └── *
│   ├── config
│   ├── main.js
│   ├── registerServiceWorker.js
│   ├── router.js
│   ├── scss
│   ├── services
│   ├── store
│   │   └── *
│   └── views
│       └── *
└── tasks
│   └── perf-test.js
├── tests
│   ├── e2e
│   └── unit
└── vue.config.js
```
下面是一些重要的条目：
    * `./src`是前端文件目录。入口是在`main.js`文件，它用于加载根级别的`App.vue`文件。
    * `./node_modules`是存储依赖项的位置。
    * `./test`包含端到端测试和单元测试。将在后面的练习中详细介绍。
    * `./src/components`包含应用中的小型、轻量级可重用组件。例如，`NewTodo`组件封装了向列表添加新事项的样式、逻辑和数据。
    * `./src/store`是用于管理应用的状态和与后端的通信的`vuex`文件。
    * `./src/views`是视图容器；负责加载组件并管理它们的交互。
    * `./src/router.js`控制路由逻辑。在我们的情形中，应用只有一个真实的端点(endpoint)。
    * `./src/scss`包含应用中使用的自定义 SCSS 文件。
    * `./*.js`中大部分是用于运行、管理应用及测试的配置文件。
    * `./server`是应用后端所需的主要文件，入口位于`app.js`文件。
    * `./server/api`是存储 API 控制器、数据模型和单元测试的位置。
    * `./server/mocks`是在不访问数据库时的模拟服务器。
    * `./server/config`存储我们的 Express.js 配置，头信息和其他中间件。
    * `./server/config/environment`存储与特定环境相关的配置。比如与 MongoDB 这样的后端服务的连接信息。
    * `./tasks`是一系列额外的`Grunt`任务，将在后面的练习中用到。
    * `Grunt`是一个用于 Node.js 项目的任务运行器。
    * `package.json`包含依赖列表，以及很多在管理应用生命周期过程中有用的脚本。

16.   为了配置用于存储在前后端构建过程中生成的二进制文件的 Nexus 服务， 我们需要执行一个叫做`prepare-nexus`的脚本。在此之前，我们还需要设置一些变量，请将下面的命令中的`<YOUR_NAME>`对应地完成修改后再执行。这个操作只需要执行一次，且在培训环境之外的位置应该是自动的。

```bash
oc login -u <username> -p <password> <CLUSTER_URL>
```
```bash
export NEXUS_SERVICE_HOST=$(oc get route nexus --template='{{.spec.host}}' -n <YOUR_NAME>-ci-cd)
```
```bash
export NEXUS_SERVICE_PORT=443
```
```bash
npm run prepare-nexus
```
<p class="tip">
<b>注意</b> - 在驻训期间，这个步骤是由 ci-cd 项目中一个更复杂的 Nexus 部署过程自动执行的。
</p>

### 二、向集群中添加配置
> _在这个练习中，我们将使用 OpenShift Applier 工具来负责创建应用所需的集群资源，比如 MongoDB，以及应用的构建和部署配置(BuildConfig, DeploymentConfig)_

1. 在你的终端环境中，切换到`todolist`应用的根目录。应用包含一个名为`.openshift-applier`的隐藏的文件夹，切换到此`.openshift-applier`目录，你应该能看到一个熟悉的 Ansible 剧本目录结构：
```
├── README.md
├── inventory
│   ├── group_vars
│   │   └── all.yml
│   └── hosts
├── params
│   └── ocp-pipeline
├── requirements.yml
├── roles
├── apply.yml
└── templates
    ├── mongodb.yml
    ├── ocp-pipeline.yml
    ├── todolist-build.yml
    └── todolist-deploy.yml
```
其中
    * `apply.yml`文件是入口
    * `inventory`包含用于操作集群的对象
    * `params`包含我们在组装模板`templates`时用到的变量
    * `templates`里的模板会在部署应用资源时用到。这些资源包括构建和部署配置(BuildConfig, DeploymentConfig)，以及服务、健康检查和应用定义的其他资源。

2. 在继续之前，我们需要将对`apply.yml`文件中的`<YOUR_NAME>`完成对应的修改。

<kbd>📝 *todolist/.openshift-applier/apply.yml*</kbd>
```
- name: Build and Deploy todolist
  hosts: app
  vars:
    namespace_prefix: '<YOUR_NAME>'
    ci_cd_namespace: '{{ namespace_prefix }}-ci-cd'
```

![applier](../images/exercise2/applier.png)

3. 完成上述修改之后，就可以运行剧本了。先安装依赖项`openshift-applier`，然后使用练习一中用过的`ansible-galaxy`工具来运行剧本(运行时，确保位于`todolist`目录)。这个过程会在集群中创建应用前端所需的各类配置。

```bash
# 请登录，如果需要的话
oc login -u <username> -p <password> <CLUSTER_URL>
```

```bash
ansible-galaxy install -r .openshift-applier/requirements.yml --roles-path=.openshift-applier/roles
```
```bash
ansible-playbook .openshift-applier/apply.yml -i .openshift-applier/inventory/
```

![ansible-success](../images/exercise2/ansible-success.png)

4. 运行成功后，请把代码中的变更提交(`commit`)并推送(`push`)到 GitLab 中。
```bash
git add .
```
```bash
git commit -m "UPDATE - change namespace vars to the teams"
```
```bash
git push
```

5.  确保构建配置(BuildConfig)已在 OpenShift 中创建完成：打开 OpenShift 控制台，切换到`<YOUR_NAME> CI-CD`命名空间，并查看`Build`以查看构建部署(BuildConfig)。
![ocp-app-bc](../images/exercise2/ocp-app-bc.png)

6.  切换到`<YOUR_NAME>-dev`命名空间，确保部署配置(DeploymentConfig)也已创建完成。
![ocp-app-dc](../images/exercise2/ocp-app-dc.png)


### 三、构建 > 烘焙 > 部署
> _在这个练习中，基于我们在本地环境中的操作，我们让它们在 OpenShift 中运行起来_

在这个练习中，我们要创建出流水线的三个阶段(即三个任务)，下面是它们具体情况的概要介绍。后面的步骤将着手实现它们。
* 一个*构建(build)*任务，负责对代码进行编译和打包：
    1. 从源代码管理设施中拉取代码 (`<YOUR_NAME>-dev`命名空间对应`develop`分支，而`<YOUR_NAME>-test`命名空间则对应`master`分支) 
    2. 依赖 node 模块依赖，运行构建、打包过程
    3. 把打好的包发送到 Nexus
    4. 对工作空间进行持久化存档，以防运行失败状况
    5. 使用 Jenkins 的标签`${JOB_NAME}.${BUILD_NUMBER}`标记当前代码库版本
* 一个*烘焙(bake)*任务，它把包获取下来，并将其置于 Linux Container 容器中
    1. 从 Nexus 获取二进制，解压缩从而获取其内容
    2. 执行`oc start-build`操作以运行构建配置(BuildConfig)，并以给定的标签`${BUILD_TAG}`标记镜像流(ImageStream)
* 一个*部署(deploy)*任务，通过更新部署配置(DeploymentConfig)上的镜像标签，将新的变更发布出去：
    1. 更新或设置 DeploymentConfig，令其使用上述新的镜像标签`${BUILD_TAG}`
    2. 发布变更版本
    3. 验证部署结果
* 接下来，我们进入这些步骤的细节

#### 三a、构建

1. 在前面的步骤中，我们已经创建了构建配置(BuildConfig)和部署配置(DeployConfig)。现在请登录 Jenkins，并创建新项目(`New Item`)，这是 Jenkins 对新任务的称谓。<br><br> ![new-item](../images/exercise2/new-item.png)

2. 创建时，设置任务的名称为`dev-todolist-build`，选择`自由风格的项目`(Freestyle Project)。我们的任务名称一般使用`<环境名>-<应用名>-<任务功能>`的格式。
![freestyle-job](../images/exercise2/freestyle-job.png)

3. 接下来是任务配置页，稍后在 Jenkins 中你还可以随时回到这个页面。接下来我们开始配置这个任务。 为了节省一些的空间，我们让 Jenkins 只保留最后一次构建的产物。请勾选`丢弃旧的构建`(Discard old builds)选项，点击`高级`(Advanced)，并将`发布包最大保留#个构建`(Max # of builds to keep with artifacts)，设置为 1。
![keep-artifacts](../images/exercise2/keep-artifacts.png)

4. 我们的 Node.js 构建过程需要在我们在一上章中构建的`jenkins-slave-npm`中运行。请在文本框`限制项目的运行节点`(Restrict where this project can be run)进行设置。
![label-jenkins-slave](../images/exercise2/label-jenkins-slave.png)

5. 在源代码管理区域，请选择 Git，填入`todolist`项目在 GitLab 上的地址，并从下拉列表中选择你的凭据(名称为`<YOUR_NAME>-ci-cd-gitlab-auth`)，将分支设置为`develop`。
![git-scm](../images/exercise2/git-scm.png)

6. 向下滚动到构建环境(Build Environment)标签页，勾选选项`Color ANSI Console Output`。
![ansi](../images/exercise2/ansi.png)

7.  转到构建部分，选择`添加构建步骤`(Add build step)，在下拉列表中选择`执行 Shell`(Execute shell)。在出现的文本框中，输入以下内容，以便为应用打包并发送到 Nexus。
```bash
set -o xtrace
npm install
npm run build:ci
npm run package
npm run publish
```
![build-step](../images/exercise2/build-step.png)

8. 滚动到底部的构建后操作(Post-build Actions)部分。从下拉列表中选择名为`归档发布包`(Archive the artifacts)的构建后操作，并在出现的文本框中填写`** `。这样就可以压缩整个工作空间，以便在需要时恢复给 Jenkins 用于检查。
![archive-artifacts](../images/exercise2/archive-artifacts.png)

9. 还是构建后操作(Post-build Actions)，从下拉列表中添加另一个名为`Git 发布器`(Git Publisher)的构建后操作。它通常用于从构建工具向源代码管理工具以 Git 签入的方式保存信息。
    * 勾选选项`仅成功时才推送`(Push Only If Build Succeeds)
    * 指定推送标签(Tag to push)为：
```bash
${JOB_NAME}.${BUILD_NUMBER}
```
    * 指定提交消息(Tag message)为：
```bash
Automated commit by jenkins from ${JOB_NAME}.${BUILD_NUMBER}
```

    * 勾选选项`创建新标签`(Create New Tag)，并把`目标远端名称`(Target remote name)设置为`origin`
![git-publisher](../images/exercise2/git-publisher.png)

10.  最后一步，为流水线的下一个任务配置触发器，其目的是使用当前的构建标记触发烘焙任务。再添加一个构建后操作，并在下拉列表中选择`触发其他工程中的参数化构建`(Trigger parameterized build on other projects)。
    * 设置要构建的工程(Project to build)为`dev-todolist-bake-deploy`
    * 设置触发条件(Trigger when)为`稳定或不稳定但不失败`(Stable or unstable but not failed)
    * 点击添加参数(Add Parameters)，并选择预定义的参数(Predefined parameters)
    * 在文本框中，以下面的方式输入我们的构建标记(BUILD_TAG)：
```bash
BUILD_TAG=${JOB_NAME}.${BUILD_NUMBER}
```
![param-trigger](../images/exercise2/param-trigger.png)
<p class="tip">
    <b>注意</b> - 在这里 Jenkins 可能会提示“没有这个工程...你指定是...”(No such project...Did you mean...)。无需担心，这是因为我们还没有创建下一个任务。
</p>

11.   点击`保存`(Save)后，你将能看到任务的概要页。这样我们就完成了*构建(Build)*阶段的设置工作。

#### 三b、烘焙与部署

1. 接下来，我们来设置*烘焙(bake)*和*部署(deploy)*阶段，具体过程会简单一些。打开 Jenkins 首页，再创建一个自由风格(Freestyle)的任务，命名为`dev-todolist-bake-deploy`。

2. 此任务需要使用从前序构建中获取的`BUILD_TAG`，因此，请在通用(General)标签页勾选选项`此工程是参数化工程`(This project is parameterized)。
    * 添加一个字符串参数(String parameter)
    * 把名称(Name)设置为`BUILD_TAG`。它会以环境变量的方式提供给任务的执行过程使用
    * 可以把`dev-todolist-build.`设置为它的默认值(default value)供手动触发时使用
    * 描述(description)并不是必须填写的，不过作为操作提示，可以使用 `前序构建的 任务名称.构建序号 ${JOB_NAME}.${BUILD_NUMBER}  比如 dev-todolist-build.1232`
<p class="tip">
    <b>注意</b> - 不要忽略了默认值 <i>dev-todolist-build</i> 之后的点(<i>.</i>)
</p>

![param-trigger-bake](../images/exercise2/param-trigger-bake.png)

3. 这次，我们设置选项`限制项目的运行节点`(Restrict where this project can be run)的值为`master`。
<p class="tip">
    <b>注意</b> <i>烘焙(bake)</i> 步骤只能在 <i>master</i> 节点上执行，这是因为它拥有烘焙过程中所需的工具。
</p>

4. 这个任务不需要配置 Git 或其他源代码管理，所以转到下方的构建环境(Build Environment)，勾选`构建开始之前先删除工作空间`(Delete workspace before build starts)。

5. 接着转到构建(Build)部分，点击`添加构建步骤`(Add build step)，在下拉列表中选择`执行 Shell`(Execute shell)。在出现的文本框中，输入下列命令(不要忘记替换其中的`<YOUR_NAME>`)，以便从 Nexus 拉取包。我们把 Jenkins 中的标记更新到部署配置(BuildConfig)中，这样可以为产品功能获得从源代码到构建产物的跟踪能力。最后，会运行`oc start-build`命令。
```bash
#!/bin/bash
set -o xtrace
echo "### START BAKE IMAGE ###"
curl -v -f \
    http://admin:admin123@${NEXUS_SERVICE_HOST}:${NEXUS_SERVICE_PORT}/repository/zip/com/redhat/todolist/${BUILD_TAG}/package-contents.zip \
    -o package-contents.zip
unzip package-contents.zip
oc project <YOUR_NAME>-ci-cd
NAME=todolist
oc patch bc ${NAME} -p "{\"spec\":{\"output\":{\"to\":{\"kind\":\"ImageStreamTag\",\"name\":\"${NAME}:${BUILD_TAG}\"}}}}"
oc start-build ${NAME} --from-dir=package-contents/ --follow
echo "### END BAKE IMAGE ###"
```
![bake-step](../images/exercise2/bake-step.png)

6. 再次点击`添加构建步骤`(Add build step)，从下拉列表中选择`执行 Shell`(Execute shell)并输入以下命令，不要忘记替换其中的`<YOUR_NAME>`。
```bash
#!/bin/bash
set -o xtrace
echo "### START DEPLOY IMAGE ###"
PIPELINES_NAMESPACE=<YOUR_NAME>-ci-cd
NAMESPACE=<YOUR_NAME>-dev
NAME=todolist
oc project ${NAMESPACE}
oc tag ${PIPELINES_NAMESPACE}/${NAME}:${BUILD_TAG} ${NAMESPACE}/${NAME}:${BUILD_TAG}
oc set env dc ${NAME} NODE_ENV=dev
oc set image dc/${NAME} ${NAME}=docker-registry.default.svc:5000/${NAMESPACE}/${NAME}:${BUILD_TAG}
oc rollout latest dc/${NAME}
echo "### END DEPLOY IMAGE ###"
```
![deploy-step](../images/exercise2/deploy-step.png)

7. 部署完成之后，OpenShift 可以验证其是否成功。下面我们再添加一个步骤：在构建(Build)标签页点击`添加构建步骤`(Add build step)，从下拉列表中选择`验证 OpenShift 部署`(Verify OpenShift Deployment)，并设置：
    * 将项目名称(Project)设置为你对应的`<YOUR_NAME>-dev`
    * 将部署配置(DeploymentConfig)设置为应用的名称`todolist`
    * 将副本数量(replica count)设置为`1`
![verify-deployment](../images/exercise2/verify-deployment.png)

8.  点击`保存`(Save)后，你将转到任务概要页。

#### 三c、流水线

1. 回到 Jenkins，使用构建流水线视图(Build Pipeline View)功能，我们可以把上面的任务串接到一起形成一条流水线，成为一个友好的单一视图。在 Jenkins 首页上，点击顶部标签页右边的加号(+)
![add-view](../images/exercise2/add-view.png)

2. 在新出现的页面上，给我们要创建的视图一个有含义的名称，比如`dev-todolist-pipeline`；然后选择构建流水线视图(Build Pipeline View)。
![new-pipeline](../images/exercise2/new-pipeline.png)

3. 请将流水线流程中的第一个任务(Inital Job)设置为`dev-todolist-build`后保存。
![pipeline-flow](../images/exercise2/pipeline-flow.png)

4. 你应该能看到一个流水线视图了。点击运行(Run)就可以启动流水线了(由于运行过程需要一段时间，所以你可以先继续后面的步骤)。
![dev-pipeline-view](../images/exercise2/dev-pipeline-view.png)

<p class="tip">
    <b>注意</b> - 流水线第一次运行时可能会失败。如果确实如此，请再运行一次，这样三个阶段就都能成功运行，最终显示出绿色的卡片状态。
</p>

5. 为了检查应用在 OpenShift 上的部署状态，请打开 OpenShift 控制台并切换到你的`dev`命名空间。你应该能够看到部署已成功，点击 URL 可以体验部署成功的应用。
![ocp-deployment](../images/exercise2/ocp-deployment.png)

### 四、Jenkinsfile
> _在这个练习中，我们要使用流水线即代码的方式在 Jenkins 中创建流水线_

1. 在云 IDE 中打开你的`todolist`应用，并打开根目录的`Jenkinsfile`文件。下面是这个文件折叠起来的结构：
![pipeline-overview](../images/exercise4/pipeline-overview.png)
其中有一些关键的信息:
    * `pipeline {}`是声明式 Jenkins 流水线的顶层结构
    * `environment {}`定义在所有构建阶段中都可以使用的环境变量
    * `options {}`包含在任务执行过程中的全局设置，比如设置控制台的颜色
    * `stage {}`每个任务(阶段)对应一个`stage`。 这就是构建真正执行的逻辑，比如`bake-image`
    * `steps {}`每个`stage`可以有一个或多个`steps`。它们可以是执行 Shell 或者检出 Git 等。
    * `agent {}`指定构建运行所在的节点，比如`jenkins-slave-npm`
    * `post {}`构建后勾子可用于指定构建后操作。Jenkins 声明式流水线语法为`success`(成功)、`failure`(失败)以及`always`(每次)等情况都提供非常好用的回调，这对控制任务流程非常有用
    * `when {}`用于控制流程。它可以用于`stage`级别并用于让流水线不进入对应的阶段，比如当分支位于`master`时，部署到`test`环境。
    
2. 我们的 Jenkinsfile 大部分内容已经完成，不过还需要少量一些工作来安排命名空间。请查找并替换 Jenkinsfile 中所有的`<YOUR_NAME>`；将`<GITLAB_USERNAME>`的值改为登录集群所用的用户名，它会在检出代码等操作时用到；将`<GITLAB_FQDN>`替换为你 GitLab 的域名(只包括域名的部分，不包括`https://`，也不包括代码仓库的名称)。

<kbd>📝 *todolist/Jenkinsfile*</kbd>
```groovy
   // Jenkinsfile
   
    environment {
        // Global Vars
        NAMESPACE_PREFIX="<YOUR_NAME>"  
        GITLAB_DOMAIN = "<GITLAB_FQDN>"
        GITLAB_USERNAME = "<GITLAB_USERNAME>"

        PIPELINES_NAMESPACE = "${NAMESPACE_PREFIX}-ci-cd"
        APP_NAME = "todolist"

        JENKINS_TAG = "${JOB_NAME}.${BUILD_NUMBER}".replace("/", "-")
        JOB_NAME = "${JOB_NAME}".replace("/", "-")

        GIT_SSL_NO_VERIFY = true
        GIT_CREDENTIALS = credentials("${NAMESPACE_PREFIX}-ci-cd-gitlab-auth")
    }
```

3. 更改完成之后，向`develop`分支推送你的变更。
```bash
git add Jenkinsfile
```
```bash
git commit -m "ADD - namespace and git repo to pipeline"
```
```bash
git push
```

4. 完成变更的推送之后，请打开 Jenkins。

5. 在 Jenkins 上，创建一个新的任务(New Item)，命名为`todolist`，并选择列表底部的`多分支流水线`(Multibranch Pipeline)作为任务类型。
![multibranch-select](../images/exercise4/multibranch-select.png)

6. 在任务的配置页上，将分支源(Branch Sources)设置为`git`
![multibranch-select-git](../images/exercise4/multibranch-select-git.png)

7. 在设置中，填入 GitLab 上的`todolist`项目的 URL，并按照之前的方式设置凭据`https://gitlab.<APPS_URL>/<YOUR_NAME>/todolist.git`
![multibranch-git](../images/exercise4/multibranch-git.png)

8. 将`扫描多分支流水线触发器`(Scan Multibranch Pipeline Triggers)设置为`由 Web 钩子扫描`(Scan by webhook)，并将令牌(token)设置为`todolist`，这也是我们在练习一开始时设置的值。它的作用是在有新变更推送到代码库时，触发任务的扫描获得新的变更。
![multibranch-webhook](../images/exercise2/multibranch-webhook.png)

9. 保存任务的配置信息，以便运行首次的扫描。日志信息会显示针对`master`和`develop`分支的扫描操作，因为它们都有`Jenkinsfile`，于是 Jenkins 会为它们动态地创建流水线。
![todolist-api-multi](../images/exercise2/todolist-api-multi.png)

10.  流水线文件里已设置，当位于`master`或者`develop`分支时， 只运行`bake`和`deploy`阶段。这是为了给工作在功能开发和缺陷修复的各个分支上的开发人员提供快速的反馈。每当有人提交时，或者创建了新的分支时，就会运行包括测试的基础构建，从而给团队一种非常快速的反馈。

11. 在`develop`和`master`分支的构建运行成功之后，我们可以了解一下 Jenkins 的 Blue Ocean 视图。在任务的概要页，点击边栏上的`Open Blue Ocean`按钮，来查看 Jenkins 的现代化视图。
![blue-ocean-todolist](../images/exercise2/blue-ocean-todolist.png)

_____

## 扩展任务
> _这是为积极进取的人士准备的额外加餐，为早期完成人士提供的继续挑战的话题。作为超纲范围，我们通常不为扩展任务提供解答。_

- 流水线任务：
    * 给每个项目的`master`分支创建流水线。在所有配置和名称中都用`test-`替换`dev-`。
    * 实现一个范围更广的端到端自动化流水线，在其中完成`.openshift-applier`步骤。
- 发布到其他环境：
    * 在`master`分支的部署过程之后，添加一个_promote-to-uat(发布到 UAT)_ 的阶段。
    * 使用上面练习过的步骤使用 OpenShift Applier 创建`uat`环境。
    * 在`test-**-deploy`之后，标记镜像，并且在不重新构建的情况下发布到 UAT 环境
- MongoDB 任务：
    * 在 UAT 环境，为 MongoDB 创建有状态副本集(Stateful Set)
    * 使用 config map 与 secret 向 Node.js 应用注入 MongoDB 相关的配置
    * 使用随机生成的用户名、密码提升数据库的安全性
- 将 Nexus 设置为`npm`的镜像仓储，并使用它来为构建过程加速

<!-- 
## Additional Reading
> List of links or other reading that might be of use / reference for the exercise

## Slide links

- [Intro](https://docs.google.com/presentation/d/1t1CONuy-_IRPZYmU010Qgk2rshiDJTennvLyQR8GllE)
- [Wrap-up](https://docs.google.com/presentation/d/1kZ8SV6iJnrKk_AqPpyPuNZifv7VzItHOB9HYdOnNJjI)
- [All Material](https://drive.google.com/drive/folders/1lf66ks2tT0eQ4A9RSU48u0ZhvBXzoHWJ) -->
