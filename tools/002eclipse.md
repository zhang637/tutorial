# Eclipse 快捷键
##一. 编辑

	Ctrl＋1	快速修复（最经典的快捷键,可以解决很多问题，比如import类、try catch包围等）
	Ctrl+Shift+F 格式化当前代码
	Ctrl+Shift+M 添加类的import导入
	Ctrl+Shift+O 组织类的import导入（既有Ctrl+Shift+M的作用，又可以帮你去除没用的导入，很有用）
	Ctrl+Y 重做（与撤销Ctrl+Z相反）
	Alt+/ 内容辅助（帮你省了多少次键盘敲打，太常用了）
	Ctrl+D 删除当前行或者多行
	Alt+↓ 当前行和下面一行交互位置（特别实用,可以省去先剪切,再粘贴了）
	Alt+↑ 当前行和上面一行交互位置（同上）
	Ctrl+Alt+↓ 复制当前行到下一行（复制增加）
	Ctrl+Alt+↑ 复制当前行到上一行（复制增加）
	Shift+Enter 在当前行的下一行插入空行（这时鼠标可以在当前行的任一位置,不一定是最后）
	Ctrl+/ 注释当前行,再按则取消注释

##二. 搜索
	Ctrl+K 参照选中的Word快速定位到下一个（如果没有选中word，则搜索上一次使用搜索的word）
	Ctrl+Shift+K 参照选中的Word快速定位到上一个
	Ctrl+J 正向增量查找（按下Ctrl+J后,你所输入的每个字母编辑器都提供快速匹配定位到某个单词,如果没有,则在状态栏中显示没有找到了,查一个单词时,特别实用,要退出这个模式，按escape建）
	Ctrl+Shift+J 反向增量查找（和上条相同,只不过是从后往前查）
	Ctrl+Shift+U 列出所有包含字符串的行
	Ctrl+H 打开搜索对话框
	Ctrl+G 工作区中的声明
	Ctrl+Shift+G 工作区中的引用
	Ctrl+Shift+H 递归查找谁引用了此变量、方法
##三. 导航
	Ctrl+Shift+T 搜索类（包括工程和关联的第三jar包）
	Ctrl+Shift+R 搜索工程中的文件
	Ctrl+E 快速显示当前Editer的下拉列表（如果当前页面没有显示的用黑体表示）
	F4 打开类型层次结构
	F3 跳转到声明处
	Alt+← 前一个编辑的页面
	Alt+→ 下一个编辑的页面（当然是针对上面那条来说了）
	Ctrl+PageUp/PageDown 在编辑器中，切换已经打开的文件
##四. 调试
	F5 单步跳入
	F6 单步跳过
	F7 单步返回
	F8 继续
	Ctrl+Shift+D 显示变量的值
	Ctrl+Shift+B 在当前行设置或者去掉断点
	Ctrl+R 运行至行(超好用，可以节省好多的断点)
##五、重构
	Alt+Shift+R 重命名方法名、属性或者变量名 （是我自己最爱用的一个了,尤其是变量和类的Rename,比手工方法能节省很多劳动力）
	Alt+Shift+M 把一段函数内的代码抽取成方法 （这是重构里面最常用的方法之一了,尤其是对一大堆泥团代码有用）
	Alt+Shift+C 修改函数结构（比较实用,有N个函数调用了这个方法,修改一次搞定）
	Alt+Shift+L 抽取本地变量（ 可以直接把一些魔法数字和字符串抽取成一个变量,尤其是多处调用的时候）
	Alt+Shift+F 把Class中的local变量变为field变量 （比较实用的功能）
	Alt+Shift+I 合并变量（可能这样说有点不妥Inline）
	Alt+Shift+V 移动函数和变量（不怎么常用）
	Alt+Shift+Z 重构的后悔药（Undo）