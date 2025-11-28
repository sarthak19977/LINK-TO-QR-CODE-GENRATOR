import os
import sys
import webbrowser
from urllib.parse import quote_plus

try:
    import requests
    _HAS_REQUESTS = True
except Exception:
    _HAS_REQUESTS = False

QR_API = "https://api.qrserver.com/v1/create-qr-code/"

def make_qr_link(data: str, size: int = 300) -> str:
    if not data:
        raise ValueError("No data provided for QR code.")
    params = f"?data={quote_plus(data)}&size={size}x{size}"
    return QR_API + params

def download_qr(link: str, out_path: str = "qrcode.png") -> str:
    if not _HAS_REQUESTS:
        raise RuntimeError("requests package not available. Install with: pip install requests")
    r = requests.get(link, stream=True, timeout=10)
    r.raise_for_status()
    with open(out_path, "wb") as f:
        for chunk in r.iter_content(8192):
            f.write(chunk)
    return os.path.abspath(out_path)

def prompt_loop():
    print("QR Link Generator")
    data = input("Enter text or URL to encode: ").strip()
    if not data:
        print("No input provided. Exiting.")
        return
    try:
        size = int(input("Enter size in pixels (default 300): ") or "300")
    except ValueError:
        size = 300
    try:
        link = make_qr_link(data, size)
    except ValueError as e:
        print("Error:", e)
        return

    print("\nGenerated QR link:")
    print(link)
    print("\nOptions:")
    print(" 1. Open link in default browser")
    print(" 2. Download PNG (requires 'requests')")
    print(" 3. Exit")
    choice = input("Choose [1/2/3]: ").strip()
    if choice == "1":
        webbrowser.open(link)
        print("Opened in browser.")
    elif choice == "2":
        if not _HAS_REQUESTS:
            print("requests not installed. Install with: pip install requests")
            return
        out_name = input("Output filename (default qrcode.png): ").strip() or "qrcode.png"
        try:
            path = download_qr(link, out_name)
            print("Saved to", path)
        except Exception as e:
            print("Download failed:", e)
    else:
        print("Done.")

if _name_ == "_main_":
    prompt_loop()
