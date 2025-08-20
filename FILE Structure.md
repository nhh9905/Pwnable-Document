---
title: FILE Structure
tags: [tài liệu]

---

- Bài viết tham khảo:
https://hackmd.io/@trhoanglan04/bof_advanced_level#FILE-Structure-Attack
https://chovid99.github.io/posts/file-structure-attack-part-1/
http://blog.angelboy.tw/
- Bài viết này mình sẽ phân tích các struct FILE trong glibc 2.35.
# Giới thiệu
- `FILE` là kiểu dữ liệu được định nghĩa trong glibc dùng để quản lý luồng dữ liệu khi đọc hoặc ghi file.
- Là 1 struct phức tạp `_IO_FILE`.
- Các thao tác như `fopen`, `fread`, `fwrite`, `fclose` đều sử dụng struct này.
- Có 3 cấu trúc file ở libc luôn tồn tại là:
    - `_IO_2_1_stdin_`: `stdin`
    - `_IO_2_1_stdout_`: `stdout`
    - `_IO_2_1_stderr_`: `stderr`
# FILE
[FILE](https://elixir.bootlin.com/glibc/glibc-2.35/source/libio/bits/types/FILE.h#L7)
```C=
typedef struct _IO_FILE FILE;
```
# _IO_FILE
[_IO_FILE](https://elixir.bootlin.com/glibc/glibc-2.35/source/libio/bits/types/struct_FILE.h#L49)
```C=
/* The tag name of this struct is _IO_FILE to preserve historic
   C++ mangled names for functions taking FILE* arguments.
   That name should not be used in new code.  */
struct _IO_FILE
{
  int _flags;		/* High-order word is _IO_MAGIC; rest is flags. */

  /* The following pointers correspond to the C++ streambuf protocol. */
  char *_IO_read_ptr;	/* Current read pointer */
  char *_IO_read_end;	/* End of get area. */
  char *_IO_read_base;	/* Start of putback+get area. */
  char *_IO_write_base;	/* Start of put area. */
  char *_IO_write_ptr;	/* Current put pointer. */
  char *_IO_write_end;	/* End of put area. */
  char *_IO_buf_base;	/* Start of reserve area. */
  char *_IO_buf_end;	/* End of reserve area. */

  /* The following fields are used to support backing up and undo. */
  char *_IO_save_base; /* Pointer to start of non-current get area. */
  char *_IO_backup_base;  /* Pointer to first valid character of backup area */
  char *_IO_save_end; /* Pointer to end of non-current get area. */

  struct _IO_marker *_markers;

  struct _IO_FILE *_chain;

  int _fileno;
  int _flags2;
  __off_t _old_offset; /* This used to be _offset but it's too small.  */

  /* 1+column number of pbase(); 0 is unknown. */
  unsigned short _cur_column;
  signed char _vtable_offset;
  char _shortbuf[1];

  _IO_lock_t *_lock;
#ifdef _IO_USE_OLD_IO_FILE
};

struct _IO_FILE_complete
{
  struct _IO_FILE _file;
#endif
  __off64_t _offset;
  /* Wide character stream stuff.  */
  struct _IO_codecvt *_codecvt;
  struct _IO_wide_data *_wide_data;
  struct _IO_FILE *_freeres_list;
  void *_freeres_buf;
  size_t __pad5;
  int _mode;
  /* Make sure we don't get into trouble again.  */
  char _unused2[15 * sizeof (int) - 4 * sizeof (void *) - sizeof (size_t)];
};
```
- Kích thước struct:
```C=
#define _GNU_SOURCE
#include <stdio.h>

int main() {
    printf("sizeof(struct _IO_FILE) = %zu\n", sizeof(struct _IO_FILE));
    return 0;
}
```
Output: `sizeof(struct _IO_FILE) = 216`
<details>
  <summary>Note</summary>

```=
_IO_FILE_plus = {
    'i386':{
		0x0:'_flags',
		0x4:'_IO_read_ptr',
		0x8:'_IO_read_end',
		0xc:'_IO_read_base',
		0x10:'_IO_write_base',
		0x14:'_IO_write_ptr',
		0x18:'_IO_write_end',
		0x1c:'_IO_buf_base',
		0x20:'_IO_buf_end',
		0x24:'_IO_save_base',
		0x28:'_IO_backup_base',
		0x2c:'_IO_save_end',
		0x30:'_markers',
		0x34:'_chain',
		0x38:'_fileno',
		0x3c:'_flags2',
		0x40:'_old_offset',
		0x44:'_cur_column',
		0x46:'_vtable_offset',
		0x47:'_shortbuf',
		0x48:'_lock',
		0x4c:'_offset',
		0x54:'_codecvt',
		0x58:'_wide_data',
		0x5c:'_freeres_list',
		0x60:'_freeres_buf',
		0x64:'__pad5',
		0x68:'_mode',
		0x6c:'_unused2',
		0x94:'vtable'
	},

	'amd64':{
		0x0:'_flags',
		0x8:'_IO_read_ptr',
		0x10:'_IO_read_end',
		0x18:'_IO_read_base',
		0x20:'_IO_write_base',
		0x28:'_IO_write_ptr',
		0x30:'_IO_write_end',
		0x38:'_IO_buf_base',
		0x40:'_IO_buf_end',
		0x48:'_IO_save_base',
		0x50:'_IO_backup_base',
		0x58:'_IO_save_end',
		0x60:'_markers',
		0x68:'_chain',
		0x70:'_fileno',
		0x74:'_flags2',
		0x78:'_old_offset',
		0x80:'_cur_column',
		0x82:'_vtable_offset',
		0x83:'_shortbuf',
		0x88:'_lock',
		0x90:'_offset',
		0x98:'_codecvt',
		0xa0:'_wide_data',
		0xa8:'_freeres_list',
		0xb0:'_freeres_buf',
		0xb8:'__pad5',
		0xc0:'_mode',
		0xc4:'_unused2',
		0xd8:'vtable'
	}
}  
```
</details>

## `int _flags`
[int _flags](https://elixir.bootlin.com/glibc/glibc-2.35/C/ident/_flags)
- Chứa các cờ (flags) điều khiển hành vi của stream như: đọc/ghi, file bị lỗi, EOF, buffer chế độ nào...
- `0xFBAD0000`: magic number dùng để xác định đây là một FILE hợp lệ, 2 byte thấp là bit flags.
```C=
#include <stdio.h>
#include <stdint.h>

int main() {
    FILE *fp = stdin;
    int flags = *((int *)fp);  // vì _flags là field đầu tiên trong struct _IO_FILE

    printf("_flags = 0x%X\n", flags);
    return 0;
}
```
Output: `_flags = 0xFBAD2088`
Giải thích:
![image](https://hackmd.io/_uploads/SkEyPkZLgx.png)
[define](https://elixir.bootlin.com/glibc/glibc-2.35/source/libio/libio.h#L66)
```C=
/* Magic number and bits for the _flags field.  The magic number is
   mostly vestigial, but preserved for compatibility.  It occupies the
   high 16 bits of _flags; the low 16 bits are actual flag bits.  */

#define _IO_MAGIC         0xFBAD0000 /* Magic number */
#define _IO_MAGIC_MASK    0xFFFF0000
#define _IO_USER_BUF          0x0001 /* Don't deallocate buffer on close. */
#define _IO_UNBUFFERED        0x0002
#define _IO_NO_READS          0x0004 /* Reading not allowed.  */
#define _IO_NO_WRITES         0x0008 /* Writing not allowed.  */
#define _IO_EOF_SEEN          0x0010
#define _IO_ERR_SEEN          0x0020
#define _IO_DELETE_DONT_CLOSE 0x0040 /* Don't call close(_fileno) on close.  */
#define _IO_LINKED            0x0080 /* In the list of all open files.  */
#define _IO_IN_BACKUP         0x0100
#define _IO_LINE_BUF          0x0200
#define _IO_TIED_PUT_GET      0x0400 /* Put and get pointer move in unison.  */
#define _IO_CURRENTLY_PUTTING 0x0800
#define _IO_IS_APPENDING      0x1000
#define _IO_IS_FILEBUF        0x2000
                           /* 0x4000  No longer used, reserved for compat.  */
#define _IO_USER_LOCK         0x8000
```
## `struct _IO_FILE *_chain`
[struct _IO_FILE *_chain](https://elixir.bootlin.com/glibc/glibc-2.35/C/ident/_chain)
- Xâu chuỗi các đối tượng `FILE` lại với nhau, tạo thành một danh sách liên kết đơn để quản lý tất cả các stream đang mở.
- Linked list này bắt đầu từ `_IO_list_all`.
[linked list](https://elixir.bootlin.com/glibc/glibc-2.35/source/libio/genops.c#L695)
```C=
for (fp = (FILE *) _IO_list_all; fp != NULL; fp = fp->_chain)
    {
      run_fp = fp;
      if (do_lock)
	_IO_flockfile (fp);

      if (((fp->_mode <= 0 && fp->_IO_write_ptr > fp->_IO_write_base)
	   || (_IO_vtable_offset (fp) == 0
	       && fp->_mode > 0 && (fp->_wide_data->_IO_write_ptr
				    > fp->_wide_data->_IO_write_base))
	   )
	  && _IO_OVERFLOW (fp, EOF) == EOF)
	result = EOF;

      if (do_lock)
	_IO_funlockfile (fp);
      run_fp = NULL;
    }
```
## `int _fileno`
- Là số nguyên đại diện cho file descriptor (fd).
- Trả về bởi sys_open.
Ví dụ:
    - `stdin` → _fileno = 0
    - `stdout` → _fileno = 1
    - `stderr` → _fileno = 2
    - `fopen("flag.txt", "r")` → _fileno = 3 hoặc số cao hơn tùy hệ thống
# _IO_list_all
[_IO_list_all](https://elixir.bootlin.com/glibc/glibc-2.35/source/libio/stdfiles.c#L56)
```C=
struct _IO_FILE_plus *_IO_list_all = &_IO_2_1_stderr_;
```
- Struct `_IO_list_all`
```gdb=
pwndbg> p _IO_list_all
$4 = (struct _IO_FILE_plus *) 0x7ffff7e1b6a0 <_IO_2_1_stderr_>
pwndbg> p _IO_2_1_stderr_.file._chain
$5 = (struct _IO_FILE *) 0x7ffff7e1b780 <_IO_2_1_stdout_>
pwndbg> p _IO_2_1_stdout_.file._chain
$6 = (struct _IO_FILE *) 0x7ffff7e1aaa0 <_IO_2_1_stdin_>
pwndbg> p _IO_2_1_stdin_.file._chain
$7 = (struct _IO_FILE *) 0x0
```
- Vẽ hình cho dễ hiểu:
![image](https://hackmd.io/_uploads/SyfwOSIZ8gl.png)
- Struct `_IO_2_1_stderr_`
```gdb=
pwndbg> p _IO_2_1_stderr_
$8 = {
  file = {
    _flags = -72540026,
    _IO_read_ptr = 0x0,
    _IO_read_end = 0x0,
    _IO_read_base = 0x0,
    _IO_write_base = 0x0,
    _IO_write_ptr = 0x0,
    _IO_write_end = 0x0,
    _IO_buf_base = 0x0,
    _IO_buf_end = 0x0,
    _IO_save_base = 0x0,
    _IO_backup_base = 0x0,
    _IO_save_end = 0x0,
    _markers = 0x0,
    _chain = 0x7ffff7e1b780 <_IO_2_1_stdout_>,
    _fileno = 2,
    _flags2 = 0,
    _old_offset = -1,
    _cur_column = 0,
    _vtable_offset = 0 '\000',
    _shortbuf = "",
    _lock = 0x7ffff7e1ca60 <_IO_stdfile_2_lock>,
    _offset = -1,
    _codecvt = 0x0,
    _wide_data = 0x7ffff7e1a8a0 <_IO_wide_data_2>,
    _freeres_list = 0x0,
    _freeres_buf = 0x0,
    __pad5 = 0,
    _mode = 0,
    _unused2 = '\000' <repeats 19 times>
  },
  vtable = 0x7ffff7e17600 <_IO_file_jumps>
}
```
# _IO_FILE_plus
[_IO_FILE_plus](https://elixir.bootlin.com/glibc/glibc-2.35/source/libio/libioP.h#L324)
```C=
struct _IO_FILE_plus
{
  FILE file;
  const struct _IO_jump_t *vtable;
};
```
- Là phiên bản mở rộng của `_IO_FILE`, `_IO_FILE_plus` = `_IO_FILE` + `vtable`.
- `vtable` (virtual table): mảng con trỏ giúp cho các hàm hoạt động thông qua quá trình thực hiện IO.
- Các phương thức (stdin, stdout, stderr) đều dùng struct này thay vì `_IO_FILE`. Kể cả khi mở file với `fopen` thì nó cũng dùng `_IO_FILE_plus`.
- Size of this struct = 0xd0 + 8 = 0xd8.
<details>
  <summary>Tại sao chúng ta dùng phiên bản mở rộng _IO_FILE_plus thay vì _IO_FILE?</summary>

  Mục đích chính để cho IO hoạt động nhanh hơn vì có `vtable`. `vtable` được khai báo với struct `_IO_jump_t`, nơi lưu trữ các con trỏ tới các phương thức IO.
    
```gdb=
pwndbg> p _IO_file_jumps
$1 = {
  __dummy = 0,
  __dummy2 = 0,
  __finish = 0x7ffff7c8bff0 <_IO_new_file_finish>,
  __overflow = 0x7ffff7c8cdc0 <_IO_new_file_overflow>,
  __underflow = 0x7ffff7c8cab0 <_IO_new_file_underflow>,
  __uflow = 0x7ffff7c8dd60 <__GI__IO_default_uflow>,
  __pbackfail = 0x7ffff7c8f280 <__GI__IO_default_pbackfail>,
  __xsputn = 0x7ffff7c8b600 <_IO_new_file_xsputn>,
  __xsgetn = 0x7ffff7c8b2b0 <__GI__IO_file_xsgetn>,
  __seekoff = 0x7ffff7c8a8e0 <_IO_new_file_seekoff>,
  __seekpos = 0x7ffff7c8e4b0 <_IO_default_seekpos>,
  __setbuf = 0x7ffff7c8a5a0 <_IO_new_file_setbuf>,
  __sync = 0x7ffff7c8a430 <_IO_new_file_sync>,
  __doallocate = 0x7ffff7c7eb10 <__GI__IO_file_doallocate>,
  __read = 0x7ffff7c8b930 <__GI__IO_file_read>,
  __write = 0x7ffff7c8aec0 <_IO_new_file_write>,
  __seek = 0x7ffff7c8a670 <__GI__IO_file_seek>,
  __close = 0x7ffff7c8a590 <__GI__IO_file_close>,
  __stat = 0x7ffff7c8aeb0 <__GI__IO_file_stat>,
  __showmanyc = 0x7ffff7c8f420 <_IO_default_showmanyc>,
  __imbue = 0x7ffff7c8f430 <_IO_default_imbue>
}
```
</details>

# _IO_jump_t
[_IO_jump_t](https://elixir.bootlin.com/glibc/glibc-2.35/source/libio/libioP.h#L293)
- Là kiểu dữ liệu của `vtable`.
```C=
struct _IO_jump_t
{
    JUMP_FIELD(size_t, __dummy);
    JUMP_FIELD(size_t, __dummy2);
    JUMP_FIELD(_IO_finish_t, __finish);
    JUMP_FIELD(_IO_overflow_t, __overflow);
    JUMP_FIELD(_IO_underflow_t, __underflow);
    JUMP_FIELD(_IO_underflow_t, __uflow);
    JUMP_FIELD(_IO_pbackfail_t, __pbackfail);
    /* showmany */
    JUMP_FIELD(_IO_xsputn_t, __xsputn);
    JUMP_FIELD(_IO_xsgetn_t, __xsgetn);
    JUMP_FIELD(_IO_seekoff_t, __seekoff);
    JUMP_FIELD(_IO_seekpos_t, __seekpos);
    JUMP_FIELD(_IO_setbuf_t, __setbuf);
    JUMP_FIELD(_IO_sync_t, __sync);
    JUMP_FIELD(_IO_doallocate_t, __doallocate);
    JUMP_FIELD(_IO_read_t, __read);
    JUMP_FIELD(_IO_write_t, __write);
    JUMP_FIELD(_IO_seek_t, __seek);
    JUMP_FIELD(_IO_close_t, __close);
    JUMP_FIELD(_IO_stat_t, __stat);
    JUMP_FIELD(_IO_showmanyc_t, __showmanyc);
    JUMP_FIELD(_IO_imbue_t, __imbue);
};
```