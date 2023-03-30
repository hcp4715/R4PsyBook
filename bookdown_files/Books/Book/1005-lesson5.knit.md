---
editor_options: 
  markdown: 
    wrap: 72
---

# 第五讲：如何清理数据—数据的预处理{#lesson-5}



## **批量读取文件**

在本节课中，我们将讲解如何对从问卷或实验软件中下载的数据进行预处理，如果我们想要进行回归或中介分析等，就需要对数据进行预处理。以得到我们最终想要分析的数据。我们将演示如何使用for loop或lapply函数将多个数据合并成一个完整的数据。

我们会给出一个完整的路径或相对路径，让R内部的计算机知道文件在哪里，并对其进行操作，比如read.csv。如果我们要读取一系列文件，我们需要将它们的相对路径读入，然后依次输入到处理器中，让它们被读取并合并成一个完整的数据框。因此，我们需要使用for循环或lapply函数将所有文件的路径列出来，并将它们输入到处理器中，让它们被读取并合并。

我们的输入是文件夹中的文件名。大家可以看一下自己电脑中的文件夹，会发现有很多子文件，比如practice、match和category，它表示的是三个时间的三个阶段 practice表示它在做练习，match表示它在做match的任务，category表示它在做categorization的一个任务。现在我们需要找出所有包含match且以out结尾的文件，因为这些是我们需要读取的文件。

### **通配符**

我们不想逐个列出每个文件名，因为这样太冗长且容易出错。我们可以使用R中的list函数，将文件夹中的所有文件列出来，然后筛选只包含match的文件名。为了实现这一点，我们需要使用通配符，它是一种特殊的符号，可以匹配任意字符。

例如，*.csv代表以csv结尾的所有文件。我们可以使用list函数扫描文件夹，找到所有以match开头且以out结尾的文件名，然后将它们合并成我们需要的文件。我们要做的第一件事是扫描这个文件夹，扫描这个文件夹，把里面所有的文件和文件夹都读取出来。但是这样并不完全符合我们的目标，因为我们只需要符合match这个条件，并且是以out结尾的数据。所以，我们需要列出match文件夹里所有的文件，但这并不符合我们的目标。这时，我们需要使用通配符来匹配文件夹里包含特殊信息的文件。也就是说，我们需要根据文件名是否包含match来筛选文件夹里的文件。

这时，我们需要使用通配符，尤其是问号这个通配符。因为在计算机语言中，问号代表任意数量的任意字符。当我们以 星号.csv结尾时，代表我们只需要扫描以.csv结尾的文件，把它们全部读出来，读取它们的完整文件名。如果它不是以.csv结尾的，我们就跳过它。问号代表单个字符，也就是信号代表任意单个字符。例如，如果我们使用file?，然后是.txt，那么它能够匹配的符合条件的文件就是file1、file2、file3等等。但是如果你是file10，它后面有两个字符，这时它就不匹配了。

中括号里的字符是或的关系，就是说，中括号里的123代表file1后面跟1、2或3都可以。这样可以任意灵活地匹配。因为有可能你不知道文件夹里有多少个文件，你就把它们全部写在中括号里，只要它包含在里面，我们就把它读取出来。


```r
library(DT)
```

```
## Warning: package 'DT' was built under R version 4.2.3
```

```r
# 所有路径使用相对路径
library(here)
```

```
## Warning: package 'here' was built under R version 4.2.3
```

```
## here() starts at D:/GitHub/R4PsyBook/bookdown_files/Books/Book
```

```r
# 包含了dplyr和%>%等好用的包的集合
library(tidyverse)
# 养成用相对路径的好习惯，便于其他人运行你的代码
WD <-  here::here()
getwd()
```

```
## [1] "D:/GitHub/R4PsyBook/bookdown_files/Books/Book"
```

### for**循环思路**

那么我们该怎么使用它呢？我们需要先加载tidyverse这个包，提醒大家在开始处理之前要load这个包，否则在使用函数时会出错。这是一个准备工作。接着，我们找到了当前的工作路径。


```r
# 把所有符合某种标题的文件全部读取到一个list中
files <- list.files(file.path("data/match"), pattern = "data_exp7_rep_match_.*\\.out$")


head(files, n = 10L)
```

```
##  [1] "data_exp7_rep_match_7302.out" "data_exp7_rep_match_7303.out"
##  [3] "data_exp7_rep_match_7304.out" "data_exp7_rep_match_7305.out"
##  [5] "data_exp7_rep_match_7306.out" "data_exp7_rep_match_7307.out"
##  [7] "data_exp7_rep_match_7308.out" "data_exp7_rep_match_7309.out"
##  [9] "data_exp7_rep_match_7310.out" "data_exp7_rep_match_7311.out"
```

```r
str(files)
```

```
##  chr [1:44] "data_exp7_rep_match_7302.out" "data_exp7_rep_match_7303.out" ...
```

我们可以用两种方法来处理这个for循环，其中一种是使用一个变量名files来存储我们从这个文件夹里扫描到的所有文件的名字。在R中，list.files()是一个函数，它可以扫描文件夹里的所有文件。第一个参数是你要在哪个文件夹里扫描。即在当前的工作目录里的data文件夹中，然后在这个文件夹中的match文件夹里进行扫描。我们使用这个pattern来筛选文件夹里的文件名，比如包含了这个match的，或者更加精确的，我们要以这个data_exp7_，然后以repeat开头的文件。然后我们使用一个通配符来表示所有以out结尾的文件都扫描进来。这样扫描完后，我们就得到了所有符合我们这个模式的文件的名字。我们只会得到它们的文件名，不包含它们的路径。如果我们把它们读取出来并列出前面10个，我们就可以看到它们确实完全符合我们上面那个规则。list.files()函数非常有用，因为它可以帮助我们快速地扫描文件夹里的所有文件。

这个操作实际上是扫描一个文件夹，并按照特定模式扫描文件。在R语言中，我们经常使用这个操作来读取数据，这是第一步。我们现在读取到的是一个向量，其中每个元素包含许多字符，每个字符代表一个文件名。这与我们之前讨论的读取单个文件不同。现在我们正在列出一个文件夹中所有符合特定模式或规则的文件名。

读取完后，我们可以使用for循环。首先，我们需要创建一个空的列表来存储读取的数据。for循环的结构是for（条件）{内容}。我们使用i来循环，in表示我们要循环的范围。在for循环中，我们使用i来逐个循环。我们首先将i设置为1，然后做某些事情，然后在i等于2时再做同样的事情，一直到i等于10。


```r
# 创建一个空的列表来存储读取的数据框
df_list <- list()
# 循环读取每个文件，处理数据并添加到列表中
for (i in seq_along(files)) { # 重复"读取到的.out个数"的次数
  # 对每个.out，使用read.table
  df <- read.table(file.path("data/match", files[i]), header = TRUE) #read.table似乎比read.csv更聪明，不需要指定分隔符
  # 给读取到的每个.out文件的每个变量统一变量格式
  df <- dplyr::filter(df, Date != "Date") %>% # 因为有些.out文件中部还有变量名，所需需要用filter把这些行过滤掉
    dplyr::mutate(Date = as.character(Date),Prac = as.character(Prac),
                  Sub = as.numeric(Sub),Age = as.numeric(Age),Sex = as.character(Sex),Hand = as.character(Hand),
                  Block = as.numeric(Block),Bin = as.numeric(Bin),Trial = as.numeric(Trial),
                  Shape = as.character(Shape),Label = as.character(Shape),Match = as.character(Match),
                  CorrResp = as.character(CorrResp),Resp = as.character(Resp),
                  ACC = as.numeric(ACC),RT = as.numeric(RT))
  # 将数据框添加到列表中
  df_list[[i]] <- df
}
# 合并所有数据框，只有当变量的属性一致时，才可以bind_rows
# bind_rows 意味着把list中的所有表格整合成一个大表格
df.mt.out.fl <- dplyr::bind_rows(df_list)
# 清除中间变量
rm(df,df_list,files,i)
# 如果你将这个步骤写成函数，则这些变量自然不会出现在全局变量中
```

在for循环中，我们使用df=read.table来读取文件。文件的路径是相对路径，是data match和files的组合。files是文件夹中的一个文件。当i等于1时，我们读取的是第一个文件。我们按照header为true的方式去读取。我们发现out文件里保存的可能不是那么统一，因此我们需要去掉重复的列名。我们可以用一个规则，即如果data再次出现，表示它重复了上面这个列名，我们就把所有这样的行去掉。我们用了上次课选择的那个筛选函数。

反向筛选掉之后，我们还可以对它的变量类型做一些变化。我们基本上就是用了as character和as numerical这两种，把所有的列都变成我们想要的类型。然后，我们生成了一个数据框，并进行了转换。当i等于1时，它就是对第一个文件做了这一番操作。然后，我们把它复制到数据列表里面，第一个位置就是第一个out文件，它经过了转换之后的数据框，就装到数据框列表里面的第一个位置。我们首先将第一个out文件转换成数据框df，并将其放入df列表的第一个位置。然后，我们通过循环读取id位中的第二个文件，并倒数10个，直到读取完所有10个文件。data列表的长度为10，其中包含我们读取的10个数据。

将所有数据放入列表后，我们可以使用bind_rows函数将它们合并成一个大的数据框。bind_rows是Dplyr中一个常用的函数，它可以通过行将不同的数据框合并。因此，data列表实际上是一个完整的列表，其中包含一个一个的out文件，经过初步转换后形成的数据框。由于它们的列都是一模一样的，我们可以直接将它们叠加在一起，最终得到一个完整的数据框。

这就是for循环的逻辑，我们首先读取我们想要的文件名，然后通过迭代的方式一个一个地读取它们，并将它们放入一个DataList中。然后，我们对这个List进行完整的合并。这个思路非常直接，大家都能够理解。当然，理解for循环的前提是我们要理解它的工作原理。为什么我每次讲for循环的时候，助教们可能觉得没有必要呢？因为for循环的使用实际上非常少。但是，我个人认为还是要讲一下for循环。因为for循环不仅可以用在这里，而且可以用在很多场景中。当你需要重复做某一件事情时，没有现成的函数可以帮你完成时，最简单的方法就是写一个for循环，它可以循环迭代帮你完成某一件事情。i可以替代很多东西，比如你有五个东西需要做同样的操作，你可以写一个i从1到5的for循环，将每个东西放进去做同样的操作。这样，你就不用一个一个复制你的代码，然后改变其中的一个值，再运行一遍。如果你经常进行批量处理，学会for循环会非常有帮助。为了更方便地读取并合并文件夹中的文件，我们通常需要使用for循环。

最终的结果应该是25920 obs 16 variables


```{=html}
<div id="htmlwidget-743dc7415163a8399393" style="width:100%;height:auto;" class="datatables html-widget"></div>
<script type="application/json" data-for="htmlwidget-743dc7415163a8399393">{"x":{"filter":"none","vertical":false,"fillContainer":true,"data":[["1","2","3","4","5","6","7","8","9","10","11","12","13","14","15","16","17","18","19","20","21","22","23","24","25","26","27","28","29","30","31","32","33","34","35","36","37","38","39","40","41","42","43","44","45","46","47","48","49","50","51","52","53","54","55","56","57","58","59","60","61","62","63","64","65","66","67","68","69","70","71","72","73","74","75","76","77","78","79","80","81","82","83","84","85","86","87","88","89","90","91","92","93","94","95","96","97","98","99","100"],["02-May-2018_14:23:06","02-May-2018_14:23:08","02-May-2018_14:23:10","02-May-2018_14:23:13","02-May-2018_14:23:15","02-May-2018_14:23:17","02-May-2018_14:23:19","02-May-2018_14:23:21","02-May-2018_14:23:24","02-May-2018_14:23:26","02-May-2018_14:23:28","02-May-2018_14:23:30","02-May-2018_14:23:32","02-May-2018_14:23:34","02-May-2018_14:23:36","02-May-2018_14:23:38","02-May-2018_14:23:40","02-May-2018_14:23:42","02-May-2018_14:23:45","02-May-2018_14:23:47","02-May-2018_14:23:49","02-May-2018_14:23:51","02-May-2018_14:23:53","02-May-2018_14:23:55","02-May-2018_14:23:57","02-May-2018_14:24:00","02-May-2018_14:24:02","02-May-2018_14:24:04","02-May-2018_14:24:06","02-May-2018_14:24:08","02-May-2018_14:24:10","02-May-2018_14:24:12","02-May-2018_14:24:15","02-May-2018_14:24:16","02-May-2018_14:24:19","02-May-2018_14:24:21","02-May-2018_14:24:23","02-May-2018_14:24:25","02-May-2018_14:24:27","02-May-2018_14:24:29","02-May-2018_14:24:32","02-May-2018_14:24:34","02-May-2018_14:24:36","02-May-2018_14:24:38","02-May-2018_14:24:40","02-May-2018_14:24:42","02-May-2018_14:24:44","02-May-2018_14:24:46","02-May-2018_14:24:48","02-May-2018_14:24:51","02-May-2018_14:24:53","02-May-2018_14:24:55","02-May-2018_14:24:57","02-May-2018_14:24:59","02-May-2018_14:25:01","02-May-2018_14:25:03","02-May-2018_14:25:05","02-May-2018_14:25:08","02-May-2018_14:25:10","02-May-2018_14:25:12","02-May-2018_14:25:14","02-May-2018_14:25:16","02-May-2018_14:25:19","02-May-2018_14:25:21","02-May-2018_14:25:23","02-May-2018_14:25:25","02-May-2018_14:25:27","02-May-2018_14:25:30","02-May-2018_14:25:32","02-May-2018_14:25:34","02-May-2018_14:25:36","02-May-2018_14:25:38","02-May-2018_14:25:53","02-May-2018_14:25:55","02-May-2018_14:25:57","02-May-2018_14:26:00","02-May-2018_14:26:02","02-May-2018_14:26:04","02-May-2018_14:26:06","02-May-2018_14:26:08","02-May-2018_14:26:10","02-May-2018_14:26:11","02-May-2018_14:26:14","02-May-2018_14:26:16","02-May-2018_14:26:18","02-May-2018_14:26:20","02-May-2018_14:26:22","02-May-2018_14:26:25","02-May-2018_14:26:26","02-May-2018_14:26:28","02-May-2018_14:26:31","02-May-2018_14:26:33","02-May-2018_14:26:35","02-May-2018_14:26:37","02-May-2018_14:26:39","02-May-2018_14:26:41","02-May-2018_14:26:43","02-May-2018_14:26:45","02-May-2018_14:26:47","02-May-2018_14:26:49"],["Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp"],[7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302],[22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22],["female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female"],["R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R"],[1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1],[1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,5,5,5,5],[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,1,2,3,4],["immoralSelf","moralOther","immoralOther","moralSelf","immoralSelf","immoralSelf","moralOther","moralSelf","moralOther","immoralSelf","moralOther","immoralOther","moralOther","moralSelf","immoralOther","immoralSelf","moralSelf","moralSelf","immoralSelf","moralOther","immoralOther","moralSelf","immoralOther","immoralOther","moralSelf","moralSelf","immoralOther","immoralOther","immoralSelf","immoralOther","moralSelf","moralOther","immoralSelf","moralOther","immoralSelf","moralSelf","immoralSelf","moralOther","moralOther","immoralSelf","moralOther","moralSelf","moralSelf","immoralOther","immoralOther","immoralSelf","immoralOther","moralOther","immoralOther","moralOther","moralSelf","immoralSelf","moralOther","moralOther","immoralOther","moralSelf","immoralOther","moralSelf","immoralSelf","moralSelf","immoralSelf","immoralOther","immoralSelf","moralOther","immoralSelf","moralOther","immoralSelf","immoralOther","moralOther","moralSelf","immoralOther","moralSelf","immoralSelf","moralOther","moralSelf","moralSelf","immoralOther","immoralOther","immoralSelf","moralOther","moralOther","moralSelf","immoralSelf","moralSelf","moralOther","immoralOther","immoralOther","immoralSelf","moralSelf","moralSelf","immoralSelf","immoralOther","moralOther","moralOther","immoralOther","immoralSelf","moralOther","moralOther","immoralOther","moralSelf"],["immoralSelf","moralOther","immoralOther","moralSelf","immoralSelf","immoralSelf","moralOther","moralSelf","moralOther","immoralSelf","moralOther","immoralOther","moralOther","moralSelf","immoralOther","immoralSelf","moralSelf","moralSelf","immoralSelf","moralOther","immoralOther","moralSelf","immoralOther","immoralOther","moralSelf","moralSelf","immoralOther","immoralOther","immoralSelf","immoralOther","moralSelf","moralOther","immoralSelf","moralOther","immoralSelf","moralSelf","immoralSelf","moralOther","moralOther","immoralSelf","moralOther","moralSelf","moralSelf","immoralOther","immoralOther","immoralSelf","immoralOther","moralOther","immoralOther","moralOther","moralSelf","immoralSelf","moralOther","moralOther","immoralOther","moralSelf","immoralOther","moralSelf","immoralSelf","moralSelf","immoralSelf","immoralOther","immoralSelf","moralOther","immoralSelf","moralOther","immoralSelf","immoralOther","moralOther","moralSelf","immoralOther","moralSelf","immoralSelf","moralOther","moralSelf","moralSelf","immoralOther","immoralOther","immoralSelf","moralOther","moralOther","moralSelf","immoralSelf","moralSelf","moralOther","immoralOther","immoralOther","immoralSelf","moralSelf","moralSelf","immoralSelf","immoralOther","moralOther","moralOther","immoralOther","immoralSelf","moralOther","moralOther","immoralOther","moralSelf"],["mismatch","mismatch","mismatch","mismatch","match","match","match","match","mismatch","mismatch","mismatch","match","match","mismatch","match","mismatch","mismatch","match","match","match","mismatch","match","match","mismatch","match","mismatch","match","mismatch","mismatch","match","match","match","mismatch","match","mismatch","mismatch","match","mismatch","mismatch","match","mismatch","mismatch","match","mismatch","match","match","mismatch","match","match","mismatch","mismatch","mismatch","match","match","mismatch","mismatch","match","mismatch","match","match","match","mismatch","mismatch","match","match","mismatch","mismatch","mismatch","mismatch","match","match","match","match","mismatch","mismatch","match","match","match","mismatch","match","match","match","mismatch","mismatch","mismatch","mismatch","mismatch","mismatch","mismatch","match","match","mismatch","match","mismatch","match","match","mismatch","match","match","match"],["n","n","n","n","m","m","m","m","n","n","n","m","m","n","m","n","n","m","m","m","n","m","m","n","m","n","m","n","n","m","m","m","n","m","n","n","m","n","n","m","n","n","m","n","m","m","n","m","m","n","n","n","m","m","n","n","m","n","m","m","m","n","n","m","m","n","n","n","n","m","m","m","m","n","n","m","m","m","n","m","m","m","n","n","n","n","n","n","n","m","m","n","m","n","m","m","n","m","m","m"],["m","n","n",null,"m","m","m","m","n","n","n","m","m","m","m","n","n","m","n","m","n","m","m","n","m","m","m","m","n","m","m","m","m","m",null,"m","m","m","n","m","n","m","m","n","m","m","n","m","m","n",null,"n","m","m","n","n","m","n","m","n","m","n","n","m","n","n","n","n","n","m","n","m","m","n","m","n","m","m","n","m","m","m","n","n","n","n","n","n","m","m","m","n","m","n","m","m","n","n","m","m"],[0,1,1,-1,1,1,1,1,1,1,1,1,1,0,1,1,1,1,0,1,1,1,1,1,1,0,1,0,1,1,1,1,0,1,-1,0,1,0,1,1,1,0,1,1,1,1,1,1,1,1,-1,1,1,1,1,1,1,1,1,0,1,1,1,1,0,1,1,1,1,1,0,1,1,1,0,0,1,1,1,1,1,1,1,1,1,1,1,1,0,1,1,1,1,1,1,1,1,0,1,1],[0.7561,0.7043,0.9903,1.042,0.8207,0.7547,0.5429,0.9009,0.9551,0.6952,0.7593,0.7135,0.5656,0.5357,0.8078,0.96,0.6661,0.6962,0.8803,0.5785,0.7845,0.8146,0.6548,0.8789,0.7131,0.8211,0.8033,0.6294,0.8095,0.6176,0.7917,0.5559,0.96,0.5381,1.042,0.8264,0.7125,0.4609,0.8027,0.7808,0.8749,0.871,0.6512,0.7554,0.6715,0.9076,0.5997,0.5218,0.7319,0.764,1.042,0.7443,0.5104,0.7706,0.6026,0.7648,0.7109,0.827,0.8571,0.8092,0.7213,0.8195,0.8896,0.5778,0.8799,0.666,1.0081,0.8562,0.5444,0.6625,0.6846,0.9027,0.8236,0.6137,0.7338,0.7419,0.684,0.6062,0.7842,0.5824,0.4126,0.5006,0.8507,0.9069,0.721,0.6351,0.8553,0.7194,0.5214,0.6597,0.8097,0.6738,0.534,0.6141,0.6462,0.7763,0.6964,0.8805,0.5546,0.9747]],"container":"<table class=\"display fill-container\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>Date<\/th>\n      <th>Prac<\/th>\n      <th>Sub<\/th>\n      <th>Age<\/th>\n      <th>Sex<\/th>\n      <th>Hand<\/th>\n      <th>Block<\/th>\n      <th>Bin<\/th>\n      <th>Trial<\/th>\n      <th>Shape<\/th>\n      <th>Label<\/th>\n      <th>Match<\/th>\n      <th>CorrResp<\/th>\n      <th>Resp<\/th>\n      <th>ACC<\/th>\n      <th>RT<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"pageLength":7,"columnDefs":[{"className":"dt-right","targets":[3,4,7,8,9,15,16]},{"orderable":false,"targets":0}],"order":[],"autoWidth":false,"orderClasses":false,"lengthMenu":[7,10,25,50,100]}},"evals":[],"jsHooks":[]}</script>
```

