---
title: Use useReducer & useContext Hooks with TypeScript in React
date: '2022-10-15'
tags: ['TypeScript', 'React']
draft: false
summary: 'In this article we learn how to use useReducer & useContext Hooks with TypeScript in React by coding a example.'
---

# Use useReducer & useContext Hooks with TypeScript in React

## Original Code

```tsx
// Counter.tsx

import { ReactNode, useReducer, ChangeEvent } from 'react'

const initState = { count: 0, text: '' }

const enum REDUCER_ACTION_TYPE {
  INCREMENT,
  DECREMENT,
  NEW_INPUT,
}

type ReducerAction = {
  type: REDUCER_ACTION_TYPE
  payload?: string
}

const reducer = (state: typeof initState, action: ReducerAction): typeof initState => {
  switch (action.type) {
    case REDUCER_ACTION_TYPE.INCREMENT:
      return { ...state, count: state.count + 1 }
    case REDUCER_ACTION_TYPE.DECREMENT:
      return { ...state, count: state.count - 1 }
    case REDUCER_ACTION_TYPE.NEW_INPUT:
      return { ...state, text: action.payload ?? '' }
    default:
      throw new Error()
  }
}

type ChildrenType = {
  children: (num: number) => ReactNode
}

const Counter = ({ children }: ChildrenType) => {
  const [state, dispatch] = useReducer(reducer, initState)

  const increment = () => dispatch({ type: REDUCER_ACTION_TYPE.INCREMENT })
  const decrement = () => dispatch({ type: REDUCER_ACTION_TYPE.DECREMENT })
  const handleTextInput = (e: ChangeEvent<HTMLInputElement>) => {
    dispatch({
      type: REDUCER_ACTION_TYPE.NEW_INPUT,
      payload: e.target.value,
    })
  }

  return (
    <>
      <h1>{children(state.count)}</h1>
      <div>
        <button onClick={increment}>+</button>
        <button onClick={decrement}>-</button>
      </div>
      <input type="text" onChange={handleTextInput} />
      <h2>{state.text}</h2>
    </>
  )
}
export default Counter
```

```tsx
// App.tsx

import Counter from './Counter'

function App() {
  return (
    <>
      <Counter>{(num: number) => <>Current Count: {num}</>}</Counter>
    </>
  )
}

export default App
```

现在 Counter 使用的是 Reducer 的形式，接下来将其改为 Context+Reducer 的形式

## useReducer

首先，新建一个`context/CounterContext.tsx`，将`Countet.tsx`内与 Reducer 的文件迁移至此。

```tsx
// CounterContext.tsx

import { useReducer, ChangeEvent, useCallback } from 'react'

const initState = { count: 0, text: '' }

const enum REDUCER_ACTION_TYPE {
  INCREMENT,
  DECREMENT,
  NEW_INPUT,
}

type ReducerAction = {
  type: REDUCER_ACTION_TYPE
  payload?: string
}

const reducer = (state: typeof initState, action: ReducerAction): typeof initState => {
  switch (action.type) {
    case REDUCER_ACTION_TYPE.INCREMENT:
      return { ...state, count: state.count + 1 }
    case REDUCER_ACTION_TYPE.DECREMENT:
      return { ...state, count: state.count - 1 }
    case REDUCER_ACTION_TYPE.NEW_INPUT:
      return { ...state, text: action.payload ?? '' }
    default:
      throw new Error()
  }
}
```

然后我们新建一个 `useCounterContext`的 hook，里面存放与`useReducer`、`dispatch`的代码，并将其返回。

```tsx
// CounterContext.tsx

const useCounterContext = (initState: StateType) => {
  const [state, dispatch] = useReducer(reducer, initState)

  const increment = useCallback(() => dispatch({ type: REDUCER_ACTION_TYPE.INCREMENT }), [])
  const decrement = useCallback(() => dispatch({ type: REDUCER_ACTION_TYPE.DECREMENT }), [])
  const handleTextInput = useCallback((e: ChangeEvent<HTMLInputElement>) => {
    dispatch({
      type: REDUCER_ACTION_TYPE.NEW_INPUT,
      payload: e.target.value,
    })
  }, [])

  return { state, increment, decrement, handleTextInput }
}
```

接下来就是创建 Context 及其 Provider 了

```tsx
// CounterContext.tsx

type UseCounterContextType = ReturnType<typeof useCounterContext>

const initContextState: UseCounterContextType = {
  state: initState,
  increment: () => {},
  decrement: () => {},
  handleTextInput: (e: ChangeEvent<HTMLInputElement>) => {},
}

export const CounterContext = createContext<UseCounterContextType>(initContextState)

type ChildrenType = { children?: ReactElement | undefined }

export const CounterProvider = ({
  children,
  ...initState
}: ChildrenType & StateType): ReactElement => (
  <CounterContext.Provider value={useCounterContext(initState)}>{children}</CounterContext.Provider>
)
```

这个时候将 Provider 加入到`App.tsx`

```tsx
// App.tsx

import Counter from './Counter'
import { CounterProvider, initState } from './context/CounterContext'

function App() {
  return (
    <>
      <CounterProvider count={initState.count} text={initState.text}>
        <Counter>{(num: number) => <>Current Count: {num}</>}</Counter>
      </CounterProvider>
    </>
  )
}

export default App
```

为了能更好的使用 Context 的内容，选择新建 hooks 的方式。可以把要分发的东西做出更细致的区分。增强可读性

```tsx
//CounterContext.tsx

type UseCounterHookType = {
  count: number
  increment: () => void
  decrement: () => void
}

export const useCounter = (): UseCounterHookType => {
  const {
    state: { count },
    increment,
    decrement,
  } = useContext(CounterContext)

  return { count, increment, decrement }
}

type UseCounterTextHookType = {
  text: string
  handleTextInput: (e: ChangeEvent<HTMLInputElement>) => void
}

export const useCounterText = (): UseCounterTextHookType => {
  const {
    state: { text },
    handleTextInput,
  } = useContext(CounterContext)
  return { text, handleTextInput }
}
```

最后回到`Counter.tsx`，就可以通过 hooks，使用 Context 了

```tsx
import { ReactNode } from 'react'
import { useCounter, useCounterText } from './context/CounterContext'

type ChildrenType = {
  children: (num: number) => ReactNode
}

const Counter = ({ children }: ChildrenType) => {
  const { count, increment, decrement } = useCounter()
  const { text, handleTextInput } = useCounterText()

  return (
    <>
      <h1>{children(count)}</h1>
      <div>
        <button onClick={increment}>+</button>
        <button onClick={decrement}>-</button>
      </div>
      <input type="text" onChange={handleTextInput} />
      <h2>{text}</h2>
    </>
  )
}
export default Counter
```
