# **Datahub元数据管理平台使用手册**

> 资料来源：https://www.yuque.com/fivedata/pehkup/iugem71z8dpbbi3g

## **前言：元数据是数据治理的灵魂**

### 1. 元数据之于数据治理

数据治理是一个庞大的系统，其中主要包括数据管控，数据质量，数据安全，数据标准。



a) 数据管控：

每一项数据变更都能得到明确记录及授权，使得数据系统变得可控，可追朔。

b) 数据质量：

主动检测发现数据异常，禁止系统运行在错误状态造成损失。

c) 数据安全

随着大数据时代的信息爆炸，数据的安全性开始与个人生活息息相关，任意安全事故对于企业商业运行来说可能都是致命一击。

d) 数据标准

所有的数据踏上规范化的进程，让所有数据系统建设有着清晰统一的目标。



我们这里主要聚集在数据管控这一块，也就是元数据的应用领域。



### 2. 什么是元数据？

一般来说是数据的数据。

具体来说，就是对动态数据的一种静态信息描述。

数据从本质上来说是二进制，动态的。

所以，为了理解数据，我们需要数据的数据。

### 3. 常用的元数据有哪些？

在数据处理中，我们首先需要定义不同阶段的数据实体，所以有了【模式元数据】。

接着我们需要定义数据实体之间的处理逻辑，叫做ETL数据处理，接着有了数据实体的【关系元数据】。

对于这些数据处理的逻辑形式，需要调度器来物理化执行，所以有了【调度元数据】。

数据处理完毕，需要发布报表，就有了【报表元数据】。

对与整个系统，会涉及不同的用户实体，就有了【用户元数据】。

当然，这些是企业数据平台最常见的元数据类型，其他的大大小小的信息还是有很多。

所以，元数据系统的建立，是企业级的信息化建设过程。



### 4. 企业数字化建设始于元数据

在互联网热潮下，企业往数据化转型已经是大势所趋，数据治理建设已经被归为企业的重要战略。而信息化建设的重要前提就是元数据。

企业的系统建设过程中，元数据缺失或者没有有效管理，致使无法系统化信息化，只能供人工使用，导致系统信息需要大量人工介入才能有效解密。



### 5. 如何构建有效的元数据系统

1、配置驱动开发

对于常用的系统的系统来说，一个好的数据通过配置文件就能全面了解。

对于ETL来说，配置信息就是实体【表结构的定义】以及关系【SQL及存储过程】。

对于调度系统来说，配置信息就是实体【作业】以及关系【依赖关系】。

对于报表系统来说，配置信息就是实体【指标】以及关系【业务使用】

大部分成熟的系统所以已经提供了非常完善的配置信息接口，所以我们做的就是把信息统一接入并使用起来。

还有一部分系统不成熟，非常依赖于文档与人工。所以要进行改造，变成配置驱动开发。通过配置捕获系统的本质。如果不能进行改造，我们也得将信息进行编码，使得静态信息尽可能在元数据之上的数据管控上进行规范有效操作。

2、如何制定配置？

系统间的配置一般采用json/avro或者使用sql接口来进行集成。

对于人工的静态配置信息来说，json处理不了多层级复杂关系，建议使用toml/yaml来处理，更加可读，也非常方便集成元数据管理。



这里，我们提到了两种配置类型。一种基本配置类型，一种高级配置类型。

基本配置类型就是数据结构，我们仅仅定义数据结构，比如：json/varo/toml/yaml。

而高级配置类型就是DSL,我们开发出一种更抽象强大的扩展，比如SQL/lisp。

从系统的扩展性来说，DSL表达力更强。所以我们可以看到，gradle对比maven做扩展就非常有优势。Emacs对比idea就非常有优势。

但是另一方面建设成本较高，但是价值非常大。所以对于数据系统来说，采用SQL高级配置能大大改善元数据的质量。这也是各个系统不遗余力推出SQL高级配置，LISP高级配置的原因。

3、元数据的开放性

前面讲过，即使企业存在大量元数据，以不同文档形式存在于企业当中。但都是需要人工介入，且无法对接系统进行使用的。

所以元数据的核心价值就在于元数据的开放性。

有了元数据系统，我们可以轻松得搜索信息，展现信息，发布信息。这样a系统的元数据可以供b系统自动化使用，其它系统又可以使用b系统的元数据，最终不断递归下去，形成了一个企业的信息化中心。而这个中心有系统的本质信息，所以整个企业的信息化数据是非常容易理解且使用的。

### 6. 认识元数据平台

1、元数据平台简介

现在才开始我们的正题。

市面上常见的元数据管理系统有如下几个：

a) linkedin datahub:

https://github.com/linkedin/datahub



b) apache atlas:

https://github.com/apache/atlas



c) lyft amundsen

https://github.com/lyft/amundsen

**当然最成熟的还是linkedin datahub**，也是今天的主角。



2、元数据组件

前面讲过元数据平台的核心在于开放性。



而元数据的主体主要是【实体】以及【关系】。