### lapply思路

但是，由于这是一个常见的操作，我们开发了一个更简洁的功能，即使用管道操作符“%>%”来代替for循环。这个符号在DPLYR包中，它将前一步的结果作为下一个函数的输入。我们可以看到，每个符号代表一个操作步骤。我们首先列出符合条件的文件名，并将其作为一个列表输入到lapply函数中。lapply函数是一个apply系列的函数，它将函数应用于一个列表上。在这里，我们将read.table函数应用于列表中的每个元素，其中文件路径是x的参数，head=true是read.table的参数。这个操作实际上就是用一行命令代替了for循环的操作。


```r
# 获取所有的.out文件名
df.mt.out.la <- list.files(file.path("data/match"), pattern = "data_exp7_rep_match_.*\\.out$") %>%
  # 对读取到的所有.out文件x都执行函数read.table
  lapply(.,function(x) read.table(file.path("data/match", x), header = TRUE)) %>% 
  # 对所有被read.table处理过的数据执行dplyr的清洗
  lapply(function(df) dplyr::filter(df, Date != "Date") %>% # 因为有些.out文件中部还有变量名，所需需要用filter把这些行过滤掉
                      dplyr::mutate(Date = as.character(Date),Prac = as.character(Prac),
                                    Sub = as.numeric(Sub),Age = as.numeric(Age),Sex = as.character(Sex),Hand = as.character(Hand),
                                    Block = as.numeric(Block),Bin = as.numeric(Bin),Trial = as.numeric(Trial),
                                    Shape = as.character(Shape),Label = as.character(Shape),Match = as.character(Match),
                                    CorrResp = as.character(CorrResp),Resp = as.character(Resp),
                                    ACC = as.numeric(ACC),RT = as.numeric(RT)
                                    ) # 有些文件里读出来的数据格式不同，在这里统一所有out文件中的数据格式
         ) %>%
  bind_rows()
```

最终，我们得到一个包含所有数据框的列表，这些数据框都经过了一系列的转换和操作，然后使用“bind_rows”将它们合并成一个数据框。虽然这个过程看起来很冗长，但思路清晰。最终，我们得到一个包含16个变量和25000多个观测值的数据框。在读取数据后，我们通常会保存中间结果，以便下次使用。我们可以再次运行这个过程，但其实重新做一遍也无妨，因为数据基本上不会变。你只需要每次运行上面的代码，然后直接使用write.csv将读取的数据保存下来。write.csv很简单，只需要将csv文件写下来即可。我们前面整理并合并后的数据框需要一个名字和路径。我们可以使用相对路径，在当前工作目录下的data/match文件夹中将其保存为match-row.csv。一个常用的参数是强制将行名写入文件中，即row names等于false。


```r
#for loop 或 lapply的都可以
write.csv(df.mt.out.fl, file = "./data/match/match_raw.csv",row.names = FALSE)
#write.csv(df.mt.out.la, file = "./data/match/match_raw.csv",row.names = FALSE)
```

## 数据预处理

假设我们已经准备好了match和penguin的数据，我们可以读取数据并开始今天的数据预处理。我们使用刚才保存的csv文件，使用head等于true和separator等于逗号来处理数据。

在tidyverse中，filter和mutate是常用的功能之一。groupby可以根据某些变量对数据进行分组，但一定要记得使用ungroup。使用summarize可以计算均值、标准差、标准误等统计量。将groupby和summarize结合起来，我们可以快速有效地得到心理学中常用的统计量，如均值、标准差、标准误和计分数等。我们可以对数据进行分组和条件筛选。刚才提到的“ungroup”是指在“summarize”之后取消分组。而“select”函数是用来选择列，与“filter”函数选择行不同。有时我们也可以用“select”函数重新排序，而“arrange”函数则是用来对整个数据框按某一列的值进行排序。

接下来我们来看一个例子，假设我们要选择1995年或之后出生的人，我们可以在管道中使用“filter”函数。管道操作有两种方式，一种是直接从输入开始，另一种是用管道符号连接函数。在“dplyr”中，函数的参数是有顺序的，第一个是数据框。我们可以用点来代表输入数据，然后用“age”作为筛选条件。筛选完后，我们可以把结果输入到下一个函数中。如果要保留结果，我们需要把它复制到一个新的变量中。比如，我们可以选择与肃清障碍相关的问卷。


```r
# 读取原始数据
df.pg.raw <-  read.csv('./data/penguin/penguin_rawdata.csv',
                       header = T, sep=",", stringsAsFactors = FALSE)
# 使用select选择age和ALEX的所有题目
df.clean.select <- df.pg.raw %>%
  dplyr::select(age, starts_with("ALEX"), eatdrink, avoidance)
#笨一点的方法，就是把16个ALEX都写出来
```

```{=html}
<div id="htmlwidget-23738b5c18f6b6e484b1" style="width:100%;height:auto;" class="datatables html-widget"></div>
<script type="application/json" data-for="htmlwidget-23738b5c18f6b6e484b1">{"x":{"filter":"none","vertical":false,"fillContainer":true,"data":[["1","2","3","4","5","6","7","8","9","10"],[1975,1995,1995,1988,1991,1995,1996,1973,1996,1996],[1,2,4,2,2,2,1,3,2,3],[1,2,1,3,1,3,1,3,3,3],[1,2,1,4,1,1,1,2,1,1],[2,2,2,5,5,3,1,3,3,2],[2,2,4,3,2,2,4,4,2,4],[1,2,1,4,2,1,1,4,2,1],[2,2,4,2,3,1,2,4,2,2],[2,2,1,4,4,2,1,4,2,4],[4,4,1,4,1,1,4,4,2,4],[1,2,2,3,1,2,2,5,2,1],[1,2,1,2,2,1,1,3,1,4],[2,2,1,2,1,2,2,2,2,1],[4,2,2,4,2,3,4,2,2,2],[3,2,1,2,4,4,3,3,4,3],[4,3,1,4,4,5,5,4,3,5],[2,2,1,3,2,4,5,1,2,4],[1,1,1,2,1,1,1,1,1,1],[3.27777777777778,3,1.61111111111111,3.94444444444444,4.94444444444444,3.77777777777778,6.44444444444444,3.22222222222222,4,5.27777777777778]],"container":"<table class=\"display fill-container\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>age<\/th>\n      <th>ALEX1<\/th>\n      <th>ALEX2<\/th>\n      <th>ALEX3<\/th>\n      <th>ALEX4<\/th>\n      <th>ALEX5<\/th>\n      <th>ALEX6<\/th>\n      <th>ALEX7<\/th>\n      <th>ALEX8<\/th>\n      <th>ALEX9<\/th>\n      <th>ALEX10<\/th>\n      <th>ALEX11<\/th>\n      <th>ALEX12<\/th>\n      <th>ALEX13<\/th>\n      <th>ALEX14<\/th>\n      <th>ALEX15<\/th>\n      <th>ALEX16<\/th>\n      <th>eatdrink<\/th>\n      <th>avoidance<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"pageLength":3,"columnDefs":[{"className":"dt-right","targets":[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19]},{"orderable":false,"targets":0}],"order":[],"autoWidth":false,"orderClasses":false,"lengthMenu":[3,10,25,50,100]}},"evals":[],"jsHooks":[]}</script>
```

startwith是tidyverse中的一个包，可以方便地选择以“Alex”开头的所有列。它本质上是一个简化的通配符，因为在DPIY中，我们经常需要选择以某个特定开头或结尾的列，如果每次都写通配符会很麻烦。使用startwith包可以直接选择以“Alex”开头的列，并得到一串字符，可以和其他选择一起使用。

使用mutate函数可以生成一个新的变量，不仅仅是求和，还可以进行任意转换，比如加减乘除或判断。在这里，我们使用mutate函数将前四个“Alex”列的得分求和，得到一个新的变量“Alex3”，表示前四个“Alex”列的得分总和。


```r
# 把ALEX1 - 4求和
df.clean.mutate_1 <- df.pg.raw %>% 
  dplyr::mutate(ALEX_SUM = ALEX1 + ALEX2 + ALEX3 + ALEX4)
```

