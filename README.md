# dva-example-user-dashboard

详见[《12 步 30 分钟，完成用户管理的 CURD 应用 (react+dva+antd)》](https://github.com/sorrycc/blog/issues/18)。

---

<p align="center">
  <img src="https://zos.alipayobjects.com/rmsportal/bmkNCEoluwGaeGjYjInf.png" />
</p>

## Getting Started
Install dependencies.

```bash
$ npm install
```

Start server.

```bash
$ npm start
```

If success, app will be open in your default browser automatically.


 ## 项目细节

 ### 项目目录
 详见 ./user-dashboard.jpg
 ### 入口页面
 ```
src\pages\.umi\umi.js  ---ReactDOM.render
```
此页面集成了一个项目的两大要素： dva (状态) 和 路由：
```
src\pages\.umi\DvaContainer.js  ---dva (状态)
src\pages\.umi\router.js ---路由
```
### dva 布局
src\pages\.umi\DvaContainer.js
src\pages\users\models\users.js (reducers effects)
src\pages\users\components\Users\Users.js (connect mapStateToProps dispatch) 【dispatch 由connect集成】

### User.js页面分析
#### subscriptions setup
进入User页面后，首先触发 src\pages\users\models\users.js 下的 ，[原因参考dva文档---异步数据初始化](https://dvajs.com/knowledgemap/#%E5%BC%82%E6%AD%A5%E6%95%B0%E6%8D%AE%E5%88%9D%E5%A7%8B%E5%8C%96):
```
subscriptions: {
    setup({ dispatch, history }) {
      return history.listen(({ pathname, query }) => {
        if (pathname === '/users') {
          dispatch({ type: 'fetch', payload: query });
        }
      });
    },
  },
```
在setup 中触发 改js下的 effects fetch

#### effects fetch
在fetch中首先 usersService.fetch 向后台请求数据；
然后将返回的数据，put触发 reducers save;
```
 *fetch({ payload: { page = 1 } }, { call, put }) {
      const { data, headers } = yield call(usersService.fetch, { page });
      yield put({
        type: 'save',
        payload: {
          data,
          total: parseInt(headers['x-total-count'], 10),
          page: parseInt(page, 10),
        },
      });
    },
```
#### reducers save
通过save reducer忘redux上造数据list，total。。。以后User页面使用。
```
 reducers: {
    save(state, { payload: { data: list, total, page } }) {
      return { ...state, list, total, page };
    },
  },
```

### 细节关注点

#### *fetch 与 yield 的 generateor写法
这里的*和yield是 generateor的写法，可到mdn网查询了解。

#### loading

发fetch请求时，通过dva-loading 配合dva中间件，会自动给redux 的store 改变store.loading的state，
在fetch开始和完成时将store.loading置为true或false：

```
//src\pages\.umi\DvaContainer.js
import createLoading from 'dva-loading';
app.use(createLoading());
```

在页面中，通过mapStateToProps拿到这个redux的state.loading值，根据这个值，自行开启或关闭loading组件或效果：
```
//src\pages\users\components\Users\Users.js
<Table
          columns={columns}
          dataSource={dataSource}
          loading={loading}
          rowKey={record => record.id}
          pagination={false}
        />

function mapStateToProps(state) {
  const { list, total, page } = state.users;
  return {
    loading: state.loading.models.users,
    list,
    total,
    page,
  };
}

export default connect(mapStateToProps)(Users);
```

#### import styles from './index.css' 的运用：

```
import styles from './index.css';
console.log(styles)//{normal: "index__normal___3v60A", content: "index__content___14HDd", main: "index__main___nz_0B"}
 <div className={styles.main}>{children}</div>
 ```
