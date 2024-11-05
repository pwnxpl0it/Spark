# idkmng: A TOML based project initializer.

idkmng is a TOML-based project initializer that helps you quickly set up project directories and files according to predefined templates. It's designed for developers who want to streamline their workflow by automating the creation of consistent project structures.

Template-based, with easy [TOML](https://toml.io/en/) support!

## Idea 🧠:
Creating a project often involves setting up specific files with unique formats. For instance, a browser extension requires files like:

```
manifest.json
Content.js
etc.. 
```
Each file has its own structure and syntax. With idkmng, you can effortlessly initialize projects with the required files and configurations using a simple command, regardless of the specific format needed.

For example, to set up a browser extension project, you can run:
```sh
$ idkmng browser_extension
```
and 💥 your project files will be ready for your initial commit. Isn’t that amazing?

idkmng is template-based, allowing you to define multiple templates for the same type of project. This means you can customize the initialization process to suit different requirements, such as variations in manifest.json or other project-specific files.


## Installation
If you have Rust 🦀 🚀 installed on your machine run the following command:

```sh
$ cargo install --git https://www.github.com/pwnxpl0it/idkmng
```

Alternatively you can go to [Releases](https://github.com/pwnxpl0it/idkmng/releases) and download the binary

```console
$ sudo tar -xzf idkmng-<RELEASE>.tar.gz -C /usr/local/bin
```

Replace <RELEASE> with the version number or tag of the release you want to install.

now you should be able to run `idkmng` in your terminal!

## Creating a template 📜
There is a template for creating a template! 
just run the following command! 
```sh
$ idkmng new
```
enter template name and you should have one, it will go inside `~/.config/idkmng/templates/TEMPLATENAME.toml`
also you can edit that Template too to create you own template that creates a template 🎉,<br>

Default templates path is `~/.config/idkmng/templates`<br>

The template structure is like the following:
```toml
[info]
name = ""
description = ""
author = ""

[[files]] # file1 
path="/path/to/file1"
content="""

Content of file 1

"""

[[files]] # file2
path="/path/to/file2"
content="""

Content of file 2

"""

#etc...
```

> [!INFO]
> Info section is not required and can be removed

placeholders have a generic format of `{{$keyword}}`

| Keyword/placeholder   | Value     | Example          |
|--------------- | ---------------  | ---------------  |
| PROJECTNAME   | Asks for project name |                   |
| CURRENTDIR    | Current directory | pwd=/foo/bar => `bar`|
| HOME          | Home directory    | `/home/user/`    |
| YYYY    | Current Year in YYYY format| 2024    |
| YY | Current Year in YY format| 24    |
| MM | Current Month | 2 |
| DD | Current Day | 24 |
| NOW | Current date and time | `2024-02-23 22:22:38.151417626 +00:00` |
| NOW_UTC | Current date and time in UTC | `2024-02-23 22:21:17.897444668 UTC` |

---
### Functions
you can enhance the functionality of your templates by using special placeholders that prompt for user input or retrieve values from environment variables. These placeholders follow the format `{{$keyword:function}}`

Example: 
```toml
# --snip
content="""
{{$TEST:read}}
"""
```

Functions supported by idkmng:

| Function   | Description    | Example  |
|--------------- | --------------- | ---------------  |
| read   | Asks for user input to replace placeholder with   | `{{$TEST:read}}` |
| env    | Replace with value from environment variables     | `{{$PATH:env}}` |

also keep in mind that once a function gets called on a keyword you can use `{{$TEST:read}}` or `{{$TEST}}` both are going to be replaced with the value that the function returned.

## Git Support 🐙
Initialize Git during project creation by using:

```console
$ idkmng /path/to/template --git
```

Or include Git option in the template itself:

```toml
[options]
git=true
project_root="{{$PROJECTNAME}}"
```

### Example Templates
Here are a few examples:

<details>
  <summary>Neovim Plugin [Click to expand]</summary>

```toml
[info]
name = "Neovim Plugin"
description = "A template for nvim plugin"
author = "Mohamed Tarek @pwnxpl0it"

[[files]]
path="{{$PROJECTNAME}}/lua/{{$PROJECTNAME}}/init.lua"
content="""
local M = {}

M.config = {}

M.setup = function ()
   if config ~= nil then
        M.config = config
    end

end

return M
"""

[[files]]
path="{{$PROJECTNAME}}/plugin/init.lua"
content="""
require("{{$PROJECTNAME}}")
"""
```

</details>


<details>
    <summary>Jekyll new blogpost [Click to expand]</summary>

I use this template to create a new post in my blog directly from CLI,This one here uses more keywords and includes a private BLOGPATH placeholder that it's value is loaded from config file.

```toml
[info]
name = "new_post"
description = "New jekyll post"
author = "Mohamed Tarek @pwnxpl0it"

[[files]]
path="{{$BLOGPATH}}/_posts/{{$YYYY}}-{{$MM}}-{{$DD}}-{{$blogtitle:read}}.markdown"
content="""
---
layout: post
title: "{{$blogtitle}}"
date: {{$NOW_UTC}}
tags: {{$Tags:read}}
---

"""

```

</details>

<details>
    <summary>Browser (Chrome) Extension [Click to expand]</summary>
This one is just for creating a really BASIC chrome extension.

```toml
[info] # Generated using `idkmng new` btw
name = "browser_extension"
description = "A Template for creating a browser extension"
author = "Mohamed Tarek @pwnxpl0it"
refrence= "https://developer.chrome.com/docs/extensions/mv3/manifest/"

[[files]]
path="{{$PROJECTNAME}}/manifest.json"
content="""
{
  "manifest_version": 3,
  "name":"{{$PROJECTNAME}}",
  "version": "1.0.1",
  "content_scripts":[
    {
     "matches":["<all_urls>"],
     "js":["content.js"]
    }
  ]
}
"""

[[files]]
path="{{$PROJECTNAME}}/content.js"
content="""
console.log("Hello world!")
"""

```

> [!TIP]
>  Info section can have any additional values, it won't get printed but maybe usefull when sharing the template or just as a reference for docs like I did here

</details>

<!--TODO: Add more examples-->

## Loading values from json
You can use json to replace placeholders in your template, idkmng will automatically load values from a json file and replace them automatically

idkmng uses JSON Query language to load values from json nodes.

This is made possible using Rust's [jq-rs crate 🦀](https://github.com/onelson/jq-rs) which has bindings to libjq.

Here is an example:

```json
{
	"user": {
		"id": "12345",
		"name": "John Doe",
		"email": "john.doe@example.com"
	},
	"status": ["200 OK"]
}
```

example template:

```toml
[[files]]
path="test"
content="""
User ID: {{$.user.id}}
User Name: {{$.user.name}}
User Email: {{$.user.email}}
Response Status: {{$.status[0]}}
"""
```

```sh
$ idkmng template --json test.json
```

Output:

```console
$ cat test

User ID: 12345
User Name: John Doe
User Email: john.doe@example.com
Response Status: 200 OK
```

> [!NOTE]
> Although this is a cool feature to automate user inputs, It comes with performance costs
> [Why?](https://github.com/onelson/jq-rs?tab=readme-ov-file#a-note-on-performance)

## Liquid Templating Support 🧪

`idkmng` now supports [Liquid](https://shopify.github.io/liquid/) templating alongside its own custom syntax. This allows you to benefit from Liquid's logic (loops, conditionals) while continuing to use `idkmng`'s powerful keyword replacement.

#### **Example:**
```toml
[[files]]
path = "output.txt"
content = """
{% for i in (1..5) %}
Example! {{ i }} {{ "{{$file:read}}" | append: ".html" }}
{% endfor %}
"""
```

- `idkmng` replaces `{{$file:read}}` with user input.
- Liquid handles loops and string manipulation.

#### **Result:**
```
Example! 1 ff.html
Example! 2 ff.html
Example! 3 ff.html
Example! 4 ff.html
Example! 5 ff.html
```

With this integration, you can create dynamic and flexible templates that combine the strengths of both `idkmng` and Liquid.

## Automated Template generation 🚀
Also there is one more time saving way! if you have some files in `/foo/bar/` you can just run `idkmng init` and it will create a template for you with directory name `bar.toml` and it will have all your files in it! 🌸

```console
$ tree
.
├── lua
│   └── test123
│       └── init.lua
└── plugin
    └── init.lua

4 directories, 2 files

$ idkmng init
Creating Template: test123.toml
```

```console
$ cat test123.toml

[[files]]
path = 'plugin/init.lua'
content = '''
require("test123")
'''

[[files]]
path = 'lua/test123/init.lua'
content = '''
local M = {}

M.config = {}

M.setup = function ()
   if config ~= nil then
        M.config = config
    end

end

return M
'''

```

## Config Keywords ⚙️
You can have your own Keywords for idkmng to replace with desired values!
Idkmng finds them stored in $HOME/.config/idkmng/config.toml Or the config path you specified using -c/--config option 🦀

```toml
[Keywords]
AUTHOR = "Mohamed Tarek"
USERNAME = "@pwnxpl0it"
GITHUB = "https://github.com/pwnxpl0it"
#etc .....
```

## 👾 Neovim plugin (idkmng.nvim) 
I wrote a neovim plugin that makes it a way easier, Check it out [idkmng.nvim](https://www.github.com/pwnxpl0it/idkmng.nvim).