在datahub里面通过图数据库**neo4j**来表达这种关系。

主要定义的实体有【用户、数据集、报表、作业、指标】，开源支持的实体有【用户，数据集】。



3、为什么需要使用图数据库？



对于同种实体之种的关系，sql数据库非常难以表达。比如用户a跟用户b的关系，就需要自join一次得到信息，而如果这种关系有多级，比如一个数据级的上三层血缘关系，那么关系型就失效了。

对于不同实体间的多种关系，使用sql数据库也是非常麻烦的。即使能做到，也是非常难于理解。而图数据我们仅仅只需要关心实体与关系，通过引擎自动帮我们遍历获取结果。



4、如何实现数据的搜索？

有了实体关系之后，我们需要进行数据的搜索。在大部分人的认识里面，数据的搜索查询采用的是索引。而索引是不支持多个范围查询的。对于搜索这种类型，有专用的倒排表结果，也就是**lucene**库。以此之上就有了最成熟的**elasticsearch**文档数据库。

何为倒排表？一般的数据库是，通过文档id去查数据信息。而倒排表是反向的，通过数据信息去搜索文档。它的原理是对于需要搜索的字段建立倒排表，通过各个字段的组合最终获取文档id。



5、如何实现数据的存储？

对于最后的数据存储，就比较简单了。

采用postgresql以及mysql这种广泛的数据库都是分分钟支持的事。在datahub里面，对于每个实体信息，它分为了各种aspect切片。比如数据集信息就包括了模式切片，所有者切片等等。



### 7. 元数据架构

在datahub里面，数据通过mce事件进入，进行处理后，生成mae事件。所以它是一个实时系统。

当然大部分人对于元数据的时效性要求不高，觉得实时提升了系统的复杂度。并且实时需要上游系统去主动推才行，实用价值不大。



其实这里是有个错误的理解。

元数据平台不是数据管控平台，它不是一个应用平台，而是一个基础平台，让其它系统扩展并集成使用。

所以，数据管控平台可以很方便的通过订阅mae消息，进行实时变更，并且完全绿色无污染。

我见过不少厂商做元数据系统，想做得非常规范，需要非常强的侵入性，甚至只能使用它们的平台才能操作。

我是非常不认可的。

元数据系统应该是个信息化系统，更大的作用大于信息化上面。不能说有了元数据系统，其它方式就不能用了，这样不是引入了一颗炸弹么？相当于学了一门武功，费了早前的武功，实在是得不偿失。

当然，可以在元数据之上建立数据管控平台，进行数据的管理，但这个也是个助手，而不是将军。

### 8. 认识datahub

![image-20230703101421745](.\images\image-20230703101421745.png)



datahub的技术栈非常特别，具有一定挑战性。

首先datahub包括了四块，**metadata,gms,etl,datahub.**



medata定义模型，gms基于模型生成服务, etl进行模型数据加工，datahub提供基于gms的元数据应用展现。

metadata这里面使用了两种数据格式：

一种是外部接入格式avro，非常实用。

另一种是内部改进的pdsc格式，那就很头疼了，外面用得很少。

gms使用了内部的rest.li，又是内部搞的一套restful框架，也还比较好用，但是应用面比较窄。

etl则是采用了linkedin家最擅长的kafka schema registry及kafka streams，好评。

datahub包括了应用后台服务以及前台展示。

后台服务采用的是play framework，这个就是比较衰的了。以前linkedin 大力推scala，搞了play framework，结果发展不动，全部切成了java版的。所以java版的web框架也用了play framework。

这东西也不错，就是也不主流了。

对于前台服务采用的是ember.js + typescript。typescript如今如日中天，非常好用。

ember.js 就比较衰了，被angular,react,vue逼上了梁上。

并且这货学习成本也不低。不过linkedin 的ui确实有考究。

对于整个系统构建采用了gradle，个人还是比较喜欢gradle的。

前面已经提到过高级配置dsl扩展性是远远超过配置文件xml的。

所以我更愿意深入学习gradle而不是maven。另一方面高级配置对于集成性会偏弱，需要预留接口给第三方使用。简单配置人人都可以解析。gradle 采用groovy实现，groovy也衰落了，现在gradle切换到kotlin了。照目前趋势，gradle超过maven是必然的趋势。



所以，对于整个技术栈来说:

gradle + es + neo4j + pg + kafka + typescript十分到味。



### 9. 元数据的使命

对于企业信息化建设来说，我们是不需要关心系统是如何实现的，我们往往想定义它的本质。

这个比较类似于现在流行的配置驱动系统建设，我定义好配置项，然后系统就能按照预期运行。

为了加深理解，我简单举几个例子。

比如SQL就是一种重要的元数据，我们不需要关心底层怎么实现，我们只需要告诉它我们想要的，它就会获取我想要的结果。所以sql是一种有效的元数据，有了sql，我就捕获到了本质。



如果把所有系统的本质捕获到了元数据中，那么通过元数据就能了解并且加快信息化建设。

