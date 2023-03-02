# 中文图书Bookdown模板的基本用法 {#usage}



## 安装设置 {#usage-ins}

使用RStudio软件完成编辑和转换功能。
在RStudio中，安装bookdown等必要的扩展包。

本模板在安装之前是一个打包的zip文件，
在适当位置解压（例如，在`C:/myproj`下），
得到`MathJax`, `Books/Cbook`, `Books/Carticle`等子目录。
本模板在`Books/Cbook`中。

为了利用模板制作自己的中文书，
将`Books/Cbook`制作一个副本，
改成适当的子目录名，如`Books/Mybook`。

打开RStudio软件，
选选单“File - New Project - Existing Directory”，
选中`Books/Mybook`子目录，确定。
这样生成一本书对应的R project（项目）。

为了将模板内容替换成自己的内容，
可以删除文件`0101-usage.Rmd`，
然后将`1001-chapter01.Rmd`制作几份副本，
如`1001-chapter01.Rmd`, `2012-chapter02.Rmd`，
`3012-chapter03.Rmd`。
各章的次序将按照前面的数值的次序排列。
将每个`.Rmd`文件内的`{#chapter01}`, `{#chapter02-sec01}`修改能够反映章节内容的标签文本。
所有的标签都不允许重复。
参见本模板中的`0101-usage.Rmd`文件。

后面的§\@ref(usage-gitbook) 和§\@ref(usage-pdfbook) 给出了将当前的书转换为网页和PDF的命令，
复制粘贴这些命令到RStudio命令行可以进行转换。


## 编写自己的内容 {#usage-writing}

### 文档结构 {#usage-writing-struct}

除了`index.Rmd`以外，
每个`.Rmd`文件是书的一章。
每章的第一行是用一个井号（`#`）引入的章标题。
节标题用两个井号开始，
小节标题用三个井号开始。
标题后面都有大括号内以井号开头的标签，
标签仅用英文大小写字母和减号。


### 图形自动编号 {#usage-writing-fig}

用R代码段生成的图形，
只要具有代码段标签，
且提供代码段选项`fig.cap="图形的说明文字"`，
就可以对图形自动编号，
并且可以用如`\@ref(fig:label)`的格式引用图形。
如：


```r
plot(1:10, main="程序生成的测试图形")
```