```{=html}
<div id="htmlwidget-a6c000540f5d3ee605de" style="width:100%;height:auto;" class="datatables html-widget"></div>
<script type="application/json" data-for="htmlwidget-a6c000540f5d3ee605de">{"x":{"filter":"none","vertical":false,"fillContainer":true,"data":[["1","2","3","4","5","6","7","8","9","10"],[1975,1995,1995,1988,1991,1995,1996,1973,1996,1996],[1,2,4,2,2,2,1,3,2,3],[1,2,1,3,1,3,1,3,3,3],[1,2,1,4,1,1,1,2,1,1],[2,2,2,5,5,3,1,3,3,2],[2,2,4,3,2,2,4,4,2,4],[1,2,1,4,2,1,1,4,2,1],[2,2,4,2,3,1,2,4,2,2],[2,2,1,4,4,2,1,4,2,4],[4,4,1,4,1,1,4,4,2,4],[1,2,2,3,1,2,2,5,2,1],[1,2,1,2,2,1,1,3,1,4],[2,2,1,2,1,2,2,2,2,1],[4,2,2,4,2,3,4,2,2,2],[3,2,1,2,4,4,3,3,4,3],[4,3,1,4,4,5,5,4,3,5],[2,2,1,3,2,4,5,1,2,4],[3.72222222222222,4.05555555555556,1.44444444444444,4.72222222222222,2.11111111111111,2,2.55555555555556,3.05555555555556,3.55555555555556,4.38888888888889],[6,0,0,2,0,0,null,0,0,0],[3.11111111111111,4.88888888888889,1.33333333333333,3.88888888888889,4.66666666666667,3.33333333333333,3.11111111111111,3.55555555555556,3.33333333333333,3.22222222222222],[3.44444444444444,2,2,3,1.77777777777778,2.44444444444444,3.77777777777778,3.33333333333333,1.88888888888889,4.22222222222222],[89,89,89,89,89,89,89,89,89,89],[3.27777777777778,3,1.61111111111111,3.94444444444444,4.94444444444444,3.77777777777778,6.44444444444444,3.22222222222222,4,5.27777777777778],[15,null,null,null,null,null,null,10,null,1],[1.63636363636364,2.18181818181818,2,3.27272727272727,2.18181818181818,1.72727272727273,1.72727272727273,3.54545454545455,2,2.63636363636364],[1,1,1,2,1,1,1,1,1,1],[5,6,1,3,1,1,2,3,4,5],[5,4,1,6,1,1,1,2,4,6],[3,2,1,6,1,1,4,3,4,6],[5,5,1,5,1,2,1,2,4,5],[5,5,1,4,1,2,4,3,4,4],[3,2,1,4,5,1,1,3,1,3],[1,3,1,3,2,1,2,3,4,5],[6,5,1,3,1,2,7,3,4,7],[5,2,1,4,7,6,1,3,4,3],[2,3,1,4,1,2,4,1,4,3],[6,7,1,6,6,2,2,5,2,6],[3,4,1,5,1,2,1,2,4,4],[3,3,1,6,2,4,2,5,4,4],[2,3,7,5,1,1,1,4,1,2],[3,4,1,5,2,2,1,5,4,1],[3,5,3,5,1,1,2,3,4,4],[6,5,1,6,2,3,6,3,4,7],[1,5,1,5,2,2,4,2,4,4],[2,3,2,5,1,5,7,3,4,5],[2,4,2,5,7,5,1,2,4,7],[5,3,2,5,2,4,6,4,4,5],[3,2,2,4,5,4,7,3,4,5],[3,2,2,3,2,3,7,3,4,6],[2,3,1,4,4,4,7,3,4,5],[5,3,1,6,2,3,7,4,4,6],[5,4,1,3,6,3,7,4,4,7],[3,5,1,3,7,1,7,4,4,7],[2,3,1,4,6,3,6,3,4,4],[4,2,2,5,6,6,6,3,4,4],[2,2,2,3,6,5,7,2,4,5],[3,4,1,4,7,4,7,2,4,3],[3,2,2,5,2,3,7,3,4,6],[4,3,3,3,7,4,6,3,4,5],[5,3,1,2,6,5,7,4,4,6],[4,3,1,3,6,3,7,6,4,5],[2,3,2,4,7,3,7,2,4,4],["9:23:38","9:23:26","8:57:08","8:55:14","8:54:35","8:39:12","8:32:08","8:28:57","8:28:03","8:25:57"],[3,2.2,1.2,3,2.6,3.6,3.8,2.4,2.6,3],[2,2,2,2,2,2,2,2,2,2],[0,0,12,0,5,0,null,1,0,2],[4,4,4,3,4,4,4,1,3,3],[4,5,1,5,5,4,5,3,3,4],[3,5,1,4,5,4,2,3,3,5],[4,5,4,5,5,4,4,2,4,3],[3,5,1,5,5,4,5,4,5,3],[3,5,1,4,4,4,2,3,4,4],[2,4,1,2,5,2,1,4,3,2],[4,5,1,2,4,2,5,5,2,2],[3,5,1,4,4,3,1,4,3,3],[2,5,1,4,5,3,3,4,3,3],[4,1,5,5,1,3,2,3,3,4],[4,2,6,4,1,3,2,3,3,5],[5,6,5,3,3,3,2,2,2,1],[4,2,4,1,2,2,2,2,1,3],[4,3,3,2,2,2,2,2,3,2],[1,2,1,2,2,4,2,3,3,3],[2,2,1,2,1,4,2,3,1,2],[17,28,24,19,29,33,53,17,17,19],[4.33333333333333,6.5,6.83333333333333,3.5,1.5,2,5.83333333333333,6.33333333333333,5.66666666666667,5.16666666666667],[2.27272727272727,2.63636363636364,2.18181818181818,2,1.90909090909091,2.45454545454545,2.81818181818182,2.54545454545455,1.81818181818182,3.90909090909091],[2,3,3,1,3,4,4,2,2,5],[2,2,4,1,2,4,4,3,3,4],[2,3,2,2,1,2,2,3,2,5],[4,3,4,3,4,4,2,2,2,4],[2,3,1,2,1,3,3,3,2,3],[2,2,1,2,1,1,2,2,1,3],[2,2,1,2,1,2,2,4,2,4],[2,3,3,1,1,1,2,3,2,3],[3,3,3,2,2,2,4,2,1,3],[2,3,1,4,2,1,2,1,1,4],[2,2,1,2,3,3,4,3,2,5],[4,2,1,2,2,3,3,3,3,4],[4,2,4,4,1,3,4,4,2,4],[2,2,1,3,1,2,5,3,1,2],[5,2,4,2,1,3,4,3,4,4],[2,2,1,3,1,2,1,2,3,5],[2,2,1,2,3,2,3,4,1,4],[4,2,1,3,2,2,5,3,1,5],[4,2,1,4,1,3,5,4,1,5],[4,2,4,4,4,2,4,4,1,5],[2,2,1,1,2,2,2,2,2,2],[2,4,2,4,3,5,2,1,3,3],[2,4,3,2,5,5,4,4,3,4],[2,4,3,1,5,5,2,5,5,5],[1,4,2,1,3,2,2,2,5,1],[1,4,4,2,5,3,1,2,5,2],[4,3,2,1,5,4,3,3,3,1],[2,4,4,2,5,3,2,3,4,2],[2,2,1,3,4,3,2,2,5,4],[5,4,3,4,4,5,4,4,3,3],[4,3,5,3,3,5,5,3,4,2],[3,4,4,2,4,5,3,2,2,5],[5,5,4,2,4,4,1,3,4,3],[4,4,4,2,4,3,2,2,2,2],[2.84615384615385,3.76923076923077,3.15384615384615,2.23076923076923,4.15384615384615,4,2.53846153846154,2.76923076923077,3.69230769230769,2.84615384615385],[1,1,1,2,1,1,2,1,1,2],[1,2,2,2,2,2,2,1,2,1],[2,1,1,1,2,2,2,4,2,2],[1,1,1,1,1,1,1,1,1,1],[null,null,null,null,null,null,null,null,null,null],[4,4,4,4,2,4,4,4,4,4],[4,4,4,4,2,4,4,2,4,4],[5,1,4,5,5,5,5,5,5,5],[null,null,4,null,null,null,null,null,null,null],[1,8,2,1,8,2,3,8,2,6],[null,6,2,null,5,1,2,3,2,3],[6,4,5,5,7,5,8,4,3,8],[3,1,5,1,3,5,8,4,2,8],[1,1,1,1,2,1,1,1,1,1],[null,null,null,null,8,null,null,null,null,null],[1,2,2,1,1,2,2,2,2,2],[null,8,8,null,null,8,8,8,8,8],[3,3,2,3,3,1,1,1,1,1],[2,8,1,8,1,null,null,null,null,null],[6,5,6,4,8,null,null,null,null,null],[1,3,1,1,1,8,6,4,1,3],[1,1,1,1,1,1,1,2,1,1],[null,null,null,null,null,null,null,2,null,null],[1,1,1,2,1,2,2,2,2,1],[" "," "," "," "," ","Rowing Club","University Friends","ODAA","Oxford Latin Speaking Society"," "],[" "," "," "," "," ","Rugby Club","Business Social"," ","Oxford Latin Conversation Society"," "],[" "," "," "," "," "," ","Lecturers/ Advisors"," ","Plato Reading Group"," "],[" "," "," "," "," "," ","Friends from home"," "," "," "],[" "," "," "," "," "," ","Neighbors"," "," "," "],[null,null,null,null,null,7,20,1,2,null],[null,null,null,null,null,7,15,null,4,null],[null,null,null,null,null,null,5,null,1,null],[null,null,null,null,null,null,9,null,null,null],[null,null,null,null,null,null,6,null,null,null],[6,6,7,2,1,1,6,6,7,7],[5,7,7,5,2,2,7,6,7,4],[5,6,7,3,2,3,5,7,6,5],[3,7,7,4,1,2,5,6,6,5],[2,6,7,5,2,2,6,7,5,5],[5,7,6,2,1,2,6,6,3,5],[4,6,6,4,1,2,5,6,3,4],[5,7,8,5,7,6,null,7,6,6],[1,2,3,1,2,4,null,2,2,3],[4,2,4,4,1,2,2,3,3,1],[4,4,5,2,1,4,4,2,5,4],[4,3,4,3,1,4,2,3,4,4],[5,3,4,3,4,4,4,4,2,4],[2,3,4,4,4,4,5,2,2,3],[2,4,4,3,1,3,4,3,1,3],[2,5,4,3,3,4,5,4,2,2],[2,2,4,3,4,3,5,3,2,4],[4,5,5,2,1,2,4,2,4,5],[2,3,2,2,1,4,1,4,4,5],[2,2,1,2,1,2,2,2,2,1],[1,2,2,2,4,4,4,2,2,2],[4,4,4,3,4,2,2,3,2,1],[5,4,2,3,2,4,3,4,4,4],[4,4,2,3,3,1,2,4,5,3],[2,3,4,4,4,5,5,4,4,4],[2,3,5,4,4,3,2,4,2,2],[4,3,4,2,4,3,4,3,3,2],[3,3,5,2,4,4,4,4,3,3],[2,3,1,3,5,4,1,3,2,4],[4,3,4,2,4,4,5,4,3,5],[3,3,5,4,4,3,2,2,3,3],[2,3,1,3,4,3,2,3,4,3],[3,3,1,4,1,2,2,2,3,3],[4,3,5,3,1,2,5,1,4,4],[3,3,3,2,1,3,3,2,3,3],[4,3,1,4,4,4,2,4,1,1],[4,5,4,2,1,3,1,4,4,5],[3,5,4,4,1,2,1,3,5,5],[3,2,2,4,4,2,5,2,2,4],[2,4,2,5,1,2,1,3,2,4],[4,3,1,2,4,2,5,1,1,4],[4,3,2,2,3,3,1,4,3,4],[1,4,3,4,4,2,1,3,2,2],[2,4,2,3,5,4,2,5,1,4],[2,4,2,4,5,4,4,4,2,4],[4,4,4,3,4,4,2,4,2,4],[3,4,2,4,4,4,4,5,5,1],[4,2,2,3,4,4,5,2,2,4],[5,3,3,2,4,2,2,2,2,4],[4,3,2,3,3,3,2,1,2,4],[4,3,5,2,4,4,4,4,4,4],[1,4,2,3,4,2,2,3,2,3],[1,2,2,4,4,2,1,2,1,2],[4,3,2,3,1,2,1,4,4,2],[4,4,4,2,1,4,1,4,3,3],[3,3,4,3,4,2,4,3,3,4],[1,2,5,4,4,2,1,2,1,2],[2,3,5,4,4,2,1,2,1,2],[4,3,5,4,4,4,3,3,3,4],[4,4,2,5,1,3,5,2,4,5],[2,4,5,3,1,2,5,4,2,4],[2,4,4,3,4,2,1,3,2,2],[4,3,4,3,5,2,4,4,2,4],[2,4,2,2,4,3,3,2,2,4],[2,4,1,2,4,2,2,2,2,2],[1,2,5,4,1,1,4,2,2,2],[3.07692307692308,2.46153846153846,2.38461538461538,2.84615384615385,2.53846153846154,2.07692307692308,1.53846153846154,3,2.61538461538462,3.15384615384615],[4,2,3,3,3,2,1,4,5,3],[2,3,3,3,4,2,2,3,2,4],[4,3,4,4,3,3,1,3,3,5],[4,3,1,2,2,2,2,3,2,4],[4,2,2,3,2,2,2,3,2,3],[3,2,1,2,1,2,2,3,2,1],[4,2,2,3,3,2,1,4,2,3],[3,2,2,2,2,2,1,4,3,4],[3,2,3,3,2,2,2,1,2,3],[2,3,3,3,3,2,1,3,4,3],[4,2,3,4,3,2,2,2,1,2],[3,4,4,3,5,4,4,4,5,4],[2,4,2,3,2,2,2,3,3,2],[1,2,2,2,3,2,1,3,3,4],[34.9,35.8,34,36.9,35.3,34.7,36,36.5,34.7,35.9],[35.9,36,36.2,37,36.1,36.4,36.3,36.6,36.3,36.3],[2,2,2,2,4,4,2,2,2,2],[35.4,35.9,35.1,36.95,35.7,35.55,36.15,36.55,35.5,36.1],[1,1,1,1,1,1,1,1,1,1],[-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667],[5,5,5,5,5,5,5,5,5,5],[1,1,1,1,1,1,1,1,1,1],[0.186319647739787,0.472682850177291,-1.77049556891649,1.0454092550523,-1.19776916404149,-1.29322356485399,-0.815951560791479,-0.386406757135223,0.0431380465210342,0.759046052614796],[-0.00190690952064737,-0.249805147204807,-1.48929633562561,0.593048860921336,1.48548251658431,0.44430991831084,2.82413300007877,-0.0514865570574791,0.642628508458168,1.7829604018053],["Oxford","Oxford","Oxford","Oxford","Oxford","Oxford","Oxford","Oxford","Oxford","Oxford"],[5,8,8,14,9,9,4,11,9,9]],"container":"<table class=\"display fill-container\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>age<\/th>\n      <th>ALEX1<\/th>\n      <th>ALEX2<\/th>\n      <th>ALEX3<\/th>\n      <th>ALEX4<\/th>\n      <th>ALEX5<\/th>\n      <th>ALEX6<\/th>\n      <th>ALEX7<\/th>\n      <th>ALEX8<\/th>\n      <th>ALEX9<\/th>\n      <th>ALEX10<\/th>\n      <th>ALEX11<\/th>\n      <th>ALEX12<\/th>\n      <th>ALEX13<\/th>\n      <th>ALEX14<\/th>\n      <th>ALEX15<\/th>\n      <th>ALEX16<\/th>\n      <th>anxiety<\/th>\n      <th>artgluctot<\/th>\n      <th>attachhome<\/th>\n      <th>attachphone<\/th>\n      <th>AvgHumidity<\/th>\n      <th>avoidance<\/th>\n      <th>cigs<\/th>\n      <th>DIDF<\/th>\n      <th>eatdrink<\/th>\n      <th>ECR1<\/th>\n      <th>ECR2<\/th>\n      <th>ECR3<\/th>\n      <th>ECR4<\/th>\n      <th>ECR5<\/th>\n      <th>ECR6<\/th>\n      <th>ECR7<\/th>\n      <th>ECR8<\/th>\n      <th>ECR9<\/th>\n      <th>ECR10<\/th>\n      <th>ECR11<\/th>\n      <th>ECR12<\/th>\n      <th>ECR13<\/th>\n      <th>ECR14<\/th>\n      <th>ECR15<\/th>\n      <th>ECR16<\/th>\n      <th>ECR17<\/th>\n      <th>ECR18<\/th>\n      <th>ECR19<\/th>\n      <th>ECR20<\/th>\n      <th>ECR21<\/th>\n      <th>ECR22<\/th>\n      <th>ECR23<\/th>\n      <th>ECR24<\/th>\n      <th>ECR25<\/th>\n      <th>ECR26<\/th>\n      <th>ECR27<\/th>\n      <th>ECR28<\/th>\n      <th>ECR29<\/th>\n      <th>ECR30<\/th>\n      <th>ECR31<\/th>\n      <th>ECR32<\/th>\n      <th>ECR33<\/th>\n      <th>ECR34<\/th>\n      <th>ECR35<\/th>\n      <th>ECR36<\/th>\n      <th>endtime<\/th>\n      <th>EOT<\/th>\n      <th>exercise<\/th>\n      <th>gluctot<\/th>\n      <th>health<\/th>\n      <th>HOME1<\/th>\n      <th>HOME2<\/th>\n      <th>HOME3<\/th>\n      <th>HOME4<\/th>\n      <th>HOME5<\/th>\n      <th>HOME6<\/th>\n      <th>HOME7<\/th>\n      <th>HOME8<\/th>\n      <th>HOME9<\/th>\n      <th>KAMF1<\/th>\n      <th>KAMF2<\/th>\n      <th>KAMF3<\/th>\n      <th>KAMF4<\/th>\n      <th>KAMF5<\/th>\n      <th>KAMF6<\/th>\n      <th>KAMF7<\/th>\n      <th>networksize<\/th>\n      <th>nostalgia<\/th>\n      <th>onlineid<\/th>\n      <th>onlineid1<\/th>\n      <th>onlineid2<\/th>\n      <th>onlineid3<\/th>\n      <th>onlineid4<\/th>\n      <th>onlineid5<\/th>\n      <th>onlineid6<\/th>\n      <th>onlineid7<\/th>\n      <th>onlineid8<\/th>\n      <th>onlineid9<\/th>\n      <th>onlineid10<\/th>\n      <th>onlineide11<\/th>\n      <th>phone1<\/th>\n      <th>phone2<\/th>\n      <th>phone3<\/th>\n      <th>phone4<\/th>\n      <th>phone5<\/th>\n      <th>phone6<\/th>\n      <th>phone7<\/th>\n      <th>phone8<\/th>\n      <th>phone9<\/th>\n      <th>romantic<\/th>\n      <th>scontrol1<\/th>\n      <th>scontrol2<\/th>\n      <th>scontrol3<\/th>\n      <th>scontrol4<\/th>\n      <th>scontrol5<\/th>\n      <th>scontrol6<\/th>\n      <th>scontrol7<\/th>\n      <th>scontrol8<\/th>\n      <th>scontrol9<\/th>\n      <th>scontrol10<\/th>\n      <th>scontrol11<\/th>\n      <th>scontrol12<\/th>\n      <th>scontrol13<\/th>\n      <th>selfcontrol<\/th>\n      <th>sex<\/th>\n      <th>smoke<\/th>\n      <th>SNI1<\/th>\n      <th>SNI2<\/th>\n      <th>SNI3<\/th>\n      <th>SNI4<\/th>\n      <th>SNI5<\/th>\n      <th>SNI6<\/th>\n      <th>SNI7<\/th>\n      <th>SNI8<\/th>\n      <th>SNI9<\/th>\n      <th>SNI10<\/th>\n      <th>SNI11<\/th>\n      <th>SNI12<\/th>\n      <th>SNI13<\/th>\n      <th>SNI14<\/th>\n      <th>SNI15<\/th>\n      <th>SNI16<\/th>\n      <th>SNI17<\/th>\n      <th>SNI18<\/th>\n      <th>SNI19<\/th>\n      <th>SNI20<\/th>\n      <th>SNI21<\/th>\n      <th>SNI22<\/th>\n      <th>SNI23<\/th>\n      <th>SNI24<\/th>\n      <th>SNI25<\/th>\n      <th>SNI26<\/th>\n      <th>SNI27<\/th>\n      <th>SNI28<\/th>\n      <th>SNI29<\/th>\n      <th>SNI30<\/th>\n      <th>SNI31<\/th>\n      <th>SNI32<\/th>\n      <th>SNS1<\/th>\n      <th>SNS2<\/th>\n      <th>SNS3<\/th>\n      <th>SNS4<\/th>\n      <th>SNS5<\/th>\n      <th>SNS6<\/th>\n      <th>SNS7<\/th>\n      <th>socialdiversity<\/th>\n      <th>socialembedded<\/th>\n      <th>STRAQ_1<\/th>\n      <th>STRAQ_2<\/th>\n      <th>STRAQ_3<\/th>\n      <th>STRAQ_4<\/th>\n      <th>STRAQ_6<\/th>\n      <th>STRAQ_7<\/th>\n      <th>STRAQ_8<\/th>\n      <th>STRAQ_9<\/th>\n      <th>STRAQ_10<\/th>\n      <th>STRAQ_11<\/th>\n      <th>STRAQ_12<\/th>\n      <th>STRAQ_19<\/th>\n      <th>STRAQ_20<\/th>\n      <th>STRAQ_21<\/th>\n      <th>STRAQ_22<\/th>\n      <th>STRAQ_23<\/th>\n      <th>STRAQ_24<\/th>\n      <th>STRAQ_25<\/th>\n      <th>STRAQ_26<\/th>\n      <th>STRAQ_27<\/th>\n      <th>STRAQ_28<\/th>\n      <th>STRAQ_29<\/th>\n      <th>STRAQ_30<\/th>\n      <th>STRAQ_31<\/th>\n      <th>STRAQ_32<\/th>\n      <th>STRAQ_33<\/th>\n      <th>STRAQ_5<\/th>\n      <th>STRAQ_13<\/th>\n      <th>STRAQ_14<\/th>\n      <th>STRAQ_15<\/th>\n      <th>STRAQ_16<\/th>\n      <th>STRAQ_17<\/th>\n      <th>STRAQ_18<\/th>\n      <th>STRAQ_34<\/th>\n      <th>STRAQ_35<\/th>\n      <th>STRAQ_36<\/th>\n      <th>STRAQ_37<\/th>\n      <th>STRAQ_38<\/th>\n      <th>STRAQ_39<\/th>\n      <th>STRAQ_40<\/th>\n      <th>STRAQ_41<\/th>\n      <th>STRAQ_42<\/th>\n      <th>STRAQ_43<\/th>\n      <th>STRAQ_44<\/th>\n      <th>STRAQ_45<\/th>\n      <th>STRAQ_46<\/th>\n      <th>STRAQ_47<\/th>\n      <th>STRAQ_48<\/th>\n      <th>STRAQ_49<\/th>\n      <th>STRAQ_50<\/th>\n      <th>STRAQ_51<\/th>\n      <th>STRAQ_52<\/th>\n      <th>STRAQ_53<\/th>\n      <th>STRAQ_54<\/th>\n      <th>STRAQ_55<\/th>\n      <th>STRAQ_56<\/th>\n      <th>STRAQ_57<\/th>\n      <th>stress<\/th>\n      <th>stress1<\/th>\n      <th>stress2<\/th>\n      <th>stress3<\/th>\n      <th>stress4<\/th>\n      <th>stress5<\/th>\n      <th>stress6<\/th>\n      <th>stress7<\/th>\n      <th>stress8<\/th>\n      <th>stress9<\/th>\n      <th>stress10<\/th>\n      <th>stress11<\/th>\n      <th>stress12<\/th>\n      <th>stress13<\/th>\n      <th>stress14<\/th>\n      <th>Temperature_t1<\/th>\n      <th>Temperature_t2<\/th>\n      <th>thermotype<\/th>\n      <th>avgtemp<\/th>\n      <th>filter_.<\/th>\n      <th>mintemp<\/th>\n      <th>language<\/th>\n      <th>langfamily<\/th>\n      <th>Zanxiety<\/th>\n      <th>Zavoidance<\/th>\n      <th>Site<\/th>\n      <th>ALEX_SUM<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"pageLength":3,"columnDefs":[{"className":"dt-right","targets":[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,57,58,59,60,61,62,64,65,66,67,68,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86,87,88,89,90,91,92,93,94,95,96,97,98,99,100,101,102,103,104,105,106,107,108,109,110,111,112,113,114,115,116,117,118,119,120,121,122,123,124,125,126,127,128,129,130,131,132,133,134,135,136,137,138,139,140,141,142,143,144,145,151,152,153,154,155,156,157,158,159,160,161,162,163,164,165,166,167,168,169,170,171,172,173,174,175,176,177,178,179,180,181,182,183,184,185,186,187,188,189,190,191,192,193,194,195,196,197,198,199,200,201,202,203,204,205,206,207,208,209,210,211,212,213,214,215,216,217,218,219,220,221,222,223,224,225,226,227,228,229,230,231,232,233,234,235,236,237,238,239,240,241,242,243,244,245,246,248]},{"orderable":false,"targets":0}],"order":[],"autoWidth":false,"orderClasses":false,"lengthMenu":[3,10,25,50,100]}},"evals":[],"jsHooks":[]}</script>
```

