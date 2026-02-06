# PA1实验报告

- ###### 1+2+...+100 程序状态机图：

  - 初始阶段与前两次循环： (0, x, x) → (1, 0, x) → (2, 0, 0) → (3, 0, 1) → (4, 1, 1) → (2, 1, 1) → (3, 1, 2) → (4, 3, 2) → (2, 3, 2) ...
  - 中间省略部分： ... → (2, sum_{n-1}, n-1) → (3, sum_{n-1}, n) → (4, sum_n, n) → ...
  - 最后两次循环与结束： ... → (2, 4753, 98) → (3, 4753, 99) → (4, 4851, 99) → (2, 4851, 99) → (3, 4851, 100) → (4, 5050, 100) → (5, 5050, 100)

- 两种花费时间情况

  - 没有简易调试器（使用 GDB）：

    - $$
      排除一个 Bug 的时间：20 \text{ 个信息} \times 30 \text{ 秒/信息} = 600 \text{ 秒} = 10 \text{ 分钟}\\总调试时间：450 \text{ 次调试} \times 10 \text{ 分钟/次} = 4500 \text{ 分钟}\\换算成小时：4500 \div 60 = \mathbf{75 \text{ 小时}}
      $$

  - 实现了简易调试器（sdb）:

    - $$
      排除一个 Bug 的时间：20 \text{ 个信息} \times 10 \text{ 秒/信息} = 200 \text{ 秒} \approx 3.33 \text{ 分钟}\\总调试时间：450 \text{ 次调试} \times 200 \text{ 秒/次} = 90000 \text{ 秒}\\换算成小时：90000 \div 3600 = \mathbf{25 \text{ 小时}}
      $$

- x86

  - EFLAGS寄存器中的CF位是什么意思?
    - https://nju-projectn.github.io/i386-manual/s02_03.htm 2.3.4 Flags Register
    - https://nju-projectn.github.io/i386-manual/appc.htm Status Flags' Functions
  - ModR/M字节是什么?
    - https://nju-projectn.github.io/i386-manual/s17_02.htm  17.2.1 ModR/M and SIB Bytes
  - mov指令的具体格式是怎么样的?s
    - https://nju-projectn.github.io/i386-manual/MOV.htm

- shell命令：

  - 完成PA1的内容之后, `nemu/`目录下的所有.c和.h和文件总共有多少行代码?你是使用什么命令得到这个结果的?
    - 323123行
    - `find nemu/ -name "*.[ch]" | xargs wc -l`
  - 和框架代码相比, 你在PA1中编写了多少行代码? 
    - 613行
    - `git diff pa0..pa1 nemu/ | grep "^+" | grep -v "^+++" | wc -l`
  - 除去空行之外, `nemu/`目录下的所有`.c`和`.h`文件总共有多少行代码?
    - 280600行
    - `find nemu/ -name "*.[ch]" | xargs grep -v "^[[:space:]]*$" | wc -l`

- 请解释gcc中的`-Wall`和`-Werror`有什么作用? 为什么要使用`-Wall`和`-Werror`?

  - **`-Wall` (All Warnings)**：开启了一组编译器认为**非常重要且易错**的警告选项（如未使用的变量、未初始化的变量、潜在的逻辑错误等）。
  - **`-Werror` (Warnings as Errors)**： 它的作用是**“将所有警告视为错误”**。只要代码中出现任何警告，编译器就会停止编译过程，并不生成可执行文件。
  - 为什么要用？：
    1. **省时**：在编译阶段发现 Bug，远比运行时用 GDB 瞎找快得多。
    2. **安全**：强制处理未初始化变量等风险，符合安全编程规范。
    3. **纪律**：保持框架代码整洁，不让致命警告淹没在琐碎警告中。
  
- > ##### kconfig生成的宏与条件编译
  >
  > 我们已经在上文提到过, kconfig会根据配置选项的结果在 `nemu/include/generated/autoconf.h`中定义一些形如`CONFIG_xxx`的宏, 我们可以在C代码中通过条件编译的功能对这些宏进行测试, 来判断是否编译某些代码. 例如, 当`CONFIG_DEVICE`这个宏没有定义时, 设备相关的代码就无需进行编译.
  >
  > 为了编写更紧凑的代码, 我们在`nemu/include/macro.h`中定义了一些专门用来对宏进行测试的宏. 例如`IFDEF(CONFIG_DEVICE, init_device());`表示, 如果定义了`CONFIG_DEVICE`, 才会调用`init_device()`函数; 而`MUXDEF(CONFIG_TRACE, "ON", "OFF")`则表示, 如果定义了`CONFIG_TRACE`, 则预处理结果为`"ON"`(`"OFF"`在预处理后会消失), 否则预处理结果为`"OFF"`.
  >
  > 这些宏的功能非常神奇, 你知道这些宏是如何工作的吗?

  - 原理（以 `MUXDEF(CONFIG_DEVICE, "ON", "OFF")` 为例）

    - 情况 A：宏 `CONFIG_DEVICE` 已定义为 `1`
      - **拼接**：`MUXDEF` 会把 `__P_DEF_` 和 `CONFIG_DEVICE` 拼接成 `__P_DEF_1`。
      - **替换**：`__P_DEF_1` 被替换为 `X, `（注意那个逗号）。
      - **展开 `MUX_WITH_COMMA`**： 它接收到的第一个参数 `contain_comma` 变成了 `X, `。 代入后变成：`CHOOSE2nd(X, "ON", "OFF")`
      - **最终选择**：
        - 参数 1：`X`
        - 参数 2：`"ON"`
        - `CHOOSE2nd` 取第二个参数，结果为 **`"ON"`**。

    - 情况 B：宏 `CONFIG_DEVICE` 未定义

      - **拼接**：由于没有定义 `CONFIG_DEVICE`，拼接结果是原样字符串 `__P_DEF_CONFIG_DEVICE`。
      - **替换**：预处理器找不到 `__P_DEF_CONFIG_DEVICE` 的定义，所以它**没有逗号**。
      - **展开 `MUX_WITH_COMMA`**： 代入后变成：`CHOOSE2nd(__P_DEF_CONFIG_DEVICE "ON", "OFF")` 注意！因为没有逗号，`__P_DEF_...` 和 `"ON"` 被看作了同一个参数（中间只是空格）。

      - **最终选择**：
        - 参数 1：`__P_DEF_CONFIG_DEVICE "ON"`
        - 参数 2：`"OFF"`
        - `CHOOSE2nd` 取第二个参数，结果为 **`"OFF"`**。

- 