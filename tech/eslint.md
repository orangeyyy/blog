# ESLint

## 概述
一个检查（linting）工具可以帮助我们规范代码，高效地进行团队合作，更能够避免编写javascript过程中出现一些愚蠢的错误。现在可以用的js检查器很多，目前最流行的有[JSLint](http://www.jslint.com/)、[JSHint](http://jshint.com/)、[JSCS](http://jscs.info/)和[ESLint](https://eslint.org/),这里不对工具做比较，如果想了解各个检查工具的优缺点可以参考[js检查工具比较](http://www.css88.com/archives/7593)。这里重点介绍ESLint,因为它是最新的，而且对ES6及JSX等现在流行的语法支持得最好的，而且他并不自己定义编码风格，规则是自由的和可插拔的。

ESLint最初是由Nicholas C. Zakas 于2013年6月创建的开源项目。它的目标是提供一个插件化的javascript代码检测工具。

## 安装
安装ESLint
```sh
npm install eslint -g
```
安装完成以后可以通过下面的命令来生成一个配置文件.eslintrc，在生成配置文件过程中会有一系列的向导来引导生成配置文件。
```sh
eslint --init
```
我们可以通过```eslint -h```查看如何使用ESLint命令行工具，不过我们一般不直接使用命令行工具，大多数情况下我们是借助IDE的eslint插件或者类似gulp的构建工具来使用ESLint。

## 配置
配置是ESLint使用的关键，它告诉ESLint该用哪些规则来检查我们的代码，ESLint被设计成完全可配置，这意味着我们可以关闭每一个规则，只运行基本的语法验证，ESLint主要提供两种方式进行配置：
* **Configuration Comments**： 使用js注释的方式直接嵌入到代码中；
* **Configuration Files**： 使用js，json或者YAML文件为整个项目制定配置信息，可以用eslintrc.*文件或者在package.json文件中的eslintConfig字段进行配置，ESLint为查找并自动读取它们。

ESLint配置文件支持以下格式：
* JavaScript： .eslintrc.js;
* YAML: .eslintrc.yaml或者 .eslintrc.yml;
* JSON: .eslintrc.json;
* Deprecated: .eslintrc,可以是json也可以是YAML；
* package.json: package.json 中的eslintConfig字段；
> PS: 如果同一个目录下有多个配置文件，查找优先级如上面的顺序从上到下；

> PS: ESLint查找配置文件的顺序类似node_modules的查找方式，从当前目录一直往上直到根目录，不过搜索路径上的配置会进行叠加，距离当前文件最近的配置，优先级最高。

>PS: 主目录下的自定义配置文件(~/.eslintrc)，如果没有其他配置文件时才会生效。

默认情况下，ESLint会在所有父级目录中寻找配置文件，一直找到根目录，不过为了将ESLint限制在一个特定的项目，我们可以在配置文件中增加"root"字段来阻止继续向上级目录寻找。
```json
{
 "root": true
}
```


ESLint支持以下配置信息：
* **Parser & Parser Options**: 配置解析器及解析器相关配置；
* **Environments**： 制定脚本的运行环境，每种环境都有一组特定的预定义全局变量；
* **Globals**：制定脚本执行期间访问的额外的全局变量；
* **Rules**： 启用的规则及各自的错误级别；



### Parser & Parser Options
ESLint 默认使用Espree作为解析器，用户可以在配置文件中指定其他解析器，例如Babel-ESLint，解析器必须符合以下要求：
* 必须是本地安装的npm模块；
* 必须兼容Esprima的接口（必须输出一个parse()方法）；
* 必须产出兼容Esprima的AST 和 token对象；
可以按照如下方式指定解析器：
```json
{
  "parser": "esprima",
  "rules": {
    "semi": "error"
  }
}
```
常用的解析器有：
* **Esprima**
* **Babel-ESLint**：对babel解析器的包装，使其兼容ESLint；
* **typescript-eslint-parser**：一个把TypeScript转换成兼容ESTree的解析器，使它可以在ESLint中使用，这样可以允许通过ESLint来解析TypeScript文件（尽管不一定通过所有ESLint规则）；

在配置文件中，使用**parserOptions**属性来设置解析器选项,可用的选项有：
* **ecmaVersion**:设置为3，5（默认），6，7或8来指定使用的ECMAScript版本；
* **sourceType**:设置为“script”（默认），“module”（如果代码是ECMAScript 模块）；
* **ecmaFeatures**：这个属性是个对象，用于指定额外的语言特性；
  * **globalReturn**：运行在全局作用域下使用return语句；
  * **impliedStrict**：启用全局strict mode；
  * **jsx**：启用jsx；
  * **experimentalObjectRestSpread**:启用对实验性的object rest、spread properties的支持，实验性功能一般不适用；

文件实例：
```json
{
  "parserOptions": {
    "ecmaVersion": 6,
    "sourceType": "module",
    "ecmaFeatures":{
      "jsx": true
    }
  }
  "rules": {
    "semi": 2
  }
}

```

> 在使用自定义解析器的时候，为了使ESLint在非ES5的特性下工作，配置parseOptions仍然是必须的。

ESLint可以让你指定你想支持的js语言选项，默认情况下ESLint支持ES5语法，你可以覆盖其设置指定启用其他ES版本或者JSX支持。

> 对JSX的支持不用于对React的支持，如果要使用react，需要使用eslint-plugin-react。

>支持ES6语法不代表支持ESLint全局变量或类型（例如：Set）因此，对于ES6语法，使用：
```json
{
  "parserOptions": {
    "ecmaVersion": 6
  }
}
```
对于新的ES6变量需要使用：
```json
{
  "env": {
    "es6": true
  }
}
```

### Evironments
环境定义了预定义的全局变量，常用的环境有：
* browser：browser全局变量；
* node：Node.js全局变量和作用域；
* commonjs：commonjs全局变量及作用域（仅为使用browserify和webpack写的只支持浏览器的代码）
* es6：支持除模块外的所有ES6特性（该选项会自动设置ecmaVersion为6）；
* amd：定义require()和define()为全局变量；

更多环境设置详见[官方文档](http://eslint.cn/docs/user-guide/configuring)。

使用实例：
```json
{
  "env": {
    "browser": true,
    "node": true
  }
}
```
> 如果像在一个插件中使用一种环境，首先要确保在plugins字段里面指定插件，然后在env字段中按照```插件名/环境名```的形式指定环境：
```json
{
  "plugins": [
    "example"
  ]
  "env": {
    "example/browser": true
  }
}

```
### Globals
当访问未定义变量时，no-undef规则会发出警告，如果想在文件中使用全局变量，需要设置全局变量，这样ESLint就不会告警了，我们可以在js注释中定义，也可以在配置文件中定义：
```javascript
/* global val1, var2*/

//如果希望上面设置的两个变量只读，可以将他们设置为false

/*global var1:false, var2:false*/
```
同样可以在配置文件中设置：
```json
{
  "globals": {
    "var1": true,
    "var2": false
  }
}

```
>PS: 可以启用no-global-assign规则来禁止对只读全局变量进行修改。

### Plugins
ESLint支持使用第三方插件，插件是npm包，通常输出规则，使用插件之前必须npm安装，插件包名一般为：eslint-plugin-xxx，在配置文件中进行配置的时候一般省略eslint-plugin-。
```json
{
  "plugins": [
    "plugin1",
    "eslint-plugin-plugin1"
  ]
}
```
>PS: 全局安装的ESLint只能使用全局安装的插件，本地安装的ESLint既可以使用本地插件也可以使用全局插件。

### Rules
规则配置是ESLint配置的核心，ESLint附带了大量的规则，我们可以通过js注释或者配置文件的方式进行配置。

规则的值可以为：
* “off” 或者 0 - 关闭规则；
* “warn” 或者1 - 开启规则，使用警告级别的错误（不导致程序退出）；
* “error” 或者2 - 开启规则，使用错误级别的错误（当触发时程序会退出）；

在注释中使用的格式为：
```javascript
/* eslint eqeqeq: "off", curly: "error" */
/* eslint eqeqeq: 0, curly: 2 */

//如果规则中有额外的选项，可以用数组的形式,数组的第一项为错误级别，是必须有的。
/* eslint quotes: ["error", "double"] */

//插件中定义的规则
/* eslint "plugin1/rule1": "error" */

```
在配置文件中的格式为：
```json
{
  "rules": {
    "eqeqeq": "off",
    "quotes": ["error", "double"],
    "plugin1/rule1": "error"
  }
}
```
> PS: 设置定义在插件中的规则时，必须使用```插件名/规则ID的形式```。

### 临时禁止规则出现警告
我们可以通过在代码中插入注释的方式来临时禁止ESLint的规则出现警告：
```js
/* eslint-disable */
alert('hello');
/* eslint-enable */

//也可指定禁止指定规则出现警告
/* eslint-disable no-alert, no-console */
alert('hello');
console.log('hello');
/* eslint-enable no-alert, no-console */
```
如果将eslint-disable放在文件顶部则对整个文件起作用。
同时也可以对一行指定的代码禁用规则：
```js
alert('hello') //eslint-disable-line

// eslint-disable-next-line
alert('hello')

//也可以指定禁止指定规则
alert('hello') //eslint-disable-line no-alert

// eslint-disable-next-line no-alert
```

> PS: 为文件的某一部分禁用警告的注释，告诉ESLint不要对禁用的代码报告规则的问题，ESLint仍然会解析整个文件，所以，禁用的代码仍然需要时有效的js语法。

### 共享配置
ESLint支持在配置文件中设置共享配置，它将提供给每一个被执行的规则，我们可以通过settings字段来进行设置。
```json
{
  "settings": {
    "shareData": "hello"
  }
}
```

### 配置扩展
一个配置文件可以基于已有的基础配置文件进行扩展，这时候就用到extends字段，基础配置文件也是npm包，名称规范为eslint-config-xxx，配置的时候往往省略eslint-config-。
extends属性值可以是：
* 一个字符串；
* 一个字符串数组，每个配置继承它前面的配置；

rules里面可以做以下事情来扩展或覆盖继承规则：
* 启用额外规则；
* 改变继承的规则级别而不改变其他选项：
  * 基本配置： "eqeqeq": ["error", "allow-null"];
  * 派生配置: "eqeqeq": "warn";
  * 最后配置："eqeqeq": ["warn", "allow-null];
* 覆盖基本配置中的规则选项：
  * 基本配置："quotes": ["error", "single", "avoid-escape"];
  * 派生配置: "quotes": ["error", "single"];
  * 最后配置: "quotes": ["error", "single"];

使用实例：
```json
{
  "extends": [
    "eslint:recommended",
    "plugin:react/recommended"
  ]
}
```

> PS: extends字段也可以是配置文件的绝对或相对路径。

### .eslintignore
我们可以在项目根目录下创建一个.eslintignore文件告诉ESLint忽略特定的文件和目录，类似.gitignore。

## 参考
* http://eslint.cn/
* http://www.css88.com/archives/7600
* http://www.css88.com/archives/7593
* http://www.jianshu.com/p/2bcdce1dc8d4
* https://www.cnblogs.com/ruanyifeng/p/5283708.html