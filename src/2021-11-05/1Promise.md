
```
        // demo1
        // var promise1 = Promise.resolve('我');
        // var promise2 = new Promise(function (resolve, reject) {
        //     setTimeout(reject, 1000, '不是');
        // });
        // var promise3 = new Promise(function (resolve, reject) {
        //     setTimeout(resolve, 2000, '舔狗');
        // });
        // Promise.allSettled([promise1, promise2, promise3])
        //     .then(function (values) {
        //         console.log(values);
        //     })
        //     .catch((err) => console.log(err));

        // demo2
        // var promise2 = new Promise(function (resolve, reject) {
        //     setTimeout(resolve, 1000, '班花');
        // });
        // var promise3 = new Promise(function (resolve, reject) {
        //     setTimeout(resolve, 2000, '舔狗');
        // });
        // Promise.all([ promise2, promise3])
        //     .then(function (values) {
        //         console.log(values);
        //     })

        // demo3
        // Promise.resolve(1)
        //     .then(2) // 注意这里
        //     .then(Promise.resolve(3))
        //     .then(console.log);   // 输出1   这一步等同 .then((res)=>{console.log(res)})

        // demo4
        // Promise.resolve(1)
        //   .then(function(){return 2})
        //   .then(Promise.resolve(3))
        //   .then(console.log)  // 输出2

        // demo5实现 promise
        const PENDING = 'pending';
    const FULFILLED = 'fulfilled';
    const REJECTED = 'rejected';
     class myPromise {
            constructor(executor) {
                let self = this;
                self.status = PENDING;
                self.value = undefined;
                self.reason = undefined;
                self.onResolvedCallbacks = [];
                self.onRejectedCallbacks = [];

                let resolve = (value) => {
                    if (self.status === PENDING) {
                        self.status = FULFILLED;
                        self.value = value;
                        self.onResolvedCallbacks.forEach((fn) => fn());
                    }
                };

                let reject = (reason) => {
                    if (self.status === PENDING) {
                        self.status = REJECTED;
                        self.reason = reason;
                        self.onRejectedCallbacks.forEach((fn) => fn());
                    }
                };
                try {
                    executor(resolve, reject);
                } catch {
                    reject(err);
                }
            }

            then(onFulfilled, onRejected) {
                //处理then里面不是回调函数情况
                //Promise/A+ 2.2.1 / Promise/A+ 2.2.5 / Promise/A+ 2.2.7.3 / Promise/A+ 2.2.7.4
                onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : (v) => v;
                onRejected =
                    typeof onRejected === 'function'
                        ? onRejected
                        : (err) => {
                              throw err;
                          };
                let self = this;
                return new myPromise((resolve, reject) => {
                    if (self.status === 'fulfilled') {
                        setTimeout(() => {
                            try {
                                let x = onFulfilled(self.value);
                                x instanceof myPromise ? x.then(resolve, reject) : resolve(x);
                            } catch (err) {
                                reject(err);
                            }
                        }, 0);
                    }
                    if (self.status === 'rejected') {
                        setTimeout(() => {
                            try {
                                let x = onRejected(self.reason);
                                x instanceof myPromise ? x.then(resolve, reject) : resolve(x);
                            } catch (err) {
                                reject(err);
                            }
                        }, 0);
                    }
                    if (self.status === 'pending') {
                        self.onResolvedCallbacks.push(() => {
                            setTimeout(() => {
                                let x = onFulfilled(self.value);
                                x instanceof myPromise ? x.then(resolve, reject) : resolve(x);
                            }, 0);
                        });
                        self.onRejectedCallbacks.push(() => {
                            setTimeout(() => {
                                let x = onRejected(self.reason);
                                x instanceof myPromise ? x.then(resolve, reject) : resolve(x);
                            }, 0);
                        });
                    }
                });
            }
            static all(values) {
            if (!Array.isArray(values)) {
              return new TypeError("请输入一个数组")
            }
            return new Promise((resolve, reject) => {
              let resultArr = [];
              let len = 0;
              const dealFn = (value, index) => {
                resultArr[index] = value;
                if (++len === values.length) {
                    resolve(resultArr)
                }
              }
              for (let i = 0; i < values.length; i++) {
                let value = values[i];
                //判断value是否还是个promise
                if (value && typeof value.then === 'function') {
                  value.then((value) => {
                    dealFn(value, i);
                  }, reject);
                } else {
                    dealFn(value, i);
                }
              }
            });
          }
        }

```

// 作者：舔狗的泪
// 链接：https://juejin.cn/post/7026896760152784910
// 来源：稀土掘金
// 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

