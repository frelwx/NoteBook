
length - TAG - [(magic - pk_der - secure_level -  payload - nv) - hash - rsa_signature]

magic length enable_auth enable_decrypt rsa_signature hash public_key nv_counter payload 

1. 我需要为我的BL2.bin（payload）添加一个头部，请你帮我用python实现，加解密使用pycryptodome库
2. 输出的镜像格式如下：magic length enable_auth enable_decrypt rsa_signature hash public_key public_key public_key_len nv_counter payload 
3. 其中length表示payload的长度，该字段长度为4字节
4. enable_auth和enable_decrypt位于一个4字节整数的bit0和bit1，
5. rsa4096签名的计算从public_key开始到payload的最后一个比特，
6. hash只计算payload，
7. public_key是一个DER格式的公钥，
8. public_key_len表示public_key的长度，该字段长度为4字节
9. nv_counter是一个4字节整数，
10. payload是需要处理的BL2文件
11. 输入文件、rsa私钥、enable_auth、enable_decrypt、nv_counter需要从命令行参数获取
12. 除了镜像之外，最终还需要输出public_ker的sha256值到文件rotpk_sha256.bin
13. 镜像制作完成后，将上述头部的字段的详细内容打印到命令行中，此外，还需要打印签名的长度、hash的长度、Public Key (DER)的长度，公钥哈希文件rotpk_sha256.bin的长度

