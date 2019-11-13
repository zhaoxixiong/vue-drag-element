# vue-drag-element

## Environment

This plugin is run with Vue2.

## Platform

Not only mobile but also web are work perfectly.

## Usage

Import component in your project, then write another a little bit code like below the component will work.

### In script

```
import VueDragElement from 'vue-drag-element'
export default {
  components:{ VueDragElement },
  ......
  data() {
    return {
      options: {
         position: {
           right:12,
           top:12
        }
      }
    };
  }
......
}
```

### In template

```vue
<vue-drag-element :options="options">
   <!-- here insert you element -->
   <div class="ball"></div>
</vue-drag-element>
```

## Option

There is just one option. but more optoins will be add in the future version.

| attribute | type   | value    | defalut value | descrition                                                   |
| --------- | ------ | -------- | ------------- | ------------------------------------------------------------ |
| options   | Object | position |               | set element position, opsition is an Object ,default vlaue is {right: 0, bottom: 0}, here are avaliable attributes [top\|right\|bottom\|right] |

Although there are four attributes can be used in position object, but actually two attribute is enough, top with left, top with right, or bottom with left , bottom with right. position object just to set drag element position in the web page.

Notice:  when you set position, you must give two attribute with value to position at the same time.