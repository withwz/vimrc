# vimrc
我的 vimrc 配置

```vimrc
nmap H ^
nmap L $
nmap J 2jzz
nmap K 2kzz
nmap <C-j> 4jzz
nmap <C-k> 4kzz
vmap H ^
vmap L $
vmap J 2jzz
vmap K 2kzz
vmap <C-j> 4jzz
vmap <C-k> 4kzz

nmap <C-i> o<Esc>
nmap <C-g> G
nmap <C-v> V
nmap <C-z> u
nmap q <Esc>

imap jj <Esc>
imap kk <Esc>
imap jk <Esc>
imap <C-j> <Esc>
imap <C-k> <Esc>
nnoremap <silent> <Tab> o<Esc>


syntax enable                " 打开语法高亮
set clipboard=unnamed        " 启用系统粘贴板
set tabstop=4                " 设置tab键的宽度
set shiftwidth=4             " 换行时行间交错使用4个空格
set autoindent               " 自动对齐
set backspace=2              " 设置退格键可用
set smartindent              " 智能自动缩进
set number                   " 在每一行最前面显示行号
set showmatch                " 高亮显示对应的括号
set mouse=a                  " 启用鼠标
set ruler                    " 在编辑过程中，在右下角显示光标位置的状态行
set cursorline               " 突出显示当前行
set noswapfile               " 设置无交换区文件"
set writebackup              " 设置无备份文件
set nobackup                 " 设置无备份文件
set autochdir                " 设定文件浏览器目录为当前目录
set foldmethod=syntax        " 选择代码折叠类型
set laststatus=2             " 开启状态栏信息
set cmdheight=1              " 命令行的高度，默认为1，这里设为1
set autoread                 " 自动检测并加载外部对文件的修改
set autowrite                " 自动检测并加载外部对文件的修改
set showcmd                  " 在状态行显示目前所执行的命令，未完成的指令片段亦会显示出来
set nofoldenable 			 " 永远不启用折叠
set relativenumber           " 设置相对行号




call plug#begin()
Plug 'preservim/NERDTree'
Plug 'easymotion/vim-easymotion'

Plug 'junegunn/fzf'
Plug 'junegunn/fzf.vim'
" 多光标（VSCode cmd+d 同款，配合下面的 n/f 映射）
Plug 'mg979/vim-visual-multi', {'branch': 'master'}
call plug#end()




" :b 切换 NERDTree
cnoreabbrev b NERDTreeToggle
" Command + B 切换 NERDTree
nnoremap <C-b> :NERDTreeToggle<CR>



nmap <space> <Plug>(easymotion-bd-w)
" Ctrl + p 查看文件列表
nmap <C-p> :Files ./<CR>



" ============ VSCode 同款：v 模式选词 n/f 跳转 ============
" v 模式选中单词后按 n：追加选择下一个相同内容（等同 VSCode cmd + d 多光标）
"   原理：第一个 <C-n> 把选区交给 vim-visual-multi，第二个 <C-n> 立即追加下一处
"   进入多光标后：n/N = 下一个/上一个出现，q = 跳过当前，<Esc> = 退出
xmap n <C-n><C-n>

" v 模式按 f：选区跳到文件内下一个相同内容（轮回跳转，不打开搜索栏）
" v 模式按 F：选区跳到上一个相同内容（轮回跳转）
vnoremap <silent> f :<C-u>call <SID>JumpOccurrence(0)<CR>gn
vnoremap <silent> F :<C-u>call <SID>JumpOccurrence(1)<CR>gn

" 取当前选区文本（用 getregion，不污染寄存器和系统剪贴板）
function! s:VisualSelText() abort
  return join(getregion(getpos("'<"), getpos("'>"), {'type': visualmode()}), "\n")
endfunction

" 设置搜索模式并把光标移到下一个/上一个匹配的开头（由映射结尾的 gn 负责选中）
function! s:JumpOccurrence(back) abort
  let l:text = s:VisualSelText()
  if l:text ==# '' | return | endif
  " \V + 转义：把选区当纯文本搜索，不解释正则符号
  let @/ = '\V' . escape(substitute(l:text, "\n", '\\n', 'g'), '\/')
  if a:back
    " 从选区开头之前往回找，找不到绕到文件末尾（轮回）
    call cursor(line("'<"), max([1, col("'<") - 1]))
    call search(@/, 'bw')
  else
    " 从选区末尾之后往前找，找不到绕到文件开头（轮回）
    call cursor(line("'>"), col("'>") + 1)
    call search(@/, 'w')
  endif
endfunction

" ============ VSCode 同款：全局单书签（m 设置，ctrl+o 回跳）============
" m     : 清除旧书签，在当前行设置新书签（全局始终只有一个，新的覆盖旧的）
" ctrl+o: 跳回书签位置（跨文件也行；没有书签时保持 Vim 默认的跳转列表回跳）
" 原理：借用 Vim 全局标记 M（跨文件、随 viminfo 持久化）
nmap m :<C-u>call <SID>MarkSet()<CR>
nmap <c-o> :<C-u>call <SID>MarkJump()<CR>

function! s:MarkSet() abort
  delmarks M
  normal! mM
  call s:MarkSync()
  echo '书签已设置 @ ' . expand('%:t') . ':' . line('.')
endfunction

function! s:MarkJump() abort
  if getpos("'M")[1] > 0
    normal! 'M
    normal! zz
    call s:MarkSync()
    echo '跳到书签 @ ' . expand('%:t') . ':' . line('.')
  else
    " 没有书签时退回默认 <C-o>（跳转列表回跳）
    execute "normal! \\<C-o>"
  endif
endfunction

" ============ 书签可视化：行号旁显示标记（重启后也恢复）============
sign define VSCodeBookmark text=>> texthl=BookmarkSign numhl=BookmarkLineNr

function! s:DefineBookmarkHL() abort
  " 书签色：橙色，想换颜色改这里
  silent! highlight BookmarkSign   ctermfg=208 cterm=bold guifg=#ffaf00 gui=bold
  silent! highlight BookmarkLineNr ctermfg=208 cterm=bold guifg=#ffaf00 gui=bold
endfunction
call s:DefineBookmarkHL()

" 把 sign 重放到标记 M 所在行（组级唯一，跨缓冲区；标记在未加载文件时也先占位）
function! s:MarkSync() abort
  call sign_unplace('vimrc_bookmark')
  let l:pos = getpos("'M")
  if l:pos[1] > 0 && l:pos[0] > 0
    call sign_place(1, 'vimrc_bookmark', 'VSCodeBookmark', l:pos[0], {'lnum': l:pos[1]})
  endif
endfunction

augroup VSCodeBookmark
  autocmd!
  " 换配色后重新应用书签颜色
  autocmd ColorScheme * call <SID>DefineBookmarkHL()
  " 切文件/增删行（Vim 标记会自动调行号，sign 跟着重放）/重启恢复时刷新
  autocmd BufEnter * call <SID>MarkSync()
  autocmd TextChanged,InsertLeave * call <SID>MarkSync()
  autocmd VimEnter * call <SID>MarkSync()
augroup END

" ============ VSCode 同款：v 模式选词 n/f 跳转 END ============

" ============ 强化 Esc：退出多光标 + 切英文输入法 ============
" 1) 多光标：VM 插入/普通模式一次 Esc 完全退出（vm#reset 对未激活是空操作）
" 2) 输入法：Esc 后自动切回英文 ABC,进 normal 直接敲命令不会打进中文
"    实现：ime-switch --serve 常驻 daemon（二进制 ~/.npm-global/bin/ime-switch,
"    源码 ~/.local/src/ime-switch.c）。Esc 只往管道写一行，亚毫秒完成；
"    Carbon 框架加载约 50ms,所以 InsertEnter 时预热，首个 Esc 已就绪。
"    daemon 随 vim 退出 EOF 自动结束；挂了下次 Esc 自动拉起；无 job 支持时同步兜底。
let g:ime_switch_bin = expand('~/.npm-global/bin/ime-switch')
let g:ime_en_id      = 'com.apple.keylayout.ABC'
let s:ime_job = 0

function! s:IMEEnsure() abort
  if exists('*job_start') && exists('*ch_sendraw')
        \ && type(s:ime_job) == v:t_job && job_status(s:ime_job) ==# 'run'
    return 1
  endif
  if !executable(g:ime_switch_bin) | return 0 | endif
  if exists('*job_start')
    " 注意: 不要加 out_io/err_io 'null' 选项——实测会让 vim 关掉 job 的
    " stdin,daemon 收到 EOF 秒退(exitval=0);daemon 正常不写 stdout,无需 null
    try
      let s:ime_job = job_start([g:ime_switch_bin, '--serve'])
    catch
      let s:ime_job = 0
    endtry
    return type(s:ime_job) == v:t_job && job_status(s:ime_job) ==# 'run'
  endif
  return 0
endfunction

function! s:IMESelect(id) abort
  if s:IMEEnsure()
    call ch_sendraw(job_getchannel(s:ime_job), a:id . "\n")
  else
    call system(g:ime_switch_bin . ' ' . a:id)
  endif
endfunction

function! s:EscAction() abort
  silent! call vm#reset()
  call s:IMESelect(g:ime_en_id)
endfunction

" <nowait>: Esc 后立即触发,不等待更长序列(如 VM 的 <esc><esc>)消歧
" ttimeoutlen=10: 终止键码序列探测的等待,Esc 映射最迟 10ms 生效(默认 1000ms 会显卡)
set ttimeout ttimeoutlen=10
nnoremap <nowait> <Esc> <Esc>:call <SID>EscAction()<CR>
inoremap <nowait> <Esc> <Esc>:call <SID>EscAction()<CR>
xnoremap <nowait> <Esc> <Esc>:call <SID>EscAction()<CR>

augroup StrongEsc
  autocmd!
  " 进插入模式预热 daemon（首次含 ~50ms 框架加载，后台完成不阻塞）
  autocmd InsertEnter * call s:IMEEnsure()
  " VM 启动时用同一动作覆盖插件的 buffer-local Esc
  autocmd User visual_multi_mappings inoremap <nowait><buffer> <Esc> <Esc>:call <SID>EscAction()<CR>
  autocmd User visual_multi_mappings nnoremap <nowait><buffer> <Esc> <Esc>:call <SID>EscAction()<CR>
augroup END









```
