# Lerna 入门教程

Lerna 是一个管理工具，用于管理包含多个软件包（package）的 JavaScript 项目。

## 背景
对于维护过多个package的同学来说，都会遇到一个选择：这些package是放在一个仓库里维护还是放在多个仓库里单独维护，数量较少的时候，多个仓库维护不会有太大问题，但是当package数量逐渐增多时，一些问题逐渐暴露出来：

* 一、package之间相互依赖，开发人员需要在本地手动执行npm link，维护版本号的更替；
* 二、issue难以统一追踪，管理，因为其分散在独立的repo里；
* 三、每一个package都包含独立的node_modules，而且大部分都包含babel,webpack等开发时依赖，安装耗时冗余并且占用过多空间。

## 介绍
lerna到底是什么呢？lerna官网上是这样描述的。
用于管理具有多个包的JavaScript项目的工具。

这个介绍可以说很清晰了，引入lerna后，上面提到的问题不仅迎刃而解，更为开发人员提供了一种管理多packages javascript项目的方式。

* 一、自动解决packages之间的依赖关系。
* 二、通过git 检测文件改动，自动发布。
* 三、根据git 提交记录，自动生成CHANGELOG

## 模式

### Fixed/Locked mode (default)  

vue,babel都是用这种，在publish的时候,会在lerna.json文件里面"version": "0.1.5",,依据这个号，进行增加，只选择一次，其他有改动的包自动更新版本号。

### Independent mode

初始化项目
```
lerna init --independent
```
lerna.json文件里面"version": "independent"

每次publish时，您都将得到一个提示符，提示每个已更改的包，以指定是补丁、次要更改、主要更改还是自定义更改。

## 开始

```
    $ npm install lerna -g
    $ mkdir lerna-demo && cd $_
    $ npm lerna init # 用的默认的固定模式，vue babel等都是这个
    
     # Add packages
    $ cd packages
    $ mkdir house window
    ...
    #分别进入两个目录初始化成包
    $ cd house
    $ npm init -y 
    $ cd ../window
    $ npm init -y

```

## 项目结构

```
➜  lerna-gp git:(master) ✗ tree
.
├── lerna.json
├── package.json
└── packages
    ├── house
    │   └── package.json
    ├── window
        └── package.json


4 directories, 5 files
```

## 设置

### git + npm
```
✗ git remote add origin git@github.com:XXX/XXX.git

#查看是否登录
✗ npm whoami
gp0320

#没有则登录 
npm login 
# 输入username password 
Logged in as XXX on https://registry.npmjs.org/. # succeed
```

### 设置 yarn的workspaces模式

>默认是npm, 而且每个子package都有自己的node_modules，通过这样设置后，只有顶层有一个node_modules

修改顶层 package.json and lerna.json

```
# package.json 文件加入
 "private": true, // 这个可加可不加，按需求来，应该是私有包的意思
  "workspaces": [
    "packages/*"
  ],

# lerna.json 文件加入
"useWorkspaces": true,
"npmClient": "yarn",
```

## 语法学习

### lerna create <name> [loc]

> 创建一个包，name包名，loc 位置可选
Examples
```
# 根目录的package.json 
 "workspaces": [
    "packages/*",
    "packages/XXX/*"
  ],
  
# 创建一个包gpnote默认放在 workspaces[0]所指位置
lerna create gpnote 

# 创建一个包gpnote指定放在 packages/XXX文件夹下，注意必须在workspaces先写入packages/XXX

lerna create gpnote packages/XXX
```

### lerna add <package>[@version] [--dev] [--exact]
> 增加本地或者远程package做为当前项目packages里面的依赖
* --dev devDependencies 替代 dependencies
* --exact 安装准确版本，就是安装的包版本前面不带^, Eg: "^2.20.0" ➜ "2.20.0"

