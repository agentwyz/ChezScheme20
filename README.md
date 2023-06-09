The Development of Chez Scheme 

Introduction:
《Chez Scheme》第一版于1984年完成，并于1985年发布。如今，我发现自己在二十多年后仍然从事与它相关的工作，这令我感到惊讶。

如果在1985年被问及对未来二十年的展望，我本会说《Chez Scheme》和Scheme本身早已成为历史的尘埃。毕竟，当时最古老的编程语言也不过是Scheme的年龄，而且许多编程语言已经兴衰更替。

尽管自那时以来有许多编程语言的兴衰，但以约1960年的Lisp为根基的Scheme依然存在。现在的用户群体比以往任何时候都更大更多元化，所以幸运的话，这门语言和实现至少还能再存在二十年。这是一个令人胆战心惊的想法。

持久性与适应性有关，当前版本的《Chez Scheme》与1985年的初版相比确实有很大的差异。它实现了一个更大、不同的语言，并拥有更丰富的编程环境。编译器和存储管理系统都更加复杂。初始版本只能运行在某一种体系结构和操作系统下，而现在的系统可以在多种不同的计算平台上运行，并且曾经支持过其他许多平台。然而，Version 7背后的原则与Version 1相同。我们的主要目标仍然是可靠性和效率。一个可靠的系统是指完全正确实现整个语言，并且由于编译器或运行环境的故障而从不崩溃。一个高效的系统是指在所有操作方面都表现出良好的性能，具有快速的编译器，能够为各种程序和编程风格生成高效的代码。多年来，虽然我们增加了许多新功能，并通过更好的反馈和调试支持改进了系统的可用性，但我们始终以我们的主要目标为依据进行改进。

上述段落最早出现在2005年出版的《Chez Scheme Version 7 用户指南》[21]的前言中。当然，该用户指南继续记录了语言的当前状态，并未再提到系统的历史。本文的目的是探索这个历史，回答系统如何以及为何成为今天的样子。

本文的其余部分首先简要描述了与《Chez Scheme》有某种先驱关系的系统（第2节）。然后描述了《Chez Scheme》初版和后续版本的动机，以及在这些版本中出现的一些重要的新语言特性或实现技术（第3至10节）。本文最后通过一些总结性的话语结束（第11节）


前身

《Chez Scheme》并非凭空出现。接下来将描述我在《Chez Scheme》之前参与过的几个Scheme或Lisp系统，这些系统在某种程度上影响了《Chez Scheme》的设计和实现。



Scheme分布式进程（Scheme Distributed Processes，简称SDP）是一个多线程的Scheme实现，由我和印第安纳大学的研究生同学Rex Dwyer于1980年至1981年编写。该系统主要用于研究Per Brinch Hansen提出的并发性分布式进程模型[34]。它源自于丹·弗里德曼在他的研究生编程语言研讨会上给我们的任务，他使用了他和鲍勃·菲尔曼（Bob Filman）编写的关于并发编程技术的书籍[32]进行教学。

SDP支持了Scheme的1978年（修订报告）版本的大部分子集[52]。除了并行处理的扩展之外，SDP还支持数组、部分应用机制、用于加载和保存定义的dskin和dskout函数，甚至还有一个结构编辑器。SDP在语义上有所偏离Scheme，通过区分false和空列表，并要求列表的cdr也必须是一个列表。SDP完全使用Simula[3]编写，并利用Simula的运行时系统，包括最重要的垃圾回收器。

尽管SDP的代码没有在我后来编写的任何Scheme系统中留存下来，但它为我提供了接触Scheme和实现任何一种Lisp方言的第一次经验。

Z80 Scheme
1981年，在我和乔治·科恩（George Cohn）作为系统程序员在印第安纳大学的学术计算中心工作时，我们决定为Z80微处理器创建一个Scheme实现。通过这样做，乔治可以教我如何使用Z80汇编语言，而我可以教他Scheme。乔治是一位非常出色的程序员，我从他那里学到了很多，我相信这是一个很好的交易。

我们完全使用Z80汇编语言在CP/M [48]操作系统下编写了Z80 Scheme系统。该系统支持了1978年版本的Scheme，包括完整的continuations。所有的值占据了32位（两个16位字）或由32位值的链接链组成。由于所有对象都按照32位边界对齐，我们能够使用每个字的低位两位进行标记和垃圾回收。该系统是解释执行的，包括一个简单的标记-清除收集器和空闲链表分配器。

大约一年后，我们创建了Z80 Scheme系统的第二个版本，进行了两个重大改变。首先，我们将收集器转换为标记-清除-压缩收集器，重新发明了似乎是由丹尼尔·爱德华兹（Daniel Edwards）首次提出的“双指针”压缩算法[49]。这个算法实际上比原来的运行速度更快，并且它使我们能够使用更快的内联分配。其次，我们取消了对完整continuations的支持，以便使用传统的递归栈。因为栈和堆是相向增长的，压缩堆也确保系统在内存真正耗尽之前不会耗尽栈或堆空间。总体而言，这个系统比原来的系统快得多，但我对失去对完整continuations的支持感到遗憾。

我们还尝试为Z80 Scheme系统编写一个编译器。我们设计了一种类似于Lisp 1.5 Lisp Assembly Language（LAP）[45]的Scheme汇编语言，将其实现为一组库例程以节省代码空间，并从Scheme源代码生成一系列对这些库例程的调用。不幸的是，由于库调用的开销，该系统的运行速度并不比直接解释器快，而且生成的代码比原始源代码还要大，所以我们放弃了这个编译器。


在1982年，我还开始实现了一种新的Scheme方言，Curry Scheme（后来简称为C-Scheme）[14]。该系统使用了一个预处理器，用Scheme编写，并通过Z80 Scheme系统进行引导。预处理器执行宏展开，并负责对应用和lambda表达式进行柯里化。这是我第一次编写宏展开器的经验，也是我第一次接触引导过程。

运行时系统和解释器最初在Z80上用Pascal实现，但后来改用VAX上的C重新编码。存储管理系统采用了内存的大页面包（Bi-BOP）表示，其中内存被分割成固定大小的段，并使用单独的段表来标识每个段中包含的对象的类型[51]。大于一个段的对象通过分配两个或多个连续的段来支持。未打包的原生整数（fixnums）通过在虚拟内存地址空间的最低和最高部分保留空白，并将相应的段表条目设置为fixnums的类型码来支持。

虽然Chez Scheme甚至还没有构思出来，但C-Scheme对于Chez Scheme是重要的一步。Chez Scheme的初始运行时系统在很大程度上借鉴了C-Scheme的系统，并且C-Scheme被用来引导第一个版本的Chez Scheme编译器


