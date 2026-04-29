<template>
  <div class="admin-profile-page">
    <div class="page-header">
      <h1>Hồ sơ Môi giới</h1>
    </div>

    <div class="profile-content">
      <div class="profile-sidebar">
        <div class="profile-card">
          <div class="profile-avatar-wrapper">
            <img :src="profile.avatar || defaultAvatar" alt="Avatar" class="profile-avatar" />
          </div>

          <h2 class="profile-name">{{ profile.ten || "Chưa cập nhật" }}</h2>
          <p class="profile-role">
            {{ profile.mo_ta || "Chuyên viên tư vấn cao cấp" }}
          </p>

          <div class="profile-info-list">
            <div class="info-item">
              <span class="label">MÃ MÔI GIỚI</span>
              <span class="value">MG-{{ profile.id }}</span>
            </div>
            <div class="info-item">
              <span class="label">NGÀY GIA NHẬP</span>
              <span class="value">{{ formatDate(profile.created_at) }}</span>
            </div>
            <div class="info-item">
              <span class="label">TRẠNG THÁI</span>
              <span class="value status-active">Đang hoạt động</span>
            </div>
          </div>
        </div>

        <!-- Thay thế card security-card cũ bằng code này -->
        <div class="security-card">
          <div class="security-header">
            <div class="security-icon" style="background: #e8f5e9; color: #10b981">
              <i class="bi bi-currency-dollar"></i>
            </div>
            <h3>Tổng tiền đã mua gói</h3>
          </div>

          <div class="mt-2 mb-3">
            <h2 style="
                font-size: 2rem;
                font-weight: 800;
                color: #1a1a1a;
                margin: 0;
              ">
              {{ formatCurrency(stats.tongTien) }}
              <span style="font-size: 1rem; color: #6c757d">VND</span>
            </h2>
            <!-- <p
              class="security-desc"
              style="color: #10b981; margin-bottom: 10px; font-weight: 600"
            >
              {{ stats.soGoiDaMua || 0 }} gói đã mua
            </p> -->
          </div>

          <!-- Progress bar thể hiện % tin đã dùng -->
          <div class="toggle-section" style="flex-direction: column; align-items: flex-start; gap: 8px">
            <div style="
                width: 100%;
                display: flex;
                justify-content: space-between;
                font-size: 0.85rem;
              ">
              <span style="color: #6c757d">Tin đã sử dụng</span>
              <span style="font-weight: 600">
                {{ stats.tinDaDang || 0 }} /
                {{ (stats.tinDaDang || 0) + (stats.tinConLai || 0) }}
              </span>
            </div>
            <div style="
                width: 100%;
                height: 6px;
                background: #e2e8f0;
                border-radius: 4px;
              ">
              <div :style="{
                width: usagePercent + '%',
                height: '100%',
                background: usagePercent > 80 ? '#f59e0b' : '#10b981',
                borderRadius: '4px',
                transition: 'width 0.3s',
              }"></div>
            </div>
          </div>
        </div>

        <!-- Thay thế phần grid 2 cột cũ bằng code này -->
        <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 16px">
          <!-- ✅ Số tin đã đăng -->
          <div class="security-card" style="padding: 16px; text-align: center">
            <i class="bi bi-file-earmark-check fs-4 text-primary" style="color: #0a0a4d"></i>
            <p style="
                font-size: 0.75rem;
                color: #6c757d;
                margin: 8px 0 4px;
                font-weight: 600;
                text-transform: uppercase;
              ">
              Số tin đã đăng
            </p>
            <h3 style="
                margin: 0;
                font-size: 1.5rem;
                font-weight: 800;
                color: #1a1a1a;
              ">
              {{ stats.tinDaDang || 0 }}
            </h3>
          </div>

          <!-- ✅ Số tin còn lại -->
          <div class="security-card" style="padding: 16px; text-align: center">
            <i class="bi bi-inbox fs-4 text-success" style="color: #10b981"></i>
            <p style="
                font-size: 0.75rem;
                color: #6c757d;
                margin: 8px 0 4px;
                font-weight: 600;
                text-transform: uppercase;
              ">
              Số tin còn lại
            </p>
            <h3 style="
                margin: 0;
                font-size: 1.5rem;
                font-weight: 800;
                color: #10b981;
              ">
              {{ stats.tinConLai || 0 }}
            </h3>
          </div>
        </div>
        <button @click="confirmLogoutAll" class="btn btn-danger">
          <i class="bi bi-shield-lock"></i>
          Đăng xuất tất cả thiết bị
        </button>
      </div>

      <div class="profile-main">
        <div class="info-card">
          <div class="card-header">
            <div class="card-title">
              <span class="title-bar"></span>
              <h3>Thông tin cá nhân</h3>
            </div>
            <button class="btn-save" @click="saveProfile" :disabled="loading">
              <i class="bi bi-save me-1"></i>
              {{ loading ? "Đang lưu..." : "Lưu thay đổi" }}
            </button>
          </div>

          <div class="card-body">
            <div class="form-row">
              <div class="form-group">
                <label>HỌ VÀ TÊN</label>
                <input type="text" v-model="profile.ten" class="form-control" :class="{ 'is-invalid': errors.ten }"
                  placeholder="Nhập họ và tên" />
                <div v-if="errors.ten" class="invalid-feedback">
                  {{ Array.isArray(errors.ten) ? errors.ten[0] : errors.ten }}
                </div>
              </div>

              <!-- Email -->
              <div class="form-group">
                <label>EMAIL</label>
                <input type="email" v-model="profile.email" class="form-control" :class="{ 'is-invalid': errors.email }"
                  placeholder="email@example.com" />
                <div v-if="errors.email" class="invalid-feedback">
                  {{
                    Array.isArray(errors.email) ? errors.email[0] : errors.email
                  }}
                </div>
              </div>
            </div>

            <div class="form-row">
              <!-- Số điện thoại -->
              <div class="form-group">
                <label>SỐ ĐIỆN THOẠI</label>
                <input type="tel" v-model="profile.so_dien_thoai" class="form-control"
                  :class="{ 'is-invalid': errors.so_dien_thoai }" placeholder="+84 xxx xxx xxx" />
                <div v-if="errors.so_dien_thoai" class="invalid-feedback">
                  {{
                    Array.isArray(errors.so_dien_thoai)
                      ? errors.so_dien_thoai[0]
                      : errors.so_dien_thoai
                  }}
                </div>
              </div>

              <!-- Link Zalo -->
              <div class="form-group">
                <label>LINK ZALO</label>
                <input type="text" v-model="profile.zalo_link" class="form-control"
                  :class="{ 'is-invalid': errors.zalo_link }" placeholder="https://zalo.me/..." />
                <div v-if="errors.zalo_link" class="invalid-feedback">
                  {{
                    Array.isArray(errors.zalo_link)
                      ? errors.zalo_link[0]
                      : errors.zalo_link
                  }}
                </div>
              </div>
            </div>

            <!-- Mô tả -->
            <div class="form-group">
              <label>MÔ TẢ</label>
              <textarea v-model="profile.mo_ta" class="form-control" :class="{ 'is-invalid': errors.mo_ta }"
                placeholder="Mô tả ngắn gọn..." rows="3"></textarea>
              <div v-if="errors.mo_ta" class="invalid-feedback">
                {{
                  Array.isArray(errors.mo_ta) ? errors.mo_ta[0] : errors.mo_ta
                }}
              </div>
            </div>
          </div>
        </div>

        <div class="info-card">
          <div class="card-header">
            <div class="card-title">
              <span class="title-bar"></span>
              <h3>Thay đổi mật khẩu</h3>
            </div>
          </div>

          <div class="card-body">
            <div class="form-row">
              <div class="form-group">
                <label>MẬT KHẨU HIỆN TẠI</label>
                <div class="password-input">
                  <input :type="showCurrentPassword ? 'text' : 'password'" v-model="passwordForm.old_password"
                    class="form-control" placeholder="••••••••" />
                  <button type="button" class="password-toggle" @click="showCurrentPassword = !showCurrentPassword">
                    <i :class="showCurrentPassword ? 'bi bi-eye-slash' : 'bi bi-eye'
                      "></i>
                  </button>
                </div>
              </div>

              <div class="form-group">
                <label>MẬT KHẨU MỚI</label>
                <input type="password" v-model="passwordForm.password" class="form-control"
                  placeholder="Mật khẩu mới" />
              </div>
            </div>

            <div class="form-group">
              <label>XÁC NHẬN MẬT KHẨU</label>
              <input type="password" v-model="passwordForm.re_password" class="form-control"
                placeholder="Nhập lại mật khẩu" />
            </div>

            <div class="password-hint">
              <em>Lưu ý: Mật khẩu nên bao gồm ít nhất 8 ký tự, bao gồm cả chữ
                hoa, chữ thường, số và ký tự đặc biệt.</em>
            </div>

            <button class="btn-update-password" @click="changePassword">
              Cập nhật mật khẩu
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- Security Alert Modal -->
  <div v-if="showLogoutAllModal" class="modal fade show d-block">
    <div class="modal-dialog modal-dialog-centered">
      <div class="modal-content">
        <div class="modal-header bg-danger text-white">
          <h5 class="modal-title">
            <i class="bi bi-exclamation-triangle"></i>
            Xác nhận đăng xuất tất cả
          </h5>
          <button type="button" class="btn-close" @click="closeModal"></button>
        </div>
        <div class="modal-body">
          <p><strong>Hành động này sẽ:</strong></p>
          <ul>
            <li>Đăng xuất bạn khỏi <b>TẤT CẢ</b> thiết bị đang đăng nhập</li>
            <!-- <li>Hủy tất cả token hiện tại</li> -->
            <li>Bạn sẽ cần đăng nhập lại trên mọi thiết bị</li>
          </ul>
          <p class="text-danger fw-bold">Bạn có chắc chắn muốn tiếp tục?</p>
        </div>
        <div class="modal-footer">
          <button type="button" class="btn btn-secondary" @click="closeModal">
            Hủy
          </button>
          <button type="button" class="btn text-center btn-danger" @click="logoutAll">
            <i class="bi bi-shield-lock"></i>
            Đăng xuất tất cả
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import axios from "axios";
import Swal from "sweetalert2";
import { useRouter } from "vue-router"; // ✅ 1. Import router

