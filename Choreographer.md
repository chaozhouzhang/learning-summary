Coordinates the timing of animations, input and drawing.
协调动画、输入和绘图的时间。

The choreographer receives timing pulses (such as vertical synchronization) from the display subsystem then schedules work to occur as part of rendering the next display frame.
编排器从显示子系统接收定时脉冲(比如垂直同步)，然后安排工作作为呈现下一个显示帧的一部分。

Applications typically interact with the choreographer indirectly using higher level abstractions in the animation framework or the view hierarchy. Here are some examples of things you can do using the higher-level APIs.

应用程序通常使用动画框架或视图层次结构中的高级抽象来间接地与编排器交互。下面是一些使用高级api可以完成的事情的示例。

