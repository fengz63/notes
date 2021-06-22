### 一、包和crate
模块系统的第一部分，我们将介绍包和 crate。crate 是一个二进制项或者库。crate root 是一个源文件，Rust 编译器以它为起始点，并构成你的 crate 的根模块。包（package） 是提供一系列功能的一个或者多个 crate。一个包会包含有一个 Cargo.toml 文件，阐述如何去构建这些 crate。

包中所包含的内容由几条规则来确立。一个包中至多 **只能** 包含一个库 crate（library crate）；包中可以包含任意多的二进制 crate（binary crate）；包中至少包含一个crate，无论是库的还是二进制的。

### 二、定义模块来控制作用域和私有性

*模块*让我们将一个crate中的代码进行分组，以提高可读性和重用性。模块还可以控制项的 *私有性*，即项是可以被外部代码使用的（public），还是作为一个内部实现的内容，不能被外部代码使用（private）。一个模块的示例如下所示：
```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}

        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}

        fn server_order() {}

        fn take_payment() {}
    }
}
```
如上所示，模块以```mod```关键字为起始，然后指定模块的名字（本例中叫做front_of_house），并且用花括号包围模块的主体。在模块内，我们还可以定义其他的模块，就像本例中的 hosting 和 serving 模块。

### 三、路径用于引用模块树中的项
rust使用路径的方式在模块树中找到一个项的位置，就像在文件系统使用路径一样，我们想要调用一个函数，就需要知道它的路径。

路径有两种形式：
+ 相对路径：从当前模块开始，以 self、super 或当前模块的标识符开头。
+ 绝对路径：从crate根开始，以 crate 名或者字面值 crate 开头。

绝对路径和相对路径都后跟一个或多个由双冒号（::）分割的标识符。

一个相互调用的示例如下所示：
```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();

    // Relative path
    front_of_house::hosting::add_to_waitlist();
}
```
上述中，```eat_at_restaurant``` 函数是我们 crate 库的一个公共API，所以我们使用```pub```关键字来标记（后面详细解释）。