export default {
  data() {
    return {
      loading: false,
      showLogoutAllModal: false,
      showCurrentPassword: false,
      errors: {},
      profile: {
        id: "",
        ten: "",
        email: "",
        so_dien_thoai: "",
        zalo_link: "",
        dia_chi: "",
        khu_vuc_hoat_dong: "",
        mo_ta: "",
        created_at: "",
      },
      passwordForm: {
        old_password: "",
        password: "",
        re_password: "",
      },

      stats: {
        tinDaDang: 0, // Số tin đã đăng
        tinConLai: 0, // Số tin còn lại
        tongTien: 0, // Tổng tiền đã mua gói
        soGoiDaMua: 0, // Số gói đã mua
      },

      defaultAvatar:
        "https://ui-avatars.com/api/?name=Moi+Gioi&background=0a0a4d&color=fff&rounded=true&size=200",
      token: localStorage.getItem("auth_token"),
    };
  },

  // ✅ 2. Khởi tạo router
  setup() {
    const router = useRouter();
    return { router };
  },

  computed: {
    // ✅ Tính % tin đã sử dụng cho progress bar
    usagePercent() {
      const total = (this.stats.tinDaDang || 0) + (this.stats.tinConLai || 0);
      if (total === 0) return 0;
      return Math.round((this.stats.tinDaDang / total) * 100);
    },
  },

  mounted() {
    this.fetchProfile();
    this.fetchStats(); // ✅ Load stats khi component mount
  },

  methods: {
    formatDate(dateString) {
      if (!dateString) return "-";
      const date = new Date(dateString);
      const day = String(date.getDate()).padStart(2, "0");
      const month = String(date.getMonth() + 1).padStart(2, "0");
      const year = date.getFullYear();
      return `${day}/${month}/${year}`;
    },

    // ✅ Format tiền VND
    formatCurrency(value) {
      if (!value && value !== 0) return "0";
      return new Intl.NumberFormat("vi-VN").format(value);
    },

    // ✅ Lấy thống kê từ API - THÊM DEBUG
    async fetchStats() {
      try {
        const token = this.token;
        const headers = { Authorization: `Bearer ${token}` };

        console.log("📡 Calling stats APIs...");

        // Gọi song song các API stats
        const [tinDaDang, tinConLai, tongTien] = await Promise.all([
          axios.get(
            "http://localhost:8000/api/moi-gioi/thong-ke/tong-tin-da-dang",
            { headers }
          ),
          axios.get("http://localhost:8000/api/moi-gioi/thong-ke/tin-con-lai", {
            headers,
          }),
          axios.get("http://localhost:8000/api/moi-gioi/thong-ke/tong-tien", {
            headers,
          }),
        ]);

        console.log("📊 tinDaDang response:", tinDaDang.data);
        console.log("📊 tinConLai response:", tinConLai.data);
        console.log("📊 tongTien response:", tongTien.data);

        // ✅ Gán dữ liệu ĐÚNG FORMAT từ backend
        this.stats = {
          tinDaDang:
            typeof tinDaDang.data?.data === "number" ? tinDaDang.data.data : 0,
          tinConLai:
            typeof tinConLai.data?.data === "number" ? tinConLai.data.data : 0,
          tongTien: Number(tongTien.data?.data) || 0,  // ✅ Fix ở đây
          soGoiDaMua: 0,
        };

        console.log("✅ Stats loaded:", this.stats);
      } catch (error) {
        console.error("❌ Lỗi load stats:", error);
        console.error("Response:", error.response?.data);
      }
    },

    // ✅ Đóng modal (sửa cách gọi Bootstrap modal)
    closeModal() {
      this.showLogoutAllModal = false;
      // Nếu dùng Bootstrap JS để ẩn modal vật lý (nếu có)
      const modalEl = document.querySelector(".modal");
      if (modalEl) {
        modalEl.classList.remove("show");
        modalEl.style.display = "none";
        document.body.classList.remove("modal-open");
        document.body.style.overflow = "";
        const backdrop = document.querySelector(".modal-backdrop");
        if (backdrop) backdrop.remove();
      }
    },

    // ✅ Xác nhận hiện modal
    confirmLogoutAll() {
      this.showLogoutAllModal = true;
      // Hiển thị modal bằng Bootstrap JS nếu cần
      this.$nextTick(() => {
        const modalEl = document.querySelector(".modal");
        if (modalEl && window.bootstrap?.Modal) {
          const modal = new window.bootstrap.Modal(modalEl);
          modal.show();
        }
      });
    },

    // ✅ Đăng xuất tất cả thiết bị (FIXED)
    async logoutAll() {
      try {
        // ✅ Tạo api instance tạm thời hoặc dùng axios trực tiếp
        const api = axios.create({
          baseURL: "http://localhost:8000/api",
          headers: {
            Accept: "application/json",
            "Content-Type": "application/json",
            Authorization: `Bearer ${this.token}`,
          },
        });

        const response = await api.post("http://localhost:8000/api/moi-gioi/dang-xuat-tat-ca");

        if (response.data.status === "success") {
          localStorage.removeItem("auth_token");
          localStorage.removeItem("user_type");
          localStorage.removeItem("user_info");

          Swal.fire({
            icon: "success",
            title: "Thành công",
            text: "Đã đăng xuất từ tất cả thiết bị!",
            timer: 2000,
            showConfirmButton: false,
          });

          setTimeout(() => {
            this.router.push("/moi-gioi/dang-nhap"); // ✅ Dùng this.router
          }, 2000);
        }
      } catch (error) {
        console.error("Logout error:", error);

        localStorage.removeItem("auth_token");
        localStorage.removeItem("user_type");
        localStorage.removeItem("user_info");

        Swal.fire({
          icon: "error",
          title: "Lỗi",
          text: "Có lỗi xảy ra khi đăng xuất",
        });

        setTimeout(() => {
          this.router.push("/moi-gioi/dang-nhap"); // ✅ Dùng this.router
        }, 2000);
      } finally {
        this.closeModal(); // ✅ Thêm 'this.'
      }
    },

    // ... giữ nguyên các method khác (fetchProfile, saveProfile, changePassword) ...
    async fetchProfile() {
      try {
        const res = await axios.get(
          "http://localhost:8000/api/moi-gioi/profile",
          {
            headers: { Authorization: `Bearer ${this.token}` },
          }
        );
        if (res.data.status) {
          Object.assign(this.profile, res.data.data);
        }
      } catch (error) {
        Swal.fire("Lỗi", "Không tải được hồ sơ", "error");
      }
    },

    async saveProfile() {
      this.loading = true;
      this.errors = {};

      try {
        const res = await axios.post(
          "http://localhost:8000/api/moi-gioi/update-profile",
          this.profile,
          {
            headers: { Authorization: `Bearer ${this.token}` },
          }
        );

        if (res.data.status === 1) {
          Swal.fire(
            "Thành công",
            res.data.message || "Cập nhật hồ sơ thành công",
            "success"
          );
          await this.fetchProfile();
        } else {
          Swal.fire(
            "Lỗi",
            res.data.message || "Không thể cập nhật hồ sơ",
            "error"
          );
        }
      } catch (error) {
        if (error.response?.status === 422) {
          this.errors = error.response.data.errors;
        } else {
          Swal.fire("Lỗi", "Đã xảy ra lỗi hệ thống", "error");
        }
        console.error("Lỗi cập nhật profile:", error);
      }
      this.loading = false;
    },

    async changePassword() {
      if (!this.passwordForm.old_password || !this.passwordForm.password) {
        Swal.fire(
          "Cảnh báo",
          "Vui lòng nhập đầy đủ thông tin mật khẩu!",
          "warning"
        );
        return;
      }
      if (this.passwordForm.password !== this.passwordForm.re_password) {
        Swal.fire("Lỗi", "Xác nhận mật khẩu không khớp", "error");
        return;
      }
      try {
        const res = await axios.post(
          "http://localhost:8000/api/moi-gioi/update-password",
          this.passwordForm,
          {
            headers: { Authorization: `Bearer ${this.token}` },
          }
        );

        if (res.data.status === 1 || res.data.status === true) {
          Swal.fire("Thành công", "Đổi mật khẩu thành công", "success");
          this.passwordForm = {
            old_password: "",
            password: "",
            re_password: "",
          };
        } else {
          Swal.fire(
            "Lỗi",
            res.data.message || "Đổi mật khẩu thất bại",
            "error"
          );
        }
      } catch (error) {
        Swal.fire(
          "Lỗi",
          error.response?.data?.message || "Đổi mật khẩu thất bại",
          "error"
        );
      }
    },
  },
};
</script>
<style scoped>
/* Input lỗi - viền đỏ */
.form-control.is-invalid {
  border-color: #dc3545 !important;
  padding-right: calc(1.5em + 0.75rem);
  background-image: url("data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 12 12' width='12' height='12' fill='none' stroke='%23dc3545'%3e%3ccircle cx='6' cy='6' r='4.5'/%3e%3cpath stroke-linejoin='round' d='M5.8 3.6h.4L6 6.5z'/%3e%3ccircle cx='6' cy='8.2' r='.6' fill='%23dc3545' stroke='none'/%3e%3c/svg%3e");
  background-repeat: no-repeat;
  background-position: right calc(0.375em + 0.1875rem) center;
  background-size: calc(0.75em + 0.375rem) calc(0.75em + 0.375rem);
}

