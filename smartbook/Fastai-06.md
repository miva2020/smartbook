# Other Computer Vision Problems

# 其它计算机视觉问题

In the previous chapter you learned some important practical techniques for training models in practice. Considerations like selecting learning rates and the number of epochs are very important to getting good results.

在上一章节我们学习了一些在实践中训练模型的一些重要特定技术。考虑的因素像选择学习率和周期数对于获得良好结果是非常重要的。

In this chapter we are going to look at two other types of computer vision problems: multi-label classification and regression. The first one is when you want to predict more than one label per image (or sometimes none at all), and the second is when your labels are one or several numbers—a quantity instead of a category.

在本章节，我们将学习另外两个类型的计算机视觉问题：多标签分类和回归。第一个问题是当我们想要预测单张图像有多个标签时（或有时间根本什么都没有）遇到到，第二个问题是当你的标注是一个或多个时：数量而不是类别。

In the process will study more deeply the output activations, targets, and loss functions in deep learning models.

在这个过程中会学习在深度学习模型中更深层的输出激活、目标和损失函数。

## Multi-Label Classification

## 多标签分类

Multi-label classification refers to the problem of identifying the categories of objects in images that may not contain exactly one type of object. There may be more than one kind of object, or there may be no objects at all in the classes that you are looking for.

多标签分类适用于那种在图像中不可能只包含一个目标类型的识别问题。有可能会超过一个目标类型，或在分类中你根本没有找到目标。

For instance, this would have been a great approach for our bear classifier. One problem with the bear classifier that we rolled out in <chapter_production> was that if a user uploaded something that wasn't any kind of bear, the model would still say it was either a grizzly, black, or teddy bear—it had no ability to predict "not a bear at all." In fact, after we have completed this chapter, it would be a great exercise for you to go back to your image classifier application, and try to retrain it using the multi-label technique, then test it by passing in an image that is not of any of your recognized classes.

例如，我们的熊分类器这一方法也许已经是一个非常好的方法了。在<章节：产品>中我们遇到了一个熊分类器的问题，如果用户上传不包含任何种类熊的图像，模型依然会说它是灰熊、黑熊或泰迪熊的一种。它不具备预测“根本没有熊”的能力。实际上，当我们完成本章节后，你重回图像分类器应用会是很好的练习。用多标签技术尝试重训练模型，然后通过传递给模型一张不包含任何你的识别分类的图像，对它进行测试。

In practice, we have not seen many examples of people training multi-label classifiers for this purpose—but we very often see both users and developers complaining about this problem. It appears that this simple solution is not at all widely understood or appreciated! Because in practice it is probably more common to have some images with zero matches or more than one match, we should probably expect in practice that multi-label classifiers are more widely applicable than single-label classifiers.

在实践中，我们没有看到很多人们以此为目的训练多标签分类器的例子，但是我们常常看到用户和开发人员抱怨这个问题。表明这个简单的解决方案根本没有被广泛的理解或认可！由于在实践中可能更常见的是很多零匹配或超过一个匹配项的图片，相比单标签分类器，我们可能应该期望在实践中多标签分类器更更广泛的应用。

First, let's see what a multi-label dataset looks like, then we'll explain how to get it ready for our model. You'll see that the architecture of the model does not change from the last chapter; only the loss function does. Let's start with the data.

首先，我们看一下多标签数据集看起来是什么样子，然后我们会解释如何为我们的模型把它准备好。你会在看到从上一章节开始，模型的架构没有做变化，只是变了损失函数。让我们从数据开始。

### The Data

### 数据

For our example we are going to use the PASCAL dataset, which can have more than one kind of classified object per image.

We begin by downloading and extracting the dataset as per usual:

我们的例子会使用PASCAL数据集，这个数据集对每张图像有超过一个种类的分类对象。与平常一样我们开始下载并抽取数据集：

```
from fastai.vision.all import *
path = untar_data(URLs.PASCAL_2007)
```

This dataset is different from the ones we have seen before, in that it is not structured by filename or folder but instead comes with a CSV (comma-separated values) file telling us what labels to use for each image. We can inspect the CSV file by reading it into a Pandas DataFrame:

这个数据集与我们以前看到的那些是不同的，也就是说它没有通过文件名或文件夹结构化，而是用CSV（逗号分割值）文件告诉我们每一张图像所用的标签是什么。我们能够把它读取到Pandas DataFrame里，来查看这个CSV文件：

```
df = pd.read_csv(path/'train.csv')
df.head()
```