需要注意的是，它是逐行运算，可以使用其他函数如rowSums，其中也用到了通配符。我们可以利用DPRY中的筛选功能，选取所有以Alex为开头的列，并对这些列进行逐行求和，以得到真正反映Alex所有项目的总和。这种方法将给出16个项目的总和，而前面提到的方法只有4个。


```r
# 对所有含有ALEX的列求和
df.clean.mutate_2 <- df.pg.raw %>% 
  dplyr::mutate(ALEX_SUM = rowSums(select(., starts_with("ALEX"))))
```

```{=html}
<div id="htmlwidget-9e12ba267078af454a9b" style="width:100%;height:auto;" class="datatables html-widget"></div>
<script type="application/json" data-for="htmlwidget-9e12ba267078af454a9b">{"x":{"filter":"none","vertical":false,"fillContainer":true,"data":[["1","2","3","4","5","6","7","8","9","10"],[1975,1995,1995,1988,1991,1995,1996,1973,1996,1996],[1,2,4,2,2,2,1,3,2,3],[1,2,1,3,1,3,1,3,3,3],[1,2,1,4,1,1,1,2,1,1],[2,2,2,5,5,3,1,3,3,2],[2,2,4,3,2,2,4,4,2,4],[1,2,1,4,2,1,1,4,2,1],[2,2,4,2,3,1,2,4,2,2],[2,2,1,4,4,2,1,4,2,4],[4,4,1,4,1,1,4,4,2,4],[1,2,2,3,1,2,2,5,2,1],[1,2,1,2,2,1,1,3,1,4],[2,2,1,2,1,2,2,2,2,1],[4,2,2,4,2,3,4,2,2,2],[3,2,1,2,4,4,3,3,4,3],[4,3,1,4,4,5,5,4,3,5],[2,2,1,3,2,4,5,1,2,4],[3.72222222222222,4.05555555555556,1.44444444444444,4.72222222222222,2.11111111111111,2,2.55555555555556,3.05555555555556,3.55555555555556,4.38888888888889],[6,0,0,2,0,0,null,0,0,0],[3.11111111111111,4.88888888888889,1.33333333333333,3.88888888888889,4.66666666666667,3.33333333333333,3.11111111111111,3.55555555555556,3.33333333333333,3.22222222222222],[3.44444444444444,2,2,3,1.77777777777778,2.44444444444444,3.77777777777778,3.33333333333333,1.88888888888889,4.22222222222222],[89,89,89,89,89,89,89,89,89,89],[3.27777777777778,3,1.61111111111111,3.94444444444444,4.94444444444444,3.77777777777778,6.44444444444444,3.22222222222222,4,5.27777777777778],[15,null,null,null,null,null,null,10,null,1],[1.63636363636364,2.18181818181818,2,3.27272727272727,2.18181818181818,1.72727272727273,1.72727272727273,3.54545454545455,2,2.63636363636364],[1,1,1,2,1,1,1,1,1,1],[5,6,1,3,1,1,2,3,4,5],[5,4,1,6,1,1,1,2,4,6],[3,2,1,6,1,1,4,3,4,6],[5,5,1,5,1,2,1,2,4,5],[5,5,1,4,1,2,4,3,4,4],[3,2,1,4,5,1,1,3,1,3],[1,3,1,3,2,1,2,3,4,5],[6,5,1,3,1,2,7,3,4,7],[5,2,1,4,7,6,1,3,4,3],[2,3,1,4,1,2,4,1,4,3],[6,7,1,6,6,2,2,5,2,6],[3,4,1,5,1,2,1,2,4,4],[3,3,1,6,2,4,2,5,4,4],[2,3,7,5,1,1,1,4,1,2],[3,4,1,5,2,2,1,5,4,1],[3,5,3,5,1,1,2,3,4,4],[6,5,1,6,2,3,6,3,4,7],[1,5,1,5,2,2,4,2,4,4],[2,3,2,5,1,5,7,3,4,5],[2,4,2,5,7,5,1,2,4,7],[5,3,2,5,2,4,6,4,4,5],[3,2,2,4,5,4,7,3,4,5],[3,2,2,3,2,3,7,3,4,6],[2,3,1,4,4,4,7,3,4,5],[5,3,1,6,2,3,7,4,4,6],[5,4,1,3,6,3,7,4,4,7],[3,5,1,3,7,1,7,4,4,7],[2,3,1,4,6,3,6,3,4,4],[4,2,2,5,6,6,6,3,4,4],[2,2,2,3,6,5,7,2,4,5],[3,4,1,4,7,4,7,2,4,3],[3,2,2,5,2,3,7,3,4,6],[4,3,3,3,7,4,6,3,4,5],[5,3,1,2,6,5,7,4,4,6],[4,3,1,3,6,3,7,6,4,5],[2,3,2,4,7,3,7,2,4,4],["9:23:38","9:23:26","8:57:08","8:55:14","8:54:35","8:39:12","8:32:08","8:28:57","8:28:03","8:25:57"],[3,2.2,1.2,3,2.6,3.6,3.8,2.4,2.6,3],[2,2,2,2,2,2,2,2,2,2],[0,0,12,0,5,0,null,1,0,2],[4,4,4,3,4,4,4,1,3,3],[4,5,1,5,5,4,5,3,3,4],[3,5,1,4,5,4,2,3,3,5],[4,5,4,5,5,4,4,2,4,3],[3,5,1,5,5,4,5,4,5,3],[3,5,1,4,4,4,2,3,4,4],[2,4,1,2,5,2,1,4,3,2],[4,5,1,2,4,2,5,5,2,2],[3,5,1,4,4,3,1,4,3,3],[2,5,1,4,5,3,3,4,3,3],[4,1,5,5,1,3,2,3,3,4],[4,2,6,4,1,3,2,3,3,5],[5,6,5,3,3,3,2,2,2,1],[4,2,4,1,2,2,2,2,1,3],[4,3,3,2,2,2,2,2,3,2],[1,2,1,2,2,4,2,3,3,3],[2,2,1,2,1,4,2,3,1,2],[17,28,24,19,29,33,53,17,17,19],[4.33333333333333,6.5,6.83333333333333,3.5,1.5,2,5.83333333333333,6.33333333333333,5.66666666666667,5.16666666666667],[2.27272727272727,2.63636363636364,2.18181818181818,2,1.90909090909091,2.45454545454545,2.81818181818182,2.54545454545455,1.81818181818182,3.90909090909091],[2,3,3,1,3,4,4,2,2,5],[2,2,4,1,2,4,4,3,3,4],[2,3,2,2,1,2,2,3,2,5],[4,3,4,3,4,4,2,2,2,4],[2,3,1,2,1,3,3,3,2,3],[2,2,1,2,1,1,2,2,1,3],[2,2,1,2,1,2,2,4,2,4],[2,3,3,1,1,1,2,3,2,3],[3,3,3,2,2,2,4,2,1,3],[2,3,1,4,2,1,2,1,1,4],[2,2,1,2,3,3,4,3,2,5],[4,2,1,2,2,3,3,3,3,4],[4,2,4,4,1,3,4,4,2,4],[2,2,1,3,1,2,5,3,1,2],[5,2,4,2,1,3,4,3,4,4],[2,2,1,3,1,2,1,2,3,5],[2,2,1,2,3,2,3,4,1,4],[4,2,1,3,2,2,5,3,1,5],[4,2,1,4,1,3,5,4,1,5],[4,2,4,4,4,2,4,4,1,5],[2,2,1,1,2,2,2,2,2,2],[2,4,2,4,3,5,2,1,3,3],[2,4,3,2,5,5,4,4,3,4],[2,4,3,1,5,5,2,5,5,5],[1,4,2,1,3,2,2,2,5,1],[1,4,4,2,5,3,1,2,5,2],[4,3,2,1,5,4,3,3,3,1],[2,4,4,2,5,3,2,3,4,2],[2,2,1,3,4,3,2,2,5,4],[5,4,3,4,4,5,4,4,3,3],[4,3,5,3,3,5,5,3,4,2],[3,4,4,2,4,5,3,2,2,5],[5,5,4,2,4,4,1,3,4,3],[4,4,4,2,4,3,2,2,2,2],[2.84615384615385,3.76923076923077,3.15384615384615,2.23076923076923,4.15384615384615,4,2.53846153846154,2.76923076923077,3.69230769230769,2.84615384615385],[1,1,1,2,1,1,2,1,1,2],[1,2,2,2,2,2,2,1,2,1],[2,1,1,1,2,2,2,4,2,2],[1,1,1,1,1,1,1,1,1,1],[null,null,null,null,null,null,null,null,null,null],[4,4,4,4,2,4,4,4,4,4],[4,4,4,4,2,4,4,2,4,4],[5,1,4,5,5,5,5,5,5,5],[null,null,4,null,null,null,null,null,null,null],[1,8,2,1,8,2,3,8,2,6],[null,6,2,null,5,1,2,3,2,3],[6,4,5,5,7,5,8,4,3,8],[3,1,5,1,3,5,8,4,2,8],[1,1,1,1,2,1,1,1,1,1],[null,null,null,null,8,null,null,null,null,null],[1,2,2,1,1,2,2,2,2,2],[null,8,8,null,null,8,8,8,8,8],[3,3,2,3,3,1,1,1,1,1],[2,8,1,8,1,null,null,null,null,null],[6,5,6,4,8,null,null,null,null,null],[1,3,1,1,1,8,6,4,1,3],[1,1,1,1,1,1,1,2,1,1],[null,null,null,null,null,null,null,2,null,null],[1,1,1,2,1,2,2,2,2,1],[" "," "," "," "," ","Rowing Club","University Friends","ODAA","Oxford Latin Speaking Society"," "],[" "," "," "," "," ","Rugby Club","Business Social"," ","Oxford Latin Conversation Society"," "],[" "," "," "," "," "," ","Lecturers/ Advisors"," ","Plato Reading Group"," "],[" "," "," "," "," "," ","Friends from home"," "," "," "],[" "," "," "," "," "," ","Neighbors"," "," "," "],[null,null,null,null,null,7,20,1,2,null],[null,null,null,null,null,7,15,null,4,null],[null,null,null,null,null,null,5,null,1,null],[null,null,null,null,null,null,9,null,null,null],[null,null,null,null,null,null,6,null,null,null],[6,6,7,2,1,1,6,6,7,7],[5,7,7,5,2,2,7,6,7,4],[5,6,7,3,2,3,5,7,6,5],[3,7,7,4,1,2,5,6,6,5],[2,6,7,5,2,2,6,7,5,5],[5,7,6,2,1,2,6,6,3,5],[4,6,6,4,1,2,5,6,3,4],[5,7,8,5,7,6,null,7,6,6],[1,2,3,1,2,4,null,2,2,3],[4,2,4,4,1,2,2,3,3,1],[4,4,5,2,1,4,4,2,5,4],[4,3,4,3,1,4,2,3,4,4],[5,3,4,3,4,4,4,4,2,4],[2,3,4,4,4,4,5,2,2,3],[2,4,4,3,1,3,4,3,1,3],[2,5,4,3,3,4,5,4,2,2],[2,2,4,3,4,3,5,3,2,4],[4,5,5,2,1,2,4,2,4,5],[2,3,2,2,1,4,1,4,4,5],[2,2,1,2,1,2,2,2,2,1],[1,2,2,2,4,4,4,2,2,2],[4,4,4,3,4,2,2,3,2,1],[5,4,2,3,2,4,3,4,4,4],[4,4,2,3,3,1,2,4,5,3],[2,3,4,4,4,5,5,4,4,4],[2,3,5,4,4,3,2,4,2,2],[4,3,4,2,4,3,4,3,3,2],[3,3,5,2,4,4,4,4,3,3],[2,3,1,3,5,4,1,3,2,4],[4,3,4,2,4,4,5,4,3,5],[3,3,5,4,4,3,2,2,3,3],[2,3,1,3,4,3,2,3,4,3],[3,3,1,4,1,2,2,2,3,3],[4,3,5,3,1,2,5,1,4,4],[3,3,3,2,1,3,3,2,3,3],[4,3,1,4,4,4,2,4,1,1],[4,5,4,2,1,3,1,4,4,5],[3,5,4,4,1,2,1,3,5,5],[3,2,2,4,4,2,5,2,2,4],[2,4,2,5,1,2,1,3,2,4],[4,3,1,2,4,2,5,1,1,4],[4,3,2,2,3,3,1,4,3,4],[1,4,3,4,4,2,1,3,2,2],[2,4,2,3,5,4,2,5,1,4],[2,4,2,4,5,4,4,4,2,4],[4,4,4,3,4,4,2,4,2,4],[3,4,2,4,4,4,4,5,5,1],[4,2,2,3,4,4,5,2,2,4],[5,3,3,2,4,2,2,2,2,4],[4,3,2,3,3,3,2,1,2,4],[4,3,5,2,4,4,4,4,4,4],[1,4,2,3,4,2,2,3,2,3],[1,2,2,4,4,2,1,2,1,2],[4,3,2,3,1,2,1,4,4,2],[4,4,4,2,1,4,1,4,3,3],[3,3,4,3,4,2,4,3,3,4],[1,2,5,4,4,2,1,2,1,2],[2,3,5,4,4,2,1,2,1,2],[4,3,5,4,4,4,3,3,3,4],[4,4,2,5,1,3,5,2,4,5],[2,4,5,3,1,2,5,4,2,4],[2,4,4,3,4,2,1,3,2,2],[4,3,4,3,5,2,4,4,2,4],[2,4,2,2,4,3,3,2,2,4],[2,4,1,2,4,2,2,2,2,2],[1,2,5,4,1,1,4,2,2,2],[3.07692307692308,2.46153846153846,2.38461538461538,2.84615384615385,2.53846153846154,2.07692307692308,1.53846153846154,3,2.61538461538462,3.15384615384615],[4,2,3,3,3,2,1,4,5,3],[2,3,3,3,4,2,2,3,2,4],[4,3,4,4,3,3,1,3,3,5],[4,3,1,2,2,2,2,3,2,4],[4,2,2,3,2,2,2,3,2,3],[3,2,1,2,1,2,2,3,2,1],[4,2,2,3,3,2,1,4,2,3],[3,2,2,2,2,2,1,4,3,4],[3,2,3,3,2,2,2,1,2,3],[2,3,3,3,3,2,1,3,4,3],[4,2,3,4,3,2,2,2,1,2],[3,4,4,3,5,4,4,4,5,4],[2,4,2,3,2,2,2,3,3,2],[1,2,2,2,3,2,1,3,3,4],[34.9,35.8,34,36.9,35.3,34.7,36,36.5,34.7,35.9],[35.9,36,36.2,37,36.1,36.4,36.3,36.6,36.3,36.3],[2,2,2,2,4,4,2,2,2,2],[35.4,35.9,35.1,36.95,35.7,35.55,36.15,36.55,35.5,36.1],[1,1,1,1,1,1,1,1,1,1],[-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667],[5,5,5,5,5,5,5,5,5,5],[1,1,1,1,1,1,1,1,1,1],[0.186319647739787,0.472682850177291,-1.77049556891649,1.0454092550523,-1.19776916404149,-1.29322356485399,-0.815951560791479,-0.386406757135223,0.0431380465210342,0.759046052614796],[-0.00190690952064737,-0.249805147204807,-1.48929633562561,0.593048860921336,1.48548251658431,0.44430991831084,2.82413300007877,-0.0514865570574791,0.642628508458168,1.7829604018053],["Oxford","Oxford","Oxford","Oxford","Oxford","Oxford","Oxford","Oxford","Oxford","Oxford"],[33,35,28,51,37,37,38,51,35,44]],"container":"<table class=\"display fill-container\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>age<\/th>\n      <th>ALEX1<\/th>\n      <th>ALEX2<\/th>\n      <th>ALEX3<\/th>\n      <th>ALEX4<\/th>\n      <th>ALEX5<\/th>\n      <th>ALEX6<\/th>\n      <th>ALEX7<\/th>\n      <th>ALEX8<\/th>\n      <th>ALEX9<\/th>\n      <th>ALEX10<\/th>\n      <th>ALEX11<\/th>\n      <th>ALEX12<\/th>\n      <th>ALEX13<\/th>\n      <th>ALEX14<\/th>\n      <th>ALEX15<\/th>\n      <th>ALEX16<\/th>\n      <th>anxiety<\/th>\n      <th>artgluctot<\/th>\n      <th>attachhome<\/th>\n      <th>attachphone<\/th>\n      <th>AvgHumidity<\/th>\n      <th>avoidance<\/th>\n      <th>cigs<\/th>\n      <th>DIDF<\/th>\n      <th>eatdrink<\/th>\n      <th>ECR1<\/th>\n      <th>ECR2<\/th>\n      <th>ECR3<\/th>\n      <th>ECR4<\/th>\n      <th>ECR5<\/th>\n      <th>ECR6<\/th>\n      <th>ECR7<\/th>\n      <th>ECR8<\/th>\n      <th>ECR9<\/th>\n      <th>ECR10<\/th>\n      <th>ECR11<\/th>\n      <th>ECR12<\/th>\n      <th>ECR13<\/th>\n      <th>ECR14<\/th>\n      <th>ECR15<\/th>\n      <th>ECR16<\/th>\n      <th>ECR17<\/th>\n      <th>ECR18<\/th>\n      <th>ECR19<\/th>\n      <th>ECR20<\/th>\n      <th>ECR21<\/th>\n      <th>ECR22<\/th>\n      <th>ECR23<\/th>\n      <th>ECR24<\/th>\n      <th>ECR25<\/th>\n      <th>ECR26<\/th>\n      <th>ECR27<\/th>\n      <th>ECR28<\/th>\n      <th>ECR29<\/th>\n      <th>ECR30<\/th>\n      <th>ECR31<\/th>\n      <th>ECR32<\/th>\n      <th>ECR33<\/th>\n      <th>ECR34<\/th>\n      <th>ECR35<\/th>\n      <th>ECR36<\/th>\n      <th>endtime<\/th>\n      <th>EOT<\/th>\n      <th>exercise<\/th>\n      <th>gluctot<\/th>\n      <th>health<\/th>\n      <th>HOME1<\/th>\n      <th>HOME2<\/th>\n      <th>HOME3<\/th>\n      <th>HOME4<\/th>\n      <th>HOME5<\/th>\n      <th>HOME6<\/th>\n      <th>HOME7<\/th>\n      <th>HOME8<\/th>\n      <th>HOME9<\/th>\n      <th>KAMF1<\/th>\n      <th>KAMF2<\/th>\n      <th>KAMF3<\/th>\n      <th>KAMF4<\/th>\n      <th>KAMF5<\/th>\n      <th>KAMF6<\/th>\n      <th>KAMF7<\/th>\n      <th>networksize<\/th>\n      <th>nostalgia<\/th>\n      <th>onlineid<\/th>\n      <th>onlineid1<\/th>\n      <th>onlineid2<\/th>\n      <th>onlineid3<\/th>\n      <th>onlineid4<\/th>\n      <th>onlineid5<\/th>\n      <th>onlineid6<\/th>\n      <th>onlineid7<\/th>\n      <th>onlineid8<\/th>\n      <th>onlineid9<\/th>\n      <th>onlineid10<\/th>\n      <th>onlineide11<\/th>\n      <th>phone1<\/th>\n      <th>phone2<\/th>\n      <th>phone3<\/th>\n      <th>phone4<\/th>\n      <th>phone5<\/th>\n      <th>phone6<\/th>\n      <th>phone7<\/th>\n      <th>phone8<\/th>\n      <th>phone9<\/th>\n      <th>romantic<\/th>\n      <th>scontrol1<\/th>\n      <th>scontrol2<\/th>\n      <th>scontrol3<\/th>\n      <th>scontrol4<\/th>\n      <th>scontrol5<\/th>\n      <th>scontrol6<\/th>\n      <th>scontrol7<\/th>\n      <th>scontrol8<\/th>\n      <th>scontrol9<\/th>\n      <th>scontrol10<\/th>\n      <th>scontrol11<\/th>\n      <th>scontrol12<\/th>\n      <th>scontrol13<\/th>\n      <th>selfcontrol<\/th>\n      <th>sex<\/th>\n      <th>smoke<\/th>\n      <th>SNI1<\/th>\n      <th>SNI2<\/th>\n      <th>SNI3<\/th>\n      <th>SNI4<\/th>\n      <th>SNI5<\/th>\n      <th>SNI6<\/th>\n      <th>SNI7<\/th>\n      <th>SNI8<\/th>\n      <th>SNI9<\/th>\n      <th>SNI10<\/th>\n      <th>SNI11<\/th>\n      <th>SNI12<\/th>\n      <th>SNI13<\/th>\n      <th>SNI14<\/th>\n      <th>SNI15<\/th>\n      <th>SNI16<\/th>\n      <th>SNI17<\/th>\n      <th>SNI18<\/th>\n      <th>SNI19<\/th>\n      <th>SNI20<\/th>\n      <th>SNI21<\/th>\n      <th>SNI22<\/th>\n      <th>SNI23<\/th>\n      <th>SNI24<\/th>\n      <th>SNI25<\/th>\n      <th>SNI26<\/th>\n      <th>SNI27<\/th>\n      <th>SNI28<\/th>\n      <th>SNI29<\/th>\n      <th>SNI30<\/th>\n      <th>SNI31<\/th>\n      <th>SNI32<\/th>\n      <th>SNS1<\/th>\n      <th>SNS2<\/th>\n      <th>SNS3<\/th>\n      <th>SNS4<\/th>\n      <th>SNS5<\/th>\n      <th>SNS6<\/th>\n      <th>SNS7<\/th>\n      <th>socialdiversity<\/th>\n      <th>socialembedded<\/th>\n      <th>STRAQ_1<\/th>\n      <th>STRAQ_2<\/th>\n      <th>STRAQ_3<\/th>\n      <th>STRAQ_4<\/th>\n      <th>STRAQ_6<\/th>\n      <th>STRAQ_7<\/th>\n      <th>STRAQ_8<\/th>\n      <th>STRAQ_9<\/th>\n      <th>STRAQ_10<\/th>\n      <th>STRAQ_11<\/th>\n      <th>STRAQ_12<\/th>\n      <th>STRAQ_19<\/th>\n      <th>STRAQ_20<\/th>\n      <th>STRAQ_21<\/th>\n      <th>STRAQ_22<\/th>\n      <th>STRAQ_23<\/th>\n      <th>STRAQ_24<\/th>\n      <th>STRAQ_25<\/th>\n      <th>STRAQ_26<\/th>\n      <th>STRAQ_27<\/th>\n      <th>STRAQ_28<\/th>\n      <th>STRAQ_29<\/th>\n      <th>STRAQ_30<\/th>\n      <th>STRAQ_31<\/th>\n      <th>STRAQ_32<\/th>\n      <th>STRAQ_33<\/th>\n      <th>STRAQ_5<\/th>\n      <th>STRAQ_13<\/th>\n      <th>STRAQ_14<\/th>\n      <th>STRAQ_15<\/th>\n      <th>STRAQ_16<\/th>\n      <th>STRAQ_17<\/th>\n      <th>STRAQ_18<\/th>\n      <th>STRAQ_34<\/th>\n      <th>STRAQ_35<\/th>\n      <th>STRAQ_36<\/th>\n      <th>STRAQ_37<\/th>\n      <th>STRAQ_38<\/th>\n      <th>STRAQ_39<\/th>\n      <th>STRAQ_40<\/th>\n      <th>STRAQ_41<\/th>\n      <th>STRAQ_42<\/th>\n      <th>STRAQ_43<\/th>\n      <th>STRAQ_44<\/th>\n      <th>STRAQ_45<\/th>\n      <th>STRAQ_46<\/th>\n      <th>STRAQ_47<\/th>\n      <th>STRAQ_48<\/th>\n      <th>STRAQ_49<\/th>\n      <th>STRAQ_50<\/th>\n      <th>STRAQ_51<\/th>\n      <th>STRAQ_52<\/th>\n      <th>STRAQ_53<\/th>\n      <th>STRAQ_54<\/th>\n      <th>STRAQ_55<\/th>\n      <th>STRAQ_56<\/th>\n      <th>STRAQ_57<\/th>\n      <th>stress<\/th>\n      <th>stress1<\/th>\n      <th>stress2<\/th>\n      <th>stress3<\/th>\n      <th>stress4<\/th>\n      <th>stress5<\/th>\n      <th>stress6<\/th>\n      <th>stress7<\/th>\n      <th>stress8<\/th>\n      <th>stress9<\/th>\n      <th>stress10<\/th>\n      <th>stress11<\/th>\n      <th>stress12<\/th>\n      <th>stress13<\/th>\n      <th>stress14<\/th>\n      <th>Temperature_t1<\/th>\n      <th>Temperature_t2<\/th>\n      <th>thermotype<\/th>\n      <th>avgtemp<\/th>\n      <th>filter_.<\/th>\n      <th>mintemp<\/th>\n      <th>language<\/th>\n      <th>langfamily<\/th>\n      <th>Zanxiety<\/th>\n      <th>Zavoidance<\/th>\n      <th>Site<\/th>\n      <th>ALEX_SUM<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"pageLength":3,"columnDefs":[{"className":"dt-right","targets":[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,57,58,59,60,61,62,64,65,66,67,68,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86,87,88,89,90,91,92,93,94,95,96,97,98,99,100,101,102,103,104,105,106,107,108,109,110,111,112,113,114,115,116,117,118,119,120,121,122,123,124,125,126,127,128,129,130,131,132,133,134,135,136,137,138,139,140,141,142,143,144,145,151,152,153,154,155,156,157,158,159,160,161,162,163,164,165,166,167,168,169,170,171,172,173,174,175,176,177,178,179,180,181,182,183,184,185,186,187,188,189,190,191,192,193,194,195,196,197,198,199,200,201,202,203,204,205,206,207,208,209,210,211,212,213,214,215,216,217,218,219,220,221,222,223,224,225,226,227,228,229,230,231,232,233,234,235,236,237,238,239,240,241,242,243,244,245,246,248]},{"orderable":false,"targets":0}],"order":[],"autoWidth":false,"orderClasses":false,"lengthMenu":[3,10,25,50,100]}},"evals":[],"jsHooks":[]}</script>
```

