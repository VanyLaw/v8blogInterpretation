# react 18 新 feature #

1. Concurrency opt-in
- 可以直接用legacy root，18 的实际做法[5]是引入了新的 Root API ReactDOM.createRoot 来与旧的 ReactDOM.render API 区分开来，你可以将整个 React 树分形成不同的 roots，用旧 API 的 legacy roots 会跑在「legacy mode 传统模式」上（相当于跑在 17 上），用新 API 的 roots 会跑在 "Concurrency opt-in" roots 下。
2. 自动batching
- React 17 只在事件回调中 batch，React 18 则会对任何来源的 setState 做尽可能多的 batching。
3. 新的startTransition 与 useDeferredValue API
- 允许你将 UI 的一部分标记为「较低的更新优先级」。
4. Suspense SSR
- 传统 SSR 的一个问题是，全量渲染话延迟太高了。而 CM + Suspence 就可以做到用 Suspence boundary 将应用分片，然后以此为单位做流式 SSR。
5. StrictMode
- 在既 double-render 之后加入了 double-effect，进一步帮助你在开发时抓出 CM-unsafe 的代码。
