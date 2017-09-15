---
title: mongodb两阶段提交的python实现
date: 2017-08-05 14:50:37
tags:
    - mongodb
    - Python
---

## 基本介绍
* 参考官方文档 https://docs.mongodb.com/manual/tutorial/perform-two-phase-commits
* 整个操作过程实际是一个两步提交的过程，涉及到三张表，包括目标表destination和原始的表source，还有一张transaction用来记录整个过程的状态
* 状态主要有Initial、Pending、Applied、Done，这四个状态是操作正常时的状态，异常时还会有Canceling和Canceled状态

## 正常操作
* 以支付业务举例，一般的支付流程是，用户购买东西，生成一个待支付的订单，然后支付这个订单

* 生成带有初始化状态的transaction记录和未知付的订单
``` python
db['orders'].insert_one({'state': 'UnPaid'})
db['transaction'].insert_one({'state': 'Initial'})
```
* Initial到Pending
``` python
db['transaction'].update_one(
    {'_id': transaction_id, 'state': 'Initial'}, 
    {'$set': {'state': 'Pending'}})

db['balance'].update_one(
    {'_id': banlance_id}, 
    {'$inc': {'balance': -amount}, '$push': {'pendingTransactions': transaction_id}})

db['orders'].update_one(
    {'_id': order_id}, 
    {'$set': {'state': 'Paid', '$push': {'pendingTransactions': transaction_id})
```

* Pending到Applied
``` python
db['transactoin'].update_one(
    {'_id': transaction_id, 'state': 'Pending'}, 
    {'$set': {'state': 'Applied'}})

db['balance'].update_one(
    {'_id': balance_id},
    {'$pull': {'pendingTransactions': transction_id}})

db['orders'].update_one(
    {'_id': order_id},
    {'$pull': {'pendingTransactions': transaction_id}})
```

* Applied到Done
``` python
db['transaction'].update_one(
    {'_id': transaction_id, 'state': 'Applied'},
    {'$set': {'state': 'Done'}})
```

## 异常处理
* 对于四个正常的状态，应该保证在任何一个状态出现问题，都能抛出错误，在下次操作时，能正确地按照异常状态执行下去
* 异常状态为Initial，直接将生成的transaction记录清楚，然后按照正常流程进行即可
``` python
db['transaction'].delete_one({'_id': transaction_id})
```
* 异常状态为Pending，需要回滚操作，将已经扣除的余额和修改的订单状态还原
``` python
db['transaction'].update_one(
    {'_id': transaction_id, 'state': 'Pending'},
    {'$set': {'state': 'Canceling'}})

db['orders'].update_one(
    {'_id': order_id},
    {'$pull': {'pendingTransactions': transaction_id}}
)

db['balance'].update_one(
    {'_id': balance_id},
    {'$inc': {'balance': amount}, '$pull': {'pendingTransactions': transaction_id}})

db['transaction'].update_one(
    {'_id': transaction_id, 'state': 'Canceling'},
    {'$set': {'state': 'Canceled'}})
```

* 异常状态为Applied，继续往下执行，不需要回滚操作
``` python
db['balance'].update_one(
    {'_id': balance_id},
    {'$pull': {'pendingTransactions': transction_id}})

db['orders'].update_one(
    {'_id': order_id},
    {'$pull': {'pendingTransactions': transaction_id}})

db['transaction'].update_one(
    {'_id': transaction_id, 'state': 'Applied'},
    {'$set': {'state': 'Done'}})
```

* 异常状态为Canceling，表示回滚操作进行了一半失败了，只需要将未生效的操作执行下去即可
``` python
db['orders'].update_one(
    {'_id': order_id},
    {'$pull': {'pendingTransactions': transaction_id}}
)

db['balance'].update_one(
    {'_id': balance_id},
    {'$inc': {'balance': amount}, '$pull': {'pendingTransactions': transaction_id}})

db['transaction'].update_one(
    {'_id': transaction_id, 'state': 'Canceling'},
    {'$set': {'state': 'Canceled'}})
```

## 总结
* 可以看到，mongodb的两阶段提交比较复杂，需要采取很多的保护性操作，在每个操作失效后，都可以在下次操作时进行弥补
* 在具体写代码的时候，需要注意pull操作在目标为空的时候是会抛出错误的，需要对这种情况作单独的处理
* 整个支付过程之前，还需要对余额不足的情况作判断，如果余额不足就不需要进行事务操作
* 为了保证提交的成功，还需要单独有一个维护程序来保证异常处理的运行
