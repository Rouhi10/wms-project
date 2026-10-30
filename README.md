# سامانه یکپارچه مدیریت انبار و فاکتور (WMS)

سامانه مدیریت انبار (Warehouse Management System) و فاکتور چندفروشگاهی برای یک اپراتور/ادمین واحد.
مدیریت کاتالوگ کالا، طرف‌حساب‌ها، فاکتورهای خرید/فروش و برگشتی، کاردکس موجودی، مانده حساب‌ها و گزارش‌ها.

## پشته فناوری

| لایه | فناوری |
|------|--------|
| Backend | Python 3.11+, Django 5.x, Django REST Framework |
| Database | PostgreSQL |
| Frontend | React 18+, Vite, Tailwind CSS, TanStack Query, React Hook Form |
| Authentication | Session-based (تک‌نقش ادمین/اپراتور در فاز ۱) |
| Methodology | Extreme Programming (XP) — TDD، release های کوچک |

## ساختار مخزن (مونو‌ریپو)

```
wms-project/
├── backend/     # Django + DRF — اپ‌ها: catalog, parties, invoicing, audit, reports
├── frontend/    # React SPA (Vite + Tailwind)
├── docs/        # مشخصات فنی، معماری و نمودارها
├── README.md
└── .gitignore
```

## قواعد کلیدی کسب‌وکار (غیرقابل مذاکره)

1. **سلسله‌مراتب فیزیکی:** انبار ← قفسه ← محصول (محل ثابت پیش‌فرض).
2. **حذف نرم:** محصول/طرف‌حساب فقط `is_active=false` می‌شوند؛ هرگز حذف فیزیکی نمی‌شوند.
3. **موجودی اولیه:** با ثبت `INITIAL_STOCK` بدون فاکتور خرید.
4. **Stock Guard:** نهایی‌سازی فاکتور فروش/برگشت خرید با موجودی منفی ممنوع است.
5. **چرخه فاکتور:** `Draft → Final → Canceled`؛ حرکت موجودی فقط در `Final`؛ قیمت خرید روی خط فاکتور قفل می‌شود.
6. **مالیات:** نرخ از دسته/زیر‌دسته کالا؛ ترتیب محاسبه: پایه → تخفیف خط → تخفیف سراسری → مالیات → قابل پرداخت.
7. **برگشتی:** فقط با فاکتور والد finalized و حداکثر به تعداد خرید/فروش اولیه.
8. **ردپای مالی:** تغییر قیمت، حذف نرم و نهایی/لغو فاکتورها در `audit_log` ثبت غیرقابل تغییر می‌شود.

## شروع سریع

> وضعیت فعلی: **پیش از Iteration 1** — فقط ساختار مخزن آماده است.

```bash
# Backend (به‌روزرسانی بعد از Iteration 1)
cd backend
python -m venv .venv
.venv\Scripts\activate        # ویندوز | macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt

# Frontend (به‌روزرسانی بعد از Iteration 1)
cd frontend
npm install
npm run dev
```

## توسعه

- **روش:** XP — هر داستان کاربری با تست شروع می‌شود (TDD)، هر commit در CI سبز است.
- **تست:** `pytest` (backend) و `vitest` (frontend).
- **نقشه راه:** ۳ تکرار — (۱) کاتالوگ/مکان/طرف‌حساب، (۲) موتور فاکتور و Stock Guard، (۳) برگشتی/کاردکس/گزارش/چاپ.

### استراتژی شاخه‌بندی

- `main` — همیشه پایدار و قابل انتشار؛ فقط release و hotfix از این‌جا جدا می‌شود.
- `develop` — خط اصلی در حال ساخت؛ همه داستان‌های کاربری این‌جا merge می‌شوند.
- `feature/US-XX-slug` — یک شاخه کوتاه‌عمر برای هر داستان کاربری، از `develop` جدا و پس از merge حذف می‌شود. نمونه:

  ```bash
  git checkout develop
  git checkout -b feature/US-01-warehouse-setup
  # ... TDD: تست سبز، کد، commit ...
  git checkout develop
  git merge --no-ff feature/US-01-warehouse-setup
  git branch -d feature/US-01-warehouse-setup
  ```

## مستندات

جزئیات معماری، شمای دیتابیس، قرارداد API و نقشه TDD در سند طرح معماری (تأییدشده) و پوشه `docs/` نگهداری می‌شود.
