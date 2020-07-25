# Unity 3D自学

## c#基础

```
//输出方法
//print("Hello World");
//Debug.Log("Hello world");

object obj1 = 10 //对象类型 特点 什么类型都可以存储 任何类型都是object类型的子类
object obj2 = "xia"
object obj3 = 10.5f
//创建变量的时候相当于创建了一个盒子存放值 值类型
//string和object类型存放的不是物品存放的是地址


//从大范围转化为小范围的需要强转 显式转换
object obj1 = 10;
int num = obj1; //这样是不允许的
int num = (int)obj1; 

//隐式转换 小范围转换为大范围
int number1 = 10;
long number2 = 100;
number2 = number1; 

//显式转换 小范围转换为大范围
int number1 = 10;
long number2 = 100;
number1 = (int)number2; 

//int string类型的转换
int hp = 300;
string str = hp.ToString();
print(str.GetType()); //打印类型

//string转整形
string str2 = "18";
int num = int.Parse(str2);

print(num);
print(num.GetType());

```

## Animator组件

我们需要播放动画的角色都需要添加Animator组件，该组件即为我们控制动画的接口。

- Controller：使用的Animator Controller文件。
- Avatar：使用的骨骼文件。
- Apply Root Motion：绑定该组件的GameObject的位置是否可以由动画进行改变（如果存在改变位移的动画）。
- Update Mode：更新模式：Normal表示使用Update进行更新，Animate Physics表示使用FixUpdate进行更新（一般用在和物体有交互的情况下），Unscale Time表示无视timeScale进行更新（一般用在UI动画中）。
- Culling Mode：剔除模式：Always Animate表示即使摄像机看不见也要进行动画播放的更新，Cull Update Transform表示摄像机看不见时停止动画播放但是位置会继续更新，Cull Completely表示摄像机看不见时停止动画的所有更新。

新建一个 `Animator Controller`。

发现的有3个默认的状态，这些状态是Unity自动帮我们创建的同时也无法删除：

- Entry：表示当进入当前状态机时的入口，该状态连接的状态会成为进入状态机后的第一个状态；
- Any State：表示任意的状态，其作用是其指向的状态是在任意时刻都可以切换过去的状态；
- Exit：表示退出当前的状态机，如果有任意状态指向该出口，表示可以从指定状态退出当前的状态机；

从资源包里面添加 5 种状态，不知道怎么回事，run 这个状态无法添加，我暂时添加了 idle5, attack1, attack2, apell1, dancel 这 5 种状态。

新建一个 `Int` 形变量 `state` 初始值为 0，表示 idle 状态，idle5, attack1, attack2, apell1, dancel 依次为 0-4 这几种状态。

5 种状态之间相互 Make Transition，添加状态变化的条件。

## 动作控制器

1.要准备好所需要的motion

2.对每一个可以互通的动作进行make transition进行连线表示可以动作进行切换

3.创建一个int标志，对每一条线都要做返回值表示在这个值等于多少时动作进行且切换

## 自制模拟摇杆

引入一个image方法 修改图像为圆形对左下角对齐然后修改坐标向右上方移动

在image方法中引入一个button方法将其中text删除，修改坐标使其与image原点相同，将button大小修改为image的一半修改button图形为圆形

给image添加一个脚本

### 脚本

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.EventSystems;

public class ButtonMove : MonoBehaviour, IDragHandler, IEndDragHandler
{
    public static ButtonMove _buttonmove;
    private Vector2 _begin; //初始位置
    private float range = 25; //定义最大范围
    void Awake() //结束拖拽之后回复到初始位置
    {
        _buttonmove = this;
        _begin = transform.position;
    }
    public void OnDrag (PointerEventData other)
    {
        transform.position = other.position;
        float myrange = Vector3.Distance(transform.position, _begin); //根据传入参数可以计算点到点的范围
        if(myrange > range)
        {
            transform.position = _begin + (other.position - _begin).normalized * range; //超出范围之后控制移动
        } else
        {
            transform.position = other.position; //不超出范围可以随意移动
        }
    }

