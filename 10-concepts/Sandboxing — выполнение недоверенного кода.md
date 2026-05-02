---
title: Sandboxing — выполнение недоверенного кода
aliases: ["Sandbox", "Same-Origin Policy", "Изоляция"]
type: concept
layer: security
chapter: 8
difficulty: intermediate
prerequisites: ["[[Безопасность сетей — обзор]]"]
related: ["[[XSS]]", "[[Web — архитектура]]"]
tags: [networking, ch08]
---
# Sandboxing — выполнение недоверенного кода

## TL;DR
Изоляция выполняющегося кода от остальной системы: **JavaScript** в браузере (Same-Origin Policy + JS-engine sandbox), **Java applets** (исторически, JVM-securityManager), **WebAssembly** (явный capability-based design), **процессная изоляция ОС** (chroot, namespaces, seccomp), **VM/контейнеры**. Цель: даже **maliciously скомпрометированный** код **не может** выйти за рамки sandbox'а.

## Какую проблему решает
Современные web/mobile/cloud системы запускают код **из ненадёжных источников**: страницы скачиваются с любых сайтов, mobile-apps от тысяч разработчиков, cloud-tenants. Если код вырывается за рамки своих привилегий → catastrophe (украсть данные, получить root, lateral movement).

## Как работает

### Browser sandbox

**JavaScript sandbox** (V8, SpiderMonkey):
- JS не может читать произвольные файлы.
- Не может делать sockets к произвольным хостам (только через fetch/WebSocket с CORS-проверкой).
- Не может выполнять native code (без plugins или WASM).

**Same-Origin Policy** (SOP):
- JS из `https://a.com` не может **читать** ответ запроса к `https://b.com`.
- Базис web-security.
- Cookies, localStorage — **per-origin**.

**CORS** (Cross-Origin Resource Sharing): mechanism для **явного разрешения** cross-origin доступа со стороны target-сервера.

**iframe sandbox** (HTML5): `<iframe sandbox="allow-scripts">` — фиктивный origin для нём.

### Process-level

**chroot** (Unix, 1979): фиктивный root-FS — process не видит остальное. Слабая (root может выйти).

**Linux namespaces** (с 2002): отдельные view'ы для PID/network/mount/user/IPC/UTS/cgroup. Базис containers.

**seccomp** (secure computing): фильтр system calls — process может вызывать только разрешённые.

**capabilities** (POSIX.1e): granular root-привилегии (CAP_NET_ADMIN, CAP_SYS_ADMIN), вместо "всё или ничего".

### Hardware-level

**Virtualization** (VMs): Intel VT-x, AMD-V — аппаратная изоляция; full OS внутри.

**Trusted Execution Environment (TEE):** SGX (Intel), TrustZone (ARM) — изолированные enclaves для кода и данных, защищённые от ОС.

### WebAssembly (WASM)
- **Capability-based by design.**
- Код может только вызывать **импортированные** функции — host решает, что разрешить.
- Линейная память; нет доступа к браузерным API без явного импорта.
- Современная alternative для performance-sensitive web-apps.

## Пример
**Chrome multi-process architecture:**
- **Renderer** (rendering JS, HTML) запускается в **sandboxed process** с минимальными правами.
- **Browser process** (privileged) общается с renderer'ом через IPC.
- При компрометации renderer'а атакующий **не может** прямо обращаться к ОС — только через IPC, проверяемый browser-процессом.
- Это спасло Chrome от десятков 0-day-exploits.

**Docker container:**
- Linux namespaces + cgroups + seccomp + capabilities.
- Контейнер видит свой PID-tree, network-stack, files.
- Не bullet-proof: kernel-vuln может escape.

**iOS app sandbox:**
- Каждое приложение в своём container'е.
- Доступ к camera/contacts/photos только через explicit user permission.
- Strong sandbox делает iOS hard для malware.

## Связи
- **Базируется на:** [[Безопасность сетей — обзор]] (defense-in-depth), ОС-механизмы изоляции.
- **Используется в:** все современные browsers, mobile OS, container platforms; [[XSS]] (защита от — JS sandbox), [[Web — архитектура]] (SOP).
- **Соседи по уровню:** **TEE/SGX** — hardware-enforced; **microservice mesh** — process-level isolation в DC.
- **Противопоставляется:** «trusted code» — нет такого; всё может быть скомпрометировано.

## Подводные камни
- **Sandbox-escape vulnerabilities** в браузерах — самые ценные exploits ($100k+ at Pwn2Own). Chrome zero-days regularly.
- **Container escapes** — runc/Docker имели CVE'ы; Kubernetes без gVisor/Kata уязвим к kernel-vulns.
- **CPU side-channel attacks** (Spectre/Meltdown 2018) — bypass sandboxes через timing.
- **Trusting hardware** — Intel ME, AMD PSP имеют свои уязвимости и backdoors.

## Дальше читать
- [[XSS]] — типичная сandboxed-environment атака.
- [[Безопасность сетей — обзор]] — общий контекст.
- Tanenbaum, гл. 8, §8.12.4 (стр. PDF 935–938).