<table style="width: 370px;border-collapse: collapse;" >
  <tr>
    <td  style="width: 10px;" align="center"></td>
    <td  style="width: 100px;" align="center">fname</td>
    <td  style="width: 160px;" align="center">labels</td>
    <td  style="width: 100px;" align="center">is_valid</td>
  </tr>
  <tr>
    <td align="center" font-weight="bold"><strong>0</strong></td>
    <td align="right">000005.jpg</td>
  	<td align="right">chair</td>
  	<td align="right">True</td>
  </tr>
  </tr>
    <td align="center"><strong>1</strong></td>
    <td align="right">000007.jpg</td>
  	<td align="right">car</td>
  	<td align="right">True</td>
  </tr>
  </tr>
    <td align="center"><strong>2</strong></td>
    <td align="right">000009.jpg</td>
  	<td align="right">horse person</td>
  	<td align="right">True</td>
  </tr>
  </tr>
    <td align="center"><strong>3</strong></td>
    <td align="right">000012.jpg</td>
  	<td align="right">car</td>
  	<td align="right">False</td>
  </tr>
  </tr>
    <td align="center" ><strong>4</strong></td>
    <td align="right">000016.jpg</td>
  	<td align="right">bicycle</td>
  	<td align="right">True</td>
  </tr>
</table>

As you can see, the list of categories in each image is shown as a space-delimited string.

正如你能看到的，每张图像的分类列表被显示为一个空格分割字符串。

### Sidebar: Pandas and DataFrames

### 侧边栏：Pandas and DataFrames

No, it’s not actually a panda! *Pandas* is a Python library that is used to manipulate and analyze tabular and time series data. The main class is `DataFrame`, which represents a table of rows and columns. You can get a DataFrame from a CSV file, a database table, Python dictionaries, and many other sources. In Jupyter, a DataFrame is output as a formatted table, as shown here.

You can access rows and columns of a DataFrame with the `iloc` property, as if it were a matrix:

是的，实际上它并不是熊猫！*Pandas*是一个Python库，用于操作和分析表格形式和时间序列数据。主要的类是`DataFrame`，它相当于一个有行和列的表。你能够从CSV文件、数据库表、Python字典和一些其它资源获取DataFrame。在Jupyter中，一个DataFrame输出的是一个格式化后的表，正如下面显示的。

你利用`iloc`特性能够获取一个DataFrame的行和列，好像它是一个矩阵：

```
df.iloc[:,0]
```

Out: $\begin{array}{l,c}
0 &      000005.jpg \\
1&       000007.jpg  \\
2&       000009.jpg \\
3&       000012.jpg  \\
4&       000016.jpg  \\
&           ...      \\
5006&    009954.jpg \\
5007&    009955.jpg \\
5008&    009958.jpg  \\
5009&    009959.jpg  \\
5010&    009961.jpg  \end{array}$
$\begin{array}{l}&&Name: fname, Length: 5011, dtype: object
 \end{array}$

```
df.iloc[0,:]
# Trailing :s are always optional (in numpy, pytorch, pandas, etc.),
#   so this is equivalent:
df.iloc[0]
```

Out: $\begin{array}{l,r} 
fname   &    000005.jpg\\
labels  &         chair\\
is\_valid &         True\end{array}$
$\begin{array}{l}&&Name: 0,& dtype: object
\end{array}$

You can also grab a column by name by indexing into a DataFrame directly:

你也能够通过列名直接索引到DataFrame中去来抓起一列：

```
df['fname']
```

Out: $\begin{array}{l,c} 0  &     000005.jpg\\
1      & 000007.jpg\\
2      & 000009.jpg\\
3      & 000012.jpg\\
4      & 000016.jpg\\
       &    ...    \\
5006   & 009954.jpg\\
5007   & 009955.jpg\\
5008   & 009958.jpg\\
5009   & 009959.jpg\\
5010   & 009961.jpg\end{array}$
$\begin{array}{l,l}&&Name: fname,& Length: 5011,& dtype: object\end{array}$

You can create new columns and do calculations using columns:

你能够创建一个新列并用列做计算：

```
tmp_df = pd.DataFrame({'a':[1,2], 'b':[3,4]})
tmp_df
```

<table style="width: 30px;border-collapse: collapse;" >
  <tr>
    <td  style="width: 10px;" align="center"></td>
    <td  style="width: 10px;" align="center"><strong>a</strong></td>
    <td  style="width: 10px;" align="center"><strong>b</strong></td>
  </tr>
  <tr>
    <td align="center" font-weight="bold"><strong>0</strong></td>
    <td align="right">1</td>
  	<td align="right">3</td>
  </tr>
  <tr>
    <td align="center" font-weight="bold"><strong>1</strong></td>
    <td align="right">2</td>
  	<td align="right">4</td>
  </tr>
</table>

```
tmp_df['c'] = tmp_df['a']+tmp_df['b']
tmp_df
```

<table style="width: 30px;border-collapse: collapse;" >
  <tr>
    <td  style="width: 10px;" align="center"></td>
    <td  style="width: 10px;" align="center"><strong>a</strong></td>
    <td  style="width: 10px;" align="center"><strong>b</strong></td>
    <td  style="width: 10px;" align="center"><strong>c</strong></td> 
  </tr>
  <tr>
    <td align="center" font-weight="bold"><strong>0</strong></td>
    <td align="right">1</td>
  	<td align="right">3</td>
    <td align="right">4</td>
  </tr>
  <tr>
    <td align="center" font-weight="bold"><strong>1</strong></td>
    <td align="right">2</td>
  	<td align="right">4</td>
  	<td align="right">6</td>
  </tr>
</table>

