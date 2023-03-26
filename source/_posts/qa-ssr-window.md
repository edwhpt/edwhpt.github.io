---
title: 【Q&A】SSR 服务端渲染获取不到 Window 对象
date: 2023-03-26 23:33:54
categories: Q&A
tags:
- ssr
- nuxt.js
- next.js
---

### 获取不到Window对象，怎么解决？

1. 在客户端代码中对 window 对象进行特殊处理，使其在服务端渲染时不会被引用。

   ```js
   if (typeof window !== 'undefined') {
     // Your code that uses the window object
   }
   ```

2. 对代码进行重构，避免对 window 对象的引用。

<!-- more -->

### Nuxt.js中获取window对象

```vue
<script>
export default {
  beforeCreate() {
    if (process.browser) {
      if ((window.navigator.userAgent.match(/(iPhone|iPod|Android|ios|iOS|iPad|Backerry|WebOS|Symbian|Windows Phone|Phone)/i))) {
        window.location = '/mobile'
      }
    }
  },
  created(){
    if (process.browser) { 
      console.log(window)
    }
  }
}
</script>
```



### Next.js中获取window对象

使用useEffect钩子

```js
function useLastSeen() {
    const [lastSeen, setLastSeen] = useState(null);
    const retrieved = useRef(false); //To get around strict mode running the hook twice
    useEffect(() => {
        if (retrieved.current) return;
        retrieved.current = true;
        setLastSeen(updateLastSeen());
    }, []);

    return lastSeen;
}
```

判断window是否存在

```javascript
const isBrowser = () => typeof window !== 'undefined'; //The approach recommended by Next.js

function WindowPage() {
    const [lastClick, setLastClick] = useState('');
    
    if (isBrowser()) { //Only add the event listener client-side
        window.addEventListener('click', (e) =>
            setLastClick(`${e.pageX}, ${e.pageY}`)
        );
    }

    return (
        <div className="m-auto rounded bg-violet-600 p-10 font-bold text-white shadow">
            Click at: {lastClick}
        </div>
    );
}
```