![(\#fig:u-w-f-ex01)图形说明文字](0101-usage_files/figure-latex/u-w-f-ex01-1.pdf) 

引用如：参见图\@ref(fig:u-w-f-ex01)。
引用中的`fig:`是必须的。

在通过LaTeX转换的PDF结果中，
这样图形是浮动的。


### 表格自动编号 {#usage-writing-tab}

用R代码`knitr::kable()`生成的表格，
只要具有代码段标签，
并且在`knitr::kable()`调用时加选项`caption="表格的说明文字"`，
就可以对表格自动编号，
并且可以用如`\@ref(tab:label)`的格式引用表格。
如：


```r
d <- data.frame("自变量"=1:10, "因变量"=(1:10)^2)
knitr::kable(d, caption="表格说明文字")
```

\begin{table}

\caption{(\#tab:u-w-tab-ex01)表格说明文字}
\centering
\begin{tabular}[t]{r|r}
\hline
自变量 & 因变量\\
\hline
1 & 1\\
\hline
2 & 4\\
\hline
3 & 9\\
\hline
4 & 16\\
\hline
5 & 25\\
\hline
6 & 36\\
\hline
7 & 49\\
\hline
8 & 64\\
\hline
9 & 81\\
\hline
10 & 100\\
\hline
\end{tabular}
\end{table}

引用如：参见表\@ref(tab:u-w-tab-ex01)。
引用中的`tab:`是必须的。

在通过LaTeX转换的PDF结果中，
这样的表格是浮动的。


### 数学公式编号 {#usage-writing-math}

不需要编号的公式，
仍可以按照一般的Rmd文件中公式的做法。
需要编号的公式，
直接写在`\begin{align}`和`\end{align}`之间，
不需要编号的行在末尾用`\nonumber`标注。
需要编号的行用`(\#eq:mylabel)`添加自定义标签，
如

\begin{align}
\Sigma =&  (\sigma_{ij})_{n\times n} \nonumber \\
=& E[(\boldsymbol{X} - \boldsymbol{\mu}) (\boldsymbol{X} - \boldsymbol{\mu})^T ] 
(\#eq:var-mat-def)
\end{align}

引用如：协方差定义见\@ref(eq:var-mat-def)。

### 文献引用与文献列表

将所有文献用bib格式保存为一个`.bib`文献库，
如模板中的样例文件`mybib.bib`。
可以用JabRef软件来管理这样的文献库，
许多其它软件都可以输出这样格式的文件库。

为了引用某一本书，
用如：参见[@Wichmann1982:RNG]。

被引用的文献将出现在一章末尾以及全书的末尾，
对PDF输出则仅出现在全书末尾。

## 转换 {#usage-output}

### 转换为网页 {#usage-gitbook}

用如下命令将整本书转换成一个每章为一个页面的网站，
称为gitbook格式：


```r
bookdown::render_book("index.Rmd", 
  output_format="bookdown::gitbook", encoding="UTF-8")
```

为查看结果，
在`_book`子目录中双击其中的`index.html`文件，
就可以在网络浏览器中查看转换的结果。
重新编译后应该点击“刷新”图标。

在章节和内容较多时，
通常不希望每次小修改之后重新编译整本书，
这时类似如下的命令可以仅编译一章，
可以节省时间，
缺点是导航目录会变得不准确。
命令如：


```r
bookdown::preview_chapter("1001-chapter01.Rmd",
  output_format="bookdown::gitbook", encoding="UTF-8")
```

单章的网页可以通过网络浏览器中的“打印”功能，
选择一个打印到PDF的打印机，
可以将单章转换成PDF格式。


### 生成PDF {#usage-pdfbook}

如果想将R Markdown文件借助于LaTeX格式转换为PDF， 
需要在系统中安装一个TeX编译器。 
现在的rmarkdown包要求使用tinytex扩展包以及配套的[TinyTeX软件包](https://yihui.name/tinytex/)，
好像不再支持使用本机原有的LaTex编译系统， 
如果不安装tinytex，编译为PDF格式时会出错。
TinyTeX优点是直接用R命令就可以安装，
更新也由R自动进行，不需要用户干预。
但是，安装时需要从国外网站下载许多文件，
有因为网络不畅通而安装失败的危险。

为了安装R的tinytex扩展包和单独的TinyTeX编译软件，应运行：


```r
install.packages('tinytex')
tinytex::install_tinytex()
```

安装过程需要从国外的服务器下载许多文件， 
在国内的网络环境下有可能因为网络超时而失败。 
如果安装成功， 
TinyTeX软件包在MS Windows系统中一般会安装在 `C:\Users\用户名\AppData\Roaming\MikTex`目录中，
其中“用户名”应替换成系统当前用户名。 
如果需要删除TinyTeX软件包， 只要直接删除那个子目录就可以。

为了判断TinyTeX是否安装成功， 在RStudio中运行


```r
tinytex::is_tinytex()
```

结果应为TRUE, 出错或者结果为FALSE都说明安装不成功。
在编译`pdf_book`时，可能会需要联网下载LaTeX所需的格式文件。

Bookdown借助操作系统中安装的LaTeX编译软件TinyTeX将整本书转换成一个PDF文件，
这需要用户对LaTeX有一定的了解，
否则一旦出错，
就完全不知道如何解决。
用户如果需要进行LaTeX定制，
可修改模板中的`preamble.tex`文件。

转换为PDF的命令如下：


```r
bookdown::render_book("index.Rmd", 
  output_format="bookdown::pdf_book", encoding="UTF-8")
```

在`_book`子目录中找到`CBook.pdf`文件，
这是转换的结果。
CBook.tex是作为中间结果的LaTeX文件，
如果出错可以从这里查找错误原因。

转换PDF对于内容多的书比较耗时，
不要过于频繁地转换PDF，
在修改书的内容时，
多用`bookdown::preview_chapter`和转换为gitbook的办法检验结果。
定期地进行转换PDF的测试。
每增加一章后都应该试着转换成PDF看有没有错误。



### 上传到网站 {#usage-website}

如果书里面没有数学公式，
则上传到网站就只要将`_book`子目录整个地用ftp软件传送到自己的网站主目录下的某个子目录即可。
但是，为了支持数学公式，就需要进行如下的目录结构设置：

1. 设自己的网站服务器目录为`/home/abc`，
   将MathJax目录上传到这个目录中。
2. 在`/home/abc`中建立新目录`Books/Mybook`。
3. 将`_book`子目录上传到`/home/abc/Books/Mybook`中。
4. 这时网站链接可能类似于`http://dept.univ.edu.cn/~abc/Books/Mybook/_book/index.html`,
   具体链接地址依赖于服务器名称与主页所在的主目录名称。

如果有多本书，
`MathJax`仅需要上传一次。
因为`MathJax`有三万多个文件，
所以上传`MathJax`会花费很长时间。



