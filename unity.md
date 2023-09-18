### Unity组件的生命周期
CSharp脚本

生命周期（按顺序）：
1. Awake
    - 最早调用，可以在此处实现单例模式
2. OnEnable
    - 组件激活后调用一次
3. Start
    - 在此处设置初始值
4. FixedUpdate
    - 固定频率调用，每次调用与上次调用的时间间隔相同
5. Update
    - 帧率调用方法，每帧调用一次，每次调用与上次调用的时间间隔不相同
6. LateUpdate
    - 在Update每调用完一次后，紧跟着调用一次
7. OnDisable
    - 与OnEnable相反，组件未激活时调用
8. OnDestroy
    - 被销毁后调用一次
