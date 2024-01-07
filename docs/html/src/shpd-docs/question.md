# 编译常见问题汇总:


# A类型：环境相关

## 1.Time outputs

出现这种问题通常是网络不佳，请更换网络或尝试使用科学上网。

## 2.Error Build Tool No Support Version 

出现这种问题是SDK和BTK不匹配，请更改为指定匹配环境

## 3.StackOver No Error Logs

出现这种问题通常是Gradle配置问题，请通过控制台详情报错修复

## B类型：地牢相关

## 1.NoTextFound

出现这种问题通常是因为写了新的多语言字段但语言内没有补上，  
遇到这种问题，请考虑使用字段缺失捕获器，或者自行检查。

注意：出现以下情况则需要考虑
```java
package com.shatteredpixel.shatteredpixeldungeon.actor.mobs;
// 类名 Class: Rat
Messages.get(this,"name")
```

这样与之对应的则应该是：   
actor.mobs.rat.name=xxxxx   

注意需要全部小写，不能使用大写，否则游戏会无法识别。

### 字段缺失检查器
#### 1.4.3优化版本

```java
	/**
	 * Resource grabbing methods
	 * 字段缺失检测器
	 */

     public static String get(String key, Object...args){
		return get(null, key, args);
	}

	public static String get(Object o, String k, Object...args){
		return get(o.getClass(), k, args);
	}

	public static String get(Class c, String k, Object...args) {
		return get(c, k, null, args);
	}

	private static String get(Class c, String k, String baseName, Object...args){
		String key;
		if (c != null){
			key = c.getName();
			key = key.replace("com.shatteredpixel.shatteredpixeldungeon.", "");
			key += "." + k;
		} else
			key = k;

		String value = getFromBundle(key.toLowerCase(Locale.ENGLISH));
		if (value != null){
			if (args.length > 0) return format(value, args);
			else return value;
		}  else {
			//Use baseName so the missing string is clear what exactly needs replacing. Otherwise, it just says java.lang.Object.[key]
			if (baseName == null) {
				baseName = key;
			}
			//this is so child classes can inherit properties from their parents.
			//in cases where text is commonly grabbed as a utility from classes that aren't mean to be instantiated
			//(e.g. flavourbuff.dispTurns()) using .class directly is probably smarter to prevent unnecessary recursive calls.
			if (c != null && c.getSuperclass() != null){
				return get(c.getSuperclass(), k, baseName, args);
			} else {
				String name = "Ms: "+baseName;
				GLog.w(name);
				return name;
			}
		}
	}
	
    private static String getFromBundle(String key){
        String result;
        for (I18NBundle b : bundles){
            result = b.get(key);
            //if it isn't the return string for no key found, return it
            if (result.length() != key.length()+6 || !result.contains(key)){
                return result;
            }
        }
        return null;
    }
```

#### 2.2.1优化版本：
```java
         /** 
          * Resource grabbing methods 
          */ 
  
         public static String get(String key, Object...args){ 
                 return get(null, key, args); 
         } 
  
         public static String get(Object o, String k, Object...args){ 
                 return get(o.getClass(), k, args); 
         } 
  
         public static String get(Class c, String k, Object...args) { 
                 return get(c, k, null, args); 
         } 
  
         private static String get(Class c, String k, String baseName, Object...args){ 
                 String key; 
                 if (c != null){ 
                         key = c.getName(); 
                         key = key.replace("com.shatteredpixel.shatteredpixeldungeon.", ""); 
                         key += "." + k; 
                 } else 
                         key = k; 
  
                 String value = getFromBundle(key.toLowerCase(Locale.CHINESE)); 
                 if (value != null){ 
                         if (args.length > 0) return format(value, args); 
                         else return value; 
                 }  else { 
                         //Use baseName so the missing string is clear what exactly needs replacing. Otherwise, it just says java.lang.Object.[key] 
                         if (baseName == null) { 
                                 baseName = key; 
                                 baseNameX = baseName; 
                         } 
                         //this is so child classes can inherit properties from their parents. 
                         //in cases where text is commonly grabbed as a utility from classes that aren't mean to be instantiated 
                         //(e.g. flavourbuff.dispTurns()) using .class directly is probably smarter to prevent unnecessary recursive calls. 
                         if (c != null && c.getSuperclass() != null){ 
                                 return get(c.getSuperclass(), k, baseName, args); 
                         } else { 
                                 //调试 
                                 if (DeviceCompat.isDebug()){ 
                                         GLog.n("Ms:"+baseName); 
                                 } 
                                 return "Ms:"+baseName; 
                         } 
                 } 
         } 
  
  
         private static String getFromBundle(String key){ 
                 String result; 
                 for (I18NBundle b : bundles){ 
                         result = b.get(key); 
                         //if it isn't the return string for no key found, return it 
                         if (result.length() != key.length()+6 || !result.contains(key)){ 
                                 return result; 
                         } 
                 } 
                 return null; 
         } 
  
```

