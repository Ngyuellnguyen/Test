# Test
#include <iostream>
#include <fstream>
#include <string>
#include <unordered_map>
#include <vector>
#include <ctime>
#include <cstdlib>
#include <sstream>
#include <iomanip>

using namespace std;

// Hàm tiện ích để băm mật khẩu
string bamMatKhau(const string& matKhau) {
    hash<string> hasher;
    size_t hashed = hasher(matKhau);
    return to_string(hashed);
}

// Tiện ích sinh OTP
string sinhOTP() {
    srand(time(0));
    string otp;
    for (int i = 0; i < 6; ++i) {
        otp += to_string(rand() % 10);
    }
    return otp;
}

// Lớp TaiKhoanNguoiDung
class TaiKhoanNguoiDung {
public:
    string tenDangNhap;
    string matKhauDaBam;
    string hoTen;
    string email;
    string soDienThoai;
    bool matKhauTamThoi;

    TaiKhoanNguoiDung(string ten, string matKhau, string hoVaTen, string mail, string sdt)
        : tenDangNhap(ten), matKhauDaBam(bamMatKhau(matKhau)), hoTen(hoVaTen), email(mail), soDienThoai(sdt), matKhauTamThoi(false) {}

    void thayDoiMatKhau(const string& matKhauMoi) {
        matKhauDaBam = bamMatKhau(matKhauMoi);
        matKhauTamThoi = false;
    }
};

// Lớp ViDiem
class ViDiem {
public:
    string maVi;
    double soDu;
    vector<string> lichSuGiaoDich;

    ViDiem(string id, double soDuBanDau = 0.0)
        : maVi(id), soDu(soDuBanDau) {}

    void themGiaoDich(const string& giaoDich) {
        lichSuGiaoDich.push_back(giaoDich);
    }
};

// Lớp HeThongQuanLy
class HeThongQuanLy {
private:
    unordered_map<string, TaiKhoanNguoiDung> nguoiDung;
    unordered_map<string, ViDiem> viDiem;

    bool xacThucOTP(const string& otp) {
        string otpNhap;
        cout << "Nhập OTP đã gửi: ";
        cin >> otpNhap;
        return otpNhap == otp;
    }

public:
    void dangKyNguoiDung(const string& tenDangNhap, const string& matKhau, const string& hoTen, const string& email, const string& soDienThoai) {
        if (nguoiDung.find(tenDangNhap) != nguoiDung.end()) {
            cout << "Tên đăng nhập đã tồn tại!\n";
            return;
        }
        nguoiDung[tenDangNhap] = TaiKhoanNguoiDung(tenDangNhap, matKhau, hoTen, email, soDienThoai);
        viDiem[tenDangNhap] = ViDiem(tenDangNhap);
        cout << "Đăng ký tài khoản thành công!\n";
    }

    bool dangNhap(const string& tenDangNhap, const string& matKhau) {
        if (nguoiDung.find(tenDangNhap) == nguoiDung.end()) {
            cout << "Tài khoản không tồn tại!\n";
            return false;
        }
        TaiKhoanNguoiDung& nguoi = nguoiDung[tenDangNhap];
        if (nguoi.matKhauDaBam == bamMatKhau(matKhau)) {
            cout << "Đăng nhập thành công!\n";
            if (nguoi.matKhauTamThoi) {
                cout << "Vui lòng thay đổi mật khẩu tạm thời ngay lập tức.\n";
            }
            return true;
        }
        cout << "Mật khẩu không đúng!\n";
        return false;
    }

    void thayDoiMatKhau(const string& tenDangNhap, const string& matKhauMoi) {
        if (nguoiDung.find(tenDangNhap) == nguoiDung.end()) {
            cout << "Tài khoản không tồn tại!\n";
            return;
        }
        string otp = sinhOTP();
        cout << "OTP xác minh: " << otp << endl;
        if (xacThucOTP(otp)) {
            nguoiDung[tenDangNhap].thayDoiMatKhau(matKhauMoi);
            cout << "Thay đổi mật khẩu thành công!\n";
        } else {
            cout << "Xác minh OTP thất bại!\n";
        }
    }

    void chuyenDiem(const string& tuNguoiDung, const string& denNguoiDung, double soDiem) {
        if (viDiem.find(tuNguoiDung) == viDiem.end() || viDiem.find(denNguoiDung) == viDiem.end()) {
            cout << "Một hoặc cả hai ví không tồn tại!\n";
            return;
        }

        ViDiem& viNguon = viDiem[tuNguoiDung];
        ViDiem& viDich = viDiem[denNguoiDung];

        if (viNguon.soDu < soDiem) {
            cout << "Số dư không đủ!\n";
            return;
        }

        string otp = sinhOTP();
        cout << "OTP xác minh: " << otp << endl;
        if (xacThucOTP(otp)) {
            viNguon.soDu -= soDiem;
            viDich.soDu += soDiem;

            stringstream giaoDich;
            giaoDich << fixed << setprecision(2) << "Đã chuyển " << soDiem << " điểm tới " << denNguoiDung;

            viNguon.themGiaoDich(giaoDich.str());
            viDich.themGiaoDich(giaoDich.str());

            cout << "Chuyển điểm thành công!\n";
        } else {
            cout << "Xác minh OTP thất bại!\n";
        }
    }

    void hienThiSoDu(const string& tenDangNhap) {
        if (viDiem.find(tenDangNhap) == viDiem.end()) {
            cout << "Ví không tồn tại!\n";
            return;
        }
        cout << "Số dư hiện tại: " << viDiem[tenDangNhap].soDu << " điểm\n";
    }
};

int main() {
    HeThongQuanLy heThong;

    // Đăng ký tài khoản
    heThong.dangKyNguoiDung("nguoidung_1", "matkhau123", "Người Dùng 1", "email1@example.com", "0123456789");

    // Đăng nhập
    if (heThong.dangNhap("nguoidung_1", "matkhau123")) {
        // Hiển thị số dư
        heThong.hienThiSoDu("nguoidung_1");

        // Thay đổi mật khẩu
        heThong.thayDoiMatKhau("nguoidung_1", "matkhau456");

        // Nâng số dư thủ công cho giao dịch chuyển
        heThong.chuyenDiem("nguoidung_1", "nguoidung_1", 1000.0); // Mô phỏi chuyển tự để kiểm tra
    }

    return 0;
}
