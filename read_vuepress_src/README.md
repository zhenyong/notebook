# Vuepress 源码阅读

# db62430a7ce2e5dc7f85594e94b22c861e32f002

## 目标

- 使用 Vue
- 文档（md）优先
- 较好兼容 gitbook，方便迁移
- Github 友好，基于相对路径的 .md 文件，彼此导航
- 附加的东西（Layout），尽可能使用 Vue/JS 去实现

## 依赖库

- css-loader 拦截处理 @import 和 url()
- file-loader 解析 import/require()，把对应文件映射成 url 并且生成文件到输出目录
- globby 路径/文件 glob 匹配
- markdown-it 实现 CommonMark 标准的 md 解析器
- mini-css-extract-plugin 抽取 css，一个 JS 对应一份 css，相较之前的 extract-text-webpack-plugin，支持异步、按需、不重复变异、配置简单
- url-loader 把某些url静态资源转化为 data:base64
- vue
- vue-loader
- vue-markdown-loade 把 md 转为 vue 组件
- vue-server-renderer
- vue-template-compiler 预编译 Vue 2.0 模板，转化为 render 方法
- webpack
- webpack-chain 提供链式 api 来生成/修改 webpack 配置

## 流程

1. 加载配置
2. 生成动态组件注册文件
3. 生成路由
4. build

#2动态组件注册文件，扫描文档目录下的 *.vue 文件，生成

    import Vue from 'vue';
    Vue.component('demo-1', () =>
      import('/Users/zhenyong/github/vuepress/docs/_components/demo-1.vue')
    );

## webpack 配置

### baseConfig.js

    config
        .set('mode', isProd ? 'production' : 'development')
        .output
          .path(path.resolve(sourceDir, 'dist'))
          .filename('[name].[chunkhash].js')
          .publicPath(publicPath)

- 设置输出目录为文档目录下 /dist
- publicPath 为根目录 `/`

    config.alias
        .set('~layout', layoutPath)

- ~layout 对应布局组件所在的目录

    config.resolve
        .set('symlinks', true)
        .extensions
          .merge(['.js', '.jsx', '.vue', '.json'])
          .end()
        .modules
          .add('node_modules')
          .add(path.resolve(sourceDir, '../node_modules'))
          .add(path.resolve(__dirname, '../../node_modules'))

- `resolve.symlinks` 解析 symlinks 到他们具体路径
- `resolve.modules`  还支持哪些目录寻找代码模块，默认 ['node_modules']

    config.resolveLoader
        .set('symlinks', true)
        .modules
          .add('node_modules')
          .add(path.resolve(sourceDir, '../node_modules'))
          .add(path.resolve(__dirname, '../../node_modules'))

- `resolveLoader` 搜索 webpack loader 使用的目录，默认 ['node_modules']

    

- `module.noParse`  禁止 webpack 处理这些模块

    config.module
        .rule('images')
          .test(/\.(png|jpe?g|gif)(\?.*)?$/)
          .use('url-loader')
            .loader('url-loader')
            .options({
              limit: 10000,
              name: `img/[name].[hash:8].[ext]`
            })

    // do not base64-inline SVGs.
      // https://github.com/facebookincubator/create-react-app/pull/1180
      config.module
        .rule('svg')
          .test(/\.(svg)(\?.*)?$/)
          .use('file-loader')
            .loader('file-loader')
            .options({
              name: `img/[name].[hash:8].[ext]`
            })

- svg 不用 base64，因为 SVG Fragment（雪碧图系统） 不支持 inline base64

    config.module
        .rule('fonts')
          .test(/\.(woff2?|eot|ttf|otf)(\?.*)?$/i)
          .use('url-loader')
            .loader('url-loader')
            .options({
              limit: inlineLimit,
              name: `fonts/[name].[hash:8].[ext]`
            })

    createCSSRule(config, 'css', /\.css$/)
      createCSSRule(config, 'scss', /\.scss$/, 'sass-loader')
      createCSSRule(config, 'sass', /\.sass$/, 'sass-loader', { indentedSyntax: true })
      createCSSRule(config, 'less', /\.less$/, 'less-loader')
      createCSSRule(config, 'stylus', /\.styl(us)?$/, 'stylus-loader')

- indentedSyntax 是支持 .sass 文件

    config
        .plugin('vue-loader')
        .use(VueLoaderPlugin)

    if (isProd) {
        config
          .plugin('extract-css')
          .use(CSSExtractPlugin, [{ filename: 'styles.css' }])
      }

- 生产模式，抽出样式文件

