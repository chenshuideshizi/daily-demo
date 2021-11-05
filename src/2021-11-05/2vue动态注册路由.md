### 如何动态注册路由？

```
├── package.json
└── src
    ├── App.vue
    ├── main.js
    ├── router.js
    └── views
        ├── About.vue
        ├── Home.vue
        └── modifiers
            ├── capture.vue
            ├── once.vue
            ├── order.vue
            ├── passive.vue
            ├── prevent.vue
            ├── self.vue
            └── stop.vue
            └── ...
```

 require.context实现动态注册路由
```
 const registerRoutes = () => {
  const contextInfo = require.context('./views', true, /.vue$/)
  const routes = contextInfo.keys().map((filePath) => {
    // filePath 形如 ./Home.vue、./modifiers/capture.vue
    // path我们希望是/home、/modifiers/capture
    // 所以需要把开头的./和.vue都替换为空
    const path = filePath.toLowerCase().replace(/^\.|\.vue/g, '')
    // name的话将/home、/modifiers/capture转成小驼峰即可
    // 把开头的/先替换掉，再把第一个/后的单词变成大写就可以了
    const name = path.replace(/^\//, '').replace(/\/(\w)/, ($0, $1) => $1.toUpperCase())
    // 通过require去读取.vue文件内容
    const component = require(`./views${filePath.replace(/^\./, '')}`).default

    return {
      path,
      name,
      component
    }
  })

  return routes
}
```

作者：前端胖头鱼
链接：https://juejin.cn/post/7026867875990208543
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。