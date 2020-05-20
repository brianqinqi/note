#### 跨域

- vue版本：2.6.11

- 根目录下新建vue.config.js

  ```javascript
  module.exports = {
    publicPath: './', // npm run build 之后的资源路径默认 ‘/’
    devServer: {
      proxy: {
        '/hehe': {
          target: 'http://v.juhe.cn/', // 目标服务器
          changeOrigin: true, // 是否改变请求源
          pathRewrite: { // 路径重写
            '^/hehe': ''
          }
        }
      }
    }
  }
  ```

- 原axios请求

  ```javascript
  axios.get('http://v.juhe.cn/jztk/query?subject=1&model=c1&key=919791fca1e9f9c1180bd5d0610d0a3c&testType=rand')
  ```

  

- 新axios请求

  ```javascript
  axios.get('hehe/jztk/query?subject=1&model=c1&key=919791fca1e9f9c1180bd5d0610d0a3c&testType=rand')
  ```

  