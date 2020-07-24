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

