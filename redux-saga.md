#redux-saga
## 先从最初的论文说起
### long-lived transaction
论文地址在[这里](http://www.cs.cornell.edu/andru/cs711/2002fa/reading/sagas.pdf)

这个我翻译为长时数据操作。根据[wik的解释](https://en.wikipedia.org/wiki/Long-lived_transaction)，一个长时数据操作是：A long-lived transaction can be thought of as a sequence of database transactions grouped to achieve a single atomic result.

并且：A common example is a multi-step sequence of requests and responses of an interaction with a user through a web client.

相信有了上面两个解释和例子就很容易想通了为什么redux-saga用于处理在redux上处理有副作用的异步操作了。





