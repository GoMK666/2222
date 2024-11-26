### **التفاصيل الكاملة لنظام إدخال وعرض بيانات الموظفين داخل شبكة محلية باستخدام Synology DiskStation وملف CSV**

---

### **كيف يعمل النظام؟**

1. **الجزء الخاص بالموظفين:**
   - يقوم الموظف بفتح صفحة ويب عبر متصفح الشبكة المحلية وإدخال بياناته (الاسم، الهاتف، البريد الإلكتروني، الملاحظة).
   - يتم إرسال البيانات إلى ملف CSV يتم تخزينه على جهاز Synology DiskStation.

2. **الجزء الخاص بالمدير:**
   - يقوم المدير بفتح صفحة ويب عبر متصفح الشبكة المحلية.
   - يتم عرض البيانات المخزنة في ملف CSV بطريقة منظمة داخل جدول.

---

### **مكان حفظ البيانات**

- **ملف CSV**:
  يتم حفظ البيانات المدخلة في ملف داخل جهاز **Synology DiskStation**، تحديدًا في مجلد **`/volume1/web/`**، الذي يستخدم لاستضافة موقعك.
  
---

### **الكودات كاملة**

#### **1. صفحة إدخال بيانات الموظفين (`employee.html`)**

```html
<!DOCTYPE html>
<html lang="ar">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>إدخال بيانات الموظفين</title>
</head>
<body>
    <h1>إدخال بيانات الموظفين</h1>
    <form action="save_data.php" method="POST">
        <label>الاسم:</label>
        <input type="text" name="name" required><br><br>

        <label>رقم الهاتف:</label>
        <input type="text" name="phone" required><br><br>

        <label>البريد الإلكتروني:</label>
        <input type="email" name="email" required><br><br>

        <label>ملاحظة:</label>
        <textarea name="note" rows="4" required></textarea><br><br>

        <button type="submit">إرسال</button>
    </form>
</body>
</html>
```

---

#### **2. كود PHP لحفظ البيانات في ملف CSV (`save_data.php`)**

```php
<?php
// مسار ملف CSV داخل Synology
$csv_file = "/volume1/web/employee_data.csv";

// استقبال البيانات من النموذج
$name = $_POST["name"] ?? "غير محدد";
$phone = $_POST["phone"] ?? "غير محدد";
$email = $_POST["email"] ?? "غير محدد";
$note = $_POST["note"] ?? "غير محدد";

// فتح ملف CSV وإضافة البيانات
$file = fopen($csv_file, "a");
if ($file) {
    fputcsv($file, [$name, $phone, $email, $note]);
    fclose($file);
    echo "تم حفظ البيانات بنجاح!";
} else {
    echo "حدث خطأ أثناء حفظ البيانات!";
}
?>
```

---

#### **3. صفحة عرض بيانات المدير (`manager.html`)**

```html
<!DOCTYPE html>
<html lang="ar">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>بيانات الموظفين</title>
</head>
<body>
    <h1>بيانات الموظفين</h1>
    <table border="1">
        <thead>
            <tr>
                <th>الاسم</th>
                <th>رقم الهاتف</th>
                <th>البريد الإلكتروني</th>
                <th>ملاحظة</th>
            </tr>
        </thead>
        <tbody>
            <?php
            // مسار ملف CSV داخل Synology
            $csv_file = "/volume1/web/employee_data.csv";

            // قراءة البيانات من CSV
            if (($file = fopen($csv_file, "r")) !== false) {
                while (($data = fgetcsv($file)) !== false) {
                    echo "<tr>";
                    foreach ($data as $cell) {
                        echo "<td>" . htmlspecialchars($cell) . "</td>";
                    }
                    echo "</tr>";
                }
                fclose($file);
            } else {
                echo "<tr><td colspan='4'>لا توجد بيانات</td></tr>";
            }
            ?>
        </tbody>
    </table>
</body>
</html>
```

---

### **الأخطاء التي قد تواجهها**

1. **الخطأ: لا يمكن كتابة البيانات إلى ملف CSV**
   - **السبب:** لا توجد صلاحيات كتابة إلى المجلد `/volume1/web/`.
   - **الحل:** قم بإعطاء صلاحيات الكتابة للمجلد باستخدام Synology File Manager.

2. **الخطأ: صفحة المدير لا تعرض البيانات**
   - **السبب:** ملف `employee_data.csv` فارغ أو غير موجود.
   - **الحل:** تأكد من أن ملف `save_data.php` يعمل بشكل صحيح ويضيف البيانات إلى ملف CSV.

3. **الخطأ: لا يمكن الوصول إلى صفحة الويب**
   - **السبب:** Web Station لم يتم تفعيله على Synology DiskStation.
   - **الحل:** قم بتفعيل Web Station من إعدادات Synology.

4. **الخطأ: اتصال الأجهزة بالشبكة غير صحيح**
   - **السبب:** الأجهزة ليست على نفس الشبكة المحلية.
   - **الحل:** تأكد من اتصال جميع الأجهزة بنفس الشبكة.

---

### **خطوات الاختبار**

1. قم برفع الملفات (`employee.html`, `save_data.php`, `manager.html`) إلى مجلد `/volume1/web/` في جهاز Synology.
2. افتح متصفحًا على جهاز موظف واكتب:
   ```
   http://[IP-ADDRESS-OF-SYNOLOGY]/employee.html
   ```
3. أدخل البيانات في النموذج واضغط على **إرسال**.
4. انتقل إلى صفحة المدير:
   ```
   http://[IP-ADDRESS-OF-SYNOLOGY]/manager.html
   ```
5. تحقق من ظهور البيانات داخل الجدول.

---

### **التحسينات الممكنة**

1. **إرسال إشعار عند إدخال البيانات**:
   - يمكن إضافة كود لإرسال إشعار عبر البريد الإلكتروني باستخدام `mail()`.

2. **استخدام واجهة أكثر تفاعلية**:
   - استبدال HTML التقليدي بـ HTML5 + CSS أو مكتبات مثل Bootstrap لتحسين واجهة المستخدم.

3. **إضافة نظام بحث للمدير**:
   - يمكن تطوير وظيفة تسمح للمدير بالبحث عن موظف معين حسب اسمه أو بريده الإلكتروني.

4. **إضافة تصدير البيانات**:
   - إنشاء زر لتصدير البيانات في ملف Excel أو PDF.
