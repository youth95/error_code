# 业务码自动生成器

## 故事

1. 作为开发者,我希望通过异常码提高研发开发速度。
2. 作为后端开发者,我希望能够通过输入少量的信息得到内容足够丰富的异常码。
3. 作为后端开发者,我希望异常码是可读、可分析的。
4. 作为后端开发者，我希望异常码能够包含一个表示以下信息的元组（服务，级别，处理方案，复杂度，是否为主流程）
5. 作为后端开发者，我希望能够通过异常码快速定位到问题，即异常码信息可以表达(where、who、what happend)

6. 作为前端开发者,我希望同一个项目中的所有服务所报告的异常码是唯一的。以便前端统一提示。
7. 作为前端开发者，我希望所有的异常信息都具备国际化。

## 方案

为了满足以上的用户故事,此项目给出以下两个个解决方案。

* 提供生成脚本`gen`生成一个内容丰富的异常码，以满足故事 1，2，3，4，6，7
* 通过定义规范以满足故事5

## gen

### 安装

下载并安装最新的deno，并在自己的`.bashrc`中配置命令行

```bash
alias gen='deno run https://raw.githubusercontent.com/youth95/error_code/gen.ts'
```

### 使用

#### Step 1

切换到一个后端项目的工作目录,执行`gen --init`,随后你将得到一个名为`.error_code_gen_rule.json`的配置文件。其内容如下

```json
{
  "service_name":"service name",
  "service_prex":"ERRO",
  "default_level":1,
  "default_complexity":1,
  "default_main_flow_flag":true,
  "default_message_and_tip":"未知异常",
  "error_code_url":"http://localhost/error_code_dir",
}
```

其中的字段含义如下

* **service_name** 服务名称，生成的时候会取当前的工作目录名作为服务名称。
* **service_prex** 服务的默认前缀，若长度不足4位,则在错误码生成过程中使用`_`向后补齐。若超过4位则取前4位。
* **default_level** 默认等级。
* **default_complexity** 默认复杂度。
* **default_main_flow_flag** 默认的主流程开关。
* **default_message_and_tip** 默认的提示信息。
* **error_code_url** 用于`lint`工具检查当前服务定义的错误码。
  
#### Step 2

使用命令`gen`,将得到一个错误码。

```bash
gen
ERRO1101-ae189db1-0276-42d0-8c31-766b6c0b8ec2 未知异常
```



错误码的结构如下。

1. **0-4位** 服务前缀
2. **5位** 是否为主流程的表示,支持0-1
3. **6位** 异常码等级,支持0-9
4. **7-8位** 异常复杂度,支持0-99
5. **9位** 分割符
6. **10-46位** uuid

同时，你将得到一个`.error_code.json`的文件在你的项目根目录中.

```json
[
  {
    "code":"ERRO1101-ae189db1-0276-42d0-8c31-766b6c0b8ec2",
    "message_and_tip":"未知异常",
    "level":1,
    "complexity":1,
    "main_flow_flag":true
  }
]
```

or

生成一个具名异常码

```bash
gen 这是一个具名异常码 -m0 -l2 -c10
ERRO1101-ae189db1-0276-42d0-8c31-766b6c0b8ec2 这是一个具名异常码
```

生成的`.error_code.json`

```json
[
  {
    "code":"ERRO0210-ae189db1-0276-42d0-8c31-766b6c0b8ec2",
    "message_and_tip":"这是一个具名异常码",
    "level":2,
    "complexity":10,
    "main_flow_flag":false
  }
]
```

若需要记录**额外信息**则可以这样

```bash
gen 这是一个具有而外辅助信息的code -m0 -l2 -c10 a b c d e f f10
ERRO1101-ae189db1-0276-42d0-8c31-766b6c0b8ec2 这是一个具名异常码
```

生成的`.error_code.json`

```json
[
  {
    "code":"ERRO0210-ae189db1-0276-42d0-8c31-766b6c0b8ec2",
    "message_and_tip":"这是一个具名异常码",
    "level":2,
    "complexity":10,
    "main_flow_flag":false,
    "ex":["a","b","c","d","e","f","f10"]
  }
]
```

#### Step 3

在后台的业务流程中加入error_code,同时实现一个接口用于返回`.error_code.json`的文件内容。

# 规范

为了实现`故事5`，需要所有后台开发人员在填写**额外信息**的时候表达出来 (where、who、what happend)。

一个好的例子

```bash
gen 用户名密码错误 /service/login.java:209 用户 登陆 失败
ERRO1101-ae189db1-0276-42d0-8c31-766b6c0b8ec2 用户名密码错误
```

同时，在向外部报出错误码的同时，需要在日志中记录该错误码。

这样我们排查问题的流程就可以更加清晰:
1. 用户向我们反馈，前台提示**用户名密码错误**.
2. 前台开发人员排查,发现报了错误码为：ERRO1101-ae189db1-0276-42d0-8c31-766b6c0b8ec2.
3. 后台通过`ERRO`可以定位到是哪个服务，并联系服务的开发人员。
4. 服务的开发人员通过搜素`日志`或`.error_code.json`中的关键字`ERRO1101-ae189db1-0276-42d0-8c31-766b6c0b8ec2`,可以得到辅助信息，从而快速定位到此次反馈对应的真实的日志片段，和代码片段。


