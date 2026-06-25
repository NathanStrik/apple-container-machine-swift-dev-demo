# Swift Development in Apple Container Machine (Ubuntu + Swift 6)

## Overview

This setup creates a persistent Linux development machine on macOS using Apple’s `container machine`.

It consists of four distinct layers:

| Layer             | Name in this README  | Purpose                       |
|-------------------|----------------------|-------------------------------|
| macOS host        | host-mac             | Your real machine             |
| container image   | swift-dev-image      | Built from Dockerfile         |
| container machine | swift-dev-machine    | Persistent Linux environment  |
| runtime OS        | Ubuntu 24.04 (Noble) | Linux userland inside machine |

---

## Key idea

A container machine behaves like a lightweight Linux server VM.

Flow:

Dockerfile → container image → container machine → SSH + VS Code remote dev

---

## 1. Build container image

```Shell
container build -t swift-dev-image:latest -f Dockerfile .
```

---

## 2. Create container machine

```Shell
container machine create \
  --cpus 4 \
  --memory 16G \
  --set-default \
  --name swift-dev-machine \
  swift-dev-image:latest
```

---

## 3. SSH config

```Shell
cat >> ~/.ssh/config <<EOT

Host swift-dev.machine
   HostName swift-dev.machine
   ForwardAgent yes
   UserKnownHostsFile /dev/null
EOT
```

```Shell
sudo container system dns create machine
```

---

## 4. Set password

```Shell
container machine run -it sudo passwd $(whoami)
```

---

## 5. Verify

```Shell
container machine ls
ping -c 1 swift-dev.machine
```

---

## 6. VS Code

```Shell
Remote-SSH → swift-dev.machine
```

---

## 7. Open project
```Shell
git clone https://github.com/swiftlang/swift-server-todos-tutorial.git
```

---

## 8. Build

```Shell
swift build
```

---

## 9. Run
```Shell
curl http://swift-dev.machine:8080/todos
```

You can optionally set a breakpoint in the `Telemetry.swift` file and set a breakpoint on the innermost statement of the `RequestLoggerInjectionMiddleware.respond()` function.

---

## Cleanup
```Shell
container machine stop swift-dev-machine
container machine rm swift-dev-machine
container image rm swift-dev-image:latest
```
