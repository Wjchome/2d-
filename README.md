# 2d-战旗
Unity项目


行动类介绍  

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

public bool CheckIsMeetTarget(Character targetCharacter)//是否满足




