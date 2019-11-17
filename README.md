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

Option is an Object. you can set some setting via option.

| value    | defalut value | descrition                                                   |
| -------- | ------------- | ------------------------------------------------------------ |
| position |               | set element position, opsition is an Object ,default vlaue is {right: 0, bottom: 0}, here are avaliable attributes [top\|right\|bottom\|right] |
| safe     | [0,0,0,0]     | dege space to retain                                         |

Although there are four attributes can be used in position object, but actually two attribute is enough, top with left, top with right, or bottom with left , bottom with right. position object just to set drag element position in the web page.

Notice:  when you set position, you must give two attribute with value to position at the same time.

## Event

There are three events can be used. all you have to do is just add the event listener to the component, three event callbacks all return a point coordinate object. 

Notice: transform(change translate) way and traditional way (change left、right、bottom or top) coordinate info sinnificance is different, because they work in different way (transform or position), and this result in different sinnificance  data to return.

| attribute | type  | descrition          |
| --------- | ----- | ------------------- |
| start     | event | start event trigger |
| move      | event | move event trigger  |
| end       | event | end event trigger   |

You can use event listener like this

```vue
<vue-drag-element :options="options" @start="start" @move="move" @end="end">
   <!-- here insert you element -->
   <div class="ball"></div>
</vue-drag-element>
```

