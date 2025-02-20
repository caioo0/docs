本文直译自《hands on ML》课后题。（有改动的以【】表示）。

## 1.自动编码器的主要任务是什么？

（a）特征提取（feature extraction）。
（b）无监督预训练（unsupervised pretraining）。
（c）降维
（d）生成模型（Generative models）
（e）异常检测（Anomaly detection，an autoencoder is generally bad at reconstructing outliers）

## 2.假设有大量无标签训练数据，只有几千有标签样本，如果要训练一个分类器，自编码器能帮什么忙？该怎么操作？

首先可以在整个数据集上（有标签+无标签）训练一个深度自编码器，然后复用它的下半部分（即从输入层到coding layer（含））当做分类器，用有标签的数据来训练分类器。如果有标签数据很少，训练分类器的时候可以冻结（freeze）复用层。

## 3.如果自编码器完美重建了输入，那么它一定是一个很好的自编码器么？该怎么评价自编码器的性能？

（1）自编码器完美重建了输入并不能说明它是一个好的编码器；可能只是一个过完备（overcomplete）的自编码器——把输入拷贝到编码层然后拷贝到输出层。事实上，即使编码层只有一个神经元，深度自编码器仍有可能把每个训练样本映射成不同的编码（例如第一个样本映射成0.001，第二个映射成0.002，第三个映射成0.003，以此类推），它可能‘记住了’（learn “by heart”）并对每个编码重建训练样本。它可以在没有学到数据的任何pattern的情况下完美重建输入。实际中，这种映射不太可能发生，但是它说明完美重建输入并不能保证自编码器学到了有用的东西。当然，如果它重建效果很差，基本保证了它是个很差的自编码器。
（2）为了评价自编码器的性能，一个方法是衡量重建误差（reconstruction loss，例如计算MSE，输出减去输入的平方的均值）。大的重建误差表明自编码器很差，但是一个小的重建误差不能保证自编码器很好。也可以根据自编码器的用途来衡量它，例如如果把它用来作为一个分类器的无标签预训练，可以通过衡量分类器的效果来衡量自编码器。
4.什么是欠完备（undercomplete）和过完备（overcomplete）自编码器？一个过度欠完备的自编码器有什么风险？一个过完备的自编码器的风险是什么？
如果编码层比输入和输出层小，则是欠完备自编码器；如果编码层比输入和输出大，则是过完备自编码器。过度欠完备的自编码器可能会无法重建输入。过完备的自编码器可能会把输入拷贝到输出，没有学到任何有用的特征。

## 6.堆栈式自编码器（stacked autoencoder）低层特征可视化的常用技术是什么？高层特征可视化呢？

低层特征可视化的常用技术是把权重向量形状改变成输入图片的形状，然后画出每个神经元的weight（例如，对MNIST数据集，把[784]维的weight向量转化成[28,28]的图片）。
要可视化高层的特征，一种技术是展示最能激活每个神经元的训练样本。

## 7. 什么是生成模型（generative model）？能否举出一个生成模型的例子？

一个生成模型是可以随机生成类似于训练集的输出的模型。例如，在MNIST数据集上训练完成之后，一个生成模型可以用来随机生成数字的图片。输出的分布一般和训练数据相近。例如，由于MNIST数据集包含同一个数字的很多图片，生成模型也会生成差不多相同数目的该数字。有些生成模型可以被参数化（can be parametrized），例如只生成某种类型的输出。生成模型的一个例子是变分自编码器。