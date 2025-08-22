# Kiến trúc (architecture)
- Mỗi kiến trúc CPU có cách quản lý bộ nhớ, thanh ghi, gọi hàm, syscall, stack khác nhau. Khi làm CTF hay khai thác thực tế, chúng ta sẽ thường gặp các kiến trúc sau:
    - x86 (32-bit)/x64 (64-bit): phổ biến, đơn giản và phù hợp cho người mới học.
    - ARM (32-bit): thường gặp trong mobile/IoT/embedded CTF.
    - ARM64 (AArch64): chủ yếu ở firmware thiết bị mạng.
    - MIPS: chủ yếu ở firmware thiết bị mạng.

# Các thanh ghi (Kiến trúc x86/x64)
Các thanh ghi chính:
- eax/rax (thanh ghi tích lũy)
- ebx/rbx (thanh ghi cơ sở)
- ecx/rcx (thanh ghi đếm)
- edx/rdx (thanh ghi dữ liệu)
- ebp/rbp (thanh ghi cơ sở)
- esp/rsp (thanh ghi con trỏ stack)
- eip/rip (thanh ghi con trỏ lệnh)

# Cách hoạt động của các vùng nhớ 
- Hoạt động theo cơ chế little-endian

> Example: 
> - Input: abcdefgh
> - Stack: hgfedcba

# Lập trình assembly
- Hiểu rõ các câu lệnh cơ bản để lập trình shellcode

# Các chế độ bảo vệ của file ELF:
- RELRO: Partial, No RELRO -> Overwrite GOT
- Canary: No canary -> easy overflow
- NX: chống thực thi shellcode
- PIE: No PIE có thể dùng các hàm trong binary mà ko cần leak exe_base, PIE enabled -> leak exe_base

# Stripped file
- Công dụng: nén dữ liệu, chống dịch ngược

# Docker

# Các kĩ thuật khai thác cơ bản
## Buffer Overflow
- Dấu hiệu nhận biết:
    - Các hàm nhập input: `gets(s)`; `scanf("%s", s)`; cho phép nhập dữ liệu với số lượng lớn hơn lượng đã được cấp phát.
    - Các hàm thao tác chuỗi: `strcpy()`, `strcat()`, `sprintf()`, `strlen()`...
    - Các hàm cấp phát: `memcpy()`...

- ret2win (return to win function)
- ret2shellcode (return to shellcode)
- ret2libc (return to library C)
- stack pivot (chuyển hướng luồng thực thi, tận dụng lệnh "leave; ret;")
- bypass canary (leak canary để vượt qua lớp bảo vệ tràn bộ nhớ)
> 1 số cách bypass canary:
> - Leak canary
> - Overwrite `__stack_chk_fail()` (RELRO not FULL)
> - Overwrite `fs_base + 0x28` to fake canary base
> - Bruteforce canary

## Format String
- Dấu hiệu nhận biết: printf(s); s = %p -> printf("%p"), từ đó có thể leak địa chỉ nằm trên thanh ghi hoặc stack.
- Kết hợp %p và %n (%hn, %hhn) để ghi đè vào hàm (RELRO not FULL)
- Dùng %p leak địa chỉ
- Dùng %s leak dữ liệu trong con trỏ

Example: a -> '/bin/sh', trong đó `a` là 1 địa chỉ hợp lệ

%s -> leak /bin/sh

%p -> leak a

## Integer Overflow
Integer range: -2<sup>31</sup> -> 2<sup>31</sup> - 1
> Mẹo để nhớ: int 4 bytes -> 32 bits

Example: Input: 
- 2147483648 -> -2147483648
- 2147483649 -> -2147483647
- ...

## Heap Exploitation
- Heap là vùng nhớ được cấp phát động, trong đó: 
    - Các hàm cấp phát vùng nhớ: `malloc()`; `calloc()`; `realloc()`.
    - Hàm giải phóng vùng nhớ: `free()`.
- Heap Overflow: tràn bộ nhớ, tương tự như Buffer Overflow

```C
char *p = malloc(0x100);
free(p);
```
- Use-After-Free (UAF): tái sử dụng lại chunk đã được giải phóng
- Double-Free (DBF): sau khi ta `malloc()` 1 chunk và `free()` nó nhưng lại không xoá con trỏ sau khi `free()`, do đó tiếp tục free cùng 1 con trỏ -> DBF

-> Tận dụng điều đó để sửa chunk
- Một số phương pháp khai thác nâng cao:
    - Tcache Poisoning
    - House of Orange
    - House of Force
    - House of Spirit
    - Unlink Attack