    public void OnEndDrag(PointerEventData other)
    {
        transform.position = _begin;
    }

    public float X //返回x轴的位置
    {
        get { return ((Vector2)transform.position - _begin).normalized.x; }
    }

    public float Y//返回y轴的位置
    {
        get { return ((Vector2)transform.position - _begin).normalized.y; }
    }
}
```

## 控制模型移动

### 脚本

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class playermove : MonoBehaviour
{
    private Rigidbody _rig;
    private Animation _ani;

    void Start()
    {
        _rig = GetComponent<Rigidbody>();
        _ani = GetComponent<Animation>();
    }


    void Update()
    {
        float X = ButtonMove._buttonmove.X;
        float Y = ButtonMove._buttonmove.Y;
        Vector3 dir = new Vector3(X, 0, Y);
        if(dir != Vector3.zero)
        {
            transform.rotation = Quaternion.Lerp(transform.rotation, Quaternion.LookRotation(dir), Time.deltaTime * 10);
            _rig.velocity = dir * 7; //给一个朝向dir的力
            _ani.Play("run");
            _rig.isKinematic = false;//物理引擎关闭
        } else
        {
            _ani.Play("idle1");
            _rig.isKinematic = true;
        }
    }
}

```

## 镜头跟随

### 脚本

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CameraMove : MonoBehaviour
{
    private Transform _player;
    private Vector3 _vec;
    // Start is called before the first frame update
    void Start()
    {
        _player = GameObject.Find("Player").transform; //读取玩家位置
        _vec = _player.position - transform.position; //计算偏移量
    }

    // Update is called once per frame
    void Update()
    {
        transform.position = _player.position - _vec; //用玩家位置减去偏移量
    }
}
```

## 小兵生成

### 脚本

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CreatSoldier : MonoBehaviour {
	[SerializeField]
	private GameObject soldier;
	[SerializeField]
	private Transform startTran;
	[SerializeField]
	private Transform soldierParent;

	//是否生成小兵默认为true
	bool isCreatSoldier = true;
	//生成小兵的数量
	public int soldierCount = 2;

	void Start() {
		StartCoroutine (Creat (0, 1, 5));
	}

	//生成一个小兵
	void CreatSmartSoldier() {
		GameObject obj = Instantiate (soldier, startTran.position, Quaternion.identity);
		//设置父物体（子物体生成后会在父物体中）
		obj.transform.parent = soldierParent; 
	}

	//协程生成一波一波小兵
	// time游戏开始后几秒开始生成士兵
	//delyTime同一波内两个小兵生成的间隔
	//spwanTime下一波小兵生成的时间间隔

	private IEnumerator Creat(float time, float delyTime, float spwanTime) {
		yield return new WaitForSeconds(time);
		while(isCreatSoldier) {
			//一个for循环代表一波小兵
			for(int i = 0; i < soldierCount; i++) {
				CreatSmartSoldier();
				yield return new WaitForSeconds(delyTime);
			}
			//等待下一波小兵生成的时间
			yield return new WaitForSeconds(spwanTime);
		}
	}
}

```

## CrossFade

```
if( Input.GetKeyDown("w") )
{

animation.Play("run");

}

如果想让两个动画切换时平滑过渡，将Play() 函数改为CrossFade("AnimName", fadeTime);

第一个参数为动画的名字，第二个参数为第一个动画和第二个动画开始的“过渡”时间。
```

## 小兵自动寻路

给小兵物体添加一个Nav Mesh Agent插件

在地图上创建cub完成路径后对路径进行烘培

### 脚本

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.AI;

public class SmartSoldier : MonoBehaviour {

	private NavMeshAgent nav;
	private Animation ani;
	public Transform target;
	public Transform[] towers; //防御塔

	void Start()
	{
		nav = GetComponent<NavMeshAgent>();
		ani = GetComponent<Animation>();
	}

