# 指南

oui针对uci封装了一些vue组件，方便开发人员调用。

``` vue
<template>
  <oui-form uci-config="test">
    <oui-named-section title="Test" name="main" v-slot="{ s }">
      <oui-form-item-input label="Name" name="name" :uci-section="s"/>
    </oui-named-section>
  </oui-form>
</template>
```
这是一个uci配置页面的基本结构。

`oui-form`组件在创建后，会通过调用ubus加载`uci-config`属性指定的uci配置文件。然后根据子组件生成表单。

## 表单验证

通过给`oui-form-item-xx`组件指定`rules`属性实现表单验证。该属性支持字符串，对象以及自定义函数。

``` vue
<oui-form-item-input label="ID" name="id" rules="integer" :uci-section="s"/>
```
等价于
``` vue
<oui-form-item-input label="ID" name="id" :rules="{type: 'integer'}" :uci-section="s"/>
```
等价于
``` vue
<oui-form-item-input label="ID" name="id" :rules="validateID" :uci-section="s"/>
...
<script>
export default {
  methods: {
    validateID(value) {
      if (!isNaN(value) && parseInt(value).toString() === value)
        return;

      return this.$t('validator.integer');
    }
  }
}
</script>
```

通过对象方式可以指定多个规则：
``` vue
<oui-form-item-input label="ID" name="id" :rules="{type: 'integer', min: 12, max: 100}" :uci-section="s"/>
```

如果该选项为必填项，可设置`required`属性，该属性类型为布尔值，默认为false。
``` vue
<oui-form-item-input label="ID" name="id" required :uci-section="s"/>
```

oui目前支持的验证规则：
- url
- email
- number
- integer
- uinteger
- min
- max
- hostname
- ip4addr
- ip6addr
- ipaddr
- netmask4
- netmask6
- host
- port
- macaddr
- uciname

## 选项依赖

选项的显示与否依赖于一个或者多个其它选项的值。

给`oui-form-item-xx`组件设置`depend`属性可以实现依赖。该属性的类型为字符串。我们需要提供一个以字符串表示的表达式。
``` js
depend="(a == 12 || a == 'x') && y == 4 && q != 5 && !z"
```

a选项依赖于b选项，如果b选项的值为'5'，a选项则显示，否则不显示：
``` vue
<oui-form-item-input label="A" name="a" depend="b == '5'" :uci-section="s"/>
<oui-form-item-input label="B" name="b"/>
```

a选项依赖于b选项和c选项，如果b选项的值为'oui'，而且c选项的值不等于'u'，a选项则显示，否则不显示：
``` vue
<oui-form-item-input label="A" name="a" depend="b == 'oui' && c != 'u'" :uci-section="s"/>
<oui-form-item-input label="B" name="b" :uci-section="s"/>
<oui-form-item-input label="C" name="c" :uci-section="s"/>
```

a选项依赖于b选项和c选项，如果b选项的值为真，而且c选项的值为假，a选项则显示，否则不显示：
``` vue
<oui-form-item-input label="A" name="a" depend="b && !c" :uci-section="s"/>
<oui-form-item-switch label="B" name="b" :uci-section="s"/>
<oui-form-item-switch label="C" name="c" :uci-section="s"/>
```