# 🧮 App 2 – Giải Toán & API

> Ứng dụng Android có 3 Activity: Giới thiệu, Giải phương trình bậc 2 với gọi API, và WebView.  
> Tương đương với bài tập đã làm trên MIT App Inventor nhưng viết bằng Java trên Android Studio.

---

## 📑 Mục Lục

- [Tổng Quan](#tổng-quan)
- [Bước 1 – Tạo Project Mới](#bước-1--tạo-project-mới)
- [Bước 2 – Cấu Hình build.gradle](#bước-2--cấu-hình-buildgradle)
- [Bước 3 – Khai Báo Resources & Manifest](#bước-3--khai-báo-resources--manifest)
- [Bước 4 – Tạo 3 Activity](#bước-4--tạo-3-activity)
- [Bước 5 – Thiết Kế Layout 3 Activity](#bước-5--thiết-kế-layout-3-activity)
- [Bước 6 – Viết Code AboutActivity](#bước-6--viết-code-aboutactivity)
- [Bước 7 – Viết Code GiaiToanActivity](#bước-7--viết-code-giaitoanactivity)
- [Bước 8 – Viết Code WebViewActivity](#bước-8--viết-code-webviewactivity)
- [Bước 9 – Kiểm Tra AndroidManifest](#bước-9--kiểm-tra-androidmanifest)
- [Bước 10 – Chạy & Kiểm Tra](#bước-10--chạy--kiểm-tra)
- [Hình Ảnh Minh Hoạ](#hình-ảnh-minh-hoạ)
- [Tổng Kết](#tổng-kết)

---

## Tổng Quan

| Activity | Chức năng | Kỹ thuật |
|---|---|---|
| **AboutActivity** | Màn hình giới thiệu + điều hướng | Intent, Button |
| **GiaiToanActivity** | Giải PT bậc 2, gửi kết quả lên API | OkHttp POST, JSON, Thread |
| **WebViewActivity** | Hiển thị trang web trong app | WebView, WebSettings |

**Luồng điều hướng:**
```
AboutActivity (màn hình chính)
    │
    ├──[Nút Giải Toán]──► GiaiToanActivity
    │                          └── Gọi API POST → https://k58kmt.tdh.io.vn/api
    │
    └──[Nút Xem Web]────► WebViewActivity
                               └── Load https://k58kmt.tdh.io.vn?masv=MSSV
```

---

## Bước 1 – Tạo Project Mới

### 1.1 Mở Android Studio → New Project

```
File → New → New Project
```

> *(Chèn ảnh: màn hình Welcome của Android Studio)*

### 1.2 Chọn Template

- Chọn **"Empty Views Activity"**
- Click **Next**

<img width="1145" height="835" alt="image" src="https://github.com/user-attachments/assets/657ba281-24d4-4102-9295-791759d90b91" />


### 1.3 Cấu Hình Project

| Trường | Giá trị |
|---|---|
| **Name** | `GiaiToanAPI` |
| **Package name** | `com.example.giaitoanapi` |
| **Save location** | Thư mục bạn muốn lưu |
| **Language** | `Java` |
| **Minimum SDK** | `API 24 ("Nougat"; Android 7.0)` |

<img width="1145" height="835" alt="image" src="https://github.com/user-attachments/assets/9faf9343-df1b-4d02-b0e6-88f2515605d1" />

- Click **Finish** → Chờ Gradle sync xong

### 1.4 Đổi tên MainActivity thành AboutActivity

Android Studio tạo sẵn `MainActivity.java`. Ta sẽ dùng nó làm `AboutActivity`:

```
Chuột phải vào MainActivity.java → Refactor → Rename
→ Gõ: AboutActivity
→ Click Refactor
```

<img width="1424" height="900" alt="image" src="https://github.com/user-attachments/assets/b2100f0e-4416-488a-a956-587acac441ae" />

> Android Studio tự cập nhật tất cả chỗ tham chiếu, kể cả trong `AndroidManifest.xml`



---

## Bước 2 – Cấu Hình build.gradle

Mở `app/build.gradle`, thêm dependency **OkHttp** để gọi API:

```gradle
android {
    namespace = "com.example.giaitoanapi"
    compileSdk = 34

    defaultConfig {
        applicationId = "com.example.giaitoanapi"
        minSdk = 24
        targetSdk = 34
        versionCode = 1
        versionName = "1.0"
    }

    buildTypes {
        release {
            isMinifyEnabled = false
        }
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_1_8
        targetCompatibility = JavaVersion.VERSION_1_8
    }
}

dependencies {
    implementation("androidx.appcompat:appcompat:1.6.1")
    implementation("com.google.android.material:material:1.11.0")
    implementation("androidx.constraintlayout:constraintlayout:2.1.4")

    // Thư viện gọi HTTP API
    implementation("com.squareup.okhttp3:okhttp:4.12.0")
}
```

→ Click **"Sync Now"** sau khi lưu

<img width="1532" height="934" alt="image" src="https://github.com/user-attachments/assets/5d1a1f76-9f04-4bee-8e76-7162c73895ca" />

---

## Bước 3 – Khai Báo Resources & Manifest

### 3.1 `res/values/strings.xml`

```xml
<resources>
    <string name="app_name">Giải Toán API</string>

    <!-- About Activity -->
    <string name="about_title">Về Ứng Dụng</string>
    <string name="about_description">Ứng dụng minh hoạ 3 Activity:\n• Giải phương trình bậc 2\n• Gửi kết quả lên API\n• Xem kết quả qua WebView</string>
    <string name="btn_go_giai_toan">🧮 Đến Giải Toán</string>
    <string name="btn_go_webview">🌐 Xem Web</string>
    <string name="author_label">Sinh viên: [Họ tên] – [MSSV]</string>

    <!-- GiaiToan Activity -->
    <string name="giai_toan_title">Giải PT Bậc 2: ax² + bx + c = 0</string>
    <string name="hint_a">Nhập hệ số a</string>
    <string name="hint_b">Nhập hệ số b</string>
    <string name="hint_c">Nhập hệ số c</string>
    <string name="btn_giai">Giải Phương Trình</string>
    <string name="ket_qua_title">Kết quả:</string>
    <string name="api_status_title">Trạng thái API:</string>
    <string name="error_empty">Vui lòng nhập đủ a, b, c</string>
    <string name="error_invalid_number">Giá trị không hợp lệ</string>
    <string name="api_sending">Đang gửi lên server...</string>
    <string name="api_success">✅ Gửi thành công! ok=%1$d, stt=%1$d</string>
    <string name="api_error">❌ Lỗi: %1$s</string>

    <!-- WebView Activity -->
    <string name="webview_title">Kết Quả Trên Web</string>
    <string name="loading_web">Đang tải trang...</string>
</resources>
```
<img width="1590" height="951" alt="image" src="https://github.com/user-attachments/assets/20f161ac-a125-443f-a658-f8fbf4fc0197" />


### 3.2 `res/values/colors.xml`

```xml
<resources>
    <color name="primary">#1565C0</color>
    <color name="primary_dark">#003c8f</color>
    <color name="accent">#FF6F00</color>
    <color name="background">#F5F7FA</color>
    <color name="card_background">#FFFFFF</color>
    <color name="text_primary">#212121</color>
    <color name="text_secondary">#616161</color>
    <color name="success_green">#2E7D32</color>
    <color name="error_red">#C62828</color>
    <color name="divider">#E0E0E0</color>
</resources>
```
<img width="1587" height="948" alt="image" src="https://github.com/user-attachments/assets/0a04601e-1551-435c-9e1a-da23db90d80e" />

---

## Bước 4 – Tạo 3 Activity

Ta đã có `AboutActivity` (đổi tên từ MainActivity). Cần tạo thêm 2 Activity nữa.

### 4.1 Tạo GiaiToanActivity

```
Chuột phải vào package gốc (com.example.giaitoanapi)
→ New → Activity → Empty Views Activity
→ Activity Name: GiaiToanActivity
→ Layout Name: activity_giai_toan   (tự điền)
→ Launcher Activity: ☐ BỎ CHỌN (không phải màn hình chính)
→ Finish
```
<img width="1424" height="1001" alt="image" src="https://github.com/user-attachments/assets/6ef1c540-0f4b-46f1-9701-739778a4d48a" />
<img width="1137" height="834" alt="image" src="https://github.com/user-attachments/assets/5ffe9aad-1217-49d9-ac8e-af4e91521ab8" />

### 4.2 Tạo WebViewActivity

```
Chuột phải vào package gốc
→ New → Activity → Empty Views Activity
→ Activity Name: WebViewActivity
→ Layout Name: activity_webview
→ Launcher Activity: ☐ BỎ CHỌN
→ Finish
```
<img width="1145" height="826" alt="image" src="https://github.com/user-attachments/assets/d70e62ed-10c2-486f-a6e5-8bb13e76212f" />

### 4.3 Kết quả – cấu trúc project

```
app/src/main/
├── java/com/example/giaitoanapi/
│   ├── AboutActivity.java          ← Activity 1 (màn hình chính)
│   ├── GiaiToanActivity.java       ← Activity 2
│   └── WebViewActivity.java        ← Activity 3
├── res/layout/
│   ├── activity_about.xml
│   ├── activity_giai_toan.xml
│   └── activity_webview.xml
└── AndroidManifest.xml             ← Đã tự thêm cả 3 activity
```

---

## Bước 5 – Thiết Kế Layout 3 Activity

### 5.1 Layout Activity 1 – `res/layout/activity_about.xml`

> **Lưu ý:** File này ban đầu tên `activity_main.xml`, Android Studio đổi tên thành `activity_about.xml` khi bạn rename Activity, hoặc bạn tự đổi thủ công.
<img width="1198" height="953" alt="image" src="https://github.com/user-attachments/assets/439faaa3-c587-4021-a32a-4d71cfa16ae2" />

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center"
    android:padding="32dp"
    android:background="@color/background">

    <!-- Icon/Logo app -->
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="🧮"
        android:textSize="72sp"
        android:layout_marginBottom="16dp" />

    <!-- Tiêu đề -->
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/about_title"
        android:textSize="26sp"
        android:textStyle="bold"
        android:textColor="@color/primary"
        android:layout_marginBottom="16dp" />

    <!-- Mô tả -->
    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/about_description"
        android:textSize="15sp"
        android:textColor="@color/text_secondary"
        android:lineSpacingMultiplier="1.6"
        android:gravity="center"
        android:layout_marginBottom="48dp" />

    <!-- Nút sang Activity 2 -->
    <Button
        android:id="@+id/btnGoGiaiToan"
        android:layout_width="match_parent"
        android:layout_height="52dp"
        android:text="@string/btn_go_giai_toan"
        android:textSize="16sp"
        android:layout_marginBottom="12dp"
        android:backgroundTint="@color/primary" />

    <!-- Nút sang Activity 3 -->
    <Button
        android:id="@+id/btnGoWebView"
        android:layout_width="match_parent"
        android:layout_height="52dp"
        android:text="@string/btn_go_webview"
        android:textSize="16sp"
        android:layout_marginBottom="32dp"
        android:backgroundTint="@color/accent" />

    <!-- Thông tin sinh viên -->
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/author_label"
        android:textSize="13sp"
        android:textColor="@color/text_secondary" />

</LinearLayout>
```
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/16742901-8ed1-4050-8519-705f92f86fd2" />



### 5.2 Layout Activity 2 – `res/layout/activity_giai_toan.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<ScrollView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/background">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="20dp">

        <!-- Tiêu đề -->
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@string/giai_toan_title"
            android:textSize="16sp"
            android:textStyle="bold"
            android:textColor="@color/primary"
            android:layout_marginBottom="20dp"
            android:gravity="center" />

        <!-- Card nhập liệu -->
        <androidx.cardview.widget.CardView
            xmlns:app="http://schemas.android.com/apk/res-auto"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            app:cardCornerRadius="12dp"
            app:cardElevation="4dp"
            android:layout_marginBottom="16dp">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:padding="16dp">

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="Nhập hệ số"
                    android:textStyle="bold"
                    android:textSize="15sp"
                    android:textColor="@color/text_primary"
                    android:layout_marginBottom="12dp" />

                <!-- Hệ số a -->
                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="horizontal"
                    android:gravity="center_vertical"
                    android:layout_marginBottom="10dp">

                    <TextView
                        android:layout_width="40dp"
                        android:layout_height="wrap_content"
                        android:text="a ="
                        android:textStyle="bold"
                        android:textColor="@color/primary" />

                    <com.google.android.material.textfield.TextInputLayout
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        style="@style/Widget.MaterialComponents.TextInputLayout.OutlinedBox.Dense">
                        <com.google.android.material.textfield.TextInputEditText
                            android:id="@+id/etA"
                            android:layout_width="match_parent"
                            android:layout_height="wrap_content"
                            android:hint="@string/hint_a"
                            android:inputType="numberDecimal|numberSigned"
                            android:imeOptions="actionNext" />
                    </com.google.android.material.textfield.TextInputLayout>
                </LinearLayout>

                <!-- Hệ số b -->
                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="horizontal"
                    android:gravity="center_vertical"
                    android:layout_marginBottom="10dp">

                    <TextView
                        android:layout_width="40dp"
                        android:layout_height="wrap_content"
                        android:text="b ="
                        android:textStyle="bold"
                        android:textColor="@color/primary" />

                    <com.google.android.material.textfield.TextInputLayout
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        style="@style/Widget.MaterialComponents.TextInputLayout.OutlinedBox.Dense">
                        <com.google.android.material.textfield.TextInputEditText
                            android:id="@+id/etB"
                            android:layout_width="match_parent"
                            android:layout_height="wrap_content"
                            android:hint="@string/hint_b"
                            android:inputType="numberDecimal|numberSigned"
                            android:imeOptions="actionNext" />
                    </com.google.android.material.textfield.TextInputLayout>
                </LinearLayout>

                <!-- Hệ số c -->
                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="horizontal"
                    android:gravity="center_vertical"
                    android:layout_marginBottom="16dp">

                    <TextView
                        android:layout_width="40dp"
                        android:layout_height="wrap_content"
                        android:text="c ="
                        android:textStyle="bold"
                        android:textColor="@color/primary" />

                    <com.google.android.material.textfield.TextInputLayout
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        style="@style/Widget.MaterialComponents.TextInputLayout.OutlinedBox.Dense">
                        <com.google.android.material.textfield.TextInputEditText
                            android:id="@+id/etC"
                            android:layout_width="match_parent"
                            android:layout_height="wrap_content"
                            android:hint="@string/hint_c"
                            android:inputType="numberDecimal|numberSigned"
                            android:imeOptions="actionDone" />
                    </com.google.android.material.textfield.TextInputLayout>
                </LinearLayout>

                <!-- Nút giải -->
                <Button
                    android:id="@+id/btnGiai"
                    android:layout_width="match_parent"
                    android:layout_height="48dp"
                    android:text="@string/btn_giai"
                    android:textSize="15sp"
                    android:backgroundTint="@color/primary" />

            </LinearLayout>
        </androidx.cardview.widget.CardView>

        <!-- Card kết quả toán -->
        <androidx.cardview.widget.CardView
            xmlns:app="http://schemas.android.com/apk/res-auto"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            app:cardCornerRadius="12dp"
            app:cardElevation="4dp"
            android:layout_marginBottom="16dp">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:padding="16dp">

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="@string/ket_qua_title"
                    android:textStyle="bold"
                    android:textSize="15sp"
                    android:textColor="@color/text_primary"
                    android:layout_marginBottom="8dp" />

                <TextView
                    android:id="@+id/tvKetQua"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="—"
                    android:textSize="15sp"
                    android:textColor="@color/success_green"
                    android:lineSpacingMultiplier="1.6"
                    android:fontFamily="monospace" />

            </LinearLayout>
        </androidx.cardview.widget.CardView>

        <!-- Card trạng thái API -->
        <androidx.cardview.widget.CardView
            xmlns:app="http://schemas.android.com/apk/res-auto"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            app:cardCornerRadius="12dp"
            app:cardElevation="4dp">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:padding="16dp">

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="@string/api_status_title"
                    android:textStyle="bold"
                    android:textSize="15sp"
                    android:textColor="@color/text_primary"
                    android:layout_marginBottom="8dp" />

                <TextView
                    android:id="@+id/tvApiStatus"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="—"
                    android:textSize="14sp"
                    android:textColor="@color/text_secondary" />

            </LinearLayout>
        </androidx.cardview.widget.CardView>

    </LinearLayout>
</ScrollView>
```
<img width="1731" height="973" alt="image" src="https://github.com/user-attachments/assets/344547ee-f723-49d6-a246-8669cee1cf5c" />



### 5.3 Layout Activity 3 – `res/layout/activity_webview.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <!-- Loading indicator -->
    <ProgressBar
        android:id="@+id/progressBar"
        style="?android:attr/progressBarStyleHorizontal"
        android:layout_width="match_parent"
        android:layout_height="4dp"
        android:visibility="visible"
        android:indeterminate="true"
        android:progressTint="@color/primary" />

    <!-- WebView chiếm toàn bộ còn lại -->
    <WebView
        android:id="@+id/webView"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1" />

</LinearLayout>
```
<img width="1728" height="962" alt="image" src="https://github.com/user-attachments/assets/e93cf09c-4ea4-4553-bd5c-38110b282aed" />


---

## Bước 6 – Viết Code AboutActivity

Mở `AboutActivity.java`:

```java
package com.example.giaitoanapi;

import android.content.Intent;
import android.os.Bundle;
import android.widget.Button;
import androidx.appcompat.app.AppCompatActivity;

public class AboutActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_about);

        // Đặt tiêu đề ActionBar – lấy từ strings.xml, KHÔNG hardcode
        setTitle(getString(R.string.about_title));

        // Ánh xạ 2 nút
        Button btnGoGiaiToan = findViewById(R.id.btnGoGiaiToan);
        Button btnGoWebView  = findViewById(R.id.btnGoWebView);

        // Sự kiện: Click nút → mở Activity tương ứng (Cách 2: setOnClickListener)
        btnGoGiaiToan.setOnClickListener(v -> {
            Intent intent = new Intent(AboutActivity.this, GiaiToanActivity.class);
            startActivity(intent);
        });

        btnGoWebView.setOnClickListener(v -> {
            Intent intent = new Intent(AboutActivity.this, WebViewActivity.class);
            startActivity(intent);
        });
    }
}
```
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/8979a305-4551-4304-965c-7c4fddb9993d" />

---

## Bước 7 – Viết Code GiaiToanActivity

Mở `GiaiToanActivity.java`:

```java
package com.example.giaitoanapi;

import android.os.Bundle;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;
import android.widget.Toast;
import androidx.appcompat.app.AppCompatActivity;
import org.json.JSONObject;
import okhttp3.*;
import java.io.IOException;

public class GiaiToanActivity extends AppCompatActivity {

    // ★ THAY BẰNG MSSV CỦA BẠN ★
    private static final String MSSV    = "YOUR_MSSV_HERE";
    private static final String API_URL = "https://k58kmt.tdh.io.vn/api";

    private EditText etA, etB, etC;
    private TextView tvKetQua, tvApiStatus;
    private final OkHttpClient httpClient = new OkHttpClient();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_giai_toan);

        // Bật nút Back
        if (getSupportActionBar() != null) {
            getSupportActionBar().setDisplayHomeAsUpEnabled(true);
            getSupportActionBar().setTitle(getString(R.string.giai_toan_title));
        }

        // Ánh xạ View
        etA         = findViewById(R.id.etA);
        etB         = findViewById(R.id.etB);
        etC         = findViewById(R.id.etC);
        tvKetQua    = findViewById(R.id.tvKetQua);
        tvApiStatus = findViewById(R.id.tvApiStatus);
        Button btnGiai = findViewById(R.id.btnGiai);

        // Gắn sự kiện cho nút Giải (Cách 2: Lambda)
        btnGiai.setOnClickListener(v -> giaiPhuongTrinh());
    }

    /**
     * Lấy input → Tính toán → Hiển thị kết quả → Gửi API
     */
    private void giaiPhuongTrinh() {
        // 1. Lấy input từ EditText
        String sA = etA.getText().toString().trim();
        String sB = etB.getText().toString().trim();
        String sC = etC.getText().toString().trim();

        // 2. Validate input
        if (sA.isEmpty() || sB.isEmpty() || sC.isEmpty()) {
            Toast.makeText(this, getString(R.string.error_empty), Toast.LENGTH_SHORT).show();
            return;
        }

        double a, b, c;
        try {
            a = Double.parseDouble(sA);
            b = Double.parseDouble(sB);
            c = Double.parseDouble(sC);
        } catch (NumberFormatException e) {
            Toast.makeText(this, getString(R.string.error_invalid_number), Toast.LENGTH_SHORT).show();
            return;
        }

        // 3. Thuật toán giải phương trình bậc 2
        String ketLuan, abcDetail;
        double nghiem = 0;

        if (a == 0) {
            // Bậc 1: bx + c = 0
            if (b == 0) {
                if (c == 0) {
                    ketLuan   = "Vô số nghiệm";
                    abcDetail = "0 = 0 (luôn đúng)";
                } else {
                    ketLuan   = "Vô nghiệm";
                    abcDetail = "0 ≠ 0 (mâu thuẫn)";
                }
            } else {
                nghiem    = -c / b;
                ketLuan   = "Một nghiệm (PT bậc 1)";
                abcDetail = String.format("x = %.4f", nghiem);
            }
        } else {
            double delta = b * b - 4 * a * c;
            if (delta < 0) {
                ketLuan   = "Vô nghiệm";
                abcDetail = String.format("Δ = %.4f < 0", delta);
            } else if (delta == 0) {
                nghiem    = -b / (2 * a);
                ketLuan   = "Nghiệm kép";
                abcDetail = String.format("x₁ = x₂ = %.4f", nghiem);
            } else {
                double x1 = (-b + Math.sqrt(delta)) / (2 * a);
                double x2 = (-b - Math.sqrt(delta)) / (2 * a);
                nghiem    = x1; // lưu nghiệm lớn hơn
                ketLuan   = "Hai nghiệm phân biệt";
                abcDetail = String.format("x₁ = %.4f\nx₂ = %.4f", x1, x2);
            }
        }

        // 4. Hiển thị kết quả lên UI – lấy chuỗi từ resources
        String display = String.format(
            "%.2f·x² + %.2f·x + %.2f = 0\n\nKết luận: %s\n%s",
            a, b, c, ketLuan, abcDetail
        );
        tvKetQua.setText(display);
        tvApiStatus.setText(getString(R.string.api_sending));

        // 5. Gọi API trên background thread (KHÔNG gọi trực tiếp trên main thread)
        final String finalKetLuan  = ketLuan;
        final String finalAbcDetail = abcDetail;
        final double finalNghiem   = nghiem;
        final double fa = a, fb = b, fc = c;

        new Thread(() -> {
            try {
                // Tạo JSON body đúng format yêu cầu
                JSONObject inputObj  = new JSONObject();
                inputObj.put("a", fa);
                inputObj.put("b", fb);
                inputObj.put("c", fc);
                inputObj.put("name", "Giải PT bậc 2 – " + MSSV);

                JSONObject outputObj = new JSONObject();
                outputObj.put("ketluan", finalKetLuan);
                outputObj.put("abc",     finalAbcDetail);
                outputObj.put("nghiem",  finalNghiem);

                JSONObject body = new JSONObject();
                body.put("app_by", MSSV);
                body.put("input",  inputObj);
                body.put("output", outputObj);

                // Tạo HTTP POST request
                RequestBody requestBody = RequestBody.create(
                    body.toString(),
                    MediaType.parse("application/json; charset=utf-8")
                );

                Request request = new Request.Builder()
                    .url(API_URL)
                    .post(requestBody)
                    .addHeader("Content-Type", "application/json")
                    .build();

                // Thực hiện request
                try (Response response = httpClient.newCall(request).execute()) {
                    String responseStr = response.body() != null
                        ? response.body().string() : "{}";

                    JSONObject result = new JSONObject(responseStr);
                    int ok  = result.optInt("ok",  -1);
                    int stt = result.optInt("stt", -1);

                    // Cập nhật UI phải chạy trên main thread
                    runOnUiThread(() -> {
                        String msg = "✅ Thành công!\nok = " + ok + "  |  stt = " + stt;
                        tvApiStatus.setText(msg);
                        tvApiStatus.setTextColor(getColor(R.color.success_green));
                    });
                }

            } catch (Exception e) {
                // Lỗi cũng phải cập nhật UI trên main thread
                runOnUiThread(() -> {
                    String errMsg = "❌ Lỗi gửi API:\n" + e.getMessage();
                    tvApiStatus.setText(errMsg);
                    tvApiStatus.setTextColor(getColor(R.color.error_red));
                });
            }
        }).start();
    }

    @Override
    public boolean onSupportNavigateUp() {
        finish();
        return true;
    }
}
```
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/8616fe49-fc96-4a93-bbf5-0f598331147f" />

**Giải thích luồng gọi API:**

```
[Main Thread] btnGiai.onClick()
      │
      ├── Tính toán → hiển thị kết quả lên UI
      │
      └── new Thread(() -> {         ← Background thread (không block UI)
              Tạo JSON body
              OkHttp POST → API
              Nhận response JSON
              runOnUiThread(() -> {   ← Trở về main thread để cập nhật UI
                  tvApiStatus.setText(...)
              });
          }).start();
```

> **Tại sao dùng Thread?**  
> Android không cho phép gọi mạng (network) trên main thread (UI thread) vì sẽ làm đơ app. Mọi tác vụ mạng phải chạy trên background thread, sau đó dùng `runOnUiThread()` để cập nhật UI.

---

## Bước 8 – Viết Code WebViewActivity

Mở `WebViewActivity.java`:

```java
package com.example.giaitoanapi;

import android.os.Bundle;
import android.view.View;
import android.webkit.WebResourceRequest;
import android.webkit.WebSettings;
import android.webkit.WebView;
import android.webkit.WebViewClient;
import android.widget.ProgressBar;
import androidx.appcompat.app.AppCompatActivity;

public class WebViewActivity extends AppCompatActivity {

    // ★ THAY BẰNG MSSV CỦA BẠN ★
    private static final String MSSV     = "YOUR_MSSV_HERE";
    private static final String BASE_URL = "https://k58kmt.tdh.io.vn";

    private WebView webView;
    private ProgressBar progressBar;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_webview);

        // Bật nút Back trên ActionBar
        if (getSupportActionBar() != null) {
            getSupportActionBar().setDisplayHomeAsUpEnabled(true);
            getSupportActionBar().setTitle(getString(R.string.webview_title));
        }

        // Ánh xạ View
        webView     = findViewById(R.id.webView);
        progressBar = findViewById(R.id.progressBar);

        // Cấu hình WebView
        WebSettings settings = webView.getSettings();
        settings.setJavaScriptEnabled(true);      // Bật JS
        settings.setDomStorageEnabled(true);       // Bật localStorage
        settings.setLoadWithOverviewMode(true);    // Scale vừa màn hình
        settings.setUseWideViewPort(true);         // Dùng viewport đầy đủ
        settings.setCacheMode(WebSettings.LOAD_DEFAULT); // Dùng cache

        // Mở link TRONG app, không nhảy ra Chrome
        webView.setWebViewClient(new WebViewClient() {

            @Override
            public boolean shouldOverrideUrlLoading(WebView view, WebResourceRequest request) {
                // Trả về false = WebView tự xử lý link, không mở browser ngoài
                return false;
            }

            @Override
            public void onPageStarted(WebView view, String url, android.graphics.Bitmap favicon) {
                super.onPageStarted(view, url, favicon);
                progressBar.setVisibility(View.VISIBLE); // Hiện loading
            }

            @Override
            public void onPageFinished(WebView view, String url) {
                super.onPageFinished(view, url);
                progressBar.setVisibility(View.GONE); // Ẩn loading
                // Cập nhật tiêu đề ActionBar theo title trang web
                if (getSupportActionBar() != null && view.getTitle() != null) {
                    getSupportActionBar().setTitle(view.getTitle());
                }
            }
        });

        // Load URL với MSSV của sinh viên
        String url = BASE_URL + "?masv=" + MSSV;
        webView.loadUrl(url);
    }

    /**
     * Nút Back vật lý: nếu WebView còn trang trước thì goBack,
     * không thì thoát Activity
     */
    @Override
    public void onBackPressed() {
        if (webView.canGoBack()) {
            webView.goBack();
        } else {
            super.onBackPressed();
        }
    }

    @Override
    public boolean onSupportNavigateUp() {
        finish();
        return true;
    }
}
```
<img width="1919" height="1056" alt="image" src="https://github.com/user-attachments/assets/c4fa8a29-1ae0-4d9f-b59c-6da30eebcdab" />

---

## Bước 9 – Kiểm Tra AndroidManifest

Mở `AndroidManifest.xml` và đảm bảo đúng như sau:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.giaitoanapi">

    <!-- ► BẮT BUỘC: quyền Internet cho gọi API và WebView -->
    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.GiaiToanAPI"
        android:usesCleartextTraffic="true">
        <!--
            usesCleartextTraffic="true"
            Cho phép HTTP (không mã hóa) nếu URL dùng http:// thay vì https://
            Nếu API dùng https:// thì không bắt buộc, nhưng để cho an toàn
        -->

        <!-- Activity 1: AboutActivity là màn hình LAUNCHER (khởi động đầu tiên) -->
        <activity
            android:name=".AboutActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <!-- Activity 2: GiaiToanActivity -->
        <activity
            android:name=".GiaiToanActivity"
            android:exported="false"
            android:parentActivityName=".AboutActivity" />

        <!-- Activity 3: WebViewActivity -->
        <activity
            android:name=".WebViewActivity"
            android:exported="false"
            android:parentActivityName=".AboutActivity" />

    </application>
</manifest>
```
<img width="1738" height="958" alt="image" src="https://github.com/user-attachments/assets/d51a852e-7eb0-4747-9881-00b50f846da0" />

**Giải thích các thuộc tính quan trọng:**

| Thuộc tính | Giá trị | Ý nghĩa |
|---|---|---|
| `exported="true"` | AboutActivity | Hệ thống có thể khởi động Activity này |
| `exported="false"` | 2 Activity còn lại | Chỉ app nội bộ mới gọi được |
| `parentActivityName` | `.AboutActivity` | Nút Back trên ActionBar về đâu |
| `uses-permission INTERNET` | — | Bắt buộc để gọi API và load WebView |
| `usesCleartextTraffic` | true | Cho phép HTTP (không chỉ HTTPS) |

---

## Bước 10 – Chạy & Kiểm Tra

### 10.1 Kết nối thiết bị / khởi động Emulator

```
Tools → Device Manager
→ Click ▶ bên cạnh thiết bị ảo để khởi động
```

Hoặc cắm thiết bị thật qua USB (đã bật Developer Options + USB Debugging).

### 10.3 Build & Run

```
Click ▶ (Run) trên toolbar
Hoặc: Shift + F10
→ Chọn thiết bị → OK
```

<img width="1717" height="862" alt="image" src="https://github.com/user-attachments/assets/727c2474-25dc-4620-9a05-4227730e0628" />


### 10.4 Test từng chức năng

**Test Activity 1 – About:**
```
☐ App mở được, hiển thị màn hình About
☐ Nút "Đến Giải Toán" → chuyển sang Activity 2
☐ Nút "Xem Web" → chuyển sang Activity 3
☐ Nút Back trên thiết bị hoạt động bình thường
```
<img width="1763" height="981" alt="image" src="https://github.com/user-attachments/assets/ca180f02-9f4b-45a1-90b6-3233728005cb" />

**Test Activity 2 – Giải Toán:**
```
☐ Nhập a=1, b=5, c=6 → kết quả: x1=3.0, x2=2.0
☐ Nhập a=1, b=2, c=5  → kết quả: Vô nghiệm (Δ < 0)
☐ Nhập a=1, b=-2, c=1 → kết quả: Nghiệm kép x=1.0
☐ Nhập a=0, b=2, c=4  → kết quả: Một nghiệm x=-2.0
☐ Để trống → hiện thông báo "Vui lòng nhập đủ a, b, c"
☐ Sau khi giải → trạng thái API hiện "Đang gửi..."
☐ Vài giây sau → API trả về "✅ Thành công! ok=1, stt=XXXX"
```
<img width="372" height="579" alt="image" src="https://github.com/user-attachments/assets/8b5232bc-d227-4afe-bf21-ab52f9eab03d" />



**Test Activity 3 – WebView:**
```
☐ Trang web load được (cần có Internet)
☐ URL hiển thị đúng: https://k58kmt.tdh.io.vn?masv=MSSV_CỦA_BẠN
☐ ProgressBar hiện khi đang load, ẩn khi xong
☐ Nút Back thiết bị: nếu đã điều hướng trong web → goBack()
☐ Nút Back trên ActionBar → về AboutActivity
```

> *(Chèn ảnh: WebView hiển thị trang web)*

### 10.5 Kiểm tra API bằng Log

Mở **Logcat** trong Android Studio (Alt+6) để xem log:

```
Filter: tag = "API" hoặc search "GiaiToan"
```

> *(Chèn ảnh: Logcat hiển thị kết quả API)*

---

## Hình Ảnh Minh Hoạ

> *(Chèn ảnh theo từng bước)*

| Bước | Mô tả | Ảnh |
|---|---|---|
| Bước 1 | Tạo project – điền thông tin | *(chèn ảnh)* |
| Bước 4 | Tạo 3 Activity – cấu trúc package | *(chèn ảnh)* |
| Bước 9 | AndroidManifest.xml hoàn chỉnh | *(chèn ảnh)* |
| Bước 10 | App chạy – Activity 1 About | *(chèn ảnh)* |
| Bước 10 | App chạy – Activity 2 Giải Toán + kết quả API | *(chèn ảnh)* |
| Bước 10 | App chạy – Activity 3 WebView | *(chèn ảnh)* |

---

## Tổng Kết

### Những gì đã làm được

| Kỹ thuật | Áp dụng |
|---|---|
| **3 Activity** | Mỗi Activity một chức năng riêng biệt |
| **Intent** | Điều hướng giữa các Activity |
| **AndroidManifest** | Khai báo Activity, quyền Internet |
| **OkHttp POST** | Gửi JSON lên server API |
| **Background Thread** | Gọi mạng không block UI |
| **runOnUiThread** | Cập nhật UI từ background thread |
| **WebView** | Hiển thị trang web trong app |
| **WebViewClient** | Xử lý loading, back navigation |
| **Resources** | `@string`, `@color` – không hardcode |
| **Event Handling** | `setOnClickListener` + Lambda |

### JSON gửi lên API

```json
{
  "app_by": "2051012345",
  "input": {
    "a": 1,
    "b": -5,
    "c": 6,
    "name": "Giải PT bậc 2 – 2051012345"
  },
  "output": {
    "ketluan": "Hai nghiệm phân biệt",
    "abc": "x₁ = 3.0000\nx₂ = 2.0000",
    "nghiem": 3.0
  }
}
```

### Response nhận về

```json
{
  "ok": 1,
  "stt": 1234
}
```

---

*App 2 hoàn thành – tương đương bài tập MIT App Inventor nhưng viết bằng Java trên Android Studio.*
