# GAIL-Generative_Analog_Tools_IC_Layout


1985年，英特尔推出了80386微处理器，它装载了275,000个晶体管，在当时令人惊叹。而到了2023年，AMD首款数据中心/HPC级的APU Instinct MI300，共有1460亿个晶体管。

如今，具有数以亿计器件的数字模块，通常通过先进的EDA工具在数小时内就能完成自动布局和布线。这对数字设计生产效率的影响确实是巨大的。但是对于模拟电路来说，其版图设计就一直是项手动任务，多年来，许多在模拟设计流程方面渐进式的改变，包括[参数化单元](https://patents.google.com/patent/US10885256B1/en)、原理图驱动版图，以及近期的模板化版图，但是目前模拟IC版图的主流设计方法仍需要大量的手工设计。

### Analog IC Layout Introduction
模拟电路的设计流程主要由原理图设计和版图设计两部分构成，原理图设计通常由模拟电路工程师根据场景需求和性能指标选择电路结构，然后根据前端电路仿真结果调整元件参数；版图设计则需要工程师在版图绘制平台确定元件摆放位置并完成连线，再考虑寄生参数的后端电路仿真验证。如果后仿结果无法满足要求，则需要返工调整版图设计甚至原理图设计。即使是小规模的模拟电路模块，版图设计通常也需要耗时几周甚至几个月，设计周期冗长。与此同时，工艺改变使得模拟设计的完成变得更加困难。如今，版图依赖效应已经变得至关重要，以至于无法预先确定最佳版图。工程师和设计师们经常需要对他们的设计进行多次的调整和重新设计，才能达到规格要求。每次迭代都需要花费数小时或数天的时间，模拟版图设计俨然已成为“循环版图设计”。

那么，为什么模拟版图设计很难实现自动化呢？在数字案例中，标准单元方法可有效地将版图设计问题抽象为一个核心问题，即只需考虑三个需要优化的主要因素：时序、功耗和面积。创建标准单元库是一项复杂的自定义任务，但是一旦有了一个具有多种输出驱动强度的双输入与非门，就无需一遍又一遍地创建新的晶体管级版图。并且VLSI工具不必考虑绝大多数复杂的DRC规则。这将物理和电路问题抽象成为了计算机和算法问题，极大的方便了学术研究和理论开发。而模拟设计要考虑的因素非常多，即使是一个简单的运算放大器，也可能需要对其输入偏置电流、失调电压、CMRR、PSRR、增益裕度、相位裕度、噪声、失真、电压摆幅等参数进行优化。


### Prior Works of Automatic Analog IC Layout 

多年来，人们在将模拟版图设计自动化方面进行了诸多尝试。其中大多数都着眼于修改数字标准单元的方法，但都没有成功。这些算法对其所生成的解决方案会对电路性能产生什么样的影响并不了解。一些尝试采用了来自数字领域的布图规划技术。布图规划算法旨在将不规则形状有效封装到最小空间中。但是，模拟版图设计主要不是封装问题。实际上，尽管紧凑而节省空间的封装很重要，但它在模拟版图设计优先级列表中排名很靠后。另一个问题是布线。在数字世界中，布局和布线是先后两步操作。首先是利用布局算法对标准单元进行布局。布局器能“顾及布线”，并能考虑布线拥堵以及来自导线的寄生电阻和寄生电容的影响，但它实际上并未对导线和标准单元进行同时布局。这种布线和布局相分离的方法不适用于模拟版图设计,它们未真正顾及到模拟，尤其是复杂的Design Rule，而是过于算法化。

模拟版图自动化的探索已有几十年的历史。20世纪八九十年代的一系列工作，早期工具包括元件生成（parametric-cell generation）、元件布局（device placement）、布线（routing）。这些工具主要采用模拟退火算法求解元件布局。后来的一些研究，如LAYGEN II，[ASTRI](https://patents.google.com/patent/US10885256B1/en)，强调专家经验的重要性，支持用户自定义版图层次化设计模板，并结合B*-tree表示布局拓扑，按照模板生成元件布局，再利用进化算法优化布线。该思路本质上要求用户提前规划元件布局方案，并按格式写入配置文件，自动化工具负责版图生成和布线。近年来的工作主要集中在目标和约束的能力上，如层次化设计、各类对称约束（对称、匹配、中心对称）、版图依赖效应（layout dependent effects）优化、屏蔽线（shielding）等。
一些
MAGICAL[2]
ALIGN[3]
BAG[4]
2018年美国国防部高等研究计划署（DARPA）发布了电子复兴计划（Electronics Resurgence Initiative，ERI），其中的电子设备智能设计（Intelligent Design of Electronic Assets，IDEA）项目旨在探索“24小时无人工干预”的数字和模拟电路版图生成方法。以此为契机，学术界出现了多个开源的模拟版图自动化工具，例如德克萨斯大学奥斯汀分校的MAGICAL[2]、明尼苏达大学的ALIGN[3]和加利福尼亚大学伯克利分校的BAG[4]。

### Challenges of Analog IC Layout






那么，模拟版图设计自动化工具需要具备哪些属性，才能完成这项看似不可能的任务呢？
● 模拟电路识别：该工具必须要能了解和理解许多模拟电路结构，并能够在它们出现在所工作的电路中时识别它们。
● 自动约束：了解电路之后，该工具需要为其自身生成适当的约束条件，而无需工程师花费数小时来编辑它们。
● 模拟版图设计策略：该工具必须能在其算法中内置许多经过实践检验的版图设计策略，并将版图设计与电路理解和约束条件联系起来。
● 遵守深入的设计规则：算法核心的深处要能真正理解设计规则。该工具不仅要有检查错误的能力，而且还必须能在第一时间避免错误的形成。
● 灵活性：模拟设计工作是非常困难的，硬性规定的版图设计风格一旦被采用，即意味着失败。严格遵守约束条件会生成低质量的版图。一个成功的模拟自动化工具需要灵活的才智，以便在需要的地方达到良好的效果。
● 易于使用：模拟设计流程已经很复杂了，流程中的任何新工具都必须让工程师的任务更容易，而不是更难。





模拟版图自动化的挑战

相较于数字电路拥有从逻辑综合到布局布线全套设计验证流程，模拟电路目前仍然缺乏设计自动化工具。特别是耗时较长的版图设计部分，由于设计方式灵活，对设计经验要求高，设计结果对电路性能影响大，因此自动化难度较大。模拟版图自动化的主要挑战可以归纳为以下几点。

设计方式灵活，求解空间大：数字电路版图是对统一风格的标准单元进行操作，互连线宽度较为统一；而模拟电路版图设计直接对晶体管、电阻、电容等元件进行摆放，元件形状各异，允许任何符合设计规则的互连形式。因此，模拟电路版图设计的求解空间大。

优化目标与约束繁多，评估难度大：数字电路主要考虑时序、功耗、面积等性能指标，在布局布线核心引擎中可以抽象到互连线长这类易于计算的目标；而模拟电路的性能指标种类多样，与电路类型有关，大多需要通过电路仿真得到，评估时间长，计算难度大。

性能对版图设计敏感，搜索难度大：模拟电路的性能对版图设计敏感度高，对称、匹配等约束略微不满足就可能导致电路不工作。抽象来说，即在巨大的解空间中，只有少数几个版图设计方案是满足要求的，搜索这样的解相当于“大海捞针”。

设计依赖专家经验，自动化算法设计难度大：模拟电路版图设计通常无明确固定的范式，受设计者经验影响较大。不同设计者甚至可能画出风格迥异的版图，这对自动化算法设计提出了很大的挑战。

综上所述，这些挑战使模拟版图至今无法做到数字版图那样的自动化程度。随着模拟版图对自动化的需求不断增加，相应的自动化算法和工具仍需众多研究人员不断探索和推动。

模拟电路版图自动化软件



TODO