Pandas is a fast and flexible library, and an important part of every data scientist’s Python toolbox. Unfortunately, its API can be rather confusing and surprising, so it takes a while to get familiar with it. If you haven’t used Pandas before, we’d suggest going through a tutorial; we are particularly fond of the book [*Python for Data Analysis*](http://shop.oreilly.com/product/0636920023784.do) by Wes McKinney, the creator of Pandas (O'Reilly). It also covers other important libraries like `matplotlib` and `numpy`. We will try to briefly describe Pandas functionality we use as we come across it, but will not go into the level of detail of McKinney’s book.

Pandas是一个快速和灵活的库，它是每个数据科学的Python工具箱一个很重要的部分。不幸的是，它的API让人比较混乱和惊讶，所以需要花一段时间来熟悉它。如果你之前没有用过Pandas，我们建议你通过一个指引来学习使用它。我们特别喜欢由Pandas的创始人韦斯·麦金尼编写的书[Pyhon数据分析](http://shop.oreilly.com/product/0636920023784.do)（欧莱礼媒体发行）。它也覆盖了其它重要的库，如`matploatlib`和`numpy`的内容。我们会尝试简短的描述我们所使用的Pandas功能设计，但是不会达到麦金尼著作的细节水平。

### End sidebar

### 侧边栏结束

Now that we have seen what the data looks like, let's make it ready for model training.

既然我们已经知道了数据的样子，那让我们为模型训练做好准备。

### Constructing a DataBlock

### 创建一个数据块

How do we convert from a `DataFrame` object to a `DataLoaders` object? We generally suggest using the data block API for creating a `DataLoaders` object, where possible, since it provides a good mix of flexibility and simplicity. Here we will show you the steps that we take to use the data blocks API to construct a `DataLoaders` object in practice, using this dataset as an example.

我们如果从一个`DataFrame`对象转换为一个`DataLoaders`对象？对于创建一个`DataLoaders`对象，我们通常建议尽可能使用数据块API，因为它提供了柔性和简洁的完美融合。这里我们使用这个数据集做为一个例子，给你展示实践中使用数据块API来构建一个`DataLoaders`对象的步骤。

As we have seen, PyTorch and fastai have two main classes for representing and accessing a training set or validation set:

- `Dataset`:: A collection that returns a tuple of your independent and dependent variable for a single item
- `DataLoader`:: An iterator that provides a stream of mini-batches, where each mini-batch is a tuple of a batch of independent variables and a batch of dependent variables

正如你看到的，PyTorch和fastai有两个主要的类代表和访问训练集和验证集：

- `Dataset`：一个集合，返回单一数据自变量和因变量的一个元组
- `DataLoader`：一个迭代器，提供一串最小批次，每一最小批次是一个自变量批次和因变量批次的元组

On top of these, fastai provides two classes for bringing your training and validation sets together:

- `Datasets`:: An object that contains a training `Dataset` and a validation `Dataset`
- `DataLoaders`:: An object that contains a training `DataLoader` and a validation `DataLoader`

在这些之上，fastai提供了两个类把你的训练和验证集汇集在一起：

- `Datasets`：一个对象，包含一个训练`Dataset`和一个验证`Dataset`
- `DataLoaders`：一个对象，包含一个训练`DataLoader`和一个验证`DataLoader`

Since a `DataLoader` builds on top of a `Dataset` and adds additional functionality to it (collating multiple items into a mini-batch), it’s often easiest to start by creating and testing `Datasets`, and then look at `DataLoaders` after that’s working.

因为`DataLoader`构建在`Dataset`之上并增加了附加功能（把多个数据项收集在一个最小批次内），通常最简单的开始创建和测试`Datasets`，然后在运行后查看`DataLoaders`。

When we create a `DataBlock`, we build up gradually, step by step, and use the notebook to check our data along the way. This is a great way to make sure that you maintain momentum as you are coding, and that you keep an eye out for any problems. It’s easy to debug, because you know that if a problem arises, it is in the line of code you just typed!

Let’s start with the simplest case, which is a data block created with no parameters:

当你创建一个`数据块`时，我们要一步一步的逐步建立，并用notebook来逐步检查我们的数据。这是一个确保你保持编码动力的一个绝佳方法，然后随时关注出现的任何问题。它很容易调试，因为你知道如果一个问题产生，这个问题就在你刚刚敲击完成的代码行内！

让我们从不传参创建一个数据块这个最简单的例子开始：

```
dblock = DataBlock()
```

We can create a `Datasets` object from this. The only thing needed is a source—in this case, our DataFrame:

这样我们就能够创建一个`Datasets`对象。在这个例子中唯一需要的一个源是—我们的DataFrame：

```
dsets = dblock.datasets(df)
```

This contains a `train` and a `valid` dataset, which we can index into:

它包含了一个我们能够索引到的`训练`和`验证`数据集，

```
len(dsets.train),len(dsets.valid)
```

Out: (4009, 1002)

```
x,y = dsets.train[0]
x,y
```

Out: $\begin{array}{llr}
(&fname  &     008663.jpg\\
&labels         & car person\\
&is\_valid     &           False\\\end{array}$
$ \begin{array}{l}& &&        Name: 4346, &dtype: object,\end{array}$
$ \begin{array}{llllr}&&&  fname       &008663.jpg\\
 &&& labels    &  car person\\
 &&& is\_valid &  False\end{array}$
$ \begin{array}{llr}&&& Name: 4346,& dtype: object&)\end{array}$

As you can see, this simply returns a row of the DataFrame, twice. This is because by default, the data block assumes we have two things: input and target. We are going to need to grab the appropriate fields from the DataFrame, which we can do by passing `get_x` and `get_y` functions:

正如你看到的，这个例子返回了两次数据帧的一行数据。因为这是默认的，数据块假设我们有两件事：输入和目标。我们需要从DataFrame合适的区域中抓取，可以通过传递`get_x`和`get_y`函数来做：

```
x['fname']
```

Out: '008663.jpg'

```
dblock = DataBlock(get_x = lambda r: r['fname'], get_y = lambda r: r['labels'])
dsets = dblock.datasets(df)
dsets.train[0]
```

Out: ('005620.jpg', 'aeroplane')

As you can see, rather than defining a function in the usual way, we are using Python’s `lambda` keyword. This is just a shortcut for defining and then referring to a function. The following more verbose approach is identical:

正如你所看到的，我们使用了Python的`lambda`关键字，而不是在普通的方法中定义一个函数。这只是定义的一个快捷方式，然后引用一个函数。下面是相同的更为冗长的方法：

```
def get_x(r): return r['fname']
def get_y(r): return r['labels']
dblock = DataBlock(get_x = get_x, get_y = get_y)
dsets = dblock.datasets(df)
dsets.train[0]
```

Out: ('002549.jpg', 'tvmonitor')

Lambda functions are great for quickly iterating, but they are not compatible with serialization, so we advise you to use the more verbose approach if you want to export your `Learner` after training (lambdas are fine if you are just experimenting).

We can see that the independent variable will need to be converted into a complete path, so that we can open it as an image, and the dependent variable will need to be split on the space character (which is the default for Python’s `split` function) so that it becomes a list:

Lambda函数对于快速迭代是极好的，但它们不兼任序列化，所以如果你想训练后输出你的`Learner`，我们建议你使用更为冗长的方法。（如果你只是做个尝试，lambdas是非常好的）

```
def get_x(r): return path/'train'/r['fname']
def get_y(r): return r['labels'].split(' ')
dblock = DataBlock(get_x = get_x, get_y = get_y)
dsets = dblock.datasets(df)
dsets.train[0]
```

Out: (Path('/home/jhoward/.fastai/data/pascal_2007/train/002844.jpg'), ['train'])

To actually open the image and do the conversion to tensors, we will need to use a set of transforms; block types will provide us with those. We can use the same block types that we have used previously, with one exception: the `ImageBlock` will work fine again, because we have a path that points to a valid image, but the `CategoryBlock` is not going to work. The problem is that block returns a single integer, but we need to be able to have multiple labels for each item. To solve this, we use a `MultiCategoryBlock`. This type of block expects to receive a list of strings, as we have in this case, so let’s test it out:

实际打开图片并做张量转换，我们会需要做一些列的转换。块类型会提供我们那些步骤。我们能够使用先前已经使用过的相同块类型，但有一个例外：`ImageBlock`会运行良好，因为我们有一个指向有效图像的路径，但是`CategoryBlock`将不会运行。问题是块返回一个单整形，但我们需要能够对每个数据项有多标签。为了解决这个问题，我们使用`MultiCateoryBlock`类型。这个块类型要求接收字符串列表，正如在这个例子中我们已经有的数据那样，让我们测试一下它的输出：

```
dblock = DataBlock(blocks=(ImageBlock, MultiCategoryBlock),
                   get_x = get_x, get_y = get_y)
dsets = dblock.datasets(df)
dsets.train[0]
```

Out: (PILImage mode=RGB size=500x375,
         TensorMultiCategory([0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 1., 0., 0., 0., 0., 0., 0., 0., 0.]))

As you can see, our list of categories is not encoded in the same way that it was for the regular `CategoryBlock`. In that case, we had a single integer representing which category was present, based on its location in our vocab. In this case, however, we instead have a list of zeros, with a one in any position where that category is present. For example, if there is a one in the second and fourth positions, then that means that vocab items two and four are present in this image. This is known as *one-hot encoding*. The reason we can’t easily just use a list of category indices is that each list would be a different length, and PyTorch requires tensors, where everything has to be the same length.

> jargon: One-hot encoding: Using a vector of zeros, with a one in each location that is represented in the data, to encode a list of integers.

正如你看到的，我们的分类列表没有以`CategoryBlock`同样的规则方法编码。在那个例子中，基于在我们词汇中它的位置，我们有一个单整形代表现在分类。虽然在这个例子中，有一个很多0列表，列表中1所在的位置是当前的分类。例如，如果1在第二和第四的位置，这就表示在这个图像中，代表两个和四个词汇数据项。这被称为*独热编码*。我们不能轻易的只用一个分类列表索引，原因是出每一列会是不一样的长度，而PyTorch需要张量，张量里所有事物必须是相同的长度。

> 术语：独热编码：使用包含一个1的0失量（1所在的每个位置是数据中所代表的类型），来编码整型列表。

Let’s check what the categories represent for this example (we are using the convenient `torch.where` function, which tells us all of the indices where our condition is true or false):

在这个例子中，我们检查一下分类代表的是什么（我们使用`torch.where`函数转换，这会告诉我们的条件是真还是假的所有索引）：

```
idxs = torch.where(dsets.train[0][1]==1.)[0]
dsets.train.vocab[idxs]
```

Out: (#1) ['dog']

With NumPy arrays, PyTorch tensors, and fastai’s `L` class, we can index directly using a list or vector, which makes a lot of code (such as this example) much clearer and more concise.

利用NumPy数组，PyTorch张量和fastai的`L`类，我们能够使用列表或失量生成一些更清晰和更简洁的代码（如这个例子）直接索引。

We have ignored the column `is_valid` up until now, which means that `DataBlock` has been using a random split by default. To explicitly choose the elements of our validation set, we need to write a function and pass it to `splitter` (or use one of fastai's predefined functions or classes). It will take the items (here our whole DataFrame) and must return two (or more) lists of integers:

到目前为止，我们已经忽略了`is_valid`列，这表示`DataBlock`已经通过默认的方式使用了随机分割。来明确的选择我们验证集的元素，我们需要编写一个函数并把它传递给`splitter`（或使用fastai先前定义的函数和类的一种）。它会携带数据项（在这里是我们整个DataFrame）并必须返回两个（或更多）整形列表： 

```
def splitter(df):
    train = df.index[~df['is_valid']].tolist()
    valid = df.index[df['is_valid']].tolist()
    return train,valid

dblock = DataBlock(blocks=(ImageBlock, MultiCategoryBlock),
                   splitter=splitter,
                   get_x=get_x, 
                   get_y=get_y)

dsets = dblock.datasets(df)
dsets.train[0]
```

Out: (PILImage mode=RGB size=500x333,
         TensorMultiCategory([0., 0., 0., 0., 0., 0., 1., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0.]))

As we have discussed, a `DataLoader` collates the items from a `Dataset` into a mini-batch. This is a tuple of tensors, where each tensor simply stacks the items from that location in the `Dataset` item.

正如我们已经讨论过的，一个`DataLoader`从一个`Dataset`收集数据项到一个最小批次中。这是一个张量元组，每个张量简单的堆砌来自`Dataset`项中该位置的数据条目。

Now that we have confirmed that the individual items look okay, there's one more step we need to ensure we can create our `DataLoaders`, which is to ensure that every item is of the same size. To do this, we can use `RandomResizedCrop`:

现在我们已经确信独立的数据项是没有问题的，还有一步需要确保我们能够创建每个数据项是相同尺寸的`DataLoaders`。为了做到这一点，我们可以使用`RandomResizedCrop`方法：

```
dblock = DataBlock(blocks=(ImageBlock, MultiCategoryBlock),
                   splitter=splitter,
                   get_x=get_x, 
                   get_y=get_y,
                   item_tfms = RandomResizedCrop(128, min_scale=0.35))
dls = dblock.dataloaders(df)
```

And now we can display a sample of our data:

现在我们能够展示一个我们数据的实例：

```
dls.show_batch(nrows=1, ncols=3)
```

Out: <img src="./_v_images/mulcat.png" alt="mulcat" style="zoom:100%;" />

Remember that if anything goes wrong when you create your `DataLoaders` from your `DataBlock`, or if you want to view exactly what happens with your `DataBlock`, you can use the `summary` method we presented in the last chapter.

记住，当创建来自你`DataBlock`的`DataLoaders`时，如果有任何问题，或如果你希望准确的看到你的`DataBlock`发生了什么，你可以使用在上一章节我们讲解的`summary`方法。

Our data is now ready for training a model. As we will see, nothing is going to change when we create our `Learner`, but behind the scenes, the fastai library will pick a new loss function for us: binary cross-entropy.

我们现在准备好了训练模型的数据。我们会看到，当我们创建我们的`Learner`时没有什么东西会改变，但是背后的景象是，fastai库会为我们选取一个新的损失函数：二值交叉熵。

### Binary Cross-Entropy

### 二值交叉熵

Now we'll create our `Learner`. We saw in <chapter_mnist_basics> that a `Learner` object contains four main things: the model, a `DataLoaders` object, an `Optimizer`, and the loss function to use. We already have our `DataLoaders`, we can leverage fastai's `resnet` models (which we'll learn how to create from scratch later), and we know how to create an `SGD` optimizer. So let's focus on ensuring we have a suitable loss function. To do this, let's use `cnn_learner` to create a `Learner`, so we can look at its activations:

现在来创建我们的`Learner`。在<章节：mnist基础>中我们看了`Learner`对象包含四个主要内容：模型、`DataLoaders`对象、优化器和要使用的损失函数。我们已经有了`DataLoaders`，能够利用fastai的`resnet`模型（后面我们会学习如何从零开始创建这一模型），我们知道如何创建一个`SGD`优化器。所以让我们聚焦在确保我们会有一个合适的损失函数。为此，让我们使用`cnn_learner`来创建一个`Learner`，以便我们能够查看它的激活情况：

```
learn = cnn_learner(dls, resnet18)
```

We also saw that the model in a `Learner` is generally an object of a class inheriting from `nn.Module`, and that we can call it using parentheses and it will return the activations of a model. You should pass it your independent variable, as a mini-batch. We can try it out by grabbing a mini batch from our `DataLoader` and then passing it to the model:

在学习器里我们也会看到模型通常是从`nn.Module`继承的一个类对象，并且我们能够使用圆括号调用它，同时它会返回模型的激活。你应该把你的自变量作为最小批次传给它。我们能够尝试从我们的`DataLoader`中抓取一个最小批次，然后把它传给模型：

```
x,y = to_cpu(dls.train.one_batch())
activs = learn.model(x)
activs.shape
```

Out: torch.Size([64, 20])

Think about why `activs` has this shape—we have a batch size of 64, and we need to calculate the probability of each of 20 categories. Here’s what one of those activations looks like:

想一下，为什么激活是这个形状。我们有一个大小为64的批次，同时我们需要计算20个分类的每个概率。下面是那些激活其中一个的样子：

```
activs[0]
```

Out: TensorImage([ 0.7476, -1.1988,  4.5421, -1.5915, -0.6749,  0.0343, -2.4930, -0.8330, -0.3817, -1.4876, -0.1683,  2.1547, -3.4151,         -1.1743,  0.1530, -1.6801, -2.3067,  0.7063, -1.3358, -0.3715], grad_fn=<AliasBackward>)

> note: Getting Model Activations: Knowing how to manually get a mini-batch and pass it into a model, and look at the activations and loss, is really important for debugging your model. It is also very helpful for learning, so that you can see exactly what is going on.

> 注释：获取模型激活：知道如何手动的获取一个最小批次，并把它传递到模型中去，然后查看激活和损失情况，这对于调试你的模型是非常重要的。它对学习也非常有帮助。因此你能够准确的看到到底发生了什么。

They aren’t yet scaled to between 0 and 1, but we learned how to do that in <chapter_mnist_basics>, using the `sigmoid` function. We also saw how to calculate a loss based on this—this is our loss function from <chapter_mnist_basics>, with the addition of `log` as discussed in the last chapter:

他们还没有缩放到0到1之间，但在<章节：mnist基础>中我们知道如何利用`sigmoid`函数来做。我们也看了如何基于此计算损失函数（这是来自<章节：mnist基础>的损失函数，并增加了在上一章节讨论过的`对数`）：

```
def binary_cross_entropy(inputs, targets):
    inputs = inputs.sigmoid()
    return -torch.where(targets==1, inputs, 1-inputs).log().mean()
```

Note that because we have a one-hot-encoded dependent variable, we can't directly use `nll_loss` or `softmax` (and therefore we can't use `cross_entropy`):

- `softmax`, as we saw, requires that all predictions sum to 1, and tends to push one activation to be much larger than the others (due to the use of `exp`); however, we may well have multiple objects that we're confident appear in an image, so restricting the maximum sum of activations to 1 is not a good idea. By the same reasoning, we may want the sum to be *less* than 1, if we don't think *any* of the categories appear in an image.
- `nll_loss`, as we saw, returns the value of just one activation: the single activation corresponding with the single label for an item. This doesn't make sense when we have multiple labels.

请注意，因为我们有了独热编码因变量，我们不能直接使用`nll_loss`或`softmax`（并且因而我们不能使用`交叉熵`）：

- `softmax`，我们知道它的预测合计需要为1，并倾向输出一个远大于其它的激活（由于`指数函数`的使用）。而然，我们可能会有多个目标我们是确认会出现在图像中，所以限制激活的最大合计为1就不是一个好主意了。基于同样的原因，如果我们不认为*其它*的分类显示在图像中，我们可能希望合计*小于*1。
- `nll_loss`，我们知道它只返回一个激活的值：单激活对应于对于每个数据项的单标签。当我们有多个标签时它就没有意义了。

On the other hand, the `binary_cross_entropy` function, which is just `mnist_loss` along with `log`, provides just what we need, thanks to the magic of PyTorch's elementwise operations. Each activation will be compared to each target for each column, so we don't have to do anything to make this function work for multiple columns.

> j: One of the things I really like about working with libraries like PyTorch, with broadcasting and elementwise operations, is that quite frequently I find I can write code that works equally well for a single item or a batch of items, without changes. `binary_cross_entropy` is a great example of this. By using these operations, we don't have to write loops ourselves, and can rely on PyTorch to do the looping we need as appropriate for the rank of the tensors we're working with.

换个角度说，`binary_cross_entropy`函数只是一个结合了`log`的`mnist_loss`函数，它正好提供了我们所需要的，这要感谢PyTorch神奇的元素操作。每个激活会与每一列的每个目标做对比，因此我们不必做任何事情，就能使得这个函数处理多列。

> 杰：我真正喜欢使用如PyTorch这种具有传播和元素操作的库的原因是，我经常发现我能够编写出对于单一数据项或批次数据项运行同样良好的代码，且不需要改代码。`binary_cross_entropy`是这样一个非常棒的例子。通过使用这些操作，我不必自己编写循环，我们需要对正在处理的数据具有合适的张量阶，且能够依赖PyTorch来做这个循环。

PyTorch already provides this function for us. In fact, it provides a number of versions, with rather confusing names!

PyTorch已经为我们提供了这个函数。实际上，它提供了很多版本，且让人相当迷惑的命名。

`F.binary_cross_entropy` and its module equivalent `nn.BCELoss` calculate cross-entropy on a one-hot-encoded target, but do not include the initial `sigmoid`. Normally for one-hot-encoded targets you'll want `F.binary_cross_entropy_with_logits` (or `nn.BCEWithLogitsLoss`), which do both sigmoid and binary cross-entropy in a single function, as in the preceding example.

`F.binary_cross_entropy`和它的模块相当于`nn.BCELoss`在独热编码目标上计算交叉熵，但是没有包含初始的`sigmoid`。通常对于独热编码目标你会想到`F.binary_cross_entropy_with_logits`（或`nn.BCEWithLogitsLoss`），正如之前的例子那样，在其单一函数里包含了sigmoid和二值交叉熵两者。

The equivalent for single-label datasets (like MNIST or the Pet dataset), where the target is encoded as a single integer, is `F.nll_loss` or `nn.NLLLoss` for the version without the initial softmax, and `F.cross_entropy` or `nn.CrossEntropyLoss` for the version with the initial softmax.

相同的，对于单标签数据集（如MNIST或宠物数据集），其目标做为一个单整形进行了编码，对于`F.nll_loss`或`nn.NLLLoss`的版本是没有包含初始的softmax，而`F.cross_entropy`或`nn.CrossEntropyLoss`的版本包含了softmax。

Since we have a one-hot-encoded target, we will use `BCEWithLogitsLoss`:

因为我们有独热编码目标，我们会使用`BCEWithLogitsLoss`：

```
loss_func = nn.BCEWithLogitsLoss()
loss = loss_func(activs, y)
loss
```

Out: TensorImage(1.0342, grad_fn=<AliasBackward>)

We don't actually need to tell fastai to use this loss function (although we can if we want) since it will be automatically chosen for us. fastai knows that the `DataLoaders` has multiple category labels, so it will use `nn.BCEWithLogitsLoss` by default.

我们实际上并不需要告诉fastai来使用这个损失函数（虽然如果我们想的话，我们可以这么做），因为它会自动为我们进行选择。fastai知道`DataLoaders`有多分类标签，所以它会默认的使用`nn.BCEWithLogitsLoss`。

One change compared to the last chapter is the metric we use: because this is a multilabel problem, we can't use the accuracy function. Why is that? Well, accuracy was comparing our outputs to our targets like so:

与上一章节相比有一个改变是我们用的指标，因为这是一个多标签问题，我们不能使用精度函数。这是为什么呢？好吧，精度是我们的输出与我们目标的对比，如下所求：

```python
def accuracy(inp, targ, axis=-1):
    "Compute accuracy with `targ` when `pred` is bs * n_classes"
    pred = inp.argmax(dim=axis)
    return (pred == targ).float().mean()
```

The class predicted was the one with the highest activation (this is what `argmax` does). Here it doesn't work because we could have more than one prediction on a single image. After applying the sigmoid to our activations (to make them between 0 and 1), we need to decide which ones are 0s and which ones are 1s by picking a *threshold*. Each value above the threshold will be considered as a 1, and each value lower than the threshold will be considered a 0:

分类预测的是最高激活的那个值（这就是`argmax`做的事情）。在这里它不奏效是因为在一张图像上我们有超过一个的预测。在我们的激活应用sigmoid后（使得激活在0到1之间），我需要通过选择一个*阈值*来决定哪些是0，哪些是1。每个大于阈值的值会被识做为1，每个小于阈值的会被识做为0：

```python
def accuracy_multi(inp, targ, thresh=0.5, sigmoid=True):
    "Compute accuracy when `inp` and `targ` are the same size."
    if sigmoid: inp = inp.sigmoid()
    return ((inp>thresh)==targ.bool()).float().mean()
```

If we pass `accuracy_multi` directly as a metric, it will use the default value for `threshold`, which is 0.5. We might want to adjust that default and create a new version of `accuracy_multi` that has a different default. To help with this, there is a function in Python called `partial`. It allows us to *bind* a function with some arguments or keyword arguments, making a new version of that function that, whenever it is called, always includes those arguments. For instance, here is a simple function taking two arguments:

如果我们直接传递`accuracy_multi`做为一个指标，它会有默认为0.5的阈值。我们可能希望调整这个默认值，并创建一个与默认值不同的新的`accuracy_multi`版本。为了完成这个工作，在Python中有一个名为`partial`函数。它允许我们*绑定*一个有一些参数或关键值参数的函数，来对这个函数生成 一个新的版本，无论何时调用它，总会包含那些参数。这里有一个取两个参数的函数例子：

```
def say_hello(name, say_what="Hello"): return f"{say_what} {name}."
say_hello('Jeremy'),say_hello('Jeremy', 'Ahoy!')
```

Out: ('Hello Jeremy.', 'Ahoy! Jeremy.')

We can switch to a French version of that function by using `partial`:

通过使用`partial`我们能够转换为一个这个函数的法语版本：

```
f = partial(say_hello, say_what="Bonjour")
f("Jeremy"),f("Sylvain")
```

Out: ('Bonjour Jeremy.', 'Bonjour Sylvain.')

We can now train our model. Let's try setting the accuracy threshold to 0.2 for our metric:

我们现在能够训练我们的模型了。对于我们的指标，让我们尝试设置精度阈值为0.2：

```
learn = cnn_learner(dls, resnet50, metrics=partial(accuracy_multi, thresh=0.2))
learn.fine_tune(3, base_lr=3e-3, freeze_epochs=4)
```

| epoch | train_loss | valid_loss | accuracy_multi |  time |
| ----: | ---------: | ---------: | -------------: | ----: |
|     0 |   0.942663 |   0.703737 |       0.233307 | 00:08 |
|     1 |   0.821548 |   0.550827 |       0.295319 | 00:08 |
|     2 |   0.604189 |   0.202585 |       0.816474 | 00:08 |
|     3 |   0.359258 |   0.123299 |       0.944283 | 00:08 |

| epoch | train_loss | valid_loss | accuracy_multi |  time |
| ----: | ---------: | ---------: | -------------: | ----: |
|     0 |   0.135746 |   0.123404 |       0.944442 | 00:09 |
|     1 |   0.118443 |   0.107534 |       0.951255 | 00:09 |
|     2 |   0.098525 |   0.104778 |       0.951554 | 00:10 |

Picking a threshold is important. If you pick a threshold that's too low, you'll often be failing to select correctly labeled objects. We can see this by changing our metric, and then calling `validate`, which returns the validation loss and metrics:

选取一个阈值是非常重要的。如果你选取的阈值太低，你会经常无法选择正确的标签目标。我们能够通过改变我们的指标来观察，然后调用`validate`，返回验证的损失和指标：

```
learn.metrics = partial(accuracy_multi, thresh=0.1)
learn.validate()
```

Out: (#2) [0.10477833449840546,0.9314740300178528]

If you pick a threshold that's too high, you'll only be selecting the objects for which your model is very confident:

如果你选择的阈值太高，你只会看到对你的模型非常确信的选择目标：

```
learn.metrics = partial(accuracy_multi, thresh=0.99)
learn.validate()
```

Out: (#2) [0.10477833449840546,0.9429482221603394]

We can find the best threshold by trying a few levels and seeing what works best. This is much faster if we just grab the predictions once:

我们能够通过尝试一些等级和查看最佳实践查找最佳阈值。如果我们只抓取一次预测，这会更快些：

```
preds,targs = learn.get_preds()
```

Then we can call the metric directly. Note that by default `get_preds` applies the output activation function (sigmoid, in this case) for us, so we'll need to tell `accuracy_multi` to not apply it:

然后我们能够直接调用指标。注意，通过默认的方式`get_preds`会为我们应用输出激活函数（在这个例子中是sigmoid），所以我们需要告诉`accuracy_multi`不用应用它：

```
accuracy_multi(preds, targs, thresh=0.9, sigmoid=False)
```

Out: TensorImage(0.9567)

We can now use this approach to find the best threshold level:

现在我们能使用这一方法来查找最优的阈值水平：

```
xs = torch.linspace(0.05,0.95,29)
accs = [accuracy_multi(preds, targs, thresh=i, sigmoid=False) for i in xs]
plt.plot(xs,accs);
```

Out: ![img](data:image/png;base64,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)

In this case, we're using the validation set to pick a hyperparameter (the threshold), which is the purpose of the validation set. Sometimes students have expressed their concern that we might be *overfitting* to the validation set, since we're trying lots of values to see which is the best. However, as you see in the plot, changing the threshold in this case results in a smooth curve, so we're clearly not picking some inappropriate outlier. This is a good example of where you have to be careful of the difference between theory (don't try lots of hyperparameter values or you might overfit the validation set) versus practice (if the relationship is smooth, then it's fine to do this).

在这个例子中，我们使用验证集来选择一个超参（阈值），这就是验证集的用途。有时学习们会表达他们的顾虑，我们在验证集上可能会*过拟*，因为我们尝试了太多的值来看那个是最好的。然而，正如你在图中看到的，在这个例子中阈值改变的结果是一个平滑的曲线，所以我们显然不会选择那些离群值。这是一个好的例子，我们必须关注理论（不要尝试太多超参值，否则你可能会过拟验证集）与实践（如果关联是平滑的，那么这样做就好了）间的差别。

This concludes the part of this chapter dedicated to multi-label classification. Next, we'll take a look at a regression problem.

本章节介绍多标签分类的部分结束了。下面，我们会看一下回归问题。