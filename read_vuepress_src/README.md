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

至此已经可以跑起来了，整体思路就是把每个 md 文件当做一个路由页面，md 文件里面也可以写 vue，然后单个 vue 文件注册为全局组件，布局主题也是一些 vue 文件。

# 8a9d88bae97cb103007e1819119f9d1d70a1e6a8

## package.json

    "bin": {
        "vuepress": "bin/vuepress.js"
      },

安装 Node 项目的时候，根据项目的 package.json 的 bin 配置，在系统相应的 xxx/bin 目录建立以 [key] 为名的软连接，全局安装可能就 /user/local/bin/，本地项目安装就 .node_modules/.bin/ 。当然 bin/vuepress.js 文件第一行要注释 **`#!/usr/bin/envnode`**   表明，使用 node 来执行

### 依赖库

- autoprefixer 解析 css 添加一些私有前缀
- buble 把 ES2015/16 的代码转化
- buble-loader
- mkdirp 就是 mkdir -p
- postcss-loader

## 改动

### app.js

- $site 数据 区分环境  `inBrowser ? window.VUEPRESS_DATA : this.$ssrContext.siteData`
- app.js 提供 createApp 工厂，给出 app Vue 实例 和 router

### Content.js

原先只是直接渲染文档页面对应的组件 h(parent.$page.componentName)，考虑同构情况的页面 title 还有国际化语言，所以从文档页面对应 frontmatter 拿到 title 还有 lang，在 created hook 设置到 $ssrContext，在 mounted hook 设置到 document.title 和 documentElement.lang

### index.ssr.html

用到了 vue-ssr-render 提供的模板内置占位

    <head>
        <meta charset="utf-8">
        <title>{{ title }}</title>
        {{{ userHeadTags }}}
        {{{ renderResourceHints() }}}
        {{{ renderStyles() }}}
        <script>
          window.VUEPRESS_DATA = {{{ JSON.stringify(siteData) }}}
        </script>
      </head>

- userHeadTags 调用 renderToString 传入的
- renderResourceHints() 此方法返回当前要渲染的面，所需的 `<link rel="preload/prefetch">` 资源，包括 preload 所需的 JavaScript 和 CSS 文件、prefetch 异步 JavaScript chunk，之后可能会用于渲染
- renderStyles() 返回内联 <style> 标签包含的 critical CSS （在要用到的 *.vue 组件的渲染过程中收集）
- renderScripts() 所需的 <script> 标签。当应用使用异步代码分割 (async code-splitting) 时，将智能推断需要引入的异步 chunk

## build.js

    const renderer = createBundleRenderer(serverBundle, {
        clientManifest,
        runInNewContext: false,
        shouldPrefetch: () => false,
        inject: false,
        template: fs.readFileSync(path.resolve(__dirname, 'app/index.ssr.html'), 'utf-8')
      })

- shouldPrefetch 控制哪些文件要 prefetch
- inject 控制使用 template 时是否执行自动注入

## SiteDataPlugin.js

ssr 的模板通过 vue-ssr-render 相关机制可以自定义输出模板，开发阶段的页面不走 ssr，那么就要自定义输出，直接在 html-webpack-plugin 的 hook，增加一些资源

## 总结

满足 ssr 的场景，把同构数据做多端处理，改造使用 createBundleRenderer。

# ad4aef05f3dc3e20f022f564ff0619509bfb27b4

- app.js mixin 的 $page 返回数据，定位具体 page 组件的逻辑放在 <Content>
- 重构，输出页面路径不用区分首页文档还是其他页面

# ef9715f51afe2d8bc49e6661ef14bdb47d063097

## 依赖库

