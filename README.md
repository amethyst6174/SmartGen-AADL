# SmartGen-AADL:多智能体协同的系统架构需求分析与AADL模型自动生成方法

## *摘要*：
嵌入式系统建模是基于模型的软件开发的重要组成,AADL因其形式化表达软硬件结构与交互关系的能力,广泛用于架构设计.大语言模型(LLMs)为从自然语言需求生成架构模型提供了新路径.然而,现有模型在需求语义理解、AADL组件边界识别与连接关系建构等方面仍存在显著不足,限制了其实用性与生成质量.为解决上述问题,本文提出一种面向嵌入式系统的智能建模方法—SmartGen-AADL,整体方法基于多智能体协同机制构建,融合语义解析、结构识别与提示增强生成等关键技术,实现从自然语言需求到结构化AADL模型的高质量转换.方法核心包括三个阶段:首先,系统通过结构化智能体完成系统架构文档中系统架构的识别与标准化需求语句的提取;随后,子问题智能体基于条目级分析与组件交互挖掘,实现对需求粒度的细化与交互关系的显式建模;最后,构件生成智能体在语义提示中融合结构引导与相似组件检索结果,引导大语言模型生成符合AADL语法规范与建模约束的组件代码.为支撑上述流程,本文构建了“条目化需求-AADL组件”的知识库以及“系统架构文档—AADL架构”语义对齐的数据集.在多个典型嵌入式系统任务上开展的实验表明,相较当前主流大语言模型,所提方法将生成组件代码错误率平均降低35.47%,F1 提升7.72%.进一步分析显示,子问题识别机制增强了对建模粒度的控制能力;系统结构树构建提供了组件组织与层级拓扑信息;通信连接识别确保了模型的接口完备性与交互闭环,三者协同促进了自然语言到AADL建模语言对齐与模型一致性的显著提升.

---