在1982年，恰逢我在丹·弗里德曼（Dan Friedman）的办公室里，在前往北卡罗来纳大学攻读研究生之前。当时，来自研究三角公园的Data General的杰德·哈里斯（Jed Harris）给丹打电话，询问他是否知道有人对前往北卡罗来纳协助他们发起一个Common Lisp项目感兴趣。丹把电话转给了我，我安顿下来后与杰德安排了见面。我以合同员工的身份雇佣进入，并独自担任DG Common Lisp团队的全部工作，大约持续了一年，杰德不时地给我提供支持。计划是将我在C-Scheme中所做的工作转换为一个运行时系统，其中包括Common Lisp核心的解释器，然后在以后的某个时候添加编译器。到年底，我已经在DG的专有系统编程语言中编写了存储管理系统、I/O系统、解释器和各种原语。此时，其他几个人被雇佣进来，Spice Lisp编译器从卡内基梅隆大学引入，与运行时系统相结合，而我则放缓了工作，偶尔提供咨询，以便能够更多地专注于我的博士研究。

在1984年的夏天，我被要求全职在DG工作，修复存储管理系统，因为该系统的替代方案运行速度比我原来的慢两个数量级，并且在第二个收集周期时崩溃。简单地恢复到我以前的代码并不是一个选择，因为现在已经添加了新的对象类型，并且一些其他对象的表示方式也发生了变化。此外，尽管我的旧收集器比替代它的收集器更快，但它仍然相当慢，在DG的旗舰MV/10000计算机上，收集一个8MB的堆大约需要一到两分钟的时间。（不要笑，我们听说DG之外的其他收集器慢得多。）因此，我被要求制作一个更快的收集器。不幸的是，我只有到七月底的时间来完成所有工作，因为DG计划在八月初的Lisp and Functional Programming与AAAI共同举办的会议上展示他们的Common Lisp。另一方面，我得到了一个出色的合作伙伴，罗布·沃伦（Rob Vollum），这让任务变得更加容易和愉快。我们每天工作18个小时，及时为演示准备好了一个稳定的存储管理系统和一个快速的收集器。通过保留（但仍然追踪）堆的静态部分中的所有系统数据结构，我们获得了大部分的性能提升。这是一种比较简陋的分代扫描技术[43, 54]，我当时还没有听说过，但它仍然相当有效。从C-Scheme继承的BiBOP表示法使我们能够避免追踪不包含指针的段，例如包含字符串的段，并且我们也不经常对这些段进行收集，以避免在它们已经分页的情况下将它们带入内存。最终结果是一个收集器平均花费约15秒来收集一个8MB的堆，按照现今的标准来看非常慢，但在当时是可观的。

据我所知，LFP和AAAI的演示非常成功。不幸的是，根据我了解的情况，DG内部的营销部门在成功投标并获得销售Common Lisp产品的权利后，很快就接到了一份有利可图的与其他项目相关的合同，因此Common Lisp产品从未真正面世。然而，我从这个项目中学到了很多东西，并且在我的Chez Scheme工作中能够加以利用。除了获得有价值的存储管理经验外，我还记得对DG对待质量保证的认真态度印象深刻，并成为了广泛开发测试套件的坚定信仰者。此外，尽管我没有直接参与编译器的工作，但我确实编写了处理lambda列表的代码，这是一项不愉快的任务，帮助我在以后的岁月中朝着更为简约的语言设计方式迈进。


在1983年秋季和1984年春季，我利用在DG的相对喘息时间，认真致力于为我的导师Gyula Mago的细胞计算机[44]设计一个Scheme的并行实现。这从我决定去UNC上学的那一天起就是我的目标。由于实际上还没有计算机，我还开始编写一个模拟器（自然是用Scheme编写）。不幸的是，C-Scheme的速度还不够快，尽管它比我尝试过的其他Scheme系统更快。我将模拟器移植到了Franz Lisp [33]，这是一个很好的系统，但我对它对函数参数的处理能力不满意——编译器似乎把这个任务交给了解释器——而且编译器和解释器之间的语义不一致也让我感到沮丧。因此，我决定在研究工作的同时，进行Scheme编译器的设计和构建，最终形成了Chez Scheme。

在设计过程中，我对C-Scheme的实现进行了性能分析，发现大部分时间都花在变量查找和堆栈帧创建上。我逐渐意识到，Scheme的典型实现模型是错误的：通过在堆中分配环境和调用帧，使得闭包的创建速度变快，但牺牲了更常见的变量引用的性能，而且在继续操作方面速度很快，但牺牲了更常见的过程调用的性能。在我们的第二个Z80 Scheme实现中，我们选择放弃了完整的续延，以实现堆栈帧的堆栈分配，T的设计者也做出了类似的选择[47]，但我不想用Chez Scheme走这条路。相反，我开始思考如何让续延“自给自足”，并且同时以某种方式减轻闭包访问对闭包创建的负担。

对于续延问题，解决方案似乎很明显：使用堆栈来进行过程调用，通过将堆栈复制到堆分配的数据结构中来实现续延捕获，通过将堆栈复制回堆栈来实现续延恢复。由于环境仍然是堆分配的，变量的值永远不会直接存储在堆栈上，也不必担心对可变变量进行多次复制的问题。另一方面，我的最终目标是使用传统的堆栈帧，其中局部变量存储在堆栈上，我不确定这样做会有什么结果。

解决闭包问题要棘手一些。在研究其他系统如何应对类似问题时，我偶然发现了Brian Randell和Lawford Russell编写的一本关于Algol 60实现的书[46]，其中描述了使用显示（display）加速访问本地函数的自由变量的方法。显示是一个存储器或寄存器的集合，每个都指向构成当前词法环境的框架之一。显示不能直接用于我的目的，但我能够进行几处调整，并从显示模型推导出显示闭包（display closure）的概念，即一个堆分配的类似向量的对象，保存着代码指针和自由变量的值[16]。

除了允许以常数时间访问所有变量之外，显示闭包还有一个额外的好处，即闭包所持有的环境不会超过其所需的环境，这有助于提高垃圾回收的效率。

对于这种表示法来说，已分配的变量是一个问题，因为一个变量的值可能出现在多个闭包中。我通过对已分配变量进行“装箱”来解决这个问题，即用指向堆分配的单元格或盒子的指针替换每个已分配变量的值，盒子中保存着实际的值。（如果一个变量在其作用域的某个地方出现在赋值语句的左侧，则假设该变量是已分配的。）后来我才了解到，Luca Cardelli在他的ML实现中也使用了类似的扁平闭包表示[11]。在ML中，变量是不可变的，因此编译器不需要引入盒子。然而，可以将引入的盒子视为ML的ref cell形式，不同之处在于，在Scheme中编译器隐式引入盒子，而在ML中，程序员必须显式引入ref cell。

