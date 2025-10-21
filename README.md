# Otomasi-Jaringan
program Otomasi Jaringan
import subprocess
import time
from datetime import datetime

LOG_FILE = "/var/log/interface_monitor.log"

def log_message(message):
    """Menulis pesan log ke file dan tampilkan di terminal"""
    timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    log_entry = f"[{timestamp}] {message}\n"
    print(log_entry.strip())
    with open(LOG_FILE, "a") as log:
        log.write(log_entry)

def get_status(interface):
    """Membaca status interface (up/down)"""
    result = subprocess.run(['cat', f'/sys/class/net/{interface}/operstate'],
                            capture_output=True, text=True)
    return result.stdout.strip()

def set_interface(interface, action):
    """Mengaktifkan atau menonaktifkan interface"""
    subprocess.run(['sudo', 'ip', 'link', 'set', interface, action])
    log_message(f"[ACTION] {interface} set {action}")

def main():
    main_if = "enp0s3"   # Interface yang dipantau
    backup_if = "dummy0" # Interface dummy yang dikontrol

    log_message("=== Monitoring enp0s3 Status Started (Control dummy0) ===")
    last_status = None

    while True:
        status = get_status(main_if)
        log_message(f"[STATUS] {main_if} is currently {status.upper()}")

        # Hanya ambil tindakan bila status berubah
        if status != last_status:
            last_status = status

            if status == "down":
                log_message(f"[WARNING] {main_if} is DOWN! Disabling {backup_if}...")
                set_interface(backup_if, "down")

            elif status == "up":
                log_message(f"[INFO] {main_if} is UP! Enabling {backup_if}...")
                set_interface(backup_if, "up")

        # Delay 5 detik sebelum cek ulang
        time.sleep(5)

if __name__ == "__main__":
    main()