此外，我们还可以使用mutate函数对数据进行重新编码，例如根据出生年龄将其分成不同的年龄段。我们可以使用case_when函数生成一个新变量，该变量根据条件变为不同的值。这种方法可以用于反向编码，例如将原来等于1的值变为5，将原来等于2的值变为4。这些技巧在心理学中非常常见，而case_when函数则是解决这些问题的好方法。


```r
df.clean.mutate_3 <- df.pg.raw %>% 
  dplyr::mutate(decade = case_when(age <= 1969 ~ 60,
                                   age >= 1970 & age <= 1979 ~ 70,
                                   age >= 1980 & age <= 1989 ~ 80,
                                   age >= 1990 & age <= 1999 ~ 90,
                                   TRUE ~ NA_real_)
                ) %>% #当括号多的时候注意括号的位置 
  dplyr::select(.,decade, everything())
```

```{=html}
<div id="htmlwidget-c5a9626179d321cc059a" style="width:100%;height:auto;" class="datatables html-widget"></div>
<script type="application/json" data-for="htmlwidget-c5a9626179d321cc059a">{"x":{"filter":"none","vertical":false,"fillContainer":true,"data":[["1","2","3","4","5","6","7","8","9","10"],[70,90,90,80,90,90,90,70,90,90],[1975,1995,1995,1988,1991,1995,1996,1973,1996,1996],[1,2,4,2,2,2,1,3,2,3],[1,2,1,3,1,3,1,3,3,3],[1,2,1,4,1,1,1,2,1,1],[2,2,2,5,5,3,1,3,3,2],[2,2,4,3,2,2,4,4,2,4],[1,2,1,4,2,1,1,4,2,1],[2,2,4,2,3,1,2,4,2,2],[2,2,1,4,4,2,1,4,2,4],[4,4,1,4,1,1,4,4,2,4],[1,2,2,3,1,2,2,5,2,1],[1,2,1,2,2,1,1,3,1,4],[2,2,1,2,1,2,2,2,2,1],[4,2,2,4,2,3,4,2,2,2],[3,2,1,2,4,4,3,3,4,3],[4,3,1,4,4,5,5,4,3,5],[2,2,1,3,2,4,5,1,2,4],[3.72222222222222,4.05555555555556,1.44444444444444,4.72222222222222,2.11111111111111,2,2.55555555555556,3.05555555555556,3.55555555555556,4.38888888888889],[6,0,0,2,0,0,null,0,0,0],[3.11111111111111,4.88888888888889,1.33333333333333,3.88888888888889,4.66666666666667,3.33333333333333,3.11111111111111,3.55555555555556,3.33333333333333,3.22222222222222],[3.44444444444444,2,2,3,1.77777777777778,2.44444444444444,3.77777777777778,3.33333333333333,1.88888888888889,4.22222222222222],[89,89,89,89,89,89,89,89,89,89],[3.27777777777778,3,1.61111111111111,3.94444444444444,4.94444444444444,3.77777777777778,6.44444444444444,3.22222222222222,4,5.27777777777778],[15,null,null,null,null,null,null,10,null,1],[1.63636363636364,2.18181818181818,2,3.27272727272727,2.18181818181818,1.72727272727273,1.72727272727273,3.54545454545455,2,2.63636363636364],[1,1,1,2,1,1,1,1,1,1],[5,6,1,3,1,1,2,3,4,5],[5,4,1,6,1,1,1,2,4,6],[3,2,1,6,1,1,4,3,4,6],[5,5,1,5,1,2,1,2,4,5],[5,5,1,4,1,2,4,3,4,4],[3,2,1,4,5,1,1,3,1,3],[1,3,1,3,2,1,2,3,4,5],[6,5,1,3,1,2,7,3,4,7],[5,2,1,4,7,6,1,3,4,3],[2,3,1,4,1,2,4,1,4,3],[6,7,1,6,6,2,2,5,2,6],[3,4,1,5,1,2,1,2,4,4],[3,3,1,6,2,4,2,5,4,4],[2,3,7,5,1,1,1,4,1,2],[3,4,1,5,2,2,1,5,4,1],[3,5,3,5,1,1,2,3,4,4],[6,5,1,6,2,3,6,3,4,7],[1,5,1,5,2,2,4,2,4,4],[2,3,2,5,1,5,7,3,4,5],[2,4,2,5,7,5,1,2,4,7],[5,3,2,5,2,4,6,4,4,5],[3,2,2,4,5,4,7,3,4,5],[3,2,2,3,2,3,7,3,4,6],[2,3,1,4,4,4,7,3,4,5],[5,3,1,6,2,3,7,4,4,6],[5,4,1,3,6,3,7,4,4,7],[3,5,1,3,7,1,7,4,4,7],[2,3,1,4,6,3,6,3,4,4],[4,2,2,5,6,6,6,3,4,4],[2,2,2,3,6,5,7,2,4,5],[3,4,1,4,7,4,7,2,4,3],[3,2,2,5,2,3,7,3,4,6],[4,3,3,3,7,4,6,3,4,5],[5,3,1,2,6,5,7,4,4,6],[4,3,1,3,6,3,7,6,4,5],[2,3,2,4,7,3,7,2,4,4],["9:23:38","9:23:26","8:57:08","8:55:14","8:54:35","8:39:12","8:32:08","8:28:57","8:28:03","8:25:57"],[3,2.2,1.2,3,2.6,3.6,3.8,2.4,2.6,3],[2,2,2,2,2,2,2,2,2,2],[0,0,12,0,5,0,null,1,0,2],[4,4,4,3,4,4,4,1,3,3],[4,5,1,5,5,4,5,3,3,4],[3,5,1,4,5,4,2,3,3,5],[4,5,4,5,5,4,4,2,4,3],[3,5,1,5,5,4,5,4,5,3],[3,5,1,4,4,4,2,3,4,4],[2,4,1,2,5,2,1,4,3,2],[4,5,1,2,4,2,5,5,2,2],[3,5,1,4,4,3,1,4,3,3],[2,5,1,4,5,3,3,4,3,3],[4,1,5,5,1,3,2,3,3,4],[4,2,6,4,1,3,2,3,3,5],[5,6,5,3,3,3,2,2,2,1],[4,2,4,1,2,2,2,2,1,3],[4,3,3,2,2,2,2,2,3,2],[1,2,1,2,2,4,2,3,3,3],[2,2,1,2,1,4,2,3,1,2],[17,28,24,19,29,33,53,17,17,19],[4.33333333333333,6.5,6.83333333333333,3.5,1.5,2,5.83333333333333,6.33333333333333,5.66666666666667,5.16666666666667],[2.27272727272727,2.63636363636364,2.18181818181818,2,1.90909090909091,2.45454545454545,2.81818181818182,2.54545454545455,1.81818181818182,3.90909090909091],[2,3,3,1,3,4,4,2,2,5],[2,2,4,1,2,4,4,3,3,4],[2,3,2,2,1,2,2,3,2,5],[4,3,4,3,4,4,2,2,2,4],[2,3,1,2,1,3,3,3,2,3],[2,2,1,2,1,1,2,2,1,3],[2,2,1,2,1,2,2,4,2,4],[2,3,3,1,1,1,2,3,2,3],[3,3,3,2,2,2,4,2,1,3],[2,3,1,4,2,1,2,1,1,4],[2,2,1,2,3,3,4,3,2,5],[4,2,1,2,2,3,3,3,3,4],[4,2,4,4,1,3,4,4,2,4],[2,2,1,3,1,2,5,3,1,2],[5,2,4,2,1,3,4,3,4,4],[2,2,1,3,1,2,1,2,3,5],[2,2,1,2,3,2,3,4,1,4],[4,2,1,3,2,2,5,3,1,5],[4,2,1,4,1,3,5,4,1,5],[4,2,4,4,4,2,4,4,1,5],[2,2,1,1,2,2,2,2,2,2],[2,4,2,4,3,5,2,1,3,3],[2,4,3,2,5,5,4,4,3,4],[2,4,3,1,5,5,2,5,5,5],[1,4,2,1,3,2,2,2,5,1],[1,4,4,2,5,3,1,2,5,2],[4,3,2,1,5,4,3,3,3,1],[2,4,4,2,5,3,2,3,4,2],[2,2,1,3,4,3,2,2,5,4],[5,4,3,4,4,5,4,4,3,3],[4,3,5,3,3,5,5,3,4,2],[3,4,4,2,4,5,3,2,2,5],[5,5,4,2,4,4,1,3,4,3],[4,4,4,2,4,3,2,2,2,2],[2.84615384615385,3.76923076923077,3.15384615384615,2.23076923076923,4.15384615384615,4,2.53846153846154,2.76923076923077,3.69230769230769,2.84615384615385],[1,1,1,2,1,1,2,1,1,2],[1,2,2,2,2,2,2,1,2,1],[2,1,1,1,2,2,2,4,2,2],[1,1,1,1,1,1,1,1,1,1],[null,null,null,null,null,null,null,null,null,null],[4,4,4,4,2,4,4,4,4,4],[4,4,4,4,2,4,4,2,4,4],[5,1,4,5,5,5,5,5,5,5],[null,null,4,null,null,null,null,null,null,null],[1,8,2,1,8,2,3,8,2,6],[null,6,2,null,5,1,2,3,2,3],[6,4,5,5,7,5,8,4,3,8],[3,1,5,1,3,5,8,4,2,8],[1,1,1,1,2,1,1,1,1,1],[null,null,null,null,8,null,null,null,null,null],[1,2,2,1,1,2,2,2,2,2],[null,8,8,null,null,8,8,8,8,8],[3,3,2,3,3,1,1,1,1,1],[2,8,1,8,1,null,null,null,null,null],[6,5,6,4,8,null,null,null,null,null],[1,3,1,1,1,8,6,4,1,3],[1,1,1,1,1,1,1,2,1,1],[null,null,null,null,null,null,null,2,null,null],[1,1,1,2,1,2,2,2,2,1],[" "," "," "," "," ","Rowing Club","University Friends","ODAA","Oxford Latin Speaking Society"," "],[" "," "," "," "," ","Rugby Club","Business Social"," ","Oxford Latin Conversation Society"," "],[" "," "," "," "," "," ","Lecturers/ Advisors"," ","Plato Reading Group"," "],[" "," "," "," "," "," ","Friends from home"," "," "," "],[" "," "," "," "," "," ","Neighbors"," "," "," "],[null,null,null,null,null,7,20,1,2,null],[null,null,null,null,null,7,15,null,4,null],[null,null,null,null,null,null,5,null,1,null],[null,null,null,null,null,null,9,null,null,null],[null,null,null,null,null,null,6,null,null,null],[6,6,7,2,1,1,6,6,7,7],[5,7,7,5,2,2,7,6,7,4],[5,6,7,3,2,3,5,7,6,5],[3,7,7,4,1,2,5,6,6,5],[2,6,7,5,2,2,6,7,5,5],[5,7,6,2,1,2,6,6,3,5],[4,6,6,4,1,2,5,6,3,4],[5,7,8,5,7,6,null,7,6,6],[1,2,3,1,2,4,null,2,2,3],[4,2,4,4,1,2,2,3,3,1],[4,4,5,2,1,4,4,2,5,4],[4,3,4,3,1,4,2,3,4,4],[5,3,4,3,4,4,4,4,2,4],[2,3,4,4,4,4,5,2,2,3],[2,4,4,3,1,3,4,3,1,3],[2,5,4,3,3,4,5,4,2,2],[2,2,4,3,4,3,5,3,2,4],[4,5,5,2,1,2,4,2,4,5],[2,3,2,2,1,4,1,4,4,5],[2,2,1,2,1,2,2,2,2,1],[1,2,2,2,4,4,4,2,2,2],[4,4,4,3,4,2,2,3,2,1],[5,4,2,3,2,4,3,4,4,4],[4,4,2,3,3,1,2,4,5,3],[2,3,4,4,4,5,5,4,4,4],[2,3,5,4,4,3,2,4,2,2],[4,3,4,2,4,3,4,3,3,2],[3,3,5,2,4,4,4,4,3,3],[2,3,1,3,5,4,1,3,2,4],[4,3,4,2,4,4,5,4,3,5],[3,3,5,4,4,3,2,2,3,3],[2,3,1,3,4,3,2,3,4,3],[3,3,1,4,1,2,2,2,3,3],[4,3,5,3,1,2,5,1,4,4],[3,3,3,2,1,3,3,2,3,3],[4,3,1,4,4,4,2,4,1,1],[4,5,4,2,1,3,1,4,4,5],[3,5,4,4,1,2,1,3,5,5],[3,2,2,4,4,2,5,2,2,4],[2,4,2,5,1,2,1,3,2,4],[4,3,1,2,4,2,5,1,1,4],[4,3,2,2,3,3,1,4,3,4],[1,4,3,4,4,2,1,3,2,2],[2,4,2,3,5,4,2,5,1,4],[2,4,2,4,5,4,4,4,2,4],[4,4,4,3,4,4,2,4,2,4],[3,4,2,4,4,4,4,5,5,1],[4,2,2,3,4,4,5,2,2,4],[5,3,3,2,4,2,2,2,2,4],[4,3,2,3,3,3,2,1,2,4],[4,3,5,2,4,4,4,4,4,4],[1,4,2,3,4,2,2,3,2,3],[1,2,2,4,4,2,1,2,1,2],[4,3,2,3,1,2,1,4,4,2],[4,4,4,2,1,4,1,4,3,3],[3,3,4,3,4,2,4,3,3,4],[1,2,5,4,4,2,1,2,1,2],[2,3,5,4,4,2,1,2,1,2],[4,3,5,4,4,4,3,3,3,4],[4,4,2,5,1,3,5,2,4,5],[2,4,5,3,1,2,5,4,2,4],[2,4,4,3,4,2,1,3,2,2],[4,3,4,3,5,2,4,4,2,4],[2,4,2,2,4,3,3,2,2,4],[2,4,1,2,4,2,2,2,2,2],[1,2,5,4,1,1,4,2,2,2],[3.07692307692308,2.46153846153846,2.38461538461538,2.84615384615385,2.53846153846154,2.07692307692308,1.53846153846154,3,2.61538461538462,3.15384615384615],[4,2,3,3,3,2,1,4,5,3],[2,3,3,3,4,2,2,3,2,4],[4,3,4,4,3,3,1,3,3,5],[4,3,1,2,2,2,2,3,2,4],[4,2,2,3,2,2,2,3,2,3],[3,2,1,2,1,2,2,3,2,1],[4,2,2,3,3,2,1,4,2,3],[3,2,2,2,2,2,1,4,3,4],[3,2,3,3,2,2,2,1,2,3],[2,3,3,3,3,2,1,3,4,3],[4,2,3,4,3,2,2,2,1,2],[3,4,4,3,5,4,4,4,5,4],[2,4,2,3,2,2,2,3,3,2],[1,2,2,2,3,2,1,3,3,4],[34.9,35.8,34,36.9,35.3,34.7,36,36.5,34.7,35.9],[35.9,36,36.2,37,36.1,36.4,36.3,36.6,36.3,36.3],[2,2,2,2,4,4,2,2,2,2],[35.4,35.9,35.1,36.95,35.7,35.55,36.15,36.55,35.5,36.1],[1,1,1,1,1,1,1,1,1,1],[-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667],[5,5,5,5,5,5,5,5,5,5],[1,1,1,1,1,1,1,1,1,1],[0.186319647739787,0.472682850177291,-1.77049556891649,1.0454092550523,-1.19776916404149,-1.29322356485399,-0.815951560791479,-0.386406757135223,0.0431380465210342,0.759046052614796],[-0.00190690952064737,-0.249805147204807,-1.48929633562561,0.593048860921336,1.48548251658431,0.44430991831084,2.82413300007877,-0.0514865570574791,0.642628508458168,1.7829604018053],["Oxford","Oxford","Oxford","Oxford","Oxford","Oxford","Oxford","Oxford","Oxford","Oxford"]],"container":"<table class=\"display fill-container\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>decade<\/th>\n      <th>age<\/th>\n      <th>ALEX1<\/th>\n      <th>ALEX2<\/th>\n      <th>ALEX3<\/th>\n      <th>ALEX4<\/th>\n      <th>ALEX5<\/th>\n      <th>ALEX6<\/th>\n      <th>ALEX7<\/th>\n      <th>ALEX8<\/th>\n      <th>ALEX9<\/th>\n      <th>ALEX10<\/th>\n      <th>ALEX11<\/th>\n      <th>ALEX12<\/th>\n      <th>ALEX13<\/th>\n      <th>ALEX14<\/th>\n      <th>ALEX15<\/th>\n      <th>ALEX16<\/th>\n      <th>anxiety<\/th>\n      <th>artgluctot<\/th>\n      <th>attachhome<\/th>\n      <th>attachphone<\/th>\n      <th>AvgHumidity<\/th>\n      <th>avoidance<\/th>\n      <th>cigs<\/th>\n      <th>DIDF<\/th>\n      <th>eatdrink<\/th>\n      <th>ECR1<\/th>\n      <th>ECR2<\/th>\n      <th>ECR3<\/th>\n      <th>ECR4<\/th>\n      <th>ECR5<\/th>\n      <th>ECR6<\/th>\n      <th>ECR7<\/th>\n      <th>ECR8<\/th>\n      <th>ECR9<\/th>\n      <th>ECR10<\/th>\n      <th>ECR11<\/th>\n      <th>ECR12<\/th>\n      <th>ECR13<\/th>\n      <th>ECR14<\/th>\n      <th>ECR15<\/th>\n      <th>ECR16<\/th>\n      <th>ECR17<\/th>\n      <th>ECR18<\/th>\n      <th>ECR19<\/th>\n      <th>ECR20<\/th>\n      <th>ECR21<\/th>\n      <th>ECR22<\/th>\n      <th>ECR23<\/th>\n      <th>ECR24<\/th>\n      <th>ECR25<\/th>\n      <th>ECR26<\/th>\n      <th>ECR27<\/th>\n      <th>ECR28<\/th>\n      <th>ECR29<\/th>\n      <th>ECR30<\/th>\n      <th>ECR31<\/th>\n      <th>ECR32<\/th>\n      <th>ECR33<\/th>\n      <th>ECR34<\/th>\n      <th>ECR35<\/th>\n      <th>ECR36<\/th>\n      <th>endtime<\/th>\n      <th>EOT<\/th>\n      <th>exercise<\/th>\n      <th>gluctot<\/th>\n      <th>health<\/th>\n      <th>HOME1<\/th>\n      <th>HOME2<\/th>\n      <th>HOME3<\/th>\n      <th>HOME4<\/th>\n      <th>HOME5<\/th>\n      <th>HOME6<\/th>\n      <th>HOME7<\/th>\n      <th>HOME8<\/th>\n      <th>HOME9<\/th>\n      <th>KAMF1<\/th>\n      <th>KAMF2<\/th>\n      <th>KAMF3<\/th>\n      <th>KAMF4<\/th>\n      <th>KAMF5<\/th>\n      <th>KAMF6<\/th>\n      <th>KAMF7<\/th>\n      <th>networksize<\/th>\n      <th>nostalgia<\/th>\n      <th>onlineid<\/th>\n      <th>onlineid1<\/th>\n      <th>onlineid2<\/th>\n      <th>onlineid3<\/th>\n      <th>onlineid4<\/th>\n      <th>onlineid5<\/th>\n      <th>onlineid6<\/th>\n      <th>onlineid7<\/th>\n      <th>onlineid8<\/th>\n      <th>onlineid9<\/th>\n      <th>onlineid10<\/th>\n      <th>onlineide11<\/th>\n      <th>phone1<\/th>\n      <th>phone2<\/th>\n      <th>phone3<\/th>\n      <th>phone4<\/th>\n      <th>phone5<\/th>\n      <th>phone6<\/th>\n      <th>phone7<\/th>\n      <th>phone8<\/th>\n      <th>phone9<\/th>\n      <th>romantic<\/th>\n      <th>scontrol1<\/th>\n      <th>scontrol2<\/th>\n      <th>scontrol3<\/th>\n      <th>scontrol4<\/th>\n      <th>scontrol5<\/th>\n      <th>scontrol6<\/th>\n      <th>scontrol7<\/th>\n      <th>scontrol8<\/th>\n      <th>scontrol9<\/th>\n      <th>scontrol10<\/th>\n      <th>scontrol11<\/th>\n      <th>scontrol12<\/th>\n      <th>scontrol13<\/th>\n      <th>selfcontrol<\/th>\n      <th>sex<\/th>\n      <th>smoke<\/th>\n      <th>SNI1<\/th>\n      <th>SNI2<\/th>\n      <th>SNI3<\/th>\n      <th>SNI4<\/th>\n      <th>SNI5<\/th>\n      <th>SNI6<\/th>\n      <th>SNI7<\/th>\n      <th>SNI8<\/th>\n      <th>SNI9<\/th>\n      <th>SNI10<\/th>\n      <th>SNI11<\/th>\n      <th>SNI12<\/th>\n      <th>SNI13<\/th>\n      <th>SNI14<\/th>\n      <th>SNI15<\/th>\n      <th>SNI16<\/th>\n      <th>SNI17<\/th>\n      <th>SNI18<\/th>\n      <th>SNI19<\/th>\n      <th>SNI20<\/th>\n      <th>SNI21<\/th>\n      <th>SNI22<\/th>\n      <th>SNI23<\/th>\n      <th>SNI24<\/th>\n      <th>SNI25<\/th>\n      <th>SNI26<\/th>\n      <th>SNI27<\/th>\n      <th>SNI28<\/th>\n      <th>SNI29<\/th>\n      <th>SNI30<\/th>\n      <th>SNI31<\/th>\n      <th>SNI32<\/th>\n      <th>SNS1<\/th>\n      <th>SNS2<\/th>\n      <th>SNS3<\/th>\n      <th>SNS4<\/th>\n      <th>SNS5<\/th>\n      <th>SNS6<\/th>\n      <th>SNS7<\/th>\n      <th>socialdiversity<\/th>\n      <th>socialembedded<\/th>\n      <th>STRAQ_1<\/th>\n      <th>STRAQ_2<\/th>\n      <th>STRAQ_3<\/th>\n      <th>STRAQ_4<\/th>\n      <th>STRAQ_6<\/th>\n      <th>STRAQ_7<\/th>\n      <th>STRAQ_8<\/th>\n      <th>STRAQ_9<\/th>\n      <th>STRAQ_10<\/th>\n      <th>STRAQ_11<\/th>\n      <th>STRAQ_12<\/th>\n      <th>STRAQ_19<\/th>\n      <th>STRAQ_20<\/th>\n      <th>STRAQ_21<\/th>\n      <th>STRAQ_22<\/th>\n      <th>STRAQ_23<\/th>\n      <th>STRAQ_24<\/th>\n      <th>STRAQ_25<\/th>\n      <th>STRAQ_26<\/th>\n      <th>STRAQ_27<\/th>\n      <th>STRAQ_28<\/th>\n      <th>STRAQ_29<\/th>\n      <th>STRAQ_30<\/th>\n      <th>STRAQ_31<\/th>\n      <th>STRAQ_32<\/th>\n      <th>STRAQ_33<\/th>\n      <th>STRAQ_5<\/th>\n      <th>STRAQ_13<\/th>\n      <th>STRAQ_14<\/th>\n      <th>STRAQ_15<\/th>\n      <th>STRAQ_16<\/th>\n      <th>STRAQ_17<\/th>\n      <th>STRAQ_18<\/th>\n      <th>STRAQ_34<\/th>\n      <th>STRAQ_35<\/th>\n      <th>STRAQ_36<\/th>\n      <th>STRAQ_37<\/th>\n      <th>STRAQ_38<\/th>\n      <th>STRAQ_39<\/th>\n      <th>STRAQ_40<\/th>\n      <th>STRAQ_41<\/th>\n      <th>STRAQ_42<\/th>\n      <th>STRAQ_43<\/th>\n      <th>STRAQ_44<\/th>\n      <th>STRAQ_45<\/th>\n      <th>STRAQ_46<\/th>\n      <th>STRAQ_47<\/th>\n      <th>STRAQ_48<\/th>\n      <th>STRAQ_49<\/th>\n      <th>STRAQ_50<\/th>\n      <th>STRAQ_51<\/th>\n      <th>STRAQ_52<\/th>\n      <th>STRAQ_53<\/th>\n      <th>STRAQ_54<\/th>\n      <th>STRAQ_55<\/th>\n      <th>STRAQ_56<\/th>\n      <th>STRAQ_57<\/th>\n      <th>stress<\/th>\n      <th>stress1<\/th>\n      <th>stress2<\/th>\n      <th>stress3<\/th>\n      <th>stress4<\/th>\n      <th>stress5<\/th>\n      <th>stress6<\/th>\n      <th>stress7<\/th>\n      <th>stress8<\/th>\n      <th>stress9<\/th>\n      <th>stress10<\/th>\n      <th>stress11<\/th>\n      <th>stress12<\/th>\n      <th>stress13<\/th>\n      <th>stress14<\/th>\n      <th>Temperature_t1<\/th>\n      <th>Temperature_t2<\/th>\n      <th>thermotype<\/th>\n      <th>avgtemp<\/th>\n      <th>filter_.<\/th>\n      <th>mintemp<\/th>\n      <th>language<\/th>\n      <th>langfamily<\/th>\n      <th>Zanxiety<\/th>\n      <th>Zavoidance<\/th>\n      <th>Site<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"pageLength":3,"columnDefs":[{"className":"dt-right","targets":[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,57,58,59,60,61,62,63,65,66,67,68,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86,87,88,89,90,91,92,93,94,95,96,97,98,99,100,101,102,103,104,105,106,107,108,109,110,111,112,113,114,115,116,117,118,119,120,121,122,123,124,125,126,127,128,129,130,131,132,133,134,135,136,137,138,139,140,141,142,143,144,145,146,152,153,154,155,156,157,158,159,160,161,162,163,164,165,166,167,168,169,170,171,172,173,174,175,176,177,178,179,180,181,182,183,184,185,186,187,188,189,190,191,192,193,194,195,196,197,198,199,200,201,202,203,204,205,206,207,208,209,210,211,212,213,214,215,216,217,218,219,220,221,222,223,224,225,226,227,228,229,230,231,232,233,234,235,236,237,238,239,240,241,242,243,244,245,246,247]},{"orderable":false,"targets":0}],"order":[],"autoWidth":false,"orderClasses":false,"lengthMenu":[3,10,25,50,100]}},"evals":[],"jsHooks":[]}</script>
```