- chokidar 包装 node.js [fs.watch](http://fs.watch) 并解决一些问题

## 改动

## prepare.js

原先不端提供 site data 的方式不一样，client 通过 <script 设置，server 通过 $ssrContext 设置，然后在计算属性 $site 里面判断环境，现在把 siteData 写到临时文件，直接在计算属性返回具体数据

pages 的原始信息使用数组存储，原先的 <url, pageObj> 在构建时不够灵活

# 79b9939fb348862a316d890dc42c278f203b5e6b

## 改动

### Content.js

页面的 meta 信息优先级（同 key）： 文档的 yaml > $page data >  $site data

默认 title 是 md 的中一级标题

# 6471d4f9db3345a4c3177584afc4fcbd7bb511d8

## 依赖库

- prismjs 语法高亮

## 改动

新增 markdown loader，使用 prismjs 作为语法高亮

# 72e5666161675e22c74ebcbcd5034534bdea9f58

## 改动

- 用配置文件 vuepress.config.js 兜底诸如 title 配置
- 页面标题："siteTitle | pageTitle"
- markdown/** 将 md 中的 md|html 链接转为 <rouuter-link>
- prepare.js 包含自定义 _theme 时设置各种属性（siteConfig、themePath、notFoundPath...）

# 936577df9202e00ad8150946a5ec0bd63a283062

meta 数据 mixin 单独文件

# 9bd8a11d62aea1bb71d8415fc820fed921fc33b0

## 依赖库

- loader-utils webpack loader 工具方法库
- markdown-it-anchor markdown-it 插件，给生成的标题标签如 <h1> 增加锚点（id 属性）

## 改动

### markdown/link.js

如果是 http 开头链接，作为外部链接，补充 target="_blank"，其余 md|html 文件则转为 <router-link>

# bd3571aeb3ee46271a976a24c9fab264a428051c

依赖库

- markdown-it-table-of-contents Markdown-it 插件，生成标题导航

# 9c83d5c758af93f20d6dbdf60b44a8f8a39c2ace

自定义的主题什么，都放到 .vuepress 目录下

# f472a47641c2715dd0d8707cd6116c2a0cd98d1b

## 依赖库

- copy-webpack-plugin webpack 中拷贝文件
- koa-static

## 改动

- 静态目录 .vuepress/public
- 输出目录可配置

# 8d3bcba03784a205eca0e5d0b8a46372c62874eb

提供 everygreen 配置，如果 false 则使用轻量 buble 来兼容老浏览器

# 4e33f5afa9bd79c7fb0a9af57d52cd2bdb2de617

commander 支持命令行提示、参数

# de457da22beef048599f3728f627e112744764fe

## 依赖库

- eslint
- eslint-plugin-vue-libs 开发 Vue 本身的内部 plugin/配置，不是针对使用 vue 的应用开发
- lint-staged 对暂存文件进行 lint
- yorkie 优化了 husky 一些小问题，使用 git hook

# 2a6fe420dc6fbd25a48e0fc6d21c53ff739196c8

## 依赖库

- 移除了 wehpack-node-external 的依赖，在 bubble 配置里 exclude node_module 了

增加一些 lint scripts，重构格式化代码

# b5c6e9eda2d9a341b50590ab740b2467fc60caa8

重构 siteConfig 配置项

# 1d726c57d79033598aae821f868b1ad6d22d3760

build 增加 -debug 参数，诸如加入 sourcemap

# ea4b85b9d68a46ca78a29968fef681d7695a6b6c

markdown 代码高亮，支持指定行数

# e242f596dfbfcf43ba634c76802e861d8fdecb4a

# 4bde115c9eeefe81d10f111d7898622779b4f2ee

## 依赖库

- markdown-it-emoji

## 改动

- 使用 markdown-it 增加  v-pre  container

# c05448d50ce31f69a7f719e378da2d6ded5477a7

## 改动

- 修改 markdown-it 对于 html 块（标签）的识别，用来支持 vue 组件

# f97e67664a8db3fde4a87c72ee96d357b7547863

## 改动

针对文件名带有 module 的样式文件，对应 loader 进行 css module 处理

    rule.use('css-loader').loader('css-loader').options({
            modules,
            minimize: isProd,
            localIdentName: `[local]_[hash:base64:8]`
          })

[name]表示标签名，[local]表示类名

自定义的 markdown-loader ，把 <script> <style> 收集起来了，转成 vue 文时，把收集的放到 < <template> 后面，可以理解成经过 markdown-loader转化后，文件长这样:

    <template>
    md 转化的到的 html
    </template>
    
    <script></script>
    <script></script>
    <style><style>

# ce86da7e438de65ad4b14f9e41dc6b44b297ea3e

css 抽取一个文件

# d9ac270b4c09d25cd3b80013672ead648517df8c

# 701658a4f88ac03e71a9b5b0b5765ca06a8374a8

支持 front 配置对应到 <head> 里面的 <meta>，例如 description

# dad75951d96d968c6fcd002c486304d849a7e1b8

约定主题文件寻找路径

# 1e74c9952cf0721b723a7b536318be21d1ca7c75

# b4f82b7f5e8823e43ce5126ce9e31fbe68287478

配置和frontmatter 修改触发更新，开发阶段的webpack热加载针对源码得修改，frontmatter 在 watch 机制里面，但是不会直接导致生成「vue源码」有变化，所以单独针对 frontmatter 提供一个更新机制

    module.exports = function (src) {
       const { markdown } = getOptions(this)
    -  const content = frontmatter.loadFront(src).__content
    +  const parsed = frontmatter.loadFront(src)
    +  const content = parsed.__content
    +  delete parsed.__content
    +
    +  // diff frontmatter, since it's not going to be part of the returned
    +  // component, changes in frontmatter do not trigger hot updates
    +  const serializedData = JSON.stringify(parsed)
    +  const cachedData = frontmatterCache.get(this.resourcePath)
    +  if (cachedData != null && cachedData !== serializedData) {
    +    // frontmatter changed... need to do a full reload
    +    module.exports.frontmatterEmitter.emit('update')
    +  }
    +  frontmatterCache.set(this.resourcePath, serializedData)
    +
       const { html, hoistedTags } = markdown.renderWithHoisting(content)

markdown-load 有个缓存表，存放 md 路径对应的 frontmatter（字符串）

# 196470994f1190c70c8d44860a9d5bb8280356e1

支持 sidebar 多级

# aef30e549f476707e0f99c50f67c1e3e617bf549

重构出 SidebarGroup 组件

# b50ff521108a26ca15ccde0e92cb603eccaa8759

子目录的 readme 路径默认为 子目录/index

# c7cd345f90d3b9d9f45bf23e29e82df51e34bfa8

前后 nav

# 39061a4907c5984f0a5d0f6b88ca89ec23cf19ad

- 页面组件懒加载
- 识别路由 hash，滚到对应锚点

# 46fcbc788ac1082a4e298ad5fa9065e3a056d681

工具方法:抽取 md 的一些标题

# 33624f310ff2d5d9b06d638a4cc1ed64abdf6bc3

## 依赖库

- optimize-css-assets-webpack-plugin 优化压缩 css 插件，解决其他插件合并重复多余 css 块的问题

## 改动

    rule.use('css-loader').loader('css-loader').options({
    ...
    +        importLoaders: 1 ，在 css-loader 之前应用的 loader 数量
           }

# 664a8e034c401c0b86d8162ae68754c51a01345c

支持 PWA

## 依赖库

- register-service-worker 封装了 service-worker 方法
- workbox-build 生成完整 service-worker 文件

# 9719bc3246afcf77f69eaf94a34f0d6b2fa9c95b

# e08e3d21dbebcbc935fd4fde90e48de1dd31ab96

# 8454c41e730eebd0298cf54a1a8c0a7b3870a949

    
    .loader(isServer ? 'css-loader/locals' : 'css-loader')
    // css-loader/locals不会嵌入 CSS，只导出映射

# dfdc00c8ba5e89ee66a9539bb9b4d4fff639d904

- cache-loader