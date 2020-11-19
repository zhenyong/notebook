## map 依赖响应粒度

```
// in store
@observable private map = observable.map(); 
...
map.set('name', 'xxx')



const Comp = () => {
  map; // 不 rerender
}

const Comp = () => {
  map.get('name'); // rerender
}
```
