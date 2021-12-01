# 表单和事件

如果性能不是一个问题（通常不是！），内联处理程序是最简单的，因为你可以直接使用类型推理和上下文类型化。

```tsx
const el = (
  <button
    onClick={event => {
      // event: React.MouseEvent<HTMLButtonElement, MouseEvent>
    }}
  />
);
```

## 基本使用

```tsx
import React, { useState } from 'react';

const App = () => {
  const [value, setValue] = useState('');

  const handleChange = (e: React.FormEvent<HTMLInputElement>) => {
    setValue(e.currentTarget.value);
  };

  return <input value={value} onChange={handleChange} />;
};
```

也可以使用`ChangeEventHandler`

```tsx
import React, { useState } from 'react';

const App = () => {
  const [value, setValue] = useState('');

  const handleChange: React.ChangeEventHandler<HTMLInputElement> = e => {
    setValue(e.currentTarget.value);
  };

  return <input value={value} onChange={handleChange} />;
};
```

如果不太关心事件的类型, 可使用`React.SyntheticEvent`, 如果需要目标表单有访问的输入类型, 可以使用类型断言.

```tsx
import React from 'react';

const App = () => {
  return (
    <form
      onSubmit={(e: React.SyntheticEvent) => {
        e.preventDefault();
        const target = e.target as typeof e.target & {
          email: { value: string };
          password: { value: string };
        };
        const email = target.email.value; // typechecks!
        const password = target.password.value; // typechecks!
      }}
    >
      <div>
        <label>
          Email:
          <input type='email' name='email' />
        </label>
      </div>
      <div>
        <label>
          Password:
          <input type='password' name='password' />
        </label>
      </div>
      <div>
        <input type='submit' value='Log in' />
      </div>
    </form>
  );
};
```

## 事件列表

| 事件类型         | 描述                                                                                                                                                                     |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| AnimationEvent   | Css 动画。                                                                                                                                                               |
| ChangeEvent      | 改变`<input>`、`<select>`和`<textarea>`元素的值。                                                                                                                        |
| ClipboardEvent   | 复制、粘贴以及剪切事件。                                                                                                                                                 |
| CompositionEvent | 由于用户间接输入文本而发生的事件 （例如，根据浏览器和电脑设置，如果你想在美国键盘上输入日语，可能会出现一个弹出窗口， 显示出额外的字符）。                               |
| DragEvent        | 用指针设备（如鼠标）进行拖放互动。                                                                                                                                       |
| FocusEvent       | 当元素获得或失去焦点时发生的事件。                                                                                                                                       |
| FormEvent        | 当一个表单或表单元素得到/失去焦点，一个表单元素的值被改变或表单被提交时，就会发生这个事件。                                                                              |
| InvalidEvent     | 当输入的有效性限制失败时启动（例如`<input type="number" max="10">`，有人会插入数字 20）。                                                                                |
| KeyboardEvent    | 用户与键盘的互动。每个事件都描述了一个单一的按键互动。                                                                                                                   |
| MouseEvent       | 由于用户与指向性设备（如鼠标）互动而发生的事件。                                                                                                                         |
| PointerEvent     | 由于用户与各种指向性设备的互动而发生的事件，如鼠标、笔/木杆、触摸屏，也支持多点触摸。除非你为旧的浏览器（IE10 或 Safari 12）开发，否则建议使用指针事件。继承自 UIEvent。 |
| TouchEvent       | 由于用户与触摸设备的交互而发生的事件。继承自 UIEvent。                                                                                                                   |
| TransitionEvent  | CSS 过渡。不完全被浏览器支持。继承自 UIEvent。                                                                                                                           |
| UIEvent          | 鼠标、触摸和指针事件的基础事件。                                                                                                                                         |
| WheelEvent       | 在鼠标滚轮或类似的输入设备上进行滚动。(注意：`wheel`事件不应该与`scroll`事件相混淆）。                                                                                   |
| SyntheticEvent   | 所有上述事件的基础事件。如果不明白事件类型时使用。                                                                                                                       |