装箱已分配变量也解决了堆栈问题，因为它允许局部变量的值（或装箱后的值）直接存储在堆栈帧中，而不必担心由于续延捕获导致帧被复制的问题。

通过新的闭包和续延模型，创建闭包的成本与自由变量的数量成正比，但访问变量的值的成本变得很小且恒定——如果在其作用域内没有对该变量的赋值，只需进行一次内存引用，否则需要两次。创建或恢复续延的成本与堆栈的大小成正比，但调用帧变为堆栈分配而不是堆分配，节省了链接开销，减少了垃圾回收的频率，并且在进行多个非尾调用时可以重用帧的共享部分。已分配变量的使用成本也变高了，但在Scheme中，已分配变量的使用不太频繁，所以这不是一个大问题。更大的问题是正确处理尾调用可能变得更昂贵。被调用者的参数必须放在与调用者的局部变量相同的位置，以防止堆栈增长，但通常需要调用者的局部变量来生成被调用者的参数。Version 1中使用的简单解决方案是将被调用者的参数放在调用者的局部变量上方，并在将控制权转移到被调用者之前将它们向下移动。

当我对新的闭包和续延模型感到兴奋时，我开发Chez Scheme的主要动力从构建用于研究的工具转变为证明我的新想法可以用于构建快速而可靠的Scheme实现，供他人以及我自己使用。由于我发现其他系统在可靠性和速度方面都有所不足，我重新将精力集中在构建一个不仅快速而且可靠的系统上，它具有完整的类型和边界检查，包括在编译代码中进行堆栈溢出检查。这种新的关注点被证明是一个强有力的动力因素，到1984年夏季开始时，我已经拥有了一个解释器和运行时系统，其中很大部分是从C-Scheme复制过来的，但融合了我从DG Common Lisp存储管理器借用的思想。我还编写了编译器的前端，包括实现新模型所需的赋值和闭包转换步骤。不幸的是，当Data General来电让我为他们的存储管理器工作时，我不得不暂时中断工作，直到秋季才能继续。

当我在秋季回来继续开发Chez Scheme时，我加强了运行时系统，并在北卡罗来纳大学研究生Bruce Smith的帮助下添加了一个大数运算包。这只留下了一个主要任务，即构建编译器的后端。

我最初打算按照Lisp的传统，为交互式使用提供一个解释器，但是Luca Cardelli来访并展示了他用于ML的灵活的交互式增量编译器，我受到启发，决定为Chez Scheme走相同的路线。这使得后端更加困难，因为我不能使用系统汇编器和链接器，即使没有进程创建开销，汇编语言的高级代码生成也无法实现。我为VAX 11/780编写的适用于C-Scheme的代码生成器也不是很有用，因为它是为生成目标代码而不是机器代码而设计的。

在过渡到后端之前，我首先进行了对操作符的参数个数的动态调整。我注意到，参数个数是根据操作符的使用方式确定的，并且在多个不同的地方可能会有所不同。由于Chez Scheme将多个操作符的组合称为复合操作符，我认为通过将操作符与其期望的参数个数相关联，可以减少运行时参数的检查次数，并且可以直接在操作符调用点内联检查。

接下来，我转向了编译器的后端。我需要一个基于堆栈的编译器，能够将Scheme代码转换为一种非常简单的指令集，该指令集使用固定大小的栈帧，以避免堆栈增长和垃圾回收的开销。我设计了一个基于栈帧的指令集，并编写了一个从中间表示到指令集的编译器。编译器将Scheme代码转换为该指令集的中间表示形式，并进行一系列优化，包括尾调用消除和局部变量分配。最后，中间表示被转换为堆栈指令集，并生成可执行代码。

最终，Chez Scheme的编译器和运行时系统的前端、中间表示和后端都得到了完善，并且在1985年初实现了一个可用的版本。随着时间的推移，我继续改进和优化Chez Scheme，使其成为一个高效、可靠的Scheme实现，受到广大用户的欢迎和使用。

在1985年春季，Bruce Smith和我创建了一份参考手册[31]。我将参考手册中的许多示例程序转化为一个测试套件的起点，然后又增加了许多额外的测试。在这个过程中，我找到并修复了一些错误，并在夏季完成了系统的1.1版本。同时，我的妻子Susan Dybvig和我创办了Cadence Research Systems来分发和进一步开发这个系统。Susan的MBA学位和在一家小型软件公司的经验与我的培训和经验相辅相成，我们设法在那个夏季首次商业发货了1.1版本。我们所有的利润从那时起都被再投资用于支付与开发相关的成本，主要是劳动力成本。Susan还与Prentice-Hall签订了一份合同，出版我们的参考手册。Prentice-Hall选择了《Scheme编程语言》作为标题，显然是为了利用《C编程语言》[39]的成功，他们也出版了那本书。我对此感到高兴，但在我看来，这提高了要求，最终我在该书最终出版前重写了大部分内容。不幸的是，我的合著者Bruce Smith没有能够在这个项目上花太多时间，最终我请求并得到了他的允许，独自完成了这个项目。我对他早期对系统和书籍的关键贡献感激不尽。

起初，我编写了所有的代码和文档，并使用一个自制的Z80 PC来管理我们的业务。这个Z80 PC拥有8MHz的Z80H处理器、128K的RAM和20MB的硬盘，在处理这些任务时实际上比新的IBM PC甚至我在UNC使用过的VAX系统要快。 （公正地说，我定制了Gosling的Emacs来模拟我们在Z80上使用的WordStar，这大大减慢了Emacs的速度，可能也是为什么VAX系统看起来比较慢的原因。）构建和测试是在北卡罗来纳州微电子中心（MCNC）的一台VAX计算机上完成的，作为交换条件，我们为Chez Scheme提供了一个免费许可证。我们最终购买了一台Sun工作站，并将开发工作转移到了该平台上，我们购买或获得了其他系统，但我们继续与客户进行交换或借用时间来进行我们没有的平台上的构建和测试。

我们购买的第一台Sun工作站是他们最低配置的系统。当然，如果我们能够证明这个成本是合理的，我会喜欢一台更快、内存更大的系统。另一方面，我认为拥有一台低配置的机器在某种程度上是一种优势：如果我能使Chez Scheme在它上面运行良好，那么Chez Scheme在任何机器上都会运行良好。

在1985年秋季，我们搬到印第安纳州布卢明顿，在那里我加入了印第安纳大学的教职。在整个学年期间，开发工作基本上被搁置，因为我需要准备并教授一个全年的编译器课程和另一门课程。然而，在春季学期期间，已经使用Chez Scheme数个月的桑迪亚国家实验室的George Davidson要求我们创建一个从VAX到基于嵌入式MC68000的系统的交叉编译器。这是一个很好的机会来开发一个MC68000的代码生成器，后来使我们能够将系统移植到Sun、Apollo和其他一些平台上。我决定雇用我的一位编译器课上的研究生在暑期帮助我完成这个项目。

