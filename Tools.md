---
title: Các công cụ cần tải
tags: [tài liệu]

---

- IDA: [IDA](https://hex-rays.com/ida-free)
- pwntools: [pwntools](https://docs.pwntools.com/en/stable/install.html)
> Lưu ý: Thêm câu lệnh `sudo rm -rf $(python3 -c "import sys; print(f'/usr/lib/python{sys.version_info.major}.{sys.version_info.minor}/EXTERNALLY-MANAGED')")` trước khi tải.
- gdb: `sudo apt-get install
libc6-dbg gdb valgrind`, sau đó thử `gdb` để xem chương trình đã được cài hay chưa.
- pwndbg: [pwndbg](https://github.com/pwndbg/pwndbg)
> Có thể dùng gef, tuy nhiên ko nên dùng gdb-peda vì không hỗ trợ python3.
- ROPgadget: [ROPgadget](https://github.com/JonathanSalwan/ROPgadget)
- pwninit: [pwninit](https://github.com/io12/pwninit/releases/tag/3.3.1)
> Cách tải:
> `sudo apt install patchelf`
> `sudo mv pwninit /usr/bin`
> `sudo chmod +x /usr/bin/pwninit`
- one_gadget: [one_gadget](https://github.com/david942j/one_gadget)
- Docker:
> `sudo apt install -y docker.io docker-compose`
`sudo docker --version` #kiểm tra phiên bản
`sudo docker-compose --version` #kiểm tra phiên bản
`sudo service docker status` #kiểm tra dịch vụ

> Khuyến khích tải các công cụ nêu trên (trừ IDA) phiên bản Ubuntu 22.04 trên máy ảo.