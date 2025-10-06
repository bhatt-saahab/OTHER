#  Consensual Typing — Phone → PC (Termux ↔ Windows) 📱➡️💻

**Goal:** Type on your phone (Termux) and see each line appear on your Windows PC.
**Port used:** `9999` (changeable).
**Flow:** Run receiver on PC → run sender on phone → type on phone → text appears on PC.

> ⚠️ **Ethics reminder:** This tool only sends text you intentionally type and submit while the sender is running. Do **not** use it to capture other people’s typing without explicit permission.

---

## 1. Windows PC — Receiver

Save this as `receiver.py` (place it on your Desktop or in a project folder)

```python
#!/usr/bin/env python3
# receiver.py — run on Windows PC

import socket
import datetime

HOST = '0.0.0.0'   # listen on all interfaces
PORT = 9999        # receiver port (change if needed)

def start_receiver():
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server_socket.bind((HOST, PORT))
    server_socket.listen(1)
    print(f"[+] Listening on {HOST}:{PORT} ... (press Ctrl+C to stop)")

    conn, addr = server_socket.accept()
    print(f"[+] Connected by {addr}")

    try:
        while True:
            data = conn.recv(4096)
            if not data:
                break
            text = data.decode(errors='replace')
            ts = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
            print(f"[{ts}] Phone says: {text}")
    finally:
        conn.close()
        server_socket.close()
        print("[+] Connection closed.")

if __name__ == "__main__":
    start_receiver()
```

### Run the receiver

1. Open **Command Prompt** (`Win + R` → `cmd` → Enter).
2. Navigate to where `receiver.py` is saved, for example:

   ```cmd
   cd %USERPROFILE%\Desktop
   ```

   or:

   ```cmd
   cd C:\path\to\folder
   ```
3. Run:

   ```cmd
   python receiver.py
   ```
4. You should see:

   ```
   [+] Listening on 0.0.0.0:9999 ... (press Ctrl+C to stop)
   ```

---

## 2. Android Phone (Termux) — Sender

### Install Termux & Python (if needed)

In Termux:

```bash
pkg update -y
pkg install python -y
```

### Create `sender.py` in Termux

Open an editor (e.g. `nano sender.py`) and paste:

```python
#!/usr/bin/env python3
# sender.py — run in Termux on your phone

import socket
import argparse

DEFAULT_PORT = 9999

def run(pc_host, port=DEFAULT_PORT):
    print(f"[+] Connecting to {pc_host}:{port} ...")
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect((pc_host, port))
        print("[+] Connected. Type lines and press Enter to send. Type 'exit' to quit.")
        while True:
            line = input("> ")
            if line.strip().lower() == "exit":
                break
            if not line:
                continue
            s.sendall(line.encode("utf-8"))
    except ConnectionRefusedError:
        print("[!] Connection refused. Is the receiver running and reachable?")
    except Exception as e:
        print("[!] Error:", e)
    finally:
        try:
            s.close()
        except:
            pass
        print("[+] Sender stopped.")

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="Consensual typing sender")
    parser.add_argument("pc_host", help="IP of the PC receiver (e.g. 10.46.80.62)")
    parser.add_argument("--port", type=int, default=DEFAULT_PORT, help="Receiver port (default 9999)")
    args = parser.parse_args()
    run(args.pc_host, args.port)
```

### Save & run

```bash
python sender.py 10.46.8.62
```

(Replace `10.46.8.62` with your PC's IP if different.)

---

## 🔍 How to get your PC IP (Windows)

Open Command Prompt and run:

```cmd
ipconfig
```

Look for **IPv4 Address** under the network adapter you're using (Wi‑Fi or Ethernet). Example: `10.46.80.62`. Use that IP in `sender.py`.

---

## 🧰 Windows Firewall — Allow the port (if needed)

If the phone can't connect, allow inbound TCP port `9999`:

Open **PowerShell as Administrator** and run:

```powershell
New-NetFirewallRule -DisplayName "ConsensualTypingReceiver" -Direction Inbound -Action Allow -Protocol TCP -LocalPort 9999
```

Or add a rule manually:
`Windows Defender Firewall → Advanced settings → Inbound Rules → New Rule` → Port → TCP → 9999 → Allow.

---


