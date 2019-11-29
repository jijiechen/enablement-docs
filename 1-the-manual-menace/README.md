# 手工操作的威胁
> 在这个练习环节，学员将利用 Ansible 自动地完成 OpenShift 项目、Git、Jenkins 和 Nexus 的部署。

![automation-xkcd](https://imgs.xkcd.com/comics/automation.png)
[image-ref](https://xkcd.com/)

## 练习简介
在这个练习中，我们将使用自动化工具创建 `CI/CD` 工具，以及 `dev` 和 `test` 等项目所需的命名空间，我们后面的部署工作将用到它们。我们借助 OpenShift 命令行工具（CLI）手动地完成这些工作；不过，在我们在不同的集群之间、不同的项目之间切换过程中，Dev 团队和 Ops 团队常常发现这些操作经常会一遍遍地重复。通过以代码的方式配置集群，我们可以很方便地将它们记录在 Git 中，并轻松地再次运行。当在这些重复的工作上花的时间减少，我们就可以提高给客户、用户创造价值的能力，关注他们复杂的需求。
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
* [Nexus](https://www.sonatype.com/nexus-repository-sonatype) - 可存储各种类型应用的仓储管理软件。也可以认为是 `npm` 和 `Docker` 仓库。
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

### 第 1 部分 - 创建云端工作空间
> _使用 Che 创建云端 IDE 环境_

1. 要创建云端 IDE 环境，请打开一个浏览器，并访问以下 URL（“神奇链接”）：

```
https://codeready-workspaces.apps.<DOMAIN_FOR_YOUR_CLASS>/dashboard/#/load-factory?name=DO500%20Template&user=admin
```

<p class="tip">
<b>提示</b> - 请使用讲师向你提供的真实、完整的 URL。
</p>

2. 请使用右边 `OpenShift 3` 按钮登录

![code-ready-workspaces](../images/exercise1/code-ready-workspaces.png)

3. 你应该能看到，工作空间开始创建

![che-workspace-create](../images/exercise1/che-workspace-create.png)

4. 最后，云端 IDE 创建完成

![che-workspace-done](../images/exercise1/che-workspace-done.png)

### 第 2 部分 - 创建 OpenShift 项目
> _使用 OpenShift Applier, 我们将会在集群中创建新的命名空间（项目），在后面的练习中会持续用到这些命名空间。_

1. 在这个课程的过程中，我们会创建两个不同的 Git 项目。请在云端环境中选择 `Terminal > Run Task` 菜单。

<p class="tip">
⛷️ <b>提示</b> ⛷️ - 如果你不计划使用云端 IDE，也可以把代码克隆到本地：https://github.com/rht-labs/enablement-ci-cd
</p>

2. 在 `dev-pod/main` 容器中运行 `che: init-ci-cd` 任务，即可将 `enablement-ci-cd` 的代码克隆到 `/projects` 目录：

![init-code1](../images/exercise1/init-code1.png)
![init-code1-complete](../images/exercise1/init-code1-complete.png)

3. 在云端 IDE 中打开 `enablement-ci-cd` 文件夹（或者，在本地机器上用你喜欢的编辑器打开），可以看到如下所示的项目结构：
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
 * `docker` 文件夹包含一些示例的 Dockerfile，在后续的构建 jenkins-slave 镜像的过程中会用到
 * `jenkins-s2i` 包含用于定制 Jenkins 在首次启动时就准备好的插件及配置信息
 * `params` 放置在加载模板时需要使用的变量
 * `templates` 是一系列 OpenShift 容器平台（OCP）资源的模板
 * `inventory/*.yml` 是 Ansible 主机清单（Inventory）文件，用于管理 OpenShift 集群的对象和资源
 * `requirements.yml` 包含运行 Ansible 剧本（Playbook）时所需的模块
 * `apply.yml` 是负责配置变量，并运行 OpenShift Applier 角色的剧本

4. 打开文件 `inventory/groups_vars/all.yml`，更改变量 `namespace_prefix`：将其中的 `<YOUR_NAME>`（包括 `<` 和 `>`）替换为讲师提供的名称或你的名字的缩写。**请不要使用大写字母或特殊字符。**比如，如果你名字的拼音是 Zhang Hongjie，你可以把 `namespace_prefix` 中的 `<YOUR_NAME>` 替换为 `hongjie` or `zhhj`。

<kbd>📝 *enablement-ci-cd/inventory/groups_vars/all.yml*</kbd>
```yaml
  namespace_prefix: "<YOUR_NAME>"
```

5. 打开文件 `inventory/host_vars/projects-and-policies.yml`，你将能看到一些已经设置好的变量，它们将用来创建 `<YOUR_NAME>-ci-cd` 命名空间。它将会被传给 OpenShift Applier，并与来自主机清单和 `apply.yml` 剧本中 `ci_cd` 中的变量一起用于调用 `templates/project-requests.yml` 模板。我们后面会在这里继续添加一些内容，不过目前我们先了解一下这些参数和模板。

6. 在文件 `inventory/host_vars/projects-and-policies.yml` 之中，你可以看到如下内容：

<kbd>📝 *enablement-ci-cd/inventory/host_vars/projects-and-policies.yml*</kbd>
```yaml
  ci_cd:
    NAMESPACE: "{{ namespace_prefix }}-ci-cd"
    NAMESPACE_DISPLAY_NAME: "{{ namespace_prefix | title }}s CI/CD"
```

 * 它定义了我们很快在部署 CI/CD 项目时所使用的变量。它依赖上面我们更新过的 `namespace_prefix` 变量。这两组变量组合起来之后，我们现在就可以将它们传给用于创建项目的模板了。你可以注意到，上面的变量的名称（即 `ci_cd`）很快就由主机清单中的 `params_from_vars` 所使用。

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

7. 接下来我们向其中添加两个新的参数字典，它们传给模板之后，就可以用于创建 `dev` 和 `test` 项目了。在文件 `enablement-ci-cd/inventory/host_vars/projects-and-policies.yml` 的顶部，请使用与现有的 `ci_cd` 类似的内容创建新的字典对象 `dev` 和 `test`。

 * 在编辑器中, 打开 `enablement-ci-cd/inventory/host_vars/projects-and-policies.yml` 并在 `openshift_cluster_content` 之前添加如下行:

<kbd>📝 *enablement-ci-cd/inventory/host_vars/projects-and-policies.yml*</kbd>

```yaml
dev:
  NAMESPACE: "{{ namespace_prefix }}-dev"
  NAMESPACE_DISPLAY_NAME: "{{ namespace_prefix | title }} Dev"

test:
  NAMESPACE: "{{ namespace_prefix }}-test"
  NAMESPACE_DISPLAY_NAME: "{{ namespace_prefix | title }} Test"
```

8. 在文件 `enablement-ci-cd/inventory/host_vars/projects-and-policies.yml` 中，针对要创建的新项目（即 dev 和 test），向已经存在的 `content` 数组中分别添加它们对应的对象。可以直接从示例的 `ci_cd_namespace` 复制，然后做一些必要的修改即可。如果你复制，请确保记得将名称设置为 `{{ dev_namespace }}` 和 `{{ test_namespace }}`，并且对应地修改 `params_from_vars`。`ci_cd_namespace`、`dev_namespace` 这些名称中变量的值是在项目根目录的 `apply.yml` 文件中定义的。

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

9. 请使用 `Terminal > Open Terminal in specific container` 菜单项，在 `dev-pod/main` 容器中打开一个控制台。

![open-terminal](../images/exercise1/open-terminal.png)

<p class="tip">
<b>提示</b> - 如果你使用 <b>z shell</b> 作为默认控制台，请执行此命令：
</p>

```
echo "zsh" >> ~/.bashrc
```

10.  切换到 `enablement-ci-cd` 目录

```bash
cd enablement-ci-cd
```

11. 在上述所有配置完成之后，就可以安装 OpenShift Applier 依赖项了

```bash
ansible-galaxy install -r requirements.yml --roles-path=roles
```

12.  在控制台中登录 OpenShift，然后按下面的方式，在主机清单上执行 Ansible 剧本（把其中的 `<CLUSTER_URL>` 替换为讲师指定的值），如果出现了安全警告，请选择接受 👍：

```bash
oc login <CLUSTER_URL>
```
```bash
ansible-playbook apply.yml -i inventory/ -e target=bootstrap
```

其中的 `-e target=bootstrap` 用于传入一个额外变量，它指出我们的剧本应该在分组为 `bootstrap` 的主机清单上运行。

13.  成功之后，你应该能看到类似于这样的输出：

![playbook-success](../images/exercise1/play-book-success.png)

13. 可以通过运行下面的命令来查看成功创建的项目

```bash
oc projects
```

![project-success](../images/exercise1/project-success.png)

### 第 3 部分 - Nexus
> _Now that we have our Projects setup; we can start to populate them with Apps to be used in our dev lifecycle_

For this part, we will use an OpenShift Container Platform **template** to install and configure Nexus. This template contains all the things needed to set up a persistent Nexus server, exposing a service and route while also creating the persistent volume needed. Have a read through the template; at the bottom you'll see a collection of parameters we will pass to the template.

<p class="tip">
<b>NOTE</b> - Below how we are utilizing an OpenShift Container Platform template from a different repository by accessing it by its RAW GitHub URL (from the redhat-cop repo in this case)
</p>

1. In your cloud ide terminal add some parameters for running the template by creating a new file in the `params` directory.
```bash
touch params/nexus
```

2. The essential params to include in this file are:

<kbd>📝 *enablement-ci-cd/params/nexus*</kbd>
```
VOLUME_CAPACITY=5Gi
MEMORY_LIMIT=1Gi
```

* You'll notice that this is different from how we defined our params for our projects. This is because there are multiple ways to do this. In cases like this, there may be a need to change some of these variables more frequently than others (i.e. giving the app more memory,etc.). In this case, it's easier to maintain them within their own separate params files.


3. Create a new object in the inventory variables `inventory/host_vars/ci-cd-tooling.yml` called `ci-cd-tooling` and populate its `content` as follows

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
<b>NOTE</b> The <i>galaxy_requirements</i> above is necessary to pull in the pre/post steps dependencies as explained under the Jenkins section below.
</p>

4. Run the OpenShift applier, specifying the tag `nexus` to speed up its execution (`-e target=tools` is to run the other inventory).
```bash
ansible-playbook apply.yml -e target=tools \
     -i inventory/ \
     -e "filter_tags=nexus"
```

5. Once successful, login to the cluster through the browser (using cluster URL) and navigate to the `<YOUR_NAME>-ci-cd`. You should see Nexus up and running. You can login with default credentials (admin / admin123) ![nexus-up-and-running](../images/exercise1/nexus-up-and-running.png)

### Part 4 - Commit CI/CD

1. Navigate to GitLab login page. You can login using your cluster credentials using the LDAP tab
![gitlab-ui](../images/exercise1/gitlab-ui.png)

2. Once logged in create a new project called `enablement-ci-cd` and mark it as internal. Once created, copy out the `git url` for use on the next step.
![gitlab-new-project](../images/exercise1/gitlab-new-project.png)

3. Commit your local project to this new remote by first removing the existing origin (github) where the Ansible project was cloned from in the first steps. Remember to substitute `<GIT_URL>` accordingly with the one created for your `enablement-ci-cd` repository a moment ago.
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

### Part 5 - MongoDB for CI tests
> _In order to run our API tests in CI in later labs; we need there to be a MongoDB available for executing our tests. As this is part of our CI/CD Lifecycle; we will add it now._

1. Open `enablement-ci-cd` in your favourite editor. Edit the `inventory/host_vars/ci-cd-tooling.yml` to include a new object for our mongodb as shown below. This item can be added below Nexus in the `ci-cd-tooling` section.

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

2. Git commit your updates to the inventory to git for traceability.
```bash
git add .
```
```bash
git commit -m "ADD - mongodb for use in the pipeline"
```
```bash
git push
```

3. Apply this change as done previously using Ansible. The deployment can be validated by going to your `<YOUR_NAME>-ci-cd` namespace and checking if it is there!
```bash
ansible-playbook apply.yml -e target=tools \
  -i inventory/ \
  -e "filter_tags=mongodb"
```
![ocp-mongo](../images/exercise3/ocp-mongo.png)

<p class="tip">
<b>NOTE</b> - When making changes to the "enablement-ci-cd" repo, you should frequently commit and push the changes to git.
</p>

### Part 6 - Jenkins & S2I
> _Create a build and deployment config for Jenkins. Add new configuration and plugins to the OpenShift default Jenkins image using s2i_

1. As before; create a new set of params by creating a `params/jenkins` file and adding some overrides to the template and updating the `<YOUR_NAME>` value accordingly.

<kbd>📝 *enablement-ci-cd/params/jenkins*</kbd>
```
MEMORY_LIMIT=3Gi
VOLUME_CAPACITY=10Gi
JVM_ARCH=x86_64
NAMESPACE=<YOUR_NAME>-ci-cd
JENKINS_OPTS=--sessionTimeout=720
```
  * You might be wondering why we have to replace <YOUR_NAME> here and can't just rely on the `namespace_prefix` variable that we've been using previously. This is because the replacement is handled by two different engines (one being ansible -- which knows about `namespace_prefix` and the other being the oc client, which does not). Because the params files are processed by the oc client, we need to update this here.

2. Add a `jenkins` variable to the Ansible inventory underneath the jenkins-mongo in  `inventory/host_vars/ci-cd-tooling.yml` as shown below to create a DeploymentConfig for Jenkins. In order for Jenkins to be able to run `npm` commands we must configure a jenkins build slave for it to use. This slave will be dynamically provisioned when we run a build. It needs to have Node.js and npm and a C compiler installed in it. 

<p class="tip">
<b>NOTE</b> These slaves can take a time to build themselves so to speed up we have placed the slave with a corresponding ImageStream within OpenShift. To leverage this existing slave image, we are using a feature of the openshift-applier to process a couple of post-steps part of the inventory. These steps are utilized to perform pre and post tasks necessary to make our inventory work correctly. In this case, we use the post steps to tag and label the jenkins-slave-npm ImageStream within our CI/CD project so Jenkins knows how to find and use it.
</p>

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
This configuration, if applied now, will create the deployment configuration needed for Jenkins but the `${NAMESPACE}:${JENKINS_IMAGE_STREAM_TAG}` in the template won't exist yet.

3. To create this image we will take the supported OpenShift Container Platform Jenkins Image and bake in some extra configuration using an [s2i](https://github.com/openshift/source-to-image) builder image. More information on Jenkins s2i is found on the [openshift/jenkins](https://github.com/openshift/jenkins#installing-using-s2i-build) GitHub page. To create an s2i configuration for Jenkins, start with the pre-canned configuration source in the `enablement-ci-cd` repo (in the jenkins-s2i directory).

The structure of the Jenkins s2i config is
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
 * `plugins.txt` is a list of `pluginId:version` for Jenkins to pre-install when starting
 * `./configuration` contains content that is placed in `${JENKINS_HOME}`. A `config.xml` could be placed in here to control the bulk of Jenkins configuration.
 * `build-failure-analyzer.xml` is config for the plugin to read the logs and look for key items based on a Regex. More on this in later lessons.
 * `init.groovy` contains a collection of settings jenkins configures itself with when launching

4. Let's add a plugin for Jenkins to be started with, [green-balls](https://plugins.jenkins.io/greenballs). This simply changes the default `SUCCESS` status of Jenkins from Blue to Green. Append the `jenkins-s2i/plugins.txt` file with
```txt
greenballs:1.15
```
![green-balls.png](../images/exercise1/green-balls.png)

Why does Jenkins use blue to represent success? More can be found [on reddit](https://www.reddit.com/r/programming/comments/4lu6q8/why_does_jenkins_have_blue_balls/) or the [Jenkins blog](https://jenkins.io/blog/2012/03/13/why-does-jenkins-have-blue-balls/).

5. Before building and deploying the Jenkins s2i; add your git credentials to it. These will be used by Jenkins to access the Git Repositories where our apps will be stored. We want Jenkins to be able to push tags to it, so write access is required. Create `params/jenkins-s2i-secret` and add the following content; replacing variables as appropriate. There is an annotation on the secret which binds the credential in Jenkins

<kbd>📝 *enablement-ci-cd/params/jenkins-s2i-secret*</kbd>
```
SECRET_NAME=gitlab-auth
USERNAME=<YOUR_LDAP_USERNAME>
PASSWORD=<YOUR_LDAP_PASSWORD>
```
where
    * `<YOUR_LDAP_USERNAME>` is the username builder pod will use to login and clone the repo with
    * `<YOUR_LDAP_PASSWORD>` is the password the builder pod will use to authenticate and clone the repo using


6. Create `params/jenkins-s2i` and add the following content; replacing variables as appropriate.

<kbd>📝 *enablement-ci-cd/params/jenkins-s2i*</kbd>
```
SOURCE_REPOSITORY_URL=<GIT_URL>
NAME=jenkins
SOURCE_REPOSITORY_CONTEXT_DIR=jenkins-s2i
SOURCE_REPOSITORY_SECRET=gitlab-auth
```
where
    * `<GIT_URL>` is the full clone path of the repo where this project is stored (including the https && .git)


7. At the top of `inventory/host_vars/ci-cd-tooling.yml` file underneath the `---`, add the following:

<kbd>📝 *enablement-ci-cd/inventory/host_vars/ci-cd-tooling.yml*</kbd>
```yaml
ci_cd:
  IMAGE_STREAM_NAMESPACE: "{{ ci_cd_namespace }}"
```

8. Create a new object `ci-cd-builds` in the Ansible `inventory/host_vars/ci-cd-tooling.yml` to drive the s2i build configuration.

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

9. Commit your code to your GitLab instance
```bash
git add .
```
```bash
git commit -m "Adding Jenkins and Jenkins s2i"
```
```bash
git push
```

10. Now your code is commited; run the OpenShift Applier to add the config to the cluster
```bash
ansible-playbook apply.yml -e target=tools \
     -i inventory/ \
     -e "filter_tags=jenkins"
```

11. This will trigger a build of the s2i and when it's complete it will add an imagestream of `<YOUR_NAME>-ci-cd/jenkins:latest` to the project. The Deployment config should kick in and deploy the image once it arrives. You can follow the build of the s2i by going to the OpenShift console's project
![jenkins-s2i-log](../images/exercise1/jenkins-s2i-log.png)

12. When the Jenkins deployment has completed; login (using your OpenShift credentials) and accept the role permissions. You should now see a fairly empty Jenkins with just the seed job

### Part 7 - Jenkins Hello World
> _To test things are working end-to-end; create a hello world job that doesn't do much but proves we can pull code from git and that our builds are green._

1. Log in to Jenkins and hit `New Item`<br>![new-item](../images/exercise1/new-item.png).

2. Create an item called `hello-world` with type `Freestyle project` ![jenkins-new-hello-world](../images/exercise1/jenkins-new-hello-world.png).

3. On the Source Code Management tab, add your `enablement-ci-cd` git repo and hit the dropdown to add your credentials we baked into the s2i on previous steps ![jenkins-scm-git](../images/exercise1/jenkins-scm-git.png)

4. On the build tab add an Execute Shell step and fill it with `echo "Hello World"` ![jenkins-hello-world](../images/exercise1/jenkins-hello-world.png).

5. Run the build and we should see it pass successfully and with a Green ball! ![jenkins-green-balls](../images/exercise1/jenkins-green-balls.png)

### Part 8 - Live, Die, Repeat
> _In this section you will prove the infra as code is working by deleting your Cluster Content and recreating it all_

1. Commit your code to the new repo in GitLab
```bash
git add .
```
```bash
git commit -m "ADD - all ci/cd contents"
```
```bash
git push
```

2. Burn your OpenShift project resources to the ground
```bash
oc delete project <YOUR_NAME>-ci-cd <YOUR_NAME>-dev <YOUR_NAME>-test
```

3. Check to see the projects that were marked for deletion are removed.
```bash
oc get projects | egrep '<YOUR_NAME>-ci-cd|<YOUR_NAME>-dev|<YOUR_NAME>-test'
```

4. Re-apply the inventory to re-create it all!
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

## Extension Tasks
> _Ideas for go-getters. Advanced topic for doers to get on with if they finish early. These will usually not have a solution and are provided for additional scope._

 - Add more secure access for Nexus (ie not admin / admin123) using the automation to drive secret creation
 - Add a SonarQube persistent deployment to the `ci-cd-deployments` section.
 - Add `jenkins.plugins.slack.SlackNotifier.xml` to `jenkins-s2i/configuration` to include URL of Slack for team build notifications and rebuild Jenkins S2I

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
