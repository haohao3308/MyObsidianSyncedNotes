https://www.npmjs.com/package/fen-cli/v/1.0.0
# 从零开发一个Node命令行工具（Cli）

## [](https://www.npmjs.com/package/fen-cli/v/1.0.0#what-%E4%BB%80%E4%B9%88%E6%98%AFcli%E5%B7%A5%E5%85%B7%EF%BC%9F)what? 什么是cli工具？

> 什么是命令行工具？ CLI全称是（Cmmand Line Interface），翻译过来是命令行界面，我理解在命令行界面使用的工具，便是命令行工具，通常也就把cli认为是命令行工具。我们常用的 git 、npm、vim 等都是命令行工具，比如我们可以通过 git clone 等命令简单把远程代码复制到本地。那对应的还有图形用户界面（GUI）使用的工具。那就提出下面的问题了

## [](https://www.npmjs.com/package/fen-cli/v/1.0.0#why%EF%BC%9F%E4%B8%BA%E4%BB%80%E4%B9%88%E9%9C%80%E8%A6%81%E4%BD%BF%E7%94%A8cli%E5%B7%A5%E5%85%B7%EF%BC%9F)why？为什么需要使用cli工具？

1.CLI与GUI相比，更节省计算机资源，在熟记命令的前提下，使用命令行界面往往操作速度要快 2. 对于开发人员而言，加快开发效率，减少人肉操作出现错误； 3. 实现工程化（初始化，开发，测试，部署等都可以通过命令行工具完成）

## [](https://www.npmjs.com/package/fen-cli/v/1.0.0#how%EF%BC%9F%E5%A6%82%E4%BD%95%E5%BC%80%E5%8F%91%E4%B8%80%E4%B8%AAcli%E5%B7%A5%E5%85%B7%EF%BC%9F)how？如何开发一个cli工具？

先从简单开始，到相对复杂

先实现几个小功能： 1.实现控制台运行fen,打印出信息 2.fen -v查看版本信息 3.fen -h获取帮助信息

# 1.创建一个目录： 

mkdir fen-cli && cd fen-cli

# 2.因为最终我们要把cli发布到npm上，所以需要初始化一个程序包:  

npm init

# 3.创建一个index.js文件 

touch index.js

# 4.打开编辑器(vscode) 

code  .

# 5.安装code命令，运行VS code并打开命令面板（ ⇧⌘P ），然后输入shell command 找到: Install 'code' command in PATH 就行了。 

然后我们打开index.js文件，添加一段测试代码：

```
console.log('hello cli!’)
```

使用过node的同学都知道，想在终端运行node程序，需要先输入node命令，比如

node index.js

可以正确输出hello cli!，那问题来了，我们使用cli的时候并没有输入node命令，怎么让终端知道这个文件应该用node程序执行呢，我们可以在index.js的顶部增加一段代码

```
#!/usr/bin/env node
```

这段代码是告诉终端，这个文件要使用node去执行。

一般cli都有一个shell命令，比如git，刚才到code等等，那第二个比较重要的问题，如果你想把当前的项目作为命令包的话，你需要告诉 npm 你的命令脚本文件是哪一个，这里我们需要给 package.json 添加一个 bin 字段，声明一个命令关键字和要执行的文件。比如叫fen。如果想声明多个命令，修改这个字段就好了。

```
# package.json
......
  "bin": {
    "fen": "index.js"
  },
......
```

然后我们测试一下,在终端中输入fen，会提示

```
zsh: command not found: fen
```

为什么会这样呢？我们想一下，通常我们在使用一个cli工具的时候都需要现安装，比如grunt-cli，使用前需要全局安装:

```
npm install grunt-cli -g
```

而我们的fen-cli并没有发布到npm上，当然也没有安装过了，所以终端现在还不认识你的命令，怎么办呢？ 通常我们想本地测试一个npm包，可以使用

```
npm link
```

这个命令，本地安装这个包，我们执行一下，然后再执行

```
fen
```

命令，看正确输出hello cli!了。

到此，一个简单的命令行工具就完成了，但是这个工具并没有任何实际用处，下面我们来一点一点增加他的功能。

首先是查看cli的版本信息，我们希望通过如下命令来查看版本信息：

```
fen -v
```

这里有两个问题

1.  如何获取-v这参数？
2.  如何获取版本信息？

在node程序中，通过process.argv可获取到命令的参数，返回的是一个数组,我们修改index.js,输出这个数组：

```
console.log(process.argv)
```

然后输入任意命令，比如：

fen -v -h -lalala

控制台会输出

```
[ '/Users/shaolong/.nvm/versions/node/v8.9.0/bin/node',
  '/Users/shaolong/.nvm/versions/node/v8.9.0/bin/fen',
  '-v',
  '-h',
  '-lalala' ]
```

这个数组的第三个参数就是我们想要的-v。 第二个问题，版本信息一般是放在package.json文件的version字段中,require进来就好了，改造后的index.js代码如下：

```
#!/usr/bin/env node
const pkg = require('./package.json')
const command = process.argv[2]

switch (command) {
    case '-v':
    console.log(pkg.version)
    break
    default:
    break
}
```

