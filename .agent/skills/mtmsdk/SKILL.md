---
name: mtmsdk
description: 正确高效的使用mtmsdk库, 正确使用`react-query`进行数据查询.
---

# mtmsdk 库介绍

mtmsdk 库主要是实现各种基于 Openapi v3.1 规格生成的客户端代码. 并且结合react, 和 `react-query`库实现高效的类型安全的丰富的数据接口,客户端代码,查询逻辑.表单输入检验等功能.


# 生成 react-query.gen 源码介绍

1. reqctquery.gen.ts 在基本客户端请求的基础上, 封装了 `react-query` 接口匹配的options. 名称通常都是在原openapi spec operation id 的基础上, 添加 options 后缀.  并且直接返回 useQuery的选项. 选项本质是对"sdk.gen.ts"基础fetch请求参数的封装,转为react-query对应的参数.

```tsx
const someQuery = useQuery({
  queryKey:["someKey"],
  queryFn: async () => {
    const data = await sessionList(...); //sessionList 来自"sdk.gen.ts"
    return data;
  },
```

如果使用"reqctquery.gen.ts" 则这样:
```tsx
const someQuery = useQuery({
  ...sessionListOptions({ ... }),
  ...otherReactQueryOtions...
})
```

对于列表查询通常会同时带有一个基本的列表查询`xxxxListOptions` 和 无限分页查询options `xxxxListInfiniteOptions`

# react-query.gen 的注意事项

- 错误例子1: 多余的封装, userQuery本身就是一个react hook, 并且本身就具备 state管理,所以一般直接在需要用的组件中直接用 useQuery(...)
```tsx
export useSomeDdata = ()=>{
  const {data} = useQuery({
    ...someQueryOptions(),
    ...otherOptionsOrOverride.
  })
  return data;
}
```

```tsx
import { sessionGetOptions } from "mtmsdk/opencode/@tanstack/react-query.gen";
const { data: sessionInfo } = useQuery({
  ...sessionGetOptions({ path: { sessionID: sessionId }, client }),
  enabled: !!sessionId && !!client,
});
```

# 如何判断react 组件是否正确或者错误使用 react-query (react-query.gen)?

1. 如果useQuery,使用了 queryKey 或者 queryFn 就是错误的,因为这是使用原始的 react-query + fetch api. 应该使用react-query.gen生成的options.

## 无限滚动分页查询

```tsx
import {
  sessionListInfiniteOptions,
} from "mtmsdk/opencode/@tanstack/react-query.gen";

const {
    data: historyData,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
    error: historyError,
  } = useInfiniteQuery({
    ...sessionListInfiniteOptions(),
    ...otherOptionsOrOverride.
  })

```