---
title: "React Redux Course Note"
date: 2023-01-21T15:32:14Z
lastmod: "2023-01-21"
tags: ["React", "Redux", "Course Note"]
draft: false
summary: "A Course Note of Udemy React Course Redux Part, includes Redux_Thunk and Redux_Saga"
layout: PostSimple
bibliography: references-data.bib
---

# Redux

> 本文为 [Complete React Developer in 2023 (w/ Redux, Hooks, GraphQL) | Udemy](https://www.udemy.com/course/complete-react-developer-zero-to-mastery/) Redux 部分课程笔记（14 章-17 章）
>
> 起始文件 GitHub Repo [ZhangMYihua/crwn-clothing-v2 at lesson-27 (github.com)](https://github.com/ZhangMYihua/crwn-clothing-v2/tree/lesson-27)

> This Post is the Note of [Complete React Developer in 2023 (w/ Redux, Hooks, GraphQL) | Udemy](https://www.udemy.com/course/complete-react-developer-zero-to-mastery/) Redux Part (From Chap 14 -> Chap 17)
>
> Start Resource GitHub Repo [ZhangMYihua/crwn-clothing-v2 at lesson-27 (github.com)](https://github.com/ZhangMYihua/crwn-clothing-v2/tree/lesson-27)

## Installation

redux react-redux redux-logger(Optional)

## Init

初始化 Store
新建 `store/store.js`

```jsx
// store/store.js

import { compose, createStore, applyMiddleware } from "redux";
import logger from "redux-logger";
```

为了创建 Store，需要一个 rootReducer
新建`store/root-reducer.js`

```jsx
// store/root-reducer.js

import { combineReducers } from "redux";
import { userReducer } from "./user/user.reducer";

/* Key为Reducer的名字，Value为Reducer本身 */
export const rootReducer = combineReducers({});
```

接下来迁移 UserContext 的内容到 rootReducer，对每一 reducer 也同样的做模块化的处理，新建`store/user/user.reducer.jsx`，然后将`user.context.jsx`里 useReducer 相关内容做复制

```jsx
// store/user/user.reducer.jsx

export const USER_ACTION_TYPES = {
  SET_CURRENT_USER: "SET_CURRENT_USER",
};

const INITIAL_STATE = {
  currentUser: null,
};

const userReducer = (state, action) => {
  const { type, payload } = action;

  switch (type) {
    case USER_ACTION_TYPES.SET_CURRENT_USER:
      return { ...state, currentUser: payload };
    default:
      throw new Error(`Unhandled type ${type} in userReducer`);
  }
};
```

此处，需要在添加 state 参数添加默认 state，此前则是通过`useReucer`hook 来设置默认 state

```jsx
// store/user/user.reducer.jsx

export const userReducer = (state = INITIAL_STATE, action) => {
  // ...
};
```

同时这里要默认返回一个 state，因为对于 Redux， Dispatch 的 Action 会分发给每一个 Reducer，
而不像 React 自带的 useReducer 一样，dispatch 只影响对应的 Reducer，因此要默认返回一个 State。

```jsx
// store/user/user.reducer.jsx

switch (type) {
  case USER_ACTION_TYPES.SET_CURRENT_USER:
    return { ...state, currentUser: payload };
  default:
    return state;
}
```

现在可以将其添加到`rootReducer`了。

```jsx
// store/root-reducer.js

export const rootReducer = combineReducers({
  user: userReducer,
});
```

回到`store.js`，完成 store 的创建

```jsx
// store.js

const middleWares = [logger];

const composedEnhancers = compose(applyMiddleware(...middleWares));

export const store = createStore(rootReducer, undefined, composedEnhancers);

/*  正常来讲，创建一个store只需要有这个rootReducer就可以了，第二个参数是任何其他默认状态，
    添加它可以让测试变容易
    第三个参数就是MiddleWare了。
*/
```

这就完成基本的创建了，接下来看如何进行 dispatch，实际使用 redux

## Dispatch

首先先去掉本身`UserContext`的`Provider`，随后进入`App.js`开始使用 redux

```jsx
// App.js

const setCurrentUser = (user) =>
  dispatch(createAction(USER_ACTION_TYPES.SET_CURRENT_USER, user));

useEffect(() => {
  const unsubscribe = onAuthStateChangedListener((user) => {
    if (user) {
      createUserDocumentFromAuth(user);
    }
    setCurrentUser(user);
  });

  return unsubscribe;
}, []);
```

从 `user.reducer.js` 将相关设计 Reducer 的操作复制，补全引入
但是此处关键在于`setCurrentUser`，原先的代码如下

```jsx
// user.reducer.js

const [{ currentUser }, dispatch] = useReducer(userReducer, INITIAL_STATE);

const setCurrentUser = (user) =>
  dispatch(createAction(USER_ACTION_TYPES.SET_CURRENT_USER, user));
```

这是涉及到原本的 useReducer 的 dispatch，因此要重写这个方法，为了更好地管理，新建
`store/user/user.action.js`

```jsx
// store/user/user.action.js

export const setCurrentUser = (user) =>
  createAction(USER_ACTION_TYPES.SET_CURRENT_USER, user);
```

此处只返回一个 Action，不去做 Dispatch，同时再对 types 也单独模块化

```jsx
// store/user/user.types.js
export const USER_ACTION_TYPES = {
  SET_CURRENT_USER: "SET_CURRENT_USER",
};
```

然后再替换`App.js`里的`setCurrentUser`即可，接下来看如何将 Redux 内部的值放入组件中

## Selectors

如何使用 Redux 的值呢？进入到`navigation.component.jsx`，这里需要使用到`currentUser`。
使用`useSelector`来提取到要的 state

```jsx
// navigation.component.jsx

// const { currentUser } = useContext(UserContext);
const currentUser = useSelector(selectCurrentUser);
```

对于每一个 reducer 的 state，都建立专门的 selector 来进行提取
新建`store/user/user.selector.js`

```jsx
// user.selector.js
export const selectCurrentUser = (state) => state.user.currentUser;
```

`state`，就是整个的 `store`，`user` 也就是选择其中的 `reducer` 分支，`currentUser` 则是这个 `reducer` 下提供的 `state`

至此，就可以在组件中使用 redux 的状态了。

### Categories Reducer

对 categoryContext 做迁移

```jsx
// category.types.js

export const CATEGORIES_ACTION_TYPES = {
  SET_CATEGORIES_MAP: "category/SET_CATEGORIES_MAP",
};
```

```jsx
// category.action.js

import { createAction } from "../../src/utils/reducer/reducer.utils";
import { CATEGORIES_ACTION_TYPES } from "./category.types";

const setCategoriesMap = (categoriesMap) =>
  createAction(CATEGORIES_ACTION_TYPES.SET_CATEGORIES_MAP, categoriesMap);
```

```jsx
// category.reducer.js

import { CATEGORIES_ACTION_TYPES } from "./category.types";

export const CATEGORIES_INITIAL_STATE = {
  categoriesMap: {},
};

export const categoriesReducer = (
  state = CATEGORIES_INITIAL_STATE,
  action = {}
) => {
  const { type, payload } = action;

  switch (type) {
    case CATEGORIES_ACTION_TYPES.SET_CATEGORIES_MAP:
      return { ...state, categoriesMap: payload };
    default:
      return state;
  }
};
```

```jsx
// root-reducer.js

import { combineReducers } from "redux";
import { userReducer } from "./user/user.reducer";
import { categoriesReducer } from "./category/category.reducer";

/* Key为Reducer的名字，Value为Reducer本身 */
export const rootReducer = combineReducers({
  user: userReducer,
  categoires: categoriesReducer,
});
```

### Categories Selectors

使用数据

```jsx
// category.selector.js

export const selectCategoriesMap = (state) => state.categories.categoriesMap;
```

搭配 useSelector 使用

```jsx
// shop.component.jsx

const dispatch = useDispatch();

useEffect(() => {
  const getCategoriesMap = async () => {
    const categoryMap = await getCategoriesAndDocuments("categories");
    dispatch(setCategoriesMap(categoryMap));
  };

  getCategoriesMap();
}, []);
```

```jsx
// category.component.jsx && categories-preview.component.jsx

const categoriesMap = useSelector(selectCategoriesMap);
```

### Business Logic in Our Selectors

`shop.component.jsx`里使用到了`getCategoriesAndDocuments`，查看其原函数

```jsx
// firebase.utils.js

export const getCategoriesAndDocuments = async () => {
  const collectionRef = collection(db, "categories");
  const q = query(collectionRef);

  const querySnapshot = await getDocs(q);
  const categoryMap = querySnapshot.docs.reduce((acc, docSnapshot) => {
    const { title, items } = docSnapshot.data();
    acc[title.toLowerCase()] = items;
    return acc;
  }, {});

  return categoryMap;
};
```

可以看到，所获得的`categoryMap`，并不是 fetch 到的最初数据，而是经过处理后的。但是对于 Redux，储存在其内部 state 的，应该是最原始 fetch 到的 data，所需要的转化后的数据，应该交给 selector 来完成。

```jsx
// firebase.utils.js

export const getCategoriesAndDocuments = async () => {
  const collectionRef = collection(db, "categories");
  const q = query(collectionRef);

  const querySnapshot = await getDocs(q);

  return querySnapshot.docs.map((docSnapshot) => docSnapshot.data());
};
```

这里返回最原始的获取到的 data，categoriseArray

修改 types

```jsx
// category.types.js

export const CATEGORIES_ACTION_TYPES = {
  SET_CATEGORIES: "category/SET_CATEGORIES_MAP",
};
```

以及 reducer

```jsx
// category.reducer.js

switch (type) {
  case CATEGORIES_ACTION_TYPES.SET_CATEGORIES:
    return { ...state, categories: payload };
  default:
    return state;
}
```

还有 action

```jsx
// category.action.js

export const setCategories = (categoriesArray) =>
  createAction(CATEGORIES_ACTION_TYPES.SET_CATEGORIES, categoriesArray);
```

此时回到`shop.component.jsx`做出更改

```jsx
//shop.component.jsx

const categoriesArray = await getCategoriesAndDocuments("categories");
dispatch(setCategories(categoriesArray));
```

最后则是修改 selector

```jsx
export const selectCategoriesMap = (state) =>
  state.categories.categories.reduce((acc, docSnapshot) => {
    const { title, items } = docSnapshot.data();
    acc[title.toLowerCase()] = items;
    return acc;
  }, {});
```

获取的 state 得到的 categories 是最原始的数据，然后做出相同的处理，最后这个 selector 返回的，仍然是 categoriesMap，所以其他部分无需更改

### What Triggers useSelector

刚才对`selectCategoriesMap`做出了更改，那在每次调用 selector 的时候，都会执行一次数据处理函数，那这会造成性能的浪费吗？什么时候`useSlector`触发呢？

我们在每一次 dispatch action 的时候，这个 action 会被 root reducer 的所有 reducer 都接受到，符合要求的 reducer 会按照 action 返回新的 state，不符合的则返回旧的 state。**但是**，对于 root reducer 他所生成的每次都会是一个全新的 state。对于使用了`useSelector`的语句，无论你的 selector 是谁，只要 state 发生了更新，`useSelector`就会再次运行一次。

对于这是否会触发重新渲染，要取决于具体的代码结构。

## Demystifying Middleware

![Demystifying-Middleware](//Images/React/Demystifying-Middleware.png)

Middleware 在整个 redux 流程中占据什么位置？

当 dispatch action 之后，都会经过 Middleware，当地一个 Middleware 处理后会调用 next()给下一个 Middleware 或者给到 reducer，当所有 Middleware 都经过了以后，就会分发给所有的 Reducer 了，正如图片所示。

来看如何编写自己的 Middleware，以 loggerMiddleware 为例。

在这之前，先看看什么是柯里化函数(Curry Function)

> ```jsx
> const curryFunc = (a) => (b, c) => a + b - c;
> ```
>
> 这就是一个柯里化函数，来看看如何使用它
>
> ```jsx
> const with3 = curryFunc(3);
> with3(2, 4); // 3 + 2 - 4
> ```
>
> 这实际是一个函数生成器，这里`with3`实际上会是一个函数`(b, c) => 3 + b - c`，我们可以依靠这个特性，创造出很多可复用的函数
>
> ```jsx
> with7(7);
> with10(10);
> ```

```jsx
const loggerMiddleware = (store) => (next) => (action) => {
  if (!action.type) return next(action);
  console.log("Type: ", action.type);
  console.log("Payload: ", action.payload);
  console.log("currentState: ", store.getState());
  next(action);
  console.log("currentState: ", store.getState());
};
```

这样就已经实现了一个简单的 loggerMiddleware。

## Redux Triggers Extra Re-renders

来看一下这个 selector 会触发什么问题。

```jsx
// category.selector.js

export const selectCategoriesMap = (state) =>
  state.categories.categories.reduce((acc, docSnapshot) => {
    const { title, items } = docSnapshot.data();
    acc[title.toLowerCase()] = items;
    return acc;
  }, {});
```

我们在`category.component.jsx`里是用了它

```jsx
// category.component.jsx

const categoriesMap = useSelector(selectCategoriesMap);
```

之前提到了，只要整个 Redux State 发生了更新，`useSelector`就会重新执行。**但是**，当`useSelector`所获取回来的数据和原先不一样的时候（此处即为`selectCategoriesMap`的返回值），他会触发整个组件的重新渲染。

这意味着只要整个 Redux State 发生了更新，`category.component.jsx`就会重新渲染，即便`CategoriesMap`实际上并没有发生改变，那么这是为什么？

看到`selectCategoriesMap`,它是用了 reduce，并把每一次 reduce 的结果放入一个新的 Object。这意味着对于`useSelector`，返回的每次都是一个新的值，即使里面实际的内容没有发生改变。也就是说，这会触发无意义的重复渲染。解决的办法则是使用**Reselect**库。

## Reselect Library

Reselect 库为我们提供了记忆选择器的概念。什么是记忆选择器？

> 记忆选择器会会缓存你先前的输出值。如果你的输入值没有改变，那么它会直接返回相同的输入值。

这对于纯函数非常有作用，例如：

```jsx
const add = (a, b) => a + b;
add(1, 3); // 4
```

纯函数相同的输入一定返回相同的输出，如果发现输入值和之前的一样，那么为什么要再去运行一次这个函数呢？直接返回相同的输出值即可。

那么如何使用 Reselect 库呢？我们需要创建输入选择器和输出选择器。
进入`category.selector.js`，首先我们需要一个初始选择器，这个选择器只返回我们需要得的 reducer state，在这里也就是 category reducer

```jsx
// category.selector.js
import { createSelector } from "reselect";

const selectCategoryReducer = (state) => state.categories;
```

接下来就是创建一个记忆选择器

```jsx
export const selectCategories = createSelector(
  [selectCategoryReducer],
  (categoriesSlice) => categoriesSlice.categories
);
```

这就是一个记忆选择器，`createSelector`的第一个参数是输入选择器，第二个参数是输出选择器。其具体作用是，当输入选择器内返回的数据不变时，不执行输出选择器的函数。只有当输入选择器的返回结果发生了改变，才调用输出选择器。所以这意味着，只有当整个`state.categories`不一样了，输出选择器才会执行。

> 输入选择器可以有多个依赖项，类似`useEffect`
>
> ```jsx
> createSelector([a, b], (aSlice, bSlice) => {}))
> ```

挑选出 categories state 以后就可以转化出 categoriesMap 了

```jsx
export const selectCategoriesMap = createSelector(
  [selectCategories],
  (categories) =>
    categories.reduce((acc, docSnapshot) => {
      const { title, items } = docSnapshot.data();
      acc[title.toLowerCase()] = items;
      return acc;
    }, {})
);
```

这意味着，当 categories state 没有发生变化的时候，不要去运行输出选择器。这样就可以很好解决掉额外重复渲染的问题了。

## Redux-Persist

`Redux-Persist`允许我们将 Redux reducer 的值持久保存在本地储存，刷新的时候不会导致数据丢失。

安装库`redux-persist`后进入`store.js`引入方法

```jsx
// store.js

import { persistStore, persistReducer } from "redux-persist";
```

随后编写配置

```jsx
// store.js
import storage from "redux-persist/lib/storage";

const persistConfig = {
  key: "root",
  storage,
  blacklist: ["user"],
};
```

key 表面想要从哪一层级开始，此处为根级别开始

storage 表面存储位置

blacklist 表面哪一个 reducer 的值不需要做储存，这里选择 user。因为每次都会重新后台验证，所以不需要储存。

现在创建`persistReducer`并应用，然后创建`persistStore`

```jsx
const persistReducer = persistReducer(persistConfig, rootReducer);

export const store = createStore(persistReducer, undefined, composedEnhancers);

export const persistor = persistStore(store);
```

回到`index.js`使用它

```jsx
// index.js

import { persistor, store } from "../store/store";
import { PersistGate } from "redux-persist/integration/react";

// ...

render(
  <React.StrictMode>
    <Provider store={store}>
      <PersistGate loading={null} persistor={persistor}>
        <BrowserRouter>
          <App />
        </BrowserRouter>
      </PersistGate>
    </Provider>
  </React.StrictMode>
  // rootElement
);
```

这里的`loading`为在 Redux 储存读取完毕前显示什么，这里为 null，即未完成之前不渲染。

## Asynchronous Redux\_ Redux-Thunk

> [Redux Fundamentals, Part 6: Async Logic and Data Fetching | Redux](https://redux.js.org/tutorials/fundamentals/part-6-async-logic#redux-middleware-and-side-effects)

Reducer 是一个纯函数，不能有任何的 Side Effect，也就是不能因为输入的不同导致函数的返回值不一样。典型的没有 Side Effect 的函数有`(a, b) => a + b`。有副作用的，包括 console，保存文件，AJAX 请求等。

任何一个 App，都一定会涉及到上面的操作，如果这些 Side Effect 我们不能放到 Reducer 里，那就把它放到 Middleware 里。Dispatch 的所有 action，都会先经过 Middleware 再到达 Reducuer。也就是说，首次 Dispatch 的 action 可以不是一个 Plain Object，可以是带有副作用的函数，等到 Middleware 处理后，由 Middleware 处理后交由 Reducer 或 重新 Dispatch 一个新的为 Plain Object 的 action。

### Using Middleware to Enable Async Logic

看看如何在 Middleware 捕获特定 Action Types 后，运行一定的异步逻辑。

```jsx
const delayedActionMiddleware = (storeAPI) => (next) => (action) => {
  if (action.type === "todos/todoAdded") {
    setTimeout(() => {
      // Delay this action by one second
      next(action);
    }, 1000);
    return;
  }

  return next(action);
};
```

可以看到，这里的问题是，每一个异步操作都要为它专门写一套捕获逻辑，是否可以写一个 Middleware，让我们 Dispatch 一个函数，Middleware 发现这是一个函数后帮我们运行它然后再去重新 Dispatch 呢？

```jsx
const asyncFunctionMiddleware = (storeAPI) => (next) => (action) => {
  // If the "action" is actually a function instead...
  if (typeof action === "function") {
    // then call the function and pass `dispatch` and `getState` as arguments
    return action(storeAPI.dispatch, storeAPI.getState);
  }

  // Otherwise, it's a normal action - send it onwards
  return next(action);
};
```

然后就可以像这样使用

```jsx
const middlewareEnhancer = applyMiddleware(asyncFunctionMiddleware);
const store = createStore(rootReducer, middlewareEnhancer);

// Write a function that has `dispatch` and `getState` as arguments
const fetchSomeData = (dispatch, getState) => {
  // Make an async HTTP request
  client.get("todos").then((todos) => {
    // Dispatch an action with the todos we received
    dispatch({ type: "todos/todosLoaded", payload: todos });
    // Check the updated store state after dispatching
    const allTodos = getState().todos;
    console.log("Number of todos after loading: ", allTodos.length);
  });
};

// Pass the _function_ we wrote to `dispatch`
store.dispatch(fetchSomeData);
// logs: 'Number of todos after loading: ###'
```

`Redux-Thunk`正是官方推出的这样一个 Middleware，让我们实现上述的效果。将异步逻辑写为 Thunk 函数，可以让我们复用一套逻辑，而不需要知道我们当下的 Redux Store 是什么样的。

### Redux-Thunk Pt.1

首先添加`Redux-Thunk`Middleware：

```jsx
// ...
import thunk from "redux-thunk";
// ...
const middleWares = [loggerMiddleware, thunk];
```

来到`shop.component.jsx`看到有一串异步代码：

```jsx
useEffect(() => {
  const getCategoriesMap = async () => {
    const categoriesArray = await getCategoriesAndDocuments("categories");
    dispatch(setCategories(categoriesArray));
  };

  getCategoriesMap();
}, []);
```

可以将这里面的`getCategoriesMap`写为 Thunk Function。

第一步，找到`category.reducer.js`，修改`CATEGORIES_INITIAL_STATE`

```jsx
export const CATEGORIES_INITIAL_STATE = {
  categories: {},
  isLoading: false,
  error: null,
};
```

因为会进行异步操作了，而且异步操作有时会报错，所以添加`isLoading`和`error`。

同时打开`category.types.js`修改 types

```jsx
export const CATEGORIES_ACTION_TYPES = {
  FETCH_CATEGORIES_START: "category/FETCH_CATEGORIES_START",
  FETCH_CATEGORIES_SUCCESS: "category/FETCH_CATEGORIES_SUCCESS",
  FETCH_CATEGORIES_FAILED: "category/FETCH_CATEGORIES_FAILED",
};
```

删除了原来的`SET_CATEGORIES: "category/SET_CATEGORIES_MAP"`，因为接下来 Middleware 内会是异步操作，是函数。在获取到想要的数据后，直接传给 Reducer 即可。成功了，就带上`FETCH_CATEGORIES_SUCCESS`的 types 和新的 state，不再需要具体的 action types。

对应的修改 Reducer

```jsx
export const categoriesReducer = (
  state = CATEGORIES_INITIAL_STATE,
  action = {}
) => {
  const { type, payload } = action;

  switch (type) {
    case CATEGORIES_ACTION_TYPES.FETCH_CATEGORIES_START:
      return { ...state, isLoading: true };
    case CATEGORIES_ACTION_TYPES.FETCH_CATEGORIES_SUCCESS:
      return { ...state, categories: payload, isLoading: false };
    case CATEGORIES_ACTION_TYPES.FETCH_CATEGORIES_FAILED:
      return { ...state, error: payload, isLoading: false };
    default:
      return state;
  }
};
```

### Redux-Thunk Pt.2

添加对应的 Actions，并删除原来的。

```jsx
// category.action.js

export const fetchCategoriesStart = () =>
  createAction(CATEGORIES_ACTION_TYPES.FETCH_CATEGORIES_START);

export const fetchCategoriesSuccess = (categoriesArray) =>
  createAction(
    CATEGORIES_ACTION_TYPES.FETCH_CATEGORIES_SUCCESS,
    categoriesArray
  );

export const fetchCategoriesFailed = (error) =>
  createAction(CATEGORIES_ACTION_TYPES.FETCH_CATEGORIES_START, error);
```

接下来将`shop.component.jsx`内的异步逻辑做出提取

```jsx
// category.action.js

export const fetchCategoriesAsync = () => async (dispatch) => {
  dispatch(fetchCategoriesStart());
  try {
    const categoriesArray = await getCategoriesAndDocuments("categories");
    dispatch(fetchCategoriesSuccess(categoriesArray));
  } catch (error) {
    dispatch(fetchCategoriesFailed(error));
  }
};
```

最后修改`shop.component.jsx`原先的异步逻辑删除，改为 Dispatch 刚才的 Thunk Function。

```jsx
useEffect(() => {
  dispatch(fetchCategoriesAsync());
}, []);
```

### Redux-Thunk Pt.3

有了 `isLoading` 的 state，可以利用起来，在加载的时候显示 `Spinner` 提供更好的体验。

创建`Spinner`

```jsx
// spinner.component.jsx

import { SpinnerContainer, SpinnerOverlay } from "./spinner.styles";

const Spinner = () => (
  <SpinnerOverlay>
    <SpinnerContainer />
  </SpinnerOverlay>
);

export default Spinner;
```

```jsx
// spinner.styles.jsx

import styled from "styled-components";

export const SpinnerOverlay = styled.div`
  height: 60vh;
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
`;

export const SpinnerContainer = styled.div`
  display: inline-block;
  width: 50px;
  height: 50px;
  border: 3px solid rgba(195, 195, 195, 0.6);
  border-radius: 50%;
  border-top-color: #636767;
  animation: spin 1s ease-in-out infinite;
  -webkit-animation: spin 1s ease-in-out infinite;
  @keyframes spin {
    to {
      -webkit-transform: rotate(360deg);
    }
  }
  @-webkit-keyframes spin {
    to {
      -webkit-transform: rotate(360deg);
    }
  }
`;
```

新增`isLoading`选择器

```jsx
// category.selector.js

export const selectCategoriesIsLoading = createSelector(
  [selectCategoryReducer],
  (categoriesSlice) => categoriesSlice.isLoading
);
```

分别在 `categories-preview.component.jsx` 和 `category.component.jsx` 使用它

```jsx
// category.component.jsx

// ...
const Category = () => {
  // ...
  const isLoading = useSelector(selectCategoriesIsLoading);
  // ...
  return (
    <Fragment>
      <Title>{category.toUpperCase()}</Title>
      {isLoading ? (
        <Spinner />
      ) : (
        // ...
      )}
    </Fragment>
  );
};

export default Category;
```

```jsx
// categories-preview.component.jsx

// ...
const CategoriesPreview = () => {
  // ...
  const isLoading = useSelector(selectCategoriesIsLoading);
  return (
    <Fragment>
      {isLoading ? (
        <Spinner />
      ) : (
        // ...
      )}
    </Fragment>
  );
};
```

## Asynchronous Redux_Redux-Saga

> [自述 · Redux-Saga (redux-saga-in-chinese.js.org)](https://redux-saga-in-chinese.js.org/)
>
> redux-saga 是一个用于管理应用程序 Side Effect（副作用，例如异步获取数据，访问浏览器缓存等）的 library，它的目标是让副作用管理更容易，执行更高效，测试更简单，在处理故障时更容易。
>
> 可以想像为，一个 saga 就像是应用程序中一个单独的线程，它独自负责处理副作用。 redux-saga 是一个 redux 中间件，意味着这个线程可以通过正常的 redux action 从主应用程序启动，暂停和取消，它能访问完整的 redux state，也可以 dispatch redux action。
>
> redux-saga 使用了 ES6 的 Generator 功能，让异步的流程更易于读取，写入和测试。通过这样的方式，这些异步的流程看起来就像是标准同步的 Javascript 代码。

Saga 虽然是一个 Middleware，但是数据流会在 Reducer 更新后，才流进 Saga，并基于 Action 执行一些业务逻辑、异步请求等。而 Saga 也可能再次新 Dispatch 一个 Action，并且新的 Action 也可能再次流入 Saga。

**Saga 的独特之处在于它会在 Reducer 更新后再执行，所以在 Saga 内部获得的 Store 值是在 Reducer 更新后的新的 Store 值。**

安装 Saga 后配置它。首先需要一个 Root Saga，就像 Root Reducer 一样。

```jsx
// root-saga.js

import { all, call } from "redux-saga/effects";

export function* rootSaga() {}
```

可以看到，这里面`rootSaga`是一个 Generator Function，因为 Saga 本身就是以 Generator Function 写的。

然后在`store.js`里添加它

```jsx
// store.js

// ...
// import thunk from "redux-thunk";
import { rootSaga } from "./root-saga";
const middleWares = [
  loggerMiddleware,
  sagaMiddleware,
  // thunk
];
// ...

export const store = createStore(persistReducer, undefined, composedEnhancers);

sagaMiddleware.run();
// ...
```

这里我们删除了`Redux Thunk`，因为这类异步控制的 Middleware 只需要一个，然后再在 `store` 被实例化以后，运行 Saga，由此就完成好配置了。

### Generator Functions

生成器（Generator Functions）是 ES6 的新特性，它类似于`Async/Await`，并且事实上`Async/Await`是基于 Generator Functions 搭建的。类似于`Async/Await`一样可以在异步操作的时候暂停执行，Generator Function 在每次碰到内部的`yeild`关键字时，都会暂停执行。

```jsx
function* gen() {
  console.log("a");
  console.log("b");
}

const g = gen();
```

带`*`的函数就是生成器。如果这是一个普通的函数，在控制台运行这段代码后，应该会分别输出`a`和`b`，并且`g`为`undefined`。但实际上不会输出，并且如果运行`console.log(g)`可以看到的是`g`为一个生成器对象`Object [Generator]`。我们可以调用`next`来恢复执行。

```jsx
g.next();
```

这个时候打开 Browser Console，可以看到有了 a、b 的输出，同时还有一行`{value: undefined, done: true}`。这里面，`done`表示该执行器内部已经执行完毕，而`value`可以在在给生成器内部加上`yeild`关键字看到其意义。

```jsx
function* gen2(i) {
  yield i;
  yield i + 10;
}

const g2 = gen2(5);

console.log(g2.next());
console.log(g2.next());
console.log(g2.next());
```

第一次输出：`{ value: 5, done: false }`。对应第一次`yield i`。

第二次输出：`{ value: 15, done: false }`。对应第二次`yield i + 10`，即 5 + 10 = 15。

第三次输出：`{ value: undefined, done: true }`。第三次返回 `value` 为空，因为内部执行已经结束。如果不想最后的 `value` 为空怎么办？只需要在函数最后加上 `return` 语句，就可以使最后一次的 `value` 的值为 `return` 返回的值。

Generator Function 可以在函数内储存多个执行，并且可以随时控制它的移动和执行，这是它的作用之一。

## Redux-Saga\_ fetchCategoriesAsync Thunk to Saga

创建第一个 Saga，用于取代 `fetchCategoriesAsync`，新建`category.saga.js`。

```jsx
// category.saga.js

import { takeLatest, all, call, put } from "redux-saga/effects";
import { getCategoriesAndDocuments } from "../../src/utils/firebase/firebase.utils";
import {
  fetchCategoriesFailed,
  fetchCategoriesSuccess,
} from "./category.action";
import { CATEGORIES_ACTION_TYPES } from "./category.types";

export const fetchCategoriesAsync = () => async (dispatch) => {
  dispatch(fetchCategoriesStart());
  try {
    const categoriesArray = await getCategoriesAndDocuments("categories");
    dispatch(fetchCategoriesSuccess(categoriesArray));
  } catch (error) {
    dispatch(fetchCategoriesFailed(error));
  }
};
```

导入这些文件，然后从 `category.action.js` Copy `fetchCategoriesAsync`。

```jsx
// category.saga.js

export function* categoriesSaga() {
  yield all([]);
}
```

此处`categoriesSaga`用于管理所有和 categories 相关的 Saga，`all`的意思是，直到其参数内的数组的所有 Saga 运行完以前，不要运行它下面剩下的代码。

```jsx
// category.saga.js

export function* onFetchCategories() {
  yield takeLatest(
    CATEGORIES_ACTION_TYPES.FETCH_CATEGORIES_START,
    fetchCategoriesAsync
  );
}
```

此处新建了一个`onFetchCategories`，里面的`takeLatest`的作用时，获取最新的 action，并在 action 为期望的 action（第一个参数）的时候，执行第二个参数的内的 Saga。

也就是说，当我们截取到最新的 action 为`CATEGORIES_ACTION_TYPES.FETCH_CATEGORIES_START`以后，就运行`fetchCategoriesAsync`

```jsx
// category.saga.js
export function* fetchCategoriesAsync() {
  try {
    const categoriesArray = yield call(getCategoriesAndDocuments("categories"));
    yield put(fetchCategoriesSuccess(categoriesArray));
  } catch (error) {
    yield put(fetchCategoriesFailed(error));
  }
}
```

这里将原先的`fetchCategoriesAsync`改造成了 Saga 形式，首先把`await`关键字改成了 `yeild`，并由 `call` 去包裹 `getCategoriesAndDocuments`，第二个参数为要传给 `getCategoriesAndDocuments` 的参数。达到和之前 `await` 关键字一样的效果。

其次，将 `dispatch` 改为 `put`，`put` 的作用和 `dispatch` 一样，就是发布一个新的 action。

```jsx
// category.saga.js

export function* categoriesSaga() {
  yield all([onFetchCategories]);
}
```

现在将`onFetchCategories`添加到`categoriesSaga`里。并且将`categoriesSaga`也添加到`root-saga`里。

```jsx
// root-saga.js

export function* rootSaga() {
  yield all([call(categoriesSaga)]);
}
```

现在，删除原先`category.action.js`里的`fetchCategoriesAsync`，然后来到`shop.component.jsx`

```jsx
// shop.component.jsx

useEffect(() => {
  dispatch(fetchCategoriesStart());
}, []);
```

回顾整个流程，首先，先将`fetchCategoriesAsync`改为 Saga 形式，其次创建`onFetchCategories`捕获 action 并在捕获成功后执行`fetchCategoriesAsync`。

然后，在`categoriesSaga`内等待拦截所有的 saga 运行完毕，最后也一样在`root saga`内，等待拦截`categoriesSaga`运行完毕。

所以，这里 dispatch 出去的 action 改为`fetchCategoriesStart`，因为 saga 会拦截后运行异步逻辑。

## Redux-Saga\_ Converting onAuthStateChanged Listener to Promise

现在的 App 内，有很多异步相关的逻辑，例如

```jsx
// sign-in-form.component.jsx

const signInWithGoogle = async () => {
  await signInWithGooglePopup();
};
```

```jsx
// App.js

useEffect(() => {
  const unsubscribe = onAuthStateChangedListener((user) => {
    if (user) {
      createUserDocumentFromAuth(user);
    }
    dispatch(setCurrentUser(user));
  });

  return unsubscribe;
}, []);
```

可以利用 Saga 让这些部分从组件内解放出来。先从`App.js`开始。

可以看到在`useEffect`内，我们监听某个 Listener 然后在某个时候退订（返回 unsubscribe），所以第一步是将检查是否有用户转换为一个基于 Promise 的函数。

```js
// firebase.utils.js

export const getCurrentUser = () =>
  new Promise((resolve, reject) => {
    const unsubscribe = onAuthStateChanged(
      auth,
      (userAuth) => {
        unsubscribe();
        resolve(userAuth);
      },
      reject
    );
  });
```

返会一个 promise，第二个参数`callback`内，一旦获取了 userAuth 就退订，避免造成一直监听而导致泄露，第三个参数则是失败后的回调，传入 reject 即可。

```js
// App.js

useEffect(() => {
  getCurrentUser();
}, []);
```

删掉原来的代码并替换。

## Redux-Saga\_ Check User Session Saga

首先在`user.types.js`添加一些新的 types。

```js
// user.types.js
export const USER_ACTION_TYPES = {
  SET_CURRENT_USER: "user/SET_CURRENT_USER",
  CHECK_USER_SESSION: "user/CHECK_USER_SESSION",
  GOOGLE_SIGN_IN_START: "user/GOOGLE_SIGN_IN_START",
  EMAIL_SIGN_IN_START: "user/EMAIL_SIGN_IN_START",
};
```

我们已经将`App.js`里原先涉及到的检查用户的代码抽离，同时，用户无论是创建还是登录，或者通过 Google 登录创建，都会经过`createUserDocumentFromAuth`这个方法。所以再新创建两个 Types，意为登陆成功后启用这个方法。

```js
// user.types.js

export const USER_ACTION_TYPES = {
  SET_CURRENT_USER: "user/SET_CURRENT_USER",
  CHECK_USER_SESSION: "user/CHECK_USER_SESSION",
  GOOGLE_SIGN_IN_START: "user/GOOGLE_SIGN_IN_START",
  EMAIL_SIGN_IN_START: "user/EMAIL_SIGN_IN_START",
  SIGN_IN_SUCCESS: "user/SIGN_IN_SUCCESS",
  SGIN_IN_FAILED: "user/SIGN_IN_FAILED",
};
```

添加新的 actions

```js
// user.actions.js

export const checkUserSession = () =>
  createAction(USER_ACTION_TYPES.CHECK_USER_SESSION);

export const googleSignInStart = () =>
  createAction(USER_ACTION_TYPES.GOOGLE_SIGN_IN_START);

export const emailSignInStart = (email, password) =>
  createAction(USER_ACTION_TYPES.EMAIL_SIGN_IN_START, { email, password });

export const signInSuccess = (user) =>
  createAction(USER_ACTION_TYPES.SIGN_IN_SUCCESS, user);

export const signInFailed = (error) =>
  createAction(USER_ACTION_TYPES.SGIN_IN_FAILED, error);
```

更新 Reducer

```js
// user.reducer.js

const INITIAL_STATE = {
  currentUser: null,
  isLoading: false,
  error: null,
};

export const userReducer = (state = INITIAL_STATE, action) => {
  const { type, payload } = action;

  switch (type) {
    case USER_ACTION_TYPES.SIGN_IN_SUCCESS:
      return { ...state, currentUser: payload };
    case USER_ACTION_TYPES.SGIN_IN_FAILED:
      return { ...state, error: payload };
    default:
      return state;
  }
};
```

`INITIAL_STATE`添加 isLoading 和 error，并更新 Reducer。
新建`user.saga.js`

```js
// user.saga.js

export function* isUserAuthenticated() {
  try {
    const userAuth = yield call(getCurrentUser);
    if (!userAuth) return;
  } catch (error) {
    yield put(signInFailed(error));
  }
}

export function* onCheckUserSession() {
  yield takeLatest(USER_ACTION_TYPES.CHECK_USER_SESSION, isUserAuthenticated);
}

export function* userSagas() {
  yield all([call(onCheckUserSession)]);
}
```

首先建立`userSagas`，然后是针对`USER_ACTION_TYPES.CHECK_USER_SESSION`作出处理的`onCheckUserSession`，`isUserAuthenticated`则是相对应的处理函数。

之前我们在`App.js`里，拿到了`user`以后调用了`createUserDocumentFromAuth`，用于获取用户具体的信息。

```jsx
// App.js

useEffect(() => {
  const unsubscribe = onAuthStateChangedListener((user) => {
    if (user) {
      createUserDocumentFromAuth(user);
    }
    dispatch(setCurrentUser(user));
  });

  return unsubscribe;
}, []);
```

现在，在`isUserAuthenticated`内我们拿到了`userAuth`，所以下一步就是建一个 Saga 函数，来调用`createUserDocumentFromAuth`。

```js
// user.saga.js

export function* getSnapshotFromUserAuth(userAuth, additionalDetails) {
  try {
    const userSnapshot = yield call(
      createUserDocumentFromAuth,
      userAuth,
      additionalDetails
    );
    yield put(signInSuccess({ id: userSnapshot.id, ...userSnapshot }));
  } catch (error) {
    yield put(signInFailed(error));
  }
}
```

拿到 `userSnapShot` 后，dispatch(put) `signInSuccess` 的 action 来设置 store 存储的用户信息。

然后再在`isUserAuthenticated`调用它。

```js
// user.saga.js

export function* isUserAuthenticated() {
  try {
    const userAuth = yield call(getCurrentUser);
    if (!userAuth) return;
    yield call(getSnapshotFromUserAuth, userAuth);
  } catch (error) {
    yield put(signInFailed(error));
  }
}
```

这个时候在 Root Saga 添加上`userSagas`。

```js
// root-saga.js

export function* rootSaga() {
  yield all([call(categoriesSaga), call(userSagas)]);
}
```

最后回到`App.js`修改即可。

```js
// App.js

useEffect(() => {
  dispatch(checkUserSession());
}, []);
```

整个流程如下：

1. `App.js`发出`checkUserSession`action。
2. `onCheckUserSession`捕获 action，调用`isUserAuthenticated
3. 获取`userAuth`，凭此调用`getSnapshotFromUserAuth`
4. `getSnapshotFromUserAuth`调用`createUserDocumentFromAuth`后拿到`userSnapshot`,凭此 dispatch`signInSuccess`action，设置当前 Redux 内储存的用户信息。

## Redux-Saga\_ Sign in Sagas

将登录的异步操作改为 Saga，包括 Google 登录和 Email 登录。

首先是 Google 登录。

```js
// user.saga.js

export function* signInWithGoogle() {
  try {
    const { user } = yield call(signInWithGooglePopup);
    yield call(getSnapshotFromUserAuth, user);
  } catch (error) {
    yield put(signInFailed(error));
  }
}

export function* onGoogleSignInStart() {
  yield takeLatest(USER_ACTION_TYPES.GOOGLE_SIGN_IN_START, signInWithGoogle);
}

export function* userSagas() {
  yield all([call(onCheckUserSession), call(onGoogleSignInStart)]);
}
```

```js
// sign-in-form.component.jsx

const dispatch = useDispatch();
// ...
const signInWithGoogle = async () => {
  dispatch(googleSignInStart());
};
```

然后在`sign-in-form.component.jsx`里将登录改为 dispatch 对应的 Action。

接下来是 Email。类似的步骤

```js
// user.saga.js

export function* signInWithEmail({ payload: { email, password } }) {
  try {
    const { user } = yield call(
      signInAuthUserWithEmailAndPassword,
      email,
      password
    );
    yield call(getSnapshotFromUserAuth, user);
  } catch (error) {
    yield put(signInFailed(error));
  }
}
export function* onEmailSignInStart() {
  yield takeLatest(USER_ACTION_TYPES.EMAIL_SIGN_IN_START, signInWithEmail);
}

export function* userSagas() {
  yield all([
    call(onCheckUserSession),
    call(onGoogleSignInStart),
    call(onEmailSignInStart),
  ]);
}
```

需要注意的是，`signInWithEmail`里的被传入参数为 action，这个 action 里面包含了 payload，所以使用解构获取 email 和 password。

```js
// sign-in-form.component.jsx

const handleSubmit = async (event) => {
  event.preventDefault();
  try {
    dispatch(emailSignInStart(email, password));
    resetFormFields();
  } catch (error) {
    console.log("user sign in failed", error);
  }
};
```

## Redux-Saga\_ Sign up Sagas

现在设置 Sign up Sagas，第一步回看原先的注册流程。

```js
// sign-up-form.component.jsx

const handleSubmit = async (event) => {
  event.preventDefault();

  if (password !== confirmPassword) return alert("passwords do not match");

  try {
    const { user } = await createAuthUserWithEmailAndPassword(email, password);
    await createUserDocumentFromAuth(user, { displayName });
    resetFormFields();
  } catch (error) {
    // ...
  }
};
```

首先确认两次密码相同后，调用`createAuthUserWithEmailAndPassword`创建用户，然后调用`createUserDocumentFromAuth`在数据库创建用户档案。但是在`createUserDocumentFromAuth`内，是会处理登录和注册两种情况的，是合并的。所以要作出区分，用不一样的 Saga。

先新建几个 Action Types 和 Actions

```js
// user.types.js

export const USER_ACTION_TYPES = {
  //...
  SIGN_UP_START: "user/SIGN_UP_START",
  SIGN_UP_SUCCESS: "user/SIGN_UP_SUCCESS",
  SIGN_UP_FAILED: "user/SIGN_UP_FAILED",
};
```

```js
// user.action.js

export const signUpStart = (email, password, displayName) =>
  createAction(USER_ACTION_TYPES.SIGN_UP_START, {
    email,
    password,
    displayName,
  });

export const signUpSuccess = (user, additionalDetails) =>
  createAction(USER_ACTION_TYPES.SIGN_UP_SUCCESS, { user, additionalDetails });

export const signUpFailed = (error) =>
  createAction(USER_ACTION_TYPES.SIGN_UP_FAILED, error);
```

然后就开始写 Saga 了

```js
// user.saga.js

export function* signUp({ payload: { email, password, displayName } }) {
  try {
    const { user } = yield call(
      createAuthUserWithEmailAndPassword,
      email,
      password
    );
    yield put(signUpSuccess(user, { displayName }));
  } catch (error) {
    yield put(signUpFailed(error));
  }
}

export function* signInAfterSignUp({ payload: { user, additionalDetails } }) {
  yield call(getSnapshotFromUserAuth, user, additionalDetails);
}

export function* onSignUpStart() {
  yield takeLatest(USER_ACTION_TYPES.SIGN_UP_START, signUp);
}

export function* onSignUpSuccess() {
  yield takeLatest(USER_ACTION_TYPES.SIGN_UP_SUCCESS, signInAfterSignUp);
}

export function* userSagas() {
  yield all([
    call(onCheckUserSession),
    call(onGoogleSignInStart),
    call(onEmailSignInStart),
    call(onSignUpStart),
    call(onSignUpSuccess),
  ]);
}
```

流程如下：

1. 捕获`USER_ACTION_TYPES.SIGN_UP_START`，调用`signUp`
2. `signUp`成功，其内部 put 新的 action`signUpSuccess`
3. 捕获`USER_ACTION_TYPES.SIGN_UP_SUCCESS`，调用`signInAfterSignUp`，用于注册成功后登录。
4. `signInAfterSignUp`调用`getSnapshotFromUserAuth`登录并获取用户信息。

```js
// sign-up-form.component.jsx

const handleSubmit = async (event) => {
  event.preventDefault();

  if (password !== confirmPassword) return alert("passwords do not match");

  try {
    dispatch(signUpStart(email, password, displayName));
    resetFormFields();
  } catch (error) {
    // ...
  }
};
```

在`sign-up-form.component.jsx`将原先部分删掉，改为 dispatch `SignUpStart`这个 action 即可。

## Redux-Saga\_ Sign out Sagas

Sign out Sagas 步骤和前两个一样。

```js
// user.types.js

export const USER_ACTION_TYPES = {
  //...
  SIGN_OUT_START: "user/SIGN_OUT_START",
  SIGN_OUT_SUCCESS: "user/SIGN_OUT_SUCCESS",
  SIGN_OUT_FAILED: "user/SIGN_OUT_FAILED",
};
```

```js
// user.action.js

export const signOutStart = () =>
  createAction(USER_ACTION_TYPES.SIGN_OUT_START);

export const signOutSuccess = () =>
  createAction(USER_ACTION_TYPES.SIGN_OUT_SUCCESS);

export const signOutFailed = (error) =>
  createAction(USER_ACTION_TYPES.SIGN_OUT_FAILED, error);
```

`reducer`也需要更新，登出后删除用户信息。

```js
// user.reducer.js

// ...
switch (type) {
  // ...
  case USER_ACTION_TYPES.SIGN_OUT_SUCCESS:
    return { ...state, currentUser: null };
  case USER_ACTION_TYPES.SIGN_IN_FAILED:
  case USER_ACTION_TYPES.SIGN_UP_FAILED:
  case USER_ACTION_TYPES.SIGN_OUT_FAILED:
    return { ...state, error: payload };
  default:
    return state;
}
```

然后来写 Saga

```js
// user.saga.js

export function* signOut() {
  try {
    yield call(signOutUser)
    yield put(signOutSuccess())
  } catch (error) {
    yield put(signOutFailed(error))
  }
}

export function onSignOutStart() {
  yield takeLatest(USER_ACTION_TYPES.SIGN_OUT_START,signOut)
}

export function* userSagas() {
  yield all([
    call(onCheckUserSession),
    call(onGoogleSignInStart),
    call(onEmailSignInStart),
    call(onSignUpStart),
    call(onSignUpSuccess),
    call(onSignOutStart)
  ]);
}
```

现在去到`navigation.component.jsx`里删掉原来`firebase.utils.js`的`signOutUser`，并改为 dispatch 一个`signOutStart`Action。

```jsx
// navigation.component.jsx

const dispatch = useDispatch();

const signOutUser = () => dispatch(signOutStart());
```
