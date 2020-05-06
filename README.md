# 变化侦测

## Object 变化侦测

### 追踪变化,收集依赖

使用 Object.defineProperty

```js
export function defineReactive(data, key, val) {
  let dep = new Dep(); //修改
  Object.defineProperty(data, key, {
    enumerable: true,
    configurable: true,
    get: function () {
      //新增
      dep.depend(); //修改
      return val;
    },
    set: function (newVal) {
      if (val === newVal) {
        return;
      }
      val = newVal;
      dep.notify(); //新增
    },
  });
}
```

### 封装 dep 类

```js
export class Dep {
  constructor() {
    this.subs = [];
  }
  addSub(sub) {
    this.subs.push(sub);
  }

  removeSub(sub) {
    remove(this.subs, sub);
  }

  depend() {
    let f = this.subs.filter((v) => {
      return v.id == window.target.id;
    });
    if (window.target && f.length == 0) {
      this.addSub(window.target);
    }
  }

  notify(val) {
    const subs = this.subs.slice();
    for (let i = 0, l = subs.length; i < l; i++) {
      subs[i].update(val);
    }
  }
}

function remove(arr, item) {
  if (arr.length) {
    const index = arr.indexOf(item);
    if (index > -1) {
      return arr.splice(index, 1);
    }
  }
}
```

### 依赖是Watcher
```js
export class Watcer {
  constructor(expOrFn, cb) {
    this.getter = parsePath(expOrFn);
    this.cb = cb;
    this.value = this.get();
    this.id = this.getId();
  }
  get() {
    window.target = this;
    let value = this.getter.call(window, window);
    window.target = undefined;
    return value;
  }
  update() {
    const oldValue = this.value;
    this.value = this.get();
    this.cb(this.value, oldValue);
  }
  getId() {
    return dt.uuid();
  }
}
const bailRE = /[^\w.$]/;
export function parsePath(path) {
  if (bailRE.test(path)) {
    return;
  }
  const segments = path.split(".");
  return function (obj) {
    for (let i = 0; i < segments.length; i++) {
      if (!obj) return;
      obj = obj[segments[i]];
    }
    return obj;
  };
}
```