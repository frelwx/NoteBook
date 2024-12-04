1. el3_entrypoint_common
	1. _init_sctlr
		1. ~(SCTLR_TE_BIT | SCTLR_EE_BIT | SCTLR_V_BIT | SCTLR_DSSBS_BIT)
	2. Switch to monitor mode
	3. Set the exception vectors (VBAR/MVBAR)
	4. el3_arch_init_common
		1. Enable the instruction cache
		2. Enable Alignment fault checking
	5. _secondary_cold_boot
	6. _init_memory
	7. _init_c_runtime
2. bl1_setup
	1. plat_setup_early_console
	2. bl1_early_platform_setup
	3. bl1_plat_arch_setup
3. bl1_main

To do list
- [ ] 对IO端口基类进行抽象，~~UART初始化添加超时机制，防挂死（考虑往Efuse中添加字段禁掉某些UART）~~或者直接使用UART1-0.5
- [ ] MMU-0.5
- [ ] 4KB PageSize适配，~~Flash gtest测试用例，完整测试用例~~-1
- [ ] BL2镜像从Flash读取，解析+AES-GCM适配-1
- [ ] Efuse代码扩容，从Efuse中读取ROTPK的HASH对比-0.5

复盘

1. 签名：解析DER格式公钥，不熟悉
2. 对称加密：依赖tf-a的脚本
3. 4KB Page Size适配：修改原来server程序
4. gtest测试用例：写红温了，因为之前没有抽象host端的程序
5. cache+mmu：不熟悉，出问题卡了很久
6. 镜像头的格式：纠结了太久
7. nor flash：重构原本的代码，添加4byte disable

措施：
1. 学习arm架构
2. 学些一些汇编
3. 代码设计时需考虑可扩展性
4. 学习一些大型项目的代码
5. 节奏状态不对，相似
6. 发现问题及早重构，比如烧录程序
7. 晚上早睡

调试遇到BUG：
1. 先找一个可行的最简单的方案，再对比
2. 飞线不靠谱，要求焊接，不要尝试飞线，浪费自己时间
3. 卡住时多想，少做，不能急着做

下周需要做的：
1. 调试fpga：先尝试rt_base版本能否读写spinand、接着解除MMU区域的访问限制，这样可以使用jlink调试。还不行便尝试MMU版本，最后不行再用逻辑分析仪采样
2. 跑一遍Efuse相关的场景用例
3. 多核启动
4. 