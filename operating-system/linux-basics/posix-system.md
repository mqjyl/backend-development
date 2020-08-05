# POSIX系统

符合POSIX的系统信号处理总结：

                1. 一旦安装了信号处理函数，它便一直安装者（较早期的系统是每运行一次就将其拆除）。

                2. 在一个信号处理函数执行期间，正被递交的信号是堵塞的。

                3. 假设一个信号在被堵塞期间产生了一次或多次，那么该信号被解堵塞之后通常仅仅递交一次，也就是说Unix信号缺省是不排队的。

                4. 利用sigprocmask函数选择性地堵塞或解堵塞一组信号是可能的。这使得我们能够做到在一段临界区代码运行期间，防止捕获某些信号。以此保护这段代码。
