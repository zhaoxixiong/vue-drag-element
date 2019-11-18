# vue-drag-element

## Environment

This plugin is run in Vue2.

## Platform

Not only mobile but also web are work perfectly.

## Usage

Import component in your project, then write another a little bit code like below the component will work.

## Option

Option is an Object. you can set some setting via option.

| value     | defalut value            | descrition                                                   |
| --------- | ------------------------ | ------------------------------------------------------------ |
| position  | { right:12, bottom: 12 } | set element position, position is an Object , if position is not set or empty, then the default value will be {right: 12, bottom: 12}, here are avaliable attributes [top\|right\|bottom\|right] |
| edgeSpace | [0,0,0,0]                | edge space to retain                                         |

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

Here is an example, pithy code and easy to use.

### In template

```vue
<vue-drag-element :options="options" @start="start" @move="move" @end="end">
   <!-- here insert you element -->
   <div class="ball"></div>
</vue-drag-element>
```

### In script

```vue
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
  },
  methods: {
    start(point) {
      console.log(point)
    },
    move(point) {
      console.log(point)
    },
    end(point) {
      console.log(point)
    }
  }
......
}
```

中文文档：<http://yunkus.com/>

