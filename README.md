import matplotlib.pyplot as plt
import numpy as np

# Chuyển đổi 6 kí tự "NAMDIH" sang unicode 8 bit 
def chuyen_doi(chuoi_vao): 
    nhiphan_list = []
    for char in chuoi_vao:
        nhiphan_char = format(ord(char), '08b')
        nhiphan_list.append(nhiphan_char)
    return ''.join(nhiphan_list)

# Tìm phần check bit CRC
def tinh_CRC(data, da_thuc_sinh):
    data = list(data)
    da_thuc_sinh = list(da_thuc_sinh)
    data.extend(['0'] * (len(da_thuc_sinh) - 1))

    for i in range(len(data) - len(da_thuc_sinh) + 1):
        if data[i] == '1':
            for j in range(len(da_thuc_sinh)):
                data[i + j] = str(int(data[i + j]) ^ int(da_thuc_sinh[j]))
    return ''.join(data[-(len(da_thuc_sinh) - 1):])

# Tính dải làm việc bên nhận
def tinh_dai(A1, A2, Nmax):
    A1_lower, A1_upper = A1 - Nmax, A1 + Nmax
    A2_lower, A2_upper = A2 - Nmax, A2 + Nmax
    # Vẽ dải làm việc bên nhận:
    x = np.array(['A1', 'A2'])
    y = np.array([A1, A2])
    plt.bar(x, y, color=['blue', 'green'], label='Điện áp A1 và A2')
    plt.plot(['A1', 'A1'], [A1_lower, A1_upper], color='red', linestyle='--', label='Dải làm việc A1\'')
    plt.fill_betweenx([A1_lower, A1_upper], -0.5, 0.5, color='red', alpha=0.1)
    plt.plot(['A2', 'A2'], [A2_lower, A2_upper], color='orange', linestyle='--', label='Dải làm việc A2\'')
    plt.fill_betweenx([A2_lower, A2_upper], 1.5, 2.5, color='orange', alpha=0.1)
    plt.ylim(0, max(y) + 2)
    plt.ylabel('Điện áp (V)')
    plt.title('Biểu đồ mức xung điện áp A1 và A2 với dải làm việc')
    plt.grid(axis='y')
    plt.legend()
    plt.show()

# Chuyển dữ liệu sang tín hiệu theo mã đường NRZ-I
def dap_NRZI(data_CRC, A1, A2):
    tthai_bandau = A1
    ket_qua = []
    for i in data_CRC:
        if i == '1':
            ket_qua.append(tthai_bandau)
        elif i == '0':
            tthai_bandau = A2 if tthai_bandau == A1 else A1
            ket_qua.append(tthai_bandau)
    return ket_qua

# Biểu diễn các tín hiệu theo mã NRZ-I
def ve_NRZI(NRZI_output):
    plt.figure(figsize=(14, 4))
    plt.title('Biểu diễn tín hiệu theo mã NRZ-I')
    x = np.arange(len(NRZI_output) + 1)  # Thêm 1 để phù hợp với số lượng xung
    plt.step(x, [NRZI_output[0]] + list(NRZI_output), where='post', label='Mã NRZI', color='green')
    plt.xlabel('Thời gian (t)')
    plt.ylabel('Điện áp (V)')
    plt.ylim(0, max(NRZI_output) + 1)
    plt.xlim(0, len(NRZI_output))
    plt.legend()
    plt.grid()
    plt.show()

# Biểu diễn các tín hiệu sau khi chịu tác động của nhiễu trên kênh truyền
def ve_sau_nhieu(NRZI_output, nhieu):
    noise_2 = np.array(NRZI_output) + np.array(nhieu)
    plt.figure(figsize=(14, 8))

    # Vẽ tín hiệu theo mã đường NRZ-I 
    plt.subplot(3, 1, 1)
    plt.title('Mã NRZI Output')
    x = np.arange(len(NRZI_output) + 1)
    plt.step(x, list(NRZI_output)+[NRZI_output[1]], where='post', label='Mã NRZI', color='green')
    plt.xlabel('Thời gian (t)')
    plt.ylabel('Điện áp (V)')
    plt.ylim(0, max(NRZI_output) + 1)
    plt.xlim(0, len(NRZI_output))
    plt.legend()
    plt.grid()

    # Vẽ các mức nhiễu trên kênh truyền
    plt.subplot(3, 1, 2)
    plt.title('Mức nhiễu')
    plt.step(x,  list(nhieu)+[nhieu[1]], where='post', label='Nhiễu', color='orange')
    plt.xlabel('Thời gian (t)')
    plt.ylabel('Điện áp (V)')
    plt.ylim(min(nhieu) - 1, max(nhieu) + 1)
    plt.xlim(0, len(nhieu))
    plt.grid(True)
    plt.legend()

    # Vẽ các tín hiệu sau khi chịu tác động của nhiễu
    plt.subplot(3, 1, 3)
    plt.title('Tín hiệu sau khi chịu tác động của nhiễu')
    plt.step(x,  list(noise_2)+[noise_2[1]], where='post', label='Điện áp (V)', color='blue')
    plt.xlabel('Thời gian (t)')
    plt.ylabel('Điện áp (V)')
    plt.ylim(min(noise_2) - 1, max(noise_2) + 1)
    plt.xlim(0, len(noise_2))
    plt.grid(True)
    plt.legend()

    plt.tight_layout()
    plt.show()

