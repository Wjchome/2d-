# 2d-战旗
Unity项目


## 行动类介绍  

ActionClass，所有行动的父类  
重要属性：  

public int actionID;//行动id，唯一标识

public int damage;//行动伤害，某些行动无伤害

public int cooldown;//行动冷却

public int actionPointCost;//行动消耗技能点

public int maxRedGridNum;//最大攻击格子数

public TargetType targetType;//攻击目标类型

public List<Vector2> pointToPoint = new List<Vector2>();//能释放的相对位移格子

重要方法：
public virtual bool IsMeetAnyBuff()//是否满足什么前置buff，默认是需要控制没有眩晕Buff

public bool CheckIsMeetTarget(Character targetCharacter)//是否满足攻击目标为可选目标

## 子类介绍
ActionClass有4个子类 分别为 DelayAction DelayWaitAction InstantAction WaitAction ,分别代表延时行动，延时等待行动，立即行动，立即等待行动  
DelayAction代表行动释放后会等待几个回合，直到几个回合后优先执行，不会造成位移  
DelayWaitAction代表行动释放后，直到几个回合后优先执行，但是会造成位移  
InstantAction代表行动释放后，将会立即执行，不会造成位移  
WaitAction代表行动释放后，将会立即执行，但是会造成位移  

## 行动配置表
行动配置表是在ActionClass的Awake()函数中进行  
1.类的结构  

public class ActionTypeInfo
{
            public Type actionType { get; private set; }
            //商店中出现的权重
            public int weightValue { get; private set; }
            //商店中该行动的价格
            public int price { get; private set; }
            public string actionName { get; private set; }
            public string actionExplain{ get; private set; }
            public int damage, cooldown, actionPointCost, maxRedGridNum;
            public TargetType targetType;
            //这个是用来局外成长的，是否解锁
            public bool isUnlock;
            
            
            public ActionTypeInfo(Type actionType, int weightValue,int price,  string actionName, string actionExplain, 
            int damage, int cooldown, int actionPointCost, int maxRedGridNum,TargetType targetType,bool isUnlock)
            {
            this.actionType = actionType;
            this.weightValue = weightValue;
            this.price = price;
            this.actionName = actionName;
            this.actionExplain = actionExplain;
            this.damage = damage;
            this.cooldown = cooldown;
            this.actionPointCost = actionPointCost;
            this.maxRedGridNum = maxRedGridNum;
            this.targetType = targetType;
            this.isUnlock = isUnlock;
            }
}

2.配置表  

            public static Dictionary<int, ActionTypeInfo> actionIDToType = new Dictionary<int, ActionTypeInfo>()
            {
                {0, new ActionTypeInfo(typeof(Move), 10,2, "移动","移动至指定位置",0,1,1,1,TargetType.Space,true)},
                {1, new ActionTypeInfo(typeof(BoxingAction), 10,5, "武器攻击","攻击目标，造成一点伤害",1,1,1,1, TargetType.Character, true)},
                ......
            }
3.ActionClass中的初始化，根据actionId  

            public void Awake()
            {   
                character = GetComponent<Character>();
                animator = GetComponent<Animator>();
                spriteRenderer = GetComponent<SpriteRenderer>();
                yellows = GetComponent<Yellow>();
                actionIcon = Resources.Load<Sprite>("Action/"+GetType().Name);
                InitActionValue();
            }
            public void InitActionValue()
            {
                damage = SaveClass.actionIDToType[actionID].damage;
                cooldown = SaveClass.actionIDToType[actionID].cooldown;
                actionPointCost = SaveClass.actionIDToType[actionID].actionPointCost;
                maxRedGridNum = SaveClass.actionIDToType[actionID].maxRedGridNum;
                targetType= SaveClass.actionIDToType[actionID].targetType;
            
            }


### 举例说明如何创造新行动  
1.最普通的攻击  
                        
            public class BoxingAction : InstantAction
            {
            
                private new void Awake()
                {
                    actionID = 1;//行动的位移标识，必须首要标明
                    base.Awake();//用来进行配置表和其他操作
            
                    
                    //设置可选的相对位置 在此例中是周围8格
                    pointToPoint = new List<Vector2> { new Vector2(1, 0), new Vector2(0, 1),  new Vector2(-1, 0),  new Vector2(0, -1),new Vector2(1,1), new Vector2(1, -1), new Vector2(-1, 1), new Vector2(-1, -1) };
            
                    //一个记录
                    lastTimeAction = -100;
                }
            
                public override void MakeDirection()
                {        
                        //设置能攻击的位置，在此例中是攻击选定格的目标
                        directions = new List<Vector2>() { mouseRelativeSelf };   
                }
            
            
            
            
                public override IEnumerator CheckTarget()
                {
                    //是用来等待动画播放到什么时候开始真正开始生效攻击，此例中是指动画做到80%就开始真正生效，用来给玩家增加体验的
                    yield return new WaitForSeconds(0.8f * clipLength);
            
                    foreach (Vector2 direction in directions)
                    {
                        Vector2 targetPos =selfVector2 + direction;//找到真正位置坐标，通过自己位置增加相对位置
            
                        if (GridManager.Instance.IsInBox(targetPos))//判断是否在格子内
                        {
                            Grid targetGrid = GridManager.Instance.gridsXY[(int)targetPos.x, (int)targetPos.y];
                            if (CheckIsMeetTarget(targetGrid.gridCharacter))//判断是否是会攻击的目标
                            {
                                targetGrid.gridCharacter.ReceiveDamage(character, damage);//攻击
                            }
                        }
                    }
                           
                }
            
            }
            
            












