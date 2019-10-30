# 并发
并发跟其他语言区别看似很大，但是底层操作系统和硬件的实现其实不可能有太大区别。

所以Rust的并发控制本质上还是和其他语言类似的。

底层api控制就是基本的mutex，atomic。

应用层就是和golang类似的channel，区别在于Rust的channel的生产者分为同步发送和异步发送。

### thread
Rust的线程对应OS线程，同步标准库。
##### 创建线程
调用`thread::spawn`函数创建线程

### channel
```rust
#[test]
fn t2() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx.send(val).unwrap();
        }
        println!("send finish");
    });

    for received in rx {
        thread::sleep(Duration::from_secs(1));
        println!("Got: {}", received);
    }
    //send finish
    //Got: hi
    //Got: from
    //Got: the
    //Got: thread
}

```
`mpsc::channel`这个形式构建出来的是异步Sender

可以看到，上例在消费者阻塞的情况下，生产者，仍然能立刻将所有信息立刻发送完毕，打印出send finish

```rust
#[test]
fn t3() {
    let (tx, rx) = mpsc::sync_channel(0);

    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx.send(val).unwrap();
        }
        println!("send finish");
    });

    for received in rx {
        println!("Got: {}", received);
        thread::sleep(Duration::from_secs(1));
    }
    //Got: hi
    //Got: from
    //Got: the
    //Got: thread
    //send finish
}
```
`mpsc::sync_channel`这个形式构建出来的是同步Sender

可以看到，在消费者被阻塞的情况下，生产者也被阻塞了。



