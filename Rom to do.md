1. 多核启动地址修改
2. 防回滚，nv_counter
3. nor flash：disable 4byte，unlock all，测试代码，xip(optional)
4. nand flash: 4kB page指令适配，测试代码
5. 解决fpga mmu无法使用的bug
6. 解决fpga jlink问题
7. 编写代码，交换一个字节中的bit[0,4]和bit[5,7]，
8. efuse存在8个address，每个address表示32个bit，address用32位寄存器的bit[0,2]表示，bit用bit[3,7表示]，我希望将其转换成用一个整数表示第几个比特，后续efuse可能会扩容到32个address


周末 to do
- [ ] 继续看cache、mmu、内存屏障
- [ ] 