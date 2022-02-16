---
title: COCOS 个人踩坑日记
tags: 
  - 前端
id: '268'
categories:
  - 前端
date: 2020-05-22 16:31:53
---

# 改变子节点的label内容

```
this.turnDuration.getChildByName("number").getComponent(cc.Label).string)
```

# 动态添加精灵（发牌）

\`\`\` <pre><code>/\*\* \* 发牌 个人 \*/ showMyCard(){ let \_this = this; for (let index = 0; index < 17; index++) { let node = new cc.Node("MyCard\_"+index); let sprite = node.addComponent(cc.Sprite); // 点击事件 node.on("mousedown", function(event){ \_this.choosCard(this); event.stopPropagation(); }, node); cc.loader.loadRes("Img/card",cc.SpriteFrame,function(err,spriteFrame){ sprite.spriteFrame = spriteFrame; }); node.parent = this.MyArea; } }, /\*\* \* 选择/取消选择 卡片 \* @param {\*} card \*/ choosCard(card){ if (card.isChoose){ var action = cc.moveTo(0.02, 10, 0); card.runAction(action); card.isChoose = false; }else{ card.isChoose = true; var action = cc.moveTo(0.02, 10, 20); card.runAction(action); } }, \`\`\`