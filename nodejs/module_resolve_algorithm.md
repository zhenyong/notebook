> [Modules: CommonJS modules | Node.js v18.2.0 Documentation](https://nodejs.org/api/modules.html)
> [Modules: Packages | Node.js v18.2.0 Documentation](https://nodejs.org/api/packages.html)

在路径 Y 的模块内 require(X):
1. 如果 X 是内置模块,
   a. 返回内置模块
   b. 结束
2. 如果 X 以 '/' 开头
   a. 设置 Y = 根目录
3. 如果 X 以 './' 或者 '/' 或者 '../' 开头
   a. LOAD_AS_FILE(Y + X)
   b. LOAD_AS_DIRECTORY(Y + X)
   c. 抛出错误 "not found"
4. 如果 X 以 '#' 开头
   a. LOAD_PACKAGE_IMPORTS(X, dirname(Y))
5. LOAD_PACKAGE_SELF(X, dirname(Y))
6. LOAD_NODE_MODULES(X, dirname(Y))
7. THROW "not found"

LOAD_AS_FILE(X)
1. 如果 X 是一个文件，以它本身的文件格式加载。结束
2. 如果 X.js 是一个文件，加载 X.js 作为 JS 代码。结束
3. 如果 X.json 是一个文件，解析 X.json 为 JS 对象。结束
4. 如果 X.node是一个文件，则将X.node作为二进制插件加载。 结束

LOAD_INDEX(X)
1. 如果 X/index.js 是一个文件, 加载 X/index.js 作为 JS 代码。结束
2. 如果 X/index.json 是一个文件，解析 X/index.json 为 JS 对象。结束
3. 如果 X/index.node是一个文件，则将X/index.node作为二进制插件加载。 结束

LOAD_AS_DIRECTORY(X)
1. 如果有 X/package.json 文件
   a. 解析 X/package.json, 找到 "main" 字段。
   b. 如果 "main" 是一个布尔假的值, GOTO 2.
   c. let M = X + (main 字段)
   d. LOAD_AS_FILE(M)
   e. LOAD_INDEX(M)
   f. 抛错误 "not found"
2. LOAD_INDEX(X)

LOAD_NODE_MODULES(X, START)
1. let DIRS = NODE_MODULES_PATHS(START)
2. for each DIR in DIRS:
   a. LOAD_PACKAGE_EXPORTS(X, DIR)
   b. LOAD_AS_FILE(DIR/X)
   c. LOAD_AS_DIRECTORY(DIR/X)

// 从一个目录开始从下往上寻找所有存在 node_modules 子目录的情况，
// 优先级：从近到远的所有 node_modules 目录，加上全局 node_modules 目录
NODE_MODULES_PATHS(START)
1. let PARTS = path split(START)
2. let I = count of PARTS - 1
3. let DIRS = []
4. while I >= 0,
   1. if PARTS[I] = "node_modules" CONTINUE
   2. DIR = path join(PARTS[0 .. I] + "node_modules")
   3. DIRS = DIR + DIRS
   4. let I = I - 1
5. return DIRS + GLOBAL_FOLDERS

LOAD_PACKAGE_IMPORTS(X, DIR)
1. 从目录 DIR 开始找到最近的包目录 SCOPE
2. 如果没找到, return.
3. 如果 SCOPE/package.json 的 "imports" 是空，return。
4. let MATCH = PACKAGE_IMPORTS_RESOLVE(X, pathToFileURL(SCOPE),
  ["node", "require"]) ESM 包解析器定义的.
5. RESOLVE_ESM_MATCH(MATCH).

LOAD_PACKAGE_EXPORTS(X, DIR)
1. 尝试把 X 当作是 NAME/子路径，其中 NAME 是带有 @scope/ 前缀的，子路径就是后面 / 开头的。
2. 如果 X 匹配不到这种模式，或者 DIR/NAME/package.json 不存在，
   return。
3. 解析 DIR/NAME/package.json，找到 "exports" 字段。
4. 如果 "exports" 空, return。
5. let MATCH = PACKAGE_EXPORTS_RESOLVE(pathToFileURL(DIR/NAME), "." + SUBPATH,
   `package.json` "exports", ["node", "require"]) ESM 包解析器定义的。
6. RESOLVE_ESM_MATCH(MATCH)

LOAD_PACKAGE_SELF(X, DIR)
1. 从目录 DIR 开始找到最近的包目录 SCOPE
2. 如果没找到, return.
3. 如果 SCOPE/package.json 的 "exports" 是空，return。
4. 如果 SCOPE/package.json 的 "name" 不是 X 的第一段路径部分，return。
5. let MATCH = PACKAGE_EXPORTS_RESOLVE(pathToFileURL(SCOPE),
   "." + X.slice("name".length), `package.json` "exports", ["node", "require"])，ESM 包解析器定义的参数.
6. RESOLVE_ESM_MATCH(MATCH)

RESOLVE_ESM_MATCH(MATCH)
1. let { RESOLVED, EXACT } = MATCH
2. let RESOLVED_PATH = fileURLToPath(RESOLVED)
3. 如果 EXACT 为 true,
   a. 如果 RESOLVED_PATH 路径对应文件存在, 按照文件本身格式加载。结束
4. 否则
   a. LOAD_AS_FILE(RESOLVED_PATH)
   b. LOAD_AS_DIRECTORY(RESOLVED_PATH)
5. 抛出错误 "not found"
