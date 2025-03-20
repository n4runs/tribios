# soal_2
2\. a)


![Screenshot 2025-03-19 233226](https://github.com/user-attachments/assets/bbc3a0ac-c72b-4c56-9611-9c9e3b141b25)

disini diminta untuk membuat login.sh dan register.sh dengan masing-masing parameter harus memiliki email dan password (dan username untuk register.sh). database yang menyimpan data dari kedua script itu disimpan di data/player.csv

b)
```
if [[ ! "$email" =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]; then
        echo "Email tidak valid! Pastikan format email benar (contoh: user@example.com)."
```
pada kedua script ditambahkan constraint agar format yang dimasukkan benar, yaitu email harus memiliki tanda @ dan titik, sementara password harus memiliki minimal 8 karakter, setidaknya satu huruf kecil, satu huruf besar, dan satu angka.

c) 
```
if [[ ! "$email" =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]; then
        echo "Email tidak valid! Pastikan format email benar (contoh: user@example.com)."
    elif awk -F',' -v e="$email" '$1 == e {found=1} END {exit !found}' data/player.csv; then
        echo "Email sudah terdaftar! Silakan gunakan email lain."
    else
        break
    fi
```
lalu ditambahkan lagi satu contraint agar tidak ada duplikasi email.

d) 
```
hashed_password=$(echo -n "$password$SALT" | sha256sum | awk '{print $1}')
echo "$email,$username,$hashed_password" >> data/player.csv
```
dan untuk menjaga keamanan password, ditambahkan algoritma hashing sha256sum agar terhindar dari brute force attacks. 
Berikut adalah tampilan dari script register.sh dan login.sh jika sudah ditambahkan constraint-constraint tersebut:
### register.sh
```
#!/bin/bash

SALT="T3rr4Bl@d3!"

while true; do
    echo -n "Email: "
    read email

    if [[ ! "$email" =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]; then
        echo "Email tidak valid! Pastikan format email benar (contoh: user@example.com)."
    elif awk -F',' -v e="$email" '$1 == e {found=1} END {exit !found}' data/player.csv; then
        echo "Email sudah terdaftar! Silakan gunakan email lain."
    else
        break
    fi
done

while true; do
    echo -n "Username: "
    read username

    if [[ -z "$username" || "$username" =~ [,\ ] ]]; then
        echo "Username tidak boleh kosong atau mengandung spasi/koma."
    else
        break
    fi
done

while true; do
    echo -n "Password: "
    read -s password
    echo ""

    if [[ ${#password} -lt 8 ]]; then
        echo "Password harus memiliki minimal 8 karakter."
    elif [[ ! "$password" =~ [a-z] ]]; then
        echo "Password harus mengandung setidaknya satu huruf kecil."
    elif [[ ! "$password" =~ [A-Z] ]]; then
        echo "Password harus mengandung setidaknya satu huruf besar."
    elif [[ ! "$password" =~ [0-9] ]]; then
        echo "Password harus mengandung setidaknya satu angka."
    else
        break
    fi
done

hashed_password=$(echo -n "$password$SALT" | sha256sum | awk '{print $1}')
echo "$email,$username,$hashed_password" >> data/player.csv

echo "Pendaftaran berhasil! Akun telah dibuat."
sleep 1

```
### login.sh
```
#!/bin/bash

SALT="T3rr4Bl@d3!"

echo "Email:"
read email
echo "Password:"
read -s password

if [[ ! "$email" =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]; then
    echo "Email tidak valid. Pastikan format email benar."
    exit 1
fi

user_data=$(grep "^$email," data/player.csv)

if [ -z "$user_data" ]; then
    echo "Email tidak terdaftar."
    exit 1
fi

stored_hashed_password=$(echo "$user_data" | cut -d',' -f3)

hashed_input_password=$(echo -n "$password$SALT" | sha256sum | awk '{print $1}')

if [ "$stored_hashed_password" != "$hashed_input_password" ]; then
    echo "Password salah."
    exit 1
fi

echo "Login berhasil!"

```

