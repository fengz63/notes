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
上述中，```eat_at_restaurant``` 函数是我们 crate 库的一个公共API，所以我们使用```pub```关键字来标记（后面详细解释）。(注：这个例子无法编译通过，主要是hosting模块是私有的)

模块不仅对于组织代码很有用，它还定义了 Rust 的 *私有性边界*（privacy boundary）：这条界线不允许外部代码了解、调用和依赖被封装的实现细节。所以，如果你希望创建一个私有函数或结构体，可以将其放入模块。

Rust中默认所有项（函数、方法、结构体、枚举、模块和常量）都是私有的。**父模块中的项不能使用子模块中的私有项，但是子模块中的项可以使用他们父模块中的项**。

**使用 pub 关键字暴露路径**

pub 关键字可以改变项的公开性，使得其他的项可以调用。

### 四、使用use关键字将名称引入作用域

前面所讲的调用函数路径较为冗长且重复，很不方便。这里我们可以使用 use 挂件子将路径一次性引入作用域，然后调用该路径中的项，就如同它们是本地项一样。如下所示：
```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
}
```
在作用域中增加 use 和路径类似于在文件系统中创建软连接（符号连接，symbolic link）。通过在 crate 根增加 ```use crate::front_of_house::hosting```，现在 hosting 在作用域中就是有效的名称了，如同 hosting 模块被定义于 crate 根一样。通过 use 引入作用域的路径也会检查私有性，同其它路径一样。

当然也可以使用 use 和相对路径来将一个项引入作用域。

**使用 as 关键字提供新的名称**

使用 use 将两个同名类型引入同一作用域这个问题还有另一个解决办法：在这个类型的路径后面，我们使用 as 指定一个新的本地名称或者别名。如下所示：
```rust
use std::fmt::Result;
use std::io::Result as IoResult;

fn function1() -> Result {
    // --snip--
}

fn function2() -> IoResult<()> {
    // --snip--
}
```

**使用 pub use 重导出名称**

当使用 use 关键字将名称导入作用域时，在新作用域中可用的名称是私有的。如果为了让调用你编写的代码的代码能够像在自己的作用域内引用这些类型，可以结合 pub 和 use。这个技术被称为 “重导出（re-exporting）”，因为这样做将项引入作用域并同时使其可供其他代码引入自己的作用域。示例如下：
```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
}
```
通过 pub use，现在可以通过新路径 hosting::add_to_waitlist 来调用 add_to_waitlist 函数。如果没有指定 pub use，eat_at_restaurant 函数可以在其作用域中调用 hosting::add_to_waitlist，但外部代码则不允许使用这个新路径。

**使用外部包**

当我们需要使用外部包时，这里以rand为例，为了在项目中使用 rand，在Cargo.toml中加入了如下行：

文件名：Cargo.toml
```rust
[dependencies]
rand = "0.5.5"
```
在 Cargo.toml 中加入 rand 依赖告诉了 Cargo 要从 crates.io 下载 rand 和其依赖，并使其可在项目代码中使用。

接着，为了将 rand 定义引入到项目包的作用域，我们加入一行 use 起始的包名，它以 rand 包名开头并列出了需要引入作用域的项。如下所示：
```rust
use rand::Rng;

fn main() {
    let secret_number = rand::thread_rng().gen_range(1, 101);
}
```

### 五、将模块分割进不同的文件

具体见：[将模块分割进不同的文件](https://kaisery.github.io/trpl-zh-cn/ch07-05-separating-modules-into-different-files.html)