# useRef的源码解读

```
//packages/react-reconciler/src/ReactFiberHooks.new.js
useRef<T>(initialValue: T): {|current: T|} {
  currentHookNameInDev = 'useRef';
  mountHookTypesDev();
  return mountRef(initialValue);
},
```