	void Update() {
		SoldierMove();
	}

	void SoldierMove() {
		if(target == null) { //判定防御塔是否还存在 如果不存在重新获取防御塔信息
			target = GetTarget();
			return ;
		}		
		ani.CrossFade ("Run");
		nav.SetDestination(target.position);
	}
	
	Transform GetTarget() {
		for(int i = 0; i < towers.Length; i++) {
			if(towers[i] != null) {
				return towers [i];
			}
		}
		return null;
	}


}
```

### 对小兵生成文本的改动

添加了防御塔对象数组和对物体移动后的改变

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CreatSoldier : MonoBehaviour {
	[SerializeField]
	private GameObject soldierPrefab;
	[SerializeField]
	private Transform startTran;
	[SerializeField]
	private Transform soldierParent;
	//虽然目标点对象不能直接托给预制体，但是我们可以在创建预制体时给他赋值
	[SerializeField]
	Transform[] towers;


	//是否生成小兵默认为true
	bool isCreatSoldier = true;
	//生成小兵的数量
	public int soldierCount = 2;

	void Start() {
		StartCoroutine (Creat (0, 1, 5));
	}

	//生成一个小兵

	void CreatSmartSoldier(Transform startTran, Transform[] towers) {
		GameObject obj = Instantiate(soldierPrefab, startTran.position, Quaternion.identity) as GameObject;
		//设置父物体
		obj.transform.parent = soldierParent; 

		SmartSoldier soldier = obj.GetComponent<SmartSoldier> (); //获取脚本
		soldier.towers = towers;
	}

	//协程生成一波一波小兵
	// time游戏开始后几秒开始生成士兵
	//delyTime同一波内两个小兵生成的间隔
	//spwanTime下一波小兵生成的时间间隔

	private IEnumerator Creat(float time, float delyTime, float spwanTime) {
		yield return new WaitForSeconds(time);
		while(isCreatSoldier) {
			//一个for循环代表一波小兵
			for(int i = 0; i < soldierCount; i++) {
				CreatSmartSoldier(startTran, towers);
				yield return new WaitForSeconds(delyTime);
			}
			//等待下一波小兵生成的时间
			yield return new WaitForSeconds(spwanTime);
		}
	}
}

```

## 多路小兵生成

创建多个cub将其放在指定位置保证小兵可以顺利进行移动；

在navigation中找到Areas设置上中下的层级遮罩

user 1代表2的1次方user 2代表2的2次方以此类推来判断层级

所以在脚本中我们定义int型变量来读取层级信息

注意：在烘培的时候要选择要进行的所有路径进行赋予层级（在object下的navigation Area进行勾选）