然后我们再执行fen -v，就可以输出版本号了。

来，尝试fen -h 打印一个大佛～

接下来我们来实现一个最常见的功能，利用cli工具初始化一个项目。

整个流程大概是这样的：

1.  cd到一个你想新建项目的目录；
2.  执行 fen init 命令，根据提示输入项目名称；
3.  cli通过git拉取模版项目代码，并拷贝到项目名称所在目录中；

为了实现这个流程，我们需要解决下面几个问题：

### [](https://www.npmjs.com/package/fen-cli/v/1.0.0#%E5%91%BD%E4%BB%A4%E5%8F%82%E6%95%B0)命令参数

上面的演示中，我们通过process.argv获取到了命令的参数，但是当一个命令有多个参数，或者像新建项目这种需要用户二次输入项目名称（我们称作“问答”）的命令时，一个简单的swith case显然不能满足我们的需求。这里我们引用一个专门处理命令行交互的包：commander。

npm install commander --save

然后改造index.js

```
#!/usr/bin/env node

const program = require('commander')

program.version(require('./package.json').version)
program.parse(process.argv)
```

运行

fen -h

会输出

Usage: fen [options] [command]

Options:

  -V, --version  output the version number

  -h, --help     output usage information

commander已经为我们创建好了帮助信息，以及两个参数-V和-h，上面代码中的program.version就是返回版本号，和之前的功能一致，program.parse是将命令参数传入commander管道中，一般放在最后执行。

接下来我们添加init的问答操作，这里有需要引入一个新的包:inquirer, 这个包可以通过配置让cli支持问答交互。

npm install inquirer --save

index.js

```
#!/usr/bin/env node

const program = require('commander')
var inquirer = require('inquirer')

const initAction = () => {
    inquirer.prompt([{
        type: 'input',
        message: '请输入项目名称:',
        name: 'name'
    }]).then(answers => {
        console.log('项目名为：', answers.name)
        console.log('正在拷贝项目，请稍等')
    })
}

program.version(require('./package.json').version)

program
    .command('init')
    .description('创建项目')
    .action(initAction)

program.parse(process.argv)
```

program.command可以定义一个命令，description添加一个描述，在--help中展示，action指定一个回调函数执行命令。inquirer.prompt可以接收一组问答对象，type字段问答类型，name指定答案的名称，然后可以在answers里通过name拿到用户的输入，问答的类型有很多种，这里我们使用input，让用户输入项目名称。

运行 fen init，然后会提示输入项目名称，输入后会打印出来。

### [](https://www.npmjs.com/package/fen-cli/v/1.0.0#%E5%88%9B%E5%BB%BA%E7%9B%AE%E5%BD%95%E5%B9%B6clone%E4%BB%A3%E7%A0%81)创建目录并clone代码

熟悉git和linux的同学几句话便可以完成这个流程：

git clone xxxxx.git --depth=1 #只克隆最近一次commit，防止仓库太大 

mv xxxxx my-project #将文件改名字 

rm -rf ./my-project/.git #移除项目下git配置，以便完全作为自己的新项目使用 

cd my-project

npm install

那有没有办在node程序中执行shell脚本呢？答案是肯定。只需要安装shelljs这个包就可以轻松搞定。

npm install shelljs --save

假定我们想克隆github上vue-admin-template这个项目的代码，并自动安装依赖，改造index.js， 再initAction函数中加上执行shell脚本的逻辑：

```
#!/usr/bin/env node

const program = require('commander')
const inquirer = require('inquirer')
const shell = require('shelljs')

const initAction = () => {
    inquirer.prompt([{
        type: 'input',
        message: '请输入项目名称:',
        name: 'name'
    }]).then(answers => {
        console.log('项目名为：', answers.name)
        console.log('正在拷贝项目，请稍等')
        
        const remote = 'https://github.com/PanJiaChen/vue-admin-template.git'
        const curName = 'vue-admin-template'
        const tarName = answers.name

        shell.exec(`
                git clone ${remote} --depth=1
                mv ${curName} ${tarName}
                rm -rf ./${tarName}/.git
                cd ${tarName}
                npm install
              `, (error, stdout, stderr) => {
            if (error) {
                console.error(`exec error: ${error}`)
                return
            }
            console.log(`${stdout}`)
            console.log(`${stderr}`)
        });
    })
}

program.version(require('./package.json').version)

program
    .command('init')
    .description('创建项目')
    .action(initAction)

program.parse(process.argv)
```

shell.exec可以帮助我们执行一段脚本，在回调函数中可以输出脚本执行的结果。

测试一下我们初始化功能：

cd ..

fen init

# 输入一个项目名称 

可以看到，cli已经自动从github上拉取vue-admin-template的代码，放在指定目录，并帮我们自动安装了依赖。

最后我们可以将这个cl包发布到npm上

npm publish

到此，本次分享就结束了，概括来说，node命令行工具只不过是包装了一下的node程序，这次分享只是抛砖引玉，想实现一个功能强大的cli工具，除了要熟悉node和文件操作，还要了解一些linux的常用命令，希望大家可以受到启发，开发自己的cli工具。

## Keywords