e) selanjutnya, bikin shell script yang dapat memantau penggunaan CPU (dalam persentase) dan model dari CPU tersebut.
### core_monitor.sh
```
#!/bin/bash

mkdir -p ./logs

LOG_FILE="./logs/core.log"

TIMESTAMP=$(date "+%Y-%m-%d %H:%M:%S")

CPU_USAGE=$(top -bn1 | awk '/Cpu\(s\)/ {print $2 + $4}')

CPU_MODEL=$(lscpu | grep "Model name" | awk -F': ' '{print $2}')

if [[ -z "$CPU_USAGE" ]]; then
    CPU_USAGE="N/A"
fi

if [[ -z "$CPU_MODEL" ]]; then
    CPU_MODEL="Unknown CPU Model"
fi

LOG_ENTRY="[$TIMESTAMP] - Core Usage [$CPU_USAGE%] - Terminal Model [$CPU_MODEL]"

echo "$LOG_ENTRY"

sleep 1

```

f) bikin satu lagi shell script untuk memantau RAM (dalam persentase) dan juga penggunaan RAM.
### frag_monitor.sh
```
#!/bin/bash

mkdir -p ./logs

LOG_FILE="./logs/fragment.log"

TIMESTAMP=$(date "+%Y-%m-%d %H:%M:%S")

TOTAL_MEM=$(grep "MemTotal" /proc/meminfo | awk '{print $2}')
AVAILABLE_MEM=$(grep "MemAvailable" /proc/meminfo | awk '{print $2}')

if [[ -z "$TOTAL_MEM" || -z "$AVAILABLE_MEM" ]]; then
    echo "Error: Unable to read memory info" >> "$LOG_FILE"
    exit 1
fi

USED_MEM=$((TOTAL_MEM - AVAILABLE_MEM))
TOTAL_MEM_MB=$((TOTAL_MEM / 1024))
AVAILABLE_MEM_MB=$((AVAILABLE_MEM / 1024))
USED_MEM_MB=$((USED_MEM / 1024))

RAM_USAGE=$(awk "BEGIN {printf \"%.2f\", ($USED_MEM / $TOTAL_MEM) * 100}")

LOG_ENTRY="[$TIMESTAMP] - Fragment Usage [$RAM_USAGE%] - Fragment Count [$USED_MEM_MB MB] - Details [Total: $TOTAL_MEM_MB MB, Available: $AVAILABLE_MEM_MB MB]"

echo "$LOG_ENTRY"

sleep 1

```

g) agar memudahkan player mengatur jadwal pemantauan system, buat manager.sh untuk menu yang harus berisikan fungsionalitas membuat dan menghapus jadwal pemantauan CPU dan RAM, dan tambahkan opsi melihat jadwal yang sedang aktif saat ini
### manager.sh
```
add_cron_job() {
    local script_path=$1
    local job_desc=$2
    local log_file

    if [[ "$script_path" == *"core_monitor.sh" ]]; then
        log_file="$(pwd)/logs/core.log"
    elif [[ "$script_path" == *"frag_monitor.sh" ]]; then
        log_file="$(pwd)/logs/fragment.log"
    else
        echo "Error: Unknown script path!"
        return 1
    fi

    local cron_job="* * * * * $script_path>>$log_file"
    sudo systemctl restart cron

    if crontab -l 2>/dev/null | grep -q "$script_path"; then
        echo "$job_desc already scheduled!"
        return
    fi

    (crontab -l 2>/dev/null; echo "$cron_job") | crontab -
    echo "$job_desc scheduled successfully!"
}

remove_cron_job() {
    local script_path=$1
    local job_desc=$2

    if crontab -l 2>/dev/null | grep -q "$script_path"; then
        crontab -l 2>/dev/null | grep -v "$script_path" | crontab -
        echo "$job_desc removed successfully!"
    else
        echo "$job_desc is not currently scheduled."
    fi
}

view_cron_jobs() {
    echo "==================="
    echo "Active Cron Jobs:"
    echo "==================="
    crontab -l 2>/dev/null || echo "No active cron jobs."
    echo "==================="
    sleep 10
}

echo "===================="
echo "  Crontab Manager"
echo "===================="
echo "1) Add CPU [Core] Usage Monitoring"
echo "2) Remove CPU [Core] Usage Monitoring"
echo "3) Add RAM [Fragment] Usage Monitoring"
echo "4) Remove RAM [Fragment] Usage Monitoring"
echo "5) View Active Jobs"
echo "6) Exit"
echo "===================="
read -p "Enter your choice: " choice </dev/tty

case $choice in
    1) add_cron_job "$(pwd)/scripts/core_monitor.sh" "CPU Monitoring" ;;
    2) remove_cron_job "$(pwd)/scripts/core_monitor.sh" "CPU Monitoring" ;;
    3) add_cron_job "$(pwd)/scripts/frag_monitor.sh" "RAM Monitoring" ;;
    4) remove_cron_job "$(pwd)/scripts/frag_monitor.sh" "RAM Monitoring" ;;
    5) view_cron_jobs; read -p "Press Enter to continue..." ;;
    6) exit 0 ;;
    *) echo "Invalid choice. Please try again."; sleep 1 ;;
esac
```
h) dikarenakan script core_monitor.sh dan frag_monitor.sh tidak mengeluarkan output pada terminal, maka buat 2 log file, core.log dan fragment.log pada folder logs, agar output dari kedua script bisa terlihat.