### 生成小兵脚本改动

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CreatSoldier : MonoBehaviour {
	[SerializeField]
	private GameObject soldierPrefab;

	[SerializeField]
	private Transform soldierParent;
	//是否生成小兵默认为true
	bool isCreatSoldier = true;
	//生成小兵的数量
	public int soldierCount = 2;

	//三路敌我防御塔的数组
	//虽然目标点对象不能直接托给预制体，但是我们可以在创建预制体时给他赋值
	[SerializeField]
	private Transform[] middleTowers;
	[SerializeField]
	private Transform[] LeftTowers;
	[SerializeField]
	private Transform[] RightTowers;
	[SerializeField]
	private Transform[] middleEnemyTowers;
	[SerializeField]
	private Transform[] LeftEnemyTowers;
	[SerializeField]
	private Transform[] RightEnemyTowers;

	//生成点数组
	[SerializeField]
	private Transform[] Start1;
	[SerializeField]
	private Transform[] Start2;



	void Start() {
		StartCoroutine (Creat (0, 1, 5));
	}

	//生成一个小兵
	//层级表示的时2的几次方所以时int型的
	void CreatSmartSoldier(Transform startTran, Transform[] towers, int road) {
		GameObject obj = Instantiate(soldierPrefab, startTran.position, Quaternion.identity) as GameObject;
		//设置父物体
		obj.transform.parent = soldierParent; 

		SmartSoldier soldier = obj.GetComponent<SmartSoldier> (); //获取脚本
		soldier.towers = towers; //指定目标防御塔
		soldier.SetRoad(road);
	}

	//协程生成一波一波小兵
	//time游戏开始后几秒开始生成士兵
	//delyTime同一波内两个小兵生成的间隔
	//spwanTime下一波小兵生成的时间间隔
	private IEnumerator Creat(float time, float delyTime, float spwanTime) {
		yield return new WaitForSeconds(time); //几秒后开始生成小兵
		while(isCreatSoldier) {
			//一个for循环代表一波小兵
			for(int i = 0; i < soldierCount; i++) { //在哪一路生成小兵
				CreatSmartSoldier(Start1[0], middleEnemyTowers, 1 << 3); //2的3次方
				CreatSmartSoldier(Start2[0], middleTowers, 1 << 3);

				CreatSmartSoldier(Start1[1], LeftEnemyTowers, 1 << 4);
				CreatSmartSoldier(Start2[1], LeftTowers, 1 << 4);

				CreatSmartSoldier(Start1[2], RightEnemyTowers, 1 << 5);
				CreatSmartSoldier(Start2[2], RightTowers, 1 << 5);


				yield return new WaitForSeconds(delyTime);//生成下一个小兵的时间间隔
			}
			//等待下一波小兵生成的时间
			yield return new WaitForSeconds(spwanTime); //生成下一波的时间间隔
		}
	}
}

```

### 小兵自动寻路脚本改动

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.AI;

public class SmartSoldier : MonoBehaviour {

	private NavMeshAgent nav;
	private Animation ani;
	public Transform target;
	public Transform[] towers; //防御塔

	void Start()
	{
		nav = GetComponent<NavMeshAgent>();
		ani = GetComponent<Animation>();
	}

	void Update() {
		SoldierMove();
	}

	void SoldierMove() {
		if(target == null) { //判定防御塔是否还存在 如果不存在重新获取防御塔信息
			target = GetTarget();
			return ;
		}		
		ani.CrossFade ("Run"); //执行小兵中的跑脚本
		nav.SetDestination(target.position); //读取防御塔的位置
	}
	
	Transform GetTarget() { //获取防御塔
		for(int i = 0; i < towers.Length; i++) {
			if(towers[i] != null) {
				return towers [i];
			}
		}
		return null;
	}

	public void SetRoad(int road)  //设置路线
		nav = GetComponent<NavMeshAgent> ();
		nav.areaMask = road; //层级遮罩 //根据层级确定行走路线
	}
}

```

## 添加标签

在Edit中project settings中找到Tags和Layers点击进入

在右侧可以添加标签

## 防御塔类型

给每一个防御塔更改标签为Tower或者EnemyTower进行敌我防御塔区分

### 脚本

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Tower : MonoBehaviour {

	public int towerType;
	// Use this for initialization
	void Start () {
		if (this.gameObject.tag.Equals ("Tower")) { //读取标签看是否是地方箭塔 //为了区分我方箭塔和小兵
			towerType = 0; //为了和小兵进行匹配所以使用int型
		} else {
			towerType = 1;
		}
	}
	
	// Update is called once per frame
	void Update () {
		
	}
}

