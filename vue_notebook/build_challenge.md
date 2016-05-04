# 编译困难

看 vue 最开始提交的代码，使用了 grunt + component 配合打包，你可能安装完相应库后发现奇怪的错误， 例如

	Fatal error: failed to lookup "seed"'s dependency "component-emitter"

这是因为 grunt 或者 component 版本不对，找到 vue 提交的日期，安装对应版本的 grunt 和 component （特别是component），参考 https://github.com/componentjs/component/releases?after=0.17.5

	npm install -g component@0.16.3

