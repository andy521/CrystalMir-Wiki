### CrystalM2Script

CrystalM2的脚本系统

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