编译器课程中有很多优秀的学生，但其中一位（无论是在字面上还是在实际上）超过了其他人。Bob Hieb身高6英尺6英寸，肩宽背阔，头发浓密，满脸胡须，面容鲜明，典型的装扮是法兰绒衬衫、牛仔裤和靴子。他看起来更像是一个伐木工人而不是计算机科学家（后来我得知他曾做过十年的木工），但他在课堂上的表现显示出巨大的潜力，所以我给他提供了这份工作。他欣然接受了，我们在他不幸去世之前一起工作了七年。

完成了MC68000交叉编译器之后，Bob和我一起在实现的其他方面工作。我们实现了不同的优化级别，实际上只是告诉编译器它可以做一些特定的事情的标志。在优化级别1下，它被允许花更多时间。在优化级别2下，它被允许假设原始函数的全局名称确实与这些原始函数绑定在一起。在优化级别3下，它被允许生成不安全的代码。我们还在编译器中引入了一些代码来优化letrec表达式、优化循环并内联大多数简单的原始函数。

由于编译器是用Scheme编写的，并在自举之后受益于其自身的优化，这些改进使编译器本身更快。实际上，优化措施不仅弥补了编译器为实施这些优化而做的额外工作，还更进一步提升了性能。这是一件好事，因为我们关心编译器的速度（毕竟，编译器被交互地用于加载源文件）。在某个时候，我们实际上制定了以下规则来限制编译开销：如果一项优化不能使编译器本身的速度提升足够以弥补进行该优化的成本，那么就放弃该优化。这排除了我们尝试过的几项优化，包括早期尝试的源代码优化器。

我们还致力于改进收集器的性能。就算法而言，我们已经尽可能地做到了紧凑。然而，我们通过将一个关键例程定义为C预处理器宏而不是C函数，据代码中的注释称，实际上获得了“10至33％的改进”。

除了移植和加快系统的速度外，我们还采用了一些新的语言特性，包括支持低级的扩展传递样式（expansion-passing-style）宏定义[24, 25]和高级的extend-syntax宏定义[41]。由于扩展的二次成本和机制的其他限制，我们直到后来才采用了卫生宏展开[41, 42]。

我们还添加了一个创建“可变参数”过程的新特性，即具有多个接口的过程，这是可选参数的一般化。case-lambda表达式类似于lambda表达式，但它具有多个子句，将形式参数列表与主体配对。当使用case-lambda创建的过程被调用时，将根据接收到的实际参数数量选择适当的子句。我们最初的设计是用不同的语法替换点接口，允许一个或多个case接受无限数量的参数，而不对这些参数的特定表示做出承诺。这实际上删除了接口中的列表，以及列表在优化对不定参数的过程调用时可能引起的各种困难[26, 28]。我们放弃了这个特性，并选择了一个不那么激进的版本，其中每个子句的形式参数列表是一个普通的lambda形式参数列表。

第2版于1987年发布，与Prentice-Hall出版的《Scheme编程语言》几乎同时出版

在版本2和版本3发布之间，我们继续微调编译器和运行时系统以提高性能，但我们大部分的时间都花在了两个新的RISC架构的移植上，改进Chez Scheme与其他语言和进程的互操作性，以及提高系统的整体可用性。

一个同时提高性能和可用性的变化是采用了一种新的续延机制。在版本1和版本2中，捕获续延意味着复制整个堆栈，并且恢复续延意味着将其全部复制回去，这意味着续延操作可能变得非常昂贵。我们通过一种新的分段堆栈方法解决了这个问题，这个方法是我和另一位研究生Carl Bruggeman与Bob一起开发的[36]。这种机制在捕获续延时消除了复制堆栈的需要，并在恢复续延时将复制量减少到一小部分固定数量的字（最多是最大帧的大小），而对正常的过程调用和返回没有增加任何开销。该机制还支持自动堆栈溢出恢复，因此只要堆空间可用于分配，堆栈空间就不会耗尽。

幸运的是，新的续延机制还使我们能够修改跟踪包，以响应Olivier Danvy的请求，以反映非尾调用（通过增加跟踪显示嵌套级别并显示返回值）和尾调用（不执行这两个操作）之间的差异。挑战是识别跟踪过程的哪些调用是非尾调用，哪些是尾调用，起初我们陷入了困境。如果跟踪包内置于解释器中，解释器可以跟踪此信息，但我们没有解释器。相反，跟踪包通过将每个跟踪过程嵌入另一个过程（跟踪包装器）来运行，跟踪包装器负责打印跟踪信息。解决方案是跟踪包维护一个跟踪续延变量，始终保存最后一个跟踪调用的续延。每个跟踪包装器将自己的续延与跟踪续延进行比较。如果它们相同，则发生了尾调用；如果它们不同，则发生了非尾调用。对于仅限于非尾调用的情况，跟踪包装器在应用嵌入的过程之前将跟踪续延设置为新的当前续延，并在之后恢复旧的跟踪续延。使用分段堆栈表示续延，续延的比较可以使用单个eq?测试进行，即指针比较。

我们进行的第一个RISC移植是到Motorola MC88000，这是在Motorola的Sam Daniel的要求下完成的，他多年来一直为Chez Scheme在Motorola推广，并成功说服Motorola在他们的Delta Series MC68000和MC88000系列机器上短暂地推出了Chez Scheme。MC88000是我们承担过的最困难的移植之一，我们花费了整个夏天的大部分时间，由我和Bob（我招募了Carl和Bob的帮助）共同努力完成了这项工作。这部分是因为它的RISC架构与CISC VAX和MC68000架构完全不同，但主要是因为操作系统、C编译器甚至硬件在移植时仍处于积极开发阶段。当然，这意味着要处理几个编译器和操作系统的错误，但最大的挑战是硬件错误。其中一个挑战性较大的问题是，当我们设置断点或单步执行代码时，它会隐藏起来。原来它是由于在从硬件返回地址寄存器移动到我们自己的返回地址寄存器时缺少了一个评分板（忙碌）检查，导致出现了竞争条件。当我们从断点或单步执行代码继续执行时，寄存器不再忙碌，我们的代码就可以正常运行了。然而，当以全速运行时，我们的代码偶尔会尝试过早访问寄存器，而不是被迫等待，就会得到错误的（旧的）值，通常导致一个过程（最终）返回给调用者的调用者，而不是返回给自己的调用者。

另一个移植是到Sparc架构，这个移植进展得更顺利，因为我们已经移植到了一个RISC架构，操作系统基本上与MC68000 Sun系统相同，而且工具和硬件更加稳定。

