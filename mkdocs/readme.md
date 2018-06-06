### mkdocs 安装

```
sudo pip3 install mkdocs
```
###官方网站

mkdocs官网   https://www.mkdocs.org/

https://readthedocs.org/

### 查看版本号

```
mkdocs --version
```
### 创建项目

```
mkdocs new my-project
```

### 生成html文件

```
mkdocs build
```
运行完编译命令以后生成的静态文件在目录site目录下。

### 常用命令

Usage: mkdocs [OPTIONS] COMMAND [ARGS]...

  MkDocs - Project documentation with Markdown.

Options:
  -V, --version  Show the version and exit.
  -q, --quiet    Silence warnings
  -v, --verbose  Enable verbose output
  -h, --help     Show this message and exit.

Commands:
  build      Build the MkDocs documentation
  gh-deploy  Deploy your documentation to GitHub Pages
  new        Create a new MkDocs project
  serve      Run the builtin development server

