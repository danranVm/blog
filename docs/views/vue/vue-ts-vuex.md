---
title: 构建类型安全的 Vue 应用(Vue Typescript Vuex)
date: 2020-03-28
categories:
  - Vue
tags:
  - Vue
  - Typescript
  - Vuex
---

::: tip
构建类型安全的 Vue 应用，基于：

- `@vue/cli`: v4.2.3
- `typescript`: v3.7.5
- `vue`: v2.6.11
- `vue-tsx-support`: v2.3.3
- `vuex`: v3.1.2
  :::

<!-- more -->

PS：由于我接触 `Vue` 的时间还非常短，如果文章中有一些错误，或者你有更好的解决方案。请不吝赐教，我将万分感谢，谢谢！

## 主要步骤

### 全局安装 vue cli

```bash
npm install -g @vue/cli
```

### 使用 cil 初始化项目

```bash
vue create vue-ts-demo
```

然后选择 `Manually select features`, 最终所有初始配置项如下图：
![Manually select features](https://cdn.jsdelivr.net/gh/danranvm/image-hosting/images/20200326000552.png)

### 支持 `tsx`

针对一些有复杂渲染逻辑的组件，使用 `jsx` 的写法可谓是真香了，所以我们也相应的支持一下 `tsx` 的写法。

```bash
yarn add vue-tsx-support
```

```ts
// 在 main.ts 添加以下代码
import 'vue-tsx-support/enable-check'
```

```js
// 新建 vue.config.js 添加以下代码
module.exports = {
  runtimeCompiler: true,
  configureWebpack: {
    resolve: {
      extensions: ['.js', '.vue', '.json', '.ts', '.tsx'],
    },
  },
}
```

至此，我们的项目已经支持了 `tsx` 。但是，默认情况下 `vue-tsx-support` 不允许使用未知的 `prop`, 举个栗子：

```ts
import Vue from 'vue'

const MyComponent = Vue.extend({
  props: { text: { type: String, required: true } },
  template: '<span>{{ text }}</span>',
})
```

我们有上面这样一个 `component`, 然后使用的时候就会得到一个错误。

```ts
// Compilation error(TS2339): Property `text` does not exist on type '...'
<MyComponent text="foo" />
```

除非我们可以保证，我们所有的 `component` 都是按照 `vue-tsx-support` 规范去创建的，否则我建议开启 `allow-unknown-props`.

```ts
// 新建 global.d.ts, 添加以下代码
import 'vue-tsx-support/options/allow-unknown-props'
```

完整支持 `tsx` 的代码(包括使用 `tsx` 改造 `Home` 和 `About` 组件)请参考这个[commit](https://github.com/danranVm/vue-ts-demo/commit/30cbede0590db9052942ffdf4f67f7929ced9ffc).  
关于 `vue-tsx-support` 更详细的说明请参考[文档](https://github.com/wonderful-panda/vue-tsx-support).

### 改造 `vuex`

由于 `vuex` 的类型支持不够，在使用的时候，没有智能提示，也不能保证类型安全。所以我们需要对它进行一番改造。

#### 改造基础类型

```ts
// 新建 module-type.ts, 给出一些基础的类型定义：
/** 提取出所有 module 的 state 类型 */
export interface State {}

/** 提取出所有 module 的 getters 类型 */
export type GettersFuncMap = {}

/** 提取出所有 module 的 mutations 类型 */
export type CommitFuncMap = {}

/** 提取出所有 module 的 actions 类型 */
export type DispatchFuncMap = {}
```

```ts
// 新建 store-type.ts, 完成相关类型推导
import { GettersFuncMap, CommitFuncMap, DispatchFuncMap, State } from './module-type'

/** Getter类型转换 Handler：GettersFuncMap => Getters */
export type GetterHandler<T extends { [key: string]: (...args: any) => any }> = {
  [P in keyof T]: ReturnType<T[P]>
}

/** 根据 GettersFuncMap 得到的 Getters 类型*/
export type Getters = GetterHandler<GettersFuncMap>

/** 将 MutationsFuncMap 的 key,value 转换成 Commit 函数的两个参数: type, payload */
export interface Commit {
  <T extends keyof CommitFuncMap>(type: T, payload?: Parameters<CommitFuncMap[T]>[1]): void
}

/** 将 ActionsFuncMap 的 key,value 转换成 Dispatch 函数的两个参数: type, payload */
export interface Dispatch {
  <T extends keyof DispatchFuncMap>(type: T, payload?: Parameters<DispatchFuncMap[T]>[1]): Promise<
    any
  >
}

// 导出全局的 Store 类型
export interface Store {
  state: State
  getters: Getters
  commit: Commit
  dispatch: Dispatch
}

// 导出全局的 ActionContext 类型
export interface ActionContext<S, G> {
  dispatch: Dispatch
  commit: Commit
  state: S
  getters: G
  rootState: State
  rootGetters: Getters
}
```

最后，由于无法直接覆盖全局的 `$store` 类型，所以另辟蹊径：

```ts
// 新建 global.d.ts, 添加以下代码
import { Store } from '@/store/store-type'

declare module 'vue/types/vue' {
  interface Vue {
    _store: Store
  }
}
```

```ts
// 修改 main.ts
const app = new Vue({
  router,
  store,
  render: h => h(App),
}).$mount('#app')

Vue.prototype._store = app.$store
```

至此，我们已经完成了基础的类型推导工作，下面来让我们创建一个示例。

#### `module` 示例

```ts
import { User } from '@/interfaces'
import { ActionContext, GetterHandler } from '../store-type'

const state = {
  users: [] as User[],
  isCached: true,
}

type StateType = typeof state

const getters = {
  getUsers(state: StateType): User[] {
    return state.users
  },
  getAgeCount(state: StateType): number {
    return state.users.reduce((prev, curr) => prev + curr.age, 0)
  },
  getIsCached(state: StateType): boolean {
    return state.isCached
  },
}

const mutations = {
  reset(state: StateType): void {
    state.users = []
    state.isCached = true
  },
  setUsers(state: StateType, users: User[]): void {
    state.users = users
  },
  setIsCached(state: StateType, isCached: boolean): void {
    state.isCached = isCached
  },
}

type GettersType = GetterHandler<typeof getters>
type UserContext = ActionContext<StateType, GettersType>

const actions = {
  updateState({ commit }: UserContext, payload: StateType) {
    commit('setUsers', payload.users)
    commit('setIsCached', payload.isCached)
  },
}

export const test = {
  state,
  getters,
  mutations,
  actions,
}
```

#### 在 `component` 中使用

```ts
@Component
export default class HelloWorld extends Vue {
  test(): void {
    /**
     * auto tip test
     * (property) State.test: {
     *    users: User[];
     *    isCached: boolean;
     *  }
     */
    this._store.state
    // Argument of type '"xxx"' is not assignable to parameter of type '"updateState"'.
    this._store.dispatch('xxx')
  }
}
```

如图所示，自动提示和类型检查都是正程工作的：
![auto tips](https://cdn.jsdelivr.net/gh/danranvm/image-hosting/images/20200328152552.png)
![type check](https://cdn.jsdelivr.net/gh/danranvm/image-hosting/images/20200328152724.png)

完整改造 `vuex` 的代码见[commit](https://github.com/danranVm/vue-ts-demo/commit/29635e1c09276ea3a9fbd3ab40c50f84e029fdbb).

## 总结

至此，我们的改造就完成了，接下来我们就可以好好享受类型安全带来的愉快开发体验了。  
同时，我们也期待下 `Vue 3.0` 的快点到来吧，原汁原味的 `typescript` 支持，会比我们这种骚操作式的改造方案要靠谱，也更加通用一些。  
最后，还记得那句名言吗？`Any application that can be written in JavaScript, will eventually be written in JavaScript`  
那么，我现在希望的是：`Any application that can be written in JavaScript, will eventually be written in Typescript`

## 参考资料

本文完整代码见[vue-ts-demo](https://github.com/danranVm/vue-ts-demo)  
`@vue/cli` 更详细的配置见[Vue CLI](https://cli.vuejs.org/zh/)  
`vuex` 改造方案来自[vuex-typescript-demo](https://github.com/BarneyZhao/vuex-typescript-demo)
