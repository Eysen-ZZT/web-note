## 小程序生命周期执行顺序就是App.onLaunch -->  App.onShow --> page.onLoad  -->  page.onShow 。为什么还要用Vue.prototype.\$onLaunched来做异步控制

虽然小程序有自身的生命周期函数，如 `App.onLaunch`、`App.onShow` 等，但使用 `Vue.prototype.$onLaunched` 来做异步控制仍有必要，以下从几个方面为你详细解释：

### 1. 小程序生命周期函数的局限性

* **生命周期顺序固定但时机复杂**：小程序的生命周期函数执行顺序虽然是固定的，例如先执行 `App.onLaunch`，再执行 `App.onShow` 等，但在实际开发中，不同的生命周期函数触发时机可能会受到多种因素影响，比如网络状况、设备性能等。在这些生命周期函数中，有些操作可能是异步的，无法保证在某个特定生命周期函数执行完毕后，相关的初始化操作已经全部完成。
* **无法精准控制异步操作完成**：`App.onLaunch` 和 `App.onShow` 主要用于小程序的启动和显示阶段，但它们并不能直接保证所有异步操作（如数据请求、插件初始化等）都已完成。如果在这些生命周期函数之后的代码依赖于这些异步操作的结果，就可能会出现问题。

### 2. Vue.prototype.$onLaunched的优势

* **灵活控制异步操作同步化**：`Vue.prototype.$onLaunched` 是一个自定义的 Promise 对象，通过它可以在代码中精确控制某些代码在特定条件满足后再执行。可以将一些关键的异步初始化操作封装在 `$onLaunched` 的 `resolve` 条件中，确保在这些操作完成后，才会执行依赖于这些操作结果的代码。
  例如，在小程序启动时需要从服务器获取一些配置信息，这个请求是异步的。可以在请求成功后调用 `this.$isResolve()` 来解决 `$onLaunched` 这个 Promise，这样其他依赖于这些配置信息的代码就可以在 `await this.$onLaunched` 之后安全地执行。

```
// 在 App.vue 中
export default {
    onLaunch() {
        // 发起异步请求
        wx.request({
            url: 'https://example.com/api/config',
            success: (res) => {
                // 请求成功后解决 $onLaunched
                this.$isResolve();
            },
            fail: (err) => {
                console.error('获取配置信息失败:', err);
            }
        });
    }
};
```

* **代码解耦和可维护性**：使用 `Vue.prototype.$onLaunched` 可以将异步初始化逻辑和业务逻辑分离，使代码结构更加清晰。不同的组件或模块可以独立地等待 `$onLaunched` 被解决，而不需要关心具体的初始化过程，提高了代码的可维护性和可扩展性。

### 3. 与小程序生命周期结合使用

`Vue.prototype.$onLaunched` 可以与小程序的生命周期函数结合使用，在小程序的生命周期函数中完成一些初始化操作，然后在合适的时机解决 `$onLaunched` 这个 Promise。例如，在 `App.onLaunch` 中发起一些异步请求，当请求成功后解决 `$onLaunched`，这样后续的组件就可以在 `$onLaunched` 解决后开始执行相关逻辑。

```
// 在 App.vue 中
export default {
    onLaunch() {
        // 发起异步请求
        wx.request({
            url: 'https://example.com/api/config',
            success: (res) => {
                // 请求成功后解决 $onLaunched
                this.$isResolve();
            },
            fail: (err) => {
                console.error('获取配置信息失败:', err);
            }
        });
    }
};
```

### 4. 注意事项

* **只能解决一次**：Promise 一旦被解决（无论是 `fulfilled` 还是 `rejected`），其状态就不能再改变。所以 `$isResolve` 只能调用一次，多次调用不会产生额外的效果。
* **错误处理**：如果在使用 `await` 等待 `$onLaunched` 时，应该使用 `try...catch` 块来捕获可能的错误，以确保代码的健壮性。虽然这里通常不会有 `rejected` 情况，但如果后续代码逻辑修改可能会引入异常。
* **作用域问题**：确保在调用 `this.$isResolve` 时，`this` 指向的是 Vue 实例，否则可能无法正确调用到 `$isResolve` 方法。

综上所述，虽然小程序有自己的生命周期函数，但使用 `Vue.prototype.$onLaunched` 可以更灵活、精确地控制异步操作的同步化，提高代码的可维护性和健壮性。