```
# 将module-1包添加到以“prefix-”为前缀的文件夹中的包中
lerna add module-1 packages/prefix-*

# 安装模块1到模块2
lerna add module-1 --scope=module-2

# 安装模块1到模块2 开发依赖
lerna add module-1 --scope=module-2 --dev

# 在除module-1外的所有模块中安装module-1
lerna add module-1

# 在所有模块中安装babel-core
lerna add babel-core
```

### lerna bootstrap
默认是npm i,因为我们指定过yarn，so,run yarn install,会把所有包的依赖安装到根node_modules.

### lerna list
列出所有的包，如果与你文夹里面的不符，进入那个包运行yarn init -y解决

```
➜  lerna-gp git:(master) ✗ lerna list
lerna notice cli v3.14.1
house
window
lerna success found 2 packages
```

### lerna import <path-to-external-repository>
导入本地已经存在的包

### lerna run
```
lerna run < script > -- [..args] # 运行所有包里面的有这个script的命令
$ lerna run --scope my-component test
```

### lerna exec
运行任意命令在每个包
```
$ lerna exec -- < command > [..args] # runs the command in all packages
$ lerna exec -- rm -rf ./node_modules
$ lerna exec -- protractor conf.js
lerna exec --scope my-component -- ls -la
```
该命令手动将缺少的版本发布到npm已经正确发布的版本上，并以静默方式失败：
```
lerna exec -- "npm publish || exit 0"
```

### lerna link
项目包建立软链，类似npm link

### lerna clean
删除所有包的node_modules目录

### lerna changed

列出下次发版lerna publish 要更新的包。

原理：
需要先git add,git commit 提交。
然后内部会运行git diff --name-only v版本号 ，搜集改动的包，就是下次要发布的。并不是网上人说的所有包都是同一个版全发布。
```
➜  lerna-repo git:(master) ✗ lerna changed                                     
info cli using local version of lerna
lerna notice cli v3.14.1
lerna info Looking for changed packages since v0.1.4
daybyday #只改过这一个 那下次publish将只上传这一个
lerna success found 1 package ready to publish

```

### lerna publish

重点注意遇到403错误请重新登录npm
```
npm login
```
会打tag，上传git,上传npm。
如果你的包名是带scope的例如："name": "@gp0320/gpwebpack",
那需要在packages.json添加
```
"publishConfig": {
    "access": "public"
  },
```
```
lerna publish 
lerna info current version 0.1.4
#这句意思是查找从v0.1.4到现在改动过的包
lerna info Looking for changed packages since v0.1.4 

? Select a new version (currently 0.1.4) Patch (0.1.5)

Changes:
 - daybyday: 0.1.3 => 0.1.5 #只改动过一个

...

Successfully published:
 - daybyday@0.1.5
lerna success published 1 package
```

## lerna.json解析
```
{
  "version": "1.1.3",
  "npmClient": "npm",
  "command": {
    "publish": {
      "ignoreChanges": [
        "ignored-file",
        "*.md"
      ]
    },
    "bootstrap": {
      "ignore": "component-*",
      "npmClientArgs": ["--no-package-lock"]      
    }
  },
  "packages": ["packages/*"]
}
```
> version：当前库的版本  
npmClient： 允许指定命令使用的client， 默认是 npm， 可以设置成 yarn  
command.publish.ignoreChanges：可以指定那些目录或者文件的变更不会被publish  
command.bootstrap.ignore：指定不受 bootstrap 命令影响的包  
command.bootstrap.npmClientArgs：指定默认传给 lerna bootstrap 命令的参数  
command.bootstrap.scope：指定那些包会受 lerna bootstrap 命令影响  
packages：指定包所在的目录  

## 最佳实践 toDo
```
采用Independent模式
根据Git提交信息，自动生成changelog
eslint规则检查
prettier自动格式化代码
提交代码，代码检查hook
遵循semver版本规范
```
大家应该也可以看出来，在开发这种工程的过程的，最为重要的一点就是规范。

因为应用场景各种各样，你必须保证发布的packge是规范的，代码是规范的，一切都是有迹可循的。这点我认为是非常重要的。