与同一进程中的其他进程和其他语言的互操作性对于我们的一些用户来说是优先考虑的。这促使我们添加了加载外部对象文件并调用这些文件中的入口点的工具，自动将参数和返回值的数据类型与C和Scheme表示之间进行转换。这些工具是模仿其他Lisp方言实现中类似功能的特性而添加的。我们还增加了支持在子进程中运行其他程序并通过Unix管道与其交互的功能。

我们为版本3编写的大部分新代码以及所有其他新版本的代码都是用Scheme编写的，一些最初用C编写的代码已转换为Scheme，其中可以以更抽象的风格进行编码，不太可能受到系统变化的影响，例如对象表示方式。在开发版本3期间，我们转换的最大一块代码是Chez Scheme的读取器，它实现了Scheme的读取过程，在加载或编译源程序时也使用该读取器。我们预计读取器可能会慢一些，但并不足以证明将读取器保留在C中的合理性。事实上，Scheme版本的读取器实际上更快，尽管Scheme版本在词法分析方面使用了互相尾递归的例程，而C版本的时间关键部分使用了while循环。虽然这种差异可能部分是由于我们可以相对轻松地调整Scheme代码，但知道Chez Scheme的编译器开始真正能够与C编译器竞争（C编译器的任务要容易得多）是令人高兴的。在接下来的发布中，通过在寄存器分配中进行重大改进（包括为过程参数分配寄存器），Scheme版本的读取器变得更快。

版本3发布于1989年。

在发布版本3后不久，我们开始对系统进行重大改造，改变了它表示Scheme值的方式，并重写了编译器和存储管理系统的主要部分。这样做的目的是消除一些对象大小的限制，并且像往常一样改进性能和可靠性。我们还希望为系统留下一个更强大的基础，考虑到我们已经进行或希望将来进行的系统修改。如果一个系统不时地进行全面改造，在新功能被添加到原本不是为其设计的基础上时，它可能会因为自身的重量而崩溃。

在Chez Scheme的前三个重大发布版本中，我们一直坚持使用了从C-Scheme采用的BiBOP机制。由于类型信息存储在一个单独的表中，BiBOP机制允许我们对fixnums和指针使用本地表示，并避免在任何数据结构中占用类型信息的空间。另一方面，进行类型检查的成本相当高。它涉及从对象的地址中提取段地址，将其用作段表的索引，并进行测试和分支。

我们还刚刚感受到另一个问题的影响，这个问题是由典型物理内存和后备存储大小的快速增加引起的。这些增加导致了增加字符串和向量的最大大小的相应压力。这意味着需要增加fixnum范围的大小，因为为了效率，这些对象的索引被表示为fixnums。同时，典型物理内存和后备存储大小的增加也意味着进程正在使用更大比例的虚拟内存地址空间。不幸的是，如第2.3节所述，我们的fixnum范围是通过牺牲部分地址空间来获取的。增加fixnum范围以允许更大的字符串和向量将减少放置它们的空间量。

这个问题迫使我们考虑从BiBOP模型转向标记指针模型。

然而，BiBOP模型具有许多我们不愿失去的优点。通过将包含指针和不包含指针的对象分开，它允许垃圾收集器避免清理不包含指针的对象（如字符串）。更一般地说，它允许收集器对不同类型的对象使用不同的清理方法，从而允许对某些对象使用更巧妙的表示方法，例如包含指向字符串缓冲区中间的“next”指针的I/O端口或包含指向代码对象中间的代码指针的闭包。它允许动态生成或加载的代码放置在堆内的不同段中，以便可以为仅包含数据的页面提供不同的保护。它还允许存储管理器优雅地处理虚拟地址空间中的“空洞”；当操作系统或其他语言运行时为其他目的保留了空间时，可能会出现空洞。

最终，我们决定转换到一种混合模型[23]，该模型使用标记指针来区分特定类型的对象，并且还使用BiBOP获得其各种好处。我们不再使用BiBOP机制来根据特定类型将对象分隔，而是根据收集器感兴趣的特征，如它们是否包含指针以及它们是否可变来进行分隔。对于我们的标记指针实现，我们采用了T [47]所使用的低标签模型，但标签的分配方式不同。这种混合机制允许将fixnum大小增加到30位，并且在许多情况下减少类型检查的开销，而不会牺牲虚拟地址空间或上述BiBOP机制的好处。

改变对象和指针的表示方式以各种可预见的方式影响了编译器。例如，它必须为访问或修改对象、执行算术和类型检查生成不同的代码。这种改变还使我们能够切换到内联分配。由于不再根据所在段的类型来区分不同类型的对象，我们能够将大多数对象（除了代码对象）分配到内存的单个区域中，然后将那些在第一次垃圾收集后幸存下来的对象进行分隔。这意味着我们只需要一个分配指针，可以将其放入寄存器中，并且分配代码序列变得足够小，以便内联某些分配操作，如对偶和闭包。这大大提高了许多程序的性能。

这些变化只是我们在重写编译器时承担的几项任务之一。另一个任务是用编译器特定的记录结构或c-records替换用于表示中间代码的列表结构。由于每个c-record是一个扁平的结构而不是一个链表，这减小了中间代码的大小和访问每个中间形式的子表达式的成本。同时，它还使我们能够减少调度开销。可靠性也得到了提高，因为在创建和调度的地点静态检查了每个记录的形状，而且c-records是不可变的，防止了意外修改。

另一个任务是用程序内寄存器分配器替换我们的本地寄存器分配器。与此同时，我们改变了Chez Scheme的调用约定，允许过程参数在寄存器中传递。在版本4之前，所有参数都在堆栈位置中传递。虽然这在当时在支持直接在内存上执行许多操作并且相对高效的CISC架构上很常见，但在RISC架构上显然不是一个好主意。对于一种强调过程调用而不是循环作为重复的基本机制的语言，我们需要一个处理调用的机制。不幸的是，寄存器分配文献几乎完全集中在为传统的（Fortran）基于循环的程序生成良好的代码上，并且要么没有涉及过程调用，要么只是简单地涉及。我们曾短暂考虑将图着色寄存器分配[12]调整为满足我们的需求，但它对调用没有特殊帮助，而且潜在的编译时间成本超出了基于增量编译的交互式系统的合理范围。