.form-control.is-invalid:focus {
  border-color: #dc3545;
  box-shadow: 0 0 0 0.2rem rgba(220, 53, 69, 0.25);
}

/* Thông báo lỗi */
.invalid-feedback {
  display: block;
  width: 100%;
  margin-top: 0.25rem;
  font-size: 0.875em;
  color: #dc3545;
}

.admin-profile-page {
  padding: 24px;
  background: #f7f6f3;
  min-height: 100vh;
}

.page-header h1 {
  font-size: 1.5rem;
  font-weight: 700;
  color: #0a0a4d;
  margin-bottom: 24px;
}

.profile-content {
  display: grid;
  grid-template-columns: 360px 1fr;
  gap: 24px;
}

/* Profile Sidebar */
.profile-sidebar {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.profile-card {
  background: white;
  border-radius: 16px;
  padding: 32px;
  text-align: center;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.04);
}

.profile-avatar-wrapper {
  position: relative;
  width: 140px;
  height: 140px;
  margin: 0 auto 20px;
}

.profile-avatar {
  width: 100%;
  height: 100%;
  border-radius: 50%;
  object-fit: cover;
  border: 4px solid #f8f9fa;
}

.edit-avatar-btn {
  position: absolute;
  bottom: 0;
  right: 0;
  width: 36px;
  height: 36px;
  border-radius: 50%;
  background: #7fffd4;
  color: white;
  border: 3px solid white;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 14px;
}

