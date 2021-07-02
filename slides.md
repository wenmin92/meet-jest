---
layout: 'cover'
background: '/bg.jpg'
download: true
---
# 初识 Jest
<p class="mx-56 author">———— 赵昌柱</p>

---

# 背景介绍
- TS, Flow, EsLint, StyleLint 等前端工具都可以帮助减少 bug 产生. 
- 引入自动化测试可以更好地降低 bug 产生的可能性.
- 流行开源库都使用自动化测试. 多人协作时可以保证代码不会被意外修改.
- 在迭代旧项目时, 如果有自动化测试, 在修bug或增加新功能时, 可以保证不会影响之前的功能.
- 测试主要分为单元测试, 集成测试, end-to-end测试(端到端测试).

---

# 目录
- Jest 环境搭建及配置
- 基础 API
- 异步测试
- Mock
- Dom 测试
- 快照

---

# 测试原理 1
### 待测代码
```javascript {6-9}
/**
 * 格式化数字
 * @param {string|number} numStr 传入小于2位数的数值或字符串表示的数值
 * @returns 不足2位数时, 前面补0
 */
function formatNumStr(numStr) {
    numStr = typeof numStr === 'string' ? numStr : numStr.toString();
    return numStr.length === 2 ? numStr : `0${numStr}`;
}

module.exports = formatNumStr;

```

---

# 测试原理 2
### 手工测试
```javascript {4-8,10-14}
let expected = '';
let result = '';

expected = '04';
result = formatNumStr(4);
if (result !== expected) {
    throw new Error(`formatNumStr(4) 应该等于 ${expected}, 但实际结果是 ${result}`);
}

expected = '12';
result = formatNumStr(12);
if (result !== expected) {
    throw new Error(`formatNumStr(12) 应该等于 ${expected}, 但实际结果是 ${result}`);
}

console.log("pass");
```
---

# 测试原理 3
### 手工测试 (封装测试函数)
```javascript {17-22}
// expect(real).toBe(expected);
function expect(actual) {
    return {
        toBe(expected) {
            if (actual !== expected) { throw new Error("预期是 ${expected}, 实际结果是 ${result}"); }
        }
    }
}
function test(desc, fn) {
    try {
        fn();
        console.log(`${desc} pass`);
    } catch (error) {
        console.log(`${desc} failed, ${error.message}`);
    }
}
test('formatNumStr(4)', () => {
    expect(formatNumStr(4)).toBe('04');
});
test('formatNumStr(12)', () => {
    expect(formatNumStr(12)).toBe('12');
});
```

---

# 测试框架对比与选择
<img class="h-5/6" src="test-framework-compare.png">

---

