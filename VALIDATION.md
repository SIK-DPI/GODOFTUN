# Validation · GODOFTUN V1.0.0

Validated on 2026-10-01. Local tests are complete; real Linux/Cloudflare-path testing and owner approval remain required before publication. The version name is not a production certification.

This documentation revision adds four-language manuals and stricter distribution checks. The installer and both Linux executables are byte-identical to the V1.0.0 build previously tested below. The Python suites and guide checks were rerun for this revision; the Go, fuzz, process-integration and vulnerability results are retained from that unchanged build, not claimed as new runs.

The delivery archive is named `GODOFTUN-V1.0.0-PUBLIC-GITHUB.zip`; `sik-dpi` remains the repository name. The 304-test Python suite was also rerun after this naming correction, with 300 passes and the same four environment-dependent skips. Installer and binary bytes remain unchanged.

| Check | Result |
|---|---|
| Go race suite | 50 top-level tests, 45 subtests and 3 fuzz seeds passed |
| Repeated race suite | 10 complete runs passed |
| Frame-header fuzzing | 1,579,861 executions passed |
| Installer, management, distribution and guide suites | 304 collected: 300 passed; 4 skipped; no failures |
| Four-language manual | 4 READMEs and 24 guides passed structural, local-link and example-parity checks; 24 distinct shell examples in source templates, 22 after version expansion, syntax-checked without execution |
| Local process integration | 12/12 profiles passed |
| Linux amd64 / arm64 builds | Built and inspected; not executed on Linux here |
| Go vet and dependency integrity | Passed |
| Source and delivered-binary vulnerability scans | No known reachable vulnerabilities; qualification below |

The final local process test transferred 83,361,792 bytes across both directions with complete payload/FIN verification. It covered MUX on/off, Fragment on/off, three padding levels, two simultaneous input ports, slow readers, diagnostics and new connections after an egress restart. The frontend was a **local TLS proxy, not Cloudflare or Nginx**.

Separate TLS tests verified accepted ECH for Chrome, Firefox and iOS profiles. They do not establish ECH availability on every domain or network.

Four checks remain unexecuted locally: one Linux CA-path check, one `flock` check and two Nginx parser checks. Real Linux installation, systemd behavior and the actual Iran–Cloudflare–foreign route still need testing. Go 1.26.8 and macOS arm64 were used locally; certificate tests used OpenSSL 3.6.2.

Known-vulnerability scans reported zero reachable advisories and zero affected imported packages. They also reported 18 advisories in unused parts of required modules. These results are not a guarantee of absence of unknown vulnerabilities.

This release keeps the terminal UI and tunnel behavior, with the final GODOFTUN V1.0.0 name embedded in the executables and installer. Twenty-two UI tests cover menu mappings, read-only menu-preview safety, narrow layouts, dark/light colors, plain output, untrusted display text, unavailable accounting data and exact release branding. Required execution/configuration markers and dependency notices remain. Public packaging checks separate the private Go source from this binary distribution; readable installer/management scripts are deliberately included.

The public package uses an explicit allowed-file list and rejects linked inputs, missing translations and unexpected files. Archive contents and checksums are verified during packaging. These checks reduce accidental disclosure; they are not a guarantee that every possible secret format can be detected. Manual topics cover installation, Cloudflare, configuration, operations, troubleshooting and security in English, Persian, Simplified Chinese and Russian. The terminal interface remains English.

## فارسی

در بازبینی راهنماها، از ۳۰۴ تست، ۳۰۰ مورد موفق بود و چهار تست وابسته به لینوکس، قفل سیستمی و Nginx اجرا نشد. ۲۸ فایل راهنمای چهارزبانه نیز بررسی شد. فایل نصب و باینری‌ها همان V1.0.0 قبلی‌اند؛ نتایج هسته و تست ارتباط محلی از همان ساخت بدون تغییر حفظ شده‌اند. تست محلی، جای نصب واقعی و مسیر ایران–کلودفلر–خارج را نمی‌گیرد. این نسخه ابتدا باید روی سرور آزمایشی بررسی شود.

## 简体中文

本次文档修订重新运行了 304 项测试：300 项通过，4 项因 Linux、系统锁或 Nginx 环境要求而跳过。28 份四语言文档也通过了检查。安装程序和二进制文件仍是原来的 V1.0.0；核心、模糊测试、本地进程通信及漏洞扫描结果沿用同一未修改构建的已有记录，并非本次重新执行。仍须在真实 Linux 服务器和伊朗—Cloudflare—境外链路上进行验收。

## Русский

При этой редакции документации повторно запущены 304 теста: 300 прошли, 4 пропущены из-за требований к Linux, системным блокировкам или Nginx. Проверены также 28 документов на четырёх языках. Установщик и двоичные файлы остались прежними, V1.0.0; результаты проверок ядра, фаззинга, локального обмена между процессами и уязвимостей взяты из предыдущего прогона того же неизменённого выпуска, а не выданы за новые. Приёмочные испытания на реальных серверах Linux и маршруте Иран—Cloudflare—зарубежный сервер ещё необходимы.
