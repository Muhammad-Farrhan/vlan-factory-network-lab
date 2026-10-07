# Troubleshooting Log

## 1. Salah pilih kabel
- Symptom: Port tidak muncul di popup.
- Cause: Menggunakan Cross-Over bukan Straight-Through.
- Fix: Hapus kabel, pasang ulang dengan Copper Straight-Through.

## 2. Config hilang setelah reset
- Symptom: VLAN dan subinterface hilang.
- Cause: File .pkt tidak disimpan (Ctrl+S) sebelum reset.
- Fix: Ulangi konfigurasi; aktifkan Auto Save di Preferences.

## 3. PC tidak dapat IP setelah ACL
- Symptom: PC stuck 169.254.x.x.
- Cause: Lupa baris `permit udp any eq 68 any eq 67` di ACL.
- Fix: Tambahkan baris DHCP di awal ACL.

## 4. Ping timeout meskipun config benar
- Symptom: Request timed out ke gateway.
- Cause: ARP cache stale di PC setelah perubahan ACL.
- Fix: Power off/on PC di Packet Tracer.

## 5. Lupa password enable
- Symptom: Tidak bisa login setelah set `enable secret`.
- Cause: Typo atau karakter khusus `!` tidak terbaca.
- Fix: Pakai password sederhana `cisco`. Kalau sudah lupa, hapus device dan ulangi.