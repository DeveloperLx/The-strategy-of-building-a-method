# 论构造方法的方法论

* 好久没有写博了。究其原因，是觉得值得讨论的，早已被各大咖拆解得一清二楚，再无需赘述了，只需有心人挖掘。

* 但是探求的空间还是有的。这不，最近刚刚换了一份工作，入职的第一件任务，就是熟悉公司早已开发n长时间的大项目，在最短的时间内上手更新维护，这当然是大多数程序员都经历过的。熟悉着熟悉着，就发现了一个非常知得讨论的问题：
	
	**为什么有的代码读得就那么轻松愉快，得心应手；**  
	**而有的代码读起来就那么痛苦不堪，步履维艰？**

* 需要指出的是，这个项目其实基本的代码规范大体上遵循得还是不错的；但仍然有诸多的地方，读来甚是头痛不已。

* 其实造成这种状况的原因有很多。但铺开大面地讲笔者并不喜欢，想来想去，挑了一个个人认为比较重要的原因，于是乎今天的题目有了：


##论构造方法的方法论
---
* 读来确是有些拗口，而且打个前哨，本文倘若是由开发经验丰富的老程序员看，可能会有种“说废话”的感觉，但当你中了它的毒，被洗了脑，惯下了它的强迫症后，你的代码品质一定会有一个质的提升，这就是其中价值的体现。

* 话休絮烦。我想以iOS为例，要讲构造方法的方法论，其实最好的素材就是苹果自身的那些api。我们花点心思分析分析，就可以体会到，实际上所有的方法，根据其命名和内容的特征，大体都可以分为两大类：**调用时机式** 和 **本身作用式**  

**下面就来一一介绍：**

### 调用时机式
---
* 这类方法的特点是，它的方法名称描述的都是调用该方法的时机，常包含时态的英语语法（如did，will等），且通常都不会含有返回值。其内容也比较便于重载复写，自由实现。通常这种形式常见于抽象基类声明的方法，子类重写父类的方法，代理方法，block，函数指针指向的函数等等地方
    
**例如：**  