![Screenshot 2025-03-20 002935](https://github.com/user-attachments/assets/9c164bda-ab09-49e4-8cc0-25470475510f)
![Screenshot 2025-03-20 002957](https://github.com/user-attachments/assets/28f234a7-f546-4dff-ab49-97bd52d9779e)

i) agar player bisa memiliki akses dari seluruh system, buat shell script terminal.sh agar system memiliki antarmuka utama yang menggabungkan semua komponen.

```
#!/bin/bash

SALT="T3rr4Bl@d3!"

main_menu() {
    while true; do
        clear
        echo "=================================="
        echo "           MAIN TERMINAL"
        echo "=================================="
        echo "1) Register New Account"
        echo "2) Login to Existing Account"
        echo "3) Exit Main Terminal"
        echo "=================================="
        read -p "Enter option [1-3]: " option </dev/tty

        case $option in
            1) ./register.sh ;;
            2) login ;;
            3) echo "Goodbye!"; exit 0 ;;
            *) echo "Invalid option. Please try again."; sleep 1 ;;
        esac
    done
}

login() {
    clear
    echo "=================================="
    echo "             Login"
    echo "=================================="
    read -p "Enter your email: " email
    read -sp "Enter your password: " password
    echo ""

    if awk -F, -v e="$email" '$1 == e {print $1, $2, $3}' data/player.csv | while read user_email user_name hashed_pass; do
        input_hashed=$(echo -n "$password$SALT" | sha256sum | awk '{print $1}')

        if [[ "$input_hashed" == "$hashed_pass" ]]; then
            echo "Login successful! Welcome, $user_name."
            sleep 1
            player_menu
            return
        fi
    done; then
        echo "Login failed. Invalid email or password."
        sleep 1
    fi
}

player_menu() {
    while true; do
        clear
        echo "=================================="
        echo "          PLAYER MENU"
        echo "=================================="
        echo "1) Crontab Manager"
        echo "2) Exit Player Menu"
        echo "=================================="
        read -r -p "Choose an option: " option </dev/tty
        case $option in
            1) ./scripts/manager.sh ;;
            2) main_menu; return ;; 
            *) echo "Invalid option. Please try again."; sleep 1 ;;
        esac
    done
}

main_menu
```
note: mengapa di terminal ini tidak memanggil ./login.sh dan membuat ulang login? karena menurut saya agar tidak bolak balik antar script mending bagian login ditambahkan lagi di terminal jadi run-nya lebih simple

kendala: -\

screenshot register.sh dari terminal: 

![image](https://github.com/user-attachments/assets/3292b61d-9f87-4143-aff0-2b938c79fea8)

screenshot login.sh dari terminal:

![image](https://github.com/user-attachments/assets/24fd95fa-757e-4dbd-84ea-49dd54cd7ac6)

screenshot tampilan dari crontab manager:

![image](https://github.com/user-attachments/assets/a6515cf2-d9d0-4758-be87-b0b83e541f3e)

screenshot struktur akhir:

![image](https://github.com/user-attachments/assets/bdf33da1-3599-48b8-9957-24541235e755)
