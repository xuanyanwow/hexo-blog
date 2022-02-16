---
title: php一份代码分层规范
tags: []
id: '314'
categories:
  - - PHP
comments: false
date: 2021-09-06 16:24:22
---

## 控制器层

*   事务开关
*   调用Service Validata进行数据验证
*   调用Process Service Do
*   *   Other Service Validata
*   *   Other Service Do

```php
public function createOrder(Request $request)
    {
        DB::beginTransaction();
        try {
            $this->orderService->validatorCreateOrder($request);
            $goods = $this->orderService->validatorGoods($request);
            // 设置商品
            $this->orderProcessService->setGoods($goods);
            // 优惠码
            $coupon = $this->orderService->validatorCoupon($request);
            // 设置优惠码
            $this->orderProcessService->setCoupon($coupon);
            $otherIpt = $this->orderService->validatorChargeInput($goods, $request);
            $this->orderProcessService->setOtherIpt($otherIpt);
            // 数量
            $this->orderProcessService->setBuyAmount($request->input('by_amount'));
            // 支付方式
            $this->orderProcessService->setPayID($request->input('payway'));
            // 下单邮箱
            $this->orderProcessService->setEmail($request->input('email'));
            // ip地址
            $this->orderProcessService->setBuyIP($request->getClientIp());
            // 查询密码
            $this->orderProcessService->setSearchPwd($request->input('search_pwd', ''));
            // 创建订单
            $order = $this->orderProcessService->createOrder();
            DB::commit();
            // 设置订单cookie
            $this->queueCookie($order->order_sn);
            return redirect(url('/bill', ['orderSN' => $order->order_sn]));
        } catch (RuleValidationException $exception) {
            DB::rollBack();
            return $this->err($exception->getMessage());
        }
    }
```

## Service层

*   Validata —— 数据有效性验证
*   Do —— 查询数据的封装 调用Model
*   支持嵌套事务的框架 Service可做事务开关

```php
    public function validatorCoupon(Request $request):? Coupon
    {
        // 如果提交了优惠码
        if ($request->filled('coupon_code')) {
            // 查询优惠码是否存在
            $coupon = $this->couponService->withHasGoods($request->input('coupon_code'), $request->input('gid'));
            // 此商品没有这个优惠码
            if (empty($coupon)) {
                throw new RuleValidationException(__('dujiaoka.prompt.coupon_does_not_exist'));
            }
            // 剩余次数不足
            if ($coupon->ret <= 0) {
                throw new RuleValidationException(__('dujiaoka.prompt.coupon_lack_of_available_opportunities'));
            }
            return $coupon;
        }
        return null;
    }
```

```php
    public function withGoodsByAmountAndStatusUnsold(int $goodsID, int $byAmount)
    {
        $carmis = Carmis::query()
            ->where('goods_id', $goodsID)
            ->where('status', Carmis::STATUS_UNSOLD)
            ->take($byAmount)
            ->get();
        return $carmis ? $carmis->toArray() : null;
    }
```

## Process Service层

复杂的逻辑，有进度的 ，比如功能的审批、比如订单的创建这种影响比较大的链条式请求

*   调用Service
*   Save Data To Do Process —— 保存数据 进行处理
*   支持嵌套事务的框架 Process Service可做事务开关

```php
            // ip地址
            $this->orderProcessService->setBuyIP($request->getClientIp());
            // 查询密码
            $this->orderProcessService->setSearchPwd($request->input('search_pwd', ''));
            // 创建订单
            $order = $this->orderProcessService->createOrder();
```

## Job层

任务层，如发送邮件任务，发送接口请求任务（ERP等），入队任务

## Model层

对数据的`自动化`进行设置，`不进行`封装 如 getByIds等，封装到逻辑中 - 时间戳自动更新 - 格式自动转化 - 附加字段 - 关联关系

## 实例举例

酒吧系统： 下单 - 控制器 - - 事务 - - 调用GoodsService验证 店铺、商品有效性（商品id有效性、是否属于当前店铺、是否下架等） - - 调用OrderProcessService，进行订单进程性处理，setStaff员工信息，setGoods下单商品信息，setTable信息等 - - OrderProcessService 调用 OrderService 进行订单插入，调用GoodsService 进行库存减少，调用TableService进行统计信息更新等 - - - 调用Job层，进行ERP信息推送对接 - - - 调用Job层，进行小票任务推送 - - 请求结束，事务、响应