### CrystalM2Script

CrystalM2的脚本系统
## 触发与特效
在技能开始和命中时, 脚本中播放额外的特效动画

![image](https://github.com/CrystalMir2/CrystalMir-Wiki/assets/143333779/234af8b9-777e-4848-a3ed-a973136edcce)


### 技能触发

#### 远程技能的开始和命中
```javascript
function QFunction$MagicBegin(magic) {

    //自身特效
    //string libraryName, int index,int imgCount, int interval,int replayCount,int delay
    ctx.playSelfEffect("Magic3",1880,10,100,0,0)//lib名必须正确.大小写敏感
    var say = `MagicBegin Spell:${magic.spell}`;
    ctx.sendMsg(6,say);
    return false;
    // return true;//返回ture.则会中断施法
}
function QFunction$MagicAttack(magic,target) {
    //目标命中特效
    //uint targetObjectID,string libraryName, int index,int imgCount, int interval,int replayCount,int delay
    if (target!=null)//判断是否命中目标
        ctx.playObjectEffect(target.ObjectID,"Monster/459",100,11,100,0,0)//lib名必须正确.大小写敏感

    var say = `MagicAttack:Spell:${magic.spell},target:${target.Info.Name},ObjectID:${target.ObjectID}`;
    ctx.sendMsg(6,say);
    return "";
}

```




## 物品
针对背包,仓库,装备栏等位置的物品进行脚本操作,如查询,删除,增加,修改等


```
function GM$GM_Test$Main() {
    var say = `Hello ,{${ctx.name}|249}(lv.${ctx.Level}) + Welcome To {${ServerName}|253} ,\\`;
    say+=`<检查背包/@检查物品(1,乌木剑,11)>|<检查装备/@检查物品(2,乌木剑,1)>|<检查仓库/@检查物品(5,乌木剑,2)>\\  \\`
    say+=`<是否佩戴屠龙+祖玛套装/@批量检查装备()>:任意2件激活套装Buffer\\  \\`
    return say;
}
```
![image](https://github.com/CrystalMir2/CrystalMir-Wiki/assets/143333779/623e155e-d029-4ea8-977f-ae137734a0a9)


#### 批量一键回收

批量模糊查找并回收背包中的物品,返回回收的数量,给与`Credit`

```javaScript
function 一键回收(){
    //批量模糊匹配改名有效
    let takeItemName = "*年雪霜|*(中量)|*太阳水|*(女)|力量*|骑士*|绿色*|乌木剑";
    var takeCount = ctx.take(takeItemName,65535);

    // //精准匹配,改名有效
    // var takeCount = asInt(ctx.take("金条",12));
    if (takeCount>0)ctx.giveCredit(takeCount);

    var list = ctx.getItemList(ItemPostionType.Inventory);

    return `回收数量:${takeItemName}x${asInt(takeCount)} \\ 背包物品:\\` + showItemList(list);
}

```

#### 检查物品数量
检查,仓库,背包中是否有物品,用于NPC任务对话,套装穿戴检测等

```javascript

`<检查背包/@检查物品(1,乌木剑,11)>|<检查装备/@检查物品(2,乌木剑,1)>|<检查仓库/@检查物品(5,乌木剑,2)>\\  \\`
`<是否佩戴屠龙+祖玛套装/@批量检查装备()>:任意2件激活套装Buffer\\  \\`


function 检查物品(pos,itemName,count){
    var isExist = ctx.checkItemPostionType(itemName,asInt(count),asInt(pos))
    return `检查位置:${pos},物品:${itemName}x${count} 结果:${isExist}`+appendBackBtn();;
}
function 批量检查装备(){
    if (ctx.checkItemW("屠龙",1)
        &&ctx.checkItemW("力量*|骑士*|绿色*",2)){
        return `祖玛屠龙套装激活:{已激活|250}`+appendBackBtn();
    }
    return `祖玛屠龙套装激活:{未激活|249}`+appendBackBtn();
}

function appendBackBtn(){
    return `\\  \\  <返回/@GM$GM_Test$Main>|<退出/@exit>`;
}
```

#### 随身仓库
打开客户端的随身仓库界面, 并遍历仓库中的物品进行展示
```javascript
function 随身仓库(){
    ctx.openClientDialog(1);
    var list = ctx.getItemList(ItemPostionType.Storage);
    if(!list)return "";
    return "仓库物品:\\" + showItemList(list);
}

function showItemList(list) {
    var itemStr="";
    for (var i = 0; i < list.length; i++) {
        var item = list[i];
        if (!item) continue;
        itemStr += `${i}:${item.FriendlyName},type:${item.Info.Type},shape:${item.Info.Shape},image:${item.Info.Image}  \\`;
    }
    return itemStr;
}
```

#### 修复持久

```JavaScript
function 全身修复(){
    ctx.repairAll()
    return `<返回/@GM$GM_Test$Main>`;
}
```

#### 物品使用触发

客户都安使用背包中的物品触发脚本
根据所使用的物品属性,如名字:`name`,别名:`friendlyName`,类型:`shape`),可动态的自定义道具的功能

```javascript
function QFunction$UseItem() {
    return 双击触发$双击触发$Main(arguments[0],arguments[1],arguments[2],arguments[3]);
}

function 双击触发$双击触发$Main(id,shape,name,friendlyName) {
    console.log("[双击触发] shape name,friendlyName:",shape,name,friendlyName);
    var say = '';
    ctx.sendMsg(1,String.format('[双击触发]id {0},shape {1}, name {2},friendlyName {3} ',id,shape,name,friendlyName));

    switch (shape){
        case 1:
            say+= 双击触发$测试卷轴();
            break
        case 3:
            say+= '您的等级{'+ctx.Level+'#Green},\r\n';
            say+= 双击触发$双击触发$传送按钮("r001",99,99,);
            say+=`<传送/@QFunction$双击触发$双击触发$传送(r001,99,99,)>桃源`;
            say+=`<传送/@QFunction$双击触发$双击触发$传送(N0,329,264,)>比奇`;
            say+=`<传送/@QFunction$双击触发$双击触发$传送(3,333,333,)>盟重`;
            say+=`<传送/@QFunction$双击触发$双击触发$传送(r001,99,99,)>GM`;
            break
        case 4://神秘卷轴
            ctx.remob('白野猪',3,7);
            ctx.giveGold(1000000);
            ctx.give('屠龙',1);
            ctx.giveCredit(100);
            ctx.givePearls(200);
            ctx.giveSkill("HalfMoon",3);
            ctx.repairAll();
            break
        case 14://金条
            ctx.giveGold(1000000);
            break
        case 15://金砖
            ctx.giveGold(5000000);
            break
        case 16://金盒
            ctx.giveGold(10000000);
            break
        case 17://修复神锤
            ctx.repairAll();
            break;
        case 18://便捷服务
            var server = '{便捷服务#red}:\r\n'
            say+= '<寄售/@QFunction$Market()> ';
            say+= '<买卖/@QFunction$SellBuy()> ';
            say+= '<锻造/@QFunction$Refine()> ';
            say+= '<仓库/@QFunction$showStorage()> ';
            return say;
        case 10000://自定义
            ctx.remob(friendlyName.split("-")[1],14400);
            break
        case 10001://自定义
            ctx.give(name.split("-")[1],1);
            break
    }
    return say;
}


```

## 数据存储

### 类型说明
主要分为3中类型, 

- 临时数据,下线清零
- 每日数据,0点清零
- 持久数据,永久保存

  
针对数据的归属对象,分为2种

- 全局,服务器持有,全局贡献
- 玩家,各自独立,仅玩家自己能访问


通常用

`G`表示全局变量,

`J`表示每日变量,0点清零

支持多种数据类型,Int,String,JS数组,JS对象

键值对的key支持中文,但不建议,数据库会以Unicode存储,不易读且占用空间


### 用法示例

```javascript
    ctx.sendMsg(6,`上次登陆时间:${ctx.load("LastLoginTime")}`)
    ctx.set("LastLoginTime",Time.Now());
    ctx.save("LastLoginTime",Time.Now());
    ctx.sendMsg(6,`本次登陆时间:${ctx.load("LastLoginTime")}`)
```

```javascript
function 全局变量(){
    let key = "全服玩家点击NPC次数";
    var temp = ctx.loadG(key);
    var totalClickNpc = temp?asInt(temp):0;
    totalClickNpc+=1;
    ctx.saveG(key,totalClickNpc);
    ctx.sendMsg(6, `${key}:` + totalClickNpc);
    return ``;
}
function 临时变量(){
    let key = "本次登录点击NPC次数";
    var temp = ctx.get(key);
    var totalClickNpc = temp?asInt(temp):0;
    totalClickNpc+=1;
    ctx.set(key,totalClickNpc);
    ctx.sendMsg(6, `${key}:` + totalClickNpc);
    rreturn ``;
}
function 玩家变量(){
    let key = "本玩家点击NPC次数";
    var temp = ctx.load(key);
    var totalClickNpc = temp?asInt(temp):0;
    totalClickNpc+=1;
    ctx.save(key,totalClickNpc);
    ctx.sendMsg(6, `${key}:` + totalClickNpc);
    return ``;
}
function 每日变量(){
    let key = "今日点击NPC次数";
    var temp = ctx.loadJ(key);
    var totalClickNpc = temp?asInt(temp):0;
    totalClickNpc+=1;
    ctx.saveJ(key,totalClickNpc);
    ctx.sendMsg(6, `${key}:` + totalClickNpc);
    return ``;
}
```
