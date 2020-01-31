# 手工操作的威胁
> 在这个练习环节，学员将利用 Ansible 自动地完成 OpenShift 项目、Git、Jenkins 和 Nexus 的部署。

![automation-xkcd](https://imgs.xkcd.com/comics/automation.png)
[image-ref](https://xkcd.com/)

## 练习简介
在这个练习中，我们将使用自动化工具创建`CI/CD`工具，以及`dev`和`test`等项目所需的命名空间，我们后面的部署工作将用到它们。我们借助 OpenShift 命令行工具(CLI)手动地完成这些工作；不过，在我们在不同的集群之间、不同的项目之间切换过程中，Dev 团队和 Ops 团队常常发现这些操作经常会一遍遍地重复。通过以代码的方式配置集群，我们可以很方便地将它们记录在 Git 中，并轻松地再次运行。当在这些重复的工作上花的时间减少，我们就可以提高给客户、用户创造价值的能力，关注他们复杂的需求。
在这个练习环节，学员将利用 Ansible 创建集群上的资源。具体来说，我们用到的是一个叫做 [OpenShift-Applier](https://github.com/redhat-cop/openshift-applier) 的工具。在项目的命名空间创建完成之后，我们将会向其中添加更多用于支持 CI/CD 的工具，比如 Jenkins、Git 和 Nexus。在后续的课程中，我们的自动构建和自动部署的过程会需要用到这些工具。此外，这一过程中，我们用 Ansible 操作 OpenShift 模板来完成它们的创建。为了验证整个机制工作良好，我们最后会将所有创建的资源全部删除，通过再次运行 Ansible 脚本来重新创建集群上的资源。

#### 为什么“配置即代码”如此重要？
* 保障 - 防止由于人们无意或随意变更导致的环境配置被意外变更的情况。与雪花服务器说再见！
* 可跟踪 - 以代码的方式描述的配置提交变更时，即意味着用户已被授权，同时这些变更可以跟踪。
* 凤凰服务器 - 将整个环境完全销毁，并重新创建——完全按照原来的样子！

_____

## 学员收获
作为学员，你将能够：

1. 运用 [openshift-applier](https://github.com/redhat-cop/openshift-applier) 自动化地创建集群中的资源
2. 使用 OpenShift 创建并管理项目命名空间
3. 部署用于支持开发流程的常用软件

## 工具和框架

* [GitLab](https://about.gitlab.com/) - 开源的 Git 服务器，现在还集成了一些 DevOps 工具链。
* [Nexus](https://www.sonatype.com/nexus-repository-sonatype) - 可存储各种类型应用的仓储管理软件。也可以认为是`npm`和`Docker`仓库。
* [Jenkins](https://jenkins.io/) - 开源的构建自动化服务器。可以借助丰富的插件实现高度定制化。
* [Ansible](https://www.ansible.com/) - 可用于部署和管理云和物理基础设施的 IT 自动化工具。
* [OpenShift Applier](https://github.com/redhat-cop/openshift-applier) - 用于向 OpenShift 集群创建 OpenShift 资源的工具
* [Eclipse Che](https://www.eclipse.org/che/) - 一个可以从浏览器访问的云端 IDE，我们使用的版本为 [`CodeReady Workspaces`](https://developers.redhat.com/products/codeready-workspaces/overview)

## 全景图
> 全景图指的是我们的架构演进图。从一个空的集群开始，我们逐步向其中添加各个项目，以及 CI/CD 工具链。

![big-picture](../images/big-picture/big-picture-1.jpg)
_____

<!-- ## 10,000 Ft View
> This exercise is aimed at the creation of the tooling that will be used to support the rest of the Exercises. The high-level goal is to create a collection of project namespaces and populate them with Git, Jenkins & Nexus.

If you're feeling confident and don't want to follow the step-by-step guide these high-level instructions should provide a challenge for you:

1. Clone the repo `https://github.com/rht-labs/enablement-ci-cd` which contains the scaffold of the project. Ensure you get all remote branches.

2. Create `<your-name>-ci-cd`, `<your-name>-dev` and `<your-name>-test` project namespaces using the inventory and run them with the [OpenShift Applier](https://github.com/redhat-cop/openshift-applier) to populate the cluster

3. Use the templates provided to create build of the jenkins-s2i. The templates are in `exercise1/jenkins-s2i`

4. Use the templates provided to create build and deployment configs in `<your-name>-ci-cd` for. Templates are on a branch called `exercise1/git-nexus` && `exercise1/jenkins`:
    * Nexus
    * GitLab
    * Jenkins (using an s2i to pre-configure Jenkins)

5. Commit your `enablement-ci-cd` repository to your GitLab repository.

6. Burn it all down and re-apply your inventory proving config-as-code works.
-->

## 操作步骤
<!-- > This is a structured guide with references to exact filenames and explanations.  -->

### 一、创建云端工作空间
> _使用 Che 创建云端 IDE 环境_

1. 要创建云端 IDE 环境，请打开一个浏览器，并访问以下 URL(“魔法链接”)：

```
https://codeready-workspaces.apps.<DOMAIN_FOR_YOUR_CLASS>/dashboard/#/load-factory?name=DO500%20Template&user=admin
```

<p class="tip">
<b>提示</b> - 请使用讲师向你提供的真实、完整的 URL。
</p>

2. 请使用右边`OpenShift 3`按钮登录

![code-ready-workspaces](../images/exercise1/code-ready-workspaces.png)

3. 你应该能看到，工作空间开始创建

![che-workspace-create](../images/exercise1/che-workspace-create.png)

4. 最后，云端 IDE 创建完成

![che-workspace-done](../images/exercise1/che-workspace-done.png)

### 二、创建 OpenShift 项目
> _使用 OpenShift Applier, 我们将会在集群中创建新的命名空间(项目)，在后面的练习中会持续用到这些命名空间。_

1. 在这个课程的过程中，我们会创建两个不同的 Git 项目。请在云端环境中选择`Terminal > Run Task`菜单。

<p class="tip">
⛷️ <b>提示</b> ⛷️ - 如果你不计划使用云端 IDE，也可以把代码克隆到本地：https://github.com/rht-labs/enablement-ci-cd
</p>

2. 在`dev-pod/main`容器中运行`che: init-ci-cd`任务，即可将`enablement-ci-cd`的代码克隆到`/projects`目录：

![init-code1](../images/exercise1/init-code1.png)
![init-code1-complete](../images/exercise1/init-code1-complete.png)

3. 在云端 IDE 中打开`enablement-ci-cd`文件夹(或者，在本地机器上用你喜欢的编辑器打开)，可以看到如下所示的项目结构：
```
.
├── README.md
├── apply.yml
├── docker
├── inventory
│   ├── group_vars
│   │   ├── all.yml
│   ├── host_vars
│   │   ├── ci-cd-tooling.yml
│   │   └── projects-and-policies.yml
│   └── hosts
├── jenkins-s2i
├── requirements.yml
└── templates
    └── project-requests.yml
```
 * `docker`文件夹包含一些示例的 Dockerfile，在后续的构建 jenkins-slave 镜像的过程中会用到
 * `jenkins-s2i`包含用于定制 Jenkins 在首次启动时就准备好的插件及配置信息
 * `params`放置在加载模板时需要使用的变量
 * `templates`是一系列 OpenShift 容器平台(OCP)资源的模板
 * `inventory/*.yml`是 Ansible 主机清单(Inventory)文件，用于管理 OpenShift 集群的对象和资源
 * `requirements.yml`包含运行 Ansible 剧本(Playbook)时所需的模块
 * `apply.yml`是负责配置变量，并运行 OpenShift Applier 角色的剧本

4. 打开文件`inventory/groups_vars/all.yml`，更改变量`namespace_prefix`：将其中的`<YOUR_NAME>`(包括`<`和`>`)替换为讲师提供的名称或你的名字的缩写。**请不要使用大写字母或特殊字符。**比如，如果你名字的拼音是 Zhang Hongjie，你可以把`namespace_prefix`中的`<YOUR_NAME>`替换为`hongjie`or`zhhj`。

<kbd>📝 *enablement-ci-cd/inventory/groups_vars/all.yml*</kbd>
```yaml
  namespace_prefix: "<YOUR_NAME>"
```

5. 打开文件`inventory/host_vars/projects-and-policies.yml`，你将能看到一些已经设置好的变量，它们将用来创建`<YOUR_NAME>-ci-cd`命名空间。它将会被传给 OpenShift Applier，并与来自主机清单和`apply.yml`剧本中`ci_cd`中的变量一起用于调用`templates/project-requests.yml`模板。我们后面会在这里继续添加一些内容，不过目前我们先了解一下这些参数和模板。

6. 在文件`inventory/host_vars/projects-and-policies.yml`之中，你可以看到如下内容：

<kbd>📝 *enablement-ci-cd/inventory/host_vars/projects-and-policies.yml*</kbd>
```yaml
  ci_cd:
    NAMESPACE: "{{ namespace_prefix }}-ci-cd"
    NAMESPACE_DISPLAY_NAME: "{{ namespace_prefix | title }}s CI/CD"
```

 * 它定义了我们很快在部署 CI/CD 项目时所使用的变量。它依赖上面我们更新过的`namespace_prefix`变量。这两组变量组合起来之后，我们现在就可以将它们传给用于创建项目的模板了。你可以注意到，上面的变量的名称(即`ci_cd`)很快就由主机清单中的`params_from_vars`所使用。

<kbd>📝 *enablement-ci-cd/inventory/host_vars/projects-and-policies.yml*</kbd>
```yaml
  ansible_connection: local
  openshift_cluster_content:
  - object: projectrequest
    content:
    - name: "{{ ci_cd_namespace }}"
      template: "{{ playbook_dir }}/templates/project-requests.yml"
      action: create
      params_from_vars: "{{ ci_cd }}"
      tags:
      - projects
```

7. 接下来我们向其中添加两个新的参数字典，它们传给模板之后，就可以用于创建`dev`和`test`项目了。在文件`enablement-ci-cd/inventory/host_vars/projects-and-policies.yml`的顶部，请使用与现有的`ci_cd`类似的内容创建新的字典对象`dev`和`test`。

 * 在编辑器中, 打开`enablement-ci-cd/inventory/host_vars/projects-and-policies.yml`并在`openshift_cluster_content`之前添加如下行:

<kbd>📝 *enablement-ci-cd/inventory/host_vars/projects-and-policies.yml*</kbd>

```yaml
dev:
  NAMESPACE: "{{ namespace_prefix }}-dev"
  NAMESPACE_DISPLAY_NAME: "{{ namespace_prefix | title }} Dev"

test:
  NAMESPACE: "{{ namespace_prefix }}-test"
  NAMESPACE_DISPLAY_NAME: "{{ namespace_prefix | title }} Test"
```

8. 在文件`enablement-ci-cd/inventory/host_vars/projects-and-policies.yml`中，针对要创建的新项目(即 dev 和 test)，向已经存在的`content`数组中分别添加它们对应的对象。可以直接从示例的`ci_cd_namespace`复制，然后做一些必要的修改即可。如果你复制，请确保记得将名称设置为`{{ dev_namespace }}`和`{{ test_namespace }}`，并且对应地修改`params_from_vars`。`ci_cd_namespace`、`dev_namespace`这些名称中变量的值是在项目根目录的`apply.yml`文件中定义的。

<kbd>📝 *enablement-ci-cd/inventory/host_vars/projects-and-policies.yml*</kbd>
```yaml
  - name: "{{ dev_namespace }}"
    template: "{{ playbook_dir }}/templates/project-requests.yml"
    action: create
    params_from_vars: "{{ dev }}"
    tags:
    - projects
  - name: "{{ test_namespace }}"
    template: "{{ playbook_dir }}/templates/project-requests.yml"
    action: create
    params_from_vars: "{{ test }}"
    tags:
    - projects
```

9. 请使用`Terminal > Open Terminal in specific container`菜单项，在`dev-pod/main`容器中打开一个控制台。

![open-terminal](../images/exercise1/open-terminal.png)

<p class="tip">
<b>提示</b> - 如果你使用 <b>z shell</b> 作为默认控制台，请执行此命令：
</p>

```
echo "zsh" >> ~/.bashrc
```

10.  切换到`enablement-ci-cd`目录

```bash
cd enablement-ci-cd
```

11. 在上述所有配置完成之后，就可以安装 OpenShift Applier 依赖项了

```bash
ansible-galaxy install -r requirements.yml --roles-path=roles
```

12.  在控制台中登录 OpenShift，然后按下面的方式，在主机清单上执行 Ansible 剧本(把其中的`<CLUSTER_URL>`替换为讲师指定的值)，如果出现了安全警告，请选择接受 👍：

```bash
oc login <CLUSTER_URL>
```
```bash
ansible-playbook apply.yml -i inventory/ -e target=bootstrap
```

其中的`-e target=bootstrap`用于传入一个额外变量，它指出我们的剧本应该在分组为`bootstrap`的主机清单上运行。

13.  成功之后，你应该能看到类似于这样的输出：

![playbook-success](../images/exercise1/play-book-success.png)

14. 可以通过运行下面的命令来查看成功创建的项目

```bash
oc projects
```

![project-success](../images/exercise1/project-success.png)

### 三、Nexus
> _现在，项目已经准备好了，我们可以开始向其中添加我们在开发过程中需要用到的应用了。_

在这一部分，我们将使用 OpenShift 容器平台的**模板**来安装并配置 Nexus。模板中包含了用于配置一个持久化的 Nexus 服务器的所有内容：暴露服务与路由，并同时创建所需的持久化存储卷。请阅读模板，在底部你能看到一系列将要传入的参数。

<p class="tip">
<b>提示</b> - 下面，我们使用的是从另一个代码仓库(这里使用的是 redhat-cop)的 GitHub 原始 URL 下载的 OpenShift 容器平台模板。
</p>

1. 在你的云端 IDE 中，在`params`目录中创建一个新文件来添加用于运行模板的参数。
```bash
touch params/nexus
```

2. 文件里需要包含的参数有:

<kbd>📝 *enablement-ci-cd/params/nexus*</kbd>
```
VOLUME_CAPACITY=5Gi
MEMORY_LIMIT=1Gi
```

* 你可能注意到了，这与项目参数的定义方式有所不同。这是因为，定义参数的方式可以有很多。在这个例子里，某些变量的的变化可能比其他变量更频繁(比如，给应用分配更多内存等等)。这种情况下，把它们放在专用的的参数文件会更好。

3. 在主机清单变量文件`inventory/host_vars/ci-cd-tooling.yml`里创建一个名为`ci-cd-tooling`新的对象，并按照以下方式设置其`content`值：

<kbd>📝 *enablement-ci-cd/inventory/host_vars/ci-cd-tooling.yml*</kbd>
```yaml
---
openshift_cluster_content:
- galaxy_requirements:
  - "{{ inventory_dir }}/../exercise-requirements.yml"
- object: ci-cd-tooling
  content:
  - name: "nexus"
    namespace: "{{ ci_cd_namespace }}"
    template: "{{ openshift_templates_raw }}/{{ openshift_templates_raw_version_tag }}/nexus/nexus-deployment-template.yml"
    params: "{{ playbook_dir }}/params/nexus"
    tags:
    - nexus
```
![ci-cd-deployments-yml](../images/exercise1/ci-cd-deployments-yml.png)

<p class="tip">
<b>提示</b> 后面会提到，上面内容中的 <i>galaxy_requirements</i> 是在拉取提前和后置步骤的依赖时需要用到的必须配置。
</p>

4. 运行 OpenShift applier，指定标签`nexus`来提高执行速度(`-e target=tools`是用于运行另一个主机清单)。
```bash
ansible-playbook apply.yml -e target=tools \
     -i inventory/ \
     -e "filter_tags=nexus"
```

5. 成功之后，从浏览器登录集群(使用集群 URL)，并切换到`<YOUR_NAME>-ci-cd`项目。你应该能看到 Nexus 已经成功启动。你可以使用默认的用户名和密码进行登录(admin / admin123)
![nexus-up-and-running](../images/exercise1/nexus-up-and-running.png)

### 四、提交 CI/CD

1. 打开 GitLab 登录页面。你可以在 LDAP 标签使用集群的登录凭据(用户名和密码)进行登录。
![gitlab-ui](../images/exercise1/gitlab-ui.png)

2. 登录完成后，创建一个名为`enablement-ci-cd`的新项目，并将其可见级别设置为`Internal`。创建完成后，在下一页将其`git url`复制出来备用。
![gitlab-new-project](../images/exercise1/gitlab-new-project.png)

3. 把本地项目，通过先移除在第一步中克隆 Ansible 项目时所使用的 GitHub 远端，即可提交到这个新的 Git 服务器。在下面的命令中，不要忘记将`<GIT_URL>`替换为你刚才创建的`enablement-ci-cd`仓库的`git url`。
```bash
git remote set-url origin <GIT_URL>
```
```bash
git add .
```
```bash
git commit -m "Adding git and nexus config"
```
```bash
git push -u origin --all
```

### 五、用于 CI 测试的 MongoDB
> _为了支持后续的操作中的 API 测试，我们需要一个供测试执行期间用的 MongoDB。它会成为我们 CI/CD 流程中的一部分，我们现在就来创建它。_

1. 在你的编辑器中打开`enablement-ci-cd`项目。编辑文件`inventory/host_vars/ci-cd-tooling.yml`，按照下面的方式为 MongoDB 添加一个新的对象。可以把它放在`ci-cd-tooling`中 Nexus 的下方。

<kbd>📝 *enablement-ci-cd/inventory/host_vars/ci-cd-tooling.yml*</kbd>
```yaml
  - name: "jenkins-mongodb"
    namespace: "{{ ci_cd_namespace }}"
    template: "openshift//mongodb-ephemeral"
    params: "{{ playbook_dir }}/params/mongodb"
    tags:
    - mongodb
```
![jenkins-mongo](../images/exercise1/jenkins-mongo.png)

2. 使用 Git 提交这些更改以保留记录。
```bash
git add .
```
```bash
git commit -m "ADD - mongodb for use in the pipeline"
```
```bash
git push
```

3. 按照上面一样的方式，执行 Ansible 以令这些变更生效。在 OpenShift 上，转到`<YOUR_NAME>-ci-cd`命名空间就可以验证部署的 MongoDB 服务是否已经出现。
```bash
ansible-playbook apply.yml -e target=tools \
  -i inventory/ \
  -e "filter_tags=mongodb"
```
![ocp-mongo](../images/exercise3/ocp-mongo.png)

<p class="tip">
<b>提示</b> - 当修改 "enablement-ci-cd" 项目时，请尽量频繁地提交并推送到 Git 服务器。
</p>

### 六、Jenkins 和 S2I
> _创建 Jenkins 的 Build Config 和 Deployment Config。使用 s2i 机制向 OpenShift 上的默认 Jenkins 镜像添加新的配置和插件_

1. 按照之前一样的方式，通过新建一个`params/jenkins`文件来创建一组参数，在模板中添加变量，然后完成`<YOUR_NAME>`的替换。

<kbd>📝 *enablement-ci-cd/params/jenkins*</kbd>
```
MEMORY_LIMIT=3Gi
VOLUME_CAPACITY=10Gi
JVM_ARCH=x86_64
NAMESPACE=<YOUR_NAME>-ci-cd
JENKINS_OPTS=--sessionTimeout=720
```
  * 你可能会好奇为什么在这里一定要替换`<YOUR_NAME>`，而不能直接使用之前用过的`namespace_prefix`变量。这是由于这里的变量替换过程是由另一个模板引擎完成的(一个是 Ansible，它可以处理`namespace_prefix`；而另一个是 oc 客户端工具，它不能处理`namespace_prefix`)。这里参数文件是用 oc 客户端工具处理的，所以在这里需要完成替换。

2. 在 Ansible 的主机清单文件`inventory/host_vars/ci-cd-tooling.yml`的 jenkins-mongo 的下方，按照下面的方式添加`jenkins`变量，以创建 Jenkins 的 DeploymentConfig。为了让 Jenkins 能够运行一系列的`npm`命令，我们需要为 Jenkins 配置一个 Build Slave，它需要安装有 Node.js、npm 和 C 编译器。这个 Slave 会在构建运行期间动态地生成。

<p class="tip">
<b>提示</b> 这些 Slave 本身的构建过程可能需要花一些时间，所以为了提高效率，我们在 OpenShift 上给 Slave 配置了对应的 ImageStream。为了利用这个预置的镜像，我们使用了 openshift-applier 中的一个功能来处理主机清单的一系列后置步骤。为了让主机清单正确工作，有一些必要的前置和后置任务。这些步骤就是用来处理这些任务。在这里，我们使用了后置步骤来标记我们的 CI/CD 项目中的 jenkins-slave-npm 这个 ImageStream，从而让 Jenkins 能够找到并使用。</p>

<kbd>📝 *enablement-ci-cd/inventory/host_vars/ci-cd-tooling.yml*</kbd>
```yaml
  - name: "jenkins"
    namespace: "{{ ci_cd_namespace }}"
    template: "{{ openshift_templates_raw }}/{{ openshift_templates_raw_version_tag }}/jenkins/jenkins-persistent-template.yml"
    params: "{{ playbook_dir }}/params/jenkins"
    post_steps:
      - role: casl-ansible/roles/openshift-imagetag
        vars:
          source_img: "quay.io/rht-labs/enablement-npm:latest"
          img_tag: "jenkins-slave-npm:latest"
      - role: casl-ansible/roles/openshift-labels
        vars:
          label: "role=jenkins-slave"
          target_object: "imagestream"
          target_name: "jenkins-slave-npm"
    tags:
    - jenkins
```
如果现在应用这个配置，它就会创建 Jenkins 所需的 DeploymentConfig，但目前`${NAMESPACE}:${JENKINS_IMAGE_STREAM_TAG}`还不存在。

3. 接下来创建这个镜像。我们基于 OpenShift 容器平台支持的 Jenkins 镜像，并使用 [s2i](https://github.com/openshift/source-to-image) 构建器镜像向其中植入一些额外的配置。关于 Jenkins s2i 的更多信息，请参考 [openshift/jenkins](https://github.com/openshift/jenkins#installing-using-s2i-build) GitHub 页面。为了创建给 Jenkins 用的 s2i 配置，我们从`enablement-ci-cd`已经准备好的原有配置开始(在`jenkins-s2i`目录)。

Jenkins s2i 配置的结构为：
```
jenkins-s2i
├── README.md
├── configuration
│   ├── build-failure-analyzer.xml
│   ├── init.groovy
│   ├── jenkins.plugins.slack.SlackNotifier.xml
│   ├── scriptApproval.xml
└── plugins.txt
```
 * `plugins.txt`是由一系列`pluginId:version`构成的列表，可用于 Jenkins 启动时自动安装这些插件
 * `./configuration`包含会置于`${JENKINS_HOME}`的内容。这里的`config.xml`文件可用于批量控制 Jenkins 设置
 * `build-failure-analyzer.xml`是用于插件读取并按正则表达式定位日志的配置，课程后面会涉及到更多这方面的内容
 * `init.groovy`包含一系列 Jenkins 在启动时对其自身进行设置的脚本

4. 作为开始，我们来给 Jenkins 添加插件 [green-balls](https://plugins.jenkins.io/greenballs)。它的功能只是将 Jenkins 默认的`SUCCESS`状态从蓝色改为绿色。在文件`jenkins-s2i/plugins.txt`末尾添加下面的内容：
```txt
greenballs:1.15
```
![green-balls.png](../images/exercise1/green-balls.png)

为什么 Jenkins 使用蓝色表示构建成功？可以在 [on reddit](https://www.reddit.com/r/programming/comments/4lu6q8/why_does_jenkins_have_blue_balls/) 或者 [Jenkins 博客](https://jenkins.io/blog/2012/03/13/why-does-jenkins-have-blue-balls/) 上阅读更多详情。

5. 在构建、部署 Jenkins s2i 之前，要为把你的 Git 登录凭据告诉它，因为Jenkins 访问存储有我们应用的 Git 仓库时需要用到。由于我们希望能从 Jenkins 向Git 仓库推送 Git 标签，所以需要写入权限。请创建`params/jenkins-s2i-secret`文件，并向其中添加下列内容，请对应地替换其中各个变量的值。我们需要修改用于值入 Jenkins 中的登录凭据的 secret。

<kbd>📝 *enablement-ci-cd/params/jenkins-s2i-secret*</kbd>
```
SECRET_NAME=gitlab-auth
USERNAME=<YOUR_LDAP_USERNAME>
PASSWORD=<YOUR_LDAP_PASSWORD>
```
其中
* `<YOUR_LDAP_USERNAME>`是构建 Slave 将用于登录并克隆项目时使用的用户名
* `<YOUR_LDAP_PASSWORD>`是构建 Slave 将用于认证并克隆项目时使用的密码


6. 创建文件`params/jenkins-s2i`，并添加如下内容，请对应地替换其中各个变量的值。

<kbd>📝 *enablement-ci-cd/params/jenkins-s2i*</kbd>
```
SOURCE_REPOSITORY_URL=<GIT_URL>
NAME=jenkins
SOURCE_REPOSITORY_CONTEXT_DIR=jenkins-s2i
SOURCE_REPOSITORY_SECRET=gitlab-auth
```
其中
* `<GIT_URL>`是存储项目所在仓库的完整路径(包括 https 和 .git)

7. 在文件`inventory/host_vars/ci-cd-tooling.yml`的顶部，`---`的下方，添加如下内容：

<kbd>📝 *enablement-ci-cd/inventory/host_vars/ci-cd-tooling.yml*</kbd>
```yaml
ci_cd:
  IMAGE_STREAM_NAMESPACE: "{{ ci_cd_namespace }}"
```

8. 在 Ansible 清单文件`inventory/host_vars/ci-cd-tooling.yml`中创建新的对象`ci-cd-builds`，用于配置 s2i build：

<kbd>📝 *enablement-ci-cd/inventory/host_vars/ci-cd-tooling.yml*</kbd>
```yaml
- object: ci-cd-builds
  content:
  - name: "jenkins-s2i-secret"
    namespace: "{{ ci_cd_namespace }}"
    template: "{{ openshift_templates_raw }}/{{ openshift_templates_raw_version_tag }}/secrets/secret-user-pass-plaintext.yml"
    params: "{{ playbook_dir }}/params/jenkins-s2i-secret"
    tags:
    - jenkins
  - name: "jenkins-s2i"
    namespace: "{{ ci_cd_namespace }}"
    template: "{{ openshift_templates_raw }}/{{ openshift_templates_raw_version_tag }}/jenkins-s2i-build/jenkins-s2i-build-template-with-secret.yml"
    params: "{{ playbook_dir }}/params/jenkins-s2i"
    params_from_vars: "{{ ci_cd }}"
    tags:
    - jenkins
```

9. 把代码提交到 GitLab
```bash
git add .
```
```bash
git commit -m "Adding Jenkins and Jenkins s2i"
```
```bash
git push
```

10. 代码提交完成之后，运行 OpenShift Applier，将这些配置添加到集群上
```bash
ansible-playbook apply.yml -e target=tools \
     -i inventory/ \
     -e "filter_tags=jenkins"
```

11.  这会触发一次 s2i 构建，完成之后，就会在项目里创建新的 ImageStream`<YOUR_NAME>-ci-cd/jenkins:latest`。一旦镜像产生，部署就会自动开始。打开 OpenShift 控制台中的项目，即可查看 s2i 构建的详细情况：
![jenkins-s2i-log](../images/exercise1/jenkins-s2i-log.png)

12. 部署完成后，Jenkins 即可登录(使用 OpenShift 身份登录，并接受角色权限)，你应该能看到一个几乎空的 Jenkins 环境，其中只包含示例任务。

### 七、Jenkins Hello World
> _为了确保各项设施都工作正常，我们来创建一个 hello world 任务，它做的事不多，却可以验证能够正确 git 拉取代码，并且构建成功后展示为绿色。_

1. 登录 Jenkins，点击`新任务`
![new-item](../images/exercise1/new-item.png).

2. 创建名为`hello-world`的任务，并设置类型为`自由风格的任务`
![jenkins-new-hello-world](../images/exercise1/jenkins-new-hello-world.png).

3. 在“源代码管理”标签，添加你的`enablement-ci-cd`项目仓库的地址，在下列列表中选择你的 Git 登录凭据，它是我们在之前的步骤中植入的。
![jenkins-scm-git](../images/exercise1/jenkins-scm-git.png)

4. 在构建标签，添加构建“执行 Shell”，并向其中填入`echo "Hello World"`
![jenkins-hello-world](../images/exercise1/jenkins-hello-world.png).

5. 运行构建，我们应该能看到它成功地通过，并展示出绿色的小球。
![jenkins-green-balls](../images/exercise1/jenkins-green-balls.png)

### 八、线上？下线！从头再来
> _在这一部分，你将通过删除集群上的资源并完全重新创建的操作来验证“基础设施即代码”是有效的。_

1. 向新的代码仓库中提交你的代码。
```bash
git add .
```
```bash
git commit -m "ADD - all ci/cd contents"
```
```bash
git push
```

2. 完全销毁 OpenShift 项目中的资源
```bash
oc delete project <YOUR_NAME>-ci-cd <YOUR_NAME>-dev <YOUR_NAME>-test
```

3. 检查各个项目，以确保我们要删除的项目确实已经消失。
```bash
oc get projects | egrep '<YOUR_NAME>-ci-cd|<YOUR_NAME>-dev|<YOUR_NAME>-test'
```

4. 重放主机清单，并再次创建所有资源！
```bash
oc login <CLUSTER_URL>
```
```bash
ansible-playbook apply.yml -i inventory/ -e target=bootstrap
```
```bash
ansible-playbook apply.yml -i inventory/ -e target=tools
```

_____

## 扩展任务
> _这部分是为提前完成的学员准备的扩展话题。通常我们并不为这些步骤提供操作细节，仅作为超纲内容提供。_

 - 以自动化的方式，基于 secret 为 Nexus 植入更安全的访问方式(比如，不使用 admin / admin123)
 - 为`ci-cd-deployments`添加具有持久化存储的 SonarQube 部署
 - 向`jenkins-s2i/configuration`添加`jenkins.plugins.slack.SlackNotifier.xml`将团队的 Slack 聊天室 URL 作为团队的构建通知，并重新构建 Jenkins S2I 过程。

_____

<!-- ## Additional Reading
> List of links and other reading material that might be of use for the exercise

## Slide links

- TBD
- TBD
- TBD -->

<!-- - [Intro](https://docs.google.com/presentation/d/1LsfAkH8GfIhulEoy_yd-usWBfDHnZEyQdNvYeTmAg4A/)
- [Wrap-up](https://docs.google.com/presentation/d/1cfyJ6SHddZNbM61oz67r870rLYVKY335zGclXN2uLMY/)
- [All Material](https://drive.google.com/drive/folders/13Bt4BXf9P2OB8VI4YQNcNONF1786dqOx) -->