我们可以先按照出生年代将数据进行分组，然后使用group by函数重新分组。我们还可以按照多个条件进行分组，例如按照年代和性别进行分组。分组完成后，我们可以使用summarize函数对每个组内部进行操作，例如求均值和标准差等。最后，我们可以使用ungroup函数将数据重新拆分，以便进行后续的运算。


```r
df.clean.group_by <- df.clean.mutate_3 %>%
  dplyr::group_by(.,decade) %>% # 根据被试的出生年代，将数据拆分
  dplyr::summarise(mean_avoidance = mean(avoidance)) %>% # 计算不同年代下被试的平均avoidance
  dplyr::ungroup()
```

```{=html}
<div id="htmlwidget-ab46d30ffc39e3f95d06" style="width:100%;height:auto;" class="datatables html-widget"></div>
<script type="application/json" data-for="htmlwidget-ab46d30ffc39e3f95d06">{"x":{"filter":"none","vertical":false,"fillContainer":true,"data":[["1","2","3","4"],[60,70,80,90],[3.13559322033898,2.89696992243867,2.91210368205253,3.15488581563949]],"container":"<table class=\"display fill-container\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>decade<\/th>\n      <th>mean_avoidance<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"pageLength":4,"columnDefs":[{"className":"dt-right","targets":[1,2]},{"orderable":false,"targets":0}],"order":[],"autoWidth":false,"orderClasses":false,"lengthMenu":[4,10,25,50,100]}},"evals":[],"jsHooks":[]}</script>
```