.profile-name {
  font-size: 1.5rem;
  font-weight: 700;
  color: #1a1a1a;
  margin: 0 0 4px;
}

.profile-role {
  font-size: 0.9rem;
  color: #0d7a5f;
  margin: 0 0 24px;
  font-weight: 500;
}

.profile-info-list {
  text-align: left;
  border-top: 1px solid #f0f0f0;
  padding-top: 20px;
}

.info-item {
  display: flex;
  justify-content: space-between;
  margin-bottom: 12px;
}

.info-item .label {
  font-size: 0.75rem;
  color: #8d93a6;
  text-transform: uppercase;
  font-weight: 600;
}

.info-item .value {
  font-size: 0.9rem;
  color: #1a1a1a;
  font-weight: 600;
}

.status-active {
  background: #7fffd4;
  color: #0d7a5f;
  padding: 4px 12px;
  border-radius: 12px;
  font-size: 0.75rem;
}

/* Security Card / Stats Card */
.security-card {
  background: white;
  border-radius: 16px;
  padding: 24px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.04);
}

.security-header {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 12px;
}

.security-icon {
  width: 40px;
  height: 40px;
  background: #fef3a2;
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 20px;
}

.security-header h3 {
  font-size: 1rem;
  font-weight: 700;
  color: #1a1a1a;
  margin: 0;
}

