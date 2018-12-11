c3786009879b026cee146fc98617b924e48764a7


$git push
Username for 'https://github.com': luck-lynnma
Password for 'https://luck-lynnma@github.com':
remote: Invalid username or password.
fatal: Authentication failed for 'https://github.com/luck-lynnma/knowledge.git/'

解决方法：

设置git config使得git记住用户名的认证信息，然后在github上申请一个Personal access token，然后push的时候在提示输密码的位置输入这个token就可以了，下次再push的时候git会自动进行验证。具体步骤：
1. $git config --global credential.helper store
2. 点击右上角头像的下来小三角 => Settings => Personal access tokens
3. Generate new token => 填写token的描述 => 选择token的权限范围（如果只是为了提交代码只勾选repo就够了） => 点击Generate Token
4. 然后会生成一个160bits（40bytes）的token，并提示你让你记住这个token，之后不会再出现。
5. 然后在你的本地库敲入：git push，输入用户名，然后在提示密码的地方把刚才的token粘贴进来就好了。