我们可以将所有学到的函数串起来，例如1.先使用filter函数选择eat drink为1的base，然后2.使用select函数选择所需变量，3.再使用mutate函数对出生年份进行编码，4.最后使用groupby和summarize函数求出按照年代求alex的均值。5.最后，我们可以使用row sum函数对alex求均值。


```r
df.pg.clean <- df.pg.raw %>%
  dplyr::filter(eatdrink == 1) %>% # 选择eatdrink为1的被试
  dplyr::select(age, starts_with("ALEX"), eatdrink, avoidance) %>%
  dplyr::mutate(ALEX1 = case_when(ALEX1 == '1' ~ '5', # 反向计分
                                  ALEX1 == '2' ~ '4',
                                  ALEX1 == '3' ~ '3',
                                  ALEX1 == '4' ~ '2',
                                  ALEX1 == '5' ~ '1',
                                  TRUE ~ as.character(ALEX1))) %>%
  dplyr::mutate(ALEX1 = as.numeric(ALEX1)) %>%
  dplyr::mutate(ALEX_SUM = rowSums(select(., starts_with("ALEX"))), # 把所有ALEX的题目分数求和
                decade = case_when(age <= 1969 ~ 60, # 把出生年份转换为年代
                                   age >= 1970 & age <= 1979 ~ 70,
                                   age >= 1980 & age <= 1989 ~ 80,
                                   age >= 1990 & age <= 1999 ~ 90,
                                   TRUE ~ NA_real_)) %>%
  dplyr::group_by(decade) %>% # 按照年代将数据拆分
  dplyr::summarise(mean_ALEX = mean(ALEX_SUM)) %>% # 计算每个年代的被试的平均的ALEX_SUM
  dplyr::ungroup() # 解除对数据的拆分
```

在这里，我们可以按照多个条件进行分组，但我们只使用了一个条件，即数十年，因此可以写得很简洁。我们使用summarize函数计算alex总分的平均值，然后取消分组。如果我们对alex这个变量感兴趣，我们可以得到每个数十年在alex得分上的平均值。

在这个过程中，我们需要理解管道的操作方式，因为它可以处理我们想要对某个变量的所有领域进行的所有操作。每个函数都可以单独使用，但将它们放入管道中可以省略一些步骤。Tidyverse中有许多包，例如startwith、case1和rowsum，它们都是Dplyr和Tidyverse系统中的函数。另一个常用的包是tidyr，它可以使我们的数据更加整洁。在Tidyverse中，我们可以使用一些常用的函数，比如separate、extract、unite和pivot。其中，separate可以按照特定规则将一个变量分割成为几列，比如将日期按照年月日分成三列。extract可以提取一个或多个特定的字符串。unite则是将多个列合并成为一列。pivot则可以将宽格式数据转换为长格式数据或者将长格式数据转换为宽格式数据。

**长数据和宽数据**

那么，什么是长格式数据和宽格式数据呢？在SPSS中，我们通常使用长格式数据来记录实验数据，比如eprime数据。长格式数据的形式可能是这样的：每个实验对象有一个ID，然后有不同的条件和变量，比如年龄和性别。每个实验对象可能有多个条件，因此我们需要以行来记录数据。每一行代表一个观察值。例如，如果我们有一个问卷调查，其中有五个问题，那么我们将每个问题的回答记录在一列中，每一行代表一个受访者。相反，宽格式数据是指我们将变量记录在多列中，每一行代表一个观察值。例如，如果我们有一个实验，其中有五个条件，那么我们将每个条件的结果记录在一列中，每一行代表一个受试者。

> 长数据和宽数据是数据分析和处理中常用的两种不同的数据结构。
>
> 长数据通常表示为一列数据包含多个变量，每个变量在不同的行中重复出现。例如，一列包含所有参与者的年龄、性别和教育水平，每个参与者在数据集中有多行记录。长数据集通常适用于需要进行聚合和统计分析的情况。
>
> 宽数据则通常表示为多列数据包含多个变量，每个变量在同一行中出现。例如，一列包含参与者的年龄，另一列包含性别，第三列包含教育水平。每个参与者只有一行记录。宽数据集通常适用于需要进行分组和比较分析的情况。
>
> 两种数据结构都有其优缺点，具体使用哪种结构取决于数据的分析目的和方法。
>
> 当一个数据集中的每一行都是单个观察单位时，通常使用宽数据格式。下面是一些宽数据格式的例子：
>
> - 一份包含人口统计数据的电子表格，每一行表示一个城市或地区，每一列则包含不同的人口统计数据，如总人口数、平均年龄、平均收入等等。
> - 一个电商网站的订单数据集，每一行表示一个订单，每一列包含订单的属性，如订单号、购买日期、购买者姓名、商品名称、数量、价格等等。
> - 一个医疗研究的数据集，每一行表示一个受试者，每一列包含该受试者的不同测量结果，如身高、体重、血压、血糖等等。
>
> 当数据集中的每一行表示的是一个观察单位的不同取值时，通常使用长数据格式。下面是一些长数据格式的例子：
>
> - 一份学生考试成绩的数据集，每一行表示一个学生的一门考试成绩，每一列包含学生的属性（如学生ID、姓名、年级、学科等）和考试成绩。
> - 一个心理学实验的数据集，每一行表示一个受试者在实验中的一次操作，每一列包含操作的属性（如受试者ID、实验条件、操作类型等）和操作结果。
> - 一份股票市场的数据集，每一行表示某支股票在某个时间点的市场数据，每一列包含市场数据的属性（如股票代码、日期、开盘价、收盘价、成交量等）。

最后，我们需要注意数据中的缺失值。有时候，我们需要将缺失值删除，以便我们可以更好地分析数据。我们可以使用dropNA函数来删除缺失值，但是我们需要谨慎使用它，因为它可能会导致一些问题。

## 函数

### 函数参数

首先，我们看到的是一个大家已经很熟悉的函数，即read.csv。我们将这个数据读取进来，成为R里面能够操纵的一个变量。read.csv实际上是一个函数名，括号里面的就是我们要输入的参数argument。第一个argument是一个文件的路径，第二个是header，表示是否使用第一行作为我们的column names，第三个是sep，表示我们指定的分格符是什么。我们还有第四个，即force，表示是否把我们读到的文件里面的字符串作为factor。每一个函数基本上都包含两个部分，即**函数名和argument**。

```{-}
function1(argument1 = 123,argument2 = "helloworld",argument3 = list1) 
#第一种输入方法：同时输入argument和value

function2(123,"helloworld",list1) 
function2("helloworld",list1) 
#第一种输入方法：也可以省略argument,顺序读取value
function2("helloworld",123,list1) 
```

我们有几种输入argument的方法，第一种是完整的输入，即function name和argument都要输入。第二个方法就是 大家可以看到我们其实有的时候是直接写这个function name，就是value1, value2, value3对吧 我们没有加argument等于什么 就相当于我们没有把这个argument写出来 那么这个时候的话 我们还是把这个顺序调换一下 我们写上面一样 就直接是value3, value1, value2 最后大家想想这两个结果会反复是一样的结果吗 不一样对吧 为什么不一样呢 所以有同学已经知道这个了对吧 实际上是我们一种方式 我们完整的输出的时候 就是会把每一个value 比方说value1就复制到argument1 那么第二种方式的话 我们是按照顺序来输入的 那么按照顺序的时候 我们写函数的时候 每一个argument就是一二三这种排列下来的，一个是result1，它有三个参数，分别对应value1、value2和value3。第二个代码是把参数的顺序调换了一下，但是结果并不会改变。这是因为我们只是改变了参数的顺序，但是它们的值并没有改变。另外，如果我们只给出了值，而没有明确指定参数，那么函数会按照顺序把值赋给参数。如果我们只给出了一个值，那么函数会默认把它赋给第一个参数，而其他参数则会有默认值。我们可以在R中使用read.csv()函数。首先，我们可以查看帮助文档，使用问号加函数名的方式。对于read.csv()函数，除了第一个argument（文件名）以外，其余argument都有默认值。例如，header默认为false，separate默认为空格，quote为一个特殊符号表示引号。另外，string as fact也是一个默认值。因此，我们只需要输入文件名这一个argument即可。如果不输入其他argument，它会使用默认值。这是R的一个重要特点，它兼具灵活性和便捷性。如果我们要完整写出来，我们需要写出文件名这个argument的名字，即file。我们可以输入相对路径或绝对路径。其他argument的默认值可以在电脑上查看。因此，我们可以更简洁地写出这个函数，只需要输入三个argument即可。函数有两个部分，一个是函数名，一个是输入的argument。我们需要按照函数定义者的要求来使用这些argument。的函数都进行了修改，这就会导致我们之前写的代码出现问题。因此，在使用别人的函数时，我们需要时刻关注它的更新和变化，以便及时调整我们的代码。另外，我们也可以通过自己编写函数来满足特定的需求，这样可以更好地掌控代码的逻辑和功能。

### groupby()

groupby有两个作用：第一个是分组，然后将其拆分为不同的小包，然后在对分组后的数据进行分析时，它将以组为单位进行操作，将每个组的结果分别计算，然后组合起来。如果我们在没有任何操作的情况下添加groupby，它会如何改变我们的数据框呢？


```r
# 读取原始数据
df.mt.raw <-  read.csv('./data/match/match_raw.csv',
                       header = T, sep=",", stringsAsFactors = FALSE) 
library("tidyverse")
group <- df.mt.raw %>% 
  group_by(Shape)
group#注意看数据框的第二行，有Groups:   Shape [4]的信息
```

```
## # A tibble: 25,920 x 16
## # Groups:   Shape [4]
##    Date        Prac    Sub   Age Sex   Hand  Block   Bin Trial Shape Label Match
##    <chr>       <chr> <int> <int> <chr> <chr> <int> <int> <int> <chr> <chr> <chr>
##  1 02-May-201~ Exp    7302    22 fema~ R         1     1     1 immo~ immo~ mism~
##  2 02-May-201~ Exp    7302    22 fema~ R         1     1     2 mora~ mora~ mism~
##  3 02-May-201~ Exp    7302    22 fema~ R         1     1     3 immo~ immo~ mism~
##  4 02-May-201~ Exp    7302    22 fema~ R         1     1     4 mora~ mora~ mism~
##  5 02-May-201~ Exp    7302    22 fema~ R         1     1     5 immo~ immo~ match
##  6 02-May-201~ Exp    7302    22 fema~ R         1     1     6 immo~ immo~ match
##  7 02-May-201~ Exp    7302    22 fema~ R         1     1     7 mora~ mora~ match
##  8 02-May-201~ Exp    7302    22 fema~ R         1     1     8 mora~ mora~ match
##  9 02-May-201~ Exp    7302    22 fema~ R         1     1     9 mora~ mora~ mism~
## 10 02-May-201~ Exp    7302    22 fema~ R         1     1    10 immo~ immo~ mism~
## # ... with 25,910 more rows, and 4 more variables: CorrResp <chr>, Resp <chr>,
## #   ACC <int>, RT <dbl>
```

