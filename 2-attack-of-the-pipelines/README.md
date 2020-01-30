# 攻克流水线

> 在这个练习环节，我们将研究一个“TODO List”应用程序，在 Jenkins 中创建流水线，从而对代码进行构建和部署。

![jenkins-time](../images/exercise2/jenkins-time.jpg)
[image-ref](https://devrant.com/rants/390132/jenkins-builds)

## 练习简介
本课将关注为应用程序创建流水线。流水线指的是将应用从源代码到最终部署的过程中的一系列步骤或阶段。流水线有可能包含多很阶段，不过一个简单的流水线也可以只包含 `build > bake > deploy` （构建 > 烘焙 > 部署）的简单流程。通常，第一个阶段是由 git 提交之类的事件所触发。

在各个阶段中，又可能有多个步骤；比如编译代码、运行测试和代码检查。这些操作的目的是为了保障产品的质量，并确保部署的内容与预期行为相符。在练习中，我们将通过 Jenkins 界面创建一条 Jenkins 流水线，从而创建一条通往生产环境的免检快车道。

首先，我们先研究一下示例应用，在本地运行它。我们的示例应用是一个 `todolist`（待办事项）应用——它也是现如今一种新的 `Hello World` 应用形态。

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
> 本练习将用到下列工具。你并不需要对他们完全熟悉，但对它们有所了解会很有帮助。你并不需要安装 Vue 和 MongoDB，它们会由我们的 `todolist` 应用自动管理。

1. [Jenkins](https://jenkins.io/) - 开源的构建自动化服务器；基于插件可实现高度定制化
2. [Node.js](https://nodejs.org/en/) - Node.js® 是一种基于 Chrome 的 V8 JavaScript 引擎的 JavaScript 运行时。Node.js 使用一种事件驱动的、无阻塞的 I/O 模式，因此显得轻量级而高效。Node.js 的包管理系统 npm 是全球最大型的开源库生态系统。
4. [MongoDB](https://www.mongodb.com/what-is-mongodb) - MongoDB 以一种灵活的、 类 JSON 文档的方式存储数据；这意味着其中每个文档的字段都可以不相同，其数据结构可以持续变更
5. [VueJS](https://vuejs.org/) - Vue (读作 /vjuː/, 与 view 发音类似) 是一款用于构建用户界面的框架，它本身也处于持续演进中。Vue 本身的设计深深地融入了“渐进式采用”的理念，在不同的场合中，它既可以被用作一个库，也可以被用作一个框架。它由两部分组成：一个专注于视图层的朴实无华的内核库，以及一个由支持性类库构成的生态系统，这些支持性类库可助你应对大型单页应用中的各种复杂性。
7. [Jenkins Pipeline](https://jenkins.io/doc/book/pipeline/) - 以 Jenkinsfile 的方式构建流水线简介
8. [Pipeline Syntax](https://jenkins.io/doc/book/pipeline/syntax/) - 声明式流水线的文档
9. [Groovy](http://groovy-lang.org/) - Groovy 是 Java 平台上一种强大的、可选类型的动态语言，它兼具静态的类型和静态编译能力，其简洁易学的语法大大提高了开发人员的效率。与各种 Java 程序集成后，它可直接为应用增加各种强大的功能，例如脚本编程接口、特定领域专用语言（DSL）的编程平台、运行时和编译期元编程与函数式编程等。Jenkinsfile 就是用 Grovvy 语言编写的，但由于有了 Jenkins 流水线 DSL，因此并不要求你非常了解 Grovvy。

## 全景图
> 在前面的练习中，我们为应用创建了一些支持性的工具。现在将开始介绍我们的示例应用，并为其创建流水线。

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

我们的 Todolist 应用程序，是一个单代码库应用（monorepo），也就是说在同一个代码库中存储着前端和后端的各个层次。

1. 在容器 `dev-pod/main` 中运行 `che: init-todolist` 任务，从而把 `todolist` 项目的代码克隆到 `/projects` 目录。

![init-code1](../images/exercise1/init-code2.png)

<p class="tip">
⛷️ <b>注意</b> ⛷️ - 如果你不使用云端 IDE，则可以从此处克隆代码库 https://github.com/rht-labs/todolist.git
</p>

2. 打开 GitLab 并登录。创建一个名为 `todolist` 的项目（访问权限选择 internal) 用于存放项目副本，复制其远程地址 ![new-gitlab-proj](../images/exercise2/new-gitlab-proj.png)

3. 在提交代码时自动触发 Jenkins 上的构建的操作在本次练习稍后才进行，不过现在我们先创建 WebHook。进入 Settings > Integrations，为新创建的项目创建 WebHook。![gitlab-integrations](../images/exercise2/gitlab-integrations.png) 

4. 在界面表单中，填入包含有 Jenkins 网址、WebHook 位置和密钥令牌的 URL；如果集群用的是自签证书，还需要禁用 SSL 验证；最后完成添加 WebHook。
```bash
https://<YOUR_JENKINS_URL>/multibranch-webhook-trigger/invoke?token=todolist
```

5. 使用 `Terminal > Open Terminal in specific container` 菜单命令在 `dev-pod/main` 容器中打开一个终端会话，然后替换下面命令中的 `<YOUR_GIT_LAB_PROJECT>` 的值之后执行，就可以将本地 `todolist` 项目副本中存储的 Git 远端替换为 GitLab 地址，并将应用代码推送到 GitLab。

```bash
cd todolist
git remote set-url origin <YOUR_GIT_LAB_PROJECT>
# 验证远端信息已更新
git remote -v
git push -u origin --all
```

6. 我们的 `todolist` 应用根目录有一个 `package.json` 文件，它的作用是定义应用的配置，包括它的依赖项、开发过程中的依赖项、脚本，以及其他一些配置。执行下面的命令可以为应用安装依赖：
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

由于我们的环境是托管在云中的，因此需要把为我们应用生成的路由更新到客户端配置中的应用 API 中。请在终端环境中运行 `fixApiUrl` 脚本。

<p class="tip">
🔥 <b>注意</b> 🔥 - 请确保在运行此命令时，你位于 workspace* 项目，否则的话终端将会崩溃，且脚本不会生效。
</p>

![fixApiUrl](../images/exercise2/fixApiUrl.png)

这一操作会更新配置文件 `index.js` 中的 API 位置。在执行命令之前，它的内容是：

<kbd>📝 *todolist/src/config/index.js*</kbd>
```
export default {
  todoEndpoint: "/api/todos"
};
```
而之后，你应该能看到类似这样的情况：

![fixApiUrl](../images/exercise2/black-magic.png)

8. 我们的 `todolist` 应用在其根目录的 `package.json` 中定义了一些脚本。下面是这些 npm 脚本部分的内容。要运行它们，请执行 `npm run <脚本名称>`.
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

9. 接下来，我们从启运数据库、运行应用开始。使用菜单 `Terminal > Open Terminal in specific container` 在容器 `dev-pod/main` 中打开一个终端，然后启动 mongo 数据库。

```bash
cd todolist
npm run mongo:start-ide
```
<p class="tip" >
<b>注意</b> - 如果不使用云端托管环境，你也可以通过 <i>npm run mongo</i> 命令来启动 mongo，它会从 [Docker Hub](https://hub.docker.com/) 拉取 `mongo` 镜像。
</p>

在云 IDE 中，会有一个弹窗问你“是否需要添加重定向”（`add a redirect`），你可以直接关闭它。
![close-popup](../images/exercise2/close-popup.png)

10.  在云 IDE 中，通过菜单 `Terminal > Open Terminal in specific container` 在容器 `dev-pod/main` 中再打开一个终端。现在，可以启动我们的 todolist 应用了。

```bash
cd todolist
npm run serve:all
```

应用启动完成后，我们就会看到一个有“打开链接”（`Open Link`）的弹窗，点击它可以打开 todolist 应用的页面。

![8080-popup](../images/exercise2/8080-popup.png)

11. 应用启动后，云 IDE 会出现一个显示 `todolist` 应用首页的预览窗口。
 ![fullstack-app](../images/exercise2/fullstack-app.png)

点击 URL 旁边的箭头按钮可以在云 IDE 之外的浏览器窗口打开预览页面。

![open-in-browser](../images/exercise2/open-in-browser.png)

<p class="tip" >
<b>注意</b> - 在本地环境中，你可以通过浏览器 (http://localhost:8080) 以打开应用的首页。
</p>

12.  在云 IDE 中，通过菜单 `Terminal > Open Terminal in specific container` 在容器 `dev-pod/main` 中打开第三个终端。请运行一次 `curl` 命令以测试各个服务都运行良好。API 应该返回预置的数据（存储在 `server/config/seed.js`）

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

1.  Within the cloud IDE a preview of `todolist` app homepage appears when you start the application
    * Click 'Todo' at the top of the home page to get to the above page.
    * The server hosting live reloads; so if you make changes to your code base the app will live update

2.   The app is a todolist manager built in Vue.js. with a Node.js backend. Play around with the App. You will notice when you add todos they appear and clear as expected. If you refresh the page your todos are persisted.

3.   The structure of the `todolist` is as follows.
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
where the following are the important things:
    * `./src` is the collection of front end files. The entrypoint is the `main.js` which is used to load the root `App.vue` file.
    * `./node_modules` is where the dependencies are stored
    * `./test` contains our end-to-end tests and unit tests. More covered on these in later exercises.
    * `./src/components` contains small, lightweight reusable components for our app. For example, the `NewTodo` component which encapsulates the styling, logic and data for adding a new todo to our list
    * `./src/store` is the `vuex` files for managing application state and backend connectivity
    * `./src/views` is the view containers; which are responsible for loading components and managing their interactions.
    * the `./src/router.js` controls routing logic. In our case the app only has one real endpoint.
    * `./src/scss` contains custom SCSS used in the application.
    * `./*.js` is mostly config files for running and managing the app and the tests.
    * `./server` is the main collection of files needed by the app. The entrypoint is the `app.js`.
    * `./server/api` is where the API's controller, data model & unit test are stored.
    * `./server/mocks` is a mock server used for when there is no DB access.
    * `./server/config` stores our Express.js config, header information and other middleware.
    * `./server/config/environment` stores environment specific config; such as connectivity to backend services like MongoDB.
    * `./tasks` is a collection of additional `Grunt` tasks which will be used in later exercises.
    * `Grunt` is a task runner for use with Node.js projects.
    * `package.json` contains the dependency list and a lot of very helpful scripts for managing the app lifecycle.

1.  To prepare Nexus to host the binaries created by the frontend and backend builds we need to run a prepare-nexus script. Before we do this we need to export some variables and change `<YOUR_NAME>` accordingly in the below commands. This is a one time activity and would be automated in a non-training environment.

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
<b>NOTE</b> - This step in a residency would be automated by a more complex nexus deployment in the ci-cd project
</p>

### Part 2 - Add configs to cluster
> _In this exercise; we will use the OpenShift Applier to drive the creation of cluster content required by the app such as MongoDB and the Apps Build / Deploy Config_

1. On your terminal navigate to the root of the `todolist` application. The app contains a hidden folder called `.openshift-applier`. Move into this `.openshift-applier` directory and you should see a familiar looking directory structure for an Ansible playbook.
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
with the following
    * the `apply.yml` file is the entrypoint.
    * the `inventory` contains the objects to populate the cluster with.
    * the `params` contains the variables we'll apply to the `templates`
    * the `templates` required by the app. These include the Build, Deploy configs as well as the services, health checks, and other app definitions.

2. Before we do this we need to change `<YOUR_NAME>` accordingly in the apply.yml file.

<kbd>📝 *todolist/.openshift-applier/apply.yml*</kbd>
```
- name: Build and Deploy todolist
  hosts: app
  vars:
    namespace_prefix: '<YOUR_NAME>'
    ci_cd_namespace: '{{ namespace_prefix }}-ci-cd'
```

![applier](../images/exercise2/applier.png)

3. With those changes in place we can now run the playbook. First install the `openshift-applier` dependency, using the `ansible-galaxy tool` as per exercise one and then run the playbook (from the todolist directory). This will populate the cluster with all the config needed for the front end app.

```bash
# login if needed
oc login -u <username> -p <password> <CLUSTER_URL>
```

```bash
ansible-galaxy install -r .openshift-applier/requirements.yml --roles-path=.openshift-applier/roles
```
```bash
ansible-playbook .openshift-applier/apply.yml -i .openshift-applier/inventory/
```

![ansible-success](../images/exercise2/ansible-success.png)

4. Once successful, `commit` and `push` your changes to gitlab.
```bash
git add .
```
```bash
git commit -m "UPDATE - change namespace vars to the teams"
```
```bash
git push
```

5.  Validate the build and deploy configs have been created in OpenShift by opening the console and checking `<YOUR_NAME> CI-CD builds` for the `BuildConfigs`
![ocp-app-bc](../images/exercise2/ocp-app-bc.png)

6. Check `<YOUR_NAME>-dev` to see the deployment configs are in place
![ocp-app-dc](../images/exercise2/ocp-app-dc.png)


### Part 3 - Build > Bake > Deploy
> _In this exercise; we take what we have working locally and get it working in OpenShift_

This exercise will involve creating three stages (or items) in our pipeline, each of these is detailed below at a very high level. Move on to the next step to begin implementation.
* a *build* job is responsible for compiling and packaging our code:
    1. Checkout from source code (`develop` for `<YOUR_NAME>-dev` & `master` for `<YOUR_NAME>-test`)
    2. Install node dependencies and run a build / package
    3. Send the package to Nexus
    4. Archive the workspace to persist the workspace in case of failure
    5. Tag the GitLab repository with the `${JOB_NAME}.${BUILD_NUMBER}` from Jenkins.
* a *bake* job should take the package and put it in a Linux Container
    1. Checkout the binary from Nexus and unzip its contents
    2. Run an oc start-build of the App's BuildConfig and tag its imagestream with the provided `${BUILD_TAG}`
* a *deploy* job should roll out the changes by updating the image tag in the DC:
    1. Patch / set the DeploymentConfig to the image's `${BUILD_TAG}`
    2. Rollout the changes
    3. Verify the deployment
* We will now go through these steps in detail.

#### 3a - Build

1. With the BuildConfig and DeployConfig in place for both the app from previous steps; Log into Jenkins and create a `New Item`. This is just jenkins speak for a new job configuration.<br><br> ![new-item](../images/exercise2/new-item.png)

2. Name this job `dev-todolist-build` and select `Freestyle Project`. Our job will take the form of `<ENV>-<APP_NAME>-<JOB_FUNCTION>`. ![freestyle-job](../images/exercise2/freestyle-job.png)

3. The page that loads is the Job Configuration page and it can be returned to at anytime from Jenkins. Let's start configuring our job. To conserve space; we will make sure Jenkins only keeps the last build's artifacts. Tick the `Discard old builds` checkbox, then `Advanced` and set `Max # of builds to keep with artifacts` to 1 as indicated below
![keep-artifacts](../images/exercise2/keep-artifacts.png)

4. Our Node.js build needs to be run on the `jenkins-slave-npm` we bought in in the previous chapter. Specify this in the box labelled `Restrict where this project can be run` ![label-jenkins-slave](../images/exercise2/label-jenkins-slave.png)

5. On the Source Code Management tab, select the Git radio button, specify the endpoint for our GitLab `todolist` Project and specify your credentials (`<YOUR_NAME>-ci-cd-gitlab-auth`) from the dropdown box. Set the Branch Specifier to `develop`. ![git-scm](../images/exercise2/git-scm.png)

6. Scroll down to the Build Environment tab and select the `Color ANSI Console Output` checkbox ![ansi](../images/exercise2/ansi.png)

7. Move on to the Build section and select `Add build step`. From the dropdown select `Execute shell`. On the box that appears; insert the following, to build package and deploy our app to Nexus:
```bash
set -o xtrace
npm install
npm run build:ci
npm run package
npm run publish
```
![build-step](../images/exercise2/build-step.png)

8. Scroll to the final section; the Post-build Actions. Add a new post-build action from the dropdown called `Archive the artifacts` and specify `**` in the box. This will zip the entire workspace and copy it back to Jenkins for inspection if needed. ![archive-artifacts](../images/exercise2/archive-artifacts.png)

9. On the Post-build Actions; Add another post-build action from the dropdown called `Git Publisher`. This is useful for tying the git check-in to the feature in your tracking tool to the built product.
    * Tick the box `Push Only If Build Succeeds`
    * Add the Tag to push of
```bash
${JOB_NAME}.${BUILD_NUMBER}
```
    * Specify the commit message to be
```bash
Automated commit by jenkins from ${JOB_NAME}.${BUILD_NUMBER}
```

    * Check `Create New Tag` and set `Target remote name` to `origin`
![git-publisher](../images/exercise2/git-publisher.png)

10. Finally; add the trigger for the next job in the pipeline. This is to trigger the bake job with the current build tag. Add another post-build action from the dropdown called `Trigger parameterized build on other projects`.
    * Set the project to build to be `dev-todolist-bake-deploy`
    * Set the condition to be `Stable or unstable but not failed`
    * Click Add Parameters dropdown and select Predefined parameters.
    * In the box, insert our BUILD_TAG as follows
```bash
BUILD_TAG=${JOB_NAME}.${BUILD_NUMBER}
```
![param-trigger](../images/exercise2/param-trigger.png)
<p class="tip">
    <b>NOTE</b> - Jenkins might say "No such project ‘dev-todolist-bake-deploy’. Did you mean ...." at this point. Don't worry; it's because we have not created the next job yet.
</p>

11. Hit `save` which will take you to the job overview page - and that's it; our *build* phase is complete!

#### 3b - bake & deploy

1. Next we will setup our *bake* and *deploy* phase; which is a little simpler. Go to Jenkins home and create another Freestyle Job (as before) called `dev-todolist-bake-deploy`.

2. This job will take in the BUILD_TAG from the previous one so check the `This project is parameterized` box on the General tab.
    * Add string parameter type
    * set the Name to `BUILD_TAG`. This will be available to the job as an Enviroment Variable.
    * You can set `dev-todolist-build.` as the default value for ease when triggering manually.
    * The description is not required but a handy one for reference would be `${JOB_NAME}.${BUILD_NUMBER} of previous build e.g. dev-todolist-build.1232`
<p class="tip">
    <b>NOTE</b> - Don't forget to include the <i>.</i> after <i>dev-todolist-build</i> in the Default Value box.
</p>

![param-trigger-bake](../images/exercise2/param-trigger-bake.png)

3. This time set the `Restrict where this project can be run` label to `master`.
<p class="tip">
    <b>NOTE</b> The <i>bake</i> step can only be executed on the <i>master</i> node because it contains the tools for baking.
</p>

4. There is no Git or SCM needed for this job so move down to the Build Environment and tick `Delete workspace before build starts`

5. Move on to the Build section and select `Add build step`. From the dropdown select `Execute shell`. On the box the appears; insert the following, to pull the package from Nexus. We patch the BuildConfig with the Jenkins Tag to get traceablility from feature to source code to built item. Finally; the oc start-build command is run:
Remember to replace `<YOUR_NAME>` accordingly.
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

6. Hit `Add build step` again and from the dropdown select `Execute shell`. Enter the following text and remember to change `<YOUR_NAME>` accordingly.
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

7. When a deployment has completed; OpenShift can verify its success. Add another step by clicking the `Add build step` on the Build tab then `Verify OpenShift Deployment` including the following:
    * Set the Project to your `<YOUR_NAME>-dev`
    * Set the DeploymentConfig to your app's name `todolist`
    * Set the replica count to `1`
![verify-deployment](../images/exercise2/verify-deployment.png)

8.  Hit `save` which will take you to the job overview page.

#### 3c - Pipeline

1. Back on Jenkins; We can tie all the jobs in the pipeline together into a nice single view using the Build Pipeline view. Back on the Jenkins home screen Click the + beside the all tab on the top.
![add-view](../images/exercise2/add-view.png)

2. On the view that loads; Give the new view a sensible name like `dev-todolist-pipeline` and select Build Pipeline View
![new-pipeline](../images/exercise2/new-pipeline.png)

3. Set the Pipeline Flow's Inital Job to `dev-todolist-build` and save.
![pipeline-flow](../images/exercise2/pipeline-flow.png)

4. You should now see the pipeline view. Run the pipeline by hitting run (you can move onto the next part while it is running as it may take some time).
![dev-pipeline-view](../images/exercise2/dev-pipeline-view.png)

<p class="tip">
    <b>NOTE</b> - The pipeline may fail on the first run. In such cases, re-run the pipeline once more and the three stages will run successfully and show three green cards.
</p>

5. To check the deployment in OpenShift; open the web console and go to your `dev` namespace. You should see the deployment was successful; hit the URL to open the app and play with the deployed.
![ocp-deployment](../images/exercise2/ocp-deployment.png)

### Part 4 - The Jenkinsfile
> _In this exercise we'll use pipeline-as-code to create a pipeline in Jenkins_

1. Open up your `todolist` application in your cloud IDE and move to the `Jenkinsfile` in the root of the project. The high-level structure of the file is shown collapsed below.
![pipeline-overview](../images/exercise4/pipeline-overview.png)
Some of the key things to note:
    * `pipeline {}` is how all declarative Jenkins pipelines begin.
    * `environment {}` defines environment variables to be used across all build stages
    * `options {}` contains specific Job specs you want to run globally across the jobs e.g. setting the terminal colour
    * `stage {}` all jobs must have one stage. This is the logical part of the build that will be executed e.g. `bake-image`
    * `steps {}` each `stage` has one or more steps involved. These could be execute shell or git checkout etc.
    * `agent {}` specifies the node the build should be run on e.g. `jenkins-slave-npm`
    * `post {}` hook is used to specify the post-build-actions. Jenkins declarative pipeline syntax provides very useful callbacks for `success`, `failure` and `always` which are useful for controlling the job flow
    * `when {}` is used for flow control. It can be used at the stage level and be used to stop pipeline entering that stage. e.g. when branch is master; deploy to `test` environment.
    
2. The Jenkinsfile is mostly complete, however some minor changes will be needed to orchestrate namespaces. Find and replace all instances of `<YOUR_NAME>` in the Jenkinsfile. Update the `<GITLAB_USERNAME>` to the one you log in to the cluster with; this variable is used in the namespace of your Git projects when checking out code etc. Replace `<GITLAB_FQDN>` with your Git domain (only the hostname, without `https://` or the repository name).

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

3. With these changes in place, push your changes to the `develop` branch.
```bash
git add Jenkinsfile
```
```bash
git commit -m "ADD - namespace and git repo to pipeline"
```
```bash
git push
```

4. When the changes have been successfully pushed; Open Jenkins.

5. Create a `New Item` on Jenkins. Give it the name `todolist` and select `Multibranch Pipeline` from the bottom of the list as the job type.
![multibranch-select](../images/exercise4/multibranch-select.png)

6. On the job's configure page; set the Branch Sources to `git`
![multibranch-select-git](../images/exercise4/multibranch-select-git.png)

7. Fill in the Git settings with your `todolist` GitLab url and set the credentials as you've done before. `https://gitlab.<APPS_URL>/<YOUR_NAME>/todolist.git`
![multibranch-git](../images/exercise4/multibranch-git.png)

8. Set the `Scan Multibranch Pipeline Triggers` to be Scan by webhook and set the token to be `todolist` as we set at the beginning of the exercise. This will trigger the job to scan for changes in the repo when there are pushes. 
![multibranch-webhook](../images/exercise2/multibranch-webhook.png)

9.  Save the Job configuration to run the intial scan. The log will show scans for `master` and `develop` branches, which have `Jenkinsfile` so pipelines are dynamically created for them.
![todolist-api-multi](../images/exercise2/todolist-api-multi.png)

10.  The pipeline file is setup to only run `bake` & `deploy` stages when on `master` or `develop` branch. This is to provide us with very fast feedback for team members working on feature or bug fix branches. Each time someone commits or creates a new branch a basic build with testing occurs to give very rapid feedback to the team. 

11.  With the builds running for  `develop` and `master` we can explore the Blue Ocean View for Jenkins. On the Job overview page, hit the `Open Blue Ocean` button on the side to see what modern Jenkins looks like.
![blue-ocean-todolist](../images/exercise2/blue-ocean-todolist.png)

_____

## Extension Tasks
> _Ideas for go-getters. Advanced topic for doers to get on with if they finish early. These will usually not have a solution available and are provided for additional scope._

- Pipeline Tasks:
    * Add pipeline for `master` branch for each project. Use `test-` instead of `dev-` across all config and names in the pipeline.
    * Do the `.openshift-applier` steps as part of the pipeline for greater end to end automation.
- Promote build:
    * Create a _promote-to-uat_ phase after the `master` branch deploy.
    * Create a `uat` env using the OpenShift Applier as seen before.
    * Tag and promote the image without rebuilding after the `test-**-deploy`
- MongoDB tasks:
    * Add MongoDB Stateful set for the UAT environment (or test).
    * Inject MongoDB config into the Node.js app using config map & secrets.
    * Improve the security of the DB by making the user /passwords randomly generated.
- Setup Nexus as an `npm` mirror registry and use it in the builds to speed up the build time.

<!-- 
## Additional Reading
> List of links or other reading that might be of use / reference for the exercise

## Slide links

- [Intro](https://docs.google.com/presentation/d/1t1CONuy-_IRPZYmU010Qgk2rshiDJTennvLyQR8GllE)
- [Wrap-up](https://docs.google.com/presentation/d/1kZ8SV6iJnrKk_AqPpyPuNZifv7VzItHOB9HYdOnNJjI)
- [All Material](https://drive.google.com/drive/folders/1lf66ks2tT0eQ4A9RSU48u0ZhvBXzoHWJ) -->
