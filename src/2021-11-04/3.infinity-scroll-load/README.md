# Infinity scroll load

**containerHeight**

父容器高度


**scrollHeight**

滚动元素的总高度

**scrollTop**

滚动上去的高度

**事件监听**

监听 scroll 事件

通过 scrollHeight - scrollTop - containerHeight < thread 判断是否需要加载页面


```
      const scrollTop = this.$refs.listBox.scrollTop
      const scrollHeight = this.$refs.listBox.scrollHeight
      if (scrollHeight - scrollTop - containerHeight < 10) {
        this.$emit('load')
      }

```