举个例子，如果我们将df.mt.raw复制到名为group的变量类中，那么如果我们直接将df.mat.raw放入其中，我们可能会看到前面的行数和列数。但是，如果我们添加了groupby，它将具有名为groups的东西，然后是形状和上面的表单。这意味着我们的表单仍然是原来的数据框，但现在它有一个标记，表示它已经在内部进行了分组，分组标准是group，即形状。如果您想删除此形状变量，就必须ungroup。否则，这个分组标签将一直存在于数据框中。因此，我们建议在进行groupby之后一定要进行ungroup，否则分组标签将一直存在于数据框中。

实际上，我们不会像现在这样无聊地直接添加groupby，然后看它会发生什么。我们需要明确后面分析的逻辑是什么。我们可以通过groupby将数据框按照base的ID分成几个小的数据框，然后以每个亚组为单位进行计算。比如，我们可以用summarize求出每个subgroup的行数，然后返回到n里面去，得到一个描述性的结果。但是，我们需要注意，当我们得到新的结果后，一定要把它ungroup掉，否则会影响后面的分析。此外，我们需要注意，如果我们再进行一个groupby，而没有ungroup前面的结果，可能会覆盖掉前面的结果，这是需要注意的问题。

如果我们不使用groupby，我们以前的做法是先申请一个中间变量，然后将数据按条件分组，选出一个subset出来，然后对每个操作分别求均值和标准差，最后将它们合并起来。现在有了groupby和summarize，我们可以在管道中一次性完成所有操作，而不需要生成大量的中间变量。这样做的好处是逻辑更清晰，代码更简洁，而且不会占用过多的内存。这就是关于groupby的说明。我们刚才展示的是使用和不使用group by的区别。使用group by可以得到每个条件下的函数值，而不使用则是整个数据的函数值。

对于group_by()函数的作用，我们可以对比不使用它的效果。


```r
library("tidyverse")
df.mt.raw <-  read.csv('./data/match/match_raw.csv',
                       header = T, sep=",", stringsAsFactors = FALSE) 
group <- df.mt.raw %>% 
  group_by(.,Shape) %>% 
  summarise(n())
DT::datatable(group)
```

```{=html}
<div id="htmlwidget-4baf6fd0766bc7eb85c8" style="width:100%;height:auto;" class="datatables html-widget"></div>
<script type="application/json" data-for="htmlwidget-4baf6fd0766bc7eb85c8">{"x":{"filter":"none","vertical":false,"data":[["1","2","3","4"],["immoralOther","immoralSelf","moralOther","moralSelf"],[6480,6480,6480,6480]],"container":"<table class=\"display\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>Shape<\/th>\n      <th>n()<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"columnDefs":[{"className":"dt-right","targets":2},{"orderable":false,"targets":0}],"order":[],"autoWidth":false,"orderClasses":false}},"evals":[],"jsHooks":[]}</script>
```

```r
ungroup <- df.mt.raw %>% 
  summarise(n())
DT::datatable(ungroup)
```

```{=html}
<div id="htmlwidget-cdaaa9034f93a10233e9" style="width:100%;height:auto;" class="datatables html-widget"></div>
<script type="application/json" data-for="htmlwidget-cdaaa9034f93a10233e9">{"x":{"filter":"none","vertical":false,"data":[["1"],[25920]],"container":"<table class=\"display\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>n()<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"columnDefs":[{"className":"dt-right","targets":1},{"orderable":false,"targets":0}],"order":[],"autoWidth":false,"orderClasses":false}},"evals":[],"jsHooks":[]}</script>
```

### 定义函数

因为我们已经讲了函数的逻辑，即函数名、参数和操作。我们可以最简单的方式就是建立一个函数。我们用function来定义函数，它本身也是一个函数，用于帮助我们定义一个函数。我们可以在括号()中输入任何我们想定义的参数。然后在大括号{}中定义函数的操作，例如求和和乘积。


```r
fun1 <- function(a = 1,b = 100){
    sum <- 0
	for (i in a:b) {
  		sum <- sum + i
	}
    return(sum)
}
```

在这个函数中，a和b都有默认值，a的默认值是1，b的默认值是100。如果我们不给它们赋值，它们就会按照默认值运行。我们可以给a和b赋值，然后运行函数，得到sum的值。这种方式非常有用，因为有时候现有的函数可能不够满足我们的需求，我们可以自己定义一个简单的函数来处理数据。我们可以定义一个函数来进行实验操作和数据分析，这样可以方便后续的优化和重复使用。

在R中，括号有不同的用法，中括号一般用于调用变量中的数据，根据变量的不同特点，可能需要输入不同的index值。索引某个变量中的元素通常使用中括号，小括号通常用作函数的参数或输入。在函数中，小括号表示输入参数的值；大括号一般表示为一个代码块。if else语句中，小括号表示输入判断条件。在for循环中，我们也需要使用小括号和大括号来输入循环条件和操作。


```r
fun1 <- function(a = 1,b = 100){#第一个{构成最外层代码块的起始
    sum <- 0
	for (i in a:b) {#第二个{构成次外层代码块的起始
  		sum <- sum + i
	}#与第二个{对应的}构成次外层代码块的结尾
    return(sum)
}#与第一个{对应的}构成最外层代码块的结尾
```

### for loop

**for (variable in sequence) {statement}**

for loop是一种循环语句，它的语法结构是for (variable in sequence) {statement}，其中variable是一个代符，sequence是一个向量或列表，而statement则是要重复执行的某一个语句。在每一次循环中，variable都被赋予为sequence里面的一个元素，然后执行一次statement，再回到sequence里面的下一个元素，不断地循环。每一次循环，variable都会变成sequence里面下一个元素，因此它只是一直在变化。


```r
sum <- 0
#variable i  sequence 1:100
for (i in 1:100) {#statement
  sum <- sum + i
}
print(sum)
```

```
## [1] 5050
```

举个例子，如果我们要对1到100的数据进行求和，我们可以定义一个sum变量，然后用for loop来实现。在这个例子中，i是variable，1:100是sequence，1到100的所有整数是1:100里面的元素。每一次循环，sum都会加上i的值，最终得到1到100的和。在第一次循环时，sum的初始值为0，i的值为1，所以sum会更新为1。在第二次循环时，i的值为2，sum的值为1，所以sum会更新为3。以此类推，直到循环结束，sum的值就是1到100的和。


```r
files <- list.files(file.path("data/match"), 
                    pattern = "data_exp7_rep_match_.*\\.out$")

df_list <- list()

for (i in seq_along(files)) {
  df <- read.table(file.path("data/match", files[i]), header = TRUE) 
  df_list[[i]] <- df
}
```

在批量读取文件的for循环中，我们读取了所有以data experiment 7开头，以out结尾的文件，使用list.files函数匹配符合模式的所有文件，得到一列文件名list，然后循环list。我们需要对1到num files进行操作，这个sequence是从1开始的。在R和MatLab中，我们也是从1开始的，但在Python中，我们是从0开始的。

在for循环中，我们使用i in seq_along(files)来循环操作，从i等于1开始，然后一直到最后一个元素。当i等于1时，我们使用read table函数，并从files中提取出第i个元素，这个元素实际上是一个字符，也就是代表某一个特定的文件。

file path函数将前面的文件夹和后面的文件名整合成一个完整的文件路径。因此，read table函数的第一个argument就是这个file path。当进行了读取操作之后。这个df实际上就是第一个out文件的数据。

在datalist中，i代表第几个元素，使用df.list[i]来index list。在第一次循环中，我们将df给到df.list的第一个元素。

因此，当i等于1时，我们开始第一次循环，首先有一个files代表需要操作的文件。多比方说什么什么.out，对吧？第一个元素是什么，第二个元素也是什么.out，一直到最后一个元素。当我们i=1的时候，我们把它的第一个元素拿出来，然后把它拿到redtable里面去。这个filepath本身又是一个函数，我们把这个第一个文件夹的名字和第一个out文件路径加起来，就是file对于filepath这个地方。然后再开启redtable的第二个argument，那就是header对于true。通过这个for loop，我们读取了所有的文件，把读取的结果复制到了df.list里面的对应元素。这个时候，df-list就更新了。以此类推直到所有的files里面的所有数据都读取完，那么这个时候我们的df-list就是一个包含了所有数据的一个list，每一个元素代表一个base的一个数据。

### 信号检测

接下来，我们回顾上一节课的练习题。我们使用数据预处理的方法，求出match数据中的deep prime，这是一个信号检测论的指标，也称为敏感性指标。

我们可以将数据分为signal和noise，其中signal是指hit，对应的是correct rejection；而如果刺激中有noise，我们认为它是有信号的，这就是FA。如果我们报告时认为刺激中有信号，但实际上是noise，这就是Miss。根据信号检测论，我们将刺激分为match和mismatch，其中match是信号，而mismatch或nonmatch是noise。我们的反应是指我们呈现的刺激。如果反应是match，那么这就是hit。对于match的trial，如果我们的反应是mismatch，那么这就是信号减速论的miss。正确率是0，因为它是错误的。对于mismatch的trial，如果我们的反应是mismatch，那么这就是CR，正确率是1。FA的正确率是0。我们需要计算d'，它等于zheat减去zFA，这是我们敏感性的指标。我们需要用R来计算每个条件下的d'，因此我们需要使用group by。最终，我们将得到一个比例，其中包括base的id、条件和d'。

我们在实际使用这些函数时，是带着目的去使用的。想要计算击中率虚报率，我们首先要知道什么情况是击中，也要让计算机知道什么情况是击中，一旦分类了，计算的时候也需要进行判断。但在分类判断之前，我们要先告诉计算机什么是正确情况什么是错误情况，哪些是这个被试做的，哪些不是，所以也需要分组。

我们有了基本的思路：1.分组，使用group_by。2.告诉计算机什么是hit，什么是miss，使用summarise函数。3.利用判断语句进行分类计算，我们这里使用了ifelse。

我们选出需要的几列，包括自变量和因变量，以及分组变量。使用select()函数，因为数据中存在缺失值，所以也需要除去它们。


```r
df.mt.clean <- df.mt.raw %>%
  dplyr::select(Sub, Block, Bin,  # block and bin
                Shape, Match, # 自变量
                ACC, RT, # 反应结果
                ) %>% 
  tidyr::drop_na()
```

在计算击中率误报率等的时候，我们针对的是被试的在某一特定实验条件下的反应。所以我们需要通过bin、block、shape、sub进行分组，这样进行计算时，就是每一个条件组下分别计算。不会出现在虚报的实验条件下计算正确拒绝的情况。


```r
df.mt.clean <- df.mt.raw %>%
  dplyr::select(Sub, Block, Bin,  # block and bin
                Shape, Match, # 自变量
                ACC, RT, # 反应结果
                ) %>% 
  tidyr::drop_na()  %>% #删除缺失值
  dplyr::group_by(Sub, Block, Bin, Shape)
```

接下来就是要使用summarise函数来给击中、虚报等分类，并使用ifelse函数根据分类计算概率


```r
df.mt.clean <- df.mt.raw %>%
  dplyr::select(Sub, Block, Bin,  # block and bin
                Shape, Match, # 自变量
                ACC, RT, # 反应结果
                ) %>% 
  tidyr::drop_na()  %>% #删除缺失值
  dplyr::group_by(Sub, Block, Bin, Shape) %>%
  dplyr::summarise(
      hit = length(ACC[Match == "match" & ACC == 1]),
      fa = length(ACC[Match == "mismatch" & ACC == 0]),
      miss = length(ACC[Match == "match" & ACC == 0]),
      cr = length(ACC[Match == "mismatch" & ACC == 1]),
      Dprime = qnorm(
        ifelse(hit / (hit + miss) < 1,
               hit / (hit + miss),
               1 - 1 / (2 * (hit + miss))
              )
           ) - qnorm(
        ifelse(fa / (fa + cr) > 0,
               fa / (fa + cr),
               1 / (2 * (fa + cr))
              )
                    ))
```

```
## `summarise()` has grouped output by 'Sub', 'Block', 'Bin'. You can override
## using the `.groups` argument.
```

在这里，我们使用了一个num来判断它们的数量。我们可以使用nrow代替。这只是其中一种做法。接下来，我们看一下acc里面所有等于match的情况。acc等于什么呢？条件是match，然后acc等于1。当我们想要计算hit时，我们需要将两个条件进行组合。一个是刺激呈现的内容，另一个是正确率。只有在match条件下反应正确的试次才是hit的试次。通过这个length，我们计算出了在每一个base、每一个block、每个bin下面以及每一个条件shape之下有多少个hit的比值。

同样的逻辑，我们通过mismatch和0计算出了FA（false alarm）。如果match的accuracy是0，我们就错失了这个信息，也就是miss。这是我们前面知识的一个简单应用。


```r
df.mt.clean <- df.mt.raw %>%
  dplyr::select(Sub, Block, Bin,  # block and bin
                Shape, Match, # 自变量
                ACC, RT, # 反应结果
                ) %>% 
  tidyr::drop_na()  %>% #删除缺失值
  dplyr::group_by(Sub, Block, Bin, Shape) %>%
  dplyr::summarise(
      hit = length(ACC[Match == "match" & ACC == 1]),
      fa = length(ACC[Match == "mismatch" & ACC == 0]),
      miss = length(ACC[Match == "match" & ACC == 0]),
      cr = length(ACC[Match == "mismatch" & ACC == 1]),
      Dprime = qnorm(
        ifelse(hit / (hit + miss) < 1,
               hit / (hit + miss),
               1 - 1 / (2 * (hit + miss))
              )
           ) - qnorm(
        ifelse(fa / (fa + cr) > 0,
               fa / (fa + cr),
               1 / (2 * (fa + cr))
              )
                    )) %>% 
  dplyr::ungroup() %>%
  select(-"hit",-"fa",-"miss",-"cr") %>%
  dplyr::group_by(Sub, Shape)  %>%
  tidyr::pivot_wider(names_from = Shape,
                     values_from = Dprime) 
```

```
## `summarise()` has grouped output by 'Sub', 'Block', 'Bin'. You can override
## using the `.groups` argument.
```

然后，我们计算出了每一个d'。我们可以看到，这个d'是用qnorm计算的。hit的比值是等于hit除以qt加miss。这个地方有点复杂，它实际上就是想要把我们这两个东西打包在一个语句里面。zheat和zfa有两个条件，首先我们看它的两个部分，一个是qnorm，表示heat的部分，另一个是fa部分。我们计算出heat rate，然后将其转换成z分数，减去quantum。这个地方为什么要用if else呢？因为会出现一个情况，比方说我的heat全部是正确的，这个时候正确率为1。正确率为1的话，我们用这个qnorm的话，它是一个无限大的一个，就是一个正向的infinity。对这种情况，我们要进行一个转换，就是我们要把这个heat rate转换成一个小于1的值，这样的话就避免它成为了一个infinity。如果它变成infinity的话，我们后面就没法进行计算了。所以，如果heat rate小于1，我们就用它。

这个是if else在这个type里面。如果前面heat rate...Heat rate相比较而言存在差异，如果它小于1，那么我们就使用它自己的值。如果它不小于1，也就是等于1，那么我们就用1减去1除以括号里的值，这样可以使得反应变得稍微小一点。这是一个常用的方法，在信号减速论里面也常用。对于FA也是同样的方法，我们使用if else，如果它大于0，那么我们就用它，如果等于0，那么我们就需要想一个办法让它变成一个不是0的值，因为如果它是0的话，那么它会变成负向无穷大的值，这样就无法进行后续的运算。接下来我们就可以算出d'，然后去掉后面所有的heat、fn、least和cr，进行后续的运算。这个逻辑清晰吗？

我们可以拆开每一个函数来看，选择columns group by，summarize group by，qvolume是一个把百分比转换成为1分数的函数，这个大家需要去后面查找。然后我们进行相减，插入条件语句，再把它转换成1分数，最后就得到d'。我们也可以进行后续的运算，再次group by，subject和shape。我们可以查看每一个被试在每一个bin上面的结果的变化。如果我们收了很多被试，可能就不需要关注每一个被试的变化，只需要报告总体的结果即可。我们可以根据time inversion里面的数据预数的一系列操作，一个一个管道全部下来，最后直接得到我们想要的结果。这样的管道操作可能需要不断迭代，刚开始可能只会做最简单的预处理，但是迭代到最后，就会形成一个很长的管道。

数据处理的流程非常清晰。在这个过程中，我们对acc进行了选择，相当于是选择了符合条件的一些行，并求出它的长度。虽然理论上讲，我们也可以用别的变量来代替acc，比如rt，但是acc是最方便的选择，因为我们后面也会用到它。但是，这并不是唯一的操作，我们还可以用其他的变量来进行操作，比如length换成n。如果我们用filter的话，也是可行的，但可能需要写更多的代码。我们要根据自己的研究目的或者想要操作的目的去进行数据处理，把各种预处理操作进行组合。
