---
layout: cover
background: /bg.jpg
download: true
title: 初识 Jest
---

# 初识 Jest
<p class="mx-56 author">———— 赵昌柱</p>

<!--
感谢领导让我作为先行者, 有机会学习自动化测试, 让我接触到一个新的开发方式.
-->

---

# 背景介绍
- TS, Flow, EsLint, StyleLint 等前端工具都可以帮助减少 bug 产生
- 引入自动化测试可以更好地降低 bug 产生的可能性
- 流行开源库都使用自动化测试. 多人协作时可以保证代码不会被意外修改
- 在迭代旧项目时, 如果有自动化测试, 在修bug或增加新功能时, 可以保证不会影响之前的功能
- 测试主要分为单元测试, 集成测试, 端到端测试(end-to-end)
- 测试编写的2种方案: TDD, BDD

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
<p class="text-xs text-gray-400">数据来源: <a href="https://sourl.cn/MFhVyj">star-history.t9t.io</a></p>

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

<div class="grid grid-cols-[8fr,7fr] gap-4">
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

### 方式3: 测试异常情况
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

<div class="grid grid-cols-4 gap-4">

- 针对单个测试
  - `beforeEach()`
  - `afterEach()`


+ 针对全局
  + `beforeAll()`
  + `afterAll()`

<div class="mt-0">👉 钩子函数 <a href="https://jestjs.io/docs/setup-teardown">Guide</a></div>

</div>
<div class="grid grid-cols-2 gap-4 mt-6">
<div>

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

</div>
<div>

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
<div class="grid grid-cols-2 gap-2">
<div>

- `jest.fn(implementation)`
- `mockImplementation(fn)`
- `mockImplementationOnce(fn)`

</div>
<div>

### 示例1: 使用 `jest.fn()`

```javascript
const myMockFn = jest.fn(cb => cb(null, true));
myMockFn((err, val) => console.log(val));
// > true
```

</div>
</div>

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

---