.security-desc {
  font-size: 0.85rem;
  color: #6c757d;
  line-height: 1.6;
}

.toggle-section {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px;
  background: #f8f9fa;
  border-radius: 12px;
}

/* Main Content */
.profile-main {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.info-card {
  background: white;
  border-radius: 16px;
  padding: 28px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.04);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 28px;
}

.card-title {
  display: flex;
  align-items: center;
  gap: 12px;
}

.title-bar {
  width: 4px;
  height: 24px;
  background: #0a0a4d;
  border-radius: 2px;
}

.card-title h3 {
  font-size: 1.1rem;
  font-weight: 700;
  color: #1a1a1a;
  margin: 0;
}

.btn-save {
  background: #0a0a4d;
  color: white;
  border: none;
  padding: 10px 24px;
  border-radius: 8px;
  font-weight: 600;
  font-size: 0.9rem;
  cursor: pointer;
  transition: all 0.2s;
}

.btn-save:hover:not(:disabled) {
  background: #1a1a6d;
  transform: translateY(-1px);
}

.btn-save:disabled {
  opacity: 0.7;
  cursor: not-allowed;
}

.form-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
  margin-bottom: 20px;
}

.form-group {
  margin-bottom: 20px;
}

.form-group label {
  display: block;
  font-size: 0.75rem;
  color: #6c757d;
  text-transform: uppercase;
  font-weight: 600;
  margin-bottom: 8px;
}