## 2.制作远程怪物被卡死

出现这种问题通常是因为远程怪物通常有一个施法(zap)的逻辑和贴图类绑定，  
倘若只有该逻辑，对应的贴图类没有该项，则卡死。以下是例子说明：

### 错误示范

```java
//Warlock是矮人术士，继承这个代表有远程

public class RangeRat extends Warlock {
   spriteClass = RangeRatSprites.class;
}

//最终效果:该怪物攻击时游戏卡死，一直无限循环。

分析，因为继承了Warlock,所以也会使用父类的：
WarlockSprites的zap,但RangeRatSprites并没有，  
所以找不到结束点，故而卡死，
注意，即便这里RangeRatSprites有zap，  
可能也会出现CastExpection错误，  
因为RangeRatSprites没有继承WarlockSprites,  
但请不要继承，因为这里只是一个错误示范，  
有关于正确示范，参考下述代码。

```

### 正确示范
```java

public class RangeRat extends Mob implements Callback {

    {
        spriteClass = RangeRatSprites.class;
    } 

	//定义一个静态类，作为一个Object法伤
	public static class DarkBolt{}
	
	protected void zap() {
		spend( TIME_TO_ZAP );

		Invisibility.dispel(this);
		Char enemy = this.enemy;
		if (hit( this, enemy, true )) {
			//TODO would be nice for this to work on ghost/statues too
			if (enemy == Dungeon.hero && Random.Int( 2 ) == 0) {
				Buff.prolong( enemy, Degrade.class, Degrade.DURATION );
				Sample.INSTANCE.play( Assets.Sounds.DEBUFF );
			}
			
			int dmg = Random.NormalIntRange( 12, 18 );
			dmg = Math.round(dmg * AscensionChallenge.statModifier(this));
			enemy.damage( dmg, new DarkBolt() );
			
			if (enemy == Dungeon.hero && !enemy.isAlive()) {
				Badges.validateDeathFromEnemyMagic();
				Dungeon.fail( this );
				GLog.n( Messages.get(this, "bolt_kill") );
			}
		} else {
			enemy.sprite.showStatus( CharSprite.NEUTRAL,  enemy.defenseVerb() );
		}
	}
	
	public void onZapComplete() {
		zap();
		next();
	}
	
    //如果上面没有接口，则这里报错
	@Override
	public void call() {
		next();
	}

}

//RangeRatSprites 需要添加的：

//代表施法等于克隆攻击动画
zap = attack.clone();

//代表施法方法 
    public void zap( int cell ) {

		super.zap( cell );

		MagicMissile.boltFromChar( parent,
				MagicMissile.SHADOW,
				this,
				cell,
				new Callback() {
					@Override
					public void call() {
						((Warlock)ch).onZapComplete();
					}
				} );
		Sample.INSTANCE.play( Assets.Sounds.ZAP );
	}
	
    //代表施法结束后动画变为闲置
	@Override
	public void onComplete( Animation anim ) {
		if (anim == zap) {
			idle();
		}
		super.onComplete( anim );
	}

分析，因为继承了Mob和接口Callback,  
所以这个可以作为远程怪物，  
通过定义一个静态类制作法伤对象，  
则可以绕过常规物理伤害防御算法，  
有关于法伤，后续还会讲的更加彻底。
最后在对应的贴图类将zap和onComlete写出，
一个远程怪就好了。
当然，定义远程怪的范围在doAttack,但我们这里暂时不做讨论。

```