1. - (void)viewDidLoad;
2. - (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath;
3. completion:(void (^ __nullable)(void))completion 	//	在某方法中的一个参数的声明
4. void CGPDFDictionaryApplyFunction(CGPDFDictionaryRef dict, CGPDFDictionaryApplierFunction function, void * info);  
//	typedef void (*CGPDFDictionaryApplierFunction)(const char * key, CGPDFObjectRef value, void * info); 其中的function参数，由用户实现的符合 CGPDFDictionaryApplierFunction 类型的函数，即属于“调用时机式”

>

* 可以看出，它们的方法名称说明的都是什么时机，或什么条件下会调用这个方法，而并不说明这些方法中具体做了什么，那些将交给实现者自由设定。因此我称之为“调用时机式”。在我们书写类似用途的方法时，都应遵循这类方法的特点，使得你的代码便于被理解：`这方法做的，就是它名称所描述的那个时机所要执行的事！方法名说明的是调用它的时机，具体做了什么跟方法名没关系！！`

###本身作用式
---
* 与调用时机式相反，这类方法的特点是，它的方法名称描述的就一定是它实实在在所实现的行为。

* 栗子放到下面再讲，现在我们首先需要指出的是，对于这类方法，它的方法名一定要确切地描述它事实所做的事，既不要存在描述了却未实现的内容，也不要有实现了却未描述到的东西，这样构造的方法才会真正得易读易懂，原子性强，轻松掌握。

* 实际我们又可将这类方法分为四小类方法。我把它们分别称作**一般工作式**、**成败状态式**、**索要返回式**、**经手加工式**，我们来一一介绍它：

######一般工作式
* 最最简单的一种形式，它的方法名称是一动作短语，并且没有返回值，完成比较单一、直白的动作。并且一定会造成超出其方法作用域范围之外的影响，比如说修改了全部变量，发出了网络请求，记录了日志log，播放了视频音频，进行了磁盘的写操作，更新了数据库等等（不然你想如果一个方法只影响了它内部的局部变量，又没有返回值，能有毛用？）  
**例如：**
    1.  - (void)reloadData;
    2.  - (void)sendEvent:(UIEvent *)event;
    3.  - (void)removeAllObjects;  
    等等  
* 常会发现，这类方法和一些“调用时机式”的方法长得是挺像的，但一定要注意这者之间构造思想上的本质差别，方法名和内容之间迥然不同的关系。

######成败状态式
* 和上面的"一般工作式"类似，它的方法名称也是一动作短语，其内容也是完成比较单一、直白的工作，并且一定会造成超出其方法作用域范围之外的影响。但有所不同的是，它所执行的操作是失败的可能性比较高的操作，如说访问磁盘文件、寻找需求对象等等，因此就一定要有一个布尔类型的返回值来表明执行的成功与否。
    例如：
    1.  - (BOOL)becomeFirstResponder;
    2.  - (BOOL)openURL:(NSURL*)url;
    3.  - (BOOL)sendAction:(SEL)action to:(id)target from:(id)sender forEvent:(UIEvent *)event;
    4.  - (BOOL)createFileAtPath:(NSString *)path contents:(NSData *)data attributes:(NSDictionary *)attr;  
    等等  
* 而有时这类方法为了说明中间发生错误的原因，还会多加入一个供输出错误的参数：(NSError **)error
* 比如：
		- (BOOL)createDirectoryAtPath:(NSString *)path withIntermediateDirectories:(BOOL)createIntermediates attributes:(NSDictionary *)attributes error:(NSError **)error;
		其中返回值和输出参数皆用来表明执行过程中可能发生的错误。
    
######索要返回式
* 这是一类目的性极强的方法。它整个方法的核心就是那个返回值，实现过程也完全地围绕生成那个返回值而展开。它的方法名称，通常即为表达它的返回值的名词短语（有些编程语言中会加个“get”前缀），或是生成返回值的动作过程   
**例如：**
    1.	- (UIResponder *)nextResponder;
    2.	- (BOOL)isFirstResponder;
    3.	- (CGRect)convertRect:(CGRect)rect toView:(nullable UIView *)view;
    4.	- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath;  
    等等
    
* 注意区别，返回值为BOOL类型的，不见得就是”成败状态式“的，也有可能就是“索要返回式”的！一定要好好看一看它的方法名字是不是就代表着那个返回值！
* 对于这种方法，我们尤其要注意避免其实现过程中出现副作用。也就是说，所有实现的代码都要“全心全意”地为生成那个返回值服务，不要做与这个过程无关的任何事情。这样的话我们使用这个方法才会像引用一个常量一样容易，而不必担心造成与访问那个返回值无关的影响。
    	
######经手加工式
* 这类方法我认为就是专为函数式编程而生。它的返回值通常都是被当前方法加工过，又便于下一个“经手加工式”的方法继续处理的变量，方法名称则常由一个非常简单的名词或动词构成，这样才便于写出链式优美的代码  
**例如：**   
	1. masonry中MASConstraint类的这个方法： - (MASConstraint *)centerX;
    2. ReactiveCocoa中RACStream类的这个方法： - (RACStream *)filter:(BOOL (^)(id value))block;  
	等等
* 返回值又立刻是要执行下一个方法的对象，如此就将若干方法连接在了一起，像流水线一样地加工一件产品，构成了非常巧妙的“经手加工式”方法。  

> 总结起来，本身作用式的方法关键的规范就是两点：一是“文要对题”，方法的名称就要真真切切，不多不少地描述了它所实现的事；二是关于返回值的逻辑，一个方法若有返回值，那这个方法就一定应当是成败状态式，索要返回式，经手加工式中的一种，千万不要毫无厘头地抛出返回值，然后再靠一堆注释来予以说明，变成了一个四不像式的方法。

---

####总结：

* 好了，本文所涉及的所有方法的类型都已讲完了。接下来要告诉大家的是，笔者认为，所有构造合理的方法就是上面的这么几种！你没有听错，换句话讲，就是讲倘若一个方法不再以上任何一种分类中，笔者就认为那是一个构造得不够科学的方法！因为你无法就凭一个方法名，就充分地掌握了它实际的内容，还有返回值的含义，掌控不了它可能会造成的我们预期之外的影响。这对阅读代码的效率来讲，无疑是一个巨大的降低，使得代码难以维护和重用，给我们的工作带来非常不利的影响。因此，我们在构造方法时，就一定要多一些思考，遵循其一般性的特点和规律，尽量地减少其别扭的感觉。要知道将来需要阅读、维护它的，很有可能就是早已忘得一干二净的自己！

* 以上都是笔者整理自身在工作实践中遇到的问题得出的结论，其中难免有倏忽之处，毕竟在这个地方领域还从未有人给出过系性统的归纳的总结。望请各位前辈不吝赐教与批评指正，欢迎提出不同的意见和想法，以帮助这方面的理论更加完善！