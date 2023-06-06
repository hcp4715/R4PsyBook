---
editor_options: 
  markdown: 
    wrap: 72
---

# 第五讲：如何清理数据—数据的预处理{#lesson-5}



## **批量读取文件**

在这节课中我们会讲述如何将我们从实验或者问卷中获得的原始数据进行整理，转化为可以用于统计分析的数据，我们有两种方法，分别是for loop方法和lapply函数方法，他们都可以将多个散数据合并为一个完整的数据。
两个方法各有优劣：
（1）for loop方法思维难度低，书写难度高，更好理解，但是代码更长，并且存在中间变量。
（2）lapply方法思维难度高，书写难度低，更难理解，但是代码更简洁，存在"看不见"的局部变量。

在我们的示例数据中，会发现有很多子文件，比如以practice、match和category等结尾的数据，它表示的是三个阶段：practice表示练习阶段，match表示match任务阶段，category表示categorization任务阶段。现在我们需要找出所有包含match且以out结尾的文件，这些才是我们需要读取并合并的数据。

### **通配符**

逐个列出我们需要合并的文件太过于麻烦且低效，我们可以通过list来列出所有文件名，然后根据一定的规则来筛选，这样可以快速得到我们需要合并的文件名。这个时候我们就要用到通配符了。

通配符是一种特殊字符，它可以在匹配文件名或其他文本字符串时代替其他字符。R中常使用的通配符包括"*""?"和"[]"
*：代表任意数量的字符，例如：
*.csv将匹配所有以.csv结尾的文件。
?：代表单个字符，例如file?.txt将匹配file1.txt，file2.txt等文件，但不会匹配file10.txt。
[]：用于匹配指定的一组字符。例如，file[123].txt将匹配file1.txt，file2.txt和file3.txt。

我们要做的第一件事是扫描这个文件夹，把里面所有的文件和文件夹都读取出来。然后我们使用通配符来匹配文件夹里包含特殊信息的文件。也就是说，我们需要根据文件名是否包含match来筛选文件夹里的文件。

我们可能更多的使用*字符，因为如果使用?这个字符的话。file?.txt，能够匹配的符合条件的文件就是file1、file2、file3等含单个字符的文件，当筛选到file10时，就会因为后面有两个字符而不匹配。

中括号里的字符是“或”的关系，就是说，中括号里的123代表file1后面跟1、2或3都可以。这样可以任意灵活地匹配。因为有可能你不知道文件夹里有多少个文件，你就把它们全部写在中括号里，只要它包含在里面，我们就把它读取出来。


```r
library(DT)
# 所有路径使用相对路径
library(here)
```

```
## here() starts at /Users/sumsum/Documents/GitHub/R4PsyBook/bookdown_files/Books/Book
```

```r
# 包含了dplyr和%>%等好用的包的集合
library(tidyverse)
# 养成用相对路径的好习惯，便于其他人运行你的代码
WD <-  here::here()
getwd()
```

```
## [1] "/Users/sumsum/Documents/GitHub/R4PsyBook/bookdown_files/Books/Book"
```

### for**循环思路**

那么我们该怎么使用它呢？我们需要先加载tidyverse这个包，提醒大家在开始处理之前要load这个包，否则在使用函数时会出错。


```r
# 把所有符合某种标题的文件全部读取到一个list中，也就是说，在相对路径"data/match"中抓取文件名包含"data_exp7_rep_match_"的文件
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

我们先使用files来存储我们从这个文件夹里扫描到的所有文件的名字
在R中，list.files()是一个函数，它可以扫描文件夹里的所有文件。第一个参数是你要在哪个文件夹里扫描————即在当前工作目录里的data文件夹中，然后在这个文件夹中的match文件夹里进行扫描。同时，我们使用pattern来筛选文件夹里的文件名，比如包含了"match"的，或者更加精确的，筛选以data_exp7__rep开头的文件。并使用一个通配符来表示筛选所有以out结尾的文件。
函数运行后，我们就得到了所有符合筛选目标的文件名。我们只会得到它们的文件名，不包含它们的路径。如果我们把它们读取出来并列出前面10个文件名，可以看到它们确实完全符合我们筛选的规则。

读取完后，我们可以使用for循环。首先，我们需要创建一个空的列表来存储读取的数据。for循环的结构是for（条件）{内容}。我们使用i来循环，in表示我们要循环的范围。我们首先将i设置为1，然后做某些事情，然后在i等于2时再做同样的事情，一直到i等于10。


```r
# 创建一个空的列表来存储读取的数据框
df_list <- list()
# 循环读取每个文件，处理数据并添加到列表中
for (i in seq_along(files)) { # 重复"读取到的.out个数"的次数。根据files的list长度生成数值的sequence。对files里的第一个值执行read.table()，并将结果放进df_list()中，接着对第二个值执行，直至循环结束整个files。
  # 对每个.out，使用read.table
  df <- read.table(file.path("data/match", files[i]), header = TRUE) #read.table似乎比read.csv更聪明，不需要指定分隔符
  # 给读取到的每个.out文件的每个变量统一变量格式
  df <- dplyr::filter(df, Date != "Date") %>% # 因为有些.out文件中部还有变量名，所以需要用filter把这些行过滤掉
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

在for循环中，我们使用df=read.table来读取文件。文件的路径是相对路径，是data match和files的组合。files是文件夹中的一个文件。
当i等于1时，我们读取的是第一个文件。我们按照header为true的方式去读取。
我们发现out文件里保存的数据有些小问题（包含多个title行），因此我们需要去掉重复的行。我们可以用一个规则，即如果data再次出现，表示它重复了上面这个列名，我们就把所有这样的行去掉。
之后，我们还可以用as.character和as.numerical来统一每个变量的变量类型。
然后，把数据放入数据框中。当i等于1时，它就是对第一个文件做了上述操作。然后，我们把它复制到数据列表里面，第一个位置就是第一个out文件，它经过了转换之后的数据框，就装到数据框列表里面的第一个位置。然后，我们通过循环读取id位中的第二个文件，并倒数10个，直到读取完所有10个文件。

将所有数据放入列表后，我们可以使用bind_rows函数将它们合并成一个大的数据框。bind_rows是Dplyr中一个常用的函数，它可以通过行将不同的数据框合并。因此，data列表实际上是一个完整的列表，其中包含一个一个的out文件的数据，经过初步转换后形成的数据框。由于它们的列都是一模一样的，我们可以直接将它们合并成一个完整的数据框。

这就是for循环的逻辑：（1）获取想要读取的文件名；（2）通过迭代的方式依次读取这些文件名，并将它们放入一个数据列表（DataList）中；（3）对这个List进行合并得到完整数据框（Dataframe)。
这个操作思路非常简洁易懂。当然，真正理解for循环的前提是要正确理解它的工作原理。此外，for循环还可以用在很多场景中：
（1）进行重复性操作。
当你需要重复大量做某件事情而没有现成函数可以辅助时，最简单的方法就是写一个for循环来完成。i可以替代很多代码，比如你有五个对象需要做同样的操作，你可以写一个i从1到5的for循环，将每个对象放进去做同样的操作。否则，你就需要不断复制你的代码，然后改变其中的一个值，再运行一遍，这样既费时又费力。
（2）应用于数据的批处理。
如果你经常进行批量处理工作，学会for循环会非常有帮助。它可以帮助我们更方便地读取并合并文件夹中的文件。

最终的结果应该是25920 obs 16 variables


```{=html}
<div class="datatables html-widget html-fill-item-overflow-hidden html-fill-item" id="htmlwidget-086044d87f0f0deda6f6" style="width:100%;height:auto;"></div>
<script type="application/json" data-for="htmlwidget-086044d87f0f0deda6f6">{"x":{"filter":"none","vertical":false,"fillContainer":true,"data":[["1","2","3","4","5","6","7","8","9","10","11","12","13","14","15","16","17","18","19","20","21","22","23","24","25","26","27","28","29","30","31","32","33","34","35","36","37","38","39","40","41","42","43","44","45","46","47","48","49","50","51","52","53","54","55","56","57","58","59","60","61","62","63","64","65","66","67","68","69","70","71","72","73","74","75","76","77","78","79","80","81","82","83","84","85","86","87","88","89","90","91","92","93","94","95","96","97","98","99","100"],["02-May-2018_14:23:06","02-May-2018_14:23:08","02-May-2018_14:23:10","02-May-2018_14:23:13","02-May-2018_14:23:15","02-May-2018_14:23:17","02-May-2018_14:23:19","02-May-2018_14:23:21","02-May-2018_14:23:24","02-May-2018_14:23:26","02-May-2018_14:23:28","02-May-2018_14:23:30","02-May-2018_14:23:32","02-May-2018_14:23:34","02-May-2018_14:23:36","02-May-2018_14:23:38","02-May-2018_14:23:40","02-May-2018_14:23:42","02-May-2018_14:23:45","02-May-2018_14:23:47","02-May-2018_14:23:49","02-May-2018_14:23:51","02-May-2018_14:23:53","02-May-2018_14:23:55","02-May-2018_14:23:57","02-May-2018_14:24:00","02-May-2018_14:24:02","02-May-2018_14:24:04","02-May-2018_14:24:06","02-May-2018_14:24:08","02-May-2018_14:24:10","02-May-2018_14:24:12","02-May-2018_14:24:15","02-May-2018_14:24:16","02-May-2018_14:24:19","02-May-2018_14:24:21","02-May-2018_14:24:23","02-May-2018_14:24:25","02-May-2018_14:24:27","02-May-2018_14:24:29","02-May-2018_14:24:32","02-May-2018_14:24:34","02-May-2018_14:24:36","02-May-2018_14:24:38","02-May-2018_14:24:40","02-May-2018_14:24:42","02-May-2018_14:24:44","02-May-2018_14:24:46","02-May-2018_14:24:48","02-May-2018_14:24:51","02-May-2018_14:24:53","02-May-2018_14:24:55","02-May-2018_14:24:57","02-May-2018_14:24:59","02-May-2018_14:25:01","02-May-2018_14:25:03","02-May-2018_14:25:05","02-May-2018_14:25:08","02-May-2018_14:25:10","02-May-2018_14:25:12","02-May-2018_14:25:14","02-May-2018_14:25:16","02-May-2018_14:25:19","02-May-2018_14:25:21","02-May-2018_14:25:23","02-May-2018_14:25:25","02-May-2018_14:25:27","02-May-2018_14:25:30","02-May-2018_14:25:32","02-May-2018_14:25:34","02-May-2018_14:25:36","02-May-2018_14:25:38","02-May-2018_14:25:53","02-May-2018_14:25:55","02-May-2018_14:25:57","02-May-2018_14:26:00","02-May-2018_14:26:02","02-May-2018_14:26:04","02-May-2018_14:26:06","02-May-2018_14:26:08","02-May-2018_14:26:10","02-May-2018_14:26:11","02-May-2018_14:26:14","02-May-2018_14:26:16","02-May-2018_14:26:18","02-May-2018_14:26:20","02-May-2018_14:26:22","02-May-2018_14:26:25","02-May-2018_14:26:26","02-May-2018_14:26:28","02-May-2018_14:26:31","02-May-2018_14:26:33","02-May-2018_14:26:35","02-May-2018_14:26:37","02-May-2018_14:26:39","02-May-2018_14:26:41","02-May-2018_14:26:43","02-May-2018_14:26:45","02-May-2018_14:26:47","02-May-2018_14:26:49"],["Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp","Exp"],[7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302,7302],[22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22,22],["female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female","female"],["R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R","R"],[1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1],[1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,4,5,5,5,5],[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,1,2,3,4],["immoralSelf","moralOther","immoralOther","moralSelf","immoralSelf","immoralSelf","moralOther","moralSelf","moralOther","immoralSelf","moralOther","immoralOther","moralOther","moralSelf","immoralOther","immoralSelf","moralSelf","moralSelf","immoralSelf","moralOther","immoralOther","moralSelf","immoralOther","immoralOther","moralSelf","moralSelf","immoralOther","immoralOther","immoralSelf","immoralOther","moralSelf","moralOther","immoralSelf","moralOther","immoralSelf","moralSelf","immoralSelf","moralOther","moralOther","immoralSelf","moralOther","moralSelf","moralSelf","immoralOther","immoralOther","immoralSelf","immoralOther","moralOther","immoralOther","moralOther","moralSelf","immoralSelf","moralOther","moralOther","immoralOther","moralSelf","immoralOther","moralSelf","immoralSelf","moralSelf","immoralSelf","immoralOther","immoralSelf","moralOther","immoralSelf","moralOther","immoralSelf","immoralOther","moralOther","moralSelf","immoralOther","moralSelf","immoralSelf","moralOther","moralSelf","moralSelf","immoralOther","immoralOther","immoralSelf","moralOther","moralOther","moralSelf","immoralSelf","moralSelf","moralOther","immoralOther","immoralOther","immoralSelf","moralSelf","moralSelf","immoralSelf","immoralOther","moralOther","moralOther","immoralOther","immoralSelf","moralOther","moralOther","immoralOther","moralSelf"],["immoralSelf","moralOther","immoralOther","moralSelf","immoralSelf","immoralSelf","moralOther","moralSelf","moralOther","immoralSelf","moralOther","immoralOther","moralOther","moralSelf","immoralOther","immoralSelf","moralSelf","moralSelf","immoralSelf","moralOther","immoralOther","moralSelf","immoralOther","immoralOther","moralSelf","moralSelf","immoralOther","immoralOther","immoralSelf","immoralOther","moralSelf","moralOther","immoralSelf","moralOther","immoralSelf","moralSelf","immoralSelf","moralOther","moralOther","immoralSelf","moralOther","moralSelf","moralSelf","immoralOther","immoralOther","immoralSelf","immoralOther","moralOther","immoralOther","moralOther","moralSelf","immoralSelf","moralOther","moralOther","immoralOther","moralSelf","immoralOther","moralSelf","immoralSelf","moralSelf","immoralSelf","immoralOther","immoralSelf","moralOther","immoralSelf","moralOther","immoralSelf","immoralOther","moralOther","moralSelf","immoralOther","moralSelf","immoralSelf","moralOther","moralSelf","moralSelf","immoralOther","immoralOther","immoralSelf","moralOther","moralOther","moralSelf","immoralSelf","moralSelf","moralOther","immoralOther","immoralOther","immoralSelf","moralSelf","moralSelf","immoralSelf","immoralOther","moralOther","moralOther","immoralOther","immoralSelf","moralOther","moralOther","immoralOther","moralSelf"],["mismatch","mismatch","mismatch","mismatch","match","match","match","match","mismatch","mismatch","mismatch","match","match","mismatch","match","mismatch","mismatch","match","match","match","mismatch","match","match","mismatch","match","mismatch","match","mismatch","mismatch","match","match","match","mismatch","match","mismatch","mismatch","match","mismatch","mismatch","match","mismatch","mismatch","match","mismatch","match","match","mismatch","match","match","mismatch","mismatch","mismatch","match","match","mismatch","mismatch","match","mismatch","match","match","match","mismatch","mismatch","match","match","mismatch","mismatch","mismatch","mismatch","match","match","match","match","mismatch","mismatch","match","match","match","mismatch","match","match","match","mismatch","mismatch","mismatch","mismatch","mismatch","mismatch","mismatch","match","match","mismatch","match","mismatch","match","match","mismatch","match","match","match"],["n","n","n","n","m","m","m","m","n","n","n","m","m","n","m","n","n","m","m","m","n","m","m","n","m","n","m","n","n","m","m","m","n","m","n","n","m","n","n","m","n","n","m","n","m","m","n","m","m","n","n","n","m","m","n","n","m","n","m","m","m","n","n","m","m","n","n","n","n","m","m","m","m","n","n","m","m","m","n","m","m","m","n","n","n","n","n","n","n","m","m","n","m","n","m","m","n","m","m","m"],["m","n","n",null,"m","m","m","m","n","n","n","m","m","m","m","n","n","m","n","m","n","m","m","n","m","m","m","m","n","m","m","m","m","m",null,"m","m","m","n","m","n","m","m","n","m","m","n","m","m","n",null,"n","m","m","n","n","m","n","m","n","m","n","n","m","n","n","n","n","n","m","n","m","m","n","m","n","m","m","n","m","m","m","n","n","n","n","n","n","m","m","m","n","m","n","m","m","n","n","m","m"],[0,1,1,-1,1,1,1,1,1,1,1,1,1,0,1,1,1,1,0,1,1,1,1,1,1,0,1,0,1,1,1,1,0,1,-1,0,1,0,1,1,1,0,1,1,1,1,1,1,1,1,-1,1,1,1,1,1,1,1,1,0,1,1,1,1,0,1,1,1,1,1,0,1,1,1,0,0,1,1,1,1,1,1,1,1,1,1,1,1,0,1,1,1,1,1,1,1,1,0,1,1],[0.7561,0.7043,0.9903,1.042,0.8207,0.7547,0.5429,0.9009,0.9551,0.6952,0.7593,0.7135,0.5656,0.5357,0.8078,0.96,0.6661,0.6962,0.8803,0.5785,0.7845,0.8146,0.6548,0.8789,0.7131,0.8211,0.8033,0.6294,0.8095,0.6176,0.7917,0.5559,0.96,0.5381,1.042,0.8264,0.7125,0.4609,0.8027,0.7808,0.8749,0.871,0.6512,0.7554,0.6715,0.9076,0.5997,0.5218,0.7319,0.764,1.042,0.7443,0.5104,0.7706,0.6026,0.7648,0.7109,0.827,0.8571,0.8092,0.7213,0.8195,0.8896,0.5778,0.8799,0.666,1.0081,0.8562,0.5444,0.6625,0.6846,0.9027,0.8236,0.6137,0.7338,0.7419,0.684,0.6062,0.7842,0.5824,0.4126,0.5006,0.8507,0.9069,0.721,0.6351,0.8553,0.7194,0.5214,0.6597,0.8097,0.6738,0.534,0.6141,0.6462,0.7763,0.6964,0.8805,0.5546,0.9747]],"container":"<table class=\"display fill-container\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>Date<\/th>\n      <th>Prac<\/th>\n      <th>Sub<\/th>\n      <th>Age<\/th>\n      <th>Sex<\/th>\n      <th>Hand<\/th>\n      <th>Block<\/th>\n      <th>Bin<\/th>\n      <th>Trial<\/th>\n      <th>Shape<\/th>\n      <th>Label<\/th>\n      <th>Match<\/th>\n      <th>CorrResp<\/th>\n      <th>Resp<\/th>\n      <th>ACC<\/th>\n      <th>RT<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"pageLength":7,"columnDefs":[{"className":"dt-right","targets":[3,4,7,8,9,15,16]},{"orderable":false,"targets":0}],"order":[],"autoWidth":false,"orderClasses":false,"lengthMenu":[7,10,25,50,100]}},"evals":[],"jsHooks":[]}</script>
```

### lapply思路

我们可以使用管道操作符“%>%”和lapply这样更方便的工具来代替for循环完成批量合并数据这类常用的操作。这个符号在Dplyr包中
管道符能够将前一步的结果作为下一个函数的输入。我们首先列出符合条件的文件名，并将其作为一个列表输入到lapply函数中。lapply函数是一个apply系列的函数，它将函数应用于一个列表上。
在这里，我们将read.table函数应用于列表中的每个元素，其中文件路径是x的参数，head=true是read.table的参数。这个操作实际上就是用一行命令代替了for循环的操作。


```r
# 获取所有的.out文件名
df.mt.out.la <- list.files(file.path("data/match"), pattern = "data_exp7_rep_match_.*\\.out$") %>%
  # 对读取到的所有.out文件x都执行函数read.table（x就是一个代称，指将上一步读取到的所有文件名暂时命名为x，命名为abc也可以。只要保证function()括号内和后面的内容是一致的就可以）
  lapply(.,function(x) read.table(file.path("data/match", x), header = TRUE)) %>% 
  # 对所有被read.table处理过的数据执行dplyr的清洗
  lapply(function(df) dplyr::filter(df, Date != "Date") %>% # 因为有些.out文件中部还有变量名，所需需要用filter把这些行过滤掉
                      dplyr::mutate(Date = as.character(Date),Prac = as.character(Prac),
                                    Sub = as.numeric(Sub),Age = as.numeric(Age),Sex = as.character(Sex),Hand = as.character(Hand),
                                    Block = as.numeric(Block),Bin = as.numeric(Bin),Trial = as.numeric(Trial),
                                    Shape = as.character(Shape),Label = as.character(Shape),Match = as.character(Match),
                                    CorrResp = as.character(CorrResp),Resp = as.character(Resp),
                                    ACC = as.numeric(ACC),RT = as.numeric(RT)
                                    ) # 有些文件里读出来的数据格式不同，在这里统一所有out文件中的数据格式成字符串类型或数值型
         ) %>%
  bind_rows()
```

最终，我们得到一个包含所有数据框的列表，这些数据框都经过了一系列的转换和操作，然后使用“bind_rows”将它们合并成一个数据框。
虽然这个过程看起来很冗长，但思路清晰。最终，我们得到一个包含16个变量和25000多个观测值的数据框。

我们读取完后，可以将读取后的数据保存一下，以便下次直接读取。
我们前面整理并合并后的数据框需要一个名字和路径。我们可以使用相对路径，在当前工作目录下的data/match文件夹中将其保存为match_row.csv。一个常用的参数是强制将行名写入文件中，即row.names等于FALSE。


```r
#for loop 或 lapply的都可以
write.csv(df.mt.out.fl, file = "./data/match/match_raw.csv",row.names = FALSE)
#write.csv(df.mt.out.la, file = "./data/match/match_raw.csv",row.names = FALSE)
```

## 数据预处理

假设我们已经准备好了match和penguin的数据，我们可以读取数据并开始数据预处理。我们使用刚才保存的csv文件，使用header和sep参数设定来处理数据。

在tidyverse中，filter和mutate是常用的功能之一。group_by可以根据某些变量对数据进行分组，但一定要记得使用ungroup。
使用summarise可以计算均值、标准差、标准误等统计量。将group_by和summarise结合起来，我们可以快速有效地得到心理学中常用的统计量，如均值、标准差、标准误等。
我们可以对数据进行分组和条件筛选。“select”函数是用来选择列，与“filter”函数选择行不同。有时我们也可以用“select”函数重新排序，而“arrange”函数则是用来对整个数据框按某一列的值进行排序。

接下来我们来看一个例子，假设我们要选择1995年或之后出生的人，我们可以在管道中使用“filter”函数。在“dplyr”中，函数的参数是有顺序的，第一个是数据框。我们可以用.来代表输入数据，然后用“age”作为筛选条件。筛选完后，我们可以把结果输入到下一个函数中。如果要保留结果，我们需要把它赋值到一个新的变量中。


```r
# 读取原始数据
df.pg.raw <-  read.csv('./data/penguin/penguin_rawdata.csv',
                       header = T, sep=",", stringsAsFactors = FALSE)
# 在数据框pg_raw中选择变量，使用select选择age和ALEX开头的所有题目
df.clean.select <- df.pg.raw %>%
  dplyr::select(age, starts_with("ALEX"), eatdrink, avoidance)
#笨一点的方法，就是把16个ALEX都写出来
```

```{=html}
<div class="datatables html-widget html-fill-item-overflow-hidden html-fill-item" id="htmlwidget-b9b18a93680a1e732d58" style="width:100%;height:auto;"></div>
<script type="application/json" data-for="htmlwidget-b9b18a93680a1e732d58">{"x":{"filter":"none","vertical":false,"fillContainer":true,"data":[["1","2","3","4","5","6","7","8","9","10"],[1975,1995,1995,1988,1991,1995,1996,1973,1996,1996],[1,2,4,2,2,2,1,3,2,3],[1,2,1,3,1,3,1,3,3,3],[1,2,1,4,1,1,1,2,1,1],[2,2,2,5,5,3,1,3,3,2],[2,2,4,3,2,2,4,4,2,4],[1,2,1,4,2,1,1,4,2,1],[2,2,4,2,3,1,2,4,2,2],[2,2,1,4,4,2,1,4,2,4],[4,4,1,4,1,1,4,4,2,4],[1,2,2,3,1,2,2,5,2,1],[1,2,1,2,2,1,1,3,1,4],[2,2,1,2,1,2,2,2,2,1],[4,2,2,4,2,3,4,2,2,2],[3,2,1,2,4,4,3,3,4,3],[4,3,1,4,4,5,5,4,3,5],[2,2,1,3,2,4,5,1,2,4],[1,1,1,2,1,1,1,1,1,1],[3.27777777777778,3,1.61111111111111,3.94444444444444,4.94444444444444,3.77777777777778,6.44444444444444,3.22222222222222,4,5.27777777777778]],"container":"<table class=\"display fill-container\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>age<\/th>\n      <th>ALEX1<\/th>\n      <th>ALEX2<\/th>\n      <th>ALEX3<\/th>\n      <th>ALEX4<\/th>\n      <th>ALEX5<\/th>\n      <th>ALEX6<\/th>\n      <th>ALEX7<\/th>\n      <th>ALEX8<\/th>\n      <th>ALEX9<\/th>\n      <th>ALEX10<\/th>\n      <th>ALEX11<\/th>\n      <th>ALEX12<\/th>\n      <th>ALEX13<\/th>\n      <th>ALEX14<\/th>\n      <th>ALEX15<\/th>\n      <th>ALEX16<\/th>\n      <th>eatdrink<\/th>\n      <th>avoidance<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"pageLength":3,"columnDefs":[{"className":"dt-right","targets":[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19]},{"orderable":false,"targets":0}],"order":[],"autoWidth":false,"orderClasses":false,"lengthMenu":[3,10,25,50,100]}},"evals":[],"jsHooks":[]}</script>
```

starts_with是tidyverse中的一个包，可以方便地选择以“ALEX”开头的所有列。它本质上是一个简化的通配符，因为在dplyr中，我们经常需要选择以某个特定开头或结尾的列，如果每次都写通配符会很麻烦。使用starts_with包可以直接选择以“ALEX”开头的列。

使用mutate函数可以生成一个新的变量，不仅仅是求和，还可以进行任意转换，比如加减乘除或判断。在这里，我们使用mutate函数将前四个“ALEX”列的得分求和，得到一个新的变量“ALEX_SUM”，表示前四个“ALEX”列的得分总和。


```r
# 把ALEX1 - 4求和
df.clean.mutate_1 <- df.pg.raw %>% 
  dplyr::mutate(ALEX_SUM = ALEX1 + ALEX2 + ALEX3 + ALEX4)
```

```{=html}
<div class="datatables html-widget html-fill-item-overflow-hidden html-fill-item" id="htmlwidget-332cbe2facec2de4f317" style="width:100%;height:auto;"></div>
<script type="application/json" data-for="htmlwidget-332cbe2facec2de4f317">{"x":{"filter":"none","vertical":false,"fillContainer":true,"data":[["1","2","3","4","5","6","7","8","9","10"],[1975,1995,1995,1988,1991,1995,1996,1973,1996,1996],[1,2,4,2,2,2,1,3,2,3],[1,2,1,3,1,3,1,3,3,3],[1,2,1,4,1,1,1,2,1,1],[2,2,2,5,5,3,1,3,3,2],[2,2,4,3,2,2,4,4,2,4],[1,2,1,4,2,1,1,4,2,1],[2,2,4,2,3,1,2,4,2,2],[2,2,1,4,4,2,1,4,2,4],[4,4,1,4,1,1,4,4,2,4],[1,2,2,3,1,2,2,5,2,1],[1,2,1,2,2,1,1,3,1,4],[2,2,1,2,1,2,2,2,2,1],[4,2,2,4,2,3,4,2,2,2],[3,2,1,2,4,4,3,3,4,3],[4,3,1,4,4,5,5,4,3,5],[2,2,1,3,2,4,5,1,2,4],[3.72222222222222,4.05555555555556,1.44444444444444,4.72222222222222,2.11111111111111,2,2.55555555555556,3.05555555555556,3.55555555555556,4.38888888888889],[6,0,0,2,0,0,null,0,0,0],[3.11111111111111,4.88888888888889,1.33333333333333,3.88888888888889,4.66666666666667,3.33333333333333,3.11111111111111,3.55555555555556,3.33333333333333,3.22222222222222],[3.44444444444444,2,2,3,1.77777777777778,2.44444444444444,3.77777777777778,3.33333333333333,1.88888888888889,4.22222222222222],[89,89,89,89,89,89,89,89,89,89],[3.27777777777778,3,1.61111111111111,3.94444444444444,4.94444444444444,3.77777777777778,6.44444444444444,3.22222222222222,4,5.27777777777778],[15,null,null,null,null,null,null,10,null,1],[1.63636363636364,2.18181818181818,2,3.27272727272727,2.18181818181818,1.72727272727273,1.72727272727273,3.54545454545455,2,2.63636363636364],[1,1,1,2,1,1,1,1,1,1],[5,6,1,3,1,1,2,3,4,5],[5,4,1,6,1,1,1,2,4,6],[3,2,1,6,1,1,4,3,4,6],[5,5,1,5,1,2,1,2,4,5],[5,5,1,4,1,2,4,3,4,4],[3,2,1,4,5,1,1,3,1,3],[1,3,1,3,2,1,2,3,4,5],[6,5,1,3,1,2,7,3,4,7],[5,2,1,4,7,6,1,3,4,3],[2,3,1,4,1,2,4,1,4,3],[6,7,1,6,6,2,2,5,2,6],[3,4,1,5,1,2,1,2,4,4],[3,3,1,6,2,4,2,5,4,4],[2,3,7,5,1,1,1,4,1,2],[3,4,1,5,2,2,1,5,4,1],[3,5,3,5,1,1,2,3,4,4],[6,5,1,6,2,3,6,3,4,7],[1,5,1,5,2,2,4,2,4,4],[2,3,2,5,1,5,7,3,4,5],[2,4,2,5,7,5,1,2,4,7],[5,3,2,5,2,4,6,4,4,5],[3,2,2,4,5,4,7,3,4,5],[3,2,2,3,2,3,7,3,4,6],[2,3,1,4,4,4,7,3,4,5],[5,3,1,6,2,3,7,4,4,6],[5,4,1,3,6,3,7,4,4,7],[3,5,1,3,7,1,7,4,4,7],[2,3,1,4,6,3,6,3,4,4],[4,2,2,5,6,6,6,3,4,4],[2,2,2,3,6,5,7,2,4,5],[3,4,1,4,7,4,7,2,4,3],[3,2,2,5,2,3,7,3,4,6],[4,3,3,3,7,4,6,3,4,5],[5,3,1,2,6,5,7,4,4,6],[4,3,1,3,6,3,7,6,4,5],[2,3,2,4,7,3,7,2,4,4],["9:23:38","9:23:26","8:57:08","8:55:14","8:54:35","8:39:12","8:32:08","8:28:57","8:28:03","8:25:57"],[3,2.2,1.2,3,2.6,3.6,3.8,2.4,2.6,3],[2,2,2,2,2,2,2,2,2,2],[0,0,12,0,5,0,null,1,0,2],[4,4,4,3,4,4,4,1,3,3],[4,5,1,5,5,4,5,3,3,4],[3,5,1,4,5,4,2,3,3,5],[4,5,4,5,5,4,4,2,4,3],[3,5,1,5,5,4,5,4,5,3],[3,5,1,4,4,4,2,3,4,4],[2,4,1,2,5,2,1,4,3,2],[4,5,1,2,4,2,5,5,2,2],[3,5,1,4,4,3,1,4,3,3],[2,5,1,4,5,3,3,4,3,3],[4,1,5,5,1,3,2,3,3,4],[4,2,6,4,1,3,2,3,3,5],[5,6,5,3,3,3,2,2,2,1],[4,2,4,1,2,2,2,2,1,3],[4,3,3,2,2,2,2,2,3,2],[1,2,1,2,2,4,2,3,3,3],[2,2,1,2,1,4,2,3,1,2],[17,28,24,19,29,33,53,17,17,19],[4.33333333333333,6.5,6.83333333333333,3.5,1.5,2,5.83333333333333,6.33333333333333,5.66666666666667,5.16666666666667],[2.27272727272727,2.63636363636364,2.18181818181818,2,1.90909090909091,2.45454545454545,2.81818181818182,2.54545454545455,1.81818181818182,3.90909090909091],[2,3,3,1,3,4,4,2,2,5],[2,2,4,1,2,4,4,3,3,4],[2,3,2,2,1,2,2,3,2,5],[4,3,4,3,4,4,2,2,2,4],[2,3,1,2,1,3,3,3,2,3],[2,2,1,2,1,1,2,2,1,3],[2,2,1,2,1,2,2,4,2,4],[2,3,3,1,1,1,2,3,2,3],[3,3,3,2,2,2,4,2,1,3],[2,3,1,4,2,1,2,1,1,4],[2,2,1,2,3,3,4,3,2,5],[4,2,1,2,2,3,3,3,3,4],[4,2,4,4,1,3,4,4,2,4],[2,2,1,3,1,2,5,3,1,2],[5,2,4,2,1,3,4,3,4,4],[2,2,1,3,1,2,1,2,3,5],[2,2,1,2,3,2,3,4,1,4],[4,2,1,3,2,2,5,3,1,5],[4,2,1,4,1,3,5,4,1,5],[4,2,4,4,4,2,4,4,1,5],[2,2,1,1,2,2,2,2,2,2],[2,4,2,4,3,5,2,1,3,3],[2,4,3,2,5,5,4,4,3,4],[2,4,3,1,5,5,2,5,5,5],[1,4,2,1,3,2,2,2,5,1],[1,4,4,2,5,3,1,2,5,2],[4,3,2,1,5,4,3,3,3,1],[2,4,4,2,5,3,2,3,4,2],[2,2,1,3,4,3,2,2,5,4],[5,4,3,4,4,5,4,4,3,3],[4,3,5,3,3,5,5,3,4,2],[3,4,4,2,4,5,3,2,2,5],[5,5,4,2,4,4,1,3,4,3],[4,4,4,2,4,3,2,2,2,2],[2.84615384615385,3.76923076923077,3.15384615384615,2.23076923076923,4.15384615384615,4,2.53846153846154,2.76923076923077,3.69230769230769,2.84615384615385],[1,1,1,2,1,1,2,1,1,2],[1,2,2,2,2,2,2,1,2,1],[2,1,1,1,2,2,2,4,2,2],[1,1,1,1,1,1,1,1,1,1],[null,null,null,null,null,null,null,null,null,null],[4,4,4,4,2,4,4,4,4,4],[4,4,4,4,2,4,4,2,4,4],[5,1,4,5,5,5,5,5,5,5],[null,null,4,null,null,null,null,null,null,null],[1,8,2,1,8,2,3,8,2,6],[null,6,2,null,5,1,2,3,2,3],[6,4,5,5,7,5,8,4,3,8],[3,1,5,1,3,5,8,4,2,8],[1,1,1,1,2,1,1,1,1,1],[null,null,null,null,8,null,null,null,null,null],[1,2,2,1,1,2,2,2,2,2],[null,8,8,null,null,8,8,8,8,8],[3,3,2,3,3,1,1,1,1,1],[2,8,1,8,1,null,null,null,null,null],[6,5,6,4,8,null,null,null,null,null],[1,3,1,1,1,8,6,4,1,3],[1,1,1,1,1,1,1,2,1,1],[null,null,null,null,null,null,null,2,null,null],[1,1,1,2,1,2,2,2,2,1],[" "," "," "," "," ","Rowing Club","University Friends","ODAA","Oxford Latin Speaking Society"," "],[" "," "," "," "," ","Rugby Club","Business Social"," ","Oxford Latin Conversation Society"," "],[" "," "," "," "," "," ","Lecturers/ Advisors"," ","Plato Reading Group"," "],[" "," "," "," "," "," ","Friends from home"," "," "," "],[" "," "," "," "," "," ","Neighbors"," "," "," "],[null,null,null,null,null,7,20,1,2,null],[null,null,null,null,null,7,15,null,4,null],[null,null,null,null,null,null,5,null,1,null],[null,null,null,null,null,null,9,null,null,null],[null,null,null,null,null,null,6,null,null,null],[6,6,7,2,1,1,6,6,7,7],[5,7,7,5,2,2,7,6,7,4],[5,6,7,3,2,3,5,7,6,5],[3,7,7,4,1,2,5,6,6,5],[2,6,7,5,2,2,6,7,5,5],[5,7,6,2,1,2,6,6,3,5],[4,6,6,4,1,2,5,6,3,4],[5,7,8,5,7,6,null,7,6,6],[1,2,3,1,2,4,null,2,2,3],[4,2,4,4,1,2,2,3,3,1],[4,4,5,2,1,4,4,2,5,4],[4,3,4,3,1,4,2,3,4,4],[5,3,4,3,4,4,4,4,2,4],[2,3,4,4,4,4,5,2,2,3],[2,4,4,3,1,3,4,3,1,3],[2,5,4,3,3,4,5,4,2,2],[2,2,4,3,4,3,5,3,2,4],[4,5,5,2,1,2,4,2,4,5],[2,3,2,2,1,4,1,4,4,5],[2,2,1,2,1,2,2,2,2,1],[1,2,2,2,4,4,4,2,2,2],[4,4,4,3,4,2,2,3,2,1],[5,4,2,3,2,4,3,4,4,4],[4,4,2,3,3,1,2,4,5,3],[2,3,4,4,4,5,5,4,4,4],[2,3,5,4,4,3,2,4,2,2],[4,3,4,2,4,3,4,3,3,2],[3,3,5,2,4,4,4,4,3,3],[2,3,1,3,5,4,1,3,2,4],[4,3,4,2,4,4,5,4,3,5],[3,3,5,4,4,3,2,2,3,3],[2,3,1,3,4,3,2,3,4,3],[3,3,1,4,1,2,2,2,3,3],[4,3,5,3,1,2,5,1,4,4],[3,3,3,2,1,3,3,2,3,3],[4,3,1,4,4,4,2,4,1,1],[4,5,4,2,1,3,1,4,4,5],[3,5,4,4,1,2,1,3,5,5],[3,2,2,4,4,2,5,2,2,4],[2,4,2,5,1,2,1,3,2,4],[4,3,1,2,4,2,5,1,1,4],[4,3,2,2,3,3,1,4,3,4],[1,4,3,4,4,2,1,3,2,2],[2,4,2,3,5,4,2,5,1,4],[2,4,2,4,5,4,4,4,2,4],[4,4,4,3,4,4,2,4,2,4],[3,4,2,4,4,4,4,5,5,1],[4,2,2,3,4,4,5,2,2,4],[5,3,3,2,4,2,2,2,2,4],[4,3,2,3,3,3,2,1,2,4],[4,3,5,2,4,4,4,4,4,4],[1,4,2,3,4,2,2,3,2,3],[1,2,2,4,4,2,1,2,1,2],[4,3,2,3,1,2,1,4,4,2],[4,4,4,2,1,4,1,4,3,3],[3,3,4,3,4,2,4,3,3,4],[1,2,5,4,4,2,1,2,1,2],[2,3,5,4,4,2,1,2,1,2],[4,3,5,4,4,4,3,3,3,4],[4,4,2,5,1,3,5,2,4,5],[2,4,5,3,1,2,5,4,2,4],[2,4,4,3,4,2,1,3,2,2],[4,3,4,3,5,2,4,4,2,4],[2,4,2,2,4,3,3,2,2,4],[2,4,1,2,4,2,2,2,2,2],[1,2,5,4,1,1,4,2,2,2],[3.07692307692308,2.46153846153846,2.38461538461538,2.84615384615385,2.53846153846154,2.07692307692308,1.53846153846154,3,2.61538461538462,3.15384615384615],[4,2,3,3,3,2,1,4,5,3],[2,3,3,3,4,2,2,3,2,4],[4,3,4,4,3,3,1,3,3,5],[4,3,1,2,2,2,2,3,2,4],[4,2,2,3,2,2,2,3,2,3],[3,2,1,2,1,2,2,3,2,1],[4,2,2,3,3,2,1,4,2,3],[3,2,2,2,2,2,1,4,3,4],[3,2,3,3,2,2,2,1,2,3],[2,3,3,3,3,2,1,3,4,3],[4,2,3,4,3,2,2,2,1,2],[3,4,4,3,5,4,4,4,5,4],[2,4,2,3,2,2,2,3,3,2],[1,2,2,2,3,2,1,3,3,4],[34.9,35.8,34,36.9,35.3,34.7,36,36.5,34.7,35.9],[35.9,36,36.2,37,36.1,36.4,36.3,36.6,36.3,36.3],[2,2,2,2,4,4,2,2,2,2],[35.4,35.9,35.1,36.95,35.7,35.55,36.15,36.55,35.5,36.1],[1,1,1,1,1,1,1,1,1,1],[-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667],[5,5,5,5,5,5,5,5,5,5],[1,1,1,1,1,1,1,1,1,1],[0.186319647739787,0.472682850177291,-1.77049556891649,1.0454092550523,-1.19776916404149,-1.29322356485399,-0.815951560791479,-0.386406757135223,0.0431380465210342,0.759046052614796],[-0.00190690952064737,-0.249805147204807,-1.48929633562561,0.593048860921336,1.48548251658431,0.44430991831084,2.82413300007877,-0.0514865570574791,0.642628508458168,1.7829604018053],["Oxford","Oxford","Oxford","Oxford","Oxford","Oxford","Oxford","Oxford","Oxford","Oxford"],[5,8,8,14,9,9,4,11,9,9]],"container":"<table class=\"display fill-container\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>age<\/th>\n      <th>ALEX1<\/th>\n      <th>ALEX2<\/th>\n      <th>ALEX3<\/th>\n      <th>ALEX4<\/th>\n      <th>ALEX5<\/th>\n      <th>ALEX6<\/th>\n      <th>ALEX7<\/th>\n      <th>ALEX8<\/th>\n      <th>ALEX9<\/th>\n      <th>ALEX10<\/th>\n      <th>ALEX11<\/th>\n      <th>ALEX12<\/th>\n      <th>ALEX13<\/th>\n      <th>ALEX14<\/th>\n      <th>ALEX15<\/th>\n      <th>ALEX16<\/th>\n      <th>anxiety<\/th>\n      <th>artgluctot<\/th>\n      <th>attachhome<\/th>\n      <th>attachphone<\/th>\n      <th>AvgHumidity<\/th>\n      <th>avoidance<\/th>\n      <th>cigs<\/th>\n      <th>DIDF<\/th>\n      <th>eatdrink<\/th>\n      <th>ECR1<\/th>\n      <th>ECR2<\/th>\n      <th>ECR3<\/th>\n      <th>ECR4<\/th>\n      <th>ECR5<\/th>\n      <th>ECR6<\/th>\n      <th>ECR7<\/th>\n      <th>ECR8<\/th>\n      <th>ECR9<\/th>\n      <th>ECR10<\/th>\n      <th>ECR11<\/th>\n      <th>ECR12<\/th>\n      <th>ECR13<\/th>\n      <th>ECR14<\/th>\n      <th>ECR15<\/th>\n      <th>ECR16<\/th>\n      <th>ECR17<\/th>\n      <th>ECR18<\/th>\n      <th>ECR19<\/th>\n      <th>ECR20<\/th>\n      <th>ECR21<\/th>\n      <th>ECR22<\/th>\n      <th>ECR23<\/th>\n      <th>ECR24<\/th>\n      <th>ECR25<\/th>\n      <th>ECR26<\/th>\n      <th>ECR27<\/th>\n      <th>ECR28<\/th>\n      <th>ECR29<\/th>\n      <th>ECR30<\/th>\n      <th>ECR31<\/th>\n      <th>ECR32<\/th>\n      <th>ECR33<\/th>\n      <th>ECR34<\/th>\n      <th>ECR35<\/th>\n      <th>ECR36<\/th>\n      <th>endtime<\/th>\n      <th>EOT<\/th>\n      <th>exercise<\/th>\n      <th>gluctot<\/th>\n      <th>health<\/th>\n      <th>HOME1<\/th>\n      <th>HOME2<\/th>\n      <th>HOME3<\/th>\n      <th>HOME4<\/th>\n      <th>HOME5<\/th>\n      <th>HOME6<\/th>\n      <th>HOME7<\/th>\n      <th>HOME8<\/th>\n      <th>HOME9<\/th>\n      <th>KAMF1<\/th>\n      <th>KAMF2<\/th>\n      <th>KAMF3<\/th>\n      <th>KAMF4<\/th>\n      <th>KAMF5<\/th>\n      <th>KAMF6<\/th>\n      <th>KAMF7<\/th>\n      <th>networksize<\/th>\n      <th>nostalgia<\/th>\n      <th>onlineid<\/th>\n      <th>onlineid1<\/th>\n      <th>onlineid2<\/th>\n      <th>onlineid3<\/th>\n      <th>onlineid4<\/th>\n      <th>onlineid5<\/th>\n      <th>onlineid6<\/th>\n      <th>onlineid7<\/th>\n      <th>onlineid8<\/th>\n      <th>onlineid9<\/th>\n      <th>onlineid10<\/th>\n      <th>onlineide11<\/th>\n      <th>phone1<\/th>\n      <th>phone2<\/th>\n      <th>phone3<\/th>\n      <th>phone4<\/th>\n      <th>phone5<\/th>\n      <th>phone6<\/th>\n      <th>phone7<\/th>\n      <th>phone8<\/th>\n      <th>phone9<\/th>\n      <th>romantic<\/th>\n      <th>scontrol1<\/th>\n      <th>scontrol2<\/th>\n      <th>scontrol3<\/th>\n      <th>scontrol4<\/th>\n      <th>scontrol5<\/th>\n      <th>scontrol6<\/th>\n      <th>scontrol7<\/th>\n      <th>scontrol8<\/th>\n      <th>scontrol9<\/th>\n      <th>scontrol10<\/th>\n      <th>scontrol11<\/th>\n      <th>scontrol12<\/th>\n      <th>scontrol13<\/th>\n      <th>selfcontrol<\/th>\n      <th>sex<\/th>\n      <th>smoke<\/th>\n      <th>SNI1<\/th>\n      <th>SNI2<\/th>\n      <th>SNI3<\/th>\n      <th>SNI4<\/th>\n      <th>SNI5<\/th>\n      <th>SNI6<\/th>\n      <th>SNI7<\/th>\n      <th>SNI8<\/th>\n      <th>SNI9<\/th>\n      <th>SNI10<\/th>\n      <th>SNI11<\/th>\n      <th>SNI12<\/th>\n      <th>SNI13<\/th>\n      <th>SNI14<\/th>\n      <th>SNI15<\/th>\n      <th>SNI16<\/th>\n      <th>SNI17<\/th>\n      <th>SNI18<\/th>\n      <th>SNI19<\/th>\n      <th>SNI20<\/th>\n      <th>SNI21<\/th>\n      <th>SNI22<\/th>\n      <th>SNI23<\/th>\n      <th>SNI24<\/th>\n      <th>SNI25<\/th>\n      <th>SNI26<\/th>\n      <th>SNI27<\/th>\n      <th>SNI28<\/th>\n      <th>SNI29<\/th>\n      <th>SNI30<\/th>\n      <th>SNI31<\/th>\n      <th>SNI32<\/th>\n      <th>SNS1<\/th>\n      <th>SNS2<\/th>\n      <th>SNS3<\/th>\n      <th>SNS4<\/th>\n      <th>SNS5<\/th>\n      <th>SNS6<\/th>\n      <th>SNS7<\/th>\n      <th>socialdiversity<\/th>\n      <th>socialembedded<\/th>\n      <th>STRAQ_1<\/th>\n      <th>STRAQ_2<\/th>\n      <th>STRAQ_3<\/th>\n      <th>STRAQ_4<\/th>\n      <th>STRAQ_6<\/th>\n      <th>STRAQ_7<\/th>\n      <th>STRAQ_8<\/th>\n      <th>STRAQ_9<\/th>\n      <th>STRAQ_10<\/th>\n      <th>STRAQ_11<\/th>\n      <th>STRAQ_12<\/th>\n      <th>STRAQ_19<\/th>\n      <th>STRAQ_20<\/th>\n      <th>STRAQ_21<\/th>\n      <th>STRAQ_22<\/th>\n      <th>STRAQ_23<\/th>\n      <th>STRAQ_24<\/th>\n      <th>STRAQ_25<\/th>\n      <th>STRAQ_26<\/th>\n      <th>STRAQ_27<\/th>\n      <th>STRAQ_28<\/th>\n      <th>STRAQ_29<\/th>\n      <th>STRAQ_30<\/th>\n      <th>STRAQ_31<\/th>\n      <th>STRAQ_32<\/th>\n      <th>STRAQ_33<\/th>\n      <th>STRAQ_5<\/th>\n      <th>STRAQ_13<\/th>\n      <th>STRAQ_14<\/th>\n      <th>STRAQ_15<\/th>\n      <th>STRAQ_16<\/th>\n      <th>STRAQ_17<\/th>\n      <th>STRAQ_18<\/th>\n      <th>STRAQ_34<\/th>\n      <th>STRAQ_35<\/th>\n      <th>STRAQ_36<\/th>\n      <th>STRAQ_37<\/th>\n      <th>STRAQ_38<\/th>\n      <th>STRAQ_39<\/th>\n      <th>STRAQ_40<\/th>\n      <th>STRAQ_41<\/th>\n      <th>STRAQ_42<\/th>\n      <th>STRAQ_43<\/th>\n      <th>STRAQ_44<\/th>\n      <th>STRAQ_45<\/th>\n      <th>STRAQ_46<\/th>\n      <th>STRAQ_47<\/th>\n      <th>STRAQ_48<\/th>\n      <th>STRAQ_49<\/th>\n      <th>STRAQ_50<\/th>\n      <th>STRAQ_51<\/th>\n      <th>STRAQ_52<\/th>\n      <th>STRAQ_53<\/th>\n      <th>STRAQ_54<\/th>\n      <th>STRAQ_55<\/th>\n      <th>STRAQ_56<\/th>\n      <th>STRAQ_57<\/th>\n      <th>stress<\/th>\n      <th>stress1<\/th>\n      <th>stress2<\/th>\n      <th>stress3<\/th>\n      <th>stress4<\/th>\n      <th>stress5<\/th>\n      <th>stress6<\/th>\n      <th>stress7<\/th>\n      <th>stress8<\/th>\n      <th>stress9<\/th>\n      <th>stress10<\/th>\n      <th>stress11<\/th>\n      <th>stress12<\/th>\n      <th>stress13<\/th>\n      <th>stress14<\/th>\n      <th>Temperature_t1<\/th>\n      <th>Temperature_t2<\/th>\n      <th>thermotype<\/th>\n      <th>avgtemp<\/th>\n      <th>filter_.<\/th>\n      <th>mintemp<\/th>\n      <th>language<\/th>\n      <th>langfamily<\/th>\n      <th>Zanxiety<\/th>\n      <th>Zavoidance<\/th>\n      <th>Site<\/th>\n      <th>ALEX_SUM<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"pageLength":3,"columnDefs":[{"className":"dt-right","targets":[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,57,58,59,60,61,62,64,65,66,67,68,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86,87,88,89,90,91,92,93,94,95,96,97,98,99,100,101,102,103,104,105,106,107,108,109,110,111,112,113,114,115,116,117,118,119,120,121,122,123,124,125,126,127,128,129,130,131,132,133,134,135,136,137,138,139,140,141,142,143,144,145,151,152,153,154,155,156,157,158,159,160,161,162,163,164,165,166,167,168,169,170,171,172,173,174,175,176,177,178,179,180,181,182,183,184,185,186,187,188,189,190,191,192,193,194,195,196,197,198,199,200,201,202,203,204,205,206,207,208,209,210,211,212,213,214,215,216,217,218,219,220,221,222,223,224,225,226,227,228,229,230,231,232,233,234,235,236,237,238,239,240,241,242,243,244,245,246,248]},{"orderable":false,"targets":0}],"order":[],"autoWidth":false,"orderClasses":false,"lengthMenu":[3,10,25,50,100]}},"evals":[],"jsHooks":[]}</script>
```

需要注意的是，它是逐行运算，可以使用其他函数如rowSums，其中也用到了通配符。我们可以利用dplyr中的筛选功能，选取所有以＂ALEX＂为开头的列，并对这些列进行逐行求和，以得到真正反映Alex所有项目的总和。这种方法将给出16个项目的总和，而前面提到的方法只有4个项目的总和。


```r
# 对所有含有ALEX的列求和
df.clean.mutate_2 <- df.pg.raw %>% 
  dplyr::mutate(ALEX_SUM = rowSums(select(., starts_with("ALEX"))))
```

```{=html}
<div class="datatables html-widget html-fill-item-overflow-hidden html-fill-item" id="htmlwidget-d4d672bc986cde16171d" style="width:100%;height:auto;"></div>
<script type="application/json" data-for="htmlwidget-d4d672bc986cde16171d">{"x":{"filter":"none","vertical":false,"fillContainer":true,"data":[["1","2","3","4","5","6","7","8","9","10"],[1975,1995,1995,1988,1991,1995,1996,1973,1996,1996],[1,2,4,2,2,2,1,3,2,3],[1,2,1,3,1,3,1,3,3,3],[1,2,1,4,1,1,1,2,1,1],[2,2,2,5,5,3,1,3,3,2],[2,2,4,3,2,2,4,4,2,4],[1,2,1,4,2,1,1,4,2,1],[2,2,4,2,3,1,2,4,2,2],[2,2,1,4,4,2,1,4,2,4],[4,4,1,4,1,1,4,4,2,4],[1,2,2,3,1,2,2,5,2,1],[1,2,1,2,2,1,1,3,1,4],[2,2,1,2,1,2,2,2,2,1],[4,2,2,4,2,3,4,2,2,2],[3,2,1,2,4,4,3,3,4,3],[4,3,1,4,4,5,5,4,3,5],[2,2,1,3,2,4,5,1,2,4],[3.72222222222222,4.05555555555556,1.44444444444444,4.72222222222222,2.11111111111111,2,2.55555555555556,3.05555555555556,3.55555555555556,4.38888888888889],[6,0,0,2,0,0,null,0,0,0],[3.11111111111111,4.88888888888889,1.33333333333333,3.88888888888889,4.66666666666667,3.33333333333333,3.11111111111111,3.55555555555556,3.33333333333333,3.22222222222222],[3.44444444444444,2,2,3,1.77777777777778,2.44444444444444,3.77777777777778,3.33333333333333,1.88888888888889,4.22222222222222],[89,89,89,89,89,89,89,89,89,89],[3.27777777777778,3,1.61111111111111,3.94444444444444,4.94444444444444,3.77777777777778,6.44444444444444,3.22222222222222,4,5.27777777777778],[15,null,null,null,null,null,null,10,null,1],[1.63636363636364,2.18181818181818,2,3.27272727272727,2.18181818181818,1.72727272727273,1.72727272727273,3.54545454545455,2,2.63636363636364],[1,1,1,2,1,1,1,1,1,1],[5,6,1,3,1,1,2,3,4,5],[5,4,1,6,1,1,1,2,4,6],[3,2,1,6,1,1,4,3,4,6],[5,5,1,5,1,2,1,2,4,5],[5,5,1,4,1,2,4,3,4,4],[3,2,1,4,5,1,1,3,1,3],[1,3,1,3,2,1,2,3,4,5],[6,5,1,3,1,2,7,3,4,7],[5,2,1,4,7,6,1,3,4,3],[2,3,1,4,1,2,4,1,4,3],[6,7,1,6,6,2,2,5,2,6],[3,4,1,5,1,2,1,2,4,4],[3,3,1,6,2,4,2,5,4,4],[2,3,7,5,1,1,1,4,1,2],[3,4,1,5,2,2,1,5,4,1],[3,5,3,5,1,1,2,3,4,4],[6,5,1,6,2,3,6,3,4,7],[1,5,1,5,2,2,4,2,4,4],[2,3,2,5,1,5,7,3,4,5],[2,4,2,5,7,5,1,2,4,7],[5,3,2,5,2,4,6,4,4,5],[3,2,2,4,5,4,7,3,4,5],[3,2,2,3,2,3,7,3,4,6],[2,3,1,4,4,4,7,3,4,5],[5,3,1,6,2,3,7,4,4,6],[5,4,1,3,6,3,7,4,4,7],[3,5,1,3,7,1,7,4,4,7],[2,3,1,4,6,3,6,3,4,4],[4,2,2,5,6,6,6,3,4,4],[2,2,2,3,6,5,7,2,4,5],[3,4,1,4,7,4,7,2,4,3],[3,2,2,5,2,3,7,3,4,6],[4,3,3,3,7,4,6,3,4,5],[5,3,1,2,6,5,7,4,4,6],[4,3,1,3,6,3,7,6,4,5],[2,3,2,4,7,3,7,2,4,4],["9:23:38","9:23:26","8:57:08","8:55:14","8:54:35","8:39:12","8:32:08","8:28:57","8:28:03","8:25:57"],[3,2.2,1.2,3,2.6,3.6,3.8,2.4,2.6,3],[2,2,2,2,2,2,2,2,2,2],[0,0,12,0,5,0,null,1,0,2],[4,4,4,3,4,4,4,1,3,3],[4,5,1,5,5,4,5,3,3,4],[3,5,1,4,5,4,2,3,3,5],[4,5,4,5,5,4,4,2,4,3],[3,5,1,5,5,4,5,4,5,3],[3,5,1,4,4,4,2,3,4,4],[2,4,1,2,5,2,1,4,3,2],[4,5,1,2,4,2,5,5,2,2],[3,5,1,4,4,3,1,4,3,3],[2,5,1,4,5,3,3,4,3,3],[4,1,5,5,1,3,2,3,3,4],[4,2,6,4,1,3,2,3,3,5],[5,6,5,3,3,3,2,2,2,1],[4,2,4,1,2,2,2,2,1,3],[4,3,3,2,2,2,2,2,3,2],[1,2,1,2,2,4,2,3,3,3],[2,2,1,2,1,4,2,3,1,2],[17,28,24,19,29,33,53,17,17,19],[4.33333333333333,6.5,6.83333333333333,3.5,1.5,2,5.83333333333333,6.33333333333333,5.66666666666667,5.16666666666667],[2.27272727272727,2.63636363636364,2.18181818181818,2,1.90909090909091,2.45454545454545,2.81818181818182,2.54545454545455,1.81818181818182,3.90909090909091],[2,3,3,1,3,4,4,2,2,5],[2,2,4,1,2,4,4,3,3,4],[2,3,2,2,1,2,2,3,2,5],[4,3,4,3,4,4,2,2,2,4],[2,3,1,2,1,3,3,3,2,3],[2,2,1,2,1,1,2,2,1,3],[2,2,1,2,1,2,2,4,2,4],[2,3,3,1,1,1,2,3,2,3],[3,3,3,2,2,2,4,2,1,3],[2,3,1,4,2,1,2,1,1,4],[2,2,1,2,3,3,4,3,2,5],[4,2,1,2,2,3,3,3,3,4],[4,2,4,4,1,3,4,4,2,4],[2,2,1,3,1,2,5,3,1,2],[5,2,4,2,1,3,4,3,4,4],[2,2,1,3,1,2,1,2,3,5],[2,2,1,2,3,2,3,4,1,4],[4,2,1,3,2,2,5,3,1,5],[4,2,1,4,1,3,5,4,1,5],[4,2,4,4,4,2,4,4,1,5],[2,2,1,1,2,2,2,2,2,2],[2,4,2,4,3,5,2,1,3,3],[2,4,3,2,5,5,4,4,3,4],[2,4,3,1,5,5,2,5,5,5],[1,4,2,1,3,2,2,2,5,1],[1,4,4,2,5,3,1,2,5,2],[4,3,2,1,5,4,3,3,3,1],[2,4,4,2,5,3,2,3,4,2],[2,2,1,3,4,3,2,2,5,4],[5,4,3,4,4,5,4,4,3,3],[4,3,5,3,3,5,5,3,4,2],[3,4,4,2,4,5,3,2,2,5],[5,5,4,2,4,4,1,3,4,3],[4,4,4,2,4,3,2,2,2,2],[2.84615384615385,3.76923076923077,3.15384615384615,2.23076923076923,4.15384615384615,4,2.53846153846154,2.76923076923077,3.69230769230769,2.84615384615385],[1,1,1,2,1,1,2,1,1,2],[1,2,2,2,2,2,2,1,2,1],[2,1,1,1,2,2,2,4,2,2],[1,1,1,1,1,1,1,1,1,1],[null,null,null,null,null,null,null,null,null,null],[4,4,4,4,2,4,4,4,4,4],[4,4,4,4,2,4,4,2,4,4],[5,1,4,5,5,5,5,5,5,5],[null,null,4,null,null,null,null,null,null,null],[1,8,2,1,8,2,3,8,2,6],[null,6,2,null,5,1,2,3,2,3],[6,4,5,5,7,5,8,4,3,8],[3,1,5,1,3,5,8,4,2,8],[1,1,1,1,2,1,1,1,1,1],[null,null,null,null,8,null,null,null,null,null],[1,2,2,1,1,2,2,2,2,2],[null,8,8,null,null,8,8,8,8,8],[3,3,2,3,3,1,1,1,1,1],[2,8,1,8,1,null,null,null,null,null],[6,5,6,4,8,null,null,null,null,null],[1,3,1,1,1,8,6,4,1,3],[1,1,1,1,1,1,1,2,1,1],[null,null,null,null,null,null,null,2,null,null],[1,1,1,2,1,2,2,2,2,1],[" "," "," "," "," ","Rowing Club","University Friends","ODAA","Oxford Latin Speaking Society"," "],[" "," "," "," "," ","Rugby Club","Business Social"," ","Oxford Latin Conversation Society"," "],[" "," "," "," "," "," ","Lecturers/ Advisors"," ","Plato Reading Group"," "],[" "," "," "," "," "," ","Friends from home"," "," "," "],[" "," "," "," "," "," ","Neighbors"," "," "," "],[null,null,null,null,null,7,20,1,2,null],[null,null,null,null,null,7,15,null,4,null],[null,null,null,null,null,null,5,null,1,null],[null,null,null,null,null,null,9,null,null,null],[null,null,null,null,null,null,6,null,null,null],[6,6,7,2,1,1,6,6,7,7],[5,7,7,5,2,2,7,6,7,4],[5,6,7,3,2,3,5,7,6,5],[3,7,7,4,1,2,5,6,6,5],[2,6,7,5,2,2,6,7,5,5],[5,7,6,2,1,2,6,6,3,5],[4,6,6,4,1,2,5,6,3,4],[5,7,8,5,7,6,null,7,6,6],[1,2,3,1,2,4,null,2,2,3],[4,2,4,4,1,2,2,3,3,1],[4,4,5,2,1,4,4,2,5,4],[4,3,4,3,1,4,2,3,4,4],[5,3,4,3,4,4,4,4,2,4],[2,3,4,4,4,4,5,2,2,3],[2,4,4,3,1,3,4,3,1,3],[2,5,4,3,3,4,5,4,2,2],[2,2,4,3,4,3,5,3,2,4],[4,5,5,2,1,2,4,2,4,5],[2,3,2,2,1,4,1,4,4,5],[2,2,1,2,1,2,2,2,2,1],[1,2,2,2,4,4,4,2,2,2],[4,4,4,3,4,2,2,3,2,1],[5,4,2,3,2,4,3,4,4,4],[4,4,2,3,3,1,2,4,5,3],[2,3,4,4,4,5,5,4,4,4],[2,3,5,4,4,3,2,4,2,2],[4,3,4,2,4,3,4,3,3,2],[3,3,5,2,4,4,4,4,3,3],[2,3,1,3,5,4,1,3,2,4],[4,3,4,2,4,4,5,4,3,5],[3,3,5,4,4,3,2,2,3,3],[2,3,1,3,4,3,2,3,4,3],[3,3,1,4,1,2,2,2,3,3],[4,3,5,3,1,2,5,1,4,4],[3,3,3,2,1,3,3,2,3,3],[4,3,1,4,4,4,2,4,1,1],[4,5,4,2,1,3,1,4,4,5],[3,5,4,4,1,2,1,3,5,5],[3,2,2,4,4,2,5,2,2,4],[2,4,2,5,1,2,1,3,2,4],[4,3,1,2,4,2,5,1,1,4],[4,3,2,2,3,3,1,4,3,4],[1,4,3,4,4,2,1,3,2,2],[2,4,2,3,5,4,2,5,1,4],[2,4,2,4,5,4,4,4,2,4],[4,4,4,3,4,4,2,4,2,4],[3,4,2,4,4,4,4,5,5,1],[4,2,2,3,4,4,5,2,2,4],[5,3,3,2,4,2,2,2,2,4],[4,3,2,3,3,3,2,1,2,4],[4,3,5,2,4,4,4,4,4,4],[1,4,2,3,4,2,2,3,2,3],[1,2,2,4,4,2,1,2,1,2],[4,3,2,3,1,2,1,4,4,2],[4,4,4,2,1,4,1,4,3,3],[3,3,4,3,4,2,4,3,3,4],[1,2,5,4,4,2,1,2,1,2],[2,3,5,4,4,2,1,2,1,2],[4,3,5,4,4,4,3,3,3,4],[4,4,2,5,1,3,5,2,4,5],[2,4,5,3,1,2,5,4,2,4],[2,4,4,3,4,2,1,3,2,2],[4,3,4,3,5,2,4,4,2,4],[2,4,2,2,4,3,3,2,2,4],[2,4,1,2,4,2,2,2,2,2],[1,2,5,4,1,1,4,2,2,2],[3.07692307692308,2.46153846153846,2.38461538461538,2.84615384615385,2.53846153846154,2.07692307692308,1.53846153846154,3,2.61538461538462,3.15384615384615],[4,2,3,3,3,2,1,4,5,3],[2,3,3,3,4,2,2,3,2,4],[4,3,4,4,3,3,1,3,3,5],[4,3,1,2,2,2,2,3,2,4],[4,2,2,3,2,2,2,3,2,3],[3,2,1,2,1,2,2,3,2,1],[4,2,2,3,3,2,1,4,2,3],[3,2,2,2,2,2,1,4,3,4],[3,2,3,3,2,2,2,1,2,3],[2,3,3,3,3,2,1,3,4,3],[4,2,3,4,3,2,2,2,1,2],[3,4,4,3,5,4,4,4,5,4],[2,4,2,3,2,2,2,3,3,2],[1,2,2,2,3,2,1,3,3,4],[34.9,35.8,34,36.9,35.3,34.7,36,36.5,34.7,35.9],[35.9,36,36.2,37,36.1,36.4,36.3,36.6,36.3,36.3],[2,2,2,2,4,4,2,2,2,2],[35.4,35.9,35.1,36.95,35.7,35.55,36.15,36.55,35.5,36.1],[1,1,1,1,1,1,1,1,1,1],[-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667],[5,5,5,5,5,5,5,5,5,5],[1,1,1,1,1,1,1,1,1,1],[0.186319647739787,0.472682850177291,-1.77049556891649,1.0454092550523,-1.19776916404149,-1.29322356485399,-0.815951560791479,-0.386406757135223,0.0431380465210342,0.759046052614796],[-0.00190690952064737,-0.249805147204807,-1.48929633562561,0.593048860921336,1.48548251658431,0.44430991831084,2.82413300007877,-0.0514865570574791,0.642628508458168,1.7829604018053],["Oxford","Oxford","Oxford","Oxford","Oxford","Oxford","Oxford","Oxford","Oxford","Oxford"],[33,35,28,51,37,37,38,51,35,44]],"container":"<table class=\"display fill-container\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>age<\/th>\n      <th>ALEX1<\/th>\n      <th>ALEX2<\/th>\n      <th>ALEX3<\/th>\n      <th>ALEX4<\/th>\n      <th>ALEX5<\/th>\n      <th>ALEX6<\/th>\n      <th>ALEX7<\/th>\n      <th>ALEX8<\/th>\n      <th>ALEX9<\/th>\n      <th>ALEX10<\/th>\n      <th>ALEX11<\/th>\n      <th>ALEX12<\/th>\n      <th>ALEX13<\/th>\n      <th>ALEX14<\/th>\n      <th>ALEX15<\/th>\n      <th>ALEX16<\/th>\n      <th>anxiety<\/th>\n      <th>artgluctot<\/th>\n      <th>attachhome<\/th>\n      <th>attachphone<\/th>\n      <th>AvgHumidity<\/th>\n      <th>avoidance<\/th>\n      <th>cigs<\/th>\n      <th>DIDF<\/th>\n      <th>eatdrink<\/th>\n      <th>ECR1<\/th>\n      <th>ECR2<\/th>\n      <th>ECR3<\/th>\n      <th>ECR4<\/th>\n      <th>ECR5<\/th>\n      <th>ECR6<\/th>\n      <th>ECR7<\/th>\n      <th>ECR8<\/th>\n      <th>ECR9<\/th>\n      <th>ECR10<\/th>\n      <th>ECR11<\/th>\n      <th>ECR12<\/th>\n      <th>ECR13<\/th>\n      <th>ECR14<\/th>\n      <th>ECR15<\/th>\n      <th>ECR16<\/th>\n      <th>ECR17<\/th>\n      <th>ECR18<\/th>\n      <th>ECR19<\/th>\n      <th>ECR20<\/th>\n      <th>ECR21<\/th>\n      <th>ECR22<\/th>\n      <th>ECR23<\/th>\n      <th>ECR24<\/th>\n      <th>ECR25<\/th>\n      <th>ECR26<\/th>\n      <th>ECR27<\/th>\n      <th>ECR28<\/th>\n      <th>ECR29<\/th>\n      <th>ECR30<\/th>\n      <th>ECR31<\/th>\n      <th>ECR32<\/th>\n      <th>ECR33<\/th>\n      <th>ECR34<\/th>\n      <th>ECR35<\/th>\n      <th>ECR36<\/th>\n      <th>endtime<\/th>\n      <th>EOT<\/th>\n      <th>exercise<\/th>\n      <th>gluctot<\/th>\n      <th>health<\/th>\n      <th>HOME1<\/th>\n      <th>HOME2<\/th>\n      <th>HOME3<\/th>\n      <th>HOME4<\/th>\n      <th>HOME5<\/th>\n      <th>HOME6<\/th>\n      <th>HOME7<\/th>\n      <th>HOME8<\/th>\n      <th>HOME9<\/th>\n      <th>KAMF1<\/th>\n      <th>KAMF2<\/th>\n      <th>KAMF3<\/th>\n      <th>KAMF4<\/th>\n      <th>KAMF5<\/th>\n      <th>KAMF6<\/th>\n      <th>KAMF7<\/th>\n      <th>networksize<\/th>\n      <th>nostalgia<\/th>\n      <th>onlineid<\/th>\n      <th>onlineid1<\/th>\n      <th>onlineid2<\/th>\n      <th>onlineid3<\/th>\n      <th>onlineid4<\/th>\n      <th>onlineid5<\/th>\n      <th>onlineid6<\/th>\n      <th>onlineid7<\/th>\n      <th>onlineid8<\/th>\n      <th>onlineid9<\/th>\n      <th>onlineid10<\/th>\n      <th>onlineide11<\/th>\n      <th>phone1<\/th>\n      <th>phone2<\/th>\n      <th>phone3<\/th>\n      <th>phone4<\/th>\n      <th>phone5<\/th>\n      <th>phone6<\/th>\n      <th>phone7<\/th>\n      <th>phone8<\/th>\n      <th>phone9<\/th>\n      <th>romantic<\/th>\n      <th>scontrol1<\/th>\n      <th>scontrol2<\/th>\n      <th>scontrol3<\/th>\n      <th>scontrol4<\/th>\n      <th>scontrol5<\/th>\n      <th>scontrol6<\/th>\n      <th>scontrol7<\/th>\n      <th>scontrol8<\/th>\n      <th>scontrol9<\/th>\n      <th>scontrol10<\/th>\n      <th>scontrol11<\/th>\n      <th>scontrol12<\/th>\n      <th>scontrol13<\/th>\n      <th>selfcontrol<\/th>\n      <th>sex<\/th>\n      <th>smoke<\/th>\n      <th>SNI1<\/th>\n      <th>SNI2<\/th>\n      <th>SNI3<\/th>\n      <th>SNI4<\/th>\n      <th>SNI5<\/th>\n      <th>SNI6<\/th>\n      <th>SNI7<\/th>\n      <th>SNI8<\/th>\n      <th>SNI9<\/th>\n      <th>SNI10<\/th>\n      <th>SNI11<\/th>\n      <th>SNI12<\/th>\n      <th>SNI13<\/th>\n      <th>SNI14<\/th>\n      <th>SNI15<\/th>\n      <th>SNI16<\/th>\n      <th>SNI17<\/th>\n      <th>SNI18<\/th>\n      <th>SNI19<\/th>\n      <th>SNI20<\/th>\n      <th>SNI21<\/th>\n      <th>SNI22<\/th>\n      <th>SNI23<\/th>\n      <th>SNI24<\/th>\n      <th>SNI25<\/th>\n      <th>SNI26<\/th>\n      <th>SNI27<\/th>\n      <th>SNI28<\/th>\n      <th>SNI29<\/th>\n      <th>SNI30<\/th>\n      <th>SNI31<\/th>\n      <th>SNI32<\/th>\n      <th>SNS1<\/th>\n      <th>SNS2<\/th>\n      <th>SNS3<\/th>\n      <th>SNS4<\/th>\n      <th>SNS5<\/th>\n      <th>SNS6<\/th>\n      <th>SNS7<\/th>\n      <th>socialdiversity<\/th>\n      <th>socialembedded<\/th>\n      <th>STRAQ_1<\/th>\n      <th>STRAQ_2<\/th>\n      <th>STRAQ_3<\/th>\n      <th>STRAQ_4<\/th>\n      <th>STRAQ_6<\/th>\n      <th>STRAQ_7<\/th>\n      <th>STRAQ_8<\/th>\n      <th>STRAQ_9<\/th>\n      <th>STRAQ_10<\/th>\n      <th>STRAQ_11<\/th>\n      <th>STRAQ_12<\/th>\n      <th>STRAQ_19<\/th>\n      <th>STRAQ_20<\/th>\n      <th>STRAQ_21<\/th>\n      <th>STRAQ_22<\/th>\n      <th>STRAQ_23<\/th>\n      <th>STRAQ_24<\/th>\n      <th>STRAQ_25<\/th>\n      <th>STRAQ_26<\/th>\n      <th>STRAQ_27<\/th>\n      <th>STRAQ_28<\/th>\n      <th>STRAQ_29<\/th>\n      <th>STRAQ_30<\/th>\n      <th>STRAQ_31<\/th>\n      <th>STRAQ_32<\/th>\n      <th>STRAQ_33<\/th>\n      <th>STRAQ_5<\/th>\n      <th>STRAQ_13<\/th>\n      <th>STRAQ_14<\/th>\n      <th>STRAQ_15<\/th>\n      <th>STRAQ_16<\/th>\n      <th>STRAQ_17<\/th>\n      <th>STRAQ_18<\/th>\n      <th>STRAQ_34<\/th>\n      <th>STRAQ_35<\/th>\n      <th>STRAQ_36<\/th>\n      <th>STRAQ_37<\/th>\n      <th>STRAQ_38<\/th>\n      <th>STRAQ_39<\/th>\n      <th>STRAQ_40<\/th>\n      <th>STRAQ_41<\/th>\n      <th>STRAQ_42<\/th>\n      <th>STRAQ_43<\/th>\n      <th>STRAQ_44<\/th>\n      <th>STRAQ_45<\/th>\n      <th>STRAQ_46<\/th>\n      <th>STRAQ_47<\/th>\n      <th>STRAQ_48<\/th>\n      <th>STRAQ_49<\/th>\n      <th>STRAQ_50<\/th>\n      <th>STRAQ_51<\/th>\n      <th>STRAQ_52<\/th>\n      <th>STRAQ_53<\/th>\n      <th>STRAQ_54<\/th>\n      <th>STRAQ_55<\/th>\n      <th>STRAQ_56<\/th>\n      <th>STRAQ_57<\/th>\n      <th>stress<\/th>\n      <th>stress1<\/th>\n      <th>stress2<\/th>\n      <th>stress3<\/th>\n      <th>stress4<\/th>\n      <th>stress5<\/th>\n      <th>stress6<\/th>\n      <th>stress7<\/th>\n      <th>stress8<\/th>\n      <th>stress9<\/th>\n      <th>stress10<\/th>\n      <th>stress11<\/th>\n      <th>stress12<\/th>\n      <th>stress13<\/th>\n      <th>stress14<\/th>\n      <th>Temperature_t1<\/th>\n      <th>Temperature_t2<\/th>\n      <th>thermotype<\/th>\n      <th>avgtemp<\/th>\n      <th>filter_.<\/th>\n      <th>mintemp<\/th>\n      <th>language<\/th>\n      <th>langfamily<\/th>\n      <th>Zanxiety<\/th>\n      <th>Zavoidance<\/th>\n      <th>Site<\/th>\n      <th>ALEX_SUM<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"pageLength":3,"columnDefs":[{"className":"dt-right","targets":[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,57,58,59,60,61,62,64,65,66,67,68,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86,87,88,89,90,91,92,93,94,95,96,97,98,99,100,101,102,103,104,105,106,107,108,109,110,111,112,113,114,115,116,117,118,119,120,121,122,123,124,125,126,127,128,129,130,131,132,133,134,135,136,137,138,139,140,141,142,143,144,145,151,152,153,154,155,156,157,158,159,160,161,162,163,164,165,166,167,168,169,170,171,172,173,174,175,176,177,178,179,180,181,182,183,184,185,186,187,188,189,190,191,192,193,194,195,196,197,198,199,200,201,202,203,204,205,206,207,208,209,210,211,212,213,214,215,216,217,218,219,220,221,222,223,224,225,226,227,228,229,230,231,232,233,234,235,236,237,238,239,240,241,242,243,244,245,246,248]},{"orderable":false,"targets":0}],"order":[],"autoWidth":false,"orderClasses":false,"lengthMenu":[3,10,25,50,100]}},"evals":[],"jsHooks":[]}</script>
```

此外，我们还可以使用mutate函数对数据进行重新编码，例如根据出生年龄将其分成不同的年龄段。我们可以使用case_when函数生成一个新变量，该变量根据条件将列的内容变为不同的值。这种方法可以用于反向编码，例如将原来等于1的值变为5，将原来等于2的值变为4等。


```r
df.clean.mutate_3 <- df.pg.raw %>% 
  dplyr::mutate(decade = case_when(age <= 1969 ~ 60,
                                   age >= 1970 & age <= 1979 ~ 70,
                                   age >= 1980 & age <= 1989 ~ 80,
                                   age >= 1990 & age <= 1999 ~ 90,
                                   TRUE ~ NA_real_)  # 后面要跟一个TRUE，防止还存在没有穷尽的情况。表示其他所有上述范围的条件都是真的，它们被赋为NA
                ) %>% #当括号多的时候注意括号的位置 
  dplyr::select(.,decade, everything())
```




```{=html}
<div class="datatables html-widget html-fill-item-overflow-hidden html-fill-item" id="htmlwidget-05eb5b188cdf43849fe1" style="width:100%;height:auto;"></div>
<script type="application/json" data-for="htmlwidget-05eb5b188cdf43849fe1">{"x":{"filter":"none","vertical":false,"fillContainer":true,"data":[["1","2","3","4","5","6","7","8","9","10"],[70,90,90,80,90,90,90,70,90,90],[1975,1995,1995,1988,1991,1995,1996,1973,1996,1996],[1,2,4,2,2,2,1,3,2,3],[1,2,1,3,1,3,1,3,3,3],[1,2,1,4,1,1,1,2,1,1],[2,2,2,5,5,3,1,3,3,2],[2,2,4,3,2,2,4,4,2,4],[1,2,1,4,2,1,1,4,2,1],[2,2,4,2,3,1,2,4,2,2],[2,2,1,4,4,2,1,4,2,4],[4,4,1,4,1,1,4,4,2,4],[1,2,2,3,1,2,2,5,2,1],[1,2,1,2,2,1,1,3,1,4],[2,2,1,2,1,2,2,2,2,1],[4,2,2,4,2,3,4,2,2,2],[3,2,1,2,4,4,3,3,4,3],[4,3,1,4,4,5,5,4,3,5],[2,2,1,3,2,4,5,1,2,4],[3.72222222222222,4.05555555555556,1.44444444444444,4.72222222222222,2.11111111111111,2,2.55555555555556,3.05555555555556,3.55555555555556,4.38888888888889],[6,0,0,2,0,0,null,0,0,0],[3.11111111111111,4.88888888888889,1.33333333333333,3.88888888888889,4.66666666666667,3.33333333333333,3.11111111111111,3.55555555555556,3.33333333333333,3.22222222222222],[3.44444444444444,2,2,3,1.77777777777778,2.44444444444444,3.77777777777778,3.33333333333333,1.88888888888889,4.22222222222222],[89,89,89,89,89,89,89,89,89,89],[3.27777777777778,3,1.61111111111111,3.94444444444444,4.94444444444444,3.77777777777778,6.44444444444444,3.22222222222222,4,5.27777777777778],[15,null,null,null,null,null,null,10,null,1],[1.63636363636364,2.18181818181818,2,3.27272727272727,2.18181818181818,1.72727272727273,1.72727272727273,3.54545454545455,2,2.63636363636364],[1,1,1,2,1,1,1,1,1,1],[5,6,1,3,1,1,2,3,4,5],[5,4,1,6,1,1,1,2,4,6],[3,2,1,6,1,1,4,3,4,6],[5,5,1,5,1,2,1,2,4,5],[5,5,1,4,1,2,4,3,4,4],[3,2,1,4,5,1,1,3,1,3],[1,3,1,3,2,1,2,3,4,5],[6,5,1,3,1,2,7,3,4,7],[5,2,1,4,7,6,1,3,4,3],[2,3,1,4,1,2,4,1,4,3],[6,7,1,6,6,2,2,5,2,6],[3,4,1,5,1,2,1,2,4,4],[3,3,1,6,2,4,2,5,4,4],[2,3,7,5,1,1,1,4,1,2],[3,4,1,5,2,2,1,5,4,1],[3,5,3,5,1,1,2,3,4,4],[6,5,1,6,2,3,6,3,4,7],[1,5,1,5,2,2,4,2,4,4],[2,3,2,5,1,5,7,3,4,5],[2,4,2,5,7,5,1,2,4,7],[5,3,2,5,2,4,6,4,4,5],[3,2,2,4,5,4,7,3,4,5],[3,2,2,3,2,3,7,3,4,6],[2,3,1,4,4,4,7,3,4,5],[5,3,1,6,2,3,7,4,4,6],[5,4,1,3,6,3,7,4,4,7],[3,5,1,3,7,1,7,4,4,7],[2,3,1,4,6,3,6,3,4,4],[4,2,2,5,6,6,6,3,4,4],[2,2,2,3,6,5,7,2,4,5],[3,4,1,4,7,4,7,2,4,3],[3,2,2,5,2,3,7,3,4,6],[4,3,3,3,7,4,6,3,4,5],[5,3,1,2,6,5,7,4,4,6],[4,3,1,3,6,3,7,6,4,5],[2,3,2,4,7,3,7,2,4,4],["9:23:38","9:23:26","8:57:08","8:55:14","8:54:35","8:39:12","8:32:08","8:28:57","8:28:03","8:25:57"],[3,2.2,1.2,3,2.6,3.6,3.8,2.4,2.6,3],[2,2,2,2,2,2,2,2,2,2],[0,0,12,0,5,0,null,1,0,2],[4,4,4,3,4,4,4,1,3,3],[4,5,1,5,5,4,5,3,3,4],[3,5,1,4,5,4,2,3,3,5],[4,5,4,5,5,4,4,2,4,3],[3,5,1,5,5,4,5,4,5,3],[3,5,1,4,4,4,2,3,4,4],[2,4,1,2,5,2,1,4,3,2],[4,5,1,2,4,2,5,5,2,2],[3,5,1,4,4,3,1,4,3,3],[2,5,1,4,5,3,3,4,3,3],[4,1,5,5,1,3,2,3,3,4],[4,2,6,4,1,3,2,3,3,5],[5,6,5,3,3,3,2,2,2,1],[4,2,4,1,2,2,2,2,1,3],[4,3,3,2,2,2,2,2,3,2],[1,2,1,2,2,4,2,3,3,3],[2,2,1,2,1,4,2,3,1,2],[17,28,24,19,29,33,53,17,17,19],[4.33333333333333,6.5,6.83333333333333,3.5,1.5,2,5.83333333333333,6.33333333333333,5.66666666666667,5.16666666666667],[2.27272727272727,2.63636363636364,2.18181818181818,2,1.90909090909091,2.45454545454545,2.81818181818182,2.54545454545455,1.81818181818182,3.90909090909091],[2,3,3,1,3,4,4,2,2,5],[2,2,4,1,2,4,4,3,3,4],[2,3,2,2,1,2,2,3,2,5],[4,3,4,3,4,4,2,2,2,4],[2,3,1,2,1,3,3,3,2,3],[2,2,1,2,1,1,2,2,1,3],[2,2,1,2,1,2,2,4,2,4],[2,3,3,1,1,1,2,3,2,3],[3,3,3,2,2,2,4,2,1,3],[2,3,1,4,2,1,2,1,1,4],[2,2,1,2,3,3,4,3,2,5],[4,2,1,2,2,3,3,3,3,4],[4,2,4,4,1,3,4,4,2,4],[2,2,1,3,1,2,5,3,1,2],[5,2,4,2,1,3,4,3,4,4],[2,2,1,3,1,2,1,2,3,5],[2,2,1,2,3,2,3,4,1,4],[4,2,1,3,2,2,5,3,1,5],[4,2,1,4,1,3,5,4,1,5],[4,2,4,4,4,2,4,4,1,5],[2,2,1,1,2,2,2,2,2,2],[2,4,2,4,3,5,2,1,3,3],[2,4,3,2,5,5,4,4,3,4],[2,4,3,1,5,5,2,5,5,5],[1,4,2,1,3,2,2,2,5,1],[1,4,4,2,5,3,1,2,5,2],[4,3,2,1,5,4,3,3,3,1],[2,4,4,2,5,3,2,3,4,2],[2,2,1,3,4,3,2,2,5,4],[5,4,3,4,4,5,4,4,3,3],[4,3,5,3,3,5,5,3,4,2],[3,4,4,2,4,5,3,2,2,5],[5,5,4,2,4,4,1,3,4,3],[4,4,4,2,4,3,2,2,2,2],[2.84615384615385,3.76923076923077,3.15384615384615,2.23076923076923,4.15384615384615,4,2.53846153846154,2.76923076923077,3.69230769230769,2.84615384615385],[1,1,1,2,1,1,2,1,1,2],[1,2,2,2,2,2,2,1,2,1],[2,1,1,1,2,2,2,4,2,2],[1,1,1,1,1,1,1,1,1,1],[null,null,null,null,null,null,null,null,null,null],[4,4,4,4,2,4,4,4,4,4],[4,4,4,4,2,4,4,2,4,4],[5,1,4,5,5,5,5,5,5,5],[null,null,4,null,null,null,null,null,null,null],[1,8,2,1,8,2,3,8,2,6],[null,6,2,null,5,1,2,3,2,3],[6,4,5,5,7,5,8,4,3,8],[3,1,5,1,3,5,8,4,2,8],[1,1,1,1,2,1,1,1,1,1],[null,null,null,null,8,null,null,null,null,null],[1,2,2,1,1,2,2,2,2,2],[null,8,8,null,null,8,8,8,8,8],[3,3,2,3,3,1,1,1,1,1],[2,8,1,8,1,null,null,null,null,null],[6,5,6,4,8,null,null,null,null,null],[1,3,1,1,1,8,6,4,1,3],[1,1,1,1,1,1,1,2,1,1],[null,null,null,null,null,null,null,2,null,null],[1,1,1,2,1,2,2,2,2,1],[" "," "," "," "," ","Rowing Club","University Friends","ODAA","Oxford Latin Speaking Society"," "],[" "," "," "," "," ","Rugby Club","Business Social"," ","Oxford Latin Conversation Society"," "],[" "," "," "," "," "," ","Lecturers/ Advisors"," ","Plato Reading Group"," "],[" "," "," "," "," "," ","Friends from home"," "," "," "],[" "," "," "," "," "," ","Neighbors"," "," "," "],[null,null,null,null,null,7,20,1,2,null],[null,null,null,null,null,7,15,null,4,null],[null,null,null,null,null,null,5,null,1,null],[null,null,null,null,null,null,9,null,null,null],[null,null,null,null,null,null,6,null,null,null],[6,6,7,2,1,1,6,6,7,7],[5,7,7,5,2,2,7,6,7,4],[5,6,7,3,2,3,5,7,6,5],[3,7,7,4,1,2,5,6,6,5],[2,6,7,5,2,2,6,7,5,5],[5,7,6,2,1,2,6,6,3,5],[4,6,6,4,1,2,5,6,3,4],[5,7,8,5,7,6,null,7,6,6],[1,2,3,1,2,4,null,2,2,3],[4,2,4,4,1,2,2,3,3,1],[4,4,5,2,1,4,4,2,5,4],[4,3,4,3,1,4,2,3,4,4],[5,3,4,3,4,4,4,4,2,4],[2,3,4,4,4,4,5,2,2,3],[2,4,4,3,1,3,4,3,1,3],[2,5,4,3,3,4,5,4,2,2],[2,2,4,3,4,3,5,3,2,4],[4,5,5,2,1,2,4,2,4,5],[2,3,2,2,1,4,1,4,4,5],[2,2,1,2,1,2,2,2,2,1],[1,2,2,2,4,4,4,2,2,2],[4,4,4,3,4,2,2,3,2,1],[5,4,2,3,2,4,3,4,4,4],[4,4,2,3,3,1,2,4,5,3],[2,3,4,4,4,5,5,4,4,4],[2,3,5,4,4,3,2,4,2,2],[4,3,4,2,4,3,4,3,3,2],[3,3,5,2,4,4,4,4,3,3],[2,3,1,3,5,4,1,3,2,4],[4,3,4,2,4,4,5,4,3,5],[3,3,5,4,4,3,2,2,3,3],[2,3,1,3,4,3,2,3,4,3],[3,3,1,4,1,2,2,2,3,3],[4,3,5,3,1,2,5,1,4,4],[3,3,3,2,1,3,3,2,3,3],[4,3,1,4,4,4,2,4,1,1],[4,5,4,2,1,3,1,4,4,5],[3,5,4,4,1,2,1,3,5,5],[3,2,2,4,4,2,5,2,2,4],[2,4,2,5,1,2,1,3,2,4],[4,3,1,2,4,2,5,1,1,4],[4,3,2,2,3,3,1,4,3,4],[1,4,3,4,4,2,1,3,2,2],[2,4,2,3,5,4,2,5,1,4],[2,4,2,4,5,4,4,4,2,4],[4,4,4,3,4,4,2,4,2,4],[3,4,2,4,4,4,4,5,5,1],[4,2,2,3,4,4,5,2,2,4],[5,3,3,2,4,2,2,2,2,4],[4,3,2,3,3,3,2,1,2,4],[4,3,5,2,4,4,4,4,4,4],[1,4,2,3,4,2,2,3,2,3],[1,2,2,4,4,2,1,2,1,2],[4,3,2,3,1,2,1,4,4,2],[4,4,4,2,1,4,1,4,3,3],[3,3,4,3,4,2,4,3,3,4],[1,2,5,4,4,2,1,2,1,2],[2,3,5,4,4,2,1,2,1,2],[4,3,5,4,4,4,3,3,3,4],[4,4,2,5,1,3,5,2,4,5],[2,4,5,3,1,2,5,4,2,4],[2,4,4,3,4,2,1,3,2,2],[4,3,4,3,5,2,4,4,2,4],[2,4,2,2,4,3,3,2,2,4],[2,4,1,2,4,2,2,2,2,2],[1,2,5,4,1,1,4,2,2,2],[3.07692307692308,2.46153846153846,2.38461538461538,2.84615384615385,2.53846153846154,2.07692307692308,1.53846153846154,3,2.61538461538462,3.15384615384615],[4,2,3,3,3,2,1,4,5,3],[2,3,3,3,4,2,2,3,2,4],[4,3,4,4,3,3,1,3,3,5],[4,3,1,2,2,2,2,3,2,4],[4,2,2,3,2,2,2,3,2,3],[3,2,1,2,1,2,2,3,2,1],[4,2,2,3,3,2,1,4,2,3],[3,2,2,2,2,2,1,4,3,4],[3,2,3,3,2,2,2,1,2,3],[2,3,3,3,3,2,1,3,4,3],[4,2,3,4,3,2,2,2,1,2],[3,4,4,3,5,4,4,4,5,4],[2,4,2,3,2,2,2,3,3,2],[1,2,2,2,3,2,1,3,3,4],[34.9,35.8,34,36.9,35.3,34.7,36,36.5,34.7,35.9],[35.9,36,36.2,37,36.1,36.4,36.3,36.6,36.3,36.3],[2,2,2,2,4,4,2,2,2,2],[35.4,35.9,35.1,36.95,35.7,35.55,36.15,36.55,35.5,36.1],[1,1,1,1,1,1,1,1,1,1],[-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667,-1.66666666666667],[5,5,5,5,5,5,5,5,5,5],[1,1,1,1,1,1,1,1,1,1],[0.186319647739787,0.472682850177291,-1.77049556891649,1.0454092550523,-1.19776916404149,-1.29322356485399,-0.815951560791479,-0.386406757135223,0.0431380465210342,0.759046052614796],[-0.00190690952064737,-0.249805147204807,-1.48929633562561,0.593048860921336,1.48548251658431,0.44430991831084,2.82413300007877,-0.0514865570574791,0.642628508458168,1.7829604018053],["Oxford","Oxford","Oxford","Oxford","Oxford","Oxford","Oxford","Oxford","Oxford","Oxford"]],"container":"<table class=\"display fill-container\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>decade<\/th>\n      <th>age<\/th>\n      <th>ALEX1<\/th>\n      <th>ALEX2<\/th>\n      <th>ALEX3<\/th>\n      <th>ALEX4<\/th>\n      <th>ALEX5<\/th>\n      <th>ALEX6<\/th>\n      <th>ALEX7<\/th>\n      <th>ALEX8<\/th>\n      <th>ALEX9<\/th>\n      <th>ALEX10<\/th>\n      <th>ALEX11<\/th>\n      <th>ALEX12<\/th>\n      <th>ALEX13<\/th>\n      <th>ALEX14<\/th>\n      <th>ALEX15<\/th>\n      <th>ALEX16<\/th>\n      <th>anxiety<\/th>\n      <th>artgluctot<\/th>\n      <th>attachhome<\/th>\n      <th>attachphone<\/th>\n      <th>AvgHumidity<\/th>\n      <th>avoidance<\/th>\n      <th>cigs<\/th>\n      <th>DIDF<\/th>\n      <th>eatdrink<\/th>\n      <th>ECR1<\/th>\n      <th>ECR2<\/th>\n      <th>ECR3<\/th>\n      <th>ECR4<\/th>\n      <th>ECR5<\/th>\n      <th>ECR6<\/th>\n      <th>ECR7<\/th>\n      <th>ECR8<\/th>\n      <th>ECR9<\/th>\n      <th>ECR10<\/th>\n      <th>ECR11<\/th>\n      <th>ECR12<\/th>\n      <th>ECR13<\/th>\n      <th>ECR14<\/th>\n      <th>ECR15<\/th>\n      <th>ECR16<\/th>\n      <th>ECR17<\/th>\n      <th>ECR18<\/th>\n      <th>ECR19<\/th>\n      <th>ECR20<\/th>\n      <th>ECR21<\/th>\n      <th>ECR22<\/th>\n      <th>ECR23<\/th>\n      <th>ECR24<\/th>\n      <th>ECR25<\/th>\n      <th>ECR26<\/th>\n      <th>ECR27<\/th>\n      <th>ECR28<\/th>\n      <th>ECR29<\/th>\n      <th>ECR30<\/th>\n      <th>ECR31<\/th>\n      <th>ECR32<\/th>\n      <th>ECR33<\/th>\n      <th>ECR34<\/th>\n      <th>ECR35<\/th>\n      <th>ECR36<\/th>\n      <th>endtime<\/th>\n      <th>EOT<\/th>\n      <th>exercise<\/th>\n      <th>gluctot<\/th>\n      <th>health<\/th>\n      <th>HOME1<\/th>\n      <th>HOME2<\/th>\n      <th>HOME3<\/th>\n      <th>HOME4<\/th>\n      <th>HOME5<\/th>\n      <th>HOME6<\/th>\n      <th>HOME7<\/th>\n      <th>HOME8<\/th>\n      <th>HOME9<\/th>\n      <th>KAMF1<\/th>\n      <th>KAMF2<\/th>\n      <th>KAMF3<\/th>\n      <th>KAMF4<\/th>\n      <th>KAMF5<\/th>\n      <th>KAMF6<\/th>\n      <th>KAMF7<\/th>\n      <th>networksize<\/th>\n      <th>nostalgia<\/th>\n      <th>onlineid<\/th>\n      <th>onlineid1<\/th>\n      <th>onlineid2<\/th>\n      <th>onlineid3<\/th>\n      <th>onlineid4<\/th>\n      <th>onlineid5<\/th>\n      <th>onlineid6<\/th>\n      <th>onlineid7<\/th>\n      <th>onlineid8<\/th>\n      <th>onlineid9<\/th>\n      <th>onlineid10<\/th>\n      <th>onlineide11<\/th>\n      <th>phone1<\/th>\n      <th>phone2<\/th>\n      <th>phone3<\/th>\n      <th>phone4<\/th>\n      <th>phone5<\/th>\n      <th>phone6<\/th>\n      <th>phone7<\/th>\n      <th>phone8<\/th>\n      <th>phone9<\/th>\n      <th>romantic<\/th>\n      <th>scontrol1<\/th>\n      <th>scontrol2<\/th>\n      <th>scontrol3<\/th>\n      <th>scontrol4<\/th>\n      <th>scontrol5<\/th>\n      <th>scontrol6<\/th>\n      <th>scontrol7<\/th>\n      <th>scontrol8<\/th>\n      <th>scontrol9<\/th>\n      <th>scontrol10<\/th>\n      <th>scontrol11<\/th>\n      <th>scontrol12<\/th>\n      <th>scontrol13<\/th>\n      <th>selfcontrol<\/th>\n      <th>sex<\/th>\n      <th>smoke<\/th>\n      <th>SNI1<\/th>\n      <th>SNI2<\/th>\n      <th>SNI3<\/th>\n      <th>SNI4<\/th>\n      <th>SNI5<\/th>\n      <th>SNI6<\/th>\n      <th>SNI7<\/th>\n      <th>SNI8<\/th>\n      <th>SNI9<\/th>\n      <th>SNI10<\/th>\n      <th>SNI11<\/th>\n      <th>SNI12<\/th>\n      <th>SNI13<\/th>\n      <th>SNI14<\/th>\n      <th>SNI15<\/th>\n      <th>SNI16<\/th>\n      <th>SNI17<\/th>\n      <th>SNI18<\/th>\n      <th>SNI19<\/th>\n      <th>SNI20<\/th>\n      <th>SNI21<\/th>\n      <th>SNI22<\/th>\n      <th>SNI23<\/th>\n      <th>SNI24<\/th>\n      <th>SNI25<\/th>\n      <th>SNI26<\/th>\n      <th>SNI27<\/th>\n      <th>SNI28<\/th>\n      <th>SNI29<\/th>\n      <th>SNI30<\/th>\n      <th>SNI31<\/th>\n      <th>SNI32<\/th>\n      <th>SNS1<\/th>\n      <th>SNS2<\/th>\n      <th>SNS3<\/th>\n      <th>SNS4<\/th>\n      <th>SNS5<\/th>\n      <th>SNS6<\/th>\n      <th>SNS7<\/th>\n      <th>socialdiversity<\/th>\n      <th>socialembedded<\/th>\n      <th>STRAQ_1<\/th>\n      <th>STRAQ_2<\/th>\n      <th>STRAQ_3<\/th>\n      <th>STRAQ_4<\/th>\n      <th>STRAQ_6<\/th>\n      <th>STRAQ_7<\/th>\n      <th>STRAQ_8<\/th>\n      <th>STRAQ_9<\/th>\n      <th>STRAQ_10<\/th>\n      <th>STRAQ_11<\/th>\n      <th>STRAQ_12<\/th>\n      <th>STRAQ_19<\/th>\n      <th>STRAQ_20<\/th>\n      <th>STRAQ_21<\/th>\n      <th>STRAQ_22<\/th>\n      <th>STRAQ_23<\/th>\n      <th>STRAQ_24<\/th>\n      <th>STRAQ_25<\/th>\n      <th>STRAQ_26<\/th>\n      <th>STRAQ_27<\/th>\n      <th>STRAQ_28<\/th>\n      <th>STRAQ_29<\/th>\n      <th>STRAQ_30<\/th>\n      <th>STRAQ_31<\/th>\n      <th>STRAQ_32<\/th>\n      <th>STRAQ_33<\/th>\n      <th>STRAQ_5<\/th>\n      <th>STRAQ_13<\/th>\n      <th>STRAQ_14<\/th>\n      <th>STRAQ_15<\/th>\n      <th>STRAQ_16<\/th>\n      <th>STRAQ_17<\/th>\n      <th>STRAQ_18<\/th>\n      <th>STRAQ_34<\/th>\n      <th>STRAQ_35<\/th>\n      <th>STRAQ_36<\/th>\n      <th>STRAQ_37<\/th>\n      <th>STRAQ_38<\/th>\n      <th>STRAQ_39<\/th>\n      <th>STRAQ_40<\/th>\n      <th>STRAQ_41<\/th>\n      <th>STRAQ_42<\/th>\n      <th>STRAQ_43<\/th>\n      <th>STRAQ_44<\/th>\n      <th>STRAQ_45<\/th>\n      <th>STRAQ_46<\/th>\n      <th>STRAQ_47<\/th>\n      <th>STRAQ_48<\/th>\n      <th>STRAQ_49<\/th>\n      <th>STRAQ_50<\/th>\n      <th>STRAQ_51<\/th>\n      <th>STRAQ_52<\/th>\n      <th>STRAQ_53<\/th>\n      <th>STRAQ_54<\/th>\n      <th>STRAQ_55<\/th>\n      <th>STRAQ_56<\/th>\n      <th>STRAQ_57<\/th>\n      <th>stress<\/th>\n      <th>stress1<\/th>\n      <th>stress2<\/th>\n      <th>stress3<\/th>\n      <th>stress4<\/th>\n      <th>stress5<\/th>\n      <th>stress6<\/th>\n      <th>stress7<\/th>\n      <th>stress8<\/th>\n      <th>stress9<\/th>\n      <th>stress10<\/th>\n      <th>stress11<\/th>\n      <th>stress12<\/th>\n      <th>stress13<\/th>\n      <th>stress14<\/th>\n      <th>Temperature_t1<\/th>\n      <th>Temperature_t2<\/th>\n      <th>thermotype<\/th>\n      <th>avgtemp<\/th>\n      <th>filter_.<\/th>\n      <th>mintemp<\/th>\n      <th>language<\/th>\n      <th>langfamily<\/th>\n      <th>Zanxiety<\/th>\n      <th>Zavoidance<\/th>\n      <th>Site<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"pageLength":3,"columnDefs":[{"className":"dt-right","targets":[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,57,58,59,60,61,62,63,65,66,67,68,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,84,85,86,87,88,89,90,91,92,93,94,95,96,97,98,99,100,101,102,103,104,105,106,107,108,109,110,111,112,113,114,115,116,117,118,119,120,121,122,123,124,125,126,127,128,129,130,131,132,133,134,135,136,137,138,139,140,141,142,143,144,145,146,152,153,154,155,156,157,158,159,160,161,162,163,164,165,166,167,168,169,170,171,172,173,174,175,176,177,178,179,180,181,182,183,184,185,186,187,188,189,190,191,192,193,194,195,196,197,198,199,200,201,202,203,204,205,206,207,208,209,210,211,212,213,214,215,216,217,218,219,220,221,222,223,224,225,226,227,228,229,230,231,232,233,234,235,236,237,238,239,240,241,242,243,244,245,246,247]},{"orderable":false,"targets":0}],"order":[],"autoWidth":false,"orderClasses":false,"lengthMenu":[3,10,25,50,100]}},"evals":[],"jsHooks":[]}</script>
```

我们可以先按照出生年代将数据进行分组，然后使用group_by函数重新分组。我们还可以按照多个条件进行分组，例如按照年代和性别进行分组。分组完成后，我们可以使用summarise函数对每个组内部进行操作，例如求均值和标准差等。最后，我们可以使用ungroup函数将数据重新拆分，以便进行后续的运算。


```r
df.clean.group_by <- df.clean.mutate_3 %>%
  dplyr::group_by(.,decade) %>% # 根据被试的出生年代，将数据拆分
  dplyr::summarise(mean_avoidance = mean(avoidance)) %>% # 计算不同年代下被试的平均avoidance
  dplyr::ungroup() #group之后一定要ungroup，否则分组的标签会一直在数据框上
```

```{=html}
<div class="datatables html-widget html-fill-item-overflow-hidden html-fill-item" id="htmlwidget-af52ea513f571e7be1cf" style="width:100%;height:auto;"></div>
<script type="application/json" data-for="htmlwidget-af52ea513f571e7be1cf">{"x":{"filter":"none","vertical":false,"fillContainer":true,"data":[["1","2","3","4"],[60,70,80,90],[3.13559322033898,2.89696992243867,2.91210368205253,3.15488581563949]],"container":"<table class=\"display fill-container\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>decade<\/th>\n      <th>mean_avoidance<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"pageLength":4,"columnDefs":[{"className":"dt-right","targets":[1,2]},{"orderable":false,"targets":0}],"order":[],"autoWidth":false,"orderClasses":false,"lengthMenu":[4,10,25,50,100]}},"evals":[],"jsHooks":[]}</script>
```

我们可以将所有学到的函数串起来
1.先使用filter函数选择eatdrink为1的被试
2.使用select函数选择所需变量
3.使用mutate函数对出生年份进行重新编码
4.使用group_by和summarise函数求出按照年代求ALEX的均值
5.使用rowSums函数对ALEX求均值。


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

在这里，我们可以按照多个条件进行分组，但我们只使用了一个条件，因此可以写得很简洁。我们使用summarise函数计算ALEX总分的平均值，然后取消分组。我们可以得到每个十年在ALEX得分上的平均值。

在这个过程中，我们需要理解管道的操作方式，因为它可以处理我们想要对某个变量的所有领域进行的所有操作。每个函数都可以单独使用，但将它们放入管道中可以省略一些步骤。

在Tidyverse中，有一些常用的实用函数，比如separate、extract、unite和pivot。
separate可以按照特定规则将一个变量分割成为几列。它更适合用于按固定分隔符分割字符串，如将日期“2022-02-25”按照年月日分成“2022”、“02”和“25”三列
extract可以提取一个或多个特定的字符串。它更适合用于从字符串中提取特定的信息，如将“John Smith”分成“John”和“Smith”两列
unite则是将多个列合并成为一列。
pivot则可以将宽格式数据转换为长格式数据，或者将长格式数据转换为宽格式数据。

**长数据和宽数据**

那么，什么是长格式数据和宽格式数据呢？
我们通常使用长格式数据来记录实验数据，比如eprime产生的数据。长格式数据的形式可能是这样的：每个实验对象有一个ID，然后有不同的条件和变量，比如年龄和性别。每个实验对象可能有多个条件，因此我们需要以行来记录数据。每一行代表一个**观察值**。
例如，如果我们有一个问卷调查，其中有五个问题，那么我们将每个问题的回答记录在一列中，每一行代表受访者的一个回答值。
相反，宽格式数据是指我们将变量记录在多列中，每一行代表一个观察值。例如，如果我们有一个实验，其中有五个条件，那么我们将每个条件的结果记录在一列中，每一行代表一个受试者。

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

## 函数

### 函数参数
学到这里，为了让大家更好的理解函数的意义和作用，在这里重新讲解一下一些常用的函数。

read.csv()的功能是将数据读取，成为R里面能够操纵的一个变量。read.csv就是我们的函数名，括号里面放置的是我们要输入的参数argument。
第一个argument是一个文件的路径，第二个是header，表示是否使用第一行作为我们的column names，第三个是sep，表示我们指定的分隔符是什么。还有第四个，force，表示是否把我们读到的文件里面的字符串作为factor。每一个函数基本上都包含两个部分，即**函数名和arguments**。

```{-}
function1(argument1 = 123,argument2 = "helloworld",argument3 = list1) 
#第一种输入方法：同时输入argument和value

function2(123,"helloworld",list1) 
function2("helloworld",list1) 
#二一种输入方法：也可以省略argument,顺序读取value
function2("helloworld",123,list1) 
```

我们有几种输入argument的方法，第一种是完整的输入，即function name和argument都要输入。第二个方法就是省略arguement name，直接输入数值，如果我们只给出了值，而没有明确指定参数，那么函数会按照顺序把值赋给参数。如果我们只给出了一个值，那么函数会默认把它赋给第一个参数，而其他参数则会有默认值。
对于read.csv()函数，除了第一个argument（文件名）以外，其余argument都有默认值。例如，header默认为false，separate默认为空格，quote为一个特殊符号表示引号。另外，stringsAsFactors也是一个默认值。
因此，我们只需要输入文件名这一个argument即可。如果不输入其他argument，它会使用默认值。这是R的一个重要特点，它兼具灵活性和便捷性。如果我们要完整写出来，我们需要写出文件名这个argument的名字，即file。我们可以输入相对路径或绝对路径。其他argument的默认值可以通过？或者help查看。

### group_by()

group_by有两个作用：第一个是分组，然后将其拆分为不同的小包。在对分组后的数据进行分析时，它将以组为单位进行操作，对每个组的结果分别计算，然后组合起来。如果我们在没有任何操作的情况下添加groupby，它会如何改变我们的数据框呢？


```r
# 读取原始数据
df.mt.raw <-  read.csv('./data/match/match_raw.csv',
                       header = T, sep=",", stringsAsFactors = FALSE) 
library("tidyverse")
df.mt.raw
```

```
##                      Date Prac  Sub Age    Sex Hand Block Bin Trial
## 1    02-May-2018_14:23:06  Exp 7302  22 female    R     1   1     1
## 2    02-May-2018_14:23:08  Exp 7302  22 female    R     1   1     2
## 3    02-May-2018_14:23:10  Exp 7302  22 female    R     1   1     3
## 4    02-May-2018_14:23:13  Exp 7302  22 female    R     1   1     4
## 5    02-May-2018_14:23:15  Exp 7302  22 female    R     1   1     5
## 6    02-May-2018_14:23:17  Exp 7302  22 female    R     1   1     6
## 7    02-May-2018_14:23:19  Exp 7302  22 female    R     1   1     7
## 8    02-May-2018_14:23:21  Exp 7302  22 female    R     1   1     8
## 9    02-May-2018_14:23:24  Exp 7302  22 female    R     1   1     9
## 10   02-May-2018_14:23:26  Exp 7302  22 female    R     1   1    10
## 11   02-May-2018_14:23:28  Exp 7302  22 female    R     1   1    11
## 12   02-May-2018_14:23:30  Exp 7302  22 female    R     1   1    12
## 13   02-May-2018_14:23:32  Exp 7302  22 female    R     1   1    13
## 14   02-May-2018_14:23:34  Exp 7302  22 female    R     1   1    14
## 15   02-May-2018_14:23:36  Exp 7302  22 female    R     1   1    15
## 16   02-May-2018_14:23:38  Exp 7302  22 female    R     1   1    16
## 17   02-May-2018_14:23:40  Exp 7302  22 female    R     1   1    17
## 18   02-May-2018_14:23:42  Exp 7302  22 female    R     1   1    18
## 19   02-May-2018_14:23:45  Exp 7302  22 female    R     1   1    19
## 20   02-May-2018_14:23:47  Exp 7302  22 female    R     1   1    20
## 21   02-May-2018_14:23:49  Exp 7302  22 female    R     1   1    21
## 22   02-May-2018_14:23:51  Exp 7302  22 female    R     1   1    22
## 23   02-May-2018_14:23:53  Exp 7302  22 female    R     1   1    23
## 24   02-May-2018_14:23:55  Exp 7302  22 female    R     1   1    24
## 25   02-May-2018_14:23:57  Exp 7302  22 female    R     1   2     1
## 26   02-May-2018_14:24:00  Exp 7302  22 female    R     1   2     2
## 27   02-May-2018_14:24:02  Exp 7302  22 female    R     1   2     3
## 28   02-May-2018_14:24:04  Exp 7302  22 female    R     1   2     4
## 29   02-May-2018_14:24:06  Exp 7302  22 female    R     1   2     5
## 30   02-May-2018_14:24:08  Exp 7302  22 female    R     1   2     6
## 31   02-May-2018_14:24:10  Exp 7302  22 female    R     1   2     7
## 32   02-May-2018_14:24:12  Exp 7302  22 female    R     1   2     8
## 33   02-May-2018_14:24:15  Exp 7302  22 female    R     1   2     9
## 34   02-May-2018_14:24:16  Exp 7302  22 female    R     1   2    10
## 35   02-May-2018_14:24:19  Exp 7302  22 female    R     1   2    11
## 36   02-May-2018_14:24:21  Exp 7302  22 female    R     1   2    12
## 37   02-May-2018_14:24:23  Exp 7302  22 female    R     1   2    13
## 38   02-May-2018_14:24:25  Exp 7302  22 female    R     1   2    14
## 39   02-May-2018_14:24:27  Exp 7302  22 female    R     1   2    15
## 40   02-May-2018_14:24:29  Exp 7302  22 female    R     1   2    16
## 41   02-May-2018_14:24:32  Exp 7302  22 female    R     1   2    17
## 42   02-May-2018_14:24:34  Exp 7302  22 female    R     1   2    18
## 43   02-May-2018_14:24:36  Exp 7302  22 female    R     1   2    19
## 44   02-May-2018_14:24:38  Exp 7302  22 female    R     1   2    20
## 45   02-May-2018_14:24:40  Exp 7302  22 female    R     1   2    21
## 46   02-May-2018_14:24:42  Exp 7302  22 female    R     1   2    22
## 47   02-May-2018_14:24:44  Exp 7302  22 female    R     1   2    23
## 48   02-May-2018_14:24:46  Exp 7302  22 female    R     1   2    24
## 49   02-May-2018_14:24:48  Exp 7302  22 female    R     1   3     1
## 50   02-May-2018_14:24:51  Exp 7302  22 female    R     1   3     2
## 51   02-May-2018_14:24:53  Exp 7302  22 female    R     1   3     3
## 52   02-May-2018_14:24:55  Exp 7302  22 female    R     1   3     4
## 53   02-May-2018_14:24:57  Exp 7302  22 female    R     1   3     5
## 54   02-May-2018_14:24:59  Exp 7302  22 female    R     1   3     6
## 55   02-May-2018_14:25:01  Exp 7302  22 female    R     1   3     7
## 56   02-May-2018_14:25:03  Exp 7302  22 female    R     1   3     8
## 57   02-May-2018_14:25:05  Exp 7302  22 female    R     1   3     9
## 58   02-May-2018_14:25:08  Exp 7302  22 female    R     1   3    10
## 59   02-May-2018_14:25:10  Exp 7302  22 female    R     1   3    11
## 60   02-May-2018_14:25:12  Exp 7302  22 female    R     1   3    12
## 61   02-May-2018_14:25:14  Exp 7302  22 female    R     1   3    13
## 62   02-May-2018_14:25:16  Exp 7302  22 female    R     1   3    14
## 63   02-May-2018_14:25:19  Exp 7302  22 female    R     1   3    15
## 64   02-May-2018_14:25:21  Exp 7302  22 female    R     1   3    16
## 65   02-May-2018_14:25:23  Exp 7302  22 female    R     1   3    17
## 66   02-May-2018_14:25:25  Exp 7302  22 female    R     1   3    18
## 67   02-May-2018_14:25:27  Exp 7302  22 female    R     1   3    19
## 68   02-May-2018_14:25:30  Exp 7302  22 female    R     1   3    20
## 69   02-May-2018_14:25:32  Exp 7302  22 female    R     1   3    21
## 70   02-May-2018_14:25:34  Exp 7302  22 female    R     1   3    22
## 71   02-May-2018_14:25:36  Exp 7302  22 female    R     1   3    23
## 72   02-May-2018_14:25:38  Exp 7302  22 female    R     1   3    24
## 73   02-May-2018_14:25:53  Exp 7302  22 female    R     1   4     1
## 74   02-May-2018_14:25:55  Exp 7302  22 female    R     1   4     2
## 75   02-May-2018_14:25:57  Exp 7302  22 female    R     1   4     3
## 76   02-May-2018_14:26:00  Exp 7302  22 female    R     1   4     4
## 77   02-May-2018_14:26:02  Exp 7302  22 female    R     1   4     5
## 78   02-May-2018_14:26:04  Exp 7302  22 female    R     1   4     6
## 79   02-May-2018_14:26:06  Exp 7302  22 female    R     1   4     7
## 80   02-May-2018_14:26:08  Exp 7302  22 female    R     1   4     8
## 81   02-May-2018_14:26:10  Exp 7302  22 female    R     1   4     9
## 82   02-May-2018_14:26:11  Exp 7302  22 female    R     1   4    10
## 83   02-May-2018_14:26:14  Exp 7302  22 female    R     1   4    11
## 84   02-May-2018_14:26:16  Exp 7302  22 female    R     1   4    12
## 85   02-May-2018_14:26:18  Exp 7302  22 female    R     1   4    13
## 86   02-May-2018_14:26:20  Exp 7302  22 female    R     1   4    14
## 87   02-May-2018_14:26:22  Exp 7302  22 female    R     1   4    15
## 88   02-May-2018_14:26:25  Exp 7302  22 female    R     1   4    16
## 89   02-May-2018_14:26:26  Exp 7302  22 female    R     1   4    17
## 90   02-May-2018_14:26:28  Exp 7302  22 female    R     1   4    18
## 91   02-May-2018_14:26:31  Exp 7302  22 female    R     1   4    19
## 92   02-May-2018_14:26:33  Exp 7302  22 female    R     1   4    20
## 93   02-May-2018_14:26:35  Exp 7302  22 female    R     1   4    21
## 94   02-May-2018_14:26:37  Exp 7302  22 female    R     1   4    22
## 95   02-May-2018_14:26:39  Exp 7302  22 female    R     1   4    23
## 96   02-May-2018_14:26:41  Exp 7302  22 female    R     1   4    24
## 97   02-May-2018_14:26:43  Exp 7302  22 female    R     1   5     1
## 98   02-May-2018_14:26:45  Exp 7302  22 female    R     1   5     2
## 99   02-May-2018_14:26:47  Exp 7302  22 female    R     1   5     3
## 100  02-May-2018_14:26:49  Exp 7302  22 female    R     1   5     4
## 101  02-May-2018_14:26:51  Exp 7302  22 female    R     1   5     5
## 102  02-May-2018_14:26:53  Exp 7302  22 female    R     1   5     6
## 103  02-May-2018_14:26:56  Exp 7302  22 female    R     1   5     7
## 104  02-May-2018_14:26:58  Exp 7302  22 female    R     1   5     8
## 105  02-May-2018_14:27:00  Exp 7302  22 female    R     1   5     9
## 106  02-May-2018_14:27:02  Exp 7302  22 female    R     1   5    10
## 107  02-May-2018_14:27:04  Exp 7302  22 female    R     1   5    11
## 108  02-May-2018_14:27:06  Exp 7302  22 female    R     1   5    12
## 109  02-May-2018_14:27:08  Exp 7302  22 female    R     1   5    13
## 110  02-May-2018_14:27:10  Exp 7302  22 female    R     1   5    14
## 111  02-May-2018_14:27:12  Exp 7302  22 female    R     1   5    15
## 112  02-May-2018_14:27:14  Exp 7302  22 female    R     1   5    16
## 113  02-May-2018_14:27:16  Exp 7302  22 female    R     1   5    17
## 114  02-May-2018_14:27:18  Exp 7302  22 female    R     1   5    18
## 115  02-May-2018_14:27:20  Exp 7302  22 female    R     1   5    19
## 116  02-May-2018_14:27:22  Exp 7302  22 female    R     1   5    20
## 117  02-May-2018_14:27:24  Exp 7302  22 female    R     1   5    21
## 118  02-May-2018_14:27:26  Exp 7302  22 female    R     1   5    22
## 119  02-May-2018_14:27:28  Exp 7302  22 female    R     1   5    23
## 120  02-May-2018_14:27:30  Exp 7302  22 female    R     1   5    24
## 121  02-May-2018_14:27:35  Exp 7302  22 female    R     2   1     1
## 122  02-May-2018_14:27:38  Exp 7302  22 female    R     2   1     2
## 123  02-May-2018_14:27:40  Exp 7302  22 female    R     2   1     3
## 124  02-May-2018_14:27:42  Exp 7302  22 female    R     2   1     4
## 125  02-May-2018_14:27:44  Exp 7302  22 female    R     2   1     5
## 126  02-May-2018_14:27:46  Exp 7302  22 female    R     2   1     6
## 127  02-May-2018_14:27:48  Exp 7302  22 female    R     2   1     7
## 128  02-May-2018_14:27:50  Exp 7302  22 female    R     2   1     8
## 129  02-May-2018_14:27:52  Exp 7302  22 female    R     2   1     9
## 130  02-May-2018_14:27:54  Exp 7302  22 female    R     2   1    10
## 131  02-May-2018_14:27:56  Exp 7302  22 female    R     2   1    11
## 132  02-May-2018_14:27:59  Exp 7302  22 female    R     2   1    12
## 133  02-May-2018_14:28:01  Exp 7302  22 female    R     2   1    13
## 134  02-May-2018_14:28:03  Exp 7302  22 female    R     2   1    14
## 135  02-May-2018_14:28:05  Exp 7302  22 female    R     2   1    15
## 136  02-May-2018_14:28:07  Exp 7302  22 female    R     2   1    16
## 137  02-May-2018_14:28:09  Exp 7302  22 female    R     2   1    17
## 138  02-May-2018_14:28:11  Exp 7302  22 female    R     2   1    18
## 139  02-May-2018_14:28:13  Exp 7302  22 female    R     2   1    19
## 140  02-May-2018_14:28:15  Exp 7302  22 female    R     2   1    20
## 141  02-May-2018_14:28:18  Exp 7302  22 female    R     2   1    21
## 142  02-May-2018_14:28:20  Exp 7302  22 female    R     2   1    22
## 143  02-May-2018_14:28:22  Exp 7302  22 female    R     2   1    23
## 144  02-May-2018_14:28:24  Exp 7302  22 female    R     2   1    24
## 145  02-May-2018_14:28:26  Exp 7302  22 female    R     2   2     1
## 146  02-May-2018_14:28:28  Exp 7302  22 female    R     2   2     2
## 147  02-May-2018_14:28:30  Exp 7302  22 female    R     2   2     3
## 148  02-May-2018_14:28:32  Exp 7302  22 female    R     2   2     4
## 149  02-May-2018_14:28:34  Exp 7302  22 female    R     2   2     5
## 150  02-May-2018_14:28:36  Exp 7302  22 female    R     2   2     6
## 151  02-May-2018_14:28:38  Exp 7302  22 female    R     2   2     7
## 152  02-May-2018_14:28:40  Exp 7302  22 female    R     2   2     8
## 153  02-May-2018_14:28:42  Exp 7302  22 female    R     2   2     9
## 154  02-May-2018_14:28:44  Exp 7302  22 female    R     2   2    10
## 155  02-May-2018_14:28:46  Exp 7302  22 female    R     2   2    11
## 156  02-May-2018_14:28:48  Exp 7302  22 female    R     2   2    12
## 157  02-May-2018_14:28:50  Exp 7302  22 female    R     2   2    13
## 158  02-May-2018_14:28:53  Exp 7302  22 female    R     2   2    14
## 159  02-May-2018_14:28:55  Exp 7302  22 female    R     2   2    15
## 160  02-May-2018_14:28:57  Exp 7302  22 female    R     2   2    16
## 161  02-May-2018_14:28:59  Exp 7302  22 female    R     2   2    17
## 162  02-May-2018_14:29:01  Exp 7302  22 female    R     2   2    18
## 163  02-May-2018_14:29:03  Exp 7302  22 female    R     2   2    19
## 164  02-May-2018_14:29:06  Exp 7302  22 female    R     2   2    20
## 165  02-May-2018_14:29:08  Exp 7302  22 female    R     2   2    21
## 166  02-May-2018_14:29:10  Exp 7302  22 female    R     2   2    22
## 167  02-May-2018_14:29:12  Exp 7302  22 female    R     2   2    23
## 168  02-May-2018_14:29:14  Exp 7302  22 female    R     2   2    24
## 169  02-May-2018_14:29:16  Exp 7302  22 female    R     2   3     1
## 170  02-May-2018_14:29:18  Exp 7302  22 female    R     2   3     2
## 171  02-May-2018_14:29:20  Exp 7302  22 female    R     2   3     3
## 172  02-May-2018_14:29:22  Exp 7302  22 female    R     2   3     4
## 173  02-May-2018_14:29:24  Exp 7302  22 female    R     2   3     5
## 174  02-May-2018_14:29:26  Exp 7302  22 female    R     2   3     6
## 175  02-May-2018_14:29:28  Exp 7302  22 female    R     2   3     7
## 176  02-May-2018_14:29:30  Exp 7302  22 female    R     2   3     8
## 177  02-May-2018_14:29:33  Exp 7302  22 female    R     2   3     9
## 178  02-May-2018_14:29:35  Exp 7302  22 female    R     2   3    10
## 179  02-May-2018_14:29:37  Exp 7302  22 female    R     2   3    11
## 180  02-May-2018_14:29:39  Exp 7302  22 female    R     2   3    12
## 181  02-May-2018_14:29:41  Exp 7302  22 female    R     2   3    13
## 182  02-May-2018_14:29:43  Exp 7302  22 female    R     2   3    14
## 183  02-May-2018_14:29:45  Exp 7302  22 female    R     2   3    15
## 184  02-May-2018_14:29:48  Exp 7302  22 female    R     2   3    16
## 185  02-May-2018_14:29:50  Exp 7302  22 female    R     2   3    17
## 186  02-May-2018_14:29:52  Exp 7302  22 female    R     2   3    18
## 187  02-May-2018_14:29:54  Exp 7302  22 female    R     2   3    19
## 188  02-May-2018_14:29:56  Exp 7302  22 female    R     2   3    20
## 189  02-May-2018_14:29:58  Exp 7302  22 female    R     2   3    21
## 190  02-May-2018_14:30:00  Exp 7302  22 female    R     2   3    22
## 191  02-May-2018_14:30:02  Exp 7302  22 female    R     2   3    23
## 192  02-May-2018_14:30:04  Exp 7302  22 female    R     2   3    24
## 193  02-May-2018_14:30:19  Exp 7302  22 female    R     2   4     1
## 194  02-May-2018_14:30:21  Exp 7302  22 female    R     2   4     2
## 195  02-May-2018_14:30:23  Exp 7302  22 female    R     2   4     3
## 196  02-May-2018_14:30:25  Exp 7302  22 female    R     2   4     4
## 197  02-May-2018_14:30:27  Exp 7302  22 female    R     2   4     5
## 198  02-May-2018_14:30:29  Exp 7302  22 female    R     2   4     6
## 199  02-May-2018_14:30:31  Exp 7302  22 female    R     2   4     7
## 200  02-May-2018_14:30:33  Exp 7302  22 female    R     2   4     8
## 201  02-May-2018_14:30:35  Exp 7302  22 female    R     2   4     9
## 202  02-May-2018_14:30:38  Exp 7302  22 female    R     2   4    10
## 203  02-May-2018_14:30:40  Exp 7302  22 female    R     2   4    11
## 204  02-May-2018_14:30:42  Exp 7302  22 female    R     2   4    12
## 205  02-May-2018_14:30:44  Exp 7302  22 female    R     2   4    13
## 206  02-May-2018_14:30:46  Exp 7302  22 female    R     2   4    14
## 207  02-May-2018_14:30:48  Exp 7302  22 female    R     2   4    15
## 208  02-May-2018_14:30:50  Exp 7302  22 female    R     2   4    16
## 209  02-May-2018_14:30:52  Exp 7302  22 female    R     2   4    17
## 210  02-May-2018_14:30:54  Exp 7302  22 female    R     2   4    18
## 211  02-May-2018_14:30:56  Exp 7302  22 female    R     2   4    19
## 212  02-May-2018_14:30:58  Exp 7302  22 female    R     2   4    20
## 213  02-May-2018_14:31:01  Exp 7302  22 female    R     2   4    21
## 214  02-May-2018_14:31:03  Exp 7302  22 female    R     2   4    22
## 215  02-May-2018_14:31:05  Exp 7302  22 female    R     2   4    23
## 216  02-May-2018_14:31:07  Exp 7302  22 female    R     2   4    24
## 217  02-May-2018_14:31:09  Exp 7302  22 female    R     2   5     1
## 218  02-May-2018_14:31:11  Exp 7302  22 female    R     2   5     2
## 219  02-May-2018_14:31:13  Exp 7302  22 female    R     2   5     3
## 220  02-May-2018_14:31:15  Exp 7302  22 female    R     2   5     4
## 221  02-May-2018_14:31:17  Exp 7302  22 female    R     2   5     5
## 222  02-May-2018_14:31:20  Exp 7302  22 female    R     2   5     6
## 223  02-May-2018_14:31:22  Exp 7302  22 female    R     2   5     7
## 224  02-May-2018_14:31:24  Exp 7302  22 female    R     2   5     8
## 225  02-May-2018_14:31:26  Exp 7302  22 female    R     2   5     9
## 226  02-May-2018_14:31:28  Exp 7302  22 female    R     2   5    10
## 227  02-May-2018_14:31:30  Exp 7302  22 female    R     2   5    11
## 228  02-May-2018_14:31:32  Exp 7302  22 female    R     2   5    12
## 229  02-May-2018_14:31:34  Exp 7302  22 female    R     2   5    13
## 230  02-May-2018_14:31:36  Exp 7302  22 female    R     2   5    14
## 231  02-May-2018_14:31:39  Exp 7302  22 female    R     2   5    15
## 232  02-May-2018_14:31:41  Exp 7302  22 female    R     2   5    16
## 233  02-May-2018_14:31:43  Exp 7302  22 female    R     2   5    17
## 234  02-May-2018_14:31:45  Exp 7302  22 female    R     2   5    18
## 235  02-May-2018_14:31:47  Exp 7302  22 female    R     2   5    19
## 236  02-May-2018_14:31:49  Exp 7302  22 female    R     2   5    20
## 237  02-May-2018_14:31:51  Exp 7302  22 female    R     2   5    21
## 238  02-May-2018_14:31:53  Exp 7302  22 female    R     2   5    22
## 239  02-May-2018_14:31:56  Exp 7302  22 female    R     2   5    23
## 240  02-May-2018_14:31:58  Exp 7302  22 female    R     2   5    24
## 241  02-May-2018_14:32:03  Exp 7302  22 female    R     3   1     1
## 242  02-May-2018_14:32:05  Exp 7302  22 female    R     3   1     2
## 243  02-May-2018_14:32:07  Exp 7302  22 female    R     3   1     3
## 244  02-May-2018_14:32:09  Exp 7302  22 female    R     3   1     4
## 245  02-May-2018_14:32:11  Exp 7302  22 female    R     3   1     5
## 246  02-May-2018_14:32:13  Exp 7302  22 female    R     3   1     6
## 247  02-May-2018_14:32:15  Exp 7302  22 female    R     3   1     7
## 248  02-May-2018_14:32:17  Exp 7302  22 female    R     3   1     8
## 249  02-May-2018_14:32:19  Exp 7302  22 female    R     3   1     9
## 250  02-May-2018_14:32:22  Exp 7302  22 female    R     3   1    10
## 251  02-May-2018_14:32:24  Exp 7302  22 female    R     3   1    11
## 252  02-May-2018_14:32:26  Exp 7302  22 female    R     3   1    12
## 253  02-May-2018_14:32:28  Exp 7302  22 female    R     3   1    13
## 254  02-May-2018_14:32:30  Exp 7302  22 female    R     3   1    14
## 255  02-May-2018_14:32:32  Exp 7302  22 female    R     3   1    15
## 256  02-May-2018_14:32:34  Exp 7302  22 female    R     3   1    16
## 257  02-May-2018_14:32:36  Exp 7302  22 female    R     3   1    17
## 258  02-May-2018_14:32:39  Exp 7302  22 female    R     3   1    18
## 259  02-May-2018_14:32:41  Exp 7302  22 female    R     3   1    19
## 260  02-May-2018_14:32:43  Exp 7302  22 female    R     3   1    20
## 261  02-May-2018_14:32:45  Exp 7302  22 female    R     3   1    21
## 262  02-May-2018_14:32:47  Exp 7302  22 female    R     3   1    22
## 263  02-May-2018_14:32:49  Exp 7302  22 female    R     3   1    23
## 264  02-May-2018_14:32:51  Exp 7302  22 female    R     3   1    24
## 265  02-May-2018_14:32:53  Exp 7302  22 female    R     3   2     1
## 266  02-May-2018_14:32:55  Exp 7302  22 female    R     3   2     2
## 267  02-May-2018_14:32:57  Exp 7302  22 female    R     3   2     3
## 268  02-May-2018_14:32:59  Exp 7302  22 female    R     3   2     4
## 269  02-May-2018_14:33:01  Exp 7302  22 female    R     3   2     5
## 270  02-May-2018_14:33:03  Exp 7302  22 female    R     3   2     6
## 271  02-May-2018_14:33:05  Exp 7302  22 female    R     3   2     7
## 272  02-May-2018_14:33:07  Exp 7302  22 female    R     3   2     8
## 273  02-May-2018_14:33:09  Exp 7302  22 female    R     3   2     9
## 274  02-May-2018_14:33:11  Exp 7302  22 female    R     3   2    10
## 275  02-May-2018_14:33:13  Exp 7302  22 female    R     3   2    11
## 276  02-May-2018_14:33:15  Exp 7302  22 female    R     3   2    12
## 277  02-May-2018_14:33:17  Exp 7302  22 female    R     3   2    13
## 278  02-May-2018_14:33:19  Exp 7302  22 female    R     3   2    14
## 279  02-May-2018_14:33:21  Exp 7302  22 female    R     3   2    15
## 280  02-May-2018_14:33:23  Exp 7302  22 female    R     3   2    16
## 281  02-May-2018_14:33:25  Exp 7302  22 female    R     3   2    17
## 282  02-May-2018_14:33:27  Exp 7302  22 female    R     3   2    18
## 283  02-May-2018_14:33:29  Exp 7302  22 female    R     3   2    19
## 284  02-May-2018_14:33:31  Exp 7302  22 female    R     3   2    20
## 285  02-May-2018_14:33:33  Exp 7302  22 female    R     3   2    21
## 286  02-May-2018_14:33:35  Exp 7302  22 female    R     3   2    22
## 287  02-May-2018_14:33:38  Exp 7302  22 female    R     3   2    23
## 288  02-May-2018_14:33:40  Exp 7302  22 female    R     3   2    24
## 289  02-May-2018_14:33:42  Exp 7302  22 female    R     3   3     1
## 290  02-May-2018_14:33:44  Exp 7302  22 female    R     3   3     2
## 291  02-May-2018_14:33:46  Exp 7302  22 female    R     3   3     3
## 292  02-May-2018_14:33:48  Exp 7302  22 female    R     3   3     4
## 293  02-May-2018_14:33:50  Exp 7302  22 female    R     3   3     5
## 294  02-May-2018_14:33:52  Exp 7302  22 female    R     3   3     6
## 295  02-May-2018_14:33:54  Exp 7302  22 female    R     3   3     7
## 296  02-May-2018_14:33:56  Exp 7302  22 female    R     3   3     8
## 297  02-May-2018_14:33:59  Exp 7302  22 female    R     3   3     9
## 298  02-May-2018_14:34:01  Exp 7302  22 female    R     3   3    10
## 299  02-May-2018_14:34:03  Exp 7302  22 female    R     3   3    11
## 300  02-May-2018_14:34:05  Exp 7302  22 female    R     3   3    12
## 301  02-May-2018_14:34:07  Exp 7302  22 female    R     3   3    13
## 302  02-May-2018_14:34:09  Exp 7302  22 female    R     3   3    14
## 303  02-May-2018_14:34:11  Exp 7302  22 female    R     3   3    15
## 304  02-May-2018_14:34:13  Exp 7302  22 female    R     3   3    16
## 305  02-May-2018_14:34:15  Exp 7302  22 female    R     3   3    17
## 306  02-May-2018_14:34:17  Exp 7302  22 female    R     3   3    18
## 307  02-May-2018_14:34:19  Exp 7302  22 female    R     3   3    19
## 308  02-May-2018_14:34:21  Exp 7302  22 female    R     3   3    20
## 309  02-May-2018_14:34:24  Exp 7302  22 female    R     3   3    21
## 310  02-May-2018_14:34:26  Exp 7302  22 female    R     3   3    22
## 311  02-May-2018_14:34:28  Exp 7302  22 female    R     3   3    23
## 312  02-May-2018_14:34:30  Exp 7302  22 female    R     3   3    24
## 313  02-May-2018_14:34:40  Exp 7302  22 female    R     3   4     1
## 314  02-May-2018_14:34:42  Exp 7302  22 female    R     3   4     2
## 315  02-May-2018_14:34:45  Exp 7302  22 female    R     3   4     3
## 316  02-May-2018_14:34:47  Exp 7302  22 female    R     3   4     4
## 317  02-May-2018_14:34:49  Exp 7302  22 female    R     3   4     5
## 318  02-May-2018_14:34:51  Exp 7302  22 female    R     3   4     6
## 319  02-May-2018_14:34:53  Exp 7302  22 female    R     3   4     7
## 320  02-May-2018_14:34:56  Exp 7302  22 female    R     3   4     8
## 321  02-May-2018_14:34:58  Exp 7302  22 female    R     3   4     9
## 322  02-May-2018_14:35:00  Exp 7302  22 female    R     3   4    10
## 323  02-May-2018_14:35:02  Exp 7302  22 female    R     3   4    11
## 324  02-May-2018_14:35:04  Exp 7302  22 female    R     3   4    12
## 325  02-May-2018_14:35:06  Exp 7302  22 female    R     3   4    13
## 326  02-May-2018_14:35:08  Exp 7302  22 female    R     3   4    14
## 327  02-May-2018_14:35:10  Exp 7302  22 female    R     3   4    15
## 328  02-May-2018_14:35:13  Exp 7302  22 female    R     3   4    16
## 329  02-May-2018_14:35:15  Exp 7302  22 female    R     3   4    17
## 330  02-May-2018_14:35:17  Exp 7302  22 female    R     3   4    18
## 331  02-May-2018_14:35:19  Exp 7302  22 female    R     3   4    19
## 332  02-May-2018_14:35:21  Exp 7302  22 female    R     3   4    20
## 333  02-May-2018_14:35:23  Exp 7302  22 female    R     3   4    21
## 334  02-May-2018_14:35:25  Exp 7302  22 female    R     3   4    22
## 335  02-May-2018_14:35:28  Exp 7302  22 female    R     3   4    23
## 336  02-May-2018_14:35:30  Exp 7302  22 female    R     3   4    24
## 337  02-May-2018_14:35:32  Exp 7302  22 female    R     3   5     1
## 338  02-May-2018_14:35:34  Exp 7302  22 female    R     3   5     2
## 339  02-May-2018_14:35:36  Exp 7302  22 female    R     3   5     3
## 340  02-May-2018_14:35:38  Exp 7302  22 female    R     3   5     4
## 341  02-May-2018_14:35:40  Exp 7302  22 female    R     3   5     5
## 342  02-May-2018_14:35:42  Exp 7302  22 female    R     3   5     6
## 343  02-May-2018_14:35:44  Exp 7302  22 female    R     3   5     7
## 344  02-May-2018_14:35:46  Exp 7302  22 female    R     3   5     8
## 345  02-May-2018_14:35:48  Exp 7302  22 female    R     3   5     9
## 346  02-May-2018_14:35:50  Exp 7302  22 female    R     3   5    10
## 347  02-May-2018_14:35:52  Exp 7302  22 female    R     3   5    11
## 348  02-May-2018_14:35:55  Exp 7302  22 female    R     3   5    12
## 349  02-May-2018_14:35:57  Exp 7302  22 female    R     3   5    13
## 350  02-May-2018_14:35:59  Exp 7302  22 female    R     3   5    14
## 351  02-May-2018_14:36:01  Exp 7302  22 female    R     3   5    15
## 352  02-May-2018_14:36:03  Exp 7302  22 female    R     3   5    16
## 353  02-May-2018_14:36:05  Exp 7302  22 female    R     3   5    17
## 354  02-May-2018_14:36:07  Exp 7302  22 female    R     3   5    18
## 355  02-May-2018_14:36:09  Exp 7302  22 female    R     3   5    19
## 356  02-May-2018_14:36:11  Exp 7302  22 female    R     3   5    20
## 357  02-May-2018_14:36:13  Exp 7302  22 female    R     3   5    21
## 358  02-May-2018_14:36:15  Exp 7302  22 female    R     3   5    22
## 359  02-May-2018_14:36:17  Exp 7302  22 female    R     3   5    23
## 360  02-May-2018_14:36:19  Exp 7302  22 female    R     3   5    24
## 361  02-May-2018_14:26:54  Exp 7303  28   male    R     1   1     1
## 362  02-May-2018_14:26:56  Exp 7303  28   male    R     1   1     2
## 363  02-May-2018_14:26:58  Exp 7303  28   male    R     1   1     3
## 364  02-May-2018_14:27:00  Exp 7303  28   male    R     1   1     4
## 365  02-May-2018_14:27:02  Exp 7303  28   male    R     1   1     5
## 366  02-May-2018_14:27:04  Exp 7303  28   male    R     1   1     6
## 367  02-May-2018_14:27:06  Exp 7303  28   male    R     1   1     7
## 368  02-May-2018_14:27:08  Exp 7303  28   male    R     1   1     8
## 369  02-May-2018_14:27:10  Exp 7303  28   male    R     1   1     9
## 370  02-May-2018_14:27:12  Exp 7303  28   male    R     1   1    10
## 371  02-May-2018_14:27:14  Exp 7303  28   male    R     1   1    11
## 372  02-May-2018_14:27:17  Exp 7303  28   male    R     1   1    12
## 373  02-May-2018_14:27:19  Exp 7303  28   male    R     1   1    13
## 374  02-May-2018_14:27:21  Exp 7303  28   male    R     1   1    14
## 375  02-May-2018_14:27:23  Exp 7303  28   male    R     1   1    15
## 376  02-May-2018_14:27:25  Exp 7303  28   male    R     1   1    16
## 377  02-May-2018_14:27:27  Exp 7303  28   male    R     1   1    17
## 378  02-May-2018_14:27:29  Exp 7303  28   male    R     1   1    18
## 379  02-May-2018_14:27:31  Exp 7303  28   male    R     1   1    19
## 380  02-May-2018_14:27:33  Exp 7303  28   male    R     1   1    20
## 381  02-May-2018_14:27:35  Exp 7303  28   male    R     1   1    21
## 382  02-May-2018_14:27:37  Exp 7303  28   male    R     1   1    22
## 383  02-May-2018_14:27:39  Exp 7303  28   male    R     1   1    23
## 384  02-May-2018_14:27:41  Exp 7303  28   male    R     1   1    24
## 385  02-May-2018_14:27:43  Exp 7303  28   male    R     1   2     1
## 386  02-May-2018_14:27:45  Exp 7303  28   male    R     1   2     2
## 387  02-May-2018_14:27:47  Exp 7303  28   male    R     1   2     3
## 388  02-May-2018_14:27:49  Exp 7303  28   male    R     1   2     4
## 389  02-May-2018_14:27:51  Exp 7303  28   male    R     1   2     5
## 390  02-May-2018_14:27:53  Exp 7303  28   male    R     1   2     6
## 391  02-May-2018_14:27:55  Exp 7303  28   male    R     1   2     7
## 392  02-May-2018_14:27:58  Exp 7303  28   male    R     1   2     8
## 393  02-May-2018_14:28:00  Exp 7303  28   male    R     1   2     9
## 394  02-May-2018_14:28:02  Exp 7303  28   male    R     1   2    10
## 395  02-May-2018_14:28:04  Exp 7303  28   male    R     1   2    11
## 396  02-May-2018_14:28:06  Exp 7303  28   male    R     1   2    12
## 397  02-May-2018_14:28:08  Exp 7303  28   male    R     1   2    13
## 398  02-May-2018_14:28:10  Exp 7303  28   male    R     1   2    14
## 399  02-May-2018_14:28:12  Exp 7303  28   male    R     1   2    15
## 400  02-May-2018_14:28:15  Exp 7303  28   male    R     1   2    16
## 401  02-May-2018_14:28:17  Exp 7303  28   male    R     1   2    17
## 402  02-May-2018_14:28:19  Exp 7303  28   male    R     1   2    18
## 403  02-May-2018_14:28:21  Exp 7303  28   male    R     1   2    19
## 404  02-May-2018_14:28:23  Exp 7303  28   male    R     1   2    20
## 405  02-May-2018_14:28:26  Exp 7303  28   male    R     1   2    21
## 406  02-May-2018_14:28:28  Exp 7303  28   male    R     1   2    22
## 407  02-May-2018_14:28:30  Exp 7303  28   male    R     1   2    23
## 408  02-May-2018_14:28:32  Exp 7303  28   male    R     1   2    24
## 409  02-May-2018_14:28:34  Exp 7303  28   male    R     1   3     1
## 410  02-May-2018_14:28:36  Exp 7303  28   male    R     1   3     2
## 411  02-May-2018_14:28:38  Exp 7303  28   male    R     1   3     3
## 412  02-May-2018_14:28:40  Exp 7303  28   male    R     1   3     4
## 413  02-May-2018_14:28:42  Exp 7303  28   male    R     1   3     5
## 414  02-May-2018_14:28:44  Exp 7303  28   male    R     1   3     6
## 415  02-May-2018_14:28:47  Exp 7303  28   male    R     1   3     7
## 416  02-May-2018_14:28:49  Exp 7303  28   male    R     1   3     8
## 417  02-May-2018_14:28:51  Exp 7303  28   male    R     1   3     9
## 418  02-May-2018_14:28:53  Exp 7303  28   male    R     1   3    10
## 419  02-May-2018_14:28:55  Exp 7303  28   male    R     1   3    11
## 420  02-May-2018_14:28:57  Exp 7303  28   male    R     1   3    12
## 421  02-May-2018_14:28:59  Exp 7303  28   male    R     1   3    13
## 422  02-May-2018_14:29:01  Exp 7303  28   male    R     1   3    14
## 423  02-May-2018_14:29:04  Exp 7303  28   male    R     1   3    15
## 424  02-May-2018_14:29:06  Exp 7303  28   male    R     1   3    16
## 425  02-May-2018_14:29:08  Exp 7303  28   male    R     1   3    17
## 426  02-May-2018_14:29:10  Exp 7303  28   male    R     1   3    18
## 427  02-May-2018_14:29:12  Exp 7303  28   male    R     1   3    19
## 428  02-May-2018_14:29:14  Exp 7303  28   male    R     1   3    20
## 429  02-May-2018_14:29:16  Exp 7303  28   male    R     1   3    21
## 430  02-May-2018_14:29:18  Exp 7303  28   male    R     1   3    22
## 431  02-May-2018_14:29:21  Exp 7303  28   male    R     1   3    23
## 432  02-May-2018_14:29:23  Exp 7303  28   male    R     1   3    24
## 433  02-May-2018_14:29:36  Exp 7303  28   male    R     1   4     1
## 434  02-May-2018_14:29:38  Exp 7303  28   male    R     1   4     2
## 435  02-May-2018_14:29:40  Exp 7303  28   male    R     1   4     3
## 436  02-May-2018_14:29:42  Exp 7303  28   male    R     1   4     4
## 437  02-May-2018_14:29:44  Exp 7303  28   male    R     1   4     5
## 438  02-May-2018_14:29:46  Exp 7303  28   male    R     1   4     6
## 439  02-May-2018_14:29:48  Exp 7303  28   male    R     1   4     7
## 440  02-May-2018_14:29:50  Exp 7303  28   male    R     1   4     8
## 441  02-May-2018_14:29:52  Exp 7303  28   male    R     1   4     9
## 442  02-May-2018_14:29:54  Exp 7303  28   male    R     1   4    10
## 443  02-May-2018_14:29:56  Exp 7303  28   male    R     1   4    11
## 444  02-May-2018_14:29:58  Exp 7303  28   male    R     1   4    12
## 445  02-May-2018_14:30:00  Exp 7303  28   male    R     1   4    13
## 446  02-May-2018_14:30:03  Exp 7303  28   male    R     1   4    14
## 447  02-May-2018_14:30:05  Exp 7303  28   male    R     1   4    15
## 448  02-May-2018_14:30:07  Exp 7303  28   male    R     1   4    16
## 449  02-May-2018_14:30:09  Exp 7303  28   male    R     1   4    17
## 450  02-May-2018_14:30:11  Exp 7303  28   male    R     1   4    18
## 451  02-May-2018_14:30:13  Exp 7303  28   male    R     1   4    19
## 452  02-May-2018_14:30:15  Exp 7303  28   male    R     1   4    20
## 453  02-May-2018_14:30:18  Exp 7303  28   male    R     1   4    21
## 454  02-May-2018_14:30:20  Exp 7303  28   male    R     1   4    22
## 455  02-May-2018_14:30:22  Exp 7303  28   male    R     1   4    23
## 456  02-May-2018_14:30:24  Exp 7303  28   male    R     1   4    24
## 457  02-May-2018_14:30:26  Exp 7303  28   male    R     1   5     1
## 458  02-May-2018_14:30:29  Exp 7303  28   male    R     1   5     2
## 459  02-May-2018_14:30:31  Exp 7303  28   male    R     1   5     3
## 460  02-May-2018_14:30:33  Exp 7303  28   male    R     1   5     4
## 461  02-May-2018_14:30:35  Exp 7303  28   male    R     1   5     5
## 462  02-May-2018_14:30:37  Exp 7303  28   male    R     1   5     6
## 463  02-May-2018_14:30:39  Exp 7303  28   male    R     1   5     7
## 464  02-May-2018_14:30:41  Exp 7303  28   male    R     1   5     8
## 465  02-May-2018_14:30:43  Exp 7303  28   male    R     1   5     9
## 466  02-May-2018_14:30:45  Exp 7303  28   male    R     1   5    10
## 467  02-May-2018_14:30:48  Exp 7303  28   male    R     1   5    11
## 468  02-May-2018_14:30:50  Exp 7303  28   male    R     1   5    12
## 469  02-May-2018_14:30:52  Exp 7303  28   male    R     1   5    13
## 470  02-May-2018_14:30:54  Exp 7303  28   male    R     1   5    14
## 471  02-May-2018_14:30:56  Exp 7303  28   male    R     1   5    15
## 472  02-May-2018_14:30:58  Exp 7303  28   male    R     1   5    16
## 473  02-May-2018_14:31:00  Exp 7303  28   male    R     1   5    17
## 474  02-May-2018_14:31:03  Exp 7303  28   male    R     1   5    18
## 475  02-May-2018_14:31:05  Exp 7303  28   male    R     1   5    19
## 476  02-May-2018_14:31:07  Exp 7303  28   male    R     1   5    20
## 477  02-May-2018_14:31:09  Exp 7303  28   male    R     1   5    21
## 478  02-May-2018_14:31:11  Exp 7303  28   male    R     1   5    22
## 479  02-May-2018_14:31:13  Exp 7303  28   male    R     1   5    23
## 480  02-May-2018_14:31:15  Exp 7303  28   male    R     1   5    24
## 481  02-May-2018_14:31:20  Exp 7303  28   male    R     2   1     1
## 482  02-May-2018_14:31:22  Exp 7303  28   male    R     2   1     2
## 483  02-May-2018_14:31:24  Exp 7303  28   male    R     2   1     3
## 484  02-May-2018_14:31:26  Exp 7303  28   male    R     2   1     4
## 485  02-May-2018_14:31:29  Exp 7303  28   male    R     2   1     5
## 486  02-May-2018_14:31:31  Exp 7303  28   male    R     2   1     6
## 487  02-May-2018_14:31:33  Exp 7303  28   male    R     2   1     7
## 488  02-May-2018_14:31:35  Exp 7303  28   male    R     2   1     8
## 489  02-May-2018_14:31:37  Exp 7303  28   male    R     2   1     9
## 490  02-May-2018_14:31:39  Exp 7303  28   male    R     2   1    10
## 491  02-May-2018_14:31:41  Exp 7303  28   male    R     2   1    11
## 492  02-May-2018_14:31:43  Exp 7303  28   male    R     2   1    12
## 493  02-May-2018_14:31:45  Exp 7303  28   male    R     2   1    13
## 494  02-May-2018_14:31:47  Exp 7303  28   male    R     2   1    14
## 495  02-May-2018_14:31:49  Exp 7303  28   male    R     2   1    15
## 496  02-May-2018_14:31:51  Exp 7303  28   male    R     2   1    16
## 497  02-May-2018_14:31:54  Exp 7303  28   male    R     2   1    17
## 498  02-May-2018_14:31:56  Exp 7303  28   male    R     2   1    18
## 499  02-May-2018_14:31:58  Exp 7303  28   male    R     2   1    19
## 500  02-May-2018_14:32:00  Exp 7303  28   male    R     2   1    20
## 501  02-May-2018_14:32:02  Exp 7303  28   male    R     2   1    21
## 502  02-May-2018_14:32:04  Exp 7303  28   male    R     2   1    22
## 503  02-May-2018_14:32:06  Exp 7303  28   male    R     2   1    23
## 504  02-May-2018_14:32:09  Exp 7303  28   male    R     2   1    24
## 505  02-May-2018_14:32:11  Exp 7303  28   male    R     2   2     1
## 506  02-May-2018_14:32:13  Exp 7303  28   male    R     2   2     2
## 507  02-May-2018_14:32:15  Exp 7303  28   male    R     2   2     3
## 508  02-May-2018_14:32:17  Exp 7303  28   male    R     2   2     4
## 509  02-May-2018_14:32:19  Exp 7303  28   male    R     2   2     5
## 510  02-May-2018_14:32:21  Exp 7303  28   male    R     2   2     6
## 511  02-May-2018_14:32:23  Exp 7303  28   male    R     2   2     7
## 512  02-May-2018_14:32:25  Exp 7303  28   male    R     2   2     8
## 513  02-May-2018_14:32:27  Exp 7303  28   male    R     2   2     9
## 514  02-May-2018_14:32:29  Exp 7303  28   male    R     2   2    10
## 515  02-May-2018_14:32:31  Exp 7303  28   male    R     2   2    11
## 516  02-May-2018_14:32:33  Exp 7303  28   male    R     2   2    12
## 517  02-May-2018_14:32:35  Exp 7303  28   male    R     2   2    13
## 518  02-May-2018_14:32:37  Exp 7303  28   male    R     2   2    14
## 519  02-May-2018_14:32:39  Exp 7303  28   male    R     2   2    15
## 520  02-May-2018_14:32:41  Exp 7303  28   male    R     2   2    16
## 521  02-May-2018_14:32:43  Exp 7303  28   male    R     2   2    17
## 522  02-May-2018_14:32:46  Exp 7303  28   male    R     2   2    18
## 523  02-May-2018_14:32:48  Exp 7303  28   male    R     2   2    19
## 524  02-May-2018_14:32:50  Exp 7303  28   male    R     2   2    20
## 525  02-May-2018_14:32:52  Exp 7303  28   male    R     2   2    21
## 526  02-May-2018_14:32:54  Exp 7303  28   male    R     2   2    22
## 527  02-May-2018_14:32:56  Exp 7303  28   male    R     2   2    23
## 528  02-May-2018_14:32:58  Exp 7303  28   male    R     2   2    24
## 529  02-May-2018_14:33:00  Exp 7303  28   male    R     2   3     1
## 530  02-May-2018_14:33:02  Exp 7303  28   male    R     2   3     2
## 531  02-May-2018_14:33:04  Exp 7303  28   male    R     2   3     3
## 532  02-May-2018_14:33:06  Exp 7303  28   male    R     2   3     4
## 533  02-May-2018_14:33:08  Exp 7303  28   male    R     2   3     5
## 534  02-May-2018_14:33:10  Exp 7303  28   male    R     2   3     6
## 535  02-May-2018_14:33:12  Exp 7303  28   male    R     2   3     7
## 536  02-May-2018_14:33:15  Exp 7303  28   male    R     2   3     8
## 537  02-May-2018_14:33:17  Exp 7303  28   male    R     2   3     9
## 538  02-May-2018_14:33:19  Exp 7303  28   male    R     2   3    10
## 539  02-May-2018_14:33:21  Exp 7303  28   male    R     2   3    11
## 540  02-May-2018_14:33:23  Exp 7303  28   male    R     2   3    12
## 541  02-May-2018_14:33:25  Exp 7303  28   male    R     2   3    13
## 542  02-May-2018_14:33:27  Exp 7303  28   male    R     2   3    14
## 543  02-May-2018_14:33:29  Exp 7303  28   male    R     2   3    15
## 544  02-May-2018_14:33:32  Exp 7303  28   male    R     2   3    16
## 545  02-May-2018_14:33:34  Exp 7303  28   male    R     2   3    17
## 546  02-May-2018_14:33:36  Exp 7303  28   male    R     2   3    18
## 547  02-May-2018_14:33:38  Exp 7303  28   male    R     2   3    19
## 548  02-May-2018_14:33:40  Exp 7303  28   male    R     2   3    20
## 549  02-May-2018_14:33:42  Exp 7303  28   male    R     2   3    21
## 550  02-May-2018_14:33:44  Exp 7303  28   male    R     2   3    22
## 551  02-May-2018_14:33:47  Exp 7303  28   male    R     2   3    23
## 552  02-May-2018_14:33:49  Exp 7303  28   male    R     2   3    24
## 553  02-May-2018_14:34:59  Exp 7303  28   male    R     2   4     1
## 554  02-May-2018_14:35:01  Exp 7303  28   male    R     2   4     2
## 555  02-May-2018_14:35:03  Exp 7303  28   male    R     2   4     3
## 556  02-May-2018_14:35:05  Exp 7303  28   male    R     2   4     4
## 557  02-May-2018_14:35:08  Exp 7303  28   male    R     2   4     5
## 558  02-May-2018_14:35:09  Exp 7303  28   male    R     2   4     6
## 559  02-May-2018_14:35:12  Exp 7303  28   male    R     2   4     7
## 560  02-May-2018_14:35:14  Exp 7303  28   male    R     2   4     8
## 561  02-May-2018_14:35:16  Exp 7303  28   male    R     2   4     9
## 562  02-May-2018_14:35:18  Exp 7303  28   male    R     2   4    10
## 563  02-May-2018_14:35:20  Exp 7303  28   male    R     2   4    11
## 564  02-May-2018_14:35:22  Exp 7303  28   male    R     2   4    12
## 565  02-May-2018_14:35:24  Exp 7303  28   male    R     2   4    13
## 566  02-May-2018_14:35:26  Exp 7303  28   male    R     2   4    14
## 567  02-May-2018_14:35:28  Exp 7303  28   male    R     2   4    15
## 568  02-May-2018_14:35:31  Exp 7303  28   male    R     2   4    16
## 569  02-May-2018_14:35:33  Exp 7303  28   male    R     2   4    17
## 570  02-May-2018_14:35:35  Exp 7303  28   male    R     2   4    18
## 571  02-May-2018_14:35:37  Exp 7303  28   male    R     2   4    19
## 572  02-May-2018_14:35:39  Exp 7303  28   male    R     2   4    20
## 573  02-May-2018_14:35:41  Exp 7303  28   male    R     2   4    21
## 574  02-May-2018_14:35:43  Exp 7303  28   male    R     2   4    22
## 575  02-May-2018_14:35:45  Exp 7303  28   male    R     2   4    23
## 576  02-May-2018_14:35:48  Exp 7303  28   male    R     2   4    24
## 577  02-May-2018_14:35:49  Exp 7303  28   male    R     2   5     1
## 578  02-May-2018_14:35:52  Exp 7303  28   male    R     2   5     2
## 579  02-May-2018_14:35:54  Exp 7303  28   male    R     2   5     3
## 580  02-May-2018_14:35:56  Exp 7303  28   male    R     2   5     4
## 581  02-May-2018_14:35:58  Exp 7303  28   male    R     2   5     5
## 582  02-May-2018_14:36:00  Exp 7303  28   male    R     2   5     6
## 583  02-May-2018_14:36:02  Exp 7303  28   male    R     2   5     7
## 584  02-May-2018_14:36:04  Exp 7303  28   male    R     2   5     8
## 585  02-May-2018_14:36:07  Exp 7303  28   male    R     2   5     9
## 586  02-May-2018_14:36:09  Exp 7303  28   male    R     2   5    10
## 587  02-May-2018_14:36:11  Exp 7303  28   male    R     2   5    11
## 588  02-May-2018_14:36:13  Exp 7303  28   male    R     2   5    12
## 589  02-May-2018_14:36:15  Exp 7303  28   male    R     2   5    13
## 590  02-May-2018_14:36:17  Exp 7303  28   male    R     2   5    14
## 591  02-May-2018_14:36:19  Exp 7303  28   male    R     2   5    15
## 592  02-May-2018_14:36:21  Exp 7303  28   male    R     2   5    16
## 593  02-May-2018_14:36:23  Exp 7303  28   male    R     2   5    17
## 594  02-May-2018_14:36:25  Exp 7303  28   male    R     2   5    18
## 595  02-May-2018_14:36:28  Exp 7303  28   male    R     2   5    19
## 596  02-May-2018_14:36:30  Exp 7303  28   male    R     2   5    20
## 597  02-May-2018_14:36:32  Exp 7303  28   male    R     2   5    21
## 598  02-May-2018_14:36:34  Exp 7303  28   male    R     2   5    22
## 599  02-May-2018_14:36:36  Exp 7303  28   male    R     2   5    23
## 600  02-May-2018_14:36:38  Exp 7303  28   male    R     2   5    24
## 601  02-May-2018_14:36:43  Exp 7303  28   male    R     3   1     1
## 602  02-May-2018_14:36:45  Exp 7303  28   male    R     3   1     2
## 603  02-May-2018_14:36:48  Exp 7303  28   male    R     3   1     3
## 604  02-May-2018_14:36:50  Exp 7303  28   male    R     3   1     4
## 605  02-May-2018_14:36:52  Exp 7303  28   male    R     3   1     5
## 606  02-May-2018_14:36:55  Exp 7303  28   male    R     3   1     6
## 607  02-May-2018_14:36:57  Exp 7303  28   male    R     3   1     7
## 608  02-May-2018_14:36:59  Exp 7303  28   male    R     3   1     8
## 609  02-May-2018_14:37:01  Exp 7303  28   male    R     3   1     9
## 610  02-May-2018_14:37:03  Exp 7303  28   male    R     3   1    10
## 611  02-May-2018_14:37:05  Exp 7303  28   male    R     3   1    11
## 612  02-May-2018_14:37:07  Exp 7303  28   male    R     3   1    12
## 613  02-May-2018_14:37:09  Exp 7303  28   male    R     3   1    13
## 614  02-May-2018_14:37:12  Exp 7303  28   male    R     3   1    14
## 615  02-May-2018_14:37:14  Exp 7303  28   male    R     3   1    15
## 616  02-May-2018_14:37:15  Exp 7303  28   male    R     3   1    16
## 617  02-May-2018_14:37:17  Exp 7303  28   male    R     3   1    17
## 618  02-May-2018_14:37:20  Exp 7303  28   male    R     3   1    18
## 619  02-May-2018_14:37:22  Exp 7303  28   male    R     3   1    19
## 620  02-May-2018_14:37:24  Exp 7303  28   male    R     3   1    20
## 621  02-May-2018_14:37:26  Exp 7303  28   male    R     3   1    21
## 622  02-May-2018_14:37:28  Exp 7303  28   male    R     3   1    22
## 623  02-May-2018_14:37:30  Exp 7303  28   male    R     3   1    23
## 624  02-May-2018_14:37:33  Exp 7303  28   male    R     3   1    24
## 625  02-May-2018_14:37:35  Exp 7303  28   male    R     3   2     1
## 626  02-May-2018_14:37:37  Exp 7303  28   male    R     3   2     2
## 627  02-May-2018_14:37:39  Exp 7303  28   male    R     3   2     3
## 628  02-May-2018_14:37:41  Exp 7303  28   male    R     3   2     4
## 629  02-May-2018_14:37:43  Exp 7303  28   male    R     3   2     5
## 630  02-May-2018_14:37:45  Exp 7303  28   male    R     3   2     6
## 631  02-May-2018_14:37:47  Exp 7303  28   male    R     3   2     7
## 632  02-May-2018_14:37:49  Exp 7303  28   male    R     3   2     8
## 633  02-May-2018_14:37:51  Exp 7303  28   male    R     3   2     9
## 634  02-May-2018_14:37:54  Exp 7303  28   male    R     3   2    10
## 635  02-May-2018_14:37:56  Exp 7303  28   male    R     3   2    11
## 636  02-May-2018_14:37:58  Exp 7303  28   male    R     3   2    12
## 637  02-May-2018_14:38:00  Exp 7303  28   male    R     3   2    13
## 638  02-May-2018_14:38:02  Exp 7303  28   male    R     3   2    14
## 639  02-May-2018_14:38:04  Exp 7303  28   male    R     3   2    15
## 640  02-May-2018_14:38:06  Exp 7303  28   male    R     3   2    16
## 641  02-May-2018_14:38:08  Exp 7303  28   male    R     3   2    17
## 642  02-May-2018_14:38:10  Exp 7303  28   male    R     3   2    18
## 643  02-May-2018_14:38:13  Exp 7303  28   male    R     3   2    19
## 644  02-May-2018_14:38:15  Exp 7303  28   male    R     3   2    20
## 645  02-May-2018_14:38:17  Exp 7303  28   male    R     3   2    21
## 646  02-May-2018_14:38:19  Exp 7303  28   male    R     3   2    22
## 647  02-May-2018_14:38:21  Exp 7303  28   male    R     3   2    23
## 648  02-May-2018_14:38:24  Exp 7303  28   male    R     3   2    24
## 649  02-May-2018_14:38:26  Exp 7303  28   male    R     3   3     1
## 650  02-May-2018_14:38:28  Exp 7303  28   male    R     3   3     2
## 651  02-May-2018_14:38:30  Exp 7303  28   male    R     3   3     3
## 652  02-May-2018_14:38:32  Exp 7303  28   male    R     3   3     4
## 653  02-May-2018_14:38:34  Exp 7303  28   male    R     3   3     5
## 654  02-May-2018_14:38:36  Exp 7303  28   male    R     3   3     6
## 655  02-May-2018_14:38:38  Exp 7303  28   male    R     3   3     7
## 656  02-May-2018_14:38:41  Exp 7303  28   male    R     3   3     8
## 657  02-May-2018_14:38:43  Exp 7303  28   male    R     3   3     9
## 658  02-May-2018_14:38:45  Exp 7303  28   male    R     3   3    10
## 659  02-May-2018_14:38:47  Exp 7303  28   male    R     3   3    11
## 660  02-May-2018_14:38:49  Exp 7303  28   male    R     3   3    12
## 661  02-May-2018_14:38:51  Exp 7303  28   male    R     3   3    13
## 662  02-May-2018_14:38:54  Exp 7303  28   male    R     3   3    14
## 663  02-May-2018_14:38:56  Exp 7303  28   male    R     3   3    15
## 664  02-May-2018_14:38:58  Exp 7303  28   male    R     3   3    16
## 665  02-May-2018_14:39:00  Exp 7303  28   male    R     3   3    17
## 666  02-May-2018_14:39:03  Exp 7303  28   male    R     3   3    18
## 667  02-May-2018_14:39:05  Exp 7303  28   male    R     3   3    19
## 668  02-May-2018_14:39:07  Exp 7303  28   male    R     3   3    20
## 669  02-May-2018_14:39:09  Exp 7303  28   male    R     3   3    21
## 670  02-May-2018_14:39:11  Exp 7303  28   male    R     3   3    22
## 671  02-May-2018_14:39:13  Exp 7303  28   male    R     3   3    23
## 672  02-May-2018_14:39:16  Exp 7303  28   male    R     3   3    24
## 673  02-May-2018_14:39:46  Exp 7303  28   male    R     3   4     1
## 674  02-May-2018_14:39:48  Exp 7303  28   male    R     3   4     2
## 675  02-May-2018_14:39:50  Exp 7303  28   male    R     3   4     3
## 676  02-May-2018_14:39:52  Exp 7303  28   male    R     3   4     4
## 677  02-May-2018_14:39:54  Exp 7303  28   male    R     3   4     5
## 678  02-May-2018_14:39:56  Exp 7303  28   male    R     3   4     6
## 679  02-May-2018_14:39:58  Exp 7303  28   male    R     3   4     7
## 680  02-May-2018_14:40:00  Exp 7303  28   male    R     3   4     8
## 681  02-May-2018_14:40:03  Exp 7303  28   male    R     3   4     9
## 682  02-May-2018_14:40:05  Exp 7303  28   male    R     3   4    10
## 683  02-May-2018_14:40:07  Exp 7303  28   male    R     3   4    11
## 684  02-May-2018_14:40:09  Exp 7303  28   male    R     3   4    12
## 685  02-May-2018_14:40:11  Exp 7303  28   male    R     3   4    13
## 686  02-May-2018_14:40:13  Exp 7303  28   male    R     3   4    14
## 687  02-May-2018_14:40:16  Exp 7303  28   male    R     3   4    15
## 688  02-May-2018_14:40:18  Exp 7303  28   male    R     3   4    16
## 689  02-May-2018_14:40:20  Exp 7303  28   male    R     3   4    17
## 690  02-May-2018_14:40:22  Exp 7303  28   male    R     3   4    18
## 691  02-May-2018_14:40:24  Exp 7303  28   male    R     3   4    19
## 692  02-May-2018_14:40:26  Exp 7303  28   male    R     3   4    20
## 693  02-May-2018_14:40:28  Exp 7303  28   male    R     3   4    21
## 694  02-May-2018_14:40:31  Exp 7303  28   male    R     3   4    22
## 695  02-May-2018_14:40:33  Exp 7303  28   male    R     3   4    23
## 696  02-May-2018_14:40:35  Exp 7303  28   male    R     3   4    24
## 697  02-May-2018_14:40:37  Exp 7303  28   male    R     3   5     1
## 698  02-May-2018_14:40:40  Exp 7303  28   male    R     3   5     2
## 699  02-May-2018_14:40:42  Exp 7303  28   male    R     3   5     3
## 700  02-May-2018_14:40:44  Exp 7303  28   male    R     3   5     4
## 701  02-May-2018_14:40:46  Exp 7303  28   male    R     3   5     5
## 702  02-May-2018_14:40:48  Exp 7303  28   male    R     3   5     6
## 703  02-May-2018_14:40:50  Exp 7303  28   male    R     3   5     7
## 704  02-May-2018_14:40:52  Exp 7303  28   male    R     3   5     8
## 705  02-May-2018_14:40:54  Exp 7303  28   male    R     3   5     9
## 706  02-May-2018_14:40:56  Exp 7303  28   male    R     3   5    10
## 707  02-May-2018_14:40:59  Exp 7303  28   male    R     3   5    11
## 708  02-May-2018_14:41:01  Exp 7303  28   male    R     3   5    12
## 709  02-May-2018_14:41:03  Exp 7303  28   male    R     3   5    13
## 710  02-May-2018_14:41:05  Exp 7303  28   male    R     3   5    14
## 711  02-May-2018_14:41:07  Exp 7303  28   male    R     3   5    15
## 712  02-May-2018_14:41:09  Exp 7303  28   male    R     3   5    16
## 713  02-May-2018_14:41:11  Exp 7303  28   male    R     3   5    17
## 714  02-May-2018_14:41:13  Exp 7303  28   male    R     3   5    18
## 715  02-May-2018_14:41:15  Exp 7303  28   male    R     3   5    19
## 716  02-May-2018_14:41:17  Exp 7303  28   male    R     3   5    20
## 717  02-May-2018_14:41:19  Exp 7303  28   male    R     3   5    21
## 718  02-May-2018_14:41:21  Exp 7303  28   male    R     3   5    22
## 719  02-May-2018_14:41:23  Exp 7303  28   male    R     3   5    23
## 720  02-May-2018_14:41:25  Exp 7303  28   male    R     3   5    24
## 721  06-May-2018_14:45:38  Exp 7304  25 female    R     1   1     1
## 722  06-May-2018_14:45:40  Exp 7304  25 female    R     1   1     2
## 723  06-May-2018_14:45:42  Exp 7304  25 female    R     1   1     3
## 724  06-May-2018_14:45:44  Exp 7304  25 female    R     1   1     4
## 725  06-May-2018_14:45:46  Exp 7304  25 female    R     1   1     5
## 726  06-May-2018_14:45:49  Exp 7304  25 female    R     1   1     6
## 727  06-May-2018_14:45:50  Exp 7304  25 female    R     1   1     7
## 728  06-May-2018_14:45:53  Exp 7304  25 female    R     1   1     8
## 729  06-May-2018_14:45:55  Exp 7304  25 female    R     1   1     9
## 730  06-May-2018_14:45:57  Exp 7304  25 female    R     1   1    10
## 731  06-May-2018_14:45:59  Exp 7304  25 female    R     1   1    11
## 732  06-May-2018_14:46:01  Exp 7304  25 female    R     1   1    12
## 733  06-May-2018_14:46:03  Exp 7304  25 female    R     1   1    13
## 734  06-May-2018_14:46:05  Exp 7304  25 female    R     1   1    14
## 735  06-May-2018_14:46:07  Exp 7304  25 female    R     1   1    15
## 736  06-May-2018_14:46:09  Exp 7304  25 female    R     1   1    16
## 737  06-May-2018_14:46:11  Exp 7304  25 female    R     1   1    17
## 738  06-May-2018_14:46:14  Exp 7304  25 female    R     1   1    18
## 739  06-May-2018_14:46:16  Exp 7304  25 female    R     1   1    19
## 740  06-May-2018_14:46:18  Exp 7304  25 female    R     1   1    20
## 741  06-May-2018_14:46:20  Exp 7304  25 female    R     1   1    21
## 742  06-May-2018_14:46:22  Exp 7304  25 female    R     1   1    22
## 743  06-May-2018_14:46:23  Exp 7304  25 female    R     1   1    23
## 744  06-May-2018_14:46:26  Exp 7304  25 female    R     1   1    24
## 745  06-May-2018_14:46:28  Exp 7304  25 female    R     1   2     1
## 746  06-May-2018_14:46:30  Exp 7304  25 female    R     1   2     2
## 747  06-May-2018_14:46:32  Exp 7304  25 female    R     1   2     3
## 748  06-May-2018_14:46:34  Exp 7304  25 female    R     1   2     4
## 749  06-May-2018_14:46:36  Exp 7304  25 female    R     1   2     5
## 750  06-May-2018_14:46:38  Exp 7304  25 female    R     1   2     6
## 751  06-May-2018_14:46:40  Exp 7304  25 female    R     1   2     7
## 752  06-May-2018_14:46:42  Exp 7304  25 female    R     1   2     8
## 753  06-May-2018_14:46:44  Exp 7304  25 female    R     1   2     9
## 754  06-May-2018_14:46:46  Exp 7304  25 female    R     1   2    10
## 755  06-May-2018_14:46:48  Exp 7304  25 female    R     1   2    11
## 756  06-May-2018_14:46:50  Exp 7304  25 female    R     1   2    12
## 757  06-May-2018_14:46:53  Exp 7304  25 female    R     1   2    13
## 758  06-May-2018_14:46:55  Exp 7304  25 female    R     1   2    14
## 759  06-May-2018_14:46:56  Exp 7304  25 female    R     1   2    15
## 760  06-May-2018_14:46:58  Exp 7304  25 female    R     1   2    16
## 761  06-May-2018_14:47:01  Exp 7304  25 female    R     1   2    17
## 762  06-May-2018_14:47:02  Exp 7304  25 female    R     1   2    18
## 763  06-May-2018_14:47:05  Exp 7304  25 female    R     1   2    19
## 764  06-May-2018_14:47:06  Exp 7304  25 female    R     1   2    20
## 765  06-May-2018_14:47:09  Exp 7304  25 female    R     1   2    21
## 766  06-May-2018_14:47:11  Exp 7304  25 female    R     1   2    22
## 767  06-May-2018_14:47:13  Exp 7304  25 female    R     1   2    23
## 768  06-May-2018_14:47:15  Exp 7304  25 female    R     1   2    24
## 769  06-May-2018_14:47:17  Exp 7304  25 female    R     1   3     1
## 770  06-May-2018_14:47:19  Exp 7304  25 female    R     1   3     2
## 771  06-May-2018_14:47:21  Exp 7304  25 female    R     1   3     3
## 772  06-May-2018_14:47:23  Exp 7304  25 female    R     1   3     4
## 773  06-May-2018_14:47:25  Exp 7304  25 female    R     1   3     5
## 774  06-May-2018_14:47:27  Exp 7304  25 female    R     1   3     6
## 775  06-May-2018_14:47:29  Exp 7304  25 female    R     1   3     7
## 776  06-May-2018_14:47:31  Exp 7304  25 female    R     1   3     8
## 777  06-May-2018_14:47:33  Exp 7304  25 female    R     1   3     9
## 778  06-May-2018_14:47:36  Exp 7304  25 female    R     1   3    10
## 779  06-May-2018_14:47:38  Exp 7304  25 female    R     1   3    11
## 780  06-May-2018_14:47:40  Exp 7304  25 female    R     1   3    12
## 781  06-May-2018_14:47:42  Exp 7304  25 female    R     1   3    13
## 782  06-May-2018_14:47:44  Exp 7304  25 female    R     1   3    14
## 783  06-May-2018_14:47:46  Exp 7304  25 female    R     1   3    15
## 784  06-May-2018_14:47:48  Exp 7304  25 female    R     1   3    16
## 785  06-May-2018_14:47:50  Exp 7304  25 female    R     1   3    17
## 786  06-May-2018_14:47:52  Exp 7304  25 female    R     1   3    18
## 787  06-May-2018_14:47:54  Exp 7304  25 female    R     1   3    19
## 788  06-May-2018_14:47:56  Exp 7304  25 female    R     1   3    20
## 789  06-May-2018_14:47:59  Exp 7304  25 female    R     1   3    21
## 790  06-May-2018_14:48:00  Exp 7304  25 female    R     1   3    22
## 791  06-May-2018_14:48:03  Exp 7304  25 female    R     1   3    23
## 792  06-May-2018_14:48:05  Exp 7304  25 female    R     1   3    24
## 793  06-May-2018_14:48:22  Exp 7304  25 female    R     1   4     1
## 794  06-May-2018_14:48:24  Exp 7304  25 female    R     1   4     2
## 795  06-May-2018_14:48:26  Exp 7304  25 female    R     1   4     3
## 796  06-May-2018_14:48:28  Exp 7304  25 female    R     1   4     4
## 797  06-May-2018_14:48:31  Exp 7304  25 female    R     1   4     5
## 798  06-May-2018_14:48:33  Exp 7304  25 female    R     1   4     6
## 799  06-May-2018_14:48:35  Exp 7304  25 female    R     1   4     7
## 800  06-May-2018_14:48:37  Exp 7304  25 female    R     1   4     8
## 801  06-May-2018_14:48:39  Exp 7304  25 female    R     1   4     9
## 802  06-May-2018_14:48:41  Exp 7304  25 female    R     1   4    10
## 803  06-May-2018_14:48:43  Exp 7304  25 female    R     1   4    11
## 804  06-May-2018_14:48:45  Exp 7304  25 female    R     1   4    12
## 805  06-May-2018_14:48:47  Exp 7304  25 female    R     1   4    13
## 806  06-May-2018_14:48:50  Exp 7304  25 female    R     1   4    14
## 807  06-May-2018_14:48:52  Exp 7304  25 female    R     1   4    15
## 808  06-May-2018_14:48:54  Exp 7304  25 female    R     1   4    16
## 809  06-May-2018_14:48:56  Exp 7304  25 female    R     1   4    17
## 810  06-May-2018_14:48:58  Exp 7304  25 female    R     1   4    18
## 811  06-May-2018_14:49:00  Exp 7304  25 female    R     1   4    19
## 812  06-May-2018_14:49:02  Exp 7304  25 female    R     1   4    20
## 813  06-May-2018_14:49:04  Exp 7304  25 female    R     1   4    21
## 814  06-May-2018_14:49:06  Exp 7304  25 female    R     1   4    22
## 815  06-May-2018_14:49:08  Exp 7304  25 female    R     1   4    23
## 816  06-May-2018_14:49:11  Exp 7304  25 female    R     1   4    24
## 817  06-May-2018_14:49:13  Exp 7304  25 female    R     1   5     1
## 818  06-May-2018_14:49:15  Exp 7304  25 female    R     1   5     2
## 819  06-May-2018_14:49:17  Exp 7304  25 female    R     1   5     3
## 820  06-May-2018_14:49:19  Exp 7304  25 female    R     1   5     4
## 821  06-May-2018_14:49:22  Exp 7304  25 female    R     1   5     5
## 822  06-May-2018_14:49:24  Exp 7304  25 female    R     1   5     6
## 823  06-May-2018_14:49:26  Exp 7304  25 female    R     1   5     7
## 824  06-May-2018_14:49:28  Exp 7304  25 female    R     1   5     8
## 825  06-May-2018_14:49:30  Exp 7304  25 female    R     1   5     9
## 826  06-May-2018_14:49:32  Exp 7304  25 female    R     1   5    10
## 827  06-May-2018_14:49:34  Exp 7304  25 female    R     1   5    11
## 828  06-May-2018_14:49:36  Exp 7304  25 female    R     1   5    12
## 829  06-May-2018_14:49:38  Exp 7304  25 female    R     1   5    13
## 830  06-May-2018_14:49:40  Exp 7304  25 female    R     1   5    14
## 831  06-May-2018_14:49:42  Exp 7304  25 female    R     1   5    15
## 832  06-May-2018_14:49:44  Exp 7304  25 female    R     1   5    16
## 833  06-May-2018_14:49:47  Exp 7304  25 female    R     1   5    17
## 834  06-May-2018_14:49:49  Exp 7304  25 female    R     1   5    18
## 835  06-May-2018_14:49:51  Exp 7304  25 female    R     1   5    19
## 836  06-May-2018_14:49:53  Exp 7304  25 female    R     1   5    20
## 837  06-May-2018_14:49:55  Exp 7304  25 female    R     1   5    21
## 838  06-May-2018_14:49:57  Exp 7304  25 female    R     1   5    22
## 839  06-May-2018_14:49:59  Exp 7304  25 female    R     1   5    23
## 840  06-May-2018_14:50:01  Exp 7304  25 female    R     1   5    24
## 841  06-May-2018_14:50:07  Exp 7304  25 female    R     2   1     1
## 842  06-May-2018_14:50:09  Exp 7304  25 female    R     2   1     2
## 843  06-May-2018_14:50:11  Exp 7304  25 female    R     2   1     3
## 844  06-May-2018_14:50:12  Exp 7304  25 female    R     2   1     4
## 845  06-May-2018_14:50:15  Exp 7304  25 female    R     2   1     5
## 846  06-May-2018_14:50:17  Exp 7304  25 female    R     2   1     6
## 847  06-May-2018_14:50:19  Exp 7304  25 female    R     2   1     7
## 848  06-May-2018_14:50:21  Exp 7304  25 female    R     2   1     8
## 849  06-May-2018_14:50:23  Exp 7304  25 female    R     2   1     9
## 850  06-May-2018_14:50:25  Exp 7304  25 female    R     2   1    10
## 851  06-May-2018_14:50:27  Exp 7304  25 female    R     2   1    11
## 852  06-May-2018_14:50:29  Exp 7304  25 female    R     2   1    12
## 853  06-May-2018_14:50:31  Exp 7304  25 female    R     2   1    13
## 854  06-May-2018_14:50:33  Exp 7304  25 female    R     2   1    14
## 855  06-May-2018_14:50:35  Exp 7304  25 female    R     2   1    15
## 856  06-May-2018_14:50:38  Exp 7304  25 female    R     2   1    16
## 857  06-May-2018_14:50:40  Exp 7304  25 female    R     2   1    17
## 858  06-May-2018_14:50:42  Exp 7304  25 female    R     2   1    18
## 859  06-May-2018_14:50:44  Exp 7304  25 female    R     2   1    19
## 860  06-May-2018_14:50:46  Exp 7304  25 female    R     2   1    20
## 861  06-May-2018_14:50:48  Exp 7304  25 female    R     2   1    21
## 862  06-May-2018_14:50:50  Exp 7304  25 female    R     2   1    22
## 863  06-May-2018_14:50:52  Exp 7304  25 female    R     2   1    23
## 864  06-May-2018_14:50:54  Exp 7304  25 female    R     2   1    24
## 865  06-May-2018_14:50:57  Exp 7304  25 female    R     2   2     1
## 866  06-May-2018_14:50:59  Exp 7304  25 female    R     2   2     2
## 867  06-May-2018_14:51:01  Exp 7304  25 female    R     2   2     3
## 868  06-May-2018_14:51:03  Exp 7304  25 female    R     2   2     4
## 869  06-May-2018_14:51:05  Exp 7304  25 female    R     2   2     5
## 870  06-May-2018_14:51:07  Exp 7304  25 female    R     2   2     6
## 871  06-May-2018_14:51:09  Exp 7304  25 female    R     2   2     7
## 872  06-May-2018_14:51:12  Exp 7304  25 female    R     2   2     8
## 873  06-May-2018_14:51:14  Exp 7304  25 female    R     2   2     9
## 874  06-May-2018_14:51:16  Exp 7304  25 female    R     2   2    10
## 875  06-May-2018_14:51:18  Exp 7304  25 female    R     2   2    11
## 876  06-May-2018_14:51:20  Exp 7304  25 female    R     2   2    12
## 877  06-May-2018_14:51:22  Exp 7304  25 female    R     2   2    13
## 878  06-May-2018_14:51:24  Exp 7304  25 female    R     2   2    14
## 879  06-May-2018_14:51:26  Exp 7304  25 female    R     2   2    15
## 880  06-May-2018_14:51:28  Exp 7304  25 female    R     2   2    16
## 881  06-May-2018_14:51:30  Exp 7304  25 female    R     2   2    17
## 882  06-May-2018_14:51:32  Exp 7304  25 female    R     2   2    18
## 883  06-May-2018_14:51:35  Exp 7304  25 female    R     2   2    19
## 884  06-May-2018_14:51:37  Exp 7304  25 female    R     2   2    20
## 885  06-May-2018_14:51:39  Exp 7304  25 female    R     2   2    21
## 886  06-May-2018_14:51:41  Exp 7304  25 female    R     2   2    22
## 887  06-May-2018_14:51:43  Exp 7304  25 female    R     2   2    23
## 888  06-May-2018_14:51:45  Exp 7304  25 female    R     2   2    24
## 889  06-May-2018_14:51:47  Exp 7304  25 female    R     2   3     1
## 890  06-May-2018_14:51:49  Exp 7304  25 female    R     2   3     2
## 891  06-May-2018_14:51:51  Exp 7304  25 female    R     2   3     3
## 892  06-May-2018_14:51:53  Exp 7304  25 female    R     2   3     4
## 893  06-May-2018_14:51:56  Exp 7304  25 female    R     2   3     5
## 894  06-May-2018_14:51:58  Exp 7304  25 female    R     2   3     6
## 895  06-May-2018_14:52:00  Exp 7304  25 female    R     2   3     7
## 896  06-May-2018_14:52:02  Exp 7304  25 female    R     2   3     8
## 897  06-May-2018_14:52:04  Exp 7304  25 female    R     2   3     9
## 898  06-May-2018_14:52:06  Exp 7304  25 female    R     2   3    10
## 899  06-May-2018_14:52:07  Exp 7304  25 female    R     2   3    11
## 900  06-May-2018_14:52:09  Exp 7304  25 female    R     2   3    12
## 901  06-May-2018_14:52:11  Exp 7304  25 female    R     2   3    13
## 902  06-May-2018_14:52:13  Exp 7304  25 female    R     2   3    14
## 903  06-May-2018_14:52:15  Exp 7304  25 female    R     2   3    15
## 904  06-May-2018_14:52:17  Exp 7304  25 female    R     2   3    16
## 905  06-May-2018_14:52:19  Exp 7304  25 female    R     2   3    17
## 906  06-May-2018_14:52:21  Exp 7304  25 female    R     2   3    18
## 907  06-May-2018_14:52:23  Exp 7304  25 female    R     2   3    19
## 908  06-May-2018_14:52:25  Exp 7304  25 female    R     2   3    20
## 909  06-May-2018_14:52:27  Exp 7304  25 female    R     2   3    21
## 910  06-May-2018_14:52:29  Exp 7304  25 female    R     2   3    22
## 911  06-May-2018_14:52:30  Exp 7304  25 female    R     2   3    23
## 912  06-May-2018_14:52:32  Exp 7304  25 female    R     2   3    24
## 913  06-May-2018_14:52:41  Exp 7304  25 female    R     2   4     1
## 914  06-May-2018_14:52:43  Exp 7304  25 female    R     2   4     2
## 915  06-May-2018_14:52:45  Exp 7304  25 female    R     2   4     3
## 916  06-May-2018_14:52:47  Exp 7304  25 female    R     2   4     4
## 917  06-May-2018_14:52:49  Exp 7304  25 female    R     2   4     5
## 918  06-May-2018_14:52:51  Exp 7304  25 female    R     2   4     6
## 919  06-May-2018_14:52:54  Exp 7304  25 female    R     2   4     7
## 920  06-May-2018_14:52:55  Exp 7304  25 female    R     2   4     8
## 921  06-May-2018_14:52:57  Exp 7304  25 female    R     2   4     9
## 922  06-May-2018_14:53:00  Exp 7304  25 female    R     2   4    10
## 923  06-May-2018_14:53:01  Exp 7304  25 female    R     2   4    11
## 924  06-May-2018_14:53:03  Exp 7304  25 female    R     2   4    12
## 925  06-May-2018_14:53:05  Exp 7304  25 female    R     2   4    13
## 926  06-May-2018_14:53:07  Exp 7304  25 female    R     2   4    14
## 927  06-May-2018_14:53:09  Exp 7304  25 female    R     2   4    15
## 928  06-May-2018_14:53:11  Exp 7304  25 female    R     2   4    16
## 929  06-May-2018_14:53:13  Exp 7304  25 female    R     2   4    17
## 930  06-May-2018_14:53:15  Exp 7304  25 female    R     2   4    18
## 931  06-May-2018_14:53:17  Exp 7304  25 female    R     2   4    19
## 932  06-May-2018_14:53:18  Exp 7304  25 female    R     2   4    20
## 933  06-May-2018_14:53:20  Exp 7304  25 female    R     2   4    21
## 934  06-May-2018_14:53:22  Exp 7304  25 female    R     2   4    22
## 935  06-May-2018_14:53:24  Exp 7304  25 female    R     2   4    23
## 936  06-May-2018_14:53:26  Exp 7304  25 female    R     2   4    24
## 937  06-May-2018_14:53:28  Exp 7304  25 female    R     2   5     1
## 938  06-May-2018_14:53:30  Exp 7304  25 female    R     2   5     2
## 939  06-May-2018_14:53:32  Exp 7304  25 female    R     2   5     3
## 940  06-May-2018_14:53:34  Exp 7304  25 female    R     2   5     4
## 941  06-May-2018_14:53:36  Exp 7304  25 female    R     2   5     5
## 942  06-May-2018_14:53:37  Exp 7304  25 female    R     2   5     6
## 943  06-May-2018_14:53:39  Exp 7304  25 female    R     2   5     7
## 944  06-May-2018_14:53:40  Exp 7304  25 female    R     2   5     8
## 945  06-May-2018_14:53:43  Exp 7304  25 female    R     2   5     9
## 946  06-May-2018_14:53:45  Exp 7304  25 female    R     2   5    10
## 947  06-May-2018_14:53:47  Exp 7304  25 female    R     2   5    11
## 948  06-May-2018_14:53:49  Exp 7304  25 female    R     2   5    12
## 949  06-May-2018_14:53:51  Exp 7304  25 female    R     2   5    13
## 950  06-May-2018_14:53:53  Exp 7304  25 female    R     2   5    14
## 951  06-May-2018_14:53:55  Exp 7304  25 female    R     2   5    15
## 952  06-May-2018_14:53:57  Exp 7304  25 female    R     2   5    16
## 953  06-May-2018_14:53:59  Exp 7304  25 female    R     2   5    17
## 954  06-May-2018_14:54:01  Exp 7304  25 female    R     2   5    18
## 955  06-May-2018_14:54:03  Exp 7304  25 female    R     2   5    19
## 956  06-May-2018_14:54:05  Exp 7304  25 female    R     2   5    20
## 957  06-May-2018_14:54:07  Exp 7304  25 female    R     2   5    21
## 958  06-May-2018_14:54:09  Exp 7304  25 female    R     2   5    22
## 959  06-May-2018_14:54:11  Exp 7304  25 female    R     2   5    23
## 960  06-May-2018_14:54:14  Exp 7304  25 female    R     2   5    24
## 961  06-May-2018_14:54:19  Exp 7304  25 female    R     3   1     1
## 962  06-May-2018_14:54:21  Exp 7304  25 female    R     3   1     2
## 963  06-May-2018_14:54:22  Exp 7304  25 female    R     3   1     3
## 964  06-May-2018_14:54:24  Exp 7304  25 female    R     3   1     4
## 965  06-May-2018_14:54:27  Exp 7304  25 female    R     3   1     5
## 966  06-May-2018_14:54:29  Exp 7304  25 female    R     3   1     6
## 967  06-May-2018_14:54:31  Exp 7304  25 female    R     3   1     7
## 968  06-May-2018_14:54:33  Exp 7304  25 female    R     3   1     8
## 969  06-May-2018_14:54:35  Exp 7304  25 female    R     3   1     9
## 970  06-May-2018_14:54:37  Exp 7304  25 female    R     3   1    10
## 971  06-May-2018_14:54:39  Exp 7304  25 female    R     3   1    11
## 972  06-May-2018_14:54:41  Exp 7304  25 female    R     3   1    12
## 973  06-May-2018_14:54:43  Exp 7304  25 female    R     3   1    13
## 974  06-May-2018_14:54:46  Exp 7304  25 female    R     3   1    14
## 975  06-May-2018_14:54:48  Exp 7304  25 female    R     3   1    15
## 976  06-May-2018_14:54:50  Exp 7304  25 female    R     3   1    16
## 977  06-May-2018_14:54:52  Exp 7304  25 female    R     3   1    17
## 978  06-May-2018_14:54:54  Exp 7304  25 female    R     3   1    18
## 979  06-May-2018_14:54:56  Exp 7304  25 female    R     3   1    19
## 980  06-May-2018_14:54:59  Exp 7304  25 female    R     3   1    20
## 981  06-May-2018_14:55:01  Exp 7304  25 female    R     3   1    21
## 982  06-May-2018_14:55:03  Exp 7304  25 female    R     3   1    22
## 983  06-May-2018_14:55:05  Exp 7304  25 female    R     3   1    23
## 984  06-May-2018_14:55:07  Exp 7304  25 female    R     3   1    24
## 985  06-May-2018_14:55:09  Exp 7304  25 female    R     3   2     1
## 986  06-May-2018_14:55:11  Exp 7304  25 female    R     3   2     2
## 987  06-May-2018_14:55:14  Exp 7304  25 female    R     3   2     3
## 988  06-May-2018_14:55:16  Exp 7304  25 female    R     3   2     4
## 989  06-May-2018_14:55:18  Exp 7304  25 female    R     3   2     5
## 990  06-May-2018_14:55:20  Exp 7304  25 female    R     3   2     6
## 991  06-May-2018_14:55:22  Exp 7304  25 female    R     3   2     7
## 992  06-May-2018_14:55:24  Exp 7304  25 female    R     3   2     8
## 993  06-May-2018_14:55:26  Exp 7304  25 female    R     3   2     9
## 994  06-May-2018_14:55:28  Exp 7304  25 female    R     3   2    10
## 995  06-May-2018_14:55:30  Exp 7304  25 female    R     3   2    11
## 996  06-May-2018_14:55:32  Exp 7304  25 female    R     3   2    12
## 997  06-May-2018_14:55:34  Exp 7304  25 female    R     3   2    13
## 998  06-May-2018_14:55:36  Exp 7304  25 female    R     3   2    14
## 999  06-May-2018_14:55:38  Exp 7304  25 female    R     3   2    15
## 1000 06-May-2018_14:55:40  Exp 7304  25 female    R     3   2    16
## 1001 06-May-2018_14:55:42  Exp 7304  25 female    R     3   2    17
## 1002 06-May-2018_14:55:44  Exp 7304  25 female    R     3   2    18
## 1003 06-May-2018_14:55:47  Exp 7304  25 female    R     3   2    19
## 1004 06-May-2018_14:55:49  Exp 7304  25 female    R     3   2    20
## 1005 06-May-2018_14:55:51  Exp 7304  25 female    R     3   2    21
## 1006 06-May-2018_14:55:53  Exp 7304  25 female    R     3   2    22
## 1007 06-May-2018_14:55:55  Exp 7304  25 female    R     3   2    23
## 1008 06-May-2018_14:55:57  Exp 7304  25 female    R     3   2    24
## 1009 06-May-2018_14:55:59  Exp 7304  25 female    R     3   3     1
## 1010 06-May-2018_14:56:01  Exp 7304  25 female    R     3   3     2
## 1011 06-May-2018_14:56:03  Exp 7304  25 female    R     3   3     3
## 1012 06-May-2018_14:56:05  Exp 7304  25 female    R     3   3     4
## 1013 06-May-2018_14:56:07  Exp 7304  25 female    R     3   3     5
## 1014 06-May-2018_14:56:09  Exp 7304  25 female    R     3   3     6
## 1015 06-May-2018_14:56:11  Exp 7304  25 female    R     3   3     7
## 1016 06-May-2018_14:56:13  Exp 7304  25 female    R     3   3     8
## 1017 06-May-2018_14:56:15  Exp 7304  25 female    R     3   3     9
## 1018 06-May-2018_14:56:18  Exp 7304  25 female    R     3   3    10
## 1019 06-May-2018_14:56:20  Exp 7304  25 female    R     3   3    11
## 1020 06-May-2018_14:56:22  Exp 7304  25 female    R     3   3    12
## 1021 06-May-2018_14:56:24  Exp 7304  25 female    R     3   3    13
## 1022 06-May-2018_14:56:26  Exp 7304  25 female    R     3   3    14
## 1023 06-May-2018_14:56:28  Exp 7304  25 female    R     3   3    15
## 1024 06-May-2018_14:56:30  Exp 7304  25 female    R     3   3    16
## 1025 06-May-2018_14:56:33  Exp 7304  25 female    R     3   3    17
## 1026 06-May-2018_14:56:35  Exp 7304  25 female    R     3   3    18
## 1027 06-May-2018_14:56:37  Exp 7304  25 female    R     3   3    19
## 1028 06-May-2018_14:56:39  Exp 7304  25 female    R     3   3    20
## 1029 06-May-2018_14:56:41  Exp 7304  25 female    R     3   3    21
## 1030 06-May-2018_14:56:43  Exp 7304  25 female    R     3   3    22
## 1031 06-May-2018_14:56:45  Exp 7304  25 female    R     3   3    23
## 1032 06-May-2018_14:56:48  Exp 7304  25 female    R     3   3    24
## 1033 06-May-2018_14:57:04  Exp 7304  25 female    R     3   4     1
## 1034 06-May-2018_14:57:06  Exp 7304  25 female    R     3   4     2
## 1035 06-May-2018_14:57:08  Exp 7304  25 female    R     3   4     3
## 1036 06-May-2018_14:57:10  Exp 7304  25 female    R     3   4     4
## 1037 06-May-2018_14:57:13  Exp 7304  25 female    R     3   4     5
## 1038 06-May-2018_14:57:15  Exp 7304  25 female    R     3   4     6
## 1039 06-May-2018_14:57:17  Exp 7304  25 female    R     3   4     7
## 1040 06-May-2018_14:57:19  Exp 7304  25 female    R     3   4     8
## 1041 06-May-2018_14:57:21  Exp 7304  25 female    R     3   4     9
## 1042 06-May-2018_14:57:24  Exp 7304  25 female    R     3   4    10
## 1043 06-May-2018_14:57:26  Exp 7304  25 female    R     3   4    11
## 1044 06-May-2018_14:57:28  Exp 7304  25 female    R     3   4    12
## 1045 06-May-2018_14:57:30  Exp 7304  25 female    R     3   4    13
## 1046 06-May-2018_14:57:32  Exp 7304  25 female    R     3   4    14
## 1047 06-May-2018_14:57:34  Exp 7304  25 female    R     3   4    15
## 1048 06-May-2018_14:57:36  Exp 7304  25 female    R     3   4    16
## 1049 06-May-2018_14:57:39  Exp 7304  25 female    R     3   4    17
## 1050 06-May-2018_14:57:40  Exp 7304  25 female    R     3   4    18
## 1051 06-May-2018_14:57:43  Exp 7304  25 female    R     3   4    19
## 1052 06-May-2018_14:57:45  Exp 7304  25 female    R     3   4    20
## 1053 06-May-2018_14:57:47  Exp 7304  25 female    R     3   4    21
## 1054 06-May-2018_14:57:49  Exp 7304  25 female    R     3   4    22
## 1055 06-May-2018_14:57:51  Exp 7304  25 female    R     3   4    23
## 1056 06-May-2018_14:57:53  Exp 7304  25 female    R     3   4    24
## 1057 06-May-2018_14:57:55  Exp 7304  25 female    R     3   5     1
## 1058 06-May-2018_14:57:57  Exp 7304  25 female    R     3   5     2
## 1059 06-May-2018_14:57:59  Exp 7304  25 female    R     3   5     3
## 1060 06-May-2018_14:58:01  Exp 7304  25 female    R     3   5     4
## 1061 06-May-2018_14:58:03  Exp 7304  25 female    R     3   5     5
## 1062 06-May-2018_14:58:05  Exp 7304  25 female    R     3   5     6
## 1063 06-May-2018_14:58:07  Exp 7304  25 female    R     3   5     7
## 1064 06-May-2018_14:58:09  Exp 7304  25 female    R     3   5     8
## 1065 06-May-2018_14:58:11  Exp 7304  25 female    R     3   5     9
## 1066 06-May-2018_14:58:13  Exp 7304  25 female    R     3   5    10
## 1067 06-May-2018_14:58:15  Exp 7304  25 female    R     3   5    11
## 1068 06-May-2018_14:58:17  Exp 7304  25 female    R     3   5    12
## 1069 06-May-2018_14:58:19  Exp 7304  25 female    R     3   5    13
## 1070 06-May-2018_14:58:21  Exp 7304  25 female    R     3   5    14
## 1071 06-May-2018_14:58:24  Exp 7304  25 female    R     3   5    15
## 1072 06-May-2018_14:58:26  Exp 7304  25 female    R     3   5    16
## 1073 06-May-2018_14:58:28  Exp 7304  25 female    R     3   5    17
## 1074 06-May-2018_14:58:30  Exp 7304  25 female    R     3   5    18
## 1075 06-May-2018_14:58:32  Exp 7304  25 female    R     3   5    19
## 1076 06-May-2018_14:58:34  Exp 7304  25 female    R     3   5    20
## 1077 06-May-2018_14:58:36  Exp 7304  25 female    R     3   5    21
## 1078 06-May-2018_14:58:38  Exp 7304  25 female    R     3   5    22
## 1079 06-May-2018_14:58:40  Exp 7304  25 female    R     3   5    23
## 1080 06-May-2018_14:58:43  Exp 7304  25 female    R     3   5    24
## 1081 06-May-2018_15:03:22  Exp 7304  25 female    R     1   1     1
## 1082 06-May-2018_15:03:23  Exp 7304  25 female    R     1   1     2
## 1083 06-May-2018_15:03:25  Exp 7304  25 female    R     1   1     3
## 1084 06-May-2018_15:03:27  Exp 7304  25 female    R     1   1     4
## 1085 06-May-2018_15:03:29  Exp 7304  25 female    R     1   1     5
## 1086 06-May-2018_15:03:30  Exp 7304  25 female    R     1   1     6
## 1087 06-May-2018_15:03:32  Exp 7304  25 female    R     1   1     7
## 1088 06-May-2018_15:03:34  Exp 7304  25 female    R     1   1     8
## 1089 06-May-2018_15:03:36  Exp 7304  25 female    R     1   1     9
## 1090 06-May-2018_15:03:38  Exp 7304  25 female    R     1   1    10
## 1091 06-May-2018_15:03:41  Exp 7304  25 female    R     1   1    11
## 1092 06-May-2018_15:03:43  Exp 7304  25 female    R     1   1    12
## 1093 06-May-2018_15:03:45  Exp 7304  25 female    R     1   1    13
## 1094 06-May-2018_15:03:47  Exp 7304  25 female    R     1   1    14
## 1095 06-May-2018_15:03:49  Exp 7304  25 female    R     1   1    15
## 1096 06-May-2018_15:03:51  Exp 7304  25 female    R     1   1    16
## 1097 06-May-2018_15:03:53  Exp 7304  25 female    R     1   1    17
## 1098 06-May-2018_15:03:55  Exp 7304  25 female    R     1   1    18
## 1099 06-May-2018_15:03:57  Exp 7304  25 female    R     1   1    19
## 1100 06-May-2018_15:03:59  Exp 7304  25 female    R     1   1    20
## 1101 06-May-2018_15:04:01  Exp 7304  25 female    R     1   1    21
## 1102 06-May-2018_15:04:03  Exp 7304  25 female    R     1   1    22
## 1103 06-May-2018_15:04:05  Exp 7304  25 female    R     1   1    23
## 1104 06-May-2018_15:04:08  Exp 7304  25 female    R     1   1    24
## 1105 06-May-2018_15:04:10  Exp 7304  25 female    R     1   2     1
## 1106 06-May-2018_15:04:12  Exp 7304  25 female    R     1   2     2
## 1107 06-May-2018_15:04:14  Exp 7304  25 female    R     1   2     3
## 1108 06-May-2018_15:04:16  Exp 7304  25 female    R     1   2     4
## 1109 06-May-2018_15:04:18  Exp 7304  25 female    R     1   2     5
## 1110 06-May-2018_15:04:20  Exp 7304  25 female    R     1   2     6
## 1111 06-May-2018_15:04:22  Exp 7304  25 female    R     1   2     7
## 1112 06-May-2018_15:04:24  Exp 7304  25 female    R     1   2     8
## 1113 06-May-2018_15:04:26  Exp 7304  25 female    R     1   2     9
## 1114 06-May-2018_15:04:28  Exp 7304  25 female    R     1   2    10
## 1115 06-May-2018_15:04:30  Exp 7304  25 female    R     1   2    11
## 1116 06-May-2018_15:04:32  Exp 7304  25 female    R     1   2    12
## 1117 06-May-2018_15:04:34  Exp 7304  25 female    R     1   2    13
## 1118 06-May-2018_15:04:36  Exp 7304  25 female    R     1   2    14
## 1119 06-May-2018_15:04:39  Exp 7304  25 female    R     1   2    15
## 1120 06-May-2018_15:04:41  Exp 7304  25 female    R     1   2    16
## 1121 06-May-2018_15:04:43  Exp 7304  25 female    R     1   2    17
## 1122 06-May-2018_15:04:45  Exp 7304  25 female    R     1   2    18
## 1123 06-May-2018_15:04:47  Exp 7304  25 female    R     1   2    19
## 1124 06-May-2018_15:04:50  Exp 7304  25 female    R     1   2    20
## 1125 06-May-2018_15:04:52  Exp 7304  25 female    R     1   2    21
## 1126 06-May-2018_15:04:54  Exp 7304  25 female    R     1   2    22
## 1127 06-May-2018_15:04:56  Exp 7304  25 female    R     1   2    23
## 1128 06-May-2018_15:04:58  Exp 7304  25 female    R     1   2    24
## 1129 06-May-2018_15:09:22  Exp 7304  25 female    R     1   1     1
## 1130 06-May-2018_15:09:25  Exp 7304  25 female    R     1   1     2
## 1131 06-May-2018_15:09:27  Exp 7304  25 female    R     1   1     3
## 1132 06-May-2018_15:09:29  Exp 7304  25 female    R     1   1     4
## 1133 06-May-2018_15:09:31  Exp 7304  25 female    R     1   1     5
## 1134 06-May-2018_15:09:33  Exp 7304  25 female    R     1   1     6
## 1135 06-May-2018_15:09:35  Exp 7304  25 female    R     1   1     7
## 1136 06-May-2018_15:09:38  Exp 7304  25 female    R     1   1     8
## 1137 06-May-2018_15:09:40  Exp 7304  25 female    R     1   1     9
## 1138 06-May-2018_15:09:42  Exp 7304  25 female    R     1   1    10
## 1139 06-May-2018_15:09:44  Exp 7304  25 female    R     1   1    11
## 1140 06-May-2018_15:09:46  Exp 7304  25 female    R     1   1    12
## 1141 06-May-2018_15:09:48  Exp 7304  25 female    R     1   1    13
## 1142 06-May-2018_15:09:50  Exp 7304  25 female    R     1   1    14
## 1143 06-May-2018_15:09:52  Exp 7304  25 female    R     1   1    15
## 1144 06-May-2018_15:09:54  Exp 7304  25 female    R     1   1    16
## 1145 06-May-2018_15:09:56  Exp 7304  25 female    R     1   1    17
## 1146 06-May-2018_15:09:59  Exp 7304  25 female    R     1   1    18
## 1147 06-May-2018_15:10:01  Exp 7304  25 female    R     1   1    19
## 1148 06-May-2018_15:10:03  Exp 7304  25 female    R     1   1    20
## 1149 06-May-2018_15:10:05  Exp 7304  25 female    R     1   1    21
## 1150 06-May-2018_15:10:07  Exp 7304  25 female    R     1   1    22
## 1151 06-May-2018_15:10:09  Exp 7304  25 female    R     1   1    23
## 1152 06-May-2018_15:10:11  Exp 7304  25 female    R     1   1    24
## 1153 06-May-2018_15:10:13  Exp 7304  25 female    R     1   2     1
## 1154 06-May-2018_15:10:15  Exp 7304  25 female    R     1   2     2
## 1155 06-May-2018_15:10:17  Exp 7304  25 female    R     1   2     3
## 1156 06-May-2018_15:10:19  Exp 7304  25 female    R     1   2     4
## 1157 06-May-2018_15:10:21  Exp 7304  25 female    R     1   2     5
## 1158 06-May-2018_15:10:23  Exp 7304  25 female    R     1   2     6
## 1159 06-May-2018_15:10:25  Exp 7304  25 female    R     1   2     7
## 1160 06-May-2018_15:10:28  Exp 7304  25 female    R     1   2     8
## 1161 06-May-2018_15:10:30  Exp 7304  25 female    R     1   2     9
## 1162 06-May-2018_15:10:32  Exp 7304  25 female    R     1   2    10
## 1163 06-May-2018_15:10:34  Exp 7304  25 female    R     1   2    11
## 1164 06-May-2018_15:10:36  Exp 7304  25 female    R     1   2    12
## 1165 06-May-2018_15:10:38  Exp 7304  25 female    R     1   2    13
## 1166 06-May-2018_15:10:41  Exp 7304  25 female    R     1   2    14
## 1167 06-May-2018_15:10:43  Exp 7304  25 female    R     1   2    15
## 1168 06-May-2018_15:10:45  Exp 7304  25 female    R     1   2    16
## 1169 06-May-2018_15:10:47  Exp 7304  25 female    R     1   2    17
## 1170 06-May-2018_15:10:49  Exp 7304  25 female    R     1   2    18
## 1171 06-May-2018_15:10:51  Exp 7304  25 female    R     1   2    19
## 1172 06-May-2018_15:10:53  Exp 7304  25 female    R     1   2    20
## 1173 06-May-2018_15:10:55  Exp 7304  25 female    R     1   2    21
## 1174 06-May-2018_15:10:57  Exp 7304  25 female    R     1   2    22
## 1175 06-May-2018_15:10:59  Exp 7304  25 female    R     1   2    23
## 1176 06-May-2018_15:11:01  Exp 7304  25 female    R     1   2    24
## 1177 06-May-2018_15:14:59  Exp 7304  25 female    R     1   1     1
## 1178 06-May-2018_15:15:01  Exp 7304  25 female    R     1   1     2
## 1179 06-May-2018_15:15:03  Exp 7304  25 female    R     1   1     3
## 1180 06-May-2018_15:15:06  Exp 7304  25 female    R     1   1     4
## 1181 06-May-2018_15:15:08  Exp 7304  25 female    R     1   1     5
## 1182 06-May-2018_15:15:10  Exp 7304  25 female    R     1   1     6
## 1183 06-May-2018_15:15:12  Exp 7304  25 female    R     1   1     7
## 1184 06-May-2018_15:15:14  Exp 7304  25 female    R     1   1     8
## 1185 06-May-2018_15:15:16  Exp 7304  25 female    R     1   1     9
## 1186 06-May-2018_15:15:18  Exp 7304  25 female    R     1   1    10
## 1187 06-May-2018_15:15:20  Exp 7304  25 female    R     1   1    11
## 1188 06-May-2018_15:15:22  Exp 7304  25 female    R     1   1    12
## 1189 06-May-2018_15:15:24  Exp 7304  25 female    R     1   1    13
## 1190 06-May-2018_15:15:26  Exp 7304  25 female    R     1   1    14
## 1191 06-May-2018_15:15:28  Exp 7304  25 female    R     1   1    15
## 1192 06-May-2018_15:15:30  Exp 7304  25 female    R     1   1    16
## 1193 06-May-2018_15:15:32  Exp 7304  25 female    R     1   1    17
## 1194 06-May-2018_15:15:34  Exp 7304  25 female    R     1   1    18
## 1195 06-May-2018_15:15:37  Exp 7304  25 female    R     1   1    19
## 1196 06-May-2018_15:15:39  Exp 7304  25 female    R     1   1    20
## 1197 06-May-2018_15:15:41  Exp 7304  25 female    R     1   1    21
## 1198 06-May-2018_15:15:43  Exp 7304  25 female    R     1   1    22
## 1199 06-May-2018_15:15:45  Exp 7304  25 female    R     1   1    23
## 1200 06-May-2018_15:15:47  Exp 7304  25 female    R     1   1    24
## 1201 06-May-2018_15:15:49  Exp 7304  25 female    R     1   2     1
## 1202 06-May-2018_15:15:51  Exp 7304  25 female    R     1   2     2
## 1203 06-May-2018_15:15:53  Exp 7304  25 female    R     1   2     3
## 1204 06-May-2018_15:15:55  Exp 7304  25 female    R     1   2     4
## 1205 06-May-2018_15:15:57  Exp 7304  25 female    R     1   2     5
## 1206 06-May-2018_15:16:00  Exp 7304  25 female    R     1   2     6
## 1207 06-May-2018_15:16:02  Exp 7304  25 female    R     1   2     7
## 1208 06-May-2018_15:16:04  Exp 7304  25 female    R     1   2     8
## 1209 06-May-2018_15:16:06  Exp 7304  25 female    R     1   2     9
## 1210 06-May-2018_15:16:07  Exp 7304  25 female    R     1   2    10
## 1211 06-May-2018_15:16:10  Exp 7304  25 female    R     1   2    11
## 1212 06-May-2018_15:16:12  Exp 7304  25 female    R     1   2    12
## 1213 06-May-2018_15:16:14  Exp 7304  25 female    R     1   2    13
## 1214 06-May-2018_15:16:16  Exp 7304  25 female    R     1   2    14
## 1215 06-May-2018_15:16:19  Exp 7304  25 female    R     1   2    15
## 1216 06-May-2018_15:16:21  Exp 7304  25 female    R     1   2    16
## 1217 06-May-2018_15:16:23  Exp 7304  25 female    R     1   2    17
## 1218 06-May-2018_15:16:25  Exp 7304  25 female    R     1   2    18
## 1219 06-May-2018_15:16:27  Exp 7304  25 female    R     1   2    19
## 1220 06-May-2018_15:16:29  Exp 7304  25 female    R     1   2    20
## 1221 06-May-2018_15:16:31  Exp 7304  25 female    R     1   2    21
## 1222 06-May-2018_15:16:33  Exp 7304  25 female    R     1   2    22
## 1223 06-May-2018_15:16:36  Exp 7304  25 female    R     1   2    23
## 1224 06-May-2018_15:16:38  Exp 7304  25 female    R     1   2    24
## 1225 06-May-2018_15:20:37  Exp 7304  25 female    R     1   1     1
## 1226 06-May-2018_15:20:39  Exp 7304  25 female    R     1   1     2
## 1227 06-May-2018_15:20:41  Exp 7304  25 female    R     1   1     3
## 1228 06-May-2018_15:20:44  Exp 7304  25 female    R     1   1     4
## 1229 06-May-2018_15:20:46  Exp 7304  25 female    R     1   1     5
## 1230 06-May-2018_15:20:48  Exp 7304  25 female    R     1   1     6
## 1231 06-May-2018_15:20:50  Exp 7304  25 female    R     1   1     7
## 1232 06-May-2018_15:20:52  Exp 7304  25 female    R     1   1     8
## 1233 06-May-2018_15:20:54  Exp 7304  25 female    R     1   1     9
## 1234 06-May-2018_15:20:56  Exp 7304  25 female    R     1   1    10
## 1235 06-May-2018_15:20:58  Exp 7304  25 female    R     1   1    11
## 1236 06-May-2018_15:21:00  Exp 7304  25 female    R     1   1    12
## 1237 06-May-2018_15:21:02  Exp 7304  25 female    R     1   1    13
## 1238 06-May-2018_15:21:04  Exp 7304  25 female    R     1   1    14
## 1239 06-May-2018_15:21:06  Exp 7304  25 female    R     1   1    15
## 1240 06-May-2018_15:21:08  Exp 7304  25 female    R     1   1    16
## 1241 06-May-2018_15:21:10  Exp 7304  25 female    R     1   1    17
## 1242 06-May-2018_15:21:12  Exp 7304  25 female    R     1   1    18
## 1243 06-May-2018_15:21:14  Exp 7304  25 female    R     1   1    19
## 1244 06-May-2018_15:21:16  Exp 7304  25 female    R     1   1    20
## 1245 06-May-2018_15:21:18  Exp 7304  25 female    R     1   1    21
## 1246 06-May-2018_15:21:20  Exp 7304  25 female    R     1   1    22
## 1247 06-May-2018_15:21:22  Exp 7304  25 female    R     1   1    23
## 1248 06-May-2018_15:21:24  Exp 7304  25 female    R     1   1    24
## 1249 06-May-2018_15:21:26  Exp 7304  25 female    R     1   2     1
## 1250 06-May-2018_15:21:28  Exp 7304  25 female    R     1   2     2
## 1251 06-May-2018_15:21:30  Exp 7304  25 female    R     1   2     3
## 1252 06-May-2018_15:21:32  Exp 7304  25 female    R     1   2     4
## 1253 06-May-2018_15:21:34  Exp 7304  25 female    R     1   2     5
## 1254 06-May-2018_15:21:36  Exp 7304  25 female    R     1   2     6
## 1255 06-May-2018_15:21:38  Exp 7304  25 female    R     1   2     7
## 1256 06-May-2018_15:21:40  Exp 7304  25 female    R     1   2     8
## 1257 06-May-2018_15:21:42  Exp 7304  25 female    R     1   2     9
## 1258 06-May-2018_15:21:44  Exp 7304  25 female    R     1   2    10
## 1259 06-May-2018_15:21:46  Exp 7304  25 female    R     1   2    11
## 1260 06-May-2018_15:21:48  Exp 7304  25 female    R     1   2    12
## 1261 06-May-2018_15:21:50  Exp 7304  25 female    R     1   2    13
## 1262 06-May-2018_15:21:52  Exp 7304  25 female    R     1   2    14
## 1263 06-May-2018_15:21:54  Exp 7304  25 female    R     1   2    15
## 1264 06-May-2018_15:21:56  Exp 7304  25 female    R     1   2    16
## 1265 06-May-2018_15:21:58  Exp 7304  25 female    R     1   2    17
## 1266 06-May-2018_15:22:00  Exp 7304  25 female    R     1   2    18
## 1267 06-May-2018_15:22:02  Exp 7304  25 female    R     1   2    19
## 1268 06-May-2018_15:22:04  Exp 7304  25 female    R     1   2    20
## 1269 06-May-2018_15:22:06  Exp 7304  25 female    R     1   2    21
## 1270 06-May-2018_15:22:08  Exp 7304  25 female    R     1   2    22
## 1271 06-May-2018_15:22:10  Exp 7304  25 female    R     1   2    23
## 1272 06-May-2018_15:22:12  Exp 7304  25 female    R     1   2    24
## 1273 06-May-2018_15:26:07  Exp 7304  25 female    R     1   1     1
## 1274 06-May-2018_15:26:09  Exp 7304  25 female    R     1   1     2
## 1275 06-May-2018_15:26:11  Exp 7304  25 female    R     1   1     3
## 1276 06-May-2018_15:26:13  Exp 7304  25 female    R     1   1     4
## 1277 06-May-2018_15:26:15  Exp 7304  25 female    R     1   1     5
## 1278 06-May-2018_15:26:17  Exp 7304  25 female    R     1   1     6
## 1279 06-May-2018_15:26:19  Exp 7304  25 female    R     1   1     7
## 1280 06-May-2018_15:26:21  Exp 7304  25 female    R     1   1     8
## 1281 06-May-2018_15:26:23  Exp 7304  25 female    R     1   1     9
## 1282 06-May-2018_15:26:25  Exp 7304  25 female    R     1   1    10
## 1283 06-May-2018_15:26:27  Exp 7304  25 female    R     1   1    11
## 1284 06-May-2018_15:26:29  Exp 7304  25 female    R     1   1    12
## 1285 06-May-2018_15:26:31  Exp 7304  25 female    R     1   1    13
## 1286 06-May-2018_15:26:33  Exp 7304  25 female    R     1   1    14
## 1287 06-May-2018_15:26:35  Exp 7304  25 female    R     1   1    15
## 1288 06-May-2018_15:26:37  Exp 7304  25 female    R     1   1    16
## 1289 06-May-2018_15:26:39  Exp 7304  25 female    R     1   1    17
## 1290 06-May-2018_15:26:42  Exp 7304  25 female    R     1   1    18
## 1291 06-May-2018_15:26:44  Exp 7304  25 female    R     1   1    19
## 1292 06-May-2018_15:26:46  Exp 7304  25 female    R     1   1    20
## 1293 06-May-2018_15:26:48  Exp 7304  25 female    R     1   1    21
## 1294 06-May-2018_15:26:50  Exp 7304  25 female    R     1   1    22
## 1295 06-May-2018_15:26:52  Exp 7304  25 female    R     1   1    23
## 1296 06-May-2018_15:26:54  Exp 7304  25 female    R     1   1    24
## 1297 06-May-2018_15:26:56  Exp 7304  25 female    R     1   2     1
## 1298 06-May-2018_15:26:58  Exp 7304  25 female    R     1   2     2
## 1299 06-May-2018_15:27:00  Exp 7304  25 female    R     1   2     3
## 1300 06-May-2018_15:27:02  Exp 7304  25 female    R     1   2     4
## 1301 06-May-2018_15:27:04  Exp 7304  25 female    R     1   2     5
## 1302 06-May-2018_15:27:06  Exp 7304  25 female    R     1   2     6
## 1303 06-May-2018_15:27:08  Exp 7304  25 female    R     1   2     7
## 1304 06-May-2018_15:27:10  Exp 7304  25 female    R     1   2     8
## 1305 06-May-2018_15:27:12  Exp 7304  25 female    R     1   2     9
## 1306 06-May-2018_15:27:15  Exp 7304  25 female    R     1   2    10
## 1307 06-May-2018_15:27:17  Exp 7304  25 female    R     1   2    11
## 1308 06-May-2018_15:27:19  Exp 7304  25 female    R     1   2    12
## 1309 06-May-2018_15:27:21  Exp 7304  25 female    R     1   2    13
## 1310 06-May-2018_15:27:23  Exp 7304  25 female    R     1   2    14
## 1311 06-May-2018_15:27:25  Exp 7304  25 female    R     1   2    15
## 1312 06-May-2018_15:27:27  Exp 7304  25 female    R     1   2    16
## 1313 06-May-2018_15:27:29  Exp 7304  25 female    R     1   2    17
## 1314 06-May-2018_15:27:31  Exp 7304  25 female    R     1   2    18
## 1315 06-May-2018_15:27:33  Exp 7304  25 female    R     1   2    19
## 1316 06-May-2018_15:27:35  Exp 7304  25 female    R     1   2    20
## 1317 06-May-2018_15:27:37  Exp 7304  25 female    R     1   2    21
## 1318 06-May-2018_15:27:39  Exp 7304  25 female    R     1   2    22
## 1319 06-May-2018_15:27:41  Exp 7304  25 female    R     1   2    23
## 1320 06-May-2018_15:27:43  Exp 7304  25 female    R     1   2    24
## 1321 06-May-2018_14:43:27  Exp 7305  18   male    R     1   1     1
## 1322 06-May-2018_14:43:30  Exp 7305  18   male    R     1   1     2
## 1323 06-May-2018_14:43:32  Exp 7305  18   male    R     1   1     3
## 1324 06-May-2018_14:43:35  Exp 7305  18   male    R     1   1     4
## 1325 06-May-2018_14:43:37  Exp 7305  18   male    R     1   1     5
## 1326 06-May-2018_14:43:39  Exp 7305  18   male    R     1   1     6
## 1327 06-May-2018_14:43:42  Exp 7305  18   male    R     1   1     7
## 1328 06-May-2018_14:43:44  Exp 7305  18   male    R     1   1     8
## 1329 06-May-2018_14:43:46  Exp 7305  18   male    R     1   1     9
## 1330 06-May-2018_14:43:48  Exp 7305  18   male    R     1   1    10
## 1331 06-May-2018_14:43:51  Exp 7305  18   male    R     1   1    11
## 1332 06-May-2018_14:43:53  Exp 7305  18   male    R     1   1    12
## 1333 06-May-2018_14:43:56  Exp 7305  18   male    R     1   1    13
## 1334 06-May-2018_14:43:58  Exp 7305  18   male    R     1   1    14
## 1335 06-May-2018_14:44:00  Exp 7305  18   male    R     1   1    15
## 1336 06-May-2018_14:44:03  Exp 7305  18   male    R     1   1    16
## 1337 06-May-2018_14:44:05  Exp 7305  18   male    R     1   1    17
## 1338 06-May-2018_14:44:08  Exp 7305  18   male    R     1   1    18
## 1339 06-May-2018_14:44:10  Exp 7305  18   male    R     1   1    19
## 1340 06-May-2018_14:44:12  Exp 7305  18   male    R     1   1    20
## 1341 06-May-2018_14:44:15  Exp 7305  18   male    R     1   1    21
## 1342 06-May-2018_14:44:17  Exp 7305  18   male    R     1   1    22
## 1343 06-May-2018_14:44:20  Exp 7305  18   male    R     1   1    23
## 1344 06-May-2018_14:44:22  Exp 7305  18   male    R     1   1    24
## 1345 06-May-2018_14:44:24  Exp 7305  18   male    R     1   2     1
## 1346 06-May-2018_14:44:26  Exp 7305  18   male    R     1   2     2
## 1347 06-May-2018_14:44:29  Exp 7305  18   male    R     1   2     3
## 1348 06-May-2018_14:44:31  Exp 7305  18   male    R     1   2     4
## 1349 06-May-2018_14:44:34  Exp 7305  18   male    R     1   2     5
## 1350 06-May-2018_14:44:36  Exp 7305  18   male    R     1   2     6
## 1351 06-May-2018_14:44:38  Exp 7305  18   male    R     1   2     7
## 1352 06-May-2018_14:44:41  Exp 7305  18   male    R     1   2     8
## 1353 06-May-2018_14:44:43  Exp 7305  18   male    R     1   2     9
## 1354 06-May-2018_14:44:45  Exp 7305  18   male    R     1   2    10
## 1355 06-May-2018_14:44:48  Exp 7305  18   male    R     1   2    11
## 1356 06-May-2018_14:44:50  Exp 7305  18   male    R     1   2    12
## 1357 06-May-2018_14:44:52  Exp 7305  18   male    R     1   2    13
## 1358 06-May-2018_14:44:55  Exp 7305  18   male    R     1   2    14
## 1359 06-May-2018_14:44:57  Exp 7305  18   male    R     1   2    15
## 1360 06-May-2018_14:45:00  Exp 7305  18   male    R     1   2    16
## 1361 06-May-2018_14:45:02  Exp 7305  18   male    R     1   2    17
## 1362 06-May-2018_14:45:04  Exp 7305  18   male    R     1   2    18
## 1363 06-May-2018_14:45:06  Exp 7305  18   male    R     1   2    19
## 1364 06-May-2018_14:45:09  Exp 7305  18   male    R     1   2    20
## 1365 06-May-2018_14:45:11  Exp 7305  18   male    R     1   2    21
## 1366 06-May-2018_14:45:13  Exp 7305  18   male    R     1   2    22
## 1367 06-May-2018_14:45:16  Exp 7305  18   male    R     1   2    23
## 1368 06-May-2018_14:45:18  Exp 7305  18   male    R     1   2    24
## 1369 06-May-2018_14:45:20  Exp 7305  18   male    R     1   3     1
## 1370 06-May-2018_14:45:23  Exp 7305  18   male    R     1   3     2
## 1371 06-May-2018_14:45:25  Exp 7305  18   male    R     1   3     3
## 1372 06-May-2018_14:45:27  Exp 7305  18   male    R     1   3     4
## 1373 06-May-2018_14:45:29  Exp 7305  18   male    R     1   3     5
## 1374 06-May-2018_14:45:32  Exp 7305  18   male    R     1   3     6
## 1375 06-May-2018_14:45:34  Exp 7305  18   male    R     1   3     7
## 1376 06-May-2018_14:45:36  Exp 7305  18   male    R     1   3     8
## 1377 06-May-2018_14:45:39  Exp 7305  18   male    R     1   3     9
## 1378 06-May-2018_14:45:41  Exp 7305  18   male    R     1   3    10
## 1379 06-May-2018_14:45:43  Exp 7305  18   male    R     1   3    11
## 1380 06-May-2018_14:45:46  Exp 7305  18   male    R     1   3    12
## 1381 06-May-2018_14:45:48  Exp 7305  18   male    R     1   3    13
## 1382 06-May-2018_14:45:50  Exp 7305  18   male    R     1   3    14
## 1383 06-May-2018_14:45:52  Exp 7305  18   male    R     1   3    15
## 1384 06-May-2018_14:45:55  Exp 7305  18   male    R     1   3    16
## 1385 06-May-2018_14:45:57  Exp 7305  18   male    R     1   3    17
## 1386 06-May-2018_14:46:00  Exp 7305  18   male    R     1   3    18
## 1387 06-May-2018_14:46:02  Exp 7305  18   male    R     1   3    19
## 1388 06-May-2018_14:46:04  Exp 7305  18   male    R     1   3    20
## 1389 06-May-2018_14:46:07  Exp 7305  18   male    R     1   3    21
## 1390 06-May-2018_14:46:09  Exp 7305  18   male    R     1   3    22
## 1391 06-May-2018_14:46:12  Exp 7305  18   male    R     1   3    23
## 1392 06-May-2018_14:46:14  Exp 7305  18   male    R     1   3    24
## 1393 06-May-2018_14:46:33  Exp 7305  18   male    R     1   4     1
## 1394 06-May-2018_14:46:36  Exp 7305  18   male    R     1   4     2
## 1395 06-May-2018_14:46:38  Exp 7305  18   male    R     1   4     3
## 1396 06-May-2018_14:46:41  Exp 7305  18   male    R     1   4     4
## 1397 06-May-2018_14:46:43  Exp 7305  18   male    R     1   4     5
## 1398 06-May-2018_14:46:45  Exp 7305  18   male    R     1   4     6
## 1399 06-May-2018_14:46:48  Exp 7305  18   male    R     1   4     7
## 1400 06-May-2018_14:46:50  Exp 7305  18   male    R     1   4     8
## 1401 06-May-2018_14:46:52  Exp 7305  18   male    R     1   4     9
## 1402 06-May-2018_14:46:55  Exp 7305  18   male    R     1   4    10
## 1403 06-May-2018_14:46:57  Exp 7305  18   male    R     1   4    11
## 1404 06-May-2018_14:47:00  Exp 7305  18   male    R     1   4    12
## 1405 06-May-2018_14:47:02  Exp 7305  18   male    R     1   4    13
## 1406 06-May-2018_14:47:04  Exp 7305  18   male    R     1   4    14
## 1407 06-May-2018_14:47:06  Exp 7305  18   male    R     1   4    15
## 1408 06-May-2018_14:47:08  Exp 7305  18   male    R     1   4    16
## 1409 06-May-2018_14:47:11  Exp 7305  18   male    R     1   4    17
## 1410 06-May-2018_14:47:13  Exp 7305  18   male    R     1   4    18
## 1411 06-May-2018_14:47:16  Exp 7305  18   male    R     1   4    19
## 1412 06-May-2018_14:47:18  Exp 7305  18   male    R     1   4    20
## 1413 06-May-2018_14:47:20  Exp 7305  18   male    R     1   4    21
## 1414 06-May-2018_14:47:23  Exp 7305  18   male    R     1   4    22
## 1415 06-May-2018_14:47:25  Exp 7305  18   male    R     1   4    23
## 1416 06-May-2018_14:47:27  Exp 7305  18   male    R     1   4    24
## 1417 06-May-2018_14:47:30  Exp 7305  18   male    R     1   5     1
## 1418 06-May-2018_14:47:32  Exp 7305  18   male    R     1   5     2
## 1419 06-May-2018_14:47:34  Exp 7305  18   male    R     1   5     3
## 1420 06-May-2018_14:47:36  Exp 7305  18   male    R     1   5     4
## 1421 06-May-2018_14:47:39  Exp 7305  18   male    R     1   5     5
## 1422 06-May-2018_14:47:41  Exp 7305  18   male    R     1   5     6
## 1423 06-May-2018_14:47:43  Exp 7305  18   male    R     1   5     7
## 1424 06-May-2018_14:47:45  Exp 7305  18   male    R     1   5     8
## 1425 06-May-2018_14:47:47  Exp 7305  18   male    R     1   5     9
## 1426 06-May-2018_14:47:50  Exp 7305  18   male    R     1   5    10
## 1427 06-May-2018_14:47:52  Exp 7305  18   male    R     1   5    11
## 1428 06-May-2018_14:47:55  Exp 7305  18   male    R     1   5    12
## 1429 06-May-2018_14:47:57  Exp 7305  18   male    R     1   5    13
## 1430 06-May-2018_14:47:59  Exp 7305  18   male    R     1   5    14
## 1431 06-May-2018_14:48:01  Exp 7305  18   male    R     1   5    15
## 1432 06-May-2018_14:48:04  Exp 7305  18   male    R     1   5    16
## 1433 06-May-2018_14:48:06  Exp 7305  18   male    R     1   5    17
## 1434 06-May-2018_14:48:09  Exp 7305  18   male    R     1   5    18
## 1435 06-May-2018_14:48:11  Exp 7305  18   male    R     1   5    19
## 1436 06-May-2018_14:48:14  Exp 7305  18   male    R     1   5    20
## 1437 06-May-2018_14:48:16  Exp 7305  18   male    R     1   5    21
## 1438 06-May-2018_14:48:18  Exp 7305  18   male    R     1   5    22
## 1439 06-May-2018_14:48:20  Exp 7305  18   male    R     1   5    23
## 1440 06-May-2018_14:48:23  Exp 7305  18   male    R     1   5    24
## 1441 06-May-2018_14:48:28  Exp 7305  18   male    R     2   1     1
## 1442 06-May-2018_14:48:30  Exp 7305  18   male    R     2   1     2
## 1443 06-May-2018_14:48:32  Exp 7305  18   male    R     2   1     3
## 1444 06-May-2018_14:48:35  Exp 7305  18   male    R     2   1     4
## 1445 06-May-2018_14:48:37  Exp 7305  18   male    R     2   1     5
## 1446 06-May-2018_14:48:39  Exp 7305  18   male    R     2   1     6
## 1447 06-May-2018_14:48:42  Exp 7305  18   male    R     2   1     7
## 1448 06-May-2018_14:48:44  Exp 7305  18   male    R     2   1     8
## 1449 06-May-2018_14:48:46  Exp 7305  18   male    R     2   1     9
## 1450 06-May-2018_14:48:48  Exp 7305  18   male    R     2   1    10
## 1451 06-May-2018_14:48:51  Exp 7305  18   male    R     2   1    11
## 1452 06-May-2018_14:48:53  Exp 7305  18   male    R     2   1    12
## 1453 06-May-2018_14:48:55  Exp 7305  18   male    R     2   1    13
## 1454 06-May-2018_14:48:57  Exp 7305  18   male    R     2   1    14
## 1455 06-May-2018_14:49:00  Exp 7305  18   male    R     2   1    15
## 1456 06-May-2018_14:49:02  Exp 7305  18   male    R     2   1    16
## 1457 06-May-2018_14:49:04  Exp 7305  18   male    R     2   1    17
## 1458 06-May-2018_14:49:06  Exp 7305  18   male    R     2   1    18
## 1459 06-May-2018_14:49:08  Exp 7305  18   male    R     2   1    19
## 1460 06-May-2018_14:49:11  Exp 7305  18   male    R     2   1    20
## 1461 06-May-2018_14:49:13  Exp 7305  18   male    R     2   1    21
## 1462 06-May-2018_14:49:15  Exp 7305  18   male    R     2   1    22
## 1463 06-May-2018_14:49:17  Exp 7305  18   male    R     2   1    23
## 1464 06-May-2018_14:49:20  Exp 7305  18   male    R     2   1    24
## 1465 06-May-2018_14:49:22  Exp 7305  18   male    R     2   2     1
## 1466 06-May-2018_14:49:24  Exp 7305  18   male    R     2   2     2
## 1467 06-May-2018_14:49:26  Exp 7305  18   male    R     2   2     3
## 1468 06-May-2018_14:49:29  Exp 7305  18   male    R     2   2     4
## 1469 06-May-2018_14:49:31  Exp 7305  18   male    R     2   2     5
## 1470 06-May-2018_14:49:33  Exp 7305  18   male    R     2   2     6
## 1471 06-May-2018_14:49:35  Exp 7305  18   male    R     2   2     7
## 1472 06-May-2018_14:49:38  Exp 7305  18   male    R     2   2     8
## 1473 06-May-2018_14:49:40  Exp 7305  18   male    R     2   2     9
## 1474 06-May-2018_14:49:42  Exp 7305  18   male    R     2   2    10
## 1475 06-May-2018_14:49:44  Exp 7305  18   male    R     2   2    11
## 1476 06-May-2018_14:49:47  Exp 7305  18   male    R     2   2    12
## 1477 06-May-2018_14:49:49  Exp 7305  18   male    R     2   2    13
## 1478 06-May-2018_14:49:51  Exp 7305  18   male    R     2   2    14
## 1479 06-May-2018_14:49:53  Exp 7305  18   male    R     2   2    15
## 1480 06-May-2018_14:49:55  Exp 7305  18   male    R     2   2    16
## 1481 06-May-2018_14:49:58  Exp 7305  18   male    R     2   2    17
## 1482 06-May-2018_14:50:00  Exp 7305  18   male    R     2   2    18
## 1483 06-May-2018_14:50:02  Exp 7305  18   male    R     2   2    19
## 1484 06-May-2018_14:50:05  Exp 7305  18   male    R     2   2    20
## 1485 06-May-2018_14:50:07  Exp 7305  18   male    R     2   2    21
## 1486 06-May-2018_14:50:09  Exp 7305  18   male    R     2   2    22
## 1487 06-May-2018_14:50:11  Exp 7305  18   male    R     2   2    23
## 1488 06-May-2018_14:50:13  Exp 7305  18   male    R     2   2    24
## 1489 06-May-2018_14:50:15  Exp 7305  18   male    R     2   3     1
## 1490 06-May-2018_14:50:18  Exp 7305  18   male    R     2   3     2
## 1491 06-May-2018_14:50:20  Exp 7305  18   male    R     2   3     3
## 1492 06-May-2018_14:50:22  Exp 7305  18   male    R     2   3     4
## 1493 06-May-2018_14:50:25  Exp 7305  18   male    R     2   3     5
## 1494 06-May-2018_14:50:27  Exp 7305  18   male    R     2   3     6
## 1495 06-May-2018_14:50:29  Exp 7305  18   male    R     2   3     7
## 1496 06-May-2018_14:50:32  Exp 7305  18   male    R     2   3     8
## 1497 06-May-2018_14:50:34  Exp 7305  18   male    R     2   3     9
## 1498 06-May-2018_14:50:36  Exp 7305  18   male    R     2   3    10
## 1499 06-May-2018_14:50:39  Exp 7305  18   male    R     2   3    11
## 1500 06-May-2018_14:50:41  Exp 7305  18   male    R     2   3    12
## 1501 06-May-2018_14:50:43  Exp 7305  18   male    R     2   3    13
## 1502 06-May-2018_14:50:46  Exp 7305  18   male    R     2   3    14
## 1503 06-May-2018_14:50:48  Exp 7305  18   male    R     2   3    15
## 1504 06-May-2018_14:50:50  Exp 7305  18   male    R     2   3    16
## 1505 06-May-2018_14:50:52  Exp 7305  18   male    R     2   3    17
## 1506 06-May-2018_14:50:55  Exp 7305  18   male    R     2   3    18
## 1507 06-May-2018_14:50:57  Exp 7305  18   male    R     2   3    19
## 1508 06-May-2018_14:50:59  Exp 7305  18   male    R     2   3    20
## 1509 06-May-2018_14:51:01  Exp 7305  18   male    R     2   3    21
## 1510 06-May-2018_14:51:04  Exp 7305  18   male    R     2   3    22
## 1511 06-May-2018_14:51:06  Exp 7305  18   male    R     2   3    23
## 1512 06-May-2018_14:51:08  Exp 7305  18   male    R     2   3    24
## 1513 06-May-2018_14:51:42  Exp 7305  18   male    R     2   4     1
## 1514 06-May-2018_14:51:44  Exp 7305  18   male    R     2   4     2
## 1515 06-May-2018_14:51:46  Exp 7305  18   male    R     2   4     3
## 1516 06-May-2018_14:51:49  Exp 7305  18   male    R     2   4     4
## 1517 06-May-2018_14:51:51  Exp 7305  18   male    R     2   4     5
## 1518 06-May-2018_14:51:53  Exp 7305  18   male    R     2   4     6
## 1519 06-May-2018_14:51:55  Exp 7305  18   male    R     2   4     7
## 1520 06-May-2018_14:51:57  Exp 7305  18   male    R     2   4     8
## 1521 06-May-2018_14:52:00  Exp 7305  18   male    R     2   4     9
## 1522 06-May-2018_14:52:02  Exp 7305  18   male    R     2   4    10
## 1523 06-May-2018_14:52:04  Exp 7305  18   male    R     2   4    11
## 1524 06-May-2018_14:52:06  Exp 7305  18   male    R     2   4    12
## 1525 06-May-2018_14:52:08  Exp 7305  18   male    R     2   4    13
## 1526 06-May-2018_14:52:11  Exp 7305  18   male    R     2   4    14
## 1527 06-May-2018_14:52:13  Exp 7305  18   male    R     2   4    15
## 1528 06-May-2018_14:52:15  Exp 7305  18   male    R     2   4    16
## 1529 06-May-2018_14:52:17  Exp 7305  18   male    R     2   4    17
## 1530 06-May-2018_14:52:19  Exp 7305  18   male    R     2   4    18
## 1531 06-May-2018_14:52:21  Exp 7305  18   male    R     2   4    19
## 1532 06-May-2018_14:52:23  Exp 7305  18   male    R     2   4    20
## 1533 06-May-2018_14:52:26  Exp 7305  18   male    R     2   4    21
## 1534 06-May-2018_14:52:28  Exp 7305  18   male    R     2   4    22
## 1535 06-May-2018_14:52:30  Exp 7305  18   male    R     2   4    23
## 1536 06-May-2018_14:52:32  Exp 7305  18   male    R     2   4    24
## 1537 06-May-2018_14:52:34  Exp 7305  18   male    R     2   5     1
## 1538 06-May-2018_14:52:37  Exp 7305  18   male    R     2   5     2
## 1539 06-May-2018_14:52:39  Exp 7305  18   male    R     2   5     3
## 1540 06-May-2018_14:52:41  Exp 7305  18   male    R     2   5     4
## 1541 06-May-2018_14:52:43  Exp 7305  18   male    R     2   5     5
## 1542 06-May-2018_14:52:45  Exp 7305  18   male    R     2   5     6
## 1543 06-May-2018_14:52:48  Exp 7305  18   male    R     2   5     7
## 1544 06-May-2018_14:52:50  Exp 7305  18   male    R     2   5     8
## 1545 06-May-2018_14:52:52  Exp 7305  18   male    R     2   5     9
## 1546 06-May-2018_14:52:54  Exp 7305  18   male    R     2   5    10
## 1547 06-May-2018_14:52:57  Exp 7305  18   male    R     2   5    11
## 1548 06-May-2018_14:52:59  Exp 7305  18   male    R     2   5    12
## 1549 06-May-2018_14:53:01  Exp 7305  18   male    R     2   5    13
## 1550 06-May-2018_14:53:03  Exp 7305  18   male    R     2   5    14
## 1551 06-May-2018_14:53:05  Exp 7305  18   male    R     2   5    15
## 1552 06-May-2018_14:53:08  Exp 7305  18   male    R     2   5    16
## 1553 06-May-2018_14:53:10  Exp 7305  18   male    R     2   5    17
## 1554 06-May-2018_14:53:12  Exp 7305  18   male    R     2   5    18
## 1555 06-May-2018_14:53:15  Exp 7305  18   male    R     2   5    19
## 1556 06-May-2018_14:53:17  Exp 7305  18   male    R     2   5    20
## 1557 06-May-2018_14:53:19  Exp 7305  18   male    R     2   5    21
## 1558 06-May-2018_14:53:21  Exp 7305  18   male    R     2   5    22
## 1559 06-May-2018_14:53:24  Exp 7305  18   male    R     2   5    23
## 1560 06-May-2018_14:53:26  Exp 7305  18   male    R     2   5    24
## 1561 06-May-2018_14:53:31  Exp 7305  18   male    R     3   1     1
## 1562 06-May-2018_14:53:33  Exp 7305  18   male    R     3   1     2
## 1563 06-May-2018_14:53:35  Exp 7305  18   male    R     3   1     3
## 1564 06-May-2018_14:53:38  Exp 7305  18   male    R     3   1     4
## 1565 06-May-2018_14:53:40  Exp 7305  18   male    R     3   1     5
## 1566 06-May-2018_14:53:42  Exp 7305  18   male    R     3   1     6
## 1567 06-May-2018_14:53:44  Exp 7305  18   male    R     3   1     7
## 1568 06-May-2018_14:53:47  Exp 7305  18   male    R     3   1     8
## 1569 06-May-2018_14:53:49  Exp 7305  18   male    R     3   1     9
## 1570 06-May-2018_14:53:51  Exp 7305  18   male    R     3   1    10
## 1571 06-May-2018_14:53:54  Exp 7305  18   male    R     3   1    11
## 1572 06-May-2018_14:53:56  Exp 7305  18   male    R     3   1    12
## 1573 06-May-2018_14:53:58  Exp 7305  18   male    R     3   1    13
## 1574 06-May-2018_14:54:00  Exp 7305  18   male    R     3   1    14
## 1575 06-May-2018_14:54:03  Exp 7305  18   male    R     3   1    15
## 1576 06-May-2018_14:54:05  Exp 7305  18   male    R     3   1    16
## 1577 06-May-2018_14:54:07  Exp 7305  18   male    R     3   1    17
## 1578 06-May-2018_14:54:10  Exp 7305  18   male    R     3   1    18
## 1579 06-May-2018_14:54:12  Exp 7305  18   male    R     3   1    19
## 1580 06-May-2018_14:54:14  Exp 7305  18   male    R     3   1    20
## 1581 06-May-2018_14:54:16  Exp 7305  18   male    R     3   1    21
## 1582 06-May-2018_14:54:19  Exp 7305  18   male    R     3   1    22
## 1583 06-May-2018_14:54:21  Exp 7305  18   male    R     3   1    23
## 1584 06-May-2018_14:54:23  Exp 7305  18   male    R     3   1    24
## 1585 06-May-2018_14:54:25  Exp 7305  18   male    R     3   2     1
## 1586 06-May-2018_14:54:28  Exp 7305  18   male    R     3   2     2
## 1587 06-May-2018_14:54:30  Exp 7305  18   male    R     3   2     3
## 1588 06-May-2018_14:54:32  Exp 7305  18   male    R     3   2     4
## 1589 06-May-2018_14:54:34  Exp 7305  18   male    R     3   2     5
## 1590 06-May-2018_14:54:36  Exp 7305  18   male    R     3   2     6
## 1591 06-May-2018_14:54:38  Exp 7305  18   male    R     3   2     7
## 1592 06-May-2018_14:54:41  Exp 7305  18   male    R     3   2     8
## 1593 06-May-2018_14:54:43  Exp 7305  18   male    R     3   2     9
## 1594 06-May-2018_14:54:45  Exp 7305  18   male    R     3   2    10
## 1595 06-May-2018_14:54:48  Exp 7305  18   male    R     3   2    11
## 1596 06-May-2018_14:54:50  Exp 7305  18   male    R     3   2    12
## 1597 06-May-2018_14:54:52  Exp 7305  18   male    R     3   2    13
## 1598 06-May-2018_14:54:55  Exp 7305  18   male    R     3   2    14
## 1599 06-May-2018_14:54:57  Exp 7305  18   male    R     3   2    15
## 1600 06-May-2018_14:54:59  Exp 7305  18   male    R     3   2    16
## 1601 06-May-2018_14:55:01  Exp 7305  18   male    R     3   2    17
## 1602 06-May-2018_14:55:04  Exp 7305  18   male    R     3   2    18
## 1603 06-May-2018_14:55:06  Exp 7305  18   male    R     3   2    19
## 1604 06-May-2018_14:55:08  Exp 7305  18   male    R     3   2    20
## 1605 06-May-2018_14:55:10  Exp 7305  18   male    R     3   2    21
## 1606 06-May-2018_14:55:13  Exp 7305  18   male    R     3   2    22
## 1607 06-May-2018_14:55:15  Exp 7305  18   male    R     3   2    23
## 1608 06-May-2018_14:55:18  Exp 7305  18   male    R     3   2    24
## 1609 06-May-2018_14:55:20  Exp 7305  18   male    R     3   3     1
## 1610 06-May-2018_14:55:22  Exp 7305  18   male    R     3   3     2
## 1611 06-May-2018_14:55:24  Exp 7305  18   male    R     3   3     3
## 1612 06-May-2018_14:55:26  Exp 7305  18   male    R     3   3     4
## 1613 06-May-2018_14:55:28  Exp 7305  18   male    R     3   3     5
## 1614 06-May-2018_14:55:31  Exp 7305  18   male    R     3   3     6
## 1615 06-May-2018_14:55:33  Exp 7305  18   male    R     3   3     7
## 1616 06-May-2018_14:55:35  Exp 7305  18   male    R     3   3     8
## 1617 06-May-2018_14:55:37  Exp 7305  18   male    R     3   3     9
## 1618 06-May-2018_14:55:40  Exp 7305  18   male    R     3   3    10
## 1619 06-May-2018_14:55:42  Exp 7305  18   male    R     3   3    11
## 1620 06-May-2018_14:55:44  Exp 7305  18   male    R     3   3    12
## 1621 06-May-2018_14:55:46  Exp 7305  18   male    R     3   3    13
## 1622 06-May-2018_14:55:49  Exp 7305  18   male    R     3   3    14
## 1623 06-May-2018_14:55:51  Exp 7305  18   male    R     3   3    15
## 1624 06-May-2018_14:55:53  Exp 7305  18   male    R     3   3    16
## 1625 06-May-2018_14:55:55  Exp 7305  18   male    R     3   3    17
## 1626 06-May-2018_14:55:58  Exp 7305  18   male    R     3   3    18
## 1627 06-May-2018_14:56:00  Exp 7305  18   male    R     3   3    19
## 1628 06-May-2018_14:56:02  Exp 7305  18   male    R     3   3    20
## 1629 06-May-2018_14:56:04  Exp 7305  18   male    R     3   3    21
## 1630 06-May-2018_14:56:07  Exp 7305  18   male    R     3   3    22
## 1631 06-May-2018_14:56:09  Exp 7305  18   male    R     3   3    23
## 1632 06-May-2018_14:56:12  Exp 7305  18   male    R     3   3    24
## 1633 06-May-2018_14:56:36  Exp 7305  18   male    R     3   4     1
## 1634 06-May-2018_14:56:38  Exp 7305  18   male    R     3   4     2
## 1635 06-May-2018_14:56:40  Exp 7305  18   male    R     3   4     3
## 1636 06-May-2018_14:56:43  Exp 7305  18   male    R     3   4     4
## 1637 06-May-2018_14:56:45  Exp 7305  18   male    R     3   4     5
## 1638 06-May-2018_14:56:47  Exp 7305  18   male    R     3   4     6
## 1639 06-May-2018_14:56:49  Exp 7305  18   male    R     3   4     7
## 1640 06-May-2018_14:56:51  Exp 7305  18   male    R     3   4     8
## 1641 06-May-2018_14:56:53  Exp 7305  18   male    R     3   4     9
## 1642 06-May-2018_14:56:56  Exp 7305  18   male    R     3   4    10
## 1643 06-May-2018_14:56:58  Exp 7305  18   male    R     3   4    11
## 1644 06-May-2018_14:57:00  Exp 7305  18   male    R     3   4    12
## 1645 06-May-2018_14:57:03  Exp 7305  18   male    R     3   4    13
## 1646 06-May-2018_14:57:05  Exp 7305  18   male    R     3   4    14
## 1647 06-May-2018_14:57:08  Exp 7305  18   male    R     3   4    15
## 1648 06-May-2018_14:57:10  Exp 7305  18   male    R     3   4    16
## 1649 06-May-2018_14:57:12  Exp 7305  18   male    R     3   4    17
## 1650 06-May-2018_14:57:15  Exp 7305  18   male    R     3   4    18
## 1651 06-May-2018_14:57:17  Exp 7305  18   male    R     3   4    19
## 1652 06-May-2018_14:57:19  Exp 7305  18   male    R     3   4    20
## 1653 06-May-2018_14:57:21  Exp 7305  18   male    R     3   4    21
## 1654 06-May-2018_14:57:24  Exp 7305  18   male    R     3   4    22
## 1655 06-May-2018_14:57:26  Exp 7305  18   male    R     3   4    23
## 1656 06-May-2018_14:57:28  Exp 7305  18   male    R     3   4    24
## 1657 06-May-2018_14:57:30  Exp 7305  18   male    R     3   5     1
## 1658 06-May-2018_14:57:32  Exp 7305  18   male    R     3   5     2
## 1659 06-May-2018_14:57:35  Exp 7305  18   male    R     3   5     3
## 1660 06-May-2018_14:57:37  Exp 7305  18   male    R     3   5     4
## 1661 06-May-2018_14:57:40  Exp 7305  18   male    R     3   5     5
## 1662 06-May-2018_14:57:42  Exp 7305  18   male    R     3   5     6
## 1663 06-May-2018_14:57:44  Exp 7305  18   male    R     3   5     7
## 1664 06-May-2018_14:57:46  Exp 7305  18   male    R     3   5     8
## 1665 06-May-2018_14:57:49  Exp 7305  18   male    R     3   5     9
## 1666 06-May-2018_14:57:51  Exp 7305  18   male    R     3   5    10
## 1667 06-May-2018_14:57:53  Exp 7305  18   male    R     3   5    11
## 1668 06-May-2018_14:57:55  Exp 7305  18   male    R     3   5    12
## 1669 06-May-2018_14:57:58  Exp 7305  18   male    R     3   5    13
## 1670 06-May-2018_14:58:00  Exp 7305  18   male    R     3   5    14
## 1671 06-May-2018_14:58:02  Exp 7305  18   male    R     3   5    15
## 1672 06-May-2018_14:58:04  Exp 7305  18   male    R     3   5    16
## 1673 06-May-2018_14:58:07  Exp 7305  18   male    R     3   5    17
## 1674 06-May-2018_14:58:09  Exp 7305  18   male    R     3   5    18
## 1675 06-May-2018_14:58:11  Exp 7305  18   male    R     3   5    19
## 1676 06-May-2018_14:58:14  Exp 7305  18   male    R     3   5    20
## 1677 06-May-2018_14:58:16  Exp 7305  18   male    R     3   5    21
## 1678 06-May-2018_14:58:18  Exp 7305  18   male    R     3   5    22
## 1679 06-May-2018_14:58:20  Exp 7305  18   male    R     3   5    23
## 1680 06-May-2018_14:58:22  Exp 7305  18   male    R     3   5    24
## 1681 06-May-2018_15:03:42  Exp 7305  18   male    R     1   1     1
## 1682 06-May-2018_15:03:45  Exp 7305  18   male    R     1   1     2
## 1683 06-May-2018_15:03:48  Exp 7305  18   male    R     1   1     3
## 1684 06-May-2018_15:03:50  Exp 7305  18   male    R     1   1     4
## 1685 06-May-2018_15:03:52  Exp 7305  18   male    R     1   1     5
## 1686 06-May-2018_15:03:55  Exp 7305  18   male    R     1   1     6
## 1687 06-May-2018_15:03:57  Exp 7305  18   male    R     1   1     7
## 1688 06-May-2018_15:04:00  Exp 7305  18   male    R     1   1     8
## 1689 06-May-2018_15:04:02  Exp 7305  18   male    R     1   1     9
## 1690 06-May-2018_15:04:04  Exp 7305  18   male    R     1   1    10
## 1691 06-May-2018_15:04:06  Exp 7305  18   male    R     1   1    11
## 1692 06-May-2018_15:04:09  Exp 7305  18   male    R     1   1    12
## 1693 06-May-2018_15:04:11  Exp 7305  18   male    R     1   1    13
## 1694 06-May-2018_15:04:13  Exp 7305  18   male    R     1   1    14
## 1695 06-May-2018_15:04:16  Exp 7305  18   male    R     1   1    15
## 1696 06-May-2018_15:04:18  Exp 7305  18   male    R     1   1    16
## 1697 06-May-2018_15:04:20  Exp 7305  18   male    R     1   1    17
## 1698 06-May-2018_15:04:22  Exp 7305  18   male    R     1   1    18
## 1699 06-May-2018_15:04:25  Exp 7305  18   male    R     1   1    19
## 1700 06-May-2018_15:04:27  Exp 7305  18   male    R     1   1    20
## 1701 06-May-2018_15:04:29  Exp 7305  18   male    R     1   1    21
## 1702 06-May-2018_15:04:32  Exp 7305  18   male    R     1   1    22
## 1703 06-May-2018_15:04:34  Exp 7305  18   male    R     1   1    23
## 1704 06-May-2018_15:04:36  Exp 7305  18   male    R     1   1    24
## 1705 06-May-2018_15:04:39  Exp 7305  18   male    R     1   2     1
## 1706 06-May-2018_15:04:41  Exp 7305  18   male    R     1   2     2
## 1707 06-May-2018_15:04:43  Exp 7305  18   male    R     1   2     3
## 1708 06-May-2018_15:04:46  Exp 7305  18   male    R     1   2     4
## 1709 06-May-2018_15:04:48  Exp 7305  18   male    R     1   2     5
## 1710 06-May-2018_15:04:50  Exp 7305  18   male    R     1   2     6
## 1711 06-May-2018_15:04:52  Exp 7305  18   male    R     1   2     7
## 1712 06-May-2018_15:04:54  Exp 7305  18   male    R     1   2     8
## 1713 06-May-2018_15:04:57  Exp 7305  18   male    R     1   2     9
## 1714 06-May-2018_15:04:59  Exp 7305  18   male    R     1   2    10
## 1715 06-May-2018_15:05:01  Exp 7305  18   male    R     1   2    11
## 1716 06-May-2018_15:05:04  Exp 7305  18   male    R     1   2    12
## 1717 06-May-2018_15:05:06  Exp 7305  18   male    R     1   2    13
## 1718 06-May-2018_15:05:08  Exp 7305  18   male    R     1   2    14
## 1719 06-May-2018_15:05:11  Exp 7305  18   male    R     1   2    15
## 1720 06-May-2018_15:05:13  Exp 7305  18   male    R     1   2    16
## 1721 06-May-2018_15:05:16  Exp 7305  18   male    R     1   2    17
## 1722 06-May-2018_15:05:18  Exp 7305  18   male    R     1   2    18
## 1723 06-May-2018_15:05:20  Exp 7305  18   male    R     1   2    19
## 1724 06-May-2018_15:05:23  Exp 7305  18   male    R     1   2    20
## 1725 06-May-2018_15:05:25  Exp 7305  18   male    R     1   2    21
## 1726 06-May-2018_15:05:28  Exp 7305  18   male    R     1   2    22
## 1727 06-May-2018_15:05:30  Exp 7305  18   male    R     1   2    23
## 1728 06-May-2018_15:05:32  Exp 7305  18   male    R     1   2    24
## 1729 06-May-2018_15:10:38  Exp 7305  18   male    R     1   1     1
## 1730 06-May-2018_15:10:41  Exp 7305  18   male    R     1   1     2
## 1731 06-May-2018_15:10:43  Exp 7305  18   male    R     1   1     3
## 1732 06-May-2018_15:10:45  Exp 7305  18   male    R     1   1     4
## 1733 06-May-2018_15:10:48  Exp 7305  18   male    R     1   1     5
## 1734 06-May-2018_15:10:50  Exp 7305  18   male    R     1   1     6
## 1735 06-May-2018_15:10:52  Exp 7305  18   male    R     1   1     7
## 1736 06-May-2018_15:10:55  Exp 7305  18   male    R     1   1     8
## 1737 06-May-2018_15:10:57  Exp 7305  18   male    R     1   1     9
## 1738 06-May-2018_15:10:59  Exp 7305  18   male    R     1   1    10
## 1739 06-May-2018_15:11:02  Exp 7305  18   male    R     1   1    11
## 1740 06-May-2018_15:11:04  Exp 7305  18   male    R     1   1    12
## 1741 06-May-2018_15:11:07  Exp 7305  18   male    R     1   1    13
## 1742 06-May-2018_15:11:09  Exp 7305  18   male    R     1   1    14
## 1743 06-May-2018_15:11:11  Exp 7305  18   male    R     1   1    15
## 1744 06-May-2018_15:11:13  Exp 7305  18   male    R     1   1    16
## 1745 06-May-2018_15:11:16  Exp 7305  18   male    R     1   1    17
## 1746 06-May-2018_15:11:18  Exp 7305  18   male    R     1   1    18
## 1747 06-May-2018_15:11:20  Exp 7305  18   male    R     1   1    19
## 1748 06-May-2018_15:11:23  Exp 7305  18   male    R     1   1    20
## 1749 06-May-2018_15:11:25  Exp 7305  18   male    R     1   1    21
## 1750 06-May-2018_15:11:28  Exp 7305  18   male    R     1   1    22
## 1751 06-May-2018_15:11:30  Exp 7305  18   male    R     1   1    23
## 1752 06-May-2018_15:11:32  Exp 7305  18   male    R     1   1    24
## 1753 06-May-2018_15:11:35  Exp 7305  18   male    R     1   2     1
## 1754 06-May-2018_15:11:37  Exp 7305  18   male    R     1   2     2
## 1755 06-May-2018_15:11:39  Exp 7305  18   male    R     1   2     3
## 1756 06-May-2018_15:11:42  Exp 7305  18   male    R     1   2     4
## 1757 06-May-2018_15:11:45  Exp 7305  18   male    R     1   2     5
## 1758 06-May-2018_15:11:47  Exp 7305  18   male    R     1   2     6
## 1759 06-May-2018_15:11:50  Exp 7305  18   male    R     1   2     7
## 1760 06-May-2018_15:11:52  Exp 7305  18   male    R     1   2     8
## 1761 06-May-2018_15:11:54  Exp 7305  18   male    R     1   2     9
## 1762 06-May-2018_15:11:56  Exp 7305  18   male    R     1   2    10
## 1763 06-May-2018_15:11:59  Exp 7305  18   male    R     1   2    11
## 1764 06-May-2018_15:12:01  Exp 7305  18   male    R     1   2    12
## 1765 06-May-2018_15:12:03  Exp 7305  18   male    R     1   2    13
## 1766 06-May-2018_15:12:05  Exp 7305  18   male    R     1   2    14
## 1767 06-May-2018_15:12:07  Exp 7305  18   male    R     1   2    15
## 1768 06-May-2018_15:12:10  Exp 7305  18   male    R     1   2    16
## 1769 06-May-2018_15:12:12  Exp 7305  18   male    R     1   2    17
## 1770 06-May-2018_15:12:15  Exp 7305  18   male    R     1   2    18
## 1771 06-May-2018_15:12:18  Exp 7305  18   male    R     1   2    19
## 1772 06-May-2018_15:12:20  Exp 7305  18   male    R     1   2    20
## 1773 06-May-2018_15:12:22  Exp 7305  18   male    R     1   2    21
## 1774 06-May-2018_15:12:25  Exp 7305  18   male    R     1   2    22
## 1775 06-May-2018_15:12:27  Exp 7305  18   male    R     1   2    23
## 1776 06-May-2018_15:12:29  Exp 7305  18   male    R     1   2    24
## 1777 06-May-2018_15:17:24  Exp 7305  18   male    R     1   1     1
## 1778 06-May-2018_15:17:27  Exp 7305  18   male    R     1   1     2
## 1779 06-May-2018_15:17:30  Exp 7305  18   male    R     1   1     3
## 1780 06-May-2018_15:17:32  Exp 7305  18   male    R     1   1     4
## 1781 06-May-2018_15:17:35  Exp 7305  18   male    R     1   1     5
## 1782 06-May-2018_15:17:37  Exp 7305  18   male    R     1   1     6
## 1783 06-May-2018_15:17:39  Exp 7305  18   male    R     1   1     7
## 1784 06-May-2018_15:17:41  Exp 7305  18   male    R     1   1     8
## 1785 06-May-2018_15:17:44  Exp 7305  18   male    R     1   1     9
## 1786 06-May-2018_15:17:46  Exp 7305  18   male    R     1   1    10
## 1787 06-May-2018_15:17:48  Exp 7305  18   male    R     1   1    11
## 1788 06-May-2018_15:17:51  Exp 7305  18   male    R     1   1    12
## 1789 06-May-2018_15:17:53  Exp 7305  18   male    R     1   1    13
## 1790 06-May-2018_15:17:56  Exp 7305  18   male    R     1   1    14
## 1791 06-May-2018_15:17:58  Exp 7305  18   male    R     1   1    15
## 1792 06-May-2018_15:18:01  Exp 7305  18   male    R     1   1    16
## 1793 06-May-2018_15:18:03  Exp 7305  18   male    R     1   1    17
## 1794 06-May-2018_15:18:05  Exp 7305  18   male    R     1   1    18
## 1795 06-May-2018_15:18:08  Exp 7305  18   male    R     1   1    19
## 1796 06-May-2018_15:18:10  Exp 7305  18   male    R     1   1    20
## 1797 06-May-2018_15:18:12  Exp 7305  18   male    R     1   1    21
## 1798 06-May-2018_15:18:15  Exp 7305  18   male    R     1   1    22
## 1799 06-May-2018_15:18:17  Exp 7305  18   male    R     1   1    23
## 1800 06-May-2018_15:18:19  Exp 7305  18   male    R     1   1    24
## 1801 06-May-2018_15:18:21  Exp 7305  18   male    R     1   2     1
## 1802 06-May-2018_15:18:24  Exp 7305  18   male    R     1   2     2
## 1803 06-May-2018_15:18:26  Exp 7305  18   male    R     1   2     3
## 1804 06-May-2018_15:18:29  Exp 7305  18   male    R     1   2     4
## 1805 06-May-2018_15:18:31  Exp 7305  18   male    R     1   2     5
## 1806 06-May-2018_15:18:33  Exp 7305  18   male    R     1   2     6
## 1807 06-May-2018_15:18:35  Exp 7305  18   male    R     1   2     7
## 1808 06-May-2018_15:18:38  Exp 7305  18   male    R     1   2     8
## 1809 06-May-2018_15:18:40  Exp 7305  18   male    R     1   2     9
## 1810 06-May-2018_15:18:43  Exp 7305  18   male    R     1   2    10
## 1811 06-May-2018_15:18:45  Exp 7305  18   male    R     1   2    11
## 1812 06-May-2018_15:18:47  Exp 7305  18   male    R     1   2    12
## 1813 06-May-2018_15:18:50  Exp 7305  18   male    R     1   2    13
## 1814 06-May-2018_15:18:52  Exp 7305  18   male    R     1   2    14
## 1815 06-May-2018_15:18:54  Exp 7305  18   male    R     1   2    15
## 1816 06-May-2018_15:18:56  Exp 7305  18   male    R     1   2    16
## 1817 06-May-2018_15:18:58  Exp 7305  18   male    R     1   2    17
## 1818 06-May-2018_15:19:00  Exp 7305  18   male    R     1   2    18
## 1819 06-May-2018_15:19:03  Exp 7305  18   male    R     1   2    19
## 1820 06-May-2018_15:19:05  Exp 7305  18   male    R     1   2    20
## 1821 06-May-2018_15:19:07  Exp 7305  18   male    R     1   2    21
## 1822 06-May-2018_15:19:10  Exp 7305  18   male    R     1   2    22
## 1823 06-May-2018_15:19:12  Exp 7305  18   male    R     1   2    23
## 1824 06-May-2018_15:19:15  Exp 7305  18   male    R     1   2    24
## 1825 06-May-2018_15:24:46  Exp 7305  18   male    R     1   1     1
## 1826 06-May-2018_15:24:48  Exp 7305  18   male    R     1   1     2
## 1827 06-May-2018_15:24:50  Exp 7305  18   male    R     1   1     3
## 1828 06-May-2018_15:24:52  Exp 7305  18   male    R     1   1     4
## 1829 06-May-2018_15:24:54  Exp 7305  18   male    R     1   1     5
## 1830 06-May-2018_15:24:57  Exp 7305  18   male    R     1   1     6
## 1831 06-May-2018_15:24:59  Exp 7305  18   male    R     1   1     7
## 1832 06-May-2018_15:25:01  Exp 7305  18   male    R     1   1     8
## 1833 06-May-2018_15:25:03  Exp 7305  18   male    R     1   1     9
## 1834 06-May-2018_15:25:06  Exp 7305  18   male    R     1   1    10
## 1835 06-May-2018_15:25:08  Exp 7305  18   male    R     1   1    11
## 1836 06-May-2018_15:25:10  Exp 7305  18   male    R     1   1    12
## 1837 06-May-2018_15:25:13  Exp 7305  18   male    R     1   1    13
## 1838 06-May-2018_15:25:15  Exp 7305  18   male    R     1   1    14
## 1839 06-May-2018_15:25:17  Exp 7305  18   male    R     1   1    15
## 1840 06-May-2018_15:25:19  Exp 7305  18   male    R     1   1    16
## 1841 06-May-2018_15:25:22  Exp 7305  18   male    R     1   1    17
## 1842 06-May-2018_15:25:24  Exp 7305  18   male    R     1   1    18
## 1843 06-May-2018_15:25:26  Exp 7305  18   male    R     1   1    19
## 1844 06-May-2018_15:25:29  Exp 7305  18   male    R     1   1    20
## 1845 06-May-2018_15:25:31  Exp 7305  18   male    R     1   1    21
## 1846 06-May-2018_15:25:33  Exp 7305  18   male    R     1   1    22
## 1847 06-May-2018_15:25:35  Exp 7305  18   male    R     1   1    23
## 1848 06-May-2018_15:25:38  Exp 7305  18   male    R     1   1    24
## 1849 06-May-2018_15:25:40  Exp 7305  18   male    R     1   2     1
## 1850 06-May-2018_15:25:43  Exp 7305  18   male    R     1   2     2
## 1851 06-May-2018_15:25:45  Exp 7305  18   male    R     1   2     3
## 1852 06-May-2018_15:25:47  Exp 7305  18   male    R     1   2     4
## 1853 06-May-2018_15:25:49  Exp 7305  18   male    R     1   2     5
## 1854 06-May-2018_15:25:51  Exp 7305  18   male    R     1   2     6
## 1855 06-May-2018_15:25:54  Exp 7305  18   male    R     1   2     7
## 1856 06-May-2018_15:25:56  Exp 7305  18   male    R     1   2     8
## 1857 06-May-2018_15:25:58  Exp 7305  18   male    R     1   2     9
## 1858 06-May-2018_15:26:01  Exp 7305  18   male    R     1   2    10
## 1859 06-May-2018_15:26:03  Exp 7305  18   male    R     1   2    11
## 1860 06-May-2018_15:26:05  Exp 7305  18   male    R     1   2    12
## 1861 06-May-2018_15:26:07  Exp 7305  18   male    R     1   2    13
## 1862 06-May-2018_15:26:10  Exp 7305  18   male    R     1   2    14
## 1863 06-May-2018_15:26:12  Exp 7305  18   male    R     1   2    15
## 1864 06-May-2018_15:26:14  Exp 7305  18   male    R     1   2    16
## 1865 06-May-2018_15:26:17  Exp 7305  18   male    R     1   2    17
## 1866 06-May-2018_15:26:20  Exp 7305  18   male    R     1   2    18
## 1867 06-May-2018_15:26:22  Exp 7305  18   male    R     1   2    19
## 1868 06-May-2018_15:26:24  Exp 7305  18   male    R     1   2    20
## 1869 06-May-2018_15:26:27  Exp 7305  18   male    R     1   2    21
## 1870 06-May-2018_15:26:29  Exp 7305  18   male    R     1   2    22
## 1871 06-May-2018_15:26:31  Exp 7305  18   male    R     1   2    23
## 1872 06-May-2018_15:26:33  Exp 7305  18   male    R     1   2    24
## 1873 06-May-2018_15:32:26  Exp 7305  18   male    R     1   1     1
## 1874 06-May-2018_15:32:28  Exp 7305  18   male    R     1   1     2
## 1875 06-May-2018_15:32:30  Exp 7305  18   male    R     1   1     3
## 1876 06-May-2018_15:32:33  Exp 7305  18   male    R     1   1     4
## 1877 06-May-2018_15:32:35  Exp 7305  18   male    R     1   1     5
## 1878 06-May-2018_15:32:37  Exp 7305  18   male    R     1   1     6
## 1879 06-May-2018_15:32:40  Exp 7305  18   male    R     1   1     7
## 1880 06-May-2018_15:32:42  Exp 7305  18   male    R     1   1     8
## 1881 06-May-2018_15:32:44  Exp 7305  18   male    R     1   1     9
## 1882 06-May-2018_15:32:46  Exp 7305  18   male    R     1   1    10
## 1883 06-May-2018_15:32:49  Exp 7305  18   male    R     1   1    11
## 1884 06-May-2018_15:32:51  Exp 7305  18   male    R     1   1    12
## 1885 06-May-2018_15:32:53  Exp 7305  18   male    R     1   1    13
## 1886 06-May-2018_15:32:56  Exp 7305  18   male    R     1   1    14
## 1887 06-May-2018_15:32:58  Exp 7305  18   male    R     1   1    15
## 1888 06-May-2018_15:33:01  Exp 7305  18   male    R     1   1    16
## 1889 06-May-2018_15:33:03  Exp 7305  18   male    R     1   1    17
## 1890 06-May-2018_15:33:05  Exp 7305  18   male    R     1   1    18
## 1891 06-May-2018_15:33:07  Exp 7305  18   male    R     1   1    19
## 1892 06-May-2018_15:33:10  Exp 7305  18   male    R     1   1    20
## 1893 06-May-2018_15:33:12  Exp 7305  18   male    R     1   1    21
## 1894 06-May-2018_15:33:14  Exp 7305  18   male    R     1   1    22
## 1895 06-May-2018_15:33:16  Exp 7305  18   male    R     1   1    23
## 1896 06-May-2018_15:33:18  Exp 7305  18   male    R     1   1    24
## 1897 06-May-2018_15:33:21  Exp 7305  18   male    R     1   2     1
## 1898 06-May-2018_15:33:23  Exp 7305  18   male    R     1   2     2
## 1899 06-May-2018_15:33:26  Exp 7305  18   male    R     1   2     3
## 1900 06-May-2018_15:33:28  Exp 7305  18   male    R     1   2     4
## 1901 06-May-2018_15:33:30  Exp 7305  18   male    R     1   2     5
## 1902 06-May-2018_15:33:32  Exp 7305  18   male    R     1   2     6
## 1903 06-May-2018_15:33:35  Exp 7305  18   male    R     1   2     7
## 1904 06-May-2018_15:33:37  Exp 7305  18   male    R     1   2     8
## 1905 06-May-2018_15:33:40  Exp 7305  18   male    R     1   2     9
## 1906 06-May-2018_15:33:42  Exp 7305  18   male    R     1   2    10
## 1907 06-May-2018_15:33:44  Exp 7305  18   male    R     1   2    11
## 1908 06-May-2018_15:33:47  Exp 7305  18   male    R     1   2    12
## 1909 06-May-2018_15:33:49  Exp 7305  18   male    R     1   2    13
## 1910 06-May-2018_15:33:51  Exp 7305  18   male    R     1   2    14
## 1911 06-May-2018_15:33:54  Exp 7305  18   male    R     1   2    15
## 1912 06-May-2018_15:33:56  Exp 7305  18   male    R     1   2    16
## 1913 06-May-2018_15:33:59  Exp 7305  18   male    R     1   2    17
## 1914 06-May-2018_15:34:01  Exp 7305  18   male    R     1   2    18
## 1915 06-May-2018_15:34:03  Exp 7305  18   male    R     1   2    19
## 1916 06-May-2018_15:34:06  Exp 7305  18   male    R     1   2    20
## 1917 06-May-2018_15:34:08  Exp 7305  18   male    R     1   2    21
## 1918 06-May-2018_15:34:11  Exp 7305  18   male    R     1   2    22
## 1919 06-May-2018_15:34:13  Exp 7305  18   male    R     1   2    23
## 1920 06-May-2018_15:34:15  Exp 7305  18   male    R     1   2    24
## 1921 09-May-2018_13:39:50  Exp 7306  21   male    R     1   1     1
## 1922 09-May-2018_13:39:53  Exp 7306  21   male    R     1   1     2
## 1923 09-May-2018_13:39:55  Exp 7306  21   male    R     1   1     3
## 1924 09-May-2018_13:39:57  Exp 7306  21   male    R     1   1     4
## 1925 09-May-2018_13:40:00  Exp 7306  21   male    R     1   1     5
## 1926 09-May-2018_13:40:02  Exp 7306  21   male    R     1   1     6
## 1927 09-May-2018_13:40:05  Exp 7306  21   male    R     1   1     7
## 1928 09-May-2018_13:40:07  Exp 7306  21   male    R     1   1     8
## 1929 09-May-2018_13:40:10  Exp 7306  21   male    R     1   1     9
## 1930 09-May-2018_13:40:12  Exp 7306  21   male    R     1   1    10
## 1931 09-May-2018_13:40:14  Exp 7306  21   male    R     1   1    11
## 1932 09-May-2018_13:40:17  Exp 7306  21   male    R     1   1    12
## 1933 09-May-2018_13:40:19  Exp 7306  21   male    R     1   1    13
## 1934 09-May-2018_13:40:22  Exp 7306  21   male    R     1   1    14
## 1935 09-May-2018_13:40:24  Exp 7306  21   male    R     1   1    15
## 1936 09-May-2018_13:40:27  Exp 7306  21   male    R     1   1    16
## 1937 09-May-2018_13:40:29  Exp 7306  21   male    R     1   1    17
## 1938 09-May-2018_13:40:32  Exp 7306  21   male    R     1   1    18
## 1939 09-May-2018_13:40:34  Exp 7306  21   male    R     1   1    19
## 1940 09-May-2018_13:40:37  Exp 7306  21   male    R     1   1    20
## 1941 09-May-2018_13:40:39  Exp 7306  21   male    R     1   1    21
## 1942 09-May-2018_13:40:41  Exp 7306  21   male    R     1   1    22
## 1943 09-May-2018_13:40:44  Exp 7306  21   male    R     1   1    23
## 1944 09-May-2018_13:40:46  Exp 7306  21   male    R     1   1    24
## 1945 09-May-2018_13:40:49  Exp 7306  21   male    R     1   2     1
## 1946 09-May-2018_13:40:51  Exp 7306  21   male    R     1   2     2
## 1947 09-May-2018_13:40:54  Exp 7306  21   male    R     1   2     3
## 1948 09-May-2018_13:40:56  Exp 7306  21   male    R     1   2     4
## 1949 09-May-2018_13:40:58  Exp 7306  21   male    R     1   2     5
## 1950 09-May-2018_13:41:01  Exp 7306  21   male    R     1   2     6
## 1951 09-May-2018_13:41:04  Exp 7306  21   male    R     1   2     7
## 1952 09-May-2018_13:41:06  Exp 7306  21   male    R     1   2     8
## 1953 09-May-2018_13:41:09  Exp 7306  21   male    R     1   2     9
## 1954 09-May-2018_13:41:11  Exp 7306  21   male    R     1   2    10
## 1955 09-May-2018_13:41:13  Exp 7306  21   male    R     1   2    11
## 1956 09-May-2018_13:41:16  Exp 7306  21   male    R     1   2    12
## 1957 09-May-2018_13:41:18  Exp 7306  21   male    R     1   2    13
## 1958 09-May-2018_13:41:20  Exp 7306  21   male    R     1   2    14
## 1959 09-May-2018_13:41:23  Exp 7306  21   male    R     1   2    15
## 1960 09-May-2018_13:41:25  Exp 7306  21   male    R     1   2    16
## 1961 09-May-2018_13:41:28  Exp 7306  21   male    R     1   2    17
## 1962 09-May-2018_13:41:30  Exp 7306  21   male    R     1   2    18
## 1963 09-May-2018_13:41:32  Exp 7306  21   male    R     1   2    19
## 1964 09-May-2018_13:41:35  Exp 7306  21   male    R     1   2    20
## 1965 09-May-2018_13:41:37  Exp 7306  21   male    R     1   2    21
## 1966 09-May-2018_13:41:40  Exp 7306  21   male    R     1   2    22
## 1967 09-May-2018_13:41:42  Exp 7306  21   male    R     1   2    23
## 1968 09-May-2018_13:41:44  Exp 7306  21   male    R     1   2    24
## 1969 09-May-2018_13:41:46  Exp 7306  21   male    R     1   3     1
## 1970 09-May-2018_13:41:49  Exp 7306  21   male    R     1   3     2
## 1971 09-May-2018_13:41:51  Exp 7306  21   male    R     1   3     3
## 1972 09-May-2018_13:41:53  Exp 7306  21   male    R     1   3     4
## 1973 09-May-2018_13:41:56  Exp 7306  21   male    R     1   3     5
## 1974 09-May-2018_13:41:58  Exp 7306  21   male    R     1   3     6
## 1975 09-May-2018_13:42:00  Exp 7306  21   male    R     1   3     7
## 1976 09-May-2018_13:42:03  Exp 7306  21   male    R     1   3     8
## 1977 09-May-2018_13:42:05  Exp 7306  21   male    R     1   3     9
## 1978 09-May-2018_13:42:07  Exp 7306  21   male    R     1   3    10
## 1979 09-May-2018_13:42:10  Exp 7306  21   male    R     1   3    11
## 1980 09-May-2018_13:42:12  Exp 7306  21   male    R     1   3    12
## 1981 09-May-2018_13:42:14  Exp 7306  21   male    R     1   3    13
## 1982 09-May-2018_13:42:17  Exp 7306  21   male    R     1   3    14
## 1983 09-May-2018_13:42:19  Exp 7306  21   male    R     1   3    15
## 1984 09-May-2018_13:42:21  Exp 7306  21   male    R     1   3    16
## 1985 09-May-2018_13:42:24  Exp 7306  21   male    R     1   3    17
## 1986 09-May-2018_13:42:26  Exp 7306  21   male    R     1   3    18
## 1987 09-May-2018_13:42:29  Exp 7306  21   male    R     1   3    19
## 1988 09-May-2018_13:42:31  Exp 7306  21   male    R     1   3    20
## 1989 09-May-2018_13:42:33  Exp 7306  21   male    R     1   3    21
## 1990 09-May-2018_13:42:36  Exp 7306  21   male    R     1   3    22
## 1991 09-May-2018_13:42:38  Exp 7306  21   male    R     1   3    23
## 1992 09-May-2018_13:42:40  Exp 7306  21   male    R     1   3    24
## 1993 09-May-2018_13:42:58  Exp 7306  21   male    R     1   4     1
## 1994 09-May-2018_13:43:01  Exp 7306  21   male    R     1   4     2
## 1995 09-May-2018_13:43:03  Exp 7306  21   male    R     1   4     3
## 1996 09-May-2018_13:43:05  Exp 7306  21   male    R     1   4     4
## 1997 09-May-2018_13:43:08  Exp 7306  21   male    R     1   4     5
## 1998 09-May-2018_13:43:10  Exp 7306  21   male    R     1   4     6
## 1999 09-May-2018_13:43:12  Exp 7306  21   male    R     1   4     7
## 2000 09-May-2018_13:43:15  Exp 7306  21   male    R     1   4     8
## 2001 09-May-2018_13:43:17  Exp 7306  21   male    R     1   4     9
## 2002 09-May-2018_13:43:19  Exp 7306  21   male    R     1   4    10
## 2003 09-May-2018_13:43:21  Exp 7306  21   male    R     1   4    11
## 2004 09-May-2018_13:43:23  Exp 7306  21   male    R     1   4    12
## 2005 09-May-2018_13:43:26  Exp 7306  21   male    R     1   4    13
## 2006 09-May-2018_13:43:28  Exp 7306  21   male    R     1   4    14
## 2007 09-May-2018_13:43:30  Exp 7306  21   male    R     1   4    15
## 2008 09-May-2018_13:43:33  Exp 7306  21   male    R     1   4    16
## 2009 09-May-2018_13:43:35  Exp 7306  21   male    R     1   4    17
## 2010 09-May-2018_13:43:37  Exp 7306  21   male    R     1   4    18
## 2011 09-May-2018_13:43:40  Exp 7306  21   male    R     1   4    19
## 2012 09-May-2018_13:43:42  Exp 7306  21   male    R     1   4    20
## 2013 09-May-2018_13:43:45  Exp 7306  21   male    R     1   4    21
## 2014 09-May-2018_13:43:47  Exp 7306  21   male    R     1   4    22
## 2015 09-May-2018_13:43:50  Exp 7306  21   male    R     1   4    23
## 2016 09-May-2018_13:43:52  Exp 7306  21   male    R     1   4    24
## 2017 09-May-2018_13:43:55  Exp 7306  21   male    R     1   5     1
## 2018 09-May-2018_13:43:57  Exp 7306  21   male    R     1   5     2
## 2019 09-May-2018_13:44:00  Exp 7306  21   male    R     1   5     3
## 2020 09-May-2018_13:44:02  Exp 7306  21   male    R     1   5     4
## 2021 09-May-2018_13:44:04  Exp 7306  21   male    R     1   5     5
## 2022 09-May-2018_13:44:07  Exp 7306  21   male    R     1   5     6
## 2023 09-May-2018_13:44:09  Exp 7306  21   male    R     1   5     7
## 2024 09-May-2018_13:44:11  Exp 7306  21   male    R     1   5     8
## 2025 09-May-2018_13:44:13  Exp 7306  21   male    R     1   5     9
## 2026 09-May-2018_13:44:16  Exp 7306  21   male    R     1   5    10
## 2027 09-May-2018_13:44:18  Exp 7306  21   male    R     1   5    11
## 2028 09-May-2018_13:44:20  Exp 7306  21   male    R     1   5    12
## 2029 09-May-2018_13:44:23  Exp 7306  21   male    R     1   5    13
## 2030 09-May-2018_13:44:25  Exp 7306  21   male    R     1   5    14
## 2031 09-May-2018_13:44:27  Exp 7306  21   male    R     1   5    15
## 2032 09-May-2018_13:44:29  Exp 7306  21   male    R     1   5    16
## 2033 09-May-2018_13:44:32  Exp 7306  21   male    R     1   5    17
## 2034 09-May-2018_13:44:34  Exp 7306  21   male    R     1   5    18
## 2035 09-May-2018_13:44:37  Exp 7306  21   male    R     1   5    19
## 2036 09-May-2018_13:44:39  Exp 7306  21   male    R     1   5    20
## 2037 09-May-2018_13:44:41  Exp 7306  21   male    R     1   5    21
## 2038 09-May-2018_13:44:44  Exp 7306  21   male    R     1   5    22
## 2039 09-May-2018_13:44:46  Exp 7306  21   male    R     1   5    23
## 2040 09-May-2018_13:44:49  Exp 7306  21   male    R     1   5    24
## 2041 09-May-2018_13:44:54  Exp 7306  21   male    R     2   1     1
## 2042 09-May-2018_13:44:57  Exp 7306  21   male    R     2   1     2
## 2043 09-May-2018_13:45:00  Exp 7306  21   male    R     2   1     3
## 2044 09-May-2018_13:45:02  Exp 7306  21   male    R     2   1     4
## 2045 09-May-2018_13:45:04  Exp 7306  21   male    R     2   1     5
## 2046 09-May-2018_13:45:07  Exp 7306  21   male    R     2   1     6
## 2047 09-May-2018_13:45:09  Exp 7306  21   male    R     2   1     7
## 2048 09-May-2018_13:45:12  Exp 7306  21   male    R     2   1     8
## 2049 09-May-2018_13:45:14  Exp 7306  21   male    R     2   1     9
## 2050 09-May-2018_13:45:16  Exp 7306  21   male    R     2   1    10
## 2051 09-May-2018_13:45:19  Exp 7306  21   male    R     2   1    11
## 2052 09-May-2018_13:45:21  Exp 7306  21   male    R     2   1    12
## 2053 09-May-2018_13:45:23  Exp 7306  21   male    R     2   1    13
## 2054 09-May-2018_13:45:25  Exp 7306  21   male    R     2   1    14
## 2055 09-May-2018_13:45:28  Exp 7306  21   male    R     2   1    15
## 2056 09-May-2018_13:45:30  Exp 7306  21   male    R     2   1    16
## 2057 09-May-2018_13:45:32  Exp 7306  21   male    R     2   1    17
## 2058 09-May-2018_13:45:34  Exp 7306  21   male    R     2   1    18
## 2059 09-May-2018_13:45:37  Exp 7306  21   male    R     2   1    19
## 2060 09-May-2018_13:45:39  Exp 7306  21   male    R     2   1    20
## 2061 09-May-2018_13:45:42  Exp 7306  21   male    R     2   1    21
## 2062 09-May-2018_13:45:44  Exp 7306  21   male    R     2   1    22
## 2063 09-May-2018_13:45:46  Exp 7306  21   male    R     2   1    23
## 2064 09-May-2018_13:45:49  Exp 7306  21   male    R     2   1    24
## 2065 09-May-2018_13:45:51  Exp 7306  21   male    R     2   2     1
## 2066 09-May-2018_13:45:54  Exp 7306  21   male    R     2   2     2
## 2067 09-May-2018_13:45:56  Exp 7306  21   male    R     2   2     3
## 2068 09-May-2018_13:45:59  Exp 7306  21   male    R     2   2     4
## 2069 09-May-2018_13:46:01  Exp 7306  21   male    R     2   2     5
## 2070 09-May-2018_13:46:03  Exp 7306  21   male    R     2   2     6
## 2071 09-May-2018_13:46:06  Exp 7306  21   male    R     2   2     7
## 2072 09-May-2018_13:46:08  Exp 7306  21   male    R     2   2     8
## 2073 09-May-2018_13:46:10  Exp 7306  21   male    R     2   2     9
## 2074 09-May-2018_13:46:13  Exp 7306  21   male    R     2   2    10
## 2075 09-May-2018_13:46:16  Exp 7306  21   male    R     2   2    11
## 2076 09-May-2018_13:46:18  Exp 7306  21   male    R     2   2    12
## 2077 09-May-2018_13:46:21  Exp 7306  21   male    R     2   2    13
## 2078 09-May-2018_13:46:23  Exp 7306  21   male    R     2   2    14
## 2079 09-May-2018_13:46:25  Exp 7306  21   male    R     2   2    15
## 2080 09-May-2018_13:46:28  Exp 7306  21   male    R     2   2    16
## 2081 09-May-2018_13:46:30  Exp 7306  21   male    R     2   2    17
## 2082 09-May-2018_13:46:32  Exp 7306  21   male    R     2   2    18
## 2083 09-May-2018_13:46:35  Exp 7306  21   male    R     2   2    19
## 2084 09-May-2018_13:46:37  Exp 7306  21   male    R     2   2    20
## 2085 09-May-2018_13:46:40  Exp 7306  21   male    R     2   2    21
## 2086 09-May-2018_13:46:42  Exp 7306  21   male    R     2   2    22
## 2087 09-May-2018_13:46:44  Exp 7306  21   male    R     2   2    23
## 2088 09-May-2018_13:46:47  Exp 7306  21   male    R     2   2    24
## 2089 09-May-2018_13:46:49  Exp 7306  21   male    R     2   3     1
## 2090 09-May-2018_13:46:52  Exp 7306  21   male    R     2   3     2
## 2091 09-May-2018_13:46:54  Exp 7306  21   male    R     2   3     3
## 2092 09-May-2018_13:46:56  Exp 7306  21   male    R     2   3     4
## 2093 09-May-2018_13:46:59  Exp 7306  21   male    R     2   3     5
## 2094 09-May-2018_13:47:01  Exp 7306  21   male    R     2   3     6
## 2095 09-May-2018_13:47:04  Exp 7306  21   male    R     2   3     7
## 2096 09-May-2018_13:47:06  Exp 7306  21   male    R     2   3     8
## 2097 09-May-2018_13:47:08  Exp 7306  21   male    R     2   3     9
## 2098 09-May-2018_13:47:11  Exp 7306  21   male    R     2   3    10
## 2099 09-May-2018_13:47:13  Exp 7306  21   male    R     2   3    11
## 2100 09-May-2018_13:47:15  Exp 7306  21   male    R     2   3    12
## 2101 09-May-2018_13:47:18  Exp 7306  21   male    R     2   3    13
## 2102 09-May-2018_13:47:20  Exp 7306  21   male    R     2   3    14
## 2103 09-May-2018_13:47:23  Exp 7306  21   male    R     2   3    15
## 2104 09-May-2018_13:47:25  Exp 7306  21   male    R     2   3    16
## 2105 09-May-2018_13:47:28  Exp 7306  21   male    R     2   3    17
## 2106 09-May-2018_13:47:30  Exp 7306  21   male    R     2   3    18
## 2107 09-May-2018_13:47:32  Exp 7306  21   male    R     2   3    19
## 2108 09-May-2018_13:47:35  Exp 7306  21   male    R     2   3    20
## 2109 09-May-2018_13:47:37  Exp 7306  21   male    R     2   3    21
## 2110 09-May-2018_13:47:39  Exp 7306  21   male    R     2   3    22
## 2111 09-May-2018_13:47:42  Exp 7306  21   male    R     2   3    23
## 2112 09-May-2018_13:47:44  Exp 7306  21   male    R     2   3    24
## 2113 09-May-2018_13:47:58  Exp 7306  21   male    R     2   4     1
## 2114 09-May-2018_13:48:00  Exp 7306  21   male    R     2   4     2
## 2115 09-May-2018_13:48:03  Exp 7306  21   male    R     2   4     3
## 2116 09-May-2018_13:48:05  Exp 7306  21   male    R     2   4     4
## 2117 09-May-2018_13:48:07  Exp 7306  21   male    R     2   4     5
## 2118 09-May-2018_13:48:10  Exp 7306  21   male    R     2   4     6
## 2119 09-May-2018_13:48:12  Exp 7306  21   male    R     2   4     7
## 2120 09-May-2018_13:48:15  Exp 7306  21   male    R     2   4     8
## 2121 09-May-2018_13:48:17  Exp 7306  21   male    R     2   4     9
## 2122 09-May-2018_13:48:19  Exp 7306  21   male    R     2   4    10
## 2123 09-May-2018_13:48:22  Exp 7306  21   male    R     2   4    11
## 2124 09-May-2018_13:48:24  Exp 7306  21   male    R     2   4    12
## 2125 09-May-2018_13:48:27  Exp 7306  21   male    R     2   4    13
## 2126 09-May-2018_13:48:29  Exp 7306  21   male    R     2   4    14
## 2127 09-May-2018_13:48:31  Exp 7306  21   male    R     2   4    15
## 2128 09-May-2018_13:48:34  Exp 7306  21   male    R     2   4    16
## 2129 09-May-2018_13:48:36  Exp 7306  21   male    R     2   4    17
## 2130 09-May-2018_13:48:38  Exp 7306  21   male    R     2   4    18
## 2131 09-May-2018_13:48:41  Exp 7306  21   male    R     2   4    19
## 2132 09-May-2018_13:48:43  Exp 7306  21   male    R     2   4    20
## 2133 09-May-2018_13:48:45  Exp 7306  21   male    R     2   4    21
## 2134 09-May-2018_13:48:48  Exp 7306  21   male    R     2   4    22
## 2135 09-May-2018_13:48:50  Exp 7306  21   male    R     2   4    23
## 2136 09-May-2018_13:48:52  Exp 7306  21   male    R     2   4    24
## 2137 09-May-2018_13:48:54  Exp 7306  21   male    R     2   5     1
## 2138 09-May-2018_13:48:57  Exp 7306  21   male    R     2   5     2
## 2139 09-May-2018_13:48:59  Exp 7306  21   male    R     2   5     3
## 2140 09-May-2018_13:49:02  Exp 7306  21   male    R     2   5     4
## 2141 09-May-2018_13:49:04  Exp 7306  21   male    R     2   5     5
## 2142 09-May-2018_13:49:06  Exp 7306  21   male    R     2   5     6
## 2143 09-May-2018_13:49:09  Exp 7306  21   male    R     2   5     7
## 2144 09-May-2018_13:49:11  Exp 7306  21   male    R     2   5     8
## 2145 09-May-2018_13:49:14  Exp 7306  21   male    R     2   5     9
## 2146 09-May-2018_13:49:16  Exp 7306  21   male    R     2   5    10
## 2147 09-May-2018_13:49:19  Exp 7306  21   male    R     2   5    11
## 2148 09-May-2018_13:49:21  Exp 7306  21   male    R     2   5    12
## 2149 09-May-2018_13:49:23  Exp 7306  21   male    R     2   5    13
## 2150 09-May-2018_13:49:26  Exp 7306  21   male    R     2   5    14
## 2151 09-May-2018_13:49:28  Exp 7306  21   male    R     2   5    15
## 2152 09-May-2018_13:49:31  Exp 7306  21   male    R     2   5    16
## 2153 09-May-2018_13:49:33  Exp 7306  21   male    R     2   5    17
## 2154 09-May-2018_13:49:36  Exp 7306  21   male    R     2   5    18
## 2155 09-May-2018_13:49:38  Exp 7306  21   male    R     2   5    19
## 2156 09-May-2018_13:49:40  Exp 7306  21   male    R     2   5    20
## 2157 09-May-2018_13:49:43  Exp 7306  21   male    R     2   5    21
## 2158 09-May-2018_13:49:45  Exp 7306  21   male    R     2   5    22
## 2159 09-May-2018_13:49:47  Exp 7306  21   male    R     2   5    23
## 2160 09-May-2018_13:49:50  Exp 7306  21   male    R     2   5    24
## 2161 09-May-2018_13:49:55  Exp 7306  21   male    R     3   1     1
## 2162 09-May-2018_13:49:58  Exp 7306  21   male    R     3   1     2
## 2163 09-May-2018_13:50:00  Exp 7306  21   male    R     3   1     3
## 2164 09-May-2018_13:50:03  Exp 7306  21   male    R     3   1     4
## 2165 09-May-2018_13:50:05  Exp 7306  21   male    R     3   1     5
## 2166 09-May-2018_13:50:08  Exp 7306  21   male    R     3   1     6
## 2167 09-May-2018_13:50:10  Exp 7306  21   male    R     3   1     7
## 2168 09-May-2018_13:50:12  Exp 7306  21   male    R     3   1     8
## 2169 09-May-2018_13:50:15  Exp 7306  21   male    R     3   1     9
## 2170 09-May-2018_13:50:17  Exp 7306  21   male    R     3   1    10
## 2171 09-May-2018_13:50:19  Exp 7306  21   male    R     3   1    11
## 2172 09-May-2018_13:50:22  Exp 7306  21   male    R     3   1    12
## 2173 09-May-2018_13:50:24  Exp 7306  21   male    R     3   1    13
## 2174 09-May-2018_13:50:27  Exp 7306  21   male    R     3   1    14
## 2175 09-May-2018_13:50:29  Exp 7306  21   male    R     3   1    15
## 2176 09-May-2018_13:50:31  Exp 7306  21   male    R     3   1    16
## 2177 09-May-2018_13:50:33  Exp 7306  21   male    R     3   1    17
## 2178 09-May-2018_13:50:36  Exp 7306  21   male    R     3   1    18
## 2179 09-May-2018_13:50:38  Exp 7306  21   male    R     3   1    19
## 2180 09-May-2018_13:50:41  Exp 7306  21   male    R     3   1    20
## 2181 09-May-2018_13:50:43  Exp 7306  21   male    R     3   1    21
## 2182 09-May-2018_13:50:46  Exp 7306  21   male    R     3   1    22
## 2183 09-May-2018_13:50:48  Exp 7306  21   male    R     3   1    23
## 2184 09-May-2018_13:50:51  Exp 7306  21   male    R     3   1    24
## 2185 09-May-2018_13:50:53  Exp 7306  21   male    R     3   2     1
## 2186 09-May-2018_13:50:55  Exp 7306  21   male    R     3   2     2
## 2187 09-May-2018_13:50:58  Exp 7306  21   male    R     3   2     3
## 2188 09-May-2018_13:51:00  Exp 7306  21   male    R     3   2     4
## 2189 09-May-2018_13:51:03  Exp 7306  21   male    R     3   2     5
## 2190 09-May-2018_13:51:05  Exp 7306  21   male    R     3   2     6
## 2191 09-May-2018_13:51:08  Exp 7306  21   male    R     3   2     7
## 2192 09-May-2018_13:51:10  Exp 7306  21   male    R     3   2     8
## 2193 09-May-2018_13:51:12  Exp 7306  21   male    R     3   2     9
## 2194 09-May-2018_13:51:15  Exp 7306  21   male    R     3   2    10
## 2195 09-May-2018_13:51:17  Exp 7306  21   male    R     3   2    11
## 2196 09-May-2018_13:51:19  Exp 7306  21   male    R     3   2    12
## 2197 09-May-2018_13:51:22  Exp 7306  21   male    R     3   2    13
## 2198 09-May-2018_13:51:24  Exp 7306  21   male    R     3   2    14
## 2199 09-May-2018_13:51:27  Exp 7306  21   male    R     3   2    15
## 2200 09-May-2018_13:51:29  Exp 7306  21   male    R     3   2    16
## 2201 09-May-2018_13:51:31  Exp 7306  21   male    R     3   2    17
## 2202 09-May-2018_13:51:34  Exp 7306  21   male    R     3   2    18
## 2203 09-May-2018_13:51:36  Exp 7306  21   male    R     3   2    19
## 2204 09-May-2018_13:51:39  Exp 7306  21   male    R     3   2    20
## 2205 09-May-2018_13:51:41  Exp 7306  21   male    R     3   2    21
## 2206 09-May-2018_13:51:43  Exp 7306  21   male    R     3   2    22
## 2207 09-May-2018_13:51:46  Exp 7306  21   male    R     3   2    23
## 2208 09-May-2018_13:51:48  Exp 7306  21   male    R     3   2    24
## 2209 09-May-2018_13:51:51  Exp 7306  21   male    R     3   3     1
## 2210 09-May-2018_13:51:53  Exp 7306  21   male    R     3   3     2
## 2211 09-May-2018_13:51:55  Exp 7306  21   male    R     3   3     3
## 2212 09-May-2018_13:51:58  Exp 7306  21   male    R     3   3     4
## 2213 09-May-2018_13:52:00  Exp 7306  21   male    R     3   3     5
## 2214 09-May-2018_13:52:03  Exp 7306  21   male    R     3   3     6
## 2215 09-May-2018_13:52:05  Exp 7306  21   male    R     3   3     7
## 2216 09-May-2018_13:52:08  Exp 7306  21   male    R     3   3     8
## 2217 09-May-2018_13:52:10  Exp 7306  21   male    R     3   3     9
## 2218 09-May-2018_13:52:12  Exp 7306  21   male    R     3   3    10
## 2219 09-May-2018_13:52:15  Exp 7306  21   male    R     3   3    11
## 2220 09-May-2018_13:52:17  Exp 7306  21   male    R     3   3    12
## 2221 09-May-2018_13:52:20  Exp 7306  21   male    R     3   3    13
## 2222 09-May-2018_13:52:22  Exp 7306  21   male    R     3   3    14
## 2223 09-May-2018_13:52:24  Exp 7306  21   male    R     3   3    15
## 2224 09-May-2018_13:52:27  Exp 7306  21   male    R     3   3    16
## 2225 09-May-2018_13:52:29  Exp 7306  21   male    R     3   3    17
## 2226 09-May-2018_13:52:31  Exp 7306  21   male    R     3   3    18
## 2227 09-May-2018_13:52:34  Exp 7306  21   male    R     3   3    19
## 2228 09-May-2018_13:52:36  Exp 7306  21   male    R     3   3    20
## 2229 09-May-2018_13:52:39  Exp 7306  21   male    R     3   3    21
## 2230 09-May-2018_13:52:41  Exp 7306  21   male    R     3   3    22
## 2231 09-May-2018_13:52:43  Exp 7306  21   male    R     3   3    23
## 2232 09-May-2018_13:52:46  Exp 7306  21   male    R     3   3    24
## 2233 09-May-2018_13:53:02  Exp 7306  21   male    R     3   4     1
## 2234 09-May-2018_13:53:05  Exp 7306  21   male    R     3   4     2
## 2235 09-May-2018_13:53:07  Exp 7306  21   male    R     3   4     3
## 2236 09-May-2018_13:53:10  Exp 7306  21   male    R     3   4     4
## 2237 09-May-2018_13:53:12  Exp 7306  21   male    R     3   4     5
## 2238 09-May-2018_13:53:14  Exp 7306  21   male    R     3   4     6
## 2239 09-May-2018_13:53:17  Exp 7306  21   male    R     3   4     7
## 2240 09-May-2018_13:53:19  Exp 7306  21   male    R     3   4     8
## 2241 09-May-2018_13:53:21  Exp 7306  21   male    R     3   4     9
## 2242 09-May-2018_13:53:24  Exp 7306  21   male    R     3   4    10
## 2243 09-May-2018_13:53:26  Exp 7306  21   male    R     3   4    11
## 2244 09-May-2018_13:53:28  Exp 7306  21   male    R     3   4    12
## 2245 09-May-2018_13:53:31  Exp 7306  21   male    R     3   4    13
## 2246 09-May-2018_13:53:33  Exp 7306  21   male    R     3   4    14
## 2247 09-May-2018_13:53:35  Exp 7306  21   male    R     3   4    15
## 2248 09-May-2018_13:53:38  Exp 7306  21   male    R     3   4    16
## 2249 09-May-2018_13:53:40  Exp 7306  21   male    R     3   4    17
## 2250 09-May-2018_13:53:43  Exp 7306  21   male    R     3   4    18
## 2251 09-May-2018_13:53:45  Exp 7306  21   male    R     3   4    19
## 2252 09-May-2018_13:53:47  Exp 7306  21   male    R     3   4    20
## 2253 09-May-2018_13:53:49  Exp 7306  21   male    R     3   4    21
## 2254 09-May-2018_13:53:52  Exp 7306  21   male    R     3   4    22
## 2255 09-May-2018_13:53:54  Exp 7306  21   male    R     3   4    23
## 2256 09-May-2018_13:53:56  Exp 7306  21   male    R     3   4    24
## 2257 09-May-2018_13:53:59  Exp 7306  21   male    R     3   5     1
## 2258 09-May-2018_13:54:01  Exp 7306  21   male    R     3   5     2
## 2259 09-May-2018_13:54:03  Exp 7306  21   male    R     3   5     3
## 2260 09-May-2018_13:54:06  Exp 7306  21   male    R     3   5     4
## 2261 09-May-2018_13:54:08  Exp 7306  21   male    R     3   5     5
## 2262 09-May-2018_13:54:11  Exp 7306  21   male    R     3   5     6
## 2263 09-May-2018_13:54:13  Exp 7306  21   male    R     3   5     7
## 2264 09-May-2018_13:54:16  Exp 7306  21   male    R     3   5     8
## 2265 09-May-2018_13:54:18  Exp 7306  21   male    R     3   5     9
## 2266 09-May-2018_13:54:20  Exp 7306  21   male    R     3   5    10
## 2267 09-May-2018_13:54:23  Exp 7306  21   male    R     3   5    11
## 2268 09-May-2018_13:54:25  Exp 7306  21   male    R     3   5    12
## 2269 09-May-2018_13:54:27  Exp 7306  21   male    R     3   5    13
## 2270 09-May-2018_13:54:30  Exp 7306  21   male    R     3   5    14
## 2271 09-May-2018_13:54:32  Exp 7306  21   male    R     3   5    15
## 2272 09-May-2018_13:54:34  Exp 7306  21   male    R     3   5    16
## 2273 09-May-2018_13:54:37  Exp 7306  21   male    R     3   5    17
## 2274 09-May-2018_13:54:39  Exp 7306  21   male    R     3   5    18
## 2275 09-May-2018_13:54:41  Exp 7306  21   male    R     3   5    19
## 2276 09-May-2018_13:54:44  Exp 7306  21   male    R     3   5    20
## 2277 09-May-2018_13:54:46  Exp 7306  21   male    R     3   5    21
## 2278 09-May-2018_13:54:48  Exp 7306  21   male    R     3   5    22
## 2279 09-May-2018_13:54:51  Exp 7306  21   male    R     3   5    23
## 2280 09-May-2018_13:54:53  Exp 7306  21   male    R     3   5    24
## 2281 09-May-2018_14:00:28  Exp 7306  21   male    R     1   1     1
## 2282 09-May-2018_14:00:30  Exp 7306  21   male    R     1   1     2
## 2283 09-May-2018_14:00:33  Exp 7306  21   male    R     1   1     3
## 2284 09-May-2018_14:00:36  Exp 7306  21   male    R     1   1     4
## 2285 09-May-2018_14:00:38  Exp 7306  21   male    R     1   1     5
## 2286 09-May-2018_14:00:40  Exp 7306  21   male    R     1   1     6
## 2287 09-May-2018_14:00:43  Exp 7306  21   male    R     1   1     7
## 2288 09-May-2018_14:00:45  Exp 7306  21   male    R     1   1     8
## 2289 09-May-2018_14:00:47  Exp 7306  21   male    R     1   1     9
## 2290 09-May-2018_14:00:50  Exp 7306  21   male    R     1   1    10
## 2291 09-May-2018_14:00:52  Exp 7306  21   male    R     1   1    11
## 2292 09-May-2018_14:00:55  Exp 7306  21   male    R     1   1    12
## 2293 09-May-2018_14:00:57  Exp 7306  21   male    R     1   1    13
## 2294 09-May-2018_14:00:59  Exp 7306  21   male    R     1   1    14
## 2295 09-May-2018_14:01:02  Exp 7306  21   male    R     1   1    15
## 2296 09-May-2018_14:01:04  Exp 7306  21   male    R     1   1    16
## 2297 09-May-2018_14:01:07  Exp 7306  21   male    R     1   1    17
## 2298 09-May-2018_14:01:09  Exp 7306  21   male    R     1   1    18
## 2299 09-May-2018_14:01:11  Exp 7306  21   male    R     1   1    19
## 2300 09-May-2018_14:01:14  Exp 7306  21   male    R     1   1    20
## 2301 09-May-2018_14:01:16  Exp 7306  21   male    R     1   1    21
## 2302 09-May-2018_14:01:18  Exp 7306  21   male    R     1   1    22
## 2303 09-May-2018_14:01:21  Exp 7306  21   male    R     1   1    23
## 2304 09-May-2018_14:01:23  Exp 7306  21   male    R     1   1    24
## 2305 09-May-2018_14:01:26  Exp 7306  21   male    R     1   2     1
## 2306 09-May-2018_14:01:28  Exp 7306  21   male    R     1   2     2
## 2307 09-May-2018_14:01:30  Exp 7306  21   male    R     1   2     3
## 2308 09-May-2018_14:01:33  Exp 7306  21   male    R     1   2     4
## 2309 09-May-2018_14:01:35  Exp 7306  21   male    R     1   2     5
## 2310 09-May-2018_14:01:37  Exp 7306  21   male    R     1   2     6
## 2311 09-May-2018_14:01:40  Exp 7306  21   male    R     1   2     7
## 2312 09-May-2018_14:01:42  Exp 7306  21   male    R     1   2     8
## 2313 09-May-2018_14:01:45  Exp 7306  21   male    R     1   2     9
## 2314 09-May-2018_14:01:47  Exp 7306  21   male    R     1   2    10
## 2315 09-May-2018_14:01:49  Exp 7306  21   male    R     1   2    11
## 2316 09-May-2018_14:01:52  Exp 7306  21   male    R     1   2    12
## 2317 09-May-2018_14:01:54  Exp 7306  21   male    R     1   2    13
## 2318 09-May-2018_14:01:56  Exp 7306  21   male    R     1   2    14
## 2319 09-May-2018_14:01:59  Exp 7306  21   male    R     1   2    15
## 2320 09-May-2018_14:02:01  Exp 7306  21   male    R     1   2    16
## 2321 09-May-2018_14:02:03  Exp 7306  21   male    R     1   2    17
## 2322 09-May-2018_14:02:06  Exp 7306  21   male    R     1   2    18
## 2323 09-May-2018_14:02:08  Exp 7306  21   male    R     1   2    19
## 2324 09-May-2018_14:02:10  Exp 7306  21   male    R     1   2    20
## 2325 09-May-2018_14:02:13  Exp 7306  21   male    R     1   2    21
## 2326 09-May-2018_14:02:15  Exp 7306  21   male    R     1   2    22
## 2327 09-May-2018_14:02:18  Exp 7306  21   male    R     1   2    23
## 2328 09-May-2018_14:02:20  Exp 7306  21   male    R     1   2    24
## 2329 09-May-2018_14:07:01  Exp 7306  21   male    R     1   1     1
## 2330 09-May-2018_14:07:04  Exp 7306  21   male    R     1   1     2
## 2331 09-May-2018_14:07:06  Exp 7306  21   male    R     1   1     3
## 2332 09-May-2018_14:07:09  Exp 7306  21   male    R     1   1     4
## 2333 09-May-2018_14:07:11  Exp 7306  21   male    R     1   1     5
## 2334 09-May-2018_14:07:14  Exp 7306  21   male    R     1   1     6
## 2335 09-May-2018_14:07:16  Exp 7306  21   male    R     1   1     7
## 2336 09-May-2018_14:07:18  Exp 7306  21   male    R     1   1     8
## 2337 09-May-2018_14:07:21  Exp 7306  21   male    R     1   1     9
## 2338 09-May-2018_14:07:23  Exp 7306  21   male    R     1   1    10
## 2339 09-May-2018_14:07:26  Exp 7306  21   male    R     1   1    11
## 2340 09-May-2018_14:07:28  Exp 7306  21   male    R     1   1    12
## 2341 09-May-2018_14:07:30  Exp 7306  21   male    R     1   1    13
## 2342 09-May-2018_14:07:33  Exp 7306  21   male    R     1   1    14
## 2343 09-May-2018_14:07:35  Exp 7306  21   male    R     1   1    15
## 2344 09-May-2018_14:07:37  Exp 7306  21   male    R     1   1    16
## 2345 09-May-2018_14:07:40  Exp 7306  21   male    R     1   1    17
## 2346 09-May-2018_14:07:42  Exp 7306  21   male    R     1   1    18
## 2347 09-May-2018_14:07:45  Exp 7306  21   male    R     1   1    19
## 2348 09-May-2018_14:07:47  Exp 7306  21   male    R     1   1    20
## 2349 09-May-2018_14:07:49  Exp 7306  21   male    R     1   1    21
## 2350 09-May-2018_14:07:52  Exp 7306  21   male    R     1   1    22
## 2351 09-May-2018_14:07:54  Exp 7306  21   male    R     1   1    23
## 2352 09-May-2018_14:07:57  Exp 7306  21   male    R     1   1    24
## 2353 09-May-2018_14:07:59  Exp 7306  21   male    R     1   2     1
## 2354 09-May-2018_14:08:01  Exp 7306  21   male    R     1   2     2
## 2355 09-May-2018_14:08:04  Exp 7306  21   male    R     1   2     3
## 2356 09-May-2018_14:08:06  Exp 7306  21   male    R     1   2     4
## 2357 09-May-2018_14:08:08  Exp 7306  21   male    R     1   2     5
## 2358 09-May-2018_14:08:10  Exp 7306  21   male    R     1   2     6
## 2359 09-May-2018_14:08:13  Exp 7306  21   male    R     1   2     7
## 2360 09-May-2018_14:08:15  Exp 7306  21   male    R     1   2     8
## 2361 09-May-2018_14:08:18  Exp 7306  21   male    R     1   2     9
## 2362 09-May-2018_14:08:20  Exp 7306  21   male    R     1   2    10
## 2363 09-May-2018_14:08:22  Exp 7306  21   male    R     1   2    11
## 2364 09-May-2018_14:08:25  Exp 7306  21   male    R     1   2    12
## 2365 09-May-2018_14:08:27  Exp 7306  21   male    R     1   2    13
## 2366 09-May-2018_14:08:29  Exp 7306  21   male    R     1   2    14
## 2367 09-May-2018_14:08:32  Exp 7306  21   male    R     1   2    15
## 2368 09-May-2018_14:08:34  Exp 7306  21   male    R     1   2    16
## 2369 09-May-2018_14:08:37  Exp 7306  21   male    R     1   2    17
## 2370 09-May-2018_14:08:39  Exp 7306  21   male    R     1   2    18
## 2371 09-May-2018_14:08:42  Exp 7306  21   male    R     1   2    19
## 2372 09-May-2018_14:08:44  Exp 7306  21   male    R     1   2    20
## 2373 09-May-2018_14:08:47  Exp 7306  21   male    R     1   2    21
## 2374 09-May-2018_14:08:49  Exp 7306  21   male    R     1   2    22
## 2375 09-May-2018_14:08:51  Exp 7306  21   male    R     1   2    23
## 2376 09-May-2018_14:08:54  Exp 7306  21   male    R     1   2    24
## 2377 09-May-2018_14:13:54  Exp 7306  21   male    R     1   1     1
## 2378 09-May-2018_14:13:57  Exp 7306  21   male    R     1   1     2
## 2379 09-May-2018_14:13:59  Exp 7306  21   male    R     1   1     3
## 2380 09-May-2018_14:14:01  Exp 7306  21   male    R     1   1     4
## 2381 09-May-2018_14:14:04  Exp 7306  21   male    R     1   1     5
## 2382 09-May-2018_14:14:06  Exp 7306  21   male    R     1   1     6
## 2383 09-May-2018_14:14:09  Exp 7306  21   male    R     1   1     7
## 2384 09-May-2018_14:14:11  Exp 7306  21   male    R     1   1     8
## 2385 09-May-2018_14:14:13  Exp 7306  21   male    R     1   1     9
## 2386 09-May-2018_14:14:16  Exp 7306  21   male    R     1   1    10
## 2387 09-May-2018_14:14:18  Exp 7306  21   male    R     1   1    11
## 2388 09-May-2018_14:14:21  Exp 7306  21   male    R     1   1    12
## 2389 09-May-2018_14:14:23  Exp 7306  21   male    R     1   1    13
## 2390 09-May-2018_14:14:25  Exp 7306  21   male    R     1   1    14
## 2391 09-May-2018_14:14:28  Exp 7306  21   male    R     1   1    15
## 2392 09-May-2018_14:14:30  Exp 7306  21   male    R     1   1    16
## 2393 09-May-2018_14:14:32  Exp 7306  21   male    R     1   1    17
## 2394 09-May-2018_14:14:35  Exp 7306  21   male    R     1   1    18
## 2395 09-May-2018_14:14:37  Exp 7306  21   male    R     1   1    19
## 2396 09-May-2018_14:14:40  Exp 7306  21   male    R     1   1    20
## 2397 09-May-2018_14:14:42  Exp 7306  21   male    R     1   1    21
## 2398 09-May-2018_14:14:45  Exp 7306  21   male    R     1   1    22
## 2399 09-May-2018_14:14:47  Exp 7306  21   male    R     1   1    23
## 2400 09-May-2018_14:14:50  Exp 7306  21   male    R     1   1    24
## 2401 09-May-2018_14:14:52  Exp 7306  21   male    R     1   2     1
## 2402 09-May-2018_14:14:55  Exp 7306  21   male    R     1   2     2
## 2403 09-May-2018_14:14:57  Exp 7306  21   male    R     1   2     3
## 2404 09-May-2018_14:15:00  Exp 7306  21   male    R     1   2     4
## 2405 09-May-2018_14:15:02  Exp 7306  21   male    R     1   2     5
## 2406 09-May-2018_14:15:05  Exp 7306  21   male    R     1   2     6
## 2407 09-May-2018_14:15:07  Exp 7306  21   male    R     1   2     7
## 2408 09-May-2018_14:15:10  Exp 7306  21   male    R     1   2     8
## 2409 09-May-2018_14:15:12  Exp 7306  21   male    R     1   2     9
## 2410 09-May-2018_14:15:14  Exp 7306  21   male    R     1   2    10
## 2411 09-May-2018_14:15:17  Exp 7306  21   male    R     1   2    11
## 2412 09-May-2018_14:15:19  Exp 7306  21   male    R     1   2    12
## 2413 09-May-2018_14:15:21  Exp 7306  21   male    R     1   2    13
## 2414 09-May-2018_14:15:24  Exp 7306  21   male    R     1   2    14
## 2415 09-May-2018_14:15:26  Exp 7306  21   male    R     1   2    15
## 2416 09-May-2018_14:15:28  Exp 7306  21   male    R     1   2    16
## 2417 09-May-2018_14:15:31  Exp 7306  21   male    R     1   2    17
## 2418 09-May-2018_14:15:33  Exp 7306  21   male    R     1   2    18
## 2419 09-May-2018_14:15:36  Exp 7306  21   male    R     1   2    19
## 2420 09-May-2018_14:15:38  Exp 7306  21   male    R     1   2    20
## 2421 09-May-2018_14:15:40  Exp 7306  21   male    R     1   2    21
## 2422 09-May-2018_14:15:42  Exp 7306  21   male    R     1   2    22
## 2423 09-May-2018_14:15:45  Exp 7306  21   male    R     1   2    23
## 2424 09-May-2018_14:15:47  Exp 7306  21   male    R     1   2    24
## 2425 09-May-2018_14:20:21  Exp 7306  21   male    R     1   1     1
## 2426 09-May-2018_14:20:24  Exp 7306  21   male    R     1   1     2
## 2427 09-May-2018_14:20:26  Exp 7306  21   male    R     1   1     3
## 2428 09-May-2018_14:20:29  Exp 7306  21   male    R     1   1     4
## 2429 09-May-2018_14:20:31  Exp 7306  21   male    R     1   1     5
## 2430 09-May-2018_14:20:33  Exp 7306  21   male    R     1   1     6
## 2431 09-May-2018_14:20:35  Exp 7306  21   male    R     1   1     7
## 2432 09-May-2018_14:20:38  Exp 7306  21   male    R     1   1     8
## 2433 09-May-2018_14:20:40  Exp 7306  21   male    R     1   1     9
## 2434 09-May-2018_14:20:42  Exp 7306  21   male    R     1   1    10
## 2435 09-May-2018_14:20:45  Exp 7306  21   male    R     1   1    11
## 2436 09-May-2018_14:20:47  Exp 7306  21   male    R     1   1    12
## 2437 09-May-2018_14:20:50  Exp 7306  21   male    R     1   1    13
## 2438 09-May-2018_14:20:52  Exp 7306  21   male    R     1   1    14
## 2439 09-May-2018_14:20:55  Exp 7306  21   male    R     1   1    15
## 2440 09-May-2018_14:20:57  Exp 7306  21   male    R     1   1    16
## 2441 09-May-2018_14:20:59  Exp 7306  21   male    R     1   1    17
## 2442 09-May-2018_14:21:02  Exp 7306  21   male    R     1   1    18
## 2443 09-May-2018_14:21:04  Exp 7306  21   male    R     1   1    19
## 2444 09-May-2018_14:21:06  Exp 7306  21   male    R     1   1    20
## 2445 09-May-2018_14:21:09  Exp 7306  21   male    R     1   1    21
## 2446 09-May-2018_14:21:11  Exp 7306  21   male    R     1   1    22
## 2447 09-May-2018_14:21:14  Exp 7306  21   male    R     1   1    23
## 2448 09-May-2018_14:21:16  Exp 7306  21   male    R     1   1    24
## 2449 09-May-2018_14:21:18  Exp 7306  21   male    R     1   2     1
## 2450 09-May-2018_14:21:20  Exp 7306  21   male    R     1   2     2
## 2451 09-May-2018_14:21:23  Exp 7306  21   male    R     1   2     3
## 2452 09-May-2018_14:21:25  Exp 7306  21   male    R     1   2     4
## 2453 09-May-2018_14:21:28  Exp 7306  21   male    R     1   2     5
## 2454 09-May-2018_14:21:30  Exp 7306  21   male    R     1   2     6
## 2455 09-May-2018_14:21:32  Exp 7306  21   male    R     1   2     7
## 2456 09-May-2018_14:21:35  Exp 7306  21   male    R     1   2     8
## 2457 09-May-2018_14:21:37  Exp 7306  21   male    R     1   2     9
## 2458 09-May-2018_14:21:40  Exp 7306  21   male    R     1   2    10
## 2459 09-May-2018_14:21:42  Exp 7306  21   male    R     1   2    11
## 2460 09-May-2018_14:21:44  Exp 7306  21   male    R     1   2    12
## 2461 09-May-2018_14:21:47  Exp 7306  21   male    R     1   2    13
## 2462 09-May-2018_14:21:49  Exp 7306  21   male    R     1   2    14
## 2463 09-May-2018_14:21:51  Exp 7306  21   male    R     1   2    15
## 2464 09-May-2018_14:21:54  Exp 7306  21   male    R     1   2    16
## 2465 09-May-2018_14:21:56  Exp 7306  21   male    R     1   2    17
## 2466 09-May-2018_14:21:58  Exp 7306  21   male    R     1   2    18
## 2467 09-May-2018_14:22:01  Exp 7306  21   male    R     1   2    19
## 2468 09-May-2018_14:22:03  Exp 7306  21   male    R     1   2    20
## 2469 09-May-2018_14:22:05  Exp 7306  21   male    R     1   2    21
## 2470 09-May-2018_14:22:08  Exp 7306  21   male    R     1   2    22
## 2471 09-May-2018_14:22:10  Exp 7306  21   male    R     1   2    23
## 2472 09-May-2018_14:22:12  Exp 7306  21   male    R     1   2    24
## 2473 09-May-2018_14:27:01  Exp 7306  21   male    R     1   1     1
## 2474 09-May-2018_14:27:03  Exp 7306  21   male    R     1   1     2
## 2475 09-May-2018_14:27:06  Exp 7306  21   male    R     1   1     3
## 2476 09-May-2018_14:27:08  Exp 7306  21   male    R     1   1     4
## 2477 09-May-2018_14:27:10  Exp 7306  21   male    R     1   1     5
## 2478 09-May-2018_14:27:12  Exp 7306  21   male    R     1   1     6
## 2479 09-May-2018_14:27:15  Exp 7306  21   male    R     1   1     7
## 2480 09-May-2018_14:27:17  Exp 7306  21   male    R     1   1     8
## 2481 09-May-2018_14:27:20  Exp 7306  21   male    R     1   1     9
## 2482 09-May-2018_14:27:22  Exp 7306  21   male    R     1   1    10
## 2483 09-May-2018_14:27:24  Exp 7306  21   male    R     1   1    11
## 2484 09-May-2018_14:27:27  Exp 7306  21   male    R     1   1    12
## 2485 09-May-2018_14:27:29  Exp 7306  21   male    R     1   1    13
## 2486 09-May-2018_14:27:32  Exp 7306  21   male    R     1   1    14
## 2487 09-May-2018_14:27:34  Exp 7306  21   male    R     1   1    15
## 2488 09-May-2018_14:27:36  Exp 7306  21   male    R     1   1    16
## 2489 09-May-2018_14:27:39  Exp 7306  21   male    R     1   1    17
## 2490 09-May-2018_14:27:41  Exp 7306  21   male    R     1   1    18
## 2491 09-May-2018_14:27:44  Exp 7306  21   male    R     1   1    19
## 2492 09-May-2018_14:27:46  Exp 7306  21   male    R     1   1    20
## 2493 09-May-2018_14:27:48  Exp 7306  21   male    R     1   1    21
## 2494 09-May-2018_14:27:50  Exp 7306  21   male    R     1   1    22
## 2495 09-May-2018_14:27:53  Exp 7306  21   male    R     1   1    23
## 2496 09-May-2018_14:27:55  Exp 7306  21   male    R     1   1    24
## 2497 09-May-2018_14:27:57  Exp 7306  21   male    R     1   2     1
## 2498 09-May-2018_14:27:59  Exp 7306  21   male    R     1   2     2
## 2499 09-May-2018_14:28:02  Exp 7306  21   male    R     1   2     3
## 2500 09-May-2018_14:28:04  Exp 7306  21   male    R     1   2     4
## 2501 09-May-2018_14:28:06  Exp 7306  21   male    R     1   2     5
## 2502 09-May-2018_14:28:09  Exp 7306  21   male    R     1   2     6
## 2503 09-May-2018_14:28:11  Exp 7306  21   male    R     1   2     7
## 2504 09-May-2018_14:28:14  Exp 7306  21   male    R     1   2     8
## 2505 09-May-2018_14:28:16  Exp 7306  21   male    R     1   2     9
## 2506 09-May-2018_14:28:19  Exp 7306  21   male    R     1   2    10
## 2507 09-May-2018_14:28:21  Exp 7306  21   male    R     1   2    11
## 2508 09-May-2018_14:28:23  Exp 7306  21   male    R     1   2    12
## 2509 09-May-2018_14:28:25  Exp 7306  21   male    R     1   2    13
## 2510 09-May-2018_14:28:28  Exp 7306  21   male    R     1   2    14
## 2511 09-May-2018_14:28:30  Exp 7306  21   male    R     1   2    15
## 2512 09-May-2018_14:28:33  Exp 7306  21   male    R     1   2    16
## 2513 09-May-2018_14:28:35  Exp 7306  21   male    R     1   2    17
## 2514 09-May-2018_14:28:38  Exp 7306  21   male    R     1   2    18
## 2515 09-May-2018_14:28:40  Exp 7306  21   male    R     1   2    19
## 2516 09-May-2018_14:28:42  Exp 7306  21   male    R     1   2    20
## 2517 09-May-2018_14:28:45  Exp 7306  21   male    R     1   2    21
## 2518 09-May-2018_14:28:47  Exp 7306  21   male    R     1   2    22
## 2519 09-May-2018_14:28:49  Exp 7306  21   male    R     1   2    23
## 2520 09-May-2018_14:28:52  Exp 7306  21   male    R     1   2    24
## 2521 09-May-2018_13:46:30  Exp 7307  22 female    R     1   1     1
## 2522 09-May-2018_13:46:32  Exp 7307  22 female    R     1   1     2
## 2523 09-May-2018_13:46:34  Exp 7307  22 female    R     1   1     3
## 2524 09-May-2018_13:46:37  Exp 7307  22 female    R     1   1     4
## 2525 09-May-2018_13:46:39  Exp 7307  22 female    R     1   1     5
## 2526 09-May-2018_13:46:42  Exp 7307  22 female    R     1   1     6
## 2527 09-May-2018_13:46:44  Exp 7307  22 female    R     1   1     7
## 2528 09-May-2018_13:46:47  Exp 7307  22 female    R     1   1     8
## 2529 09-May-2018_13:46:50  Exp 7307  22 female    R     1   1     9
## 2530 09-May-2018_13:46:52  Exp 7307  22 female    R     1   1    10
## 2531 09-May-2018_13:46:55  Exp 7307  22 female    R     1   1    11
## 2532 09-May-2018_13:46:57  Exp 7307  22 female    R     1   1    12
## 2533 09-May-2018_13:47:00  Exp 7307  22 female    R     1   1    13
## 2534 09-May-2018_13:47:02  Exp 7307  22 female    R     1   1    14
## 2535 09-May-2018_13:47:04  Exp 7307  22 female    R     1   1    15
## 2536 09-May-2018_13:47:07  Exp 7307  22 female    R     1   1    16
## 2537 09-May-2018_13:47:09  Exp 7307  22 female    R     1   1    17
## 2538 09-May-2018_13:47:12  Exp 7307  22 female    R     1   1    18
## 2539 09-May-2018_13:47:14  Exp 7307  22 female    R     1   1    19
## 2540 09-May-2018_13:47:16  Exp 7307  22 female    R     1   1    20
## 2541 09-May-2018_13:47:18  Exp 7307  22 female    R     1   1    21
## 2542 09-May-2018_13:47:21  Exp 7307  22 female    R     1   1    22
## 2543 09-May-2018_13:47:23  Exp 7307  22 female    R     1   1    23
## 2544 09-May-2018_13:47:25  Exp 7307  22 female    R     1   1    24
## 2545 09-May-2018_13:47:28  Exp 7307  22 female    R     1   2     1
## 2546 09-May-2018_13:47:30  Exp 7307  22 female    R     1   2     2
## 2547 09-May-2018_13:47:32  Exp 7307  22 female    R     1   2     3
## 2548 09-May-2018_13:47:35  Exp 7307  22 female    R     1   2     4
## 2549 09-May-2018_13:47:37  Exp 7307  22 female    R     1   2     5
## 2550 09-May-2018_13:47:40  Exp 7307  22 female    R     1   2     6
## 2551 09-May-2018_13:47:42  Exp 7307  22 female    R     1   2     7
## 2552 09-May-2018_13:47:45  Exp 7307  22 female    R     1   2     8
## 2553 09-May-2018_13:47:47  Exp 7307  22 female    R     1   2     9
## 2554 09-May-2018_13:47:50  Exp 7307  22 female    R     1   2    10
## 2555 09-May-2018_13:47:52  Exp 7307  22 female    R     1   2    11
## 2556 09-May-2018_13:47:54  Exp 7307  22 female    R     1   2    12
## 2557 09-May-2018_13:47:57  Exp 7307  22 female    R     1   2    13
## 2558 09-May-2018_13:48:00  Exp 7307  22 female    R     1   2    14
## 2559 09-May-2018_13:48:02  Exp 7307  22 female    R     1   2    15
## 2560 09-May-2018_13:48:05  Exp 7307  22 female    R     1   2    16
## 2561 09-May-2018_13:48:07  Exp 7307  22 female    R     1   2    17
## 2562 09-May-2018_13:48:10  Exp 7307  22 female    R     1   2    18
## 2563 09-May-2018_13:48:12  Exp 7307  22 female    R     1   2    19
## 2564 09-May-2018_13:48:14  Exp 7307  22 female    R     1   2    20
## 2565 09-May-2018_13:48:17  Exp 7307  22 female    R     1   2    21
## 2566 09-May-2018_13:48:19  Exp 7307  22 female    R     1   2    22
## 2567 09-May-2018_13:48:22  Exp 7307  22 female    R     1   2    23
## 2568 09-May-2018_13:48:25  Exp 7307  22 female    R     1   2    24
## 2569 09-May-2018_13:48:27  Exp 7307  22 female    R     1   3     1
## 2570 09-May-2018_13:48:29  Exp 7307  22 female    R     1   3     2
## 2571 09-May-2018_13:48:32  Exp 7307  22 female    R     1   3     3
## 2572 09-May-2018_13:48:34  Exp 7307  22 female    R     1   3     4
## 2573 09-May-2018_13:48:36  Exp 7307  22 female    R     1   3     5
## 2574 09-May-2018_13:48:38  Exp 7307  22 female    R     1   3     6
## 2575 09-May-2018_13:48:41  Exp 7307  22 female    R     1   3     7
## 2576 09-May-2018_13:48:44  Exp 7307  22 female    R     1   3     8
## 2577 09-May-2018_13:48:46  Exp 7307  22 female    R     1   3     9
## 2578 09-May-2018_13:48:48  Exp 7307  22 female    R     1   3    10
## 2579 09-May-2018_13:48:51  Exp 7307  22 female    R     1   3    11
## 2580 09-May-2018_13:48:53  Exp 7307  22 female    R     1   3    12
## 2581 09-May-2018_13:48:55  Exp 7307  22 female    R     1   3    13
## 2582 09-May-2018_13:48:57  Exp 7307  22 female    R     1   3    14
## 2583 09-May-2018_13:48:59  Exp 7307  22 female    R     1   3    15
## 2584 09-May-2018_13:49:02  Exp 7307  22 female    R     1   3    16
## 2585 09-May-2018_13:49:04  Exp 7307  22 female    R     1   3    17
## 2586 09-May-2018_13:49:06  Exp 7307  22 female    R     1   3    18
## 2587 09-May-2018_13:49:09  Exp 7307  22 female    R     1   3    19
## 2588 09-May-2018_13:49:11  Exp 7307  22 female    R     1   3    20
## 2589 09-May-2018_13:49:13  Exp 7307  22 female    R     1   3    21
## 2590 09-May-2018_13:49:16  Exp 7307  22 female    R     1   3    22
## 2591 09-May-2018_13:49:18  Exp 7307  22 female    R     1   3    23
## 2592 09-May-2018_13:49:20  Exp 7307  22 female    R     1   3    24
## 2593 09-May-2018_13:49:45  Exp 7307  22 female    R     1   4     1
## 2594 09-May-2018_13:49:47  Exp 7307  22 female    R     1   4     2
## 2595 09-May-2018_13:49:49  Exp 7307  22 female    R     1   4     3
## 2596 09-May-2018_13:49:52  Exp 7307  22 female    R     1   4     4
## 2597 09-May-2018_13:49:54  Exp 7307  22 female    R     1   4     5
## 2598 09-May-2018_13:49:56  Exp 7307  22 female    R     1   4     6
## 2599 09-May-2018_13:49:59  Exp 7307  22 female    R     1   4     7
## 2600 09-May-2018_13:50:01  Exp 7307  22 female    R     1   4     8
## 2601 09-May-2018_13:50:03  Exp 7307  22 female    R     1   4     9
## 2602 09-May-2018_13:50:05  Exp 7307  22 female    R     1   4    10
## 2603 09-May-2018_13:50:07  Exp 7307  22 female    R     1   4    11
## 2604 09-May-2018_13:50:10  Exp 7307  22 female    R     1   4    12
## 2605 09-May-2018_13:50:12  Exp 7307  22 female    R     1   4    13
## 2606 09-May-2018_13:50:15  Exp 7307  22 female    R     1   4    14
## 2607 09-May-2018_13:50:17  Exp 7307  22 female    R     1   4    15
## 2608 09-May-2018_13:50:19  Exp 7307  22 female    R     1   4    16
## 2609 09-May-2018_13:50:22  Exp 7307  22 female    R     1   4    17
## 2610 09-May-2018_13:50:24  Exp 7307  22 female    R     1   4    18
## 2611 09-May-2018_13:50:26  Exp 7307  22 female    R     1   4    19
## 2612 09-May-2018_13:50:29  Exp 7307  22 female    R     1   4    20
## 2613 09-May-2018_13:50:32  Exp 7307  22 female    R     1   4    21
## 2614 09-May-2018_13:50:34  Exp 7307  22 female    R     1   4    22
## 2615 09-May-2018_13:50:37  Exp 7307  22 female    R     1   4    23
## 2616 09-May-2018_13:50:39  Exp 7307  22 female    R     1   4    24
## 2617 09-May-2018_13:50:41  Exp 7307  22 female    R     1   5     1
## 2618 09-May-2018_13:50:44  Exp 7307  22 female    R     1   5     2
## 2619 09-May-2018_13:50:46  Exp 7307  22 female    R     1   5     3
## 2620 09-May-2018_13:50:49  Exp 7307  22 female    R     1   5     4
## 2621 09-May-2018_13:50:51  Exp 7307  22 female    R     1   5     5
## 2622 09-May-2018_13:50:53  Exp 7307  22 female    R     1   5     6
## 2623 09-May-2018_13:50:56  Exp 7307  22 female    R     1   5     7
## 2624 09-May-2018_13:50:58  Exp 7307  22 female    R     1   5     8
## 2625 09-May-2018_13:51:00  Exp 7307  22 female    R     1   5     9
## 2626 09-May-2018_13:51:02  Exp 7307  22 female    R     1   5    10
## 2627 09-May-2018_13:51:05  Exp 7307  22 female    R     1   5    11
## 2628 09-May-2018_13:51:07  Exp 7307  22 female    R     1   5    12
## 2629 09-May-2018_13:51:09  Exp 7307  22 female    R     1   5    13
## 2630 09-May-2018_13:51:12  Exp 7307  22 female    R     1   5    14
## 2631 09-May-2018_13:51:14  Exp 7307  22 female    R     1   5    15
## 2632 09-May-2018_13:51:16  Exp 7307  22 female    R     1   5    16
## 2633 09-May-2018_13:51:19  Exp 7307  22 female    R     1   5    17
## 2634 09-May-2018_13:51:21  Exp 7307  22 female    R     1   5    18
## 2635 09-May-2018_13:51:23  Exp 7307  22 female    R     1   5    19
## 2636 09-May-2018_13:51:26  Exp 7307  22 female    R     1   5    20
## 2637 09-May-2018_13:51:28  Exp 7307  22 female    R     1   5    21
## 2638 09-May-2018_13:51:31  Exp 7307  22 female    R     1   5    22
## 2639 09-May-2018_13:51:33  Exp 7307  22 female    R     1   5    23
## 2640 09-May-2018_13:51:35  Exp 7307  22 female    R     1   5    24
## 2641 09-May-2018_13:51:41  Exp 7307  22 female    R     2   1     1
## 2642 09-May-2018_13:51:44  Exp 7307  22 female    R     2   1     2
## 2643 09-May-2018_13:51:46  Exp 7307  22 female    R     2   1     3
## 2644 09-May-2018_13:51:48  Exp 7307  22 female    R     2   1     4
## 2645 09-May-2018_13:51:51  Exp 7307  22 female    R     2   1     5
## 2646 09-May-2018_13:51:53  Exp 7307  22 female    R     2   1     6
## 2647 09-May-2018_13:51:55  Exp 7307  22 female    R     2   1     7
## 2648 09-May-2018_13:51:58  Exp 7307  22 female    R     2   1     8
## 2649 09-May-2018_13:52:00  Exp 7307  22 female    R     2   1     9
## 2650 09-May-2018_13:52:02  Exp 7307  22 female    R     2   1    10
## 2651 09-May-2018_13:52:05  Exp 7307  22 female    R     2   1    11
## 2652 09-May-2018_13:52:07  Exp 7307  22 female    R     2   1    12
## 2653 09-May-2018_13:52:09  Exp 7307  22 female    R     2   1    13
## 2654 09-May-2018_13:52:12  Exp 7307  22 female    R     2   1    14
## 2655 09-May-2018_13:52:14  Exp 7307  22 female    R     2   1    15
## 2656 09-May-2018_13:52:17  Exp 7307  22 female    R     2   1    16
## 2657 09-May-2018_13:52:19  Exp 7307  22 female    R     2   1    17
## 2658 09-May-2018_13:52:21  Exp 7307  22 female    R     2   1    18
## 2659 09-May-2018_13:52:23  Exp 7307  22 female    R     2   1    19
## 2660 09-May-2018_13:52:26  Exp 7307  22 female    R     2   1    20
## 2661 09-May-2018_13:52:28  Exp 7307  22 female    R     2   1    21
## 2662 09-May-2018_13:52:30  Exp 7307  22 female    R     2   1    22
## 2663 09-May-2018_13:52:32  Exp 7307  22 female    R     2   1    23
## 2664 09-May-2018_13:52:35  Exp 7307  22 female    R     2   1    24
## 2665 09-May-2018_13:52:37  Exp 7307  22 female    R     2   2     1
## 2666 09-May-2018_13:52:40  Exp 7307  22 female    R     2   2     2
## 2667 09-May-2018_13:52:42  Exp 7307  22 female    R     2   2     3
## 2668 09-May-2018_13:52:44  Exp 7307  22 female    R     2   2     4
## 2669 09-May-2018_13:52:47  Exp 7307  22 female    R     2   2     5
## 2670 09-May-2018_13:52:49  Exp 7307  22 female    R     2   2     6
## 2671 09-May-2018_13:52:51  Exp 7307  22 female    R     2   2     7
## 2672 09-May-2018_13:52:53  Exp 7307  22 female    R     2   2     8
## 2673 09-May-2018_13:52:56  Exp 7307  22 female    R     2   2     9
## 2674 09-May-2018_13:52:58  Exp 7307  22 female    R     2   2    10
## 2675 09-May-2018_13:53:00  Exp 7307  22 female    R     2   2    11
## 2676 09-May-2018_13:53:03  Exp 7307  22 female    R     2   2    12
## 2677 09-May-2018_13:53:05  Exp 7307  22 female    R     2   2    13
## 2678 09-May-2018_13:53:08  Exp 7307  22 female    R     2   2    14
## 2679 09-May-2018_13:53:10  Exp 7307  22 female    R     2   2    15
## 2680 09-May-2018_13:53:13  Exp 7307  22 female    R     2   2    16
## 2681 09-May-2018_13:53:15  Exp 7307  22 female    R     2   2    17
## 2682 09-May-2018_13:53:17  Exp 7307  22 female    R     2   2    18
## 2683 09-May-2018_13:53:20  Exp 7307  22 female    R     2   2    19
## 2684 09-May-2018_13:53:22  Exp 7307  22 female    R     2   2    20
## 2685 09-May-2018_13:53:25  Exp 7307  22 female    R     2   2    21
## 2686 09-May-2018_13:53:27  Exp 7307  22 female    R     2   2    22
## 2687 09-May-2018_13:53:30  Exp 7307  22 female    R     2   2    23
## 2688 09-May-2018_13:53:32  Exp 7307  22 female    R     2   2    24
## 2689 09-May-2018_13:53:35  Exp 7307  22 female    R     2   3     1
## 2690 09-May-2018_13:53:37  Exp 7307  22 female    R     2   3     2
## 2691 09-May-2018_13:53:39  Exp 7307  22 female    R     2   3     3
## 2692 09-May-2018_13:53:42  Exp 7307  22 female    R     2   3     4
## 2693 09-May-2018_13:53:44  Exp 7307  22 female    R     2   3     5
## 2694 09-May-2018_13:53:47  Exp 7307  22 female    R     2   3     6
## 2695 09-May-2018_13:53:49  Exp 7307  22 female    R     2   3     7
## 2696 09-May-2018_13:53:52  Exp 7307  22 female    R     2   3     8
## 2697 09-May-2018_13:53:54  Exp 7307  22 female    R     2   3     9
## 2698 09-May-2018_13:53:56  Exp 7307  22 female    R     2   3    10
## 2699 09-May-2018_13:53:59  Exp 7307  22 female    R     2   3    11
## 2700 09-May-2018_13:54:02  Exp 7307  22 female    R     2   3    12
## 2701 09-May-2018_13:54:04  Exp 7307  22 female    R     2   3    13
## 2702 09-May-2018_13:54:07  Exp 7307  22 female    R     2   3    14
## 2703 09-May-2018_13:54:09  Exp 7307  22 female    R     2   3    15
## 2704 09-May-2018_13:54:11  Exp 7307  22 female    R     2   3    16
## 2705 09-May-2018_13:54:14  Exp 7307  22 female    R     2   3    17
## 2706 09-May-2018_13:54:16  Exp 7307  22 female    R     2   3    18
## 2707 09-May-2018_13:54:19  Exp 7307  22 female    R     2   3    19
## 2708 09-May-2018_13:54:21  Exp 7307  22 female    R     2   3    20
## 2709 09-May-2018_13:54:24  Exp 7307  22 female    R     2   3    21
## 2710 09-May-2018_13:54:26  Exp 7307  22 female    R     2   3    22
## 2711 09-May-2018_13:54:28  Exp 7307  22 female    R     2   3    23
## 2712 09-May-2018_13:54:31  Exp 7307  22 female    R     2   3    24
## 2713 09-May-2018_13:55:11  Exp 7307  22 female    R     2   4     1
## 2714 09-May-2018_13:55:13  Exp 7307  22 female    R     2   4     2
## 2715 09-May-2018_13:55:15  Exp 7307  22 female    R     2   4     3
## 2716 09-May-2018_13:55:17  Exp 7307  22 female    R     2   4     4
## 2717 09-May-2018_13:55:20  Exp 7307  22 female    R     2   4     5
## 2718 09-May-2018_13:55:22  Exp 7307  22 female    R     2   4     6
## 2719 09-May-2018_13:55:24  Exp 7307  22 female    R     2   4     7
## 2720 09-May-2018_13:55:26  Exp 7307  22 female    R     2   4     8
## 2721 09-May-2018_13:55:29  Exp 7307  22 female    R     2   4     9
## 2722 09-May-2018_13:55:31  Exp 7307  22 female    R     2   4    10
## 2723 09-May-2018_13:55:33  Exp 7307  22 female    R     2   4    11
## 2724 09-May-2018_13:55:36  Exp 7307  22 female    R     2   4    12
## 2725 09-May-2018_13:55:38  Exp 7307  22 female    R     2   4    13
## 2726 09-May-2018_13:55:41  Exp 7307  22 female    R     2   4    14
## 2727 09-May-2018_13:55:43  Exp 7307  22 female    R     2   4    15
## 2728 09-May-2018_13:55:45  Exp 7307  22 female    R     2   4    16
## 2729 09-May-2018_13:55:48  Exp 7307  22 female    R     2   4    17
## 2730 09-May-2018_13:55:50  Exp 7307  22 female    R     2   4    18
## 2731 09-May-2018_13:55:53  Exp 7307  22 female    R     2   4    19
## 2732 09-May-2018_13:55:55  Exp 7307  22 female    R     2   4    20
## 2733 09-May-2018_13:55:58  Exp 7307  22 female    R     2   4    21
## 2734 09-May-2018_13:56:00  Exp 7307  22 female    R     2   4    22
## 2735 09-May-2018_13:56:03  Exp 7307  22 female    R     2   4    23
## 2736 09-May-2018_13:56:05  Exp 7307  22 female    R     2   4    24
## 2737 09-May-2018_13:56:08  Exp 7307  22 female    R     2   5     1
## 2738 09-May-2018_13:56:10  Exp 7307  22 female    R     2   5     2
## 2739 09-May-2018_13:56:12  Exp 7307  22 female    R     2   5     3
## 2740 09-May-2018_13:56:15  Exp 7307  22 female    R     2   5     4
## 2741 09-May-2018_13:56:17  Exp 7307  22 female    R     2   5     5
## 2742 09-May-2018_13:56:20  Exp 7307  22 female    R     2   5     6
## 2743 09-May-2018_13:56:22  Exp 7307  22 female    R     2   5     7
## 2744 09-May-2018_13:56:25  Exp 7307  22 female    R     2   5     8
## 2745 09-May-2018_13:56:27  Exp 7307  22 female    R     2   5     9
## 2746 09-May-2018_13:56:29  Exp 7307  22 female    R     2   5    10
## 2747 09-May-2018_13:56:32  Exp 7307  22 female    R     2   5    11
## 2748 09-May-2018_13:56:34  Exp 7307  22 female    R     2   5    12
## 2749 09-May-2018_13:56:37  Exp 7307  22 female    R     2   5    13
## 2750 09-May-2018_13:56:39  Exp 7307  22 female    R     2   5    14
## 2751 09-May-2018_13:56:41  Exp 7307  22 female    R     2   5    15
## 2752 09-May-2018_13:56:44  Exp 7307  22 female    R     2   5    16
## 2753 09-May-2018_13:56:46  Exp 7307  22 female    R     2   5    17
## 2754 09-May-2018_13:56:48  Exp 7307  22 female    R     2   5    18
## 2755 09-May-2018_13:56:51  Exp 7307  22 female    R     2   5    19
## 2756 09-May-2018_13:56:53  Exp 7307  22 female    R     2   5    20
## 2757 09-May-2018_13:56:56  Exp 7307  22 female    R     2   5    21
## 2758 09-May-2018_13:56:58  Exp 7307  22 female    R     2   5    22
## 2759 09-May-2018_13:57:00  Exp 7307  22 female    R     2   5    23
## 2760 09-May-2018_13:57:03  Exp 7307  22 female    R     2   5    24
## 2761 09-May-2018_13:57:08  Exp 7307  22 female    R     3   1     1
## 2762 09-May-2018_13:57:11  Exp 7307  22 female    R     3   1     2
## 2763 09-May-2018_13:57:13  Exp 7307  22 female    R     3   1     3
## 2764 09-May-2018_13:57:15  Exp 7307  22 female    R     3   1     4
## 2765 09-May-2018_13:57:18  Exp 7307  22 female    R     3   1     5
## 2766 09-May-2018_13:57:20  Exp 7307  22 female    R     3   1     6
## 2767 09-May-2018_13:57:22  Exp 7307  22 female    R     3   1     7
## 2768 09-May-2018_13:57:25  Exp 7307  22 female    R     3   1     8
## 2769 09-May-2018_13:57:27  Exp 7307  22 female    R     3   1     9
## 2770 09-May-2018_13:57:29  Exp 7307  22 female    R     3   1    10
## 2771 09-May-2018_13:57:31  Exp 7307  22 female    R     3   1    11
## 2772 09-May-2018_13:57:34  Exp 7307  22 female    R     3   1    12
## 2773 09-May-2018_13:57:36  Exp 7307  22 female    R     3   1    13
## 2774 09-May-2018_13:57:38  Exp 7307  22 female    R     3   1    14
## 2775 09-May-2018_13:57:40  Exp 7307  22 female    R     3   1    15
## 2776 09-May-2018_13:57:43  Exp 7307  22 female    R     3   1    16
## 2777 09-May-2018_13:57:45  Exp 7307  22 female    R     3   1    17
## 2778 09-May-2018_13:57:47  Exp 7307  22 female    R     3   1    18
## 2779 09-May-2018_13:57:49  Exp 7307  22 female    R     3   1    19
## 2780 09-May-2018_13:57:52  Exp 7307  22 female    R     3   1    20
## 2781 09-May-2018_13:57:54  Exp 7307  22 female    R     3   1    21
## 2782 09-May-2018_13:57:56  Exp 7307  22 female    R     3   1    22
## 2783 09-May-2018_13:57:59  Exp 7307  22 female    R     3   1    23
## 2784 09-May-2018_13:58:01  Exp 7307  22 female    R     3   1    24
## 2785 09-May-2018_13:58:03  Exp 7307  22 female    R     3   2     1
## 2786 09-May-2018_13:58:06  Exp 7307  22 female    R     3   2     2
## 2787 09-May-2018_13:58:08  Exp 7307  22 female    R     3   2     3
## 2788 09-May-2018_13:58:11  Exp 7307  22 female    R     3   2     4
## 2789 09-May-2018_13:58:13  Exp 7307  22 female    R     3   2     5
## 2790 09-May-2018_13:58:15  Exp 7307  22 female    R     3   2     6
## 2791 09-May-2018_13:58:17  Exp 7307  22 female    R     3   2     7
## 2792 09-May-2018_13:58:20  Exp 7307  22 female    R     3   2     8
## 2793 09-May-2018_13:58:22  Exp 7307  22 female    R     3   2     9
## 2794 09-May-2018_13:58:25  Exp 7307  22 female    R     3   2    10
## 2795 09-May-2018_13:58:27  Exp 7307  22 female    R     3   2    11
## 2796 09-May-2018_13:58:29  Exp 7307  22 female    R     3   2    12
## 2797 09-May-2018_13:58:32  Exp 7307  22 female    R     3   2    13
## 2798 09-May-2018_13:58:34  Exp 7307  22 female    R     3   2    14
## 2799 09-May-2018_13:58:37  Exp 7307  22 female    R     3   2    15
## 2800 09-May-2018_13:58:39  Exp 7307  22 female    R     3   2    16
## 2801 09-May-2018_13:58:42  Exp 7307  22 female    R     3   2    17
## 2802 09-May-2018_13:58:44  Exp 7307  22 female    R     3   2    18
## 2803 09-May-2018_13:58:46  Exp 7307  22 female    R     3   2    19
## 2804 09-May-2018_13:58:49  Exp 7307  22 female    R     3   2    20
## 2805 09-May-2018_13:58:51  Exp 7307  22 female    R     3   2    21
## 2806 09-May-2018_13:58:53  Exp 7307  22 female    R     3   2    22
## 2807 09-May-2018_13:58:56  Exp 7307  22 female    R     3   2    23
## 2808 09-May-2018_13:58:58  Exp 7307  22 female    R     3   2    24
## 2809 09-May-2018_13:59:01  Exp 7307  22 female    R     3   3     1
## 2810 09-May-2018_13:59:03  Exp 7307  22 female    R     3   3     2
## 2811 09-May-2018_13:59:05  Exp 7307  22 female    R     3   3     3
## 2812 09-May-2018_13:59:07  Exp 7307  22 female    R     3   3     4
## 2813 09-May-2018_13:59:10  Exp 7307  22 female    R     3   3     5
## 2814 09-May-2018_13:59:12  Exp 7307  22 female    R     3   3     6
## 2815 09-May-2018_13:59:14  Exp 7307  22 female    R     3   3     7
## 2816 09-May-2018_13:59:16  Exp 7307  22 female    R     3   3     8
## 2817 09-May-2018_13:59:19  Exp 7307  22 female    R     3   3     9
## 2818 09-May-2018_13:59:21  Exp 7307  22 female    R     3   3    10
## 2819 09-May-2018_13:59:24  Exp 7307  22 female    R     3   3    11
## 2820 09-May-2018_13:59:26  Exp 7307  22 female    R     3   3    12
## 2821 09-May-2018_13:59:28  Exp 7307  22 female    R     3   3    13
## 2822 09-May-2018_13:59:31  Exp 7307  22 female    R     3   3    14
## 2823 09-May-2018_13:59:33  Exp 7307  22 female    R     3   3    15
## 2824 09-May-2018_13:59:36  Exp 7307  22 female    R     3   3    16
## 2825 09-May-2018_13:59:38  Exp 7307  22 female    R     3   3    17
## 2826 09-May-2018_13:59:40  Exp 7307  22 female    R     3   3    18
## 2827 09-May-2018_13:59:43  Exp 7307  22 female    R     3   3    19
## 2828 09-May-2018_13:59:45  Exp 7307  22 female    R     3   3    20
## 2829 09-May-2018_13:59:47  Exp 7307  22 female    R     3   3    21
## 2830 09-May-2018_13:59:50  Exp 7307  22 female    R     3   3    22
## 2831 09-May-2018_13:59:52  Exp 7307  22 female    R     3   3    23
## 2832 09-May-2018_13:59:55  Exp 7307  22 female    R     3   3    24
## 2833 09-May-2018_14:00:28  Exp 7307  22 female    R     3   4     1
## 2834 09-May-2018_14:00:30  Exp 7307  22 female    R     3   4     2
## 2835 09-May-2018_14:00:32  Exp 7307  22 female    R     3   4     3
## 2836 09-May-2018_14:00:35  Exp 7307  22 female    R     3   4     4
## 2837 09-May-2018_14:00:37  Exp 7307  22 female    R     3   4     5
## 2838 09-May-2018_14:00:39  Exp 7307  22 female    R     3   4     6
## 2839 09-May-2018_14:00:41  Exp 7307  22 female    R     3   4     7
## 2840 09-May-2018_14:00:44  Exp 7307  22 female    R     3   4     8
## 2841 09-May-2018_14:00:46  Exp 7307  22 female    R     3   4     9
## 2842 09-May-2018_14:00:48  Exp 7307  22 female    R     3   4    10
## 2843 09-May-2018_14:00:50  Exp 7307  22 female    R     3   4    11
## 2844 09-May-2018_14:00:53  Exp 7307  22 female    R     3   4    12
## 2845 09-May-2018_14:00:55  Exp 7307  22 female    R     3   4    13
## 2846 09-May-2018_14:00:57  Exp 7307  22 female    R     3   4    14
## 2847 09-May-2018_14:00:59  Exp 7307  22 female    R     3   4    15
## 2848 09-May-2018_14:01:02  Exp 7307  22 female    R     3   4    16
## 2849 09-May-2018_14:01:04  Exp 7307  22 female    R     3   4    17
## 2850 09-May-2018_14:01:06  Exp 7307  22 female    R     3   4    18
## 2851 09-May-2018_14:01:09  Exp 7307  22 female    R     3   4    19
## 2852 09-May-2018_14:01:11  Exp 7307  22 female    R     3   4    20
## 2853 09-May-2018_14:01:13  Exp 7307  22 female    R     3   4    21
## 2854 09-May-2018_14:01:15  Exp 7307  22 female    R     3   4    22
## 2855 09-May-2018_14:01:18  Exp 7307  22 female    R     3   4    23
## 2856 09-May-2018_14:01:20  Exp 7307  22 female    R     3   4    24
## 2857 09-May-2018_14:01:23  Exp 7307  22 female    R     3   5     1
## 2858 09-May-2018_14:01:25  Exp 7307  22 female    R     3   5     2
## 2859 09-May-2018_14:01:27  Exp 7307  22 female    R     3   5     3
## 2860 09-May-2018_14:01:29  Exp 7307  22 female    R     3   5     4
## 2861 09-May-2018_14:01:31  Exp 7307  22 female    R     3   5     5
## 2862 09-May-2018_14:01:34  Exp 7307  22 female    R     3   5     6
## 2863 09-May-2018_14:01:36  Exp 7307  22 female    R     3   5     7
## 2864 09-May-2018_14:01:38  Exp 7307  22 female    R     3   5     8
## 2865 09-May-2018_14:01:41  Exp 7307  22 female    R     3   5     9
## 2866 09-May-2018_14:01:43  Exp 7307  22 female    R     3   5    10
## 2867 09-May-2018_14:01:45  Exp 7307  22 female    R     3   5    11
## 2868 09-May-2018_14:01:48  Exp 7307  22 female    R     3   5    12
## 2869 09-May-2018_14:01:50  Exp 7307  22 female    R     3   5    13
## 2870 09-May-2018_14:01:52  Exp 7307  22 female    R     3   5    14
## 2871 09-May-2018_14:01:55  Exp 7307  22 female    R     3   5    15
## 2872 09-May-2018_14:01:57  Exp 7307  22 female    R     3   5    16
## 2873 09-May-2018_14:01:59  Exp 7307  22 female    R     3   5    17
## 2874 09-May-2018_14:02:01  Exp 7307  22 female    R     3   5    18
## 2875 09-May-2018_14:02:04  Exp 7307  22 female    R     3   5    19
## 2876 09-May-2018_14:02:06  Exp 7307  22 female    R     3   5    20
## 2877 09-May-2018_14:02:08  Exp 7307  22 female    R     3   5    21
## 2878 09-May-2018_14:02:11  Exp 7307  22 female    R     3   5    22
## 2879 09-May-2018_14:02:13  Exp 7307  22 female    R     3   5    23
## 2880 09-May-2018_14:02:15  Exp 7307  22 female    R     3   5    24
## 2881 09-May-2018_14:07:23  Exp 7307  22 female    R     1   1     1
## 2882 09-May-2018_14:07:25  Exp 7307  22 female    R     1   1     2
## 2883 09-May-2018_14:07:27  Exp 7307  22 female    R     1   1     3
## 2884 09-May-2018_14:07:30  Exp 7307  22 female    R     1   1     4
## 2885 09-May-2018_14:07:32  Exp 7307  22 female    R     1   1     5
## 2886 09-May-2018_14:07:35  Exp 7307  22 female    R     1   1     6
## 2887 09-May-2018_14:07:37  Exp 7307  22 female    R     1   1     7
## 2888 09-May-2018_14:07:40  Exp 7307  22 female    R     1   1     8
## 2889 09-May-2018_14:07:42  Exp 7307  22 female    R     1   1     9
## 2890 09-May-2018_14:07:45  Exp 7307  22 female    R     1   1    10
## 2891 09-May-2018_14:07:47  Exp 7307  22 female    R     1   1    11
## 2892 09-May-2018_14:07:49  Exp 7307  22 female    R     1   1    12
## 2893 09-May-2018_14:07:52  Exp 7307  22 female    R     1   1    13
## 2894 09-May-2018_14:07:55  Exp 7307  22 female    R     1   1    14
## 2895 09-May-2018_14:07:57  Exp 7307  22 female    R     1   1    15
## 2896 09-May-2018_14:07:59  Exp 7307  22 female    R     1   1    16
## 2897 09-May-2018_14:08:02  Exp 7307  22 female    R     1   1    17
## 2898 09-May-2018_14:08:04  Exp 7307  22 female    R     1   1    18
## 2899 09-May-2018_14:08:06  Exp 7307  22 female    R     1   1    19
## 2900 09-May-2018_14:08:09  Exp 7307  22 female    R     1   1    20
## 2901 09-May-2018_14:08:12  Exp 7307  22 female    R     1   1    21
## 2902 09-May-2018_14:08:14  Exp 7307  22 female    R     1   1    22
## 2903 09-May-2018_14:08:17  Exp 7307  22 female    R     1   1    23
## 2904 09-May-2018_14:08:19  Exp 7307  22 female    R     1   1    24
## 2905 09-May-2018_14:08:21  Exp 7307  22 female    R     1   2     1
## 2906 09-May-2018_14:08:24  Exp 7307  22 female    R     1   2     2
## 2907 09-May-2018_14:08:26  Exp 7307  22 female    R     1   2     3
## 2908 09-May-2018_14:08:28  Exp 7307  22 female    R     1   2     4
## 2909 09-May-2018_14:08:30  Exp 7307  22 female    R     1   2     5
## 2910 09-May-2018_14:08:33  Exp 7307  22 female    R     1   2     6
## 2911 09-May-2018_14:08:35  Exp 7307  22 female    R     1   2     7
## 2912 09-May-2018_14:08:37  Exp 7307  22 female    R     1   2     8
## 2913 09-May-2018_14:08:40  Exp 7307  22 female    R     1   2     9
## 2914 09-May-2018_14:08:42  Exp 7307  22 female    R     1   2    10
## 2915 09-May-2018_14:08:45  Exp 7307  22 female    R     1   2    11
## 2916 09-May-2018_14:08:47  Exp 7307  22 female    R     1   2    12
## 2917 09-May-2018_14:08:49  Exp 7307  22 female    R     1   2    13
## 2918 09-May-2018_14:08:52  Exp 7307  22 female    R     1   2    14
## 2919 09-May-2018_14:08:54  Exp 7307  22 female    R     1   2    15
## 2920 09-May-2018_14:08:57  Exp 7307  22 female    R     1   2    16
## 2921 09-May-2018_14:08:59  Exp 7307  22 female    R     1   2    17
## 2922 09-May-2018_14:09:01  Exp 7307  22 female    R     1   2    18
## 2923 09-May-2018_14:09:03  Exp 7307  22 female    R     1   2    19
## 2924 09-May-2018_14:09:06  Exp 7307  22 female    R     1   2    20
## 2925 09-May-2018_14:09:08  Exp 7307  22 female    R     1   2    21
## 2926 09-May-2018_14:09:10  Exp 7307  22 female    R     1   2    22
## 2927 09-May-2018_14:09:12  Exp 7307  22 female    R     1   2    23
## 2928 09-May-2018_14:09:15  Exp 7307  22 female    R     1   2    24
## 2929 09-May-2018_14:14:30  Exp 7307  22 female    R     1   1     1
## 2930 09-May-2018_14:14:32  Exp 7307  22 female    R     1   1     2
## 2931 09-May-2018_14:14:34  Exp 7307  22 female    R     1   1     3
## 2932 09-May-2018_14:14:37  Exp 7307  22 female    R     1   1     4
## 2933 09-May-2018_14:14:39  Exp 7307  22 female    R     1   1     5
## 2934 09-May-2018_14:14:41  Exp 7307  22 female    R     1   1     6
## 2935 09-May-2018_14:14:44  Exp 7307  22 female    R     1   1     7
## 2936 09-May-2018_14:14:46  Exp 7307  22 female    R     1   1     8
## 2937 09-May-2018_14:14:48  Exp 7307  22 female    R     1   1     9
## 2938 09-May-2018_14:14:51  Exp 7307  22 female    R     1   1    10
## 2939 09-May-2018_14:14:53  Exp 7307  22 female    R     1   1    11
## 2940 09-May-2018_14:14:56  Exp 7307  22 female    R     1   1    12
## 2941 09-May-2018_14:14:58  Exp 7307  22 female    R     1   1    13
## 2942 09-May-2018_14:15:01  Exp 7307  22 female    R     1   1    14
## 2943 09-May-2018_14:15:03  Exp 7307  22 female    R     1   1    15
## 2944 09-May-2018_14:15:05  Exp 7307  22 female    R     1   1    16
## 2945 09-May-2018_14:15:08  Exp 7307  22 female    R     1   1    17
## 2946 09-May-2018_14:15:10  Exp 7307  22 female    R     1   1    18
## 2947 09-May-2018_14:15:12  Exp 7307  22 female    R     1   1    19
## 2948 09-May-2018_14:15:14  Exp 7307  22 female    R     1   1    20
## 2949 09-May-2018_14:15:17  Exp 7307  22 female    R     1   1    21
## 2950 09-May-2018_14:15:19  Exp 7307  22 female    R     1   1    22
## 2951 09-May-2018_14:15:21  Exp 7307  22 female    R     1   1    23
## 2952 09-May-2018_14:15:24  Exp 7307  22 female    R     1   1    24
## 2953 09-May-2018_14:15:26  Exp 7307  22 female    R     1   2     1
## 2954 09-May-2018_14:15:28  Exp 7307  22 female    R     1   2     2
## 2955 09-May-2018_14:15:30  Exp 7307  22 female    R     1   2     3
## 2956 09-May-2018_14:15:33  Exp 7307  22 female    R     1   2     4
## 2957 09-May-2018_14:15:35  Exp 7307  22 female    R     1   2     5
## 2958 09-May-2018_14:15:37  Exp 7307  22 female    R     1   2     6
## 2959 09-May-2018_14:15:40  Exp 7307  22 female    R     1   2     7
## 2960 09-May-2018_14:15:42  Exp 7307  22 female    R     1   2     8
## 2961 09-May-2018_14:15:44  Exp 7307  22 female    R     1   2     9
## 2962 09-May-2018_14:15:47  Exp 7307  22 female    R     1   2    10
## 2963 09-May-2018_14:15:49  Exp 7307  22 female    R     1   2    11
## 2964 09-May-2018_14:15:52  Exp 7307  22 female    R     1   2    12
## 2965 09-May-2018_14:15:54  Exp 7307  22 female    R     1   2    13
## 2966 09-May-2018_14:15:56  Exp 7307  22 female    R     1   2    14
## 2967 09-May-2018_14:15:59  Exp 7307  22 female    R     1   2    15
## 2968 09-May-2018_14:16:01  Exp 7307  22 female    R     1   2    16
## 2969 09-May-2018_14:16:03  Exp 7307  22 female    R     1   2    17
## 2970 09-May-2018_14:16:06  Exp 7307  22 female    R     1   2    18
## 2971 09-May-2018_14:16:08  Exp 7307  22 female    R     1   2    19
## 2972 09-May-2018_14:16:10  Exp 7307  22 female    R     1   2    20
## 2973 09-May-2018_14:16:13  Exp 7307  22 female    R     1   2    21
## 2974 09-May-2018_14:16:15  Exp 7307  22 female    R     1   2    22
## 2975 09-May-2018_14:16:17  Exp 7307  22 female    R     1   2    23
## 2976 09-May-2018_14:16:20  Exp 7307  22 female    R     1   2    24
## 2977 09-May-2018_14:20:50  Exp 7307  22 female    R     1   1     1
## 2978 09-May-2018_14:20:52  Exp 7307  22 female    R     1   1     2
## 2979 09-May-2018_14:20:55  Exp 7307  22 female    R     1   1     3
## 2980 09-May-2018_14:20:57  Exp 7307  22 female    R     1   1     4
## 2981 09-May-2018_14:20:59  Exp 7307  22 female    R     1   1     5
## 2982 09-May-2018_14:21:02  Exp 7307  22 female    R     1   1     6
## 2983 09-May-2018_14:21:04  Exp 7307  22 female    R     1   1     7
## 2984 09-May-2018_14:21:06  Exp 7307  22 female    R     1   1     8
## 2985 09-May-2018_14:21:09  Exp 7307  22 female    R     1   1     9
## 2986 09-May-2018_14:21:11  Exp 7307  22 female    R     1   1    10
## 2987 09-May-2018_14:21:13  Exp 7307  22 female    R     1   1    11
## 2988 09-May-2018_14:21:15  Exp 7307  22 female    R     1   1    12
## 2989 09-May-2018_14:21:18  Exp 7307  22 female    R     1   1    13
## 2990 09-May-2018_14:21:20  Exp 7307  22 female    R     1   1    14
## 2991 09-May-2018_14:21:22  Exp 7307  22 female    R     1   1    15
## 2992 09-May-2018_14:21:25  Exp 7307  22 female    R     1   1    16
## 2993 09-May-2018_14:21:27  Exp 7307  22 female    R     1   1    17
## 2994 09-May-2018_14:21:30  Exp 7307  22 female    R     1   1    18
## 2995 09-May-2018_14:21:32  Exp 7307  22 female    R     1   1    19
## 2996 09-May-2018_14:21:34  Exp 7307  22 female    R     1   1    20
## 2997 09-May-2018_14:21:36  Exp 7307  22 female    R     1   1    21
## 2998 09-May-2018_14:21:39  Exp 7307  22 female    R     1   1    22
## 2999 09-May-2018_14:21:41  Exp 7307  22 female    R     1   1    23
## 3000 09-May-2018_14:21:43  Exp 7307  22 female    R     1   1    24
## 3001 09-May-2018_14:21:46  Exp 7307  22 female    R     1   2     1
## 3002 09-May-2018_14:21:48  Exp 7307  22 female    R     1   2     2
## 3003 09-May-2018_14:21:51  Exp 7307  22 female    R     1   2     3
## 3004 09-May-2018_14:21:53  Exp 7307  22 female    R     1   2     4
## 3005 09-May-2018_14:21:55  Exp 7307  22 female    R     1   2     5
## 3006 09-May-2018_14:21:58  Exp 7307  22 female    R     1   2     6
## 3007 09-May-2018_14:22:00  Exp 7307  22 female    R     1   2     7
## 3008 09-May-2018_14:22:02  Exp 7307  22 female    R     1   2     8
## 3009 09-May-2018_14:22:04  Exp 7307  22 female    R     1   2     9
## 3010 09-May-2018_14:22:07  Exp 7307  22 female    R     1   2    10
## 3011 09-May-2018_14:22:09  Exp 7307  22 female    R     1   2    11
## 3012 09-May-2018_14:22:11  Exp 7307  22 female    R     1   2    12
## 3013 09-May-2018_14:22:13  Exp 7307  22 female    R     1   2    13
## 3014 09-May-2018_14:22:16  Exp 7307  22 female    R     1   2    14
## 3015 09-May-2018_14:22:18  Exp 7307  22 female    R     1   2    15
## 3016 09-May-2018_14:22:20  Exp 7307  22 female    R     1   2    16
## 3017 09-May-2018_14:22:23  Exp 7307  22 female    R     1   2    17
## 3018 09-May-2018_14:22:25  Exp 7307  22 female    R     1   2    18
## 3019 09-May-2018_14:22:27  Exp 7307  22 female    R     1   2    19
## 3020 09-May-2018_14:22:29  Exp 7307  22 female    R     1   2    20
## 3021 09-May-2018_14:22:32  Exp 7307  22 female    R     1   2    21
## 3022 09-May-2018_14:22:34  Exp 7307  22 female    R     1   2    22
## 3023 09-May-2018_14:22:36  Exp 7307  22 female    R     1   2    23
## 3024 09-May-2018_14:22:38  Exp 7307  22 female    R     1   2    24
## 3025 09-May-2018_14:28:36  Exp 7307  22 female    R     1   1     1
## 3026 09-May-2018_14:28:38  Exp 7307  22 female    R     1   1     2
## 3027 09-May-2018_14:28:40  Exp 7307  22 female    R     1   1     3
## 3028 09-May-2018_14:28:43  Exp 7307  22 female    R     1   1     4
## 3029 09-May-2018_14:28:45  Exp 7307  22 female    R     1   1     5
## 3030 09-May-2018_14:28:47  Exp 7307  22 female    R     1   1     6
## 3031 09-May-2018_14:28:50  Exp 7307  22 female    R     1   1     7
## 3032 09-May-2018_14:28:52  Exp 7307  22 female    R     1   1     8
## 3033 09-May-2018_14:28:54  Exp 7307  22 female    R     1   1     9
## 3034 09-May-2018_14:28:56  Exp 7307  22 female    R     1   1    10
## 3035 09-May-2018_14:28:59  Exp 7307  22 female    R     1   1    11
## 3036 09-May-2018_14:29:01  Exp 7307  22 female    R     1   1    12
## 3037 09-May-2018_14:29:03  Exp 7307  22 female    R     1   1    13
## 3038 09-May-2018_14:29:06  Exp 7307  22 female    R     1   1    14
## 3039 09-May-2018_14:29:08  Exp 7307  22 female    R     1   1    15
## 3040 09-May-2018_14:29:11  Exp 7307  22 female    R     1   1    16
## 3041 09-May-2018_14:29:13  Exp 7307  22 female    R     1   1    17
## 3042 09-May-2018_14:29:15  Exp 7307  22 female    R     1   1    18
## 3043 09-May-2018_14:29:18  Exp 7307  22 female    R     1   1    19
## 3044 09-May-2018_14:29:20  Exp 7307  22 female    R     1   1    20
## 3045 09-May-2018_14:29:22  Exp 7307  22 female    R     1   1    21
## 3046 09-May-2018_14:29:25  Exp 7307  22 female    R     1   1    22
## 3047 09-May-2018_14:29:27  Exp 7307  22 female    R     1   1    23
## 3048 09-May-2018_14:29:29  Exp 7307  22 female    R     1   1    24
## 3049 09-May-2018_14:29:32  Exp 7307  22 female    R     1   2     1
## 3050 09-May-2018_14:29:34  Exp 7307  22 female    R     1   2     2
## 3051 09-May-2018_14:29:36  Exp 7307  22 female    R     1   2     3
## 3052 09-May-2018_14:29:39  Exp 7307  22 female    R     1   2     4
## 3053 09-May-2018_14:29:41  Exp 7307  22 female    R     1   2     5
## 3054 09-May-2018_14:29:43  Exp 7307  22 female    R     1   2     6
## 3055 09-May-2018_14:29:46  Exp 7307  22 female    R     1   2     7
## 3056 09-May-2018_14:29:48  Exp 7307  22 female    R     1   2     8
## 3057 09-May-2018_14:29:50  Exp 7307  22 female    R     1   2     9
## 3058 09-May-2018_14:29:53  Exp 7307  22 female    R     1   2    10
## 3059 09-May-2018_14:29:55  Exp 7307  22 female    R     1   2    11
## 3060 09-May-2018_14:29:58  Exp 7307  22 female    R     1   2    12
## 3061 09-May-2018_14:30:00  Exp 7307  22 female    R     1   2    13
## 3062 09-May-2018_14:30:02  Exp 7307  22 female    R     1   2    14
## 3063 09-May-2018_14:30:05  Exp 7307  22 female    R     1   2    15
## 3064 09-May-2018_14:30:07  Exp 7307  22 female    R     1   2    16
## 3065 09-May-2018_14:30:09  Exp 7307  22 female    R     1   2    17
## 3066 09-May-2018_14:30:12  Exp 7307  22 female    R     1   2    18
## 3067 09-May-2018_14:30:14  Exp 7307  22 female    R     1   2    19
## 3068 09-May-2018_14:30:16  Exp 7307  22 female    R     1   2    20
## 3069 09-May-2018_14:30:18  Exp 7307  22 female    R     1   2    21
## 3070 09-May-2018_14:30:21  Exp 7307  22 female    R     1   2    22
## 3071 09-May-2018_14:30:23  Exp 7307  22 female    R     1   2    23
## 3072 09-May-2018_14:30:25  Exp 7307  22 female    R     1   2    24
## 3073 09-May-2018_14:35:06  Exp 7307  22 female    R     1   1     1
## 3074 09-May-2018_14:35:08  Exp 7307  22 female    R     1   1     2
## 3075 09-May-2018_14:35:10  Exp 7307  22 female    R     1   1     3
## 3076 09-May-2018_14:35:13  Exp 7307  22 female    R     1   1     4
## 3077 09-May-2018_14:35:15  Exp 7307  22 female    R     1   1     5
## 3078 09-May-2018_14:35:17  Exp 7307  22 female    R     1   1     6
## 3079 09-May-2018_14:35:20  Exp 7307  22 female    R     1   1     7
## 3080 09-May-2018_14:35:22  Exp 7307  22 female    R     1   1     8
## 3081 09-May-2018_14:35:24  Exp 7307  22 female    R     1   1     9
## 3082 09-May-2018_14:35:27  Exp 7307  22 female    R     1   1    10
## 3083 09-May-2018_14:35:29  Exp 7307  22 female    R     1   1    11
## 3084 09-May-2018_14:35:31  Exp 7307  22 female    R     1   1    12
## 3085 09-May-2018_14:35:34  Exp 7307  22 female    R     1   1    13
## 3086 09-May-2018_14:35:36  Exp 7307  22 female    R     1   1    14
## 3087 09-May-2018_14:35:38  Exp 7307  22 female    R     1   1    15
## 3088 09-May-2018_14:35:40  Exp 7307  22 female    R     1   1    16
## 3089 09-May-2018_14:35:43  Exp 7307  22 female    R     1   1    17
## 3090 09-May-2018_14:35:45  Exp 7307  22 female    R     1   1    18
## 3091 09-May-2018_14:35:48  Exp 7307  22 female    R     1   1    19
## 3092 09-May-2018_14:35:50  Exp 7307  22 female    R     1   1    20
## 3093 09-May-2018_14:35:52  Exp 7307  22 female    R     1   1    21
## 3094 09-May-2018_14:35:54  Exp 7307  22 female    R     1   1    22
## 3095 09-May-2018_14:35:57  Exp 7307  22 female    R     1   1    23
## 3096 09-May-2018_14:35:59  Exp 7307  22 female    R     1   1    24
## 3097 09-May-2018_14:36:01  Exp 7307  22 female    R     1   2     1
## 3098 09-May-2018_14:36:04  Exp 7307  22 female    R     1   2     2
## 3099 09-May-2018_14:36:06  Exp 7307  22 female    R     1   2     3
## 3100 09-May-2018_14:36:08  Exp 7307  22 female    R     1   2     4
## 3101 09-May-2018_14:36:10  Exp 7307  22 female    R     1   2     5
## 3102 09-May-2018_14:36:13  Exp 7307  22 female    R     1   2     6
## 3103 09-May-2018_14:36:15  Exp 7307  22 female    R     1   2     7
## 3104 09-May-2018_14:36:18  Exp 7307  22 female    R     1   2     8
## 3105 09-May-2018_14:36:20  Exp 7307  22 female    R     1   2     9
## 3106 09-May-2018_14:36:22  Exp 7307  22 female    R     1   2    10
## 3107 09-May-2018_14:36:25  Exp 7307  22 female    R     1   2    11
## 3108 09-May-2018_14:36:27  Exp 7307  22 female    R     1   2    12
## 3109 09-May-2018_14:36:29  Exp 7307  22 female    R     1   2    13
## 3110 09-May-2018_14:36:32  Exp 7307  22 female    R     1   2    14
## 3111 09-May-2018_14:36:34  Exp 7307  22 female    R     1   2    15
## 3112 09-May-2018_14:36:36  Exp 7307  22 female    R     1   2    16
## 3113 09-May-2018_14:36:39  Exp 7307  22 female    R     1   2    17
## 3114 09-May-2018_14:36:41  Exp 7307  22 female    R     1   2    18
## 3115 09-May-2018_14:36:43  Exp 7307  22 female    R     1   2    19
## 3116 09-May-2018_14:36:45  Exp 7307  22 female    R     1   2    20
## 3117 09-May-2018_14:36:48  Exp 7307  22 female    R     1   2    21
## 3118 09-May-2018_14:36:50  Exp 7307  22 female    R     1   2    22
## 3119 09-May-2018_14:36:52  Exp 7307  22 female    R     1   2    23
## 3120 09-May-2018_14:36:55  Exp 7307  22 female    R     1   2    24
## 3121 10-May-2018_13:27:25  Exp 7308  20 female    R     1   1     1
## 3122 10-May-2018_13:27:27  Exp 7308  20 female    R     1   1     2
## 3123 10-May-2018_13:27:30  Exp 7308  20 female    R     1   1     3
## 3124 10-May-2018_13:27:32  Exp 7308  20 female    R     1   1     4
## 3125 10-May-2018_13:27:35  Exp 7308  20 female    R     1   1     5
## 3126 10-May-2018_13:27:37  Exp 7308  20 female    R     1   1     6
## 3127 10-May-2018_13:27:40  Exp 7308  20 female    R     1   1     7
## 3128 10-May-2018_13:27:42  Exp 7308  20 female    R     1   1     8
## 3129 10-May-2018_13:27:45  Exp 7308  20 female    R     1   1     9
## 3130 10-May-2018_13:27:47  Exp 7308  20 female    R     1   1    10
## 3131 10-May-2018_13:27:49  Exp 7308  20 female    R     1   1    11
## 3132 10-May-2018_13:27:52  Exp 7308  20 female    R     1   1    12
## 3133 10-May-2018_13:27:54  Exp 7308  20 female    R     1   1    13
## 3134 10-May-2018_13:27:56  Exp 7308  20 female    R     1   1    14
## 3135 10-May-2018_13:27:59  Exp 7308  20 female    R     1   1    15
## 3136 10-May-2018_13:28:01  Exp 7308  20 female    R     1   1    16
## 3137 10-May-2018_13:28:04  Exp 7308  20 female    R     1   1    17
## 3138 10-May-2018_13:28:06  Exp 7308  20 female    R     1   1    18
## 3139 10-May-2018_13:28:09  Exp 7308  20 female    R     1   1    19
## 3140 10-May-2018_13:28:11  Exp 7308  20 female    R     1   1    20
## 3141 10-May-2018_13:28:14  Exp 7308  20 female    R     1   1    21
## 3142 10-May-2018_13:28:16  Exp 7308  20 female    R     1   1    22
## 3143 10-May-2018_13:28:18  Exp 7308  20 female    R     1   1    23
## 3144 10-May-2018_13:28:21  Exp 7308  20 female    R     1   1    24
## 3145 10-May-2018_13:28:23  Exp 7308  20 female    R     1   2     1
## 3146 10-May-2018_13:28:26  Exp 7308  20 female    R     1   2     2
## 3147 10-May-2018_13:28:28  Exp 7308  20 female    R     1   2     3
## 3148 10-May-2018_13:28:31  Exp 7308  20 female    R     1   2     4
## 3149 10-May-2018_13:28:33  Exp 7308  20 female    R     1   2     5
## 3150 10-May-2018_13:28:35  Exp 7308  20 female    R     1   2     6
## 3151 10-May-2018_13:28:38  Exp 7308  20 female    R     1   2     7
## 3152 10-May-2018_13:28:40  Exp 7308  20 female    R     1   2     8
## 3153 10-May-2018_13:28:43  Exp 7308  20 female    R     1   2     9
## 3154 10-May-2018_13:28:45  Exp 7308  20 female    R     1   2    10
## 3155 10-May-2018_13:28:48  Exp 7308  20 female    R     1   2    11
## 3156 10-May-2018_13:28:50  Exp 7308  20 female    R     1   2    12
## 3157 10-May-2018_13:28:53  Exp 7308  20 female    R     1   2    13
## 3158 10-May-2018_13:28:55  Exp 7308  20 female    R     1   2    14
## 3159 10-May-2018_13:28:58  Exp 7308  20 female    R     1   2    15
## 3160 10-May-2018_13:29:00  Exp 7308  20 female    R     1   2    16
## 3161 10-May-2018_13:29:02  Exp 7308  20 female    R     1   2    17
## 3162 10-May-2018_13:29:05  Exp 7308  20 female    R     1   2    18
## 3163 10-May-2018_13:29:07  Exp 7308  20 female    R     1   2    19
## 3164 10-May-2018_13:29:09  Exp 7308  20 female    R     1   2    20
## 3165 10-May-2018_13:29:12  Exp 7308  20 female    R     1   2    21
## 3166 10-May-2018_13:29:14  Exp 7308  20 female    R     1   2    22
## 3167 10-May-2018_13:29:16  Exp 7308  20 female    R     1   2    23
## 3168 10-May-2018_13:29:19  Exp 7308  20 female    R     1   2    24
## 3169 10-May-2018_13:29:21  Exp 7308  20 female    R     1   3     1
## 3170 10-May-2018_13:29:24  Exp 7308  20 female    R     1   3     2
## 3171 10-May-2018_13:29:26  Exp 7308  20 female    R     1   3     3
## 3172 10-May-2018_13:29:29  Exp 7308  20 female    R     1   3     4
## 3173 10-May-2018_13:29:31  Exp 7308  20 female    R     1   3     5
## 3174 10-May-2018_13:29:34  Exp 7308  20 female    R     1   3     6
## 3175 10-May-2018_13:29:36  Exp 7308  20 female    R     1   3     7
## 3176 10-May-2018_13:29:38  Exp 7308  20 female    R     1   3     8
## 3177 10-May-2018_13:29:41  Exp 7308  20 female    R     1   3     9
## 3178 10-May-2018_13:29:43  Exp 7308  20 female    R     1   3    10
## 3179 10-May-2018_13:29:45  Exp 7308  20 female    R     1   3    11
## 3180 10-May-2018_13:29:48  Exp 7308  20 female    R     1   3    12
## 3181 10-May-2018_13:29:50  Exp 7308  20 female    R     1   3    13
## 3182 10-May-2018_13:29:52  Exp 7308  20 female    R     1   3    14
## 3183 10-May-2018_13:29:55  Exp 7308  20 female    R     1   3    15
## 3184 10-May-2018_13:29:57  Exp 7308  20 female    R     1   3    16
## 3185 10-May-2018_13:30:00  Exp 7308  20 female    R     1   3    17
## 3186 10-May-2018_13:30:03  Exp 7308  20 female    R     1   3    18
## 3187 10-May-2018_13:30:05  Exp 7308  20 female    R     1   3    19
## 3188 10-May-2018_13:30:07  Exp 7308  20 female    R     1   3    20
## 3189 10-May-2018_13:30:10  Exp 7308  20 female    R     1   3    21
## 3190 10-May-2018_13:30:12  Exp 7308  20 female    R     1   3    22
## 3191 10-May-2018_13:30:14  Exp 7308  20 female    R     1   3    23
## 3192 10-May-2018_13:30:17  Exp 7308  20 female    R     1   3    24
## 3193 10-May-2018_13:30:33  Exp 7308  20 female    R     1   4     1
## 3194 10-May-2018_13:30:35  Exp 7308  20 female    R     1   4     2
## 3195 10-May-2018_13:30:38  Exp 7308  20 female    R     1   4     3
## 3196 10-May-2018_13:30:40  Exp 7308  20 female    R     1   4     4
## 3197 10-May-2018_13:30:42  Exp 7308  20 female    R     1   4     5
## 3198 10-May-2018_13:30:45  Exp 7308  20 female    R     1   4     6
## 3199 10-May-2018_13:30:48  Exp 7308  20 female    R     1   4     7
## 3200 10-May-2018_13:30:50  Exp 7308  20 female    R     1   4     8
## 3201 10-May-2018_13:30:52  Exp 7308  20 female    R     1   4     9
## 3202 10-May-2018_13:30:55  Exp 7308  20 female    R     1   4    10
## 3203 10-May-2018_13:30:57  Exp 7308  20 female    R     1   4    11
## 3204 10-May-2018_13:30:59  Exp 7308  20 female    R     1   4    12
## 3205 10-May-2018_13:31:02  Exp 7308  20 female    R     1   4    13
## 3206 10-May-2018_13:31:04  Exp 7308  20 female    R     1   4    14
## 3207 10-May-2018_13:31:07  Exp 7308  20 female    R     1   4    15
## 3208 10-May-2018_13:31:09  Exp 7308  20 female    R     1   4    16
## 3209 10-May-2018_13:31:12  Exp 7308  20 female    R     1   4    17
## 3210 10-May-2018_13:31:14  Exp 7308  20 female    R     1   4    18
## 3211 10-May-2018_13:31:17  Exp 7308  20 female    R     1   4    19
## 3212 10-May-2018_13:31:19  Exp 7308  20 female    R     1   4    20
## 3213 10-May-2018_13:31:21  Exp 7308  20 female    R     1   4    21
## 3214 10-May-2018_13:31:23  Exp 7308  20 female    R     1   4    22
## 3215 10-May-2018_13:31:26  Exp 7308  20 female    R     1   4    23
## 3216 10-May-2018_13:31:28  Exp 7308  20 female    R     1   4    24
## 3217 10-May-2018_13:31:30  Exp 7308  20 female    R     1   5     1
## 3218 10-May-2018_13:31:33  Exp 7308  20 female    R     1   5     2
## 3219 10-May-2018_13:31:35  Exp 7308  20 female    R     1   5     3
## 3220 10-May-2018_13:31:37  Exp 7308  20 female    R     1   5     4
## 3221 10-May-2018_13:31:40  Exp 7308  20 female    R     1   5     5
## 3222 10-May-2018_13:31:42  Exp 7308  20 female    R     1   5     6
## 3223 10-May-2018_13:31:44  Exp 7308  20 female    R     1   5     7
## 3224 10-May-2018_13:31:47  Exp 7308  20 female    R     1   5     8
## 3225 10-May-2018_13:31:49  Exp 7308  20 female    R     1   5     9
## 3226 10-May-2018_13:31:52  Exp 7308  20 female    R     1   5    10
## 3227 10-May-2018_13:31:54  Exp 7308  20 female    R     1   5    11
## 3228 10-May-2018_13:31:56  Exp 7308  20 female    R     1   5    12
## 3229 10-May-2018_13:31:58  Exp 7308  20 female    R     1   5    13
## 3230 10-May-2018_13:32:01  Exp 7308  20 female    R     1   5    14
## 3231 10-May-2018_13:32:03  Exp 7308  20 female    R     1   5    15
## 3232 10-May-2018_13:32:06  Exp 7308  20 female    R     1   5    16
## 3233 10-May-2018_13:32:08  Exp 7308  20 female    R     1   5    17
## 3234 10-May-2018_13:32:11  Exp 7308  20 female    R     1   5    18
## 3235 10-May-2018_13:32:13  Exp 7308  20 female    R     1   5    19
## 3236 10-May-2018_13:32:15  Exp 7308  20 female    R     1   5    20
## 3237 10-May-2018_13:32:18  Exp 7308  20 female    R     1   5    21
## 3238 10-May-2018_13:32:20  Exp 7308  20 female    R     1   5    22
## 3239 10-May-2018_13:32:22  Exp 7308  20 female    R     1   5    23
## 3240 10-May-2018_13:32:25  Exp 7308  20 female    R     1   5    24
## 3241 10-May-2018_13:32:30  Exp 7308  20 female    R     2   1     1
## 3242 10-May-2018_13:32:33  Exp 7308  20 female    R     2   1     2
## 3243 10-May-2018_13:32:35  Exp 7308  20 female    R     2   1     3
## 3244 10-May-2018_13:32:38  Exp 7308  20 female    R     2   1     4
## 3245 10-May-2018_13:32:40  Exp 7308  20 female    R     2   1     5
## 3246 10-May-2018_13:32:42  Exp 7308  20 female    R     2   1     6
## 3247 10-May-2018_13:32:45  Exp 7308  20 female    R     2   1     7
## 3248 10-May-2018_13:32:47  Exp 7308  20 female    R     2   1     8
## 3249 10-May-2018_13:32:50  Exp 7308  20 female    R     2   1     9
## 3250 10-May-2018_13:32:52  Exp 7308  20 female    R     2   1    10
## 3251 10-May-2018_13:32:54  Exp 7308  20 female    R     2   1    11
## 3252 10-May-2018_13:32:57  Exp 7308  20 female    R     2   1    12
## 3253 10-May-2018_13:32:59  Exp 7308  20 female    R     2   1    13
## 3254 10-May-2018_13:33:01  Exp 7308  20 female    R     2   1    14
## 3255 10-May-2018_13:33:04  Exp 7308  20 female    R     2   1    15
## 3256 10-May-2018_13:33:06  Exp 7308  20 female    R     2   1    16
## 3257 10-May-2018_13:33:08  Exp 7308  20 female    R     2   1    17
## 3258 10-May-2018_13:33:11  Exp 7308  20 female    R     2   1    18
## 3259 10-May-2018_13:33:13  Exp 7308  20 female    R     2   1    19
## 3260 10-May-2018_13:33:15  Exp 7308  20 female    R     2   1    20
## 3261 10-May-2018_13:33:18  Exp 7308  20 female    R     2   1    21
## 3262 10-May-2018_13:33:20  Exp 7308  20 female    R     2   1    22
## 3263 10-May-2018_13:33:23  Exp 7308  20 female    R     2   1    23
## 3264 10-May-2018_13:33:25  Exp 7308  20 female    R     2   1    24
## 3265 10-May-2018_13:33:27  Exp 7308  20 female    R     2   2     1
## 3266 10-May-2018_13:33:30  Exp 7308  20 female    R     2   2     2
## 3267 10-May-2018_13:33:32  Exp 7308  20 female    R     2   2     3
## 3268 10-May-2018_13:33:35  Exp 7308  20 female    R     2   2     4
## 3269 10-May-2018_13:33:37  Exp 7308  20 female    R     2   2     5
## 3270 10-May-2018_13:33:39  Exp 7308  20 female    R     2   2     6
## 3271 10-May-2018_13:33:42  Exp 7308  20 female    R     2   2     7
## 3272 10-May-2018_13:33:44  Exp 7308  20 female    R     2   2     8
## 3273 10-May-2018_13:33:46  Exp 7308  20 female    R     2   2     9
## 3274 10-May-2018_13:33:49  Exp 7308  20 female    R     2   2    10
## 3275 10-May-2018_13:33:51  Exp 7308  20 female    R     2   2    11
## 3276 10-May-2018_13:33:54  Exp 7308  20 female    R     2   2    12
## 3277 10-May-2018_13:33:56  Exp 7308  20 female    R     2   2    13
## 3278 10-May-2018_13:33:58  Exp 7308  20 female    R     2   2    14
## 3279 10-May-2018_13:34:01  Exp 7308  20 female    R     2   2    15
## 3280 10-May-2018_13:34:03  Exp 7308  20 female    R     2   2    16
## 3281 10-May-2018_13:34:06  Exp 7308  20 female    R     2   2    17
## 3282 10-May-2018_13:34:08  Exp 7308  20 female    R     2   2    18
## 3283 10-May-2018_13:34:11  Exp 7308  20 female    R     2   2    19
## 3284 10-May-2018_13:34:13  Exp 7308  20 female    R     2   2    20
## 3285 10-May-2018_13:34:15  Exp 7308  20 female    R     2   2    21
## 3286 10-May-2018_13:34:18  Exp 7308  20 female    R     2   2    22
## 3287 10-May-2018_13:34:20  Exp 7308  20 female    R     2   2    23
## 3288 10-May-2018_13:34:23  Exp 7308  20 female    R     2   2    24
## 3289 10-May-2018_13:34:25  Exp 7308  20 female    R     2   3     1
## 3290 10-May-2018_13:34:28  Exp 7308  20 female    R     2   3     2
## 3291 10-May-2018_13:34:30  Exp 7308  20 female    R     2   3     3
## 3292 10-May-2018_13:34:32  Exp 7308  20 female    R     2   3     4
## 3293 10-May-2018_13:34:35  Exp 7308  20 female    R     2   3     5
## 3294 10-May-2018_13:34:37  Exp 7308  20 female    R     2   3     6
## 3295 10-May-2018_13:34:40  Exp 7308  20 female    R     2   3     7
## 3296 10-May-2018_13:34:42  Exp 7308  20 female    R     2   3     8
## 3297 10-May-2018_13:34:44  Exp 7308  20 female    R     2   3     9
## 3298 10-May-2018_13:34:47  Exp 7308  20 female    R     2   3    10
## 3299 10-May-2018_13:34:49  Exp 7308  20 female    R     2   3    11
## 3300 10-May-2018_13:34:51  Exp 7308  20 female    R     2   3    12
## 3301 10-May-2018_13:34:54  Exp 7308  20 female    R     2   3    13
## 3302 10-May-2018_13:34:56  Exp 7308  20 female    R     2   3    14
## 3303 10-May-2018_13:34:58  Exp 7308  20 female    R     2   3    15
## 3304 10-May-2018_13:35:01  Exp 7308  20 female    R     2   3    16
## 3305 10-May-2018_13:35:03  Exp 7308  20 female    R     2   3    17
## 3306 10-May-2018_13:35:05  Exp 7308  20 female    R     2   3    18
## 3307 10-May-2018_13:35:08  Exp 7308  20 female    R     2   3    19
## 3308 10-May-2018_13:35:10  Exp 7308  20 female    R     2   3    20
## 3309 10-May-2018_13:35:13  Exp 7308  20 female    R     2   3    21
## 3310 10-May-2018_13:35:15  Exp 7308  20 female    R     2   3    22
## 3311 10-May-2018_13:35:18  Exp 7308  20 female    R     2   3    23
## 3312 10-May-2018_13:35:20  Exp 7308  20 female    R     2   3    24
## 3313 10-May-2018_13:35:44  Exp 7308  20 female    R     2   4     1
## 3314 10-May-2018_13:35:47  Exp 7308  20 female    R     2   4     2
## 3315 10-May-2018_13:35:49  Exp 7308  20 female    R     2   4     3
## 3316 10-May-2018_13:35:52  Exp 7308  20 female    R     2   4     4
## 3317 10-May-2018_13:35:54  Exp 7308  20 female    R     2   4     5
## 3318 10-May-2018_13:35:56  Exp 7308  20 female    R     2   4     6
## 3319 10-May-2018_13:35:59  Exp 7308  20 female    R     2   4     7
## 3320 10-May-2018_13:36:01  Exp 7308  20 female    R     2   4     8
## 3321 10-May-2018_13:36:03  Exp 7308  20 female    R     2   4     9
## 3322 10-May-2018_13:36:06  Exp 7308  20 female    R     2   4    10
## 3323 10-May-2018_13:36:08  Exp 7308  20 female    R     2   4    11
## 3324 10-May-2018_13:36:10  Exp 7308  20 female    R     2   4    12
## 3325 10-May-2018_13:36:13  Exp 7308  20 female    R     2   4    13
## 3326 10-May-2018_13:36:15  Exp 7308  20 female    R     2   4    14
## 3327 10-May-2018_13:36:18  Exp 7308  20 female    R     2   4    15
## 3328 10-May-2018_13:36:20  Exp 7308  20 female    R     2   4    16
## 3329 10-May-2018_13:36:22  Exp 7308  20 female    R     2   4    17
## 3330 10-May-2018_13:36:25  Exp 7308  20 female    R     2   4    18
## 3331 10-May-2018_13:36:27  Exp 7308  20 female    R     2   4    19
## 3332 10-May-2018_13:36:29  Exp 7308  20 female    R     2   4    20
## 3333 10-May-2018_13:36:32  Exp 7308  20 female    R     2   4    21
## 3334 10-May-2018_13:36:34  Exp 7308  20 female    R     2   4    22
## 3335 10-May-2018_13:36:37  Exp 7308  20 female    R     2   4    23
## 3336 10-May-2018_13:36:39  Exp 7308  20 female    R     2   4    24
## 3337 10-May-2018_13:36:41  Exp 7308  20 female    R     2   5     1
## 3338 10-May-2018_13:36:44  Exp 7308  20 female    R     2   5     2
## 3339 10-May-2018_13:36:46  Exp 7308  20 female    R     2   5     3
## 3340 10-May-2018_13:36:49  Exp 7308  20 female    R     2   5     4
## 3341 10-May-2018_13:36:51  Exp 7308  20 female    R     2   5     5
## 3342 10-May-2018_13:36:53  Exp 7308  20 female    R     2   5     6
## 3343 10-May-2018_13:36:56  Exp 7308  20 female    R     2   5     7
## 3344 10-May-2018_13:36:59  Exp 7308  20 female    R     2   5     8
## 3345 10-May-2018_13:37:01  Exp 7308  20 female    R     2   5     9
## 3346 10-May-2018_13:37:03  Exp 7308  20 female    R     2   5    10
## 3347 10-May-2018_13:37:06  Exp 7308  20 female    R     2   5    11
## 3348 10-May-2018_13:37:08  Exp 7308  20 female    R     2   5    12
## 3349 10-May-2018_13:37:11  Exp 7308  20 female    R     2   5    13
## 3350 10-May-2018_13:37:13  Exp 7308  20 female    R     2   5    14
## 3351 10-May-2018_13:37:15  Exp 7308  20 female    R     2   5    15
## 3352 10-May-2018_13:37:18  Exp 7308  20 female    R     2   5    16
## 3353 10-May-2018_13:37:20  Exp 7308  20 female    R     2   5    17
## 3354 10-May-2018_13:37:22  Exp 7308  20 female    R     2   5    18
## 3355 10-May-2018_13:37:25  Exp 7308  20 female    R     2   5    19
## 3356 10-May-2018_13:37:27  Exp 7308  20 female    R     2   5    20
## 3357 10-May-2018_13:37:29  Exp 7308  20 female    R     2   5    21
## 3358 10-May-2018_13:37:32  Exp 7308  20 female    R     2   5    22
## 3359 10-May-2018_13:37:34  Exp 7308  20 female    R     2   5    23
## 3360 10-May-2018_13:37:36  Exp 7308  20 female    R     2   5    24
## 3361 10-May-2018_13:37:42  Exp 7308  20 female    R     3   1     1
## 3362 10-May-2018_13:37:44  Exp 7308  20 female    R     3   1     2
## 3363 10-May-2018_13:37:46  Exp 7308  20 female    R     3   1     3
## 3364 10-May-2018_13:37:48  Exp 7308  20 female    R     3   1     4
## 3365 10-May-2018_13:37:51  Exp 7308  20 female    R     3   1     5
## 3366 10-May-2018_13:37:53  Exp 7308  20 female    R     3   1     6
## 3367 10-May-2018_13:37:56  Exp 7308  20 female    R     3   1     7
## 3368 10-May-2018_13:37:58  Exp 7308  20 female    R     3   1     8
## 3369 10-May-2018_13:38:01  Exp 7308  20 female    R     3   1     9
## 3370 10-May-2018_13:38:03  Exp 7308  20 female    R     3   1    10
## 3371 10-May-2018_13:38:05  Exp 7308  20 female    R     3   1    11
## 3372 10-May-2018_13:38:08  Exp 7308  20 female    R     3   1    12
## 3373 10-May-2018_13:38:10  Exp 7308  20 female    R     3   1    13
## 3374 10-May-2018_13:38:12  Exp 7308  20 female    R     3   1    14
## 3375 10-May-2018_13:38:15  Exp 7308  20 female    R     3   1    15
## 3376 10-May-2018_13:38:17  Exp 7308  20 female    R     3   1    16
## 3377 10-May-2018_13:38:19  Exp 7308  20 female    R     3   1    17
## 3378 10-May-2018_13:38:22  Exp 7308  20 female    R     3   1    18
## 3379 10-May-2018_13:38:24  Exp 7308  20 female    R     3   1    19
## 3380 10-May-2018_13:38:26  Exp 7308  20 female    R     3   1    20
## 3381 10-May-2018_13:38:29  Exp 7308  20 female    R     3   1    21
## 3382 10-May-2018_13:38:31  Exp 7308  20 female    R     3   1    22
## 3383 10-May-2018_13:38:34  Exp 7308  20 female    R     3   1    23
## 3384 10-May-2018_13:38:36  Exp 7308  20 female    R     3   1    24
## 3385 10-May-2018_13:38:38  Exp 7308  20 female    R     3   2     1
## 3386 10-May-2018_13:38:41  Exp 7308  20 female    R     3   2     2
## 3387 10-May-2018_13:38:43  Exp 7308  20 female    R     3   2     3
## 3388 10-May-2018_13:38:45  Exp 7308  20 female    R     3   2     4
## 3389 10-May-2018_13:38:48  Exp 7308  20 female    R     3   2     5
## 3390 10-May-2018_13:38:50  Exp 7308  20 female    R     3   2     6
## 3391 10-May-2018_13:38:52  Exp 7308  20 female    R     3   2     7
## 3392 10-May-2018_13:38:55  Exp 7308  20 female    R     3   2     8
## 3393 10-May-2018_13:38:57  Exp 7308  20 female    R     3   2     9
## 3394 10-May-2018_13:39:00  Exp 7308  20 female    R     3   2    10
## 3395 10-May-2018_13:39:02  Exp 7308  20 female    R     3   2    11
## 3396 10-May-2018_13:39:04  Exp 7308  20 female    R     3   2    12
## 3397 10-May-2018_13:39:07  Exp 7308  20 female    R     3   2    13
## 3398 10-May-2018_13:39:09  Exp 7308  20 female    R     3   2    14
## 3399 10-May-2018_13:39:12  Exp 7308  20 female    R     3   2    15
## 3400 10-May-2018_13:39:14  Exp 7308  20 female    R     3   2    16
## 3401 10-May-2018_13:39:16  Exp 7308  20 female    R     3   2    17
## 3402 10-May-2018_13:39:19  Exp 7308  20 female    R     3   2    18
## 3403 10-May-2018_13:39:21  Exp 7308  20 female    R     3   2    19
## 3404 10-May-2018_13:39:23  Exp 7308  20 female    R     3   2    20
## 3405 10-May-2018_13:39:25  Exp 7308  20 female    R     3   2    21
## 3406 10-May-2018_13:39:28  Exp 7308  20 female    R     3   2    22
## 3407 10-May-2018_13:39:30  Exp 7308  20 female    R     3   2    23
## 3408 10-May-2018_13:39:32  Exp 7308  20 female    R     3   2    24
## 3409 10-May-2018_13:39:35  Exp 7308  20 female    R     3   3     1
## 3410 10-May-2018_13:39:37  Exp 7308  20 female    R     3   3     2
## 3411 10-May-2018_13:39:39  Exp 7308  20 female    R     3   3     3
## 3412 10-May-2018_13:39:42  Exp 7308  20 female    R     3   3     4
## 3413 10-May-2018_13:39:44  Exp 7308  20 female    R     3   3     5
## 3414 10-May-2018_13:39:46  Exp 7308  20 female    R     3   3     6
## 3415 10-May-2018_13:39:49  Exp 7308  20 female    R     3   3     7
## 3416 10-May-2018_13:39:51  Exp 7308  20 female    R     3   3     8
## 3417 10-May-2018_13:39:53  Exp 7308  20 female    R     3   3     9
## 3418 10-May-2018_13:39:56  Exp 7308  20 female    R     3   3    10
## 3419 10-May-2018_13:39:58  Exp 7308  20 female    R     3   3    11
## 3420 10-May-2018_13:40:00  Exp 7308  20 female    R     3   3    12
## 3421 10-May-2018_13:40:03  Exp 7308  20 female    R     3   3    13
## 3422 10-May-2018_13:40:05  Exp 7308  20 female    R     3   3    14
## 3423 10-May-2018_13:40:07  Exp 7308  20 female    R     3   3    15
## 3424 10-May-2018_13:40:10  Exp 7308  20 female    R     3   3    16
## 3425 10-May-2018_13:40:12  Exp 7308  20 female    R     3   3    17
## 3426 10-May-2018_13:40:14  Exp 7308  20 female    R     3   3    18
## 3427 10-May-2018_13:40:17  Exp 7308  20 female    R     3   3    19
## 3428 10-May-2018_13:40:19  Exp 7308  20 female    R     3   3    20
## 3429 10-May-2018_13:40:21  Exp 7308  20 female    R     3   3    21
## 3430 10-May-2018_13:40:24  Exp 7308  20 female    R     3   3    22
## 3431 10-May-2018_13:40:26  Exp 7308  20 female    R     3   3    23
## 3432 10-May-2018_13:40:28  Exp 7308  20 female    R     3   3    24
## 3433 10-May-2018_13:40:39  Exp 7308  20 female    R     3   4     1
## 3434 10-May-2018_13:40:41  Exp 7308  20 female    R     3   4     2
## 3435 10-May-2018_13:40:43  Exp 7308  20 female    R     3   4     3
## 3436 10-May-2018_13:40:46  Exp 7308  20 female    R     3   4     4
## 3437 10-May-2018_13:40:48  Exp 7308  20 female    R     3   4     5
## 3438 10-May-2018_13:40:50  Exp 7308  20 female    R     3   4     6
## 3439 10-May-2018_13:40:53  Exp 7308  20 female    R     3   4     7
## 3440 10-May-2018_13:40:55  Exp 7308  20 female    R     3   4     8
## 3441 10-May-2018_13:40:58  Exp 7308  20 female    R     3   4     9
## 3442 10-May-2018_13:41:00  Exp 7308  20 female    R     3   4    10
## 3443 10-May-2018_13:41:03  Exp 7308  20 female    R     3   4    11
## 3444 10-May-2018_13:41:05  Exp 7308  20 female    R     3   4    12
## 3445 10-May-2018_13:41:07  Exp 7308  20 female    R     3   4    13
## 3446 10-May-2018_13:41:10  Exp 7308  20 female    R     3   4    14
## 3447 10-May-2018_13:41:12  Exp 7308  20 female    R     3   4    15
## 3448 10-May-2018_13:41:15  Exp 7308  20 female    R     3   4    16
## 3449 10-May-2018_13:41:17  Exp 7308  20 female    R     3   4    17
## 3450 10-May-2018_13:41:20  Exp 7308  20 female    R     3   4    18
## 3451 10-May-2018_13:41:22  Exp 7308  20 female    R     3   4    19
## 3452 10-May-2018_13:41:24  Exp 7308  20 female    R     3   4    20
## 3453 10-May-2018_13:41:27  Exp 7308  20 female    R     3   4    21
## 3454 10-May-2018_13:41:29  Exp 7308  20 female    R     3   4    22
## 3455 10-May-2018_13:41:31  Exp 7308  20 female    R     3   4    23
## 3456 10-May-2018_13:41:33  Exp 7308  20 female    R     3   4    24
## 3457 10-May-2018_13:41:36  Exp 7308  20 female    R     3   5     1
## 3458 10-May-2018_13:41:38  Exp 7308  20 female    R     3   5     2
## 3459 10-May-2018_13:41:40  Exp 7308  20 female    R     3   5     3
## 3460 10-May-2018_13:41:42  Exp 7308  20 female    R     3   5     4
## 3461 10-May-2018_13:41:45  Exp 7308  20 female    R     3   5     5
## 3462 10-May-2018_13:41:47  Exp 7308  20 female    R     3   5     6
## 3463 10-May-2018_13:41:50  Exp 7308  20 female    R     3   5     7
## 3464 10-May-2018_13:41:52  Exp 7308  20 female    R     3   5     8
## 3465 10-May-2018_13:41:54  Exp 7308  20 female    R     3   5     9
## 3466 10-May-2018_13:41:57  Exp 7308  20 female    R     3   5    10
## 3467 10-May-2018_13:41:59  Exp 7308  20 female    R     3   5    11
## 3468 10-May-2018_13:42:01  Exp 7308  20 female    R     3   5    12
## 3469 10-May-2018_13:42:04  Exp 7308  20 female    R     3   5    13
## 3470 10-May-2018_13:42:06  Exp 7308  20 female    R     3   5    14
## 3471 10-May-2018_13:42:09  Exp 7308  20 female    R     3   5    15
## 3472 10-May-2018_13:42:11  Exp 7308  20 female    R     3   5    16
## 3473 10-May-2018_13:42:13  Exp 7308  20 female    R     3   5    17
## 3474 10-May-2018_13:42:15  Exp 7308  20 female    R     3   5    18
## 3475 10-May-2018_13:42:18  Exp 7308  20 female    R     3   5    19
## 3476 10-May-2018_13:42:20  Exp 7308  20 female    R     3   5    20
## 3477 10-May-2018_13:42:22  Exp 7308  20 female    R     3   5    21
## 3478 10-May-2018_13:42:25  Exp 7308  20 female    R     3   5    22
## 3479 10-May-2018_13:42:27  Exp 7308  20 female    R     3   5    23
## 3480 10-May-2018_13:42:29  Exp 7308  20 female    R     3   5    24
## 3481 10-May-2018_13:47:03  Exp 7308  20 female    R     1   1     1
## 3482 10-May-2018_13:47:06  Exp 7308  20 female    R     1   1     2
## 3483 10-May-2018_13:47:08  Exp 7308  20 female    R     1   1     3
## 3484 10-May-2018_13:47:10  Exp 7308  20 female    R     1   1     4
## 3485 10-May-2018_13:47:13  Exp 7308  20 female    R     1   1     5
## 3486 10-May-2018_13:47:15  Exp 7308  20 female    R     1   1     6
## 3487 10-May-2018_13:47:18  Exp 7308  20 female    R     1   1     7
## 3488 10-May-2018_13:47:20  Exp 7308  20 female    R     1   1     8
## 3489 10-May-2018_13:47:23  Exp 7308  20 female    R     1   1     9
## 3490 10-May-2018_13:47:25  Exp 7308  20 female    R     1   1    10
## 3491 10-May-2018_13:47:27  Exp 7308  20 female    R     1   1    11
## 3492 10-May-2018_13:47:30  Exp 7308  20 female    R     1   1    12
## 3493 10-May-2018_13:47:32  Exp 7308  20 female    R     1   1    13
## 3494 10-May-2018_13:47:34  Exp 7308  20 female    R     1   1    14
## 3495 10-May-2018_13:47:37  Exp 7308  20 female    R     1   1    15
## 3496 10-May-2018_13:47:39  Exp 7308  20 female    R     1   1    16
## 3497 10-May-2018_13:47:42  Exp 7308  20 female    R     1   1    17
## 3498 10-May-2018_13:47:44  Exp 7308  20 female    R     1   1    18
## 3499 10-May-2018_13:47:46  Exp 7308  20 female    R     1   1    19
## 3500 10-May-2018_13:47:49  Exp 7308  20 female    R     1   1    20
## 3501 10-May-2018_13:47:51  Exp 7308  20 female    R     1   1    21
## 3502 10-May-2018_13:47:53  Exp 7308  20 female    R     1   1    22
## 3503 10-May-2018_13:47:56  Exp 7308  20 female    R     1   1    23
## 3504 10-May-2018_13:47:58  Exp 7308  20 female    R     1   1    24
## 3505 10-May-2018_13:48:01  Exp 7308  20 female    R     1   2     1
## 3506 10-May-2018_13:48:03  Exp 7308  20 female    R     1   2     2
## 3507 10-May-2018_13:48:05  Exp 7308  20 female    R     1   2     3
## 3508 10-May-2018_13:48:07  Exp 7308  20 female    R     1   2     4
## 3509 10-May-2018_13:48:10  Exp 7308  20 female    R     1   2     5
## 3510 10-May-2018_13:48:12  Exp 7308  20 female    R     1   2     6
## 3511 10-May-2018_13:48:14  Exp 7308  20 female    R     1   2     7
## 3512 10-May-2018_13:48:17  Exp 7308  20 female    R     1   2     8
## 3513 10-May-2018_13:48:19  Exp 7308  20 female    R     1   2     9
## 3514 10-May-2018_13:48:21  Exp 7308  20 female    R     1   2    10
## 3515 10-May-2018_13:48:24  Exp 7308  20 female    R     1   2    11
## 3516 10-May-2018_13:48:27  Exp 7308  20 female    R     1   2    12
## 3517 10-May-2018_13:48:29  Exp 7308  20 female    R     1   2    13
## 3518 10-May-2018_13:48:31  Exp 7308  20 female    R     1   2    14
## 3519 10-May-2018_13:48:34  Exp 7308  20 female    R     1   2    15
## 3520 10-May-2018_13:48:36  Exp 7308  20 female    R     1   2    16
## 3521 10-May-2018_13:48:38  Exp 7308  20 female    R     1   2    17
## 3522 10-May-2018_13:48:41  Exp 7308  20 female    R     1   2    18
## 3523 10-May-2018_13:48:44  Exp 7308  20 female    R     1   2    19
## 3524 10-May-2018_13:48:46  Exp 7308  20 female    R     1   2    20
## 3525 10-May-2018_13:48:48  Exp 7308  20 female    R     1   2    21
## 3526 10-May-2018_13:48:51  Exp 7308  20 female    R     1   2    22
## 3527 10-May-2018_13:48:53  Exp 7308  20 female    R     1   2    23
## 3528 10-May-2018_13:48:56  Exp 7308  20 female    R     1   2    24
## 3529 10-May-2018_13:53:35  Exp 7308  20 female    R     1   1     1
## 3530 10-May-2018_13:53:38  Exp 7308  20 female    R     1   1     2
## 3531 10-May-2018_13:53:40  Exp 7308  20 female    R     1   1     3
## 3532 10-May-2018_13:53:42  Exp 7308  20 female    R     1   1     4
## 3533 10-May-2018_13:53:45  Exp 7308  20 female    R     1   1     5
## 3534 10-May-2018_13:53:48  Exp 7308  20 female    R     1   1     6
## 3535 10-May-2018_13:53:50  Exp 7308  20 female    R     1   1     7
## 3536 10-May-2018_13:53:52  Exp 7308  20 female    R     1   1     8
## 3537 10-May-2018_13:53:54  Exp 7308  20 female    R     1   1     9
## 3538 10-May-2018_13:53:57  Exp 7308  20 female    R     1   1    10
## 3539 10-May-2018_13:53:59  Exp 7308  20 female    R     1   1    11
## 3540 10-May-2018_13:54:01  Exp 7308  20 female    R     1   1    12
## 3541 10-May-2018_13:54:03  Exp 7308  20 female    R     1   1    13
## 3542 10-May-2018_13:54:06  Exp 7308  20 female    R     1   1    14
## 3543 10-May-2018_13:54:08  Exp 7308  20 female    R     1   1    15
## 3544 10-May-2018_13:54:11  Exp 7308  20 female    R     1   1    16
## 3545 10-May-2018_13:54:13  Exp 7308  20 female    R     1   1    17
## 3546 10-May-2018_13:54:15  Exp 7308  20 female    R     1   1    18
## 3547 10-May-2018_13:54:17  Exp 7308  20 female    R     1   1    19
## 3548 10-May-2018_13:54:20  Exp 7308  20 female    R     1   1    20
## 3549 10-May-2018_13:54:22  Exp 7308  20 female    R     1   1    21
## 3550 10-May-2018_13:54:25  Exp 7308  20 female    R     1   1    22
## 3551 10-May-2018_13:54:27  Exp 7308  20 female    R     1   1    23
## 3552 10-May-2018_13:54:29  Exp 7308  20 female    R     1   1    24
## 3553 10-May-2018_13:54:32  Exp 7308  20 female    R     1   2     1
## 3554 10-May-2018_13:54:34  Exp 7308  20 female    R     1   2     2
## 3555 10-May-2018_13:54:36  Exp 7308  20 female    R     1   2     3
## 3556 10-May-2018_13:54:39  Exp 7308  20 female    R     1   2     4
## 3557 10-May-2018_13:54:41  Exp 7308  20 female    R     1   2     5
## 3558 10-May-2018_13:54:44  Exp 7308  20 female    R     1   2     6
## 3559 10-May-2018_13:54:46  Exp 7308  20 female    R     1   2     7
## 3560 10-May-2018_13:54:48  Exp 7308  20 female    R     1   2     8
## 3561 10-May-2018_13:54:51  Exp 7308  20 female    R     1   2     9
## 3562 10-May-2018_13:54:53  Exp 7308  20 female    R     1   2    10
## 3563 10-May-2018_13:54:55  Exp 7308  20 female    R     1   2    11
## 3564 10-May-2018_13:54:58  Exp 7308  20 female    R     1   2    12
## 3565 10-May-2018_13:55:00  Exp 7308  20 female    R     1   2    13
## 3566 10-May-2018_13:55:02  Exp 7308  20 female    R     1   2    14
## 3567 10-May-2018_13:55:05  Exp 7308  20 female    R     1   2    15
## 3568 10-May-2018_13:55:07  Exp 7308  20 female    R     1   2    16
## 3569 10-May-2018_13:55:09  Exp 7308  20 female    R     1   2    17
## 3570 10-May-2018_13:55:11  Exp 7308  20 female    R     1   2    18
## 3571 10-May-2018_13:55:14  Exp 7308  20 female    R     1   2    19
## 3572 10-May-2018_13:55:16  Exp 7308  20 female    R     1   2    20
## 3573 10-May-2018_13:55:19  Exp 7308  20 female    R     1   2    21
## 3574 10-May-2018_13:55:21  Exp 7308  20 female    R     1   2    22
## 3575 10-May-2018_13:55:23  Exp 7308  20 female    R     1   2    23
## 3576 10-May-2018_13:55:25  Exp 7308  20 female    R     1   2    24
## 3577 10-May-2018_13:59:35  Exp 7308  20 female    R     1   1     1
## 3578 10-May-2018_13:59:38  Exp 7308  20 female    R     1   1     2
## 3579 10-May-2018_13:59:40  Exp 7308  20 female    R     1   1     3
## 3580 10-May-2018_13:59:42  Exp 7308  20 female    R     1   1     4
## 3581 10-May-2018_13:59:45  Exp 7308  20 female    R     1   1     5
## 3582 10-May-2018_13:59:47  Exp 7308  20 female    R     1   1     6
## 3583 10-May-2018_13:59:50  Exp 7308  20 female    R     1   1     7
## 3584 10-May-2018_13:59:52  Exp 7308  20 female    R     1   1     8
## 3585 10-May-2018_13:59:54  Exp 7308  20 female    R     1   1     9
## 3586 10-May-2018_13:59:56  Exp 7308  20 female    R     1   1    10
## 3587 10-May-2018_13:59:59  Exp 7308  20 female    R     1   1    11
## 3588 10-May-2018_14:00:01  Exp 7308  20 female    R     1   1    12
## 3589 10-May-2018_14:00:04  Exp 7308  20 female    R     1   1    13
## 3590 10-May-2018_14:00:06  Exp 7308  20 female    R     1   1    14
## 3591 10-May-2018_14:00:09  Exp 7308  20 female    R     1   1    15
## 3592 10-May-2018_14:00:11  Exp 7308  20 female    R     1   1    16
## 3593 10-May-2018_14:00:14  Exp 7308  20 female    R     1   1    17
## 3594 10-May-2018_14:00:16  Exp 7308  20 female    R     1   1    18
## 3595 10-May-2018_14:00:19  Exp 7308  20 female    R     1   1    19
## 3596 10-May-2018_14:00:21  Exp 7308  20 female    R     1   1    20
## 3597 10-May-2018_14:00:23  Exp 7308  20 female    R     1   1    21
## 3598 10-May-2018_14:00:26  Exp 7308  20 female    R     1   1    22
## 3599 10-May-2018_14:00:28  Exp 7308  20 female    R     1   1    23
## 3600 10-May-2018_14:00:30  Exp 7308  20 female    R     1   1    24
## 3601 10-May-2018_14:00:32  Exp 7308  20 female    R     1   2     1
## 3602 10-May-2018_14:00:35  Exp 7308  20 female    R     1   2     2
## 3603 10-May-2018_14:00:37  Exp 7308  20 female    R     1   2     3
## 3604 10-May-2018_14:00:39  Exp 7308  20 female    R     1   2     4
## 3605 10-May-2018_14:00:42  Exp 7308  20 female    R     1   2     5
## 3606 10-May-2018_14:00:44  Exp 7308  20 female    R     1   2     6
## 3607 10-May-2018_14:00:46  Exp 7308  20 female    R     1   2     7
## 3608 10-May-2018_14:00:49  Exp 7308  20 female    R     1   2     8
## 3609 10-May-2018_14:00:51  Exp 7308  20 female    R     1   2     9
## 3610 10-May-2018_14:00:53  Exp 7308  20 female    R     1   2    10
## 3611 10-May-2018_14:00:56  Exp 7308  20 female    R     1   2    11
## 3612 10-May-2018_14:00:58  Exp 7308  20 female    R     1   2    12
## 3613 10-May-2018_14:01:00  Exp 7308  20 female    R     1   2    13
## 3614 10-May-2018_14:01:02  Exp 7308  20 female    R     1   2    14
## 3615 10-May-2018_14:01:05  Exp 7308  20 female    R     1   2    15
## 3616 10-May-2018_14:01:07  Exp 7308  20 female    R     1   2    16
## 3617 10-May-2018_14:01:09  Exp 7308  20 female    R     1   2    17
## 3618 10-May-2018_14:01:11  Exp 7308  20 female    R     1   2    18
## 3619 10-May-2018_14:01:13  Exp 7308  20 female    R     1   2    19
## 3620 10-May-2018_14:01:16  Exp 7308  20 female    R     1   2    20
## 3621 10-May-2018_14:01:18  Exp 7308  20 female    R     1   2    21
## 3622 10-May-2018_14:01:20  Exp 7308  20 female    R     1   2    22
## 3623 10-May-2018_14:01:23  Exp 7308  20 female    R     1   2    23
## 3624 10-May-2018_14:01:25  Exp 7308  20 female    R     1   2    24
## 3625 10-May-2018_14:05:46  Exp 7308  20 female    R     1   1     1
## 3626 10-May-2018_14:05:49  Exp 7308  20 female    R     1   1     2
## 3627 10-May-2018_14:05:51  Exp 7308  20 female    R     1   1     3
## 3628 10-May-2018_14:05:53  Exp 7308  20 female    R     1   1     4
## 3629 10-May-2018_14:05:56  Exp 7308  20 female    R     1   1     5
## 3630 10-May-2018_14:05:58  Exp 7308  20 female    R     1   1     6
## 3631 10-May-2018_14:06:00  Exp 7308  20 female    R     1   1     7
## 3632 10-May-2018_14:06:02  Exp 7308  20 female    R     1   1     8
## 3633 10-May-2018_14:06:04  Exp 7308  20 female    R     1   1     9
## 3634 10-May-2018_14:06:07  Exp 7308  20 female    R     1   1    10
## 3635 10-May-2018_14:06:09  Exp 7308  20 female    R     1   1    11
## 3636 10-May-2018_14:06:11  Exp 7308  20 female    R     1   1    12
## 3637 10-May-2018_14:06:13  Exp 7308  20 female    R     1   1    13
## 3638 10-May-2018_14:06:16  Exp 7308  20 female    R     1   1    14
## 3639 10-May-2018_14:06:18  Exp 7308  20 female    R     1   1    15
## 3640 10-May-2018_14:06:20  Exp 7308  20 female    R     1   1    16
## 3641 10-May-2018_14:06:23  Exp 7308  20 female    R     1   1    17
## 3642 10-May-2018_14:06:25  Exp 7308  20 female    R     1   1    18
## 3643 10-May-2018_14:06:27  Exp 7308  20 female    R     1   1    19
## 3644 10-May-2018_14:06:30  Exp 7308  20 female    R     1   1    20
## 3645 10-May-2018_14:06:32  Exp 7308  20 female    R     1   1    21
## 3646 10-May-2018_14:06:35  Exp 7308  20 female    R     1   1    22
## 3647 10-May-2018_14:06:37  Exp 7308  20 female    R     1   1    23
## 3648 10-May-2018_14:06:39  Exp 7308  20 female    R     1   1    24
## 3649 10-May-2018_14:06:41  Exp 7308  20 female    R     1   2     1
## 3650 10-May-2018_14:06:44  Exp 7308  20 female    R     1   2     2
## 3651 10-May-2018_14:06:46  Exp 7308  20 female    R     1   2     3
## 3652 10-May-2018_14:06:49  Exp 7308  20 female    R     1   2     4
## 3653 10-May-2018_14:06:51  Exp 7308  20 female    R     1   2     5
## 3654 10-May-2018_14:06:53  Exp 7308  20 female    R     1   2     6
## 3655 10-May-2018_14:06:55  Exp 7308  20 female    R     1   2     7
## 3656 10-May-2018_14:06:58  Exp 7308  20 female    R     1   2     8
## 3657 10-May-2018_14:07:00  Exp 7308  20 female    R     1   2     9
## 3658 10-May-2018_14:07:02  Exp 7308  20 female    R     1   2    10
## 3659 10-May-2018_14:07:04  Exp 7308  20 female    R     1   2    11
## 3660 10-May-2018_14:07:07  Exp 7308  20 female    R     1   2    12
## 3661 10-May-2018_14:07:09  Exp 7308  20 female    R     1   2    13
## 3662 10-May-2018_14:07:11  Exp 7308  20 female    R     1   2    14
## 3663 10-May-2018_14:07:14  Exp 7308  20 female    R     1   2    15
## 3664 10-May-2018_14:07:16  Exp 7308  20 female    R     1   2    16
## 3665 10-May-2018_14:07:18  Exp 7308  20 female    R     1   2    17
## 3666 10-May-2018_14:07:20  Exp 7308  20 female    R     1   2    18
## 3667 10-May-2018_14:07:23  Exp 7308  20 female    R     1   2    19
## 3668 10-May-2018_14:07:25  Exp 7308  20 female    R     1   2    20
## 3669 10-May-2018_14:07:27  Exp 7308  20 female    R     1   2    21
## 3670 10-May-2018_14:07:29  Exp 7308  20 female    R     1   2    22
## 3671 10-May-2018_14:07:32  Exp 7308  20 female    R     1   2    23
## 3672 10-May-2018_14:07:34  Exp 7308  20 female    R     1   2    24
## 3673 10-May-2018_14:11:34  Exp 7308  20 female    R     1   1     1
## 3674 10-May-2018_14:11:36  Exp 7308  20 female    R     1   1     2
## 3675 10-May-2018_14:11:39  Exp 7308  20 female    R     1   1     3
## 3676 10-May-2018_14:11:41  Exp 7308  20 female    R     1   1     4
## 3677 10-May-2018_14:11:44  Exp 7308  20 female    R     1   1     5
## 3678 10-May-2018_14:11:46  Exp 7308  20 female    R     1   1     6
## 3679 10-May-2018_14:11:48  Exp 7308  20 female    R     1   1     7
## 3680 10-May-2018_14:11:50  Exp 7308  20 female    R     1   1     8
## 3681 10-May-2018_14:11:52  Exp 7308  20 female    R     1   1     9
## 3682 10-May-2018_14:11:55  Exp 7308  20 female    R     1   1    10
## 3683 10-May-2018_14:11:57  Exp 7308  20 female    R     1   1    11
## 3684 10-May-2018_14:12:00  Exp 7308  20 female    R     1   1    12
## 3685 10-May-2018_14:12:02  Exp 7308  20 female    R     1   1    13
## 3686 10-May-2018_14:12:04  Exp 7308  20 female    R     1   1    14
## 3687 10-May-2018_14:12:06  Exp 7308  20 female    R     1   1    15
## 3688 10-May-2018_14:12:09  Exp 7308  20 female    R     1   1    16
## 3689 10-May-2018_14:12:11  Exp 7308  20 female    R     1   1    17
## 3690 10-May-2018_14:12:13  Exp 7308  20 female    R     1   1    18
## 3691 10-May-2018_14:12:16  Exp 7308  20 female    R     1   1    19
## 3692 10-May-2018_14:12:18  Exp 7308  20 female    R     1   1    20
## 3693 10-May-2018_14:12:20  Exp 7308  20 female    R     1   1    21
## 3694 10-May-2018_14:12:23  Exp 7308  20 female    R     1   1    22
## 3695 10-May-2018_14:12:25  Exp 7308  20 female    R     1   1    23
## 3696 10-May-2018_14:12:27  Exp 7308  20 female    R     1   1    24
## 3697 10-May-2018_14:12:29  Exp 7308  20 female    R     1   2     1
## 3698 10-May-2018_14:12:31  Exp 7308  20 female    R     1   2     2
## 3699 10-May-2018_14:12:34  Exp 7308  20 female    R     1   2     3
## 3700 10-May-2018_14:12:36  Exp 7308  20 female    R     1   2     4
## 3701 10-May-2018_14:12:38  Exp 7308  20 female    R     1   2     5
## 3702 10-May-2018_14:12:40  Exp 7308  20 female    R     1   2     6
## 3703 10-May-2018_14:12:43  Exp 7308  20 female    R     1   2     7
## 3704 10-May-2018_14:12:45  Exp 7308  20 female    R     1   2     8
## 3705 10-May-2018_14:12:47  Exp 7308  20 female    R     1   2     9
## 3706 10-May-2018_14:12:49  Exp 7308  20 female    R     1   2    10
## 3707 10-May-2018_14:12:52  Exp 7308  20 female    R     1   2    11
## 3708 10-May-2018_14:12:54  Exp 7308  20 female    R     1   2    12
## 3709 10-May-2018_14:12:56  Exp 7308  20 female    R     1   2    13
## 3710 10-May-2018_14:12:58  Exp 7308  20 female    R     1   2    14
## 3711 10-May-2018_14:13:01  Exp 7308  20 female    R     1   2    15
## 3712 10-May-2018_14:13:03  Exp 7308  20 female    R     1   2    16
## 3713 10-May-2018_14:13:05  Exp 7308  20 female    R     1   2    17
## 3714 10-May-2018_14:13:08  Exp 7308  20 female    R     1   2    18
## 3715 10-May-2018_14:13:10  Exp 7308  20 female    R     1   2    19
## 3716 10-May-2018_14:13:12  Exp 7308  20 female    R     1   2    20
## 3717 10-May-2018_14:13:14  Exp 7308  20 female    R     1   2    21
## 3718 10-May-2018_14:13:17  Exp 7308  20 female    R     1   2    22
## 3719 10-May-2018_14:13:19  Exp 7308  20 female    R     1   2    23
## 3720 10-May-2018_14:13:21  Exp 7308  20 female    R     1   2    24
## 3721 12-May-2018_14:13:13  Exp 7309  26      1    R     1   1     1
## 3722 12-May-2018_14:13:15  Exp 7309  26      1    R     1   1     2
## 3723 12-May-2018_14:13:18  Exp 7309  26      1    R     1   1     3
## 3724 12-May-2018_14:13:20  Exp 7309  26      1    R     1   1     4
## 3725 12-May-2018_14:13:23  Exp 7309  26      1    R     1   1     5
## 3726 12-May-2018_14:13:25  Exp 7309  26      1    R     1   1     6
## 3727 12-May-2018_14:13:27  Exp 7309  26      1    R     1   1     7
## 3728 12-May-2018_14:13:30  Exp 7309  26      1    R     1   1     8
## 3729 12-May-2018_14:13:32  Exp 7309  26      1    R     1   1     9
## 3730 12-May-2018_14:13:35  Exp 7309  26      1    R     1   1    10
## 3731 12-May-2018_14:13:37  Exp 7309  26      1    R     1   1    11
## 3732 12-May-2018_14:13:40  Exp 7309  26      1    R     1   1    12
## 3733 12-May-2018_14:13:43  Exp 7309  26      1    R     1   1    13
## 3734 12-May-2018_14:13:45  Exp 7309  26      1    R     1   1    14
## 3735 12-May-2018_14:13:47  Exp 7309  26      1    R     1   1    15
## 3736 12-May-2018_14:13:50  Exp 7309  26      1    R     1   1    16
## 3737 12-May-2018_14:13:52  Exp 7309  26      1    R     1   1    17
## 3738 12-May-2018_14:13:55  Exp 7309  26      1    R     1   1    18
## 3739 12-May-2018_14:13:57  Exp 7309  26      1    R     1   1    19
## 3740 12-May-2018_14:13:59  Exp 7309  26      1    R     1   1    20
## 3741 12-May-2018_14:14:02  Exp 7309  26      1    R     1   1    21
## 3742 12-May-2018_14:14:04  Exp 7309  26      1    R     1   1    22
## 3743 12-May-2018_14:14:06  Exp 7309  26      1    R     1   1    23
## 3744 12-May-2018_14:14:09  Exp 7309  26      1    R     1   1    24
## 3745 12-May-2018_14:14:11  Exp 7309  26      1    R     1   2     1
## 3746 12-May-2018_14:14:13  Exp 7309  26      1    R     1   2     2
## 3747 12-May-2018_14:14:16  Exp 7309  26      1    R     1   2     3
## 3748 12-May-2018_14:14:18  Exp 7309  26      1    R     1   2     4
## 3749 12-May-2018_14:14:20  Exp 7309  26      1    R     1   2     5
## 3750 12-May-2018_14:14:23  Exp 7309  26      1    R     1   2     6
## 3751 12-May-2018_14:14:25  Exp 7309  26      1    R     1   2     7
## 3752 12-May-2018_14:14:28  Exp 7309  26      1    R     1   2     8
## 3753 12-May-2018_14:14:30  Exp 7309  26      1    R     1   2     9
## 3754 12-May-2018_14:14:33  Exp 7309  26      1    R     1   2    10
## 3755 12-May-2018_14:14:35  Exp 7309  26      1    R     1   2    11
## 3756 12-May-2018_14:14:37  Exp 7309  26      1    R     1   2    12
## 3757 12-May-2018_14:14:39  Exp 7309  26      1    R     1   2    13
## 3758 12-May-2018_14:14:42  Exp 7309  26      1    R     1   2    14
## 3759 12-May-2018_14:14:44  Exp 7309  26      1    R     1   2    15
## 3760 12-May-2018_14:14:47  Exp 7309  26      1    R     1   2    16
## 3761 12-May-2018_14:14:49  Exp 7309  26      1    R     1   2    17
## 3762 12-May-2018_14:14:52  Exp 7309  26      1    R     1   2    18
## 3763 12-May-2018_14:14:54  Exp 7309  26      1    R     1   2    19
## 3764 12-May-2018_14:14:56  Exp 7309  26      1    R     1   2    20
## 3765 12-May-2018_14:14:58  Exp 7309  26      1    R     1   2    21
## 3766 12-May-2018_14:15:01  Exp 7309  26      1    R     1   2    22
## 3767 12-May-2018_14:15:03  Exp 7309  26      1    R     1   2    23
## 3768 12-May-2018_14:15:05  Exp 7309  26      1    R     1   2    24
## 3769 12-May-2018_14:15:08  Exp 7309  26      1    R     1   3     1
## 3770 12-May-2018_14:15:10  Exp 7309  26      1    R     1   3     2
## 3771 12-May-2018_14:15:13  Exp 7309  26      1    R     1   3     3
## 3772 12-May-2018_14:15:15  Exp 7309  26      1    R     1   3     4
## 3773 12-May-2018_14:15:17  Exp 7309  26      1    R     1   3     5
## 3774 12-May-2018_14:15:20  Exp 7309  26      1    R     1   3     6
## 3775 12-May-2018_14:15:22  Exp 7309  26      1    R     1   3     7
## 3776 12-May-2018_14:15:24  Exp 7309  26      1    R     1   3     8
## 3777 12-May-2018_14:15:27  Exp 7309  26      1    R     1   3     9
## 3778 12-May-2018_14:15:29  Exp 7309  26      1    R     1   3    10
## 3779 12-May-2018_14:15:31  Exp 7309  26      1    R     1   3    11
## 3780 12-May-2018_14:15:34  Exp 7309  26      1    R     1   3    12
## 3781 12-May-2018_14:15:36  Exp 7309  26      1    R     1   3    13
## 3782 12-May-2018_14:15:39  Exp 7309  26      1    R     1   3    14
## 3783 12-May-2018_14:15:41  Exp 7309  26      1    R     1   3    15
## 3784 12-May-2018_14:15:43  Exp 7309  26      1    R     1   3    16
## 3785 12-May-2018_14:15:46  Exp 7309  26      1    R     1   3    17
## 3786 12-May-2018_14:15:48  Exp 7309  26      1    R     1   3    18
## 3787 12-May-2018_14:15:50  Exp 7309  26      1    R     1   3    19
## 3788 12-May-2018_14:15:53  Exp 7309  26      1    R     1   3    20
## 3789 12-May-2018_14:15:55  Exp 7309  26      1    R     1   3    21
## 3790 12-May-2018_14:15:58  Exp 7309  26      1    R     1   3    22
## 3791 12-May-2018_14:16:00  Exp 7309  26      1    R     1   3    23
## 3792 12-May-2018_14:16:02  Exp 7309  26      1    R     1   3    24
## 3793 12-May-2018_14:16:25  Exp 7309  26      1    R     1   4     1
## 3794 12-May-2018_14:16:27  Exp 7309  26      1    R     1   4     2
## 3795 12-May-2018_14:16:29  Exp 7309  26      1    R     1   4     3
## 3796 12-May-2018_14:16:32  Exp 7309  26      1    R     1   4     4
## 3797 12-May-2018_14:16:34  Exp 7309  26      1    R     1   4     5
## 3798 12-May-2018_14:16:36  Exp 7309  26      1    R     1   4     6
## 3799 12-May-2018_14:16:39  Exp 7309  26      1    R     1   4     7
## 3800 12-May-2018_14:16:41  Exp 7309  26      1    R     1   4     8
## 3801 12-May-2018_14:16:43  Exp 7309  26      1    R     1   4     9
## 3802 12-May-2018_14:16:46  Exp 7309  26      1    R     1   4    10
## 3803 12-May-2018_14:16:48  Exp 7309  26      1    R     1   4    11
## 3804 12-May-2018_14:16:50  Exp 7309  26      1    R     1   4    12
## 3805 12-May-2018_14:16:53  Exp 7309  26      1    R     1   4    13
## 3806 12-May-2018_14:16:56  Exp 7309  26      1    R     1   4    14
## 3807 12-May-2018_14:16:58  Exp 7309  26      1    R     1   4    15
## 3808 12-May-2018_14:17:00  Exp 7309  26      1    R     1   4    16
## 3809 12-May-2018_14:17:03  Exp 7309  26      1    R     1   4    17
## 3810 12-May-2018_14:17:05  Exp 7309  26      1    R     1   4    18
## 3811 12-May-2018_14:17:07  Exp 7309  26      1    R     1   4    19
## 3812 12-May-2018_14:17:10  Exp 7309  26      1    R     1   4    20
## 3813 12-May-2018_14:17:12  Exp 7309  26      1    R     1   4    21
## 3814 12-May-2018_14:17:15  Exp 7309  26      1    R     1   4    22
## 3815 12-May-2018_14:17:17  Exp 7309  26      1    R     1   4    23
## 3816 12-May-2018_14:17:20  Exp 7309  26      1    R     1   4    24
## 3817 12-May-2018_14:17:22  Exp 7309  26      1    R     1   5     1
## 3818 12-May-2018_14:17:25  Exp 7309  26      1    R     1   5     2
## 3819 12-May-2018_14:17:27  Exp 7309  26      1    R     1   5     3
## 3820 12-May-2018_14:17:30  Exp 7309  26      1    R     1   5     4
## 3821 12-May-2018_14:17:32  Exp 7309  26      1    R     1   5     5
## 3822 12-May-2018_14:17:34  Exp 7309  26      1    R     1   5     6
## 3823 12-May-2018_14:17:37  Exp 7309  26      1    R     1   5     7
## 3824 12-May-2018_14:17:39  Exp 7309  26      1    R     1   5     8
## 3825 12-May-2018_14:17:42  Exp 7309  26      1    R     1   5     9
## 3826 12-May-2018_14:17:44  Exp 7309  26      1    R     1   5    10
## 3827 12-May-2018_14:17:46  Exp 7309  26      1    R     1   5    11
## 3828 12-May-2018_14:17:49  Exp 7309  26      1    R     1   5    12
## 3829 12-May-2018_14:17:51  Exp 7309  26      1    R     1   5    13
## 3830 12-May-2018_14:17:54  Exp 7309  26      1    R     1   5    14
## 3831 12-May-2018_14:17:56  Exp 7309  26      1    R     1   5    15
## 3832 12-May-2018_14:17:59  Exp 7309  26      1    R     1   5    16
## 3833 12-May-2018_14:18:01  Exp 7309  26      1    R     1   5    17
## 3834 12-May-2018_14:18:04  Exp 7309  26      1    R     1   5    18
## 3835 12-May-2018_14:18:06  Exp 7309  26      1    R     1   5    19
## 3836 12-May-2018_14:18:08  Exp 7309  26      1    R     1   5    20
## 3837 12-May-2018_14:18:11  Exp 7309  26      1    R     1   5    21
## 3838 12-May-2018_14:18:13  Exp 7309  26      1    R     1   5    22
## 3839 12-May-2018_14:18:15  Exp 7309  26      1    R     1   5    23
## 3840 12-May-2018_14:18:17  Exp 7309  26      1    R     1   5    24
## 3841 12-May-2018_14:18:23  Exp 7309  26      1    R     2   1     1
## 3842 12-May-2018_14:18:25  Exp 7309  26      1    R     2   1     2
## 3843 12-May-2018_14:18:27  Exp 7309  26      1    R     2   1     3
## 3844 12-May-2018_14:18:30  Exp 7309  26      1    R     2   1     4
## 3845 12-May-2018_14:18:32  Exp 7309  26      1    R     2   1     5
## 3846 12-May-2018_14:18:34  Exp 7309  26      1    R     2   1     6
## 3847 12-May-2018_14:18:37  Exp 7309  26      1    R     2   1     7
## 3848 12-May-2018_14:18:39  Exp 7309  26      1    R     2   1     8
## 3849 12-May-2018_14:18:41  Exp 7309  26      1    R     2   1     9
## 3850 12-May-2018_14:18:43  Exp 7309  26      1    R     2   1    10
## 3851 12-May-2018_14:18:46  Exp 7309  26      1    R     2   1    11
## 3852 12-May-2018_14:18:48  Exp 7309  26      1    R     2   1    12
## 3853 12-May-2018_14:18:51  Exp 7309  26      1    R     2   1    13
## 3854 12-May-2018_14:18:53  Exp 7309  26      1    R     2   1    14
## 3855 12-May-2018_14:18:55  Exp 7309  26      1    R     2   1    15
## 3856 12-May-2018_14:18:57  Exp 7309  26      1    R     2   1    16
## 3857 12-May-2018_14:19:00  Exp 7309  26      1    R     2   1    17
## 3858 12-May-2018_14:19:02  Exp 7309  26      1    R     2   1    18
## 3859 12-May-2018_14:19:05  Exp 7309  26      1    R     2   1    19
## 3860 12-May-2018_14:19:07  Exp 7309  26      1    R     2   1    20
## 3861 12-May-2018_14:19:09  Exp 7309  26      1    R     2   1    21
## 3862 12-May-2018_14:19:11  Exp 7309  26      1    R     2   1    22
## 3863 12-May-2018_14:19:14  Exp 7309  26      1    R     2   1    23
## 3864 12-May-2018_14:19:16  Exp 7309  26      1    R     2   1    24
## 3865 12-May-2018_14:19:18  Exp 7309  26      1    R     2   2     1
## 3866 12-May-2018_14:19:21  Exp 7309  26      1    R     2   2     2
## 3867 12-May-2018_14:19:23  Exp 7309  26      1    R     2   2     3
## 3868 12-May-2018_14:19:25  Exp 7309  26      1    R     2   2     4
## 3869 12-May-2018_14:19:27  Exp 7309  26      1    R     2   2     5
## 3870 12-May-2018_14:19:30  Exp 7309  26      1    R     2   2     6
## 3871 12-May-2018_14:19:32  Exp 7309  26      1    R     2   2     7
## 3872 12-May-2018_14:19:35  Exp 7309  26      1    R     2   2     8
## 3873 12-May-2018_14:19:37  Exp 7309  26      1    R     2   2     9
## 3874 12-May-2018_14:19:39  Exp 7309  26      1    R     2   2    10
## 3875 12-May-2018_14:19:42  Exp 7309  26      1    R     2   2    11
## 3876 12-May-2018_14:19:44  Exp 7309  26      1    R     2   2    12
## 3877 12-May-2018_14:19:46  Exp 7309  26      1    R     2   2    13
## 3878 12-May-2018_14:19:49  Exp 7309  26      1    R     2   2    14
## 3879 12-May-2018_14:19:51  Exp 7309  26      1    R     2   2    15
## 3880 12-May-2018_14:19:54  Exp 7309  26      1    R     2   2    16
## 3881 12-May-2018_14:19:56  Exp 7309  26      1    R     2   2    17
## 3882 12-May-2018_14:19:59  Exp 7309  26      1    R     2   2    18
## 3883 12-May-2018_14:20:01  Exp 7309  26      1    R     2   2    19
## 3884 12-May-2018_14:20:03  Exp 7309  26      1    R     2   2    20
## 3885 12-May-2018_14:20:06  Exp 7309  26      1    R     2   2    21
## 3886 12-May-2018_14:20:08  Exp 7309  26      1    R     2   2    22
## 3887 12-May-2018_14:20:10  Exp 7309  26      1    R     2   2    23
## 3888 12-May-2018_14:20:12  Exp 7309  26      1    R     2   2    24
## 3889 12-May-2018_14:20:15  Exp 7309  26      1    R     2   3     1
## 3890 12-May-2018_14:20:17  Exp 7309  26      1    R     2   3     2
## 3891 12-May-2018_14:20:19  Exp 7309  26      1    R     2   3     3
## 3892 12-May-2018_14:20:22  Exp 7309  26      1    R     2   3     4
## 3893 12-May-2018_14:20:24  Exp 7309  26      1    R     2   3     5
## 3894 12-May-2018_14:20:26  Exp 7309  26      1    R     2   3     6
## 3895 12-May-2018_14:20:28  Exp 7309  26      1    R     2   3     7
## 3896 12-May-2018_14:20:31  Exp 7309  26      1    R     2   3     8
## 3897 12-May-2018_14:20:34  Exp 7309  26      1    R     2   3     9
## 3898 12-May-2018_14:20:36  Exp 7309  26      1    R     2   3    10
## 3899 12-May-2018_14:20:38  Exp 7309  26      1    R     2   3    11
## 3900 12-May-2018_14:20:41  Exp 7309  26      1    R     2   3    12
## 3901 12-May-2018_14:20:43  Exp 7309  26      1    R     2   3    13
## 3902 12-May-2018_14:20:45  Exp 7309  26      1    R     2   3    14
## 3903 12-May-2018_14:20:48  Exp 7309  26      1    R     2   3    15
## 3904 12-May-2018_14:20:50  Exp 7309  26      1    R     2   3    16
## 3905 12-May-2018_14:20:52  Exp 7309  26      1    R     2   3    17
## 3906 12-May-2018_14:20:55  Exp 7309  26      1    R     2   3    18
## 3907 12-May-2018_14:20:57  Exp 7309  26      1    R     2   3    19
## 3908 12-May-2018_14:20:59  Exp 7309  26      1    R     2   3    20
## 3909 12-May-2018_14:21:02  Exp 7309  26      1    R     2   3    21
## 3910 12-May-2018_14:21:04  Exp 7309  26      1    R     2   3    22
## 3911 12-May-2018_14:21:07  Exp 7309  26      1    R     2   3    23
## 3912 12-May-2018_14:21:09  Exp 7309  26      1    R     2   3    24
## 3913 12-May-2018_14:21:25  Exp 7309  26      1    R     2   4     1
## 3914 12-May-2018_14:21:27  Exp 7309  26      1    R     2   4     2
## 3915 12-May-2018_14:21:30  Exp 7309  26      1    R     2   4     3
## 3916 12-May-2018_14:21:32  Exp 7309  26      1    R     2   4     4
## 3917 12-May-2018_14:21:34  Exp 7309  26      1    R     2   4     5
## 3918 12-May-2018_14:21:37  Exp 7309  26      1    R     2   4     6
## 3919 12-May-2018_14:21:39  Exp 7309  26      1    R     2   4     7
## 3920 12-May-2018_14:21:42  Exp 7309  26      1    R     2   4     8
## 3921 12-May-2018_14:21:44  Exp 7309  26      1    R     2   4     9
## 3922 12-May-2018_14:21:46  Exp 7309  26      1    R     2   4    10
## 3923 12-May-2018_14:21:49  Exp 7309  26      1    R     2   4    11
## 3924 12-May-2018_14:21:51  Exp 7309  26      1    R     2   4    12
## 3925 12-May-2018_14:21:53  Exp 7309  26      1    R     2   4    13
## 3926 12-May-2018_14:21:55  Exp 7309  26      1    R     2   4    14
## 3927 12-May-2018_14:21:58  Exp 7309  26      1    R     2   4    15
## 3928 12-May-2018_14:22:00  Exp 7309  26      1    R     2   4    16
## 3929 12-May-2018_14:22:03  Exp 7309  26      1    R     2   4    17
## 3930 12-May-2018_14:22:05  Exp 7309  26      1    R     2   4    18
## 3931 12-May-2018_14:22:07  Exp 7309  26      1    R     2   4    19
## 3932 12-May-2018_14:22:10  Exp 7309  26      1    R     2   4    20
## 3933 12-May-2018_14:22:12  Exp 7309  26      1    R     2   4    21
## 3934 12-May-2018_14:22:14  Exp 7309  26      1    R     2   4    22
## 3935 12-May-2018_14:22:17  Exp 7309  26      1    R     2   4    23
## 3936 12-May-2018_14:22:19  Exp 7309  26      1    R     2   4    24
## 3937 12-May-2018_14:22:21  Exp 7309  26      1    R     2   5     1
## 3938 12-May-2018_14:22:23  Exp 7309  26      1    R     2   5     2
## 3939 12-May-2018_14:22:25  Exp 7309  26      1    R     2   5     3
## 3940 12-May-2018_14:22:28  Exp 7309  26      1    R     2   5     4
## 3941 12-May-2018_14:22:30  Exp 7309  26      1    R     2   5     5
## 3942 12-May-2018_14:22:32  Exp 7309  26      1    R     2   5     6
## 3943 12-May-2018_14:22:35  Exp 7309  26      1    R     2   5     7
## 3944 12-May-2018_14:22:37  Exp 7309  26      1    R     2   5     8
## 3945 12-May-2018_14:22:39  Exp 7309  26      1    R     2   5     9
## 3946 12-May-2018_14:22:42  Exp 7309  26      1    R     2   5    10
## 3947 12-May-2018_14:22:44  Exp 7309  26      1    R     2   5    11
## 3948 12-May-2018_14:22:46  Exp 7309  26      1    R     2   5    12
## 3949 12-May-2018_14:22:48  Exp 7309  26      1    R     2   5    13
## 3950 12-May-2018_14:22:51  Exp 7309  26      1    R     2   5    14
## 3951 12-May-2018_14:22:53  Exp 7309  26      1    R     2   5    15
## 3952 12-May-2018_14:22:55  Exp 7309  26      1    R     2   5    16
## 3953 12-May-2018_14:22:57  Exp 7309  26      1    R     2   5    17
## 3954 12-May-2018_14:23:00  Exp 7309  26      1    R     2   5    18
## 3955 12-May-2018_14:23:02  Exp 7309  26      1    R     2   5    19
## 3956 12-May-2018_14:23:04  Exp 7309  26      1    R     2   5    20
## 3957 12-May-2018_14:23:06  Exp 7309  26      1    R     2   5    21
## 3958 12-May-2018_14:23:09  Exp 7309  26      1    R     2   5    22
## 3959 12-May-2018_14:23:11  Exp 7309  26      1    R     2   5    23
## 3960 12-May-2018_14:23:13  Exp 7309  26      1    R     2   5    24
## 3961 12-May-2018_14:23:19  Exp 7309  26      1    R     3   1     1
## 3962 12-May-2018_14:23:21  Exp 7309  26      1    R     3   1     2
## 3963 12-May-2018_14:23:24  Exp 7309  26      1    R     3   1     3
## 3964 12-May-2018_14:23:26  Exp 7309  26      1    R     3   1     4
## 3965 12-May-2018_14:23:29  Exp 7309  26      1    R     3   1     5
## 3966 12-May-2018_14:23:31  Exp 7309  26      1    R     3   1     6
## 3967 12-May-2018_14:23:34  Exp 7309  26      1    R     3   1     7
## 3968 12-May-2018_14:23:36  Exp 7309  26      1    R     3   1     8
## 3969 12-May-2018_14:23:38  Exp 7309  26      1    R     3   1     9
## 3970 12-May-2018_14:23:41  Exp 7309  26      1    R     3   1    10
## 3971 12-May-2018_14:23:43  Exp 7309  26      1    R     3   1    11
## 3972 12-May-2018_14:23:45  Exp 7309  26      1    R     3   1    12
## 3973 12-May-2018_14:23:47  Exp 7309  26      1    R     3   1    13
## 3974 12-May-2018_14:23:49  Exp 7309  26      1    R     3   1    14
## 3975 12-May-2018_14:23:52  Exp 7309  26      1    R     3   1    15
## 3976 12-May-2018_14:23:54  Exp 7309  26      1    R     3   1    16
## 3977 12-May-2018_14:23:56  Exp 7309  26      1    R     3   1    17
## 3978 12-May-2018_14:23:59  Exp 7309  26      1    R     3   1    18
## 3979 12-May-2018_14:24:01  Exp 7309  26      1    R     3   1    19
## 3980 12-May-2018_14:24:03  Exp 7309  26      1    R     3   1    20
## 3981 12-May-2018_14:24:05  Exp 7309  26      1    R     3   1    21
## 3982 12-May-2018_14:24:07  Exp 7309  26      1    R     3   1    22
## 3983 12-May-2018_14:24:10  Exp 7309  26      1    R     3   1    23
## 3984 12-May-2018_14:24:12  Exp 7309  26      1    R     3   1    24
## 3985 12-May-2018_14:24:14  Exp 7309  26      1    R     3   2     1
## 3986 12-May-2018_14:24:16  Exp 7309  26      1    R     3   2     2
## 3987 12-May-2018_14:24:18  Exp 7309  26      1    R     3   2     3
## 3988 12-May-2018_14:24:21  Exp 7309  26      1    R     3   2     4
## 3989 12-May-2018_14:24:23  Exp 7309  26      1    R     3   2     5
## 3990 12-May-2018_14:24:26  Exp 7309  26      1    R     3   2     6
## 3991 12-May-2018_14:24:28  Exp 7309  26      1    R     3   2     7
## 3992 12-May-2018_14:24:30  Exp 7309  26      1    R     3   2     8
## 3993 12-May-2018_14:24:33  Exp 7309  26      1    R     3   2     9
## 3994 12-May-2018_14:24:35  Exp 7309  26      1    R     3   2    10
## 3995 12-May-2018_14:24:37  Exp 7309  26      1    R     3   2    11
## 3996 12-May-2018_14:24:39  Exp 7309  26      1    R     3   2    12
## 3997 12-May-2018_14:24:42  Exp 7309  26      1    R     3   2    13
## 3998 12-May-2018_14:24:44  Exp 7309  26      1    R     3   2    14
## 3999 12-May-2018_14:24:46  Exp 7309  26      1    R     3   2    15
## 4000 12-May-2018_14:24:48  Exp 7309  26      1    R     3   2    16
## 4001 12-May-2018_14:24:51  Exp 7309  26      1    R     3   2    17
## 4002 12-May-2018_14:24:53  Exp 7309  26      1    R     3   2    18
## 4003 12-May-2018_14:24:55  Exp 7309  26      1    R     3   2    19
## 4004 12-May-2018_14:24:58  Exp 7309  26      1    R     3   2    20
## 4005 12-May-2018_14:25:00  Exp 7309  26      1    R     3   2    21
## 4006 12-May-2018_14:25:02  Exp 7309  26      1    R     3   2    22
## 4007 12-May-2018_14:25:05  Exp 7309  26      1    R     3   2    23
## 4008 12-May-2018_14:25:07  Exp 7309  26      1    R     3   2    24
## 4009 12-May-2018_14:25:09  Exp 7309  26      1    R     3   3     1
## 4010 12-May-2018_14:25:12  Exp 7309  26      1    R     3   3     2
## 4011 12-May-2018_14:25:14  Exp 7309  26      1    R     3   3     3
## 4012 12-May-2018_14:25:16  Exp 7309  26      1    R     3   3     4
## 4013 12-May-2018_14:25:18  Exp 7309  26      1    R     3   3     5
## 4014 12-May-2018_14:25:21  Exp 7309  26      1    R     3   3     6
## 4015 12-May-2018_14:25:23  Exp 7309  26      1    R     3   3     7
## 4016 12-May-2018_14:25:25  Exp 7309  26      1    R     3   3     8
## 4017 12-May-2018_14:25:28  Exp 7309  26      1    R     3   3     9
## 4018 12-May-2018_14:25:30  Exp 7309  26      1    R     3   3    10
## 4019 12-May-2018_14:25:33  Exp 7309  26      1    R     3   3    11
## 4020 12-May-2018_14:25:35  Exp 7309  26      1    R     3   3    12
## 4021 12-May-2018_14:25:37  Exp 7309  26      1    R     3   3    13
## 4022 12-May-2018_14:25:40  Exp 7309  26      1    R     3   3    14
## 4023 12-May-2018_14:25:42  Exp 7309  26      1    R     3   3    15
## 4024 12-May-2018_14:25:44  Exp 7309  26      1    R     3   3    16
## 4025 12-May-2018_14:25:46  Exp 7309  26      1    R     3   3    17
## 4026 12-May-2018_14:25:49  Exp 7309  26      1    R     3   3    18
## 4027 12-May-2018_14:25:51  Exp 7309  26      1    R     3   3    19
## 4028 12-May-2018_14:25:53  Exp 7309  26      1    R     3   3    20
## 4029 12-May-2018_14:25:56  Exp 7309  26      1    R     3   3    21
## 4030 12-May-2018_14:25:58  Exp 7309  26      1    R     3   3    22
## 4031 12-May-2018_14:26:00  Exp 7309  26      1    R     3   3    23
## 4032 12-May-2018_14:26:03  Exp 7309  26      1    R     3   3    24
## 4033 12-May-2018_14:26:23  Exp 7309  26      1    R     3   4     1
## 4034 12-May-2018_14:26:25  Exp 7309  26      1    R     3   4     2
## 4035 12-May-2018_14:26:27  Exp 7309  26      1    R     3   4     3
## 4036 12-May-2018_14:26:30  Exp 7309  26      1    R     3   4     4
## 4037 12-May-2018_14:26:32  Exp 7309  26      1    R     3   4     5
## 4038 12-May-2018_14:26:34  Exp 7309  26      1    R     3   4     6
## 4039 12-May-2018_14:26:36  Exp 7309  26      1    R     3   4     7
## 4040 12-May-2018_14:26:38  Exp 7309  26      1    R     3   4     8
## 4041 12-May-2018_14:26:41  Exp 7309  26      1    R     3   4     9
## 4042 12-May-2018_14:26:43  Exp 7309  26      1    R     3   4    10
## 4043 12-May-2018_14:26:45  Exp 7309  26      1    R     3   4    11
## 4044 12-May-2018_14:26:47  Exp 7309  26      1    R     3   4    12
## 4045 12-May-2018_14:26:50  Exp 7309  26      1    R     3   4    13
## 4046 12-May-2018_14:26:52  Exp 7309  26      1    R     3   4    14
## 4047 12-May-2018_14:26:55  Exp 7309  26      1    R     3   4    15
## 4048 12-May-2018_14:26:57  Exp 7309  26      1    R     3   4    16
## 4049 12-May-2018_14:26:59  Exp 7309  26      1    R     3   4    17
## 4050 12-May-2018_14:27:01  Exp 7309  26      1    R     3   4    18
## 4051 12-May-2018_14:27:04  Exp 7309  26      1    R     3   4    19
## 4052 12-May-2018_14:27:06  Exp 7309  26      1    R     3   4    20
## 4053 12-May-2018_14:27:09  Exp 7309  26      1    R     3   4    21
## 4054 12-May-2018_14:27:11  Exp 7309  26      1    R     3   4    22
## 4055 12-May-2018_14:27:13  Exp 7309  26      1    R     3   4    23
## 4056 12-May-2018_14:27:16  Exp 7309  26      1    R     3   4    24
## 4057 12-May-2018_14:27:18  Exp 7309  26      1    R     3   5     1
## 4058 12-May-2018_14:27:20  Exp 7309  26      1    R     3   5     2
## 4059 12-May-2018_14:27:23  Exp 7309  26      1    R     3   5     3
## 4060 12-May-2018_14:27:25  Exp 7309  26      1    R     3   5     4
## 4061 12-May-2018_14:27:27  Exp 7309  26      1    R     3   5     5
## 4062 12-May-2018_14:27:30  Exp 7309  26      1    R     3   5     6
## 4063 12-May-2018_14:27:32  Exp 7309  26      1    R     3   5     7
## 4064 12-May-2018_14:27:34  Exp 7309  26      1    R     3   5     8
## 4065 12-May-2018_14:27:37  Exp 7309  26      1    R     3   5     9
## 4066 12-May-2018_14:27:39  Exp 7309  26      1    R     3   5    10
## 4067 12-May-2018_14:27:42  Exp 7309  26      1    R     3   5    11
## 4068 12-May-2018_14:27:44  Exp 7309  26      1    R     3   5    12
## 4069 12-May-2018_14:27:46  Exp 7309  26      1    R     3   5    13
## 4070 12-May-2018_14:27:49  Exp 7309  26      1    R     3   5    14
## 4071 12-May-2018_14:27:51  Exp 7309  26      1    R     3   5    15
## 4072 12-May-2018_14:27:53  Exp 7309  26      1    R     3   5    16
## 4073 12-May-2018_14:27:56  Exp 7309  26      1    R     3   5    17
## 4074 12-May-2018_14:27:58  Exp 7309  26      1    R     3   5    18
## 4075 12-May-2018_14:28:00  Exp 7309  26      1    R     3   5    19
## 4076 12-May-2018_14:28:03  Exp 7309  26      1    R     3   5    20
## 4077 12-May-2018_14:28:05  Exp 7309  26      1    R     3   5    21
## 4078 12-May-2018_14:28:07  Exp 7309  26      1    R     3   5    22
## 4079 12-May-2018_14:28:09  Exp 7309  26      1    R     3   5    23
## 4080 12-May-2018_14:28:12  Exp 7309  26      1    R     3   5    24
## 4081 12-May-2018_14:33:21  Exp 7309  26      1    R     1   1     1
## 4082 12-May-2018_14:33:23  Exp 7309  26      1    R     1   1     2
## 4083 12-May-2018_14:33:25  Exp 7309  26      1    R     1   1     3
## 4084 12-May-2018_14:33:28  Exp 7309  26      1    R     1   1     4
## 4085 12-May-2018_14:33:30  Exp 7309  26      1    R     1   1     5
## 4086 12-May-2018_14:33:32  Exp 7309  26      1    R     1   1     6
## 4087 12-May-2018_14:33:35  Exp 7309  26      1    R     1   1     7
## 4088 12-May-2018_14:33:37  Exp 7309  26      1    R     1   1     8
## 4089 12-May-2018_14:33:39  Exp 7309  26      1    R     1   1     9
## 4090 12-May-2018_14:33:41  Exp 7309  26      1    R     1   1    10
## 4091 12-May-2018_14:33:44  Exp 7309  26      1    R     1   1    11
## 4092 12-May-2018_14:33:46  Exp 7309  26      1    R     1   1    12
## 4093 12-May-2018_14:33:48  Exp 7309  26      1    R     1   1    13
## 4094 12-May-2018_14:33:50  Exp 7309  26      1    R     1   1    14
## 4095 12-May-2018_14:33:53  Exp 7309  26      1    R     1   1    15
## 4096 12-May-2018_14:33:55  Exp 7309  26      1    R     1   1    16
## 4097 12-May-2018_14:33:58  Exp 7309  26      1    R     1   1    17
## 4098 12-May-2018_14:34:00  Exp 7309  26      1    R     1   1    18
## 4099 12-May-2018_14:34:02  Exp 7309  26      1    R     1   1    19
## 4100 12-May-2018_14:34:04  Exp 7309  26      1    R     1   1    20
## 4101 12-May-2018_14:34:07  Exp 7309  26      1    R     1   1    21
## 4102 12-May-2018_14:34:09  Exp 7309  26      1    R     1   1    22
## 4103 12-May-2018_14:34:12  Exp 7309  26      1    R     1   1    23
## 4104 12-May-2018_14:34:14  Exp 7309  26      1    R     1   1    24
## 4105 12-May-2018_14:34:16  Exp 7309  26      1    R     1   2     1
## 4106 12-May-2018_14:34:18  Exp 7309  26      1    R     1   2     2
## 4107 12-May-2018_14:34:21  Exp 7309  26      1    R     1   2     3
## 4108 12-May-2018_14:34:23  Exp 7309  26      1    R     1   2     4
## 4109 12-May-2018_14:34:25  Exp 7309  26      1    R     1   2     5
## 4110 12-May-2018_14:34:27  Exp 7309  26      1    R     1   2     6
## 4111 12-May-2018_14:34:29  Exp 7309  26      1    R     1   2     7
## 4112 12-May-2018_14:34:32  Exp 7309  26      1    R     1   2     8
## 4113 12-May-2018_14:34:34  Exp 7309  26      1    R     1   2     9
## 4114 12-May-2018_14:34:36  Exp 7309  26      1    R     1   2    10
## 4115 12-May-2018_14:34:39  Exp 7309  26      1    R     1   2    11
## 4116 12-May-2018_14:34:41  Exp 7309  26      1    R     1   2    12
## 4117 12-May-2018_14:34:43  Exp 7309  26      1    R     1   2    13
## 4118 12-May-2018_14:34:45  Exp 7309  26      1    R     1   2    14
## 4119 12-May-2018_14:34:48  Exp 7309  26      1    R     1   2    15
## 4120 12-May-2018_14:34:50  Exp 7309  26      1    R     1   2    16
## 4121 12-May-2018_14:34:52  Exp 7309  26      1    R     1   2    17
## 4122 12-May-2018_14:34:55  Exp 7309  26      1    R     1   2    18
## 4123 12-May-2018_14:34:57  Exp 7309  26      1    R     1   2    19
## 4124 12-May-2018_14:34:59  Exp 7309  26      1    R     1   2    20
## 4125 12-May-2018_14:35:02  Exp 7309  26      1    R     1   2    21
## 4126 12-May-2018_14:35:04  Exp 7309  26      1    R     1   2    22
## 4127 12-May-2018_14:35:06  Exp 7309  26      1    R     1   2    23
## 4128 12-May-2018_14:35:09  Exp 7309  26      1    R     1   2    24
## 4129 12-May-2018_14:40:01  Exp 7309  26      1    R     1   1     1
## 4130 12-May-2018_14:40:04  Exp 7309  26      1    R     1   1     2
## 4131 12-May-2018_14:40:06  Exp 7309  26      1    R     1   1     3
## 4132 12-May-2018_14:40:09  Exp 7309  26      1    R     1   1     4
## 4133 12-May-2018_14:40:11  Exp 7309  26      1    R     1   1     5
## 4134 12-May-2018_14:40:13  Exp 7309  26      1    R     1   1     6
## 4135 12-May-2018_14:40:16  Exp 7309  26      1    R     1   1     7
## 4136 12-May-2018_14:40:18  Exp 7309  26      1    R     1   1     8
## 4137 12-May-2018_14:40:20  Exp 7309  26      1    R     1   1     9
## 4138 12-May-2018_14:40:23  Exp 7309  26      1    R     1   1    10
## 4139 12-May-2018_14:40:25  Exp 7309  26      1    R     1   1    11
## 4140 12-May-2018_14:40:27  Exp 7309  26      1    R     1   1    12
## 4141 12-May-2018_14:40:30  Exp 7309  26      1    R     1   1    13
## 4142 12-May-2018_14:40:32  Exp 7309  26      1    R     1   1    14
## 4143 12-May-2018_14:40:34  Exp 7309  26      1    R     1   1    15
## 4144 12-May-2018_14:40:37  Exp 7309  26      1    R     1   1    16
## 4145 12-May-2018_14:40:39  Exp 7309  26      1    R     1   1    17
## 4146 12-May-2018_14:40:41  Exp 7309  26      1    R     1   1    18
## 4147 12-May-2018_14:40:43  Exp 7309  26      1    R     1   1    19
## 4148 12-May-2018_14:40:46  Exp 7309  26      1    R     1   1    20
## 4149 12-May-2018_14:40:48  Exp 7309  26      1    R     1   1    21
## 4150 12-May-2018_14:40:50  Exp 7309  26      1    R     1   1    22
## 4151 12-May-2018_14:40:52  Exp 7309  26      1    R     1   1    23
## 4152 12-May-2018_14:40:54  Exp 7309  26      1    R     1   1    24
## 4153 12-May-2018_14:40:57  Exp 7309  26      1    R     1   2     1
## 4154 12-May-2018_14:40:59  Exp 7309  26      1    R     1   2     2
## 4155 12-May-2018_14:41:01  Exp 7309  26      1    R     1   2     3
## 4156 12-May-2018_14:41:04  Exp 7309  26      1    R     1   2     4
## 4157 12-May-2018_14:41:06  Exp 7309  26      1    R     1   2     5
## 4158 12-May-2018_14:41:08  Exp 7309  26      1    R     1   2     6
## 4159 12-May-2018_14:41:11  Exp 7309  26      1    R     1   2     7
## 4160 12-May-2018_14:41:13  Exp 7309  26      1    R     1   2     8
## 4161 12-May-2018_14:41:15  Exp 7309  26      1    R     1   2     9
## 4162 12-May-2018_14:41:18  Exp 7309  26      1    R     1   2    10
## 4163 12-May-2018_14:41:20  Exp 7309  26      1    R     1   2    11
## 4164 12-May-2018_14:41:22  Exp 7309  26      1    R     1   2    12
## 4165 12-May-2018_14:41:25  Exp 7309  26      1    R     1   2    13
## 4166 12-May-2018_14:41:27  Exp 7309  26      1    R     1   2    14
## 4167 12-May-2018_14:41:29  Exp 7309  26      1    R     1   2    15
## 4168 12-May-2018_14:41:31  Exp 7309  26      1    R     1   2    16
## 4169 12-May-2018_14:41:34  Exp 7309  26      1    R     1   2    17
## 4170 12-May-2018_14:41:36  Exp 7309  26      1    R     1   2    18
## 4171 12-May-2018_14:41:38  Exp 7309  26      1    R     1   2    19
## 4172 12-May-2018_14:41:41  Exp 7309  26      1    R     1   2    20
## 4173 12-May-2018_14:41:43  Exp 7309  26      1    R     1   2    21
## 4174 12-May-2018_14:41:45  Exp 7309  26      1    R     1   2    22
## 4175 12-May-2018_14:41:48  Exp 7309  26      1    R     1   2    23
## 4176 12-May-2018_14:41:50  Exp 7309  26      1    R     1   2    24
## 4177 12-May-2018_14:46:08  Exp 7309  26      1    R     1   1     1
## 4178 12-May-2018_14:46:10  Exp 7309  26      1    R     1   1     2
## 4179 12-May-2018_14:46:13  Exp 7309  26      1    R     1   1     3
## 4180 12-May-2018_14:46:15  Exp 7309  26      1    R     1   1     4
## 4181 12-May-2018_14:46:17  Exp 7309  26      1    R     1   1     5
## 4182 12-May-2018_14:46:20  Exp 7309  26      1    R     1   1     6
## 4183 12-May-2018_14:46:22  Exp 7309  26      1    R     1   1     7
## 4184 12-May-2018_14:46:24  Exp 7309  26      1    R     1   1     8
## 4185 12-May-2018_14:46:27  Exp 7309  26      1    R     1   1     9
## 4186 12-May-2018_14:46:29  Exp 7309  26      1    R     1   1    10
## 4187 12-May-2018_14:46:31  Exp 7309  26      1    R     1   1    11
## 4188 12-May-2018_14:46:33  Exp 7309  26      1    R     1   1    12
## 4189 12-May-2018_14:46:36  Exp 7309  26      1    R     1   1    13
## 4190 12-May-2018_14:46:38  Exp 7309  26      1    R     1   1    14
## 4191 12-May-2018_14:46:40  Exp 7309  26      1    R     1   1    15
## 4192 12-May-2018_14:46:43  Exp 7309  26      1    R     1   1    16
## 4193 12-May-2018_14:46:45  Exp 7309  26      1    R     1   1    17
## 4194 12-May-2018_14:46:47  Exp 7309  26      1    R     1   1    18
## 4195 12-May-2018_14:46:49  Exp 7309  26      1    R     1   1    19
## 4196 12-May-2018_14:46:51  Exp 7309  26      1    R     1   1    20
## 4197 12-May-2018_14:46:54  Exp 7309  26      1    R     1   1    21
## 4198 12-May-2018_14:46:56  Exp 7309  26      1    R     1   1    22
## 4199 12-May-2018_14:46:58  Exp 7309  26      1    R     1   1    23
## 4200 12-May-2018_14:47:01  Exp 7309  26      1    R     1   1    24
## 4201 12-May-2018_14:47:03  Exp 7309  26      1    R     1   2     1
## 4202 12-May-2018_14:47:05  Exp 7309  26      1    R     1   2     2
## 4203 12-May-2018_14:47:08  Exp 7309  26      1    R     1   2     3
## 4204 12-May-2018_14:47:10  Exp 7309  26      1    R     1   2     4
## 4205 12-May-2018_14:47:12  Exp 7309  26      1    R     1   2     5
## 4206 12-May-2018_14:47:14  Exp 7309  26      1    R     1   2     6
## 4207 12-May-2018_14:47:17  Exp 7309  26      1    R     1   2     7
## 4208 12-May-2018_14:47:19  Exp 7309  26      1    R     1   2     8
## 4209 12-May-2018_14:47:21  Exp 7309  26      1    R     1   2     9
## 4210 12-May-2018_14:47:24  Exp 7309  26      1    R     1   2    10
## 4211 12-May-2018_14:47:26  Exp 7309  26      1    R     1   2    11
## 4212 12-May-2018_14:47:28  Exp 7309  26      1    R     1   2    12
## 4213 12-May-2018_14:47:31  Exp 7309  26      1    R     1   2    13
## 4214 12-May-2018_14:47:33  Exp 7309  26      1    R     1   2    14
## 4215 12-May-2018_14:47:35  Exp 7309  26      1    R     1   2    15
## 4216 12-May-2018_14:47:37  Exp 7309  26      1    R     1   2    16
## 4217 12-May-2018_14:47:40  Exp 7309  26      1    R     1   2    17
## 4218 12-May-2018_14:47:42  Exp 7309  26      1    R     1   2    18
## 4219 12-May-2018_14:47:44  Exp 7309  26      1    R     1   2    19
## 4220 12-May-2018_14:47:46  Exp 7309  26      1    R     1   2    20
## 4221 12-May-2018_14:47:49  Exp 7309  26      1    R     1   2    21
## 4222 12-May-2018_14:47:51  Exp 7309  26      1    R     1   2    22
## 4223 12-May-2018_14:47:53  Exp 7309  26      1    R     1   2    23
## 4224 12-May-2018_14:47:55  Exp 7309  26      1    R     1   2    24
## 4225 12-May-2018_14:52:12  Exp 7309  26      1    R     1   1     1
## 4226 12-May-2018_14:52:14  Exp 7309  26      1    R     1   1     2
## 4227 12-May-2018_14:52:16  Exp 7309  26      1    R     1   1     3
## 4228 12-May-2018_14:52:19  Exp 7309  26      1    R     1   1     4
## 4229 12-May-2018_14:52:21  Exp 7309  26      1    R     1   1     5
## 4230 12-May-2018_14:52:24  Exp 7309  26      1    R     1   1     6
## 4231 12-May-2018_14:52:26  Exp 7309  26      1    R     1   1     7
## 4232 12-May-2018_14:52:28  Exp 7309  26      1    R     1   1     8
## 4233 12-May-2018_14:52:31  Exp 7309  26      1    R     1   1     9
## 4234 12-May-2018_14:52:33  Exp 7309  26      1    R     1   1    10
## 4235 12-May-2018_14:52:35  Exp 7309  26      1    R     1   1    11
## 4236 12-May-2018_14:52:38  Exp 7309  26      1    R     1   1    12
## 4237 12-May-2018_14:52:40  Exp 7309  26      1    R     1   1    13
## 4238 12-May-2018_14:52:42  Exp 7309  26      1    R     1   1    14
## 4239 12-May-2018_14:52:44  Exp 7309  26      1    R     1   1    15
## 4240 12-May-2018_14:52:47  Exp 7309  26      1    R     1   1    16
## 4241 12-May-2018_14:52:49  Exp 7309  26      1    R     1   1    17
## 4242 12-May-2018_14:52:51  Exp 7309  26      1    R     1   1    18
## 4243 12-May-2018_14:52:53  Exp 7309  26      1    R     1   1    19
## 4244 12-May-2018_14:52:56  Exp 7309  26      1    R     1   1    20
## 4245 12-May-2018_14:52:58  Exp 7309  26      1    R     1   1    21
## 4246 12-May-2018_14:53:00  Exp 7309  26      1    R     1   1    22
## 4247 12-May-2018_14:53:02  Exp 7309  26      1    R     1   1    23
## 4248 12-May-2018_14:53:04  Exp 7309  26      1    R     1   1    24
## 4249 12-May-2018_14:53:07  Exp 7309  26      1    R     1   2     1
## 4250 12-May-2018_14:53:09  Exp 7309  26      1    R     1   2     2
## 4251 12-May-2018_14:53:11  Exp 7309  26      1    R     1   2     3
## 4252 12-May-2018_14:53:14  Exp 7309  26      1    R     1   2     4
## 4253 12-May-2018_14:53:16  Exp 7309  26      1    R     1   2     5
## 4254 12-May-2018_14:53:19  Exp 7309  26      1    R     1   2     6
## 4255 12-May-2018_14:53:21  Exp 7309  26      1    R     1   2     7
## 4256 12-May-2018_14:53:23  Exp 7309  26      1    R     1   2     8
## 4257 12-May-2018_14:53:25  Exp 7309  26      1    R     1   2     9
## 4258 12-May-2018_14:53:28  Exp 7309  26      1    R     1   2    10
## 4259 12-May-2018_14:53:30  Exp 7309  26      1    R     1   2    11
## 4260 12-May-2018_14:53:33  Exp 7309  26      1    R     1   2    12
## 4261 12-May-2018_14:53:35  Exp 7309  26      1    R     1   2    13
## 4262 12-May-2018_14:53:37  Exp 7309  26      1    R     1   2    14
## 4263 12-May-2018_14:53:40  Exp 7309  26      1    R     1   2    15
## 4264 12-May-2018_14:53:42  Exp 7309  26      1    R     1   2    16
## 4265 12-May-2018_14:53:44  Exp 7309  26      1    R     1   2    17
## 4266 12-May-2018_14:53:46  Exp 7309  26      1    R     1   2    18
## 4267 12-May-2018_14:53:49  Exp 7309  26      1    R     1   2    19
## 4268 12-May-2018_14:53:51  Exp 7309  26      1    R     1   2    20
## 4269 12-May-2018_14:53:53  Exp 7309  26      1    R     1   2    21
## 4270 12-May-2018_14:53:56  Exp 7309  26      1    R     1   2    22
## 4271 12-May-2018_14:53:58  Exp 7309  26      1    R     1   2    23
## 4272 12-May-2018_14:54:00  Exp 7309  26      1    R     1   2    24
## 4273 12-May-2018_14:58:12  Exp 7309  26      1    R     1   1     1
## 4274 12-May-2018_14:58:14  Exp 7309  26      1    R     1   1     2
## 4275 12-May-2018_14:58:16  Exp 7309  26      1    R     1   1     3
## 4276 12-May-2018_14:58:19  Exp 7309  26      1    R     1   1     4
## 4277 12-May-2018_14:58:21  Exp 7309  26      1    R     1   1     5
## 4278 12-May-2018_14:58:23  Exp 7309  26      1    R     1   1     6
## 4279 12-May-2018_14:58:25  Exp 7309  26      1    R     1   1     7
## 4280 12-May-2018_14:58:28  Exp 7309  26      1    R     1   1     8
## 4281 12-May-2018_14:58:30  Exp 7309  26      1    R     1   1     9
## 4282 12-May-2018_14:58:32  Exp 7309  26      1    R     1   1    10
## 4283 12-May-2018_14:58:34  Exp 7309  26      1    R     1   1    11
## 4284 12-May-2018_14:58:37  Exp 7309  26      1    R     1   1    12
## 4285 12-May-2018_14:58:39  Exp 7309  26      1    R     1   1    13
## 4286 12-May-2018_14:58:41  Exp 7309  26      1    R     1   1    14
## 4287 12-May-2018_14:58:44  Exp 7309  26      1    R     1   1    15
## 4288 12-May-2018_14:58:46  Exp 7309  26      1    R     1   1    16
## 4289 12-May-2018_14:58:48  Exp 7309  26      1    R     1   1    17
## 4290 12-May-2018_14:58:51  Exp 7309  26      1    R     1   1    18
## 4291 12-May-2018_14:58:53  Exp 7309  26      1    R     1   1    19
## 4292 12-May-2018_14:58:55  Exp 7309  26      1    R     1   1    20
## 4293 12-May-2018_14:58:57  Exp 7309  26      1    R     1   1    21
## 4294 12-May-2018_14:58:59  Exp 7309  26      1    R     1   1    22
## 4295 12-May-2018_14:59:02  Exp 7309  26      1    R     1   1    23
## 4296 12-May-2018_14:59:04  Exp 7309  26      1    R     1   1    24
## 4297 12-May-2018_14:59:07  Exp 7309  26      1    R     1   2     1
## 4298 12-May-2018_14:59:09  Exp 7309  26      1    R     1   2     2
## 4299 12-May-2018_14:59:11  Exp 7309  26      1    R     1   2     3
## 4300 12-May-2018_14:59:13  Exp 7309  26      1    R     1   2     4
## 4301 12-May-2018_14:59:16  Exp 7309  26      1    R     1   2     5
## 4302 12-May-2018_14:59:18  Exp 7309  26      1    R     1   2     6
## 4303 12-May-2018_14:59:20  Exp 7309  26      1    R     1   2     7
## 4304 12-May-2018_14:59:23  Exp 7309  26      1    R     1   2     8
## 4305 12-May-2018_14:59:25  Exp 7309  26      1    R     1   2     9
## 4306 12-May-2018_14:59:28  Exp 7309  26      1    R     1   2    10
## 4307 12-May-2018_14:59:30  Exp 7309  26      1    R     1   2    11
## 4308 12-May-2018_14:59:32  Exp 7309  26      1    R     1   2    12
## 4309 12-May-2018_14:59:34  Exp 7309  26      1    R     1   2    13
## 4310 12-May-2018_14:59:37  Exp 7309  26      1    R     1   2    14
## 4311 12-May-2018_14:59:39  Exp 7309  26      1    R     1   2    15
## 4312 12-May-2018_14:59:41  Exp 7309  26      1    R     1   2    16
## 4313 12-May-2018_14:59:44  Exp 7309  26      1    R     1   2    17
## 4314 12-May-2018_14:59:46  Exp 7309  26      1    R     1   2    18
## 4315 12-May-2018_14:59:48  Exp 7309  26      1    R     1   2    19
## 4316 12-May-2018_14:59:51  Exp 7309  26      1    R     1   2    20
## 4317 12-May-2018_14:59:53  Exp 7309  26      1    R     1   2    21
## 4318 12-May-2018_14:59:55  Exp 7309  26      1    R     1   2    22
## 4319 12-May-2018_14:59:58  Exp 7309  26      1    R     1   2    23
## 4320 12-May-2018_15:00:00  Exp 7309  26      1    R     1   2    24
## 4321 12-May-2018_16:03:31  Exp 7310  21      2    R     1   1     1
## 4322 12-May-2018_16:03:33  Exp 7310  21      2    R     1   1     2
## 4323 12-May-2018_16:03:35  Exp 7310  21      2    R     1   1     3
## 4324 12-May-2018_16:03:37  Exp 7310  21      2    R     1   1     4
## 4325 12-May-2018_16:03:40  Exp 7310  21      2    R     1   1     5
## 4326 12-May-2018_16:03:42  Exp 7310  21      2    R     1   1     6
## 4327 12-May-2018_16:03:44  Exp 7310  21      2    R     1   1     7
## 4328 12-May-2018_16:03:46  Exp 7310  21      2    R     1   1     8
## 4329 12-May-2018_16:03:48  Exp 7310  21      2    R     1   1     9
## 4330 12-May-2018_16:03:50  Exp 7310  21      2    R     1   1    10
## 4331 12-May-2018_16:03:52  Exp 7310  21      2    R     1   1    11
## 4332 12-May-2018_16:03:54  Exp 7310  21      2    R     1   1    12
## 4333 12-May-2018_16:03:56  Exp 7310  21      2    R     1   1    13
## 4334 12-May-2018_16:03:58  Exp 7310  21      2    R     1   1    14
## 4335 12-May-2018_16:04:00  Exp 7310  21      2    R     1   1    15
## 4336 12-May-2018_16:04:03  Exp 7310  21      2    R     1   1    16
## 4337 12-May-2018_16:04:05  Exp 7310  21      2    R     1   1    17
## 4338 12-May-2018_16:04:07  Exp 7310  21      2    R     1   1    18
## 4339 12-May-2018_16:04:09  Exp 7310  21      2    R     1   1    19
## 4340 12-May-2018_16:04:11  Exp 7310  21      2    R     1   1    20
## 4341 12-May-2018_16:04:14  Exp 7310  21      2    R     1   1    21
## 4342 12-May-2018_16:04:16  Exp 7310  21      2    R     1   1    22
## 4343 12-May-2018_16:04:19  Exp 7310  21      2    R     1   1    23
## 4344 12-May-2018_16:04:22  Exp 7310  21      2    R     1   1    24
## 4345 12-May-2018_16:04:24  Exp 7310  21      2    R     1   2     1
## 4346 12-May-2018_16:04:26  Exp 7310  21      2    R     1   2     2
## 4347 12-May-2018_16:04:28  Exp 7310  21      2    R     1   2     3
## 4348 12-May-2018_16:04:31  Exp 7310  21      2    R     1   2     4
## 4349 12-May-2018_16:04:33  Exp 7310  21      2    R     1   2     5
## 4350 12-May-2018_16:04:35  Exp 7310  21      2    R     1   2     6
## 4351 12-May-2018_16:04:38  Exp 7310  21      2    R     1   2     7
## 4352 12-May-2018_16:04:40  Exp 7310  21      2    R     1   2     8
## 4353 12-May-2018_16:04:42  Exp 7310  21      2    R     1   2     9
## 4354 12-May-2018_16:04:44  Exp 7310  21      2    R     1   2    10
## 4355 12-May-2018_16:04:46  Exp 7310  21      2    R     1   2    11
## 4356 12-May-2018_16:04:48  Exp 7310  21      2    R     1   2    12
## 4357 12-May-2018_16:04:50  Exp 7310  21      2    R     1   2    13
## 4358 12-May-2018_16:04:52  Exp 7310  21      2    R     1   2    14
## 4359 12-May-2018_16:04:55  Exp 7310  21      2    R     1   2    15
## 4360 12-May-2018_16:04:57  Exp 7310  21      2    R     1   2    16
## 4361 12-May-2018_16:04:59  Exp 7310  21      2    R     1   2    17
## 4362 12-May-2018_16:05:01  Exp 7310  21      2    R     1   2    18
## 4363 12-May-2018_16:05:03  Exp 7310  21      2    R     1   2    19
## 4364 12-May-2018_16:05:05  Exp 7310  21      2    R     1   2    20
## 4365 12-May-2018_16:05:08  Exp 7310  21      2    R     1   2    21
## 4366 12-May-2018_16:05:10  Exp 7310  21      2    R     1   2    22
## 4367 12-May-2018_16:05:12  Exp 7310  21      2    R     1   2    23
## 4368 12-May-2018_16:05:14  Exp 7310  21      2    R     1   2    24
## 4369 12-May-2018_16:05:16  Exp 7310  21      2    R     1   3     1
## 4370 12-May-2018_16:05:19  Exp 7310  21      2    R     1   3     2
## 4371 12-May-2018_16:05:21  Exp 7310  21      2    R     1   3     3
## 4372 12-May-2018_16:05:23  Exp 7310  21      2    R     1   3     4
## 4373 12-May-2018_16:05:25  Exp 7310  21      2    R     1   3     5
## 4374 12-May-2018_16:05:28  Exp 7310  21      2    R     1   3     6
## 4375 12-May-2018_16:05:30  Exp 7310  21      2    R     1   3     7
## 4376 12-May-2018_16:05:32  Exp 7310  21      2    R     1   3     8
## 4377 12-May-2018_16:05:34  Exp 7310  21      2    R     1   3     9
## 4378 12-May-2018_16:05:37  Exp 7310  21      2    R     1   3    10
## 4379 12-May-2018_16:05:39  Exp 7310  21      2    R     1   3    11
## 4380 12-May-2018_16:05:41  Exp 7310  21      2    R     1   3    12
## 4381 12-May-2018_16:05:43  Exp 7310  21      2    R     1   3    13
## 4382 12-May-2018_16:05:45  Exp 7310  21      2    R     1   3    14
## 4383 12-May-2018_16:05:47  Exp 7310  21      2    R     1   3    15
## 4384 12-May-2018_16:05:49  Exp 7310  21      2    R     1   3    16
## 4385 12-May-2018_16:05:52  Exp 7310  21      2    R     1   3    17
## 4386 12-May-2018_16:05:54  Exp 7310  21      2    R     1   3    18
## 4387 12-May-2018_16:05:56  Exp 7310  21      2    R     1   3    19
## 4388 12-May-2018_16:05:58  Exp 7310  21      2    R     1   3    20
## 4389 12-May-2018_16:06:00  Exp 7310  21      2    R     1   3    21
## 4390 12-May-2018_16:06:02  Exp 7310  21      2    R     1   3    22
## 4391 12-May-2018_16:06:04  Exp 7310  21      2    R     1   3    23
## 4392 12-May-2018_16:06:06  Exp 7310  21      2    R     1   3    24
## 4393 12-May-2018_16:06:29  Exp 7310  21      2    R     1   4     1
## 4394 12-May-2018_16:06:31  Exp 7310  21      2    R     1   4     2
## 4395 12-May-2018_16:06:33  Exp 7310  21      2    R     1   4     3
## 4396 12-May-2018_16:06:35  Exp 7310  21      2    R     1   4     4
## 4397 12-May-2018_16:06:37  Exp 7310  21      2    R     1   4     5
## 4398 12-May-2018_16:06:40  Exp 7310  21      2    R     1   4     6
## 4399 12-May-2018_16:06:42  Exp 7310  21      2    R     1   4     7
## 4400 12-May-2018_16:06:44  Exp 7310  21      2    R     1   4     8
## 4401 12-May-2018_16:06:46  Exp 7310  21      2    R     1   4     9
## 4402 12-May-2018_16:06:48  Exp 7310  21      2    R     1   4    10
## 4403 12-May-2018_16:06:50  Exp 7310  21      2    R     1   4    11
## 4404 12-May-2018_16:06:53  Exp 7310  21      2    R     1   4    12
## 4405 12-May-2018_16:06:55  Exp 7310  21      2    R     1   4    13
## 4406 12-May-2018_16:06:57  Exp 7310  21      2    R     1   4    14
## 4407 12-May-2018_16:06:59  Exp 7310  21      2    R     1   4    15
## 4408 12-May-2018_16:07:01  Exp 7310  21      2    R     1   4    16
## 4409 12-May-2018_16:07:03  Exp 7310  21      2    R     1   4    17
## 4410 12-May-2018_16:07:05  Exp 7310  21      2    R     1   4    18
## 4411 12-May-2018_16:07:07  Exp 7310  21      2    R     1   4    19
## 4412 12-May-2018_16:07:09  Exp 7310  21      2    R     1   4    20
## 4413 12-May-2018_16:07:11  Exp 7310  21      2    R     1   4    21
## 4414 12-May-2018_16:07:13  Exp 7310  21      2    R     1   4    22
## 4415 12-May-2018_16:07:16  Exp 7310  21      2    R     1   4    23
## 4416 12-May-2018_16:07:18  Exp 7310  21      2    R     1   4    24
## 4417 12-May-2018_16:07:20  Exp 7310  21      2    R     1   5     1
## 4418 12-May-2018_16:07:22  Exp 7310  21      2    R     1   5     2
## 4419 12-May-2018_16:07:24  Exp 7310  21      2    R     1   5     3
## 4420 12-May-2018_16:07:26  Exp 7310  21      2    R     1   5     4
## 4421 12-May-2018_16:07:28  Exp 7310  21      2    R     1   5     5
## 4422 12-May-2018_16:07:30  Exp 7310  21      2    R     1   5     6
## 4423 12-May-2018_16:07:32  Exp 7310  21      2    R     1   5     7
## 4424 12-May-2018_16:07:35  Exp 7310  21      2    R     1   5     8
## 4425 12-May-2018_16:07:37  Exp 7310  21      2    R     1   5     9
## 4426 12-May-2018_16:07:39  Exp 7310  21      2    R     1   5    10
## 4427 12-May-2018_16:07:41  Exp 7310  21      2    R     1   5    11
## 4428 12-May-2018_16:07:43  Exp 7310  21      2    R     1   5    12
## 4429 12-May-2018_16:07:45  Exp 7310  21      2    R     1   5    13
## 4430 12-May-2018_16:07:48  Exp 7310  21      2    R     1   5    14
## 4431 12-May-2018_16:07:50  Exp 7310  21      2    R     1   5    15
## 4432 12-May-2018_16:07:52  Exp 7310  21      2    R     1   5    16
## 4433 12-May-2018_16:07:54  Exp 7310  21      2    R     1   5    17
## 4434 12-May-2018_16:07:56  Exp 7310  21      2    R     1   5    18
## 4435 12-May-2018_16:07:58  Exp 7310  21      2    R     1   5    19
## 4436 12-May-2018_16:08:00  Exp 7310  21      2    R     1   5    20
## 4437 12-May-2018_16:08:03  Exp 7310  21      2    R     1   5    21
## 4438 12-May-2018_16:08:05  Exp 7310  21      2    R     1   5    22
## 4439 12-May-2018_16:08:07  Exp 7310  21      2    R     1   5    23
## 4440 12-May-2018_16:08:09  Exp 7310  21      2    R     1   5    24
## 4441 12-May-2018_16:08:15  Exp 7310  21      2    R     2   1     1
## 4442 12-May-2018_16:08:17  Exp 7310  21      2    R     2   1     2
## 4443 12-May-2018_16:08:19  Exp 7310  21      2    R     2   1     3
## 4444 12-May-2018_16:08:21  Exp 7310  21      2    R     2   1     4
## 4445 12-May-2018_16:08:23  Exp 7310  21      2    R     2   1     5
## 4446 12-May-2018_16:08:26  Exp 7310  21      2    R     2   1     6
## 4447 12-May-2018_16:08:28  Exp 7310  21      2    R     2   1     7
## 4448 12-May-2018_16:08:30  Exp 7310  21      2    R     2   1     8
## 4449 12-May-2018_16:08:32  Exp 7310  21      2    R     2   1     9
## 4450 12-May-2018_16:08:34  Exp 7310  21      2    R     2   1    10
## 4451 12-May-2018_16:08:37  Exp 7310  21      2    R     2   1    11
## 4452 12-May-2018_16:08:39  Exp 7310  21      2    R     2   1    12
## 4453 12-May-2018_16:08:41  Exp 7310  21      2    R     2   1    13
## 4454 12-May-2018_16:08:43  Exp 7310  21      2    R     2   1    14
## 4455 12-May-2018_16:08:45  Exp 7310  21      2    R     2   1    15
## 4456 12-May-2018_16:08:47  Exp 7310  21      2    R     2   1    16
## 4457 12-May-2018_16:08:49  Exp 7310  21      2    R     2   1    17
## 4458 12-May-2018_16:08:52  Exp 7310  21      2    R     2   1    18
## 4459 12-May-2018_16:08:54  Exp 7310  21      2    R     2   1    19
## 4460 12-May-2018_16:08:56  Exp 7310  21      2    R     2   1    20
## 4461 12-May-2018_16:08:58  Exp 7310  21      2    R     2   1    21
## 4462 12-May-2018_16:09:00  Exp 7310  21      2    R     2   1    22
## 4463 12-May-2018_16:09:02  Exp 7310  21      2    R     2   1    23
## 4464 12-May-2018_16:09:04  Exp 7310  21      2    R     2   1    24
## 4465 12-May-2018_16:09:06  Exp 7310  21      2    R     2   2     1
## 4466 12-May-2018_16:09:08  Exp 7310  21      2    R     2   2     2
## 4467 12-May-2018_16:09:11  Exp 7310  21      2    R     2   2     3
## 4468 12-May-2018_16:09:13  Exp 7310  21      2    R     2   2     4
## 4469 12-May-2018_16:09:15  Exp 7310  21      2    R     2   2     5
## 4470 12-May-2018_16:09:17  Exp 7310  21      2    R     2   2     6
## 4471 12-May-2018_16:09:19  Exp 7310  21      2    R     2   2     7
## 4472 12-May-2018_16:09:21  Exp 7310  21      2    R     2   2     8
## 4473 12-May-2018_16:09:24  Exp 7310  21      2    R     2   2     9
## 4474 12-May-2018_16:09:26  Exp 7310  21      2    R     2   2    10
## 4475 12-May-2018_16:09:28  Exp 7310  21      2    R     2   2    11
## 4476 12-May-2018_16:09:30  Exp 7310  21      2    R     2   2    12
## 4477 12-May-2018_16:09:32  Exp 7310  21      2    R     2   2    13
## 4478 12-May-2018_16:09:34  Exp 7310  21      2    R     2   2    14
## 4479 12-May-2018_16:09:36  Exp 7310  21      2    R     2   2    15
## 4480 12-May-2018_16:09:38  Exp 7310  21      2    R     2   2    16
## 4481 12-May-2018_16:09:40  Exp 7310  21      2    R     2   2    17
## 4482 12-May-2018_16:09:43  Exp 7310  21      2    R     2   2    18
## 4483 12-May-2018_16:09:45  Exp 7310  21      2    R     2   2    19
## 4484 12-May-2018_16:09:47  Exp 7310  21      2    R     2   2    20
## 4485 12-May-2018_16:09:49  Exp 7310  21      2    R     2   2    21
## 4486 12-May-2018_16:09:51  Exp 7310  21      2    R     2   2    22
## 4487 12-May-2018_16:09:53  Exp 7310  21      2    R     2   2    23
## 4488 12-May-2018_16:09:56  Exp 7310  21      2    R     2   2    24
## 4489 12-May-2018_16:09:58  Exp 7310  21      2    R     2   3     1
## 4490 12-May-2018_16:10:00  Exp 7310  21      2    R     2   3     2
## 4491 12-May-2018_16:10:02  Exp 7310  21      2    R     2   3     3
## 4492 12-May-2018_16:10:04  Exp 7310  21      2    R     2   3     4
## 4493 12-May-2018_16:10:06  Exp 7310  21      2    R     2   3     5
## 4494 12-May-2018_16:10:08  Exp 7310  21      2    R     2   3     6
## 4495 12-May-2018_16:10:11  Exp 7310  21      2    R     2   3     7
## 4496 12-May-2018_16:10:13  Exp 7310  21      2    R     2   3     8
## 4497 12-May-2018_16:10:15  Exp 7310  21      2    R     2   3     9
## 4498 12-May-2018_16:10:17  Exp 7310  21      2    R     2   3    10
## 4499 12-May-2018_16:10:19  Exp 7310  21      2    R     2   3    11
## 4500 12-May-2018_16:10:21  Exp 7310  21      2    R     2   3    12
## 4501 12-May-2018_16:10:23  Exp 7310  21      2    R     2   3    13
## 4502 12-May-2018_16:10:25  Exp 7310  21      2    R     2   3    14
## 4503 12-May-2018_16:10:28  Exp 7310  21      2    R     2   3    15
## 4504 12-May-2018_16:10:30  Exp 7310  21      2    R     2   3    16
## 4505 12-May-2018_16:10:32  Exp 7310  21      2    R     2   3    17
## 4506 12-May-2018_16:10:34  Exp 7310  21      2    R     2   3    18
## 4507 12-May-2018_16:10:36  Exp 7310  21      2    R     2   3    19
## 4508 12-May-2018_16:10:38  Exp 7310  21      2    R     2   3    20
## 4509 12-May-2018_16:10:40  Exp 7310  21      2    R     2   3    21
## 4510 12-May-2018_16:10:43  Exp 7310  21      2    R     2   3    22
## 4511 12-May-2018_16:10:45  Exp 7310  21      2    R     2   3    23
## 4512 12-May-2018_16:10:47  Exp 7310  21      2    R     2   3    24
## 4513 12-May-2018_16:11:06  Exp 7310  21      2    R     2   4     1
## 4514 12-May-2018_16:11:08  Exp 7310  21      2    R     2   4     2
## 4515 12-May-2018_16:11:10  Exp 7310  21      2    R     2   4     3
## 4516 12-May-2018_16:11:13  Exp 7310  21      2    R     2   4     4
## 4517 12-May-2018_16:11:15  Exp 7310  21      2    R     2   4     5
## 4518 12-May-2018_16:11:17  Exp 7310  21      2    R     2   4     6
## 4519 12-May-2018_16:11:19  Exp 7310  21      2    R     2   4     7
## 4520 12-May-2018_16:11:21  Exp 7310  21      2    R     2   4     8
## 4521 12-May-2018_16:11:24  Exp 7310  21      2    R     2   4     9
## 4522 12-May-2018_16:11:26  Exp 7310  21      2    R     2   4    10
## 4523 12-May-2018_16:11:28  Exp 7310  21      2    R     2   4    11
## 4524 12-May-2018_16:11:30  Exp 7310  21      2    R     2   4    12
## 4525 12-May-2018_16:11:32  Exp 7310  21      2    R     2   4    13
## 4526 12-May-2018_16:11:34  Exp 7310  21      2    R     2   4    14
## 4527 12-May-2018_16:11:36  Exp 7310  21      2    R     2   4    15
## 4528 12-May-2018_16:11:39  Exp 7310  21      2    R     2   4    16
## 4529 12-May-2018_16:11:41  Exp 7310  21      2    R     2   4    17
## 4530 12-May-2018_16:11:43  Exp 7310  21      2    R     2   4    18
## 4531 12-May-2018_16:11:45  Exp 7310  21      2    R     2   4    19
## 4532 12-May-2018_16:11:47  Exp 7310  21      2    R     2   4    20
## 4533 12-May-2018_16:11:49  Exp 7310  21      2    R     2   4    21
## 4534 12-May-2018_16:11:51  Exp 7310  21      2    R     2   4    22
## 4535 12-May-2018_16:11:53  Exp 7310  21      2    R     2   4    23
## 4536 12-May-2018_16:11:55  Exp 7310  21      2    R     2   4    24
## 4537 12-May-2018_16:11:57  Exp 7310  21      2    R     2   5     1
## 4538 12-May-2018_16:12:00  Exp 7310  21      2    R     2   5     2
## 4539 12-May-2018_16:12:02  Exp 7310  21      2    R     2   5     3
## 4540 12-May-2018_16:12:04  Exp 7310  21      2    R     2   5     4
## 4541 12-May-2018_16:12:06  Exp 7310  21      2    R     2   5     5
## 4542 12-May-2018_16:12:08  Exp 7310  21      2    R     2   5     6
## 4543 12-May-2018_16:12:10  Exp 7310  21      2    R     2   5     7
## 4544 12-May-2018_16:12:13  Exp 7310  21      2    R     2   5     8
## 4545 12-May-2018_16:12:15  Exp 7310  21      2    R     2   5     9
## 4546 12-May-2018_16:12:17  Exp 7310  21      2    R     2   5    10
## 4547 12-May-2018_16:12:19  Exp 7310  21      2    R     2   5    11
## 4548 12-May-2018_16:12:21  Exp 7310  21      2    R     2   5    12
## 4549 12-May-2018_16:12:23  Exp 7310  21      2    R     2   5    13
## 4550 12-May-2018_16:12:25  Exp 7310  21      2    R     2   5    14
## 4551 12-May-2018_16:12:27  Exp 7310  21      2    R     2   5    15
## 4552 12-May-2018_16:12:29  Exp 7310  21      2    R     2   5    16
## 4553 12-May-2018_16:12:31  Exp 7310  21      2    R     2   5    17
## 4554 12-May-2018_16:12:34  Exp 7310  21      2    R     2   5    18
## 4555 12-May-2018_16:12:36  Exp 7310  21      2    R     2   5    19
## 4556 12-May-2018_16:12:38  Exp 7310  21      2    R     2   5    20
## 4557 12-May-2018_16:12:40  Exp 7310  21      2    R     2   5    21
## 4558 12-May-2018_16:12:42  Exp 7310  21      2    R     2   5    22
## 4559 12-May-2018_16:12:45  Exp 7310  21      2    R     2   5    23
## 4560 12-May-2018_16:12:47  Exp 7310  21      2    R     2   5    24
## 4561 12-May-2018_16:12:52  Exp 7310  21      2    R     3   1     1
## 4562 12-May-2018_16:12:54  Exp 7310  21      2    R     3   1     2
## 4563 12-May-2018_16:12:56  Exp 7310  21      2    R     3   1     3
## 4564 12-May-2018_16:12:58  Exp 7310  21      2    R     3   1     4
## 4565 12-May-2018_16:13:00  Exp 7310  21      2    R     3   1     5
## 4566 12-May-2018_16:13:02  Exp 7310  21      2    R     3   1     6
## 4567 12-May-2018_16:13:04  Exp 7310  21      2    R     3   1     7
## 4568 12-May-2018_16:13:06  Exp 7310  21      2    R     3   1     8
## 4569 12-May-2018_16:13:09  Exp 7310  21      2    R     3   1     9
## 4570 12-May-2018_16:13:11  Exp 7310  21      2    R     3   1    10
## 4571 12-May-2018_16:13:13  Exp 7310  21      2    R     3   1    11
## 4572 12-May-2018_16:13:15  Exp 7310  21      2    R     3   1    12
## 4573 12-May-2018_16:13:17  Exp 7310  21      2    R     3   1    13
## 4574 12-May-2018_16:13:19  Exp 7310  21      2    R     3   1    14
## 4575 12-May-2018_16:13:21  Exp 7310  21      2    R     3   1    15
## 4576 12-May-2018_16:13:23  Exp 7310  21      2    R     3   1    16
## 4577 12-May-2018_16:13:26  Exp 7310  21      2    R     3   1    17
## 4578 12-May-2018_16:13:28  Exp 7310  21      2    R     3   1    18
## 4579 12-May-2018_16:13:30  Exp 7310  21      2    R     3   1    19
## 4580 12-May-2018_16:13:32  Exp 7310  21      2    R     3   1    20
## 4581 12-May-2018_16:13:34  Exp 7310  21      2    R     3   1    21
## 4582 12-May-2018_16:13:36  Exp 7310  21      2    R     3   1    22
## 4583 12-May-2018_16:13:38  Exp 7310  21      2    R     3   1    23
## 4584 12-May-2018_16:13:40  Exp 7310  21      2    R     3   1    24
## 4585 12-May-2018_16:13:43  Exp 7310  21      2    R     3   2     1
## 4586 12-May-2018_16:13:45  Exp 7310  21      2    R     3   2     2
## 4587 12-May-2018_16:13:47  Exp 7310  21      2    R     3   2     3
## 4588 12-May-2018_16:13:49  Exp 7310  21      2    R     3   2     4
## 4589 12-May-2018_16:13:51  Exp 7310  21      2    R     3   2     5
## 4590 12-May-2018_16:13:53  Exp 7310  21      2    R     3   2     6
## 4591 12-May-2018_16:13:56  Exp 7310  21      2    R     3   2     7
## 4592 12-May-2018_16:13:58  Exp 7310  21      2    R     3   2     8
## 4593 12-May-2018_16:14:00  Exp 7310  21      2    R     3   2     9
## 4594 12-May-2018_16:14:02  Exp 7310  21      2    R     3   2    10
## 4595 12-May-2018_16:14:04  Exp 7310  21      2    R     3   2    11
## 4596 12-May-2018_16:14:06  Exp 7310  21      2    R     3   2    12
## 4597 12-May-2018_16:14:08  Exp 7310  21      2    R     3   2    13
## 4598 12-May-2018_16:14:10  Exp 7310  21      2    R     3   2    14
## 4599 12-May-2018_16:14:13  Exp 7310  21      2    R     3   2    15
## 4600 12-May-2018_16:14:15  Exp 7310  21      2    R     3   2    16
## 4601 12-May-2018_16:14:17  Exp 7310  21      2    R     3   2    17
## 4602 12-May-2018_16:14:20  Exp 7310  21      2    R     3   2    18
## 4603 12-May-2018_16:14:22  Exp 7310  21      2    R     3   2    19
## 4604 12-May-2018_16:14:24  Exp 7310  21      2    R     3   2    20
## 4605 12-May-2018_16:14:26  Exp 7310  21      2    R     3   2    21
## 4606 12-May-2018_16:14:28  Exp 7310  21      2    R     3   2    22
## 4607 12-May-2018_16:14:30  Exp 7310  21      2    R     3   2    23
## 4608 12-May-2018_16:14:33  Exp 7310  21      2    R     3   2    24
## 4609 12-May-2018_16:14:35  Exp 7310  21      2    R     3   3     1
## 4610 12-May-2018_16:14:37  Exp 7310  21      2    R     3   3     2
## 4611 12-May-2018_16:14:39  Exp 7310  21      2    R     3   3     3
## 4612 12-May-2018_16:14:42  Exp 7310  21      2    R     3   3     4
## 4613 12-May-2018_16:14:44  Exp 7310  21      2    R     3   3     5
## 4614 12-May-2018_16:14:46  Exp 7310  21      2    R     3   3     6
## 4615 12-May-2018_16:14:48  Exp 7310  21      2    R     3   3     7
## 4616 12-May-2018_16:14:50  Exp 7310  21      2    R     3   3     8
## 4617 12-May-2018_16:14:52  Exp 7310  21      2    R     3   3     9
## 4618 12-May-2018_16:14:54  Exp 7310  21      2    R     3   3    10
## 4619 12-May-2018_16:14:57  Exp 7310  21      2    R     3   3    11
## 4620 12-May-2018_16:14:59  Exp 7310  21      2    R     3   3    12
## 4621 12-May-2018_16:15:01  Exp 7310  21      2    R     3   3    13
## 4622 12-May-2018_16:15:03  Exp 7310  21      2    R     3   3    14
## 4623 12-May-2018_16:15:05  Exp 7310  21      2    R     3   3    15
## 4624 12-May-2018_16:15:07  Exp 7310  21      2    R     3   3    16
## 4625 12-May-2018_16:15:09  Exp 7310  21      2    R     3   3    17
## 4626 12-May-2018_16:15:11  Exp 7310  21      2    R     3   3    18
## 4627 12-May-2018_16:15:13  Exp 7310  21      2    R     3   3    19
## 4628 12-May-2018_16:15:16  Exp 7310  21      2    R     3   3    20
## 4629 12-May-2018_16:15:18  Exp 7310  21      2    R     3   3    21
## 4630 12-May-2018_16:15:20  Exp 7310  21      2    R     3   3    22
## 4631 12-May-2018_16:15:22  Exp 7310  21      2    R     3   3    23
## 4632 12-May-2018_16:15:24  Exp 7310  21      2    R     3   3    24
## 4633 12-May-2018_16:15:40  Exp 7310  21      2    R     3   4     1
## 4634 12-May-2018_16:15:42  Exp 7310  21      2    R     3   4     2
## 4635 12-May-2018_16:15:44  Exp 7310  21      2    R     3   4     3
## 4636 12-May-2018_16:15:47  Exp 7310  21      2    R     3   4     4
## 4637 12-May-2018_16:15:49  Exp 7310  21      2    R     3   4     5
## 4638 12-May-2018_16:15:51  Exp 7310  21      2    R     3   4     6
## 4639 12-May-2018_16:15:53  Exp 7310  21      2    R     3   4     7
## 4640 12-May-2018_16:15:55  Exp 7310  21      2    R     3   4     8
## 4641 12-May-2018_16:15:57  Exp 7310  21      2    R     3   4     9
## 4642 12-May-2018_16:16:00  Exp 7310  21      2    R     3   4    10
## 4643 12-May-2018_16:16:02  Exp 7310  21      2    R     3   4    11
## 4644 12-May-2018_16:16:04  Exp 7310  21      2    R     3   4    12
## 4645 12-May-2018_16:16:06  Exp 7310  21      2    R     3   4    13
## 4646 12-May-2018_16:16:08  Exp 7310  21      2    R     3   4    14
## 4647 12-May-2018_16:16:10  Exp 7310  21      2    R     3   4    15
## 4648 12-May-2018_16:16:12  Exp 7310  21      2    R     3   4    16
## 4649 12-May-2018_16:16:15  Exp 7310  21      2    R     3   4    17
## 4650 12-May-2018_16:16:16  Exp 7310  21      2    R     3   4    18
## 4651 12-May-2018_16:16:19  Exp 7310  21      2    R     3   4    19
## 4652 12-May-2018_16:16:21  Exp 7310  21      2    R     3   4    20
## 4653 12-May-2018_16:16:23  Exp 7310  21      2    R     3   4    21
## 4654 12-May-2018_16:16:25  Exp 7310  21      2    R     3   4    22
## 4655 12-May-2018_16:16:27  Exp 7310  21      2    R     3   4    23
## 4656 12-May-2018_16:16:29  Exp 7310  21      2    R     3   4    24
## 4657 12-May-2018_16:16:31  Exp 7310  21      2    R     3   5     1
## 4658 12-May-2018_16:16:33  Exp 7310  21      2    R     3   5     2
## 4659 12-May-2018_16:16:35  Exp 7310  21      2    R     3   5     3
## 4660 12-May-2018_16:16:38  Exp 7310  21      2    R     3   5     4
## 4661 12-May-2018_16:16:40  Exp 7310  21      2    R     3   5     5
## 4662 12-May-2018_16:16:42  Exp 7310  21      2    R     3   5     6
## 4663 12-May-2018_16:16:44  Exp 7310  21      2    R     3   5     7
## 4664 12-May-2018_16:16:46  Exp 7310  21      2    R     3   5     8
## 4665 12-May-2018_16:16:48  Exp 7310  21      2    R     3   5     9
## 4666 12-May-2018_16:16:50  Exp 7310  21      2    R     3   5    10
## 4667 12-May-2018_16:16:52  Exp 7310  21      2    R     3   5    11
## 4668 12-May-2018_16:16:54  Exp 7310  21      2    R     3   5    12
## 4669 12-May-2018_16:16:56  Exp 7310  21      2    R     3   5    13
## 4670 12-May-2018_16:16:59  Exp 7310  21      2    R     3   5    14
## 4671 12-May-2018_16:17:01  Exp 7310  21      2    R     3   5    15
## 4672 12-May-2018_16:17:03  Exp 7310  21      2    R     3   5    16
## 4673 12-May-2018_16:17:05  Exp 7310  21      2    R     3   5    17
## 4674 12-May-2018_16:17:07  Exp 7310  21      2    R     3   5    18
## 4675 12-May-2018_16:17:09  Exp 7310  21      2    R     3   5    19
## 4676 12-May-2018_16:17:11  Exp 7310  21      2    R     3   5    20
## 4677 12-May-2018_16:17:13  Exp 7310  21      2    R     3   5    21
## 4678 12-May-2018_16:17:15  Exp 7310  21      2    R     3   5    22
## 4679 12-May-2018_16:17:17  Exp 7310  21      2    R     3   5    23
## 4680 12-May-2018_16:17:20  Exp 7310  21      2    R     3   5    24
## 4681 12-May-2018_16:22:33  Exp 7310  21      2    R     1   1     1
## 4682 12-May-2018_16:22:35  Exp 7310  21      2    R     1   1     2
## 4683 12-May-2018_16:22:37  Exp 7310  21      2    R     1   1     3
## 4684 12-May-2018_16:22:40  Exp 7310  21      2    R     1   1     4
## 4685 12-May-2018_16:22:42  Exp 7310  21      2    R     1   1     5
## 4686 12-May-2018_16:22:44  Exp 7310  21      2    R     1   1     6
## 4687 12-May-2018_16:22:46  Exp 7310  21      2    R     1   1     7
## 4688 12-May-2018_16:22:48  Exp 7310  21      2    R     1   1     8
## 4689 12-May-2018_16:22:50  Exp 7310  21      2    R     1   1     9
## 4690 12-May-2018_16:22:53  Exp 7310  21      2    R     1   1    10
## 4691 12-May-2018_16:22:55  Exp 7310  21      2    R     1   1    11
## 4692 12-May-2018_16:22:57  Exp 7310  21      2    R     1   1    12
## 4693 12-May-2018_16:22:59  Exp 7310  21      2    R     1   1    13
## 4694 12-May-2018_16:23:01  Exp 7310  21      2    R     1   1    14
## 4695 12-May-2018_16:23:04  Exp 7310  21      2    R     1   1    15
## 4696 12-May-2018_16:23:06  Exp 7310  21      2    R     1   1    16
## 4697 12-May-2018_16:23:08  Exp 7310  21      2    R     1   1    17
## 4698 12-May-2018_16:23:10  Exp 7310  21      2    R     1   1    18
## 4699 12-May-2018_16:23:12  Exp 7310  21      2    R     1   1    19
## 4700 12-May-2018_16:23:14  Exp 7310  21      2    R     1   1    20
## 4701 12-May-2018_16:23:17  Exp 7310  21      2    R     1   1    21
## 4702 12-May-2018_16:23:19  Exp 7310  21      2    R     1   1    22
## 4703 12-May-2018_16:23:21  Exp 7310  21      2    R     1   1    23
## 4704 12-May-2018_16:23:23  Exp 7310  21      2    R     1   1    24
## 4705 12-May-2018_16:23:25  Exp 7310  21      2    R     1   2     1
## 4706 12-May-2018_16:23:28  Exp 7310  21      2    R     1   2     2
## 4707 12-May-2018_16:23:30  Exp 7310  21      2    R     1   2     3
## 4708 12-May-2018_16:23:32  Exp 7310  21      2    R     1   2     4
## 4709 12-May-2018_16:23:34  Exp 7310  21      2    R     1   2     5
## 4710 12-May-2018_16:23:36  Exp 7310  21      2    R     1   2     6
## 4711 12-May-2018_16:23:39  Exp 7310  21      2    R     1   2     7
## 4712 12-May-2018_16:23:41  Exp 7310  21      2    R     1   2     8
## 4713 12-May-2018_16:23:43  Exp 7310  21      2    R     1   2     9
## 4714 12-May-2018_16:23:45  Exp 7310  21      2    R     1   2    10
## 4715 12-May-2018_16:23:47  Exp 7310  21      2    R     1   2    11
## 4716 12-May-2018_16:23:49  Exp 7310  21      2    R     1   2    12
## 4717 12-May-2018_16:23:51  Exp 7310  21      2    R     1   2    13
## 4718 12-May-2018_16:23:54  Exp 7310  21      2    R     1   2    14
## 4719 12-May-2018_16:23:56  Exp 7310  21      2    R     1   2    15
## 4720 12-May-2018_16:23:58  Exp 7310  21      2    R     1   2    16
## 4721 12-May-2018_16:24:00  Exp 7310  21      2    R     1   2    17
## 4722 12-May-2018_16:24:02  Exp 7310  21      2    R     1   2    18
## 4723 12-May-2018_16:24:04  Exp 7310  21      2    R     1   2    19
## 4724 12-May-2018_16:24:06  Exp 7310  21      2    R     1   2    20
## 4725 12-May-2018_16:24:08  Exp 7310  21      2    R     1   2    21
## 4726 12-May-2018_16:24:10  Exp 7310  21      2    R     1   2    22
## 4727 12-May-2018_16:24:13  Exp 7310  21      2    R     1   2    23
## 4728 12-May-2018_16:24:15  Exp 7310  21      2    R     1   2    24
## 4729 12-May-2018_16:28:40  Exp 7310  21      2    R     1   1     1
## 4730 12-May-2018_16:28:43  Exp 7310  21      2    R     1   1     2
## 4731 12-May-2018_16:28:45  Exp 7310  21      2    R     1   1     3
## 4732 12-May-2018_16:28:47  Exp 7310  21      2    R     1   1     4
## 4733 12-May-2018_16:28:49  Exp 7310  21      2    R     1   1     5
## 4734 12-May-2018_16:28:51  Exp 7310  21      2    R     1   1     6
## 4735 12-May-2018_16:28:53  Exp 7310  21      2    R     1   1     7
## 4736 12-May-2018_16:28:55  Exp 7310  21      2    R     1   1     8
## 4737 12-May-2018_16:28:57  Exp 7310  21      2    R     1   1     9
## 4738 12-May-2018_16:28:59  Exp 7310  21      2    R     1   1    10
## 4739 12-May-2018_16:29:01  Exp 7310  21      2    R     1   1    11
## 4740 12-May-2018_16:29:04  Exp 7310  21      2    R     1   1    12
## 4741 12-May-2018_16:29:06  Exp 7310  21      2    R     1   1    13
## 4742 12-May-2018_16:29:08  Exp 7310  21      2    R     1   1    14
## 4743 12-May-2018_16:29:10  Exp 7310  21      2    R     1   1    15
## 4744 12-May-2018_16:29:12  Exp 7310  21      2    R     1   1    16
## 4745 12-May-2018_16:29:14  Exp 7310  21      2    R     1   1    17
## 4746 12-May-2018_16:29:16  Exp 7310  21      2    R     1   1    18
## 4747 12-May-2018_16:29:19  Exp 7310  21      2    R     1   1    19
## 4748 12-May-2018_16:29:21  Exp 7310  21      2    R     1   1    20
## 4749 12-May-2018_16:29:23  Exp 7310  21      2    R     1   1    21
## 4750 12-May-2018_16:29:25  Exp 7310  21      2    R     1   1    22
## 4751 12-May-2018_16:29:27  Exp 7310  21      2    R     1   1    23
## 4752 12-May-2018_16:29:29  Exp 7310  21      2    R     1   1    24
## 4753 12-May-2018_16:29:31  Exp 7310  21      2    R     1   2     1
## 4754 12-May-2018_16:29:34  Exp 7310  21      2    R     1   2     2
## 4755 12-May-2018_16:29:36  Exp 7310  21      2    R     1   2     3
## 4756 12-May-2018_16:29:38  Exp 7310  21      2    R     1   2     4
## 4757 12-May-2018_16:29:40  Exp 7310  21      2    R     1   2     5
## 4758 12-May-2018_16:29:43  Exp 7310  21      2    R     1   2     6
## 4759 12-May-2018_16:29:45  Exp 7310  21      2    R     1   2     7
## 4760 12-May-2018_16:29:47  Exp 7310  21      2    R     1   2     8
## 4761 12-May-2018_16:29:49  Exp 7310  21      2    R     1   2     9
## 4762 12-May-2018_16:29:51  Exp 7310  21      2    R     1   2    10
## 4763 12-May-2018_16:29:54  Exp 7310  21      2    R     1   2    11
## 4764 12-May-2018_16:29:56  Exp 7310  21      2    R     1   2    12
## 4765 12-May-2018_16:29:58  Exp 7310  21      2    R     1   2    13
## 4766 12-May-2018_16:30:00  Exp 7310  21      2    R     1   2    14
## 4767 12-May-2018_16:30:02  Exp 7310  21      2    R     1   2    15
## 4768 12-May-2018_16:30:05  Exp 7310  21      2    R     1   2    16
## 4769 12-May-2018_16:30:07  Exp 7310  21      2    R     1   2    17
## 4770 12-May-2018_16:30:09  Exp 7310  21      2    R     1   2    18
## 4771 12-May-2018_16:30:11  Exp 7310  21      2    R     1   2    19
## 4772 12-May-2018_16:30:13  Exp 7310  21      2    R     1   2    20
## 4773 12-May-2018_16:30:15  Exp 7310  21      2    R     1   2    21
## 4774 12-May-2018_16:30:17  Exp 7310  21      2    R     1   2    22
## 4775 12-May-2018_16:30:20  Exp 7310  21      2    R     1   2    23
## 4776 12-May-2018_16:30:22  Exp 7310  21      2    R     1   2    24
## 4777 12-May-2018_16:34:44  Exp 7310  21      2    R     1   1     1
## 4778 12-May-2018_16:34:46  Exp 7310  21      2    R     1   1     2
## 4779 12-May-2018_16:34:48  Exp 7310  21      2    R     1   1     3
## 4780 12-May-2018_16:34:51  Exp 7310  21      2    R     1   1     4
## 4781 12-May-2018_16:34:52  Exp 7310  21      2    R     1   1     5
## 4782 12-May-2018_16:34:55  Exp 7310  21      2    R     1   1     6
## 4783 12-May-2018_16:34:57  Exp 7310  21      2    R     1   1     7
## 4784 12-May-2018_16:34:59  Exp 7310  21      2    R     1   1     8
## 4785 12-May-2018_16:35:01  Exp 7310  21      2    R     1   1     9
## 4786 12-May-2018_16:35:03  Exp 7310  21      2    R     1   1    10
## 4787 12-May-2018_16:35:05  Exp 7310  21      2    R     1   1    11
## 4788 12-May-2018_16:35:07  Exp 7310  21      2    R     1   1    12
## 4789 12-May-2018_16:35:10  Exp 7310  21      2    R     1   1    13
## 4790 12-May-2018_16:35:12  Exp 7310  21      2    R     1   1    14
## 4791 12-May-2018_16:35:14  Exp 7310  21      2    R     1   1    15
## 4792 12-May-2018_16:35:16  Exp 7310  21      2    R     1   1    16
## 4793 12-May-2018_16:35:18  Exp 7310  21      2    R     1   1    17
## 4794 12-May-2018_16:35:21  Exp 7310  21      2    R     1   1    18
## 4795 12-May-2018_16:35:23  Exp 7310  21      2    R     1   1    19
## 4796 12-May-2018_16:35:25  Exp 7310  21      2    R     1   1    20
## 4797 12-May-2018_16:35:27  Exp 7310  21      2    R     1   1    21
## 4798 12-May-2018_16:35:29  Exp 7310  21      2    R     1   1    22
## 4799 12-May-2018_16:35:31  Exp 7310  21      2    R     1   1    23
## 4800 12-May-2018_16:35:33  Exp 7310  21      2    R     1   1    24
## 4801 12-May-2018_16:35:35  Exp 7310  21      2    R     1   2     1
## 4802 12-May-2018_16:35:38  Exp 7310  21      2    R     1   2     2
## 4803 12-May-2018_16:35:40  Exp 7310  21      2    R     1   2     3
## 4804 12-May-2018_16:35:42  Exp 7310  21      2    R     1   2     4
## 4805 12-May-2018_16:35:44  Exp 7310  21      2    R     1   2     5
## 4806 12-May-2018_16:35:46  Exp 7310  21      2    R     1   2     6
## 4807 12-May-2018_16:35:48  Exp 7310  21      2    R     1   2     7
## 4808 12-May-2018_16:35:51  Exp 7310  21      2    R     1   2     8
## 4809 12-May-2018_16:35:53  Exp 7310  21      2    R     1   2     9
## 4810 12-May-2018_16:35:55  Exp 7310  21      2    R     1   2    10
## 4811 12-May-2018_16:35:57  Exp 7310  21      2    R     1   2    11
## 4812 12-May-2018_16:35:59  Exp 7310  21      2    R     1   2    12
## 4813 12-May-2018_16:36:01  Exp 7310  21      2    R     1   2    13
## 4814 12-May-2018_16:36:03  Exp 7310  21      2    R     1   2    14
## 4815 12-May-2018_16:36:05  Exp 7310  21      2    R     1   2    15
## 4816 12-May-2018_16:36:08  Exp 7310  21      2    R     1   2    16
## 4817 12-May-2018_16:36:10  Exp 7310  21      2    R     1   2    17
## 4818 12-May-2018_16:36:12  Exp 7310  21      2    R     1   2    18
## 4819 12-May-2018_16:36:14  Exp 7310  21      2    R     1   2    19
## 4820 12-May-2018_16:36:16  Exp 7310  21      2    R     1   2    20
## 4821 12-May-2018_16:36:18  Exp 7310  21      2    R     1   2    21
## 4822 12-May-2018_16:36:20  Exp 7310  21      2    R     1   2    22
## 4823 12-May-2018_16:36:22  Exp 7310  21      2    R     1   2    23
## 4824 12-May-2018_16:36:25  Exp 7310  21      2    R     1   2    24
## 4825 12-May-2018_16:40:38  Exp 7310  21      2    R     1   1     1
## 4826 12-May-2018_16:40:40  Exp 7310  21      2    R     1   1     2
## 4827 12-May-2018_16:40:42  Exp 7310  21      2    R     1   1     3
## 4828 12-May-2018_16:40:44  Exp 7310  21      2    R     1   1     4
## 4829 12-May-2018_16:40:46  Exp 7310  21      2    R     1   1     5
## 4830 12-May-2018_16:40:48  Exp 7310  21      2    R     1   1     6
## 4831 12-May-2018_16:40:50  Exp 7310  21      2    R     1   1     7
## 4832 12-May-2018_16:40:52  Exp 7310  21      2    R     1   1     8
## 4833 12-May-2018_16:40:55  Exp 7310  21      2    R     1   1     9
## 4834 12-May-2018_16:40:57  Exp 7310  21      2    R     1   1    10
## 4835 12-May-2018_16:40:59  Exp 7310  21      2    R     1   1    11
## 4836 12-May-2018_16:41:01  Exp 7310  21      2    R     1   1    12
## 4837 12-May-2018_16:41:03  Exp 7310  21      2    R     1   1    13
## 4838 12-May-2018_16:41:05  Exp 7310  21      2    R     1   1    14
## 4839 12-May-2018_16:41:07  Exp 7310  21      2    R     1   1    15
## 4840 12-May-2018_16:41:09  Exp 7310  21      2    R     1   1    16
## 4841 12-May-2018_16:41:12  Exp 7310  21      2    R     1   1    17
## 4842 12-May-2018_16:41:14  Exp 7310  21      2    R     1   1    18
## 4843 12-May-2018_16:41:16  Exp 7310  21      2    R     1   1    19
## 4844 12-May-2018_16:41:18  Exp 7310  21      2    R     1   1    20
## 4845 12-May-2018_16:41:20  Exp 7310  21      2    R     1   1    21
## 4846 12-May-2018_16:41:22  Exp 7310  21      2    R     1   1    22
## 4847 12-May-2018_16:41:25  Exp 7310  21      2    R     1   1    23
## 4848 12-May-2018_16:41:27  Exp 7310  21      2    R     1   1    24
## 4849 12-May-2018_16:41:29  Exp 7310  21      2    R     1   2     1
## 4850 12-May-2018_16:41:31  Exp 7310  21      2    R     1   2     2
## 4851 12-May-2018_16:41:33  Exp 7310  21      2    R     1   2     3
## 4852 12-May-2018_16:41:35  Exp 7310  21      2    R     1   2     4
## 4853 12-May-2018_16:41:37  Exp 7310  21      2    R     1   2     5
## 4854 12-May-2018_16:41:40  Exp 7310  21      2    R     1   2     6
## 4855 12-May-2018_16:41:42  Exp 7310  21      2    R     1   2     7
## 4856 12-May-2018_16:41:44  Exp 7310  21      2    R     1   2     8
## 4857 12-May-2018_16:41:46  Exp 7310  21      2    R     1   2     9
## 4858 12-May-2018_16:41:49  Exp 7310  21      2    R     1   2    10
## 4859 12-May-2018_16:41:51  Exp 7310  21      2    R     1   2    11
## 4860 12-May-2018_16:41:53  Exp 7310  21      2    R     1   2    12
## 4861 12-May-2018_16:41:55  Exp 7310  21      2    R     1   2    13
## 4862 12-May-2018_16:41:58  Exp 7310  21      2    R     1   2    14
## 4863 12-May-2018_16:42:00  Exp 7310  21      2    R     1   2    15
## 4864 12-May-2018_16:42:02  Exp 7310  21      2    R     1   2    16
## 4865 12-May-2018_16:42:04  Exp 7310  21      2    R     1   2    17
## 4866 12-May-2018_16:42:07  Exp 7310  21      2    R     1   2    18
## 4867 12-May-2018_16:42:09  Exp 7310  21      2    R     1   2    19
## 4868 12-May-2018_16:42:11  Exp 7310  21      2    R     1   2    20
## 4869 12-May-2018_16:42:13  Exp 7310  21      2    R     1   2    21
## 4870 12-May-2018_16:42:15  Exp 7310  21      2    R     1   2    22
## 4871 12-May-2018_16:42:17  Exp 7310  21      2    R     1   2    23
## 4872 12-May-2018_16:42:19  Exp 7310  21      2    R     1   2    24
## 4873 12-May-2018_16:46:27  Exp 7310  21      2    R     1   1     1
## 4874 12-May-2018_16:46:29  Exp 7310  21      2    R     1   1     2
## 4875 12-May-2018_16:46:31  Exp 7310  21      2    R     1   1     3
## 4876 12-May-2018_16:46:34  Exp 7310  21      2    R     1   1     4
## 4877 12-May-2018_16:46:36  Exp 7310  21      2    R     1   1     5
## 4878 12-May-2018_16:46:38  Exp 7310  21      2    R     1   1     6
## 4879 12-May-2018_16:46:40  Exp 7310  21      2    R     1   1     7
## 4880 12-May-2018_16:46:42  Exp 7310  21      2    R     1   1     8
## 4881 12-May-2018_16:46:44  Exp 7310  21      2    R     1   1     9
## 4882 12-May-2018_16:46:46  Exp 7310  21      2    R     1   1    10
## 4883 12-May-2018_16:46:48  Exp 7310  21      2    R     1   1    11
## 4884 12-May-2018_16:46:50  Exp 7310  21      2    R     1   1    12
## 4885 12-May-2018_16:46:53  Exp 7310  21      2    R     1   1    13
## 4886 12-May-2018_16:46:55  Exp 7310  21      2    R     1   1    14
## 4887 12-May-2018_16:46:57  Exp 7310  21      2    R     1   1    15
## 4888 12-May-2018_16:46:59  Exp 7310  21      2    R     1   1    16
## 4889 12-May-2018_16:47:01  Exp 7310  21      2    R     1   1    17
## 4890 12-May-2018_16:47:03  Exp 7310  21      2    R     1   1    18
## 4891 12-May-2018_16:47:05  Exp 7310  21      2    R     1   1    19
## 4892 12-May-2018_16:47:08  Exp 7310  21      2    R     1   1    20
## 4893 12-May-2018_16:47:10  Exp 7310  21      2    R     1   1    21
## 4894 12-May-2018_16:47:12  Exp 7310  21      2    R     1   1    22
## 4895 12-May-2018_16:47:14  Exp 7310  21      2    R     1   1    23
## 4896 12-May-2018_16:47:17  Exp 7310  21      2    R     1   1    24
## 4897 12-May-2018_16:47:19  Exp 7310  21      2    R     1   2     1
## 4898 12-May-2018_16:47:21  Exp 7310  21      2    R     1   2     2
## 4899 12-May-2018_16:47:23  Exp 7310  21      2    R     1   2     3
## 4900 12-May-2018_16:47:25  Exp 7310  21      2    R     1   2     4
## 4901 12-May-2018_16:47:28  Exp 7310  21      2    R     1   2     5
## 4902 12-May-2018_16:47:30  Exp 7310  21      2    R     1   2     6
## 4903 12-May-2018_16:47:32  Exp 7310  21      2    R     1   2     7
## 4904 12-May-2018_16:47:34  Exp 7310  21      2    R     1   2     8
## 4905 12-May-2018_16:47:36  Exp 7310  21      2    R     1   2     9
## 4906 12-May-2018_16:47:38  Exp 7310  21      2    R     1   2    10
## 4907 12-May-2018_16:47:40  Exp 7310  21      2    R     1   2    11
## 4908 12-May-2018_16:47:43  Exp 7310  21      2    R     1   2    12
## 4909 12-May-2018_16:47:45  Exp 7310  21      2    R     1   2    13
## 4910 12-May-2018_16:47:47  Exp 7310  21      2    R     1   2    14
## 4911 12-May-2018_16:47:49  Exp 7310  21      2    R     1   2    15
## 4912 12-May-2018_16:47:51  Exp 7310  21      2    R     1   2    16
## 4913 12-May-2018_16:47:53  Exp 7310  21      2    R     1   2    17
## 4914 12-May-2018_16:47:55  Exp 7310  21      2    R     1   2    18
## 4915 12-May-2018_16:47:58  Exp 7310  21      2    R     1   2    19
## 4916 12-May-2018_16:48:00  Exp 7310  21      2    R     1   2    20
## 4917 12-May-2018_16:48:02  Exp 7310  21      2    R     1   2    21
## 4918 12-May-2018_16:48:04  Exp 7310  21      2    R     1   2    22
## 4919 12-May-2018_16:48:06  Exp 7310  21      2    R     1   2    23
## 4920 12-May-2018_16:48:08  Exp 7310  21      2    R     1   2    24
## 4921 12-May-2018_16:05:45  Exp 7311  20      2    R     1   1     1
## 4922 12-May-2018_16:05:47  Exp 7311  20      2    R     1   1     2
## 4923 12-May-2018_16:05:49  Exp 7311  20      2    R     1   1     3
## 4924 12-May-2018_16:05:51  Exp 7311  20      2    R     1   1     4
## 4925 12-May-2018_16:05:53  Exp 7311  20      2    R     1   1     5
## 4926 12-May-2018_16:05:55  Exp 7311  20      2    R     1   1     6
## 4927 12-May-2018_16:05:58  Exp 7311  20      2    R     1   1     7
## 4928 12-May-2018_16:06:00  Exp 7311  20      2    R     1   1     8
## 4929 12-May-2018_16:06:02  Exp 7311  20      2    R     1   1     9
## 4930 12-May-2018_16:06:04  Exp 7311  20      2    R     1   1    10
## 4931 12-May-2018_16:06:06  Exp 7311  20      2    R     1   1    11
## 4932 12-May-2018_16:06:08  Exp 7311  20      2    R     1   1    12
## 4933 12-May-2018_16:06:10  Exp 7311  20      2    R     1   1    13
## 4934 12-May-2018_16:06:12  Exp 7311  20      2    R     1   1    14
## 4935 12-May-2018_16:06:14  Exp 7311  20      2    R     1   1    15
## 4936 12-May-2018_16:06:17  Exp 7311  20      2    R     1   1    16
## 4937 12-May-2018_16:06:19  Exp 7311  20      2    R     1   1    17
## 4938 12-May-2018_16:06:21  Exp 7311  20      2    R     1   1    18
## 4939 12-May-2018_16:06:23  Exp 7311  20      2    R     1   1    19
## 4940 12-May-2018_16:06:25  Exp 7311  20      2    R     1   1    20
## 4941 12-May-2018_16:06:28  Exp 7311  20      2    R     1   1    21
## 4942 12-May-2018_16:06:30  Exp 7311  20      2    R     1   1    22
## 4943 12-May-2018_16:06:32  Exp 7311  20      2    R     1   1    23
## 4944 12-May-2018_16:06:34  Exp 7311  20      2    R     1   1    24
## 4945 12-May-2018_16:06:37  Exp 7311  20      2    R     1   2     1
## 4946 12-May-2018_16:06:39  Exp 7311  20      2    R     1   2     2
## 4947 12-May-2018_16:06:41  Exp 7311  20      2    R     1   2     3
## 4948 12-May-2018_16:06:43  Exp 7311  20      2    R     1   2     4
## 4949 12-May-2018_16:06:45  Exp 7311  20      2    R     1   2     5
## 4950 12-May-2018_16:06:47  Exp 7311  20      2    R     1   2     6
## 4951 12-May-2018_16:06:49  Exp 7311  20      2    R     1   2     7
## 4952 12-May-2018_16:06:51  Exp 7311  20      2    R     1   2     8
## 4953 12-May-2018_16:06:54  Exp 7311  20      2    R     1   2     9
## 4954 12-May-2018_16:06:56  Exp 7311  20      2    R     1   2    10
## 4955 12-May-2018_16:06:58  Exp 7311  20      2    R     1   2    11
## 4956 12-May-2018_16:07:00  Exp 7311  20      2    R     1   2    12
## 4957 12-May-2018_16:07:02  Exp 7311  20      2    R     1   2    13
## 4958 12-May-2018_16:07:04  Exp 7311  20      2    R     1   2    14
## 4959 12-May-2018_16:07:06  Exp 7311  20      2    R     1   2    15
## 4960 12-May-2018_16:07:09  Exp 7311  20      2    R     1   2    16
## 4961 12-May-2018_16:07:11  Exp 7311  20      2    R     1   2    17
## 4962 12-May-2018_16:07:13  Exp 7311  20      2    R     1   2    18
## 4963 12-May-2018_16:07:15  Exp 7311  20      2    R     1   2    19
## 4964 12-May-2018_16:07:17  Exp 7311  20      2    R     1   2    20
## 4965 12-May-2018_16:07:20  Exp 7311  20      2    R     1   2    21
## 4966 12-May-2018_16:07:22  Exp 7311  20      2    R     1   2    22
## 4967 12-May-2018_16:07:24  Exp 7311  20      2    R     1   2    23
## 4968 12-May-2018_16:07:26  Exp 7311  20      2    R     1   2    24
## 4969 12-May-2018_16:07:29  Exp 7311  20      2    R     1   3     1
## 4970 12-May-2018_16:07:31  Exp 7311  20      2    R     1   3     2
## 4971 12-May-2018_16:07:33  Exp 7311  20      2    R     1   3     3
## 4972 12-May-2018_16:07:35  Exp 7311  20      2    R     1   3     4
## 4973 12-May-2018_16:07:37  Exp 7311  20      2    R     1   3     5
## 4974 12-May-2018_16:07:39  Exp 7311  20      2    R     1   3     6
## 4975 12-May-2018_16:07:41  Exp 7311  20      2    R     1   3     7
## 4976 12-May-2018_16:07:44  Exp 7311  20      2    R     1   3     8
## 4977 12-May-2018_16:07:46  Exp 7311  20      2    R     1   3     9
## 4978 12-May-2018_16:07:48  Exp 7311  20      2    R     1   3    10
## 4979 12-May-2018_16:07:50  Exp 7311  20      2    R     1   3    11
## 4980 12-May-2018_16:07:52  Exp 7311  20      2    R     1   3    12
## 4981 12-May-2018_16:07:54  Exp 7311  20      2    R     1   3    13
## 4982 12-May-2018_16:07:56  Exp 7311  20      2    R     1   3    14
## 4983 12-May-2018_16:07:58  Exp 7311  20      2    R     1   3    15
## 4984 12-May-2018_16:08:01  Exp 7311  20      2    R     1   3    16
## 4985 12-May-2018_16:08:03  Exp 7311  20      2    R     1   3    17
## 4986 12-May-2018_16:08:05  Exp 7311  20      2    R     1   3    18
## 4987 12-May-2018_16:08:08  Exp 7311  20      2    R     1   3    19
## 4988 12-May-2018_16:08:10  Exp 7311  20      2    R     1   3    20
## 4989 12-May-2018_16:08:12  Exp 7311  20      2    R     1   3    21
## 4990 12-May-2018_16:08:14  Exp 7311  20      2    R     1   3    22
## 4991 12-May-2018_16:08:16  Exp 7311  20      2    R     1   3    23
## 4992 12-May-2018_16:08:18  Exp 7311  20      2    R     1   3    24
## 4993 12-May-2018_16:08:35  Exp 7311  20      2    R     1   4     1
## 4994 12-May-2018_16:08:37  Exp 7311  20      2    R     1   4     2
## 4995 12-May-2018_16:08:40  Exp 7311  20      2    R     1   4     3
## 4996 12-May-2018_16:08:42  Exp 7311  20      2    R     1   4     4
## 4997 12-May-2018_16:08:44  Exp 7311  20      2    R     1   4     5
## 4998 12-May-2018_16:08:46  Exp 7311  20      2    R     1   4     6
## 4999 12-May-2018_16:08:48  Exp 7311  20      2    R     1   4     7
## 5000 12-May-2018_16:08:50  Exp 7311  20      2    R     1   4     8
## 5001 12-May-2018_16:08:52  Exp 7311  20      2    R     1   4     9
## 5002 12-May-2018_16:08:54  Exp 7311  20      2    R     1   4    10
## 5003 12-May-2018_16:08:56  Exp 7311  20      2    R     1   4    11
## 5004 12-May-2018_16:08:59  Exp 7311  20      2    R     1   4    12
## 5005 12-May-2018_16:09:01  Exp 7311  20      2    R     1   4    13
## 5006 12-May-2018_16:09:03  Exp 7311  20      2    R     1   4    14
## 5007 12-May-2018_16:09:05  Exp 7311  20      2    R     1   4    15
## 5008 12-May-2018_16:09:07  Exp 7311  20      2    R     1   4    16
## 5009 12-May-2018_16:09:09  Exp 7311  20      2    R     1   4    17
## 5010 12-May-2018_16:09:11  Exp 7311  20      2    R     1   4    18
## 5011 12-May-2018_16:09:13  Exp 7311  20      2    R     1   4    19
## 5012 12-May-2018_16:09:15  Exp 7311  20      2    R     1   4    20
## 5013 12-May-2018_16:09:18  Exp 7311  20      2    R     1   4    21
## 5014 12-May-2018_16:09:20  Exp 7311  20      2    R     1   4    22
## 5015 12-May-2018_16:09:22  Exp 7311  20      2    R     1   4    23
## 5016 12-May-2018_16:09:24  Exp 7311  20      2    R     1   4    24
## 5017 12-May-2018_16:09:26  Exp 7311  20      2    R     1   5     1
## 5018 12-May-2018_16:09:28  Exp 7311  20      2    R     1   5     2
## 5019 12-May-2018_16:09:30  Exp 7311  20      2    R     1   5     3
## 5020 12-May-2018_16:09:33  Exp 7311  20      2    R     1   5     4
## 5021 12-May-2018_16:09:35  Exp 7311  20      2    R     1   5     5
## 5022 12-May-2018_16:09:37  Exp 7311  20      2    R     1   5     6
## 5023 12-May-2018_16:09:39  Exp 7311  20      2    R     1   5     7
## 5024 12-May-2018_16:09:42  Exp 7311  20      2    R     1   5     8
## 5025 12-May-2018_16:09:44  Exp 7311  20      2    R     1   5     9
## 5026 12-May-2018_16:09:46  Exp 7311  20      2    R     1   5    10
## 5027 12-May-2018_16:09:48  Exp 7311  20      2    R     1   5    11
## 5028 12-May-2018_16:09:51  Exp 7311  20      2    R     1   5    12
## 5029 12-May-2018_16:09:53  Exp 7311  20      2    R     1   5    13
## 5030 12-May-2018_16:09:55  Exp 7311  20      2    R     1   5    14
## 5031 12-May-2018_16:09:57  Exp 7311  20      2    R     1   5    15
## 5032 12-May-2018_16:09:59  Exp 7311  20      2    R     1   5    16
## 5033 12-May-2018_16:10:01  Exp 7311  20      2    R     1   5    17
## 5034 12-May-2018_16:10:03  Exp 7311  20      2    R     1   5    18
## 5035 12-May-2018_16:10:05  Exp 7311  20      2    R     1   5    19
## 5036 12-May-2018_16:10:08  Exp 7311  20      2    R     1   5    20
## 5037 12-May-2018_16:10:10  Exp 7311  20      2    R     1   5    21
## 5038 12-May-2018_16:10:12  Exp 7311  20      2    R     1   5    22
## 5039 12-May-2018_16:10:14  Exp 7311  20      2    R     1   5    23
## 5040 12-May-2018_16:10:16  Exp 7311  20      2    R     1   5    24
## 5041 12-May-2018_16:10:22  Exp 7311  20      2    R     2   1     1
## 5042 12-May-2018_16:10:24  Exp 7311  20      2    R     2   1     2
## 5043 12-May-2018_16:10:26  Exp 7311  20      2    R     2   1     3
## 5044 12-May-2018_16:10:28  Exp 7311  20      2    R     2   1     4
## 5045 12-May-2018_16:10:30  Exp 7311  20      2    R     2   1     5
## 5046 12-May-2018_16:10:33  Exp 7311  20      2    R     2   1     6
## 5047 12-May-2018_16:10:35  Exp 7311  20      2    R     2   1     7
## 5048 12-May-2018_16:10:37  Exp 7311  20      2    R     2   1     8
## 5049 12-May-2018_16:10:39  Exp 7311  20      2    R     2   1     9
## 5050 12-May-2018_16:10:41  Exp 7311  20      2    R     2   1    10
## 5051 12-May-2018_16:10:43  Exp 7311  20      2    R     2   1    11
## 5052 12-May-2018_16:10:45  Exp 7311  20      2    R     2   1    12
## 5053 12-May-2018_16:10:48  Exp 7311  20      2    R     2   1    13
## 5054 12-May-2018_16:10:50  Exp 7311  20      2    R     2   1    14
## 5055 12-May-2018_16:10:52  Exp 7311  20      2    R     2   1    15
## 5056 12-May-2018_16:10:54  Exp 7311  20      2    R     2   1    16
## 5057 12-May-2018_16:10:56  Exp 7311  20      2    R     2   1    17
## 5058 12-May-2018_16:10:59  Exp 7311  20      2    R     2   1    18
## 5059 12-May-2018_16:11:01  Exp 7311  20      2    R     2   1    19
## 5060 12-May-2018_16:11:03  Exp 7311  20      2    R     2   1    20
## 5061 12-May-2018_16:11:05  Exp 7311  20      2    R     2   1    21
## 5062 12-May-2018_16:11:07  Exp 7311  20      2    R     2   1    22
## 5063 12-May-2018_16:11:09  Exp 7311  20      2    R     2   1    23
## 5064 12-May-2018_16:11:11  Exp 7311  20      2    R     2   1    24
## 5065 12-May-2018_16:11:13  Exp 7311  20      2    R     2   2     1
## 5066 12-May-2018_16:11:15  Exp 7311  20      2    R     2   2     2
## 5067 12-May-2018_16:11:17  Exp 7311  20      2    R     2   2     3
## 5068 12-May-2018_16:11:19  Exp 7311  20      2    R     2   2     4
## 5069 12-May-2018_16:11:21  Exp 7311  20      2    R     2   2     5
## 5070 12-May-2018_16:11:23  Exp 7311  20      2    R     2   2     6
## 5071 12-May-2018_16:11:26  Exp 7311  20      2    R     2   2     7
## 5072 12-May-2018_16:11:28  Exp 7311  20      2    R     2   2     8
## 5073 12-May-2018_16:11:30  Exp 7311  20      2    R     2   2     9
## 5074 12-May-2018_16:11:32  Exp 7311  20      2    R     2   2    10
## 5075 12-May-2018_16:11:34  Exp 7311  20      2    R     2   2    11
## 5076 12-May-2018_16:11:36  Exp 7311  20      2    R     2   2    12
## 5077 12-May-2018_16:11:38  Exp 7311  20      2    R     2   2    13
## 5078 12-May-2018_16:11:40  Exp 7311  20      2    R     2   2    14
## 5079 12-May-2018_16:11:42  Exp 7311  20      2    R     2   2    15
## 5080 12-May-2018_16:11:44  Exp 7311  20      2    R     2   2    16
## 5081 12-May-2018_16:11:47  Exp 7311  20      2    R     2   2    17
## 5082 12-May-2018_16:11:49  Exp 7311  20      2    R     2   2    18
## 5083 12-May-2018_16:11:51  Exp 7311  20      2    R     2   2    19
## 5084 12-May-2018_16:11:53  Exp 7311  20      2    R     2   2    20
## 5085 12-May-2018_16:11:55  Exp 7311  20      2    R     2   2    21
## 5086 12-May-2018_16:11:57  Exp 7311  20      2    R     2   2    22
## 5087 12-May-2018_16:11:59  Exp 7311  20      2    R     2   2    23
## 5088 12-May-2018_16:12:02  Exp 7311  20      2    R     2   2    24
## 5089 12-May-2018_16:12:04  Exp 7311  20      2    R     2   3     1
## 5090 12-May-2018_16:12:06  Exp 7311  20      2    R     2   3     2
## 5091 12-May-2018_16:12:08  Exp 7311  20      2    R     2   3     3
## 5092 12-May-2018_16:12:10  Exp 7311  20      2    R     2   3     4
## 5093 12-May-2018_16:12:13  Exp 7311  20      2    R     2   3     5
## 5094 12-May-2018_16:12:15  Exp 7311  20      2    R     2   3     6
## 5095 12-May-2018_16:12:17  Exp 7311  20      2    R     2   3     7
## 5096 12-May-2018_16:12:19  Exp 7311  20      2    R     2   3     8
## 5097 12-May-2018_16:12:21  Exp 7311  20      2    R     2   3     9
## 5098 12-May-2018_16:12:23  Exp 7311  20      2    R     2   3    10
## 5099 12-May-2018_16:12:25  Exp 7311  20      2    R     2   3    11
## 5100 12-May-2018_16:12:27  Exp 7311  20      2    R     2   3    12
## 5101 12-May-2018_16:12:29  Exp 7311  20      2    R     2   3    13
## 5102 12-May-2018_16:12:31  Exp 7311  20      2    R     2   3    14
## 5103 12-May-2018_16:12:33  Exp 7311  20      2    R     2   3    15
## 5104 12-May-2018_16:12:36  Exp 7311  20      2    R     2   3    16
## 5105 12-May-2018_16:12:38  Exp 7311  20      2    R     2   3    17
## 5106 12-May-2018_16:12:40  Exp 7311  20      2    R     2   3    18
## 5107 12-May-2018_16:12:42  Exp 7311  20      2    R     2   3    19
## 5108 12-May-2018_16:12:44  Exp 7311  20      2    R     2   3    20
## 5109 12-May-2018_16:12:47  Exp 7311  20      2    R     2   3    21
## 5110 12-May-2018_16:12:49  Exp 7311  20      2    R     2   3    22
## 5111 12-May-2018_16:12:51  Exp 7311  20      2    R     2   3    23
## 5112 12-May-2018_16:12:53  Exp 7311  20      2    R     2   3    24
## 5113 12-May-2018_16:13:11  Exp 7311  20      2    R     2   4     1
## 5114 12-May-2018_16:13:13  Exp 7311  20      2    R     2   4     2
## 5115 12-May-2018_16:13:15  Exp 7311  20      2    R     2   4     3
## 5116 12-May-2018_16:13:17  Exp 7311  20      2    R     2   4     4
## 5117 12-May-2018_16:13:19  Exp 7311  20      2    R     2   4     5
## 5118 12-May-2018_16:13:21  Exp 7311  20      2    R     2   4     6
## 5119 12-May-2018_16:13:23  Exp 7311  20      2    R     2   4     7
## 5120 12-May-2018_16:13:25  Exp 7311  20      2    R     2   4     8
## 5121 12-May-2018_16:13:27  Exp 7311  20      2    R     2   4     9
## 5122 12-May-2018_16:13:30  Exp 7311  20      2    R     2   4    10
## 5123 12-May-2018_16:13:32  Exp 7311  20      2    R     2   4    11
## 5124 12-May-2018_16:13:34  Exp 7311  20      2    R     2   4    12
## 5125 12-May-2018_16:13:36  Exp 7311  20      2    R     2   4    13
## 5126 12-May-2018_16:13:38  Exp 7311  20      2    R     2   4    14
## 5127 12-May-2018_16:13:41  Exp 7311  20      2    R     2   4    15
## 5128 12-May-2018_16:13:43  Exp 7311  20      2    R     2   4    16
## 5129 12-May-2018_16:13:45  Exp 7311  20      2    R     2   4    17
## 5130 12-May-2018_16:13:47  Exp 7311  20      2    R     2   4    18
## 5131 12-May-2018_16:13:49  Exp 7311  20      2    R     2   4    19
## 5132 12-May-2018_16:13:51  Exp 7311  20      2    R     2   4    20
## 5133 12-May-2018_16:13:53  Exp 7311  20      2    R     2   4    21
## 5134 12-May-2018_16:13:56  Exp 7311  20      2    R     2   4    22
## 5135 12-May-2018_16:13:58  Exp 7311  20      2    R     2   4    23
## 5136 12-May-2018_16:14:00  Exp 7311  20      2    R     2   4    24
## 5137 12-May-2018_16:14:02  Exp 7311  20      2    R     2   5     1
## 5138 12-May-2018_16:14:04  Exp 7311  20      2    R     2   5     2
## 5139 12-May-2018_16:14:06  Exp 7311  20      2    R     2   5     3
## 5140 12-May-2018_16:14:08  Exp 7311  20      2    R     2   5     4
## 5141 12-May-2018_16:14:10  Exp 7311  20      2    R     2   5     5
## 5142 12-May-2018_16:14:12  Exp 7311  20      2    R     2   5     6
## 5143 12-May-2018_16:14:14  Exp 7311  20      2    R     2   5     7
## 5144 12-May-2018_16:14:17  Exp 7311  20      2    R     2   5     8
## 5145 12-May-2018_16:14:19  Exp 7311  20      2    R     2   5     9
## 5146 12-May-2018_16:14:21  Exp 7311  20      2    R     2   5    10
## 5147 12-May-2018_16:14:23  Exp 7311  20      2    R     2   5    11
## 5148 12-May-2018_16:14:26  Exp 7311  20      2    R     2   5    12
## 5149 12-May-2018_16:14:28  Exp 7311  20      2    R     2   5    13
## 5150 12-May-2018_16:14:30  Exp 7311  20      2    R     2   5    14
## 5151 12-May-2018_16:14:32  Exp 7311  20      2    R     2   5    15
## 5152 12-May-2018_16:14:34  Exp 7311  20      2    R     2   5    16
## 5153 12-May-2018_16:14:37  Exp 7311  20      2    R     2   5    17
## 5154 12-May-2018_16:14:39  Exp 7311  20      2    R     2   5    18
## 5155 12-May-2018_16:14:41  Exp 7311  20      2    R     2   5    19
## 5156 12-May-2018_16:14:43  Exp 7311  20      2    R     2   5    20
## 5157 12-May-2018_16:14:45  Exp 7311  20      2    R     2   5    21
## 5158 12-May-2018_16:14:47  Exp 7311  20      2    R     2   5    22
## 5159 12-May-2018_16:14:50  Exp 7311  20      2    R     2   5    23
## 5160 12-May-2018_16:14:52  Exp 7311  20      2    R     2   5    24
## 5161 12-May-2018_16:14:57  Exp 7311  20      2    R     3   1     1
## 5162 12-May-2018_16:14:59  Exp 7311  20      2    R     3   1     2
## 5163 12-May-2018_16:15:01  Exp 7311  20      2    R     3   1     3
## 5164 12-May-2018_16:15:04  Exp 7311  20      2    R     3   1     4
## 5165 12-May-2018_16:15:06  Exp 7311  20      2    R     3   1     5
## 5166 12-May-2018_16:15:08  Exp 7311  20      2    R     3   1     6
## 5167 12-May-2018_16:15:10  Exp 7311  20      2    R     3   1     7
## 5168 12-May-2018_16:15:12  Exp 7311  20      2    R     3   1     8
## 5169 12-May-2018_16:15:15  Exp 7311  20      2    R     3   1     9
## 5170 12-May-2018_16:15:17  Exp 7311  20      2    R     3   1    10
## 5171 12-May-2018_16:15:19  Exp 7311  20      2    R     3   1    11
## 5172 12-May-2018_16:15:21  Exp 7311  20      2    R     3   1    12
## 5173 12-May-2018_16:15:23  Exp 7311  20      2    R     3   1    13
## 5174 12-May-2018_16:15:26  Exp 7311  20      2    R     3   1    14
## 5175 12-May-2018_16:15:28  Exp 7311  20      2    R     3   1    15
## 5176 12-May-2018_16:15:30  Exp 7311  20      2    R     3   1    16
## 5177 12-May-2018_16:15:32  Exp 7311  20      2    R     3   1    17
## 5178 12-May-2018_16:15:35  Exp 7311  20      2    R     3   1    18
## 5179 12-May-2018_16:15:37  Exp 7311  20      2    R     3   1    19
## 5180 12-May-2018_16:15:39  Exp 7311  20      2    R     3   1    20
## 5181 12-May-2018_16:15:41  Exp 7311  20      2    R     3   1    21
## 5182 12-May-2018_16:15:43  Exp 7311  20      2    R     3   1    22
## 5183 12-May-2018_16:15:45  Exp 7311  20      2    R     3   1    23
## 5184 12-May-2018_16:15:48  Exp 7311  20      2    R     3   1    24
## 5185 12-May-2018_16:15:50  Exp 7311  20      2    R     3   2     1
## 5186 12-May-2018_16:15:52  Exp 7311  20      2    R     3   2     2
## 5187 12-May-2018_16:15:54  Exp 7311  20      2    R     3   2     3
## 5188 12-May-2018_16:15:56  Exp 7311  20      2    R     3   2     4
## 5189 12-May-2018_16:15:58  Exp 7311  20      2    R     3   2     5
## 5190 12-May-2018_16:16:00  Exp 7311  20      2    R     3   2     6
## 5191 12-May-2018_16:16:02  Exp 7311  20      2    R     3   2     7
## 5192 12-May-2018_16:16:05  Exp 7311  20      2    R     3   2     8
## 5193 12-May-2018_16:16:07  Exp 7311  20      2    R     3   2     9
## 5194 12-May-2018_16:16:09  Exp 7311  20      2    R     3   2    10
## 5195 12-May-2018_16:16:11  Exp 7311  20      2    R     3   2    11
## 5196 12-May-2018_16:16:13  Exp 7311  20      2    R     3   2    12
## 5197 12-May-2018_16:16:15  Exp 7311  20      2    R     3   2    13
## 5198 12-May-2018_16:16:18  Exp 7311  20      2    R     3   2    14
## 5199 12-May-2018_16:16:20  Exp 7311  20      2    R     3   2    15
## 5200 12-May-2018_16:16:22  Exp 7311  20      2    R     3   2    16
## 5201 12-May-2018_16:16:24  Exp 7311  20      2    R     3   2    17
## 5202 12-May-2018_16:16:26  Exp 7311  20      2    R     3   2    18
## 5203 12-May-2018_16:16:28  Exp 7311  20      2    R     3   2    19
## 5204 12-May-2018_16:16:30  Exp 7311  20      2    R     3   2    20
## 5205 12-May-2018_16:16:32  Exp 7311  20      2    R     3   2    21
## 5206 12-May-2018_16:16:34  Exp 7311  20      2    R     3   2    22
## 5207 12-May-2018_16:16:37  Exp 7311  20      2    R     3   2    23
## 5208 12-May-2018_16:16:39  Exp 7311  20      2    R     3   2    24
## 5209 12-May-2018_16:16:41  Exp 7311  20      2    R     3   3     1
## 5210 12-May-2018_16:16:43  Exp 7311  20      2    R     3   3     2
## 5211 12-May-2018_16:16:45  Exp 7311  20      2    R     3   3     3
## 5212 12-May-2018_16:16:47  Exp 7311  20      2    R     3   3     4
## 5213 12-May-2018_16:16:50  Exp 7311  20      2    R     3   3     5
## 5214 12-May-2018_16:16:52  Exp 7311  20      2    R     3   3     6
## 5215 12-May-2018_16:16:55  Exp 7311  20      2    R     3   3     7
## 5216 12-May-2018_16:16:57  Exp 7311  20      2    R     3   3     8
## 5217 12-May-2018_16:16:59  Exp 7311  20      2    R     3   3     9
## 5218 12-May-2018_16:17:01  Exp 7311  20      2    R     3   3    10
## 5219 12-May-2018_16:17:04  Exp 7311  20      2    R     3   3    11
## 5220 12-May-2018_16:17:06  Exp 7311  20      2    R     3   3    12
## 5221 12-May-2018_16:17:08  Exp 7311  20      2    R     3   3    13
## 5222 12-May-2018_16:17:10  Exp 7311  20      2    R     3   3    14
## 5223 12-May-2018_16:17:12  Exp 7311  20      2    R     3   3    15
## 5224 12-May-2018_16:17:15  Exp 7311  20      2    R     3   3    16
## 5225 12-May-2018_16:17:17  Exp 7311  20      2    R     3   3    17
## 5226 12-May-2018_16:17:19  Exp 7311  20      2    R     3   3    18
## 5227 12-May-2018_16:17:21  Exp 7311  20      2    R     3   3    19
## 5228 12-May-2018_16:17:23  Exp 7311  20      2    R     3   3    20
## 5229 12-May-2018_16:17:25  Exp 7311  20      2    R     3   3    21
## 5230 12-May-2018_16:17:28  Exp 7311  20      2    R     3   3    22
## 5231 12-May-2018_16:17:30  Exp 7311  20      2    R     3   3    23
## 5232 12-May-2018_16:17:32  Exp 7311  20      2    R     3   3    24
## 5233 12-May-2018_16:17:47  Exp 7311  20      2    R     3   4     1
## 5234 12-May-2018_16:17:49  Exp 7311  20      2    R     3   4     2
## 5235 12-May-2018_16:17:51  Exp 7311  20      2    R     3   4     3
## 5236 12-May-2018_16:17:54  Exp 7311  20      2    R     3   4     4
## 5237 12-May-2018_16:17:56  Exp 7311  20      2    R     3   4     5
## 5238 12-May-2018_16:17:58  Exp 7311  20      2    R     3   4     6
## 5239 12-May-2018_16:18:01  Exp 7311  20      2    R     3   4     7
## 5240 12-May-2018_16:18:03  Exp 7311  20      2    R     3   4     8
## 5241 12-May-2018_16:18:05  Exp 7311  20      2    R     3   4     9
## 5242 12-May-2018_16:18:07  Exp 7311  20      2    R     3   4    10
## 5243 12-May-2018_16:18:10  Exp 7311  20      2    R     3   4    11
## 5244 12-May-2018_16:18:12  Exp 7311  20      2    R     3   4    12
## 5245 12-May-2018_16:18:14  Exp 7311  20      2    R     3   4    13
## 5246 12-May-2018_16:18:16  Exp 7311  20      2    R     3   4    14
## 5247 12-May-2018_16:18:18  Exp 7311  20      2    R     3   4    15
## 5248 12-May-2018_16:18:20  Exp 7311  20      2    R     3   4    16
## 5249 12-May-2018_16:18:23  Exp 7311  20      2    R     3   4    17
## 5250 12-May-2018_16:18:25  Exp 7311  20      2    R     3   4    18
## 5251 12-May-2018_16:18:27  Exp 7311  20      2    R     3   4    19
## 5252 12-May-2018_16:18:29  Exp 7311  20      2    R     3   4    20
## 5253 12-May-2018_16:18:32  Exp 7311  20      2    R     3   4    21
## 5254 12-May-2018_16:18:34  Exp 7311  20      2    R     3   4    22
## 5255 12-May-2018_16:18:36  Exp 7311  20      2    R     3   4    23
## 5256 12-May-2018_16:18:38  Exp 7311  20      2    R     3   4    24
## 5257 12-May-2018_16:18:40  Exp 7311  20      2    R     3   5     1
## 5258 12-May-2018_16:18:43  Exp 7311  20      2    R     3   5     2
## 5259 12-May-2018_16:18:45  Exp 7311  20      2    R     3   5     3
## 5260 12-May-2018_16:18:47  Exp 7311  20      2    R     3   5     4
## 5261 12-May-2018_16:18:49  Exp 7311  20      2    R     3   5     5
## 5262 12-May-2018_16:18:51  Exp 7311  20      2    R     3   5     6
## 5263 12-May-2018_16:18:53  Exp 7311  20      2    R     3   5     7
## 5264 12-May-2018_16:18:56  Exp 7311  20      2    R     3   5     8
## 5265 12-May-2018_16:18:58  Exp 7311  20      2    R     3   5     9
## 5266 12-May-2018_16:19:00  Exp 7311  20      2    R     3   5    10
## 5267 12-May-2018_16:19:02  Exp 7311  20      2    R     3   5    11
## 5268 12-May-2018_16:19:04  Exp 7311  20      2    R     3   5    12
## 5269 12-May-2018_16:19:06  Exp 7311  20      2    R     3   5    13
## 5270 12-May-2018_16:19:09  Exp 7311  20      2    R     3   5    14
## 5271 12-May-2018_16:19:11  Exp 7311  20      2    R     3   5    15
## 5272 12-May-2018_16:19:13  Exp 7311  20      2    R     3   5    16
## 5273 12-May-2018_16:19:15  Exp 7311  20      2    R     3   5    17
## 5274 12-May-2018_16:19:17  Exp 7311  20      2    R     3   5    18
## 5275 12-May-2018_16:19:19  Exp 7311  20      2    R     3   5    19
## 5276 12-May-2018_16:19:22  Exp 7311  20      2    R     3   5    20
## 5277 12-May-2018_16:19:24  Exp 7311  20      2    R     3   5    21
## 5278 12-May-2018_16:19:26  Exp 7311  20      2    R     3   5    22
## 5279 12-May-2018_16:19:28  Exp 7311  20      2    R     3   5    23
## 5280 12-May-2018_16:19:31  Exp 7311  20      2    R     3   5    24
## 5281 12-May-2018_16:23:48  Exp 7311  20      2    R     1   1     1
## 5282 12-May-2018_16:23:50  Exp 7311  20      2    R     1   1     2
## 5283 12-May-2018_16:23:52  Exp 7311  20      2    R     1   1     3
## 5284 12-May-2018_16:23:54  Exp 7311  20      2    R     1   1     4
## 5285 12-May-2018_16:23:57  Exp 7311  20      2    R     1   1     5
## 5286 12-May-2018_16:23:59  Exp 7311  20      2    R     1   1     6
## 5287 12-May-2018_16:24:01  Exp 7311  20      2    R     1   1     7
## 5288 12-May-2018_16:24:03  Exp 7311  20      2    R     1   1     8
## 5289 12-May-2018_16:24:05  Exp 7311  20      2    R     1   1     9
## 5290 12-May-2018_16:24:07  Exp 7311  20      2    R     1   1    10
## 5291 12-May-2018_16:24:09  Exp 7311  20      2    R     1   1    11
## 5292 12-May-2018_16:24:12  Exp 7311  20      2    R     1   1    12
## 5293 12-May-2018_16:24:14  Exp 7311  20      2    R     1   1    13
## 5294 12-May-2018_16:24:16  Exp 7311  20      2    R     1   1    14
## 5295 12-May-2018_16:24:18  Exp 7311  20      2    R     1   1    15
## 5296 12-May-2018_16:24:20  Exp 7311  20      2    R     1   1    16
## 5297 12-May-2018_16:24:22  Exp 7311  20      2    R     1   1    17
## 5298 12-May-2018_16:24:24  Exp 7311  20      2    R     1   1    18
## 5299 12-May-2018_16:24:26  Exp 7311  20      2    R     1   1    19
## 5300 12-May-2018_16:24:28  Exp 7311  20      2    R     1   1    20
## 5301 12-May-2018_16:24:31  Exp 7311  20      2    R     1   1    21
## 5302 12-May-2018_16:24:33  Exp 7311  20      2    R     1   1    22
## 5303 12-May-2018_16:24:35  Exp 7311  20      2    R     1   1    23
## 5304 12-May-2018_16:24:37  Exp 7311  20      2    R     1   1    24
## 5305 12-May-2018_16:24:39  Exp 7311  20      2    R     1   2     1
## 5306 12-May-2018_16:24:42  Exp 7311  20      2    R     1   2     2
## 5307 12-May-2018_16:24:44  Exp 7311  20      2    R     1   2     3
## 5308 12-May-2018_16:24:46  Exp 7311  20      2    R     1   2     4
## 5309 12-May-2018_16:24:48  Exp 7311  20      2    R     1   2     5
## 5310 12-May-2018_16:24:50  Exp 7311  20      2    R     1   2     6
## 5311 12-May-2018_16:24:52  Exp 7311  20      2    R     1   2     7
## 5312 12-May-2018_16:24:54  Exp 7311  20      2    R     1   2     8
## 5313 12-May-2018_16:24:57  Exp 7311  20      2    R     1   2     9
## 5314 12-May-2018_16:24:59  Exp 7311  20      2    R     1   2    10
## 5315 12-May-2018_16:25:01  Exp 7311  20      2    R     1   2    11
## 5316 12-May-2018_16:25:03  Exp 7311  20      2    R     1   2    12
## 5317 12-May-2018_16:25:06  Exp 7311  20      2    R     1   2    13
## 5318 12-May-2018_16:25:08  Exp 7311  20      2    R     1   2    14
## 5319 12-May-2018_16:25:10  Exp 7311  20      2    R     1   2    15
## 5320 12-May-2018_16:25:12  Exp 7311  20      2    R     1   2    16
## 5321 12-May-2018_16:25:14  Exp 7311  20      2    R     1   2    17
## 5322 12-May-2018_16:25:16  Exp 7311  20      2    R     1   2    18
## 5323 12-May-2018_16:25:18  Exp 7311  20      2    R     1   2    19
## 5324 12-May-2018_16:25:21  Exp 7311  20      2    R     1   2    20
## 5325 12-May-2018_16:25:23  Exp 7311  20      2    R     1   2    21
## 5326 12-May-2018_16:25:25  Exp 7311  20      2    R     1   2    22
## 5327 12-May-2018_16:25:27  Exp 7311  20      2    R     1   2    23
## 5328 12-May-2018_16:25:30  Exp 7311  20      2    R     1   2    24
## 5329 12-May-2018_16:29:40  Exp 7311  20      2    R     1   1     1
## 5330 12-May-2018_16:29:42  Exp 7311  20      2    R     1   1     2
## 5331 12-May-2018_16:29:44  Exp 7311  20      2    R     1   1     3
## 5332 12-May-2018_16:29:46  Exp 7311  20      2    R     1   1     4
## 5333 12-May-2018_16:29:49  Exp 7311  20      2    R     1   1     5
## 5334 12-May-2018_16:29:51  Exp 7311  20      2    R     1   1     6
## 5335 12-May-2018_16:29:53  Exp 7311  20      2    R     1   1     7
## 5336 12-May-2018_16:29:55  Exp 7311  20      2    R     1   1     8
## 5337 12-May-2018_16:29:57  Exp 7311  20      2    R     1   1     9
## 5338 12-May-2018_16:30:00  Exp 7311  20      2    R     1   1    10
## 5339 12-May-2018_16:30:02  Exp 7311  20      2    R     1   1    11
## 5340 12-May-2018_16:30:04  Exp 7311  20      2    R     1   1    12
## 5341 12-May-2018_16:30:06  Exp 7311  20      2    R     1   1    13
## 5342 12-May-2018_16:30:08  Exp 7311  20      2    R     1   1    14
## 5343 12-May-2018_16:30:11  Exp 7311  20      2    R     1   1    15
## 5344 12-May-2018_16:30:13  Exp 7311  20      2    R     1   1    16
## 5345 12-May-2018_16:30:15  Exp 7311  20      2    R     1   1    17
## 5346 12-May-2018_16:30:17  Exp 7311  20      2    R     1   1    18
## 5347 12-May-2018_16:30:20  Exp 7311  20      2    R     1   1    19
## 5348 12-May-2018_16:30:22  Exp 7311  20      2    R     1   1    20
## 5349 12-May-2018_16:30:24  Exp 7311  20      2    R     1   1    21
## 5350 12-May-2018_16:30:26  Exp 7311  20      2    R     1   1    22
## 5351 12-May-2018_16:30:29  Exp 7311  20      2    R     1   1    23
## 5352 12-May-2018_16:30:31  Exp 7311  20      2    R     1   1    24
## 5353 12-May-2018_16:30:33  Exp 7311  20      2    R     1   2     1
## 5354 12-May-2018_16:30:35  Exp 7311  20      2    R     1   2     2
## 5355 12-May-2018_16:30:37  Exp 7311  20      2    R     1   2     3
## 5356 12-May-2018_16:30:40  Exp 7311  20      2    R     1   2     4
## 5357 12-May-2018_16:30:42  Exp 7311  20      2    R     1   2     5
## 5358 12-May-2018_16:30:44  Exp 7311  20      2    R     1   2     6
## 5359 12-May-2018_16:30:46  Exp 7311  20      2    R     1   2     7
## 5360 12-May-2018_16:30:48  Exp 7311  20      2    R     1   2     8
## 5361 12-May-2018_16:30:51  Exp 7311  20      2    R     1   2     9
## 5362 12-May-2018_16:30:53  Exp 7311  20      2    R     1   2    10
## 5363 12-May-2018_16:30:55  Exp 7311  20      2    R     1   2    11
## 5364 12-May-2018_16:30:57  Exp 7311  20      2    R     1   2    12
## 5365 12-May-2018_16:30:59  Exp 7311  20      2    R     1   2    13
## 5366 12-May-2018_16:31:02  Exp 7311  20      2    R     1   2    14
## 5367 12-May-2018_16:31:04  Exp 7311  20      2    R     1   2    15
## 5368 12-May-2018_16:31:06  Exp 7311  20      2    R     1   2    16
## 5369 12-May-2018_16:31:08  Exp 7311  20      2    R     1   2    17
## 5370 12-May-2018_16:31:10  Exp 7311  20      2    R     1   2    18
## 5371 12-May-2018_16:31:13  Exp 7311  20      2    R     1   2    19
## 5372 12-May-2018_16:31:15  Exp 7311  20      2    R     1   2    20
## 5373 12-May-2018_16:31:17  Exp 7311  20      2    R     1   2    21
## 5374 12-May-2018_16:31:19  Exp 7311  20      2    R     1   2    22
## 5375 12-May-2018_16:31:22  Exp 7311  20      2    R     1   2    23
## 5376 12-May-2018_16:31:24  Exp 7311  20      2    R     1   2    24
## 5377 12-May-2018_16:35:15  Exp 7311  20      2    R     1   1     1
## 5378 12-May-2018_16:35:18  Exp 7311  20      2    R     1   1     2
## 5379 12-May-2018_16:35:20  Exp 7311  20      2    R     1   1     3
## 5380 12-May-2018_16:35:22  Exp 7311  20      2    R     1   1     4
## 5381 12-May-2018_16:35:24  Exp 7311  20      2    R     1   1     5
## 5382 12-May-2018_16:35:27  Exp 7311  20      2    R     1   1     6
## 5383 12-May-2018_16:35:29  Exp 7311  20      2    R     1   1     7
## 5384 12-May-2018_16:35:31  Exp 7311  20      2    R     1   1     8
## 5385 12-May-2018_16:35:33  Exp 7311  20      2    R     1   1     9
## 5386 12-May-2018_16:35:36  Exp 7311  20      2    R     1   1    10
## 5387 12-May-2018_16:35:38  Exp 7311  20      2    R     1   1    11
## 5388 12-May-2018_16:35:40  Exp 7311  20      2    R     1   1    12
## 5389 12-May-2018_16:35:42  Exp 7311  20      2    R     1   1    13
## 5390 12-May-2018_16:35:44  Exp 7311  20      2    R     1   1    14
## 5391 12-May-2018_16:35:47  Exp 7311  20      2    R     1   1    15
## 5392 12-May-2018_16:35:49  Exp 7311  20      2    R     1   1    16
## 5393 12-May-2018_16:35:51  Exp 7311  20      2    R     1   1    17
## 5394 12-May-2018_16:35:54  Exp 7311  20      2    R     1   1    18
## 5395 12-May-2018_16:35:56  Exp 7311  20      2    R     1   1    19
## 5396 12-May-2018_16:35:58  Exp 7311  20      2    R     1   1    20
## 5397 12-May-2018_16:36:00  Exp 7311  20      2    R     1   1    21
## 5398 12-May-2018_16:36:02  Exp 7311  20      2    R     1   1    22
## 5399 12-May-2018_16:36:04  Exp 7311  20      2    R     1   1    23
## 5400 12-May-2018_16:36:06  Exp 7311  20      2    R     1   1    24
## 5401 12-May-2018_16:36:09  Exp 7311  20      2    R     1   2     1
## 5402 12-May-2018_16:36:11  Exp 7311  20      2    R     1   2     2
## 5403 12-May-2018_16:36:13  Exp 7311  20      2    R     1   2     3
## 5404 12-May-2018_16:36:16  Exp 7311  20      2    R     1   2     4
## 5405 12-May-2018_16:36:18  Exp 7311  20      2    R     1   2     5
## 5406 12-May-2018_16:36:20  Exp 7311  20      2    R     1   2     6
## 5407 12-May-2018_16:36:23  Exp 7311  20      2    R     1   2     7
## 5408 12-May-2018_16:36:25  Exp 7311  20      2    R     1   2     8
## 5409 12-May-2018_16:36:27  Exp 7311  20      2    R     1   2     9
## 5410 12-May-2018_16:36:29  Exp 7311  20      2    R     1   2    10
## 5411 12-May-2018_16:36:32  Exp 7311  20      2    R     1   2    11
## 5412 12-May-2018_16:36:34  Exp 7311  20      2    R     1   2    12
## 5413 12-May-2018_16:36:36  Exp 7311  20      2    R     1   2    13
## 5414 12-May-2018_16:36:38  Exp 7311  20      2    R     1   2    14
## 5415 12-May-2018_16:36:40  Exp 7311  20      2    R     1   2    15
## 5416 12-May-2018_16:36:43  Exp 7311  20      2    R     1   2    16
## 5417 12-May-2018_16:36:45  Exp 7311  20      2    R     1   2    17
## 5418 12-May-2018_16:36:47  Exp 7311  20      2    R     1   2    18
## 5419 12-May-2018_16:36:49  Exp 7311  20      2    R     1   2    19
## 5420 12-May-2018_16:36:51  Exp 7311  20      2    R     1   2    20
## 5421 12-May-2018_16:36:54  Exp 7311  20      2    R     1   2    21
## 5422 12-May-2018_16:36:56  Exp 7311  20      2    R     1   2    22
## 5423 12-May-2018_16:36:58  Exp 7311  20      2    R     1   2    23
## 5424 12-May-2018_16:37:00  Exp 7311  20      2    R     1   2    24
## 5425 12-May-2018_16:40:51  Exp 7311  20      2    R     1   1     1
## 5426 12-May-2018_16:40:53  Exp 7311  20      2    R     1   1     2
## 5427 12-May-2018_16:40:55  Exp 7311  20      2    R     1   1     3
## 5428 12-May-2018_16:40:57  Exp 7311  20      2    R     1   1     4
## 5429 12-May-2018_16:40:59  Exp 7311  20      2    R     1   1     5
## 5430 12-May-2018_16:41:01  Exp 7311  20      2    R     1   1     6
## 5431 12-May-2018_16:41:03  Exp 7311  20      2    R     1   1     7
## 5432 12-May-2018_16:41:05  Exp 7311  20      2    R     1   1     8
## 5433 12-May-2018_16:41:07  Exp 7311  20      2    R     1   1     9
## 5434 12-May-2018_16:41:10  Exp 7311  20      2    R     1   1    10
## 5435 12-May-2018_16:41:12  Exp 7311  20      2    R     1   1    11
## 5436 12-May-2018_16:41:14  Exp 7311  20      2    R     1   1    12
## 5437 12-May-2018_16:41:16  Exp 7311  20      2    R     1   1    13
## 5438 12-May-2018_16:41:19  Exp 7311  20      2    R     1   1    14
## 5439 12-May-2018_16:41:21  Exp 7311  20      2    R     1   1    15
## 5440 12-May-2018_16:41:23  Exp 7311  20      2    R     1   1    16
## 5441 12-May-2018_16:41:25  Exp 7311  20      2    R     1   1    17
## 5442 12-May-2018_16:41:27  Exp 7311  20      2    R     1   1    18
## 5443 12-May-2018_16:41:29  Exp 7311  20      2    R     1   1    19
## 5444 12-May-2018_16:41:31  Exp 7311  20      2    R     1   1    20
## 5445 12-May-2018_16:41:34  Exp 7311  20      2    R     1   1    21
## 5446 12-May-2018_16:41:36  Exp 7311  20      2    R     1   1    22
## 5447 12-May-2018_16:41:38  Exp 7311  20      2    R     1   1    23
## 5448 12-May-2018_16:41:40  Exp 7311  20      2    R     1   1    24
## 5449 12-May-2018_16:41:42  Exp 7311  20      2    R     1   2     1
## 5450 12-May-2018_16:41:44  Exp 7311  20      2    R     1   2     2
## 5451 12-May-2018_16:41:46  Exp 7311  20      2    R     1   2     3
## 5452 12-May-2018_16:41:48  Exp 7311  20      2    R     1   2     4
## 5453 12-May-2018_16:41:50  Exp 7311  20      2    R     1   2     5
## 5454 12-May-2018_16:41:53  Exp 7311  20      2    R     1   2     6
## 5455 12-May-2018_16:41:55  Exp 7311  20      2    R     1   2     7
## 5456 12-May-2018_16:41:57  Exp 7311  20      2    R     1   2     8
## 5457 12-May-2018_16:42:00  Exp 7311  20      2    R     1   2     9
## 5458 12-May-2018_16:42:02  Exp 7311  20      2    R     1   2    10
## 5459 12-May-2018_16:42:04  Exp 7311  20      2    R     1   2    11
## 5460 12-May-2018_16:42:06  Exp 7311  20      2    R     1   2    12
## 5461 12-May-2018_16:42:09  Exp 7311  20      2    R     1   2    13
## 5462 12-May-2018_16:42:11  Exp 7311  20      2    R     1   2    14
## 5463 12-May-2018_16:42:13  Exp 7311  20      2    R     1   2    15
## 5464 12-May-2018_16:42:15  Exp 7311  20      2    R     1   2    16
## 5465 12-May-2018_16:42:17  Exp 7311  20      2    R     1   2    17
## 5466 12-May-2018_16:42:20  Exp 7311  20      2    R     1   2    18
## 5467 12-May-2018_16:42:22  Exp 7311  20      2    R     1   2    19
## 5468 12-May-2018_16:42:24  Exp 7311  20      2    R     1   2    20
## 5469 12-May-2018_16:42:26  Exp 7311  20      2    R     1   2    21
## 5470 12-May-2018_16:42:28  Exp 7311  20      2    R     1   2    22
## 5471 12-May-2018_16:42:30  Exp 7311  20      2    R     1   2    23
## 5472 12-May-2018_16:42:33  Exp 7311  20      2    R     1   2    24
## 5473 12-May-2018_16:46:34  Exp 7311  20      2    R     1   1     1
## 5474 12-May-2018_16:46:37  Exp 7311  20      2    R     1   1     2
## 5475 12-May-2018_16:46:39  Exp 7311  20      2    R     1   1     3
## 5476 12-May-2018_16:46:41  Exp 7311  20      2    R     1   1     4
## 5477 12-May-2018_16:46:43  Exp 7311  20      2    R     1   1     5
## 5478 12-May-2018_16:46:45  Exp 7311  20      2    R     1   1     6
## 5479 12-May-2018_16:46:47  Exp 7311  20      2    R     1   1     7
## 5480 12-May-2018_16:46:50  Exp 7311  20      2    R     1   1     8
## 5481 12-May-2018_16:46:52  Exp 7311  20      2    R     1   1     9
## 5482 12-May-2018_16:46:54  Exp 7311  20      2    R     1   1    10
## 5483 12-May-2018_16:46:56  Exp 7311  20      2    R     1   1    11
## 5484 12-May-2018_16:46:59  Exp 7311  20      2    R     1   1    12
## 5485 12-May-2018_16:47:01  Exp 7311  20      2    R     1   1    13
## 5486 12-May-2018_16:47:03  Exp 7311  20      2    R     1   1    14
## 5487 12-May-2018_16:47:05  Exp 7311  20      2    R     1   1    15
## 5488 12-May-2018_16:47:07  Exp 7311  20      2    R     1   1    16
## 5489 12-May-2018_16:47:09  Exp 7311  20      2    R     1   1    17
## 5490 12-May-2018_16:47:12  Exp 7311  20      2    R     1   1    18
## 5491 12-May-2018_16:47:14  Exp 7311  20      2    R     1   1    19
## 5492 12-May-2018_16:47:16  Exp 7311  20      2    R     1   1    20
## 5493 12-May-2018_16:47:18  Exp 7311  20      2    R     1   1    21
## 5494 12-May-2018_16:47:20  Exp 7311  20      2    R     1   1    22
## 5495 12-May-2018_16:47:23  Exp 7311  20      2    R     1   1    23
## 5496 12-May-2018_16:47:25  Exp 7311  20      2    R     1   1    24
## 5497 12-May-2018_16:47:27  Exp 7311  20      2    R     1   2     1
## 5498 12-May-2018_16:47:29  Exp 7311  20      2    R     1   2     2
## 5499 12-May-2018_16:47:32  Exp 7311  20      2    R     1   2     3
## 5500 12-May-2018_16:47:34  Exp 7311  20      2    R     1   2     4
## 5501 12-May-2018_16:47:36  Exp 7311  20      2    R     1   2     5
## 5502 12-May-2018_16:47:38  Exp 7311  20      2    R     1   2     6
## 5503 12-May-2018_16:47:40  Exp 7311  20      2    R     1   2     7
## 5504 12-May-2018_16:47:42  Exp 7311  20      2    R     1   2     8
## 5505 12-May-2018_16:47:45  Exp 7311  20      2    R     1   2     9
## 5506 12-May-2018_16:47:47  Exp 7311  20      2    R     1   2    10
## 5507 12-May-2018_16:47:49  Exp 7311  20      2    R     1   2    11
## 5508 12-May-2018_16:47:51  Exp 7311  20      2    R     1   2    12
## 5509 12-May-2018_16:47:54  Exp 7311  20      2    R     1   2    13
## 5510 12-May-2018_16:47:56  Exp 7311  20      2    R     1   2    14
## 5511 12-May-2018_16:47:58  Exp 7311  20      2    R     1   2    15
## 5512 12-May-2018_16:48:00  Exp 7311  20      2    R     1   2    16
## 5513 12-May-2018_16:48:02  Exp 7311  20      2    R     1   2    17
## 5514 12-May-2018_16:48:05  Exp 7311  20      2    R     1   2    18
## 5515 12-May-2018_16:48:07  Exp 7311  20      2    R     1   2    19
## 5516 12-May-2018_16:48:09  Exp 7311  20      2    R     1   2    20
## 5517 12-May-2018_16:48:11  Exp 7311  20      2    R     1   2    21
## 5518 12-May-2018_16:48:13  Exp 7311  20      2    R     1   2    22
## 5519 12-May-2018_16:48:15  Exp 7311  20      2    R     1   2    23
## 5520 12-May-2018_16:48:17  Exp 7311  20      2    R     1   2    24
## 5521 13-May-2018_14:10:14  Exp 7312  21 female    R     1   1     1
## 5522 13-May-2018_14:10:16  Exp 7312  21 female    R     1   1     2
## 5523 13-May-2018_14:10:18  Exp 7312  21 female    R     1   1     3
## 5524 13-May-2018_14:10:21  Exp 7312  21 female    R     1   1     4
## 5525 13-May-2018_14:10:23  Exp 7312  21 female    R     1   1     5
## 5526 13-May-2018_14:10:26  Exp 7312  21 female    R     1   1     6
## 5527 13-May-2018_14:10:28  Exp 7312  21 female    R     1   1     7
## 5528 13-May-2018_14:10:31  Exp 7312  21 female    R     1   1     8
## 5529 13-May-2018_14:10:33  Exp 7312  21 female    R     1   1     9
## 5530 13-May-2018_14:10:36  Exp 7312  21 female    R     1   1    10
## 5531 13-May-2018_14:10:38  Exp 7312  21 female    R     1   1    11
## 5532 13-May-2018_14:10:40  Exp 7312  21 female    R     1   1    12
## 5533 13-May-2018_14:10:42  Exp 7312  21 female    R     1   1    13
## 5534 13-May-2018_14:10:45  Exp 7312  21 female    R     1   1    14
## 5535 13-May-2018_14:10:47  Exp 7312  21 female    R     1   1    15
## 5536 13-May-2018_14:10:49  Exp 7312  21 female    R     1   1    16
## 5537 13-May-2018_14:10:52  Exp 7312  21 female    R     1   1    17
## 5538 13-May-2018_14:10:55  Exp 7312  21 female    R     1   1    18
## 5539 13-May-2018_14:10:57  Exp 7312  21 female    R     1   1    19
## 5540 13-May-2018_14:11:00  Exp 7312  21 female    R     1   1    20
## 5541 13-May-2018_14:11:02  Exp 7312  21 female    R     1   1    21
## 5542 13-May-2018_14:11:05  Exp 7312  21 female    R     1   1    22
## 5543 13-May-2018_14:11:08  Exp 7312  21 female    R     1   1    23
## 5544 13-May-2018_14:11:10  Exp 7312  21 female    R     1   1    24
## 5545 13-May-2018_14:11:12  Exp 7312  21 female    R     1   2     1
## 5546 13-May-2018_14:11:15  Exp 7312  21 female    R     1   2     2
## 5547 13-May-2018_14:11:17  Exp 7312  21 female    R     1   2     3
## 5548 13-May-2018_14:11:20  Exp 7312  21 female    R     1   2     4
## 5549 13-May-2018_14:11:22  Exp 7312  21 female    R     1   2     5
## 5550 13-May-2018_14:11:25  Exp 7312  21 female    R     1   2     6
## 5551 13-May-2018_14:11:27  Exp 7312  21 female    R     1   2     7
## 5552 13-May-2018_14:11:29  Exp 7312  21 female    R     1   2     8
## 5553 13-May-2018_14:11:32  Exp 7312  21 female    R     1   2     9
## 5554 13-May-2018_14:11:35  Exp 7312  21 female    R     1   2    10
## 5555 13-May-2018_14:11:37  Exp 7312  21 female    R     1   2    11
## 5556 13-May-2018_14:11:39  Exp 7312  21 female    R     1   2    12
## 5557 13-May-2018_14:11:42  Exp 7312  21 female    R     1   2    13
## 5558 13-May-2018_14:11:44  Exp 7312  21 female    R     1   2    14
## 5559 13-May-2018_14:11:46  Exp 7312  21 female    R     1   2    15
## 5560 13-May-2018_14:11:49  Exp 7312  21 female    R     1   2    16
## 5561 13-May-2018_14:11:52  Exp 7312  21 female    R     1   2    17
## 5562 13-May-2018_14:11:54  Exp 7312  21 female    R     1   2    18
## 5563 13-May-2018_14:11:57  Exp 7312  21 female    R     1   2    19
## 5564 13-May-2018_14:11:59  Exp 7312  21 female    R     1   2    20
## 5565 13-May-2018_14:12:02  Exp 7312  21 female    R     1   2    21
## 5566 13-May-2018_14:12:04  Exp 7312  21 female    R     1   2    22
## 5567 13-May-2018_14:12:07  Exp 7312  21 female    R     1   2    23
## 5568 13-May-2018_14:12:09  Exp 7312  21 female    R     1   2    24
## 5569 13-May-2018_14:12:12  Exp 7312  21 female    R     1   3     1
## 5570 13-May-2018_14:12:14  Exp 7312  21 female    R     1   3     2
## 5571 13-May-2018_14:12:17  Exp 7312  21 female    R     1   3     3
## 5572 13-May-2018_14:12:19  Exp 7312  21 female    R     1   3     4
## 5573 13-May-2018_14:12:22  Exp 7312  21 female    R     1   3     5
## 5574 13-May-2018_14:12:24  Exp 7312  21 female    R     1   3     6
## 5575 13-May-2018_14:12:27  Exp 7312  21 female    R     1   3     7
## 5576 13-May-2018_14:12:29  Exp 7312  21 female    R     1   3     8
## 5577 13-May-2018_14:12:32  Exp 7312  21 female    R     1   3     9
## 5578 13-May-2018_14:12:34  Exp 7312  21 female    R     1   3    10
## 5579 13-May-2018_14:12:36  Exp 7312  21 female    R     1   3    11
## 5580 13-May-2018_14:12:39  Exp 7312  21 female    R     1   3    12
## 5581 13-May-2018_14:12:41  Exp 7312  21 female    R     1   3    13
## 5582 13-May-2018_14:12:44  Exp 7312  21 female    R     1   3    14
## 5583 13-May-2018_14:12:46  Exp 7312  21 female    R     1   3    15
## 5584 13-May-2018_14:12:49  Exp 7312  21 female    R     1   3    16
## 5585 13-May-2018_14:12:51  Exp 7312  21 female    R     1   3    17
## 5586 13-May-2018_14:12:53  Exp 7312  21 female    R     1   3    18
## 5587 13-May-2018_14:12:55  Exp 7312  21 female    R     1   3    19
## 5588 13-May-2018_14:12:58  Exp 7312  21 female    R     1   3    20
## 5589 13-May-2018_14:13:00  Exp 7312  21 female    R     1   3    21
## 5590 13-May-2018_14:13:02  Exp 7312  21 female    R     1   3    22
## 5591 13-May-2018_14:13:05  Exp 7312  21 female    R     1   3    23
## 5592 13-May-2018_14:13:07  Exp 7312  21 female    R     1   3    24
## 5593 13-May-2018_14:13:23  Exp 7312  21 female    R     1   4     1
## 5594 13-May-2018_14:13:26  Exp 7312  21 female    R     1   4     2
## 5595 13-May-2018_14:13:28  Exp 7312  21 female    R     1   4     3
## 5596 13-May-2018_14:13:31  Exp 7312  21 female    R     1   4     4
## 5597 13-May-2018_14:13:33  Exp 7312  21 female    R     1   4     5
## 5598 13-May-2018_14:13:35  Exp 7312  21 female    R     1   4     6
## 5599 13-May-2018_14:13:38  Exp 7312  21 female    R     1   4     7
## 5600 13-May-2018_14:13:40  Exp 7312  21 female    R     1   4     8
## 5601 13-May-2018_14:13:43  Exp 7312  21 female    R     1   4     9
## 5602 13-May-2018_14:13:45  Exp 7312  21 female    R     1   4    10
## 5603 13-May-2018_14:13:47  Exp 7312  21 female    R     1   4    11
## 5604 13-May-2018_14:13:50  Exp 7312  21 female    R     1   4    12
## 5605 13-May-2018_14:13:52  Exp 7312  21 female    R     1   4    13
## 5606 13-May-2018_14:13:55  Exp 7312  21 female    R     1   4    14
## 5607 13-May-2018_14:13:57  Exp 7312  21 female    R     1   4    15
## 5608 13-May-2018_14:13:59  Exp 7312  21 female    R     1   4    16
## 5609 13-May-2018_14:14:02  Exp 7312  21 female    R     1   4    17
## 5610 13-May-2018_14:14:04  Exp 7312  21 female    R     1   4    18
## 5611 13-May-2018_14:14:07  Exp 7312  21 female    R     1   4    19
## 5612 13-May-2018_14:14:09  Exp 7312  21 female    R     1   4    20
## 5613 13-May-2018_14:14:12  Exp 7312  21 female    R     1   4    21
## 5614 13-May-2018_14:14:14  Exp 7312  21 female    R     1   4    22
## 5615 13-May-2018_14:14:16  Exp 7312  21 female    R     1   4    23
## 5616 13-May-2018_14:14:18  Exp 7312  21 female    R     1   4    24
## 5617 13-May-2018_14:14:21  Exp 7312  21 female    R     1   5     1
## 5618 13-May-2018_14:14:23  Exp 7312  21 female    R     1   5     2
## 5619 13-May-2018_14:14:26  Exp 7312  21 female    R     1   5     3
## 5620 13-May-2018_14:14:28  Exp 7312  21 female    R     1   5     4
## 5621 13-May-2018_14:14:30  Exp 7312  21 female    R     1   5     5
## 5622 13-May-2018_14:14:33  Exp 7312  21 female    R     1   5     6
## 5623 13-May-2018_14:14:35  Exp 7312  21 female    R     1   5     7
## 5624 13-May-2018_14:14:37  Exp 7312  21 female    R     1   5     8
## 5625 13-May-2018_14:14:40  Exp 7312  21 female    R     1   5     9
## 5626 13-May-2018_14:14:42  Exp 7312  21 female    R     1   5    10
## 5627 13-May-2018_14:14:44  Exp 7312  21 female    R     1   5    11
## 5628 13-May-2018_14:14:47  Exp 7312  21 female    R     1   5    12
## 5629 13-May-2018_14:14:49  Exp 7312  21 female    R     1   5    13
## 5630 13-May-2018_14:14:51  Exp 7312  21 female    R     1   5    14
## 5631 13-May-2018_14:14:54  Exp 7312  21 female    R     1   5    15
## 5632 13-May-2018_14:14:56  Exp 7312  21 female    R     1   5    16
## 5633 13-May-2018_14:14:58  Exp 7312  21 female    R     1   5    17
## 5634 13-May-2018_14:15:01  Exp 7312  21 female    R     1   5    18
## 5635 13-May-2018_14:15:03  Exp 7312  21 female    R     1   5    19
## 5636 13-May-2018_14:15:06  Exp 7312  21 female    R     1   5    20
## 5637 13-May-2018_14:15:08  Exp 7312  21 female    R     1   5    21
## 5638 13-May-2018_14:15:10  Exp 7312  21 female    R     1   5    22
## 5639 13-May-2018_14:15:13  Exp 7312  21 female    R     1   5    23
## 5640 13-May-2018_14:15:15  Exp 7312  21 female    R     1   5    24
## 5641 13-May-2018_14:15:21  Exp 7312  21 female    R     2   1     1
## 5642 13-May-2018_14:15:23  Exp 7312  21 female    R     2   1     2
## 5643 13-May-2018_14:15:25  Exp 7312  21 female    R     2   1     3
## 5644 13-May-2018_14:15:28  Exp 7312  21 female    R     2   1     4
## 5645 13-May-2018_14:15:30  Exp 7312  21 female    R     2   1     5
## 5646 13-May-2018_14:15:32  Exp 7312  21 female    R     2   1     6
## 5647 13-May-2018_14:15:35  Exp 7312  21 female    R     2   1     7
## 5648 13-May-2018_14:15:38  Exp 7312  21 female    R     2   1     8
## 5649 13-May-2018_14:15:40  Exp 7312  21 female    R     2   1     9
## 5650 13-May-2018_14:15:42  Exp 7312  21 female    R     2   1    10
## 5651 13-May-2018_14:15:44  Exp 7312  21 female    R     2   1    11
## 5652 13-May-2018_14:15:47  Exp 7312  21 female    R     2   1    12
## 5653 13-May-2018_14:15:50  Exp 7312  21 female    R     2   1    13
## 5654 13-May-2018_14:15:52  Exp 7312  21 female    R     2   1    14
## 5655 13-May-2018_14:15:55  Exp 7312  21 female    R     2   1    15
## 5656 13-May-2018_14:15:57  Exp 7312  21 female    R     2   1    16
## 5657 13-May-2018_14:16:00  Exp 7312  21 female    R     2   1    17
## 5658 13-May-2018_14:16:02  Exp 7312  21 female    R     2   1    18
## 5659 13-May-2018_14:16:04  Exp 7312  21 female    R     2   1    19
## 5660 13-May-2018_14:16:06  Exp 7312  21 female    R     2   1    20
## 5661 13-May-2018_14:16:09  Exp 7312  21 female    R     2   1    21
## 5662 13-May-2018_14:16:11  Exp 7312  21 female    R     2   1    22
## 5663 13-May-2018_14:16:13  Exp 7312  21 female    R     2   1    23
## 5664 13-May-2018_14:16:16  Exp 7312  21 female    R     2   1    24
## 5665 13-May-2018_14:16:18  Exp 7312  21 female    R     2   2     1
## 5666 13-May-2018_14:16:20  Exp 7312  21 female    R     2   2     2
## 5667 13-May-2018_14:16:23  Exp 7312  21 female    R     2   2     3
## 5668 13-May-2018_14:16:25  Exp 7312  21 female    R     2   2     4
## 5669 13-May-2018_14:16:27  Exp 7312  21 female    R     2   2     5
## 5670 13-May-2018_14:16:30  Exp 7312  21 female    R     2   2     6
## 5671 13-May-2018_14:16:32  Exp 7312  21 female    R     2   2     7
## 5672 13-May-2018_14:16:34  Exp 7312  21 female    R     2   2     8
## 5673 13-May-2018_14:16:37  Exp 7312  21 female    R     2   2     9
## 5674 13-May-2018_14:16:39  Exp 7312  21 female    R     2   2    10
## 5675 13-May-2018_14:16:41  Exp 7312  21 female    R     2   2    11
## 5676 13-May-2018_14:16:44  Exp 7312  21 female    R     2   2    12
## 5677 13-May-2018_14:16:46  Exp 7312  21 female    R     2   2    13
## 5678 13-May-2018_14:16:48  Exp 7312  21 female    R     2   2    14
## 5679 13-May-2018_14:16:51  Exp 7312  21 female    R     2   2    15
## 5680 13-May-2018_14:16:53  Exp 7312  21 female    R     2   2    16
## 5681 13-May-2018_14:16:56  Exp 7312  21 female    R     2   2    17
## 5682 13-May-2018_14:16:58  Exp 7312  21 female    R     2   2    18
## 5683 13-May-2018_14:17:00  Exp 7312  21 female    R     2   2    19
## 5684 13-May-2018_14:17:03  Exp 7312  21 female    R     2   2    20
## 5685 13-May-2018_14:17:05  Exp 7312  21 female    R     2   2    21
## 5686 13-May-2018_14:17:08  Exp 7312  21 female    R     2   2    22
## 5687 13-May-2018_14:17:10  Exp 7312  21 female    R     2   2    23
## 5688 13-May-2018_14:17:12  Exp 7312  21 female    R     2   2    24
## 5689 13-May-2018_14:17:14  Exp 7312  21 female    R     2   3     1
## 5690 13-May-2018_14:17:17  Exp 7312  21 female    R     2   3     2
## 5691 13-May-2018_14:17:19  Exp 7312  21 female    R     2   3     3
## 5692 13-May-2018_14:17:22  Exp 7312  21 female    R     2   3     4
## 5693 13-May-2018_14:17:25  Exp 7312  21 female    R     2   3     5
## 5694 13-May-2018_14:17:27  Exp 7312  21 female    R     2   3     6
## 5695 13-May-2018_14:17:29  Exp 7312  21 female    R     2   3     7
## 5696 13-May-2018_14:17:32  Exp 7312  21 female    R     2   3     8
## 5697 13-May-2018_14:17:34  Exp 7312  21 female    R     2   3     9
## 5698 13-May-2018_14:17:37  Exp 7312  21 female    R     2   3    10
## 5699 13-May-2018_14:17:39  Exp 7312  21 female    R     2   3    11
## 5700 13-May-2018_14:17:41  Exp 7312  21 female    R     2   3    12
## 5701 13-May-2018_14:17:44  Exp 7312  21 female    R     2   3    13
## 5702 13-May-2018_14:17:46  Exp 7312  21 female    R     2   3    14
## 5703 13-May-2018_14:17:48  Exp 7312  21 female    R     2   3    15
## 5704 13-May-2018_14:17:51  Exp 7312  21 female    R     2   3    16
## 5705 13-May-2018_14:17:53  Exp 7312  21 female    R     2   3    17
## 5706 13-May-2018_14:17:56  Exp 7312  21 female    R     2   3    18
## 5707 13-May-2018_14:17:58  Exp 7312  21 female    R     2   3    19
## 5708 13-May-2018_14:18:00  Exp 7312  21 female    R     2   3    20
## 5709 13-May-2018_14:18:03  Exp 7312  21 female    R     2   3    21
## 5710 13-May-2018_14:18:05  Exp 7312  21 female    R     2   3    22
## 5711 13-May-2018_14:18:08  Exp 7312  21 female    R     2   3    23
## 5712 13-May-2018_14:18:10  Exp 7312  21 female    R     2   3    24
## 5713 13-May-2018_14:18:35  Exp 7312  21 female    R     2   4     1
## 5714 13-May-2018_14:18:37  Exp 7312  21 female    R     2   4     2
## 5715 13-May-2018_14:18:40  Exp 7312  21 female    R     2   4     3
## 5716 13-May-2018_14:18:42  Exp 7312  21 female    R     2   4     4
## 5717 13-May-2018_14:18:45  Exp 7312  21 female    R     2   4     5
## 5718 13-May-2018_14:18:47  Exp 7312  21 female    R     2   4     6
## 5719 13-May-2018_14:18:50  Exp 7312  21 female    R     2   4     7
## 5720 13-May-2018_14:18:52  Exp 7312  21 female    R     2   4     8
## 5721 13-May-2018_14:18:55  Exp 7312  21 female    R     2   4     9
## 5722 13-May-2018_14:18:57  Exp 7312  21 female    R     2   4    10
## 5723 13-May-2018_14:18:59  Exp 7312  21 female    R     2   4    11
## 5724 13-May-2018_14:19:02  Exp 7312  21 female    R     2   4    12
## 5725 13-May-2018_14:19:04  Exp 7312  21 female    R     2   4    13
## 5726 13-May-2018_14:19:06  Exp 7312  21 female    R     2   4    14
## 5727 13-May-2018_14:19:09  Exp 7312  21 female    R     2   4    15
## 5728 13-May-2018_14:19:11  Exp 7312  21 female    R     2   4    16
## 5729 13-May-2018_14:19:13  Exp 7312  21 female    R     2   4    17
## 5730 13-May-2018_14:19:16  Exp 7312  21 female    R     2   4    18
## 5731 13-May-2018_14:19:19  Exp 7312  21 female    R     2   4    19
## 5732 13-May-2018_14:19:21  Exp 7312  21 female    R     2   4    20
## 5733 13-May-2018_14:19:23  Exp 7312  21 female    R     2   4    21
## 5734 13-May-2018_14:19:26  Exp 7312  21 female    R     2   4    22
## 5735 13-May-2018_14:19:28  Exp 7312  21 female    R     2   4    23
## 5736 13-May-2018_14:19:31  Exp 7312  21 female    R     2   4    24
## 5737 13-May-2018_14:19:33  Exp 7312  21 female    R     2   5     1
## 5738 13-May-2018_14:19:35  Exp 7312  21 female    R     2   5     2
## 5739 13-May-2018_14:19:38  Exp 7312  21 female    R     2   5     3
## 5740 13-May-2018_14:19:40  Exp 7312  21 female    R     2   5     4
## 5741 13-May-2018_14:19:43  Exp 7312  21 female    R     2   5     5
## 5742 13-May-2018_14:19:45  Exp 7312  21 female    R     2   5     6
## 5743 13-May-2018_14:19:47  Exp 7312  21 female    R     2   5     7
## 5744 13-May-2018_14:19:50  Exp 7312  21 female    R     2   5     8
## 5745 13-May-2018_14:19:52  Exp 7312  21 female    R     2   5     9
## 5746 13-May-2018_14:19:55  Exp 7312  21 female    R     2   5    10
## 5747 13-May-2018_14:19:57  Exp 7312  21 female    R     2   5    11
## 5748 13-May-2018_14:20:00  Exp 7312  21 female    R     2   5    12
## 5749 13-May-2018_14:20:02  Exp 7312  21 female    R     2   5    13
## 5750 13-May-2018_14:20:05  Exp 7312  21 female    R     2   5    14
## 5751 13-May-2018_14:20:07  Exp 7312  21 female    R     2   5    15
## 5752 13-May-2018_14:20:10  Exp 7312  21 female    R     2   5    16
## 5753 13-May-2018_14:20:12  Exp 7312  21 female    R     2   5    17
## 5754 13-May-2018_14:20:14  Exp 7312  21 female    R     2   5    18
## 5755 13-May-2018_14:20:17  Exp 7312  21 female    R     2   5    19
## 5756 13-May-2018_14:20:19  Exp 7312  21 female    R     2   5    20
## 5757 13-May-2018_14:20:21  Exp 7312  21 female    R     2   5    21
## 5758 13-May-2018_14:20:24  Exp 7312  21 female    R     2   5    22
## 5759 13-May-2018_14:20:26  Exp 7312  21 female    R     2   5    23
## 5760 13-May-2018_14:20:29  Exp 7312  21 female    R     2   5    24
## 5761 13-May-2018_14:20:34  Exp 7312  21 female    R     3   1     1
## 5762 13-May-2018_14:20:37  Exp 7312  21 female    R     3   1     2
## 5763 13-May-2018_14:20:39  Exp 7312  21 female    R     3   1     3
## 5764 13-May-2018_14:20:42  Exp 7312  21 female    R     3   1     4
## 5765 13-May-2018_14:20:44  Exp 7312  21 female    R     3   1     5
## 5766 13-May-2018_14:20:46  Exp 7312  21 female    R     3   1     6
## 5767 13-May-2018_14:20:49  Exp 7312  21 female    R     3   1     7
## 5768 13-May-2018_14:20:51  Exp 7312  21 female    R     3   1     8
## 5769 13-May-2018_14:20:54  Exp 7312  21 female    R     3   1     9
## 5770 13-May-2018_14:20:56  Exp 7312  21 female    R     3   1    10
## 5771 13-May-2018_14:20:58  Exp 7312  21 female    R     3   1    11
## 5772 13-May-2018_14:21:01  Exp 7312  21 female    R     3   1    12
## 5773 13-May-2018_14:21:03  Exp 7312  21 female    R     3   1    13
## 5774 13-May-2018_14:21:05  Exp 7312  21 female    R     3   1    14
## 5775 13-May-2018_14:21:07  Exp 7312  21 female    R     3   1    15
## 5776 13-May-2018_14:21:10  Exp 7312  21 female    R     3   1    16
## 5777 13-May-2018_14:21:12  Exp 7312  21 female    R     3   1    17
## 5778 13-May-2018_14:21:15  Exp 7312  21 female    R     3   1    18
## 5779 13-May-2018_14:21:17  Exp 7312  21 female    R     3   1    19
## 5780 13-May-2018_14:21:19  Exp 7312  21 female    R     3   1    20
## 5781 13-May-2018_14:21:22  Exp 7312  21 female    R     3   1    21
## 5782 13-May-2018_14:21:25  Exp 7312  21 female    R     3   1    22
## 5783 13-May-2018_14:21:27  Exp 7312  21 female    R     3   1    23
## 5784 13-May-2018_14:21:29  Exp 7312  21 female    R     3   1    24
## 5785 13-May-2018_14:21:32  Exp 7312  21 female    R     3   2     1
## 5786 13-May-2018_14:21:34  Exp 7312  21 female    R     3   2     2
## 5787 13-May-2018_14:21:36  Exp 7312  21 female    R     3   2     3
## 5788 13-May-2018_14:21:39  Exp 7312  21 female    R     3   2     4
## 5789 13-May-2018_14:21:41  Exp 7312  21 female    R     3   2     5
## 5790 13-May-2018_14:21:43  Exp 7312  21 female    R     3   2     6
## 5791 13-May-2018_14:21:46  Exp 7312  21 female    R     3   2     7
## 5792 13-May-2018_14:21:48  Exp 7312  21 female    R     3   2     8
## 5793 13-May-2018_14:21:51  Exp 7312  21 female    R     3   2     9
## 5794 13-May-2018_14:21:53  Exp 7312  21 female    R     3   2    10
## 5795 13-May-2018_14:21:56  Exp 7312  21 female    R     3   2    11
## 5796 13-May-2018_14:21:58  Exp 7312  21 female    R     3   2    12
## 5797 13-May-2018_14:22:00  Exp 7312  21 female    R     3   2    13
## 5798 13-May-2018_14:22:03  Exp 7312  21 female    R     3   2    14
## 5799 13-May-2018_14:22:05  Exp 7312  21 female    R     3   2    15
## 5800 13-May-2018_14:22:08  Exp 7312  21 female    R     3   2    16
## 5801 13-May-2018_14:22:10  Exp 7312  21 female    R     3   2    17
## 5802 13-May-2018_14:22:12  Exp 7312  21 female    R     3   2    18
## 5803 13-May-2018_14:22:15  Exp 7312  21 female    R     3   2    19
## 5804 13-May-2018_14:22:17  Exp 7312  21 female    R     3   2    20
## 5805 13-May-2018_14:22:20  Exp 7312  21 female    R     3   2    21
## 5806 13-May-2018_14:22:22  Exp 7312  21 female    R     3   2    22
## 5807 13-May-2018_14:22:24  Exp 7312  21 female    R     3   2    23
## 5808 13-May-2018_14:22:27  Exp 7312  21 female    R     3   2    24
## 5809 13-May-2018_14:22:29  Exp 7312  21 female    R     3   3     1
## 5810 13-May-2018_14:22:31  Exp 7312  21 female    R     3   3     2
## 5811 13-May-2018_14:22:34  Exp 7312  21 female    R     3   3     3
## 5812 13-May-2018_14:22:36  Exp 7312  21 female    R     3   3     4
## 5813 13-May-2018_14:22:38  Exp 7312  21 female    R     3   3     5
## 5814 13-May-2018_14:22:41  Exp 7312  21 female    R     3   3     6
## 5815 13-May-2018_14:22:44  Exp 7312  21 female    R     3   3     7
## 5816 13-May-2018_14:22:46  Exp 7312  21 female    R     3   3     8
## 5817 13-May-2018_14:22:48  Exp 7312  21 female    R     3   3     9
## 5818 13-May-2018_14:22:51  Exp 7312  21 female    R     3   3    10
## 5819 13-May-2018_14:22:53  Exp 7312  21 female    R     3   3    11
## 5820 13-May-2018_14:22:55  Exp 7312  21 female    R     3   3    12
## 5821 13-May-2018_14:22:58  Exp 7312  21 female    R     3   3    13
## 5822 13-May-2018_14:23:00  Exp 7312  21 female    R     3   3    14
## 5823 13-May-2018_14:23:03  Exp 7312  21 female    R     3   3    15
## 5824 13-May-2018_14:23:05  Exp 7312  21 female    R     3   3    16
## 5825 13-May-2018_14:23:07  Exp 7312  21 female    R     3   3    17
## 5826 13-May-2018_14:23:10  Exp 7312  21 female    R     3   3    18
## 5827 13-May-2018_14:23:12  Exp 7312  21 female    R     3   3    19
## 5828 13-May-2018_14:23:15  Exp 7312  21 female    R     3   3    20
## 5829 13-May-2018_14:23:17  Exp 7312  21 female    R     3   3    21
## 5830 13-May-2018_14:23:19  Exp 7312  21 female    R     3   3    22
## 5831 13-May-2018_14:23:22  Exp 7312  21 female    R     3   3    23
## 5832 13-May-2018_14:23:24  Exp 7312  21 female    R     3   3    24
## 5833 13-May-2018_14:23:34  Exp 7312  21 female    R     3   4     1
## 5834 13-May-2018_14:23:36  Exp 7312  21 female    R     3   4     2
## 5835 13-May-2018_14:23:39  Exp 7312  21 female    R     3   4     3
## 5836 13-May-2018_14:23:41  Exp 7312  21 female    R     3   4     4
## 5837 13-May-2018_14:23:43  Exp 7312  21 female    R     3   4     5
## 5838 13-May-2018_14:23:46  Exp 7312  21 female    R     3   4     6
## 5839 13-May-2018_14:23:48  Exp 7312  21 female    R     3   4     7
## 5840 13-May-2018_14:23:51  Exp 7312  21 female    R     3   4     8
## 5841 13-May-2018_14:23:53  Exp 7312  21 female    R     3   4     9
## 5842 13-May-2018_14:23:56  Exp 7312  21 female    R     3   4    10
## 5843 13-May-2018_14:23:58  Exp 7312  21 female    R     3   4    11
## 5844 13-May-2018_14:24:01  Exp 7312  21 female    R     3   4    12
## 5845 13-May-2018_14:24:03  Exp 7312  21 female    R     3   4    13
## 5846 13-May-2018_14:24:06  Exp 7312  21 female    R     3   4    14
## 5847 13-May-2018_14:24:09  Exp 7312  21 female    R     3   4    15
## 5848 13-May-2018_14:24:11  Exp 7312  21 female    R     3   4    16
## 5849 13-May-2018_14:24:13  Exp 7312  21 female    R     3   4    17
## 5850 13-May-2018_14:24:16  Exp 7312  21 female    R     3   4    18
## 5851 13-May-2018_14:24:18  Exp 7312  21 female    R     3   4    19
## 5852 13-May-2018_14:24:20  Exp 7312  21 female    R     3   4    20
## 5853 13-May-2018_14:24:23  Exp 7312  21 female    R     3   4    21
## 5854 13-May-2018_14:24:25  Exp 7312  21 female    R     3   4    22
## 5855 13-May-2018_14:24:28  Exp 7312  21 female    R     3   4    23
## 5856 13-May-2018_14:24:30  Exp 7312  21 female    R     3   4    24
## 5857 13-May-2018_14:24:33  Exp 7312  21 female    R     3   5     1
## 5858 13-May-2018_14:24:35  Exp 7312  21 female    R     3   5     2
## 5859 13-May-2018_14:24:38  Exp 7312  21 female    R     3   5     3
## 5860 13-May-2018_14:24:40  Exp 7312  21 female    R     3   5     4
## 5861 13-May-2018_14:24:43  Exp 7312  21 female    R     3   5     5
## 5862 13-May-2018_14:24:45  Exp 7312  21 female    R     3   5     6
## 5863 13-May-2018_14:24:48  Exp 7312  21 female    R     3   5     7
## 5864 13-May-2018_14:24:50  Exp 7312  21 female    R     3   5     8
## 5865 13-May-2018_14:24:52  Exp 7312  21 female    R     3   5     9
## 5866 13-May-2018_14:24:55  Exp 7312  21 female    R     3   5    10
## 5867 13-May-2018_14:24:57  Exp 7312  21 female    R     3   5    11
## 5868 13-May-2018_14:24:59  Exp 7312  21 female    R     3   5    12
## 5869 13-May-2018_14:25:02  Exp 7312  21 female    R     3   5    13
## 5870 13-May-2018_14:25:04  Exp 7312  21 female    R     3   5    14
## 5871 13-May-2018_14:25:06  Exp 7312  21 female    R     3   5    15
## 5872 13-May-2018_14:25:08  Exp 7312  21 female    R     3   5    16
## 5873 13-May-2018_14:25:11  Exp 7312  21 female    R     3   5    17
## 5874 13-May-2018_14:25:13  Exp 7312  21 female    R     3   5    18
## 5875 13-May-2018_14:25:15  Exp 7312  21 female    R     3   5    19
## 5876 13-May-2018_14:25:18  Exp 7312  21 female    R     3   5    20
## 5877 13-May-2018_14:25:20  Exp 7312  21 female    R     3   5    21
## 5878 13-May-2018_14:25:23  Exp 7312  21 female    R     3   5    22
## 5879 13-May-2018_14:25:25  Exp 7312  21 female    R     3   5    23
## 5880 13-May-2018_14:25:27  Exp 7312  21 female    R     3   5    24
## 5881 13-May-2018_14:30:11  Exp 7312  21 female    R     1   1     1
## 5882 13-May-2018_14:30:13  Exp 7312  21 female    R     1   1     2
## 5883 13-May-2018_14:30:15  Exp 7312  21 female    R     1   1     3
## 5884 13-May-2018_14:30:18  Exp 7312  21 female    R     1   1     4
## 5885 13-May-2018_14:30:20  Exp 7312  21 female    R     1   1     5
## 5886 13-May-2018_14:30:22  Exp 7312  21 female    R     1   1     6
## 5887 13-May-2018_14:30:25  Exp 7312  21 female    R     1   1     7
## 5888 13-May-2018_14:30:27  Exp 7312  21 female    R     1   1     8
## 5889 13-May-2018_14:30:30  Exp 7312  21 female    R     1   1     9
## 5890 13-May-2018_14:30:32  Exp 7312  21 female    R     1   1    10
## 5891 13-May-2018_14:30:34  Exp 7312  21 female    R     1   1    11
## 5892 13-May-2018_14:30:37  Exp 7312  21 female    R     1   1    12
## 5893 13-May-2018_14:30:40  Exp 7312  21 female    R     1   1    13
## 5894 13-May-2018_14:30:42  Exp 7312  21 female    R     1   1    14
## 5895 13-May-2018_14:30:45  Exp 7312  21 female    R     1   1    15
## 5896 13-May-2018_14:30:47  Exp 7312  21 female    R     1   1    16
## 5897 13-May-2018_14:30:50  Exp 7312  21 female    R     1   1    17
## 5898 13-May-2018_14:30:52  Exp 7312  21 female    R     1   1    18
## 5899 13-May-2018_14:30:54  Exp 7312  21 female    R     1   1    19
## 5900 13-May-2018_14:30:57  Exp 7312  21 female    R     1   1    20
## 5901 13-May-2018_14:30:59  Exp 7312  21 female    R     1   1    21
## 5902 13-May-2018_14:31:02  Exp 7312  21 female    R     1   1    22
## 5903 13-May-2018_14:31:04  Exp 7312  21 female    R     1   1    23
## 5904 13-May-2018_14:31:07  Exp 7312  21 female    R     1   1    24
## 5905 13-May-2018_14:31:09  Exp 7312  21 female    R     1   2     1
## 5906 13-May-2018_14:31:12  Exp 7312  21 female    R     1   2     2
## 5907 13-May-2018_14:31:14  Exp 7312  21 female    R     1   2     3
## 5908 13-May-2018_14:31:17  Exp 7312  21 female    R     1   2     4
## 5909 13-May-2018_14:31:19  Exp 7312  21 female    R     1   2     5
## 5910 13-May-2018_14:31:21  Exp 7312  21 female    R     1   2     6
## 5911 13-May-2018_14:31:24  Exp 7312  21 female    R     1   2     7
## 5912 13-May-2018_14:31:26  Exp 7312  21 female    R     1   2     8
## 5913 13-May-2018_14:31:29  Exp 7312  21 female    R     1   2     9
## 5914 13-May-2018_14:31:31  Exp 7312  21 female    R     1   2    10
## 5915 13-May-2018_14:31:34  Exp 7312  21 female    R     1   2    11
## 5916 13-May-2018_14:31:36  Exp 7312  21 female    R     1   2    12
## 5917 13-May-2018_14:31:38  Exp 7312  21 female    R     1   2    13
## 5918 13-May-2018_14:31:41  Exp 7312  21 female    R     1   2    14
## 5919 13-May-2018_14:31:43  Exp 7312  21 female    R     1   2    15
## 5920 13-May-2018_14:31:46  Exp 7312  21 female    R     1   2    16
## 5921 13-May-2018_14:31:48  Exp 7312  21 female    R     1   2    17
## 5922 13-May-2018_14:31:50  Exp 7312  21 female    R     1   2    18
## 5923 13-May-2018_14:31:53  Exp 7312  21 female    R     1   2    19
## 5924 13-May-2018_14:31:55  Exp 7312  21 female    R     1   2    20
## 5925 13-May-2018_14:31:58  Exp 7312  21 female    R     1   2    21
## 5926 13-May-2018_14:32:00  Exp 7312  21 female    R     1   2    22
## 5927 13-May-2018_14:32:02  Exp 7312  21 female    R     1   2    23
## 5928 13-May-2018_14:32:05  Exp 7312  21 female    R     1   2    24
## 5929 13-May-2018_14:36:34  Exp 7312  21 female    R     1   1     1
## 5930 13-May-2018_14:36:36  Exp 7312  21 female    R     1   1     2
## 5931 13-May-2018_14:36:39  Exp 7312  21 female    R     1   1     3
## 5932 13-May-2018_14:36:41  Exp 7312  21 female    R     1   1     4
## 5933 13-May-2018_14:36:44  Exp 7312  21 female    R     1   1     5
## 5934 13-May-2018_14:36:46  Exp 7312  21 female    R     1   1     6
## 5935 13-May-2018_14:36:48  Exp 7312  21 female    R     1   1     7
## 5936 13-May-2018_14:36:51  Exp 7312  21 female    R     1   1     8
## 5937 13-May-2018_14:36:53  Exp 7312  21 female    R     1   1     9
## 5938 13-May-2018_14:36:56  Exp 7312  21 female    R     1   1    10
## 5939 13-May-2018_14:36:58  Exp 7312  21 female    R     1   1    11
## 5940 13-May-2018_14:37:00  Exp 7312  21 female    R     1   1    12
## 5941 13-May-2018_14:37:02  Exp 7312  21 female    R     1   1    13
## 5942 13-May-2018_14:37:05  Exp 7312  21 female    R     1   1    14
## 5943 13-May-2018_14:37:07  Exp 7312  21 female    R     1   1    15
## 5944 13-May-2018_14:37:09  Exp 7312  21 female    R     1   1    16
## 5945 13-May-2018_14:37:12  Exp 7312  21 female    R     1   1    17
## 5946 13-May-2018_14:37:14  Exp 7312  21 female    R     1   1    18
## 5947 13-May-2018_14:37:17  Exp 7312  21 female    R     1   1    19
## 5948 13-May-2018_14:37:19  Exp 7312  21 female    R     1   1    20
## 5949 13-May-2018_14:37:22  Exp 7312  21 female    R     1   1    21
## 5950 13-May-2018_14:37:24  Exp 7312  21 female    R     1   1    22
## 5951 13-May-2018_14:37:26  Exp 7312  21 female    R     1   1    23
## 5952 13-May-2018_14:37:29  Exp 7312  21 female    R     1   1    24
## 5953 13-May-2018_14:37:31  Exp 7312  21 female    R     1   2     1
## 5954 13-May-2018_14:37:34  Exp 7312  21 female    R     1   2     2
## 5955 13-May-2018_14:37:36  Exp 7312  21 female    R     1   2     3
## 5956 13-May-2018_14:37:38  Exp 7312  21 female    R     1   2     4
## 5957 13-May-2018_14:37:41  Exp 7312  21 female    R     1   2     5
## 5958 13-May-2018_14:37:43  Exp 7312  21 female    R     1   2     6
## 5959 13-May-2018_14:37:45  Exp 7312  21 female    R     1   2     7
## 5960 13-May-2018_14:37:48  Exp 7312  21 female    R     1   2     8
## 5961 13-May-2018_14:37:50  Exp 7312  21 female    R     1   2     9
## 5962 13-May-2018_14:37:52  Exp 7312  21 female    R     1   2    10
## 5963 13-May-2018_14:37:55  Exp 7312  21 female    R     1   2    11
## 5964 13-May-2018_14:37:57  Exp 7312  21 female    R     1   2    12
## 5965 13-May-2018_14:37:59  Exp 7312  21 female    R     1   2    13
## 5966 13-May-2018_14:38:02  Exp 7312  21 female    R     1   2    14
## 5967 13-May-2018_14:38:04  Exp 7312  21 female    R     1   2    15
## 5968 13-May-2018_14:38:07  Exp 7312  21 female    R     1   2    16
## 5969 13-May-2018_14:38:09  Exp 7312  21 female    R     1   2    17
## 5970 13-May-2018_14:38:11  Exp 7312  21 female    R     1   2    18
## 5971 13-May-2018_14:38:14  Exp 7312  21 female    R     1   2    19
## 5972 13-May-2018_14:38:16  Exp 7312  21 female    R     1   2    20
## 5973 13-May-2018_14:38:18  Exp 7312  21 female    R     1   2    21
## 5974 13-May-2018_14:38:20  Exp 7312  21 female    R     1   2    22
## 5975 13-May-2018_14:38:23  Exp 7312  21 female    R     1   2    23
## 5976 13-May-2018_14:38:25  Exp 7312  21 female    R     1   2    24
## 5977 13-May-2018_14:42:40  Exp 7312  21 female    R     1   1     1
## 5978 13-May-2018_14:42:43  Exp 7312  21 female    R     1   1     2
## 5979 13-May-2018_14:42:45  Exp 7312  21 female    R     1   1     3
## 5980 13-May-2018_14:42:47  Exp 7312  21 female    R     1   1     4
## 5981 13-May-2018_14:42:50  Exp 7312  21 female    R     1   1     5
## 5982 13-May-2018_14:42:52  Exp 7312  21 female    R     1   1     6
## 5983 13-May-2018_14:42:54  Exp 7312  21 female    R     1   1     7
## 5984 13-May-2018_14:42:57  Exp 7312  21 female    R     1   1     8
## 5985 13-May-2018_14:42:59  Exp 7312  21 female    R     1   1     9
## 5986 13-May-2018_14:43:01  Exp 7312  21 female    R     1   1    10
## 5987 13-May-2018_14:43:03  Exp 7312  21 female    R     1   1    11
## 5988 13-May-2018_14:43:06  Exp 7312  21 female    R     1   1    12
## 5989 13-May-2018_14:43:08  Exp 7312  21 female    R     1   1    13
## 5990 13-May-2018_14:43:10  Exp 7312  21 female    R     1   1    14
## 5991 13-May-2018_14:43:13  Exp 7312  21 female    R     1   1    15
## 5992 13-May-2018_14:43:15  Exp 7312  21 female    R     1   1    16
## 5993 13-May-2018_14:43:17  Exp 7312  21 female    R     1   1    17
## 5994 13-May-2018_14:43:20  Exp 7312  21 female    R     1   1    18
## 5995 13-May-2018_14:43:22  Exp 7312  21 female    R     1   1    19
## 5996 13-May-2018_14:43:24  Exp 7312  21 female    R     1   1    20
## 5997 13-May-2018_14:43:26  Exp 7312  21 female    R     1   1    21
## 5998 13-May-2018_14:43:29  Exp 7312  21 female    R     1   1    22
## 5999 13-May-2018_14:43:31  Exp 7312  21 female    R     1   1    23
## 6000 13-May-2018_14:43:33  Exp 7312  21 female    R     1   1    24
## 6001 13-May-2018_14:43:35  Exp 7312  21 female    R     1   2     1
## 6002 13-May-2018_14:43:38  Exp 7312  21 female    R     1   2     2
## 6003 13-May-2018_14:43:40  Exp 7312  21 female    R     1   2     3
## 6004 13-May-2018_14:43:42  Exp 7312  21 female    R     1   2     4
## 6005 13-May-2018_14:43:44  Exp 7312  21 female    R     1   2     5
## 6006 13-May-2018_14:43:46  Exp 7312  21 female    R     1   2     6
## 6007 13-May-2018_14:43:49  Exp 7312  21 female    R     1   2     7
## 6008 13-May-2018_14:43:51  Exp 7312  21 female    R     1   2     8
## 6009 13-May-2018_14:43:53  Exp 7312  21 female    R     1   2     9
## 6010 13-May-2018_14:43:56  Exp 7312  21 female    R     1   2    10
## 6011 13-May-2018_14:43:58  Exp 7312  21 female    R     1   2    11
## 6012 13-May-2018_14:44:01  Exp 7312  21 female    R     1   2    12
## 6013 13-May-2018_14:44:03  Exp 7312  21 female    R     1   2    13
## 6014 13-May-2018_14:44:05  Exp 7312  21 female    R     1   2    14
## 6015 13-May-2018_14:44:08  Exp 7312  21 female    R     1   2    15
## 6016 13-May-2018_14:44:10  Exp 7312  21 female    R     1   2    16
## 6017 13-May-2018_14:44:12  Exp 7312  21 female    R     1   2    17
## 6018 13-May-2018_14:44:15  Exp 7312  21 female    R     1   2    18
## 6019 13-May-2018_14:44:17  Exp 7312  21 female    R     1   2    19
## 6020 13-May-2018_14:44:19  Exp 7312  21 female    R     1   2    20
## 6021 13-May-2018_14:44:22  Exp 7312  21 female    R     1   2    21
## 6022 13-May-2018_14:44:24  Exp 7312  21 female    R     1   2    22
## 6023 13-May-2018_14:44:26  Exp 7312  21 female    R     1   2    23
## 6024 13-May-2018_14:44:29  Exp 7312  21 female    R     1   2    24
## 6025 13-May-2018_14:48:35  Exp 7312  21 female    R     1   1     1
## 6026 13-May-2018_14:48:38  Exp 7312  21 female    R     1   1     2
## 6027 13-May-2018_14:48:41  Exp 7312  21 female    R     1   1     3
## 6028 13-May-2018_14:48:43  Exp 7312  21 female    R     1   1     4
## 6029 13-May-2018_14:48:46  Exp 7312  21 female    R     1   1     5
## 6030 13-May-2018_14:48:48  Exp 7312  21 female    R     1   1     6
## 6031 13-May-2018_14:48:50  Exp 7312  21 female    R     1   1     7
## 6032 13-May-2018_14:48:52  Exp 7312  21 female    R     1   1     8
## 6033 13-May-2018_14:48:55  Exp 7312  21 female    R     1   1     9
## 6034 13-May-2018_14:48:57  Exp 7312  21 female    R     1   1    10
## 6035 13-May-2018_14:49:00  Exp 7312  21 female    R     1   1    11
## 6036 13-May-2018_14:49:02  Exp 7312  21 female    R     1   1    12
## 6037 13-May-2018_14:49:04  Exp 7312  21 female    R     1   1    13
## 6038 13-May-2018_14:49:06  Exp 7312  21 female    R     1   1    14
## 6039 13-May-2018_14:49:09  Exp 7312  21 female    R     1   1    15
## 6040 13-May-2018_14:49:11  Exp 7312  21 female    R     1   1    16
## 6041 13-May-2018_14:49:13  Exp 7312  21 female    R     1   1    17
## 6042 13-May-2018_14:49:16  Exp 7312  21 female    R     1   1    18
## 6043 13-May-2018_14:49:18  Exp 7312  21 female    R     1   1    19
## 6044 13-May-2018_14:49:20  Exp 7312  21 female    R     1   1    20
## 6045 13-May-2018_14:49:23  Exp 7312  21 female    R     1   1    21
## 6046 13-May-2018_14:49:25  Exp 7312  21 female    R     1   1    22
## 6047 13-May-2018_14:49:27  Exp 7312  21 female    R     1   1    23
## 6048 13-May-2018_14:49:30  Exp 7312  21 female    R     1   1    24
## 6049 13-May-2018_14:49:32  Exp 7312  21 female    R     1   2     1
## 6050 13-May-2018_14:49:35  Exp 7312  21 female    R     1   2     2
## 6051 13-May-2018_14:49:37  Exp 7312  21 female    R     1   2     3
## 6052 13-May-2018_14:49:40  Exp 7312  21 female    R     1   2     4
## 6053 13-May-2018_14:49:42  Exp 7312  21 female    R     1   2     5
## 6054 13-May-2018_14:49:44  Exp 7312  21 female    R     1   2     6
## 6055 13-May-2018_14:49:47  Exp 7312  21 female    R     1   2     7
## 6056 13-May-2018_14:49:49  Exp 7312  21 female    R     1   2     8
## 6057 13-May-2018_14:49:52  Exp 7312  21 female    R     1   2     9
## 6058 13-May-2018_14:49:54  Exp 7312  21 female    R     1   2    10
## 6059 13-May-2018_14:49:56  Exp 7312  21 female    R     1   2    11
## 6060 13-May-2018_14:49:58  Exp 7312  21 female    R     1   2    12
## 6061 13-May-2018_14:50:01  Exp 7312  21 female    R     1   2    13
## 6062 13-May-2018_14:50:03  Exp 7312  21 female    R     1   2    14
## 6063 13-May-2018_14:50:06  Exp 7312  21 female    R     1   2    15
## 6064 13-May-2018_14:50:08  Exp 7312  21 female    R     1   2    16
## 6065 13-May-2018_14:50:10  Exp 7312  21 female    R     1   2    17
## 6066 13-May-2018_14:50:13  Exp 7312  21 female    R     1   2    18
## 6067 13-May-2018_14:50:15  Exp 7312  21 female    R     1   2    19
## 6068 13-May-2018_14:50:18  Exp 7312  21 female    R     1   2    20
## 6069 13-May-2018_14:50:20  Exp 7312  21 female    R     1   2    21
## 6070 13-May-2018_14:50:22  Exp 7312  21 female    R     1   2    22
## 6071 13-May-2018_14:50:25  Exp 7312  21 female    R     1   2    23
## 6072 13-May-2018_14:50:27  Exp 7312  21 female    R     1   2    24
## 6073 13-May-2018_14:54:27  Exp 7312  21 female    R     1   1     1
## 6074 13-May-2018_14:54:30  Exp 7312  21 female    R     1   1     2
## 6075 13-May-2018_14:54:32  Exp 7312  21 female    R     1   1     3
## 6076 13-May-2018_14:54:35  Exp 7312  21 female    R     1   1     4
## 6077 13-May-2018_14:54:37  Exp 7312  21 female    R     1   1     5
## 6078 13-May-2018_14:54:39  Exp 7312  21 female    R     1   1     6
## 6079 13-May-2018_14:54:41  Exp 7312  21 female    R     1   1     7
## 6080 13-May-2018_14:54:44  Exp 7312  21 female    R     1   1     8
## 6081 13-May-2018_14:54:46  Exp 7312  21 female    R     1   1     9
## 6082 13-May-2018_14:54:48  Exp 7312  21 female    R     1   1    10
## 6083 13-May-2018_14:54:51  Exp 7312  21 female    R     1   1    11
## 6084 13-May-2018_14:54:53  Exp 7312  21 female    R     1   1    12
## 6085 13-May-2018_14:54:55  Exp 7312  21 female    R     1   1    13
## 6086 13-May-2018_14:54:58  Exp 7312  21 female    R     1   1    14
## 6087 13-May-2018_14:55:00  Exp 7312  21 female    R     1   1    15
## 6088 13-May-2018_14:55:03  Exp 7312  21 female    R     1   1    16
## 6089 13-May-2018_14:55:05  Exp 7312  21 female    R     1   1    17
## 6090 13-May-2018_14:55:07  Exp 7312  21 female    R     1   1    18
## 6091 13-May-2018_14:55:10  Exp 7312  21 female    R     1   1    19
## 6092 13-May-2018_14:55:12  Exp 7312  21 female    R     1   1    20
## 6093 13-May-2018_14:55:14  Exp 7312  21 female    R     1   1    21
## 6094 13-May-2018_14:55:17  Exp 7312  21 female    R     1   1    22
## 6095 13-May-2018_14:55:19  Exp 7312  21 female    R     1   1    23
## 6096 13-May-2018_14:55:22  Exp 7312  21 female    R     1   1    24
## 6097 13-May-2018_14:55:24  Exp 7312  21 female    R     1   2     1
## 6098 13-May-2018_14:55:26  Exp 7312  21 female    R     1   2     2
## 6099 13-May-2018_14:55:29  Exp 7312  21 female    R     1   2     3
## 6100 13-May-2018_14:55:31  Exp 7312  21 female    R     1   2     4
## 6101 13-May-2018_14:55:33  Exp 7312  21 female    R     1   2     5
## 6102 13-May-2018_14:55:36  Exp 7312  21 female    R     1   2     6
## 6103 13-May-2018_14:55:38  Exp 7312  21 female    R     1   2     7
## 6104 13-May-2018_14:55:41  Exp 7312  21 female    R     1   2     8
## 6105 13-May-2018_14:55:43  Exp 7312  21 female    R     1   2     9
## 6106 13-May-2018_14:55:46  Exp 7312  21 female    R     1   2    10
## 6107 13-May-2018_14:55:48  Exp 7312  21 female    R     1   2    11
## 6108 13-May-2018_14:55:50  Exp 7312  21 female    R     1   2    12
## 6109 13-May-2018_14:55:52  Exp 7312  21 female    R     1   2    13
## 6110 13-May-2018_14:55:55  Exp 7312  21 female    R     1   2    14
## 6111 13-May-2018_14:55:57  Exp 7312  21 female    R     1   2    15
## 6112 13-May-2018_14:56:00  Exp 7312  21 female    R     1   2    16
## 6113 13-May-2018_14:56:02  Exp 7312  21 female    R     1   2    17
## 6114 13-May-2018_14:56:05  Exp 7312  21 female    R     1   2    18
## 6115 13-May-2018_14:56:07  Exp 7312  21 female    R     1   2    19
## 6116 13-May-2018_14:56:10  Exp 7312  21 female    R     1   2    20
## 6117 13-May-2018_14:56:12  Exp 7312  21 female    R     1   2    21
## 6118 13-May-2018_14:56:14  Exp 7312  21 female    R     1   2    22
## 6119 13-May-2018_14:56:16  Exp 7312  21 female    R     1   2    23
## 6120 13-May-2018_14:56:19  Exp 7312  21 female    R     1   2    24
## 6121 13-May-2018_16:07:12  Exp 7313  20 female    R     1   1     1
## 6122 13-May-2018_16:07:14  Exp 7313  20 female    R     1   1     2
## 6123 13-May-2018_16:07:16  Exp 7313  20 female    R     1   1     3
## 6124 13-May-2018_16:07:18  Exp 7313  20 female    R     1   1     4
## 6125 13-May-2018_16:07:20  Exp 7313  20 female    R     1   1     5
## 6126 13-May-2018_16:07:21  Exp 7313  20 female    R     1   1     6
## 6127 13-May-2018_16:07:23  Exp 7313  20 female    R     1   1     7
## 6128 13-May-2018_16:07:25  Exp 7313  20 female    R     1   1     8
## 6129 13-May-2018_16:07:27  Exp 7313  20 female    R     1   1     9
## 6130 13-May-2018_16:07:29  Exp 7313  20 female    R     1   1    10
## 6131 13-May-2018_16:07:31  Exp 7313  20 female    R     1   1    11
## 6132 13-May-2018_16:07:33  Exp 7313  20 female    R     1   1    12
## 6133 13-May-2018_16:07:35  Exp 7313  20 female    R     1   1    13
## 6134 13-May-2018_16:07:37  Exp 7313  20 female    R     1   1    14
## 6135 13-May-2018_16:07:38  Exp 7313  20 female    R     1   1    15
## 6136 13-May-2018_16:07:40  Exp 7313  20 female    R     1   1    16
## 6137 13-May-2018_16:07:42  Exp 7313  20 female    R     1   1    17
## 6138 13-May-2018_16:07:44  Exp 7313  20 female    R     1   1    18
## 6139 13-May-2018_16:07:46  Exp 7313  20 female    R     1   1    19
## 6140 13-May-2018_16:07:47  Exp 7313  20 female    R     1   1    20
## 6141 13-May-2018_16:07:49  Exp 7313  20 female    R     1   1    21
## 6142 13-May-2018_16:07:51  Exp 7313  20 female    R     1   1    22
## 6143 13-May-2018_16:07:53  Exp 7313  20 female    R     1   1    23
## 6144 13-May-2018_16:07:55  Exp 7313  20 female    R     1   1    24
## 6145 13-May-2018_16:07:56  Exp 7313  20 female    R     1   2     1
## 6146 13-May-2018_16:07:58  Exp 7313  20 female    R     1   2     2
## 6147 13-May-2018_16:08:00  Exp 7313  20 female    R     1   2     3
## 6148 13-May-2018_16:08:02  Exp 7313  20 female    R     1   2     4
## 6149 13-May-2018_16:08:04  Exp 7313  20 female    R     1   2     5
## 6150 13-May-2018_16:08:05  Exp 7313  20 female    R     1   2     6
## 6151 13-May-2018_16:08:07  Exp 7313  20 female    R     1   2     7
## 6152 13-May-2018_16:08:09  Exp 7313  20 female    R     1   2     8
## 6153 13-May-2018_16:08:11  Exp 7313  20 female    R     1   2     9
## 6154 13-May-2018_16:08:12  Exp 7313  20 female    R     1   2    10
## 6155 13-May-2018_16:08:14  Exp 7313  20 female    R     1   2    11
## 6156 13-May-2018_16:08:16  Exp 7313  20 female    R     1   2    12
## 6157 13-May-2018_16:08:19  Exp 7313  20 female    R     1   2    13
## 6158 13-May-2018_16:08:21  Exp 7313  20 female    R     1   2    14
## 6159 13-May-2018_16:08:22  Exp 7313  20 female    R     1   2    15
## 6160 13-May-2018_16:08:24  Exp 7313  20 female    R     1   2    16
## 6161 13-May-2018_16:08:26  Exp 7313  20 female    R     1   2    17
## 6162 13-May-2018_16:08:28  Exp 7313  20 female    R     1   2    18
## 6163 13-May-2018_16:08:30  Exp 7313  20 female    R     1   2    19
## 6164 13-May-2018_16:08:31  Exp 7313  20 female    R     1   2    20
## 6165 13-May-2018_16:08:33  Exp 7313  20 female    R     1   2    21
## 6166 13-May-2018_16:08:36  Exp 7313  20 female    R     1   2    22
## 6167 13-May-2018_16:08:38  Exp 7313  20 female    R     1   2    23
## 6168 13-May-2018_16:08:40  Exp 7313  20 female    R     1   2    24
## 6169 13-May-2018_16:08:41  Exp 7313  20 female    R     1   3     1
## 6170 13-May-2018_16:08:43  Exp 7313  20 female    R     1   3     2
## 6171 13-May-2018_16:08:45  Exp 7313  20 female    R     1   3     3
## 6172 13-May-2018_16:08:47  Exp 7313  20 female    R     1   3     4
## 6173 13-May-2018_16:08:49  Exp 7313  20 female    R     1   3     5
## 6174 13-May-2018_16:08:51  Exp 7313  20 female    R     1   3     6
## 6175 13-May-2018_16:08:53  Exp 7313  20 female    R     1   3     7
## 6176 13-May-2018_16:08:55  Exp 7313  20 female    R     1   3     8
## 6177 13-May-2018_16:08:57  Exp 7313  20 female    R     1   3     9
## 6178 13-May-2018_16:08:58  Exp 7313  20 female    R     1   3    10
## 6179 13-May-2018_16:09:00  Exp 7313  20 female    R     1   3    11
## 6180 13-May-2018_16:09:02  Exp 7313  20 female    R     1   3    12
## 6181 13-May-2018_16:09:04  Exp 7313  20 female    R     1   3    13
## 6182 13-May-2018_16:09:06  Exp 7313  20 female    R     1   3    14
## 6183 13-May-2018_16:09:07  Exp 7313  20 female    R     1   3    15
## 6184 13-May-2018_16:09:09  Exp 7313  20 female    R     1   3    16
## 6185 13-May-2018_16:09:11  Exp 7313  20 female    R     1   3    17
## 6186 13-May-2018_16:09:13  Exp 7313  20 female    R     1   3    18
## 6187 13-May-2018_16:09:15  Exp 7313  20 female    R     1   3    19
## 6188 13-May-2018_16:09:17  Exp 7313  20 female    R     1   3    20
## 6189 13-May-2018_16:09:19  Exp 7313  20 female    R     1   3    21
## 6190 13-May-2018_16:09:21  Exp 7313  20 female    R     1   3    22
## 6191 13-May-2018_16:09:23  Exp 7313  20 female    R     1   3    23
## 6192 13-May-2018_16:09:25  Exp 7313  20 female    R     1   3    24
## 6193 13-May-2018_16:09:52  Exp 7313  20 female    R     1   4     1
## 6194 13-May-2018_16:09:54  Exp 7313  20 female    R     1   4     2
## 6195 13-May-2018_16:09:56  Exp 7313  20 female    R     1   4     3
## 6196 13-May-2018_16:09:58  Exp 7313  20 female    R     1   4     4
## 6197 13-May-2018_16:10:00  Exp 7313  20 female    R     1   4     5
## 6198 13-May-2018_16:10:02  Exp 7313  20 female    R     1   4     6
## 6199 13-May-2018_16:10:04  Exp 7313  20 female    R     1   4     7
## 6200 13-May-2018_16:10:06  Exp 7313  20 female    R     1   4     8
## 6201 13-May-2018_16:10:08  Exp 7313  20 female    R     1   4     9
## 6202 13-May-2018_16:10:10  Exp 7313  20 female    R     1   4    10
## 6203 13-May-2018_16:10:12  Exp 7313  20 female    R     1   4    11
## 6204 13-May-2018_16:10:14  Exp 7313  20 female    R     1   4    12
## 6205 13-May-2018_16:10:16  Exp 7313  20 female    R     1   4    13
## 6206 13-May-2018_16:10:17  Exp 7313  20 female    R     1   4    14
## 6207 13-May-2018_16:10:19  Exp 7313  20 female    R     1   4    15
## 6208 13-May-2018_16:10:21  Exp 7313  20 female    R     1   4    16
## 6209 13-May-2018_16:10:23  Exp 7313  20 female    R     1   4    17
## 6210 13-May-2018_16:10:25  Exp 7313  20 female    R     1   4    18
## 6211 13-May-2018_16:10:27  Exp 7313  20 female    R     1   4    19
## 6212 13-May-2018_16:10:29  Exp 7313  20 female    R     1   4    20
## 6213 13-May-2018_16:10:31  Exp 7313  20 female    R     1   4    21
## 6214 13-May-2018_16:10:33  Exp 7313  20 female    R     1   4    22
## 6215 13-May-2018_16:10:35  Exp 7313  20 female    R     1   4    23
## 6216 13-May-2018_16:10:37  Exp 7313  20 female    R     1   4    24
## 6217 13-May-2018_16:10:39  Exp 7313  20 female    R     1   5     1
## 6218 13-May-2018_16:10:41  Exp 7313  20 female    R     1   5     2
## 6219 13-May-2018_16:10:43  Exp 7313  20 female    R     1   5     3
## 6220 13-May-2018_16:10:45  Exp 7313  20 female    R     1   5     4
## 6221 13-May-2018_16:10:47  Exp 7313  20 female    R     1   5     5
## 6222 13-May-2018_16:10:49  Exp 7313  20 female    R     1   5     6
## 6223 13-May-2018_16:10:51  Exp 7313  20 female    R     1   5     7
## 6224 13-May-2018_16:10:53  Exp 7313  20 female    R     1   5     8
## 6225 13-May-2018_16:10:55  Exp 7313  20 female    R     1   5     9
## 6226 13-May-2018_16:10:57  Exp 7313  20 female    R     1   5    10
## 6227 13-May-2018_16:10:59  Exp 7313  20 female    R     1   5    11
## 6228 13-May-2018_16:11:01  Exp 7313  20 female    R     1   5    12
## 6229 13-May-2018_16:11:03  Exp 7313  20 female    R     1   5    13
## 6230 13-May-2018_16:11:05  Exp 7313  20 female    R     1   5    14
## 6231 13-May-2018_16:11:07  Exp 7313  20 female    R     1   5    15
## 6232 13-May-2018_16:11:09  Exp 7313  20 female    R     1   5    16
## 6233 13-May-2018_16:11:11  Exp 7313  20 female    R     1   5    17
## 6234 13-May-2018_16:11:14  Exp 7313  20 female    R     1   5    18
## 6235 13-May-2018_16:11:16  Exp 7313  20 female    R     1   5    19
## 6236 13-May-2018_16:11:18  Exp 7313  20 female    R     1   5    20
## 6237 13-May-2018_16:11:20  Exp 7313  20 female    R     1   5    21
## 6238 13-May-2018_16:11:22  Exp 7313  20 female    R     1   5    22
## 6239 13-May-2018_16:11:24  Exp 7313  20 female    R     1   5    23
## 6240 13-May-2018_16:11:26  Exp 7313  20 female    R     1   5    24
## 6241 13-May-2018_16:11:31  Exp 7313  20 female    R     2   1     1
## 6242 13-May-2018_16:11:33  Exp 7313  20 female    R     2   1     2
## 6243 13-May-2018_16:11:35  Exp 7313  20 female    R     2   1     3
## 6244 13-May-2018_16:11:37  Exp 7313  20 female    R     2   1     4
## 6245 13-May-2018_16:11:39  Exp 7313  20 female    R     2   1     5
## 6246 13-May-2018_16:11:41  Exp 7313  20 female    R     2   1     6
## 6247 13-May-2018_16:11:43  Exp 7313  20 female    R     2   1     7
## 6248 13-May-2018_16:11:45  Exp 7313  20 female    R     2   1     8
## 6249 13-May-2018_16:11:47  Exp 7313  20 female    R     2   1     9
##             Shape        Label    Match CorrResp  Resp ACC     RT
## 1     immoralSelf  immoralSelf mismatch        n     m   0 0.7561
## 2      moralOther   moralOther mismatch        n     n   1 0.7043
## 3    immoralOther immoralOther mismatch        n     n   1 0.9903
## 4       moralSelf    moralSelf mismatch        n  <NA>  -1 1.0420
## 5     immoralSelf  immoralSelf    match        m     m   1 0.8207
## 6     immoralSelf  immoralSelf    match        m     m   1 0.7547
## 7      moralOther   moralOther    match        m     m   1 0.5429
## 8       moralSelf    moralSelf    match        m     m   1 0.9009
## 9      moralOther   moralOther mismatch        n     n   1 0.9551
## 10    immoralSelf  immoralSelf mismatch        n     n   1 0.6952
## 11     moralOther   moralOther mismatch        n     n   1 0.7593
## 12   immoralOther immoralOther    match        m     m   1 0.7135
## 13     moralOther   moralOther    match        m     m   1 0.5656
## 14      moralSelf    moralSelf mismatch        n     m   0 0.5357
## 15   immoralOther immoralOther    match        m     m   1 0.8078
## 16    immoralSelf  immoralSelf mismatch        n     n   1 0.9600
## 17      moralSelf    moralSelf mismatch        n     n   1 0.6661
## 18      moralSelf    moralSelf    match        m     m   1 0.6962
## 19    immoralSelf  immoralSelf    match        m     n   0 0.8803
## 20     moralOther   moralOther    match        m     m   1 0.5785
## 21   immoralOther immoralOther mismatch        n     n   1 0.7845
## 22      moralSelf    moralSelf    match        m     m   1 0.8146
## 23   immoralOther immoralOther    match        m     m   1 0.6548
## 24   immoralOther immoralOther mismatch        n     n   1 0.8789
## 25      moralSelf    moralSelf    match        m     m   1 0.7131
## 26      moralSelf    moralSelf mismatch        n     m   0 0.8211
## 27   immoralOther immoralOther    match        m     m   1 0.8033
## 28   immoralOther immoralOther mismatch        n     m   0 0.6294
## 29    immoralSelf  immoralSelf mismatch        n     n   1 0.8095
## 30   immoralOther immoralOther    match        m     m   1 0.6176
## 31      moralSelf    moralSelf    match        m     m   1 0.7917
## 32     moralOther   moralOther    match        m     m   1 0.5559
## 33    immoralSelf  immoralSelf mismatch        n     m   0 0.9600
## 34     moralOther   moralOther    match        m     m   1 0.5381
## 35    immoralSelf  immoralSelf mismatch        n  <NA>  -1 1.0420
## 36      moralSelf    moralSelf mismatch        n     m   0 0.8264
## 37    immoralSelf  immoralSelf    match        m     m   1 0.7125
## 38     moralOther   moralOther mismatch        n     m   0 0.4609
## 39     moralOther   moralOther mismatch        n     n   1 0.8027
## 40    immoralSelf  immoralSelf    match        m     m   1 0.7808
## 41     moralOther   moralOther mismatch        n     n   1 0.8749
## 42      moralSelf    moralSelf mismatch        n     m   0 0.8710
## 43      moralSelf    moralSelf    match        m     m   1 0.6512
## 44   immoralOther immoralOther mismatch        n     n   1 0.7554
## 45   immoralOther immoralOther    match        m     m   1 0.6715
## 46    immoralSelf  immoralSelf    match        m     m   1 0.9076
## 47   immoralOther immoralOther mismatch        n     n   1 0.5997
## 48     moralOther   moralOther    match        m     m   1 0.5218
## 49   immoralOther immoralOther    match        m     m   1 0.7319
## 50     moralOther   moralOther mismatch        n     n   1 0.7640
## 51      moralSelf    moralSelf mismatch        n  <NA>  -1 1.0420
## 52    immoralSelf  immoralSelf mismatch        n     n   1 0.7443
## 53     moralOther   moralOther    match        m     m   1 0.5104
## 54     moralOther   moralOther    match        m     m   1 0.7706
## 55   immoralOther immoralOther mismatch        n     n   1 0.6026
## 56      moralSelf    moralSelf mismatch        n     n   1 0.7648
## 57   immoralOther immoralOther    match        m     m   1 0.7109
## 58      moralSelf    moralSelf mismatch        n     n   1 0.8270
## 59    immoralSelf  immoralSelf    match        m     m   1 0.8571
## 60      moralSelf    moralSelf    match        m     n   0 0.8092
## 61    immoralSelf  immoralSelf    match        m     m   1 0.7213
## 62   immoralOther immoralOther mismatch        n     n   1 0.8195
## 63    immoralSelf  immoralSelf mismatch        n     n   1 0.8896
## 64     moralOther   moralOther    match        m     m   1 0.5778
## 65    immoralSelf  immoralSelf    match        m     n   0 0.8799
## 66     moralOther   moralOther mismatch        n     n   1 0.6660
## 67    immoralSelf  immoralSelf mismatch        n     n   1 1.0081
## 68   immoralOther immoralOther mismatch        n     n   1 0.8562
## 69     moralOther   moralOther mismatch        n     n   1 0.5444
## 70      moralSelf    moralSelf    match        m     m   1 0.6625
## 71   immoralOther immoralOther    match        m     n   0 0.6846
## 72      moralSelf    moralSelf    match        m     m   1 0.9027
## 73    immoralSelf  immoralSelf    match        m     m   1 0.8236
## 74     moralOther   moralOther mismatch        n     n   1 0.6137
## 75      moralSelf    moralSelf mismatch        n     m   0 0.7338
## 76      moralSelf    moralSelf    match        m     n   0 0.7419
## 77   immoralOther immoralOther    match        m     m   1 0.6840
## 78   immoralOther immoralOther    match        m     m   1 0.6062
## 79    immoralSelf  immoralSelf mismatch        n     n   1 0.7842
## 80     moralOther   moralOther    match        m     m   1 0.5824
## 81     moralOther   moralOther    match        m     m   1 0.4126
## 82      moralSelf    moralSelf    match        m     m   1 0.5006
## 83    immoralSelf  immoralSelf mismatch        n     n   1 0.8507
## 84      moralSelf    moralSelf mismatch        n     n   1 0.9069
## 85     moralOther   moralOther mismatch        n     n   1 0.7210
## 86   immoralOther immoralOther mismatch        n     n   1 0.6351
## 87   immoralOther immoralOther mismatch        n     n   1 0.8553
## 88    immoralSelf  immoralSelf mismatch        n     n   1 0.7194
## 89      moralSelf    moralSelf mismatch        n     m   0 0.5214
## 90      moralSelf    moralSelf    match        m     m   1 0.6597
## 91    immoralSelf  immoralSelf    match        m     m   1 0.8097
## 92   immoralOther immoralOther mismatch        n     n   1 0.6738
## 93     moralOther   moralOther    match        m     m   1 0.5340
## 94     moralOther   moralOther mismatch        n     n   1 0.6141
## 95   immoralOther immoralOther    match        m     m   1 0.6462
## 96    immoralSelf  immoralSelf    match        m     m   1 0.7763
## 97     moralOther   moralOther mismatch        n     n   1 0.6964
## 98     moralOther   moralOther    match        m     n   0 0.8805
## 99   immoralOther immoralOther    match        m     m   1 0.5546
## 100     moralSelf    moralSelf    match        m     m   1 0.9747
## 101   immoralSelf  immoralSelf    match        m     m   1 0.6149
## 102    moralOther   moralOther    match        m     m   1 0.6150
## 103  immoralOther immoralOther mismatch        n     m   0 0.6552
## 104  immoralOther immoralOther    match        m     m   1 0.6552
## 105  immoralOther immoralOther mismatch        n     n   1 0.5994
## 106     moralSelf    moralSelf mismatch        n     m   0 0.6495
## 107    moralOther   moralOther mismatch        n     n   1 0.6736
## 108  immoralOther immoralOther mismatch        n     n   1 0.7737
## 109   immoralSelf  immoralSelf mismatch        n     n   1 0.7338
## 110     moralSelf    moralSelf mismatch        n     n   1 0.7340
## 111   immoralSelf  immoralSelf    match        m     m   1 0.6221
## 112   immoralSelf  immoralSelf    match        m     m   1 0.5482
## 113   immoralSelf  immoralSelf mismatch        n     n   1 0.5923
## 114    moralOther   moralOther mismatch        n     n   1 0.6445
## 115   immoralSelf  immoralSelf mismatch        n     n   1 0.4385
## 116  immoralOther immoralOther    match        m     m   1 0.7766
## 117     moralSelf    moralSelf    match        m     m   1 0.7527
## 118     moralSelf    moralSelf    match        m     m   1 0.5169
## 119    moralOther   moralOther    match        m     m   1 0.6570
## 120     moralSelf    moralSelf mismatch        n     n   1 0.8091
## 121    moralOther   moralOther    match        m     m   1 0.7233
## 122    moralOther   moralOther mismatch        n     n   1 0.9175
## 123   immoralSelf  immoralSelf mismatch        n     n   1 0.7036
## 124  immoralOther immoralOther mismatch        n     n   1 0.6777
## 125   immoralSelf  immoralSelf mismatch        n     n   1 0.7278
## 126    moralOther   moralOther mismatch        n     n   1 0.6960
## 127  immoralOther immoralOther mismatch        n     n   1 0.8401
## 128   immoralSelf  immoralSelf mismatch        n     n   1 0.7543
## 129     moralSelf    moralSelf    match        m     m   1 0.6304
## 130    moralOther   moralOther    match        m     m   1 0.5224
## 131   immoralSelf  immoralSelf    match        m     n   0 0.6446
## 132     moralSelf    moralSelf mismatch        n     n   1 0.8867
## 133  immoralOther immoralOther    match        m     m   1 0.7048
## 134   immoralSelf  immoralSelf    match        m     n   0 0.7030
## 135    moralOther   moralOther mismatch        n     n   1 0.6690
## 136   immoralSelf  immoralSelf    match        m     m   1 0.6172
## 137    moralOther   moralOther    match        m     m   1 0.6233
## 138  immoralOther immoralOther    match        m     m   1 0.5735
## 139     moralSelf    moralSelf    match        m     m   1 0.6936
## 140     moralSelf    moralSelf mismatch        n     m   0 1.0136
## 141  immoralOther immoralOther mismatch        n     n   1 0.8218
## 142  immoralOther immoralOther    match        m     m   1 0.6439
## 143     moralSelf    moralSelf mismatch        n     m   0 0.7100
## 144     moralSelf    moralSelf    match        m     m   1 0.9141
## 145  immoralOther immoralOther mismatch        n     n   1 0.7343
## 146     moralSelf    moralSelf mismatch        n     n   1 0.6464
## 147    moralOther   moralOther mismatch        n     m   0 0.5765
## 148     moralSelf    moralSelf    match        m     m   1 0.6566
## 149   immoralSelf  immoralSelf    match        m     m   1 0.8089
## 150  immoralOther immoralOther    match        m     m   1 0.6248
## 151     moralSelf    moralSelf    match        m     m   1 0.7109
## 152    moralOther   moralOther    match        m     m   1 0.6191
## 153    moralOther   moralOther    match        m     m   1 0.4171
## 154    moralOther   moralOther    match        m     m   1 0.4333
## 155   immoralSelf  immoralSelf mismatch        n     n   1 0.6934
## 156    moralOther   moralOther mismatch        n     n   1 0.7015
## 157  immoralOther immoralOther    match        m     m   1 0.5717
## 158  immoralOther immoralOther mismatch        n     n   1 0.9557
## 159     moralSelf    moralSelf mismatch        n     n   1 0.7838
## 160   immoralSelf  immoralSelf mismatch        n     n   1 0.8540
## 161   immoralSelf  immoralSelf    match        m     n   0 0.9501
## 162     moralSelf    moralSelf mismatch        n     n   1 0.7062
## 163   immoralSelf  immoralSelf    match        m     m   1 0.6644
## 164     moralSelf    moralSelf    match        m     m   1 0.7664
## 165   immoralSelf  immoralSelf mismatch        n     n   1 0.6406
## 166    moralOther   moralOther mismatch        n     n   1 0.6827
## 167  immoralOther immoralOther mismatch        n     n   1 0.8848
## 168  immoralOther immoralOther    match        m     m   1 0.6469
## 169  immoralOther immoralOther    match        m     m   1 0.4731
## 170    moralOther   moralOther mismatch        n     m   0 0.3812
## 171     moralSelf    moralSelf    match        m     m   1 0.5952
## 172  immoralOther immoralOther    match        m     m   1 0.8074
## 173   immoralSelf  immoralSelf mismatch        n     n   1 0.5675
## 174     moralSelf    moralSelf mismatch        n     n   1 0.8476
## 175  immoralOther immoralOther mismatch        n     n   1 0.7278
## 176   immoralSelf  immoralSelf    match        m     m   1 0.6079
## 177   immoralSelf  immoralSelf    match        m  <NA>  -1 1.0420
## 178     moralSelf    moralSelf mismatch        n     n   1 0.6901
## 179  immoralOther immoralOther mismatch        n     n   1 0.9862
## 180    moralOther   moralOther mismatch        n     n   1 0.7924
## 181   immoralSelf  immoralSelf mismatch        n     n   1 0.7325
## 182  immoralOther immoralOther    match        m     n   0 0.5727
## 183    moralOther   moralOther mismatch        n     n   1 0.8048
## 184     moralSelf    moralSelf mismatch        n     n   1 0.6549
## 185     moralSelf    moralSelf    match        m     m   1 0.6631
## 186  immoralOther immoralOther mismatch        n  <NA>  -1 1.0420
## 187   immoralSelf  immoralSelf mismatch        n     n   1 0.7672
## 188     moralSelf    moralSelf    match        m     m   1 0.7353
## 189    moralOther   moralOther    match        m     m   1 0.5515
## 190   immoralSelf  immoralSelf    match        m     m   1 0.6276
## 191    moralOther   moralOther    match        m     m   1 0.5277
## 192    moralOther   moralOther    match        m     m   1 0.4878
## 193     moralSelf    moralSelf    match        m     m   1 0.5327
## 194   immoralSelf  immoralSelf    match        m     m   1 0.6848
## 195    moralOther   moralOther    match        m     m   1 0.4549
## 196  immoralOther immoralOther    match        m     m   1 0.5350
## 197    moralOther   moralOther mismatch        n     n   1 0.5831
## 198     moralSelf    moralSelf mismatch        n     n   1 0.7973
## 199  immoralOther immoralOther    match        m     m   1 0.5433
## 200     moralSelf    moralSelf mismatch        n     n   1 0.5974
## 201  immoralOther immoralOther mismatch        n     n   1 0.6796
## 202     moralSelf    moralSelf mismatch        n     n   1 0.8337
## 203   immoralSelf  immoralSelf mismatch        n     m   0 0.5898
## 204    moralOther   moralOther mismatch        n     n   1 0.6259
## 205   immoralSelf  immoralSelf    match        m     m   1 0.7600
## 206  immoralOther immoralOther mismatch        n     n   1 0.6662
## 207    moralOther   moralOther    match        m     m   1 0.6003
## 208     moralSelf    moralSelf    match        m     n   0 0.7144
## 209    moralOther   moralOther    match        m     m   1 0.7025
## 210  immoralOther immoralOther mismatch        n     n   1 0.6766
## 211   immoralSelf  immoralSelf mismatch        n     n   1 0.8327
## 212  immoralOther immoralOther    match        m     m   1 0.7649
## 213   immoralSelf  immoralSelf    match        m     m   1 0.7991
## 214     moralSelf    moralSelf    match        m     m   1 0.6411
## 215   immoralSelf  immoralSelf mismatch        n     n   1 0.6912
## 216    moralOther   moralOther mismatch        n     n   1 0.6434
## 217    moralOther   moralOther    match        m     m   1 0.8215
## 218     moralSelf    moralSelf mismatch        n     n   1 0.7476
## 219     moralSelf    moralSelf mismatch        n     n   1 0.8797
## 220    moralOther   moralOther    match        m     m   1 0.6258
## 221  immoralOther immoralOther mismatch        n     m   0 0.6160
## 222  immoralOther immoralOther    match        m     n   0 0.9041
## 223    moralOther   moralOther mismatch        n     n   1 0.6702
## 224     moralSelf    moralSelf    match        m     m   1 0.6423
## 225   immoralSelf  immoralSelf mismatch        n     n   1 0.7164
## 226   immoralSelf  immoralSelf mismatch        n     n   1 0.8245
## 227   immoralSelf  immoralSelf mismatch        n     n   1 0.6947
## 228   immoralSelf  immoralSelf    match        m     n   0 0.6229
## 229  immoralOther immoralOther mismatch        n     n   1 0.6849
## 230  immoralOther immoralOther    match        m     m   1 0.7510
## 231  immoralOther immoralOther mismatch        n     n   1 0.8111
## 232     moralSelf    moralSelf mismatch        n     n   1 0.7233
## 233    moralOther   moralOther mismatch        n     n   1 0.8134
## 234    moralOther   moralOther    match        m     m   1 0.7335
## 235   immoralSelf  immoralSelf    match        m     m   1 0.7816
## 236     moralSelf    moralSelf    match        m     m   1 0.6757
## 237     moralSelf    moralSelf    match        m     m   1 0.5516
## 238   immoralSelf  immoralSelf    match        m     m   1 0.7800
## 239    moralOther   moralOther mismatch        n     n   1 0.8421
## 240  immoralOther immoralOther    match        m     m   1 0.6523
## 241    moralOther   moralOther    match        m     m   1 0.5825
## 242  immoralOther immoralOther    match        m     m   1 0.7726
## 243  immoralOther immoralOther    match        m     m   1 0.5407
## 244     moralSelf    moralSelf mismatch        n     n   1 0.7548
## 245   immoralSelf  immoralSelf    match        m     m   1 0.8230
## 246   immoralSelf  immoralSelf mismatch        n     n   1 0.8051
## 247  immoralOther immoralOther mismatch        n     n   1 0.7352
## 248   immoralSelf  immoralSelf    match        m     m   1 0.6634
## 249  immoralOther immoralOther mismatch        n     n   1 0.7255
## 250    moralOther   moralOther mismatch        n     n   1 0.7596
## 251  immoralOther immoralOther    match        m     m   1 0.6657
## 252     moralSelf    moralSelf mismatch        n     n   1 0.6478
## 253    moralOther   moralOther mismatch        n     n   1 0.8559
## 254     moralSelf    moralSelf mismatch        n     n   1 0.6961
## 255     moralSelf    moralSelf    match        m  <NA>  -1 1.0420
## 256     moralSelf    moralSelf    match        m     n   0 0.5864
## 257    moralOther   moralOther    match        m     m   1 0.6484
## 258     moralSelf    moralSelf    match        m     m   1 0.7446
## 259  immoralOther immoralOther mismatch        n     n   1 0.7027
## 260   immoralSelf  immoralSelf mismatch        n     n   1 0.6928
## 261   immoralSelf  immoralSelf    match        m     m   1 0.4849
## 262   immoralSelf  immoralSelf mismatch        n     n   1 0.5990
## 263    moralOther   moralOther mismatch        n     n   1 0.8572
## 264    moralOther   moralOther    match        m     n   0 0.5433
## 265    moralOther   moralOther mismatch        n     n   1 0.6774
## 266     moralSelf    moralSelf    match        m     m   1 0.5915
## 267  immoralOther immoralOther mismatch        n     n   1 0.7156
## 268  immoralOther immoralOther    match        m     m   1 0.5757
## 269    moralOther   moralOther    match        m     m   1 0.6719
## 270     moralSelf    moralSelf    match        m     m   1 0.6359
## 271   immoralSelf  immoralSelf    match        m     m   1 0.5040
## 272    moralOther   moralOther    match        m     m   1 0.5742
## 273    moralOther   moralOther mismatch        n     m   0 0.5662
## 274   immoralSelf  immoralSelf    match        m     m   1 0.5844
## 275    moralOther   moralOther mismatch        n     n   1 0.6546
## 276   immoralSelf  immoralSelf    match        m     m   1 0.5906
## 277     moralSelf    moralSelf mismatch        n     n   1 0.8028
## 278   immoralSelf  immoralSelf mismatch        n     m   0 0.6448
## 279     moralSelf    moralSelf mismatch        n     n   1 0.7029
## 280     moralSelf    moralSelf mismatch        n     n   1 0.6451
## 281   immoralSelf  immoralSelf mismatch        n     n   1 0.7272
## 282    moralOther   moralOther    match        m     m   1 0.6233
## 283  immoralOther immoralOther mismatch        n     m   0 0.6935
## 284  immoralOther immoralOther    match        m     m   1 0.6455
## 285  immoralOther immoralOther mismatch        n     n   1 0.6877
## 286  immoralOther immoralOther    match        m     m   1 0.6558
## 287   immoralSelf  immoralSelf mismatch        n     n   1 0.6879
## 288     moralSelf    moralSelf    match        m     n   0 0.7681
## 289   immoralSelf  immoralSelf mismatch        n     m   0 0.7201
## 290  immoralOther immoralOther    match        m     m   1 0.6262
## 291   immoralSelf  immoralSelf    match        m     m   1 0.7044
## 292  immoralOther immoralOther    match        m     m   1 0.5825
## 293    moralOther   moralOther mismatch        n     n   1 0.8366
## 294  immoralOther immoralOther mismatch        n     n   1 0.7687
## 295  immoralOther immoralOther mismatch        n     n   1 0.6648
## 296   immoralSelf  immoralSelf mismatch        n     m   0 0.6230
## 297  immoralOther immoralOther mismatch        n     n   1 0.8054
## 298   immoralSelf  immoralSelf mismatch        n     n   1 0.7272
## 299   immoralSelf  immoralSelf    match        m     m   1 0.6633
## 300    moralOther   moralOther    match        m     m   1 0.6774
## 301     moralSelf    moralSelf    match        m     m   1 0.7595
## 302    moralOther   moralOther    match        m     m   1 0.6416
## 303     moralSelf    moralSelf mismatch        n     n   1 0.7878
## 304     moralSelf    moralSelf    match        m     m   1 0.5839
## 305     moralSelf    moralSelf    match        m     m   1 0.6141
## 306    moralOther   moralOther    match        m     m   1 0.6781
## 307   immoralSelf  immoralSelf    match        m     m   1 0.6402
## 308     moralSelf    moralSelf mismatch        n     n   1 0.6744
## 309  immoralOther immoralOther    match        m     m   1 0.8785
## 310    moralOther   moralOther mismatch        n     n   1 0.7526
## 311    moralOther   moralOther mismatch        n     n   1 0.8447
## 312     moralSelf    moralSelf mismatch        n     m   0 0.6229
## 313  immoralOther immoralOther    match        m     n   0 0.7675
## 314    moralOther   moralOther    match        m     m   1 0.5276
## 315  immoralOther immoralOther    match        m     m   1 0.8237
## 316   immoralSelf  immoralSelf mismatch        n     n   1 0.9900
## 317    moralOther   moralOther    match        m     m   1 0.7619
## 318     moralSelf    moralSelf mismatch        n     n   1 0.7082
## 319  immoralOther immoralOther    match        m     m   1 0.7522
## 320     moralSelf    moralSelf    match        m     m   1 0.7803
## 321   immoralSelf  immoralSelf    match        m     m   1 0.6424
## 322     moralSelf    moralSelf mismatch        n     n   1 0.6765
## 323     moralSelf    moralSelf mismatch        n     n   1 0.8247
## 324     moralSelf    moralSelf    match        m     m   1 0.6549
## 325   immoralSelf  immoralSelf    match        m     m   1 0.6629
## 326    moralOther   moralOther mismatch        n     n   1 0.7330
## 327   immoralSelf  immoralSelf mismatch        n     n   1 0.7891
## 328  immoralOther immoralOther mismatch        n  <NA>  -1 1.0420
## 329  immoralOther immoralOther mismatch        n  <NA>  -1 1.0420
## 330   immoralSelf  immoralSelf    match        m     m   1 0.6935
## 331  immoralOther immoralOther mismatch        n     n   1 0.6377
## 332    moralOther   moralOther mismatch        n     n   1 0.7518
## 333   immoralSelf  immoralSelf mismatch        n     n   1 0.7239
## 334    moralOther   moralOther mismatch        n     n   1 0.6460
## 335     moralSelf    moralSelf    match        m     m   1 0.7361
## 336    moralOther   moralOther    match        m     m   1 0.6323
## 337    moralOther   moralOther mismatch        n     n   1 0.7163
## 338     moralSelf    moralSelf mismatch        n     m   0 0.7205
## 339    moralOther   moralOther    match        m     m   1 0.6986
## 340  immoralOther immoralOther mismatch        n     n   1 0.6967
## 341   immoralSelf  immoralSelf mismatch        n     n   1 0.7288
## 342     moralSelf    moralSelf    match        m     m   1 0.6010
## 343  immoralOther immoralOther    match        m     n   0 0.6431
## 344    moralOther   moralOther mismatch        n     n   1 0.6292
## 345   immoralSelf  immoralSelf    match        m     m   1 0.7373
## 346   immoralSelf  immoralSelf mismatch        n     n   1 0.6874
## 347    moralOther   moralOther    match        m     m   1 0.6475
## 348     moralSelf    moralSelf    match        m     m   1 0.7757
## 349     moralSelf    moralSelf    match        m     m   1 0.6478
## 350  immoralOther immoralOther    match        m     m   1 0.6399
## 351  immoralOther immoralOther    match        m     m   1 0.5701
## 352  immoralOther immoralOther mismatch        n     n   1 0.6822
## 353     moralSelf    moralSelf mismatch        n     n   1 0.9402
## 354    moralOther   moralOther    match        m     m   1 0.5384
## 355  immoralOther immoralOther mismatch        n     n   1 0.6165
## 356    moralOther   moralOther mismatch        n     n   1 0.6806
## 357   immoralSelf  immoralSelf    match        m     m   1 0.7307
## 358     moralSelf    moralSelf mismatch        n     n   1 0.7868
## 359   immoralSelf  immoralSelf mismatch        n     n   1 0.7850
## 360   immoralSelf  immoralSelf    match        m     m   1 0.6551
## 361   immoralSelf  immoralSelf mismatch        n     n   1 0.6804
## 362    moralOther   moralOther mismatch        n     n   1 0.5540
## 363  immoralOther immoralOther mismatch        n     n   1 0.6528
## 364     moralSelf    moralSelf mismatch        n     n   1 0.6016
## 365   immoralSelf  immoralSelf    match        m     n   0 0.6214
## 366   immoralSelf  immoralSelf    match        m     n   0 0.7058
## 367    moralOther   moralOther    match        m     m   1 0.6918
## 368     moralSelf    moralSelf    match        m     m   1 0.6334
## 369    moralOther   moralOther mismatch        n     n   1 0.7380
## 370   immoralSelf  immoralSelf mismatch        n     n   1 0.8765
## 371    moralOther   moralOther mismatch        n     m   0 0.6097
## 372  immoralOther immoralOther    match        m     m   1 0.7058
## 373    moralOther   moralOther    match        m     m   1 0.6758
## 374     moralSelf    moralSelf mismatch        n     m   0 0.6251
## 375  immoralOther immoralOther    match        m     n   0 0.9173
## 376   immoralSelf  immoralSelf mismatch        n     m   0 0.6835
## 377     moralSelf    moralSelf mismatch        n     m   0 0.4930
## 378     moralSelf    moralSelf    match        m     m   1 0.5227
## 379   immoralSelf  immoralSelf    match        m     n   0 0.7209
## 380    moralOther   moralOther    match        m     m   1 0.6392
## 381  immoralOther immoralOther mismatch        n     n   1 0.6637
## 382     moralSelf    moralSelf    match        m     m   1 0.6088
## 383  immoralOther immoralOther    match        m     n   0 0.5807
## 384  immoralOther immoralOther mismatch        n     n   1 0.7360
## 385     moralSelf    moralSelf    match        m     m   1 0.6104
## 386     moralSelf    moralSelf mismatch        n     n   1 0.6665
## 387  immoralOther immoralOther    match        m     m   1 0.6515
## 388  immoralOther immoralOther mismatch        n     m   0 0.6243
## 389   immoralSelf  immoralSelf mismatch        n     n   1 0.8286
## 390  immoralOther immoralOther    match        m     m   1 0.7210
## 391     moralSelf    moralSelf    match        m     m   1 0.6552
## 392    moralOther   moralOther    match        m     n   0 0.6440
## 393   immoralSelf  immoralSelf mismatch        n     m   0 0.7965
## 394    moralOther   moralOther    match        m     m   1 0.6921
## 395   immoralSelf  immoralSelf mismatch        n     n   1 0.8660
## 396     moralSelf    moralSelf mismatch        n     m   0 0.7070
## 397   immoralSelf  immoralSelf    match        m     n   0 0.7329
## 398    moralOther   moralOther mismatch        n     m   0 0.6153
## 399    moralOther   moralOther mismatch        n     n   1 0.7513
## 400   immoralSelf  immoralSelf    match        m     m   1 0.8663
## 401    moralOther   moralOther mismatch        n     n   1 0.8193
## 402     moralSelf    moralSelf mismatch        n     n   1 0.9074
## 403     moralSelf    moralSelf    match        m     m   1 0.6614
## 404  immoralOther immoralOther mismatch        n     m   0 0.9264
## 405  immoralOther immoralOther    match        m     m   1 0.6808
## 406   immoralSelf  immoralSelf    match        m     m   1 0.7625
## 407  immoralOther immoralOther mismatch        n     n   1 0.8735
## 408    moralOther   moralOther    match        m     m   1 0.5428
## 409  immoralOther immoralOther    match        m     n   0 0.6695
## 410    moralOther   moralOther mismatch        n     n   1 0.6271
## 411     moralSelf    moralSelf mismatch        n     n   1 0.7910
## 412   immoralSelf  immoralSelf mismatch        n     n   1 0.7107
## 413    moralOther   moralOther    match        m     m   1 0.6565
## 414    moralOther   moralOther    match        m     m   1 0.6177
## 415  immoralOther immoralOther mismatch        n     n   1 0.8417
## 416     moralSelf    moralSelf mismatch        n     n   1 0.6582
## 417  immoralOther immoralOther    match        m     n   0 0.7074
## 418     moralSelf    moralSelf mismatch        n     n   1 0.7733
## 419   immoralSelf  immoralSelf    match        m     m   1 0.7804
## 420     moralSelf    moralSelf    match        m     m   1 0.7198
## 421   immoralSelf  immoralSelf    match        m     n   0 0.8460
## 422  immoralOther immoralOther mismatch        n     n   1 0.6828
## 423   immoralSelf  immoralSelf mismatch        n     m   0 0.8843
## 424    moralOther   moralOther    match        m     m   1 0.6699
## 425   immoralSelf  immoralSelf    match        m     m   1 0.6911
## 426    moralOther   moralOther mismatch        n     m   0 0.6007
## 427   immoralSelf  immoralSelf mismatch        n     n   1 0.8526
## 428  immoralOther immoralOther mismatch        n     n   1 0.7775
## 429    moralOther   moralOther mismatch        n     n   1 0.8048
## 430     moralSelf    moralSelf    match        m     m   1 0.7448
## 431  immoralOther immoralOther    match        m     m   1 0.8394
## 432     moralSelf    moralSelf    match        m     m   1 0.5999
## 433   immoralSelf  immoralSelf    match        m     m   1 0.6555
## 434    moralOther   moralOther mismatch        n     n   1 0.7643
## 435     moralSelf    moralSelf mismatch        n     m   0 0.5875
## 436     moralSelf    moralSelf    match        m     m   1 0.7150
## 437  immoralOther immoralOther    match        m     m   1 0.9692
## 438  immoralOther immoralOther    match        m     m   1 0.5684
## 439   immoralSelf  immoralSelf mismatch        n     n   1 0.6474
## 440    moralOther   moralOther    match        m     m   1 0.6923
## 441    moralOther   moralOther    match        m     m   1 0.5941
## 442     moralSelf    moralSelf    match        m     m   1 0.6497
## 443   immoralSelf  immoralSelf mismatch        n     n   1 0.8865
## 444     moralSelf    moralSelf mismatch        n     m   0 0.6240
## 445    moralOther   moralOther mismatch        n     m   0 0.6123
## 446  immoralOther immoralOther mismatch        n     n   1 0.7805
## 447  immoralOther immoralOther mismatch        n     n   1 0.8557
## 448   immoralSelf  immoralSelf mismatch        n     m   0 0.8046
## 449     moralSelf    moralSelf mismatch        n     m   0 0.6403
## 450     moralSelf    moralSelf    match        m     m   1 0.6330
## 451   immoralSelf  immoralSelf    match        m     n   0 0.6495
## 452  immoralOther immoralOther mismatch        n     n   1 0.9023
## 453    moralOther   moralOther    match        m     m   1 1.0280
## 454    moralOther   moralOther mismatch        n     m   0 0.7924
## 455  immoralOther immoralOther    match        m     n   0 0.6560
## 456   immoralSelf  immoralSelf    match        m     n   0 0.8331
## 457    moralOther   moralOther mismatch        n     n   1 0.7857
## 458    moralOther   moralOther    match        m     m   1 0.8491
## 459  immoralOther immoralOther    match        m     m   1 0.7980
## 460     moralSelf    moralSelf    match        m     m   1 0.6420
## 461   immoralSelf  immoralSelf    match        m     m   1 0.7224
## 462    moralOther   moralOther    match        m     m   1 0.5847
## 463  immoralOther immoralOther mismatch        n     m   0 0.7880
## 464  immoralOther immoralOther    match        m     m   1 0.8436
## 465  immoralOther immoralOther mismatch        n     n   1 0.7082
## 466     moralSelf    moralSelf mismatch        n     m   0 0.5741
## 467    moralOther   moralOther mismatch        n     n   1 0.7936
## 468  immoralOther immoralOther mismatch        n     n   1 0.8092
## 469   immoralSelf  immoralSelf mismatch        n     n   1 0.6653
## 470     moralSelf    moralSelf mismatch        n     n   1 0.7783
## 471   immoralSelf  immoralSelf    match        m     m   1 0.7577
## 472   immoralSelf  immoralSelf    match        m     m   1 0.6566
## 473   immoralSelf  immoralSelf mismatch        n     n   1 0.7457
## 474    moralOther   moralOther mismatch        n     n   1 0.8483
## 475   immoralSelf  immoralSelf mismatch        n     n   1 0.7890
## 476  immoralOther immoralOther    match        m     m   1 0.6207
## 477     moralSelf    moralSelf    match        m     m   1 0.5770
## 478     moralSelf    moralSelf    match        m     m   1 0.5245
## 479    moralOther   moralOther    match        m     m   1 0.6346
## 480     moralSelf    moralSelf mismatch        n     m   0 0.7471
## 481    moralOther   moralOther    match        m  <NA>  -1 1.0411
## 482    moralOther   moralOther mismatch        n     n   1 0.5757
## 483   immoralSelf  immoralSelf mismatch        n     n   1 0.6671
## 484  immoralOther immoralOther mismatch        n     n   1 0.8121
## 485   immoralSelf  immoralSelf mismatch        n     n   1 0.7242
## 486    moralOther   moralOther mismatch        n     m   0 0.5783
## 487  immoralOther immoralOther mismatch        n     n   1 0.6457
## 488   immoralSelf  immoralSelf mismatch        n     n   1 0.6105
## 489     moralSelf    moralSelf    match        m     n   0 0.7146
## 490    moralOther   moralOther    match        m     m   1 0.6168
## 491   immoralSelf  immoralSelf    match        m     m   1 0.7129
## 492     moralSelf    moralSelf mismatch        n     m   0 0.5469
## 493  immoralOther immoralOther    match        m     n   0 0.6416
## 494   immoralSelf  immoralSelf    match        m     m   1 0.9061
## 495    moralOther   moralOther mismatch        n     n   1 0.8921
## 496   immoralSelf  immoralSelf    match        m     m   1 0.7778
## 497    moralOther   moralOther    match        m     m   1 0.7573
## 498  immoralOther immoralOther    match        m     m   1 0.6561
## 499     moralSelf    moralSelf    match        m     m   1 0.6650
## 500     moralSelf    moralSelf mismatch        n     n   1 0.8103
## 501  immoralOther immoralOther mismatch        n     n   1 0.7063
## 502  immoralOther immoralOther    match        m     m   1 0.5642
## 503     moralSelf    moralSelf mismatch        n     n   1 0.9875
## 504     moralSelf    moralSelf    match        m     m   1 0.8111
## 505  immoralOther immoralOther mismatch        n     n   1 0.7471
## 506     moralSelf    moralSelf mismatch        n     n   1 0.7778
## 507    moralOther   moralOther mismatch        n     n   1 0.7651
## 508     moralSelf    moralSelf    match        m     m   1 0.6124
## 509   immoralSelf  immoralSelf    match        m     n   0 0.6443
## 510  immoralOther immoralOther    match        m     m   1 0.7932
## 511     moralSelf    moralSelf    match        m     m   1 0.7048
## 512    moralOther   moralOther    match        m     m   1 0.5629
## 513    moralOther   moralOther    match        m     m   1 0.5699
## 514    moralOther   moralOther    match        m     m   1 0.6330
## 515   immoralSelf  immoralSelf mismatch        n     n   1 0.6096
## 516    moralOther   moralOther mismatch        n     m   0 0.5537
## 517  immoralOther immoralOther    match        m     m   1 0.6845
## 518  immoralOther immoralOther mismatch        n     n   1 0.6620
## 519     moralSelf    moralSelf mismatch        n     n   1 0.7191
## 520   immoralSelf  immoralSelf mismatch        n     m   0 0.6853
## 521   immoralSelf  immoralSelf    match        m     m   1 0.6788
## 522     moralSelf    moralSelf mismatch        n     n   1 0.6521
## 523   immoralSelf  immoralSelf    match        m     n   0 0.6088
## 524     moralSelf    moralSelf    match        m     m   1 0.6489
## 525   immoralSelf  immoralSelf mismatch        n     n   1 0.7097
## 526    moralOther   moralOther mismatch        n     n   1 0.7037
## 527  immoralOther immoralOther mismatch        n     n   1 0.6577
## 528  immoralOther immoralOther    match        m     m   1 0.6107
## 529  immoralOther immoralOther    match        m     m   1 0.6588
## 530    moralOther   moralOther mismatch        n     n   1 0.7558
## 531     moralSelf    moralSelf    match        m     m   1 0.5107
## 532  immoralOther immoralOther    match        m     n   0 0.6167
## 533   immoralSelf  immoralSelf mismatch        n     n   1 0.8488
## 534     moralSelf    moralSelf mismatch        n     m   0 0.6456
## 535  immoralOther immoralOther mismatch        n     n   1 0.8187
## 536   immoralSelf  immoralSelf    match        m     m   1 0.9704
## 537   immoralSelf  immoralSelf    match        m     m   1 0.5936
## 538     moralSelf    moralSelf mismatch        n     n   1 0.7771
## 539  immoralOther immoralOther mismatch        n     m   0 0.7085
## 540    moralOther   moralOther mismatch        n     m   0 0.7185
## 541   immoralSelf  immoralSelf mismatch        n     n   1 0.7648
## 542  immoralOther immoralOther    match        m     m   1 0.7319
## 543    moralOther   moralOther mismatch        n     n   1 0.7344
## 544     moralSelf    moralSelf mismatch        n     n   1 0.7528
## 545     moralSelf    moralSelf    match        m     m   1 0.5397
## 546  immoralOther immoralOther mismatch        n     n   1 0.6983
## 547   immoralSelf  immoralSelf mismatch        n     m   0 1.0160
## 548     moralSelf    moralSelf    match        m     m   1 0.8002
## 549    moralOther   moralOther    match        m     m   1 0.7641
## 550   immoralSelf  immoralSelf    match        m     m   1 0.7312
## 551    moralOther   moralOther    match        m     m   1 0.6936
## 552    moralOther   moralOther    match        m     m   1 0.6514
## 553     moralSelf    moralSelf    match        m     m   1 0.5113
## 554   immoralSelf  immoralSelf    match        m     n   0 0.5612
## 555    moralOther   moralOther    match        m     m   1 0.6201
## 556  immoralOther immoralOther    match        m     m   1 0.6281
## 557    moralOther   moralOther mismatch        n     n   1 0.7205
## 558     moralSelf    moralSelf mismatch        n     m   0 0.5828
## 559  immoralOther immoralOther    match        m     m   1 0.6740
## 560     moralSelf    moralSelf mismatch        n     n   1 0.8475
## 561  immoralOther immoralOther mismatch        n     n   1 0.7161
## 562     moralSelf    moralSelf mismatch        n     n   1 0.6583
## 563   immoralSelf  immoralSelf mismatch        n     n   1 0.8194
## 564    moralOther   moralOther mismatch        n     n   1 0.6354
## 565   immoralSelf  immoralSelf    match        m     m   1 0.7639
## 566  immoralOther immoralOther mismatch        n     n   1 0.7231
## 567    moralOther   moralOther    match        m     m   1 0.6494
## 568     moralSelf    moralSelf    match        m     n   0 0.8062
## 569    moralOther   moralOther    match        m     n   0 0.7541
## 570  immoralOther immoralOther mismatch        n     n   1 0.7889
## 571   immoralSelf  immoralSelf mismatch        n     m   0 0.8205
## 572  immoralOther immoralOther    match        m     m   1 0.7049
## 573   immoralSelf  immoralSelf    match        m     m   1 0.4588
## 574     moralSelf    moralSelf    match        m     m   1 0.6920
## 575   immoralSelf  immoralSelf mismatch        n     n   1 0.7739
## 576    moralOther   moralOther mismatch        n     m   0 0.7772
## 577    moralOther   moralOther    match        m     m   1 0.5647
## 578     moralSelf    moralSelf mismatch        n     m   0 0.7398
## 579     moralSelf    moralSelf mismatch        n     n   1 0.8345
## 580    moralOther   moralOther    match        m     m   1 0.7390
## 581  immoralOther immoralOther mismatch        n     n   1 0.7456
## 582  immoralOther immoralOther    match        m     m   1 0.6803
## 583    moralOther   moralOther mismatch        n     m   0 0.8579
## 584     moralSelf    moralSelf    match        m     m   1 0.5309
## 585   immoralSelf  immoralSelf mismatch        n     m   0 0.9173
## 586   immoralSelf  immoralSelf mismatch        n     n   1 0.8196
## 587   immoralSelf  immoralSelf mismatch        n     n   1 0.6916
## 588   immoralSelf  immoralSelf    match        m     m   1 0.6814
## 589  immoralOther immoralOther mismatch        n     n   1 0.7228
## 590  immoralOther immoralOther    match        m     m   1 0.6331
## 591  immoralOther immoralOther mismatch        n     n   1 0.8096
## 592     moralSelf    moralSelf mismatch        n     n   1 0.7298
## 593    moralOther   moralOther mismatch        n     n   1 0.5882
## 594    moralOther   moralOther    match        m     m   1 0.5957
## 595   immoralSelf  immoralSelf    match        m     m   1 0.7393
## 596     moralSelf    moralSelf    match        m     m   1 0.6261
## 597     moralSelf    moralSelf    match        m     m   1 0.7024
## 598   immoralSelf  immoralSelf    match        m     m   1 0.7080
## 599    moralOther   moralOther mismatch        n     m   0 0.6382
## 600  immoralOther immoralOther    match        m     m   1 0.8227
## 601    moralOther   moralOther    match        m     m   1 0.9998
## 602  immoralOther immoralOther    match        m     m   1 0.6756
## 603  immoralOther immoralOther    match        m     n   0 0.7929
## 604     moralSelf    moralSelf mismatch        n     m   0 0.8287
## 605   immoralSelf  immoralSelf    match        m     m   1 0.9211
## 606   immoralSelf  immoralSelf mismatch        n     n   1 1.0073
## 607  immoralOther immoralOther mismatch        n     m   0 0.6672
## 608   immoralSelf  immoralSelf    match        m     m   1 0.8202
## 609  immoralOther immoralOther mismatch        n     n   1 0.6282
## 610    moralOther   moralOther mismatch        n     m   0 0.6967
## 611  immoralOther immoralOther    match        m     m   1 0.7743
## 612     moralSelf    moralSelf mismatch        n     n   1 0.9296
## 613    moralOther   moralOther mismatch        n     n   1 0.7241
## 614     moralSelf    moralSelf mismatch        n     n   1 0.8183
## 615     moralSelf    moralSelf    match        m     m   1 0.5304
## 616     moralSelf    moralSelf    match        m     m   1 0.4689
## 617    moralOther   moralOther    match        m     m   1 0.6342
## 618     moralSelf    moralSelf    match        m     m   1 0.8027
## 619  immoralOther immoralOther mismatch        n     n   1 0.9585
## 620   immoralSelf  immoralSelf mismatch        n     m   0 0.7695
## 621   immoralSelf  immoralSelf    match        m     n   0 0.7286
## 622   immoralSelf  immoralSelf mismatch        n     n   1 0.7711
## 623    moralOther   moralOther mismatch        n     m   0 0.6742
## 624    moralOther   moralOther    match        m     m   1 1.0235
## 625    moralOther   moralOther mismatch        n     n   1 0.6919
## 626     moralSelf    moralSelf    match        m     m   1 0.5935
## 627  immoralOther immoralOther mismatch        n     n   1 0.8091
## 628  immoralOther immoralOther    match        m     m   1 0.7532
## 629    moralOther   moralOther    match        m     m   1 0.7723
## 630     moralSelf    moralSelf    match        m     m   1 0.6672
## 631   immoralSelf  immoralSelf    match        m     n   0 0.6363
## 632    moralOther   moralOther    match        m     m   1 0.6049
## 633    moralOther   moralOther mismatch        n     n   1 0.5448
## 634   immoralSelf  immoralSelf    match        m     m   1 0.7033
## 635    moralOther   moralOther mismatch        n     n   1 0.6894
## 636   immoralSelf  immoralSelf    match        m     m   1 0.7068
## 637     moralSelf    moralSelf mismatch        n     m   0 0.5408
## 638   immoralSelf  immoralSelf mismatch        n     n   1 0.6675
## 639     moralSelf    moralSelf mismatch        n     n   1 0.7485
## 640     moralSelf    moralSelf mismatch        n     n   1 0.8351
## 641   immoralSelf  immoralSelf mismatch        n     n   1 0.8756
## 642    moralOther   moralOther    match        m     n   0 0.7210
## 643  immoralOther immoralOther mismatch        n     n   1 0.8713
## 644  immoralOther immoralOther    match        m     m   1 0.9885
## 645  immoralOther immoralOther mismatch        n     n   1 0.7241
## 646  immoralOther immoralOther    match        m     m   1 0.8104
## 647   immoralSelf  immoralSelf mismatch        n     n   1 0.7784
## 648     moralSelf    moralSelf    match        m     m   1 0.6618
## 649   immoralSelf  immoralSelf mismatch        n     n   1 0.7748
## 650  immoralOther immoralOther    match        m     m   1 0.6422
## 651   immoralSelf  immoralSelf    match        m     m   1 0.6668
## 652  immoralOther immoralOther    match        m     n   0 0.7558
## 653    moralOther   moralOther mismatch        n     m   0 0.6707
## 654  immoralOther immoralOther mismatch        n     n   1 0.8200
## 655  immoralOther immoralOther mismatch        n     n   1 0.8200
## 656   immoralSelf  immoralSelf mismatch        n     n   1 0.7802
## 657  immoralOther immoralOther mismatch        n     n   1 0.7275
## 658   immoralSelf  immoralSelf mismatch        n     m   0 0.8340
## 659   immoralSelf  immoralSelf    match        m     m   1 0.8585
## 660    moralOther   moralOther    match        m     m   1 0.9316
## 661     moralSelf    moralSelf    match        m     n   0 0.6860
## 662    moralOther   moralOther    match        m     m   1 0.9035
## 663     moralSelf    moralSelf mismatch        n     n   1 0.8095
## 664     moralSelf    moralSelf    match        m     m   1 0.8655
## 665     moralSelf    moralSelf    match        m     m   1 0.7307
## 666    moralOther   moralOther    match        m     m   1 0.8769
## 667   immoralSelf  immoralSelf    match        m     m   1 0.9622
## 668     moralSelf    moralSelf mismatch        n     n   1 0.6975
## 669  immoralOther immoralOther    match        m     m   1 0.7672
## 670    moralOther   moralOther mismatch        n     n   1 0.7664
## 671    moralOther   moralOther mismatch        n     n   1 0.7496
## 672     moralSelf    moralSelf mismatch        n     n   1 0.8442
## 673  immoralOther immoralOther    match        m     m   1 0.8402
## 674    moralOther   moralOther    match        m     m   1 0.7288
## 675  immoralOther immoralOther    match        m     m   1 0.8033
## 676   immoralSelf  immoralSelf mismatch        n     n   1 0.7671
## 677    moralOther   moralOther    match        m     m   1 0.6463
## 678     moralSelf    moralSelf mismatch        n     m   0 0.7471
## 679  immoralOther immoralOther    match        m     m   1 0.7057
## 680     moralSelf    moralSelf    match        m     m   1 0.7477
## 681   immoralSelf  immoralSelf    match        m     m   1 0.7224
## 682     moralSelf    moralSelf mismatch        n     n   1 0.8325
## 683     moralSelf    moralSelf mismatch        n     m   0 0.8091
## 684     moralSelf    moralSelf    match        m     m   1 0.7373
## 685   immoralSelf  immoralSelf    match        m     m   1 0.6916
## 686    moralOther   moralOther mismatch        n     n   1 0.7854
## 687   immoralSelf  immoralSelf mismatch        n     n   1 0.7611
## 688  immoralOther immoralOther mismatch        n     n   1 0.7400
## 689  immoralOther immoralOther mismatch        n     n   1 0.7626
## 690   immoralSelf  immoralSelf    match        m     n   0 0.7058
## 691  immoralOther immoralOther mismatch        n     n   1 0.8276
## 692    moralOther   moralOther mismatch        n     n   1 0.8401
## 693   immoralSelf  immoralSelf mismatch        n     n   1 0.7287
## 694    moralOther   moralOther mismatch        n     n   1 0.8431
## 695     moralSelf    moralSelf    match        m     m   1 0.7476
## 696    moralOther   moralOther    match        m     n   0 0.8343
## 697    moralOther   moralOther mismatch        n     n   1 0.7229
## 698     moralSelf    moralSelf mismatch        n     m   0 0.9850
## 699    moralOther   moralOther    match        m     m   1 0.9084
## 700  immoralOther immoralOther mismatch        n     n   1 0.6864
## 701   immoralSelf  immoralSelf mismatch        n     n   1 0.8639
## 702     moralSelf    moralSelf    match        m     m   1 0.6090
## 703  immoralOther immoralOther    match        m     m   1 0.7450
## 704    moralOther   moralOther mismatch        n     n   1 0.5757
## 705   immoralSelf  immoralSelf    match        m     m   1 0.6591
## 706   immoralSelf  immoralSelf mismatch        n     n   1 0.7562
## 707    moralOther   moralOther    match        m     m   1 0.8951
## 708     moralSelf    moralSelf    match        m     m   1 0.6049
## 709     moralSelf    moralSelf    match        m     m   1 0.6806
## 710  immoralOther immoralOther    match        m     n   0 0.5344
## 711  immoralOther immoralOther    match        m     m   1 0.6328
## 712  immoralOther immoralOther mismatch        n     n   1 0.7214
## 713     moralSelf    moralSelf mismatch        n     n   1 0.6876
## 714    moralOther   moralOther    match        m     n   0 0.6730
## 715  immoralOther immoralOther mismatch        n     n   1 0.5823
## 716    moralOther   moralOther mismatch        n     m   0 0.5057
## 717   immoralSelf  immoralSelf    match        m     m   1 0.5477
## 718     moralSelf    moralSelf mismatch        n     n   1 0.8023
## 719   immoralSelf  immoralSelf mismatch        n     n   1 0.7662
## 720   immoralSelf  immoralSelf    match        m     m   1 0.7414
## 721  immoralOther immoralOther mismatch        m     n   0 0.7247
## 722   immoralSelf  immoralSelf mismatch        m     m   1 0.8786
## 723    moralOther   moralOther    match        n     n   1 0.8167
## 724  immoralOther immoralOther    match        n     m   0 0.8728
## 725    moralOther   moralOther mismatch        m     n   0 0.6849
## 726   immoralSelf  immoralSelf mismatch        m     n   0 0.6951
## 727     moralSelf    moralSelf    match        n     n   1 0.4951
## 728  immoralOther immoralOther    match        n     m   0 0.8713
## 729     moralSelf    moralSelf    match        n     n   1 0.5954
## 730  immoralOther immoralOther    match        n     n   1 0.6875
## 731     moralSelf    moralSelf mismatch        m     m   1 0.6796
## 732    moralOther   moralOther mismatch        m     n   0 0.8177
## 733   immoralSelf  immoralSelf    match        n     m   0 0.7439
## 734     moralSelf    moralSelf mismatch        m     n   0 0.3500
## 735   immoralSelf  immoralSelf mismatch        m  <NA>  -1 1.0330
## 736  immoralOther immoralOther mismatch        m     n   0 0.7582
## 737     moralSelf    moralSelf mismatch        m     m   1 0.7363
## 738  immoralOther immoralOther mismatch        m  <NA>  -1 1.0330
## 739   immoralSelf  immoralSelf    match        n     m   0 0.8926
## 740    moralOther   moralOther    match        n     m   0 0.8527
## 741    moralOther   moralOther mismatch        m     n   0 0.6848
## 742   immoralSelf  immoralSelf    match        n     m   0 0.4070
## 743     moralSelf    moralSelf    match        n     n   1 0.3611
## 744    moralOther   moralOther    match        n     n   1 0.9631
## 745  immoralOther immoralOther    match        n     m   0 0.7173
## 746   immoralSelf  immoralSelf mismatch        m     m   1 0.7514
## 747    moralOther   moralOther mismatch        m     m   1 0.9234
## 748    moralOther   moralOther mismatch        m     m   1 0.5976
## 749   immoralSelf  immoralSelf mismatch        m     m   1 0.7138
## 750   immoralSelf  immoralSelf    match        n     n   1 0.7979
## 751     moralSelf    moralSelf mismatch        m     n   0 0.4800
## 752  immoralOther immoralOther mismatch        m     m   1 0.6661
## 753     moralSelf    moralSelf    match        n     n   1 0.6882
## 754    moralOther   moralOther    match        n     m   0 0.6884
## 755  immoralOther immoralOther    match        n     n   1 0.8164
## 756   immoralSelf  immoralSelf mismatch        m     m   1 0.6246
## 757  immoralOther immoralOther mismatch        m     m   1 0.8146
## 758    moralOther   moralOther    match        n     n   1 0.6148
## 759   immoralSelf  immoralSelf    match        n     n   1 0.6229
## 760     moralSelf    moralSelf mismatch        m     m   1 0.5970
## 761  immoralOther immoralOther mismatch        m     n   0 0.7612
## 762  immoralOther immoralOther    match        n     m   0 0.6112
## 763     moralSelf    moralSelf mismatch        m     m   1 0.7074
## 764     moralSelf    moralSelf    match        n     n   1 0.5754
## 765    moralOther   moralOther mismatch        m     m   1 0.9195
## 766   immoralSelf  immoralSelf    match        n     n   1 0.6337
## 767     moralSelf    moralSelf    match        n     n   1 0.7498
## 768    moralOther   moralOther    match        n     n   1 0.6819
## 769     moralSelf    moralSelf mismatch        m     n   0 0.4120
## 770  immoralOther immoralOther    match        n     m   0 0.6201
## 771   immoralSelf  immoralSelf    match        n     n   1 0.7542
## 772  immoralOther immoralOther mismatch        m     n   0 0.7904
## 773    moralOther   moralOther    match        n     n   1 0.6825
## 774     moralSelf    moralSelf mismatch        m     m   1 0.5885
## 775    moralOther   moralOther mismatch        m     n   0 0.7607
## 776   immoralSelf  immoralSelf    match        n     n   1 0.6188
## 777   immoralSelf  immoralSelf mismatch        m     n   0 0.9849
## 778  immoralOther immoralOther    match        n     n   1 0.9591
## 779    moralOther   moralOther    match        n     m   0 0.8971
## 780   immoralSelf  immoralSelf    match        n     n   1 0.7594
## 781  immoralOther immoralOther mismatch        m     m   1 0.7855
## 782     moralSelf    moralSelf    match        n     n   1 0.5276
## 783    moralOther   moralOther mismatch        m     n   0 0.8717
## 784     moralSelf    moralSelf    match        n     n   1 0.4998
## 785     moralSelf    moralSelf mismatch        m     m   1 0.8979
## 786   immoralSelf  immoralSelf mismatch        m     n   0 0.8000
## 787     moralSelf    moralSelf    match        n     n   1 0.5782
## 788   immoralSelf  immoralSelf mismatch        m     m   1 0.7762
## 789  immoralOther immoralOther mismatch        m     n   0 0.7924
## 790    moralOther   moralOther mismatch        m     n   0 0.5726
## 791  immoralOther immoralOther    match        n     n   1 0.8526
## 792    moralOther   moralOther    match        n     m   0 0.6608
## 793   immoralSelf  immoralSelf mismatch        m     m   1 0.8517
## 794  immoralOther immoralOther    match        n     n   1 0.7318
## 795    moralOther   moralOther mismatch        m     m   1 0.7239
## 796     moralSelf    moralSelf    match        n     n   1 0.5501
## 797   immoralSelf  immoralSelf    match        n     n   1 0.9543
## 798  immoralOther immoralOther    match        n     n   1 0.9184
## 799    moralOther   moralOther    match        n     n   1 0.7604
## 800    moralOther   moralOther mismatch        m     m   1 0.6986
## 801     moralSelf    moralSelf mismatch        m     m   1 0.7847
## 802    moralOther   moralOther mismatch        m     n   0 0.5588
## 803   immoralSelf  immoralSelf mismatch        m     m   1 0.7849
## 804     moralSelf    moralSelf mismatch        m     n   0 0.6310
## 805   immoralSelf  immoralSelf mismatch        m     m   1 0.8032
## 806    moralOther   moralOther    match        n     n   1 0.8693
## 807     moralSelf    moralSelf    match        n     n   1 0.6174
## 808  immoralOther immoralOther    match        n     n   1 0.8956
## 809   immoralSelf  immoralSelf    match        n     n   1 0.6877
## 810   immoralSelf  immoralSelf    match        n     m   0 0.5997
## 811     moralSelf    moralSelf    match        n     n   1 0.5780
## 812  immoralOther immoralOther mismatch        m     m   1 0.8400
## 813  immoralOther immoralOther mismatch        m     n   0 0.6981
## 814  immoralOther immoralOther mismatch        m     m   1 0.7442
## 815     moralSelf    moralSelf mismatch        m     m   1 0.8383
## 816    moralOther   moralOther    match        n     n   1 0.8884
## 817    moralOther   moralOther mismatch        m     m   1 0.7686
## 818  immoralOther immoralOther mismatch        m  <NA>  -1 1.0330
## 819  immoralOther immoralOther mismatch        m     n   0 0.6928
## 820    moralOther   moralOther mismatch        m     n   0 0.8450
## 821   immoralSelf  immoralSelf mismatch        m     m   1 0.9511
## 822    moralOther   moralOther    match        n     m   0 0.7312
## 823     moralSelf    moralSelf    match        n     n   1 0.5393
## 824   immoralSelf  immoralSelf    match        n     m   0 0.7455
## 825    moralOther   moralOther    match        n     n   1 0.7196
## 826     moralSelf    moralSelf mismatch        m     m   1 0.7936
## 827   immoralSelf  immoralSelf mismatch        m     n   0 0.9178
## 828     moralSelf    moralSelf    match        n     n   1 0.5279
## 829    moralOther   moralOther    match        n     n   1 0.9600
## 830   immoralSelf  immoralSelf    match        n     n   1 0.6501
## 831    moralOther   moralOther mismatch        m     m   1 0.6902
## 832  immoralOther immoralOther    match        n     n   1 0.6664
## 833     moralSelf    moralSelf mismatch        m     m   1 0.9445
## 834   immoralSelf  immoralSelf mismatch        m     m   1 0.7826
## 835  immoralOther immoralOther    match        n     n   1 0.8048
## 836     moralSelf    moralSelf    match        n     n   1 0.6228
## 837  immoralOther immoralOther    match        n     m   0 0.5650
## 838   immoralSelf  immoralSelf    match        n     m   0 0.6791
## 839     moralSelf    moralSelf mismatch        m     m   1 0.8492
## 840  immoralOther immoralOther mismatch        m     n   0 0.9694
## 841    moralOther   moralOther    match        n     n   1 0.8856
## 842     moralSelf    moralSelf mismatch        m     m   1 0.6958
## 843    moralOther   moralOther mismatch        m     n   0 0.5759
## 844     moralSelf    moralSelf    match        n     n   1 0.4960
## 845    moralOther   moralOther    match        n     n   1 0.8721
## 846     moralSelf    moralSelf mismatch        m     m   1 0.6682
## 847    moralOther   moralOther mismatch        m     n   0 0.6563
## 848   immoralSelf  immoralSelf mismatch        m     n   0 0.6884
## 849     moralSelf    moralSelf    match        n     n   1 0.5605
## 850  immoralOther immoralOther mismatch        m     n   0 0.7867
## 851   immoralSelf  immoralSelf mismatch        m     m   1 0.7768
## 852   immoralSelf  immoralSelf mismatch        m     m   1 0.7689
## 853    moralOther   moralOther    match        n     n   1 0.7211
## 854   immoralSelf  immoralSelf    match        n     n   1 0.7231
## 855  immoralOther immoralOther mismatch        m  <NA>  -1 1.0330
## 856  immoralOther immoralOther    match        n     n   1 0.7774
## 857  immoralOther immoralOther    match        n     n   1 0.7995
## 858     moralSelf    moralSelf mismatch        m     m   1 0.6896
## 859   immoralSelf  immoralSelf    match        n     n   1 0.6257
## 860     moralSelf    moralSelf    match        n     n   1 0.6799
## 861  immoralOther immoralOther    match        n     n   1 0.8740
## 862    moralOther   moralOther mismatch        m     m   1 0.7181
## 863   immoralSelf  immoralSelf    match        n     n   1 0.7442
## 864  immoralOther immoralOther mismatch        m     m   1 0.8463
## 865     moralSelf    moralSelf mismatch        m     m   1 0.8724
## 866  immoralOther immoralOther mismatch        m     m   1 0.8446
## 867   immoralSelf  immoralSelf    match        n     n   1 0.7347
## 868    moralOther   moralOther mismatch        m     m   1 0.7668
## 869    moralOther   moralOther    match        n     m   0 0.8809
## 870   immoralSelf  immoralSelf    match        n     n   1 0.8110
## 871  immoralOther immoralOther mismatch        m     m   1 0.7392
## 872  immoralOther immoralOther    match        n     n   1 0.8333
## 873  immoralOther immoralOther mismatch        m     m   1 0.8374
## 874    moralOther   moralOther    match        n     n   1 0.8155
## 875     moralSelf    moralSelf    match        n     n   1 0.6557
## 876     moralSelf    moralSelf    match        n     n   1 0.6158
## 877   immoralSelf  immoralSelf mismatch        m     m   1 0.7119
## 878    moralOther   moralOther mismatch        m     m   1 0.7561
## 879  immoralOther immoralOther    match        n     n   1 0.8241
## 880    moralOther   moralOther mismatch        m     m   1 0.7103
## 881   immoralSelf  immoralSelf mismatch        m     n   0 0.7544
## 882     moralSelf    moralSelf mismatch        m     m   1 0.7186
## 883     moralSelf    moralSelf mismatch        m     m   1 0.8006
## 884  immoralOther immoralOther    match        n     m   0 0.8508
## 885   immoralSelf  immoralSelf    match        n  <NA>  -1 1.0330
## 886   immoralSelf  immoralSelf mismatch        m     m   1 0.6311
## 887     moralSelf    moralSelf    match        n     n   1 0.6112
## 888    moralOther   moralOther    match        n     n   1 0.8992
## 889     moralSelf    moralSelf mismatch        m     n   0 0.5694
## 890     moralSelf    moralSelf    match        n     n   1 0.5295
## 891    moralOther   moralOther mismatch        m     n   0 0.6576
## 892  immoralOther immoralOther    match        n     n   1 0.7997
## 893  immoralOther immoralOther mismatch        m     m   1 0.8738
## 894     moralSelf    moralSelf mismatch        m     n   0 0.6219
## 895  immoralOther immoralOther    match        n  <NA>  -1 1.0330
## 896   immoralSelf  immoralSelf    match        n     n   1 0.6163
## 897    moralOther   moralOther mismatch        m     n   0 0.4003
## 898   immoralSelf  immoralSelf mismatch        m     n   0 0.5944
## 899     moralSelf    moralSelf    match        n     n   1 0.3586
## 900     moralSelf    moralSelf mismatch        m     n   0 0.3286
## 901    moralOther   moralOther    match        n  <NA>  -1 1.0329
## 902    moralOther   moralOther mismatch        m     n   0 0.5888
## 903   immoralSelf  immoralSelf    match        n     n   1 0.6489
## 904    moralOther   moralOther    match        n     n   1 0.6251
## 905  immoralOther immoralOther    match        n     m   0 0.5591
## 906   immoralSelf  immoralSelf    match        n     n   1 0.5533
## 907  immoralOther immoralOther mismatch        m     m   1 0.5894
## 908    moralOther   moralOther    match        n     n   1 0.5535
## 909   immoralSelf  immoralSelf mismatch        m     m   1 0.6856
## 910     moralSelf    moralSelf    match        n     n   1 0.3437
## 911   immoralSelf  immoralSelf mismatch        m     n   0 0.3338
## 912  immoralOther immoralOther mismatch        m     m   1 0.3979
## 913     moralSelf    moralSelf    match        n     n   1 0.6184
## 914    moralOther   moralOther mismatch        m     m   1 0.7045
## 915  immoralOther immoralOther    match        n     n   1 0.8546
## 916    moralOther   moralOther    match        n     m   0 0.8307
## 917  immoralOther immoralOther    match        n     n   1 0.8988
## 918    moralOther   moralOther    match        n     n   1 0.6729
## 919  immoralOther immoralOther mismatch        m  <NA>  -1 1.0330
## 920   immoralSelf  immoralSelf    match        n     n   1 0.3612
## 921     moralSelf    moralSelf mismatch        m     n   0 0.4673
## 922  immoralOther immoralOther mismatch        m  <NA>  -1 1.0330
## 923     moralSelf    moralSelf    match        n     n   1 0.3876
## 924   immoralSelf  immoralSelf mismatch        m     m   1 0.5037
## 925  immoralOther immoralOther mismatch        m     n   0 0.6138
## 926  immoralOther immoralOther    match        n     m   0 0.3661
## 927   immoralSelf  immoralSelf    match        n     m   0 0.8239
## 928   immoralSelf  immoralSelf    match        n     n   1 0.6541
## 929     moralSelf    moralSelf    match        n     n   1 0.4462
## 930    moralOther   moralOther    match        n     n   1 0.8163
## 931     moralSelf    moralSelf mismatch        m     n   0 0.3604
## 932    moralOther   moralOther mismatch        m     n   0 0.2985
## 933    moralOther   moralOther mismatch        m     n   0 0.5847
## 934     moralSelf    moralSelf mismatch        m     n   0 0.4547
## 935   immoralSelf  immoralSelf mismatch        m     n   0 0.6468
## 936   immoralSelf  immoralSelf mismatch        m     m   1 0.8569
## 937     moralSelf    moralSelf mismatch        m     n   0 0.5531
## 938   immoralSelf  immoralSelf mismatch        m     m   1 0.3892
## 939     moralSelf    moralSelf    match        n     m   0 0.7853
## 940    moralOther   moralOther mismatch        m     n   0 0.7354
## 941     moralSelf    moralSelf mismatch        m     n   0 0.2396
## 942    moralOther   moralOther    match        n     m   0 0.2336
## 943    moralOther   moralOther mismatch        m     n   0 0.1657
## 944     moralSelf    moralSelf    match        n     n   1 0.3518
## 945  immoralOther immoralOther    match        n     m   0 0.8359
## 946   immoralSelf  immoralSelf    match        n     n   1 0.5600
## 947    moralOther   moralOther    match        n     n   1 0.7961
## 948  immoralOther immoralOther mismatch        m     n   0 0.7762
## 949   immoralSelf  immoralSelf    match        n     n   1 0.5844
## 950  immoralOther immoralOther    match        n     n   1 0.6064
## 951  immoralOther immoralOther    match        n     n   1 0.6166
## 952   immoralSelf  immoralSelf mismatch        m     n   0 0.8967
## 953   immoralSelf  immoralSelf    match        n     n   1 0.8248
## 954  immoralOther immoralOther mismatch        m     m   1 0.3910
## 955     moralSelf    moralSelf    match        n     n   1 0.6590
## 956   immoralSelf  immoralSelf mismatch        m     m   1 0.8172
## 957    moralOther   moralOther mismatch        m     n   0 0.8152
## 958    moralOther   moralOther    match        n     m   0 0.9114
## 959     moralSelf    moralSelf mismatch        m     m   1 0.7156
## 960  immoralOther immoralOther mismatch        m     n   0 0.7996
## 961   immoralSelf  immoralSelf mismatch        m     n   0 0.7499
## 962     moralSelf    moralSelf    match        n     n   1 0.4900
## 963     moralSelf    moralSelf mismatch        m     m   1 0.6101
## 964     moralSelf    moralSelf    match        n     n   1 0.5942
## 965    moralOther   moralOther    match        n     n   1 0.8384
## 966  immoralOther immoralOther mismatch        m     m   1 0.7285
## 967   immoralSelf  immoralSelf    match        n     n   1 0.7286
## 968  immoralOther immoralOther mismatch        m     n   0 0.8167
## 969  immoralOther immoralOther mismatch        m     m   1 0.6009
## 970   immoralSelf  immoralSelf mismatch        m     m   1 0.8590
## 971   immoralSelf  immoralSelf    match        n     n   1 0.6571
## 972     moralSelf    moralSelf    match        n     n   1 0.5913
## 973    moralOther   moralOther    match        n     n   1 0.8273
## 974    moralOther   moralOther mismatch        m  <NA>  -1 1.0330
## 975     moralSelf    moralSelf mismatch        m     m   1 0.6996
## 976     moralSelf    moralSelf mismatch        m     m   1 0.8417
## 977   immoralSelf  immoralSelf mismatch        m     m   1 0.7479
## 978  immoralOther immoralOther    match        n     n   1 0.8800
## 979    moralOther   moralOther mismatch        m     m   1 0.9300
## 980    moralOther   moralOther mismatch        m     m   1 0.8503
## 981   immoralSelf  immoralSelf    match        n     n   1 0.8264
## 982  immoralOther immoralOther    match        n     m   0 0.6724
## 983    moralOther   moralOther    match        n     m   0 0.8926
## 984  immoralOther immoralOther    match        n     n   1 0.7406
## 985    moralOther   moralOther    match        n     n   1 0.7388
## 986   immoralSelf  immoralSelf mismatch        m     m   1 0.7049
## 987  immoralOther immoralOther mismatch        m     m   1 0.8632
## 988   immoralSelf  immoralSelf    match        n     n   1 0.7332
## 989   immoralSelf  immoralSelf mismatch        m     m   1 0.6053
## 990    moralOther   moralOther    match        n     n   1 0.8073
## 991    moralOther   moralOther    match        n     n   1 0.8395
## 992   immoralSelf  immoralSelf    match        n     n   1 0.6816
## 993    moralOther   moralOther mismatch        m     m   1 0.6438
## 994  immoralOther immoralOther mismatch        m     n   0 0.7979
## 995     moralSelf    moralSelf mismatch        m     m   1 0.7040
## 996     moralSelf    moralSelf    match        n     n   1 0.6061
## 997     moralSelf    moralSelf    match        n     n   1 0.5042
## 998   immoralSelf  immoralSelf mismatch        m     m   1 0.8063
## 999  immoralOther immoralOther    match        n     n   1 0.7605
## 1000   moralOther   moralOther mismatch        m     m   1 0.8105
## 1001 immoralOther immoralOther    match        n     m   0 0.8107
## 1002    moralSelf    moralSelf mismatch        m     n   0 0.5948
## 1003   moralOther   moralOther mismatch        m  <NA>  -1 1.0330
## 1004 immoralOther immoralOther mismatch        m     m   1 0.9070
## 1005    moralSelf    moralSelf mismatch        m     m   1 0.7371
## 1006  immoralSelf  immoralSelf    match        n     n   1 0.6233
## 1007    moralSelf    moralSelf    match        n     n   1 0.5575
## 1008 immoralOther immoralOther    match        n     m   0 0.6475
## 1009 immoralOther immoralOther    match        n     n   1 0.6637
## 1010 immoralOther immoralOther mismatch        m     m   1 0.6937
## 1011    moralSelf    moralSelf mismatch        m     m   1 0.8138
## 1012    moralSelf    moralSelf    match        n     n   1 0.6781
## 1013    moralSelf    moralSelf mismatch        m     m   1 0.6721
## 1014    moralSelf    moralSelf    match        n     n   1 0.4602
## 1015  immoralSelf  immoralSelf mismatch        m     m   1 0.8503
## 1016   moralOther   moralOther mismatch        m     n   0 0.6745
## 1017  immoralSelf  immoralSelf mismatch        m     m   1 0.8146
## 1018  immoralSelf  immoralSelf mismatch        m     m   1 0.8387
## 1019 immoralOther immoralOther    match        n     m   0 0.8648
## 1020  immoralSelf  immoralSelf    match        n     m   0 0.8950
## 1021   moralOther   moralOther mismatch        m     n   0 0.7910
## 1022   moralOther   moralOther    match        n     n   1 0.6832
## 1023    moralSelf    moralSelf    match        n     n   1 0.5973
## 1024 immoralOther immoralOther mismatch        m     m   1 0.6154
## 1025   moralOther   moralOther    match        n  <NA>  -1 1.0330
## 1026  immoralSelf  immoralSelf    match        n     n   1 0.7797
## 1027  immoralSelf  immoralSelf    match        n     m   0 0.5458
## 1028 immoralOther immoralOther    match        n     n   1 0.6719
## 1029    moralSelf    moralSelf mismatch        m     m   1 0.7640
## 1030   moralOther   moralOther    match        n  <NA>  -1 1.0330
## 1031   moralOther   moralOther mismatch        m     m   1 0.8582
## 1032 immoralOther immoralOther mismatch        m     n   0 0.9364
## 1033   moralOther   moralOther mismatch        m     m   1 0.7833
## 1034   moralOther   moralOther    match        n     n   1 0.8454
## 1035    moralSelf    moralSelf mismatch        m     m   1 0.8475
## 1036   moralOther   moralOther mismatch        m     m   1 0.8077
## 1037  immoralSelf  immoralSelf    match        n     n   1 0.7918
## 1038 immoralOther immoralOther    match        n  <NA>  -1 1.0330
## 1039  immoralSelf  immoralSelf mismatch        m     m   1 0.7921
## 1040    moralSelf    moralSelf mismatch        m     m   1 0.8442
## 1041 immoralOther immoralOther mismatch        m     m   1 0.7824
## 1042  immoralSelf  immoralSelf mismatch        m     m   1 0.7885
## 1043 immoralOther immoralOther    match        n     n   1 0.7366
## 1044  immoralSelf  immoralSelf    match        n     m   0 0.8567
## 1045   moralOther   moralOther    match        n     n   1 0.7207
## 1046   moralOther   moralOther mismatch        m     m   1 0.7069
## 1047  immoralSelf  immoralSelf mismatch        m     n   0 1.0090
## 1048 immoralOther immoralOther mismatch        m     m   1 0.7611
## 1049 immoralOther immoralOther mismatch        m     m   1 0.7712
## 1050    moralSelf    moralSelf    match        n     n   1 0.5634
## 1051  immoralSelf  immoralSelf    match        n     n   1 0.7095
## 1052    moralSelf    moralSelf mismatch        m     m   1 0.8656
## 1053    moralSelf    moralSelf    match        n     n   1 0.6557
## 1054   moralOther   moralOther    match        n     n   1 0.8078
## 1055    moralSelf    moralSelf    match        n     n   1 0.5520
## 1056 immoralOther immoralOther    match        n     n   1 0.9321
## 1057   moralOther   moralOther    match        n     n   1 0.8122
## 1058   moralOther   moralOther mismatch        m     m   1 0.6444
## 1059 immoralOther immoralOther    match        n     n   1 0.6304
## 1060    moralSelf    moralSelf mismatch        m     n   0 0.5706
## 1061   moralOther   moralOther    match        n     n   1 0.7466
## 1062  immoralSelf  immoralSelf mismatch        m     m   1 0.6729
## 1063    moralSelf    moralSelf mismatch        m     m   1 0.6929
## 1064   moralOther   moralOther    match        n     n   1 0.5251
## 1065  immoralSelf  immoralSelf mismatch        m     m   1 0.6391
## 1066    moralSelf    moralSelf mismatch        m     n   0 0.5292
## 1067    moralSelf    moralSelf    match        n     n   1 0.5294
## 1068  immoralSelf  immoralSelf    match        n     m   0 0.8894
## 1069 immoralOther immoralOther    match        n     n   1 0.6815
## 1070   moralOther   moralOther mismatch        m     m   1 0.8597
## 1071 immoralOther immoralOther mismatch        m     n   0 0.8818
## 1072 immoralOther immoralOther mismatch        m     m   1 0.9719
## 1073   moralOther   moralOther mismatch        m     m   1 0.9641
## 1074 immoralOther immoralOther    match        n     n   1 0.7822
## 1075    moralSelf    moralSelf    match        n     n   1 0.5164
## 1076  immoralSelf  immoralSelf mismatch        m     m   1 0.5685
## 1077    moralSelf    moralSelf    match        n     n   1 0.4886
## 1078  immoralSelf  immoralSelf    match        n     n   1 0.6487
## 1079 immoralOther immoralOther mismatch        m  <NA>  -1 1.0330
## 1080  immoralSelf  immoralSelf    match        n     n   1 0.8849
## 1081  immoralSelf  immoralSelf    match        n     h   2 0.9487
## 1082  immoralSelf  immoralSelf mismatch        m     h   2 0.3929
## 1083 immoralOther immoralOther    match        n     h   2 0.1331
## 1084 immoralOther immoralOther    match        n     m   0 0.5330
## 1085 immoralOther immoralOther mismatch        m     m   1 0.6612
## 1086    moralSelf    moralSelf    match        n     n   1 0.4433
## 1087    moralSelf    moralSelf    match        n     n   1 0.5493
## 1088  immoralSelf  immoralSelf mismatch        m     m   1 0.7855
## 1089  immoralSelf  immoralSelf    match        n     n   1 0.6556
## 1090    moralSelf    moralSelf mismatch        m     n   0 0.6717
## 1091   moralOther   moralOther    match        n     n   1 0.7638
## 1092  immoralSelf  immoralSelf    match        n     n   1 0.6219
## 1093   moralOther   moralOther    match        n     n   1 0.7561
## 1094 immoralOther immoralOther mismatch        m     n   0 0.8002
## 1095   moralOther   moralOther    match        n     n   1 0.6344
## 1096   moralOther   moralOther mismatch        m     n   0 0.8464
## 1097   moralOther   moralOther mismatch        m     m   1 0.8325
## 1098   moralOther   moralOther mismatch        m     m   1 0.7486
## 1099    moralSelf    moralSelf mismatch        m     m   1 0.7688
## 1100    moralSelf    moralSelf    match        n     n   1 0.6010
## 1101    moralSelf    moralSelf mismatch        m     m   1 0.8271
## 1102  immoralSelf  immoralSelf mismatch        m     n   0 0.6412
## 1103 immoralOther immoralOther mismatch        m     m   1 0.6592
## 1104 immoralOther immoralOther    match        n     n   1 0.7454
## 1105 immoralOther immoralOther    match        n     n   1 0.7755
## 1106  immoralSelf  immoralSelf    match        n     n   1 0.6876
## 1107   moralOther   moralOther    match        n     n   1 0.7597
## 1108   moralOther   moralOther    match        n     n   1 0.5858
## 1109    moralSelf    moralSelf mismatch        m     m   1 0.6479
## 1110    moralSelf    moralSelf    match        n     n   1 0.5761
## 1111   moralOther   moralOther mismatch        m     m   1 0.7441
## 1112  immoralSelf  immoralSelf mismatch        m     m   1 0.8063
## 1113  immoralSelf  immoralSelf mismatch        m     m   1 0.8085
## 1114 immoralOther immoralOther mismatch        m     n   0 0.8165
## 1115 immoralOther immoralOther mismatch        m     m   1 0.6247
## 1116    moralSelf    moralSelf    match        n     n   1 0.5988
## 1117 immoralOther immoralOther mismatch        m     m   1 0.8249
## 1118    moralSelf    moralSelf mismatch        m     m   1 0.7590
## 1119 immoralOther immoralOther    match        n     m   0 0.8331
## 1120  immoralSelf  immoralSelf mismatch        m     m   1 0.6213
## 1121   moralOther   moralOther    match        n     m   0 0.7234
## 1122    moralSelf    moralSelf mismatch        m     n   0 0.9575
## 1123 immoralOther immoralOther    match        n     n   1 0.9036
## 1124  immoralSelf  immoralSelf    match        n     n   1 0.9577
## 1125  immoralSelf  immoralSelf    match        n     n   1 0.6399
## 1126   moralOther   moralOther mismatch        m     m   1 0.9140
## 1127   moralOther   moralOther mismatch        m     m   1 0.8381
## 1128    moralSelf    moralSelf    match        n     n   1 0.5202
## 1129    moralSelf    moralSelf mismatch        m     u   2 0.7173
## 1130 immoralOther immoralOther mismatch        m  <NA>  -1 1.0330
## 1131 immoralOther immoralOther mismatch        m     m   1 1.0034
## 1132 immoralOther immoralOther    match        n     n   1 0.9176
## 1133  immoralSelf  immoralSelf    match        n     n   1 0.9117
## 1134   moralOther   moralOther mismatch        m     m   1 0.6038
## 1135  immoralSelf  immoralSelf mismatch        m     n   0 0.7980
## 1136  immoralSelf  immoralSelf mismatch        m     m   1 0.8881
## 1137   moralOther   moralOther    match        n     n   1 0.7283
## 1138    moralSelf    moralSelf    match        n     n   1 0.5284
## 1139 immoralOther immoralOther    match        n     n   1 0.8084
## 1140 immoralOther immoralOther mismatch        m     n   0 0.6966
## 1141   moralOther   moralOther mismatch        m     m   1 0.7847
## 1142    moralSelf    moralSelf mismatch        m     n   0 0.8228
## 1143    moralSelf    moralSelf    match        n     n   1 0.5409
## 1144  immoralSelf  immoralSelf    match        n     m   0 0.8110
## 1145   moralOther   moralOther    match        n     n   1 0.8592
## 1146   moralOther   moralOther    match        n     n   1 0.7693
## 1147 immoralOther immoralOther    match        n     n   1 0.9294
## 1148  immoralSelf  immoralSelf    match        n     n   1 0.6736
## 1149   moralOther   moralOther mismatch        m     m   1 0.8376
## 1150    moralSelf    moralSelf mismatch        m     m   1 0.6238
## 1151    moralSelf    moralSelf    match        n     n   1 0.5499
## 1152  immoralSelf  immoralSelf mismatch        m     n   0 0.8100
## 1153    moralSelf    moralSelf mismatch        m     n   0 0.6981
## 1154  immoralSelf  immoralSelf    match        n     n   1 0.6402
## 1155   moralOther   moralOther    match        n     n   1 0.6104
## 1156   moralOther   moralOther    match        n     n   1 0.6904
## 1157  immoralSelf  immoralSelf mismatch        m     m   1 0.7706
## 1158 immoralOther immoralOther mismatch        m     m   1 0.8507
## 1159 immoralOther immoralOther    match        n     n   1 0.6588
## 1160  immoralSelf  immoralSelf mismatch        m     m   1 0.8089
## 1161 immoralOther immoralOther    match        n     m   0 1.0090
## 1162   moralOther   moralOther    match        n     n   1 0.6812
## 1163 immoralOther immoralOther mismatch        m     m   1 0.7313
## 1164  immoralSelf  immoralSelf    match        n     n   1 0.6594
## 1165    moralSelf    moralSelf mismatch        m     m   1 0.7935
## 1166 immoralOther immoralOther mismatch        m     n   0 0.9416
## 1167   moralOther   moralOther mismatch        m     n   0 0.8118
## 1168    moralSelf    moralSelf    match        n     n   1 0.6039
## 1169  immoralSelf  immoralSelf    match        n     n   1 0.9900
## 1170    moralSelf    moralSelf mismatch        m     m   1 0.7062
## 1171  immoralSelf  immoralSelf mismatch        m     m   1 0.8963
## 1172    moralSelf    moralSelf    match        n     n   1 0.5044
## 1173    moralSelf    moralSelf    match        n     n   1 0.4705
## 1174   moralOther   moralOther mismatch        m     n   0 0.6266
## 1175   moralOther   moralOther mismatch        m     m   1 0.6728
## 1176 immoralOther immoralOther    match        n     n   1 0.7168
## 1177   moralOther   moralOther mismatch        m     m   1 0.7423
## 1178 immoralOther immoralOther    match        n     n   1 0.7804
## 1179  immoralSelf  immoralSelf mismatch        m     n   0 0.5226
## 1180 immoralOther immoralOther mismatch        m  <NA>  -1 1.0330
## 1181  immoralSelf  immoralSelf mismatch        m     m   1 0.6509
## 1182 immoralOther immoralOther    match        n     n   1 0.6990
## 1183    moralSelf    moralSelf mismatch        m     n   0 0.8731
## 1184    moralSelf    moralSelf    match        n     n   1 0.5812
## 1185    moralSelf    moralSelf mismatch        m     m   1 0.7233
## 1186 immoralOther immoralOther    match        n     n   1 0.7334
## 1187  immoralSelf  immoralSelf    match        n     n   1 0.6375
## 1188  immoralSelf  immoralSelf    match        n     n   1 0.8076
## 1189    moralSelf    moralSelf mismatch        m     m   1 0.6638
## 1190  immoralSelf  immoralSelf mismatch        m     m   1 0.8779
## 1191    moralSelf    moralSelf    match        n     n   1 0.5281
## 1192   moralOther   moralOther    match        n     m   0 0.7281
## 1193    moralSelf    moralSelf    match        n     n   1 0.7043
## 1194 immoralOther immoralOther mismatch        m     m   1 0.8544
## 1195   moralOther   moralOther    match        n     n   1 0.7265
## 1196   moralOther   moralOther mismatch        m     m   1 0.7286
## 1197   moralOther   moralOther    match        n     n   1 0.6727
## 1198   moralOther   moralOther mismatch        m     m   1 0.7329
## 1199  immoralSelf  immoralSelf    match        n     n   1 0.6530
## 1200 immoralOther immoralOther mismatch        m     m   1 0.7650
## 1201 immoralOther immoralOther    match        n     n   1 0.8873
## 1202 immoralOther immoralOther mismatch        m     m   1 0.6393
## 1203  immoralSelf  immoralSelf    match        n     n   1 0.8495
## 1204  immoralSelf  immoralSelf mismatch        m     m   1 0.7616
## 1205 immoralOther immoralOther mismatch        m     m   1 0.7077
## 1206   moralOther   moralOther mismatch        m     m   1 0.7118
## 1207    moralSelf    moralSelf mismatch        m     m   1 0.7240
## 1208   moralOther   moralOther    match        n     n   1 0.6940
## 1209   moralOther   moralOther mismatch        m     m   1 0.6382
## 1210    moralSelf    moralSelf    match        n     n   1 0.5462
## 1211  immoralSelf  immoralSelf    match        n     n   1 0.9843
## 1212  immoralSelf  immoralSelf    match        n     n   1 0.6945
## 1213    moralSelf    moralSelf    match        n     n   1 0.7827
## 1214  immoralSelf  immoralSelf mismatch        m     m   1 0.8287
## 1215    moralSelf    moralSelf mismatch        m     m   1 0.9928
## 1216 immoralOther immoralOther    match        n     n   1 0.7650
## 1217   moralOther   moralOther    match        n     n   1 0.7751
## 1218    moralSelf    moralSelf    match        n     n   1 0.6633
## 1219  immoralSelf  immoralSelf mismatch        m     m   1 0.8293
## 1220    moralSelf    moralSelf mismatch        m     m   1 0.9134
## 1221   moralOther   moralOther    match        n     n   1 0.7976
## 1222 immoralOther immoralOther mismatch        m     m   1 0.7997
## 1223 immoralOther immoralOther    match        n  <NA>  -1 1.0330
## 1224   moralOther   moralOther mismatch        m     m   1 0.8060
## 1225  immoralSelf  immoralSelf    match        n     n   1 0.7516
## 1226  immoralSelf  immoralSelf    match        n     m   0 0.6277
## 1227   moralOther   moralOther mismatch        m  <NA>  -1 1.0330
## 1228 immoralOther immoralOther    match        n     n   1 0.8119
## 1229   moralOther   moralOther mismatch        m     n   0 0.6841
## 1230    moralSelf    moralSelf    match        n     n   1 0.5422
## 1231  immoralSelf  immoralSelf mismatch        m     m   1 0.6763
## 1232 immoralOther immoralOther mismatch        m     n   0 0.8004
## 1233    moralSelf    moralSelf    match        n     n   1 0.5306
## 1234   moralOther   moralOther mismatch        m     m   1 0.7787
## 1235   moralOther   moralOther    match        n     n   1 0.6807
## 1236  immoralSelf  immoralSelf mismatch        m     m   1 0.7149
## 1237   moralOther   moralOther    match        n     n   1 0.8470
## 1238 immoralOther immoralOther    match        n     n   1 0.7051
## 1239    moralSelf    moralSelf mismatch        m     m   1 0.6473
## 1240    moralSelf    moralSelf mismatch        m     m   1 0.6954
## 1241  immoralSelf  immoralSelf mismatch        m     m   1 0.7035
## 1242    moralSelf    moralSelf mismatch        m     m   1 0.5655
## 1243 immoralOther immoralOther mismatch        m     m   1 0.7117
## 1244  immoralSelf  immoralSelf    match        n     n   1 0.6438
## 1245 immoralOther immoralOther mismatch        m     m   1 0.6539
## 1246 immoralOther immoralOther    match        n     n   1 0.6940
## 1247   moralOther   moralOther    match        n     n   1 0.8941
## 1248    moralSelf    moralSelf    match        n     n   1 0.5663
## 1249    moralSelf    moralSelf    match        n     n   1 0.4564
## 1250  immoralSelf  immoralSelf    match        n     n   1 0.6085
## 1251 immoralOther immoralOther mismatch        m     n   0 0.9846
## 1252    moralSelf    moralSelf mismatch        m     n   0 0.4867
## 1253  immoralSelf  immoralSelf mismatch        m     n   0 0.5529
## 1254   moralOther   moralOther mismatch        m     n   0 0.7010
## 1255  immoralSelf  immoralSelf mismatch        m     n   0 0.7791
## 1256   moralOther   moralOther mismatch        m     m   1 0.7452
## 1257  immoralSelf  immoralSelf    match        n     n   1 0.6233
## 1258    moralSelf    moralSelf    match        n     n   1 0.4854
## 1259 immoralOther immoralOther mismatch        m     m   1 0.7355
## 1260    moralSelf    moralSelf    match        n     n   1 0.4076
## 1261  immoralSelf  immoralSelf mismatch        m     m   1 0.6797
## 1262    moralSelf    moralSelf mismatch        m     m   1 0.5698
## 1263  immoralSelf  immoralSelf    match        n     n   1 0.5860
## 1264    moralSelf    moralSelf mismatch        m     m   1 0.5840
## 1265 immoralOther immoralOther    match        n     n   1 0.7422
## 1266   moralOther   moralOther    match        n     n   1 0.6203
## 1267   moralOther   moralOther    match        n     n   1 0.5864
## 1268   moralOther   moralOther    match        n     n   1 0.6965
## 1269 immoralOther immoralOther mismatch        m     m   1 0.6406
## 1270 immoralOther immoralOther    match        n     n   1 0.7787
## 1271   moralOther   moralOther mismatch        m     n   0 0.6809
## 1272 immoralOther immoralOther    match        n     n   1 0.6028
## 1273 immoralOther immoralOther mismatch        m     m   1 0.7243
## 1274 immoralOther immoralOther    match        n     n   1 0.7824
## 1275   moralOther   moralOther    match        n     n   1 0.8925
## 1276  immoralSelf  immoralSelf    match        n     n   1 0.6287
## 1277    moralSelf    moralSelf    match        n     n   1 0.4248
## 1278    moralSelf    moralSelf    match        n     n   1 0.5749
## 1279 immoralOther immoralOther    match        n     m   0 0.6550
## 1280   moralOther   moralOther mismatch        m     m   1 0.8331
## 1281 immoralOther immoralOther    match        n     n   1 0.6532
## 1282  immoralSelf  immoralSelf mismatch        m     m   1 0.8213
## 1283 immoralOther immoralOther mismatch        m     m   1 0.7394
## 1284  immoralSelf  immoralSelf    match        n     n   1 0.5616
## 1285    moralSelf    moralSelf    match        n     n   1 0.4758
## 1286   moralOther   moralOther    match        n     n   1 0.7518
## 1287    moralSelf    moralSelf mismatch        m     m   1 0.7318
## 1288    moralSelf    moralSelf mismatch        m     m   1 0.8120
## 1289   moralOther   moralOther mismatch        m     n   0 0.7881
## 1290  immoralSelf  immoralSelf mismatch        m     m   1 0.9523
## 1291  immoralSelf  immoralSelf mismatch        m     m   1 0.7165
## 1292    moralSelf    moralSelf mismatch        m     m   1 0.6166
## 1293 immoralOther immoralOther mismatch        m     n   0 0.9446
## 1294  immoralSelf  immoralSelf    match        n     n   1 0.8307
## 1295   moralOther   moralOther    match        n     n   1 0.6188
## 1296   moralOther   moralOther mismatch        m     m   1 0.6410
## 1297  immoralSelf  immoralSelf    match        n     n   1 0.5392
## 1298    moralSelf    moralSelf mismatch        m     m   1 0.6732
## 1299    moralSelf    moralSelf    match        n     n   1 0.6933
## 1300   moralOther   moralOther mismatch        m     n   0 0.7495
## 1301    moralSelf    moralSelf mismatch        m     m   1 0.7615
## 1302   moralOther   moralOther mismatch        m     m   1 0.8597
## 1303    moralSelf    moralSelf    match        n     n   1 0.5298
## 1304  immoralSelf  immoralSelf mismatch        m     n   0 0.6180
## 1305 immoralOther immoralOther    match        n     m   0 0.7681
## 1306  immoralSelf  immoralSelf mismatch        m     m   1 0.7762
## 1307  immoralSelf  immoralSelf mismatch        m     m   1 0.9843
## 1308   moralOther   moralOther    match        n     n   1 0.8944
## 1309 immoralOther immoralOther    match        n     n   1 0.6225
## 1310 immoralOther immoralOther mismatch        m     m   1 0.6607
## 1311   moralOther   moralOther    match        n     n   1 0.5867
## 1312   moralOther   moralOther    match        n     n   1 0.5449
## 1313 immoralOther immoralOther mismatch        m     m   1 0.7510
## 1314    moralSelf    moralSelf mismatch        m     m   1 0.6511
## 1315  immoralSelf  immoralSelf    match        n     n   1 0.6512
## 1316    moralSelf    moralSelf    match        n     n   1 0.4514
## 1317 immoralOther immoralOther    match        n     m   0 0.7555
## 1318 immoralOther immoralOther mismatch        m     m   1 0.6316
## 1319  immoralSelf  immoralSelf    match        n     n   1 0.5957
## 1320   moralOther   moralOther mismatch        m     n   0 0.7678
## 1321   moralOther   moralOther    match        n     m   0 0.7289
## 1322   moralOther   moralOther mismatch        m     m   1 1.0604
## 1323    moralSelf    moralSelf mismatch        m     n   0 0.7264
## 1324  immoralSelf  immoralSelf    match        n     m   0 0.6498
## 1325   moralOther   moralOther mismatch        m     m   1 0.8315
## 1326  immoralSelf  immoralSelf mismatch        m     n   0 0.6529
## 1327    moralSelf    moralSelf mismatch        m     n   0 0.6906
## 1328 immoralOther immoralOther    match        n     n   1 0.6210
## 1329 immoralOther immoralOther    match        n     n   1 0.4502
## 1330 immoralOther immoralOther mismatch        m     m   1 0.5520
## 1331    moralSelf    moralSelf    match        n     m   0 0.7558
## 1332    moralSelf    moralSelf    match        n     n   1 0.7437
## 1333 immoralOther immoralOther mismatch        m     m   1 0.9672
## 1334 immoralOther immoralOther mismatch        m     m   1 0.5595
## 1335    moralSelf    moralSelf    match        n     n   1 0.9073
## 1336    moralSelf    moralSelf mismatch        m     m   1 0.6703
## 1337  immoralSelf  immoralSelf mismatch        m     n   0 0.7324
## 1338 immoralOther immoralOther    match        n     m   0 0.9838
## 1339  immoralSelf  immoralSelf    match        n     m   0 0.9402
## 1340  immoralSelf  immoralSelf mismatch        m     n   0 0.7318
## 1341   moralOther   moralOther mismatch        m     n   0 0.6713
## 1342  immoralSelf  immoralSelf    match        n     m   0 0.7254
## 1343   moralOther   moralOther    match        n     m   0 0.7886
## 1344   moralOther   moralOther    match        n     n   1 0.5731
## 1345    moralSelf    moralSelf mismatch        m     m   1 0.6693
## 1346 immoralOther immoralOther    match        n     n   1 0.8034
## 1347    moralSelf    moralSelf    match        n     n   1 0.8403
## 1348 immoralOther immoralOther    match        n     n   1 0.6418
## 1349 immoralOther immoralOther mismatch        m     m   1 0.6512
## 1350   moralOther   moralOther    match        n     n   1 0.5529
## 1351 immoralOther immoralOther mismatch        m     m   1 0.9888
## 1352  immoralSelf  immoralSelf mismatch        m     n   0 0.7895
## 1353  immoralSelf  immoralSelf    match        n     m   0 0.7501
## 1354  immoralSelf  immoralSelf    match        n     n   1 0.4898
## 1355    moralSelf    moralSelf    match        n     n   1 0.7723
## 1356  immoralSelf  immoralSelf mismatch        m     m   1 0.6284
## 1357  immoralSelf  immoralSelf    match        n     n   1 0.7937
## 1358   moralOther   moralOther    match        n     n   1 0.6024
## 1359    moralSelf    moralSelf mismatch        m     m   1 0.7911
## 1360  immoralSelf  immoralSelf mismatch        m     m   1 0.9196
## 1361   moralOther   moralOther mismatch        m     m   1 0.7988
## 1362   moralOther   moralOther    match        n     n   1 0.5796
## 1363    moralSelf    moralSelf    match        n     n   1 0.6159
## 1364   moralOther   moralOther mismatch        m     m   1 0.6091
## 1365 immoralOther immoralOther    match        n     n   1 0.8260
## 1366 immoralOther immoralOther mismatch        m     m   1 0.6713
## 1367   moralOther   moralOther mismatch        m     n   0 0.6135
## 1368    moralSelf    moralSelf mismatch        m     n   0 0.7265
## 1369    moralSelf    moralSelf mismatch        m     m   1 0.7139
## 1370   moralOther   moralOther mismatch        m     m   1 0.7329
## 1371   moralOther   moralOther    match        n     n   1 0.6883
## 1372  immoralSelf  immoralSelf mismatch        m     m   1 0.6267
## 1373   moralOther   moralOther    match        n     n   1 0.4640
## 1374  immoralSelf  immoralSelf    match        n     m   0 0.7620
## 1375 immoralOther immoralOther    match        n     n   1 0.7381
## 1376 immoralOther immoralOther    match        n     n   1 0.4534
## 1377  immoralSelf  immoralSelf mismatch        m     m   1 0.5792
## 1378  immoralSelf  immoralSelf    match        n     n   1 1.0715
## 1379    moralSelf    moralSelf    match        n     n   1 0.6138
## 1380    moralSelf    moralSelf mismatch        m     n   0 0.7427
## 1381   moralOther   moralOther    match        n     n   1 0.4624
## 1382   moralOther   moralOther mismatch        m     m   1 0.5444
## 1383  immoralSelf  immoralSelf mismatch        m     n   0 0.5759
## 1384 immoralOther immoralOther mismatch        m     m   1 0.9323
## 1385  immoralSelf  immoralSelf    match        n     n   1 0.7357
## 1386   moralOther   moralOther mismatch        m     m   1 0.7391
## 1387    moralSelf    moralSelf mismatch        m     n   0 0.5867
## 1388 immoralOther immoralOther mismatch        m     m   1 0.7154
## 1389 immoralOther immoralOther    match        n     n   1 0.6745
## 1390    moralSelf    moralSelf    match        n     m   0 0.8006
## 1391 immoralOther immoralOther mismatch        m     m   1 1.0534
## 1392    moralSelf    moralSelf    match        n     n   1 0.6832
## 1393    moralSelf    moralSelf    match        n     n   1 0.6172
## 1394 immoralOther immoralOther    match        n     n   1 0.6582
## 1395  immoralSelf  immoralSelf    match        n     n   1 0.8922
## 1396  immoralSelf  immoralSelf mismatch        m     m   1 0.7229
## 1397   moralOther   moralOther    match        n     n   1 0.6421
## 1398   moralOther   moralOther mismatch        m     m   1 0.6835
## 1399  immoralSelf  immoralSelf mismatch        m     m   1 1.0621
## 1400 immoralOther immoralOther mismatch        m     m   1 0.6961
## 1401 immoralOther immoralOther    match        n     n   1 0.5508
## 1402  immoralSelf  immoralSelf    match        n     m   0 0.7705
## 1403  immoralSelf  immoralSelf mismatch        m     m   1 0.7386
## 1404 immoralOther immoralOther mismatch        m     m   1 0.8983
## 1405   moralOther   moralOther    match        n     n   1 0.5449
## 1406  immoralSelf  immoralSelf    match        n     m   0 0.4485
## 1407    moralSelf    moralSelf mismatch        m     m   1 0.8623
## 1408   moralOther   moralOther    match        n     n   1 0.4563
## 1409   moralOther   moralOther mismatch        m     n   0 0.8103
## 1410    moralSelf    moralSelf mismatch        m     m   1 0.7714
## 1411   moralOther   moralOther mismatch        m     m   1 0.7154
## 1412 immoralOther immoralOther    match        n     n   1 0.6266
## 1413    moralSelf    moralSelf    match        n     n   1 0.7119
## 1414    moralSelf    moralSelf    match        n     n   1 0.7068
## 1415    moralSelf    moralSelf mismatch        m     m   1 1.0057
## 1416 immoralOther immoralOther mismatch        m     n   0 0.5865
## 1417   moralOther   moralOther    match        n     n   1 0.4510
## 1418  immoralSelf  immoralSelf    match        n     n   1 0.6968
## 1419  immoralSelf  immoralSelf mismatch        m     m   1 0.6555
## 1420    moralSelf    moralSelf    match        n     n   1 0.6213
## 1421  immoralSelf  immoralSelf    match        n     n   1 0.6105
## 1422  immoralSelf  immoralSelf    match        n     n   1 0.4676
## 1423 immoralOther immoralOther    match        n     n   1 0.6295
## 1424   moralOther   moralOther mismatch        m     m   1 0.6988
## 1425   moralOther   moralOther    match        n     n   1 0.4894
## 1426 immoralOther immoralOther mismatch        m     m   1 0.7638
## 1427  immoralSelf  immoralSelf mismatch        m     n   0 0.7400
## 1428    moralSelf    moralSelf mismatch        m     m   1 0.8757
## 1429 immoralOther immoralOther    match        n     n   1 0.4579
## 1430   moralOther   moralOther mismatch        m     n   0 0.5638
## 1431    moralSelf    moralSelf mismatch        m     n   0 0.6438
## 1432 immoralOther immoralOther    match        n     n   1 0.6694
## 1433   moralOther   moralOther mismatch        m     m   1 0.8516
## 1434    moralSelf    moralSelf mismatch        m     m   1 0.8173
## 1435  immoralSelf  immoralSelf mismatch        m  <NA>  -1 1.0841
## 1436    moralSelf    moralSelf    match        n     n   1 0.6689
## 1437 immoralOther immoralOther mismatch        m     m   1 0.7391
## 1438    moralSelf    moralSelf    match        n     n   1 0.5148
## 1439   moralOther   moralOther    match        n     n   1 0.6658
## 1440 immoralOther immoralOther mismatch        m     m   1 0.6598
## 1441 immoralOther immoralOther    match        n     n   1 0.6069
## 1442   moralOther   moralOther mismatch        m     m   1 0.6117
## 1443    moralSelf    moralSelf    match        n     n   1 0.6447
## 1444   moralOther   moralOther mismatch        m     m   1 0.5545
## 1445   moralOther   moralOther    match        n     n   1 0.6704
## 1446  immoralSelf  immoralSelf    match        n     n   1 0.7965
## 1447 immoralOther immoralOther mismatch        m     m   1 0.5410
## 1448  immoralSelf  immoralSelf    match        n     n   1 0.5088
## 1449  immoralSelf  immoralSelf mismatch        m     m   1 0.8636
## 1450    moralSelf    moralSelf    match        n     n   1 0.5297
## 1451  immoralSelf  immoralSelf mismatch        m     m   1 0.6849
## 1452 immoralOther immoralOther    match        n     n   1 0.5274
## 1453    moralSelf    moralSelf mismatch        m     m   1 0.6428
## 1454  immoralSelf  immoralSelf    match        n     n   1 0.5563
## 1455   moralOther   moralOther mismatch        m     m   1 0.6322
## 1456    moralSelf    moralSelf mismatch        m     m   1 0.6775
## 1457 immoralOther immoralOther mismatch        m     n   0 0.3957
## 1458    moralSelf    moralSelf    match        n     n   1 0.5803
## 1459    moralSelf    moralSelf mismatch        m     m   1 0.6006
## 1460   moralOther   moralOther    match        n     n   1 0.5974
## 1461  immoralSelf  immoralSelf mismatch        m     m   1 0.6501
## 1462 immoralOther immoralOther    match        n     n   1 0.6479
## 1463 immoralOther immoralOther mismatch        m     n   0 0.6456
## 1464   moralOther   moralOther    match        n     n   1 0.5314
## 1465   moralOther   moralOther    match        n     m   0 0.5587
## 1466   moralOther   moralOther mismatch        m     m   1 0.7626
## 1467    moralSelf    moralSelf    match        n     n   1 0.5788
## 1468  immoralSelf  immoralSelf    match        n     n   1 0.5591
## 1469  immoralSelf  immoralSelf    match        n     n   1 0.6669
## 1470 immoralOther immoralOther mismatch        m     m   1 0.5568
## 1471   moralOther   moralOther mismatch        m     m   1 0.6239
## 1472    moralSelf    moralSelf mismatch        m     n   0 0.5660
## 1473 immoralOther immoralOther mismatch        m     m   1 0.6221
## 1474   moralOther   moralOther    match        n     n   1 0.6434
## 1475  immoralSelf  immoralSelf mismatch        m     m   1 0.7728
## 1476    moralSelf    moralSelf mismatch        m     m   1 0.6530
## 1477 immoralOther immoralOther mismatch        m     m   1 0.5707
## 1478    moralSelf    moralSelf    match        n     n   1 0.5708
## 1479   moralOther   moralOther mismatch        m     n   0 0.5948
## 1480    moralSelf    moralSelf    match        n     n   1 0.4194
## 1481 immoralOther immoralOther    match        n     n   1 0.7565
## 1482    moralSelf    moralSelf mismatch        m     m   1 0.7203
## 1483  immoralSelf  immoralSelf    match        n     n   1 0.6554
## 1484   moralOther   moralOther    match        n     n   1 0.5332
## 1485 immoralOther immoralOther    match        n     m   0 0.5046
## 1486  immoralSelf  immoralSelf mismatch        m     n   0 0.4355
## 1487 immoralOther immoralOther    match        n     n   1 0.5170
## 1488  immoralSelf  immoralSelf mismatch        m     n   0 0.6922
## 1489 immoralOther immoralOther    match        n     n   1 0.5707
## 1490   moralOther   moralOther mismatch        m     m   1 0.7790
## 1491  immoralSelf  immoralSelf    match        n     n   1 0.6074
## 1492 immoralOther immoralOther mismatch        m     n   0 0.5161
## 1493   moralOther   moralOther mismatch        m     m   1 0.6913
## 1494   moralOther   moralOther    match        n     m   0 0.8179
## 1495 immoralOther immoralOther    match        n     n   1 0.6349
## 1496    moralSelf    moralSelf mismatch        m     m   1 0.7085
## 1497  immoralSelf  immoralSelf mismatch        m     m   1 0.7994
## 1498  immoralSelf  immoralSelf mismatch        m     n   0 0.5882
## 1499    moralSelf    moralSelf mismatch        m     n   0 0.8607
## 1500    moralSelf    moralSelf    match        n     m   0 0.7027
## 1501    moralSelf    moralSelf mismatch        m     m   1 0.6854
## 1502 immoralOther immoralOther mismatch        m     m   1 0.7199
## 1503    moralSelf    moralSelf    match        n     n   1 0.7190
## 1504  immoralSelf  immoralSelf    match        n     n   1 0.3982
## 1505 immoralOther immoralOther    match        n     n   1 0.5550
## 1506  immoralSelf  immoralSelf mismatch        m     m   1 0.6469
## 1507 immoralOther immoralOther mismatch        m     m   1 0.6766
## 1508   moralOther   moralOther    match        n     n   1 0.6028
## 1509  immoralSelf  immoralSelf    match        n     n   1 0.5036
## 1510   moralOther   moralOther    match        n     n   1 0.5225
## 1511   moralOther   moralOther mismatch        m     m   1 0.5137
## 1512    moralSelf    moralSelf    match        n     n   1 0.4845
## 1513    moralSelf    moralSelf    match        n     n   1 0.4949
## 1514   moralOther   moralOther mismatch        m     m   1 0.6295
## 1515  immoralSelf  immoralSelf    match        n     n   1 0.5230
## 1516   moralOther   moralOther mismatch        m     n   0 0.6740
## 1517 immoralOther immoralOther mismatch        m     m   1 0.5363
## 1518    moralSelf    moralSelf mismatch        m     m   1 0.7158
## 1519 immoralOther immoralOther    match        n     n   1 0.5149
## 1520   moralOther   moralOther    match        n     n   1 0.6259
## 1521 immoralOther immoralOther mismatch        m     n   0 0.6071
## 1522   moralOther   moralOther mismatch        m     m   1 0.5800
## 1523    moralSelf    moralSelf    match        n     n   1 0.4724
## 1524 immoralOther immoralOther    match        n     n   1 0.5505
## 1525  immoralSelf  immoralSelf mismatch        m     m   1 0.6344
## 1526  immoralSelf  immoralSelf    match        n     n   1 0.3799
## 1527    moralSelf    moralSelf mismatch        m     m   1 0.5682
## 1528   moralOther   moralOther    match        n     n   1 0.4964
## 1529   moralOther   moralOther    match        n     n   1 0.6070
## 1530  immoralSelf  immoralSelf    match        n     n   1 0.4438
## 1531  immoralSelf  immoralSelf mismatch        m     m   1 0.5813
## 1532    moralSelf    moralSelf    match        n     n   1 0.4336
## 1533  immoralSelf  immoralSelf mismatch        m     m   1 0.5872
## 1534 immoralOther immoralOther mismatch        m     n   0 0.4197
## 1535 immoralOther immoralOther    match        n     n   1 0.6287
## 1536    moralSelf    moralSelf mismatch        m     m   1 0.7060
## 1537    moralSelf    moralSelf mismatch        m     n   0 0.4529
## 1538   moralOther   moralOther    match        n     n   1 0.7226
## 1539 immoralOther immoralOther    match        n     n   1 0.5619
## 1540   moralOther   moralOther mismatch        m     m   1 0.5300
## 1541   moralOther   moralOther    match        n     n   1 0.4613
## 1542  immoralSelf  immoralSelf    match        n     n   1 0.6072
## 1543   moralOther   moralOther mismatch        m     m   1 0.7160
## 1544  immoralSelf  immoralSelf mismatch        m     n   0 0.5551
## 1545  immoralSelf  immoralSelf    match        n     n   1 0.5750
## 1546    moralSelf    moralSelf    match        n     n   1 0.5314
## 1547  immoralSelf  immoralSelf    match        n     n   1 0.5267
## 1548   moralOther   moralOther    match        n     n   1 0.4822
## 1549    moralSelf    moralSelf mismatch        m     n   0 0.5807
## 1550 immoralOther immoralOther mismatch        m     m   1 0.6648
## 1551 immoralOther immoralOther mismatch        m     m   1 0.7148
## 1552 immoralOther immoralOther mismatch        m     m   1 0.5220
## 1553    moralSelf    moralSelf    match        n     n   1 0.4732
## 1554    moralSelf    moralSelf mismatch        m     m   1 0.8073
## 1555  immoralSelf  immoralSelf mismatch        m     m   1 0.7321
## 1556  immoralSelf  immoralSelf mismatch        m     n   0 0.4955
## 1557    moralSelf    moralSelf    match        n     n   1 0.5743
## 1558   moralOther   moralOther mismatch        m     m   1 0.7706
## 1559 immoralOther immoralOther    match        n     n   1 0.7787
## 1560 immoralOther immoralOther    match        n     n   1 0.5351
## 1561  immoralSelf  immoralSelf    match        n     n   1 0.5710
## 1562 immoralOther immoralOther    match        n     n   1 0.5871
## 1563 immoralOther immoralOther    match        n     n   1 0.4836
## 1564 immoralOther immoralOther mismatch        m     n   0 0.4461
## 1565   moralOther   moralOther    match        n     n   1 0.5717
## 1566 immoralOther immoralOther mismatch        m     m   1 0.6439
## 1567  immoralSelf  immoralSelf mismatch        m     n   0 0.6695
## 1568   moralOther   moralOther mismatch        m     m   1 0.6996
## 1569   moralOther   moralOther mismatch        m     m   1 0.8583
## 1570  immoralSelf  immoralSelf mismatch        m     m   1 0.6283
## 1571    moralSelf    moralSelf mismatch        m     m   1 0.6095
## 1572  immoralSelf  immoralSelf    match        n     n   1 0.6026
## 1573 immoralOther immoralOther mismatch        m     m   1 0.7434
## 1574    moralSelf    moralSelf    match        n     n   1 0.6152
## 1575    moralSelf    moralSelf    match        n     m   0 0.5921
## 1576   moralOther   moralOther    match        n     n   1 0.7366
## 1577    moralSelf    moralSelf mismatch        m     m   1 0.6359
## 1578   moralOther   moralOther mismatch        m     m   1 0.6616
## 1579  immoralSelf  immoralSelf    match        n     n   1 0.6394
## 1580    moralSelf    moralSelf    match        n     n   1 0.5692
## 1581  immoralSelf  immoralSelf mismatch        m     m   1 0.6412
## 1582   moralOther   moralOther    match        n     n   1 0.5547
## 1583 immoralOther immoralOther    match        n     n   1 0.6065
## 1584    moralSelf    moralSelf mismatch        m     n   0 0.5633
## 1585  immoralSelf  immoralSelf    match        n     n   1 0.5713
## 1586 immoralOther immoralOther mismatch        m     m   1 0.7154
## 1587    moralSelf    moralSelf    match        n     n   1 0.5305
## 1588  immoralSelf  immoralSelf    match        n     n   1 0.4860
## 1589    moralSelf    moralSelf    match        n     n   1 0.4965
## 1590 immoralOther immoralOther mismatch        m     m   1 0.5512
## 1591  immoralSelf  immoralSelf    match        n     n   1 0.4509
## 1592    moralSelf    moralSelf mismatch        m     m   1 0.7527
## 1593   moralOther   moralOther mismatch        m     m   1 0.7006
## 1594  immoralSelf  immoralSelf mismatch        m     m   1 0.6433
## 1595   moralOther   moralOther    match        n     n   1 1.0367
## 1596   moralOther   moralOther    match        n     n   1 0.4863
## 1597   moralOther   moralOther mismatch        m     m   1 0.5207
## 1598  immoralSelf  immoralSelf mismatch        m     m   1 0.7439
## 1599    moralSelf    moralSelf    match        n     n   1 0.4795
## 1600    moralSelf    moralSelf mismatch        m     m   1 0.5140
## 1601 immoralOther immoralOther    match        n     n   1 0.5408
## 1602  immoralSelf  immoralSelf mismatch        m     m   1 0.9963
## 1603   moralOther   moralOther    match        n     n   1 0.5490
## 1604   moralOther   moralOther mismatch        m     m   1 0.6687
## 1605 immoralOther immoralOther mismatch        m     m   1 0.6750
## 1606 immoralOther immoralOther    match        n     n   1 0.6250
## 1607 immoralOther immoralOther    match        n     n   1 0.7584
## 1608    moralSelf    moralSelf mismatch        m     m   1 0.9303
## 1609    moralSelf    moralSelf    match        n     n   1 0.4616
## 1610  immoralSelf  immoralSelf    match        n     n   1 0.3913
## 1611 immoralOther immoralOther    match        n     n   1 0.4882
## 1612  immoralSelf  immoralSelf mismatch        m     n   0 0.6426
## 1613    moralSelf    moralSelf    match        n     n   1 0.4521
## 1614   moralOther   moralOther mismatch        m     m   1 0.7139
## 1615   moralOther   moralOther mismatch        m     m   1 0.7327
## 1616 immoralOther immoralOther mismatch        m     m   1 0.6482
## 1617   moralOther   moralOther    match        n     n   1 0.5099
## 1618  immoralSelf  immoralSelf mismatch        m     m   1 0.7768
## 1619 immoralOther immoralOther    match        n     n   1 0.6051
## 1620    moralSelf    moralSelf mismatch        m     m   1 0.6019
## 1621 immoralOther immoralOther    match        n     n   1 0.5107
## 1622   moralOther   moralOther mismatch        m     m   1 0.7536
## 1623  immoralSelf  immoralSelf    match        n     n   1 0.6215
## 1624    moralSelf    moralSelf mismatch        m     m   1 0.6588
## 1625  immoralSelf  immoralSelf    match        n     n   1 0.5967
## 1626   moralOther   moralOther    match        n     n   1 0.6534
## 1627 immoralOther immoralOther mismatch        m     m   1 0.6190
## 1628    moralSelf    moralSelf    match        n     n   1 0.5321
## 1629  immoralSelf  immoralSelf mismatch        m     m   1 0.6555
## 1630   moralOther   moralOther    match        n     n   1 0.7654
## 1631 immoralOther immoralOther mismatch        m     m   1 0.6053
## 1632    moralSelf    moralSelf mismatch        m     m   1 0.8981
## 1633    moralSelf    moralSelf    match        n     n   1 0.5100
## 1634   moralOther   moralOther mismatch        m     m   1 0.8489
## 1635 immoralOther immoralOther mismatch        m     m   1 0.6626
## 1636    moralSelf    moralSelf    match        n     n   1 0.5446
## 1637   moralOther   moralOther mismatch        m     m   1 0.5363
## 1638    moralSelf    moralSelf    match        n     n   1 0.4036
## 1639   moralOther   moralOther mismatch        m     m   1 0.5204
## 1640   moralOther   moralOther    match        n     m   0 0.6395
## 1641  immoralSelf  immoralSelf    match        n     n   1 0.6811
## 1642  immoralSelf  immoralSelf mismatch        m     m   1 0.7714
## 1643 immoralOther immoralOther mismatch        m     m   1 0.6436
## 1644  immoralSelf  immoralSelf mismatch        m     m   1 0.6851
## 1645    moralSelf    moralSelf mismatch        m     m   1 0.7596
## 1646   moralOther   moralOther    match        n     n   1 0.7394
## 1647  immoralSelf  immoralSelf mismatch        m     m   1 0.7311
## 1648    moralSelf    moralSelf mismatch        m     m   1 0.6624
## 1649  immoralSelf  immoralSelf    match        n     n   1 0.6564
## 1650 immoralOther immoralOther    match        n     n   1 0.7544
## 1651    moralSelf    moralSelf mismatch        m     m   1 0.6943
## 1652 immoralOther immoralOther mismatch        m     m   1 0.7970
## 1653  immoralSelf  immoralSelf    match        n     n   1 0.4374
## 1654 immoralOther immoralOther    match        n     n   1 0.6550
## 1655   moralOther   moralOther    match        n     n   1 0.5247
## 1656 immoralOther immoralOther    match        n     n   1 0.6599
## 1657 immoralOther immoralOther    match        n     n   1 0.4299
## 1658   moralOther   moralOther    match        n     n   1 0.6032
## 1659   moralOther   moralOther mismatch        m     m   1 0.8559
## 1660 immoralOther immoralOther mismatch        m     m   1 0.7619
## 1661 immoralOther immoralOther mismatch        m     m   1 0.8020
## 1662   moralOther   moralOther mismatch        m     m   1 0.5669
## 1663   moralOther   moralOther mismatch        m     n   0 0.5270
## 1664    moralSelf    moralSelf mismatch        m     m   1 0.6903
## 1665   moralOther   moralOther    match        n     n   1 0.6048
## 1666  immoralSelf  immoralSelf    match        n     n   1 0.6495
## 1667    moralSelf    moralSelf mismatch        m     m   1 0.6153
## 1668 immoralOther immoralOther    match        n     n   1 0.6004
## 1669  immoralSelf  immoralSelf mismatch        m     m   1 0.7411
## 1670 immoralOther immoralOther mismatch        m     m   1 0.6447
## 1671   moralOther   moralOther    match        n     n   1 0.5145
## 1672    moralSelf    moralSelf mismatch        m     m   1 0.5494
## 1673    moralSelf    moralSelf    match        n     n   1 0.6371
## 1674 immoralOther immoralOther    match        n     n   1 0.5986
## 1675  immoralSelf  immoralSelf mismatch        m     m   1 0.9555
## 1676  immoralSelf  immoralSelf    match        n     n   1 0.5352
## 1677  immoralSelf  immoralSelf    match        n     n   1 0.5226
## 1678    moralSelf    moralSelf    match        n     n   1 0.6178
## 1679  immoralSelf  immoralSelf mismatch        m     m   1 0.6268
## 1680    moralSelf    moralSelf    match        n     n   1 0.4080
## 1681  immoralSelf  immoralSelf    match        n     y   2 0.9544
## 1682    moralSelf    moralSelf mismatch        m     n   0 1.0304
## 1683    moralSelf    moralSelf    match        n  <NA>  -1 1.0841
## 1684 immoralOther immoralOther mismatch        m     m   1 0.9421
## 1685   moralOther   moralOther    match        n     n   1 0.5419
## 1686  immoralSelf  immoralSelf mismatch        m     m   1 0.7094
## 1687  immoralSelf  immoralSelf mismatch        m     m   1 1.0244
## 1688    moralSelf    moralSelf    match        n     n   1 0.6536
## 1689 immoralOther immoralOther    match        n     n   1 0.7313
## 1690   moralOther   moralOther mismatch        m     n   0 0.5108
## 1691   moralOther   moralOther    match        n     n   1 0.3937
## 1692    moralSelf    moralSelf mismatch        m     m   1 0.7382
## 1693  immoralSelf  immoralSelf mismatch        m     m   1 0.7378
## 1694 immoralOther immoralOther    match        n     n   1 0.5493
## 1695  immoralSelf  immoralSelf    match        n     m   0 0.8129
## 1696    moralSelf    moralSelf mismatch        m     m   1 0.9099
## 1697  immoralSelf  immoralSelf    match        n     n   1 0.5608
## 1698   moralOther   moralOther    match        n     n   1 0.4207
## 1699 immoralOther immoralOther    match        n     n   1 0.5779
## 1700    moralSelf    moralSelf    match        n     n   1 0.6063
## 1701   moralOther   moralOther mismatch        m     n   0 0.5230
## 1702 immoralOther immoralOther mismatch        m     m   1 0.9462
## 1703   moralOther   moralOther mismatch        m     m   1 0.7940
## 1704 immoralOther immoralOther mismatch        m     m   1 0.7144
## 1705 immoralOther immoralOther mismatch        m     m   1 0.9893
## 1706   moralOther   moralOther    match        n     n   1 0.4779
## 1707 immoralOther immoralOther    match        n     n   1 0.6441
## 1708    moralSelf    moralSelf    match        n     n   1 0.5939
## 1709  immoralSelf  immoralSelf    match        n     n   1 0.6824
## 1710   moralOther   moralOther    match        n     n   1 0.5169
## 1711   moralOther   moralOther    match        n     n   1 0.5720
## 1712    moralSelf    moralSelf    match        n     n   1 0.5241
## 1713  immoralSelf  immoralSelf mismatch        m     m   1 0.6993
## 1714    moralSelf    moralSelf mismatch        m     m   1 0.7140
## 1715    moralSelf    moralSelf mismatch        m     m   1 0.6529
## 1716  immoralSelf  immoralSelf mismatch        m     m   1 0.7548
## 1717   moralOther   moralOther mismatch        m     m   1 0.6465
## 1718    moralSelf    moralSelf mismatch        m     m   1 0.6122
## 1719 immoralOther immoralOther mismatch        m     m   1 0.7173
## 1720   moralOther   moralOther mismatch        m     m   1 0.7644
## 1721    moralSelf    moralSelf    match        n     n   1 0.8365
## 1722  immoralSelf  immoralSelf    match        n     m   0 1.0061
## 1723 immoralOther immoralOther    match        n     n   1 0.6348
## 1724   moralOther   moralOther mismatch        m     m   1 0.7324
## 1725 immoralOther immoralOther mismatch        m     m   1 0.6877
## 1726 immoralOther immoralOther    match        n     n   1 0.9622
## 1727  immoralSelf  immoralSelf    match        n     n   1 0.6423
## 1728  immoralSelf  immoralSelf mismatch        m     m   1 0.8118
## 1729  immoralSelf  immoralSelf    match        n     j   2 0.9661
## 1730   moralOther   moralOther mismatch        m  <NA>  -1 1.0841
## 1731   moralOther   moralOther mismatch        m     n   0 0.7687
## 1732  immoralSelf  immoralSelf    match        n     n   1 0.6409
## 1733   moralOther   moralOther    match        n     n   1 0.6264
## 1734    moralSelf    moralSelf mismatch        m     m   1 0.6956
## 1735  immoralSelf  immoralSelf    match        n     n   1 0.5743
## 1736    moralSelf    moralSelf mismatch        m     m   1 0.7467
## 1737    moralSelf    moralSelf mismatch        m     n   0 0.7542
## 1738 immoralOther immoralOther    match        n     n   1 0.6781
## 1739 immoralOther immoralOther mismatch        m     m   1 0.6284
## 1740 immoralOther immoralOther mismatch        m     m   1 0.8576
## 1741    moralSelf    moralSelf    match        n     n   1 0.9556
## 1742  immoralSelf  immoralSelf mismatch        m     m   1 0.7275
## 1743    moralSelf    moralSelf    match        n     n   1 0.4989
## 1744 immoralOther immoralOther    match        n     n   1 0.5295
## 1745   moralOther   moralOther    match        n     n   1 0.8210
## 1746    moralSelf    moralSelf    match        n     n   1 0.5943
## 1747   moralOther   moralOther    match        n     n   1 0.5308
## 1748 immoralOther immoralOther mismatch        m  <NA>  -1 1.0841
## 1749 immoralOther immoralOther    match        n     n   1 0.7607
## 1750   moralOther   moralOther mismatch        m     m   1 0.6605
## 1751  immoralSelf  immoralSelf mismatch        m     m   1 0.8465
## 1752  immoralSelf  immoralSelf mismatch        m     m   1 0.6842
## 1753  immoralSelf  immoralSelf    match        n     n   1 0.6547
## 1754 immoralOther immoralOther mismatch        m     m   1 0.8044
## 1755 immoralOther immoralOther mismatch        m     m   1 0.7772
## 1756   moralOther   moralOther    match        n     n   1 0.9256
## 1757  immoralSelf  immoralSelf mismatch        m  <NA>  -1 1.0841
## 1758  immoralSelf  immoralSelf mismatch        m  <NA>  -1 1.0841
## 1759 immoralOther immoralOther    match        n     n   1 0.6416
## 1760    moralSelf    moralSelf    match        n     n   1 0.5233
## 1761  immoralSelf  immoralSelf    match        n     n   1 0.6583
## 1762  immoralSelf  immoralSelf    match        n     n   1 0.5324
## 1763    moralSelf    moralSelf mismatch        m     m   1 0.5837
## 1764    moralSelf    moralSelf    match        n     n   1 0.3800
## 1765    moralSelf    moralSelf mismatch        m     m   1 0.5683
## 1766    moralSelf    moralSelf mismatch        m     m   1 0.7125
## 1767  immoralSelf  immoralSelf mismatch        m     m   1 0.7154
## 1768   moralOther   moralOther mismatch        m     m   1 0.8505
## 1769   moralOther   moralOther mismatch        m     m   1 0.8163
## 1770 immoralOther immoralOther mismatch        m  <NA>  -1 1.0841
## 1771   moralOther   moralOther mismatch        m  <NA>  -1 1.0841
## 1772 immoralOther immoralOther    match        n     n   1 0.5423
## 1773   moralOther   moralOther    match        n     n   1 0.7178
## 1774 immoralOther immoralOther    match        n     n   1 0.9010
## 1775    moralSelf    moralSelf    match        n     n   1 0.4837
## 1776   moralOther   moralOther    match        n     n   1 0.4221
## 1777   moralOther   moralOther    match        n     n   1 0.8235
## 1778    moralSelf    moralSelf mismatch        m     m   1 0.9409
## 1779   moralOther   moralOther mismatch        m  <NA>  -1 1.0841
## 1780   moralOther   moralOther mismatch        m     m   1 1.0070
## 1781 immoralOther immoralOther mismatch        m     m   1 0.7878
## 1782   moralOther   moralOther mismatch        m     m   1 0.5724
## 1783 immoralOther immoralOther    match        n     n   1 0.6286
## 1784 immoralOther immoralOther mismatch        m     m   1 0.4977
## 1785  immoralSelf  immoralSelf mismatch        m     n   0 0.6325
## 1786 immoralOther immoralOther    match        n     m   0 0.5059
## 1787  immoralSelf  immoralSelf    match        n     n   1 0.5648
## 1788    moralSelf    moralSelf mismatch        m  <NA>  -1 1.0841
## 1789 immoralOther immoralOther    match        n     n   1 0.7515
## 1790    moralSelf    moralSelf mismatch        m     m   1 0.9873
## 1791  immoralSelf  immoralSelf    match        n  <NA>  -1 1.0841
## 1792 immoralOther immoralOther mismatch        m  <NA>  -1 1.0841
## 1793  immoralSelf  immoralSelf    match        n     n   1 0.5208
## 1794  immoralSelf  immoralSelf mismatch        m     m   1 0.6639
## 1795  immoralSelf  immoralSelf mismatch        m     n   0 0.8899
## 1796   moralOther   moralOther    match        n     n   1 0.5724
## 1797    moralSelf    moralSelf    match        n     n   1 0.5485
## 1798    moralSelf    moralSelf    match        n     n   1 0.6200
## 1799    moralSelf    moralSelf    match        n     n   1 0.4771
## 1800   moralOther   moralOther    match        n     n   1 0.5794
## 1801  immoralSelf  immoralSelf mismatch        m     m   1 0.6238
## 1802  immoralSelf  immoralSelf    match        n     n   1 0.9089
## 1803   moralOther   moralOther mismatch        m     n   0 1.0558
## 1804 immoralOther immoralOther    match        n     n   1 0.8738
## 1805 immoralOther immoralOther mismatch        m     m   1 0.6321
## 1806 immoralOther immoralOther    match        n     n   1 0.5253
## 1807   moralOther   moralOther    match        n     n   1 0.4125
## 1808 immoralOther immoralOther mismatch        m  <NA>  -1 1.0841
## 1809   moralOther   moralOther    match        n     n   1 0.6320
## 1810  immoralSelf  immoralSelf    match        n     n   1 0.7333
## 1811    moralSelf    moralSelf mismatch        m     m   1 0.8726
## 1812  immoralSelf  immoralSelf mismatch        m     n   0 0.5028
## 1813  immoralSelf  immoralSelf    match        n     n   1 0.5016
## 1814    moralSelf    moralSelf mismatch        m     m   1 0.6442
## 1815   moralOther   moralOther    match        n     n   1 0.4259
## 1816    moralSelf    moralSelf    match        n     n   1 0.5513
## 1817   moralOther   moralOther mismatch        m     m   1 0.6591
## 1818    moralSelf    moralSelf    match        n     n   1 0.4131
## 1819    moralSelf    moralSelf mismatch        m     m   1 0.6540
## 1820   moralOther   moralOther mismatch        m     m   1 0.6919
## 1821  immoralSelf  immoralSelf mismatch        m     m   1 0.7544
## 1822 immoralOther immoralOther mismatch        m     m   1 0.8064
## 1823    moralSelf    moralSelf    match        n     n   1 0.5313
## 1824 immoralOther immoralOther    match        n     n   1 0.9025
## 1825 immoralOther immoralOther mismatch        m     m   1 0.6293
## 1826 immoralOther immoralOther    match        n     n   1 0.4586
## 1827   moralOther   moralOther    match        n     n   1 0.5646
## 1828   moralOther   moralOther    match        n     n   1 0.4126
## 1829   moralOther   moralOther    match        n     n   1 0.5096
## 1830  immoralSelf  immoralSelf mismatch        m     m   1 0.6725
## 1831    moralSelf    moralSelf    match        n     n   1 0.4949
## 1832 immoralOther immoralOther    match        n     n   1 0.6455
## 1833   moralOther   moralOther mismatch        m     m   1 0.5872
## 1834   moralOther   moralOther mismatch        m     n   0 0.6039
## 1835 immoralOther immoralOther mismatch        m     m   1 0.8326
## 1836  immoralSelf  immoralSelf mismatch        m     m   1 0.6501
## 1837    moralSelf    moralSelf mismatch        m     m   1 0.6158
## 1838    moralSelf    moralSelf    match        n     n   1 0.7289
## 1839    moralSelf    moralSelf mismatch        m     n   0 0.6043
## 1840 immoralOther immoralOther    match        n     n   1 0.6491
## 1841  immoralSelf  immoralSelf    match        n     n   1 0.7588
## 1842  immoralSelf  immoralSelf    match        n     n   1 0.4588
## 1843   moralOther   moralOther mismatch        m     m   1 0.6607
## 1844  immoralSelf  immoralSelf mismatch        m     m   1 0.7267
## 1845  immoralSelf  immoralSelf    match        n     n   1 0.6181
## 1846    moralSelf    moralSelf mismatch        m     n   0 0.4510
## 1847    moralSelf    moralSelf    match        n     m   0 0.7369
## 1848 immoralOther immoralOther mismatch        m     m   1 0.8441
## 1849 immoralOther immoralOther mismatch        m     m   1 0.9539
## 1850    moralSelf    moralSelf mismatch        m     m   1 0.6538
## 1851  immoralSelf  immoralSelf mismatch        m     n   0 0.5876
## 1852   moralOther   moralOther    match        n     n   1 0.5481
## 1853  immoralSelf  immoralSelf    match        n     n   1 0.4917
## 1854    moralSelf    moralSelf    match        n     n   1 0.5144
## 1855    moralSelf    moralSelf mismatch        m     m   1 0.6773
## 1856 immoralOther immoralOther mismatch        m     m   1 0.8436
## 1857  immoralSelf  immoralSelf    match        n     n   1 0.7011
## 1858    moralSelf    moralSelf mismatch        m     m   1 0.5877
## 1859 immoralOther immoralOther mismatch        m     m   1 0.7883
## 1860    moralSelf    moralSelf    match        n     n   1 0.4449
## 1861   moralOther   moralOther mismatch        m     m   1 0.7305
## 1862 immoralOther immoralOther    match        n     n   1 0.7100
## 1863   moralOther   moralOther mismatch        m     m   1 0.6569
## 1864 immoralOther immoralOther    match        n     n   1 0.6749
## 1865   moralOther   moralOther    match        n     n   1 0.9531
## 1866  immoralSelf  immoralSelf mismatch        m  <NA>  -1 1.0841
## 1867  immoralSelf  immoralSelf    match        n     n   1 0.4954
## 1868   moralOther   moralOther mismatch        m     m   1 0.7662
## 1869  immoralSelf  immoralSelf mismatch        m     n   0 0.6863
## 1870 immoralOther immoralOther    match        n     n   1 0.6247
## 1871   moralOther   moralOther    match        n     n   1 0.7020
## 1872    moralSelf    moralSelf    match        n     n   1 0.7246
## 1873 immoralOther immoralOther mismatch        m     m   1 0.9736
## 1874 immoralOther immoralOther mismatch        m     m   1 0.7540
## 1875  immoralSelf  immoralSelf    match        n     n   1 0.5499
## 1876 immoralOther immoralOther    match        n     n   1 0.6055
## 1877   moralOther   moralOther mismatch        m     m   1 0.8503
## 1878  immoralSelf  immoralSelf    match        n     n   1 0.5921
## 1879    moralSelf    moralSelf    match        n     n   1 0.6965
## 1880   moralOther   moralOther    match        n     n   1 0.7673
## 1881  immoralSelf  immoralSelf mismatch        m     m   1 0.6074
## 1882   moralOther   moralOther    match        n     n   1 0.4602
## 1883  immoralSelf  immoralSelf mismatch        m     m   1 0.6301
## 1884   moralOther   moralOther mismatch        m     n   0 0.5394
## 1885    moralSelf    moralSelf mismatch        m     n   0 0.7870
## 1886   moralOther   moralOther mismatch        m  <NA>  -1 1.0841
## 1887    moralSelf    moralSelf    match        n     n   1 0.5981
## 1888  immoralSelf  immoralSelf mismatch        m     m   1 0.7709
## 1889  immoralSelf  immoralSelf    match        n     n   1 0.6271
## 1890    moralSelf    moralSelf mismatch        m     n   0 0.5763
## 1891 immoralOther immoralOther mismatch        m     m   1 0.6046
## 1892    moralSelf    moralSelf    match        n     n   1 0.5533
## 1893 immoralOther immoralOther    match        n     n   1 0.6692
## 1894 immoralOther immoralOther    match        n     n   1 0.6116
## 1895    moralSelf    moralSelf mismatch        m     m   1 0.5885
## 1896   moralOther   moralOther    match        n     n   1 0.4772
## 1897 immoralOther immoralOther    match        n     n   1 0.7713
## 1898   moralOther   moralOther    match        n     n   1 0.5233
## 1899 immoralOther immoralOther    match        n     n   1 1.0345
## 1900    moralSelf    moralSelf    match        n     n   1 0.5560
## 1901    moralSelf    moralSelf    match        n     n   1 0.4400
## 1902  immoralSelf  immoralSelf mismatch        m     n   0 0.7534
## 1903  immoralSelf  immoralSelf mismatch        m     m   1 0.6612
## 1904 immoralOther immoralOther mismatch        m     m   1 0.7832
## 1905    moralSelf    moralSelf mismatch        m  <NA>  -1 1.0841
## 1906 immoralOther immoralOther    match        n     n   1 0.7501
## 1907   moralOther   moralOther    match        n     n   1 0.6819
## 1908   moralOther   moralOther mismatch        m     m   1 0.6523
## 1909    moralSelf    moralSelf mismatch        m     n   0 0.4741
## 1910  immoralSelf  immoralSelf mismatch        m     m   1 0.7204
## 1911   moralOther   moralOther mismatch        m     n   0 0.7515
## 1912   moralOther   moralOther mismatch        m     n   0 0.9473
## 1913 immoralOther immoralOther mismatch        m     m   1 1.0270
## 1914  immoralSelf  immoralSelf    match        n     n   1 0.6082
## 1915    moralSelf    moralSelf mismatch        m     m   1 0.7330
## 1916  immoralSelf  immoralSelf    match        n     n   1 0.6724
## 1917   moralOther   moralOther    match        n     n   1 0.8707
## 1918  immoralSelf  immoralSelf    match        n     n   1 0.9169
## 1919    moralSelf    moralSelf    match        n     n   1 0.6040
## 1920 immoralOther immoralOther mismatch        m     n   0 0.7129
## 1921  immoralSelf  immoralSelf    match        n     m   0 0.8691
## 1922  immoralSelf  immoralSelf mismatch        m     m   1 0.7853
## 1923   moralOther   moralOther    match        n     n   1 0.5514
## 1924    moralSelf    moralSelf mismatch        m     m   1 0.8115
## 1925 immoralOther immoralOther mismatch        m     m   1 0.8117
## 1926 immoralOther immoralOther mismatch        m     m   1 0.7399
## 1927   moralOther   moralOther mismatch        m     m   1 0.8179
## 1928 immoralOther immoralOther    match        n     n   1 0.9121
## 1929  immoralSelf  immoralSelf mismatch        m     n   0 0.9803
## 1930 immoralOther immoralOther    match        n     n   1 0.8184
## 1931   moralOther   moralOther mismatch        m     m   1 0.6405
## 1932   moralOther   moralOther    match        n     n   1 0.5306
## 1933  immoralSelf  immoralSelf    match        n     n   1 0.8908
## 1934    moralSelf    moralSelf mismatch        m     m   1 0.8669
## 1935  immoralSelf  immoralSelf mismatch        m     n   0 1.0251
## 1936 immoralOther immoralOther    match        n     n   1 0.8232
## 1937    moralSelf    moralSelf mismatch        m     n   0 0.9573
## 1938    moralSelf    moralSelf    match        n     n   1 0.7675
## 1939   moralOther   moralOther mismatch        m     m   1 0.7276
## 1940    moralSelf    moralSelf    match        n     m   0 1.0797
## 1941    moralSelf    moralSelf    match        n     n   1 0.5219
## 1942   moralOther   moralOther    match        n     n   1 0.5481
## 1943 immoralOther immoralOther mismatch        m     m   1 0.8242
## 1944  immoralSelf  immoralSelf    match        n     n   1 0.8063
## 1945 immoralOther immoralOther    match        n  <NA>  -1 1.0850
## 1946   moralOther   moralOther mismatch        m     n   0 0.6505
## 1947  immoralSelf  immoralSelf    match        n  <NA>  -1 1.0850
## 1948    moralSelf    moralSelf    match        n     m   0 0.7329
## 1949   moralOther   moralOther mismatch        m     m   1 0.7970
## 1950    moralSelf    moralSelf    match        n     n   1 1.0472
## 1951 immoralOther immoralOther mismatch        m  <NA>  -1 1.0850
## 1952  immoralSelf  immoralSelf mismatch        m     m   1 1.0335
## 1953   moralOther   moralOther mismatch        m     m   1 0.7416
## 1954  immoralSelf  immoralSelf    match        n     n   1 0.5958
## 1955    moralSelf    moralSelf    match        n     n   1 0.6639
## 1956  immoralSelf  immoralSelf    match        n     n   1 0.7580
## 1957    moralSelf    moralSelf mismatch        m     m   1 0.7282
## 1958   moralOther   moralOther    match        n     n   1 0.4403
## 1959 immoralOther immoralOther mismatch        m     m   1 0.9545
## 1960    moralSelf    moralSelf mismatch        m  <NA>  -1 1.0850
## 1961 immoralOther immoralOther    match        n     n   1 0.9927
## 1962   moralOther   moralOther    match        n     n   1 0.4448
## 1963   moralOther   moralOther    match        n     n   1 0.6389
## 1964 immoralOther immoralOther mismatch        m     n   0 0.6491
## 1965  immoralSelf  immoralSelf mismatch        m     m   1 0.8492
## 1966 immoralOther immoralOther    match        n     n   1 0.7533
## 1967  immoralSelf  immoralSelf mismatch        m     n   0 0.8774
## 1968    moralSelf    moralSelf mismatch        m     n   0 0.5677
## 1969   moralOther   moralOther    match        n     n   1 0.4897
## 1970   moralOther   moralOther mismatch        m     n   0 0.4499
## 1971    moralSelf    moralSelf mismatch        m     m   1 0.7062
## 1972  immoralSelf  immoralSelf    match        n     n   1 0.8721
## 1973   moralOther   moralOther mismatch        m     m   1 0.7642
## 1974  immoralSelf  immoralSelf mismatch        m     m   1 0.7904
## 1975    moralSelf    moralSelf mismatch        m     m   1 0.7065
## 1976 immoralOther immoralOther    match        n     n   1 0.7687
## 1977 immoralOther immoralOther    match        n     n   1 0.5849
## 1978 immoralOther immoralOther mismatch        m     m   1 0.6950
## 1979    moralSelf    moralSelf    match        n     n   1 0.6390
## 1980    moralSelf    moralSelf    match        n     n   1 0.8812
## 1981 immoralOther immoralOther mismatch        m     m   1 0.6753
## 1982 immoralOther immoralOther mismatch        m     m   1 0.5515
## 1983    moralSelf    moralSelf    match        n     m   0 0.6676
## 1984    moralSelf    moralSelf mismatch        m     m   1 0.6398
## 1985  immoralSelf  immoralSelf mismatch        m     n   0 0.7781
## 1986 immoralOther immoralOther    match        n     n   1 0.9120
## 1987  immoralSelf  immoralSelf    match        n     n   1 0.7881
## 1988  immoralSelf  immoralSelf mismatch        m     m   1 0.8163
## 1989   moralOther   moralOther mismatch        m     n   0 0.5684
## 1990  immoralSelf  immoralSelf    match        n     n   1 0.8405
## 1991   moralOther   moralOther    match        n     n   1 0.4988
## 1992   moralOther   moralOther    match        n     n   1 0.5448
## 1993    moralSelf    moralSelf mismatch        m     m   1 0.7978
## 1994 immoralOther immoralOther    match        n     n   1 0.9359
## 1995    moralSelf    moralSelf    match        n     n   1 0.7922
## 1996 immoralOther immoralOther    match        n     n   1 0.6442
## 1997 immoralOther immoralOther mismatch        m     m   1 0.6623
## 1998   moralOther   moralOther    match        n     n   1 0.4765
## 1999 immoralOther immoralOther mismatch        m     m   1 0.7206
## 2000  immoralSelf  immoralSelf mismatch        m     m   1 0.8587
## 2001  immoralSelf  immoralSelf    match        n     n   1 0.4329
## 2002  immoralSelf  immoralSelf    match        n     n   1 0.4690
## 2003    moralSelf    moralSelf    match        n     n   1 0.6651
## 2004  immoralSelf  immoralSelf mismatch        m     m   1 0.8232
## 2005  immoralSelf  immoralSelf    match        n     n   1 0.7654
## 2006   moralOther   moralOther    match        n     n   1 0.4636
## 2007    moralSelf    moralSelf mismatch        m     m   1 0.6456
## 2008  immoralSelf  immoralSelf mismatch        m     n   0 0.7678
## 2009   moralOther   moralOther mismatch        m     m   1 0.9600
## 2010   moralOther   moralOther    match        n     n   1 0.5621
## 2011    moralSelf    moralSelf    match        n     n   1 0.8204
## 2012   moralOther   moralOther mismatch        m     m   1 0.5703
## 2013 immoralOther immoralOther    match        n     m   0 1.0505
## 2014 immoralOther immoralOther mismatch        m     m   1 0.7946
## 2015   moralOther   moralOther mismatch        m     m   1 0.9427
## 2016    moralSelf    moralSelf mismatch        m     n   0 0.9009
## 2017    moralSelf    moralSelf mismatch        m     m   1 0.7630
## 2018   moralOther   moralOther mismatch        m  <NA>  -1 1.0850
## 2019   moralOther   moralOther    match        n     n   1 0.6393
## 2020  immoralSelf  immoralSelf mismatch        m     m   1 0.6893
## 2021   moralOther   moralOther    match        n     n   1 0.5695
## 2022  immoralSelf  immoralSelf    match        n     n   1 0.8977
## 2023 immoralOther immoralOther    match        n     n   1 0.8798
## 2024 immoralOther immoralOther    match        n     n   1 0.3940
## 2025  immoralSelf  immoralSelf mismatch        m     m   1 0.5820
## 2026  immoralSelf  immoralSelf    match        n     n   1 0.5962
## 2027    moralSelf    moralSelf    match        n     n   1 0.8403
## 2028    moralSelf    moralSelf mismatch        m     m   1 0.7865
## 2029   moralOther   moralOther    match        n     n   1 0.4406
## 2030   moralOther   moralOther mismatch        m     m   1 0.6027
## 2031  immoralSelf  immoralSelf mismatch        m     n   0 0.6448
## 2032 immoralOther immoralOther mismatch        m     m   1 0.8229
## 2033  immoralSelf  immoralSelf    match        n     n   1 1.0371
## 2034   moralOther   moralOther mismatch        m     m   1 0.5993
## 2035    moralSelf    moralSelf mismatch        m     m   1 0.7534
## 2036 immoralOther immoralOther mismatch        m     m   1 0.7815
## 2037 immoralOther immoralOther    match        n     m   0 0.5477
## 2038    moralSelf    moralSelf    match        n     m   0 0.6798
## 2039 immoralOther immoralOther mismatch        m     m   1 0.9859
## 2040    moralSelf    moralSelf    match        n     n   1 0.6721
## 2041    moralSelf    moralSelf    match        n  <NA>  -1 1.0850
## 2042 immoralOther immoralOther    match        n  <NA>  -1 1.0850
## 2043  immoralSelf  immoralSelf    match        n     n   1 0.9467
## 2044  immoralSelf  immoralSelf mismatch        m     m   1 0.7368
## 2045   moralOther   moralOther    match        n     n   1 0.5290
## 2046   moralOther   moralOther mismatch        m     m   1 0.8890
## 2047  immoralSelf  immoralSelf mismatch        m     m   1 0.8252
## 2048 immoralOther immoralOther mismatch        m     m   1 0.8634
## 2049 immoralOther immoralOther    match        n     n   1 0.6536
## 2050  immoralSelf  immoralSelf    match        n     n   1 0.7577
## 2051  immoralSelf  immoralSelf mismatch        m     m   1 0.6798
## 2052 immoralOther immoralOther mismatch        m     n   0 0.8258
## 2053   moralOther   moralOther    match        n     n   1 0.5041
## 2054  immoralSelf  immoralSelf    match        n     n   1 0.5421
## 2055    moralSelf    moralSelf mismatch        m     m   1 0.7003
## 2056   moralOther   moralOther    match        n     n   1 0.4824
## 2057   moralOther   moralOther mismatch        m     m   1 0.5246
## 2058    moralSelf    moralSelf mismatch        m     m   1 0.9427
## 2059   moralOther   moralOther mismatch        m     m   1 0.8208
## 2060 immoralOther immoralOther    match        n     n   1 0.6829
## 2061    moralSelf    moralSelf    match        n     n   1 0.6931
## 2062    moralSelf    moralSelf    match        n     m   0 0.9652
## 2063    moralSelf    moralSelf mismatch        m     m   1 0.7354
## 2064 immoralOther immoralOther mismatch        m     m   1 0.9675
## 2065   moralOther   moralOther    match        n     n   1 0.5036
## 2066  immoralSelf  immoralSelf    match        n     n   1 0.9817
## 2067  immoralSelf  immoralSelf mismatch        m     m   1 0.8199
## 2068    moralSelf    moralSelf    match        n     n   1 0.8500
## 2069  immoralSelf  immoralSelf    match        n     n   1 0.7302
## 2070  immoralSelf  immoralSelf    match        n     n   1 0.5863
## 2071 immoralOther immoralOther    match        n     n   1 0.8325
## 2072   moralOther   moralOther mismatch        m     m   1 0.6446
## 2073   moralOther   moralOther    match        n     n   1 0.7507
## 2074 immoralOther immoralOther mismatch        m     n   0 1.0189
## 2075  immoralSelf  immoralSelf mismatch        m     m   1 0.8810
## 2076    moralSelf    moralSelf mismatch        m     m   1 0.9471
## 2077 immoralOther immoralOther    match        n     n   1 0.9032
## 2078   moralOther   moralOther mismatch        m     n   0 0.5174
## 2079    moralSelf    moralSelf mismatch        m     m   1 0.6795
## 2080 immoralOther immoralOther    match        n     n   1 0.7697
## 2081   moralOther   moralOther mismatch        m     n   0 0.5618
## 2082    moralSelf    moralSelf mismatch        m     m   1 0.7880
## 2083  immoralSelf  immoralSelf mismatch        m     m   1 0.9520
## 2084    moralSelf    moralSelf    match        n     m   0 0.8042
## 2085 immoralOther immoralOther mismatch        m  <NA>  -1 1.0850
## 2086    moralSelf    moralSelf    match        n     n   1 0.8404
## 2087   moralOther   moralOther    match        n     n   1 0.4826
## 2088 immoralOther immoralOther mismatch        m     m   1 0.8128
## 2089 immoralOther immoralOther    match        n     n   1 0.7250
## 2090   moralOther   moralOther mismatch        m     m   1 0.7431
## 2091    moralSelf    moralSelf    match        n     n   1 0.7252
## 2092   moralOther   moralOther mismatch        m     m   1 0.7193
## 2093   moralOther   moralOther    match        n     n   1 0.6175
## 2094  immoralSelf  immoralSelf    match        n     n   1 0.7276
## 2095 immoralOther immoralOther mismatch        m     m   1 1.0797
## 2096  immoralSelf  immoralSelf    match        n     n   1 0.7459
## 2097  immoralSelf  immoralSelf mismatch        m     m   1 0.6800
## 2098    moralSelf    moralSelf    match        n     n   1 0.6181
## 2099  immoralSelf  immoralSelf mismatch        m     m   1 0.8322
## 2100 immoralOther immoralOther    match        n     n   1 0.8124
## 2101    moralSelf    moralSelf mismatch        m     m   1 0.9565
## 2102  immoralSelf  immoralSelf    match        n     n   1 0.7767
## 2103   moralOther   moralOther mismatch        m     m   1 0.7528
## 2104    moralSelf    moralSelf mismatch        m     m   1 0.8370
## 2105 immoralOther immoralOther mismatch        m     m   1 0.7371
## 2106    moralSelf    moralSelf    match        n     m   0 0.7672
## 2107    moralSelf    moralSelf mismatch        m     m   1 0.7913
## 2108   moralOther   moralOther    match        n     n   1 0.5795
## 2109  immoralSelf  immoralSelf mismatch        m     m   1 0.6096
## 2110 immoralOther immoralOther    match        n     n   1 0.7298
## 2111 immoralOther immoralOther mismatch        m     n   0 0.9058
## 2112   moralOther   moralOther    match        n     n   1 0.5600
## 2113   moralOther   moralOther    match        n     n   1 0.5629
## 2114   moralOther   moralOther mismatch        m     m   1 0.6290
## 2115    moralSelf    moralSelf    match        n     m   0 0.6011
## 2116  immoralSelf  immoralSelf    match        n     n   1 0.5792
## 2117  immoralSelf  immoralSelf    match        n     n   1 0.7454
## 2118 immoralOther immoralOther mismatch        m     m   1 0.8394
## 2119   moralOther   moralOther mismatch        m     m   1 0.8016
## 2120    moralSelf    moralSelf mismatch        m     n   0 1.0198
## 2121 immoralOther immoralOther mismatch        m     m   1 0.8819
## 2122   moralOther   moralOther    match        n     n   1 0.5320
## 2123  immoralSelf  immoralSelf mismatch        m     n   0 0.7402
## 2124    moralSelf    moralSelf mismatch        m     m   1 0.8502
## 2125 immoralOther immoralOther mismatch        m     m   1 0.8825
## 2126    moralSelf    moralSelf    match        n     n   1 0.5966
## 2127   moralOther   moralOther mismatch        m     n   0 0.5447
## 2128    moralSelf    moralSelf    match        n     m   0 0.8468
## 2129 immoralOther immoralOther    match        n     n   1 0.7609
## 2130    moralSelf    moralSelf mismatch        m     m   1 0.6992
## 2131  immoralSelf  immoralSelf    match        n     n   1 0.6733
## 2132   moralOther   moralOther    match        n     n   1 0.4854
## 2133 immoralOther immoralOther    match        n     n   1 0.6795
## 2134  immoralSelf  immoralSelf mismatch        m     n   0 0.9136
## 2135 immoralOther immoralOther    match        n     n   1 0.5978
## 2136  immoralSelf  immoralSelf mismatch        m     n   0 0.7918
## 2137 immoralOther immoralOther    match        n     n   1 0.5640
## 2138   moralOther   moralOther mismatch        m     m   1 0.8141
## 2139  immoralSelf  immoralSelf    match        n     n   1 0.7903
## 2140 immoralOther immoralOther mismatch        m     m   1 0.9144
## 2141   moralOther   moralOther mismatch        m     m   1 0.6866
## 2142   moralOther   moralOther    match        n     n   1 0.6166
## 2143 immoralOther immoralOther    match        n     n   1 0.6729
## 2144    moralSelf    moralSelf mismatch        m     m   1 0.8449
## 2145  immoralSelf  immoralSelf mismatch        m     m   1 0.8871
## 2146  immoralSelf  immoralSelf mismatch        m     m   1 0.9033
## 2147    moralSelf    moralSelf mismatch        m     m   1 0.6773
## 2148    moralSelf    moralSelf    match        n     n   1 0.8475
## 2149    moralSelf    moralSelf mismatch        m     m   1 0.7757
## 2150 immoralOther immoralOther mismatch        m     m   1 0.9597
## 2151    moralSelf    moralSelf    match        n     n   1 0.7779
## 2152  immoralSelf  immoralSelf    match        n     n   1 0.7841
## 2153 immoralOther immoralOther    match        n     n   1 0.8142
## 2154  immoralSelf  immoralSelf mismatch        m     m   1 0.8704
## 2155 immoralOther immoralOther mismatch        m     m   1 0.6185
## 2156   moralOther   moralOther    match        n     n   1 0.6326
## 2157  immoralSelf  immoralSelf    match        n     m   0 0.9086
## 2158   moralOther   moralOther    match        n     n   1 0.5768
## 2159   moralOther   moralOther mismatch        m     m   1 0.7530
## 2160    moralSelf    moralSelf    match        n     n   1 0.7812
## 2161    moralSelf    moralSelf    match        n     m   0 0.8835
## 2162   moralOther   moralOther mismatch        m     m   1 0.7896
## 2163  immoralSelf  immoralSelf    match        n     n   1 0.6978
## 2164   moralOther   moralOther mismatch        m     m   1 0.7918
## 2165 immoralOther immoralOther mismatch        m     m   1 1.0681
## 2166    moralSelf    moralSelf mismatch        m     m   1 0.6802
## 2167 immoralOther immoralOther    match        n     n   1 0.7703
## 2168   moralOther   moralOther    match        n     n   1 0.6024
## 2169 immoralOther immoralOther mismatch        m     m   1 0.8605
## 2170   moralOther   moralOther mismatch        m     m   1 0.7387
## 2171    moralSelf    moralSelf    match        n     n   1 0.8249
## 2172 immoralOther immoralOther    match        n     n   1 0.7190
## 2173  immoralSelf  immoralSelf mismatch        m     m   1 0.6574
## 2174  immoralSelf  immoralSelf    match        n     n   1 0.8793
## 2175    moralSelf    moralSelf mismatch        m     m   1 0.8014
## 2176   moralOther   moralOther    match        n     n   1 0.5555
## 2177   moralOther   moralOther    match        n     n   1 0.6156
## 2178  immoralSelf  immoralSelf    match        n     n   1 0.7518
## 2179  immoralSelf  immoralSelf mismatch        m     m   1 0.6599
## 2180    moralSelf    moralSelf    match        n     n   1 0.9860
## 2181  immoralSelf  immoralSelf mismatch        m     m   1 0.9122
## 2182 immoralOther immoralOther mismatch        m     m   1 0.7563
## 2183 immoralOther immoralOther    match        n     n   1 0.9044
## 2184    moralSelf    moralSelf mismatch        m     m   1 0.8386
## 2185    moralSelf    moralSelf mismatch        m     m   1 0.6967
## 2186   moralOther   moralOther    match        n     n   1 0.5529
## 2187 immoralOther immoralOther    match        n     n   1 0.9010
## 2188   moralOther   moralOther mismatch        m     m   1 1.0271
## 2189   moralOther   moralOther    match        n     n   1 0.5753
## 2190  immoralSelf  immoralSelf    match        n     n   1 0.7674
## 2191   moralOther   moralOther mismatch        m     m   1 0.9115
## 2192  immoralSelf  immoralSelf mismatch        m     m   1 0.7717
## 2193  immoralSelf  immoralSelf    match        n     n   1 0.7558
## 2194    moralSelf    moralSelf    match        n     n   1 0.8400
## 2195  immoralSelf  immoralSelf    match        n     n   1 0.7221
## 2196   moralOther   moralOther    match        n     n   1 0.5723
## 2197    moralSelf    moralSelf mismatch        m     m   1 0.9245
## 2198 immoralOther immoralOther mismatch        m     m   1 0.8945
## 2199 immoralOther immoralOther mismatch        m     m   1 0.9026
## 2200 immoralOther immoralOther mismatch        m     m   1 0.6368
## 2201    moralSelf    moralSelf    match        n     n   1 0.6889
## 2202    moralSelf    moralSelf mismatch        m     m   1 0.6571
## 2203  immoralSelf  immoralSelf mismatch        m     m   1 0.8872
## 2204  immoralSelf  immoralSelf mismatch        m     m   1 0.9113
## 2205    moralSelf    moralSelf    match        n     n   1 0.6674
## 2206   moralOther   moralOther mismatch        m     m   1 0.7356
## 2207 immoralOther immoralOther    match        n     n   1 0.7678
## 2208 immoralOther immoralOther    match        n     n   1 0.6559
## 2209  immoralSelf  immoralSelf    match        n     n   1 0.8241
## 2210 immoralOther immoralOther    match        n     n   1 0.7742
## 2211 immoralOther immoralOther    match        n     n   1 0.5902
## 2212 immoralOther immoralOther mismatch        m     m   1 0.8904
## 2213   moralOther   moralOther    match        n     n   1 0.6666
## 2214 immoralOther immoralOther mismatch        m     m   1 0.9687
## 2215  immoralSelf  immoralSelf mismatch        m     m   1 0.7928
## 2216   moralOther   moralOther mismatch        m     m   1 0.8130
## 2217   moralOther   moralOther mismatch        m     m   1 0.8371
## 2218  immoralSelf  immoralSelf mismatch        m     m   1 0.7673
## 2219    moralSelf    moralSelf mismatch        m     m   1 0.9034
## 2220  immoralSelf  immoralSelf    match        n     n   1 0.7816
## 2221 immoralOther immoralOther mismatch        m     m   1 0.7557
## 2222    moralSelf    moralSelf    match        n     n   1 0.8717
## 2223    moralSelf    moralSelf    match        n     n   1 0.5880
## 2224   moralOther   moralOther    match        n     n   1 0.6641
## 2225    moralSelf    moralSelf mismatch        m     m   1 0.7102
## 2226   moralOther   moralOther mismatch        m     m   1 0.7943
## 2227  immoralSelf  immoralSelf    match        n     n   1 0.7425
## 2228    moralSelf    moralSelf    match        n     n   1 0.8846
## 2229  immoralSelf  immoralSelf mismatch        m     m   1 0.7347
## 2230   moralOther   moralOther    match        n     n   1 0.6550
## 2231 immoralOther immoralOther    match        n     n   1 0.8470
## 2232    moralSelf    moralSelf mismatch        m     m   1 0.7292
## 2233  immoralSelf  immoralSelf    match        n     n   1 0.8080
## 2234 immoralOther immoralOther mismatch        m     m   1 1.0561
## 2235    moralSelf    moralSelf    match        n     n   1 0.7843
## 2236  immoralSelf  immoralSelf    match        n     n   1 0.7345
## 2237    moralSelf    moralSelf    match        n     n   1 0.7985
## 2238 immoralOther immoralOther mismatch        m     m   1 0.7927
## 2239  immoralSelf  immoralSelf    match        n     n   1 0.6689
## 2240    moralSelf    moralSelf mismatch        m     m   1 0.7290
## 2241   moralOther   moralOther mismatch        m     m   1 0.7292
## 2242  immoralSelf  immoralSelf mismatch        m     m   1 0.6413
## 2243   moralOther   moralOther    match        n     n   1 0.6274
## 2244   moralOther   moralOther    match        n     n   1 0.6155
## 2245   moralOther   moralOther mismatch        m     m   1 0.6956
## 2246  immoralSelf  immoralSelf mismatch        m     m   1 0.7759
## 2247    moralSelf    moralSelf    match        n     n   1 0.8719
## 2248    moralSelf    moralSelf mismatch        m     m   1 0.7641
## 2249 immoralOther immoralOther    match        n     n   1 0.7742
## 2250  immoralSelf  immoralSelf mismatch        m     m   1 0.7424
## 2251   moralOther   moralOther    match        n     n   1 0.5566
## 2252   moralOther   moralOther mismatch        m     m   1 0.6487
## 2253 immoralOther immoralOther mismatch        m     m   1 0.6248
## 2254 immoralOther immoralOther    match        n     n   1 0.7508
## 2255 immoralOther immoralOther    match        n     n   1 0.7071
## 2256    moralSelf    moralSelf mismatch        m     m   1 0.6791
## 2257    moralSelf    moralSelf    match        n     n   1 0.6653
## 2258  immoralSelf  immoralSelf    match        n     n   1 0.7514
## 2259 immoralOther immoralOther    match        n     m   0 0.7875
## 2260  immoralSelf  immoralSelf mismatch        m     n   0 0.9837
## 2261    moralSelf    moralSelf    match        n     n   1 0.7258
## 2262   moralOther   moralOther mismatch        m     m   1 0.7199
## 2263   moralOther   moralOther mismatch        m     m   1 0.7781
## 2264 immoralOther immoralOther mismatch        m     m   1 0.8082
## 2265   moralOther   moralOther    match        n     n   1 0.6484
## 2266  immoralSelf  immoralSelf mismatch        m     m   1 0.7765
## 2267 immoralOther immoralOther    match        n     n   1 0.7207
## 2268    moralSelf    moralSelf mismatch        m     m   1 0.7547
## 2269 immoralOther immoralOther    match        n     n   1 0.7910
## 2270   moralOther   moralOther mismatch        m     n   0 0.5630
## 2271  immoralSelf  immoralSelf    match        n     n   1 0.8051
## 2272    moralSelf    moralSelf mismatch        m     m   1 0.6953
## 2273  immoralSelf  immoralSelf    match        n     n   1 0.7833
## 2274   moralOther   moralOther    match        n     n   1 0.5897
## 2275 immoralOther immoralOther mismatch        m     m   1 0.6497
## 2276    moralSelf    moralSelf    match        n     n   1 0.9058
## 2277  immoralSelf  immoralSelf mismatch        m     m   1 0.7599
## 2278   moralOther   moralOther    match        n     n   1 0.5861
## 2279 immoralOther immoralOther mismatch        m     m   1 0.8562
## 2280    moralSelf    moralSelf mismatch        m     m   1 0.8163
## 2281    moralSelf    moralSelf    match        n     m   0 0.9933
## 2282 immoralOther immoralOther mismatch        m     m   1 0.8856
## 2283  immoralSelf  immoralSelf mismatch        m     m   1 0.9116
## 2284   moralOther   moralOther    match        n     n   1 0.9158
## 2285    moralSelf    moralSelf    match        n     n   1 0.8079
## 2286  immoralSelf  immoralSelf    match        n     n   1 0.6881
## 2287    moralSelf    moralSelf mismatch        m     m   1 0.6962
## 2288  immoralSelf  immoralSelf mismatch        m     m   1 0.7363
## 2289 immoralOther immoralOther mismatch        m     m   1 0.6484
## 2290 immoralOther immoralOther    match        n     n   1 0.7366
## 2291  immoralSelf  immoralSelf    match        n     n   1 0.7288
## 2292 immoralOther immoralOther    match        n     n   1 1.0329
## 2293 immoralOther immoralOther    match        n     n   1 0.5650
## 2294   moralOther   moralOther    match        n     n   1 0.7111
## 2295   moralOther   moralOther mismatch        m  <NA>  -1 1.0850
## 2296    moralSelf    moralSelf    match        n     n   1 0.8374
## 2297   moralOther   moralOther mismatch        m     m   1 0.9276
## 2298   moralOther   moralOther    match        n     n   1 0.4637
## 2299  immoralSelf  immoralSelf    match        n     n   1 0.7818
## 2300   moralOther   moralOther mismatch        m     m   1 0.6359
## 2301    moralSelf    moralSelf mismatch        m     m   1 0.6962
## 2302 immoralOther immoralOther mismatch        m     m   1 0.8402
## 2303  immoralSelf  immoralSelf mismatch        m     m   1 0.5945
## 2304    moralSelf    moralSelf mismatch        m     m   1 0.7346
## 2305   moralOther   moralOther mismatch        m     m   1 0.8546
## 2306  immoralSelf  immoralSelf mismatch        m     m   1 0.8388
## 2307  immoralSelf  immoralSelf    match        n     n   1 0.7129
## 2308   moralOther   moralOther mismatch        m     m   1 0.9171
## 2309   moralOther   moralOther    match        n     n   1 0.6012
## 2310   moralOther   moralOther    match        n     n   1 0.5314
## 2311 immoralOther immoralOther    match        n     m   0 1.0115
## 2312 immoralOther immoralOther mismatch        m     m   1 0.8516
## 2313    moralSelf    moralSelf mismatch        m     m   1 0.7638
## 2314  immoralSelf  immoralSelf mismatch        m     m   1 0.7099
## 2315   moralOther   moralOther    match        n     n   1 0.6980
## 2316    moralSelf    moralSelf    match        n     m   0 0.8322
## 2317  immoralSelf  immoralSelf mismatch        m     n   0 0.5322
## 2318    moralSelf    moralSelf mismatch        m     m   1 0.6844
## 2319 immoralOther immoralOther    match        n     n   1 0.7106
## 2320    moralSelf    moralSelf mismatch        m     m   1 0.6906
## 2321  immoralSelf  immoralSelf    match        n     n   1 0.6905
## 2322  immoralSelf  immoralSelf    match        n     n   1 0.6189
## 2323   moralOther   moralOther mismatch        m     m   1 0.7450
## 2324 immoralOther immoralOther    match        n     n   1 0.7912
## 2325    moralSelf    moralSelf    match        n     n   1 0.7473
## 2326 immoralOther immoralOther mismatch        m     m   1 0.9355
## 2327    moralSelf    moralSelf    match        n     n   1 0.6877
## 2328 immoralOther immoralOther mismatch        m     m   1 0.6480
## 2329  immoralSelf  immoralSelf    match        n     u   2 0.8238
## 2330 immoralOther immoralOther    match        n  <NA>  -1 1.0850
## 2331 immoralOther immoralOther mismatch        m     m   1 0.9360
## 2332  immoralSelf  immoralSelf mismatch        m     n   0 0.8561
## 2333   moralOther   moralOther mismatch        m     m   1 0.7843
## 2334   moralOther   moralOther    match        n     n   1 0.8624
## 2335   moralOther   moralOther mismatch        m     m   1 0.5487
## 2336    moralSelf    moralSelf mismatch        m     m   1 0.9287
## 2337    moralSelf    moralSelf    match        n     n   1 0.9128
## 2338    moralSelf    moralSelf mismatch        m     m   1 0.7169
## 2339  immoralSelf  immoralSelf mismatch        m     m   1 0.7371
## 2340 immoralOther immoralOther    match        n     n   1 0.6733
## 2341 immoralOther immoralOther    match        n     n   1 0.7174
## 2342   moralOther   moralOther mismatch        m     m   1 0.8075
## 2343   moralOther   moralOther    match        n     n   1 0.6316
## 2344    moralSelf    moralSelf mismatch        m     m   1 0.6918
## 2345    moralSelf    moralSelf    match        n     n   1 0.9319
## 2346   moralOther   moralOther    match        n     n   1 0.6920
## 2347  immoralSelf  immoralSelf mismatch        m     m   1 0.7722
## 2348 immoralOther immoralOther mismatch        m     m   1 0.8923
## 2349 immoralOther immoralOther mismatch        m     m   1 0.6765
## 2350  immoralSelf  immoralSelf    match        n     n   1 1.0325
## 2351  immoralSelf  immoralSelf    match        n     n   1 0.5647
## 2352    moralSelf    moralSelf    match        n     m   0 0.7348
## 2353  immoralSelf  immoralSelf    match        n     n   1 0.6790
## 2354 immoralOther immoralOther    match        n     n   1 0.7851
## 2355   moralOther   moralOther    match        n     n   1 0.4873
## 2356   moralOther   moralOther mismatch        m     m   1 0.7194
## 2357    moralSelf    moralSelf mismatch        m     n   0 0.5855
## 2358 immoralOther immoralOther mismatch        m     m   1 0.6316
## 2359    moralSelf    moralSelf    match        n     n   1 0.7318
## 2360  immoralSelf  immoralSelf    match        n     n   1 0.7079
## 2361    moralSelf    moralSelf    match        n     n   1 0.9841
## 2362    moralSelf    moralSelf mismatch        m     m   1 0.7342
## 2363  immoralSelf  immoralSelf mismatch        m     m   1 0.7103
## 2364 immoralOther immoralOther mismatch        m     m   1 0.7225
## 2365  immoralSelf  immoralSelf mismatch        m     m   1 0.7326
## 2366  immoralSelf  immoralSelf    match        n     n   1 0.6368
## 2367 immoralOther immoralOther    match        n     n   1 0.8888
## 2368   moralOther   moralOther mismatch        m     m   1 0.8410
## 2369   moralOther   moralOther mismatch        m     m   1 0.7791
## 2370    moralSelf    moralSelf    match        n     m   0 0.7372
## 2371  immoralSelf  immoralSelf mismatch        m     m   1 0.9674
## 2372 immoralOther immoralOther mismatch        m     m   1 1.0716
## 2373   moralOther   moralOther    match        n     n   1 0.6278
## 2374    moralSelf    moralSelf mismatch        m     m   1 0.7358
## 2375 immoralOther immoralOther    match        n     n   1 0.7520
## 2376   moralOther   moralOther    match        n     n   1 0.5481
## 2377   moralOther   moralOther mismatch        m     m   1 0.8752
## 2378  immoralSelf  immoralSelf    match        n     n   1 0.8073
## 2379    moralSelf    moralSelf mismatch        m     m   1 0.7275
## 2380 immoralOther immoralOther mismatch        m     m   1 0.7436
## 2381   moralOther   moralOther mismatch        m     m   1 0.6937
## 2382 immoralOther immoralOther    match        n     n   1 0.7099
## 2383  immoralSelf  immoralSelf    match        n     n   1 0.8181
## 2384   moralOther   moralOther    match        n     n   1 0.5601
## 2385    moralSelf    moralSelf mismatch        m     m   1 0.7703
## 2386    moralSelf    moralSelf    match        n     n   1 0.7864
## 2387 immoralOther immoralOther    match        n     n   1 0.9446
## 2388 immoralOther immoralOther mismatch        m  <NA>  -1 1.0850
## 2389   moralOther   moralOther    match        n     n   1 0.4969
## 2390   moralOther   moralOther    match        n     n   1 0.5929
## 2391    moralSelf    moralSelf    match        n     n   1 0.7651
## 2392  immoralSelf  immoralSelf mismatch        m     m   1 0.6392
## 2393  immoralSelf  immoralSelf mismatch        m     n   0 0.9694
## 2394 immoralOther immoralOther    match        n     n   1 0.6895
## 2395    moralSelf    moralSelf mismatch        m     m   1 0.9056
## 2396    moralSelf    moralSelf    match        n     n   1 0.9357
## 2397   moralOther   moralOther mismatch        m     m   1 0.8159
## 2398 immoralOther immoralOther mismatch        m     m   1 0.8640
## 2399  immoralSelf  immoralSelf    match        n     n   1 0.9021
## 2400  immoralSelf  immoralSelf mismatch        m     n   0 0.8283
## 2401    moralSelf    moralSelf mismatch        m     n   0 1.0244
## 2402   moralOther   moralOther mismatch        m     m   1 0.7587
## 2403  immoralSelf  immoralSelf    match        n     n   1 0.7767
## 2404    moralSelf    moralSelf    match        n     n   1 1.0489
## 2405  immoralSelf  immoralSelf mismatch        m     m   1 0.9530
## 2406    moralSelf    moralSelf mismatch        m     m   1 0.8293
## 2407   moralOther   moralOther    match        n     n   1 0.5854
## 2408    moralSelf    moralSelf    match        n     n   1 0.8395
## 2409 immoralOther immoralOther    match        n     n   1 0.7216
## 2410  immoralSelf  immoralSelf mismatch        m     n   0 0.7557
## 2411 immoralOther immoralOther    match        n     n   1 0.6480
## 2412 immoralOther immoralOther mismatch        m     m   1 0.7121
## 2413  immoralSelf  immoralSelf    match        n     n   1 0.8761
## 2414 immoralOther immoralOther mismatch        m     m   1 0.6083
## 2415    moralSelf    moralSelf    match        n     n   1 0.6964
## 2416  immoralSelf  immoralSelf    match        n     n   1 0.7445
## 2417    moralSelf    moralSelf mismatch        m     m   1 0.8787
## 2418   moralOther   moralOther mismatch        m     m   1 0.7608
## 2419 immoralOther immoralOther mismatch        m     m   1 0.7069
## 2420   moralOther   moralOther    match        n     n   1 0.5851
## 2421   moralOther   moralOther mismatch        m     m   1 0.6312
## 2422   moralOther   moralOther    match        n     n   1 0.6514
## 2423  immoralSelf  immoralSelf mismatch        m     m   1 0.7314
## 2424 immoralOther immoralOther    match        n     n   1 0.6276
## 2425   moralOther   moralOther mismatch        m     m   1 0.7912
## 2426    moralSelf    moralSelf    match        n     n   1 0.7313
## 2427   moralOther   moralOther mismatch        m     m   1 0.6514
## 2428 immoralOther immoralOther    match        n     n   1 0.8196
## 2429  immoralSelf  immoralSelf mismatch        m     m   1 0.5860
## 2430  immoralSelf  immoralSelf    match        n     n   1 0.4880
## 2431 immoralOther immoralOther mismatch        m     m   1 0.6879
## 2432  immoralSelf  immoralSelf mismatch        m     m   1 0.8321
## 2433 immoralOther immoralOther    match        n     n   1 0.6763
## 2434   moralOther   moralOther    match        n     n   1 0.5925
## 2435  immoralSelf  immoralSelf mismatch        m  <NA>  -1 1.0850
## 2436 immoralOther immoralOther mismatch        m     m   1 0.6308
## 2437    moralSelf    moralSelf mismatch        m     m   1 0.9228
## 2438    moralSelf    moralSelf mismatch        m     n   0 0.8048
## 2439  immoralSelf  immoralSelf    match        n     n   1 0.7991
## 2440   moralOther   moralOther mismatch        m     m   1 0.6892
## 2441  immoralSelf  immoralSelf    match        n     n   1 0.7934
## 2442 immoralOther immoralOther    match        n     m   0 0.8215
## 2443   moralOther   moralOther    match        n     n   1 0.5955
## 2444   moralOther   moralOther    match        n     n   1 0.6558
## 2445 immoralOther immoralOther mismatch        m     m   1 0.8559
## 2446    moralSelf    moralSelf mismatch        m     m   1 0.8000
## 2447    moralSelf    moralSelf    match        n     n   1 0.8503
## 2448    moralSelf    moralSelf    match        n     n   1 0.5223
## 2449   moralOther   moralOther mismatch        m     n   0 0.4684
## 2450 immoralOther immoralOther    match        n     n   1 0.7765
## 2451    moralSelf    moralSelf mismatch        m     m   1 0.8487
## 2452  immoralSelf  immoralSelf mismatch        m     n   0 0.8808
## 2453  immoralSelf  immoralSelf    match        n     n   1 0.6910
## 2454    moralSelf    moralSelf mismatch        m     m   1 0.6431
## 2455 immoralOther immoralOther mismatch        m     m   1 0.8373
## 2456  immoralSelf  immoralSelf    match        n     n   1 0.6954
## 2457    moralSelf    moralSelf    match        n     n   1 0.7755
## 2458    moralSelf    moralSelf mismatch        m     m   1 0.9036
## 2459 immoralOther immoralOther    match        n     n   1 0.9018
## 2460  immoralSelf  immoralSelf mismatch        m     m   1 0.7000
## 2461   moralOther   moralOther    match        n     n   1 0.5702
## 2462 immoralOther immoralOther    match        n     n   1 0.7302
## 2463   moralOther   moralOther    match        n     n   1 0.5383
## 2464  immoralSelf  immoralSelf mismatch        m     m   1 0.8084
## 2465    moralSelf    moralSelf    match        n     n   1 0.6726
## 2466   moralOther   moralOther mismatch        m     n   0 0.7967
## 2467 immoralOther immoralOther mismatch        m     m   1 0.7209
## 2468  immoralSelf  immoralSelf    match        n     n   1 0.7470
## 2469   moralOther   moralOther mismatch        m     m   1 0.6571
## 2470 immoralOther immoralOther mismatch        m     m   1 0.6172
## 2471    moralSelf    moralSelf    match        n     n   1 0.7514
## 2472   moralOther   moralOther    match        n     n   1 0.6915
## 2473  immoralSelf  immoralSelf    match        n     m   0 0.7499
## 2474 immoralOther immoralOther mismatch        m     m   1 0.6501
## 2475 immoralOther immoralOther mismatch        m     m   1 0.6182
## 2476 immoralOther immoralOther mismatch        m     m   1 0.6484
## 2477   moralOther   moralOther mismatch        m     n   0 0.4805
## 2478 immoralOther immoralOther    match        n     n   1 0.7865
## 2479   moralOther   moralOther mismatch        m     m   1 0.7609
## 2480    moralSelf    moralSelf    match        n     n   1 0.7869
## 2481 immoralOther immoralOther    match        n     n   1 0.6330
## 2482  immoralSelf  immoralSelf    match        n     n   1 0.6851
## 2483    moralSelf    moralSelf mismatch        m     m   1 0.8392
## 2484  immoralSelf  immoralSelf mismatch        m     m   1 1.0174
## 2485  immoralSelf  immoralSelf mismatch        m     n   0 0.8555
## 2486  immoralSelf  immoralSelf    match        n     n   1 0.8637
## 2487    moralSelf    moralSelf mismatch        m     m   1 0.6538
## 2488   moralOther   moralOther mismatch        m     m   1 0.6140
## 2489  immoralSelf  immoralSelf mismatch        m     m   1 0.9021
## 2490    moralSelf    moralSelf    match        n     n   1 0.7003
## 2491 immoralOther immoralOther    match        n     n   1 0.6824
## 2492   moralOther   moralOther    match        n     n   1 0.5965
## 2493   moralOther   moralOther    match        n     m   0 0.6326
## 2494    moralSelf    moralSelf    match        n     n   1 0.7007
## 2495    moralSelf    moralSelf mismatch        m     m   1 0.7308
## 2496   moralOther   moralOther    match        n     n   1 0.5310
## 2497   moralOther   moralOther    match        n     n   1 0.5231
## 2498    moralSelf    moralSelf mismatch        m     m   1 0.7173
## 2499  immoralSelf  immoralSelf    match        n     n   1 0.7113
## 2500   moralOther   moralOther    match        n     n   1 0.4915
## 2501   moralOther   moralOther mismatch        m     m   1 0.7076
## 2502 immoralOther immoralOther    match        n     n   1 0.7918
## 2503 immoralOther immoralOther mismatch        m  <NA>  -1 1.0850
## 2504  immoralSelf  immoralSelf    match        n     n   1 0.5980
## 2505  immoralSelf  immoralSelf mismatch        m     m   1 0.6242
## 2506  immoralSelf  immoralSelf mismatch        m     n   0 1.0304
## 2507    moralSelf    moralSelf mismatch        m     m   1 0.8745
## 2508    moralSelf    moralSelf    match        n     m   0 0.6946
## 2509    moralSelf    moralSelf    match        n     n   1 0.5187
## 2510 immoralOther immoralOther mismatch        m     m   1 0.8009
## 2511 immoralOther immoralOther    match        n     n   1 0.8191
## 2512 immoralOther immoralOther    match        n     n   1 0.8172
## 2513    moralSelf    moralSelf mismatch        m     n   0 0.6313
## 2514  immoralSelf  immoralSelf    match        n     n   1 0.8514
## 2515   moralOther   moralOther    match        n     n   1 0.5397
## 2516   moralOther   moralOther mismatch        m     m   1 0.8257
## 2517 immoralOther immoralOther mismatch        m     m   1 0.9278
## 2518   moralOther   moralOther mismatch        m     m   1 0.7840
## 2519    moralSelf    moralSelf    match        n     n   1 0.6302
## 2520  immoralSelf  immoralSelf mismatch        m     m   1 0.6902
## 2521  immoralSelf  immoralSelf    match        n     m   0 0.9977
## 2522  immoralSelf  immoralSelf mismatch        m     m   1 0.8345
## 2523   moralOther   moralOther    match        n     m   0 0.7400
## 2524    moralSelf    moralSelf mismatch        m     n   0 0.8037
## 2525 immoralOther immoralOther mismatch        m     n   0 0.6726
## 2526 immoralOther immoralOther mismatch        m     n   0 1.0227
## 2527   moralOther   moralOther mismatch        m     m   1 0.9399
## 2528 immoralOther immoralOther    match        n     n   1 1.0276
## 2529  immoralSelf  immoralSelf mismatch        m     m   1 0.8170
## 2530 immoralOther immoralOther    match        n     n   1 0.9860
## 2531   moralOther   moralOther mismatch        m     m   1 0.9945
## 2532   moralOther   moralOther    match        n     n   1 1.0512
## 2533  immoralSelf  immoralSelf    match        n     n   1 0.6409
## 2534    moralSelf    moralSelf mismatch        m     m   1 0.8823
## 2535  immoralSelf  immoralSelf mismatch        m     n   0 0.5569
## 2536 immoralOther immoralOther    match        n     m   0 0.7847
## 2537    moralSelf    moralSelf mismatch        m     m   1 0.8050
## 2538    moralSelf    moralSelf    match        n     n   1 0.6980
## 2539   moralOther   moralOther mismatch        m     m   1 0.6564
## 2540    moralSelf    moralSelf    match        n     n   1 0.5945
## 2541    moralSelf    moralSelf    match        n     n   1 0.5551
## 2542   moralOther   moralOther    match        n     m   0 0.6629
## 2543 immoralOther immoralOther mismatch        m     m   1 0.8649
## 2544  immoralSelf  immoralSelf    match        n     n   1 0.6748
## 2545 immoralOther immoralOther    match        n     m   0 0.7573
## 2546   moralOther   moralOther mismatch        m     m   1 0.8492
## 2547  immoralSelf  immoralSelf    match        n     n   1 0.5669
## 2548    moralSelf    moralSelf    match        n     n   1 0.7110
## 2549   moralOther   moralOther mismatch        m     m   1 0.9459
## 2550    moralSelf    moralSelf    match        n     n   1 0.6655
## 2551 immoralOther immoralOther mismatch        m     m   1 0.9476
## 2552  immoralSelf  immoralSelf mismatch        m     m   1 0.8192
## 2553   moralOther   moralOther mismatch        m     m   1 1.0362
## 2554  immoralSelf  immoralSelf    match        n     n   1 0.8218
## 2555    moralSelf    moralSelf    match        n     n   1 0.6271
## 2556  immoralSelf  immoralSelf    match        n     n   1 0.7443
## 2557    moralSelf    moralSelf mismatch        m     m   1 0.8959
## 2558   moralOther   moralOther    match        n     n   1 0.9827
## 2559 immoralOther immoralOther mismatch        m     m   1 0.9309
## 2560    moralSelf    moralSelf mismatch        m     n   0 0.7185
## 2561 immoralOther immoralOther    match        n  <NA>  -1 1.0841
## 2562   moralOther   moralOther    match        n     n   1 0.8880
## 2563   moralOther   moralOther    match        n     n   1 0.5545
## 2564 immoralOther immoralOther mismatch        m     m   1 0.8543
## 2565  immoralSelf  immoralSelf mismatch        m     m   1 0.6681
## 2566 immoralOther immoralOther    match        n  <NA>  -1 1.0841
## 2567  immoralSelf  immoralSelf mismatch        m     m   1 0.9968
## 2568    moralSelf    moralSelf mismatch        m     m   1 0.8213
## 2569   moralOther   moralOther    match        n     m   0 0.8350
## 2570   moralOther   moralOther mismatch        m     n   0 0.6525
## 2571    moralSelf    moralSelf mismatch        m     m   1 0.8103
## 2572  immoralSelf  immoralSelf    match        n     n   1 0.5713
## 2573   moralOther   moralOther mismatch        m     n   0 0.5314
## 2574  immoralSelf  immoralSelf mismatch        m     n   0 0.6789
## 2575    moralSelf    moralSelf mismatch        m     m   1 1.0530
## 2576 immoralOther immoralOther    match        n     n   1 0.8667
## 2577 immoralOther immoralOther    match        n     n   1 0.5168
## 2578 immoralOther immoralOther mismatch        m     n   0 1.0040
## 2579    moralSelf    moralSelf    match        n     n   1 0.6166
## 2580    moralSelf    moralSelf    match        n     n   1 0.4739
## 2581 immoralOther immoralOther mismatch        m     n   0 0.4640
## 2582 immoralOther immoralOther mismatch        m     m   1 0.5139
## 2583    moralSelf    moralSelf    match        n     n   1 0.6368
## 2584    moralSelf    moralSelf mismatch        m     m   1 0.7663
## 2585  immoralSelf  immoralSelf mismatch        m     m   1 0.7745
## 2586 immoralOther immoralOther    match        n     m   0 0.6988
## 2587  immoralSelf  immoralSelf    match        n     n   1 0.6495
## 2588  immoralSelf  immoralSelf mismatch        m     m   1 0.6232
## 2589   moralOther   moralOther mismatch        m     n   0 0.6845
## 2590  immoralSelf  immoralSelf    match        n     n   1 0.6469
## 2591   moralOther   moralOther    match        n     n   1 0.8287
## 2592   moralOther   moralOther    match        n     n   1 0.5619
## 2593    moralSelf    moralSelf mismatch        m     m   1 0.7366
## 2594 immoralOther immoralOther    match        n     n   1 0.8761
## 2595    moralSelf    moralSelf    match        n     n   1 0.5062
## 2596 immoralOther immoralOther    match        n     n   1 0.8691
## 2597 immoralOther immoralOther mismatch        m     m   1 0.6834
## 2598   moralOther   moralOther    match        n     n   1 0.5498
## 2599 immoralOther immoralOther mismatch        m     m   1 0.8054
## 2600  immoralSelf  immoralSelf mismatch        m     n   0 0.4902
## 2601  immoralSelf  immoralSelf    match        n     n   1 0.5487
## 2602  immoralSelf  immoralSelf    match        n     n   1 0.4043
## 2603    moralSelf    moralSelf    match        n     m   0 0.5852
## 2604  immoralSelf  immoralSelf mismatch        m     m   1 0.6855
## 2605  immoralSelf  immoralSelf    match        n     n   1 0.6558
## 2606   moralOther   moralOther    match        n     n   1 0.9176
## 2607    moralSelf    moralSelf mismatch        m     n   0 0.7328
## 2608  immoralSelf  immoralSelf mismatch        m     m   1 0.7602
## 2609   moralOther   moralOther mismatch        m     n   0 0.8440
## 2610   moralOther   moralOther    match        n     n   1 0.5857
## 2611    moralSelf    moralSelf    match        n     n   1 0.7304
## 2612   moralOther   moralOther mismatch        m     n   0 0.8857
## 2613 immoralOther immoralOther    match        n     n   1 0.9442
## 2614 immoralOther immoralOther mismatch        m     m   1 0.6959
## 2615   moralOther   moralOther mismatch        m     m   1 1.0145
## 2616    moralSelf    moralSelf mismatch        m     n   0 0.6716
## 2617    moralSelf    moralSelf mismatch        m     m   1 0.9577
## 2618   moralOther   moralOther mismatch        m     m   1 0.9296
## 2619   moralOther   moralOther    match        n     n   1 0.7091
## 2620  immoralSelf  immoralSelf mismatch        m     n   0 0.7201
## 2621   moralOther   moralOther    match        n     n   1 0.8071
## 2622  immoralSelf  immoralSelf    match        n     n   1 0.6839
## 2623 immoralOther immoralOther    match        n     m   0 0.8783
## 2624 immoralOther immoralOther    match        n     n   1 0.4766
## 2625  immoralSelf  immoralSelf mismatch        m     m   1 0.6509
## 2626  immoralSelf  immoralSelf    match        n     n   1 0.5447
## 2627    moralSelf    moralSelf    match        n     n   1 0.5601
## 2628    moralSelf    moralSelf mismatch        m     m   1 0.7081
## 2629   moralOther   moralOther    match        n     n   1 0.7029
## 2630   moralOther   moralOther mismatch        m     m   1 0.8818
## 2631  immoralSelf  immoralSelf mismatch        m     n   0 0.4603
## 2632 immoralOther immoralOther mismatch        m     m   1 0.6543
## 2633  immoralSelf  immoralSelf    match        n     n   1 0.6840
## 2634   moralOther   moralOther mismatch        m     m   1 0.7025
## 2635    moralSelf    moralSelf mismatch        m     m   1 0.9171
## 2636 immoralOther immoralOther mismatch        m     m   1 0.8363
## 2637 immoralOther immoralOther    match        n     n   1 0.5658
## 2638    moralSelf    moralSelf    match        n     n   1 0.7659
## 2639 immoralOther immoralOther mismatch        m     m   1 0.9340
## 2640    moralSelf    moralSelf    match        n     n   1 0.7054
## 2641    moralSelf    moralSelf    match        n  <NA>  -1 1.0841
## 2642 immoralOther immoralOther    match        n  <NA>  -1 1.0841
## 2643  immoralSelf  immoralSelf    match        n     n   1 0.5740
## 2644  immoralSelf  immoralSelf mismatch        m     m   1 0.6664
## 2645   moralOther   moralOther    match        n     m   0 0.7244
## 2646   moralOther   moralOther mismatch        m     m   1 0.6516
## 2647  immoralSelf  immoralSelf mismatch        m     m   1 0.6653
## 2648 immoralOther immoralOther mismatch        m     m   1 0.7793
## 2649 immoralOther immoralOther    match        n     n   1 0.8156
## 2650  immoralSelf  immoralSelf    match        n     n   1 0.6246
## 2651  immoralSelf  immoralSelf mismatch        m     m   1 0.8620
## 2652 immoralOther immoralOther mismatch        m     m   1 0.6159
## 2653   moralOther   moralOther    match        n     m   0 0.5290
## 2654  immoralSelf  immoralSelf    match        n     n   1 0.7003
## 2655    moralSelf    moralSelf mismatch        m     m   1 0.9231
## 2656   moralOther   moralOther    match        n     m   0 0.6821
## 2657   moralOther   moralOther mismatch        m     m   1 0.6766
## 2658    moralSelf    moralSelf mismatch        m     m   1 0.5547
## 2659   moralOther   moralOther mismatch        m     m   1 0.7587
## 2660 immoralOther immoralOther    match        n     n   1 0.7626
## 2661    moralSelf    moralSelf    match        n     n   1 0.5707
## 2662    moralSelf    moralSelf    match        n     n   1 0.5227
## 2663    moralSelf    moralSelf mismatch        m     m   1 0.6499
## 2664 immoralOther immoralOther mismatch        m     m   1 0.8556
## 2665   moralOther   moralOther    match        n     n   1 0.7494
## 2666  immoralSelf  immoralSelf    match        n     n   1 0.8210
## 2667  immoralSelf  immoralSelf mismatch        m     m   1 0.5703
## 2668    moralSelf    moralSelf    match        n     n   1 0.6904
## 2669  immoralSelf  immoralSelf    match        n     n   1 0.6048
## 2670  immoralSelf  immoralSelf    match        n     n   1 0.5056
## 2671 immoralOther immoralOther    match        n     m   0 0.5246
## 2672   moralOther   moralOther mismatch        m     m   1 0.5877
## 2673   moralOther   moralOther    match        n     n   1 1.0203
## 2674 immoralOther immoralOther mismatch        m     n   0 0.7253
## 2675  immoralSelf  immoralSelf mismatch        m     n   0 0.4845
## 2676    moralSelf    moralSelf mismatch        m     m   1 0.9989
## 2677 immoralOther immoralOther    match        n     n   1 0.6597
## 2678   moralOther   moralOther mismatch        m     m   1 0.8218
## 2679    moralSelf    moralSelf mismatch        m     m   1 0.9709
## 2680 immoralOther immoralOther    match        n     n   1 0.7870
## 2681   moralOther   moralOther mismatch        m     n   0 0.5957
## 2682    moralSelf    moralSelf mismatch        m     m   1 0.8202
## 2683  immoralSelf  immoralSelf mismatch        m     m   1 0.7092
## 2684    moralSelf    moralSelf    match        n  <NA>  -1 1.0841
## 2685 immoralOther immoralOther mismatch        m     m   1 0.7307
## 2686    moralSelf    moralSelf    match        n     n   1 0.7501
## 2687   moralOther   moralOther    match        n     m   0 1.0098
## 2688 immoralOther immoralOther mismatch        m     m   1 1.0109
## 2689 immoralOther immoralOther    match        n     n   1 0.5880
## 2690   moralOther   moralOther mismatch        m     m   1 0.8686
## 2691    moralSelf    moralSelf    match        n     n   1 0.6108
## 2692   moralOther   moralOther mismatch        m     m   1 0.7078
## 2693   moralOther   moralOther    match        n     m   0 1.0067
## 2694  immoralSelf  immoralSelf    match        n     n   1 0.8195
## 2695 immoralOther immoralOther mismatch        m     m   1 0.7566
## 2696  immoralSelf  immoralSelf    match        n     n   1 1.0404
## 2697  immoralSelf  immoralSelf mismatch        m     m   1 0.6978
## 2698    moralSelf    moralSelf    match        n     m   0 0.7046
## 2699  immoralSelf  immoralSelf mismatch        m     m   1 0.8916
## 2700 immoralOther immoralOther    match        n     m   0 0.9783
## 2701    moralSelf    moralSelf mismatch        m     m   1 0.7506
## 2702  immoralSelf  immoralSelf    match        n     n   1 0.8905
## 2703   moralOther   moralOther mismatch        m     m   1 0.6769
## 2704    moralSelf    moralSelf mismatch        m     m   1 0.8192
## 2705 immoralOther immoralOther mismatch        m     n   0 0.7483
## 2706    moralSelf    moralSelf    match        n     n   1 0.9079
## 2707    moralSelf    moralSelf mismatch        m     m   1 0.7108
## 2708   moralOther   moralOther    match        n     n   1 1.0417
## 2709  immoralSelf  immoralSelf mismatch        m     m   1 0.7152
## 2710 immoralOther immoralOther    match        n     n   1 0.8664
## 2711 immoralOther immoralOther mismatch        m     m   1 0.6044
## 2712   moralOther   moralOther    match        n     n   1 0.9371
## 2713   moralOther   moralOther    match        n     n   1 0.7468
## 2714   moralOther   moralOther mismatch        m     m   1 0.5865
## 2715    moralSelf    moralSelf    match        n     n   1 0.5790
## 2716  immoralSelf  immoralSelf    match        n     n   1 0.5113
## 2717  immoralSelf  immoralSelf    match        n     n   1 0.5062
## 2718 immoralOther immoralOther mismatch        m     m   1 0.5972
## 2719   moralOther   moralOther mismatch        m     m   1 0.6820
## 2720    moralSelf    moralSelf mismatch        m     m   1 0.6445
## 2721 immoralOther immoralOther mismatch        m     m   1 0.8262
## 2722   moralOther   moralOther    match        n     n   1 0.5195
## 2723  immoralSelf  immoralSelf mismatch        m     m   1 0.7346
## 2724    moralSelf    moralSelf mismatch        m     m   1 0.7540
## 2725 immoralOther immoralOther mismatch        m     m   1 0.8539
## 2726    moralSelf    moralSelf    match        n     n   1 0.6516
## 2727   moralOther   moralOther mismatch        m     m   1 0.8413
## 2728    moralSelf    moralSelf    match        n     n   1 0.6509
## 2729 immoralOther immoralOther    match        n     n   1 0.9846
## 2730    moralSelf    moralSelf mismatch        m     m   1 0.8412
## 2731  immoralSelf  immoralSelf    match        n     n   1 0.7628
## 2732   moralOther   moralOther    match        n     n   1 0.8748
## 2733 immoralOther immoralOther    match        n     n   1 0.7050
## 2734  immoralSelf  immoralSelf mismatch        m     m   1 0.8119
## 2735 immoralOther immoralOther    match        n     n   1 0.9730
## 2736  immoralSelf  immoralSelf mismatch        m     m   1 0.9331
## 2737 immoralOther immoralOther    match        n     m   0 0.6965
## 2738   moralOther   moralOther mismatch        m     m   1 0.9112
## 2739  immoralSelf  immoralSelf    match        n     n   1 0.6261
## 2740 immoralOther immoralOther mismatch        m     m   1 0.9114
## 2741   moralOther   moralOther mismatch        m     m   1 0.8503
## 2742   moralOther   moralOther    match        n     m   0 0.7521
## 2743 immoralOther immoralOther    match        n     n   1 0.7559
## 2744    moralSelf    moralSelf mismatch        m     m   1 0.7037
## 2745  immoralSelf  immoralSelf mismatch        m     m   1 0.7387
## 2746  immoralSelf  immoralSelf mismatch        m     m   1 0.7783
## 2747    moralSelf    moralSelf mismatch        m     m   1 0.8547
## 2748    moralSelf    moralSelf    match        n     n   1 0.6444
## 2749    moralSelf    moralSelf mismatch        m     m   1 0.9941
## 2750 immoralOther immoralOther mismatch        m     m   1 0.6987
## 2751    moralSelf    moralSelf    match        n     n   1 0.7215
## 2752  immoralSelf  immoralSelf    match        n     n   1 0.5926
## 2753 immoralOther immoralOther    match        n     m   0 0.7533
## 2754  immoralSelf  immoralSelf mismatch        m     m   1 0.7810
## 2755 immoralOther immoralOther mismatch        m     m   1 0.7932
## 2756   moralOther   moralOther    match        n     n   1 0.6179
## 2757  immoralSelf  immoralSelf    match        n     n   1 0.9390
## 2758   moralOther   moralOther    match        n     n   1 0.6267
## 2759   moralOther   moralOther mismatch        m     n   0 0.7598
## 2760    moralSelf    moralSelf    match        n     n   1 0.6038
## 2761    moralSelf    moralSelf    match        n  <NA>  -1 1.0841
## 2762   moralOther   moralOther mismatch        m     m   1 0.8878
## 2763  immoralSelf  immoralSelf    match        n     n   1 0.5862
## 2764   moralOther   moralOther mismatch        m     m   1 0.7308
## 2765 immoralOther immoralOther mismatch        m     m   1 0.6063
## 2766    moralSelf    moralSelf mismatch        m     m   1 0.7070
## 2767 immoralOther immoralOther    match        n     n   1 0.7739
## 2768   moralOther   moralOther    match        n     n   1 0.4983
## 2769 immoralOther immoralOther mismatch        m     m   1 0.6649
## 2770   moralOther   moralOther mismatch        m     m   1 0.6109
## 2771    moralSelf    moralSelf    match        n     n   1 0.6280
## 2772 immoralOther immoralOther    match        n     n   1 0.7853
## 2773  immoralSelf  immoralSelf mismatch        m     m   1 0.5778
## 2774  immoralSelf  immoralSelf    match        n     n   1 0.5021
## 2775    moralSelf    moralSelf mismatch        m     m   1 0.7328
## 2776   moralOther   moralOther    match        n     n   1 0.7522
## 2777   moralOther   moralOther    match        n     n   1 0.5480
## 2778  immoralSelf  immoralSelf    match        n     n   1 0.5317
## 2779  immoralSelf  immoralSelf mismatch        m     m   1 0.5351
## 2780    moralSelf    moralSelf    match        n     n   1 0.6825
## 2781  immoralSelf  immoralSelf mismatch        m     m   1 0.5249
## 2782 immoralOther immoralOther mismatch        m     m   1 0.8040
## 2783 immoralOther immoralOther    match        n     n   1 0.6489
## 2784    moralSelf    moralSelf mismatch        m     m   1 0.7105
## 2785    moralSelf    moralSelf mismatch        m     n   0 0.8496
## 2786   moralOther   moralOther    match        n     n   1 0.6314
## 2787 immoralOther immoralOther    match        n  <NA>  -1 1.0841
## 2788   moralOther   moralOther mismatch        m     m   1 0.6951
## 2789   moralOther   moralOther    match        n     n   1 0.5816
## 2790  immoralSelf  immoralSelf    match        n     m   0 0.5780
## 2791   moralOther   moralOther mismatch        m     n   0 0.4384
## 2792  immoralSelf  immoralSelf mismatch        m     m   1 0.8958
## 2793  immoralSelf  immoralSelf    match        n     n   1 0.7185
## 2794    moralSelf    moralSelf    match        n     n   1 0.9897
## 2795  immoralSelf  immoralSelf    match        n     n   1 0.5902
## 2796   moralOther   moralOther    match        n     n   1 0.6788
## 2797    moralSelf    moralSelf mismatch        m     m   1 0.8210
## 2798 immoralOther immoralOther mismatch        m     m   1 0.8343
## 2799 immoralOther immoralOther mismatch        m     n   0 1.0518
## 2800 immoralOther immoralOther mismatch        m     m   1 0.8335
## 2801    moralSelf    moralSelf    match        n     n   1 0.6591
## 2802    moralSelf    moralSelf mismatch        m     m   1 0.8530
## 2803  immoralSelf  immoralSelf mismatch        m     m   1 0.6268
## 2804  immoralSelf  immoralSelf mismatch        m     m   1 0.5842
## 2805    moralSelf    moralSelf    match        n     m   0 0.5884
## 2806   moralOther   moralOther mismatch        m     n   0 0.6850
## 2807 immoralOther immoralOther    match        n  <NA>  -1 1.0841
## 2808 immoralOther immoralOther    match        n     n   1 0.8259
## 2809  immoralSelf  immoralSelf    match        n     n   1 0.6792
## 2810 immoralOther immoralOther    match        n     m   0 0.6054
## 2811 immoralOther immoralOther    match        n     n   1 0.5782
## 2812 immoralOther immoralOther mismatch        m     n   0 0.7666
## 2813   moralOther   moralOther    match        n     n   1 0.5186
## 2814 immoralOther immoralOther mismatch        m     m   1 0.6858
## 2815  immoralSelf  immoralSelf mismatch        m     n   0 0.6643
## 2816   moralOther   moralOther mismatch        m     m   1 0.6742
## 2817   moralOther   moralOther mismatch        m     m   1 0.8484
## 2818  immoralSelf  immoralSelf mismatch        m     m   1 0.6941
## 2819    moralSelf    moralSelf mismatch        m     m   1 0.8208
## 2820  immoralSelf  immoralSelf    match        n     n   1 0.7302
## 2821 immoralOther immoralOther mismatch        m     m   1 0.7735
## 2822    moralSelf    moralSelf    match        n     n   1 0.6258
## 2823    moralSelf    moralSelf    match        n     n   1 0.6872
## 2824   moralOther   moralOther    match        n     n   1 0.9136
## 2825    moralSelf    moralSelf mismatch        m     m   1 0.7645
## 2826   moralOther   moralOther mismatch        m     n   0 0.6765
## 2827  immoralSelf  immoralSelf    match        n     n   1 0.5867
## 2828    moralSelf    moralSelf    match        n     n   1 0.8274
## 2829  immoralSelf  immoralSelf mismatch        m     m   1 0.6087
## 2830   moralOther   moralOther    match        n     n   1 0.6935
## 2831 immoralOther immoralOther    match        n     n   1 0.7482
## 2832    moralSelf    moralSelf mismatch        m     m   1 0.9477
## 2833  immoralSelf  immoralSelf    match        n     n   1 0.6381
## 2834 immoralOther immoralOther mismatch        m     m   1 0.7276
## 2835    moralSelf    moralSelf    match        n     n   1 0.5070
## 2836  immoralSelf  immoralSelf    match        n     n   1 0.5179
## 2837    moralSelf    moralSelf    match        n     m   0 0.6291
## 2838 immoralOther immoralOther mismatch        m     m   1 0.5863
## 2839  immoralSelf  immoralSelf    match        n     n   1 0.6349
## 2840    moralSelf    moralSelf mismatch        m     m   1 0.6844
## 2841   moralOther   moralOther mismatch        m     m   1 0.6309
## 2842  immoralSelf  immoralSelf mismatch        m     m   1 0.6121
## 2843   moralOther   moralOther    match        n     n   1 0.5892
## 2844   moralOther   moralOther    match        n     n   1 0.6858
## 2845   moralOther   moralOther mismatch        m     m   1 0.5442
## 2846  immoralSelf  immoralSelf mismatch        m     m   1 0.7598
## 2847    moralSelf    moralSelf    match        n     n   1 0.5956
## 2848    moralSelf    moralSelf mismatch        m     m   1 0.6443
## 2849 immoralOther immoralOther    match        n     n   1 0.7541
## 2850  immoralSelf  immoralSelf mismatch        m     m   1 0.5659
## 2851   moralOther   moralOther    match        n     n   1 0.6059
## 2852   moralOther   moralOther mismatch        m     m   1 0.5627
## 2853 immoralOther immoralOther mismatch        m     m   1 0.6909
## 2854 immoralOther immoralOther    match        n     n   1 0.6373
## 2855 immoralOther immoralOther    match        n     n   1 0.7508
## 2856    moralSelf    moralSelf mismatch        m     m   1 0.8586
## 2857    moralSelf    moralSelf    match        n     n   1 0.6847
## 2858  immoralSelf  immoralSelf    match        n     n   1 0.4631
## 2859 immoralOther immoralOther    match        n     m   0 0.6250
## 2860  immoralSelf  immoralSelf mismatch        m     m   1 0.6064
## 2861    moralSelf    moralSelf    match        n     n   1 0.5551
## 2862   moralOther   moralOther mismatch        m     m   1 0.6150
## 2863   moralOther   moralOther mismatch        m     m   1 0.7760
## 2864 immoralOther immoralOther mismatch        m     m   1 0.8603
## 2865   moralOther   moralOther    match        n     n   1 0.8623
## 2866  immoralSelf  immoralSelf mismatch        m     m   1 0.6243
## 2867 immoralOther immoralOther    match        n     n   1 0.6536
## 2868    moralSelf    moralSelf mismatch        m     m   1 0.6433
## 2869 immoralOther immoralOther    match        n     n   1 0.7489
## 2870   moralOther   moralOther mismatch        m     n   0 0.6925
## 2871  immoralSelf  immoralSelf    match        n     n   1 0.5232
## 2872    moralSelf    moralSelf mismatch        m     n   0 0.5944
## 2873  immoralSelf  immoralSelf    match        n     n   1 0.5629
## 2874   moralOther   moralOther    match        n     n   1 0.8110
## 2875 immoralOther immoralOther mismatch        m     m   1 0.5800
## 2876    moralSelf    moralSelf    match        n     n   1 0.7124
## 2877  immoralSelf  immoralSelf mismatch        m     m   1 0.8032
## 2878   moralOther   moralOther    match        n     n   1 0.6081
## 2879 immoralOther immoralOther mismatch        m     m   1 0.5808
## 2880    moralSelf    moralSelf mismatch        m     m   1 0.7692
## 2881    moralSelf    moralSelf    match        n     n   1 0.9881
## 2882 immoralOther immoralOther mismatch        m     m   1 0.8927
## 2883  immoralSelf  immoralSelf mismatch        m     m   1 0.7075
## 2884   moralOther   moralOther    match        n     n   1 0.7822
## 2885    moralSelf    moralSelf    match        n     n   1 0.8826
## 2886  immoralSelf  immoralSelf    match        n     m   0 0.6930
## 2887    moralSelf    moralSelf mismatch        m     m   1 0.7638
## 2888  immoralSelf  immoralSelf mismatch        m     m   1 0.9718
## 2889 immoralOther immoralOther mismatch        m     m   1 0.8680
## 2890 immoralOther immoralOther    match        n     n   1 0.6182
## 2891  immoralSelf  immoralSelf    match        n     n   1 0.7072
## 2892 immoralOther immoralOther    match        n     m   0 0.9741
## 2893 immoralOther immoralOther    match        n     n   1 1.0184
## 2894   moralOther   moralOther    match        n     n   1 0.8116
## 2895   moralOther   moralOther mismatch        m     m   1 0.5807
## 2896    moralSelf    moralSelf    match        n     n   1 0.6489
## 2897   moralOther   moralOther mismatch        m     m   1 1.0386
## 2898   moralOther   moralOther    match        n     n   1 0.9122
## 2899  immoralSelf  immoralSelf    match        n     n   1 0.5312
## 2900   moralOther   moralOther mismatch        m     m   1 0.9985
## 2901    moralSelf    moralSelf mismatch        m     m   1 0.8593
## 2902 immoralOther immoralOther mismatch        m     m   1 0.7733
## 2903  immoralSelf  immoralSelf mismatch        m  <NA>  -1 1.0841
## 2904    moralSelf    moralSelf mismatch        m     m   1 0.6879
## 2905   moralOther   moralOther mismatch        m     n   0 0.6423
## 2906  immoralSelf  immoralSelf mismatch        m     m   1 0.7478
## 2907  immoralSelf  immoralSelf    match        n     n   1 0.5714
## 2908   moralOther   moralOther mismatch        m     m   1 0.6276
## 2909   moralOther   moralOther    match        n     n   1 0.7609
## 2910   moralOther   moralOther    match        n     n   1 0.6127
## 2911 immoralOther immoralOther    match        n     n   1 0.7577
## 2912 immoralOther immoralOther mismatch        m     m   1 0.6415
## 2913    moralSelf    moralSelf mismatch        m     m   1 0.7792
## 2914  immoralSelf  immoralSelf mismatch        m     m   1 1.0474
## 2915   moralOther   moralOther    match        n     n   1 0.6613
## 2916    moralSelf    moralSelf    match        n     n   1 0.6631
## 2917  immoralSelf  immoralSelf mismatch        m     m   1 0.6091
## 2918    moralSelf    moralSelf mismatch        m     m   1 0.6982
## 2919 immoralOther immoralOther    match        n     n   1 0.9768
## 2920    moralSelf    moralSelf mismatch        m     m   1 0.8051
## 2921  immoralSelf  immoralSelf    match        n     n   1 0.5461
## 2922  immoralSelf  immoralSelf    match        n     n   1 0.4656
## 2923   moralOther   moralOther mismatch        m     m   1 0.6196
## 2924 immoralOther immoralOther    match        n     n   1 0.9006
## 2925    moralSelf    moralSelf    match        n     m   0 0.6912
## 2926 immoralOther immoralOther mismatch        m     m   1 0.5381
## 2927    moralSelf    moralSelf    match        n     n   1 0.6214
## 2928 immoralOther immoralOther mismatch        m     m   1 0.6666
## 2929  immoralSelf  immoralSelf    match        n     n   1 0.5446
## 2930 immoralOther immoralOther    match        n     n   1 0.7123
## 2931 immoralOther immoralOther mismatch        m     m   1 0.5712
## 2932  immoralSelf  immoralSelf mismatch        m     m   1 0.7553
## 2933   moralOther   moralOther mismatch        m     m   1 0.7832
## 2934   moralOther   moralOther    match        n     m   0 0.5076
## 2935   moralOther   moralOther mismatch        m     m   1 0.5824
## 2936    moralSelf    moralSelf mismatch        m     m   1 0.7548
## 2937    moralSelf    moralSelf    match        n     m   0 0.7747
## 2938    moralSelf    moralSelf mismatch        m     m   1 1.0190
## 2939  immoralSelf  immoralSelf mismatch        m     n   0 0.5241
## 2940 immoralOther immoralOther    match        n  <NA>  -1 1.0841
## 2941 immoralOther immoralOther    match        n     n   1 0.9576
## 2942   moralOther   moralOther mismatch        m     m   1 0.6016
## 2943   moralOther   moralOther    match        n     n   1 0.5423
## 2944    moralSelf    moralSelf mismatch        m     m   1 0.7579
## 2945    moralSelf    moralSelf    match        n     n   1 0.5778
## 2946   moralOther   moralOther    match        n     n   1 0.6702
## 2947  immoralSelf  immoralSelf mismatch        m     m   1 0.5564
## 2948 immoralOther immoralOther mismatch        m     m   1 0.7442
## 2949 immoralOther immoralOther mismatch        m     m   1 0.7439
## 2950  immoralSelf  immoralSelf    match        n     n   1 0.7435
## 2951  immoralSelf  immoralSelf    match        n     n   1 0.6392
## 2952    moralSelf    moralSelf    match        n     n   1 0.6726
## 2953  immoralSelf  immoralSelf    match        n     m   0 0.5027
## 2954 immoralOther immoralOther    match        n     m   0 0.7335
## 2955   moralOther   moralOther    match        n     n   1 0.4409
## 2956   moralOther   moralOther mismatch        m     m   1 0.6983
## 2957    moralSelf    moralSelf mismatch        m     n   0 0.6810
## 2958 immoralOther immoralOther mismatch        m     m   1 0.8436
## 2959    moralSelf    moralSelf    match        n     n   1 0.9810
## 2960  immoralSelf  immoralSelf    match        n     n   1 0.5774
## 2961    moralSelf    moralSelf    match        n     n   1 0.6298
## 2962    moralSelf    moralSelf mismatch        m     m   1 0.8910
## 2963  immoralSelf  immoralSelf mismatch        m     n   0 0.7057
## 2964 immoralOther immoralOther mismatch        m     m   1 0.7646
## 2965  immoralSelf  immoralSelf mismatch        m     m   1 0.7967
## 2966  immoralSelf  immoralSelf    match        n     n   1 0.6133
## 2967 immoralOther immoralOther    match        n     n   1 0.7982
## 2968   moralOther   moralOther mismatch        m     m   1 0.7471
## 2969   moralOther   moralOther mismatch        m     m   1 0.6428
## 2970    moralSelf    moralSelf    match        n     n   1 0.6122
## 2971  immoralSelf  immoralSelf mismatch        m     m   1 0.7652
## 2972 immoralOther immoralOther mismatch        m     m   1 0.7013
## 2973   moralOther   moralOther    match        n     n   1 0.5481
## 2974    moralSelf    moralSelf mismatch        m     m   1 0.6517
## 2975 immoralOther immoralOther    match        n     n   1 0.6733
## 2976   moralOther   moralOther    match        n     n   1 0.8157
## 2977   moralOther   moralOther mismatch        m     m   1 0.8151
## 2978  immoralSelf  immoralSelf    match        n     n   1 0.6802
## 2979    moralSelf    moralSelf mismatch        m     m   1 0.8826
## 2980 immoralOther immoralOther mismatch        m     m   1 0.7650
## 2981   moralOther   moralOther mismatch        m     m   1 0.6052
## 2982 immoralOther immoralOther    match        n     n   1 1.0500
## 2983  immoralSelf  immoralSelf    match        n     n   1 0.5037
## 2984   moralOther   moralOther    match        n     n   1 0.6027
## 2985    moralSelf    moralSelf mismatch        m     m   1 0.7194
## 2986    moralSelf    moralSelf    match        n     n   1 0.6547
## 2987 immoralOther immoralOther    match        n     n   1 0.6203
## 2988 immoralOther immoralOther mismatch        m     m   1 0.6214
## 2989   moralOther   moralOther    match        n     n   1 0.9386
## 2990   moralOther   moralOther    match        n     n   1 0.5223
## 2991    moralSelf    moralSelf    match        n     n   1 0.6174
## 2992  immoralSelf  immoralSelf mismatch        m     m   1 0.6104
## 2993  immoralSelf  immoralSelf mismatch        m  <NA>  -1 1.0841
## 2994 immoralOther immoralOther    match        n     n   1 0.6219
## 2995    moralSelf    moralSelf mismatch        m     m   1 0.6912
## 2996    moralSelf    moralSelf    match        n     n   1 0.5859
## 2997   moralOther   moralOther mismatch        m     n   0 0.5146
## 2998 immoralOther immoralOther mismatch        m     n   0 0.8294
## 2999  immoralSelf  immoralSelf    match        n     n   1 0.7307
## 3000  immoralSelf  immoralSelf mismatch        m     m   1 0.7419
## 3001    moralSelf    moralSelf mismatch        m     n   0 0.5497
## 3002   moralOther   moralOther mismatch        m     m   1 0.7333
## 3003  immoralSelf  immoralSelf    match        n  <NA>  -1 1.0841
## 3004    moralSelf    moralSelf    match        n     n   1 0.6112
## 3005  immoralSelf  immoralSelf mismatch        m     m   1 0.8122
## 3006    moralSelf    moralSelf mismatch        m     m   1 0.7652
## 3007   moralOther   moralOther    match        n     n   1 0.5413
## 3008    moralSelf    moralSelf    match        n     n   1 0.5810
## 3009 immoralOther immoralOther    match        n     m   0 0.5853
## 3010  immoralSelf  immoralSelf mismatch        m     m   1 0.6658
## 3011 immoralOther immoralOther    match        n     n   1 0.5799
## 3012 immoralOther immoralOther mismatch        m     m   1 0.5762
## 3013  immoralSelf  immoralSelf    match        n     n   1 0.5645
## 3014 immoralOther immoralOther mismatch        m     n   0 0.9647
## 3015    moralSelf    moralSelf    match        n     n   1 0.4927
## 3016  immoralSelf  immoralSelf    match        n     n   1 0.5554
## 3017    moralSelf    moralSelf mismatch        m     n   0 0.8552
## 3018   moralOther   moralOther mismatch        m     m   1 0.5330
## 3019 immoralOther immoralOther mismatch        m     n   0 0.6084
## 3020   moralOther   moralOther    match        n     n   1 0.6611
## 3021   moralOther   moralOther mismatch        m     m   1 0.5270
## 3022   moralOther   moralOther    match        n     n   1 0.5785
## 3023  immoralSelf  immoralSelf mismatch        m     m   1 0.6309
## 3024 immoralOther immoralOther    match        n     n   1 0.7641
## 3025   moralOther   moralOther mismatch        m     m   1 0.6580
## 3026    moralSelf    moralSelf    match        n     n   1 0.5082
## 3027   moralOther   moralOther mismatch        m     m   1 0.4550
## 3028 immoralOther immoralOther    match        n     m   0 0.5728
## 3029  immoralSelf  immoralSelf mismatch        m     m   1 0.9889
## 3030  immoralSelf  immoralSelf    match        n     n   1 0.4774
## 3031 immoralOther immoralOther mismatch        m     m   1 0.6596
## 3032  immoralSelf  immoralSelf mismatch        m     m   1 0.7816
## 3033 immoralOther immoralOther    match        n     n   1 0.4900
## 3034   moralOther   moralOther    match        n     n   1 0.6764
## 3035  immoralSelf  immoralSelf mismatch        m     m   1 0.6426
## 3036 immoralOther immoralOther mismatch        m     m   1 0.6681
## 3037    moralSelf    moralSelf mismatch        m     m   1 0.9063
## 3038    moralSelf    moralSelf mismatch        m     m   1 0.7972
## 3039  immoralSelf  immoralSelf    match        n     n   1 0.7301
## 3040   moralOther   moralOther mismatch        m     m   1 1.0614
## 3041  immoralSelf  immoralSelf    match        n     n   1 0.4954
## 3042 immoralOther immoralOther    match        n     n   1 0.6861
## 3043   moralOther   moralOther    match        n     n   1 0.6566
## 3044   moralOther   moralOther    match        n     n   1 0.6505
## 3045 immoralOther immoralOther mismatch        m     m   1 0.6962
## 3046    moralSelf    moralSelf mismatch        m     m   1 0.7029
## 3047    moralSelf    moralSelf    match        n     m   0 0.7059
## 3048    moralSelf    moralSelf    match        n     n   1 0.5968
## 3049   moralOther   moralOther mismatch        m     n   0 0.6453
## 3050 immoralOther immoralOther    match        n     n   1 1.0271
## 3051    moralSelf    moralSelf mismatch        m     n   0 0.6164
## 3052  immoralSelf  immoralSelf mismatch        m     m   1 0.7058
## 3053  immoralSelf  immoralSelf    match        n     n   1 0.6283
## 3054    moralSelf    moralSelf mismatch        m     m   1 0.7136
## 3055 immoralOther immoralOther mismatch        m     m   1 0.8365
## 3056  immoralSelf  immoralSelf    match        n     n   1 0.7580
## 3057    moralSelf    moralSelf    match        n     n   1 0.6499
## 3058    moralSelf    moralSelf mismatch        m     m   1 0.6957
## 3059 immoralOther immoralOther    match        n     n   1 0.7344
## 3060  immoralSelf  immoralSelf mismatch        m     m   1 0.6817
## 3061   moralOther   moralOther    match        n     m   0 0.7322
## 3062 immoralOther immoralOther    match        n     n   1 0.8476
## 3063   moralOther   moralOther    match        n     n   1 0.7893
## 3064  immoralSelf  immoralSelf mismatch        m     n   0 0.5259
## 3065    moralSelf    moralSelf    match        n     n   1 0.7810
## 3066   moralOther   moralOther mismatch        m     m   1 0.7134
## 3067 immoralOther immoralOther mismatch        m     m   1 0.6283
## 3068  immoralSelf  immoralSelf    match        n     n   1 0.6495
## 3069   moralOther   moralOther mismatch        m     m   1 0.5514
## 3070 immoralOther immoralOther mismatch        m     m   1 0.6352
## 3071    moralSelf    moralSelf    match        n     m   0 0.7727
## 3072   moralOther   moralOther    match        n     m   0 0.5968
## 3073  immoralSelf  immoralSelf    match        n     n   1 0.6894
## 3074 immoralOther immoralOther mismatch        m     m   1 0.5719
## 3075 immoralOther immoralOther mismatch        m     m   1 0.7080
## 3076 immoralOther immoralOther mismatch        m     m   1 0.7428
## 3077   moralOther   moralOther mismatch        m     m   1 0.7826
## 3078 immoralOther immoralOther    match        n     n   1 0.7789
## 3079   moralOther   moralOther mismatch        m     m   1 0.6152
## 3080    moralSelf    moralSelf    match        n     n   1 0.6962
## 3081 immoralOther immoralOther    match        n     n   1 0.7189
## 3082  immoralSelf  immoralSelf    match        n     n   1 0.5661
## 3083    moralSelf    moralSelf mismatch        m     m   1 0.9421
## 3084  immoralSelf  immoralSelf mismatch        m     m   1 0.6778
## 3085  immoralSelf  immoralSelf mismatch        m     m   1 0.6120
## 3086  immoralSelf  immoralSelf    match        n     n   1 0.5651
## 3087    moralSelf    moralSelf mismatch        m     m   1 0.8291
## 3088   moralOther   moralOther mismatch        m     m   1 0.5304
## 3089  immoralSelf  immoralSelf mismatch        m     m   1 0.7897
## 3090    moralSelf    moralSelf    match        n     n   1 0.6384
## 3091 immoralOther immoralOther    match        n     n   1 0.7919
## 3092   moralOther   moralOther    match        n     n   1 0.6084
## 3093   moralOther   moralOther    match        n     n   1 0.5012
## 3094    moralSelf    moralSelf    match        n     n   1 0.6040
## 3095    moralSelf    moralSelf mismatch        m     m   1 0.8967
## 3096   moralOther   moralOther    match        n     n   1 0.6394
## 3097   moralOther   moralOther    match        n     n   1 0.5449
## 3098    moralSelf    moralSelf mismatch        m     m   1 0.8086
## 3099  immoralSelf  immoralSelf    match        n     n   1 0.5733
## 3100   moralOther   moralOther    match        n     n   1 0.6374
## 3101   moralOther   moralOther mismatch        m     m   1 0.6869
## 3102 immoralOther immoralOther    match        n     n   1 0.6814
## 3103 immoralOther immoralOther mismatch        m     m   1 0.7719
## 3104  immoralSelf  immoralSelf    match        n     n   1 0.8199
## 3105  immoralSelf  immoralSelf mismatch        m     n   0 0.5729
## 3106  immoralSelf  immoralSelf mismatch        m     m   1 1.0770
## 3107    moralSelf    moralSelf mismatch        m     m   1 0.7234
## 3108    moralSelf    moralSelf    match        n     n   1 0.5705
## 3109    moralSelf    moralSelf    match        n     n   1 0.7066
## 3110 immoralOther immoralOther mismatch        m     m   1 0.7095
## 3111 immoralOther immoralOther    match        n     n   1 0.8084
## 3112 immoralOther immoralOther    match        n     n   1 0.7492
## 3113    moralSelf    moralSelf mismatch        m     m   1 0.6128
## 3114  immoralSelf  immoralSelf    match        n     n   1 0.5098
## 3115   moralOther   moralOther    match        n     m   0 0.6327
## 3116   moralOther   moralOther mismatch        m     m   1 0.6742
## 3117 immoralOther immoralOther mismatch        m     n   0 0.6005
## 3118   moralOther   moralOther mismatch        m     n   0 0.6774
## 3119    moralSelf    moralSelf    match        n     n   1 0.7155
## 3120  immoralSelf  immoralSelf mismatch        m  <NA>  -1 1.0841
## 3121   moralOther   moralOther    match        m     m   1 0.7954
## 3122   moralOther   moralOther mismatch        n     m   0 0.6636
## 3123    moralSelf    moralSelf mismatch        n     m   0 0.7178
## 3124  immoralSelf  immoralSelf    match        m  <NA>  -1 1.0850
## 3125   moralOther   moralOther mismatch        n     n   1 0.8420
## 3126  immoralSelf  immoralSelf mismatch        n     m   0 0.8281
## 3127    moralSelf    moralSelf mismatch        n     n   1 1.0642
## 3128 immoralOther immoralOther    match        m     m   1 0.7744
## 3129 immoralOther immoralOther    match        m     m   1 0.6946
## 3130 immoralOther immoralOther mismatch        n     n   1 0.6627
## 3131    moralSelf    moralSelf    match        m     m   1 0.7328
## 3132    moralSelf    moralSelf    match        m     m   1 0.5410
## 3133 immoralOther immoralOther mismatch        n     n   1 0.8672
## 3134 immoralOther immoralOther mismatch        n     n   1 0.7188
## 3135    moralSelf    moralSelf    match        m     m   1 0.5617
## 3136    moralSelf    moralSelf mismatch        n     n   1 0.7395
## 3137  immoralSelf  immoralSelf mismatch        n     n   1 1.0416
## 3138 immoralOther immoralOther    match        m     m   1 0.8277
## 3139  immoralSelf  immoralSelf    match        m     n   0 1.0079
## 3140  immoralSelf  immoralSelf mismatch        n     n   1 0.8962
## 3141   moralOther   moralOther mismatch        n     m   0 0.7922
## 3142  immoralSelf  immoralSelf    match        m     n   0 0.8044
## 3143   moralOther   moralOther    match        m     m   1 0.7505
## 3144   moralOther   moralOther    match        m     n   0 0.7146
## 3145    moralSelf    moralSelf mismatch        n     n   1 0.8128
## 3146 immoralOther immoralOther    match        m     m   1 1.0129
## 3147    moralSelf    moralSelf    match        m     m   1 0.6051
## 3148 immoralOther immoralOther    match        m     m   1 0.8072
## 3149 immoralOther immoralOther mismatch        n     n   1 0.7993
## 3150   moralOther   moralOther    match        m     m   1 0.7454
## 3151 immoralOther immoralOther mismatch        n     n   1 0.8796
## 3152  immoralSelf  immoralSelf mismatch        n     n   1 0.7777
## 3153  immoralSelf  immoralSelf    match        m     m   1 0.7359
## 3154  immoralSelf  immoralSelf    match        m  <NA>  -1 1.0850
## 3155    moralSelf    moralSelf    match        m     m   1 0.5781
## 3156  immoralSelf  immoralSelf mismatch        n     n   1 0.7602
## 3157  immoralSelf  immoralSelf    match        m     m   1 1.0584
## 3158   moralOther   moralOther    match        m     m   1 1.0325
## 3159    moralSelf    moralSelf mismatch        n     n   1 0.6528
## 3160  immoralSelf  immoralSelf mismatch        n     m   0 0.7968
## 3161   moralOther   moralOther mismatch        n     m   0 0.6329
## 3162   moralOther   moralOther    match        m     m   1 0.7812
## 3163    moralSelf    moralSelf    match        m     m   1 0.9472
## 3164   moralOther   moralOther mismatch        n     n   1 0.5833
## 3165 immoralOther immoralOther    match        m     n   0 0.6376
## 3166 immoralOther immoralOther mismatch        n     n   1 0.6476
## 3167   moralOther   moralOther mismatch        n     n   1 0.6805
## 3168    moralSelf    moralSelf mismatch        n     n   1 0.7859
## 3169    moralSelf    moralSelf mismatch        n     n   1 0.7760
## 3170   moralOther   moralOther mismatch        n     n   1 1.0162
## 3171   moralOther   moralOther    match        m     m   1 0.9123
## 3172  immoralSelf  immoralSelf mismatch        n     n   1 0.9245
## 3173   moralOther   moralOther    match        m     m   1 0.7826
## 3174  immoralSelf  immoralSelf    match        m     m   1 1.0288
## 3175 immoralOther immoralOther    match        m     m   1 0.6490
## 3176 immoralOther immoralOther    match        m     m   1 0.6250
## 3177  immoralSelf  immoralSelf mismatch        n     m   0 0.6472
## 3178  immoralSelf  immoralSelf    match        m     m   1 0.7913
## 3179    moralSelf    moralSelf    match        m     m   1 0.5555
## 3180    moralSelf    moralSelf mismatch        n     m   0 0.6157
## 3181   moralOther   moralOther    match        m     m   1 0.7196
## 3182   moralOther   moralOther mismatch        n     m   0 0.7138
## 3183  immoralSelf  immoralSelf mismatch        n     n   1 1.0379
## 3184 immoralOther immoralOther mismatch        n     n   1 0.8241
## 3185  immoralSelf  immoralSelf    match        m     n   0 0.9341
## 3186   moralOther   moralOther mismatch        n  <NA>  -1 1.0850
## 3187    moralSelf    moralSelf mismatch        n     m   0 0.6965
## 3188 immoralOther immoralOther mismatch        n     n   1 0.8252
## 3189 immoralOther immoralOther    match        m     n   0 0.8468
## 3190    moralSelf    moralSelf    match        m     m   1 0.5689
## 3191 immoralOther immoralOther mismatch        n     n   1 0.5771
## 3192    moralSelf    moralSelf    match        m     m   1 0.7372
## 3193    moralSelf    moralSelf    match        m     m   1 0.5361
## 3194 immoralOther immoralOther    match        m     m   1 0.6640
## 3195  immoralSelf  immoralSelf    match        m     m   1 0.8464
## 3196  immoralSelf  immoralSelf mismatch        n     n   1 0.7445
## 3197   moralOther   moralOther    match        m     m   1 0.6306
## 3198   moralOther   moralOther mismatch        n     n   1 0.7389
## 3199  immoralSelf  immoralSelf mismatch        n  <NA>  -1 1.0850
## 3200 immoralOther immoralOther mismatch        n     n   1 0.6754
## 3201 immoralOther immoralOther    match        m     m   1 0.7592
## 3202  immoralSelf  immoralSelf    match        m  <NA>  -1 1.0850
## 3203  immoralSelf  immoralSelf mismatch        n     n   1 0.6915
## 3204 immoralOther immoralOther mismatch        n     n   1 0.6037
## 3205   moralOther   moralOther    match        m     m   1 0.8217
## 3206  immoralSelf  immoralSelf    match        m     n   0 0.7799
## 3207    moralSelf    moralSelf mismatch        n     n   1 0.8901
## 3208   moralOther   moralOther    match        m     m   1 0.7944
## 3209   moralOther   moralOther mismatch        n     n   1 0.7103
## 3210    moralSelf    moralSelf mismatch        n     n   1 0.8265
## 3211   moralOther   moralOther mismatch        n     n   1 0.8725
## 3212 immoralOther immoralOther    match        m     m   1 0.6528
## 3213    moralSelf    moralSelf    match        m     m   1 0.5568
## 3214    moralSelf    moralSelf    match        m     m   1 0.6011
## 3215    moralSelf    moralSelf mismatch        n     n   1 0.7471
## 3216 immoralOther immoralOther mismatch        n     n   1 0.6792
## 3217   moralOther   moralOther    match        m     m   1 0.8014
## 3218  immoralSelf  immoralSelf    match        m     n   0 0.6596
## 3219  immoralSelf  immoralSelf mismatch        n     n   1 0.8177
## 3220    moralSelf    moralSelf    match        m     m   1 0.6398
## 3221  immoralSelf  immoralSelf    match        m     n   0 0.6019
## 3222  immoralSelf  immoralSelf    match        m     n   0 0.6041
## 3223 immoralOther immoralOther    match        m     n   0 0.6461
## 3224   moralOther   moralOther mismatch        n     n   1 0.7602
## 3225   moralOther   moralOther    match        m     m   1 0.9145
## 3226 immoralOther immoralOther mismatch        n     m   0 0.9666
## 3227  immoralSelf  immoralSelf mismatch        n     m   0 0.5447
## 3228    moralSelf    moralSelf mismatch        n     n   1 0.6871
## 3229 immoralOther immoralOther    match        m     m   1 0.7050
## 3230   moralOther   moralOther mismatch        n     n   1 0.9591
## 3231    moralSelf    moralSelf mismatch        n     n   1 0.7452
## 3232 immoralOther immoralOther    match        m     m   1 0.6634
## 3233   moralOther   moralOther mismatch        n     n   1 1.0215
## 3234    moralSelf    moralSelf mismatch        n     n   1 0.7636
## 3235  immoralSelf  immoralSelf mismatch        n     n   1 0.8298
## 3236    moralSelf    moralSelf    match        m     m   1 0.5779
## 3237 immoralOther immoralOther mismatch        n     n   1 0.8000
## 3238    moralSelf    moralSelf    match        m     m   1 0.5862
## 3239   moralOther   moralOther    match        m     m   1 0.6483
## 3240 immoralOther immoralOther mismatch        n     n   1 0.7205
## 3241 immoralOther immoralOther    match        m  <NA>  -1 1.0850
## 3242   moralOther   moralOther mismatch        n     n   1 0.7590
## 3243    moralSelf    moralSelf    match        m     m   1 0.6091
## 3244   moralOther   moralOther mismatch        n     n   1 0.8492
## 3245   moralOther   moralOther    match        m     m   1 0.8413
## 3246  immoralSelf  immoralSelf    match        m     n   0 0.7314
## 3247 immoralOther immoralOther mismatch        n     n   1 0.5475
## 3248  immoralSelf  immoralSelf    match        m     m   1 0.7757
## 3249  immoralSelf  immoralSelf mismatch        n     n   1 1.0640
## 3250    moralSelf    moralSelf    match        m     m   1 0.5500
## 3251  immoralSelf  immoralSelf mismatch        n     m   0 0.9702
## 3252 immoralOther immoralOther    match        m     m   1 0.6183
## 3253    moralSelf    moralSelf mismatch        n     n   1 0.8004
## 3254  immoralSelf  immoralSelf    match        m     m   1 0.7146
## 3255   moralOther   moralOther mismatch        n     n   1 0.6226
## 3256    moralSelf    moralSelf mismatch        n     n   1 0.6449
## 3257 immoralOther immoralOther mismatch        n     n   1 0.7749
## 3258    moralSelf    moralSelf    match        m     m   1 0.8390
## 3259    moralSelf    moralSelf mismatch        n     n   1 0.6653
## 3260   moralOther   moralOther    match        m     n   0 0.6634
## 3261  immoralSelf  immoralSelf mismatch        n     n   1 0.5735
## 3262 immoralOther immoralOther    match        m     m   1 0.7655
## 3263 immoralOther immoralOther mismatch        n     n   1 1.0398
## 3264   moralOther   moralOther    match        m     m   1 0.8019
## 3265   moralOther   moralOther    match        m     m   1 0.6361
## 3266   moralOther   moralOther mismatch        n  <NA>  -1 1.0850
## 3267    moralSelf    moralSelf    match        m     m   1 0.4903
## 3268  immoralSelf  immoralSelf    match        m  <NA>  -1 1.0850
## 3269  immoralSelf  immoralSelf    match        m     m   1 0.4685
## 3270 immoralOther immoralOther mismatch        n     n   1 0.6008
## 3271   moralOther   moralOther mismatch        n     m   0 0.8589
## 3272    moralSelf    moralSelf mismatch        n     n   1 0.6170
## 3273 immoralOther immoralOther mismatch        n     m   0 0.8172
## 3274   moralOther   moralOther    match        m     m   1 0.8793
## 3275  immoralSelf  immoralSelf mismatch        n     n   1 0.7855
## 3276    moralSelf    moralSelf mismatch        n     n   1 0.8235
## 3277 immoralOther immoralOther mismatch        n     n   1 0.6937
## 3278    moralSelf    moralSelf    match        m     m   1 0.7338
## 3279   moralOther   moralOther mismatch        n     n   1 0.8380
## 3280    moralSelf    moralSelf    match        m     n   0 0.7122
## 3281 immoralOther immoralOther    match        m     m   1 0.9162
## 3282    moralSelf    moralSelf mismatch        n     n   1 0.9043
## 3283  immoralSelf  immoralSelf    match        m     m   1 0.8464
## 3284   moralOther   moralOther    match        m     m   1 0.7686
## 3285 immoralOther immoralOther    match        m     m   1 0.6488
## 3286  immoralSelf  immoralSelf mismatch        n     n   1 0.8089
## 3287 immoralOther immoralOther    match        m     m   1 0.8170
## 3288  immoralSelf  immoralSelf mismatch        n     n   1 0.7611
## 3289 immoralOther immoralOther    match        m     m   1 0.7392
## 3290   moralOther   moralOther mismatch        n     n   1 0.8254
## 3291  immoralSelf  immoralSelf    match        m     m   1 0.6796
## 3292 immoralOther immoralOther mismatch        n     n   1 0.6017
## 3293   moralOther   moralOther mismatch        n     n   1 0.9239
## 3294   moralOther   moralOther    match        m     m   1 0.8620
## 3295 immoralOther immoralOther    match        m     m   1 0.7642
## 3296    moralSelf    moralSelf mismatch        n     n   1 0.8942
## 3297  immoralSelf  immoralSelf mismatch        n     n   1 0.7584
## 3298  immoralSelf  immoralSelf mismatch        n     n   1 0.8885
## 3299    moralSelf    moralSelf mismatch        n     m   0 0.5367
## 3300    moralSelf    moralSelf    match        m     m   1 0.7447
## 3301    moralSelf    moralSelf mismatch        n     n   1 0.6709
## 3302 immoralOther immoralOther mismatch        n     n   1 0.6271
## 3303    moralSelf    moralSelf    match        m     m   1 0.6091
## 3304  immoralSelf  immoralSelf    match        m     m   1 0.7853
## 3305 immoralOther immoralOther    match        m     m   1 0.7514
## 3306  immoralSelf  immoralSelf mismatch        n     n   1 0.7156
## 3307 immoralOther immoralOther mismatch        n     n   1 0.7198
## 3308   moralOther   moralOther    match        m     m   1 0.9538
## 3309  immoralSelf  immoralSelf    match        m     m   1 1.0461
## 3310   moralOther   moralOther    match        m     m   1 0.8562
## 3311   moralOther   moralOther mismatch        n     n   1 0.9924
## 3312    moralSelf    moralSelf    match        m     m   1 0.5404
## 3313    moralSelf    moralSelf    match        m     m   1 0.4978
## 3314   moralOther   moralOther mismatch        n     n   1 0.6800
## 3315  immoralSelf  immoralSelf    match        m     m   1 0.8022
## 3316   moralOther   moralOther mismatch        n     n   1 0.8042
## 3317 immoralOther immoralOther mismatch        n     n   1 0.8542
## 3318    moralSelf    moralSelf mismatch        n     n   1 0.7505
## 3319 immoralOther immoralOther    match        m     m   1 0.6186
## 3320   moralOther   moralOther    match        m     m   1 0.7288
## 3321 immoralOther immoralOther mismatch        n     n   1 0.7130
## 3322   moralOther   moralOther mismatch        n     n   1 0.7890
## 3323    moralSelf    moralSelf    match        m     m   1 0.5611
## 3324 immoralOther immoralOther    match        m     n   0 0.7872
## 3325  immoralSelf  immoralSelf mismatch        n     n   1 0.9834
## 3326  immoralSelf  immoralSelf    match        m     n   0 0.6935
## 3327    moralSelf    moralSelf mismatch        n     n   1 0.7097
## 3328   moralOther   moralOther    match        m     m   1 0.8021
## 3329   moralOther   moralOther    match        m     m   1 0.4759
## 3330  immoralSelf  immoralSelf    match        m     m   1 0.9201
## 3331  immoralSelf  immoralSelf mismatch        n     n   1 0.6022
## 3332    moralSelf    moralSelf    match        m     m   1 0.5724
## 3333  immoralSelf  immoralSelf mismatch        n     n   1 0.9404
## 3334 immoralOther immoralOther mismatch        n     n   1 0.8424
## 3335 immoralOther immoralOther    match        m     m   1 0.8367
## 3336    moralSelf    moralSelf mismatch        n     n   1 0.6409
## 3337    moralSelf    moralSelf mismatch        n     n   1 0.8350
## 3338   moralOther   moralOther    match        m     m   1 0.7193
## 3339 immoralOther immoralOther    match        m     m   1 0.8653
## 3340   moralOther   moralOther mismatch        n     n   1 0.8392
## 3341   moralOther   moralOther    match        m     n   0 0.8496
## 3342  immoralSelf  immoralSelf    match        m     n   0 0.6737
## 3343   moralOther   moralOther mismatch        n     n   1 1.0539
## 3344  immoralSelf  immoralSelf mismatch        n     n   1 0.9740
## 3345  immoralSelf  immoralSelf    match        m     m   1 0.8860
## 3346    moralSelf    moralSelf    match        m     m   1 0.4802
## 3347  immoralSelf  immoralSelf    match        m     n   0 0.6984
## 3348   moralOther   moralOther    match        m  <NA>  -1 1.0850
## 3349    moralSelf    moralSelf mismatch        n     n   1 0.8187
## 3350 immoralOther immoralOther mismatch        n     n   1 0.6888
## 3351 immoralOther immoralOther mismatch        n     n   1 0.5610
## 3352 immoralOther immoralOther mismatch        n     n   1 0.7151
## 3353    moralSelf    moralSelf    match        m     m   1 0.5272
## 3354    moralSelf    moralSelf mismatch        n     n   1 0.9673
## 3355  immoralSelf  immoralSelf mismatch        n     n   1 0.6876
## 3356  immoralSelf  immoralSelf mismatch        n     n   1 0.6876
## 3357    moralSelf    moralSelf    match        m     m   1 0.6558
## 3358   moralOther   moralOther mismatch        n     n   1 0.8559
## 3359 immoralOther immoralOther    match        m     m   1 0.6240
## 3360 immoralOther immoralOther    match        m     m   1 0.5743
## 3361  immoralSelf  immoralSelf    match        m     m   1 0.7704
## 3362 immoralOther immoralOther    match        m     m   1 0.6505
## 3363 immoralOther immoralOther    match        m     m   1 0.5547
## 3364 immoralOther immoralOther mismatch        n     n   1 0.6068
## 3365   moralOther   moralOther    match        m     m   1 0.8970
## 3366 immoralOther immoralOther mismatch        n     n   1 0.6251
## 3367  immoralSelf  immoralSelf mismatch        n     m   0 0.7273
## 3368   moralOther   moralOther mismatch        n     n   1 0.8953
## 3369   moralOther   moralOther mismatch        n     n   1 0.7995
## 3370  immoralSelf  immoralSelf mismatch        n  <NA>  -1 1.0850
## 3371    moralSelf    moralSelf mismatch        n     n   1 0.6358
## 3372  immoralSelf  immoralSelf    match        m     m   1 1.0077
## 3373 immoralOther immoralOther mismatch        n     n   1 0.5920
## 3374    moralSelf    moralSelf    match        m     m   1 0.5322
## 3375    moralSelf    moralSelf    match        m     m   1 0.7003
## 3376   moralOther   moralOther    match        m     m   1 0.6665
## 3377    moralSelf    moralSelf mismatch        n     n   1 0.5366
## 3378   moralOther   moralOther mismatch        n     n   1 0.8086
## 3379  immoralSelf  immoralSelf    match        m     m   1 0.5769
## 3380    moralSelf    moralSelf    match        m     m   1 0.8490
## 3381  immoralSelf  immoralSelf mismatch        n     n   1 0.9531
## 3382   moralOther   moralOther    match        m     m   1 0.7333
## 3383 immoralOther immoralOther    match        m     m   1 0.6774
## 3384    moralSelf    moralSelf mismatch        n     n   1 0.6396
## 3385  immoralSelf  immoralSelf    match        m     m   1 0.6896
## 3386 immoralOther immoralOther mismatch        n     n   1 0.8978
## 3387    moralSelf    moralSelf    match        m     m   1 0.6079
## 3388  immoralSelf  immoralSelf    match        m     m   1 0.8560
## 3389    moralSelf    moralSelf    match        m     m   1 0.6722
## 3390 immoralOther immoralOther mismatch        n     n   1 0.6604
## 3391  immoralSelf  immoralSelf    match        m     m   1 0.7464
## 3392    moralSelf    moralSelf mismatch        n     n   1 0.7686
## 3393   moralOther   moralOther mismatch        n     n   1 0.6887
## 3394  immoralSelf  immoralSelf mismatch        n     n   1 0.9768
## 3395   moralOther   moralOther    match        m     n   0 0.6810
## 3396   moralOther   moralOther    match        m     m   1 0.7472
## 3397   moralOther   moralOther mismatch        n     n   1 0.7673
## 3398  immoralSelf  immoralSelf mismatch        n     n   1 0.6475
## 3399    moralSelf    moralSelf    match        m     m   1 0.7835
## 3400    moralSelf    moralSelf mismatch        n     n   1 0.7096
## 3401 immoralOther immoralOther    match        m     m   1 0.6178
## 3402  immoralSelf  immoralSelf mismatch        n     n   1 0.9359
## 3403   moralOther   moralOther    match        m     m   1 0.6001
## 3404   moralOther   moralOther mismatch        n     m   0 0.6823
## 3405 immoralOther immoralOther mismatch        n     n   1 0.6363
## 3406 immoralOther immoralOther    match        m     n   0 0.6806
## 3407 immoralOther immoralOther    match        m     m   1 0.6186
## 3408    moralSelf    moralSelf mismatch        n     m   0 0.6727
## 3409    moralSelf    moralSelf    match        m     m   1 0.6289
## 3410  immoralSelf  immoralSelf    match        m     m   1 0.8409
## 3411 immoralOther immoralOther    match        m     m   1 0.6111
## 3412  immoralSelf  immoralSelf mismatch        n     n   1 0.7472
## 3413    moralSelf    moralSelf    match        m     m   1 0.5514
## 3414   moralOther   moralOther mismatch        n     n   1 0.8035
## 3415   moralOther   moralOther mismatch        n     m   0 0.8217
## 3416 immoralOther immoralOther mismatch        n     n   1 0.7398
## 3417   moralOther   moralOther    match        m     m   1 0.6019
## 3418  immoralSelf  immoralSelf mismatch        n     n   1 0.6602
## 3419 immoralOther immoralOther    match        m     m   1 0.6902
## 3420    moralSelf    moralSelf mismatch        n     n   1 0.6984
## 3421 immoralOther immoralOther    match        m     m   1 0.6484
## 3422   moralOther   moralOther mismatch        n     m   0 0.5206
## 3423  immoralSelf  immoralSelf    match        m     n   0 0.7467
## 3424    moralSelf    moralSelf mismatch        n     n   1 0.8968
## 3425  immoralSelf  immoralSelf    match        m     n   0 0.7770
## 3426   moralOther   moralOther    match        m     n   0 0.8012
## 3427 immoralOther immoralOther mismatch        n     n   1 0.6432
## 3428    moralSelf    moralSelf    match        m     n   0 0.7415
## 3429  immoralSelf  immoralSelf mismatch        n     n   1 0.7155
## 3430   moralOther   moralOther    match        m     m   1 0.7277
## 3431 immoralOther immoralOther mismatch        n     n   1 0.6398
## 3432    moralSelf    moralSelf mismatch        n     n   1 0.7059
## 3433    moralSelf    moralSelf    match        m     m   1 0.5505
## 3434   moralOther   moralOther mismatch        n     m   0 0.7386
## 3435 immoralOther immoralOther mismatch        n     n   1 0.6008
## 3436    moralSelf    moralSelf    match        m     m   1 0.6509
## 3437   moralOther   moralOther mismatch        n     m   0 0.5870
## 3438    moralSelf    moralSelf    match        m     m   1 0.6951
## 3439   moralOther   moralOther mismatch        n     n   1 0.9272
## 3440   moralOther   moralOther    match        m     m   1 0.7994
## 3441  immoralSelf  immoralSelf    match        m     m   1 0.8335
## 3442  immoralSelf  immoralSelf mismatch        n     n   1 1.0457
## 3443 immoralOther immoralOther mismatch        n     n   1 0.6478
## 3444  immoralSelf  immoralSelf mismatch        n     m   0 0.8320
## 3445    moralSelf    moralSelf mismatch        n     n   1 0.6921
## 3446   moralOther   moralOther    match        m     m   1 0.8202
## 3447  immoralSelf  immoralSelf mismatch        n     n   1 0.9784
## 3448    moralSelf    moralSelf mismatch        n     n   1 0.7768
## 3449  immoralSelf  immoralSelf    match        m     m   1 0.6487
## 3450 immoralOther immoralOther    match        m     n   0 0.9048
## 3451    moralSelf    moralSelf mismatch        n     n   1 0.6310
## 3452 immoralOther immoralOther mismatch        n     m   0 0.5872
## 3453  immoralSelf  immoralSelf    match        m     m   1 0.8152
## 3454 immoralOther immoralOther    match        m     m   1 0.5271
## 3455   moralOther   moralOther    match        m     m   1 0.7514
## 3456 immoralOther immoralOther    match        m     m   1 0.6836
## 3457 immoralOther immoralOther    match        m     m   1 0.5497
## 3458   moralOther   moralOther    match        m     m   1 0.4979
## 3459   moralOther   moralOther mismatch        n     m   0 0.5840
## 3460 immoralOther immoralOther mismatch        n     n   1 0.7901
## 3461 immoralOther immoralOther mismatch        n     n   1 0.9462
## 3462   moralOther   moralOther mismatch        n     m   0 0.8806
## 3463   moralOther   moralOther mismatch        n     m   0 0.7225
## 3464    moralSelf    moralSelf mismatch        n     n   1 0.7407
## 3465   moralOther   moralOther    match        m     m   1 0.7147
## 3466  immoralSelf  immoralSelf    match        m     n   0 0.6149
## 3467    moralSelf    moralSelf mismatch        n     n   1 0.5751
## 3468 immoralOther immoralOther    match        m     m   1 0.7914
## 3469  immoralSelf  immoralSelf mismatch        n     n   1 0.9233
## 3470 immoralOther immoralOther mismatch        n     n   1 0.5894
## 3471   moralOther   moralOther    match        m     m   1 0.8015
## 3472    moralSelf    moralSelf mismatch        n     n   1 0.6598
## 3473    moralSelf    moralSelf    match        m     m   1 0.6018
## 3474 immoralOther immoralOther    match        m     m   1 0.6920
## 3475  immoralSelf  immoralSelf mismatch        n     n   1 0.7000
## 3476  immoralSelf  immoralSelf    match        m     n   0 0.6587
## 3477  immoralSelf  immoralSelf    match        m     m   1 0.6724
## 3478    moralSelf    moralSelf    match        m     m   1 0.6526
## 3479  immoralSelf  immoralSelf mismatch        n     n   1 0.8206
## 3480    moralSelf    moralSelf    match        m     m   1 0.7788
## 3481  immoralSelf  immoralSelf    match        m     n   0 0.7584
## 3482    moralSelf    moralSelf mismatch        n     n   1 0.8145
## 3483    moralSelf    moralSelf    match        m     m   1 0.5806
## 3484 immoralOther immoralOther mismatch        n     n   1 0.8187
## 3485   moralOther   moralOther    match        m     m   1 0.8168
## 3486  immoralSelf  immoralSelf mismatch        n     n   1 0.9531
## 3487  immoralSelf  immoralSelf mismatch        n     n   1 0.7251
## 3488    moralSelf    moralSelf    match        m     m   1 0.6553
## 3489 immoralOther immoralOther    match        m     m   1 1.0634
## 3490   moralOther   moralOther mismatch        n     n   1 0.7575
## 3491   moralOther   moralOther    match        m     m   1 0.6557
## 3492    moralSelf    moralSelf mismatch        n     n   1 0.7279
## 3493  immoralSelf  immoralSelf mismatch        n     n   1 0.7440
## 3494 immoralOther immoralOther    match        m     m   1 0.6941
## 3495  immoralSelf  immoralSelf    match        m     m   1 0.7742
## 3496    moralSelf    moralSelf mismatch        n     n   1 0.8624
## 3497  immoralSelf  immoralSelf    match        m     m   1 0.7325
## 3498   moralOther   moralOther    match        m     m   1 0.7647
## 3499 immoralOther immoralOther    match        m     m   1 0.7429
## 3500    moralSelf    moralSelf    match        m     m   1 0.7170
## 3501   moralOther   moralOther mismatch        n     m   0 0.6570
## 3502 immoralOther immoralOther mismatch        n     n   1 0.7111
## 3503   moralOther   moralOther mismatch        n     n   1 0.7713
## 3504 immoralOther immoralOther mismatch        n     n   1 0.8354
## 3505 immoralOther immoralOther mismatch        n     n   1 0.6639
## 3506   moralOther   moralOther    match        m     m   1 0.7017
## 3507 immoralOther immoralOther    match        m     m   1 0.6518
## 3508    moralSelf    moralSelf    match        m     m   1 0.6120
## 3509  immoralSelf  immoralSelf    match        m     m   1 0.6921
## 3510   moralOther   moralOther    match        m     m   1 0.7242
## 3511   moralOther   moralOther    match        m     m   1 0.7103
## 3512    moralSelf    moralSelf    match        m     m   1 0.7385
## 3513  immoralSelf  immoralSelf mismatch        n     m   0 0.6327
## 3514    moralSelf    moralSelf mismatch        n     m   0 0.7648
## 3515    moralSelf    moralSelf mismatch        n     n   1 1.0069
## 3516  immoralSelf  immoralSelf mismatch        n     n   1 0.9531
## 3517   moralOther   moralOther mismatch        n     n   1 0.8372
## 3518    moralSelf    moralSelf mismatch        n     n   1 0.6473
## 3519 immoralOther immoralOther mismatch        n     n   1 0.8394
## 3520   moralOther   moralOther mismatch        n     n   1 0.6336
## 3521    moralSelf    moralSelf    match        m     m   1 0.6697
## 3522  immoralSelf  immoralSelf    match        m     m   1 0.8658
## 3523 immoralOther immoralOther    match        m     m   1 0.9680
## 3524   moralOther   moralOther mismatch        n     n   1 0.8882
## 3525 immoralOther immoralOther mismatch        n     n   1 0.6804
## 3526 immoralOther immoralOther    match        m     m   1 0.7864
## 3527  immoralSelf  immoralSelf    match        m     m   1 0.8886
## 3528  immoralSelf  immoralSelf mismatch        n  <NA>  -1 1.0850
## 3529  immoralSelf  immoralSelf    match        m     m   1 1.0246
## 3530   moralOther   moralOther mismatch        n     n   1 0.9347
## 3531   moralOther   moralOther mismatch        n     n   1 0.6229
## 3532  immoralSelf  immoralSelf    match        m     m   1 0.7330
## 3533   moralOther   moralOther    match        m     m   1 1.0371
## 3534    moralSelf    moralSelf mismatch        n     n   1 0.8794
## 3535  immoralSelf  immoralSelf    match        m     m   1 0.6895
## 3536    moralSelf    moralSelf mismatch        n     n   1 0.6095
## 3537    moralSelf    moralSelf mismatch        n     n   1 0.6657
## 3538 immoralOther immoralOther    match        m     m   1 0.6798
## 3539 immoralOther immoralOther mismatch        n     n   1 0.7219
## 3540 immoralOther immoralOther mismatch        n     n   1 0.5801
## 3541    moralSelf    moralSelf    match        m     m   1 0.5303
## 3542  immoralSelf  immoralSelf mismatch        n     n   1 0.8023
## 3543    moralSelf    moralSelf    match        m     n   0 0.6524
## 3544 immoralOther immoralOther    match        m     m   1 0.8846
## 3545   moralOther   moralOther    match        m     m   1 0.7267
## 3546    moralSelf    moralSelf    match        m     m   1 0.6009
## 3547   moralOther   moralOther    match        m     m   1 0.6590
## 3548 immoralOther immoralOther mismatch        n     n   1 0.8091
## 3549 immoralOther immoralOther    match        m     m   1 0.7132
## 3550   moralOther   moralOther mismatch        n     n   1 0.6134
## 3551  immoralSelf  immoralSelf mismatch        n     n   1 0.6776
## 3552  immoralSelf  immoralSelf mismatch        n     n   1 0.7196
## 3553  immoralSelf  immoralSelf    match        m     m   1 0.8738
## 3554 immoralOther immoralOther mismatch        n     n   1 0.6699
## 3555 immoralOther immoralOther mismatch        n     n   1 0.6501
## 3556   moralOther   moralOther    match        m     n   0 0.9222
## 3557  immoralSelf  immoralSelf mismatch        n     n   1 0.8043
## 3558  immoralSelf  immoralSelf mismatch        n     n   1 0.6865
## 3559 immoralOther immoralOther    match        m     m   1 0.8886
## 3560    moralSelf    moralSelf    match        m     m   1 0.7527
## 3561  immoralSelf  immoralSelf    match        m     m   1 0.7409
## 3562  immoralSelf  immoralSelf    match        m     m   1 0.7231
## 3563    moralSelf    moralSelf mismatch        n     n   1 0.6292
## 3564    moralSelf    moralSelf    match        m     m   1 0.5774
## 3565    moralSelf    moralSelf mismatch        n     n   1 0.9513
## 3566    moralSelf    moralSelf mismatch        n     n   1 0.5796
## 3567  immoralSelf  immoralSelf mismatch        n     n   1 0.6017
## 3568   moralOther   moralOther mismatch        n     n   1 0.6198
## 3569   moralOther   moralOther mismatch        n     n   1 0.6020
## 3570 immoralOther immoralOther mismatch        n     n   1 0.6439
## 3571   moralOther   moralOther mismatch        n     n   1 0.6462
## 3572 immoralOther immoralOther    match        m     m   1 0.8084
## 3573   moralOther   moralOther    match        m     m   1 0.7685
## 3574 immoralOther immoralOther    match        m     m   1 0.7046
## 3575    moralSelf    moralSelf    match        m     m   1 0.5747
## 3576   moralOther   moralOther    match        m     m   1 0.5748
## 3577   moralOther   moralOther    match        m     m   1 0.7651
## 3578    moralSelf    moralSelf mismatch        n     n   1 0.6552
## 3579   moralOther   moralOther mismatch        n     n   1 0.7514
## 3580   moralOther   moralOther mismatch        n     n   1 0.9154
## 3581 immoralOther immoralOther mismatch        n     n   1 0.6316
## 3582   moralOther   moralOther mismatch        n     n   1 0.8756
## 3583 immoralOther immoralOther    match        m     m   1 0.6718
## 3584 immoralOther immoralOther mismatch        n     n   1 0.6680
## 3585  immoralSelf  immoralSelf mismatch        n     n   1 0.6562
## 3586 immoralOther immoralOther    match        m     m   1 0.6883
## 3587  immoralSelf  immoralSelf    match        m     m   1 0.7604
## 3588    moralSelf    moralSelf mismatch        n     m   0 0.8105
## 3589 immoralOther immoralOther    match        m     m   1 0.8666
## 3590    moralSelf    moralSelf mismatch        n     n   1 0.7209
## 3591  immoralSelf  immoralSelf    match        m  <NA>  -1 1.0850
## 3592 immoralOther immoralOther mismatch        n     n   1 0.7651
## 3593  immoralSelf  immoralSelf    match        m     m   1 0.8312
## 3594  immoralSelf  immoralSelf mismatch        n     n   1 0.6994
## 3595  immoralSelf  immoralSelf mismatch        n     n   1 1.0336
## 3596   moralOther   moralOther    match        m     m   1 0.7736
## 3597    moralSelf    moralSelf    match        m     m   1 0.7738
## 3598    moralSelf    moralSelf    match        m     m   1 0.6779
## 3599    moralSelf    moralSelf    match        m     m   1 0.6080
## 3600   moralOther   moralOther    match        m     m   1 0.6161
## 3601  immoralSelf  immoralSelf mismatch        n     n   1 0.5764
## 3602  immoralSelf  immoralSelf    match        m     n   0 0.8964
## 3603   moralOther   moralOther mismatch        n     n   1 0.5925
## 3604 immoralOther immoralOther    match        m     m   1 0.5968
## 3605 immoralOther immoralOther mismatch        n     n   1 0.6728
## 3606 immoralOther immoralOther    match        m     m   1 0.8450
## 3607   moralOther   moralOther    match        m     m   1 0.5591
## 3608 immoralOther immoralOther mismatch        n     n   1 0.7452
## 3609   moralOther   moralOther    match        m     m   1 0.6394
## 3610  immoralSelf  immoralSelf    match        m     m   1 0.5936
## 3611    moralSelf    moralSelf mismatch        n     n   1 0.6537
## 3612  immoralSelf  immoralSelf mismatch        n     m   0 0.6537
## 3613  immoralSelf  immoralSelf    match        m     m   1 0.7899
## 3614    moralSelf    moralSelf mismatch        n     n   1 0.5060
## 3615   moralOther   moralOther    match        m     m   1 0.5921
## 3616    moralSelf    moralSelf    match        m     m   1 0.5483
## 3617   moralOther   moralOther mismatch        n     n   1 0.5925
## 3618    moralSelf    moralSelf    match        m     m   1 0.4845
## 3619    moralSelf    moralSelf mismatch        n     n   1 0.7106
## 3620   moralOther   moralOther mismatch        n     n   1 0.8668
## 3621  immoralSelf  immoralSelf mismatch        n     n   1 0.6649
## 3622 immoralOther immoralOther mismatch        n     n   1 0.6470
## 3623    moralSelf    moralSelf    match        m     m   1 0.5452
## 3624 immoralOther immoralOther    match        m     m   1 0.6553
## 3625 immoralOther immoralOther mismatch        n     n   1 0.7461
## 3626 immoralOther immoralOther    match        m     m   1 0.6562
## 3627   moralOther   moralOther    match        m     m   1 0.8004
## 3628   moralOther   moralOther    match        m     m   1 0.6985
## 3629   moralOther   moralOther    match        m     m   1 0.5208
## 3630  immoralSelf  immoralSelf mismatch        n     n   1 0.5068
## 3631    moralSelf    moralSelf    match        m     m   1 0.4969
## 3632 immoralOther immoralOther    match        m     m   1 0.6570
## 3633   moralOther   moralOther mismatch        n     n   1 0.6312
## 3634   moralOther   moralOther mismatch        n     n   1 0.5473
## 3635 immoralOther immoralOther mismatch        n     n   1 0.6234
## 3636  immoralSelf  immoralSelf mismatch        n     n   1 0.6856
## 3637    moralSelf    moralSelf mismatch        n     n   1 0.6237
## 3638    moralSelf    moralSelf    match        m     m   1 0.5898
## 3639    moralSelf    moralSelf mismatch        n     n   1 0.7060
## 3640 immoralOther immoralOther    match        m     m   1 0.7041
## 3641  immoralSelf  immoralSelf    match        m  <NA>  -1 1.0850
## 3642  immoralSelf  immoralSelf    match        m     m   1 0.5924
## 3643   moralOther   moralOther mismatch        n     n   1 0.7245
## 3644  immoralSelf  immoralSelf mismatch        n     n   1 0.8867
## 3645  immoralSelf  immoralSelf    match        m     m   1 0.7127
## 3646    moralSelf    moralSelf mismatch        n     m   0 0.6129
## 3647    moralSelf    moralSelf    match        m     m   1 0.5490
## 3648 immoralOther immoralOther mismatch        n     n   1 0.7212
## 3649 immoralOther immoralOther mismatch        n     n   1 0.5872
## 3650    moralSelf    moralSelf mismatch        n     n   1 0.6634
## 3651  immoralSelf  immoralSelf mismatch        n     n   1 0.7015
## 3652   moralOther   moralOther    match        m     n   0 1.0037
## 3653  immoralSelf  immoralSelf    match        m     m   1 0.7498
## 3654    moralSelf    moralSelf    match        m     m   1 0.6260
## 3655    moralSelf    moralSelf mismatch        n     n   1 0.6240
## 3656 immoralOther immoralOther mismatch        n     n   1 0.5742
## 3657  immoralSelf  immoralSelf    match        m     m   1 0.5984
## 3658    moralSelf    moralSelf mismatch        n     n   1 0.6085
## 3659 immoralOther immoralOther mismatch        n     n   1 0.5525
## 3660    moralSelf    moralSelf    match        m     m   1 0.6447
## 3661   moralOther   moralOther mismatch        n     n   1 0.8309
## 3662 immoralOther immoralOther    match        m     m   1 0.7470
## 3663   moralOther   moralOther mismatch        n     n   1 0.6171
## 3664 immoralOther immoralOther    match        m     m   1 0.6712
## 3665   moralOther   moralOther    match        m     m   1 0.6114
## 3666  immoralSelf  immoralSelf mismatch        n     n   1 0.6516
## 3667  immoralSelf  immoralSelf    match        m     m   1 0.6596
## 3668   moralOther   moralOther mismatch        n     n   1 0.5378
## 3669  immoralSelf  immoralSelf mismatch        n     n   1 0.6482
## 3670 immoralOther immoralOther    match        m     n   0 0.6000
## 3671   moralOther   moralOther    match        m     n   0 0.6021
## 3672    moralSelf    moralSelf    match        m     n   0 0.5323
## 3673 immoralOther immoralOther mismatch        n     n   1 0.8601
## 3674 immoralOther immoralOther mismatch        n     n   1 0.7540
## 3675  immoralSelf  immoralSelf    match        m     m   1 0.7662
## 3676 immoralOther immoralOther    match        m     m   1 0.7523
## 3677   moralOther   moralOther mismatch        n     n   1 0.7884
## 3678  immoralSelf  immoralSelf    match        m     m   1 0.5771
## 3679    moralSelf    moralSelf    match        m     m   1 0.5991
## 3680   moralOther   moralOther    match        m     m   1 0.6252
## 3681  immoralSelf  immoralSelf mismatch        n     n   1 0.5597
## 3682   moralOther   moralOther    match        m     m   1 0.6656
## 3683  immoralSelf  immoralSelf mismatch        n     n   1 0.9835
## 3684   moralOther   moralOther mismatch        n     n   1 0.6938
## 3685    moralSelf    moralSelf mismatch        n     n   1 0.6936
## 3686   moralOther   moralOther mismatch        n     n   1 0.5741
## 3687    moralSelf    moralSelf    match        m     m   1 0.5505
## 3688  immoralSelf  immoralSelf mismatch        n     n   1 0.6265
## 3689  immoralSelf  immoralSelf    match        m     n   0 0.7978
## 3690    moralSelf    moralSelf mismatch        n     m   0 0.6725
## 3691 immoralOther immoralOther mismatch        n     n   1 0.6121
## 3692    moralSelf    moralSelf    match        m     m   1 0.7646
## 3693 immoralOther immoralOther    match        m     m   1 0.6392
## 3694 immoralOther immoralOther    match        m     m   1 0.5931
## 3695    moralSelf    moralSelf mismatch        n     n   1 0.5487
## 3696   moralOther   moralOther    match        m     m   1 0.6414
## 3697 immoralOther immoralOther    match        m     m   1 0.5712
## 3698   moralOther   moralOther    match        m     m   1 0.6116
## 3699 immoralOther immoralOther    match        m     m   1 0.6194
## 3700    moralSelf    moralSelf    match        m     m   1 0.5941
## 3701    moralSelf    moralSelf    match        m     m   1 0.5663
## 3702  immoralSelf  immoralSelf mismatch        n     n   1 0.7101
## 3703  immoralSelf  immoralSelf mismatch        n     n   1 0.6022
## 3704 immoralOther immoralOther mismatch        n     n   1 0.5970
## 3705    moralSelf    moralSelf mismatch        n     n   1 0.5184
## 3706 immoralOther immoralOther    match        m     m   1 0.5921
## 3707   moralOther   moralOther    match        m     m   1 0.7079
## 3708   moralOther   moralOther mismatch        n     n   1 0.6333
## 3709    moralSelf    moralSelf mismatch        n     n   1 0.6210
## 3710  immoralSelf  immoralSelf mismatch        n     n   1 0.6672
## 3711   moralOther   moralOther mismatch        n     n   1 0.7994
## 3712   moralOther   moralOther mismatch        n     n   1 0.6091
## 3713 immoralOther immoralOther mismatch        n     n   1 0.5852
## 3714  immoralSelf  immoralSelf    match        m     n   0 0.6479
## 3715    moralSelf    moralSelf mismatch        n     n   1 0.6239
## 3716  immoralSelf  immoralSelf    match        m     m   1 0.6933
## 3717   moralOther   moralOther    match        m     m   1 0.7336
## 3718  immoralSelf  immoralSelf    match        m     m   1 0.6300
## 3719    moralSelf    moralSelf    match        m     m   1 0.5058
## 3720 immoralOther immoralOther mismatch        n     n   1 0.5693
## 3721  immoralSelf  immoralSelf    match        m     m   1 0.7271
## 3722  immoralSelf  immoralSelf mismatch        n     n   1 0.8873
## 3723   moralOther   moralOther    match        m     n   0 0.9274
## 3724    moralSelf    moralSelf mismatch        n     n   1 0.6716
## 3725 immoralOther immoralOther mismatch        n     n   1 0.6837
## 3726 immoralOther immoralOther mismatch        n     n   1 0.8219
## 3727   moralOther   moralOther mismatch        n     n   1 0.8520
## 3728 immoralOther immoralOther    match        m     m   1 0.8922
## 3729  immoralSelf  immoralSelf mismatch        n     n   1 0.7803
## 3730 immoralOther immoralOther    match        m     n   0 0.9304
## 3731   moralOther   moralOther mismatch        n     n   1 0.8585
## 3732   moralOther   moralOther    match        m     n   0 0.9607
## 3733  immoralSelf  immoralSelf    match        m     m   1 0.8748
## 3734    moralSelf    moralSelf mismatch        n     n   1 0.5670
## 3735  immoralSelf  immoralSelf mismatch        n     n   1 0.8651
## 3736 immoralOther immoralOther    match        m     m   1 0.7833
## 3737    moralSelf    moralSelf mismatch        n     n   1 0.7254
## 3738    moralSelf    moralSelf    match        m     n   0 0.9115
## 3739   moralOther   moralOther mismatch        n     n   1 0.7076
## 3740    moralSelf    moralSelf    match        m     m   1 0.7518
## 3741    moralSelf    moralSelf    match        m     m   1 0.7559
## 3742   moralOther   moralOther    match        m     m   1 0.8560
## 3743 immoralOther immoralOther mismatch        n     n   1 0.6642
## 3744  immoralSelf  immoralSelf    match        m     m   1 0.8223
## 3745 immoralOther immoralOther    match        m     m   1 0.6844
## 3746   moralOther   moralOther mismatch        n     m   0 0.6706
## 3747  immoralSelf  immoralSelf    match        m     m   1 0.8747
## 3748    moralSelf    moralSelf    match        m     m   1 0.6388
## 3749   moralOther   moralOther mismatch        n     m   0 0.6090
## 3750    moralSelf    moralSelf    match        m     m   1 0.6651
## 3751 immoralOther immoralOther mismatch        n     m   0 0.6152
## 3752  immoralSelf  immoralSelf mismatch        n     m   0 0.7235
## 3753   moralOther   moralOther mismatch        n     m   0 0.8855
## 3754  immoralSelf  immoralSelf    match        m     m   1 0.7597
## 3755    moralSelf    moralSelf    match        m     m   1 0.5618
## 3756  immoralSelf  immoralSelf    match        m     m   1 0.6358
## 3757    moralSelf    moralSelf mismatch        n     n   1 0.7361
## 3758   moralOther   moralOther    match        m     m   1 0.6902
## 3759 immoralOther immoralOther mismatch        n     n   1 0.7863
## 3760    moralSelf    moralSelf mismatch        n     m   0 0.7284
## 3761 immoralOther immoralOther    match        m     m   1 0.7926
## 3762   moralOther   moralOther    match        m     m   1 0.7488
## 3763   moralOther   moralOther    match        m     m   1 0.6329
## 3764 immoralOther immoralOther mismatch        n     m   0 0.6450
## 3765  immoralSelf  immoralSelf mismatch        n     m   0 0.6872
## 3766 immoralOther immoralOther    match        m     m   1 0.6633
## 3767  immoralSelf  immoralSelf mismatch        n     n   1 0.8214
## 3768    moralSelf    moralSelf mismatch        n     n   1 0.7555
## 3769   moralOther   moralOther    match        m     m   1 0.8097
## 3770   moralOther   moralOther mismatch        n     n   1 0.7298
## 3771    moralSelf    moralSelf mismatch        n     m   0 0.5859
## 3772  immoralSelf  immoralSelf    match        m     m   1 0.8081
## 3773   moralOther   moralOther mismatch        n     n   1 0.8642
## 3774  immoralSelf  immoralSelf mismatch        n     m   0 0.7104
## 3775    moralSelf    moralSelf mismatch        n     m   0 0.6505
## 3776 immoralOther immoralOther    match        m     m   1 0.7866
## 3777 immoralOther immoralOther    match        m     m   1 0.7048
## 3778 immoralOther immoralOther mismatch        n     n   1 0.7429
## 3779    moralSelf    moralSelf    match        m     m   1 0.6051
## 3780    moralSelf    moralSelf    match        m     m   1 0.6471
## 3781 immoralOther immoralOther mismatch        n     n   1 0.7612
## 3782 immoralOther immoralOther mismatch        n     n   1 0.8994
## 3783    moralSelf    moralSelf    match        m     m   1 0.6256
## 3784    moralSelf    moralSelf mismatch        n     n   1 0.6717
## 3785  immoralSelf  immoralSelf mismatch        n     n   1 0.8758
## 3786 immoralOther immoralOther    match        m     m   1 0.6799
## 3787  immoralSelf  immoralSelf    match        m     m   1 0.8421
## 3788  immoralSelf  immoralSelf mismatch        n     n   1 0.8122
## 3789   moralOther   moralOther mismatch        n     n   1 0.9483
## 3790  immoralSelf  immoralSelf    match        m     m   1 0.6645
## 3791   moralOther   moralOther    match        m     n   0 0.6866
## 3792   moralOther   moralOther    match        m     m   1 0.6808
## 3793    moralSelf    moralSelf mismatch        n     n   1 0.7420
## 3794 immoralOther immoralOther    match        m     m   1 0.6682
## 3795    moralSelf    moralSelf    match        m     m   1 0.5523
## 3796 immoralOther immoralOther    match        m     n   0 0.8365
## 3797 immoralOther immoralOther mismatch        n     n   1 0.6245
## 3798   moralOther   moralOther    match        m     n   0 0.7586
## 3799 immoralOther immoralOther mismatch        n     n   1 0.7688
## 3800  immoralSelf  immoralSelf mismatch        n     n   1 0.8089
## 3801  immoralSelf  immoralSelf    match        m     m   1 0.6811
## 3802  immoralSelf  immoralSelf    match        m     m   1 0.6912
## 3803    moralSelf    moralSelf    match        m     m   1 0.5793
## 3804  immoralSelf  immoralSelf mismatch        n     m   0 0.8334
## 3805  immoralSelf  immoralSelf    match        m     m   1 1.0776
## 3806   moralOther   moralOther    match        m     m   1 0.8877
## 3807    moralSelf    moralSelf mismatch        n     n   1 0.7279
## 3808  immoralSelf  immoralSelf mismatch        n     n   1 0.9120
## 3809   moralOther   moralOther mismatch        n     m   0 0.6521
## 3810   moralOther   moralOther    match        m     m   1 0.6923
## 3811    moralSelf    moralSelf    match        m     m   1 0.6844
## 3812   moralOther   moralOther mismatch        n     m   0 0.9345
## 3813 immoralOther immoralOther    match        m     m   1 0.7107
## 3814 immoralOther immoralOther mismatch        n     m   0 0.8108
## 3815   moralOther   moralOther mismatch        n     n   1 0.9389
## 3816    moralSelf    moralSelf mismatch        n     n   1 0.7050
## 3817    moralSelf    moralSelf mismatch        n     m   0 0.7994
## 3818   moralOther   moralOther mismatch        n  <NA>  -1 1.0850
## 3819   moralOther   moralOther    match        m     n   0 0.7556
## 3820  immoralSelf  immoralSelf mismatch        n     n   1 0.8397
## 3821   moralOther   moralOther    match        m     m   1 0.8178
## 3822  immoralSelf  immoralSelf    match        m     m   1 0.7040
## 3823 immoralOther immoralOther    match        m     m   1 0.5581
## 3824 immoralOther immoralOther    match        m     m   1 0.8963
## 3825  immoralSelf  immoralSelf mismatch        n     n   1 0.8243
## 3826  immoralSelf  immoralSelf    match        m     m   1 0.9425
## 3827    moralSelf    moralSelf    match        m     m   1 0.6367
## 3828    moralSelf    moralSelf mismatch        n     m   0 0.7207
## 3829   moralOther   moralOther    match        m     n   0 0.9949
## 3830   moralOther   moralOther mismatch        n  <NA>  -1 1.0850
## 3831  immoralSelf  immoralSelf mismatch        n     n   1 0.7292
## 3832 immoralOther immoralOther mismatch        n     n   1 0.7853
## 3833  immoralSelf  immoralSelf    match        m     m   1 0.7594
## 3834   moralOther   moralOther mismatch        n     n   1 0.7216
## 3835    moralSelf    moralSelf mismatch        n     n   1 0.7637
## 3836 immoralOther immoralOther mismatch        n     n   1 0.6458
## 3837 immoralOther immoralOther    match        m     m   1 0.7280
## 3838    moralSelf    moralSelf    match        m     m   1 0.6962
## 3839 immoralOther immoralOther mismatch        n     n   1 0.6082
## 3840    moralSelf    moralSelf    match        m     n   0 0.6884
## 3841    moralSelf    moralSelf    match        m     m   1 0.7547
## 3842 immoralOther immoralOther    match        m     m   1 0.6069
## 3843  immoralSelf  immoralSelf    match        m     m   1 0.6970
## 3844  immoralSelf  immoralSelf mismatch        n     m   0 0.8251
## 3845   moralOther   moralOther    match        m     m   1 0.5753
## 3846   moralOther   moralOther mismatch        n     n   1 0.6473
## 3847  immoralSelf  immoralSelf mismatch        n     m   0 0.5274
## 3848 immoralOther immoralOther mismatch        n     n   1 0.6556
## 3849 immoralOther immoralOther    match        m     m   1 0.7118
## 3850  immoralSelf  immoralSelf    match        m     m   1 0.7158
## 3851  immoralSelf  immoralSelf mismatch        n     m   0 0.9361
## 3852 immoralOther immoralOther mismatch        n     n   1 0.7122
## 3853   moralOther   moralOther    match        m     m   1 0.4923
## 3854  immoralSelf  immoralSelf    match        m     m   1 0.8283
## 3855    moralSelf    moralSelf mismatch        n     m   0 0.5386
## 3856   moralOther   moralOther    match        m     m   1 0.5927
## 3857   moralOther   moralOther mismatch        n     m   0 0.8528
## 3858    moralSelf    moralSelf mismatch        n     m   0 0.7409
## 3859   moralOther   moralOther mismatch        n     m   0 0.5790
## 3860 immoralOther immoralOther    match        m     m   1 0.7212
## 3861    moralSelf    moralSelf    match        m     m   1 0.5713
## 3862    moralSelf    moralSelf    match        m     m   1 0.5955
## 3863    moralSelf    moralSelf mismatch        n     n   1 0.5916
## 3864 immoralOther immoralOther mismatch        n     m   0 0.8117
## 3865   moralOther   moralOther    match        m     n   0 0.8198
## 3866  immoralSelf  immoralSelf    match        m     m   1 0.5779
## 3867  immoralSelf  immoralSelf mismatch        n     m   0 0.7201
## 3868    moralSelf    moralSelf    match        m     m   1 0.5382
## 3869  immoralSelf  immoralSelf    match        m     m   1 0.6403
## 3870  immoralSelf  immoralSelf    match        m     m   1 0.8185
## 3871 immoralOther immoralOther    match        m     m   1 0.6406
## 3872   moralOther   moralOther mismatch        n     n   1 0.8587
## 3873   moralOther   moralOther    match        m     m   1 0.6489
## 3874 immoralOther immoralOther mismatch        n     n   1 0.6650
## 3875  immoralSelf  immoralSelf mismatch        n     n   1 0.7672
## 3876    moralSelf    moralSelf mismatch        n     n   1 0.7112
## 3877 immoralOther immoralOther    match        m     m   1 0.6674
## 3878   moralOther   moralOther mismatch        n     n   1 0.8155
## 3879    moralSelf    moralSelf mismatch        n     n   1 0.7917
## 3880 immoralOther immoralOther    match        m     m   1 0.7878
## 3881   moralOther   moralOther mismatch        n     n   1 0.9439
## 3882    moralSelf    moralSelf mismatch        n     n   1 0.9261
## 3883  immoralSelf  immoralSelf mismatch        n     m   0 0.7683
## 3884    moralSelf    moralSelf    match        m     m   1 0.6644
## 3885 immoralOther immoralOther mismatch        n     m   0 0.6466
## 3886    moralSelf    moralSelf    match        m     m   1 0.5847
## 3887   moralOther   moralOther    match        m     m   1 0.5668
## 3888 immoralOther immoralOther mismatch        n     n   1 0.6729
## 3889 immoralOther immoralOther    match        m     m   1 0.7330
## 3890   moralOther   moralOther mismatch        n     m   0 0.7491
## 3891    moralSelf    moralSelf    match        m     m   1 0.5693
## 3892   moralOther   moralOther mismatch        n     n   1 0.7934
## 3893   moralOther   moralOther    match        m     m   1 0.6455
## 3894  immoralSelf  immoralSelf    match        m     n   0 0.6957
## 3895 immoralOther immoralOther mismatch        n     m   0 0.6079
## 3896  immoralSelf  immoralSelf    match        m     m   1 0.8640
## 3897  immoralSelf  immoralSelf mismatch        n     m   0 1.0621
## 3898    moralSelf    moralSelf    match        m     m   1 0.7561
## 3899  immoralSelf  immoralSelf mismatch        n     m   0 0.6803
## 3900 immoralOther immoralOther    match        m     m   1 0.6105
## 3901    moralSelf    moralSelf mismatch        n     n   1 0.7946
## 3902  immoralSelf  immoralSelf    match        m     m   1 0.8467
## 3903   moralOther   moralOther mismatch        n     m   0 0.6569
## 3904    moralSelf    moralSelf mismatch        n     n   1 0.7450
## 3905 immoralOther immoralOther mismatch        n     n   1 0.6631
## 3906    moralSelf    moralSelf    match        m     m   1 0.7652
## 3907    moralSelf    moralSelf mismatch        n     n   1 0.7754
## 3908   moralOther   moralOther    match        m     m   1 0.6555
## 3909  immoralSelf  immoralSelf mismatch        n     n   1 0.9357
## 3910 immoralOther immoralOther    match        m     m   1 0.5759
## 3911 immoralOther immoralOther mismatch        n     n   1 0.6399
## 3912   moralOther   moralOther    match        m     m   1 0.6981
## 3913   moralOther   moralOther    match        m     m   1 0.7110
## 3914   moralOther   moralOther mismatch        n     n   1 0.6111
## 3915    moralSelf    moralSelf    match        m     m   1 0.5473
## 3916  immoralSelf  immoralSelf    match        m     m   1 0.7034
## 3917  immoralSelf  immoralSelf    match        m     n   0 0.7175
## 3918 immoralOther immoralOther mismatch        n     n   1 0.7758
## 3919   moralOther   moralOther mismatch        n     n   1 0.6798
## 3920    moralSelf    moralSelf mismatch        n     n   1 0.9219
## 3921 immoralOther immoralOther mismatch        n     n   1 0.5481
## 3922   moralOther   moralOther    match        m     n   0 0.6802
## 3923  immoralSelf  immoralSelf mismatch        n     n   1 0.8903
## 3924    moralSelf    moralSelf mismatch        n     n   1 0.6345
## 3925 immoralOther immoralOther mismatch        n     n   1 0.6626
## 3926    moralSelf    moralSelf    match        m     m   1 0.6127
## 3927   moralOther   moralOther mismatch        n     n   1 0.9089
## 3928    moralSelf    moralSelf    match        m     m   1 0.6810
## 3929 immoralOther immoralOther    match        m     m   1 0.7231
## 3930    moralSelf    moralSelf mismatch        n     n   1 0.7092
## 3931  immoralSelf  immoralSelf    match        m     m   1 0.8414
## 3932   moralOther   moralOther    match        m     m   1 0.7315
## 3933 immoralOther immoralOther    match        m     m   1 0.6197
## 3934  immoralSelf  immoralSelf mismatch        n     n   1 0.7698
## 3935 immoralOther immoralOther    match        m     m   1 0.5779
## 3936  immoralSelf  immoralSelf mismatch        n     m   0 0.5681
## 3937 immoralOther immoralOther    match        m     m   1 0.6482
## 3938   moralOther   moralOther mismatch        n     n   1 0.6003
## 3939  immoralSelf  immoralSelf    match        m     m   1 0.5064
## 3940 immoralOther immoralOther mismatch        n     m   0 0.6885
## 3941   moralOther   moralOther mismatch        n     m   0 0.5927
## 3942   moralOther   moralOther    match        m     m   1 0.6049
## 3943 immoralOther immoralOther    match        m     m   1 0.6790
## 3944    moralSelf    moralSelf mismatch        n     m   0 0.7451
## 3945  immoralSelf  immoralSelf mismatch        n     m   0 0.9032
## 3946  immoralSelf  immoralSelf mismatch        n     m   0 0.6853
## 3947    moralSelf    moralSelf mismatch        n     m   0 0.5354
## 3948    moralSelf    moralSelf    match        m     m   1 0.5676
## 3949    moralSelf    moralSelf mismatch        n     n   1 0.6557
## 3950 immoralOther immoralOther mismatch        n     n   1 0.6639
## 3951    moralSelf    moralSelf    match        m     m   1 0.5180
## 3952  immoralSelf  immoralSelf    match        m     m   1 0.6401
## 3953 immoralOther immoralOther    match        m     m   1 0.5702
## 3954  immoralSelf  immoralSelf mismatch        n     n   1 0.6503
## 3955 immoralOther immoralOther mismatch        n     m   0 0.5065
## 3956   moralOther   moralOther    match        m     m   1 0.7926
## 3957  immoralSelf  immoralSelf    match        m     m   1 0.6846
## 3958   moralOther   moralOther    match        m     m   1 0.6949
## 3959   moralOther   moralOther mismatch        n     n   1 0.7350
## 3960    moralSelf    moralSelf    match        m     m   1 0.4952
## 3961    moralSelf    moralSelf    match        m  <NA>  -1 1.0850
## 3962   moralOther   moralOther mismatch        n     m   0 0.8096
## 3963  immoralSelf  immoralSelf    match        m     m   1 0.6817
## 3964   moralOther   moralOther mismatch        n     m   0 0.9479
## 3965 immoralOther immoralOther mismatch        n     n   1 0.9440
## 3966    moralSelf    moralSelf mismatch        n     n   1 0.6461
## 3967 immoralOther immoralOther    match        m     m   1 0.7283
## 3968   moralOther   moralOther    match        m     m   1 0.6884
## 3969 immoralOther immoralOther mismatch        n     n   1 0.6885
## 3970   moralOther   moralOther mismatch        n     n   1 0.8167
## 3971    moralSelf    moralSelf    match        m     m   1 0.5048
## 3972 immoralOther immoralOther    match        m     m   1 0.5509
## 3973  immoralSelf  immoralSelf mismatch        n     n   1 0.6031
## 3974  immoralSelf  immoralSelf    match        m     m   1 0.7331
## 3975    moralSelf    moralSelf mismatch        n     n   1 0.6294
## 3976   moralOther   moralOther    match        m     m   1 0.4895
## 3977   moralOther   moralOther    match        m     n   0 0.7936
## 3978  immoralSelf  immoralSelf    match        m     m   1 0.6777
## 3979  immoralSelf  immoralSelf mismatch        n     n   1 0.6478
## 3980    moralSelf    moralSelf    match        m     m   1 0.5440
## 3981  immoralSelf  immoralSelf mismatch        n     m   0 0.5180
## 3982 immoralOther immoralOther mismatch        n     m   0 0.4643
## 3983 immoralOther immoralOther    match        m     m   1 0.7823
## 3984    moralSelf    moralSelf mismatch        n     m   0 0.5164
## 3985    moralSelf    moralSelf mismatch        n     m   0 0.5206
## 3986   moralOther   moralOther    match        m     m   1 0.6427
## 3987 immoralOther immoralOther    match        m     n   0 0.6048
## 3988   moralOther   moralOther mismatch        n     m   0 0.7030
## 3989   moralOther   moralOther    match        m     m   1 0.9731
## 3990  immoralSelf  immoralSelf    match        m     m   1 0.7652
## 3991   moralOther   moralOther mismatch        n     n   1 0.6954
## 3992  immoralSelf  immoralSelf mismatch        n     m   0 0.5515
## 3993  immoralSelf  immoralSelf    match        m     m   1 0.6597
## 3994    moralSelf    moralSelf    match        m     m   1 0.6018
## 3995  immoralSelf  immoralSelf    match        m     n   0 0.6359
## 3996   moralOther   moralOther    match        m     m   1 0.6000
## 3997    moralSelf    moralSelf mismatch        n     n   1 0.6662
## 3998 immoralOther immoralOther mismatch        n     n   1 0.7123
## 3999 immoralOther immoralOther mismatch        n     n   1 0.6184
## 4000 immoralOther immoralOther mismatch        n     m   0 0.6485
## 4001    moralSelf    moralSelf    match        m     m   1 0.6007
## 4002    moralSelf    moralSelf mismatch        n     n   1 0.6908
## 4003  immoralSelf  immoralSelf mismatch        n     m   0 0.7150
## 4004  immoralSelf  immoralSelf mismatch        n     n   1 0.6871
## 4005    moralSelf    moralSelf    match        m     m   1 0.5611
## 4006   moralOther   moralOther mismatch        n     m   0 0.7393
## 4007 immoralOther immoralOther    match        m     n   0 0.7454
## 4008 immoralOther immoralOther    match        m     m   1 0.6956
## 4009  immoralSelf  immoralSelf    match        m     m   1 0.7277
## 4010 immoralOther immoralOther    match        m     m   1 0.5999
## 4011 immoralOther immoralOther    match        m     m   1 0.5460
## 4012 immoralOther immoralOther mismatch        n     n   1 0.6561
## 4013   moralOther   moralOther    match        m     m   1 0.6322
## 4014 immoralOther immoralOther mismatch        n     n   1 0.7323
## 4015  immoralSelf  immoralSelf mismatch        n     m   0 0.7965
## 4016   moralOther   moralOther mismatch        n     n   1 0.6806
## 4017   moralOther   moralOther mismatch        n     n   1 0.8028
## 4018  immoralSelf  immoralSelf mismatch        n     n   1 0.8529
## 4019    moralSelf    moralSelf mismatch        n     n   1 0.7650
## 4020  immoralSelf  immoralSelf    match        m     m   1 0.6632
## 4021 immoralOther immoralOther mismatch        n     n   1 0.8133
## 4022    moralSelf    moralSelf    match        m     m   1 0.5575
## 4023    moralSelf    moralSelf    match        m     m   1 0.5836
## 4024   moralOther   moralOther    match        m     m   1 0.7576
## 4025    moralSelf    moralSelf mismatch        n     n   1 0.6238
## 4026   moralOther   moralOther mismatch        n     n   1 0.6939
## 4027  immoralSelf  immoralSelf    match        m     m   1 0.5101
## 4028    moralSelf    moralSelf    match        m     m   1 0.7062
## 4029  immoralSelf  immoralSelf mismatch        n     m   0 0.7523
## 4030   moralOther   moralOther    match        m     m   1 0.6285
## 4031 immoralOther immoralOther    match        m     m   1 0.7206
## 4032    moralSelf    moralSelf mismatch        n     n   1 0.8107
## 4033  immoralSelf  immoralSelf    match        m     n   0 0.8139
## 4034 immoralOther immoralOther mismatch        n     m   0 0.7019
## 4035    moralSelf    moralSelf    match        m     m   1 0.5721
## 4036  immoralSelf  immoralSelf    match        m     m   1 0.7242
## 4037    moralSelf    moralSelf    match        m     m   1 0.5984
## 4038 immoralOther immoralOther mismatch        n     n   1 0.5765
## 4039  immoralSelf  immoralSelf    match        m     n   0 0.6726
## 4040    moralSelf    moralSelf mismatch        n     m   0 0.4528
## 4041   moralOther   moralOther mismatch        n     m   0 0.5649
## 4042  immoralSelf  immoralSelf mismatch        n     m   0 0.6870
## 4043   moralOther   moralOther    match        m     m   1 0.6891
## 4044   moralOther   moralOther    match        m     m   1 0.6333
## 4045   moralOther   moralOther mismatch        n     m   0 0.9414
## 4046  immoralSelf  immoralSelf mismatch        n     n   1 0.7715
## 4047    moralSelf    moralSelf    match        m     m   1 0.5636
## 4048    moralSelf    moralSelf mismatch        n     m   0 0.6378
## 4049 immoralOther immoralOther    match        m     m   1 0.6960
## 4050  immoralSelf  immoralSelf mismatch        n     m   0 0.6560
## 4051   moralOther   moralOther    match        m     m   1 0.9442
## 4052   moralOther   moralOther mismatch        n     n   1 0.7664
## 4053 immoralOther immoralOther mismatch        n     n   1 0.8245
## 4054 immoralOther immoralOther    match        m     m   1 0.6306
## 4055 immoralOther immoralOther    match        m     m   1 0.5387
## 4056    moralSelf    moralSelf mismatch        n     n   1 0.7129
## 4057    moralSelf    moralSelf    match        m     m   1 0.5810
## 4058  immoralSelf  immoralSelf    match        m     m   1 0.8831
## 4059 immoralOther immoralOther    match        m     m   1 0.6212
## 4060  immoralSelf  immoralSelf mismatch        n     m   0 0.6993
## 4061    moralSelf    moralSelf    match        m     m   1 0.5295
## 4062   moralOther   moralOther mismatch        n     n   1 0.8656
## 4063   moralOther   moralOther mismatch        n     m   0 0.7998
## 4064 immoralOther immoralOther mismatch        n     n   1 0.6739
## 4065   moralOther   moralOther    match        m     n   0 0.8141
## 4066  immoralSelf  immoralSelf mismatch        n     m   0 0.7741
## 4067 immoralOther immoralOther    match        m     m   1 0.7584
## 4068    moralSelf    moralSelf mismatch        n     n   1 0.6885
## 4069 immoralOther immoralOther    match        m     m   1 0.5607
## 4070   moralOther   moralOther mismatch        n     m   0 0.6747
## 4071  immoralSelf  immoralSelf    match        m     n   0 0.6948
## 4072    moralSelf    moralSelf mismatch        n     n   1 0.8070
## 4073  immoralSelf  immoralSelf    match        m     m   1 0.7211
## 4074   moralOther   moralOther    match        m     m   1 0.7873
## 4075 immoralOther immoralOther mismatch        n     n   1 0.6634
## 4076    moralSelf    moralSelf    match        m     m   1 0.6695
## 4077  immoralSelf  immoralSelf mismatch        n     m   0 0.5776
## 4078   moralOther   moralOther    match        m     m   1 0.5758
## 4079 immoralOther immoralOther mismatch        n     n   1 0.6959
## 4080    moralSelf    moralSelf mismatch        n     n   1 0.7280
## 4081    moralSelf    moralSelf    match        m     m   1 0.4157
## 4082 immoralOther immoralOther mismatch        n     n   1 0.5117
## 4083  immoralSelf  immoralSelf mismatch        n     n   1 0.6338
## 4084   moralOther   moralOther    match        m     m   1 0.8379
## 4085    moralSelf    moralSelf    match        m     m   1 0.5121
## 4086  immoralSelf  immoralSelf    match        m     n   0 0.6842
## 4087    moralSelf    moralSelf mismatch        n     n   1 0.7583
## 4088  immoralSelf  immoralSelf mismatch        n     n   1 0.6645
## 4089 immoralOther immoralOther mismatch        n     n   1 0.6226
## 4090 immoralOther immoralOther    match        m     n   0 0.5887
## 4091  immoralSelf  immoralSelf    match        m     n   0 0.6328
## 4092 immoralOther immoralOther    match        m     m   1 0.7010
## 4093 immoralOther immoralOther    match        m     m   1 0.6431
## 4094   moralOther   moralOther    match        m     m   1 0.5392
## 4095   moralOther   moralOther mismatch        n     m   0 0.6814
## 4096    moralSelf    moralSelf    match        m     m   1 0.6835
## 4097   moralOther   moralOther mismatch        n     m   0 0.9335
## 4098   moralOther   moralOther    match        m     m   1 0.5738
## 4099  immoralSelf  immoralSelf    match        m     n   0 0.6299
## 4100   moralOther   moralOther mismatch        n     n   1 0.7220
## 4101    moralSelf    moralSelf mismatch        n     n   1 0.7642
## 4102 immoralOther immoralOther mismatch        n     n   1 0.8863
## 4103  immoralSelf  immoralSelf mismatch        n     n   1 0.7764
## 4104    moralSelf    moralSelf mismatch        n     n   1 0.5926
## 4105   moralOther   moralOther mismatch        n     n   1 0.7006
## 4106  immoralSelf  immoralSelf mismatch        n     m   0 0.4828
## 4107  immoralSelf  immoralSelf    match        m     n   0 0.5330
## 4108   moralOther   moralOther mismatch        n     m   0 0.5971
## 4109   moralOther   moralOther    match        m     m   1 0.6252
## 4110   moralOther   moralOther    match        m     n   0 0.6073
## 4111 immoralOther immoralOther    match        m     m   1 0.5994
## 4112 immoralOther immoralOther mismatch        n     n   1 0.6656
## 4113    moralSelf    moralSelf mismatch        n     n   1 0.6077
## 4114  immoralSelf  immoralSelf mismatch        n     n   1 0.6558
## 4115   moralOther   moralOther    match        m     m   1 0.5759
## 4116    moralSelf    moralSelf    match        m     m   1 0.5680
## 4117  immoralSelf  immoralSelf mismatch        n     n   1 0.5843
## 4118    moralSelf    moralSelf mismatch        n     n   1 0.7743
## 4119 immoralOther immoralOther    match        m     m   1 0.7665
## 4120    moralSelf    moralSelf mismatch        n     n   1 0.6566
## 4121  immoralSelf  immoralSelf    match        m     m   1 0.6648
## 4122  immoralSelf  immoralSelf    match        m     m   1 0.7188
## 4123   moralOther   moralOther mismatch        n     m   0 0.8091
## 4124 immoralOther immoralOther    match        m     m   1 0.6491
## 4125    moralSelf    moralSelf    match        m     m   1 0.5932
## 4126 immoralOther immoralOther mismatch        n     n   1 0.8053
## 4127    moralSelf    moralSelf    match        m     m   1 0.5115
## 4128 immoralOther immoralOther mismatch        n     m   0 0.7637
## 4129  immoralSelf  immoralSelf    match        m     n   0 0.8062
## 4130 immoralOther immoralOther    match        m     m   1 0.7524
## 4131 immoralOther immoralOther mismatch        n     n   1 0.7645
## 4132  immoralSelf  immoralSelf mismatch        n     n   1 0.9106
## 4133   moralOther   moralOther mismatch        n     n   1 0.8188
## 4134   moralOther   moralOther    match        m     m   1 0.6729
## 4135   moralOther   moralOther mismatch        n     m   0 0.8291
## 4136    moralSelf    moralSelf mismatch        n     n   1 0.8732
## 4137    moralSelf    moralSelf    match        m     m   1 0.5813
## 4138    moralSelf    moralSelf mismatch        n     n   1 0.5955
## 4139  immoralSelf  immoralSelf mismatch        n     n   1 0.6875
## 4140 immoralOther immoralOther    match        m     m   1 0.7677
## 4141 immoralOther immoralOther    match        m     m   1 0.6639
## 4142   moralOther   moralOther mismatch        n     n   1 0.6380
## 4143   moralOther   moralOther    match        m     m   1 0.5761
## 4144    moralSelf    moralSelf mismatch        n     n   1 0.7282
## 4145    moralSelf    moralSelf    match        m     m   1 0.6084
## 4146   moralOther   moralOther    match        m     m   1 0.6805
## 4147  immoralSelf  immoralSelf mismatch        n     n   1 0.7226
## 4148 immoralOther immoralOther mismatch        n     n   1 0.6287
## 4149 immoralOther immoralOther mismatch        n     n   1 0.5368
## 4150  immoralSelf  immoralSelf    match        m     m   1 0.5690
## 4151  immoralSelf  immoralSelf    match        m     n   0 0.5771
## 4152    moralSelf    moralSelf    match        m     m   1 0.5213
## 4153  immoralSelf  immoralSelf    match        m     n   0 0.5393
## 4154 immoralOther immoralOther    match        m     n   0 0.6815
## 4155   moralOther   moralOther    match        m     n   0 0.7476
## 4156   moralOther   moralOther mismatch        n     n   1 0.8078
## 4157    moralSelf    moralSelf mismatch        n     m   0 0.6479
## 4158 immoralOther immoralOther mismatch        n     n   1 0.6640
## 4159    moralSelf    moralSelf    match        m     m   1 0.6461
## 4160  immoralSelf  immoralSelf    match        m     m   1 0.9042
## 4161    moralSelf    moralSelf    match        m     m   1 0.7664
## 4162    moralSelf    moralSelf mismatch        n     n   1 0.7046
## 4163  immoralSelf  immoralSelf mismatch        n     m   0 0.6866
## 4164 immoralOther immoralOther mismatch        n     n   1 0.6968
## 4165  immoralSelf  immoralSelf mismatch        n     n   1 0.6409
## 4166  immoralSelf  immoralSelf    match        m     m   1 0.6111
## 4167 immoralOther immoralOther    match        m     m   1 0.5312
## 4168   moralOther   moralOther mismatch        n     n   1 0.6434
## 4169   moralOther   moralOther mismatch        n     n   1 0.7495
## 4170    moralSelf    moralSelf    match        m     m   1 0.5296
## 4171  immoralSelf  immoralSelf mismatch        n     n   1 0.6337
## 4172 immoralOther immoralOther mismatch        n     n   1 0.7578
## 4173   moralOther   moralOther    match        m     n   0 0.7359
## 4174    moralSelf    moralSelf mismatch        n     n   1 0.6321
## 4175 immoralOther immoralOther    match        m     m   1 0.7163
## 4176   moralOther   moralOther    match        m     m   1 0.6324
## 4177   moralOther   moralOther mismatch        n     n   1 0.7650
## 4178  immoralSelf  immoralSelf    match        m     m   1 0.7671
## 4179    moralSelf    moralSelf mismatch        n     n   1 0.7932
## 4180 immoralOther immoralOther mismatch        n     n   1 0.6214
## 4181   moralOther   moralOther mismatch        n     n   1 0.7074
## 4182 immoralOther immoralOther    match        m     m   1 0.6476
## 4183  immoralSelf  immoralSelf    match        m     n   0 0.6478
## 4184   moralOther   moralOther    match        m     m   1 0.8159
## 4185    moralSelf    moralSelf mismatch        n     m   0 0.5681
## 4186    moralSelf    moralSelf    match        m     m   1 0.6241
## 4187 immoralOther immoralOther    match        m     m   1 0.6383
## 4188 immoralOther immoralOther mismatch        n     n   1 0.6324
## 4189   moralOther   moralOther    match        m     m   1 0.7725
## 4190   moralOther   moralOther    match        m     m   1 0.4927
## 4191    moralSelf    moralSelf    match        m     n   0 0.6688
## 4192  immoralSelf  immoralSelf mismatch        n     n   1 0.7133
## 4193  immoralSelf  immoralSelf mismatch        n     m   0 0.6371
## 4194 immoralOther immoralOther    match        m     m   1 0.6892
## 4195    moralSelf    moralSelf mismatch        n     m   0 0.5612
## 4196    moralSelf    moralSelf    match        m     m   1 0.5474
## 4197   moralOther   moralOther mismatch        n     n   1 0.6476
## 4198 immoralOther immoralOther mismatch        n     m   0 0.5677
## 4199  immoralSelf  immoralSelf    match        m     m   1 0.7278
## 4200  immoralSelf  immoralSelf mismatch        n     n   1 0.7680
## 4201    moralSelf    moralSelf mismatch        n     n   1 0.8401
## 4202   moralOther   moralOther mismatch        n     n   1 0.6583
## 4203  immoralSelf  immoralSelf    match        m     n   0 0.6724
## 4204    moralSelf    moralSelf    match        m     m   1 0.5405
## 4205  immoralSelf  immoralSelf mismatch        n     n   1 0.6186
## 4206    moralSelf    moralSelf mismatch        n     m   0 0.5767
## 4207   moralOther   moralOther    match        m     n   0 0.8169
## 4208    moralSelf    moralSelf    match        m     m   1 0.5850
## 4209 immoralOther immoralOther    match        m     m   1 0.6312
## 4210  immoralSelf  immoralSelf mismatch        n     n   1 0.7232
## 4211 immoralOther immoralOther    match        m     m   1 0.7174
## 4212 immoralOther immoralOther mismatch        n     m   0 0.6394
## 4213  immoralSelf  immoralSelf    match        m     n   0 0.7377
## 4214 immoralOther immoralOther mismatch        n     n   1 0.7758
## 4215    moralSelf    moralSelf    match        m     m   1 0.5279
## 4216  immoralSelf  immoralSelf    match        m     m   1 0.6080
## 4217    moralSelf    moralSelf mismatch        n     n   1 0.7042
## 4218   moralOther   moralOther mismatch        n     n   1 0.6143
## 4219 immoralOther immoralOther mismatch        n     n   1 0.5744
## 4220   moralOther   moralOther    match        m     m   1 0.4946
## 4221   moralOther   moralOther mismatch        n     m   0 0.6547
## 4222   moralOther   moralOther    match        m     m   1 0.6069
## 4223  immoralSelf  immoralSelf mismatch        n     m   0 0.6889
## 4224 immoralOther immoralOther    match        m     m   1 0.6251
## 4225   moralOther   moralOther mismatch        n     m   0 0.7996
## 4226    moralSelf    moralSelf    match        m     m   1 0.5777
## 4227   moralOther   moralOther mismatch        n     m   0 0.8158
## 4228 immoralOther immoralOther    match        m     n   0 0.5840
## 4229  immoralSelf  immoralSelf mismatch        n     n   1 0.9741
## 4230  immoralSelf  immoralSelf    match        m     m   1 0.6462
## 4231 immoralOther immoralOther mismatch        n     n   1 0.7524
## 4232  immoralSelf  immoralSelf mismatch        n     m   0 0.8125
## 4233 immoralOther immoralOther    match        m     m   1 0.8126
## 4234   moralOther   moralOther    match        m     n   0 0.7008
## 4235  immoralSelf  immoralSelf mismatch        n     n   1 0.5709
## 4236 immoralOther immoralOther mismatch        n     n   1 0.7630
## 4237    moralSelf    moralSelf mismatch        n     m   0 0.4853
## 4238    moralSelf    moralSelf mismatch        n     n   1 0.6954
## 4239  immoralSelf  immoralSelf    match        m     n   0 0.5674
## 4240   moralOther   moralOther mismatch        n     n   1 0.7596
## 4241  immoralSelf  immoralSelf    match        m     m   1 0.4577
## 4242 immoralOther immoralOther    match        m     n   0 0.5839
## 4243   moralOther   moralOther    match        m     m   1 0.5739
## 4244   moralOther   moralOther    match        m     m   1 0.6461
## 4245 immoralOther immoralOther mismatch        n     m   0 0.4642
## 4246    moralSelf    moralSelf mismatch        n     m   0 0.6864
## 4247    moralSelf    moralSelf    match        m     m   1 0.6084
## 4248    moralSelf    moralSelf    match        m     m   1 0.4966
## 4249   moralOther   moralOther mismatch        n     n   1 0.6807
## 4250 immoralOther immoralOther    match        m     n   0 0.7628
## 4251    moralSelf    moralSelf mismatch        n     m   0 0.6210
## 4252  immoralSelf  immoralSelf mismatch        n     m   0 0.7711
## 4253  immoralSelf  immoralSelf    match        m     n   0 0.7712
## 4254    moralSelf    moralSelf mismatch        n     n   1 0.9393
## 4255 immoralOther immoralOther mismatch        n     m   0 0.5455
## 4256  immoralSelf  immoralSelf    match        m     m   1 0.7516
## 4257    moralSelf    moralSelf    match        m     m   1 0.5478
## 4258    moralSelf    moralSelf mismatch        n     n   1 0.6320
## 4259 immoralOther immoralOther    match        m     m   1 0.8761
## 4260  immoralSelf  immoralSelf mismatch        n     n   1 0.8482
## 4261   moralOther   moralOther    match        m     m   1 0.6803
## 4262 immoralOther immoralOther    match        m     m   1 0.6745
## 4263   moralOther   moralOther    match        m     m   1 0.7346
## 4264  immoralSelf  immoralSelf mismatch        n     n   1 0.6948
## 4265    moralSelf    moralSelf    match        m     m   1 0.5668
## 4266   moralOther   moralOther mismatch        n     n   1 0.6010
## 4267 immoralOther immoralOther mismatch        n     n   1 0.7711
## 4268  immoralSelf  immoralSelf    match        m     m   1 0.6832
## 4269   moralOther   moralOther mismatch        n     n   1 0.7734
## 4270 immoralOther immoralOther mismatch        n     m   0 0.7895
## 4271    moralSelf    moralSelf    match        m     m   1 0.6417
## 4272   moralOther   moralOther    match        m     m   1 0.6357
## 4273  immoralSelf  immoralSelf    match        m     n   0 0.8060
## 4274 immoralOther immoralOther mismatch        n     n   1 0.6720
## 4275 immoralOther immoralOther mismatch        n     m   0 0.6762
## 4276 immoralOther immoralOther mismatch        n     n   1 0.6644
## 4277   moralOther   moralOther mismatch        n     m   0 0.6945
## 4278 immoralOther immoralOther    match        m     m   1 0.6547
## 4279   moralOther   moralOther mismatch        n     n   1 0.6947
## 4280    moralSelf    moralSelf    match        m     m   1 0.5109
## 4281 immoralOther immoralOther    match        m     n   0 0.5470
## 4282  immoralSelf  immoralSelf    match        m     m   1 0.8011
## 4283    moralSelf    moralSelf mismatch        n     m   0 0.5073
## 4284  immoralSelf  immoralSelf mismatch        n     n   1 0.7674
## 4285  immoralSelf  immoralSelf mismatch        n     n   1 0.7116
## 4286  immoralSelf  immoralSelf    match        m     m   1 0.6597
## 4287    moralSelf    moralSelf mismatch        n     m   0 0.6098
## 4288   moralOther   moralOther mismatch        n     n   1 0.9219
## 4289  immoralSelf  immoralSelf mismatch        n     n   1 0.6441
## 4290    moralSelf    moralSelf    match        m     m   1 0.5182
## 4291 immoralOther immoralOther    match        m     n   0 0.5283
## 4292   moralOther   moralOther    match        m     n   0 0.6245
## 4293   moralOther   moralOther    match        m     n   0 0.5746
## 4294    moralSelf    moralSelf    match        m     m   1 0.5427
## 4295    moralSelf    moralSelf mismatch        n     n   1 0.9008
## 4296   moralOther   moralOther    match        m     m   1 0.6429
## 4297   moralOther   moralOther    match        m     n   0 0.7651
## 4298    moralSelf    moralSelf mismatch        n     n   1 0.7514
## 4299  immoralSelf  immoralSelf    match        m     m   1 0.6674
## 4300   moralOther   moralOther    match        m     m   1 0.5755
## 4301   moralOther   moralOther mismatch        n     m   0 0.6157
## 4302 immoralOther immoralOther    match        m     m   1 0.6397
## 4303 immoralOther immoralOther mismatch        n     n   1 0.7779
## 4304  immoralSelf  immoralSelf    match        m     n   0 0.6880
## 4305  immoralSelf  immoralSelf mismatch        n     n   1 0.8081
## 4306  immoralSelf  immoralSelf mismatch        n     m   0 0.8643
## 4307    moralSelf    moralSelf mismatch        n     n   1 0.7424
## 4308    moralSelf    moralSelf    match        m     n   0 0.7006
## 4309    moralSelf    moralSelf    match        m     m   1 0.5627
## 4310 immoralOther immoralOther mismatch        n     n   1 0.7488
## 4311 immoralOther immoralOther    match        m     m   1 0.5450
## 4312 immoralOther immoralOther    match        m     m   1 0.6231
## 4313    moralSelf    moralSelf mismatch        n     m   0 0.6612
## 4314  immoralSelf  immoralSelf    match        m     n   0 0.6593
## 4315   moralOther   moralOther    match        m     m   1 0.6175
## 4316   moralOther   moralOther mismatch        n     n   1 0.8795
## 4317 immoralOther immoralOther mismatch        n     n   1 0.7938
## 4318   moralOther   moralOther mismatch        n     m   0 0.7579
## 4319    moralSelf    moralSelf    match        m     m   1 0.6401
## 4320  immoralSelf  immoralSelf mismatch        n     n   1 0.6741
## 4321   moralOther   moralOther mismatch        n     n   1 0.5062
## 4322  immoralSelf  immoralSelf    match        m     m   1 0.6682
## 4323  immoralSelf  immoralSelf    match        m     m   1 0.6123
## 4324  immoralSelf  immoralSelf mismatch        n     n   1 0.6484
## 4325   moralOther   moralOther mismatch        n     n   1 0.7326
## 4326    moralSelf    moralSelf mismatch        n     n   1 0.5767
## 4327    moralSelf    moralSelf    match        m     m   1 0.5409
## 4328  immoralSelf  immoralSelf mismatch        n     n   1 0.7193
## 4329   moralOther   moralOther    match        m     m   1 0.4451
## 4330    moralSelf    moralSelf mismatch        n     m   0 0.5152
## 4331   moralOther   moralOther mismatch        n     n   1 0.5153
## 4332 immoralOther immoralOther mismatch        n     n   1 0.7154
## 4333   moralOther   moralOther    match        m     m   1 0.5475
## 4334    moralSelf    moralSelf    match        m     m   1 0.5637
## 4335    moralSelf    moralSelf    match        m     m   1 0.5379
## 4336  immoralSelf  immoralSelf mismatch        n     n   1 0.6599
## 4337  immoralSelf  immoralSelf    match        m     m   1 0.6541
## 4338    moralSelf    moralSelf mismatch        n     m   0 0.9701
## 4339   moralOther   moralOther    match        m     m   1 0.4863
## 4340 immoralOther immoralOther mismatch        n     n   1 0.6364
## 4341 immoralOther immoralOther    match        m     m   1 0.8765
## 4342 immoralOther immoralOther    match        m     n   0 1.0686
## 4343 immoralOther immoralOther mismatch        n     n   1 0.9249
## 4344 immoralOther immoralOther    match        m     m   1 1.0830
## 4345  immoralSelf  immoralSelf    match        m     m   1 0.9771
## 4346    moralSelf    moralSelf    match        m     m   1 0.5413
## 4347 immoralOther immoralOther mismatch        n     n   1 0.7673
## 4348   moralOther   moralOther mismatch        n  <NA>  -1 1.1430
## 4349    moralSelf    moralSelf    match        m     m   1 0.5496
## 4350   moralOther   moralOther    match        m     m   1 0.6217
## 4351  immoralSelf  immoralSelf mismatch        n     m   0 0.6719
## 4352 immoralOther immoralOther mismatch        n     n   1 0.5920
## 4353  immoralSelf  immoralSelf mismatch        n     n   1 0.6801
## 4354  immoralSelf  immoralSelf mismatch        n     n   1 0.6622
## 4355   moralOther   moralOther    match        m     m   1 0.4886
## 4356 immoralOther immoralOther    match        m     n   0 0.6524
## 4357   moralOther   moralOther    match        m     m   1 0.5606
## 4358   moralOther   moralOther mismatch        n     n   1 0.5507
## 4359    moralSelf    moralSelf mismatch        n     n   1 0.6868
## 4360  immoralSelf  immoralSelf    match        m     m   1 0.7409
## 4361    moralSelf    moralSelf    match        m     m   1 0.4791
## 4362    moralSelf    moralSelf mismatch        n     m   0 0.8132
## 4363   moralOther   moralOther mismatch        n     n   1 0.5773
## 4364 immoralOther immoralOther mismatch        n     n   1 0.6115
## 4365    moralSelf    moralSelf mismatch        n     n   1 0.7756
## 4366  immoralSelf  immoralSelf    match        m     m   1 0.6597
## 4367 immoralOther immoralOther    match        m     m   1 0.7098
## 4368 immoralOther immoralOther    match        m     m   1 0.4860
## 4369  immoralSelf  immoralSelf    match        m     m   1 0.6601
## 4370 immoralOther immoralOther mismatch        n     n   1 0.8623
## 4371  immoralSelf  immoralSelf mismatch        n     n   1 0.7123
## 4372 immoralOther immoralOther mismatch        n     m   0 0.5125
## 4373    moralSelf    moralSelf mismatch        n     n   1 0.6086
## 4374  immoralSelf  immoralSelf mismatch        n     n   1 0.8687
## 4375    moralSelf    moralSelf mismatch        n     n   1 0.6128
## 4376 immoralOther immoralOther    match        m     n   0 0.5690
## 4377    moralSelf    moralSelf    match        m     m   1 1.0611
## 4378 immoralOther immoralOther mismatch        n     n   1 0.7433
## 4379  immoralSelf  immoralSelf    match        m     n   0 0.7214
## 4380   moralOther   moralOther    match        m     m   1 0.4315
## 4381   moralOther   moralOther mismatch        n     n   1 0.5356
## 4382   moralOther   moralOther    match        m     m   1 0.4677
## 4383   moralOther   moralOther    match        m     m   1 0.5158
## 4384  immoralSelf  immoralSelf mismatch        n     m   0 0.7320
## 4385   moralOther   moralOther mismatch        n     n   1 0.7441
## 4386  immoralSelf  immoralSelf    match        m     m   1 0.6342
## 4387    moralSelf    moralSelf    match        m     m   1 0.5544
## 4388   moralOther   moralOther mismatch        n     m   0 0.4745
## 4389 immoralOther immoralOther    match        m     m   1 0.7305
## 4390    moralSelf    moralSelf    match        m     m   1 0.4947
## 4391    moralSelf    moralSelf mismatch        n     n   1 0.7468
## 4392 immoralOther immoralOther    match        m     m   1 0.5569
## 4393    moralSelf    moralSelf mismatch        n     n   1 0.7281
## 4394   moralOther   moralOther mismatch        n     n   1 0.5163
## 4395 immoralOther immoralOther mismatch        n     n   1 0.6525
## 4396    moralSelf    moralSelf    match        m     m   1 0.5846
## 4397 immoralOther immoralOther mismatch        n     n   1 0.5307
## 4398  immoralSelf  immoralSelf mismatch        n     n   1 0.7508
## 4399    moralSelf    moralSelf    match        m     m   1 0.4709
## 4400 immoralOther immoralOther    match        m     m   1 0.6550
## 4401  immoralSelf  immoralSelf    match        m     m   1 0.8512
## 4402  immoralSelf  immoralSelf    match        m     m   1 0.6073
## 4403   moralOther   moralOther    match        m     m   1 0.4595
## 4404 immoralOther immoralOther mismatch        n     n   1 0.6376
## 4405 immoralOther immoralOther    match        m     m   1 0.5797
## 4406    moralSelf    moralSelf mismatch        n     m   0 0.4698
## 4407  immoralSelf  immoralSelf mismatch        n     n   1 0.7019
## 4408   moralOther   moralOther mismatch        n     n   1 0.5200
## 4409 immoralOther immoralOther    match        m     m   1 0.6382
## 4410    moralSelf    moralSelf    match        m     m   1 0.4923
## 4411   moralOther   moralOther    match        m     m   1 0.5204
## 4412   moralOther   moralOther mismatch        n     n   1 0.5025
## 4413  immoralSelf  immoralSelf    match        m     m   1 0.6326
## 4414  immoralSelf  immoralSelf mismatch        n     n   1 0.5767
## 4415    moralSelf    moralSelf mismatch        n     n   1 0.5809
## 4416   moralOther   moralOther    match        m     m   1 0.4630
## 4417  immoralSelf  immoralSelf mismatch        n     n   1 0.6011
## 4418 immoralOther immoralOther mismatch        n     n   1 0.5672
## 4419   moralOther   moralOther    match        m     n   0 0.5553
## 4420    moralSelf    moralSelf    match        m     m   1 0.6114
## 4421   moralOther   moralOther    match        m     m   1 0.5036
## 4422  immoralSelf  immoralSelf    match        m     m   1 0.5857
## 4423  immoralSelf  immoralSelf    match        m     m   1 0.7077
## 4424    moralSelf    moralSelf    match        m     m   1 0.6200
## 4425 immoralOther immoralOther    match        m     m   1 0.5740
## 4426    moralSelf    moralSelf    match        m     m   1 0.4741
## 4427   moralOther   moralOther mismatch        n     m   0 0.7543
## 4428 immoralOther immoralOther    match        m     m   1 0.4984
## 4429  immoralSelf  immoralSelf    match        m     m   1 0.6685
## 4430   moralOther   moralOther mismatch        n     n   1 0.8366
## 4431 immoralOther immoralOther    match        m     m   1 0.6608
## 4432    moralSelf    moralSelf mismatch        n     n   1 0.7909
## 4433  immoralSelf  immoralSelf mismatch        n     m   0 0.5834
## 4434   moralOther   moralOther    match        m     m   1 0.4731
## 4435 immoralOther immoralOther mismatch        n     n   1 0.5933
## 4436 immoralOther immoralOther mismatch        n     n   1 0.7054
## 4437    moralSelf    moralSelf mismatch        n     n   1 0.8115
## 4438  immoralSelf  immoralSelf mismatch        n     n   1 0.8076
## 4439    moralSelf    moralSelf mismatch        n     n   1 0.6438
## 4440   moralOther   moralOther mismatch        n     n   1 0.5859
## 4441 immoralOther immoralOther mismatch        n     n   1 0.6802
## 4442   moralOther   moralOther mismatch        n     n   1 0.6224
## 4443  immoralSelf  immoralSelf mismatch        n     n   1 0.7285
## 4444   moralOther   moralOther    match        m     m   1 0.5245
## 4445 immoralOther immoralOther    match        m     m   1 0.8506
## 4446  immoralSelf  immoralSelf    match        m     m   1 0.8308
## 4447    moralSelf    moralSelf mismatch        n     m   0 0.6629
## 4448   moralOther   moralOther mismatch        n     n   1 0.6011
## 4449   moralOther   moralOther mismatch        n     n   1 0.5512
## 4450    moralSelf    moralSelf mismatch        n     n   1 0.5913
## 4451  immoralSelf  immoralSelf    match        m     m   1 0.6474
## 4452  immoralSelf  immoralSelf    match        m     m   1 0.5475
## 4453 immoralOther immoralOther mismatch        n     n   1 0.5636
## 4454 immoralOther immoralOther    match        m     m   1 0.6818
## 4455  immoralSelf  immoralSelf mismatch        n     m   0 0.5279
## 4456    moralSelf    moralSelf    match        m     m   1 0.5480
## 4457    moralSelf    moralSelf mismatch        n     n   1 0.8202
## 4458    moralSelf    moralSelf    match        m     m   1 0.5983
## 4459  immoralSelf  immoralSelf mismatch        n     n   1 0.6605
## 4460 immoralOther immoralOther mismatch        n     n   1 0.5906
## 4461   moralOther   moralOther    match        m     m   1 0.4626
## 4462    moralSelf    moralSelf    match        m     m   1 0.5688
## 4463 immoralOther immoralOther    match        m     n   0 0.5889
## 4464   moralOther   moralOther    match        m     m   1 0.4370
## 4465 immoralOther immoralOther mismatch        n     n   1 0.8851
## 4466    moralSelf    moralSelf    match        m     m   1 0.4673
## 4467    moralSelf    moralSelf mismatch        n     n   1 0.7954
## 4468   moralOther   moralOther    match        m     m   1 0.4755
## 4469   moralOther   moralOther mismatch        n     n   1 0.5476
## 4470   moralOther   moralOther mismatch        n     n   1 0.5477
## 4471    moralSelf    moralSelf mismatch        n     n   1 0.8679
## 4472  immoralSelf  immoralSelf    match        m     m   1 0.6440
## 4473  immoralSelf  immoralSelf mismatch        n     n   1 0.5061
## 4474 immoralOther immoralOther mismatch        n     n   1 0.8363
## 4475   moralOther   moralOther    match        m     m   1 0.4684
## 4476 immoralOther immoralOther    match        m     n   0 0.4925
## 4477  immoralSelf  immoralSelf mismatch        n     n   1 0.5846
## 4478   moralOther   moralOther    match        m     m   1 0.5388
## 4479 immoralOther immoralOther    match        m     m   1 0.6369
## 4480    moralSelf    moralSelf    match        m     m   1 0.6530
## 4481  immoralSelf  immoralSelf mismatch        n     n   1 0.5371
## 4482 immoralOther immoralOther mismatch        n     n   1 0.6612
## 4483  immoralSelf  immoralSelf    match        m     m   1 0.7513
## 4484   moralOther   moralOther mismatch        n     n   1 0.5434
## 4485    moralSelf    moralSelf    match        m     m   1 0.6016
## 4486  immoralSelf  immoralSelf    match        m     m   1 0.5677
## 4487 immoralOther immoralOther    match        m     m   1 0.6358
## 4488    moralSelf    moralSelf mismatch        n     n   1 0.6040
## 4489   moralOther   moralOther    match        m     m   1 0.5621
## 4490  immoralSelf  immoralSelf mismatch        n     n   1 0.6642
## 4491    moralSelf    moralSelf mismatch        n     n   1 0.7463
## 4492 immoralOther immoralOther mismatch        n     n   1 0.6764
## 4493    moralSelf    moralSelf    match        m     m   1 0.4846
## 4494  immoralSelf  immoralSelf mismatch        n     n   1 0.5386
## 4495    moralSelf    moralSelf mismatch        n     n   1 0.6527
## 4496   moralOther   moralOther mismatch        n     n   1 0.5369
## 4497   moralOther   moralOther mismatch        n     n   1 0.5331
## 4498    moralSelf    moralSelf    match        m     m   1 0.5771
## 4499  immoralSelf  immoralSelf    match        m     n   0 0.5872
## 4500   moralOther   moralOther mismatch        n     n   1 0.5953
## 4501    moralSelf    moralSelf    match        m     m   1 0.6915
## 4502  immoralSelf  immoralSelf    match        m     m   1 0.6796
## 4503   moralOther   moralOther    match        m     m   1 0.5257
## 4504 immoralOther immoralOther    match        m     m   1 0.5959
## 4505  immoralSelf  immoralSelf mismatch        n     m   0 0.5319
## 4506 immoralOther immoralOther    match        m     m   1 0.7361
## 4507   moralOther   moralOther    match        m     m   1 0.4842
## 4508  immoralSelf  immoralSelf    match        m     n   0 0.8903
## 4509    moralSelf    moralSelf mismatch        n     n   1 0.5985
## 4510 immoralOther immoralOther    match        m     m   1 0.5326
## 4511 immoralOther immoralOther mismatch        n     n   1 0.5767
## 4512 immoralOther immoralOther mismatch        n     n   1 0.6288
## 4513  immoralSelf  immoralSelf    match        m     m   1 0.7859
## 4514 immoralOther immoralOther    match        m     m   1 0.5561
## 4515  immoralSelf  immoralSelf mismatch        n     n   1 0.6682
## 4516    moralSelf    moralSelf mismatch        n     n   1 0.8363
## 4517  immoralSelf  immoralSelf    match        m     m   1 0.6604
## 4518 immoralOther immoralOther    match        m     m   1 0.5906
## 4519  immoralSelf  immoralSelf    match        m     m   1 0.7667
## 4520   moralOther   moralOther    match        m     m   1 0.4848
## 4521   moralOther   moralOther mismatch        n     n   1 0.6509
## 4522 immoralOther immoralOther mismatch        n     n   1 0.6471
## 4523 immoralOther immoralOther mismatch        n     n   1 0.6211
## 4524    moralSelf    moralSelf mismatch        n     n   1 0.6235
## 4525  immoralSelf  immoralSelf mismatch        n     n   1 0.6255
## 4526 immoralOther immoralOther    match        m     m   1 0.6995
## 4527 immoralOther immoralOther mismatch        n     n   1 0.5176
## 4528   moralOther   moralOther mismatch        n     n   1 0.6197
## 4529    moralSelf    moralSelf    match        m     m   1 0.4939
## 4530  immoralSelf  immoralSelf mismatch        n     n   1 0.5381
## 4531   moralOther   moralOther    match        m     m   1 0.4602
## 4532    moralSelf    moralSelf    match        m     n   0 0.6542
## 4533   moralOther   moralOther    match        m     m   1 0.4664
## 4534    moralSelf    moralSelf    match        m     m   1 0.4605
## 4535   moralOther   moralOther mismatch        n     n   1 0.5426
## 4536    moralSelf    moralSelf mismatch        n     n   1 0.7447
## 4537    moralSelf    moralSelf    match        m     m   1 0.6588
## 4538 immoralOther immoralOther    match        m     m   1 0.7409
## 4539   moralOther   moralOther    match        m     m   1 0.5431
## 4540  immoralSelf  immoralSelf mismatch        n     n   1 0.6653
## 4541  immoralSelf  immoralSelf    match        m     m   1 0.8033
## 4542   moralOther   moralOther mismatch        n     n   1 0.5374
## 4543    moralSelf    moralSelf    match        m     m   1 0.5716
## 4544 immoralOther immoralOther mismatch        n     n   1 0.6157
## 4545 immoralOther immoralOther    match        m     m   1 0.7158
## 4546   moralOther   moralOther    match        m     m   1 0.5399
## 4547   moralOther   moralOther    match        m     m   1 0.5100
## 4548 immoralOther immoralOther mismatch        n     n   1 0.5582
## 4549 immoralOther immoralOther    match        m     m   1 0.5243
## 4550  immoralSelf  immoralSelf    match        m     m   1 0.6104
## 4551    moralSelf    moralSelf mismatch        n     n   1 0.6065
## 4552   moralOther   moralOther mismatch        n     n   1 0.5487
## 4553    moralSelf    moralSelf    match        m     m   1 0.4368
## 4554 immoralOther immoralOther mismatch        n     n   1 0.6770
## 4555  immoralSelf  immoralSelf    match        m     m   1 0.5570
## 4556   moralOther   moralOther mismatch        n     n   1 0.5331
## 4557  immoralSelf  immoralSelf mismatch        n     n   1 0.6572
## 4558  immoralSelf  immoralSelf mismatch        n     n   1 0.9794
## 4559    moralSelf    moralSelf mismatch        n     n   1 0.7495
## 4560    moralSelf    moralSelf mismatch        n     n   1 0.6776
## 4561 immoralOther immoralOther mismatch        n     n   1 0.5919
## 4562 immoralOther immoralOther mismatch        n     n   1 0.6161
## 4563    moralSelf    moralSelf    match        m     m   1 0.4922
## 4564  immoralSelf  immoralSelf    match        m     m   1 0.6482
## 4565    moralSelf    moralSelf    match        m     m   1 0.4764
## 4566    moralSelf    moralSelf mismatch        n     m   0 0.4926
## 4567   moralOther   moralOther mismatch        n     n   1 0.5367
## 4568   moralOther   moralOther    match        m     m   1 0.5407
## 4569    moralSelf    moralSelf mismatch        n     n   1 0.7590
## 4570  immoralSelf  immoralSelf    match        m     m   1 0.5170
## 4571    moralSelf    moralSelf    match        m     m   1 0.5091
## 4572   moralOther   moralOther    match        m     m   1 0.4932
## 4573  immoralSelf  immoralSelf mismatch        n     n   1 0.6173
## 4574  immoralSelf  immoralSelf mismatch        n     n   1 0.7235
## 4575 immoralOther immoralOther    match        m     n   0 0.5537
## 4576 immoralOther immoralOther    match        m     n   0 0.5697
## 4577   moralOther   moralOther mismatch        n     n   1 0.5819
## 4578  immoralSelf  immoralSelf mismatch        n     n   1 0.5759
## 4579    moralSelf    moralSelf mismatch        n     n   1 0.6921
## 4580  immoralSelf  immoralSelf    match        m     m   1 0.5842
## 4581 immoralOther immoralOther    match        m     m   1 0.6503
## 4582 immoralOther immoralOther mismatch        n     n   1 0.6865
## 4583   moralOther   moralOther    match        m     m   1 0.5325
## 4584   moralOther   moralOther mismatch        n     n   1 0.5127
## 4585    moralSelf    moralSelf mismatch        n     n   1 0.6409
## 4586   moralOther   moralOther    match        m     m   1 0.4949
## 4587    moralSelf    moralSelf    match        m     n   0 0.6990
## 4588  immoralSelf  immoralSelf    match        m     m   1 0.7572
## 4589   moralOther   moralOther mismatch        n     n   1 0.5714
## 4590  immoralSelf  immoralSelf    match        m     m   1 0.5994
## 4591   moralOther   moralOther mismatch        n     n   1 0.6536
## 4592   moralOther   moralOther    match        m     m   1 0.5856
## 4593 immoralOther immoralOther    match        m     m   1 0.4758
## 4594 immoralOther immoralOther mismatch        n     n   1 0.5719
## 4595    moralSelf    moralSelf mismatch        n     n   1 0.6240
## 4596  immoralSelf  immoralSelf mismatch        n     n   1 0.7621
## 4597    moralSelf    moralSelf    match        m     m   1 0.5283
## 4598    moralSelf    moralSelf mismatch        n     n   1 0.5804
## 4599    moralSelf    moralSelf    match        m     m   1 0.8145
## 4600  immoralSelf  immoralSelf mismatch        n     n   1 0.6587
## 4601  immoralSelf  immoralSelf mismatch        n     m   0 1.0208
## 4602   moralOther   moralOther    match        m     m   1 0.6870
## 4603   moralOther   moralOther mismatch        n     n   1 0.6450
## 4604 immoralOther immoralOther mismatch        n     n   1 0.6432
## 4605 immoralOther immoralOther    match        m     m   1 0.6173
## 4606 immoralOther immoralOther mismatch        n     n   1 0.5954
## 4607 immoralOther immoralOther    match        m     m   1 0.5235
## 4608  immoralSelf  immoralSelf    match        m     m   1 0.6576
## 4609   moralOther   moralOther mismatch        n     n   1 0.6437
## 4610    moralSelf    moralSelf    match        m     m   1 0.6260
## 4611 immoralOther immoralOther mismatch        n     n   1 0.9560
## 4612  immoralSelf  immoralSelf mismatch        n     n   1 0.6201
## 4613  immoralSelf  immoralSelf mismatch        n     n   1 0.9183
## 4614    moralSelf    moralSelf mismatch        n     n   1 0.6444
## 4615    moralSelf    moralSelf    match        m     m   1 0.4905
## 4616   moralOther   moralOther mismatch        n     n   1 0.5347
## 4617   moralOther   moralOther    match        m     m   1 0.5948
## 4618   moralOther   moralOther mismatch        n     n   1 0.6189
## 4619    moralSelf    moralSelf mismatch        n     n   1 0.5490
## 4620 immoralOther immoralOther    match        m     m   1 0.6291
## 4621   moralOther   moralOther    match        m     m   1 0.5332
## 4622  immoralSelf  immoralSelf    match        m     m   1 0.6414
## 4623 immoralOther immoralOther    match        m     m   1 0.5915
## 4624 immoralOther immoralOther mismatch        n     n   1 0.5036
## 4625 immoralOther immoralOther    match        m     m   1 0.5457
## 4626    moralSelf    moralSelf mismatch        n     n   1 0.6358
## 4627  immoralSelf  immoralSelf    match        m     m   1 0.6359
## 4628    moralSelf    moralSelf    match        m     m   1 0.6601
## 4629  immoralSelf  immoralSelf mismatch        n     n   1 0.6542
## 4630 immoralOther immoralOther mismatch        n     n   1 0.6424
## 4631  immoralSelf  immoralSelf    match        m     m   1 0.5284
## 4632   moralOther   moralOther    match        m     m   1 0.5086
## 4633  immoralSelf  immoralSelf    match        m     m   1 0.6016
## 4634   moralOther   moralOther    match        m     m   1 0.4876
## 4635  immoralSelf  immoralSelf mismatch        n     n   1 0.5397
## 4636   moralOther   moralOther mismatch        n     n   1 0.9419
## 4637  immoralSelf  immoralSelf    match        m     m   1 0.7440
## 4638  immoralSelf  immoralSelf    match        m     m   1 0.5941
## 4639  immoralSelf  immoralSelf mismatch        n     n   1 0.5542
## 4640    moralSelf    moralSelf    match        m     m   1 0.5063
## 4641    moralSelf    moralSelf mismatch        n     n   1 0.5565
## 4642 immoralOther immoralOther    match        m     m   1 0.6366
## 4643 immoralOther immoralOther    match        m     m   1 0.5007
## 4644    moralSelf    moralSelf mismatch        n     n   1 0.7508
## 4645    moralSelf    moralSelf mismatch        n     n   1 0.6790
## 4646   moralOther   moralOther mismatch        n     n   1 0.7251
## 4647  immoralSelf  immoralSelf mismatch        n     n   1 0.6032
## 4648    moralSelf    moralSelf    match        m     m   1 0.5135
## 4649 immoralOther immoralOther mismatch        n     n   1 0.5374
## 4650   moralOther   moralOther    match        m     m   1 0.4355
## 4651 immoralOther immoralOther mismatch        n     n   1 0.5397
## 4652    moralSelf    moralSelf    match        m     m   1 0.6058
## 4653   moralOther   moralOther mismatch        n     n   1 0.4839
## 4654 immoralOther immoralOther mismatch        n     n   1 0.5220
## 4655   moralOther   moralOther    match        m     m   1 0.5522
## 4656 immoralOther immoralOther    match        m     m   1 0.7203
## 4657 immoralOther immoralOther    match        m     m   1 0.6464
## 4658    moralSelf    moralSelf    match        m     m   1 0.5565
## 4659 immoralOther immoralOther mismatch        n     n   1 0.5887
## 4660   moralOther   moralOther mismatch        n     n   1 0.5247
## 4661   moralOther   moralOther    match        m     m   1 0.5389
## 4662  immoralSelf  immoralSelf mismatch        n     n   1 0.6931
## 4663   moralOther   moralOther    match        m     m   1 0.5051
## 4664  immoralSelf  immoralSelf    match        m     m   1 0.5872
## 4665    moralSelf    moralSelf mismatch        n     n   1 0.6193
## 4666 immoralOther immoralOther    match        m     m   1 0.6295
## 4667   moralOther   moralOther    match        m     m   1 0.5096
## 4668    moralSelf    moralSelf    match        m     m   1 0.4937
## 4669    moralSelf    moralSelf mismatch        n     n   1 0.5378
## 4670   moralOther   moralOther mismatch        n     n   1 0.6039
## 4671 immoralOther immoralOther    match        m     m   1 0.5620
## 4672    moralSelf    moralSelf mismatch        n     n   1 0.5282
## 4673    moralSelf    moralSelf    match        m     m   1 0.5003
## 4674  immoralSelf  immoralSelf    match        m     m   1 0.5904
## 4675 immoralOther immoralOther mismatch        n     n   1 0.6625
## 4676 immoralOther immoralOther mismatch        n     n   1 0.5766
## 4677  immoralSelf  immoralSelf mismatch        n     n   1 0.5489
## 4678   moralOther   moralOther mismatch        n     n   1 0.5809
## 4679  immoralSelf  immoralSelf mismatch        n     n   1 0.5270
## 4680  immoralSelf  immoralSelf    match        m     m   1 0.9811
## 4681   moralOther   moralOther    match        m     m   1 0.6829
## 4682    moralSelf    moralSelf    match        m     m   1 0.6010
## 4683  immoralSelf  immoralSelf mismatch        n     n   1 0.6711
## 4684 immoralOther immoralOther mismatch        n     n   1 0.7351
## 4685   moralOther   moralOther    match        m     m   1 0.4434
## 4686    moralSelf    moralSelf mismatch        n     n   1 0.8095
## 4687 immoralOther immoralOther mismatch        n     n   1 0.5416
## 4688   moralOther   moralOther mismatch        n     n   1 0.6077
## 4689  immoralSelf  immoralSelf    match        m     m   1 0.6999
## 4690 immoralOther immoralOther mismatch        n     n   1 0.5260
## 4691    moralSelf    moralSelf mismatch        n     n   1 0.8041
## 4692    moralSelf    moralSelf    match        m     m   1 0.6342
## 4693    moralSelf    moralSelf mismatch        n     n   1 0.6344
## 4694  immoralSelf  immoralSelf    match        m     m   1 0.5545
## 4695   moralOther   moralOther    match        m     m   1 0.7386
## 4696  immoralSelf  immoralSelf mismatch        n     n   1 0.7167
## 4697 immoralOther immoralOther    match        m     m   1 0.6288
## 4698   moralOther   moralOther mismatch        n     n   1 0.5570
## 4699  immoralSelf  immoralSelf mismatch        n     n   1 0.7332
## 4700   moralOther   moralOther mismatch        n     n   1 0.6653
## 4701 immoralOther immoralOther    match        m     m   1 0.5793
## 4702    moralSelf    moralSelf    match        m     m   1 0.5814
## 4703 immoralOther immoralOther    match        m     m   1 0.5756
## 4704  immoralSelf  immoralSelf    match        m     m   1 0.8037
## 4705  immoralSelf  immoralSelf    match        m     m   1 0.5858
## 4706   moralOther   moralOther mismatch        n     n   1 0.8599
## 4707    moralSelf    moralSelf mismatch        n     n   1 0.8221
## 4708   moralOther   moralOther mismatch        n     n   1 0.6083
## 4709    moralSelf    moralSelf    match        m     m   1 0.5003
## 4710 immoralOther immoralOther mismatch        n     n   1 0.8385
## 4711 immoralOther immoralOther    match        m     m   1 0.5886
## 4712  immoralSelf  immoralSelf    match        m     m   1 0.6448
## 4713    moralSelf    moralSelf    match        m     m   1 0.4669
## 4714  immoralSelf  immoralSelf mismatch        n     n   1 0.5969
## 4715    moralSelf    moralSelf mismatch        n     m   0 0.5090
## 4716    moralSelf    moralSelf mismatch        n     n   1 0.6612
## 4717 immoralOther immoralOther mismatch        n     n   1 0.6233
## 4718  immoralSelf  immoralSelf mismatch        n     n   1 0.7854
## 4719   moralOther   moralOther mismatch        n     n   1 0.4996
## 4720 immoralOther immoralOther    match        m     m   1 0.6216
## 4721  immoralSelf  immoralSelf mismatch        n     n   1 0.5998
## 4722   moralOther   moralOther    match        m     m   1 0.6139
## 4723  immoralSelf  immoralSelf    match        m     m   1 0.6500
## 4724   moralOther   moralOther    match        m     n   0 0.5501
## 4725 immoralOther immoralOther mismatch        n     n   1 0.5903
## 4726    moralSelf    moralSelf    match        m     m   1 0.5184
## 4727   moralOther   moralOther    match        m     m   1 0.8049
## 4728 immoralOther immoralOther    match        m     m   1 0.5867
## 4729 immoralOther immoralOther mismatch        n     n   1 0.5917
## 4730   moralOther   moralOther mismatch        n     n   1 0.5838
## 4731   moralOther   moralOther mismatch        n     n   1 0.6099
## 4732   moralOther   moralOther    match        m     m   1 0.4461
## 4733    moralSelf    moralSelf mismatch        n     m   0 0.4842
## 4734 immoralOther immoralOther    match        m     m   1 0.6262
## 4735   moralOther   moralOther    match        m     m   1 0.5404
## 4736  immoralSelf  immoralSelf mismatch        n     n   1 0.5106
## 4737 immoralOther immoralOther    match        m     m   1 0.6466
## 4738 immoralOther immoralOther    match        m     m   1 0.6768
## 4739  immoralSelf  immoralSelf mismatch        n     n   1 0.5569
## 4740    moralSelf    moralSelf mismatch        n     n   1 0.8050
## 4741  immoralSelf  immoralSelf    match        m     m   1 0.6671
## 4742    moralSelf    moralSelf    match        m     m   1 0.5074
## 4743    moralSelf    moralSelf mismatch        n     n   1 0.6194
## 4744  immoralSelf  immoralSelf mismatch        n     n   1 0.5735
## 4745   moralOther   moralOther mismatch        n     n   1 0.5376
## 4746    moralSelf    moralSelf    match        m     m   1 0.5238
## 4747    moralSelf    moralSelf    match        m     m   1 0.6819
## 4748  immoralSelf  immoralSelf    match        m     m   1 0.7440
## 4749 immoralOther immoralOther mismatch        n     n   1 0.5621
## 4750 immoralOther immoralOther mismatch        n     n   1 0.5682
## 4751   moralOther   moralOther    match        m     m   1 0.5563
## 4752  immoralSelf  immoralSelf    match        m     m   1 0.6605
## 4753    moralSelf    moralSelf    match        m     m   1 0.5346
## 4754  immoralSelf  immoralSelf mismatch        n     n   1 0.6587
## 4755  immoralSelf  immoralSelf    match        m     m   1 0.7408
## 4756 immoralOther immoralOther    match        m     m   1 0.5750
## 4757    moralSelf    moralSelf mismatch        n     n   1 0.9151
## 4758 immoralOther immoralOther mismatch        n     n   1 0.7792
## 4759  immoralSelf  immoralSelf mismatch        n     m   0 0.7574
## 4760 immoralOther immoralOther    match        m     m   1 0.5394
## 4761  immoralSelf  immoralSelf    match        m     m   1 0.7416
## 4762   moralOther   moralOther mismatch        n     n   1 0.5758
## 4763 immoralOther immoralOther mismatch        n     n   1 0.6198
## 4764   moralOther   moralOther mismatch        n     n   1 0.8380
## 4765 immoralOther immoralOther    match        m     m   1 0.5641
## 4766   moralOther   moralOther mismatch        n     n   1 0.7062
## 4767    moralSelf    moralSelf    match        m     m   1 0.5804
## 4768    moralSelf    moralSelf    match        m     m   1 0.6625
## 4769   moralOther   moralOther    match        m     n   0 0.5526
## 4770 immoralOther immoralOther mismatch        n     n   1 0.6327
## 4771    moralSelf    moralSelf mismatch        n     n   1 0.8089
## 4772    moralSelf    moralSelf mismatch        n     n   1 0.6049
## 4773  immoralSelf  immoralSelf mismatch        n     n   1 0.6111
## 4774   moralOther   moralOther    match        m     m   1 0.5273
## 4775   moralOther   moralOther    match        m     m   1 0.6914
## 4776  immoralSelf  immoralSelf    match        m     m   1 0.7354
## 4777    moralSelf    moralSelf mismatch        n     n   1 0.5964
## 4778  immoralSelf  immoralSelf    match        m     m   1 0.5484
## 4779  immoralSelf  immoralSelf mismatch        n     n   1 0.5165
## 4780  immoralSelf  immoralSelf    match        m     m   1 0.6207
## 4781   moralOther   moralOther mismatch        n     m   0 0.4229
## 4782 immoralOther immoralOther    match        m     m   1 0.5769
## 4783  immoralSelf  immoralSelf mismatch        n     n   1 0.5971
## 4784  immoralSelf  immoralSelf    match        m     m   1 0.6831
## 4785    moralSelf    moralSelf    match        m     m   1 0.5293
## 4786   moralOther   moralOther    match        m     m   1 0.5734
## 4787    moralSelf    moralSelf    match        m     m   1 0.7055
## 4788 immoralOther immoralOther mismatch        n     n   1 0.6437
## 4789    moralSelf    moralSelf mismatch        n     n   1 0.7378
## 4790   moralOther   moralOther mismatch        n     n   1 0.6519
## 4791   moralOther   moralOther mismatch        n     n   1 0.7041
## 4792    moralSelf    moralSelf    match        m     m   1 0.5142
## 4793 immoralOther immoralOther mismatch        n     n   1 0.6182
## 4794   moralOther   moralOther    match        m     m   1 0.6364
## 4795  immoralSelf  immoralSelf mismatch        n     n   1 0.5887
## 4796 immoralOther immoralOther mismatch        n     n   1 0.5086
## 4797 immoralOther immoralOther    match        m     n   0 0.5407
## 4798   moralOther   moralOther    match        m     m   1 0.6389
## 4799 immoralOther immoralOther    match        m     m   1 0.5490
## 4800    moralSelf    moralSelf mismatch        n     n   1 0.6290
## 4801 immoralOther immoralOther mismatch        n     n   1 0.5332
## 4802   moralOther   moralOther mismatch        n     n   1 0.7933
## 4803  immoralSelf  immoralSelf mismatch        n     n   1 0.5135
## 4804    moralSelf    moralSelf mismatch        n     n   1 0.8016
## 4805  immoralSelf  immoralSelf    match        m     m   1 0.5998
## 4806  immoralSelf  immoralSelf mismatch        n     n   1 0.5259
## 4807    moralSelf    moralSelf mismatch        n     n   1 0.6599
## 4808 immoralOther immoralOther mismatch        n     n   1 0.7901
## 4809  immoralSelf  immoralSelf    match        m     m   1 0.6302
## 4810 immoralOther immoralOther    match        m     m   1 0.6864
## 4811  immoralSelf  immoralSelf mismatch        n     n   1 0.5485
## 4812   moralOther   moralOther    match        m     m   1 0.4526
## 4813   moralOther   moralOther mismatch        n     n   1 0.5287
## 4814    moralSelf    moralSelf    match        m     m   1 0.5308
## 4815  immoralSelf  immoralSelf    match        m     m   1 0.8230
## 4816 immoralOther immoralOther    match        m     m   1 0.6011
## 4817    moralSelf    moralSelf mismatch        n     n   1 0.6952
## 4818 immoralOther immoralOther    match        m     m   1 0.6033
## 4819 immoralOther immoralOther mismatch        n     n   1 0.5454
## 4820   moralOther   moralOther    match        m     m   1 0.4915
## 4821   moralOther   moralOther    match        m     m   1 0.5597
## 4822    moralSelf    moralSelf    match        m     m   1 0.5818
## 4823   moralOther   moralOther mismatch        n     n   1 0.6799
## 4824    moralSelf    moralSelf    match        m     m   1 0.5181
## 4825    moralSelf    moralSelf mismatch        n     n   1 0.5084
## 4826   moralOther   moralOther    match        m     m   1 0.4205
## 4827  immoralSelf  immoralSelf    match        m     m   1 0.5746
## 4828 immoralOther immoralOther    match        m     m   1 0.5868
## 4829 immoralOther immoralOther mismatch        n     n   1 0.5809
## 4830 immoralOther immoralOther    match        m     m   1 0.5270
## 4831 immoralOther immoralOther mismatch        n     n   1 0.5731
## 4832    moralSelf    moralSelf mismatch        n     m   0 0.4573
## 4833  immoralSelf  immoralSelf    match        m     m   1 0.7073
## 4834   moralOther   moralOther mismatch        n     n   1 0.5875
## 4835  immoralSelf  immoralSelf mismatch        n     n   1 0.5796
## 4836  immoralSelf  immoralSelf mismatch        n     n   1 0.5257
## 4837   moralOther   moralOther    match        m     m   1 0.5318
## 4838   moralOther   moralOther mismatch        n     n   1 0.5359
## 4839  immoralSelf  immoralSelf    match        m     m   1 0.6120
## 4840    moralSelf    moralSelf    match        m     m   1 0.6403
## 4841    moralSelf    moralSelf    match        m     m   1 0.7903
## 4842   moralOther   moralOther mismatch        n     n   1 0.6545
## 4843    moralSelf    moralSelf    match        m     m   1 0.4666
## 4844   moralOther   moralOther    match        m     m   1 0.4847
## 4845 immoralOther immoralOther    match        m     m   1 0.6108
## 4846    moralSelf    moralSelf mismatch        n     m   0 0.4629
## 4847 immoralOther immoralOther mismatch        n     n   1 1.0170
## 4848  immoralSelf  immoralSelf mismatch        n     n   1 0.6771
## 4849    moralSelf    moralSelf mismatch        n     n   1 0.6273
## 4850  immoralSelf  immoralSelf mismatch        n     n   1 0.6274
## 4851   moralOther   moralOther    match        m     m   1 0.4760
## 4852  immoralSelf  immoralSelf    match        m     m   1 0.6117
## 4853 immoralOther immoralOther mismatch        n     n   1 0.5997
## 4854   moralOther   moralOther    match        m     m   1 0.6619
## 4855 immoralOther immoralOther    match        m     m   1 0.6160
## 4856    moralSelf    moralSelf mismatch        n     n   1 0.7641
## 4857    moralSelf    moralSelf mismatch        n     n   1 0.7223
## 4858 immoralOther immoralOther mismatch        n     n   1 0.8561
## 4859 immoralOther immoralOther    match        m     m   1 0.6605
## 4860  immoralSelf  immoralSelf    match        m     m   1 0.8547
## 4861    moralSelf    moralSelf    match        m     m   1 0.6348
## 4862   moralOther   moralOther    match        m     m   1 0.7149
## 4863  immoralSelf  immoralSelf mismatch        n     n   1 0.6750
## 4864 immoralOther immoralOther mismatch        n     n   1 0.8591
## 4865    moralSelf    moralSelf    match        m     m   1 0.6453
## 4866   moralOther   moralOther mismatch        n     n   1 0.6676
## 4867    moralSelf    moralSelf    match        m     m   1 0.5396
## 4868 immoralOther immoralOther    match        m     m   1 0.5737
## 4869  immoralSelf  immoralSelf    match        m     m   1 0.5218
## 4870   moralOther   moralOther mismatch        n     n   1 0.5199
## 4871   moralOther   moralOther mismatch        n     n   1 0.5740
## 4872  immoralSelf  immoralSelf mismatch        n     n   1 0.6021
## 4873 immoralOther immoralOther    match        m     m   1 0.5502
## 4874  immoralSelf  immoralSelf mismatch        n     n   1 0.5343
## 4875 immoralOther immoralOther mismatch        n     n   1 0.5784
## 4876    moralSelf    moralSelf mismatch        n     n   1 0.6606
## 4877   moralOther   moralOther mismatch        n     n   1 0.6946
## 4878    moralSelf    moralSelf    match        m     m   1 0.5789
## 4879 immoralOther immoralOther mismatch        n     n   1 0.5249
## 4880    moralSelf    moralSelf mismatch        n     n   1 0.6190
## 4881   moralOther   moralOther    match        m     m   1 0.5892
## 4882    moralSelf    moralSelf    match        m     n   0 0.5013
## 4883 immoralOther immoralOther    match        m     m   1 0.5514
## 4884    moralSelf    moralSelf mismatch        n     n   1 0.5595
## 4885   moralOther   moralOther mismatch        n     n   1 0.5976
## 4886   moralOther   moralOther    match        m     m   1 0.5718
## 4887 immoralOther immoralOther    match        m     m   1 0.5839
## 4888  immoralSelf  immoralSelf mismatch        n     n   1 0.4660
## 4889 immoralOther immoralOther mismatch        n     n   1 0.7721
## 4890    moralSelf    moralSelf    match        m     m   1 0.5843
## 4891  immoralSelf  immoralSelf mismatch        n     n   1 0.5783
## 4892  immoralSelf  immoralSelf    match        m     m   1 0.6685
## 4893  immoralSelf  immoralSelf    match        m     m   1 0.6206
## 4894   moralOther   moralOther mismatch        n  <NA>  -1 1.1430
## 4895  immoralSelf  immoralSelf    match        m     m   1 0.6488
## 4896   moralOther   moralOther    match        m     m   1 0.6050
## 4897 immoralOther immoralOther mismatch        n     n   1 0.7311
## 4898  immoralSelf  immoralSelf    match        m     m   1 0.5752
## 4899 immoralOther immoralOther mismatch        n     n   1 0.6833
## 4900 immoralOther immoralOther mismatch        n     n   1 0.6975
## 4901   moralOther   moralOther    match        m     m   1 0.6997
## 4902   moralOther   moralOther mismatch        n     n   1 0.5418
## 4903  immoralSelf  immoralSelf mismatch        n     n   1 0.6639
## 4904    moralSelf    moralSelf mismatch        n     n   1 0.6100
## 4905    moralSelf    moralSelf mismatch        n     n   1 0.5421
## 4906 immoralOther immoralOther    match        m     m   1 0.5842
## 4907    moralSelf    moralSelf mismatch        n     n   1 0.6203
## 4908 immoralOther immoralOther    match        m     m   1 0.5745
## 4909    moralSelf    moralSelf    match        m     m   1 0.7226
## 4910  immoralSelf  immoralSelf mismatch        n     n   1 0.6566
## 4911  immoralSelf  immoralSelf    match        m     m   1 0.6927
## 4912    moralSelf    moralSelf    match        m     m   1 0.4890
## 4913   moralOther   moralOther    match        m     m   1 0.7091
## 4914    moralSelf    moralSelf    match        m     m   1 0.4772
## 4915   moralOther   moralOther mismatch        n     n   1 0.5474
## 4916  immoralSelf  immoralSelf    match        m     m   1 0.5235
## 4917 immoralOther immoralOther    match        m     m   1 0.7535
## 4918   moralOther   moralOther    match        m     m   1 0.5058
## 4919  immoralSelf  immoralSelf mismatch        n     n   1 0.6278
## 4920   moralOther   moralOther mismatch        n     n   1 0.6679
## 4921    moralSelf    moralSelf mismatch        n     n   1 0.5358
## 4922   moralOther   moralOther mismatch        n     n   1 0.5793
## 4923   moralOther   moralOther    match        m     m   1 0.4716
## 4924  immoralSelf  immoralSelf mismatch        n     n   1 0.4379
## 4925   moralOther   moralOther    match        m     m   1 0.5912
## 4926  immoralSelf  immoralSelf    match        m     m   1 0.3918
## 4927 immoralOther immoralOther    match        m     n   0 0.5124
## 4928 immoralOther immoralOther    match        m     m   1 0.4673
## 4929  immoralSelf  immoralSelf mismatch        n     n   1 0.4293
## 4930  immoralSelf  immoralSelf    match        m     m   1 0.4026
## 4931    moralSelf    moralSelf    match        m     n   0 0.5593
## 4932    moralSelf    moralSelf mismatch        n     m   0 0.4992
## 4933   moralOther   moralOther    match        m     m   1 0.5058
## 4934   moralOther   moralOther mismatch        n     n   1 0.5648
## 4935  immoralSelf  immoralSelf mismatch        n     m   0 0.3888
## 4936 immoralOther immoralOther mismatch        n     m   0 0.4934
## 4937  immoralSelf  immoralSelf    match        m     m   1 0.4121
## 4938   moralOther   moralOther mismatch        n     n   1 0.5171
## 4939    moralSelf    moralSelf mismatch        n     n   1 0.7803
## 4940 immoralOther immoralOther mismatch        n     m   0 0.5606
## 4941 immoralOther immoralOther    match        m     m   1 0.7004
## 4942    moralSelf    moralSelf    match        m     m   1 0.6351
## 4943 immoralOther immoralOther mismatch        n     n   1 0.5967
## 4944    moralSelf    moralSelf    match        m     m   1 0.5011
## 4945    moralSelf    moralSelf    match        m     m   1 0.6198
## 4946 immoralOther immoralOther    match        m     m   1 0.6129
## 4947  immoralSelf  immoralSelf    match        m     m   1 0.4379
## 4948  immoralSelf  immoralSelf mismatch        n     m   0 0.5115
## 4949   moralOther   moralOther    match        m     m   1 0.4503
## 4950   moralOther   moralOther mismatch        n     m   0 0.4240
## 4951  immoralSelf  immoralSelf mismatch        n     m   0 0.3814
## 4952 immoralOther immoralOther mismatch        n     m   0 0.4576
## 4953 immoralOther immoralOther    match        m     m   1 0.6276
## 4954  immoralSelf  immoralSelf    match        m     m   1 0.5289
## 4955  immoralSelf  immoralSelf mismatch        n     n   1 0.5163
## 4956 immoralOther immoralOther mismatch        n     m   0 0.4835
## 4957   moralOther   moralOther    match        m     m   1 0.4219
## 4958  immoralSelf  immoralSelf    match        m     m   1 0.4830
## 4959    moralSelf    moralSelf mismatch        n     m   0 0.6295
## 4960   moralOther   moralOther    match        m     m   1 0.5149
## 4961   moralOther   moralOther mismatch        n     m   0 0.4259
## 4962    moralSelf    moralSelf mismatch        n     m   0 0.5512
## 4963   moralOther   moralOther mismatch        n     n   1 0.7711
## 4964 immoralOther immoralOther    match        m     m   1 0.5552
## 4965    moralSelf    moralSelf    match        m     m   1 0.7430
## 4966    moralSelf    moralSelf    match        m     m   1 0.5426
## 4967    moralSelf    moralSelf mismatch        n     m   0 0.5823
## 4968 immoralOther immoralOther mismatch        n     n   1 0.6106
## 4969   moralOther   moralOther    match        m     m   1 0.5236
## 4970  immoralSelf  immoralSelf    match        m     m   1 0.5628
## 4971  immoralSelf  immoralSelf mismatch        n     n   1 0.4987
## 4972    moralSelf    moralSelf    match        m     m   1 0.5535
## 4973  immoralSelf  immoralSelf    match        m     m   1 0.4053
## 4974  immoralSelf  immoralSelf    match        m     m   1 0.4342
## 4975 immoralOther immoralOther    match        m     n   0 0.5477
## 4976   moralOther   moralOther mismatch        n     n   1 0.5632
## 4977   moralOther   moralOther    match        m     m   1 0.3554
## 4978 immoralOther immoralOther mismatch        n     m   0 0.6072
## 4979  immoralSelf  immoralSelf mismatch        n     m   0 0.3962
## 4980    moralSelf    moralSelf mismatch        n     m   0 0.4205
## 4981 immoralOther immoralOther    match        m     m   1 0.6696
## 4982   moralOther   moralOther mismatch        n     m   0 0.4278
## 4983    moralSelf    moralSelf mismatch        n     m   0 0.5210
## 4984 immoralOther immoralOther    match        m     m   1 0.6402
## 4985   moralOther   moralOther mismatch        n     m   0 0.6817
## 4986    moralSelf    moralSelf mismatch        n     n   1 0.7642
## 4987  immoralSelf  immoralSelf mismatch        n     n   1 0.6122
## 4988    moralSelf    moralSelf    match        m     m   1 0.4613
## 4989 immoralOther immoralOther mismatch        n     m   0 0.6472
## 4990    moralSelf    moralSelf    match        m     m   1 0.5250
## 4991   moralOther   moralOther    match        m     m   1 0.5800
## 4992 immoralOther immoralOther mismatch        n     m   0 0.5285
## 4993 immoralOther immoralOther    match        m     n   0 0.5959
## 4994   moralOther   moralOther mismatch        n     n   1 0.5885
## 4995    moralSelf    moralSelf    match        m     m   1 0.4531
## 4996   moralOther   moralOther mismatch        n     n   1 0.5548
## 4997   moralOther   moralOther    match        m     m   1 0.4787
## 4998  immoralSelf  immoralSelf    match        m     n   0 0.4448
## 4999 immoralOther immoralOther mismatch        n     m   0 0.4666
## 5000  immoralSelf  immoralSelf    match        m     m   1 0.4605
## 5001  immoralSelf  immoralSelf mismatch        n     n   1 0.5266
## 5002    moralSelf    moralSelf    match        m     m   1 0.4179
## 5003  immoralSelf  immoralSelf mismatch        n     n   1 0.4909
## 5004 immoralOther immoralOther    match        m     m   1 0.4614
## 5005    moralSelf    moralSelf mismatch        n     m   0 0.4314
## 5006  immoralSelf  immoralSelf    match        m     m   1 0.3005
## 5007   moralOther   moralOther mismatch        n     n   1 0.4434
## 5008    moralSelf    moralSelf mismatch        n     m   0 0.4528
## 5009 immoralOther immoralOther mismatch        n     n   1 0.6504
## 5010    moralSelf    moralSelf    match        m     m   1 0.4962
## 5011    moralSelf    moralSelf mismatch        n     n   1 0.4868
## 5012   moralOther   moralOther    match        m     m   1 0.5533
## 5013  immoralSelf  immoralSelf mismatch        n     n   1 0.5092
## 5014 immoralOther immoralOther    match        m     n   0 0.5761
## 5015 immoralOther immoralOther mismatch        n     m   0 0.5004
## 5016   moralOther   moralOther    match        m     m   1 0.4671
## 5017   moralOther   moralOther    match        m     n   0 0.4609
## 5018   moralOther   moralOther mismatch        n     n   1 0.6070
## 5019    moralSelf    moralSelf    match        m     m   1 0.5158
## 5020  immoralSelf  immoralSelf    match        m     m   1 0.4750
## 5021  immoralSelf  immoralSelf    match        m     m   1 0.5610
## 5022 immoralOther immoralOther mismatch        n     n   1 0.4930
## 5023   moralOther   moralOther mismatch        n     n   1 0.6596
## 5024    moralSelf    moralSelf mismatch        n     n   1 0.8136
## 5025 immoralOther immoralOther mismatch        n     n   1 0.7027
## 5026   moralOther   moralOther    match        m     m   1 0.5494
## 5027  immoralSelf  immoralSelf mismatch        n     n   1 0.4770
## 5028    moralSelf    moralSelf mismatch        n     m   0 0.6431
## 5029 immoralOther immoralOther mismatch        n     n   1 0.4686
## 5030    moralSelf    moralSelf    match        m     m   1 0.5869
## 5031   moralOther   moralOther mismatch        n     n   1 0.4194
## 5032    moralSelf    moralSelf    match        m     m   1 0.4604
## 5033 immoralOther immoralOther    match        m     n   0 0.5025
## 5034    moralSelf    moralSelf mismatch        n     m   0 0.4691
## 5035  immoralSelf  immoralSelf    match        m     m   1 0.4593
## 5036   moralOther   moralOther    match        m     m   1 0.5493
## 5037 immoralOther immoralOther    match        m     m   1 0.4689
## 5038  immoralSelf  immoralSelf mismatch        n     n   1 0.5151
## 5039 immoralOther immoralOther    match        m     m   1 0.5222
## 5040  immoralSelf  immoralSelf mismatch        n     n   1 0.5054
## 5041 immoralOther immoralOther    match        m  <NA>  -1 1.0841
## 5042   moralOther   moralOther mismatch        n     n   1 0.5876
## 5043  immoralSelf  immoralSelf    match        m     m   1 0.4521
## 5044 immoralOther immoralOther mismatch        n     m   0 0.5379
## 5045   moralOther   moralOther mismatch        n     n   1 0.5411
## 5046   moralOther   moralOther    match        m     m   1 0.5088
## 5047 immoralOther immoralOther    match        m     m   1 0.3837
## 5048    moralSelf    moralSelf mismatch        n     m   0 0.5480
## 5049  immoralSelf  immoralSelf mismatch        n     n   1 0.5157
## 5050  immoralSelf  immoralSelf mismatch        n     m   0 0.4428
## 5051    moralSelf    moralSelf mismatch        n     m   0 0.5004
## 5052    moralSelf    moralSelf    match        m     n   0 0.4990
## 5053    moralSelf    moralSelf mismatch        n     n   1 0.7617
## 5054 immoralOther immoralOther mismatch        n     m   0 0.5496
## 5055    moralSelf    moralSelf    match        m     m   1 0.5092
## 5056  immoralSelf  immoralSelf    match        m     n   0 0.4080
## 5057 immoralOther immoralOther    match        m     n   0 0.7249
## 5058  immoralSelf  immoralSelf mismatch        n     n   1 0.5079
## 5059 immoralOther immoralOther mismatch        n     m   0 0.4229
## 5060   moralOther   moralOther    match        m     m   1 0.4362
## 5061  immoralSelf  immoralSelf    match        m     m   1 0.4457
## 5062   moralOther   moralOther    match        m     m   1 0.5155
## 5063   moralOther   moralOther mismatch        n     n   1 0.3946
## 5064    moralSelf    moralSelf    match        m     m   1 0.4751
## 5065    moralSelf    moralSelf    match        m     n   0 0.4252
## 5066   moralOther   moralOther mismatch        n     n   1 0.4465
## 5067  immoralSelf  immoralSelf    match        m     m   1 0.3163
## 5068   moralOther   moralOther mismatch        n     n   1 0.4033
## 5069 immoralOther immoralOther mismatch        n     m   0 0.4803
## 5070    moralSelf    moralSelf mismatch        n     m   0 0.6186
## 5071 immoralOther immoralOther    match        m     m   1 0.4437
## 5072   moralOther   moralOther    match        m     m   1 0.5011
## 5073 immoralOther immoralOther mismatch        n     n   1 0.4758
## 5074   moralOther   moralOther mismatch        n     m   0 0.4341
## 5075    moralSelf    moralSelf    match        m     m   1 0.5475
## 5076 immoralOther immoralOther    match        m     n   0 0.4672
## 5077  immoralSelf  immoralSelf mismatch        n     n   1 0.4531
## 5078  immoralSelf  immoralSelf    match        m     m   1 0.4427
## 5079    moralSelf    moralSelf mismatch        n     n   1 0.4603
## 5080   moralOther   moralOther    match        m     m   1 0.3422
## 5081   moralOther   moralOther    match        m     n   0 0.5579
## 5082  immoralSelf  immoralSelf    match        m     n   0 0.4017
## 5083  immoralSelf  immoralSelf mismatch        n     n   1 0.4625
## 5084    moralSelf    moralSelf    match        m     m   1 0.5045
## 5085  immoralSelf  immoralSelf mismatch        n     n   1 0.4195
## 5086 immoralOther immoralOther mismatch        n     m   0 0.5644
## 5087 immoralOther immoralOther    match        m     m   1 0.7405
## 5088    moralSelf    moralSelf mismatch        n     m   0 0.6760
## 5089    moralSelf    moralSelf mismatch        n     n   1 0.5863
## 5090   moralOther   moralOther    match        m     m   1 0.4429
## 5091 immoralOther immoralOther    match        m     n   0 0.6364
## 5092   moralOther   moralOther mismatch        n     n   1 0.5178
## 5093   moralOther   moralOther    match        m     m   1 0.3971
## 5094  immoralSelf  immoralSelf    match        m     m   1 0.3858
## 5095   moralOther   moralOther mismatch        n     m   0 0.4584
## 5096  immoralSelf  immoralSelf mismatch        n     n   1 0.5324
## 5097  immoralSelf  immoralSelf    match        m     m   1 0.4477
## 5098    moralSelf    moralSelf    match        m     m   1 0.4215
## 5099  immoralSelf  immoralSelf    match        m     m   1 0.4747
## 5100   moralOther   moralOther    match        m     m   1 0.4329
## 5101    moralSelf    moralSelf mismatch        n     m   0 0.4423
## 5102 immoralOther immoralOther mismatch        n     m   0 0.4439
## 5103 immoralOther immoralOther mismatch        n     n   1 0.5974
## 5104 immoralOther immoralOther mismatch        n     n   1 0.6102
## 5105    moralSelf    moralSelf    match        m     m   1 0.4992
## 5106    moralSelf    moralSelf mismatch        n     n   1 0.6179
## 5107  immoralSelf  immoralSelf mismatch        n     n   1 0.5309
## 5108  immoralSelf  immoralSelf mismatch        n     n   1 0.5343
## 5109    moralSelf    moralSelf    match        m     n   0 0.4977
## 5110   moralOther   moralOther mismatch        n     n   1 0.6164
## 5111 immoralOther immoralOther    match        m     m   1 0.5694
## 5112 immoralOther immoralOther    match        m     n   0 0.4735
## 5113  immoralSelf  immoralSelf    match        m     m   1 0.5462
## 5114 immoralOther immoralOther    match        m     m   1 0.6339
## 5115 immoralOther immoralOther    match        m     n   0 0.4513
## 5116 immoralOther immoralOther mismatch        n     m   0 0.2970
## 5117   moralOther   moralOther    match        m     m   1 0.4636
## 5118 immoralOther immoralOther mismatch        n     m   0 0.4095
## 5119  immoralSelf  immoralSelf mismatch        n     m   0 0.2827
## 5120   moralOther   moralOther mismatch        n     m   0 0.5411
## 5121   moralOther   moralOther mismatch        n     n   1 0.7567
## 5122  immoralSelf  immoralSelf mismatch        n     n   1 0.5365
## 5123    moralSelf    moralSelf mismatch        n     m   0 0.5080
## 5124  immoralSelf  immoralSelf    match        m     m   1 0.4229
## 5125 immoralOther immoralOther mismatch        n     n   1 0.6921
## 5126    moralSelf    moralSelf    match        m     m   1 0.5309
## 5127    moralSelf    moralSelf    match        m     m   1 0.6381
## 5128   moralOther   moralOther    match        m     m   1 0.5917
## 5129    moralSelf    moralSelf mismatch        n     n   1 0.6402
## 5130   moralOther   moralOther mismatch        n     m   0 0.4338
## 5131  immoralSelf  immoralSelf    match        m     m   1 0.3793
## 5132    moralSelf    moralSelf    match        m     n   0 0.6076
## 5133  immoralSelf  immoralSelf mismatch        n     n   1 0.4203
## 5134   moralOther   moralOther    match        m     m   1 0.4614
## 5135 immoralOther immoralOther    match        m     m   1 0.5674
## 5136    moralSelf    moralSelf mismatch        n     m   0 0.5034
## 5137  immoralSelf  immoralSelf    match        m     m   1 0.3544
## 5138 immoralOther immoralOther mismatch        n     m   0 0.4622
## 5139    moralSelf    moralSelf    match        m     m   1 0.3842
## 5140  immoralSelf  immoralSelf    match        m     m   1 0.3324
## 5141    moralSelf    moralSelf    match        m     m   1 0.5278
## 5142 immoralOther immoralOther mismatch        n     n   1 0.4512
## 5143  immoralSelf  immoralSelf    match        m     m   1 0.4329
## 5144    moralSelf    moralSelf mismatch        n     m   0 0.7144
## 5145   moralOther   moralOther mismatch        n     n   1 0.6693
## 5146  immoralSelf  immoralSelf mismatch        n     n   1 0.5315
## 5147   moralOther   moralOther    match        m     m   1 0.5750
## 5148   moralOther   moralOther    match        m     m   1 0.5872
## 5149   moralOther   moralOther mismatch        n     n   1 0.5719
## 5150  immoralSelf  immoralSelf mismatch        n     n   1 0.4999
## 5151    moralSelf    moralSelf    match        m     m   1 0.6825
## 5152    moralSelf    moralSelf mismatch        n     n   1 0.5650
## 5153 immoralOther immoralOther    match        m     m   1 0.7011
## 5154  immoralSelf  immoralSelf mismatch        n     n   1 0.4519
## 5155   moralOther   moralOther    match        m     m   1 0.4815
## 5156   moralOther   moralOther mismatch        n     n   1 0.5560
## 5157 immoralOther immoralOther mismatch        n     m   0 0.5119
## 5158 immoralOther immoralOther    match        m     m   1 0.5627
## 5159 immoralOther immoralOther    match        m     m   1 0.5227
## 5160    moralSelf    moralSelf mismatch        n     m   0 0.5539
## 5161    moralSelf    moralSelf    match        m     n   0 0.6303
## 5162  immoralSelf  immoralSelf    match        m     m   1 0.5476
## 5163 immoralOther immoralOther    match        m     m   1 0.5553
## 5164  immoralSelf  immoralSelf mismatch        n     n   1 0.5030
## 5165    moralSelf    moralSelf    match        m     m   1 0.5540
## 5166   moralOther   moralOther mismatch        n     n   1 0.4938
## 5167   moralOther   moralOther mismatch        n     n   1 0.7646
## 5168 immoralOther immoralOther mismatch        n     m   0 0.5647
## 5169   moralOther   moralOther    match        m     m   1 0.6768
## 5170  immoralSelf  immoralSelf mismatch        n     n   1 0.4670
## 5171 immoralOther immoralOther    match        m     m   1 0.5168
## 5172    moralSelf    moralSelf mismatch        n     m   0 0.6120
## 5173 immoralOther immoralOther    match        m     m   1 0.5170
## 5174   moralOther   moralOther mismatch        n     n   1 0.5482
## 5175  immoralSelf  immoralSelf    match        m     m   1 0.5558
## 5176    moralSelf    moralSelf mismatch        n     m   0 0.6557
## 5177  immoralSelf  immoralSelf    match        m     m   1 0.5094
## 5178   moralOther   moralOther    match        m     m   1 0.7523
## 5179 immoralOther immoralOther mismatch        n     n   1 0.5801
## 5180    moralSelf    moralSelf    match        m     m   1 0.4725
## 5181  immoralSelf  immoralSelf mismatch        n     n   1 0.4947
## 5182   moralOther   moralOther    match        m     m   1 0.4453
## 5183 immoralOther immoralOther mismatch        n     n   1 0.6191
## 5184    moralSelf    moralSelf mismatch        n     n   1 0.7961
## 5185    moralSelf    moralSelf    match        m     m   1 0.4607
## 5186   moralOther   moralOther mismatch        n     n   1 0.4387
## 5187 immoralOther immoralOther mismatch        n     n   1 0.4562
## 5188    moralSelf    moralSelf    match        m     m   1 0.4581
## 5189   moralOther   moralOther mismatch        n     n   1 0.4841
## 5190    moralSelf    moralSelf    match        m     m   1 0.4305
## 5191   moralOther   moralOther mismatch        n     n   1 0.4917
## 5192   moralOther   moralOther    match        m     m   1 0.5625
## 5193  immoralSelf  immoralSelf    match        m     m   1 0.4746
## 5194  immoralSelf  immoralSelf mismatch        n     n   1 0.4806
## 5195 immoralOther immoralOther mismatch        n     n   1 0.6031
## 5196  immoralSelf  immoralSelf mismatch        n     n   1 0.4640
## 5197    moralSelf    moralSelf mismatch        n     m   0 0.4259
## 5198   moralOther   moralOther    match        m     m   1 0.6072
## 5199  immoralSelf  immoralSelf mismatch        n     n   1 0.4441
## 5200    moralSelf    moralSelf mismatch        n     m   0 0.4817
## 5201  immoralSelf  immoralSelf    match        m     m   1 0.4841
## 5202 immoralOther immoralOther    match        m     m   1 0.5346
## 5203    moralSelf    moralSelf mismatch        n     m   0 0.4900
## 5204 immoralOther immoralOther mismatch        n     n   1 0.4924
## 5205  immoralSelf  immoralSelf    match        m     m   1 0.4111
## 5206 immoralOther immoralOther    match        m     n   0 0.4761
## 5207   moralOther   moralOther    match        m     m   1 0.4902
## 5208 immoralOther immoralOther    match        m     m   1 0.5247
## 5209 immoralOther immoralOther    match        m     m   1 0.4919
## 5210   moralOther   moralOther    match        m     m   1 0.5065
## 5211   moralOther   moralOther mismatch        n     m   0 0.5255
## 5212 immoralOther immoralOther mismatch        n     n   1 0.5806
## 5213 immoralOther immoralOther mismatch        n     n   1 0.6729
## 5214   moralOther   moralOther mismatch        n     n   1 0.9432
## 5215   moralOther   moralOther mismatch        n     n   1 0.7587
## 5216    moralSelf    moralSelf mismatch        n     n   1 0.7946
## 5217   moralOther   moralOther    match        m     m   1 0.5472
## 5218  immoralSelf  immoralSelf    match        m     m   1 0.4989
## 5219    moralSelf    moralSelf mismatch        n     n   1 0.5855
## 5220 immoralOther immoralOther    match        m     m   1 0.5460
## 5221  immoralSelf  immoralSelf mismatch        n     n   1 0.6977
## 5222 immoralOther immoralOther mismatch        n     n   1 0.5363
## 5223   moralOther   moralOther    match        m     m   1 0.5237
## 5224    moralSelf    moralSelf mismatch        n     n   1 0.5629
## 5225    moralSelf    moralSelf    match        m     m   1 0.5309
## 5226 immoralOther immoralOther    match        m     n   0 0.4783
## 5227  immoralSelf  immoralSelf mismatch        n     n   1 0.5245
## 5228  immoralSelf  immoralSelf    match        m     m   1 0.5317
## 5229  immoralSelf  immoralSelf    match        m     m   1 0.5591
## 5230    moralSelf    moralSelf    match        m     n   0 0.6670
## 5231  immoralSelf  immoralSelf mismatch        n     n   1 0.5169
## 5232    moralSelf    moralSelf    match        m     m   1 0.5639
## 5233  immoralSelf  immoralSelf    match        m     m   1 0.5133
## 5234   moralOther   moralOther mismatch        n     n   1 0.7802
## 5235  immoralSelf  immoralSelf mismatch        n     n   1 0.4805
## 5236    moralSelf    moralSelf mismatch        n     m   0 0.7231
## 5237   moralOther   moralOther mismatch        n     n   1 0.6981
## 5238 immoralOther immoralOther    match        m     m   1 0.6569
## 5239    moralSelf    moralSelf mismatch        n     n   1 0.6748
## 5240   moralOther   moralOther mismatch        n     m   0 0.5050
## 5241    moralSelf    moralSelf    match        m     m   1 0.7879
## 5242 immoralOther immoralOther mismatch        n     n   1 0.6525
## 5243  immoralSelf  immoralSelf mismatch        n     n   1 0.5941
## 5244   moralOther   moralOther    match        m     m   1 0.4988
## 5245 immoralOther immoralOther mismatch        n     n   1 0.5774
## 5246    moralSelf    moralSelf    match        m     m   1 0.4218
## 5247 immoralOther immoralOther    match        m     m   1 0.6110
## 5248   moralOther   moralOther    match        m     m   1 0.5321
## 5249   moralOther   moralOther    match        m     m   1 0.6154
## 5250 immoralOther immoralOther mismatch        n     n   1 0.7204
## 5251  immoralSelf  immoralSelf    match        m     m   1 0.4556
## 5252 immoralOther immoralOther    match        m     m   1 0.5733
## 5253    moralSelf    moralSelf mismatch        n     m   0 0.6375
## 5254  immoralSelf  immoralSelf mismatch        n     n   1 0.6387
## 5255  immoralSelf  immoralSelf    match        m     m   1 0.4484
## 5256    moralSelf    moralSelf    match        m     m   1 0.7102
## 5257  immoralSelf  immoralSelf    match        m     m   1 0.4972
## 5258   moralOther   moralOther mismatch        n     n   1 0.6157
## 5259 immoralOther immoralOther mismatch        n     n   1 0.6728
## 5260   moralOther   moralOther    match        m     m   1 0.4870
## 5261 immoralOther immoralOther mismatch        n     n   1 0.4894
## 5262 immoralOther immoralOther    match        m     m   1 0.5238
## 5263  immoralSelf  immoralSelf mismatch        n     n   1 0.4911
## 5264    moralSelf    moralSelf    match        m     m   1 0.6055
## 5265    moralSelf    moralSelf mismatch        n     n   1 0.5702
## 5266    moralSelf    moralSelf    match        m     m   1 0.4983
## 5267   moralOther   moralOther    match        m     m   1 0.5690
## 5268 immoralOther immoralOther mismatch        n     n   1 0.5058
## 5269  immoralSelf  immoralSelf    match        m     m   1 0.4120
## 5270    moralSelf    moralSelf    match        m     n   0 0.4770
## 5271   moralOther   moralOther mismatch        n     n   1 0.5632
## 5272  immoralSelf  immoralSelf mismatch        n     n   1 0.5553
## 5273   moralOther   moralOther    match        m     m   1 0.4151
## 5274  immoralSelf  immoralSelf    match        m     m   1 0.5041
## 5275 immoralOther immoralOther    match        m     m   1 0.7151
## 5276   moralOther   moralOther mismatch        n     n   1 0.6902
## 5277    moralSelf    moralSelf mismatch        n     m   0 0.5326
## 5278    moralSelf    moralSelf mismatch        n     n   1 0.8641
## 5279  immoralSelf  immoralSelf mismatch        n     m   0 0.4180
## 5280 immoralOther immoralOther    match        m     m   1 0.5550
## 5281 immoralOther immoralOther mismatch        n     n   1 0.4681
## 5282 immoralOther immoralOther    match        m     m   1 0.6424
## 5283  immoralSelf  immoralSelf    match        m     m   1 0.5639
## 5284    moralSelf    moralSelf mismatch        n     n   1 0.5960
## 5285   moralOther   moralOther    match        m     m   1 0.4846
## 5286  immoralSelf  immoralSelf mismatch        n     n   1 0.4631
## 5287   moralOther   moralOther    match        m     m   1 0.5049
## 5288   moralOther   moralOther mismatch        n     n   1 0.4359
## 5289  immoralSelf  immoralSelf    match        m     m   1 0.3254
## 5290    moralSelf    moralSelf    match        m     m   1 0.6606
## 5291    moralSelf    moralSelf    match        m     n   0 0.4385
## 5292 immoralOther immoralOther mismatch        n     n   1 0.7039
## 5293  immoralSelf  immoralSelf mismatch        n     m   0 0.3789
## 5294    moralSelf    moralSelf mismatch        n     n   1 0.5752
## 5295   moralOther   moralOther mismatch        n     m   0 0.4035
## 5296    moralSelf    moralSelf mismatch        n     m   0 0.4163
## 5297 immoralOther immoralOther    match        m     m   1 0.5613
## 5298   moralOther   moralOther mismatch        n     n   1 0.5492
## 5299 immoralOther immoralOther mismatch        n     n   1 0.6128
## 5300  immoralSelf  immoralSelf    match        m     m   1 0.4299
## 5301    moralSelf    moralSelf    match        m     m   1 0.4831
## 5302  immoralSelf  immoralSelf mismatch        n     n   1 0.5015
## 5303   moralOther   moralOther    match        m     m   1 0.5082
## 5304 immoralOther immoralOther    match        m     m   1 0.5671
## 5305 immoralOther immoralOther    match        m     m   1 0.6872
## 5306 immoralOther immoralOther mismatch        n     n   1 0.5936
## 5307  immoralSelf  immoralSelf    match        m     m   1 0.4342
## 5308    moralSelf    moralSelf    match        m     m   1 0.4198
## 5309 immoralOther immoralOther mismatch        n     m   0 0.5487
## 5310  immoralSelf  immoralSelf mismatch        n     n   1 0.5244
## 5311   moralOther   moralOther mismatch        n     m   0 0.4915
## 5312   moralOther   moralOther    match        m     m   1 0.6261
## 5313 immoralOther immoralOther mismatch        n     n   1 0.6073
## 5314   moralOther   moralOther mismatch        n     n   1 0.5482
## 5315  immoralSelf  immoralSelf mismatch        n     n   1 0.5158
## 5316 immoralOther immoralOther    match        m     m   1 0.7710
## 5317 immoralOther immoralOther    match        m     m   1 0.5070
## 5318   moralOther   moralOther    match        m     m   1 0.5659
## 5319    moralSelf    moralSelf mismatch        n     m   0 0.4941
## 5320    moralSelf    moralSelf    match        m     m   1 0.5007
## 5321  immoralSelf  immoralSelf    match        m     m   1 0.4033
## 5322   moralOther   moralOther    match        m     n   0 0.5681
## 5323  immoralSelf  immoralSelf    match        m     m   1 0.5122
## 5324   moralOther   moralOther mismatch        n     m   0 0.7391
## 5325    moralSelf    moralSelf mismatch        n     n   1 0.7628
## 5326    moralSelf    moralSelf    match        m     m   1 0.5389
## 5327  immoralSelf  immoralSelf mismatch        n     n   1 0.4745
## 5328    moralSelf    moralSelf mismatch        n     m   0 0.4967
## 5329 immoralOther immoralOther    match        m     n   0 0.4241
## 5330    moralSelf    moralSelf mismatch        n     m   0 0.4934
## 5331  immoralSelf  immoralSelf    match        m     m   1 0.3561
## 5332    moralSelf    moralSelf mismatch        n     n   1 0.5519
## 5333  immoralSelf  immoralSelf    match        m     m   1 0.4519
## 5334    moralSelf    moralSelf    match        m     m   1 0.5615
## 5335 immoralOther immoralOther mismatch        n     n   1 0.4853
## 5336   moralOther   moralOther    match        m     m   1 0.5438
## 5337 immoralOther immoralOther mismatch        n     n   1 0.6155
## 5338   moralOther   moralOther mismatch        n     n   1 0.6643
## 5339 immoralOther immoralOther    match        m     m   1 0.5464
## 5340   moralOther   moralOther mismatch        n     n   1 0.5300
## 5341    moralSelf    moralSelf    match        m     m   1 0.5494
## 5342 immoralOther immoralOther mismatch        n     n   1 0.6130
## 5343  immoralSelf  immoralSelf    match        m     m   1 0.5340
## 5344   moralOther   moralOther    match        m     m   1 0.6414
## 5345 immoralOther immoralOther    match        m     m   1 0.6030
## 5346   moralOther   moralOther mismatch        n     n   1 0.7197
## 5347    moralSelf    moralSelf    match        m     m   1 0.5828
## 5348  immoralSelf  immoralSelf mismatch        n     n   1 0.5551
## 5349    moralSelf    moralSelf mismatch        n     n   1 0.6710
## 5350  immoralSelf  immoralSelf mismatch        n     n   1 0.5252
## 5351   moralOther   moralOther    match        m     n   0 0.7085
## 5352  immoralSelf  immoralSelf mismatch        n     n   1 0.5433
## 5353 immoralOther immoralOther mismatch        n     n   1 0.5750
## 5354    moralSelf    moralSelf    match        m     m   1 0.5952
## 5355    moralSelf    moralSelf mismatch        n     n   1 0.5318
## 5356 immoralOther immoralOther    match        m     m   1 0.7112
## 5357   moralOther   moralOther    match        m     m   1 0.6261
## 5358    moralSelf    moralSelf    match        m     m   1 0.5994
## 5359 immoralOther immoralOther mismatch        n     n   1 0.4441
## 5360 immoralOther immoralOther    match        m     m   1 0.5817
## 5361  immoralSelf  immoralSelf mismatch        n     n   1 0.5141
## 5362   moralOther   moralOther mismatch        n     n   1 0.5889
## 5363    moralSelf    moralSelf mismatch        n     m   0 0.5014
## 5364  immoralSelf  immoralSelf    match        m     m   1 0.5561
## 5365    moralSelf    moralSelf mismatch        n     n   1 0.6560
## 5366  immoralSelf  immoralSelf mismatch        n     n   1 0.6259
## 5367  immoralSelf  immoralSelf    match        m     m   1 0.5991
## 5368   moralOther   moralOther    match        m     m   1 0.5000
## 5369  immoralSelf  immoralSelf mismatch        n     n   1 0.4425
## 5370   moralOther   moralOther mismatch        n     n   1 0.6442
## 5371 immoralOther immoralOther    match        m     m   1 0.6019
## 5372   moralOther   moralOther    match        m     m   1 0.6467
## 5373   moralOther   moralOther mismatch        n     n   1 0.5244
## 5374 immoralOther immoralOther mismatch        n     m   0 0.5955
## 5375    moralSelf    moralSelf    match        m     m   1 0.7402
## 5376  immoralSelf  immoralSelf    match        m     m   1 0.5877
## 5377   moralOther   moralOther    match        m     m   1 0.5064
## 5378  immoralSelf  immoralSelf mismatch        n     n   1 0.5254
## 5379 immoralOther immoralOther    match        m     m   1 0.7005
## 5380    moralSelf    moralSelf    match        m     n   0 0.6832
## 5381  immoralSelf  immoralSelf mismatch        n     n   1 0.5016
## 5382    moralSelf    moralSelf mismatch        n     n   1 0.5163
## 5383    moralSelf    moralSelf mismatch        n     m   0 0.5634
## 5384  immoralSelf  immoralSelf mismatch        n     m   0 0.6675
## 5385 immoralOther immoralOther mismatch        n     n   1 0.6295
## 5386    moralSelf    moralSelf    match        m     m   1 0.5548
## 5387 immoralOther immoralOther    match        m     m   1 0.6226
## 5388  immoralSelf  immoralSelf    match        m     m   1 0.4518
## 5389   moralOther   moralOther    match        m     m   1 0.5775
## 5390 immoralOther immoralOther    match        m     m   1 0.6059
## 5391   moralOther   moralOther mismatch        n     n   1 0.7227
## 5392   moralOther   moralOther mismatch        n     n   1 0.5619
## 5393   moralOther   moralOther mismatch        n     n   1 0.6979
## 5394 immoralOther immoralOther mismatch        n     n   1 0.8326
## 5395  immoralSelf  immoralSelf    match        m     m   1 0.5141
## 5396    moralSelf    moralSelf mismatch        n     n   1 0.5331
## 5397 immoralOther immoralOther mismatch        n     n   1 0.5285
## 5398  immoralSelf  immoralSelf    match        m     m   1 0.4598
## 5399    moralSelf    moralSelf    match        m     m   1 0.5258
## 5400   moralOther   moralOther    match        m     n   0 0.4610
## 5401  immoralSelf  immoralSelf    match        m     m   1 0.5509
## 5402    moralSelf    moralSelf    match        m     m   1 0.6827
## 5403   moralOther   moralOther    match        m     m   1 0.6371
## 5404   moralOther   moralOther mismatch        n     n   1 0.8786
## 5405   moralOther   moralOther mismatch        n     n   1 0.6928
## 5406   moralOther   moralOther mismatch        n     n   1 0.5075
## 5407 immoralOther immoralOther mismatch        n     n   1 0.6464
## 5408 immoralOther immoralOther    match        m     m   1 0.6442
## 5409   moralOther   moralOther    match        m     m   1 0.5138
## 5410    moralSelf    moralSelf    match        m     m   1 0.6528
## 5411  immoralSelf  immoralSelf mismatch        n     n   1 0.6665
## 5412    moralSelf    moralSelf mismatch        n     m   0 0.5006
## 5413   moralOther   moralOther    match        m     m   1 0.5392
## 5414 immoralOther immoralOther    match        m     m   1 0.5789
## 5415  immoralSelf  immoralSelf mismatch        n     n   1 0.5351
## 5416 immoralOther immoralOther mismatch        n     m   0 0.7546
## 5417 immoralOther immoralOther    match        m     m   1 0.5984
## 5418    moralSelf    moralSelf mismatch        n     m   0 0.6752
## 5419    moralSelf    moralSelf    match        m     m   1 0.4975
## 5420  immoralSelf  immoralSelf mismatch        n     n   1 0.4801
## 5421    moralSelf    moralSelf mismatch        n     m   0 0.5385
## 5422  immoralSelf  immoralSelf    match        m     m   1 0.4102
## 5423 immoralOther immoralOther mismatch        n     n   1 0.5391
## 5424  immoralSelf  immoralSelf    match        m     m   1 0.6027
## 5425   moralOther   moralOther    match        m     m   1 0.4423
## 5426   moralOther   moralOther mismatch        n     n   1 0.4118
## 5427    moralSelf    moralSelf mismatch        n     n   1 0.5691
## 5428    moralSelf    moralSelf    match        m     m   1 0.4252
## 5429    moralSelf    moralSelf mismatch        n     n   1 0.4785
## 5430 immoralOther immoralOther mismatch        n     n   1 0.4606
## 5431  immoralSelf  immoralSelf mismatch        n     m   0 0.3266
## 5432  immoralSelf  immoralSelf mismatch        n     n   1 0.4659
## 5433  immoralSelf  immoralSelf mismatch        n     n   1 0.5879
## 5434 immoralOther immoralOther    match        m     m   1 0.6124
## 5435 immoralOther immoralOther mismatch        n     n   1 0.7255
## 5436    moralSelf    moralSelf    match        m     m   1 0.5806
## 5437 immoralOther immoralOther    match        m     m   1 0.6170
## 5438    moralSelf    moralSelf mismatch        n     n   1 0.4901
## 5439   moralOther   moralOther    match        m     m   1 0.4764
## 5440  immoralSelf  immoralSelf    match        m     m   1 0.4827
## 5441   moralOther   moralOther mismatch        n     n   1 0.5411
## 5442 immoralOther immoralOther mismatch        n     n   1 0.6447
## 5443   moralOther   moralOther    match        m     m   1 0.4424
## 5444 immoralOther immoralOther    match        m     m   1 0.5640
## 5445    moralSelf    moralSelf    match        m     m   1 0.5880
## 5446  immoralSelf  immoralSelf    match        m     m   1 0.4686
## 5447  immoralSelf  immoralSelf    match        m     m   1 0.4507
## 5448   moralOther   moralOther mismatch        n     m   0 0.6005
## 5449    moralSelf    moralSelf    match        m     m   1 0.4612
## 5450 immoralOther immoralOther mismatch        n     n   1 0.4473
## 5451  immoralSelf  immoralSelf    match        m     m   1 0.4050
## 5452  immoralSelf  immoralSelf    match        m     m   1 0.4177
## 5453  immoralSelf  immoralSelf    match        m     m   1 0.4508
## 5454    moralSelf    moralSelf mismatch        n     n   1 0.5205
## 5455 immoralOther immoralOther mismatch        n     n   1 0.6877
## 5456 immoralOther immoralOther    match        m     m   1 0.6581
## 5457   moralOther   moralOther    match        m     m   1 0.6521
## 5458   moralOther   moralOther    match        m     n   0 0.6898
## 5459 immoralOther immoralOther    match        m     m   1 0.5882
## 5460   moralOther   moralOther mismatch        n     n   1 0.5488
## 5461   moralOther   moralOther mismatch        n     n   1 0.6924
## 5462    moralSelf    moralSelf mismatch        n     n   1 0.5871
## 5463    moralSelf    moralSelf mismatch        n     n   1 0.5316
## 5464   moralOther   moralOther    match        m     m   1 0.5590
## 5465    moralSelf    moralSelf    match        m     m   1 0.6908
## 5466   moralOther   moralOther mismatch        n     n   1 0.5333
## 5467 immoralOther immoralOther    match        m     m   1 0.7047
## 5468  immoralSelf  immoralSelf mismatch        n     n   1 0.4756
## 5469  immoralSelf  immoralSelf mismatch        n     n   1 0.5218
## 5470    moralSelf    moralSelf    match        m     m   1 0.5450
## 5471  immoralSelf  immoralSelf mismatch        n     n   1 0.4646
## 5472 immoralOther immoralOther mismatch        n     n   1 0.5226
## 5473  immoralSelf  immoralSelf    match        m     m   1 0.3915
## 5474    moralSelf    moralSelf mismatch        n     n   1 0.5763
## 5475 immoralOther immoralOther    match        m     m   1 0.6206
## 5476   moralOther   moralOther mismatch        n     n   1 0.5379
## 5477    moralSelf    moralSelf    match        m     m   1 0.5573
## 5478    moralSelf    moralSelf    match        m     n   0 0.4491
## 5479    moralSelf    moralSelf mismatch        n     n   1 0.4628
## 5480  immoralSelf  immoralSelf mismatch        n     n   1 0.7368
## 5481   moralOther   moralOther    match        m     m   1 0.4761
## 5482    moralSelf    moralSelf    match        m     m   1 0.6265
## 5483 immoralOther immoralOther mismatch        n     n   1 0.5357
## 5484   moralOther   moralOther mismatch        n     m   0 0.5230
## 5485  immoralSelf  immoralSelf    match        m     m   1 0.4901
## 5486 immoralOther immoralOther mismatch        n     n   1 0.5247
## 5487   moralOther   moralOther    match        m     m   1 0.5717
## 5488   moralOther   moralOther mismatch        n     n   1 0.4598
## 5489  immoralSelf  immoralSelf mismatch        n     n   1 0.7017
## 5490 immoralOther immoralOther    match        m     m   1 0.5565
## 5491 immoralOther immoralOther    match        m     m   1 0.5203
## 5492  immoralSelf  immoralSelf    match        m     m   1 0.5754
## 5493  immoralSelf  immoralSelf mismatch        n     n   1 0.4757
## 5494   moralOther   moralOther    match        m     m   1 0.6740
## 5495    moralSelf    moralSelf mismatch        n     n   1 0.7122
## 5496 immoralOther immoralOther mismatch        n     n   1 0.6591
## 5497 immoralOther immoralOther    match        m     m   1 0.5250
## 5498  immoralSelf  immoralSelf    match        m     m   1 0.5162
## 5499 immoralOther immoralOther    match        m     m   1 0.6193
## 5500    moralSelf    moralSelf mismatch        n     n   1 0.5644
## 5501 immoralOther immoralOther mismatch        n     n   1 0.5644
## 5502  immoralSelf  immoralSelf mismatch        n     n   1 0.5165
## 5503    moralSelf    moralSelf    match        m     m   1 0.5237
## 5504  immoralSelf  immoralSelf    match        m     m   1 0.5709
## 5505  immoralSelf  immoralSelf mismatch        n     n   1 0.4909
## 5506    moralSelf    moralSelf mismatch        n     n   1 0.5813
## 5507   moralOther   moralOther    match        m     m   1 0.5537
## 5508   moralOther   moralOther mismatch        n     n   1 0.6776
## 5509    moralSelf    moralSelf mismatch        n     n   1 0.5717
## 5510   moralOther   moralOther    match        m     m   1 0.4838
## 5511  immoralSelf  immoralSelf mismatch        n     n   1 0.5103
## 5512    moralSelf    moralSelf    match        m     m   1 0.6251
## 5513   moralOther   moralOther mismatch        n     n   1 0.6544
## 5514 immoralOther immoralOther mismatch        n     n   1 0.6042
## 5515 immoralOther immoralOther    match        m     m   1 0.5689
## 5516  immoralSelf  immoralSelf    match        m     m   1 0.5770
## 5517   moralOther   moralOther    match        m     m   1 0.5013
## 5518   moralOther   moralOther mismatch        n     n   1 0.4680
## 5519    moralSelf    moralSelf    match        m     m   1 0.4102
## 5520 immoralOther immoralOther mismatch        n     m   0 0.4912
## 5521   moralOther   moralOther    match        n    ,<   2 0.9446
## 5522   moralOther   moralOther mismatch        m  <NA>  -1 1.0841
## 5523    moralSelf    moralSelf mismatch        m     n   0 0.4028
## 5524  immoralSelf  immoralSelf    match        n  <NA>  -1 1.0841
## 5525   moralOther   moralOther mismatch        m     m   1 0.5940
## 5526  immoralSelf  immoralSelf mismatch        m     n   0 0.9066
## 5527    moralSelf    moralSelf mismatch        m     m   1 0.8215
## 5528 immoralOther immoralOther    match        n     n   1 1.0348
## 5529 immoralOther immoralOther    match        n     n   1 0.8043
## 5530 immoralOther immoralOther mismatch        m     m   1 0.7371
## 5531    moralSelf    moralSelf    match        n     n   1 0.4445
## 5532    moralSelf    moralSelf    match        n     n   1 0.4663
## 5533 immoralOther immoralOther mismatch        m     m   1 0.8362
## 5534 immoralOther immoralOther mismatch        m     m   1 0.7897
## 5535    moralSelf    moralSelf    match        n     n   1 0.6302
## 5536    moralSelf    moralSelf mismatch        m     m   1 0.7076
## 5537  immoralSelf  immoralSelf mismatch        m     n   0 0.8784
## 5538 immoralOther immoralOther    match        n  <NA>  -1 1.0841
## 5539  immoralSelf  immoralSelf    match        n     n   1 0.9831
## 5540  immoralSelf  immoralSelf mismatch        m     n   0 1.0835
## 5541   moralOther   moralOther mismatch        m     m   1 0.7899
## 5542  immoralSelf  immoralSelf    match        n     n   1 0.9825
## 5543   moralOther   moralOther    match        n     n   1 1.0828
## 5544   moralOther   moralOther    match        n     m   0 0.6533
## 5545    moralSelf    moralSelf mismatch        m     m   1 0.7950
## 5546 immoralOther immoralOther    match        n     n   1 0.6675
## 5547    moralSelf    moralSelf    match        n     n   1 0.6737
## 5548 immoralOther immoralOther    match        n     n   1 0.9280
## 5549 immoralOther immoralOther mismatch        m     n   0 0.9474
## 5550   moralOther   moralOther    match        n  <NA>  -1 1.0841
## 5551 immoralOther immoralOther mismatch        m     m   1 0.6615
## 5552  immoralSelf  immoralSelf mismatch        m     m   1 0.6714
## 5553  immoralSelf  immoralSelf    match        n     n   1 0.7016
## 5554  immoralSelf  immoralSelf    match        n  <NA>  -1 1.0841
## 5555    moralSelf    moralSelf    match        n     n   1 0.6711
## 5556  immoralSelf  immoralSelf mismatch        m     n   0 0.8370
## 5557  immoralSelf  immoralSelf    match        n     n   1 0.8545
## 5558   moralOther   moralOther    match        n     m   0 0.6123
## 5559    moralSelf    moralSelf mismatch        m     m   1 0.7813
## 5560  immoralSelf  immoralSelf mismatch        m  <NA>  -1 1.0841
## 5561   moralOther   moralOther mismatch        m     m   1 1.0440
## 5562   moralOther   moralOther    match        n     m   0 0.9058
## 5563    moralSelf    moralSelf    match        n     n   1 0.6128
## 5564   moralOther   moralOther mismatch        m  <NA>  -1 1.0841
## 5565 immoralOther immoralOther    match        n  <NA>  -1 1.0841
## 5566 immoralOther immoralOther mismatch        m     n   0 0.6186
## 5567   moralOther   moralOther mismatch        m     m   1 0.6197
## 5568    moralSelf    moralSelf mismatch        m     m   1 0.8768
## 5569    moralSelf    moralSelf mismatch        m     m   1 0.8750
## 5570   moralOther   moralOther mismatch        m  <NA>  -1 1.0841
## 5571   moralOther   moralOther    match        n     m   0 0.5877
## 5572  immoralSelf  immoralSelf mismatch        m     m   1 0.8601
## 5573   moralOther   moralOther    match        n     m   0 0.9102
## 5574  immoralSelf  immoralSelf    match        n     n   1 1.0331
## 5575 immoralOther immoralOther    match        n     m   0 0.7226
## 5576 immoralOther immoralOther    match        n     m   0 0.7538
## 5577  immoralSelf  immoralSelf mismatch        m     m   1 0.9736
## 5578  immoralSelf  immoralSelf    match        n     n   1 0.8420
## 5579    moralSelf    moralSelf    match        n     n   1 0.6274
## 5580    moralSelf    moralSelf mismatch        m     m   1 0.6487
## 5581   moralOther   moralOther    match        n     n   1 1.0545
## 5582   moralOther   moralOther mismatch        m     m   1 0.8122
## 5583  immoralSelf  immoralSelf mismatch        m     m   1 0.7332
## 5584 immoralOther immoralOther mismatch        m     m   1 0.8247
## 5585  immoralSelf  immoralSelf    match        n     n   1 0.7580
## 5586   moralOther   moralOther mismatch        m     m   1 0.6017
## 5587    moralSelf    moralSelf mismatch        m     n   0 0.5425
## 5588 immoralOther immoralOther mismatch        m     m   1 0.7902
## 5589 immoralOther immoralOther    match        n     n   1 0.7827
## 5590    moralSelf    moralSelf    match        n     n   1 0.6030
## 5591 immoralOther immoralOther mismatch        m     n   0 0.8399
## 5592    moralSelf    moralSelf    match        n     n   1 0.6094
## 5593    moralSelf    moralSelf    match        n     n   1 0.5785
## 5594 immoralOther immoralOther    match        n     m   0 0.9587
## 5595  immoralSelf  immoralSelf    match        n     n   1 0.6507
## 5596  immoralSelf  immoralSelf mismatch        m     m   1 0.6964
## 5597   moralOther   moralOther    match        n     n   1 1.0070
## 5598   moralOther   moralOther mismatch        m     m   1 0.6439
## 5599  immoralSelf  immoralSelf mismatch        m     m   1 0.7413
## 5600 immoralOther immoralOther mismatch        m     m   1 0.7730
## 5601 immoralOther immoralOther    match        n     n   1 0.9251
## 5602  immoralSelf  immoralSelf    match        n     m   0 0.6362
## 5603  immoralSelf  immoralSelf mismatch        m     m   1 0.6857
## 5604 immoralOther immoralOther mismatch        m     m   1 0.8161
## 5605   moralOther   moralOther    match        n     m   0 0.5852
## 5606  immoralSelf  immoralSelf    match        n     n   1 1.0018
## 5607    moralSelf    moralSelf mismatch        m     m   1 0.7106
## 5608   moralOther   moralOther    match        n     n   1 0.6495
## 5609   moralOther   moralOther mismatch        m     m   1 0.7832
## 5610    moralSelf    moralSelf mismatch        m     m   1 0.7156
## 5611   moralOther   moralOther mismatch        m  <NA>  -1 1.0841
## 5612 immoralOther immoralOther    match        n  <NA>  -1 1.0841
## 5613    moralSelf    moralSelf    match        n     n   1 0.4957
## 5614    moralSelf    moralSelf    match        n     n   1 0.5023
## 5615    moralSelf    moralSelf mismatch        m     m   1 0.5489
## 5616 immoralOther immoralOther mismatch        m     n   0 0.9086
## 5617   moralOther   moralOther    match        n     n   1 0.9835
## 5618  immoralSelf  immoralSelf    match        n     n   1 0.7239
## 5619  immoralSelf  immoralSelf mismatch        m     m   1 0.7070
## 5620    moralSelf    moralSelf    match        n     n   1 0.5099
## 5621  immoralSelf  immoralSelf    match        n     n   1 0.7608
## 5622  immoralSelf  immoralSelf    match        n     m   0 0.6926
## 5623 immoralOther immoralOther    match        n     m   0 0.7553
## 5624   moralOther   moralOther mismatch        m     n   0 0.5433
## 5625   moralOther   moralOther    match        n     m   0 0.7588
## 5626 immoralOther immoralOther mismatch        m     n   0 0.5866
## 5627  immoralSelf  immoralSelf mismatch        m     m   1 0.7712
## 5628    moralSelf    moralSelf mismatch        m     m   1 0.6993
## 5629 immoralOther immoralOther    match        n     m   0 1.0340
## 5630   moralOther   moralOther mismatch        m     m   1 0.6756
## 5631    moralSelf    moralSelf mismatch        m     n   0 0.5057
## 5632 immoralOther immoralOther    match        n     m   0 0.6526
## 5633   moralOther   moralOther mismatch        m     m   1 0.9064
## 5634    moralSelf    moralSelf mismatch        m     m   1 0.5573
## 5635  immoralSelf  immoralSelf mismatch        m     m   1 0.9371
## 5636    moralSelf    moralSelf    match        n     n   1 0.7966
## 5637 immoralOther immoralOther mismatch        m     n   0 0.8531
## 5638    moralSelf    moralSelf    match        n     n   1 0.5709
## 5639   moralOther   moralOther    match        n     m   0 0.6829
## 5640 immoralOther immoralOther mismatch        m     n   0 0.8054
## 5641 immoralOther immoralOther    match        n  <NA>  -1 1.0841
## 5642   moralOther   moralOther mismatch        m     m   1 0.7017
## 5643    moralSelf    moralSelf    match        n     n   1 0.5244
## 5644   moralOther   moralOther mismatch        m     m   1 0.6275
## 5645   moralOther   moralOther    match        n     n   1 0.6808
## 5646  immoralSelf  immoralSelf    match        n     n   1 0.8352
## 5647 immoralOther immoralOther mismatch        m  <NA>  -1 1.0841
## 5648  immoralSelf  immoralSelf    match        n     n   1 0.8473
## 5649  immoralSelf  immoralSelf mismatch        m     m   1 0.7330
## 5650    moralSelf    moralSelf    match        n     n   1 0.5523
## 5651  immoralSelf  immoralSelf mismatch        m     m   1 0.6521
## 5652 immoralOther immoralOther    match        n  <NA>  -1 1.0841
## 5653    moralSelf    moralSelf mismatch        m     m   1 0.8763
## 5654  immoralSelf  immoralSelf    match        n     n   1 0.8026
## 5655   moralOther   moralOther mismatch        m     n   0 0.8554
## 5656    moralSelf    moralSelf mismatch        m     m   1 0.8691
## 5657 immoralOther immoralOther mismatch        m     m   1 0.8273
## 5658    moralSelf    moralSelf    match        n     n   1 0.6006
## 5659    moralSelf    moralSelf mismatch        m     m   1 0.6294
## 5660   moralOther   moralOther    match        n     n   1 0.7707
## 5661  immoralSelf  immoralSelf mismatch        m     m   1 0.6348
## 5662 immoralOther immoralOther    match        n     n   1 0.7722
## 5663 immoralOther immoralOther mismatch        m     m   1 0.7644
## 5664   moralOther   moralOther    match        n     n   1 0.8444
## 5665   moralOther   moralOther    match        n     m   0 0.5702
## 5666   moralOther   moralOther mismatch        m     m   1 0.6823
## 5667    moralSelf    moralSelf    match        n     n   1 0.4687
## 5668  immoralSelf  immoralSelf    match        n     n   1 0.7149
## 5669  immoralSelf  immoralSelf    match        n     n   1 0.6420
## 5670 immoralOther immoralOther mismatch        m     m   1 0.6756
## 5671   moralOther   moralOther mismatch        m     m   1 0.6578
## 5672    moralSelf    moralSelf mismatch        m     m   1 0.7238
## 5673 immoralOther immoralOther mismatch        m     m   1 0.9070
## 5674   moralOther   moralOther    match        n     m   0 0.5899
## 5675  immoralSelf  immoralSelf mismatch        m     m   1 0.8464
## 5676    moralSelf    moralSelf mismatch        m     m   1 0.6601
## 5677 immoralOther immoralOther mismatch        m     m   1 0.7261
## 5678    moralSelf    moralSelf    match        n     n   1 0.5093
## 5679   moralOther   moralOther mismatch        m     m   1 0.7682
## 5680    moralSelf    moralSelf    match        n     n   1 0.7922
## 5681 immoralOther immoralOther    match        n  <NA>  -1 1.0841
## 5682    moralSelf    moralSelf mismatch        m     m   1 0.6993
## 5683  immoralSelf  immoralSelf    match        n     n   1 0.7700
## 5684   moralOther   moralOther    match        n     m   0 0.7140
## 5685 immoralOther immoralOther    match        n     n   1 0.7890
## 5686  immoralSelf  immoralSelf mismatch        m     m   1 0.7335
## 5687 immoralOther immoralOther    match        n     n   1 0.6969
## 5688  immoralSelf  immoralSelf mismatch        m     n   0 0.4796
## 5689 immoralOther immoralOther    match        n     m   0 0.8020
## 5690   moralOther   moralOther mismatch        m     n   0 0.9988
## 5691  immoralSelf  immoralSelf    match        n     n   1 0.7316
## 5692 immoralOther immoralOther mismatch        m     n   0 0.9270
## 5693   moralOther   moralOther mismatch        m     m   1 0.8905
## 5694   moralOther   moralOther    match        n     n   1 0.6528
## 5695 immoralOther immoralOther    match        n     n   1 0.7546
## 5696    moralSelf    moralSelf mismatch        m     m   1 1.0144
## 5697  immoralSelf  immoralSelf mismatch        m     m   1 0.9435
## 5698  immoralSelf  immoralSelf mismatch        m     n   0 0.5911
## 5699    moralSelf    moralSelf mismatch        m     n   0 0.7276
## 5700    moralSelf    moralSelf    match        n     n   1 0.7871
## 5701    moralSelf    moralSelf mismatch        m     m   1 0.7077
## 5702 immoralOther immoralOther mismatch        m     m   1 0.7505
## 5703    moralSelf    moralSelf    match        n     n   1 0.6424
## 5704  immoralSelf  immoralSelf    match        n     m   0 0.5880
## 5705 immoralOther immoralOther    match        n     n   1 0.8845
## 5706  immoralSelf  immoralSelf mismatch        m     m   1 0.8470
## 5707 immoralOther immoralOther mismatch        m     m   1 0.7566
## 5708   moralOther   moralOther    match        n     n   1 0.7525
## 5709  immoralSelf  immoralSelf    match        n     n   1 0.8204
## 5710   moralOther   moralOther    match        n     m   0 0.7094
## 5711   moralOther   moralOther mismatch        m     m   1 0.8164
## 5712    moralSelf    moralSelf    match        n     n   1 0.6734
## 5713    moralSelf    moralSelf    match        n  <NA>  -1 1.0841
## 5714   moralOther   moralOther mismatch        m  <NA>  -1 1.0841
## 5715  immoralSelf  immoralSelf    match        n     n   1 0.8392
## 5716   moralOther   moralOther mismatch        m     m   1 0.9047
## 5717 immoralOther immoralOther mismatch        m  <NA>  -1 1.0841
## 5718    moralSelf    moralSelf mismatch        m     m   1 0.5741
## 5719 immoralOther immoralOther    match        n     m   0 0.9704
## 5720   moralOther   moralOther    match        n     m   0 0.6665
## 5721 immoralOther immoralOther mismatch        m     m   1 0.7405
## 5722   moralOther   moralOther mismatch        m     m   1 0.8681
## 5723    moralSelf    moralSelf    match        n     n   1 0.5864
## 5724 immoralOther immoralOther    match        n     n   1 1.0750
## 5725  immoralSelf  immoralSelf mismatch        m     m   1 0.5932
## 5726  immoralSelf  immoralSelf    match        n     n   1 0.6737
## 5727    moralSelf    moralSelf mismatch        m     m   1 0.6959
## 5728   moralOther   moralOther    match        n     m   0 0.6946
## 5729   moralOther   moralOther    match        n     n   1 0.6932
## 5730  immoralSelf  immoralSelf    match        n     n   1 0.9079
## 5731  immoralSelf  immoralSelf mismatch        m     m   1 0.9668
## 5732    moralSelf    moralSelf    match        n     n   1 0.5109
## 5733  immoralSelf  immoralSelf mismatch        m     m   1 0.8578
## 5734 immoralOther immoralOther mismatch        m     m   1 1.0838
## 5735 immoralOther immoralOther    match        n     n   1 0.8703
## 5736    moralSelf    moralSelf mismatch        m     m   1 0.7405
## 5737    moralSelf    moralSelf mismatch        m     n   0 0.6521
## 5738   moralOther   moralOther    match        n     n   1 0.6979
## 5739 immoralOther immoralOther    match        n     n   1 1.0405
## 5740   moralOther   moralOther mismatch        m     m   1 0.6101
## 5741   moralOther   moralOther    match        n     n   1 0.5872
## 5742  immoralSelf  immoralSelf    match        n     n   1 0.5956
## 5743   moralOther   moralOther mismatch        m     m   1 0.8282
## 5744  immoralSelf  immoralSelf mismatch        m     m   1 0.9695
## 5745  immoralSelf  immoralSelf    match        n     n   1 0.7936
## 5746    moralSelf    moralSelf    match        n     n   1 0.6261
## 5747  immoralSelf  immoralSelf    match        n  <NA>  -1 1.0841
## 5748   moralOther   moralOther    match        n     n   1 0.9779
## 5749    moralSelf    moralSelf mismatch        m     n   0 0.5902
## 5750 immoralOther immoralOther mismatch        m     m   1 0.8629
## 5751 immoralOther immoralOther mismatch        m     m   1 0.9448
## 5752 immoralOther immoralOther mismatch        m  <NA>  -1 1.0841
## 5753    moralSelf    moralSelf    match        n     n   1 0.5548
## 5754    moralSelf    moralSelf mismatch        m     m   1 0.6867
## 5755  immoralSelf  immoralSelf mismatch        m     m   1 0.9051
## 5756  immoralSelf  immoralSelf mismatch        m     m   1 0.7000
## 5757    moralSelf    moralSelf    match        n     n   1 0.5228
## 5758   moralOther   moralOther mismatch        m  <NA>  -1 1.0841
## 5759 immoralOther immoralOther    match        n     m   0 0.5964
## 5760 immoralOther immoralOther    match        n     n   1 0.7969
## 5761  immoralSelf  immoralSelf    match        n     m   0 0.9180
## 5762 immoralOther immoralOther    match        n     n   1 0.8052
## 5763 immoralOther immoralOther    match        n     n   1 0.6980
## 5764 immoralOther immoralOther mismatch        m     m   1 0.7448
## 5765   moralOther   moralOther    match        n     n   1 0.9843
## 5766 immoralOther immoralOther mismatch        m     m   1 0.6527
## 5767  immoralSelf  immoralSelf mismatch        m     m   1 0.7304
## 5768   moralOther   moralOther mismatch        m     m   1 0.7178
## 5769   moralOther   moralOther mismatch        m     m   1 0.8130
## 5770  immoralSelf  immoralSelf mismatch        m     m   1 0.7099
## 5771    moralSelf    moralSelf mismatch        m     m   1 0.7368
## 5772  immoralSelf  immoralSelf    match        n     n   1 0.8923
## 5773 immoralOther immoralOther mismatch        m     m   1 0.6110
## 5774    moralSelf    moralSelf    match        n     n   1 0.6520
## 5775    moralSelf    moralSelf    match        n     n   1 0.4817
## 5776   moralOther   moralOther    match        n     n   1 0.7963
## 5777    moralSelf    moralSelf mismatch        m     m   1 0.7327
## 5778   moralOther   moralOther mismatch        m     m   1 0.8321
## 5779  immoralSelf  immoralSelf    match        n     n   1 0.6934
## 5780    moralSelf    moralSelf    match        n     n   1 0.7400
## 5781  immoralSelf  immoralSelf mismatch        m     m   1 0.9957
## 5782   moralOther   moralOther    match        n     n   1 0.8283
## 5783 immoralOther immoralOther    match        n     n   1 0.7456
## 5784    moralSelf    moralSelf mismatch        m     n   0 0.6972
## 5785  immoralSelf  immoralSelf    match        n     n   1 0.7038
## 5786 immoralOther immoralOther mismatch        m     m   1 0.9469
## 5787    moralSelf    moralSelf    match        n     n   1 0.6264
## 5788  immoralSelf  immoralSelf    match        n     n   1 0.6797
## 5789    moralSelf    moralSelf    match        n     n   1 0.7781
## 5790 immoralOther immoralOther mismatch        m     m   1 0.6704
## 5791  immoralSelf  immoralSelf    match        n     n   1 1.0286
## 5792    moralSelf    moralSelf mismatch        m     m   1 0.7380
## 5793   moralOther   moralOther mismatch        m     m   1 0.8053
## 5794  immoralSelf  immoralSelf mismatch        m     m   1 0.8101
## 5795   moralOther   moralOther    match        n     n   1 0.6911
## 5796   moralOther   moralOther    match        n     n   1 0.9219
## 5797   moralOther   moralOther mismatch        m     m   1 0.6491
## 5798  immoralSelf  immoralSelf mismatch        m     m   1 0.8467
## 5799    moralSelf    moralSelf    match        n     n   1 0.6365
## 5800    moralSelf    moralSelf mismatch        m     m   1 0.7420
## 5801 immoralOther immoralOther    match        n     n   1 0.7496
## 5802  immoralSelf  immoralSelf mismatch        m     m   1 0.6932
## 5803   moralOther   moralOther    match        n     n   1 0.7479
## 5804   moralOther   moralOther mismatch        m     m   1 0.6755
## 5805 immoralOther immoralOther mismatch        m     m   1 0.9377
## 5806 immoralOther immoralOther    match        n     n   1 0.7651
## 5807 immoralOther immoralOther    match        n     n   1 0.6692
## 5808    moralSelf    moralSelf mismatch        m     m   1 0.7875
## 5809    moralSelf    moralSelf    match        n     n   1 0.6119
## 5810  immoralSelf  immoralSelf    match        n     n   1 0.7010
## 5811 immoralOther immoralOther    match        n     n   1 0.8837
## 5812  immoralSelf  immoralSelf mismatch        m     m   1 0.6222
## 5813    moralSelf    moralSelf    match        n     n   1 0.6835
## 5814   moralOther   moralOther mismatch        m     m   1 0.7819
## 5815   moralOther   moralOther mismatch        m     m   1 1.0822
## 5816 immoralOther immoralOther mismatch        m     m   1 0.8926
## 5817   moralOther   moralOther    match        n     n   1 0.6593
## 5818  immoralSelf  immoralSelf mismatch        m     m   1 0.6533
## 5819 immoralOther immoralOther    match        n     n   1 0.6991
## 5820    moralSelf    moralSelf mismatch        m     m   1 0.6497
## 5821 immoralOther immoralOther    match        n     n   1 0.8075
## 5822   moralOther   moralOther mismatch        m     m   1 0.8203
## 5823  immoralSelf  immoralSelf    match        n     n   1 0.7894
## 5824    moralSelf    moralSelf mismatch        m     m   1 0.8218
## 5825  immoralSelf  immoralSelf    match        n     n   1 0.8512
## 5826   moralOther   moralOther    match        n     n   1 0.9609
## 5827 immoralOther immoralOther mismatch        m     m   1 0.7648
## 5828    moralSelf    moralSelf    match        n     n   1 0.7488
## 5829  immoralSelf  immoralSelf mismatch        m     m   1 0.6045
## 5830   moralOther   moralOther    match        n     n   1 0.7693
## 5831 immoralOther immoralOther mismatch        m     m   1 0.9295
## 5832    moralSelf    moralSelf mismatch        m     m   1 0.6928
## 5833    moralSelf    moralSelf    match        n     n   1 0.6334
## 5834   moralOther   moralOther mismatch        m     m   1 0.6909
## 5835 immoralOther immoralOther mismatch        m     m   1 0.7014
## 5836    moralSelf    moralSelf    match        n     n   1 0.6600
## 5837   moralOther   moralOther mismatch        m     m   1 0.8380
## 5838    moralSelf    moralSelf    match        n     n   1 0.7275
## 5839   moralOther   moralOther mismatch        m     m   1 0.9149
## 5840   moralOther   moralOther    match        n     n   1 0.9301
## 5841  immoralSelf  immoralSelf    match        n     n   1 0.9735
## 5842  immoralSelf  immoralSelf mismatch        m     m   1 0.6257
## 5843 immoralOther immoralOther mismatch        m     m   1 0.9351
## 5844  immoralSelf  immoralSelf mismatch        m  <NA>  -1 1.0841
## 5845    moralSelf    moralSelf mismatch        m     n   0 0.8689
## 5846   moralOther   moralOther    match        n     n   1 0.8911
## 5847  immoralSelf  immoralSelf mismatch        m     m   1 0.9058
## 5848    moralSelf    moralSelf mismatch        m     m   1 0.6367
## 5849  immoralSelf  immoralSelf    match        n     n   1 0.6862
## 5850 immoralOther immoralOther    match        n     n   1 0.8726
## 5851    moralSelf    moralSelf mismatch        m     m   1 0.7109
## 5852 immoralOther immoralOther mismatch        m     n   0 0.7058
## 5853  immoralSelf  immoralSelf    match        n     n   1 0.7087
## 5854 immoralOther immoralOther    match        n     n   1 1.0636
## 5855   moralOther   moralOther    match        n     n   1 0.8816
## 5856 immoralOther immoralOther    match        n     n   1 0.9160
## 5857 immoralOther immoralOther    match        n     n   1 0.8032
## 5858   moralOther   moralOther    match        n     n   1 0.6960
## 5859   moralOther   moralOther mismatch        m     m   1 0.8787
## 5860 immoralOther immoralOther mismatch        m     m   1 0.8769
## 5861 immoralOther immoralOther mismatch        m     m   1 0.7231
## 5862   moralOther   moralOther mismatch        m     m   1 0.8422
## 5863   moralOther   moralOther mismatch        m     m   1 0.9237
## 5864    moralSelf    moralSelf mismatch        m     m   1 0.7469
## 5865   moralOther   moralOther    match        n     n   1 0.6346
## 5866  immoralSelf  immoralSelf    match        n     n   1 0.9641
## 5867    moralSelf    moralSelf mismatch        m     m   1 0.6442
## 5868 immoralOther immoralOther    match        n     n   1 0.7699
## 5869  immoralSelf  immoralSelf mismatch        m     m   1 0.4819
## 5870 immoralOther immoralOther mismatch        m     m   1 0.6764
## 5871   moralOther   moralOther    match        n     n   1 0.5626
## 5872    moralSelf    moralSelf mismatch        m     m   1 0.6028
## 5873    moralSelf    moralSelf    match        n     n   1 0.9036
## 5874 immoralOther immoralOther    match        n     n   1 0.6665
## 5875  immoralSelf  immoralSelf mismatch        m     m   1 0.6044
## 5876  immoralSelf  immoralSelf    match        n     n   1 0.7932
## 5877  immoralSelf  immoralSelf    match        n     n   1 0.7538
## 5878    moralSelf    moralSelf    match        n     n   1 0.6616
## 5879  immoralSelf  immoralSelf mismatch        m     m   1 0.6236
## 5880    moralSelf    moralSelf    match        n     n   1 0.7809
## 5881  immoralSelf  immoralSelf    match        n     m   0 1.0795
## 5882    moralSelf    moralSelf mismatch        m     m   1 0.7780
## 5883    moralSelf    moralSelf    match        n     n   1 0.6064
## 5884 immoralOther immoralOther mismatch        m     m   1 0.9951
## 5885   moralOther   moralOther    match        n     n   1 0.7557
## 5886  immoralSelf  immoralSelf mismatch        m     m   1 0.5356
## 5887  immoralSelf  immoralSelf mismatch        m     m   1 0.6909
## 5888    moralSelf    moralSelf    match        n     n   1 0.5333
## 5889 immoralOther immoralOther    match        n     m   0 0.9048
## 5890   moralOther   moralOther mismatch        m     n   0 0.7477
## 5891   moralOther   moralOther    match        n     n   1 0.9393
## 5892    moralSelf    moralSelf mismatch        m     m   1 0.8749
## 5893  immoralSelf  immoralSelf mismatch        m     m   1 1.0332
## 5894 immoralOther immoralOther    match        n     n   1 0.8827
## 5895  immoralSelf  immoralSelf    match        n     n   1 0.9411
## 5896    moralSelf    moralSelf mismatch        m     m   1 0.9488
## 5897  immoralSelf  immoralSelf    match        n     n   1 0.7724
## 5898   moralOther   moralOther    match        n     m   0 0.5805
## 5899 immoralOther immoralOther    match        n     n   1 0.8009
## 5900    moralSelf    moralSelf    match        n     n   1 0.6216
## 5901   moralOther   moralOther mismatch        m     m   1 0.9469
## 5902 immoralOther immoralOther mismatch        m     m   1 0.8745
## 5903   moralOther   moralOther mismatch        m     m   1 0.8087
## 5904 immoralOther immoralOther mismatch        m  <NA>  -1 1.0841
## 5905 immoralOther immoralOther mismatch        m     m   1 0.8960
## 5906   moralOther   moralOther    match        n     n   1 0.7106
## 5907 immoralOther immoralOther    match        n     m   0 0.8976
## 5908    moralSelf    moralSelf    match        n     n   1 0.6243
## 5909  immoralSelf  immoralSelf    match        n     n   1 0.8375
## 5910   moralOther   moralOther    match        n     n   1 0.6631
## 5911   moralOther   moralOther    match        n     n   1 0.9851
## 5912    moralSelf    moralSelf    match        n     n   1 0.5974
## 5913  immoralSelf  immoralSelf mismatch        m     m   1 0.9142
## 5914    moralSelf    moralSelf mismatch        m     m   1 0.7491
## 5915    moralSelf    moralSelf mismatch        m     m   1 0.9887
## 5916  immoralSelf  immoralSelf mismatch        m     m   1 0.7573
## 5917   moralOther   moralOther mismatch        m     m   1 0.6651
## 5918    moralSelf    moralSelf mismatch        m     m   1 0.6512
## 5919 immoralOther immoralOther mismatch        m     n   0 0.8329
## 5920   moralOther   moralOther mismatch        m     m   1 0.8264
## 5921    moralSelf    moralSelf    match        n     n   1 0.7036
## 5922  immoralSelf  immoralSelf    match        n     n   1 0.7866
## 5923 immoralOther immoralOther    match        n     n   1 0.9151
## 5924   moralOther   moralOther mismatch        m     m   1 0.6583
## 5925 immoralOther immoralOther mismatch        m     m   1 0.8043
## 5926 immoralOther immoralOther    match        n     n   1 0.8010
## 5927  immoralSelf  immoralSelf    match        n     n   1 0.7579
## 5928  immoralSelf  immoralSelf mismatch        m     m   1 0.8577
## 5929  immoralSelf  immoralSelf    match        n     n   1 0.7083
## 5930   moralOther   moralOther mismatch        m     m   1 0.9593
## 5931   moralOther   moralOther mismatch        m     m   1 0.8512
## 5932  immoralSelf  immoralSelf    match        n     n   1 0.8408
## 5933   moralOther   moralOther    match        n     n   1 0.7944
## 5934    moralSelf    moralSelf mismatch        m     m   1 0.7709
## 5935  immoralSelf  immoralSelf    match        n     n   1 0.5710
## 5936    moralSelf    moralSelf mismatch        m     m   1 0.8512
## 5937    moralSelf    moralSelf mismatch        m     n   0 0.7929
## 5938 immoralOther immoralOther    match        n  <NA>  -1 1.0841
## 5939 immoralOther immoralOther mismatch        m     m   1 0.6599
## 5940 immoralOther immoralOther mismatch        m     m   1 0.6059
## 5941    moralSelf    moralSelf    match        n     n   1 0.5867
## 5942  immoralSelf  immoralSelf mismatch        m     m   1 0.6992
## 5943    moralSelf    moralSelf    match        n     n   1 0.5860
## 5944 immoralOther immoralOther    match        n     n   1 0.7385
## 5945   moralOther   moralOther    match        n     n   1 0.7541
## 5946    moralSelf    moralSelf    match        n     n   1 0.8860
## 5947   moralOther   moralOther    match        n     n   1 0.7284
## 5948 immoralOther immoralOther mismatch        m     m   1 0.8757
## 5949 immoralOther immoralOther    match        n     n   1 0.7780
## 5950   moralOther   moralOther mismatch        m     m   1 0.7263
## 5951  immoralSelf  immoralSelf mismatch        m     m   1 0.6535
## 5952  immoralSelf  immoralSelf mismatch        m     m   1 0.7792
## 5953  immoralSelf  immoralSelf    match        n     n   1 0.6075
## 5954 immoralOther immoralOther mismatch        m  <NA>  -1 1.0841
## 5955 immoralOther immoralOther mismatch        m     m   1 0.7749
## 5956   moralOther   moralOther    match        n     n   1 0.6111
## 5957  immoralSelf  immoralSelf mismatch        m     m   1 0.6202
## 5958  immoralSelf  immoralSelf mismatch        m     m   1 0.8292
## 5959 immoralOther immoralOther    match        n     n   1 0.7624
## 5960    moralSelf    moralSelf    match        n     n   1 0.6746
## 5961  immoralSelf  immoralSelf    match        n     n   1 0.6968
## 5962  immoralSelf  immoralSelf    match        n     n   1 0.7195
## 5963    moralSelf    moralSelf mismatch        m     m   1 0.7425
## 5964    moralSelf    moralSelf    match        n     n   1 0.5422
## 5965    moralSelf    moralSelf mismatch        m     m   1 0.8458
## 5966    moralSelf    moralSelf mismatch        m     m   1 0.7876
## 5967  immoralSelf  immoralSelf mismatch        m     m   1 0.7482
## 5968   moralOther   moralOther mismatch        m     m   1 0.7078
## 5969   moralOther   moralOther mismatch        m     m   1 0.6706
## 5970 immoralOther immoralOther mismatch        m     m   1 0.6769
## 5971   moralOther   moralOther mismatch        m     m   1 0.7631
## 5972 immoralOther immoralOther    match        n     n   1 0.7392
## 5973   moralOther   moralOther    match        n     m   0 0.5548
## 5974 immoralOther immoralOther    match        n     n   1 0.7267
## 5975    moralSelf    moralSelf    match        n     n   1 0.6821
## 5976   moralOther   moralOther    match        n     n   1 0.8925
## 5977   moralOther   moralOther    match        n     n   1 0.7632
## 5978    moralSelf    moralSelf mismatch        m     m   1 0.7474
## 5979   moralOther   moralOther mismatch        m     m   1 0.7070
## 5980   moralOther   moralOther mismatch        m     m   1 0.7421
## 5981 immoralOther immoralOther mismatch        m     m   1 0.6376
## 5982   moralOther   moralOther mismatch        m     m   1 0.6551
## 5983 immoralOther immoralOther    match        n     n   1 0.8048
## 5984 immoralOther immoralOther mismatch        m     m   1 0.5856
## 5985  immoralSelf  immoralSelf mismatch        m     m   1 0.5461
## 5986 immoralOther immoralOther    match        n     n   1 0.7139
## 5987  immoralSelf  immoralSelf    match        n     n   1 0.6607
## 5988    moralSelf    moralSelf mismatch        m     n   0 0.6147
## 5989 immoralOther immoralOther    match        n     n   1 0.6717
## 5990    moralSelf    moralSelf mismatch        m     m   1 0.6299
## 5991  immoralSelf  immoralSelf    match        n     n   1 0.8112
## 5992 immoralOther immoralOther mismatch        m     m   1 0.5642
## 5993  immoralSelf  immoralSelf    match        n     n   1 0.9963
## 5994  immoralSelf  immoralSelf mismatch        m     m   1 0.6529
## 5995  immoralSelf  immoralSelf mismatch        m     m   1 0.6586
## 5996   moralOther   moralOther    match        n     n   1 0.7087
## 5997    moralSelf    moralSelf    match        n     n   1 0.5195
## 5998    moralSelf    moralSelf    match        n     n   1 0.4867
## 5999    moralSelf    moralSelf    match        n     n   1 0.4971
## 6000   moralOther   moralOther    match        n     n   1 0.5838
## 6001  immoralSelf  immoralSelf mismatch        m     m   1 0.5642
## 6002  immoralSelf  immoralSelf    match        n     n   1 0.7402
## 6003   moralOther   moralOther mismatch        m     m   1 0.6438
## 6004 immoralOther immoralOther    match        n     n   1 0.5733
## 6005 immoralOther immoralOther mismatch        m     m   1 0.5574
## 6006 immoralOther immoralOther    match        n     n   1 0.5932
## 6007   moralOther   moralOther    match        n     m   0 0.6179
## 6008 immoralOther immoralOther mismatch        m     n   0 0.6908
## 6009   moralOther   moralOther    match        n     n   1 0.7092
## 6010  immoralSelf  immoralSelf    match        n     n   1 0.8722
## 6011    moralSelf    moralSelf mismatch        m     m   1 0.7424
## 6012  immoralSelf  immoralSelf mismatch        m     m   1 0.8060
## 6013  immoralSelf  immoralSelf    match        n     n   1 0.8109
## 6014    moralSelf    moralSelf mismatch        m     m   1 0.6758
## 6015   moralOther   moralOther    match        n     n   1 0.7061
## 6016    moralSelf    moralSelf    match        n     n   1 0.5730
## 6017   moralOther   moralOther mismatch        m     m   1 0.9011
## 6018    moralSelf    moralSelf    match        n     n   1 0.5638
## 6019    moralSelf    moralSelf mismatch        m     m   1 0.6758
## 6020   moralOther   moralOther mismatch        m     m   1 0.8741
## 6021  immoralSelf  immoralSelf mismatch        m     m   1 0.7443
## 6022 immoralOther immoralOther mismatch        m     m   1 0.7518
## 6023    moralSelf    moralSelf    match        n     n   1 0.5958
## 6024 immoralOther immoralOther    match        n     n   1 0.6283
## 6025 immoralOther immoralOther mismatch        m     u   2 0.9580
## 6026 immoralOther immoralOther    match        n  <NA>  -1 1.0841
## 6027   moralOther   moralOther    match        n  <NA>  -1 1.0841
## 6028   moralOther   moralOther    match        n     n   1 0.8949
## 6029   moralOther   moralOther    match        n     n   1 0.5096
## 6030  immoralSelf  immoralSelf mismatch        m     m   1 0.6645
## 6031    moralSelf    moralSelf    match        n     n   1 0.4584
## 6032 immoralOther immoralOther    match        n     n   1 0.9484
## 6033   moralOther   moralOther mismatch        m     m   1 0.7401
## 6034   moralOther   moralOther mismatch        m     m   1 0.5957
## 6035 immoralOther immoralOther mismatch        m     m   1 0.7722
## 6036  immoralSelf  immoralSelf mismatch        m     m   1 0.7045
## 6037    moralSelf    moralSelf mismatch        m     m   1 0.6275
## 6038    moralSelf    moralSelf    match        n     n   1 0.5447
## 6039    moralSelf    moralSelf mismatch        m     m   1 0.7764
## 6040 immoralOther immoralOther    match        n     n   1 0.7727
## 6041  immoralSelf  immoralSelf    match        n     m   0 0.5728
## 6042  immoralSelf  immoralSelf    match        n     n   1 0.6208
## 6043   moralOther   moralOther mismatch        m     m   1 0.7301
## 6044  immoralSelf  immoralSelf mismatch        m     m   1 0.7096
## 6045  immoralSelf  immoralSelf    match        n     n   1 0.9284
## 6046    moralSelf    moralSelf mismatch        m     m   1 0.8359
## 6047    moralSelf    moralSelf    match        n     n   1 0.5173
## 6048 immoralOther immoralOther mismatch        m     m   1 1.0045
## 6049 immoralOther immoralOther mismatch        m     n   0 0.7293
## 6050    moralSelf    moralSelf mismatch        m     m   1 0.8207
## 6051  immoralSelf  immoralSelf mismatch        m     m   1 0.8660
## 6052   moralOther   moralOther    match        n     n   1 0.8039
## 6053  immoralSelf  immoralSelf    match        n     n   1 0.8407
## 6054    moralSelf    moralSelf    match        n     n   1 0.5222
## 6055    moralSelf    moralSelf mismatch        m     m   1 0.6974
## 6056 immoralOther immoralOther mismatch        m     m   1 0.6721
## 6057  immoralSelf  immoralSelf    match        n     n   1 0.8303
## 6058    moralSelf    moralSelf mismatch        m     m   1 0.6276
## 6059 immoralOther immoralOther mismatch        m     m   1 0.8168
## 6060    moralSelf    moralSelf    match        n     n   1 0.6019
## 6061   moralOther   moralOther mismatch        m     m   1 0.6947
## 6062 immoralOther immoralOther    match        n     n   1 0.6853
## 6063   moralOther   moralOther mismatch        m     m   1 0.7838
## 6064 immoralOther immoralOther    match        n     n   1 0.7641
## 6065   moralOther   moralOther    match        n     n   1 0.7483
## 6066  immoralSelf  immoralSelf mismatch        m     m   1 0.8758
## 6067  immoralSelf  immoralSelf    match        n     n   1 0.8660
## 6068   moralOther   moralOther mismatch        m     m   1 0.7561
## 6069  immoralSelf  immoralSelf mismatch        m     m   1 0.8479
## 6070 immoralOther immoralOther    match        n     n   1 0.5976
## 6071   moralOther   moralOther    match        n     m   0 0.8184
## 6072    moralSelf    moralSelf    match        n     n   1 0.7715
## 6073 immoralOther immoralOther mismatch        m     u   2 0.9286
## 6074 immoralOther immoralOther mismatch        m     m   1 0.8762
## 6075  immoralSelf  immoralSelf    match        n     n   1 0.6664
## 6076 immoralOther immoralOther    match        n     m   0 0.8444
## 6077   moralOther   moralOther mismatch        m     m   1 0.5221
## 6078  immoralSelf  immoralSelf    match        n     n   1 0.6253
## 6079    moralSelf    moralSelf    match        n     n   1 0.6705
## 6080   moralOther   moralOther    match        n     n   1 0.6448
## 6081  immoralSelf  immoralSelf mismatch        m     m   1 0.7304
## 6082   moralOther   moralOther    match        n     m   0 0.7898
## 6083  immoralSelf  immoralSelf mismatch        m     m   1 0.8064
## 6084   moralOther   moralOther mismatch        m     n   0 0.6192
## 6085    moralSelf    moralSelf mismatch        m     m   1 0.6843
## 6086   moralOther   moralOther mismatch        m     n   0 0.6467
## 6087    moralSelf    moralSelf    match        n     n   1 0.5644
## 6088  immoralSelf  immoralSelf mismatch        m     m   1 0.9725
## 6089  immoralSelf  immoralSelf    match        n     n   1 0.7566
## 6090    moralSelf    moralSelf mismatch        m     n   0 0.5445
## 6091 immoralOther immoralOther mismatch        m     m   1 0.8881
## 6092    moralSelf    moralSelf    match        n     n   1 0.6506
## 6093 immoralOther immoralOther    match        n     n   1 0.8323
## 6094 immoralOther immoralOther    match        n     m   0 0.9458
## 6095    moralSelf    moralSelf mismatch        m     m   1 0.7135
## 6096   moralOther   moralOther    match        n     n   1 0.7003
## 6097 immoralOther immoralOther    match        n     n   1 0.7790
## 6098   moralOther   moralOther    match        n     n   1 0.6793
## 6099 immoralOther immoralOther    match        n     n   1 0.6258
## 6100    moralSelf    moralSelf    match        n     n   1 0.6150
## 6101    moralSelf    moralSelf    match        n     m   0 0.8641
## 6102  immoralSelf  immoralSelf mismatch        m     m   1 0.8500
## 6103  immoralSelf  immoralSelf mismatch        m     m   1 0.7678
## 6104 immoralOther immoralOther mismatch        m     m   1 0.8319
## 6105    moralSelf    moralSelf mismatch        m     m   1 0.7412
## 6106 immoralOther immoralOther    match        n     n   1 0.8288
## 6107   moralOther   moralOther    match        n     n   1 0.7140
## 6108   moralOther   moralOther mismatch        m     m   1 0.7011
## 6109    moralSelf    moralSelf mismatch        m     n   0 0.5397
## 6110  immoralSelf  immoralSelf mismatch        m     m   1 0.7392
## 6111   moralOther   moralOther mismatch        m     m   1 0.9549
## 6112   moralOther   moralOther mismatch        m     m   1 0.7988
## 6113 immoralOther immoralOther mismatch        m     m   1 0.6196
## 6114  immoralSelf  immoralSelf    match        n     n   1 0.8767
## 6115    moralSelf    moralSelf mismatch        m     m   1 0.8029
## 6116  immoralSelf  immoralSelf    match        n     n   1 0.9436
## 6117   moralOther   moralOther    match        n     n   1 0.5593
## 6118  immoralSelf  immoralSelf    match        n     n   1 0.7472
## 6119    moralSelf    moralSelf    match        n     n   1 0.6907
## 6120 immoralOther immoralOther mismatch        m     m   1 0.7813
## 6121  immoralSelf  immoralSelf    match        n  <NA>  -1 1.0731
## 6122 immoralOther immoralOther mismatch        m  <NA>  -1 1.0731
## 6123  immoralSelf  immoralSelf mismatch        m     m   1 0.1061
## 6124 immoralOther immoralOther mismatch        m     n   0 0.3659
## 6125    moralSelf    moralSelf mismatch        m     m   1 0.4079
## 6126  immoralSelf  immoralSelf mismatch        m     m   1 0.3105
## 6127    moralSelf    moralSelf mismatch        m     m   1 0.2673
## 6128 immoralOther immoralOther    match        n     m   0 0.2872
## 6129    moralSelf    moralSelf    match        n     m   0 0.3235
## 6130 immoralOther immoralOther mismatch        m     n   0 0.3805
## 6131  immoralSelf  immoralSelf    match        n     n   1 0.5147
## 6132   moralOther   moralOther    match        n     m   0 0.2998
## 6133   moralOther   moralOther mismatch        m     n   0 0.5242
## 6134   moralOther   moralOther    match        n     m   0 0.4213
## 6135   moralOther   moralOther    match        n     m   0 0.2285
## 6136  immoralSelf  immoralSelf mismatch        m     m   1 0.3075
## 6137   moralOther   moralOther mismatch        m     m   1 0.3603
## 6138  immoralSelf  immoralSelf    match        n     m   0 0.2300
## 6139    moralSelf    moralSelf    match        n     n   1 0.1061
## 6140   moralOther   moralOther mismatch        m     m   1 0.1859
## 6141 immoralOther immoralOther    match        n     n   1 0.1645
## 6142    moralSelf    moralSelf    match        n     m   0 0.2063
## 6143    moralSelf    moralSelf mismatch        m     n   0 0.5009
## 6144 immoralOther immoralOther    match        n     n   1 0.2694
## 6145    moralSelf    moralSelf mismatch        m     m   1 0.2855
## 6146   moralOther   moralOther mismatch        m     m   1 0.2819
## 6147 immoralOther immoralOther mismatch        m     m   1 0.5421
## 6148    moralSelf    moralSelf    match        n     m   0 0.2397
## 6149 immoralOther immoralOther mismatch        m     n   0 0.1871
## 6150  immoralSelf  immoralSelf mismatch        m     m   1 0.1061
## 6151    moralSelf    moralSelf    match        n     m   0 0.2423
## 6152 immoralOther immoralOther    match        n     m   0 0.1061
## 6153  immoralSelf  immoralSelf    match        n     n   1 0.3063
## 6154  immoralSelf  immoralSelf    match        n     m   0 0.3112
## 6155   moralOther   moralOther    match        n     m   0 0.3480
## 6156 immoralOther immoralOther mismatch        m     n   0 0.4454
## 6157 immoralOther immoralOther    match        n  <NA>  -1 1.0731
## 6158    moralSelf    moralSelf mismatch        m     m   1 0.3812
## 6159  immoralSelf  immoralSelf mismatch        m     n   0 0.1061
## 6160   moralOther   moralOther mismatch        m     m   1 0.3522
## 6161 immoralOther immoralOther    match        n     n   1 0.3738
## 6162    moralSelf    moralSelf    match        n     m   0 0.2040
## 6163   moralOther   moralOther    match        n     n   1 0.1061
## 6164   moralOther   moralOther mismatch        m     n   0 0.3754
## 6165  immoralSelf  immoralSelf    match        n     m   0 0.3576
## 6166  immoralSelf  immoralSelf mismatch        m  <NA>  -1 1.0731
## 6167    moralSelf    moralSelf mismatch        m     m   1 0.3154
## 6168   moralOther   moralOther    match        n     n   1 0.3805
## 6169  immoralSelf  immoralSelf mismatch        m     m   1 0.1866
## 6170 immoralOther immoralOther mismatch        m     n   0 0.3891
## 6171   moralOther   moralOther    match        n     m   0 0.2155
## 6172    moralSelf    moralSelf    match        n     m   0 0.3144
## 6173   moralOther   moralOther    match        n     n   1 0.4712
## 6174  immoralSelf  immoralSelf    match        n     m   0 0.2713
## 6175  immoralSelf  immoralSelf    match        n     m   0 0.4714
## 6176    moralSelf    moralSelf    match        n     n   1 0.4314
## 6177 immoralOther immoralOther    match        n     n   1 0.4305
## 6178    moralSelf    moralSelf    match        n     m   0 0.3336
## 6179   moralOther   moralOther mismatch        m     n   0 0.3229
## 6180 immoralOther immoralOther    match        n     m   0 0.1061
## 6181  immoralSelf  immoralSelf    match        n     m   0 0.3527
## 6182   moralOther   moralOther mismatch        m     m   1 0.1824
## 6183 immoralOther immoralOther    match        n     n   1 0.1060
## 6184    moralSelf    moralSelf mismatch        m     m   1 0.3534
## 6185  immoralSelf  immoralSelf mismatch        m     m   1 0.4071
## 6186   moralOther   moralOther    match        n     m   0 0.3818
## 6187 immoralOther immoralOther mismatch        m  <NA>  -1 1.0731
## 6188 immoralOther immoralOther mismatch        m     m   1 0.1801
## 6189    moralSelf    moralSelf mismatch        m     m   1 0.1061
## 6190  immoralSelf  immoralSelf mismatch        m     m   1 0.1751
## 6191    moralSelf    moralSelf mismatch        m  <NA>  -1 1.0731
## 6192   moralOther   moralOther mismatch        m     m   1 0.2533
## 6193 immoralOther immoralOther mismatch        m  <NA>  -1 1.0731
## 6194   moralOther   moralOther mismatch        m space   2 0.7207
## 6195  immoralSelf  immoralSelf mismatch        m     m   1 0.5618
## 6196   moralOther   moralOther    match        n     m   0 0.3318
## 6197 immoralOther immoralOther    match        n     n   1 0.4970
## 6198  immoralSelf  immoralSelf    match        n     m   0 0.3776
## 6199    moralSelf    moralSelf mismatch        m     m   1 0.4718
## 6200   moralOther   moralOther mismatch        m     m   1 0.3518
## 6201   moralOther   moralOther mismatch        m     m   1 0.5016
## 6202    moralSelf    moralSelf mismatch        m     m   1 0.5101
## 6203  immoralSelf  immoralSelf    match        n     m   0 0.3869
## 6204  immoralSelf  immoralSelf    match        n     m   0 0.3493
## 6205 immoralOther immoralOther mismatch        m     n   0 0.3028
## 6206 immoralOther immoralOther    match        n     m   0 0.2633
## 6207  immoralSelf  immoralSelf mismatch        m     m   1 0.3312
## 6208    moralSelf    moralSelf    match        n     m   0 0.2884
## 6209    moralSelf    moralSelf mismatch        m     m   1 0.3167
## 6210    moralSelf    moralSelf    match        n     m   0 0.3258
## 6211  immoralSelf  immoralSelf mismatch        m     n   0 0.5348
## 6212 immoralOther immoralOther mismatch        m     n   0 0.5000
## 6213   moralOther   moralOther    match        n     n   1 0.5806
## 6214    moralSelf    moralSelf    match        n     m   0 0.3948
## 6215 immoralOther immoralOther    match        n     m   0 0.5092
## 6216   moralOther   moralOther    match        n  <NA>  -1 1.0731
## 6217 immoralOther immoralOther mismatch        m     n   0 0.3301
## 6218    moralSelf    moralSelf    match        n     m   0 0.3594
## 6219    moralSelf    moralSelf mismatch        m     m   1 0.1732
## 6220   moralOther   moralOther    match        n     m   0 0.3350
## 6221   moralOther   moralOther mismatch        m     m   1 0.4843
## 6222   moralOther   moralOther mismatch        m     m   1 0.5446
## 6223    moralSelf    moralSelf mismatch        m     m   1 0.6421
## 6224  immoralSelf  immoralSelf    match        n     m   0 0.5014
## 6225  immoralSelf  immoralSelf mismatch        m     m   1 0.5820
## 6226 immoralOther immoralOther mismatch        m     m   1 0.5962
## 6227   moralOther   moralOther    match        n     n   1 0.7026
## 6228 immoralOther immoralOther    match        n     n   1 0.4792
## 6229  immoralSelf  immoralSelf mismatch        m     m   1 0.5033
## 6230   moralOther   moralOther    match        n     m   0 0.3960
## 6231 immoralOther immoralOther    match        n     n   1 0.4464
## 6232    moralSelf    moralSelf    match        n     m   0 0.4461
## 6233  immoralSelf  immoralSelf mismatch        m     m   1 0.5497
## 6234 immoralOther immoralOther mismatch        m     m   1 0.6552
## 6235  immoralSelf  immoralSelf    match        n     m   0 0.4228
## 6236   moralOther   moralOther mismatch        m     m   1 0.5740
## 6237    moralSelf    moralSelf    match        n     m   0 0.5401
## 6238  immoralSelf  immoralSelf    match        n     m   0 0.5576
## 6239 immoralOther immoralOther    match        n     n   1 0.5154
## 6240    moralSelf    moralSelf mismatch        m     m   1 0.5244
## 6241   moralOther   moralOther    match        n  <NA>  -1 1.0731
## 6242  immoralSelf  immoralSelf mismatch        m     m   1 0.1062
## 6243    moralSelf    moralSelf mismatch        m     m   1 0.3389
## 6244 immoralOther immoralOther mismatch        m     m   1 0.6284
## 6245    moralSelf    moralSelf    match        n     m   0 0.5075
## 6246  immoralSelf  immoralSelf mismatch        m     m   1 0.4403
## 6247    moralSelf    moralSelf mismatch        m     m   1 0.6117
## 6248   moralOther   moralOther mismatch        m     m   1 0.3745
## 6249   moralOther   moralOther mismatch        m     m   1 0.5087
##  [ reached 'max' / getOption("max.print") -- omitted 19671 rows ]
```

```r
group <- df.mt.raw %>% 
  group_by(Shape)
group#注意看数据框的第二行，有Groups:   Shape [4]的信息
```

```
## # A tibble: 25,920 × 16
## # Groups:   Shape [4]
##    Date        Prac    Sub   Age Sex   Hand  Block   Bin Trial Shape Label Match
##    <chr>       <chr> <int> <int> <chr> <chr> <int> <int> <int> <chr> <chr> <chr>
##  1 02-May-201… Exp    7302    22 fema… R         1     1     1 immo… immo… mism…
##  2 02-May-201… Exp    7302    22 fema… R         1     1     2 mora… mora… mism…
##  3 02-May-201… Exp    7302    22 fema… R         1     1     3 immo… immo… mism…
##  4 02-May-201… Exp    7302    22 fema… R         1     1     4 mora… mora… mism…
##  5 02-May-201… Exp    7302    22 fema… R         1     1     5 immo… immo… match
##  6 02-May-201… Exp    7302    22 fema… R         1     1     6 immo… immo… match
##  7 02-May-201… Exp    7302    22 fema… R         1     1     7 mora… mora… match
##  8 02-May-201… Exp    7302    22 fema… R         1     1     8 mora… mora… match
##  9 02-May-201… Exp    7302    22 fema… R         1     1     9 mora… mora… mism…
## 10 02-May-201… Exp    7302    22 fema… R         1     1    10 immo… immo… mism…
## # ℹ 25,910 more rows
## # ℹ 4 more variables: CorrResp <chr>, Resp <chr>, ACC <int>, RT <dbl>
```

对比一下，经过了group_by后的数据框，增加了一个Groups:Shape [4]的提示，它有一个标记，表示它已经在内部进行了分组，分组标准是shape，即形状。如果想删除该shape变量，就必须ungroup。否则，这个分组标签将一直存在于数据框中。因此，我们建议在进行groupby之后一定要进行ungroup，否则分组标签将一直存在于数据框中。

实际上，我们不会像现在这样无聊地直接添加group_by，然后看它会发生什么。我们需要明确后面分析的逻辑是什么。我们可以通过group_by将数据框按照base的ID分成几个小的数据框，然后以每个亚组为单位进行计算。比如，我们可以用summarise求出每个subgroup的行数，然后返回到n里面去，得到一个描述性的结果。但是，我们需要注意，当我们得到新的结果后，一定要把它ungroup掉，否则会影响后面的分析。此外，我们需要注意，如果我们再进行一个group_by，而没有ungroup前面的结果，可能会覆盖掉前面的结果，这是需要注意的问题。

如果我们不使用group_by，我们以前的做法是先申请一个中间变量，然后将数据按条件分组，选出一个subset出来，然后对每个操作分别求均值和标准差，最后将它们合并起来。现在有了group_by和summarise，我们可以在管道中一次性完成所有操作，而不需要生成大量的中间变量。这样做的好处是逻辑更清晰，代码更简洁，而且不会占用过多的内存
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
<div class="datatables html-widget html-fill-item-overflow-hidden html-fill-item" id="htmlwidget-503d8462dba8709e7319" style="width:100%;height:auto;"></div>
<script type="application/json" data-for="htmlwidget-503d8462dba8709e7319">{"x":{"filter":"none","vertical":false,"data":[["1","2","3","4"],["immoralOther","immoralSelf","moralOther","moralSelf"],[6480,6480,6480,6480]],"container":"<table class=\"display\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>Shape<\/th>\n      <th>n()<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"columnDefs":[{"className":"dt-right","targets":2},{"orderable":false,"targets":0}],"order":[],"autoWidth":false,"orderClasses":false}},"evals":[],"jsHooks":[]}</script>
```

```r
ungroup <- df.mt.raw %>% 
  summarise(n())
DT::datatable(ungroup)
```

```{=html}
<div class="datatables html-widget html-fill-item-overflow-hidden html-fill-item" id="htmlwidget-b63e8e78948392f27812" style="width:100%;height:auto;"></div>
<script type="application/json" data-for="htmlwidget-b63e8e78948392f27812">{"x":{"filter":"none","vertical":false,"data":[["1"],[25920]],"container":"<table class=\"display\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>n()<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"columnDefs":[{"className":"dt-right","targets":1},{"orderable":false,"targets":0}],"order":[],"autoWidth":false,"orderClasses":false}},"evals":[],"jsHooks":[]}</script>
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

for loop是一种循环语句，它的语法结构是for (variable in sequence) {statement}，其中variable是一个代符，sequence是一个向量或列表，而statement则是要重复执行的某一个语句。在每一次循环中，variable都被赋予为sequence里面的一个元素，然后执行一次statement，再回到sequence里面的下一个元素，不断地循环。每一次循环，variable都会变成sequence里的下一个元素，因此它只是一直在变化。


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

在批量读取文件的for循环中，我们读取了所有以data_exp7开头，以out结尾的文件，使用list.files函数匹配符合模式的所有文件，得到一列文件名list，然后循环list。我们需要对1到num files进行操作，这个sequence是从1开始的。在R和MatLab中，我们也是从1开始的，但在Python中，我们是从0开始的。

在for循环中，我们使用i in seq_along(files)来循环操作，从i等于1开始，然后一直到最后一个元素。当i等于1时，我们使用read table函数，并从files中提取出第i个元素，这个元素实际上是一个字符，也就是代表某一个特定的文件。

file path函数将前面的文件夹和后面的文件名整合成一个完整的文件路径。因此，read table函数的第一个argument就是这个file path。当进行了读取操作之后。这个df实际上就是第一个out文件的数据。

在datalist中，i代表第几个元素，使用df.list[i]来index list。在第一次循环中，我们将df给到df.list的第一个元素。

因此，当i等于1时，我们开始第一次循环，首先有一个files代表需要操作的文件。比方说xxxxxx.out，对吧？第一个元素是什么，第二个元素也是什么.out，一直到最后一个元素。当我们i=1的时候，我们把它的第一个元素拿出来，然后把它拿到read.table里面去。这个filepath本身又是一个函数，我们把这个第一个文件夹的名字和第一个out文件路径加起来，就是file.filepath这个地方。然后再开启read.table的第二个argument，那就是header=true。通过这个for loop，我们读取了所有的文件，把读取的结果复制到了df.list里面的对应元素。这个时候，df_list就更新了。以此类推直到所有的files里面的所有数据都读取完，那么这个时候我们的df_list就是一个包含了所有数据的一个list，每一个元素代表一个base的一个数据。

### 信号检测

接下来，我们回顾上一节课的练习题。我们使用数据预处理的方法，求出match数据中的d prime，这是一个信号检测论的指标，也称为敏感性指标。

我们可以将数据分为signal和noise，其中signal是指hit，对应的是correct rejection；而如果刺激中有noise，我们认为它是有信号的，这就是FA。如果我们报告时认为刺激中有信号，但实际上是noise，这就是Miss。根据信号检测论，我们将刺激分为match和mismatch，其中match是信号，而mismatch或nonmatch是noise。我们的反应是指我们呈现的刺激。如果反应是match，那么这就是hit。对于match的trial，如果我们的反应是mismatch，那么这就是信号减速论的miss。正确率是0，因为它是错误的。对于mismatch的trial，如果我们的反应是mismatch，那么这就是CR，正确率是1。FA的正确率是0。我们需要计算d'，它等于zhit减去zFA，这是我们敏感性的指标。我们需要用R来计算每个条件下的d'，因此我们需要使用group_by。最终，我们将得到一个比例，其中包括base的id、条件和d'。

我们在实际使用这些函数时，是带着目的去使用的。想要计算击中率虚报率，我们首先要知道什么情况是击中，也要让计算机知道什么情况是击中，一旦分类了，计算的时候也需要进行判断。但在分类判断之前，我们要先告诉计算机什么是正确情况什么是错误情况，哪些是这个被试做的，哪些不是，所以也需要分组。

我们有了基本的思路：1.分组，使用group_by。2.告诉计算机什么是hit，什么是miss，使用summarise函数。3.利用判断语句进行分类计算，我们这里使用了ifelse。

我们选出需要的几列，包括自变量和因变量，以及分组变量。使用select()函数，因为数据中存在缺失值，所以也需要提剔除它们。


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

在这里，我们使用了一个num来判断它们的数量。我们可以使用nrow代替。这只是其中一种做法。接下来，我们看一下ACC里面所有等于match的情况。ACC等于什么呢？条件是match，然后ACC等于1。当我们想要计算hit时，我们需要将两个条件进行组合。一个是刺激呈现的内容，另一个是正确率。只有在match条件下反应正确的试次才是hit的试次。通过这个length，我们计算出了在每一个base、每一个block、每个bin下面以及每一个条件shape之下有多少个hit的比值。

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

然后，我们计算出了每一个d'。我们可以看到，这个d'是用qnorm计算的。hit的比值是等于hit除以hit加miss。这个地方有点复杂，它实际上就是想要把这两个东西打包在一个语句里面。zheat和zfa有两个条件，首先我们看它的两个部分，一个是qnorm，表示hit的部分，另一个是FA部分。我们计算出hit rate，然后将其转换成z分数，减去quantum。这个地方为什么要用if else呢？因为会出现一个比较特殊的情况，比方说也许某个被试的hit全部是正确的，即正确率为1，此时再用qnorm的话，结果是一个无限大的数，就是一个正向的infinity。对这种可能的特殊情况，我们需要把hit rate转换成一个小于1的值，来避免出现infinity。因为我们无法对infinity进行进一步的计算。所以，如果hit rate小于1，我们就用它。

这个是if else在这个type里面的应用。如果前面hit rate小于1，那么我们就使用它自己的值。如果它不小于1，也就是等于1，那么我们就用1减去1除以括号里的值，这样可以使得结果变得稍微小一点。这是一个常用的方法，在信号检测论里面也常用。对于FA也是同样的方法，我们使用if else，如果它大于0，那么我们就用它；如果等于0，那么我们就需要想一个办法让它变成一个不是0的值，来避免得到一个负向无穷大的结果，这样就无法进行后续的运算。接下来我们就可以算出d'，然后去掉后面所有的hit、fn、miss和cr，进行后续的运算。这个逻辑清晰吗？

我们可以拆开每一个函数来看，选择columns group by，summarise group by，qvolume是一个把百分比转换成为1分数的函数，这个大家需要去后面查找。然后我们进行相减，插入条件语句，再把它转换成1分数，最后就得到d'。我们也可以进行后续的运算，再次group_by，subject和shape。我们可以查看每一个被试在每一个bin上面的结果的变化。如果我们收了很多被试，可能就不需要关注每一个被试的变化，只需要报告总体的结果即可。我们可以根据time inversion里面的数据预数的一系列操作，一个一个管道下来，最后直接得到我们想要的结果。这样的管道操作可能需要不断迭代，刚开始可能只会做最简单的预处理，但是迭代到最后，就会形成一个很长的管道。

数据处理的流程非常清晰。在这个过程中，我们对ACC进行了选择，相当于是选择了符合条件的一些行，并求出它的长度。虽然理论上讲，我们也可以用别的变量来代替ACC，比如RT，但是ACC是最方便的选择，因为我们后面也会用到它。但是，这并不是唯一的操作，我们还可以用其他的变量来进行操作，比如length换成n。如果我们用filter的话，也是可行的，但可能需要写更多的代码。我们要根据自己的研究目的或者想要操作的目的去进行数据处理，把各种预处理操作进行组合。
