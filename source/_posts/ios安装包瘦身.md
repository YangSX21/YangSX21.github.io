---
title: ios安装包瘦身
date: 2017-09-10 14:51:09
tags: ios
---
>入职两个多月，最近的一个月做的都是安装包瘦身，其实最后效果也不是特别明显，但还是整理一下，因为之后大概不做这个瘦身了。
<!--more-->

#### 安装包结构
打开.app文件可以清楚的看到安装包包括可执行文件，资源文件，配置文件以及第三方的bundle包。

其中第三方的bundle其实也是静态的，作为第三方的framework库的一种资源文件进行使用，而且因为是第三方制作，也不好进行精简；而配置文件占比小，没必要进行精简。所以主要需要精简的是资源文件和可执行文件。

#### 资源文件的精简
资源文件主要是图片资源，声音资源，文本文件以及StoryBoard等，其中StoryBoard的本质也是一种文本文件。

文本文件没用就删除，有用就保留，占比很小，不用特意理会。
声音文件占比也不多，如果有点大可以用如下命令进行压缩。
```
afconvert -f AIFC -d ima4 tritone.caf
```

我们主要需要考虑的是**图片资源**，图片资源的处理分为冗余图片的删除和大图片的压缩。
##### 图片资源的压缩
在ios开发中，图片资源是可以通过bundle，CreateGroup和Xcassets等方式调用，我建议图片可以都转为png或者pdf后通过Xcassets调用，这样xcode会自动进行无损压缩，效率很高。
###### Xcassets
Xcassets引入png，是按照引入时设置生成相对应的1x，2x或者3x图出现在安装包的Assets.car中，Xcassets引入pdf，会把pdf作为1x图的尺寸，自动生成1x，2x，3x图，和压缩后的pdf原件一并出现在Assets.car文件中。并且由于苹果的App Thining机制，在用户下载时会根据相应的机型下载对应的图片资源。非常方便不会冗余。推荐拆Assets.car的工具：[themeEngine](https://github.com/alexzielenski/ThemeEngine)，另外还使用过[cartool](https://github.com/steventroughtonsmith/cartool)，好像解压的图片不够全。
###### 其他
如果使用别的方式引入位图，可以用[ImageOptiom](https://imageoptim.com/howto.html)进行压缩，在80%即以上属于无损，低于80就是有损压缩了，这个压缩工具蛮方便的，可以整个文件夹批量导入图片进行压缩。

##### 冗余图片的删除
推荐：[LSUnusedResources](http://blog.lessfun.com/blog/2015/09/02/find-unused-resources-in-xcode-project/)，很好用，没有什么差错，直接用这个好了。

另外其实还有一种冗余：有些人在引入1x，2x，3x图的时候，用了两个一样的图分别放入2x，3x。这样是完全没用的吧。Python写了个脚本，搜出来删掉了，开发了一个[脚本](https://github.com/YangSX21/Script-repository/tree/master/%E6%B8%85%E9%99%A4%E5%86%97%E4%BD%99%E5%9B%BE%E7%89%87)。可能不太好用，需要自行修改部分。
#### 可执行文件的精简
首先推荐付费ios开发工具，[AppCode](https://www.jetbrains.com/objc/)。可以使用inspect code功能进行无用类，方法等扫描。从网上口碑来说用起来不错。至于本人，在下载之后的试用期间，大概是公司项目太庞大了，一整天都没扫描完。而且公司也没有出钱买这个软件的意思……

所以只能自己动手，查资料开始手动精简了。
##### 文件结构的分析
首先必须分析可执行文件的组成，可执行文件的分析主要是通过两种方法。

一种是直接分析可执行文件，一般是通过[MachOview](https://github.com/gdbinit/MachOView)工具解开Mach-O文件进行分析，不知道是因为公司的Mac加了限制还是其他原因，我没办法运行这个软件，所以不清楚具体怎么做。

另一种是分析LinkMap文件获得可执行文件的结构构成，LInkMap文件是文件编译过程中生成的符号表文件，包含了整个可执行文件的代码段，存储位置之类的信息。
###### LInkMap文件分析
在Xcode中Build Setting->Linking->Write Link Map Files的勾打上，在Path to Link Map File选项中的地址就能看到在编译时生成的LinkMap文件，这是个纯文本文件，可以用文字编辑器直接打开。

LInkMap文件主要是三部分，第一部分列出的是目标文件列表，第二部分是一个段表，描述各个段在最后编译成的可执行文件中的偏移位置及大小，最后一部分按上表顺序，列出具体的按每个文件列出每个对应字段的位置和占用空间。描述相当有序，可以尝试用脚本直接输出每个模块的相应大小。

这个博客给出了一个[脚本](http://www.cnblogs.com/LeeGof/p/6803146.html?utm_source=itdadao&utm_medium=referral)，使用上来说问题不大。

###### *模块精简
上部分的LinkMap能够直接分析出每个模块所占据的大小，如果有非常大的模块，特别有些第三方模块，可以尝试进行精简，这部分其实是最大的一块瘦身项目。如果有些第三方模块占据很大空间，所用功能其实又不多的话，强烈建议重新自己写一下这部分功能，会是一个很大的瘦身点。

再提一句，我公司的项目是包括一个Watch App，Watch App包括的第三方库和主App的是通用的，然后发现是分别编译的，一下子变成了两倍大……我拿到瘦身任务之后，LinkMap一分析，直接把Watch App的第三方库中最大的那个，把有用的代码剥离出来之后少了十几M，完成了任务。所以你的项目如果有Watch App，不妨排查一下。

可执行文件的精简可以分为**类级别的精简**和**方法级别的精简**。
##### 类级别的精简
有些类被申明之后没有被使用过，这就是冗余的类，可以删除。基于此，查找冗余类，首先需要获得项目中所有被引用到的类，然后需要定义使用的标准，查找出所有被使用的类，求差集之后获得冗余类。

下面说到的代码都在[这里](https://github.com/YangSX21/Script-repository/tree/master/%E6%B8%85%E9%99%A4%E5%86%97%E4%BD%99%E6%96%87%E4%BB%B6)。

###### 所有被引用的类

首先读取项目文件夹下面的所有.m/h文件，
```
def Getallfile(rootDir): #遍历目录下所有文件
    for lists in os.listdir(rootDir): 
        path = os.path.join(rootDir, lists) 
        if os.path.isdir(path): #递归
            Getallfile(path) 
        else:
            ex = os.path.splitext(path)[1]  #根据扩展名把resource和projectFlie分出来
            if ex == '.m' or ex == '.mm' or ex == '.h':
                resourcefile.append(path)
                writeAllResourceFile.append(path + '\n')
```
这个其实并不是所有被引用的文件，由于有部分文件会放在文件夹中但是没有在Xcode中加入，这也算是废文件，但因为没有引用，不会对安装包大小产生影响，所以不用管。

我搜出这些文件主要是为了获得文件地址，之后要用相应的地址来读取文件。

被引用文件的列表在.pbxproj，该文件显示了一个project的整体配置，包括工程文件关联信息，如PBXBuildFile、PBXFileReference，组织结构分类信息，如PBXGroup，项目工程配置信息，如XCBuildConfiguration、XCConfigurationList。具体形式不需要太过清楚，我们只需要抓取其中.m/h结尾的字符串就行了。
```
for e in pbxprojFile:
    f = open(e, 'r')
    content = f.read()
    #正则表达式以h，m结尾，前面是字母的字符串
    array = re.findall(r'\s+([\w,\+]+\.[h,m]{1,2})\s+',content)
    #print array
    see = set(array)#把list变成了set 
    totalClass = totalClass|see
    writeAllReferenceFile = writeAllReferenceFile|see
    f.close()
```
然后把上面获得的文件地址和这个引用的文件列表相对应就能获得引用的文件及其地址。

接下来通过@interface这个前缀进行正则查找，遍历所有的被引用文件，找出所有被引用的类。代码如下，
```
for x in resourcefile:#去掉了未引用的.m.mm.h文件之后，文件夹内的文件
    f = open(x,'r')
    y = x + '\n'
    ff.writelines(y)
    content = f.read()
    array = re.findall(r'@interface\s+([\w,\+]+)\s+:',content)#类名
    for xx in array:
        allClassDic[xx] = x#文件中所有interface的类的路径，value为path，key为类名
    f.close()
ff.close()
```
###### 所有被使用的类
首先，被plist，xml，json文件提到的类，认为是有用的，读取plist，xml，json文件的代码与上面类似，不赘述。

接下来就是在所有被引用的.m和h文件中进行检索，在h文件中的使用归纳为继承，建立指针和define，在m文件中是建立对象，建立指针和动态调用，代码如下。
```
    if os.path.splitext(path)[1] == '.h':
        match = re.search(r'((#define|\:|\[)\s*(%s)(\s+|\<|\*))' % className, content)#继承
        #match = re.search(r':(\s+(%s)\s+)' % className,content)#继承
        if not match:
           match = re.search(r'(%s)\s*\*' % className,content)#建指针
    else:
        match = re.search(r'(%s)\s+\w+' % className,content)#建对象
        if not match:
            match = re.search(r'@"(%s)"' % className,content)#动态调用
            if not match:
                match = re.search(r'(%s)\s*\*' % className,content)#建指针
```
至此大概获得了所有已经被使用的类。与上面获得的所有被引用的类求差集之后能够获得所有未使用的类，这就是能删除的类了。
###### 删除未使用类
上面获得的未使用类如果不多的话，推荐还是手动排除一遍删了吧。

如果很多就只能依靠程序了，我现在想来读取每个文件对定义类的部分进行注释是个好主意，但是当时在师傅推荐下，直接把未使用的类推广到未使用的文件进行了删除，确实好做了不少，但应该有不少的缺漏。下面是我的思路。

一般一个文件只会定义一个类，所以类推到文件没有太多的缺失。（大概没错吧……）遍历被引用的文件，如果该文件中声明的类都是未使用的类，认为该文件是无用的，在pbxproj文件中直接删去引用。代码如下，
```
def filiter(NouseDir,generFile):
    classname = []
    f = open(NouseDir,'r')
    g = open(generFile,'w+')
    while 1:
        string = f.readline()
        if '：' in string:
            line = string.split('：')[2]
            name = string.split('：')[1].split(' ')[0]
            classname.append(name)
            line2 = line[:-1]
            ff = open(line2,'r')
            comtent = ff.read()
            ff.close()
            array = re.findall(r'@interface\s+([\w,\+]+)\s+:',comtent)
            if set(classname) & set(array) == set(array):
                g.writelines(line2 + '\n')
        if not string:
            break
    f.close()
    g.close()
```
这个方法明显是有缺漏的，比如该文件中定义了全局的变量，该文件定义了某个宏之类的。所以用这个方法，记得手动筛查一遍。

筛查好的文件，主要是要在pbxproj文件中删掉，在.m/.h中注释掉。

pbxproj文件处理。
```
for x in phxPath:#按照pbx文件进行查找删除，同时备份在指定目录
	index += 1
	print'共有',total,'个，正在处理第',index,'个pbx。'
	with open(x,'r') as f:
		lines = f.readlines()
		shutil.copy(x,saveDir)#把pbx文件进行复制
		Dirname = os.path.split(x)[0]#取文件的上一级名称进行重命名
		Dirname2 = os.path.split(Dirname)[1]
		os.rename(saveDir+'/project.pbxproj',saveDir+'/'+os.path.splitext(Dirname2)[0]+'.pbxproject') 
	with open(x ,'w+') as g:
		for line in lines:
			for pattern in patterns:#按照引用文件进行查找
				match = re.search(r'\s+(%s)\.' % pattern, line)#匹配以空格开头以.结尾的行
				if match:
					flag = False
			if flag:
				g.writelines(line)
			flag = True
```
m、h文件处理。
```
for pattern in patterns:#按照所有引用的文件进行进行查找注释
	index += 1
	print'共有',total,'个，正在处理第',index,'个引用文件。'
	for WillComFile in AllFilePath:
		if pattern in WillComFile:
			pass
		else:
			with open(WillComFile,'r') as f:
				lines = f.readlines()
			with open(WillComFile,'w') as g:
				for line in lines:
					match = re.search(r'(\"|\/)(%s)\.h' % pattern,line)#匹配以“或者\开头，以.h结尾的行
					if match:
						g.writelines('//'+line)#把相应行注释
					else:
						g.writelines(line)
```
##### 方法级别的精简
这个其实我没有做好，可以主要参考下这篇[文章](http://www.jianshu.com/p/a53480ad0364)。不只是文章中说的那个工具，它还提到了微信和阿里安装包瘦身用的一些方法，但都没有具体工具，思路可以参考一下。


#### 参考资料
1. [Ios安装包瘦身的那些事](http://www.cnblogs.com/LeeGof/p/6803146.html?utm_source=itdadao&utm_medium=referral)
2. [ios开发之静态库](http://www.cnblogs.com/mylizh/p/3971428.html)
3. [手游开发者“利器”：苹果应用瘦身功能设置](http://www.cocoachina.com/ios/20150612/12135.html)
4. [ipa包大小之LinkMap文件分析](https://sanwen8.cn/p/5942YDZ.html)
5. [project.pbxproj最熟悉的陌生人](http://www.olinone.com/?p=215)
6. [使用Swift3开发了个……一键全部清理](http://www.jianshu.com/p/a53480ad0364)