因此，Bob和我开发了自己的寄存器分配算法。该算法首先将寄存器分配给传入参数，然后按照先来先服务的原则，从过程的抽象语法树的叶节点开始，对绑定进行自底向上的遍历，逐个分配寄存器。为了能够利用不再需要的值的寄存器来存储其他值，该算法仅对寄存器值进行了一种廉价的活性分析形式，允许使用廉价的固定位数逻辑操作。算法的初始版本在每个非尾调用周围保存和恢复活跃的寄存器值。一旦算法运行正常，我们收集了一组基准程序的各种动态信息，包括我们自己的编译器。根据这些信息，我们转而采用“惰性保存”策略，即只有在必要时才保存活跃的寄存器值。我们还添加了在每次调用时使用的混洗机制。混洗器在每个调用点对参数重新排序，以减少寄存器保存的数量，并允许参数值直接放置在它们的输出寄存器或堆栈位置中，几乎不需要额外的移动。混洗避免了第3节中描述的尾调用参数的移动。尽管寄存器分配算法主要设计用于提高速度，但其效果出奇的好。该算法的改进版本在几年后发表[7]。

还有一个任务是改进编译器对内联原语操作符的支持，以允许不能完全内联的操作符进行部分内联。我们利用这一点内联了一些安全版本的原语操作符，这些操作符调用了外部的错误处理程序，并在各种通用数值操作符（如+）的fixnum情况下进行了内联。相同的机制还允许我们引入一些平凡但有用的程序转换，例如当一个参数是常量且与eq?相比较时，将eqv?调用替换为更便宜的eq?调用。

最后一个编译器任务是改进对浮点数操作的支持，并添加对复数操作的支持。我们添加了一组新的flonum操作符，如fl+，它们在不安全的代码中进行内联，并在安全代码中进行部分内联。我们还添加了一组类似的“复数flonum”操作符，如cfl+，它们用于组合flonums和不精确的复数。在支持复数操作时，一个主要的挑战是设计一种高效的不精确复数表示方法。我们不希望将其表示为指向flonums的一对指针，因为这会增加额外的间接开销，但我们也希望避免在提取实部或虚部时分配flonum对象的开销。

我们通过一种简单的技巧既避免了间接开销，又减小了flonums的大小一半。由于垃圾收集器在工作时会移动对象，它需要在每个对象中留下转发标记和地址的空间。这对于浮点数来说是个问题，因为我们选择的任何标记都可能与原始的浮点数据无法区分。在支持IEEE浮点数的系统中，我们考虑将转发地址编码为与NaN相对应的位模式，但发现一些体系结构和操作系统没有记录它们的指令和库可能产生的NaN的集合。因此，每个flonum中都包含了额外的空间用于转发标记和地址。这个技巧是去除这个空间，并修改垃圾收集器，使其在flonum中永远不存储转发地址。当然，如果存在两个指向flonum的指针，flonum可能会被复制，但这并不会引起特殊困难。复制可以通过eq?来检测，但这并不违反eq?的语义，当给定两个数值参数时，eq?总是允许返回#f。一旦我们进行了这个改变，我们就能够将不精确复数表示为一对双精度浮点数（又名flonums），并使用简单的指针算术来提取实部和虚部。

Scheme值的表示方式的改变，新的复数类型的添加以及在flonums中消除转发地址的改变，使我们不得不对垃圾收集器进行一些调整，但这些调整相对较小。我们给自己设定的主要挑战是将我们的停止-复制垃圾收集器转换为分代垃圾收集器[43, 54]。与我们的值表示方式改变一样，这种转换是由典型物理内存大小的增加所推动的。随着内存大小的增加，典型堆的大小也增加了，而垃圾收集的成本也随之增加。我们转向分代收集来减少成本。分代收集的基本理论是，经过多次收集仍然存活的旧对象比年轻对象更不可能是垃圾，因此不需要经常进行收集。

我们的变种[23]采用了几个代，代数在系统构建时确定。默认情况下使用五代，其中第0代是最年轻的，第3代是最老的可收集代，第4代是静态代，包含初始堆中的代码，在系统和应用程序“引导”代码加载后。新对象分配到第0代，当它们在较年轻的代中存活了一定数量的收集时，它们将被提升到更高的代。每个代被收集的频率以及一个对象在被提升到更高级别之前必须存活的收集次数可以由程序员设置。默认情况下，第n代在每次垃圾收集器运行的4n次时进行收集，因此第0代每次都被收集，第1代每四次被收集，第2代每16次被收集，第3代每64次被收集。这种相当任意的策略最初只是一个测试的技巧，但事实证明它效果很好，实际上比我们尝试过的几种更复杂的策略效果更好，因此自那时以来一直是默认的。它在实践中很好地降低了收集的平均时间，而没有与将动态分配对象“过早终身”[54]到从不收集的代中相关的潜在问题。

关于我们在整改过程中进行的各种改变的性能优势，我没有记录，但总体运行速度的改善约为50%。更令人印象深刻的是，尽管增加了新的寄存器分配步骤，编译器的速度也提高了30%。

虽然我们Version 4中的大部分改变与整改有关，但我们还添加了几个新功能。其中之一是一个新的检查器，具有编程和交互界面。检查器允许程序员查看和在适当的情况下修改任何对象的内容，包括错误或键盘中断的继续。为了支持源代码信息的列出和在继续或闭包中保存的值的变量名称的正确标记，我们不得不对编译器进行额外的更改，以便它能够通过各个步骤跟踪源代码信息。这事实上是一个相当直接的过程，只需向每个c-record添加一个源字段，将信息传播到每个步骤，并最终将信息与生成的代码对象中的入口和返回点的地址关联起来。支持检查的功能在不牺牲性能的情况下实现，因此我们能够在解决系统中一个明显的缺陷时保持忠于我们的优先事项。

我们原本计划在1992年夏天开始Version 5的工作，但是Bob Hieb在一场车祸中不幸去世，他的女儿Iva也同样遇难。这场悲剧本可以导致Chez Scheme的开发中断，但我在处理自己的悲痛时，部分通过疯狂地赶工来完成我们已经开始或计划进行的Chez Scheme开发工作，以及IU的一些未完成的研究工作。

Carl Bruggeman在完成后者方面提供了特别的帮助，因为他加入与我一起完成了语法案例（syntax-case）宏系统的工作，而这个系统是Bob的博士研究的核心。我们在同一年完成了该系统的工作，并发表了一对描述该系统的技术报告[17, 37]，一年后发表了结合了这两个技术报告部分内容的期刊文章[29]。我将语法案例系统整合到了Chez Scheme中，并发表了一个可移植版本，此后一直与Chez Scheme版本保持同步。

我在1992年进行的大部分其他系统改变都是在代码生成、编译时检查等方面的例行改进，但一个重要的任务是对局部调用进行优化。在此之前，大多数局部调用涉及到一个procedure?的检查和通过过程闭包进行间接跳转。唯一的例外是直接调用被识别为let表达式的等价形式，以及编译器识别为循环参与的调用。实际上，全局调用更便宜，因为在第3节中描述的代码指针技巧消除了procedure?的检查。

