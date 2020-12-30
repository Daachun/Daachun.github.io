# Unity Shader ShaderLab速查

# CG编程

CG通常采用动态编译的方式（也支持静态编译），在宿主程序运行时，利用CG运行库动态编译CG代码。

> 注意到了cemu模拟器玩塞尔达，有编译shader这一步骤

## CG基本数据类型

|  数据类型  |                             描述                             |
| :--------: | :----------------------------------------------------------: |
|   float    |  32位浮点数据，一个符号位，浮点数据类型被所有的profile支持   |
|    half    |                         16位浮点数据                         |
|    int     |    32位整形数据，有些profile会将int类型作为float类型使用     |
|   fixed    |         12位定点数，被所有的fragment profiles所支持          |
|    bool    | 布尔数据，通常用于if和条件操作符`?:`，布尔数据类型被所有的profiles支持 |
|  simpler*  | 纹理对象的句柄（the handle to a texture object），分为六类：sampler,sampler1D,sampler2D,sampler3D,samplerCUBE和samplerRECT。<br />DirectX profiles不支持samplerRECT类型，除此之外这些类型被所有的pixelprofiles和NV40 vertex program profile所支持。<br />不远的将来，顶点程序将广泛支持纹理操作。 |
|   string   | 字符类型，该类型不被当前存在的profile所支持，实际上也没有必要在CG程序中用到字符类型，但是你可以通过CG runtime API声明该类型变量，并赋值；因此，该类型变量可以保存CG文件的信息。 |
| ***备注*** | ***string基本不用。CG内置了向量数据类型，基于基础数据类习惯。例如：float4表示float类型的4元向量。*** |

## CG数组

|   类型   |                             描述                             |
| :------: | :----------------------------------------------------------: |
| 一维数组 | float a[10];// 声明了一个数组，包含10个float类型数据<br />float a[4] = {1.0, 2.0, 3.0, 4.0};// 初始化一个数组 |
| 多维数组 |                                                              |
|   备注   |                                                              |





## CG 结构体