```

## 防御塔范围

添加碰撞体如果小兵进入碰撞体就插入到list中如果小兵消失出List

英雄同理

### 脚本

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Tower : MonoBehaviour {

	public List <GameObject> listSoldier = new List<GameObject> (); //小兵攻击箭塔队列
	public List <GameObject> listHero = new List<GameObject> (); //英雄攻击箭塔队列
	public int towerType;


	// Use this for initialization
	void Start () {
		if (this.gameObject.tag.Equals ("Tower")) { //读取标签看是否是地方箭塔 //为了区分我方箭塔和小兵
			towerType = 0; //为了和小兵进行匹配所以使用int型
		} else {
			towerType = 1;
		}
	}
	
	void OnTriggerEnter(Collider col) { //触发函数
		if (col.gameObject.tag == "Player") {
			listHero.Add (col.gameObject);
		} else {
			SmartSoldier soldier = col.GetComponent<SmartSoldier> ();
			if (soldier && soldier.type != towerType) { //判断小兵是否不存在或者类型不匹配
				//print(listSoldier.Count);
				listSoldier.Add (col.gameObject); //插入到队列中
			}
		}
	}

	void OnTriggerExit(Collider col) {
		if(col.gameObject.tag == "Player") {
			listHero.Remove (col.gameObject);
		} else {
			listSoldier.Remove (col.gameObject); //删除出队列
		}
	}
} 
```

## 子弹移动和伤害

创建子弹预制体

给预制体添加rigidboday和spherecolider组件使子弹与小兵英雄可以发生碰撞

### 脚本

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Bullet : MonoBehaviour {
	public Tower tower;
	private GameObject target;
	// Use this for initialization

	public float speed = 20f;

	void Start () {
		tower = GetComponentInParent<Tower> (); //箭塔对于子弹来说相当于父物体
		Destroy (this.gameObject, 1f);
	}
	 
	void OnTriggerEnter(Collider col) {
		if (col.gameObject.tag == "Soldier") {
			//销毁小兵
			Health hp = col.GetComponent<Health>();
			if (hp) {
				hp.TakeDamage (0.5f);
				if (hp.hp.Value <= 0) {
					tower.listSoldier.Remove (col.gameObject);
					Destroy (col.gameObject);
				}
				Destroy (this.gameObject);
			}
		} else if(col.gameObject.tag == "Palyer"){ //防止触发箭塔条件
			//销毁英雄
			Health hp = col.GetComponent<Health>();
			if (hp) {
				hp.TakeDamage (0.5f);
				if (hp.hp.Value <= 0) {
					tower.listHero.Remove (col.gameObject);
					Destroy (col.gameObject);
				}
				Destroy (this.gameObject);
			}
		}
	}

	public void SetTarget(GameObject target) { 
		this.target = target;
	}

	// Update is called once per frame
	void Update () {
		if(target) {
			Vector3 dir = target.transform.position - transform.position;//三维向量
			GetComponent<Rigidbody>().velocity = dir.normalized * speed;
		} else {
			Destroy (this.gameObject);
		}
	}
}
```

## 血条显示

将血条值作为背景的子物体

### 脚本

```

using UnityEngine;
using System.Collections;
[ExecuteInEditMode]
public class spriteSlider : MonoBehaviour {
 
    [SerializeField]
    private Transform front;
    public float m_value;
 
    public float Value
    {
        get
        {
            return m_value;
        }
        set
        {
            m_value = value;
            front.localScale = new Vector3(value, 1, 1);
            front.localPosition = new Vector3((1 - m_value) * -0.8f, 0);//这个式子可以自己设置数值关观察下  调整数值使减血后的血条向左对齐
          
        }
    }   
}
```

## 减血方法

### 脚本

```

using UnityEngine;
using System.Collections;
 
public class Health : MonoBehaviour {
 
    public spriteSlider hp;
    private uint value=1;//uint 表示值不可能为负
 
	// Use this for initialization
	void Start () {
        hp=GetComponentInChildren<spriteSlider>();//代码找到组建，不用一个个拖拽
        Init();
	}
 
    public void Init()
    {
        hp.Value = value;//血条初始化值为1
    }
    //减血的方法
    public void TakeDamage(float damage)
    {
 
        hp.Value -= damage;
    }
	
}
```

## 将事件挂载到动画上

找到动画

找到预制体摁ctirl+6或者在windows中找到Animation

进入后如果是只读（无法插入事件）的动画就找到原动画ctirl+d重新生成一个自己的动画就可以插入事件了

找到想要插入的帧数添加事件

一般动画只添加一个事件防止出现错误





