为了优化局部调用，我修改了编译器，记录了未分配的变量何时直接绑定到lambda或case-lambda表达式，然后为通过这些变量进行的调用生成更有效的代码。特别地，编译器消除了procedure?的检查，用跳转到lambda主体（通过参数计数检查或直接进入适当的case-lambda主体）替换了闭包间接跳转，仅在过程有自由变量时传递闭包指针，并且对于"rest"参数，在已知参数数量且不需要循环的调用点分配列表。在许多情况下，过程只以这种方式被调用。如果它没有自由变量，闭包将永远不会被使用，因此编译器消除了闭包创建代码和它的let-或letrec-绑定。编译器通过识别在一组互相递归的过程中仅需要闭包来保存其他过程的闭包时，安排更多过程符合这种情况。

局部调用的优化产生了很大的差异-在我们将全局调用转换为局部调用的基准版本中，性能提高了15-50%。再次说明，引导编译器从自身的优化中受益，因为编译器的平均速度提高了约25%。

回顾起来，这些优化似乎并不是很困难，我的研究生编译器课程的学生能够在几个每周作业的过程中实现它们。然而，在当时，我发现它们极具挑战性。当然，我没有一份指导我如何进行的任务描述，并且我的中间语言更加复杂。困难的另一个原因是，我把优化塞入了现有的编译器步骤中，而我当时可能应该添加一个或多个单独的步骤。额外的步骤并不像当时我认为的那样昂贵，而且简化了编译器，并且在以后的年份中修改编译器的复杂性和相对容易程度将更多地弥补了编译时间的较小成本。

1993年，我招募了我的一位研究生Mike Ashley与我一起致力于支持多返回值。我们希望拥有一种机制，就像我们的延续机制一样，在正常的单值返回和返回点的效率方面是不受影响的。我们还希望该机制在接收到错误数量的值时发出错误信号。我们通过一种能够高效处理多个值和单个值的返回和返回点的机制[2]实现了这两个目标。

我还招募了另一位研究生Oscar Waddell，将Chez Scheme移植到运行Digital的OSF/1操作系统的Alpha处理器上。这是Oscar和我一起合作的许多项目中的第一个项目。除了Alpha移植本身外，该项目最有用的成果是消除了系统源代码中的汇编代码。我们之前一直在使用一组机器相关的m4[40]宏和大部分机器无关的汇编文件来减少与移植到新架构有关的汇编代码数量，但这种方法难以处理，并且不能完全摆脱与汇编器语法的许多特殊性的打交道。我们用一种类似汇编的、机器无关的语言取代了汇编代码，通过将代码输入到我们编译器的代码生成器中实现了这一点。

还有一个值得提及的Version 5改变，适用于喜欢简单但有效的技巧的人。这个特殊的技巧将生成符号或gensym的成本降低了25倍。这个技巧是将gensym的名称生成推迟到第一次打印时，这样(gensym)就变成了一个廉价的内联分配操作。大多数创建的gensym，例如在宏展开过程中，从未被打印，因此节省是真实的，并且对于大量使用gensym的程序具有重要的影响。

版本5和版本6之间，系统开发持续快速进行。我们增加了对源文件信息的跟踪支持，通过读取器、展开器和编译过程，使检查器能够显示原始源代码而不是展开器的输出。我们还添加了对不相交记录类型、一次性延续[4]、模块[56]和我最喜欢的新语言特性之一-数据注释的支持。使用语法#;，数据注释可以注释掉整个S表达式，例如(a #;b c)将被解析为(a c)。我之所以想到这样的需求，是因为看到人们使用引号来注释顶层表达式，然后在不在顶层时遇到问题时感到沮丧。

我们还添加了从C调用Scheme的支持，从而为嵌套的Scheme/C调用打开了可能性。我们通过在调用Scheme之前保存C堆栈上下文（使用setjmp），避免了延续的困难。在从Scheme第一次返回时，恢复这个上下文（使用longjmp），此时上下文和任何动态从属上下文都被标记为无效。尝试返回到无效的上下文将导致错误信号。这允许在Scheme方面执行任意延续操作，只要不尝试第二次返回到C帧或动态嵌套在C堆栈中的帧。

我的另一位研究生Bob Burger被聘请来将Chez Scheme移植到两种新的架构，即HP/UX下的HP PA-RISC架构和AIX下的PowerPC架构。我还整合了Bob对我在1990年编写的一个浮点数打印机的几个改进，该打印机基于Guy Steele和Jon White的一篇论文[53]。Bob的改进使算法更简单、更高效[5]。Bob的博士研究是关于基于剖析的动态重编译[6]，尽管我们从未采用过动态重编译支持，但我们确实借助Mike Ashley的帮助将Bob的剖析支持整合到了Chez Scheme中。

版本6的一个目标是提供一个完整的运行时系统，以简化编译的Chez Scheme程序的交付。Scheme的运行时系统如果没有eval就不完整，因此我们决定包含一个我们在版本2中悄悄引入的用于交叉编译的解释器。借助解释器和其余的运行时系统，我们已经实现了一个工作中的Scheme系统的99%，因此我们决定将读取-求值-打印循环（REPL）包含在其中，并将运行时系统与解释器作为一个完整的、独立的系统发布。该系统使用与Chez Scheme相同的源代码构建，只是没有编译器。我们将该系统命名为Petite Chez Scheme，因为没有编译器，它比整个系统更小。在发布系统之前，我们在解释器中进行了一些调整以提高其速度。人们经常对解释器的速度给予很高的评价，而且他们经常会惊讶地发现它是用Scheme编写的。出于某种原因，他们认为用C编写的解释器应该更快，尽管使用良好的Scheme编译器编译的Scheme解释器可以利用内置的对尾调用和延续的适当处理的支持，以及编译的库代码。

当然，一如既往，我们还致力于改进编译代码的速度。我们早些年编写了一个源代码优化器，但因为未能通过性能测试（即没有使引导编译器的代码足够快），我们放弃了它。然而，在1996年初，Suresh Jagannathan和Andrew Wright的一篇论文《流导向内联》[38]的一个预发布版本激发了我们的行动力。这篇论文很有意思，因为它声称对一组大型Scheme程序实现了一些令人印象深刻的速度改进。尤其引人关注的是，这些改进是通过在Chez Scheme编译器之前运行的优化器实现的。其中一些改进结果是由于将全局调用转换为局部调用，从而触发了第8节中描述的局部调用优化，但即使在调整了这一点之后，结果仍然非常出色。尽管他们的分析在我们的编译器中的编译时间成本不切实际，但我们现在知道好的性能改进是可行的，只要我们能够找出如何做到。

Mike Ashley已经在开发自己的流分析器，并将其改编为实现类似于Jagannathan和Wright的算法。实际上，他能够在结果上略微改进，但尽管他的流分析器快得多，但对于我们的目的来说仍然不实用。

