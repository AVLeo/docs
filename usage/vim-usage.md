# vim 使用
1. 编译gbk编码文件
编辑文件/etc/vim/vimrc，添加或修改
```
let &termencoding=&encoding
set fileencodings=utf-8,gbk,ucs-bom,cp936
```
2. 自动保存上次编辑位置
编辑文件/etc/vim/vimrc, 添加或修改
```
if has("autocmd")
  au BufReadPost * if line("'\"") > 1 && line("'\"") <= line("$") | exe "normal! g'\"" | endif
endif
```