# Hàm chuyển đổi tín hiệu thành dữ liệu
def dap_to_bits(A1, A2, Nmax, sau_nhieu, tt_bandau):
    bits = []
    tt_hientai = tt_bandau
    for i in sau_nhieu:
        if tt_hientai > A1 - Nmax and tt_hientai < A1 + Nmax:
            if i > A1 - Nmax and i < A1 + Nmax:
                bits.append(1)
            elif i > A2 - Nmax and i < A2 + Nmax:
                bits.append(0)
                tt_hientai = i
            else:
                bits.append('_')
        elif tt_hientai > A2 - Nmax and tt_hientai < A2 + Nmax:
            if i > A2 - Nmax and i < A2 + Nmax:
                bits.append(1)
            elif i > A1 - Nmax and i < A1 + Nmax:
                bits.append(0)
                tt_hientai = i
            else:
                bits.append('_')
        else:
            bits.append('_')
    return bits

# Kiểm tra crc bên nhận
def kiem_tra_crc(kqua_bits_string, da_thuc_sinh):
    phan_du = tinh_CRC(kqua_bits_string, da_thuc_sinh)
    if phan_du == '0' * (len(da_thuc_sinh) - 1):
        return 1
    else:
        return 0

def main():
    print("\====== HỆ THỐNG TRUYỀN THÔNG CÓ:======\n")
    # Nhập chuỗi kí tự
    chuoi_vao = input("Nhập vào 6 ký tự: ")
    while len(chuoi_vao) != 6:
        chuoi_vao = input("Nhập lại đúng 6 ký tự: ")
    # Nhập các mức điện áp
    A1 = float(input("Nhập giá trị điện áp A1 (V): "))
    A2 = float(input("Nhập giá trị điện áp A2 (V): "))
    while (A1 < 0 and A2 > 0) or (A1 > 0 and A2 < 0):
        A1, A2 = map(float, input("Nhập lại a1 và a2 (cách nhau bởi dấu cách): ").split())
    # Nhập đa thức sinh 
    da_thuc_sinh = input("Nhập đa thức sinh G(x): ")
    # Nhập các mức nhiễu 
    nhieu_input = input("Nhập dãy nhiễu các giá trị điện áp cách nhau bởi dấu cách (nhập đủ 64 mức): ")
    nhieu = [float(x) for x in nhieu_input.split(' ')]
    print("\n=============== BÊN GỬI =================\n")
    # In chuỗi dữ liệu 
    data = chuyen_doi(chuoi_vao)
    print("Dữ liệu bên gửi :", data)
    # In mức điện áp chọn 
    print(f"Giá trị điện áp A1 - A2: {A1} V - {A2} V")
    # Tính Nmax 
    SNR_min = 6.02
    Nmax = round((A1 - A2) / (10 ** (SNR_min / 20)), 2)
    # IN Nmax và dải làm việc:
    print(f"Nmax = {Nmax} V")
    tinh_dai(A1, A2, Nmax)
    # Tính T(x) và in 
    data_CRC = data + tinh_CRC(data, da_thuc_sinh) 
    print("Dữ liệu sau qua mã kiểm soát lỗi CRC:", data_CRC)
    # IN các dữ liệu chuyển sang tín hiệu theo mã đường 
    tt_bandau = A1
    NRZI_output = dap_NRZI(data_CRC, A1, A2)
    ve_NRZI(NRZI_output)
    print("\n============ KÊNH TRUYỀN =============\n")
    # IN mức nhiễu 
    print("Dãy nhiễu đã nhập:", nhieu)
    # Biểu diễn tín hiệu sau khi chịu tác động của nhiễu 
    ve_sau_nhieu(NRZI_output, nhieu)
    # Tính các mức điện áp sau nhiễu và in ra
    sau_nhieu = [a + b for a, b in zip(NRZI_output, nhieu)]
    print("Dữ liệu điện áp sau khi lọc nhiễu:", sau_nhieu)
    print("\n============ BÊN NHẬN===================\n")
    print(f"Dải làm việc A1': {A1 - Nmax} V < A1' < {A1 + Nmax} V")
    print(f"Dải làm việc A2': {A2 - Nmax} V < A2' < {A2 + Nmax} V")
    # Mã đường NRZ-I và đổi các tín hiệu thành dữ liệu
    kq_bits = dap_to_bits(A1, A2, Nmax, sau_nhieu, tt_bandau)
    kqua_bits_string = ''.join(str(bit) for bit in kq_bits)
    print("Dữ liệu bên nhận", kqua_bits_string)
    if '_' in kq_bits:    
        print("Dữ liệu bên nhận sau khi mã hóa bị thiếu mà mã đường NRZI phát hiện được")
    # Mã kiểm soát lỗi CRC
    else:   
        k = kiem_tra_crc(kqua_bits_string, da_thuc_sinh)
        if k == 1:
            print("Dữ liệu bên nhận sau khi mã hóa đúng")
        else:
            print("Dữ liệu bên nhận sau khi mã hóa bị sai mà mã kiểm soát lỗi CRC phát hiện được")

if __name__ == "__main__":
    main()
