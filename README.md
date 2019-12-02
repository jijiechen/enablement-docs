# DevOps 文化和实践

![jenkins-crio-ocp-star-wars-kubes](./images/jenkins-crio-ocp-star-wars-kubes.png)
[image-ref](https://www.cloudbees.com/blog/get-ripped-jenkins-docker-industrial-strength-continuous-delivery)

## 学员收益
* 在现代化软件开发实践中沉浸式体验 DevOps 文化的实战式运用
* 让学员在 DevOps 之路的开端就感受到团队文化的变化

## 集群信息

完成动手练习过程需要一个 OpenShift 集群。讲师将通过电子邮件和现场讲解为学员提供以下关于 OpenShift 集群的信息：
 - <**CLUSTER_URL**> -- OpenShift Web 控制台及 API 服务器的 URL
 - <**APPS_URL**> -- 在集群上部署的应用使用的通用域名

## 学员准备
下面列举了在课堂上执行练习所需要的最小硬件需求：
* 64 位 Intel i5/i7 或同等性能 CPU，启用虚拟化扩展功能
* 4GB 内存
* 80GB 硬盘

动手练习的过程通过过如下环境测试：
 * Fedora 29 or 30 64-bit x86_64
 * Microsoft Windows 10 Pro 64-bit
 * macOS 10.14 "Mojave"

下面的表格列举了运行动手练习环境所需要的软件需求：

 | 软件 | 版本 | 检查方法 |
 | -------- | ------- | ----- |
 | OCP 命令行客户端 | v3.11 | $ oc version &#124; grep -i --color oc |
 | Ansible | => v2.8 | $ ansible --version &#124; grep -i --color ansible |
 | Node.js | v8.x LTS | $ node -v |
 | 安装 Git | | $ git --version |
 | Google Chrome 浏览器 | (>60) | 如果 Google Chrome 是你的默认浏览器，请点击 [此处](chrome://version/)，否则请复制链接 `chrome://version/` 到 Chome 浏览器中打开 |
 | 最新版的 Docker | 社区版 | $ docker --version |
 | JDK | v1.8 | $ java -version |
 | 访问 OpenShift 集群 | | `oc login -u <username> -p <password> <CLUSTER_URL>` |
 | 文本编辑器，比如 Atom，或者 Visual Studio Code <br><br> （练习内容是以 `VSCode` 创建的，所以课件中的截图会与它相匹配）| - | - |


 <p class="tip">
 <b>提示</b> - 你需要在你的系统中有管理员或超级用户访问权限，以安装所依赖的软件。不支持使用被锁定或权限不足的账号。
 </p>

### Linux

1. Enable the nodejs 8 modules in Fedora and install the latest nodejs 8.x LTS release.
```bash
dnf module enable nodejs:8
dnf module install -y nodejs:8
```
or use the [node-version-manager](https://github.com/nvm-sh/nvm#install--update-script) to control your node versions across multiple projects.


2. Install OpenJDK version 1.8.
```bash
dnf install java-1.8.0-openjdk-devel
```

3. Install Chromium version 70 or higher.
```bash
dnf install chromium
```

4. Install Docker, Git and Ansible.
```bash
dnf install git ansible docker
systemctl enable docker
systemctl start docker
```

5. enable non-root user access to docker
```bash
groupadd docker
systemctl restart docker
usermod -aG docker $USER
```

6. Install the OpenShift 3.11 client binary.
```bash
dnf install origin-clients
```

### macOS
1. Install HomeBrew for macOS by following the installation instructions at https://brew.sh/

2. Install Node.js version 8.x LTS using the brew command, and follow the instructions to add the **node** binary to the **PATH** environment variable.
```bash
brew install node@8
```
or use the [node-version-manager](https://github.com/nvm-sh/nvm#install--update-script) to control your node versions across multiple projects.

3. Install JDK version 1.8 for MacOS by using the installer from the Oracle website at https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html

4. Install Google Chrome version 70 or higher by downloading it from https://google.com/chrome

5. Install Docker for Mac by following the instructions from https://store.docker.com/editions/community/docker-ce-desktop-mac

6. Install Git using brew
```bash
brew install git
```

7. You need Ansible version 2.8 or greater to run the exercise playbooks. If you have not installed Ansible, or have an older version, install or upgrade it using brew
```bash
brew install ansible
```

8. Download and uncompress the OpenShift 3.11 client binary archive. Copy the ***oc*** binary to ***/usr/local/bin*** folder on your system
```bash
curl -L -O https://github.com/openshift/origin/releases/download/v3.11.0/openshift-origin-client-tools-v3.11.0-0cbc58b-mac.zip
unzip openshift-origin-client-tools-v3.11.0-0cbc58b-mac.zip
cp oc /usr/local/bin/
chmod +x /usr/local/bin/oc
```

### Microsoft Windows

You will be adding several directories to the ***PATH*** environment variable. Follow the instructions at https://www.architectryan.com/2018/03/17/add-to-the-path-on-windows-10/ to edit the ***PATH*** environment variable on Windows 10.

1. Download the Node.js 8.x LTS Windows 64-bit binary archive file from https://nodejs.org/dist/latest-v8.x/. The binary will be named as ***node-v8.x.y-win-x64.zip***, where 'x' and 'y' indicates the major and minor version of the latest Node.js 8 LTS release. 

2. Extract the zip file archive under a suitable folder in the ***C:\*** drive, for example ***C:\nodejs***. Make sure that your directory name does not have any spaces in it.

3. Add the directory where you uncompressed the zip file to the ***PATH*** environment variable, so that the ***node.exe*** and ***npm.cmd*** executable files are available in the system path.

4. Install JDK version 1.8 for Windows 64-bit by using the installer from the Oracle website at https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html

Ensure that you install the JDK into a directory which has no spaces in its name, for example ***C:\Java***. Add a new environment variable called ***JAVA_HOME*** which points to the top level directory where you installed the JDK.

5. Add the ***JAVA_HOME\bin*** directory to the ***PATH*** environment variable, so that the ***java.exe*** and ***javac.exe*** executable files are available in the system path.

6. Install Google Chrome version 70 or higher by downloading it from https://google.com/chrome

7. Install Docker for Windows by following the instructions from https://hub.docker.com/editions/community/docker-ce-desktop-windows. Before downloading, you will be asked to create an account. Register and download Docker for Windows.

Ensure that you install Docker into a directory which has no spaces in its name, for example ***C:\Docker***. You will be prompted to enable Hyper-V during installation.

Windows will restart a couple of times after enabling Hyper-V and continue with the Docker installation. Follow the instructions provided in the link above to test and verify your Docker installation.

7. Download and install Git for Windows by using the 64-bit installer from https://github.com/git-for-windows/git/releases/download/v2.20.0.windows.1/Git-2.20.0-64-bit.exe.

8. Follow the instructions at https://www.atlassian.com/git/tutorials/install-git#windows to install and verify your Git installation. Ensure that you install Git into a directory which has no spaces in its name, for example ***C:\Git***.

<p class="tip">
<b>NOTE</b> - Ensure that the <i>Git Bash</i> component is selected during installation. You will need this to run bash shell scripts during the execution of lab exercises.
</p>

9. Download and install Atom text editor from https://github.com/atom/atom/releases/download/v1.33.0/AtomSetup-x64.exe, or Visual Studio Code from https://code.visualstudio.com/docs/?dv=win64

10. You will use a custom container image for running OpenShift client commands, and Ansible playbooks. You will map a directory on your local Windows system containing Ansible playbooks to a directory inside the container, and run the Ansible playbooks from within the container.

11. Execute the following commands in the ***Windows Command Prompt*** to run Ansible playbooks on Windows systems:

* Pull the container image containing the tools and utilities that are required for running Ansible playbooks:
```bash
docker pull quay.io/redhat/do500-toolbox
```
* Create a working directory under your ***C:\*** drive to store the lab exercise files and Ansible playbooks
```bash
mkdir C:\do500-workspace
```

* Launch the container and run a quick test:
```bash
docker run -it -v C:/do500-workspace:/home/tool-box/workarea:Z quay.io/redhat/do500-toolbox /bin/bash
```
<p class="tip">
<b>提示</b> - When you launch the container for the very first time, you will be prompted by Docker, as well as the Windows security subsystem to allow read and write access to the ***C:\do500-workspace*** directory. Allow read and write access to this directory by entering your Windows username and password when prompted.
</p>

* Once you are inside the container, you can log in to the OpenShift cluster using the OpenShift ***oc*** command-line client, and verify that Ansible is installed:
```bash
bash-4.4$ oc login -u <username> -p <password> <CLUSTER_URL>
bash-4.4$ oc version
bash-4.4$ ansible --version
bash-4.4$ exit
```

Your instructor will provide the ***username*** and ***password*** information for the OpenShift cluster when you are ready to run the lab exercises.

## Git 与容器基础
 - Git tutorial covering the basics - https://try.github.io/
 - Handy guide for those new to containers - https://developers.redhat.com/blog/2018/02/22/container-terminology-practical-introduction/

## 设置 IDE
The following plug-ins are useful for providing syntax highlighting for various lab files:
 - YAML Syntax Highlighter
 - Autosave
 - JavaScript Syntax Highlighter
 - Vue.js
 - ESlint
 - Jenkinsfile
