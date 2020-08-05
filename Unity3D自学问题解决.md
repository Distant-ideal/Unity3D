# Unity3D自学问题解决

## 1.unity中Animator controller无法给动作添加motion	

只需要在这个动作的上级模型里修改Rig参数Animation Type为generic即可

## 2.如何将贴图导入到模型中

在下方import new asset导入想要插入的贴图然后导入给模型即可

## 3.导入刚体后模型一直下坠

将Rigidboday属性中is Kinematic勾选

## 4.无法找到要运行的脚本

在Animation属性中找到size进行调整，将想要进行的动作添加进去

## 5.The AnimationClip 'run' used by the Animation component 'player' must be marked as Legacy.

FBX导入时将Rig中Animation Type选为Legacy

## 6.在完成脚本后要将物体拖拽到脚本对应位置

否则在运行脚本时候会出现找不到对象的问题

## 7.层级划分完成后一定要进行烘培

否则会导致物体无法进行移动

## 8.当脚本添加游戏对象过程中，弹出的出错窗口： “Can't add script....." ？

原因是Unity 规定脚本的文件名称必须与类名相同，否则报错。请更改Unity脚本的名称或者类的名称。

## 9.在学生学习导航寻路过程中，在运行过程中遇到的运行时错误信息：   "SetDestination" can only be called on an active agent that has been placed on a NavMesh"？

典型导航寻路错误，主要原因是你需要导航的游戏对象，放置的位置不对，要么y轴远离了“地面”（NavMesh）,要么离开了烘培的"地面"。请检查与更改相关寻路主角的Y轴位置

## 10.用户拿到的工程文件，发生打不开的错误（不报错）。 也就是Unity 无论怎样都打不开指定的Unity 项目？

一般是因为Unity 对中文支持的不好，所以工程所在路径不能有中文。 请把你的工程文件的所在路径进行检查，把相关中文路径去除即可。

## 11.当用户导入.unitypackage 文件的过程中显示错误信息：  "Error While importing package: Couldn't decompress package.Failed importing package ....."?。

*这个问题一般也是因为Unity 对中文支持的不好，所以需要导入的*.unitypackage 文件所在路径不能有中文。 请把你的“包”（或者一些*.unitypackage 插件）文件的所在路径进行检查，把相关中文路径去除即可

## 12.程序运行过程中出现的一个运行时错误信息： “MissingReferenceException: The object of type 'GameObject' has been destroyed”

缺少引用异常！通常原因是由于指定的游戏对象已经销毁了，而其他代码还要访问（调用），造成的错误！。

## 13.添加动画事件时，若发现画是只读的Read—only不能添加监听事件

解决方法：重新复制一份动画，用重新复制的这份动画。

## 14.You are trying to create a MonoBehaviour using the 'new' keyword

习惯了写单例，或者常规的单例模式

当你继承了MonoBehaviour 如果你这样写

你会发现instance 永远为空的（即使走了这一步的instance = new Single();） 而且你回收到如下的警告

You are trying to create a MonoBehaviour using the 'new' keyword.  This is not allowed.  MonoBehaviours can only be added using AddComponent().  Alternatively, your script can inherit from ScriptableObject or no base class at all