#### 项目说明
我们将本项目的核心模块部署在一个[轻量级云服务器](http://43.143.182.28:8080/)上，您可以基于此试验本文所属的从需求文档到AADL模型的转换
同时，您可以在[aadl_example](https://github.com/amethyst6174/aadl_example)中找到本文总结的更多样例

*注意事项*：
- 需求文档请使用`.docx`格式上传
- 由于服务器的承载能力，我们使用API调度模型构建智能体，因此在白天高峰期响应速度会受到限制，极端情况下可能出现链接超时问题

---

#### 演示说明
您可以上传需求文档，并等待最终的传回的压缩文件，其中：
- `summary.txt`展示了组件结构树
- `output.txt`对需求进行拆分理解
- `step1`进行需求细化与分析
- `step2`进行知识库查询
- `step3`创建`AADL`文件

下文展示了一个空调系统架构需求文档的生成周期

![首界面](./init.png)

![上传文件](./upload.png)

![等待响应](./working.png)

![接收结果](./success.png)

---

#### 提示词说明

本文应用智能体设计AADL代码生成的多个阶段，下文将逐一列举：

###### *结构树生成*

其中：
- `requirement`来自原始需求文档

```
<background>
你是一名精通AADL语言的软件工程师，正在根据一份需求文档分析其对应AADL项目的结构
</background>

<adaptive_thinking_framework>
你应当严格遵循以下思考步骤进行分析：
  - 整体概述：明确文档对应的项目
  - 章节排布：明确各章节之间的关系，（例如：按照功能分章、按照组件类型分章、或其他）
  - 组件分析：详细理解每章节内容，从aadl语言的13类组件（system、process、thread、subprogram、data、processor、memory、bus等）的角度解释文档内容（注意：有些组件并没有显示的定义在文章中，例如一些subprogram功能）
  - 构建组件树：根据上一步骤的组件内容，综合文章信息，给出树形结构的组件层次关系
</adaptive_thinking_framework>

<output_example>
以下是一个组件树的结构样例：
System: 飞机高度控制系统 (Height_Control_System)
├── Device: 气压传感器 (Pressure_Sensor)
├── Device: 空速控制指令传感器 (Control_Command_Sensor)
├── Processor: 高度计算分区 (Height_Computation_Processor)
├── Process: 高度计算任务 (Height_Calculation_Process)
│   └── Thread: 高度计算线程 (Height_Calculation_Thread)
│       └── Subprogram: 高度计算程序1 (Altitude_Calc_Program)
├── Processor: 高度响应分区 (Height_Response_Processor)
├── Process: 高度响应任务 (Height_Response_Process)
│   └── Thread: 高度响应线程 (Height_Response_Thread)
├── Device: 显示器 (Display_Device)
├── Device: 油门控制单元 (Throttle_Controller)
├── Device: 升降舵控制单元 (Elevator_Controller)
├── Device: 副翼控制单元 (Aileron_Controller)
├── Bus: PCIe总线 (PCIe_Bus_1, PCIe_Bus_2, PCIe_Bus_3)
</output_example>

<instruction>
你应当遵从以下指令，以优化结果：
- 构成组件树后，再次检查需求文件，*必须*确保没有遗漏组件，不需要给出分析过程
- *禁止*自由创造不属于需求文档的内容，严格遵循说明
- 组件关系应当遵从aadl规范（例如sunprogram应当位于thread内而不是其他位置）
</instruction>

<input>
以下是你将阅读的需求文档：
{requirement}
</input>

<output>
你将按照以下要求输出：
- 输出一个完整的组件树结构
- *禁止*给出任何多余的说明，我将进行正则提取
- 必须使用组件英文名称
<output>
```

###### *组件细节分析*

其中：
- `requirement`来自需求文档
- `summary`来自上一环节生成的组件树

```
<background>
你是一名精通AADL语言的软件工程师，正在根据一份需求文档和一份组件结构树，分析其对应AADL项目的组件细节
</background>

<adaptive_thinking_framework>
你应当严格遵循以下思考步骤进行分析：
  - 整体概述：明确文档对应的项目
  - 章节排布：明确各章节之间的关系，（例如：按照功能分章、按照组件类型分章、或其他）
  - 确认对应关系：根据组件树中的每个组件，确定其在原文章对应的段落（也许不止一处）
  - 分析组件属性：根据上一步骤确定的一系列段落内容，记录每个组件的持有的属性信息
  - 分析组件连接关系：分析组件间的连接关系（绑定、包含、连接等）
  - 模块组织：通过若干功能模块组织上述信息
</adaptive_thinking_framework>

<instruction>
你应当遵从以下指令，以优化结果：
  - 对于每个组件都应当详细检查原需求文件内容，*必须*确保属性没有遗漏
  - *禁止*自由发挥，构建不存在的属性
  - 连接关系*严格遵守*AS5506C语法规范，不能构建不合法的连接方式（例如process与processor应该在system绑定，而不是在其他环节）
</instruction>

<output_example>
以下给出了一个模块的输出样例：
$module start$
飞行控制模块
模块名称：
    flight_control
所含组件：
    Device: Threttle_Controller 
    Device: Elevator_Controller 
    Device: Aileron_Controller
内部子组件：
    无
功能说明：
    接受来自“高度相应模块”的Helight_Response_Processor的信号
具体行为：
    控制发动机推力（油门）
    调整飞机仰角（升降舵）
    稳定飞行方向（尾翼）
关联组件：
    processor: height_Response_Processor
    Bus: PCIe_Bus_3
$module end$
</output_example>

<input>
以下是你将阅读的文档：
[原始需求]：{requirement}
[组件树]：{summary}
</input>

<output>
你将按照以下要求输出：
  - 按照你划分的功能模块，参照example格式输出
  - ‘所含组件’: 代表当前模块内高层级组件
  - ‘内部子组件’: 代表‘所含组件’之中更细节的组件划分
  - ‘功能说明’: 概括了该模块的功能
  - ‘具体行为’: 反映了模块内各个组件的属性，功能
  - ‘关联组件’: 指代并不定义在当前模块内，但与项目中当前模块内组件有关的外部组件
  - 每个模块使用‘$module start$’与‘$module end$’分隔
  - *禁止*输出其他说明与无关内容，便于我进行正则
  - *严格遵守*我给出的结构与分隔符
  - 必须使用各个组件的英文名称
</output>
```

###### *组件细节分析*

其中：
- `req_slice`来自需求文档的切片
- `analysis_data`来自第二环节进行的组件需求细节分析
- `summary`来自第一环节生成的组件树
- `rag_prompt`来自RAG环节

同时：
- `syntax_filter`用于修复生成结果的语法缺陷。应当根据使用模型动态调整，或根据当前生成效果动态补增。例如，使用`deepseek-r1`模型可能需要强调`禁止在开始处标注aadlversion 2;`，使用`claude-3-7`模型可能需要说明将`implementation`与`type`在
分别声明与构建，而不能合为整体
```
<background>
你是一名熟悉 AADL（Architecture Analysis and Design Language）语言标准的系统建模专家。请根据以下信息，生成符合AADL语法的结构化代码
</background>

<adaptive_thinking_framework>
# 你将获得以下信息：原始需求(来自一个完整项目的部分)，对需求的分析，项目组件架构，通过知识库检索到的规则(这一项可能为空)
- *原始需求*是你关注的重点
- *对需求的分析*用来辅助你思考，但它可能有误区，若有冲突依据*原始需求*为准
- *项目组件架构*描述了完整的项目中所含的主要组件和架构关系，能帮助你理解需求调用与连接关系
- *通过知识库检索到的规则*只是提示一些语法规则与相关组件提示信息，它与需求本身无关，提示的规则未必有用，可以选择性忽略，有时该模块本身不会存在
</adaptive_thinking_framework>

<original_requirement>
# 原始需求的使用遵循以下规则：
- 原始需求描述的内容是一个完整项目的部分功能模块
- *所含组件*与*内部子组件*是你在代码中需要定义或实例化的，而*关联组件*在其他文件有详细定义，它的用途是帮助你进行组件链接
- 为了确保代码能进行检验，仅当需要调用*关联组件*时，你可以简单定义，其他情况下则不需要编写定义
- *功能说明*和*具体行为*描述代码的功能用途，你需要通过aadl代码实现
- 仅当*功能说明*和*具体行为*某些条目无法用aadl代码描述（若aadl代码足以完成描述，禁止使用下文C代码格式），你可以将这些特殊条目通过调用C代码的格式实现，其格式为：
    ```
    Source_Language => (C); -- 源语言为，需在实现中连接函数
    Source_Name     => "function_name: xxxx"; -- 指定应连接的外部函数名
    Source_Text     => ("source_file: xxx.c");-- 指定该函数所在的源文件
    ```
- 以下是*原始需求*的内容：{req_slice}
</original_requirement>

<analysis_of_requirement>
# 需求的分析遵循以下规则：
- 按照字典列表逐条给出了对需求的细致分析
- 信息不一定完全正确，只作为你的生成参考，若出现冲突以*原始需求*为标准
- 以下是对需求分析：{analysis_data}
</analysis_of_requirement>

<summary_of_total_project>
# 项目组件架构的使用遵循以下规则：
- 该部分提供了项目整体使用的组件的概述以及层级/调用关系
- 该关系帮助你构建各个组件的包含/调用/连接关系
- 以下是对于项目整体的架构设计{summary}
</summary_of_total_project>

<rag_data>
# 检索遵循以下规则：
- 下文的信息来自AS5506C语法库与相关知识库，但与待生成目标无直接联系
- 对于每条检索在应用前进行判断，若无用则忽略本条信息
</rag_data>

<output>
# 你将按照一下规则输出aadl代码：
- 务必确保语法的正确性，这是你的第一前提
- 严格遵循标准 AADL 语法（AS5506C）
- 输出内容使用以下结构标记进行包裹（注意有两组尖括号）：
   <<BEGIN_AADL>>：代码开始
   <<END_AADL>>：代码结束
- 禁止使用未声明过的属性/组件；允许调用Base_types等官方默认package，但必须通过with引用，且需要放置在public后
- 部分组件虽然并不在当前模块中声明，但由于其被引用，你需要进行简单声明，以确保代码直接通过OSATE语法检验
- 禁止创建多余的组件代码；不要输出多余解释，禁止在开始处标注`aadlversion 2;`，仅输出符合要求的代码，且必须使用英文
- 使用形如下文
package model
public
with xxx;
...
end model;
格式包裹整个package，其中model应当替换为本模块名称，xxx是可能引用的官方package
</output>

<syntax_filter>
1. 禁止使用未声明过的属性/组件；允许调用Base_types等官方默认package，但需要通过with引用
2. 禁止在开始处标注`aadlversion 2;`，必须仅输出符合要求的代码，且aadl所有代码必须使用英文，禁用中文
3. 子程序调只能定义在组件的实现（implementation）中，格式如下，注意分号的使用
        calls function_name:{{
             calc_sequence: subprogram Altitude_Calc_Program.Impl;}};
其中function_name和calc_sequence都是可替换的名称
4. 对于一些属性的赋值，若该属性值并非官方定义，使用类似
Communication_Properties::Protocol => "ISO11898_1";
形式描述，而不是其他（此处仅为示例，并非真实情况）
5. 所有的实现(implementation)，必须要在其前进行声明，而不是直接使用
6. 有些组件虽然并不在当前模块中声明，但由于其被引用，你需要进行简单声明，以确保代码直接通过OSATE语法检验
7. 组件声明中的features和组件实现（implementation）中的properties不能混用，有些属性或连接需要在这两种结构内使用
8. 若features，properties/connection没有属性，则不需要声明这些关键词
</syntax_filter>
```