# Jest 优点
官网: [https://jestjs.io/](https://jestjs.io/)
- 速度快 (仅执行改变的代码)
- API简单
- 易配置 (开箱即用)
- 隔离性好
- 监控模式
- IDE整合
- Snapshot
- 多项目并行
- 覆盖率
- Mock 丰富

---

# Jest 环境搭建
1. 安装 Node.js
2. `npm install --save-dev jest`
3. 导出待测代码, 在测试文件中引入 (如果不编译, 直接在浏览器中使用的代码, 通过 `try...catch...` 导出)
4. 添加 script, `test: "jest"`

<br>
<br>

### 测试代码与之前相同
```javascript
const formatNumStr = require('./math');

test('formatNumStr(4)', () => {
    expect(formatNumStr(4)).toBe('04');
});
test('formatNumStr(12)', () => {
    expect(formatNumStr(12)).toBe('12');
});
```

---

# Jest 配置
1. 可以零配置运行
2. 生成配置文件 `npx jest --init`
3. watch 模式, `--watch` or `--watchAll`
4. 生成测试覆盖率报告, `--coverage` 和 `coverageDirectory`
5. 👉 配置 [Ref](https://jestjs.io/docs/configuration)
6. 支持 ESM  
    ### 安装 Babel
    ```bash
    yarn add --dev babel-jest @babel/core @babel/preset-env
    ```
    ### 配置 Babel
    ```javascript
    // babel.config.js
    module.exports = {
        presets: [['@babel/preset-env', {targets: {node: 'current'}}]],
    };
    ```

---

# 匹配器

<div class="flex gap-4">
<div class="flex-1">

👉 匹配器 [Guide](https://jestjs.io/docs/using-matchers), [Ref](https://jestjs.io/docs/expect)


- 通用
  - `.toBe(value)`
  - `.toEqual(value)`
  - `.not`
- 真假
  - `.toBeNull()`
  - `.toBeUndefined()`
  - `.toBeDefined()`
  - `.toBeTruthy()`
  - `.toBeFalsy()`

</div>
<div class="flex-1">

- 数字
  - `.toBeGreaterThan(number | bigint)`
  - `.toBeGreaterThanOrEqual(number | bigint)`
  - `.toBeLessThan(number | bigint)`
  - `.toBeLessThanOrEqual(number | bigint)`
  - `.toBeCloseTo(number, numDigits?)`
- 字符串
  - `.toMatch(regexp | string)`
- 数组
  - `.toContain(item)`
- 异常
  - `.toThrow(error?)`

</div>
</div>


<!-- # 命令行的使用 (可选) -->

---

# 测试异步代码

<div class="grid grid-cols-2 gap-4">
<div>

### 待测代码, 传统回调形式
```javascript{3-6}
function fetchData1(callback) {
    // do something
    setTimeout(() => {
        const data = "some data";
        callback(data);
    }, 500);
}
```

</div>
<div>

### 错误的示例
```javascript{2-4}
test('the data is "some data" (错误的方式1)', () => {
    fetchData1((data) => {
        expect(data).toBe("some data2");
    });
});
```

</div>
<div>

### 待测代码, Promise形式
```javascript{3-9}
function fetchData2(fakeError) {
    // do something
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            const data = "some data";
            if (fakeError) reject(new Error("fake error"));
            else resolve(data);
        }, 500);
    });
}
```

</div>
<div>

### 错误的示例
```javascript{2-4}
test('the data is "some data" (错误的方式2)', () => {
    fetchData2().then((data) => {
        expect(data).toBe("some data2");
    });
});
```

</div>
</div>

---

# 测试异步代码 - 方式1
### 方式1: 使用回调参数 done
```javascript{1,3-4}
test('the data is "some data" (done)', (done) => {
    fetchData1((data) => {
        expect(data).toBe("some data2");
        done();
    });
});
```

### 方式1: 使用回调参数 done (优化)
```javascript{1,3-8}
test('the data is "some data" (done (优化))', (done) => {
    fetchData1((data) => {
        try {
            expect(data).toBe("some data2");
            done();
        } catch (error) {
            done(error);
        }
    });
});
```

---

# 测试异步代码 - 方式2

<div class="grid grid-cols-2 gap-4">
<div>

### 方式2: 通过返回 promise
```javascript{2-4}
test('the data is "some data" (返回 promise)', () => {
    return fetchData2().then((data) => {
        expect(data).toBe("some data");
    });
});
```

### 方式2: 通过返回 promise (简化)
```javascript{2}
test('the data is "some data" (返回 promise (简化))', () => {
    return expect(fetchData2()).resolves.toBe("some data2");
});
```

</div>
<div>

### 方式2: 测试异常情况
```javascript{2-5,9}
test('error occur (返回 promise)', () => {
    expect.assertions(1);
    return fetchData2(true).catch((err) => {
        expect(err.toString()).toMatch("fake error");
    });
});

test('error occur (返回 promise (简化))', () => {
    return expect(fetchData2(true)).rejects.toThrow("fake error");
});
```

</div>
</div>

---

# 测试异步代码 - 方式3
### 方式3: 使用 async/await
```javascript{1-2}
test('the data is "some data" (使用 async/await)', async () => {
    await expect(fetchData2()).resolves.toBe("some data");
})
```
<br>
<br>

### 方式2: 测试异常情况
```javascript{1-2}
test('error occur (使用 async/await)', async () => {
    await expect(fetchData2(true)).rejects.toThrow("fake error");
});
```
<br>
<br>

👉 异步 [Guide](https://jestjs.io/docs/asynchronous), [Example](https://jestjs.io/docs/tutorial-async)

---

# 钩子函数
执行一些初始化, 清理等工作

<div class="grid grid-cols-3 gap-4">

- 针对单个测试
  - `beforeEach()`
  - `afterEach()`


+ 针对全局
  + `beforeAll()`
  + `afterAll()`

<div class="mt-0">👉 钩子函数 <a href="https://jestjs.io/docs/setup-teardown">Guide</a></div>

</div>
<div class="flex gap-4 mt-6">

### 示例
```javascript
beforeAll (() => console.log('1 - beforeAll'));
afterAll  (() => console.log('1 - afterAll'));
beforeEach(() => console.log('1 - beforeEach'));
afterEach (() => console.log('1 - afterEach'));
test  ('', () => console.log('1 - test'));
describe('Scoped / Nested block', () => {
    beforeAll (() => console.log('2 - beforeAll'));
    afterAll  (() => console.log('2 - afterAll'));
    beforeEach(() => console.log('2 - beforeEach'));
    afterEach (() => console.log('2 - afterEach'));
    test  ('', () => console.log('2 - test'));
});
```

### 输出
```txt
1 - beforeAll
1 - beforeEach
1 - test
1 - afterEach
2 - beforeAll
1 - beforeEach
2 - beforeEach
2 - test
2 - afterEach
1 - afterEach
2 - afterAll
1 - afterAll
```

</div>

---

# 作用域
- 默认情况下, 作用于整个测试文件
- 如果使用 `describe` 对测试进行了分组, 则只在该 `describe` 块下有效
- 通过例子了解执行顺序
- 通过例子了解直接写在 describe 中的代码与钩子函数的代码的执行顺序

---

# 作用域 - 示例

<div class="grid grid-cols-2 gap-4">


```javascript
describe('outer', () => {
    console.log('describe outer-a');
    describe('describe inner 1', () => {
        console.log('describe inner 1');
        test('test 1', () => {
            console.log('test for describe inner 1');
            expect(true).toEqual(true);
        });
    });
    console.log('describe outer-b');
    test('test 1', () => {
        console.log('test for describe outer');
        expect(true).toEqual(true);
    });
    describe('describe inner 2', () => {
        console.log('describe inner 2');
        test('test for describe inner 2', () => {
            console.log('test for describe inner 2');
            expect(false).toEqual(false);
        });
    });
    console.log('describe outer-c');
});
```

<div>

### 输出

```txt
// describe outer-a
// describe inner 1
// describe outer-b
// describe inner 2
// describe outer-c
// test for describe inner 1
// test for describe outer
// test for describe inner 2
```

</div>
</div>

---

# Mock
- mock 通过抹去函数的实现细节来测试代码之间的联系.
- 捕获对函数的调用, 调用时传递的参数, new 的实例, 还可以配置返回值.
- 单元测试中, 为了消除其他模块对待测模块的影响, 需要对其他模块进行 mock.
- 有两种 mock 方式:
    1. 直接在测试代码中创建
    2. 手动 mock 以覆盖模块依赖
- mock 的能力
  - `.mock` 属性
  - mock 返回值
  - mock 模块
  - mock 实现

<br>

👉 Mock [Guide](https://jestjs.io/docs/mock-functions), [Ref](https://jestjs.io/docs/mock-function-api)

---

# Mock - `.mock` 属性
- 通过调用 `jest.fn()` 即可创建一个 mock 函数
- 所有 mock 函数都有一个 `.mock` 属性, 保存了函数调用和返回值的信息


```javascript{7,9}
function forEach(items, callback) {
    for (let index = 0; index < items.length; index++) {
        callback(items[index]);
    }
}

const mockCallback = jest.fn(x => 42 + x);
forEach([0, 1], mockCallback);
console.log(mockCallback.mock);
```

### 输出

```json
{
    calls: [ [ 0 ], [ 1 ] ],
    instances: [ undefined, undefined ],
    invocationCallOrder: [ 1, 2 ],
    results: [ { type: 'return', value: 42 }, { type: 'return', value: 43 } ]
}
```

---

# Mock - mock 返回值
默认 mock 的函数被调用时, 返回 undefined, 我们可以指定任意返回值.

<div class="grid grid-cols-2 gap-4">

- 通用
  - `mockFn.mockReturnValue(value)`
  - `mockFn.mockReturnValueOnce(value)`
- 异步
  - `mockFn.mockResolvedValue(value)`
  - `mockFn.mockResolvedValueOnce(value)`
  - `mockFn.mockRejectedValue(value)`
  - `mockFn.mockRejectedValueOnce(value)`
- 其他
  - `mockFn.mockReturnThis()`

<div>

### 示例
```javascript
test("mock return value", () => {
    const myMock = jest.fn();

    console.log(myMock());
    // > undefined

    myMock.mockReturnValueOnce(10)
        .mockReturnValueOnce('x')
        .mockReturnValue(true);

    console.log(myMock(), myMock(), myMock(), myMock());
    // > 10, 'x', true, true
});
```

</div>
</div>

---

# Mock - mock 模块

<div class="grid grid-cols-2 gap-4">
<div>

### 待测代码
```javascript {6}
// users.js
import axios from 'axios';

class Users {
  static all() {
    return axios.get('/users.json').then(resp => resp.data);
  }
}
export default Users;
```

</div>
<div>

### 测试代码
```javascript {2,5,9}
// users.test.js
import axios from 'axios'; // 实际导入是在 jest.mock 之后
import Users from './users';

jest.mock('axios'); // axios 对象中的所有方法都会被 mock

test('should fetch users', async () => {
    const users = [{ name: 'Bob' }];
    axios.get.mockResolvedValue({ data: users }); // get 方法已被 mock

    // or you could use the following depending on your use case:
    // axios.get.mockImplementation(() => Promise.resolve(resp))

    await Users.all().then(data => expect(data).toEqual(users));
    expect(axios.get.mock.calls.length).toBe(1);
    expect(axios.get.mock.calls[0][0]).toBe('/users.json');
});
```

</div>
</div>

---

# Mock - mock 实现 (Implementations)
如果需要返回值的情况比较复杂, 还可以自定义实现
<div class="flex gap-2">
<div>

- `jest.fn(implementation)`
- `mockImplementation(fn)`
- `mockImplementationOnce(fn)`

<br>

### 示例1: 使用 `jest.fn()`

```javascript
const myMockFn = jest.fn(cb => cb(null, true));~~~~
myMockFn((err, val) => console.log(val));
// > true
```

</div>
<div>

### 示例2: 使用 `mockImplementation(fn)`

```javascript
// foo.js
module.exports = function () {
  // some implementation;
};

// test.js
jest.mock('../foo'); // this happens automatically with automocking
const foo = require('../foo');

// foo is a mock function
foo.mockImplementation(() => 42);
foo();
// > 42
```

</div>
</div>


---

# snapshot

---

# Mock Timers (可选)

---

# ES6 类的测试

---

# DOM 测试

---

# AngularJS
