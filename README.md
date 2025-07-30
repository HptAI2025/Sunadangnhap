# Tài liệu quy trình xác thực trong Suna

## Mục lục
1. [Quy trình đăng ký tài khoản](#quy-trình-đăng-ký-tài-khoản)
2. [Quy trình đăng nhập](#quy-trình-đăng-nhập)
3. [Quản lý phiên đăng nhập](#quản-lý-phiên-đăng-nhập)
4. [Bảo mật](#bảo-mật)
5. [Tích hợp với Suna Agent](#tích-hợp-với-suna-agent)

---

## Quy trình đăng ký tài khoản

### 1. Tổng quan về quy trình đăng ký

Suna sử dụng Supabase làm nền tảng xác thực và quản lý người dùng. Quy trình đăng ký bao gồm các bước sau:

#### Các phương thức đăng ký:
1. **Đăng ký bằng email/mật khẩu**: Người dùng nhập email và mật khẩu để tạo tài khoản
2. **Đăng ký bằng Google**: Sử dụng OAuth với Google
3. **Đăng ký bằng GitHub**: Sử dụng OAuth với GitHub

### 2. Quy trình đăng ký chi tiết

#### Đăng ký bằng email/mật khẩu:

1. **Nhập thông tin**: Người dùng nhập email, mật khẩu và xác nhận mật khẩu trong form đăng ký
   - Giao diện form được định nghĩa trong `/frontend/src/app/auth/page.tsx`
   - Xử lý form được thực hiện bởi hàm `handleSignUp`

2. **Xác thực dữ liệu**: Hệ thống kiểm tra:
   - Email hợp lệ (phải có ký tự @)
   - Mật khẩu đủ dài (tối thiểu 6 ký tự)
   - Mật khẩu và xác nhận mật khẩu khớp nhau

3. **Gửi yêu cầu đăng ký**: Khi người dùng nhấn "Create account", hệ thống gọi hàm `signUp` trong `/frontend/src/app/auth/actions.ts`
   - Hàm này gọi `supabase.auth.signUp()` để tạo tài khoản trong Supabase
   - Cấu hình `emailRedirectTo` để chuyển hướng người dùng sau khi xác thực email

4. **Xác thực email**: 
   - Supabase gửi email xác thực đến địa chỉ email người dùng
   - Người dùng nhấp vào liên kết trong email để xác thực tài khoản
   - Liên kết chuyển hướng người dùng đến `/auth/callback` với mã xác thực

5. **Xử lý callback**: 
   - Route `/auth/callback` trong `/frontend/src/app/auth/callback/route.ts` xử lý mã xác thực
   - Gọi `supabase.auth.exchangeCodeForSession()` để đổi mã xác thực lấy phiên đăng nhập
   - Chuyển hướng người dùng đến trang dashboard

6. **Cài đặt Suna Agent**:
   - Sau khi đăng nhập thành công, hệ thống kiểm tra xem người dùng mới được tạo trong vòng 10 phút qua không
   - Nếu là người dùng mới, hệ thống gọi API để cài đặt Suna Agent cho người dùng đó
   - Quá trình này được xử lý bởi hàm `checkAndInstallSunaAgent` trong `/frontend/src/lib/utils/install-suna-agent.ts`

7. **Gửi email chào mừng**:
   - Hệ thống gửi email chào mừng đến người dùng mới
   - Xử lý bởi hàm `sendWelcomeEmail` trong `/frontend/src/app/auth/actions.ts`

#### Đăng ký bằng Google/GitHub:

1. **Khởi tạo OAuth**: Người dùng nhấp vào nút "Sign in with Google" hoặc "Sign in with GitHub"
   - Các component `GoogleSignIn` và `GitHubSignIn` xử lý quá trình này

2. **Xác thực với nhà cung cấp**: 
   - Supabase xử lý quá trình OAuth với Google/GitHub
   - Người dùng đăng nhập vào tài khoản Google/GitHub và cấp quyền cho ứng dụng

3. **Callback và tạo phiên**: 
   - Sau khi xác thực thành công, nhà cung cấp chuyển hướng người dùng trở lại ứng dụng
   - Supabase xử lý callback và tạo phiên đăng nhập
   - `AuthProvider` trong `/frontend/src/components/AuthProvider.tsx` lắng nghe sự kiện đăng nhập và cập nhật trạng thái

4. **Cài đặt Suna Agent**: Tương tự như đăng ký bằng email

### 3. Luồng dữ liệu

1. **Frontend → Supabase**: Yêu cầu đăng ký/đăng nhập
2. **Supabase → Email người dùng**: Gửi email xác thực
3. **Email → Frontend**: Người dùng nhấp vào liên kết xác thực
4. **Frontend → Supabase**: Đổi mã xác thực lấy phiên đăng nhập
5. **Frontend → Backend**: Cài đặt Suna Agent và gửi email chào mừng

---

## Quy trình đăng nhập

### 1. Các phương thức đăng nhập

Suna hỗ trợ ba phương thức đăng nhập chính:
1. **Đăng nhập bằng email/mật khẩu**
2. **Đăng nhập bằng Google** (OAuth)
3. **Đăng nhập bằng GitHub** (OAuth)

### 2. Quy trình đăng nhập chi tiết

#### Đăng nhập bằng email/mật khẩu:

1. **Hiển thị form đăng nhập**:
   - Giao diện form được định nghĩa trong `/frontend/src/app/auth/page.tsx`
   - Form hiển thị các trường email và mật khẩu

2. **Nhập thông tin đăng nhập**:
   - Người dùng nhập email và mật khẩu
   - Frontend thực hiện kiểm tra cơ bản (email hợp lệ, mật khẩu đủ dài)

3. **Xử lý đăng nhập**:
   - Khi người dùng nhấn "Sign in", hàm `handleSignIn` trong `page.tsx` được gọi
   - Hàm này gọi server action `signIn` từ `/frontend/src/app/auth/actions.ts`

4. **Xác thực với Supabase**:
   - Server action `signIn` gọi `supabase.auth.signInWithPassword()`
   - Supabase kiểm tra thông tin đăng nhập và trả về kết quả

5. **Xử lý kết quả đăng nhập**:
   - Nếu đăng nhập thành công, server action trả về `{ success: true, redirectTo: '/dashboard' }`
   - Frontend chuyển hướng người dùng đến trang dashboard
   - Nếu đăng nhập thất bại, hiển thị thông báo lỗi

6. **Cập nhật trạng thái đăng nhập**:
   - `AuthProvider` phát hiện sự kiện đăng nhập thông qua `onAuthStateChange`
   - Cập nhật state với thông tin người dùng và phiên đăng nhập

#### Đăng nhập bằng Google/GitHub:

1. **Khởi tạo OAuth**:
   - Người dùng nhấp vào nút "Sign in with Google" hoặc "Sign in with GitHub"
   - Các component `GoogleSignIn` và `GitHubSignIn` xử lý quá trình này

2. **Chuyển hướng đến nhà cung cấp**:
   - Supabase xử lý việc chuyển hướng người dùng đến trang đăng nhập của Google/GitHub
   - Người dùng đăng nhập vào tài khoản và cấp quyền cho ứng dụng

3. **Callback và xử lý phiên**:
   - Sau khi xác thực thành công, nhà cung cấp chuyển hướng người dùng trở lại ứng dụng
   - Route `/auth/callback` trong `/frontend/src/app/auth/callback/route.ts` xử lý callback
   - Gọi `supabase.auth.exchangeCodeForSession()` để đổi mã xác thực lấy phiên đăng nhập

4. **Chuyển hướng sau đăng nhập**:
   - Sau khi xử lý callback, người dùng được chuyển hướng đến trang dashboard
   - `AuthProvider` phát hiện sự kiện đăng nhập và cập nhật trạng thái

### 3. Luồng dữ liệu trong quá trình đăng nhập

#### Đăng nhập bằng email/mật khẩu:
```
Người dùng → Frontend → Server Action → Supabase → Server Action → Frontend → Chuyển hướng
```

#### Đăng nhập bằng OAuth:
```
Người dùng → Frontend → Supabase → Google/GitHub → Supabase → Callback Route → Frontend → Chuyển hướng
```

---

## Quản lý phiên đăng nhập

### 1. Lưu trữ phiên
- Supabase lưu trữ thông tin phiên trong cookie
- `createClient` trong `/frontend/src/lib/supabase/server.ts` và `/frontend/src/lib/supabase/client.ts` xử lý việc đọc/ghi cookie

### 2. Kiểm tra phiên hiện tại
- Khi ứng dụng khởi động, `AuthProvider` gọi `supabase.auth.getSession()` để kiểm tra phiên hiện tại
- Nếu phiên hợp lệ, người dùng được coi là đã đăng nhập

### 3. Làm mới phiên
- Supabase tự động làm mới token khi hết hạn
- `AuthProvider` lắng nghe sự kiện `TOKEN_REFRESHED` để cập nhật trạng thái

### 4. Đăng xuất
- Khi người dùng đăng xuất, hàm `signOut` trong `actions.ts` được gọi
- Hàm này gọi `supabase.auth.signOut()` để hủy phiên
- `AuthProvider` phát hiện sự kiện `SIGNED_OUT` và cập nhật trạng thái

---

## Bảo mật

### 1. Bảo mật trong quá trình đăng ký/đăng nhập
- **HTTPS**: Tất cả các yêu cầu API đều sử dụng HTTPS
- **Xử lý mật khẩu**: Mật khẩu được xử lý bởi Supabase, không lưu trữ trực tiếp trong ứng dụng
- **Xác thực email**: Yêu cầu xác thực email để kích hoạt tài khoản
- **Xác thực hai yếu tố**: Có hỗ trợ xác thực qua điện thoại (phone verification)
- **Phiên có thời hạn**: Phiên đăng nhập có thời hạn và được làm mới tự động
- **Bảo vệ CSRF**: Supabase xử lý bảo vệ CSRF thông qua cơ chế cookie

### 2. Xử lý lỗi đăng nhập
- **Hiển thị thông báo lỗi**: Nếu đăng nhập thất bại, thông báo lỗi được hiển thị bằng toast
- **Xử lý lỗi callback**: Nếu có lỗi trong quá trình callback, người dùng được chuyển hướng đến trang đăng nhập với thông báo lỗi
- **Quên mật khẩu**: Người dùng có thể yêu cầu đặt lại mật khẩu thông qua liên kết "Forgot password?"

---

## Tích hợp với Suna Agent

### 1. Quy trình kích hoạt Suna Agent sau khi xác thực

1. **Frontend phát hiện đăng nhập thành công**:
   - Khi người dùng đăng nhập thành công, `AuthProvider` trong frontend phát hiện sự kiện đăng nhập thông qua hàm lắng nghe `onAuthStateChange` của Supabase
   - Cụ thể, trong file `/frontend/src/components/AuthProvider.tsx`, dòng 49-82 lắng nghe sự kiện `SIGNED_IN`

2. **Frontend gọi hàm cài đặt Suna Agent**:
   - Khi phát hiện sự kiện `SIGNED_IN`, frontend gọi hàm `checkAndInstallSunaAgent` (dòng 66)
   - Hàm này kiểm tra xem người dùng có phải mới tạo trong 10 phút gần đây không (dựa vào thời gian tạo tài khoản)

3. **Frontend gọi API đến Backend để cài đặt Suna Agent**:
   - Nếu là người dùng mới, frontend gọi hàm `installSunaForNewUser` trong file `/frontend/src/lib/utils/install-suna-agent.ts`
   - Hàm này thực hiện một request POST đến endpoint `/admin/suna-agents/install-user/${userId}` của backend
   - Request này bao gồm API key quản trị (`KORTIX_ADMIN_API_KEY`) để xác thực

4. **Backend xử lý yêu cầu cài đặt Suna Agent**:
   - Backend nhận request, xác thực API key
   - Backend tạo và cấu hình Suna Agent cho người dùng mới
   - Backend trả về ID của agent đã tạo

### 2. Luồng dữ liệu
```
Người dùng → Supabase (xác thực) → Frontend (phát hiện đăng nhập) → Backend (cài đặt Suna Agent)
```

### 3. Tích hợp với các tính năng khác

1. **Theo dõi phương thức đăng nhập**:
   - Suna theo dõi phương thức đăng nhập gần đây nhất của người dùng
   - Sử dụng `useAuthMethodTracking` từ `/frontend/src/lib/stores/auth-tracking.ts`

2. **Chuyển hướng thông minh**:
   - Hệ thống hỗ trợ chuyển hướng người dùng đến trang họ đã truy cập trước khi đăng nhập
   - Sử dụng tham số `returnUrl` trong URL