### clientConfig

    config
        .entry('app')
          .add(path.resolve(__dirname, '../app/clientEntry.js'))

    config.node
        .merge({
          // prevent webpack from injecting useless setImmediate polyfill because Vue
          // source contains it (although only uses it if it's native).
          setImmediate: false,
          global: false,
          // process is injected via DefinePlugin, although some 3rd party
          // libraries may require a mock to work properly (#934)
          process: 'mock',
          // prevent webpack from injecting mocks to Node native modules
          // that does not make sense for the client
          dgram: 'empty',
          fs: 'empty',
          net: 'empty',
          tls: 'empty',
          child_process: 'empty'
        })

- 运行在 Node 中的代码，可能要在别的环境执行，这时候需要把 Node 中的一些方法变量处理
- Node 中 global 是全局命名空间对象，在浏览器端运营的代码，就好替换掉 `global` 为 window

### serverConfig

    config
        .target('node')
        .externals(nodeExternals({
          whitelist: [/\.css$/, /\?vue&type=style/]
        }))

防止将某些 import 的包(package)打包到 bundle 中，而是在运行时(runtime)再去从外部获取这些扩展依赖，webpack-node-externals 能够排除 node_modules 目录中所有模块

    config.output
        .filename('server-bundle.js')
        .libraryTarget('commonjs2')

commonjs 是 exports 的标准，没有具体语言实现的意思，Node 实现了这个标准，叫 commonjs2

## App

    // expose Vue Press data
    const g = typeof window !== 'undefined' ? window : global
    const $site = Vue.prototype.$site = g.VUE_PRESS_DATA
    
    Vue.mixin({
      computed: {
        $page () {
          return $site.pages[this.$route.path]

全局可访问整个静态网站的元信息 $site，当前（路由）页面信息 $page 

### clientEntry

router 准备好直接 mount

### serverEntry

vue ssr，得看下文档？？

# 4cbb6b61151245ef3b39233c8880d2bd950be997

## 依赖库

- noprogress 页面顶部进度条
- rimraf 就是 `rm -rf`

## 代码

    // vuepress.js
    build(path.resolve(__dirname, '../docs')).catch(err => {
      console.log(err)
    })

目测 build 返回 Promse 了

之前的 /docs/layout/layout.vue 用来文档布局，改成 /docs/theme/App.vue

    // app/app.js
    Vue.component('Content', {
      functional: true,
      render (h, { parent }) {
        return h('page-' + parent.$page.name)
      }
    })

实现了全局组件 <Content/> 会渲染为当前页面的内容

    // defaulut-theme/App.vue
    路由的开始和结束，分别是用 noprogress

## 主流程

    // build 方法 
    实现了 #4 build，生成 webpack clientConfig 调用
    
    #3 生成注册表的方式，增加扫描 *.md 文件，注册为组件
    
    #2 生成路由映射，扫描 *.md，每个对应一个路由，路由组件统一是 Layout（App.vue）

### webpack 配置

    // baseConfig
    config.module
        .rule('markdown')
          .test(/\.md$/)
          .use('vue-loader')
            .loader('vue-loader')
            .options({
              compilerOptions: {
                preserveWhitespace: false
              }
            })
            .end()
          .use('markdown-to-vue-loader')
            .loader('markdown-to-vue-loader')

use 就是按照从右到左（下到上）的顺序执行 loader

    // baseConfig
    function createCSSRule (lang, test, loader, options) {

对于每种 css 处理器，它的后置 loader 都一样，所以封装一个设置 css 后续 loader 的逻辑

# 890f929667b091dd2384c521664cd9762be5aba7

## 依赖库

- yaml-front-matter 解析文章 `--—` 开头结尾部分的 yaml / json
- chalk 控制终端文字样式
- wepack-server  相当于 webpack-plugin-serve 的命令行工具，启动 webpack 开发服务器
- koa-connect 在 Koa 中使用 Express 中间件
- connect-history-api-fallback 针对使用 history 路由的单页，后台返回 index.html
- html-webpack-plugin 创建 html 文件包含 webpack 输出的文件

## 代码

    // app.js
    -const $site = Vue.prototype.$site = g.VUEPRESS_DATA
    +const $site = g.VUEPRESS_DATA
     
     Vue.mixin({
       computed: {
    +    $site () {
    +      return $site
    +    },

目测 computed 的好处就是，计算属性是一个方法，每次执行拿到最新，而 prototype 赋值那就是一次性的

    -    return h('page-' + parent.$page.name)
    +    return h(parent.$page.componentName)

把潜规则封装在底层，外层调用者不要操这个心

    // App.vue
    -    <Index v-if="$page.isIndex" />
    +    <Index v-if="$page.path === '/index'" />

isIndex 相当于一个中间的计算属性，多一次主观计算，多一次复杂

## 总结

至此已经可以跑起来了，整体思路就是把每个 md 文件当做一个路由页面，md 文件里面也可以写 vue，然后 vue 组件注册为全局组件，配合一些布局组件渲染。
