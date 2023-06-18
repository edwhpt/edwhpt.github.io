---
title: 【Vue】插槽（v-slot）
date: 2023-03-26 23:15:16
categories: Vue
tags:
- vue
- v-slot
---

### slot (父组件 在子组件`<slot> </slot>`处插入内容)

Vue 实现了一套内容分发的 API，将`<slot>`元素作为承载分发内容的出口，这是vue文档上的说明。具体来说，slot就是可以让你在组件内添加内容的‘空间’。

<!-- more -->

举个例子：

```vue
<!-- 子组件 ： (假设名为：ebutton) -->
<template>
  <div class= 'button'>
      <button>  </button>
  </div>
</template>

<!-- 父组件：（引用子组件 ebutton）-->
<template>
  <div class= 'app'>
     <ebutton> </ebutton>
  </div>
</template>
```
我们知道，如果直接想要在父组件中的`<ebutton></ebutton>` 中添加内容，是不会在页面上渲染的。那么我们如何使添加的内容能够显示呢？在子组件内添加slot 即可。

```vue
<!-- 子组件 ： (假设名为：ebutton) -->
<template>
  <div class= 'button'>
      <button></button>
      <slot></slot>       //slot 可以放在任意位置。（这个位置就是父组件添加内容的显示位置）
  </div> 
</template>
```

子组件可以在任意位置添加slot , 这个位置就是父组件添加内容的显示位置。

### 编译作用域 （父组件 在子组件`<slot> </slot>`处插入 data）

上面我们了解了，slot 其实就是能够让我们在父组件中添加内容到子组件的‘空间’。我们可以添加父组件内任意的data值，比如这样：

```vue
<!-- 父组件：（引用子组件 ebutton）-->
<template>
  <div class= 'app'>
     <ebutton> {{ parent }}</ebutton>
  </div>
</template>

<script>
  new Vue({
    el:'.app',
    data:{
      parent:'父组件'
    }
  })
</script>

```

使用数据的语法完全没有变，但是，我们能否直接使用子组件内的数据呢？显然不行！！

```vue
// 子组件 ： (假设名为：ebutton)
<template>
  <div class= 'button'>
      <button> </button>
       <slot></slot>
  </div>
</template>

<script>
  new Vue({
    el:'.button',
    data:{
      child:'子组件'
    }
  })
</script>

<!-- 父组件：（引用子组件 ebutton）-->
<template>
  <div class= 'app'>
     <ebutton> {{ child }}</ebutton>
  </div>
</template>
```

直接传入子组件内的数据是不可以的。因为：父级模板里的所有内容都是在父级作用域中编译的；子模板里的所有内容都是在子作用域中编译的。

### 后备内容 (子组件`<slot> </slot>`设置默认值)

所谓的后背内容，其实就是slot的默认值，有时我没有在父组件内添加内容，那么 slot就会显示默认值，如：

```vue
<!-- 子组件 ： (假设名为：ebutton) -->
<template>
  <div class= 'button'>
      <button>  </button>
      <slot> 这就是默认值 </slot>
  </div>
</template>
```

### 具名插槽 （子组件 多个`<slot ></slot>` `<slot></slot>` 对应插入内容）

有时候，也许子组件内的slot不止一个，那么我们如何在父组件中，精确的在想要的位置，插入对应的内容呢？ 给插槽命一个名即可，即添加name属性。

```vue
<!-- 子组件 ： (假设名为：ebutton) -->
<template>
  <div class= 'button'>
      <button>  </button>
      <slot name='one'> 这就是默认值1</slot>
      <slot name='two'> 这就是默认值2 </slot>
      <slot name='three'> 这就是默认值3 </slot>
  </div>
</template>
```

父组件通过v-slot : name 的方式添加内容：

```vue
<!-- 父组件：（引用子组件 ebutton）-->
<template>
  <div class= 'app'>
     <ebutton> 
        <template v-slot:one> 这是插入到one插槽的内容 </template>
        <template v-slot:two> 这是插入到two插槽的内容 </template>
        <template v-slot:three> 这是插入到three插槽的内容 </template>
     </ebutton>
  </div>
</template>
```

当然 vue 为了方便，书写 v-slot:one 的形式时，可以简写为 #one

### 作用域插槽 ( 父组件 在子组件 `<slot> </slot>` 处使用子组件 data)

通过slot 我们可以在父组件为子组件添加内容，通过给slot命名的方式，我们可以添加不止一个位置的内容。但是我们添加的数据都是父组件内的。上面我们说过不能直接使用子组件内的数据，但是我们是否有其他的方法，让我们能够使用子组件的数据呢？ 其实我们也可以使用v-slot的方式：

```vue
<!-- 子组件 ： (假设名为：ebutton) -->
<template>
  <div class= 'button'>
      <button>  </button>
      <slot name='one' :value1='child1'> 这就是默认值1</slot>    //绑定child1的数据
      <slot :value2='child2'> 这就是默认值2 </slot>  //绑定child2的数据，这里我没有命名slot
  </div>           
</template>

<script>
  new Vue({
    el:'.button',
    data:{
      child1:'数据1',
      child2:'数据2'
    }
  })
</script>

<!-- 父组件：（引用子组件 ebutton）-->
<template>
  <div class= 'app'>
     <ebutton> 

        <!-- 通过v-slot的语法 将插槽 one 的值赋值给slotonevalue -->
        <template v-slot:one='slotonevalue'>  
           {{ slotonevalue.value1 }}
        </template>

        <!-- 同上，由于子组件没有给slot命名，默认值就为default -->
        <template v-slot:default='slottwovalue'> 
           {{ slottwovalue.value2 }}
        </template>

     </ebutton>
  </div>
</template>
```

总结来说就是：

- 首先在子组件的slot上动态绑定一个值( :key='value')
- 然后在父组件通过v-slot : name = ‘values ’的方式将这个值赋值给 values
- 最后通过{{ values.key }}的方式获取数据