.form-control {
  width: 100%;
  padding: 12px 16px;
  border: 1px solid #e9ecef;
  border-radius: 8px;
  font-size: 0.95rem;
  background: #f8f9fa;
  transition: all 0.2s;
  font-family: inherit;
}

.form-control:focus {
  outline: none;
  border-color: #0a0a4d;
  background: white;
  box-shadow: 0 0 0 3px rgba(10, 10, 77, 0.1);
}

textarea.form-control {
  resize: vertical;
}

.password-input {
  position: relative;
}

.password-toggle {
  position: absolute;
  right: 12px;
  top: 50%;
  transform: translateY(-50%);
  background: none;
  border: none;
  cursor: pointer;
  color: #6c757d;
  font-size: 18px;
}

.password-hint {
  font-size: 0.8rem;
  color: #6c757d;
  margin: 16px 0;
  font-style: italic;
}

.btn-update-password {
  background: #e9ecef;
  color: #495057;
  border: none;
  padding: 12px 28px;
  border-radius: 8px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
}

.btn-update-password:hover {
  background: #dee2e6;
}

/* Responsive */
@media (max-width: 1024px) {
  .profile-content {
    grid-template-columns: 1fr;
  }

  .form-row {
    grid-template-columns: 1fr;
  }
}
</style>