因此，Oscar Waddell和我开始创建我们自己的内联器，约束条件是它应该快速和线性。我们希望它也能产生与流导向内联器取得的类似结果。经过数月的努力，我们有了一个内联器，性能甚至超出了我们的期望，在所有基准测试中生成至少相当的代码，并在一些情况下明显击败了流导向内联器[55]。一个原因是我们算法的“在线”特性，允许内联器基于已经转换的子表达式做出决策，而流导向内联器在任何转换发生之前完全运行分析。另一个原因是我们在决定何时内联和何时不内联时使用的方法。内联器只是尝试所有内联，当剩余代码的大小或内联尝试所花费的时间超过预定限制时，终止内联。在采用这种直截了当的方法之前，我们尝试了几种启发式方法来限制代码大小和内联时间，但是启发式方法不可避免地会限制或允许更多的内联。我们将内联器整合到Chez Scheme编译器中，并在Petite Chez Scheme中将其用作解释器预处理。

在此期间，Oscar Waddell被选为负责开发GUI API的负责人，以支持Scheme在一个由NSF支持的教育基础设施项目中的使用。最初的名字是“Bob”。我想这是借鉴了同名的微软产品，但我不确定。无论如何，这个名字没有流行起来，而改为了Scheme Widget Library。简写为SWL，并发音为“swill”，这个名字仍然反映了Oscar对系统在早期的负面意见。然而，最终这个系统非常出色，包括一个用于Scheme的交互式开发环境以及构建图形应用程序的工具。Oscar贡献了大部分工作，其他几个人也做出了贡献，尤其是Carl Bruggeman、Bob Burger、Erik Hilsdale和John Zuckerman。Abstrax公司的John LaLonde也支持了这个项目，我们从他在Motorola工作时写的一个系统中借鉴了一些想法。该系统现在与Chez Scheme一起发布，我也做出了一些小的贡献。许多程序员仍然使用emacs和Chez Scheme，像我这样的高级用户使用vi和Chez Scheme，但是其他很多人在Windows和Mac下使用SWL作为与Chez Scheme或Petite Chez Scheme的接口。

尽管版本6和版本7之间过了许多年，但我们并没有闲着，发布了几个较小的更新版本。我们增加了各种新功能，如大数的逻辑操作、文件压缩、支持Scheme脚本（包括脚本编译）、Windows注册表原语、eq?哈希表和apropos。我们扩展了gensyms，允许创建全局唯一标识符，添加了记录继承功能，并支持根据应用程序可执行文件名称自动加载堆和引导文件。在一阵疯狂之下，我还实现了Common Lisp的格式化功能，这在潜在上非常有用，但即使在实现后，我也无法记住每个指令是什么，尤其是使用冒号和at-sign修饰符的组合来完成特定任务。我本打算实现一个不那么复杂的东西，但不知道何时停下来。

除了这些相对较小的变化外，Oscar和我首次将系统移植到了64位体系结构，并创建了一个基于Posix线程的多线程版本的系统，允许应用程序利用多个处理器和处理器核心。针对64位Sparc体系结构的64位移植本应相对简单，但我们遇到了许多32位依赖问题。你可能认为一个经历了从16位微处理器地址空间过渡到32位地址空间的人会避免这样的依赖性，但当然，我从未想过Chez Scheme会存在足够长的时间以至于这成为一个问题。至少现在我们已经为过渡到128位地址空间做好了准备。

线程系统带来了许多挑战，特别是在使系统的部分部分线程安全方面。幸运的是，我们早期的设计决策简化了项目的其他方面。我们决定坚持使用BiBOP和分段内存，使得为每个线程分配独立的分配区域变得简单。独立的分配区域对于避免在每个分配操作上进行同步是必不可少的，否则性能会受到严重影响。我们的分段堆栈表示方法也有所帮助，因为堆栈溢出恢复机制允许每个线程以较小的堆栈开始，根据需要添加新的段来扩展。我们对启动垃圾回收的模型也有帮助。回收是通过中断触发的，中断是由基于分段分配器自上次回收以来分配的字节数设置的。这种方法很好地推广到允许多个线程同步进行回收。当一个线程收到回收中断而其他线程仍然活动时，它会等待其他线程。当最后一个活动线程同步、阻塞或退出时，回收被启动。

然而，跟踪对旧代的赋值却成为一个问题。分代收集器必须在旧代对象内的位置包含指向新代的指针时得到通知，以便在对新代进行收集时可以追踪它。我们的收集器使用了一种修改过的标记卡系统，其中这些信息存储在全局表中。不幸的是，存储信息不是原子操作，而且在每次变异操作上进行同步开销太大，即使一个人认为程序应尽可能避免变异。为了解决这个问题，我们在本地分配区域的末尾维护了一个变异位置的日志，其中日志指针向分配指针增长，就像第二个版本的Z80 Scheme系统对堆栈和分配指针所做的一样。同步操作无需发生，直到日志和分配指针相遇时，此时扫描记录的条目以记录有关从旧代到新代的指针的信息，并在必要时获取新的分配区域。



开发类似Chez Scheme的系统是一个逐步完善的过程。即使在1983年时我已经知道今天的知识，要一次性创建像现在这样的Chez Scheme系统仍然是难以想象的困难。如果我尝试过类似的事情，我很可能在开始之前就放弃了。相反，我从一个简单的模型开始，用于表示Scheme值，一个执行少量优化的小型编译器，一个简单的停止-复制垃圾收集器和一个小型运行时系统。从那时起，随着功能的添加、扩展、删除和替换，它不断增长和减小。

当然，为了使这个长达数十年的开发工作成功，我们不得不做一些常常令人不愉快的事情，比如放弃我们认为聪明的代码，重写我们希望再也不用看到的代码，即使不方便也及时修复错误，以及在添加新功能时扩展测试套件。如果我们没有这样做，系统将充满无用、混乱、错误和未经测试的代码，使得难以进行维护和扩展，甚至使得编写新代码等有趣的工作变得令人不愉快。

随着时间的推移，我们的优先事项也变得更加精细，特别是在效率方面。我一直认为效率不仅仅是编译代码的速度，还包括编译时间、内存使用和整体系统响应性。多年来，我逐渐认识到性能的统一性和连续性也很重要。我对处理大型程序或处理大量数据的程序也更加关注。除了效率和可靠性之外，还出现了其他优先事项，例如符合标准、能够与其他语言编写的代码优雅地交互，以及整体系统的可用性。

即使经过二十年的改进，该系统还远未达到我希望的状态。有许多可以提高性能的方法，许多代码可以更加简洁，还有许多我想要添加或扩展的功能。我们的待办事项列表有数百个条目，涵盖了从简单的琐事到研究项目的范围。我并不抱怨。如果待办事项列表曾经变为空，我会知道是时候结束了。
