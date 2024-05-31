# Vite 配置自动导入 Element Plus 的 Icon 图标

> Element Plus 官方文档对于 Icon 图标的自动导入，描述的很笼统。这里根据实际的操在进行记录

```python
Element Plus 对于图标的自动导入描述如下：

使用 unplugin-icons 和 unplugin-auto-import 从 iconify 中自动导入任何图标集。 您可以参考此模板

参考模板地址：https://github.com/sxzz/element-plus-best-practices/blob/db2dfc983ccda5570033a0ac608a1bd9d9a7f658/vite.config.ts#L21-L58
```

## 1 安装依赖包

```python
pnpm install -D unplugin-icons unplugin-vue-components unplugin-auto-import
```

## 2 修改配置文件

因为针对插件单独写了一个函数，所以这里只展示针对 plugins 的函数

```python
import vue from '@vitejs/plugin-vue'

// 导入 mock 组件
import { viteMockServe } from 'vite-plugin-mock'

// 导入 mock 组件
import legacy from '@vitejs/plugin-legacy'

import AutoImport from 'unplugin-auto-import/vite'
import Components from 'unplugin-vue-components/vite'

// 按需导入 Element Plus 组件
import { ElementPlusResolver } from 'unplugin-vue-components/resolvers'
import VueDevTools from 'vite-plugin-vue-devtools'

// 自动导入 Element Plus 的 Icon 组件
import IconsResolver from 'unplugin-icons/resolver'
import Icons from 'unplugin-icons/vite'

const getPlugins = (command, mode, env) => {
  return [
    vue(),
    AutoImport({
      imports: ['vue', 'vue-router'],
      dts: './auto-imports.d.ts',
      resolvers: [ElementPlusResolver()]
    }),
    Components({
      resolvers: [
		// 自动注册图标组件
        IconsResolver({
		  // 修改 Icon 组件前缀，不设置则默认为 i，禁用则设置为 false
          prefix: false,
		  // 指定图标集，这里意味着使用了 Element Plus 的图标集
          enabledCollections: 'ep'
        }),
        ElementPlusResolver()
      ]
    }),
    Icons(),
    legacy({
      targets: ['defaults', 'not IE 11']
    }),
    viteMockServe({
      enable: true,
      logger: false
    }),
    VueDevTools()
  ]
}

export default getPlugins
```

## 3 使用图标

参考官方文档，复制代码，不引用任何的图标组件，直接使用

```python
    <el-icon :size="20"><Plus /></el-icon>
    <el-icon>
      <Edit />
    </el-icon>
```

但实际上，这段代码会报错，提示找不到对应的组件，正确的写法如下：

```python
    <el-icon :size="20"><EpPlus /></el-icon>
    <el-icon>
      <ep-Edit />
    </el-icon>
```

可以看到，正确的代码比错误的使用，差异就在于 `ep` 这个前缀。

**解释**
使用组件解析器 IconsResolver 时，必须遵循名称转换才能正确推断图标，格式如下：

```python
{prefix}-{collection}-{icon}
```

所以在模板中使用图标使，有两种方式

-   第一种：ep-{图标名称} ：这种是以中划线进行连接，例如 ep-Edit
-   第二种：Ep{图标名称} ：这种是以驼峰的格式，例如 EpPlus

## 4 IconsResolver 配置

查看下 IconsResolver 函数中参数的类型，类型如下：

```python
interface ComponentResolverOption {
    /**
     * Prefix for resolving components name.
     * Set '' to disable prefix.
     *
     * @default 'i'
     */
    prefix?: string | false;
    /**
     * Iconify collection names to that enable for resolving.
     *
     * @default [all collections]
     */
    enabledCollections?: string | string[];
    /**
     * Icon collections aliases.
     *
     * The `aliases` keys are the `alias` and the values are the `name` for the collection.
     *
     * Instead using `<i-icon-park-abnormal />` we can use `<i-park-abnormal />` configuring:
     * `alias: { park: 'icon-park' }`
     */
    alias?: Record<string, string>;
    /**
     * Name for custom collections provide by loaders.
     */
    customCollections?: string | string[];
    /**
     * Extension for the resolved id
     * Set `jsx` for JSX components
     *
     * @default ''
     */
    extension?: string;
    /**
     * @deprecated renamed to `prefix`
     */
    componentPrefix?: string;
    /**
     * For collections strict matching.
     * Default is `false`, not side effect.
     * Set `true` to enable strict matching with `-` suffix for all collections.
     */
    strict?: boolean;
}
```

---

```python
{prefix}-{collection}-{icon}
```

## prefix

```python
    /**
     * Prefix for resolving components name.
     * Set '' to disable prefix.
     *
     * @default 'i'
     */
    prefix?: string | false;
```

prefix 配置的值，可以看到，类型是字符串或者 false。假如配置如下：

```python
IconsResolver({
	prefix: 'gregeo',
	...
})
```
那么在使用图标组件时，按照之前的说明，有两种方式

- gregeo-ep-Plus
- GregeoEpPlus

## collection

```python
    /**
     * Iconify collection names to that enable for resolving.
     *
     * @default [all collections]
     */
    enabledCollections?: string | string[];
```

可以看到 enabledCollections 可以为单个字符串，或者是字符串数组。图标集可参考 https://icon-sets.iconify.design/

## icon

则是图标集中某个图标的 name