# Mock Timers (可选)
Jest 可以用允许你控制时间流逝的函数来替换定时器, 👉 [Guide](https://jestjs.io/docs/timer-mocks)

- `jest.useFakeTimers()` - 使用伪造版本的定时器函数
- `jest.useRealTimers()` - 使用真实版本的定时器函数
- `jest.runAllTimers()` - 耗尽任务队列, 包括任务中安排的新的任务
- `jest.runOnlyPendingTimers()` - 只执行当前待定的宏任务, 这些任务中安排的新的宏任务不会执行
- `jest.advanceTimersByTime(msToRun)` - 只执行宏任务队列, 所有的定时器提前 msToRun 毫秒, 执行时间段内待定的宏任务, 包括任务中安排的新的任务

---

# 对 DOM 节点操作的测试
Jest 带有 jsdom, 它模拟了一个DOM环境.

我们可以直接使用 document 对象, 就像在浏览器中一样.

```javascript {7-11,16,18}
import $ from 'jquery';
import {registerClick} from '../displayUser.js';
import {fetchCurrentUser} from '../fetchCurrentUser.js';
jest.mock('../fetchCurrentUser.js');

test('displays a user after a click', () => {
    document.body.innerHTML = 
    '<div>' +
    '  <span id="username" />' +
    '  <button id="button" />' +
    '</div>';
    fetchCurrentUser.mockImplementation(cb => {
        cb({fullName: 'Johnny Cash', loggedIn: true});
    });
    registerClick();
    $('#button').trigger('click');
    expect(fetchCurrentUser).toBeCalled();
    expect($('#username').text()).toEqual('Johnny Cash - Logged In');
});
```

---

# 快照 (snapshot) - 1
当想要确保对象不会发生意外变化时, 快照测试非常有用. 👉 [Guide](https://jestjs.io/docs/snapshot-testing)

- 快照测试就是将当前的值(快照)与之前保存的值(快照)进行比对. 
- 不论是因为 bug, 还是因为更改实现, 如果两次快照内容不同, 该测试用例失败.
- 初次进行快照测试时, 会把当前快照保存, 作为下次测试时的对比对象. 
- 快照保存在与测试文件同级的 `__snapshots__` 文件夹下.
- 快照测试其实就是做字符串对比, 只不过对比的对象一个是当前值, 一个是历史值.
- 进行快照时, Jest 会自动进行格式化, 确保快照的可读性.
- 应该将快照当作代码对待, 并提交到代码库.

<div class="grid grid-cols-2 gap-2 mt-1">
<div>

### 快照测试

```javascript {5}
test('Link changes the class when hovered', () => {
  const tree = renderer.create(
    <Link page="http://www.facebook.com">Facebook</Link>
  ).toJSON();
  expect(tree).toMatchSnapshot();
}
```

</div>
<div>

### 快照文件

```javascript
exports[`Link changes the class when hovered 1`] = `
<a
  className="normal"
  href="http://www.facebook.com"
  onMouseEnter={[Function]}
  onMouseLeave={[Function]}
>
  Facebook
</a>
`;
```

</div>
</div>

---

# 快照 (snapshot) - 2

### 更新快照
- 如果是有意的更改导致快照测试失败, 可以通过命令 `jest --updateSnapshot` (或 `-u`) 对快照进行更新.
- 默认更新所有失败的快照. 可以先修复 bug 造成的快照测试失败, 然后再更新. 
- 或者使用 `--testNamePattern` 选项限定更新的测试用例.
- 失败的快照也可以在交互的 watch 模式中进行更新.

<div class="mt-4">
  <img width="377" class="inline-block w-1/4 mr-4" src="snapshot-failed.png">
  <img width="446" class="inline-block w-1/3 mr-4" src="watch-mode.png">
</div>

---

# 快照 (snapshot) - 3

### 内联快照 (Inline Snapshots)
- 内联快照. 不在外部文件中, 而是直接在测试代码中生成快照. 方便查看. 使用 `.toMatchInlineSnapshot()` 方法进行内联快照测试.

### 属性匹配器 (Property Matchers)
- 动态内容, 如 id, 日期 等, 可以提供一个非对称匹配器.

<div class="grid grid-cols-[2fr,3fr] gap-2 mt-2">
<div>

### 内联快照

```javascript {6-12}
it('inline snapshot', () => {
    const obj = {
        msg: 'hello, jest!',
        date: new Date(),
    };
    expect(obj).toMatchInlineSnapshot(`
Object {
  "date": 2021-07-06T10:42:55.545Z,
  "msg": "hello, jest!",
}
`);
});
```

</div>
<div>

### 属性匹配器

```javascript {6-12}
it('inline snapshot', () => {
    const obj = {
        msg: 'hello, jest!',
        date: new Date(),
    };
    expect(obj).toMatchInlineSnapshot({ date: expect.any(Date) }, `
Object {
  "date": Any<Date>,
  "msg": "hello, jest!",
}
`);
});
```

</div>
</div>

---

# ES6 类的测试

---

# React

---

# AngularJS


---

# 单元测试, 集成测试

<div class="grid grid-cols-2 gap-4">
<div>

### 单元测试
- 测试独立模块
- 测试更细致
- 提高边界覆盖率
- 无法保证功能连接起来是没有问题的
- 如果改变代码实现细节, 需要更改对应单元测试代码
- 需要编写大量测试代码

</div>
<div>

### 集成测试
- 联动测试
- 进行整体代码的测试
- 如果通过测试, 说明代码一定可以运行

</div>
</div>

---

# TDD, BDD

<div class="grid grid-cols-2 gap-4">
<div>

### TDD: Test Driven Development  
1. 先写测试再写代码
2. 一般结合单元测试使用, 是白盒测试
3. 测试重点在代码
4. 安全感低
5. 速度快
6. 适合工具库的测试

</div>
<div>

### BDD: Behavior Driven Development
1. 先写代码再写测试
2. 一般结合集成测试使用, 是黑盒测试
3. 测试重点在 UI (DOM)
4. 安全感高
5. 速度慢
6. 适合业务 UI 测试

</div>
</div>

---
layout: center
class: text-center
---

<h1 style="font-size:96px;letter-spacing:0.3em">谢谢!</h1>